## 6주차

📌 **643~664**
### 프록시 동등성

프록시 동등성에 대해 주의할 점들을 살펴보자
먼저 프록시 동등성을 비교하기 위한 코드다.

```java
@Entity
public class User {
    @Id
    @GeneratedValue
    private Long id;
    
    private String name;
    
    // 생성자, getter, setter 생략
    
    @Override
    public boolean equals(Object o) {
        if (this == o) return true;
        if (!(o instanceof User)) return false;
        User user = (User) o;
        return Objects.equals(getId(), user.getId());
    }
    
    // hashcode 생략
}
```

여기서 주의할 점 몇가지를 소개한다.

1. **Instanceof 사용**: 프록시 객체는 원본 엔티티를 상속받기 때문에 , `instanceof` 검사 시 주의가 필요하다.
   (JPA가 생성하는 프록시 객체는 원본 엔티티 클래스를 상속받는다. 예를 들어, User 엔티티의 프록시는 **User 클래스를 상속받은 새로운 클래스**라는 의미)

2.** getter 사용** : equals를 보면 `user.id`가 아닌 `user.getId()`로 값을 가져오는 것을 볼 수 있다.
이유는 다음과 같다.
프록시 객체에서는 user.id와 같이 필드에 접근하려 하면, **프록시 객체는 실제 값을 가지고 있지 않기 때문**에 `null`을 반환하게 된다.

📌 **684~702**

1. **트랜잭션의 격리 수준**
   -트랜잭션 격리 수준은 **동시에 실행되는 트랜잭션 간의 데이터 접근**을 어느정도까지 **허용**할지를 결정한다.
   -`4가지 격리 수준`을 가지는데 다음과 같다.
   **(READ_UNCOMMITTED, READ_COMMITTED, REPEATABLE_READ, SERIALIZABLE)**
   -격리 수준이 **낮을수록 성능은 좋을 수 있지만 **`동시성 문제`(Dirty_Read,Repeatable_Read...)가 발생할 수 있다.

2. **낙관적 락**
   -낙관적 락은 트랜잭션이 데이터를 변경할 때 **데이터가 변경되었는지 확인하는 방식**이다.
   -이를 통해 동시성 제어를 할 수 있으며, 동시성이 낮은 환경에서 유용하다.
   -`@Version` 어노테이션을 통해 구현할 수있다.

3. **비관적 락**
   -비관적 락은 **데이터를 변경하기 전에 명시적으로 락을 걸어 다른 트랜잭션의 접근을 막는 방식**이다.
   -비관적 락은 데이터 무결성을 보장하지만, 리소스를 오래 점유할 수 있어 성능이 떨어질 수 있다.
   -`@Lock `어노테이션을 통해 비관적 락을 구현할 수 있다.

📌 **703~723**
### 낙관적 락
`낙관적 락`은 **충돌이 발생하지 않을 것이라고 가정**하고, 실제로 데이터를 수정할 때 충돌 확인

다음과 같은 특징
- **동시성 문제가 적은 환경**
- 락을 사용하지 않으니 성능상 이점
- 버전 정보를 이용함

```java
@Entity
public class Product {
    @Id
    private Long id;
    
    private String name;
    
    @Version
    private Long version;
    
    // getters and setters
}
```
```java
@Transactional
public void updateProduct(Long id, String newName) {
    Product product = entityManager.find(Product.class, id);
    product.setName(newName);
    // 저장 시 버전 체크, 충돌 시 OptimisticLockException 발생
}
```

### 비관적 락
`비관적 락`은 **충돌이 발생할 것이라고 가정**하고, 데이터를 읽는 시점에 락을 건다.

다음과 같은 특징
- **동시성 문제가 자주 발생하는 환경**
- 동시 요청이 많은 경우 성능 저하 가능성

```java
@Entity
public class Account {
    @Id
    private Long id;
    
    private BigDecimal balance;
    
    // getters and setters
}
```

```java
@Transactional
public void withdraw(Long id, BigDecimal amount) {
    Account account = entityManager.find(Account.class, id, LockModeType.PESSIMISTIC_WRITE);
    if (account.getBalance().compareTo(amount) >= 0) {
        account.setBalance(account.getBalance().subtract(amount));
    } else {
        throw new InsufficientBalanceException();
    }
}
```

- 위 예제에서는 보는 것과 같이 `LockModeType.PESSIMISTIC_WRITE`를 사용함
- 이는 데이터베이스에서 쓰기 락을 획득했다는 의미
- 이는 **다른 트랜잭션의 읽기와 쓰기를 모두 방지**한다.
- 따라서 가장 엄격한 비관적 락이라 데이터를 수정할 예정이고, 다른 트랜잭션의 간섭을 완전히 차단해야할 경우 사용

