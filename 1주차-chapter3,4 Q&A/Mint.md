### 1. [0701] 90~111p

- 엔티티 매니저 팩토리: 엔티티 매니저를 생산하는 공장
- 엔티티 매니저: 엔티티 관리자, 엔티티 저장 가상 DB
- 영속성 컨텍스트: 엔티티를 영구 저장하는 환경
- 1차 캐시: 영속성 컨텍스트가 가지고 있는 내부의 캐시
- 쓰기 지연: 트랜잭션 커밋 전까지 내부 쿼리 저장소에 쿼리 모아두기
- flush: 쓰기 지연 sql 저장소에 모인 쿼리를 DB에 반영, 영속성 컨텍스트와 DB 동기화
- 변경 감지(dirty checking): 엔티티의 변경 사항을 데이터베이스에 자동 반영

❓ 식별자를 기준으로 조회하는 find() 메서드를 호출할 때 플러시가 실행되지 않는 이유?

### 2. [0702] 111~135

- 지연로딩: 실제 객체 대신 프록시 객체를 로딩하고, 해당 객체를 실제 사용할 때 영속성 컨텍스트를 통해 데이터를 불러오는 방법
- 병합(merge()): 준영속/비영속 상태의 엔티티를 받아서 새로운 영속 상태의 엔티티를 반환
- @Entity: 테이블과 매핑할 클래스에 필수로 붙임
- @Table: 엔티티와 매핑할 테이블
- 기본키 매핑: 직접 할당, 자동 생성(IDENTITY, SEQUENCE, TABLE)

❓ @Entity name vs @Table name의 차이점

❓ @Entity 적용 시 final 클래스, enum, interface, inner 클래스에 사용이 불가능한 이유?

### 3. [0703] 136~155

- 직접 할당 전략: 직접 식별자를 할당한다.
- identity 전략: db에 엔티티를 저장해서 식별자 값을 획득한 후 영속성 컨텍스트에 저장한다.
- sequence 전략: db 시퀀스에서 식별자 값을 획득한 후 영속성 컨텍스트에 저장한다.
- table 전략: 키 생성 전용 테이블을 만든다, db 시퀀스를 흉내내는 전략
- @Column: 객체 필드를 테이블 컬럼에 매핑
- @Enumerated: enum 타입 매핑
- @Temporal: 날짜 타입 매핑
- @Lob: BLOB, CLOB 타입과 매핑
- @Transient: db에 저장하지 않고 조회하지도 않음
- @Access: jpa가 엔티티에 접근하는 방식을 지정

❓ table 전략과 최적화 (141p 부분)에 대한 이해

❓ @Temporal 대신 time 패키지를 사용하는지?

❓ @Lob 사용 시 어느 정도의 긴 문자열일 때 사용하는 게 좋을지?

❓ 프로퍼티 접근 타입에서 getter를 사용할 때 필드가 생기는 이유?

### 4. [0704] 156~175

- 객체: 참조를 통해 연관관계 찾음(엔티티 그래프 탐색)
- 테이블: 외래키 join을 통해 연관관계 찾음
- 방향: 단방향, 양방향
- 다중성: 다대일, 일대다, 일대일, 다대다
- 연관관계 주인: 양방향 연관관계 시 연관관계 주인이 필요
- @ManyToOne: 기본 FetchType.EAGER
- @OneToMany: 기본 FetchType.LAZY

❓ jpa에서 엔티티를 저장할 때 연관된 모든 엔티티가 영속상태여야 하는 이유? (174p)

### 5. [0705] 176~195

- jpql: 객체를 대상으로 하고 sql보다 간결하다.
- em.update()는 없다. 단순히 엔티티만 변경하면 커밋 시 flush가 일어나며 변경 감지 기능이 작동하여 변경사항이 db에 자동 반영한다.
- 단방향 연관관계: 언제나 연관관계의 주인이다. (외래키 관리자이다.)
- 양방향 연관관계: @ManyToOne, @JoinColumn <-> @OneToMany(mappedBy=반대쪽 매핑의 필드 이름)
- 연관관계의 주인 (외래키 관리자): 다대일, 일대다에서 다
- 연관관계 편의 메서드: 객체까지 고려하면 양쪽 다 관계를 맺어야 한다. + 기존 관계 제거

### 6. [0706] 196~217

- 연관관계 매핑: 다중성(일대일 vs 일대다), 단방향/양방향, 연관관계의 주인
- 다대일: 다가 연관관계의 주인이다.

❓ 연관관계 메서드는 다쪽에 두는 게 좋을까? 일쪽에 두는 게 좋을까? 아님 양쪽에 두는 것이 좋을까? (198p, 208p)

