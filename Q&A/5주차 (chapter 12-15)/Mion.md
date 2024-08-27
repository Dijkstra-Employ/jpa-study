## 5주차

📌 **538~558**
1. 데이터 JPA에서 `findOne`과 `getOne`의 차이점

1-1. **findOne()**
- **즉시 조회**: 실제 데이터베이스에서 엔티티를 즉시 조회한다.

1-2. **getOne()**
- **지연로딩**: getOne() 메서드는 요청한 ID에 해당하는 엔티티의 **프록시 객체를 반환**한다.
- 따라서 실제 데이터베이스에서 즉시 조회하지 않고, **필요한 시점에 데이터를 로딩**한다.

2. `Page`과 `Pageable` 정리

2-1. **Pageable**
- **페이지 번호, 페이지 크기, 정렬 방식 등**을 포함하여 데이터베이스에서 특정 페이지의 데이터를 가져오는데 필요한 정보를 제공한다.
- 즉, 데이터베이스에서 **특정 페이지의 데이터를 요청**할 때 사용

- 주요 메소드
  `getPageNumber()`: 현재 페이지 번호를 반환한다.
  `getPageSize()`: 한 페이지에 포함될 데이터 수를 반환한다.
  `getSort()`: 정렬 정보를 반환한다.

2-2. **Page**
- 실제로 데이터가 포함된 페이지를 나타내는 인터페이스
- `Pageable`의 요청에 따라 데이터베이스에서 조회된 결과를 포함한다.

- 주요 메서드:
  `getContent()`: 현재 페이지의 데이터 리스트를 반환한다.
  `getTotalElements()`: 전체 데이터 수를 반환한다.
  `getTotalPages()`: 전체 페이지 수를 반환한다.
  `hasNext()`: 다음 페이지가 존재하는지 여부를 반환한다.

간단한 예시

### 인터페이스
```java
public interface EmployeeRepository extends JpaRepository<Employee, Long> {
    Page<Employee> findAll(Pageable pageable);
}
```

### 서비스
```java
@Service
public class EmployeeService {

    @Autowired
    private EmployeeRepository employeeRepository;

    public Page<Employee> getEmployees(int page, int size) {
        Pageable pageable = PageRequest.of(page, size); // Pageable 생성
        return employeeRepository.findAll(pageable); // Page 반환
    }
}
```

### 컨트롤러
```java
@RestController
public class EmployeeController {

    @Autowired
    private EmployeeService employeeService;

    @GetMapping("/employees")
    public Page<Employee> getEmployees(@RequestParam int page, @RequestParam int size) {
        return employeeService.getEmployees(page, size);
    }
}
```

📌 **559~577**

## 명세 적용
- 명세를 사용하면 동적 쿼리를 더 유연하고 안전하게 작성 가능

### 비교

**명세를 사용하지 않는 경우**
```java
--- 생략
    public List<User> findUsers(String name, Integer minAge, String city) {
        List<User> allUsers = userRepository.findAll();
        
        return allUsers.stream()
            .filter(user -> name == null || user.getName().contains(name))
            .filter(user -> minAge == null || user.getAge() >= minAge)
            .filter(user -> city == null || user.getCity().equals(city))
            .collect(Collectors.toList());
    }
}
```

**명세를 사용하는 경우**
```java
--- 생략
    public List<User> findUsers(String name, Integer minAge, String city) {
        Specification<User> spec = Specification.where(null);
        
        if (name != null) {
            spec = spec.and((root, query, cb) -> 
                cb.like(root.get("name"), "%" + name + "%"));
        }
        if (minAge != null) {
            spec = spec.and((root, query, cb) -> 
                cb.greaterThanOrEqualTo(root.get("age"), minAge));
        }
        return userRepository.findAll(spec);
    }
}
```

**1. 성능**
- 명세를 사용하지 않는 방식은 모든 사용자를 `메모리`에 로드한 후 필터링
- 하지만 명세를 사용하면 `데이터베이스`에서 필요한 데이터만 가져온다.

**2. 확장성**
- 명세는 새로운 `검색 조건`을 추가하기 쉽다.

예를 들어 `이메일 검색`은 다음과 같이 나타낼 수 있다.
```java
spec = spec.and((root, query, cb) -> 
        cb.like(root.get("email"), "%" + emailDomain));
```

**3. 복잡한 쿼리**
- 명세는 OR 조건이나 서브쿼리 등 복잡한 조건 쉽게 추가 가능

정리하자면 `명세`는 보통 **대량의 데이터를 다룰 때 성능상의 이점이 크다**고 생각한다.


📌 **578~ 603**
## OSIV
`OSIV`는 **영속성 컨텍스트를 뷰까지** 열어둔다는 뜻이다.
왜 그래야하냐?  모든 문제는 엔티티가 **프리젠테이션 계층에서 준영속 상태**이기 때문에 그렇다.
따라서 영속상태로 유지된다면 뷰에서도 `지연 로딩`을 사용할 수 있다는 장점이 있다.