### 다음 비관적 락 모드
`PESSIMISTIC_READ` : **다른 트랜잭션의 읽기는 허용하지만 쓰기는 방지**
- 주로 데이터를 읽어야 하지만 다른 트랜잭션이 그 데이터를 변경하는 것을 원하지 않을 때 사용.

```java
@Entity
public class StockItem {
    @Id
    private Long id;
    private String name;
    private int quantity;
    // getters and setters
}

@Transactional
public int checkStockQuantity(Long itemId) {
    StockItem item = entityManager.find(StockItem.class, itemId, LockModeType.PESSIMISTIC_READ);
    // 재고 수량을 확인하는 동안 다른 트랜잭션이 수정하지 못하도록 함
    return item.getQuantity();
}
```

`LockModeType.PESSIMISTIC_FORCE_INCREMENT`: PESSIMISTIC_WRITE와 유사하지만, 엔티티에 버전 필드가 있는 경우 **강제로 버전을 증가**시킨다.
따라서 이는 **낙관적 락과 비관적 락을 혼합**하여 사용하는 방식이라고 생각하면 된다.

```java
@Entity
public class Document {
    @Id
    private Long id;
    private String content;
    
    @Version
    private Long version;
    // getters and setters
}

@Transactional
public void updateDocument(Long documentId, String newContent) {
    Document document = entityManager.find(Document.class, documentId, LockModeType.PESSIMISTIC_FORCE_INCREMENT);
    document.setContent(newContent);
    // 저장 시 버전이 자동으로 증가됨
}
```

결론적으로 어떤 **상황에 맞는 락**을 선택할 수 있는지 알아보자

- **단순 읽기 작업이 많은 경우**: `낙관적 락` 또는 `PESSIMISTIC_READ`
- **빈번한 수정이 있지만 버전 관리가 필요 없는 경우**: `PESSIMISTIC_WRITE`
- **수정과 동시에 버전 관리가 필요한 중요 데이터**: `PESSIMISTIC_FORCE_INCREMENT`

## 2차 캐시
`2차 캐시`는 여러 세션이나 트랜잭션 간에 **데이터를 공유**할 수 있게 해주는 기능이다.
이를 통해 **데이터베이스 접근을 줄이고** 애플리케이션의 성능을 향상시킬 수 있다.

지금 예제는 여러 캐시 전략 중 `CacheConcurrencyStrategy`를 사용한 예제이다.
(사용하려면 의존성 추가해줘야함)
```java
import org.hibernate.annotations.Cache;
import org.hibernate.annotations.CacheConcurrencyStrategy;

@Entity
@Cache(usage = CacheConcurrencyStrategy.READ_WRITE)
public class Product {
    @Id
    private Long id;
    
    private String name;
    private BigDecimal price;
    
    // getters and setters
}
```

- **캐시 사용 예시**
```java
@Service
public class ProductService {
    
    @PersistenceContext
    private EntityManager entityManager;
    
    public Product getProduct(Long id) {
        // 첫 번째 조회 시 데이터베이스에서 가져옴
        Product product = entityManager.find(Product.class, id);
        
        // 두 번째 조회 시 2차 캐시에서 가져옴 (데이터베이스 접근 없음)
        Product cachedProduct = entityManager.find(Product.class, id);
        
        return cachedProduct;
    }
}
```
다음과 같이 **첫 번째 find 호출** 시 데이터베이스에서 데이터를 가져오고 2차 캐시에 저장한다. 이후 동일한 ID로 조회할 때는 **데이터베이스 접근 없이 2차 캐시**에서 데이터를 가져온다.

근데 `CacheConcurrencyStrategy.READ_WRITE` 그래서 뭔데??
- `READ_WRITE 전략`은 **캐시된 데이터의 읽기와 쓰기 작업을 모두 허용**
- 여러 트랜잭션이 동시에 같은 데이터에 접근할 때 **일관성을 유지**

**?? 읽기 쓰기 허용인데 어떻게 데이터의 일관성을 유지한다는 거지 ??**

- `읽기 작업` : 캐시된 데이터가 있으면 캐시에서 읽고,
  캐시에 없으면 데이터베이스에서 읽고 캐시에 저장
- `쓰기 작업` : 트랜잭션이 커밋되기 전에는 캐시를 직접 수정하지 않고,
  대신 'soft lock'이라는 메커니즘을 사용