❓ 208p에서 왜 기존 연관관계 제거는 안 했을까?

- 일대다: 엔티티 매핑 테이블과 외래키 존재 테이블이 달라서 연관관계 처리를 위해 update sql을 추가로 실행해야 한다.

❓ @JoinColumn은 외래키의 위치가 아닌가? 일대다 예제 이해가 안 된다. (210p)

- 일대다 양방향 매핑은 존재하지 않는다. 즉, @OneToMany는 연관관계의 주인이 될 수 없다.
- 일대다 양방향을 굳이 만든다면, 일대다 단방향 매핑 반대편에 같은 외래키를 사용하는 다대일 단방향 매핑을 읽기 전용으로 하나 추가하면 된다.

- 일대일: 주 테이블에 외래키를 두고 대상 테이블을 참조하는 방법 vs 대상 테이블에 외래키를 두는 방법

### 7. [0707] 218~241

- 주테이블 vs 대상테이블: 비즈니스 관점에서 조회를 더 많이 하는 곳이 주테이블
- 일대일 단방향: 대상 테이블에 외래키가 있는 단방향 관계는 jpa에서 지원하지 않는다.
- 일대일 양방향: 대상 테이블을 연관관계 주인으로 만든다.
- 프록시를 사용할 때 외래키를 직접 관리하지 않는 일대일 관계는 지연 로딩으로 설정해도 즉시 로딩된다.
- 다대다: 보통 일대다, 다대일 관계로 풀어내는 연결 테이블을 사용한다.
    - 연결 테이블을 자동으로 처리하는 방법: @ManyToMany (편리하지만 연결 테이블에 컬럼을 추가할 수 없으므로 자주 사용되지 않음)
    - 연결 테이블을 직접 만드는 방법
- 식별 관계: @IdClass, 부모 테이블의 기본 키를 받아서 자신의 기본키+외래키로 사용하는 것
- 비식별 관계: 부모 테이블의 기본키는 외래키로 사용하고 자신의 기본키는 새로운 식별자를 사용 (단순하고 편리하게 ORM 매핑 가능)

❓ 식별 테이블의 부모 테이블이란 무엇일까?

❓ 237p에서 일대다, 다대일, 일대일의 연관관계 메서드의 차이는 무엇일까?

## 질문 & 답변

### 1. ❓ 식별자를 기준으로 조회하는 find() 메서드를 호출할 때 플러시가 실행되지 않는 이유? - 108p

💬 find() 메서드는 먼저 1차 캐시에서 엔티티를 찾는다.
1차 캐시에 엔티티가 존재하면 데이터베이스에 접근할 필요가 없기 때문에 플러시가 실행되지 않는다.
> 플러시는 영속성 컨텍스트의 변경 내용을 데이터베이스에 동기화하는 작업이다.

하지만 `find()` 메서드를 호출할 때 **1차 캐시에 엔티티가 없어서 데이터베이스에서 조회해야 하는 경우** 플러시가 호출된다.

✅ JPQL이나 Criteria API를 사용하여 조회할 때는 데이터베이스에 SQL을 실행해야 하므로 플러시가 자동으로 호출된다.
이는 영속성 컨텍스트의 변경 내용을 데이터베이스에 반영하여 조회 결과의 일관성을 유지하기 위함이다.

```java
// 식별자를 기준으로 엔티티 조회 (플러시 실행되지 않음)
Member member=entityManager.find(Member.class,1L);

// JPQL을 사용하여 엔티티 조회 (플러시 자동 호출)
        List<Member> members=entityManager.createQuery("SELECT m FROM Member m",Member.class)
        .getResultList();
```

### 2. ❓ @Entity name vs @Table name의 차이점 - 123p

💬 @Entity의 name : **JPA에서 사용할 엔티티의 이름**을 지정하는 데 사용되고, 주로 JPQL에서 엔티티를 참조할 때 사용된다. (기본값은 클래스 이름을 그대로 사용)

@Table의 name : **매핑할 데이터베이스 테이블의 이름**을 지정하는 데 사용된다 (기본값은 엔티티 이름)

> @Entity의 name과 @Table의 name이 다른 경우, JPQL에서는 @Entity의 name을 사용하고 실제 데이터베이스 테이블과의 매핑에는 @Table의 name이 사용된다.

```java

@Entity(name = "Member")
@Table(name = "member_table")
public class Member {
}
```

### 3. ❓ @Entity 적용 시 final 클래스, enum, interface, inner 클래스에 사용이 불가능한 이유? - 123p