### 문제정의
OSIV의 필요성을 코드를 통해 알아보자.
- 엔티티
```JAVA
@Entity
public class Order {
    @Id
    private Long id;
    
    @ManyToOne(fetch = FetchType.LAZY)
    private Customer customer;
    
    @OneToMany(mappedBy = "order", fetch = FetchType.LAZY)
    private List<OrderItem> items;
    
    // getters and setters
}

@Entity
public class Customer {
    @Id
    private Long id;
    private String name;
    
    // getters and setters
}

@Entity
public class OrderItem {
    @Id
    private Long id;
    
    @ManyToOne
    private Order order;
    
    private String productName;
    
    // getters and setters
}
```
- 컨트롤러
```JAVA
@Controller
public class OrderController {
    
    @Autowired
    private OrderService orderService;
    
    @GetMapping("/order/{id}")
    public String getOrder(@PathVariable Long id, Model model) {
        Order order = orderService.getOrderById(id);
        model.addAttribute("order", order);
        return "orderDetails";
    }
}
```
컨트롤러에서 주문 정보를 조회하고 뷰에 전달하는 상황을 생각!
- 서비스
```JAVA
@Service
@Transactional
public class OrderService {
    
    @Autowired
    private OrderRepository orderRepository;
    
    public Order getOrderById(Long id) {
        return orderRepository.findById(id).orElseThrow(() -> new OrderNotFoundException(id));
    }
}
```
이후 뷰를 통해 주문정보를 나타내려고 하는 상황이다. (뷰는 타임리프로 일단은 표시하겠다.)
```html
<h1>Order Details</h1>
<p>Order ID: <span th:text="${order.id}"></span></p>
<p>Customer Name: <span th:text="${order.customer.name}"></span></p>
<h2>Order Items:</h2>
<ul>
    <li th:each="item : ${order.items}" th:text="${item.productName}"></li>
</ul>
```

위 상황에서는 어떤 문제가 발생할까?

**트랜잭션이 이미 서비스 계층에서 종료**되었기 때문에, 뷰에서 `order.customer.name`이나 `order.items`에 접근하려고 하면 `LazyInitializationException`이 발생할것이다.

`OSIV`를 사용하면 **뷰 렌더링 시점까지 영속성 컨텍스트가 활성화**되어있어, **지연 로딩된 엔티티에 접근**할 수 있다.

이런 OSIV를 사용하지 않고 문제를 해결하려면
1. **필요한 데이터를 모두 로딩**
2. **DTO사용하여 필요한 데이터만 조회**


근데 보통 **도메인 엔티티에 특정 메서드가 캡슐화**되어있다면, 이 메서드를 호출하는 곳이 **서비스 계층**이라면 `OSIV`를 적용할 필요가 없지 않나?

📌 **603~622**
Q. `Set`은 엔티티를 추가할 때 중복된 엔티티가 있는지 비교해야 한다. 따라서 **엔티티를 추가할 때 지연 로딩된 컬렉션을 초기화**한다.  (해당 구문에서 초기화의 의미가 모호해서 찾아보았다...)

다음과 같은 예시가 있다고 가정해보자.

```java
@Entity
public class Author {
    @Id
    @GeneratedValue
    private Long id;
    
    private String name;
    
    @OneToMany(mappedBy = "author", fetch = FetchType.LAZY)
    private Set<Book> books = new HashSet<>();

    // 생성자, getter, setter 등
}

@Entity
public class Book {
    @Id
    @GeneratedValue
    private Long id;
    
    private String title;
    
    @ManyToOne
    private Author author;

    // 생성자, getter, setter 등
}
```

```java
@Transactional
public void addBookToAuthor(Long authorId, Book newBook) {
    Author author = entityManager.find(Author.class, authorId);
    author.getBooks().add(newBook);
    newBook.setAuthor(author);
}
```

1. `author.getBooks()`를 호출하면, **프록시 객체**가 반환된다.
2. `add(newBook)`호출하면, JPA는 **Set에 이미 Book이 존재하는지 확인**해야한다.
3. JPA는 이 시점에 데이터베이스 쿼리를 실행하여 Author의 모든 Book들을 로드한다. **(해당 과정이 "지연 로딩된 컬렉션을 초기화"하는 과정)**

즉, "초기화한다"는 구문에서의 의미는 **지연 로딩된 컬렉션의 실제 데이터를 데이터베이스에서 불러오는 과정**

4. 데이터가 로드되면, JPA는 newBook과 기존 Book들을 비교하며 **중복 여부**를 살핀다.
5. 중복이 없으면 **Set에 추가**된다.

- 만약 `즉시 로딩`이라면 Author 엔티티를 로드할 때 이미 모든 Book데이터가 함께 로드되었기 때문에 add 메서드 호출시 **추가 쿼리가 날라가지 않는다.**
