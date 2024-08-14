## 3주차

📌 **346 ~ 366**
**객체지향 쿼리**
`ORM`을 사용하면 데이터베이스 테이블이 아닌 엔티티 객체를 대상으로 개발하기 때문에 검색도 **테이블이 아닌 엔티티 대상**으로 하는 방법이 필요한데 이를 위한 대표적인 `객체지향 SQL`을 소개한다.

`Criteria`나 `QueryDsl`은 `JPQL`을 편하게 작성하도록 도와주는 하나의 빌더 클래스일 뿐이므로 **JPQL을 확실히 이해**해야 나머지도 이해하기 쉬워진다.

- `JPQL`: 엔티티 객체를 조회하는 객체지향 쿼리, SQL을 추상화해서 **특정 데이터베이스에 의존하지 않는다**. 즉, `Oracle`, `MySQL`, `PostgreSQL` 등 **어떤 데이터베이스를 사용하더라도 동일한 JPQL 쿼리를 작성**할 수 있다는 의미다.

- `Criteria` : 프로그래밍 코드로 `JPQL`을 작성할 수 있어 **컴파일 시점에 오류를 발견할 수 있다**는 장점이 있다. 즉, 문자가 아닌 코드로 JPQL을 작성한다는 점이 JPQL과의 차이점이다. 하지만 이러한 **장점들을 상쇄할 정도의 복잡한 코드**는 사용할 마음이 사라지게 만드는 것도 사실이다.

- `QueryDsl` : `Criteria`의 복잡한 코드를 **단순화** 시킨것이 `QueryDsl`이라 이해하면 쉽다.

- `네이티브 SQL` : JPA는 SQL을 직접 사용할 수 있는 기능도 지원한다. 따라서 **SQL은 지원하지만 JPQL은 지원하지 않는 기능을 사용할 때** 네이티브 SQL을 사용한다. 하지만 특정 데이터베이스에 의존하는 SQL을 작성해야한다는 단점이 있다.

📌 **367 ~ 388**
**페치조인?**
페치(fetch) 조인은 JPQL에서 **성능 최적화**를 위해 제공하는 기능이다.
**연관된 엔티티나 컬렉션을 한 번에 같이 조회**하는 기능이다.
명령어는 `join fetch`로 사용할 수 있다.

**페치 조인과 일반 조인의 차이?**
JPQL에서 어떤 특정 엔티티를 조인했을 때 **모든 엔티티가 조회될거라 기대하면 안된다.**
JPQL은 결과를 반환할 때 **연관관계**까지 고려하지 않기 때문이다.
따라서 연관된 엔티티를 같이 조회하고 싶다면 `fetch`를 명시하는 것이 좋다.

이러한 페치 조인은 SQL 한번으로 연관된 엔티티를 함께 조회할 수 있어 성능을 최적화할 수 있다. 특히 페티 조인은 **글로벌 로딩 전략(EAGER,LAZY)**보다 **우선**한다.
따라서 **지연로딩을 사용하고 최적화가 필요하다면 페치 조인을 명시하는 것이 효과적**일 것 같다.

하지만 이러한 페치 조인도 **단점**이 몇가지 존재하는데,
- 페치 조인 대상에는 별칭을 줄 수 없다.
- 둘 이상의 컬렉션을 페치할 수 없다.
- 컬렉션을 페치 조인하면 페이징 API를 사용할 수 없다.

📌 **389~418**
**사용자 정의 함수**
말 그대로 사용자가 미리 함수를 정의해 놓고 이후 사용하는 것이다. 근데 이거 어떻게 사용해?

`Mysql` 기준으로 설명해보겠다.
먼저 내가 알아본 방법이다.

**1. 사용자 정의 함수를 생성하고 **

```sql
DELIMITER //

CREATE FUNCTION calculate_age(birthdate DATE) 
RETURNS INT
DETERMINISTIC
BEGIN
    DECLARE age INT;
    SET age = YEAR(CURDATE()) - YEAR(birthdate);
    IF (MONTH(CURDATE()) < MONTH(birthdate) OR 
        (MONTH(CURDATE()) = MONTH(birthdate) AND DAY(CURDATE()) < DAY(birthdate)))
    THEN
        SET age = age - 1;
    END IF;
    RETURN age;
END //

DELIMITER ;
```
(대충 이런 사용자 함수가 있다고 가정)

**2. xml에 함수를 등록한다.**
```java
<function-node name="calculate_age" function-name="calculate_age"/>
```

**3. 사용자정의 함수 사용한다. **

음... **XML에 등록하지 않고 사용**하는 방법은 없나?

일단 1번은 동일하다. 사용자 정의 함수를 생성한다. 그리고 다음 4가지 방법이 존재한다.

**1. @NamedNativeQuery 사용.**
**2. @Query 어노테이션 사용**
**3. EntityManager를 통한 네이티브 쿼리 실행**
**4. Hibernate Dialect 확장**

```java
public class CustomMySQLDialect extends MySQL8Dialect {
    public CustomMySQLDialect() {
        super();
        registerFunction("calculate_age", new SQLFunctionTemplate(StandardBasicTypes.INTEGER, "calculate_age(?1)"));
    }
}
``` 

📌 **418~434**
아무리 생각해도 `Criteria`이거 마음에 안드는 놈이다.
`JPQL`과 비교하여 **문법 오류를 컴파일 단계에서 잡을 수 있다는 것** 그리고 **동적 쿼리**에 대한 장점... 제외하고는 복잡해서 솔직히 별로 쓰고 싶지도 않고 알고싶지도 않다. 지금은 그냥 있다는 것만 알아두면 좋을 것같다는 것이 내 개인적인 의견이다.

**Querydsl Q 클래스**

- Q클래스는 JPA 엔티티에 대응하는 정적 메타 모델 클래스다. Q 클래스는 프로세서에 의해 자동 생성된다.

**"왜 굳이 엔티티 클래스 대신 Q클래스를 만들어서 사용할까?"**

`Entity`

```java
@Entity
public class User {
    @Id @GeneratedValue
    private Long id;
    private String username;
    private int age;
    private String email;
}
```

`Q클래스`

```java
public class QUser extends EntityPathBase<User> {
    public static final QUser user = new QUser("user");
    public final NumberPath<Long> id = createNumber("id", Long.class);
    public final StringPath username = createString("username");
    public final NumberPath<Integer> age = createNumber("age", Integer.class);
    public final StringPath email = createString("email");
}
```

1. `user.age`나 `user.username`은 타입이 정확히 지정되어 있어 **타입에 맞지 않는 연산이나 비교를 시도하면 컴파일 시점에 오류**를 잡을 수 있다.

2. 또한 Q 클래스를 보면 **엔티티 속성을 정적**으로 표현하므로 IDE에서 자동 완성을 지원하기 때문에 쿼리 작성이 편리하다.