💬 @Entity가 적용된 클래스는 JPA에 의해 관리되고 프록시 객체를 생성할 수 있어야 한다.
final 클래스, enum, interface, inner 클래스는 아래와 같은 이유로 @Entity로 사용할 수 없다.

- final 클래스: 상속이 불가능하므로 프록시 객체를 생성할 수 없다.
- enum: 실질적인 final class, 상속이 불가능하고 인스턴스 생성이 제한되어 있어 프록시 객체를 생성할 수 없다.
- interface: 인스턴스를 직접 생성할 수 없어 엔티티로 사용할 수 없다.
- inner 클래스: 외부 클래스에 대한 참조가 필요하므로 프록시 객체 생성이 어렵다.

@Entity를 적용하려면 일반적인 클래스여야 하며, 상속 및 프록시 객체 생성이 가능해야 한다.

```java
// final 클래스에 @Entity 적용 (불가능)
@Entity
public final class Member {
}

// enum에 @Entity 적용 (불가능)
@Entity
public enum MemberType {
}
```

#### enum:

- enum은 열거형 상수 집합으로, 클래스 내부에서 미리 정의된 상수 인스턴스만 존재한다(컴파일 타임에 결정).
- 즉, enum의 인스턴스는 런타임에 동적으로 생성되는 것이 아니라 컴파일 타임에 이미 결정되어 있습니다.
- 따라서 enum은 런타임에 동적으로 프록시 객체를 생성할 수 없다.

#### 프록시 객체와 상속:

- JPA에서 프록시 객체는 지연 로딩(Lazy Loading)을 지원하기 위해 사용된다.
- 프록시 객체는 실제 엔티티 객체를 상속받아 생성되며, 엔티티 객체의 기능을 대신 수행한다.
- 이를 위해 프록시 객체는 실제 엔티티 객체의 메서드를 오버라이드하고, 필요한 경우 실제 엔티티 객체를 데이터베이스에서 로딩한다.

#### 3. inner 클래스:

- inner 클래스는 **외부 클래스의 인스턴스에 대한 참조**를 암묵적으로 가지고 있다.
- 즉, inner 클래스의 인스턴스는 외부 클래스의 인스턴스 없이는 생성될 수 없다.
- 프록시 객체는 내부적으로 리플렉션을 사용하여 동적으로 생성되는데, inner 클래스의 경우 외부 클래스의 인스턴스 참조를 파라미터로 전달해야 한다.

### 4. ❓ table 전략과 최적화 - 141p

💬
시퀀스 전략은 다음 식별자 조회후 자동으로 시퀀스가 증가하므로 update 쿼리를 하지 않는다.
테이블 전략은 select + update 두 쿼리 모두 나간다.

```java

@Entity
@TableGenerator(
        name = "MEMBER_SEQ_GENERATOR",
        table = "MY_SEQUENCES",
        pkColumnValue = "MEMBER_SEQ",
        allocationSize = 50
)
public class Member {
    @Id
    @GeneratedValue(strategy = GenerationType.TABLE, generator = "MEMBER_SEQ_GENERATOR")
    private Long id;
}
```

### 5. ❓ @Temporal 대신 time 패키지를 사용하는 것이 어떨까?? - 149p

💬 JPA 2.2부터는 자바 8의 time 패키지(`java.time.*`)를 지원하므로 `@Temporal` 어노테이션 대신 `LocalDate`, `LocalDateTime`, `LocalTime` 등의 타입을
필드에 사용할 수 있다.
데이터베이스의 DATE, TIMESTAMP 타입과 매핑된다.

```java

@Entity
public class Member {
    private LocalDate birthDate;
    private LocalDateTime createdAt;
}
```

### 6. ❓ @Lob 사용 시 어느 정도의 긴 문자열일 때 사용하는 게 좋을지? - 151p

💬 @Lob 어노테이션은 **CLOB**(Character Large Object)이나 **BLOB**(Binary Large Object) 타입과 매핑될 때 사용한다.
일반적으로 긴 텍스트 데이터나 바이너리 데이터를 저장할 때 사용된다.
보통 수백 자 이상의 긴 문자열이나 대용량 바이너리 데이터를 저장할 때 @Lob을 사용하는 것이 좋다.

> @Lob을 사용하면 데이터 조회 시 성능 저하가 발생할 수 있으므로, 너무 많은 @Lob 컬럼을 사용하는 것은 피하자.

```java

@Entity
public class Post {
    @Id
    private Long id;

    private String title;

    @Lob
    private String content;
}
```

### 7. ❓ 프로퍼티 접근 타입에서 getter를 사용할 때 필드가 존재하는 이유? - 153p

💬 프로퍼티 접근 타입(@Access(AccessType.PROPERTY))을 사용하면 JPA는 엔티티의 필드가 아닌 getter 메서드를 사용하여 데이터에 접근한다.
하지만 JPA는 내부적으로 엔티티 인스턴스를 생성할 때 리플렉션을 사용하므로, **해당 프로퍼티에 대한 필드가 존재해야 한다.** 즉 저장이 안된다.

따라서 프로퍼티 접근 타입을 사용할 때는 대응하는 필드를 선언해야 한다.

```java

@Entity
@Access(AccessType.PROPERTY)
public class Member {
    private Long id;
    private String name;

    @Id
    public Long getId() {
        return id;
    }

    public void setId(Long id) {
        this.id = id;
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }
}
```

### 8. ❓ jpa에서 엔티티를 저장할 때 연관된 모든 엔티티가 영속상태여야 하는 이유? (174p)

💬 JPA에서 엔티티를 저장할 때 연관된 모든 엔티티가 영속 상태여야 하는 이유는 데이터 무결성을 유지하고 일관성 있는 상태로 데이터베이스에 저장하기 위함이다.

영속 상태의 엔티티는 JPA에 의해 관리되며, 변경 감지(Dirty Checking)와 같은 기능을 통해 데이터베이스와 동기화된다.
만약 연관된 엔티티 중 일부가 비영속 상태라면, 해당 엔티티의 변경 사항이 데이터베이스에 반영되지 않아 데이터 무결성이 깨질 수 있다.

예를 들어, 주문(Order) 엔티티와 주문 상품(OrderItem) 엔티티가 있을 때, 주문을 저장할 때 주문 상품도 함께 저장되어야 한다.
이때 주문 상품 엔티티가 비영속 상태라면, 주문은 저장되지만 주문 상품은 저장되지 않아 데이터 무결성이 깨진다.

```java

@Entity
public class Order {
    @Id
    private Long id;

    @OneToMany(mappedBy = "order", cascade = CascadeType.PERSIST)
    private List<OrderItem> orderItems = new ArrayList<>();
}

@Entity
public class OrderItem {
    @Id
    private Long id;

    @ManyToOne
    @JoinColumn(name = "order_id")
    private Order order;
}

    // 주문 저장 시 연관된 주문 상품도 함께 저장
    Order order = new Order();
    OrderItem orderItem1 = new OrderItem();
    OrderItem orderItem2 = new OrderItem();

order.getOrderItems().add(orderItem1);
        order.getOrderItems().add(orderItem2);

        entityManager.persist(order);
```

> Order 엔티티를 저장할 때, cascade = CascadeType.PERSIST 옵션으로 인해 연관된 OrderItem 엔티티도 함께 영속 상태로 만들어 저장할 수 있다.
> 이를 통해 데이터 무결성을 유지할 수 있다.

### 9. ❓ 연관관계 메서드는 다쪽에 두는 게 좋을까? 일쪽에 두는 게 좋을까? 아님 양쪽에 두는 것이 좋을까? (198p, 208p)

💬 일반적으로 다쪽에 두는 것이 좋다.

- 다쪽에 연관관계 메서드를 두면 외래 키를 관리하는 쪽에서 관계를 설정하므로 직관적이다.
- 일쪽에 연관관계 메서드를 두면 컬렉션을 관리하는 쪽에서 관계를 설정하므로 편리할 수 있다. 그러나 외래키를 직접 관리하지 않는 쪽에서 관계를 설정하게 된다.
- 양쪽에 연관관계 메서드를 두는 것은 어느 쪽에서든 관계 설정이 가능하지만 코드 중복이 발생한다.

양방향 관계의 경우, 한 쪽에만 연관관계 메서드를 두고 그 메서드 내에서 양쪽 관계를 모두 설정하는 것이 좋다.
연관관계의 주인 쪽에 연관관계 메서드를 두는 것이 좋다. (주로 다(Many) 쪽이 연관관계의 주인)

```java
public class Order {
    @ManyToOne
    private Member member;

    public void setMember(Member member) {
        // 기존 관계 제거
        if (this.member != null) {
            this.member.getOrders().remove(this);
        }
        this.member = member;
        // 새로운 관계 설정
        if (member != null) {
            member.getOrders().add(this);
        }
    }
}
```

### 10. ❓ 208p에서 왜 기존 연관관계 제거는 안 했을까?

💬 편의상 생략된 것 같다. 그러나 연관관게 변경시 기존 연관관곌르 제거하는 것이 안전하다.

```java
public void setTeam(Team team){
        // 기존 연관관계 제거
        if(this.team!=null){
        this.team.getMembers().remove(this);
        }

        this.team=team;
        team.getMembers().add(this);
        }
```

### 11. ❓ @JoinColumn은 외래키의 위치가 아닌가? 일대다 예제 이해가 안 된다. (210p)

💬 @JoinColumn은 외래 키를 매핑할 때 사용하는 어노테이션으로, 일반적으로 외래 키가 있는 테이블에 위치한다.

일대다 단방향 관계에서는 외래 키가 다쪽 테이블에 있음에도 불구하고, 일쪽 엔티티에 @JoinColumn을 사용하여 매핑한다. 직관적이지 않다.

```java

@Entity
public class Team {
    @Id
    private Long id;

    @OneToMany
    @JoinColumn(name = "team_id")
    private List<Member> members = new ArrayList<>();
}

@Entity
public class Member {
    @Id
    private Long id;
}
```

> 객체 모델에서는 Team이 members를 가지고 있지만, 데이터베이스에서는 Member 테이블이 TEAM_ID를 가진다.
> 이 불일치를 해결하기 위해 @JoinColumn을 Team 엔티티에 두고, JPA가 이를 해석하여 Member 테이블에 외래 키를 생성한다.

이는 @JoinColumn이 외래키의 물리적인 위치를 나타내는 것이아니라. 논리적인 관계를 표현하기 때문이다.
> 실무에서는 일대다 단방향보다는 다대일 양방향 관계를 주로 사용한다.

### 12. ❓ 식별 테이블의 부모 테이블이란 무엇일까? - 228p

💬 식별 관계에서 부모 테이블이란, **자식 테이블의 기본 키(Primary Key)에 포함되는 외래 키(Foreign Key)를 제공하는 테이블**이다.

식별 관계는 자식 테이블의 기본 키가 부모 테이블의 기본 키를 포함하는 관계이다.
즉, 자식 테이블의 기본 키는 부모 테이블의 기본 키와 자신의 키로 구성된다.

```
Department (부모 테이블)
- id (PK)
- name

Student (자식 테이블)
- id (PK, FK to Department.id)
- department_id (PK, FK to Department.id)
- name
```

식별 관계는 주로 부모-자식 관계가 명확하고, 자식 테이블이 부모 테이블에 강하게 의존하는 경우에 사용된다.
그러나 식별 관계는 테이블 구조를 복잡하게 만들기 때문에 꼭 필요한 경우에만 사용하는 것이 좋다.

### 13. ❓ 237p에서 일대다, 다대일, 일대일의 연관관계 메서드의 차이는 무엇일까?

💬

1. 일대다 연관관계 메서드:
    - 일쪽 엔티티에서 다쪽 엔티티의 컬렉션을 관리한다.
    - 다쪽 엔티티에는 별도의 연관관계 메서드가 없다.
    - 예시:
    ```java
      public class Team {
      private List<Member> members = new ArrayList<>();
   
          public void addMember(Member member) {
              this.members.add(member);
              if (member.getTeam() != this) {
                  member.setTeam(this);
              }
          }
      }
     ```

2. 다대일 연관관계 메서드:
    - 다쪽 엔티티에서 일쪽 엔티티를 참조하는 필드를 관리한다.
    - 보통 다쪽 엔티티에 연관관계 메서드를 작성한다.
    - 예시:
    ```java
      public class Member {
         private Team team;
      
         public void setTeam(Team team) {
         // 기존 팀과의 관계를 제거
         if (this.team != null) {
         this.team.getMembers().remove(this);
         }
         this.team = team;
         // 새로운 팀에 멤버 추가
         if (team != null) {
         team.getMembers().add(this);
         }
         }
      }
   ```

3. 일대일 연관관계 메서드:
    - 주 테이블(또는 외래 키를 가진 쪽)에서 연관관계 메서드를 작성한다.
    - 주 테이블에 연관관계 메서드를 작성하는 경우:
    ```java
    public class Member {
        private Locker locker;
         
        public void setLocker(Locker locker) {
            this.locker = locker;
            if (locker != null && locker.getMember() != this) {
                locker.setMember(this);
            }
        }
   }
   ```

- 일대다 연관관계에서는 일쪽 엔티티에서 연관관계 메서드를 작성하고,
- 다대일 연관관계에서는 보통 다쪽 엔티티에 연관관계 메서드를 작성한다.
- 일대일 연관관계에서는 주 테이블에 연관관계 메서드를 작성하면 된다.
- 다대다(Many-to-Many) 연관관계에서 연관관계 메서드를 작성할 때는 일반적으로 양쪽 엔티티 모두에 연관관계 메서드를 작성한다.
