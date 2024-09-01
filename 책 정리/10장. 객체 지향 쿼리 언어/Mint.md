# 10장. 객체 지향 쿼리 언어
- `JPQL`은 가장 중요한 객체지향 쿼리 언어다.
    - `Criteria`나 `QueryDSL`은 결국 `JPQL`을 편리하게 사용하도록 도와주는 기술이다.

# 10.1 객체지향 쿼리 소개

### 엔티티 조회
- 식별자로 조회(`EntityManager.find()`)
- 객체 그래프 탐색(`a.getB()`)
> db에서 데이터를 검색해 가져오고싶다면?

- ORM : 데이터베이스 테이블이 아닌 엔티티 객체를 대상으로 개발
### JPQL
- `테이블`이 아닌 `객체`를 대상으로 검색하는 `객체지향 쿼리`
- `SQL`을 추상화해서 **특정 db sql에 의존하지 않는다.**
    - `JPA`는 `JPQL`을 분석하여 적절한 `SQL`을 만들어 `db`를 조회하고, `엔티티` 객체를 생성하여 반환한다.

> `SQL` = 데이터베이스 테이블을 대상으로 하는 데이터 중심의 쿼리
> `JPQL` = 엔티티 객체를 대상으로 하는 객체지향 쿼리

### JPA가 공식적으로 지원하는 기능
- JPQL(Java Persistence Query Lanaguage)
- Criteria 쿼리 : JPQL을 편하게 작성하도록 도와주는 API 빌더 클래스
- 네이티브 SQL : JPA에서 JPQL 대신 직접 SQL 사용

### JPQ가 공식 지원하는 기능은 아니지만 알아둘 가치가 있는 기능
- QueryDSL : Criteria 쿼리처럼 JPQL을 편하게 작성하도록 도와주는 빌더 클래스 모음, 비표준 오픈소스 프레임워크
- JDBC 직접 사용, MyBatis 같은 SQL 매퍼 프레임워크 사용

## JPQL 소개
- JPQL : 엔티티 객체를 조회하는 객체지향 쿼리
- SQL을 추상화하여 특정 db에 의존하지 않는다.
    - db 방언만 변경하면 JPQL을 수정하지 않아도 db를 변경할 수 있다.
- SQL 보다 간결하다.
    - 엔티티 직접 조회, 묵시적 조인, 다형성 지원

### 예제 코드
```java
String jpql = "select m from Member as m where m.username = 'kim'";
List<Member> resultList = em.createQuery(jpql, Member.class).getResultList();
```

## Criteria 쿼리 소개
- `Criteria` : JPQL을 생성하는 빌더 클래스
- 문자가 아닌 프로그래밍 코드로 JPQL을 작성할 수 있다.
    - `JPQL` : 문자로 작성하므로 런타임 시점에 오류가 발생한다.
    - `Criteria` : 코드이므로 컴파일 시점에 오류가 발생한다.

### Criteria 장점
- `컴파일 시점`에 오류를 발견할 수 있다.
- `코드 자동완성`을 지원한다.
- `동적 쿼리`를 작성하기 편하다.

```java
CriteriaBuilder cb = em.getCriteriaBuilder();
CriteriaQuery<Member> query = cb.createQuery(Member.class);

// 루트 클래스(조회를 시작할 클래스)
Root<Member> m = query.from(Member.class);

CriteriaQuery<Member> cq = query.select(m).where(cb.equal(m.get("username"), "kim"));
List<Member> resultList = em.createQuery(cq).getResultList();
```

#### 메타 모델
- 필드를 문자가 아닌 코드로 작성하고 싶다면?
- 어노테이션 프로세서 기능을 사용하여 어노테이션을 분석하여 클래스를 생성할 수 있다.
    - Member 엔티티로부터 Member_ 라는 Criteria 전용 클래스를 생성할 수 있다.
```java
m.get(Member_.username)
```

### Criteria 단점
- 모든 장점을 상쇄할 정도로 복잡하고 장황하다.
    - 코드가 한눈에 들어오지 않는다.

## QueryDSL 소개
- `JPQL 빌더` 역할을 한다.
- `코드 기반`이면서 단순하고 사용하기 쉽다.
- 작성한 코드도 `JPQL`과 비슷해서 한눈에 들어온다.
>김영한님은 Criteria보다 QueryDSL을 선호한다고 하신다.

### QClass
- 어노테이션 프로세서를 사용하여 만든 쿼리 전용 클래스

### 예시 코드
```java
JPQQuery query = new JPAQuery(em);
QMember member = QMember.member;

List<Member> members = 
	query.from(member)
	.where(member.username.eq("kim"))
	.list(member);
```

## Native SQL소개
- JPA에서 지원하는 `SQL을 직접 사용하는 기능`
- `특정 db에 의존하는 기능`을 사용해야할때 사용한다.
    - `CONNECT BY`, `SQL 힌트`
- `SQL`은 지원하지만 `JPQL`은 지원하지 않을 때 사용한다.

### 단점
- **특정 db에 의존하는 sql을 작성해야 한다.**
    - db를 변경하면 네이티브 sql도 수정해야한다.

### 예시 코드
```java
String sql = "SELECT ID, AGE, TEAM_ID, NAME FROM MEMBER WHERE NAME = 'kim'";
List<Member> resultList = 
	em.createNativeQuery(sql, Member.class).getResultList();
```
> createNativeQuery() 를 사용한다.

## JDBC 직접 사용, MyBatis 같은 SQL Mapper 프레임워크 사용
- JDBC 커넥션에 직접 접근하고 싶으면 JPA 구현체가 제공하는 방법을 사용해야한다.
- hibernate 예시
```java
Session session = entityManager.unwrap(Session.class);
session.doWork(){

	@Override
	public void execute(Connection connection) throws SQLException {
	
	}				
}
```

### JDBC나 MyBatis를 JPA와 함께 사용하면 영속성 컨텍스트를 적절한 시점에 강제로 `flush` 해야한다.
- `JDBC`나 `MyBatis` 같은 `SQL Mapper`는 `JPA`를 우회해서 데이터베이스에 접근한다.
    - `JPA`를 우회하는 `SQL`은 **JPA가 전혀 인식하지 못한다.**
    - `영속성 컨텍스트`가 반영되지 않은 상태에서 **db에 직접 조회**하여 `데이터 무결성`을 훼손할 수 있다.
- `JPA`를 우회해서 `SQL`을 실행하기 직전에 **영속성 컨텍스트를 수동으로 flush**해서 db와 영속성 컨텍스트를 `동기화`한다.
    - 스프링 프레임워크를 사용하면 `JPA`와 `MyBatis`를 적절히 통합할 수 있다.
    - 스프링 프레임워크의 `AOP`를 사용하여 JPA를 우회하여 **db에 접근하는 메서드를 호출할때마다 영속성 컨텍스트를 flush할 수 있다**.

# 10.2 JPQL
### JPQL 특징
- `객체지향 쿼리 언어`
    - 테이블을 대상으로 하지 않고 엔티티를 대상으로 쿼리한다.
- `SQL`을 추상화하여 **특정 db의 SQL에 의존하지 않는다.**
- `JPQL`은 결국 `SQL`로 변환된다.

### 기본 문법과 쿼리 API
- `SELECT`, `UPDATE`, `DELETE` 문을 사용할 수 있다.
- 엔티티 저장시 엔티티 매니저의 `persist()`를 사용하므로 `INSERT`는 없다.
```sql
select_문 :: = 
	select_절
	from_절
	[where_절]
	[having_절]
	[orderby_절]

update_문 :: update_절 [where_절]
delete_문 :: delete_절 [where_절]
```
> 전체 구조는 sql과 비슷하다.

### SELECT문
```
SELECT m from MEMBER AS m where m.username = 'Hello';
```
- 대소문자 구분
- 엔티티 이름으로 조회
- 별칭은 필수
    - AS는 생략할 수 있다.

### TypeQuery, Query
- 작성한 JPQL을 실행하려면 쿼리 객체를 만들어야 한다.
- `TypeQuery`: 반환할 타입을 명확하게 지정할 수 있을 때
- `Query` : 반환 타입을 명확하게 지정할 수 없을 때

#### TypeQuery
```java
TypedQuery<Member> query =
	em.createQuery("SELECT m FROM Member m", Member.class);

List<Member> resultList = query.getResultList();
for (Member member: resultList){
	System.out.println("member = " + member);
}
```
> createQuery()에 반환할 타입을 지정하면 TypeQuery를 반환하고, 지정하지 않으면 Query를 반환한다.

#### Query
```java
Query query =
	em.createQuery("SELECT m.username, m.age from Member m");
List resultList = query.getResultList();

for (Object o : resultList){
	Object[] result = (Object[]) o; 
	System.out.println("username = " + result[0]);
	System.out.println("age = " + result[1]);
}
```

- SELECT 절에서 여러 엔티티나 컬럼을 선택할 때는 반환 타입이 명확하지 않으므로 `Query` 객체를 사용해야한다.
    - 조회 대상이 둘 이상이면 `Object[]`를 반환하고, 조회 대상이 하나면 `Object`를 반환한다.

### 결과 조회
- `query.getResultList()` : 결과를 예제로 반환한다. 결과가 없으면 빈 컬렉션을 반환한다.
- `query.getSingleResult()` : 결과가 정확히 **하나**일때 사용한다.
    - 결과가 없으면 `NoResultException`이 발생한다.
    - 결과가 1개보다 많으면 `NonUniqueResultException`이 발생한다.

### 파라미터 바인딩
- JDBC는 위치 기준 파라미터 바인딩을 지원한다.
- JPQL은 위치 기준 파라미터 바인딩과 이름 기준 파라미터 바인딩을 지원한다.

#### 이름 기준 파라미터
- 파라미터를 이름으로 구분
```java
String usernameParam = "User1";

List<Member> query =
	em.createQuery("SELECT m from Member m where m.username = :username", Member.class);
      .setParameter("username", usernameParam)
      .getResultList();
```

#### 위치 기준 파라미터
- 파라미터를 위치로 구분
```java
List<Member> members = 
	em.createQuery("SELECT m from Member m where m.username = ?1", Member.class)
		.setParameber(1, usernameParam)
		.getResultList();
```

> 위치 기준 파라미터 방식보다는 이름 기준 파라미터 바인딩 방식을 사용하는 것이 더 명확하다.

#### 파라미터 바인딩 - SQL 인젝션
- 파라미터 바인딩을 사용하지 않고 직접 문자를 더해 만들어 넣으면 SQL 인젝션 공격을 당할 수 있다.
- 성능 향상
    - 파라미터의 값이 달라도 같은 쿼리로 인식해서 JPQL을 SQL로 파싱한 결과를 재사용할 수 있다.
    - db도 sql을 파싱해서 사용하는데 같은 쿼리는 파싱한 결과를 재사용할 수 있다.
> 파라미터 바인딩 방식은 선택이 아닌 필수다.

## 프로젝션
- SELECT 절에 조회할 대상을 지정하는 것
- 엔티티, 임베디드 타입, 스칼라 타입이 있다.

### 엔티티 프로젝션
```java
SELECT m From Member m
SELECT m.team From Member m
```
- 이렇게 조회한 엔티티는 영속성 컨텍스트에서 관리된다.

### 임베디드 타입 프로젝션
- 조회의 시작점이 될 수 없다.
    - `엔티티`를 통해서 `임베디드 타입`을 조회할 수 있다.
- 엔티티가 아닌 값 타입이므로 영속성 컨텍스트에서 관리되지 않는다.
```java
String query = "SELECT o.address FROM Order o";
List<Address> addresses = em.createQuery(query, Address.class)
							.getResultList();
```

### 스칼라 타입 프로젝션
- 기본 데이터 타입
    - 숫자, 문자, 날짜
- 중복 데이터를 제거하려면 `DISTINCT` 를 사용한다.
```java
List<String> usernames = 
	em.createQuery("SELECT DISTINCT username FROM Member m", String.class)
		.getResultList();
```

### 여러 값 조회
- **프로젝션에 여러 값을 선택**하면 `TypeQuery`를 사용할 수 없고, 대신에 `Query`를 사용해야한다.
```java
List<Object[]> resultList = em.createQuery("SELECT m.username, m.age FROM Member m").getResultList();

for (Object[] row : resultList) {
	String username = (String) row[0];
	Integer age = (Integer) row[1];
}
```

- 엔티티 값도 여러 값을 조회할 수 있다.
```java
List<Object[] resultList = 
	em.createQuery("SELECT o.member, o.product, o.orderAmount FROM Order o")
		.getResultList();

for (Object[] row : resultList){
	Member member = (Member) row[0];
	Product product = (Product) row[1];
	int orderAmount = (Integer) row[2];
}
```

### NEW 명령어
- `DTO`와 같은 의미있는 객체로 변환해서 사용할 수 있다.
```java
TypedQuery<UserDTO> query =
	em.createQuery("SELECT new jpabook.jpql.UserDTO(m.username, m.age FROM Member m", UserDTO.class);

List<UserDTO> resultList = query.getResultList();
```
- 클래스의 생성자에 jpql 조회 결과를 넘겨줄 수 있다.
- 패키지 명을 포함한 전체 클래스명을 입력해야한다.
- 순서와 타입이 일치하는 생성자가 필요하다.

### 페이징 API
- `setFirstResult` : 조회 시작 위치
- `setMaxResults` : 조회할 데이터 수
```java
TypedQuery<Member> query =
	em.createQuery("SELECT m FROM Member m ORDER BY m.username DESC", Member.class);
query.getFirstResult(10);
query.setMaxResults(20);
query.getResultList();
```

> 페이징 SQL을 더 최적화하고 싶다면 JPA가 제공하는 페이징 API가 아닌 네이티브 SQL을 직접 사용해야한다.

### 집합과 정렬
- 집합함수와 함께 통계정보를 구할 때 사용한다.
```sql
select
	COUNT(m),
	SUM(m.age),
	AVG(m.age),
	MAX(m.age),
	MIN(m.age)
from Member m
```

#### 집합 함수 참고사항
- NULL 값은 무시하므로 통계에 잡히지 않는다.
- 만약 값이 없는데 SUM, AVG, MAX, MIN을 사용하면 NULL 값이 된다. COUNT는 0이 된다.
- `DISTINCT`를 집합 함수 안에 사용해서 중복된 값을 제거하고 나서 집합을 구할 수 있다.
- `DISTINCT`를 COUNT에서 사용할 때 임베디드 타입은 지원하지 않는다.

### GROUP BY, HAVING
- GROUP BY : 통계 데이터를 구할 때 특정 그룹끼리 묶어준다.
- HAVING : GROUP BY로 그룹화한 통계 데이터를 기준으로 필터링한다.
```sql
select t.name. COUNT(m.age), AVG(m.age), MAX(m.age), MIN(m.age)
from Member m LEFT JOIN m.team t
GROUP BY t.name
HAVING AVG(m.age) >= 10
```

> 통계 쿼리는 보통 전체 데이터를 기준으로 처리하므로 실시간으로 사용하기에는 부담이 많다.
> 통계 결과만 저장하는 테이블을 별도로 만들어두고 통계 쿼리를 실행해서 그 결과를 보관하는 것이 좋다.

### 정렬
- `ASC` : 오름차순
- `DESC` : 내림차순
```sql
select t.name, COUNT(m.age) as cnt
from Member m LEFT JOIN m.team t
GROUP BY t.name
ORDER BY cnt
```

## JPQL 조인
### 내부 조인
- (INNER) JOIN
- JPQL 조인은 연관 필드를 사용한다.
```sql
SELECT m.usrename, t.name
FROM Member m JOIN m.team t
WHERE t.name = '팀A'
ORDER BY m.age DESC
```

### 외부 조인
- LEFT (OUTER) JOIN
```sql
SELECT
	M.ID AS ID,
	M.AGE AS AGE
	M.TEAM_ID AS TEAM_ID,
	M.NAME AS NAME
FROM
	MEMBER M LEFT OUTER JOIN TEAM T ON M.TEAM_ID=T.ID
WHERE
	T.NAME=?
```

### 컬렉션 조인
- 일대다 관계나 다대다 관계처럼 컬렉션을 사용하는 곳에 조인하는 것
- 다대일 조인 : 단일 값 연관 필드
- 일대다 조인 : 컬렉션 값 연관 필드

### 세타 조인
- 내부 조인만 지원한다.
- 전혀 관계없는 엔티티도 조인할 수 있다.
```sql
// jpql
select count(m) from Member m, Team t
where m.usrename = t.name

// sql
SELECT COUNT(M.ID)
FROM MEMBER M CROSS JOIN TEAM T
WHERE M.USERNAME=T.NAME
```

### JOIN ON
- `ON` : 조인 대상 필터링하고 조인할 수 있다.
- 내부 조인의 `ON`은 `WHERE` 절을 사용할 때와 결과가 같으므로 `외부 조인`에서만 사용한다.
```sql
// JPQL
select m,t from Member m
left join m.team t on t.name = 'A'

// SQL
select m.*, t.* FROM Member m
LEFT JOIN Team tm ON m.TEAM_ID=t.id and t.name = 'A';
```

## fetch join
- JPQL에서 `성능 최적화`를 위해 제공하는 기능
    - **연관된 엔티티와 컬렉션을 한번에 같이 조회**하는 기능
    - `join fetch` 명령어를 사용한다.
- 별칭을 사용할 수 없다.
    - hibernate는 별칭을 허용한다.

### 엔티티 fetch join
```sql
select m from Member m join fetch m.team
```
> 회원과 팀을 함께 조회한다.

- 연관된 엔티티가 프록시가 아닌 실제 엔티티이므로 `지연 로딩`이 이뤄지지 않는다.
- 엔티티가 준영속 상태가 되어도 연관된 엔티티를 조회할 수 있다.
    - 이미 데이터가 로드되어있기 때문이다.

### 컬렉션 fetch join
- 일대다 관계 컬렉션 fetch join하기
```sql
select t 
from Team t join fetch t.members
where t.name = '팀A';
```

> 일대다 조인은 결과가 증가할 수 있다.
> 다대일 조인은 결과가 증가하지 않는다.


#### distinct로 중복 결과 제거
```sql
select distinct t 
from Team t join fetch t.members
where t.name = '팀A';
```
> db 테이블 컬럼은 team과 연관된 member가 다르므로 연관된 멤버만큼의 컬럼이 조회된다.
> distinct t는 애플리케이션에서 중복 결과를 제거한다.

### fetch join vs 일반 join
```sql
select t 
from Team t join t.members m
where t.name = '팀A';
```
- `일반 join` : select 절에 지정한 엔티티만 조회한다.
  - 연관된 엔티티(member)는 조회하지 않는다.
- `fetch join` : 연관된 엔티티까지 조회한다.

### fetch join의 특징
- 연관된 엔티티를 함께 조회할 수 있어서 `SQL 호출 횟수`를 줄여 **성능이 향상된다.**
- `글로벌 로딩 전략`보다 `fetch join`이 우선한다.
  - **글로벌 로딩 전략은 `지연 로딩`을 사용하고 최적화가 필요하면 `fetch join`을 사용하자.**
- 연관된 엔티티를 `쿼리 시점`에 조회하므로 `지연 로딩`이 발생하지 않는다.
  - `준영속 상태`에서도 이미 메모리에 로드되어있으므로 `객체 그래프`를 탐색할 수 있다

### fetch join의 한계
```java
SELECT o FROM Order o JOIN FETCH o.items i WHERE i.price > 1000 // 잘못됨
```
- **`fetch join` 대상에는 `별칭`을 줄 수 없다.**
  - hibernate와 같은 구현체는 별칭을 사용할 수 있으나, 잘못 사용하여 연관된 데이터 수가 달라져 데이터 무결성이 깨질 수 있다. (특히 2차 캐시에서 주의해야한다.)
  - **일부만 로딩**되어 객체의 상태가 불완전한 상태에서 공유하면 문제가 생긴다.
```java
SELECT o FROM Order o JOIN FETCH o.items JOIN FETCH o.payments // 잘못됨
```
- **둘 이상의 컬렉션**을 `fetch` 할 수 없다.
  - 카테시안 곱이 만들어지며 예외가 발생한다.
  - 데이터 양이 많아져 중복 데이터 처리나 정합성 문제가 생긴다.
```java
String jpql = "SELECT o FROM Order o JOIN FETCH o.items";
List<Order> orders = em.createQuery(jpql, Order.class)
                       .setFirstResult(0)  // 시작 위치 
                       .setMaxResults(10)  // 가져올 개수
                       .getResultList(); // 잘못됨
```
- **컬렉션을 `fetch join`하면 `페이징 api`를 사용할 수 없다.**
  - 연관된 모든 엔티티를 조회하여 가져온다.
    - order가 20개이고, 연관된 item이 각 5개이면 100개의 행을 조회하여 가져온다.
    - `중복된` 주문을 제거하여 20개의 order 객체를 생성한다.
    - `페이징`이 적용되어 처음 10개의 주문이 반환된다.
  - **페이징은 마지막에 적용되므로 데이터베이스에서 모든 데이터를 가져오게 되어 페이징 최적화가 되지 않는다.**
  - 데이터가 많을 경우 `OutOfMemoryError`가 발생한다.
> 컬렉션(일대다)가 아닌 단일 값 연관필드(일대일, 일대다)는 fetch join하여 페이징 api를 사용할 수 있다.
```java
SELECT new com.example.OrderDTO(o.id, o.orderDate, c.name) 
FROM Order o JOIN o.customer c
```
- **`fetch join`은 객체 그래프를 유지할 때 사용하면 효과적이지만, 여러 테이블을 조인하여 `엔티티가 아닌 다른 모양`으로 결과를 내야 한다면 join하여 `dto`로 반환하는 것이 더 효과적이다.**
  - 일부 데이터만 가져오는 것이 메모리 사용을 줄일 수 있다.

## 경로 표현식
- JPQL에서 사용하며 . 을 찍어 객체 그래프를 탐색한다.
- 묵시적 조인이 발생한다.

### 용어 정리
- 상태 필드 : 단순히 값을 저장하기 위한 필드
  - ex) m.age
- 연관 필드 : 연관관계 필드, 임베디드 타입 포함
  - 단일값 연관 필드 : `@ManyToOne`, `@OneToOne`, 대상이 엔티티
    -  ex) m.team
  - 컬렉션 값 연관 필드 : `@OneToMany`, `@ManyToMany` 대상이 컬렉션
    -  ex) m.orders

### 경로 표현식과 특징
- `상태 필드 경로` : 경로 탐색의 끝, **더는 탐색할 수 없다**.
- `단일 값 연관 경로` : 묵시적으로 `내부 조인`이 일어난다. **계속 탐색할 수 있다.**
- `컬렉션 값 연관 경로` : 묵시적으로 `내부 조인`이 일어난다. **더는 탐색할 수 없다.**
  - **`명시적 조인(join)`으로 `별칭`을 얻으면 별칭으로 탐색할 수 있다.**
```sql
select m.username from Team t join t.members m
```

#### 명시적 조인 vs 묵시적 조인
- `명시적 조인` : `JOIN`을 직접 적어주는 것
```sql
SELECT m FROM Member m JOIN m.team t
```
- `묵시적 조인` : `경로 표현식`에 의해 묵시적으로 조인이 일어나는 것, `내부 조인`만 할 수 있다.
```sql
SELECT m.team FROM Member m
```
> 임베디드 타입 조인시 이미 조인 대상에 임베디드 타입을 포함하는 엔티티의 테이블이 존재한다면 조인이 발생하지 않는다.

> `묵시적 조인`은 한눈에 파악하기 어려우므로 **성능이 중요할 경우 `명시적 조인`을 사용하자.**

## 서브 쿼리
- WHERE, HAVING 절에서만 사용할 수 있다.

### 서브 쿼리 함수
- EXISTS
  - 결과 존재하면 참
- ALL | ANY | SOME
  - ALL : 조건을 모두 만족하면 참
  - ANY 혹은 SOME : 조건을 하나라도 만족하면 참
- IN
  - 서브쿼리의 결과 중 하나라도 같은 것이 있으면 참

## 조건식
### 타입 표현
- 문자 : `''`
- 숫자 : `L`,`D`,`F`
- 날짜 : `DATE`, `TIME`, `DATETIME`
- Boolean : `TRUE`, `FALSE`
- Enum
- 엔티티 타입

### 연산자 우선 순위
- 1. 경로 탐색 연산
- 2. 수학 연산
- 3. 비교 연산
- 4. 논리 연산

### 논리 연산과 비교식
- 논리 연산
  - AND, OR, NOT
- 비교식
  - = | < | > | <= | >= | <>

### Between, IN, LIKE, NULL
- `Between` A AND B
  - A~B 값 사이
- `IN` : X와 같은 값이 하나라도 있으면 참
- `Like` : 문자 표현식과 패턴 값 비교
  - `%` : 아무 값들이 입력되어도 된다.
  - `_` : 아무 값들이 입력되어도 되지만 값은 있어야한다.
- `IS NULL` : NULL인지 비교한다.

### 컬렉션 식
- `IS EMPTY` : 컬렉션에 값이 비었으면 참
- `MEMBER` : 엔티티나 값에 컬렉션이 포함되어 있으면 참

### 스칼라 식
- 수학식
- 문자함수
  - `CONCAT`, `SUBSTRING`, `LOWER`, ...
- 수학함수
  - `ABS`, `SQRT`, `MOD`
- 날짜함수
  - `CURRENT_DATE`
  - `CURRENT_TIME`
  - `CURRENT_TIMESTAMP`

### CASE 식
- `기본 CASE`
  - 조건식 사용 가능
- `심플 CASE`
  - 조건식 사용 불가, 문법 단순
- `COALESCE`
  - 스칼라식(첫번째 인자)을 조회하여 null이 아니면 두번째 인자 반환
- `NULLIF`
  - 두 값이 같으면 null 반환, 다르면 첫번째값 반환

## 다형성 쿼리
- 부모 엔티티를 조회하면 그 자식 엔티티도 함께 조회한다.

### TYPE
- 엔티티 상속 구조에서 조회 대상을 특정 자식 타입으로 한정할 때 사용한다.
```sql
select i from Item i
where type(i) IN (Book, Movie)
```
> Item에서 해당 Dtype만 조회한다.

### TREAT
- 부모 타입을 특정 자식 타입으로 다룰 때 사용한다.
```sql
select i from Item i where treat(i as Book).author = 'kim'
```

### 사용자 정의 함수 호출
- 방언 클래스를 상속해서 구현하고 사용할 데이터베이스 함수를 미리 등록한다.

### 기타 정리
- enum은 = 비교 연산만 지원한다.
- 임베디드 타입은 비교를 지원하지 않느다.

#### Empty String
- JPA는 '' 길이 0인 Empty String으로 정했지만 db마다 null로 사용하는 곳도 있다.

#### null 정의
- 조건을 만족하는 데이터가 하나도 없으면 null이다.
- null == null 은 알 수 없는 값이다.
- AND
  - True < Null < False
- OR
  - False < Null < True

## 엔티티 직접 사용
### 기본키 값
- `객체 인스턴스`는 `참조 값`으로 식별하고, 테이블 `row`는 `기본키` 값으로 식별한다.
  - `jpql`에서 `엔티티` 객체를 직접 사용하면, `sql`에서는 `해당 엔티티의 기본 키` 값을 사용한다.
- 식별자를 직접 사용해도 결과는 같다.
```java
// 두 쿼리는 같은 sql을 실행한다.
select m from Member m where m = :member;
select m from Member m where m.id = :memberId;
```

### 외래키 값
```java
// 두 쿼리는 같은 sql을 실행한다.
select m from Member m where m.team = :team
select m from Member m.team.id = :teamId
```
- 생성되는 sql은 같다.

## NamedQuery : 정적 쿼리
### JPQL 쿼리
- `동적 쿼리` : JPQL을 `문자`로 완성해서 직접 넘긴다. `런타임`에 특정 조건에 따라 **jpql을 동적으로 구성**할 수 있다.
- `정적 쿼리` : 미리 정의한 쿼리에 `이름`을 부여해서 필요할 때 사용할 수 있다. `NamedQuery`라 한다.
  - 한번 정의하면 변경할 수 없는 정적인 쿼리이다.
  - 파싱된 결과를 재사용하므로 성능상 이점이 있다.

### NamedQuery 어노테이션에 정의
```java
@Entity
@NamedQuery (
	name = "Member.findByUsername",
	query = "select m from Member m where m.username = :username")
public class Member {
}
```

- 직접 사용하기
```java
List<Member> resultList = em.createNamedQuery("Member.findByUsername",
	Member.class)
	.setParameter("username", "회원1")
	.getResultList();
```

> XML은 멀티 라인을 지원하므로 NamedQuery는 XML에 정의하는 것이 더 편리하다.
> XML과 어노테이션에 같은 설정이 있으면 XML이 우선권을 가진다.


# 10.3 Criteria

### Criteria
- JPQL을 자바 코드로 작성하도록 도와주는 빌더 클래스 api
- 문자가 아닌 코드로 jpql을 작성하므로 문법 오류를 `컴파일 단계`에서 잡을 수 있고, `동적 쿼리`를 안전하게 생성할 수 있다.
- 코드가 복잡하고 장황해서 이해하기 힘들다.

## Criteria 기초
```java
CriteriaBuilder cb = em.getCriteriaBuilder();

CriteriaQuery<Member> cq = cb.createQuery(Member.class);

Root<Member> m = cq.from(Member.class);
cq.select(m);

TypedQuery<Member> query = em.createQuery(cq);
List<Member> members = query.getResultList();
```
- 1. 먼저 `Criteria 빌더`를 얻어야한다.
- 2. `Criteria 쿼리`를 생성한다.
- 3. `FROM` 절을 생성한다.
- 4. `SELECT` 절을 생성한다.

#### 쿼리 루트
- 조회의 시작점이다.
- JPQL의 별칭이라 생각하면 된다.
- 엔티티에만 부여할 수 있다.


## Criteria 쿼리 생성
```java
public interface CriteriaBuilder {
	CriteriaQuery<Object> createQuery();
	<T> CriteriaQuery<T> createQuery(Class<T> resultClass);
	CriteriaQuery<Tuple> createTupleQuery();
}
```
- 반환 타입으로 지정하면 createQuery에서 반환 타입을 지정하지 않아도 된다.

```java
CriteriaBuilder cb = em.getCriteriaBuilder();
CriteriaQuery<Member> cq = cb.createQuery(Member.class);

List<Member> resultList = em.createQuery(cq).getResultList();
```
- 반환 타입을 지정할 수 없거나 반환 타입이 둘 이상이면 `Object` 또는 `Object[]`를 지정하면 된다.
```java
// Object
CriteriaBuilder cb = em.getCriteriaBuilder();
CriteriaQuery<Object> cq = cb.createQuery();
List<Object> resultList = em.createQuery(cq).getResultList();

// Object[]
CriteriaBuilder cb = em.getCriteriaBuilder();
CriteriaQuery<Object[]> cq = cb.createQuery(Object[].class);
List<Object[]> resultList = em.createQuery(cq).getResultList();
```
- 튜플을 이용해서 반환 타입을 받아도 된다.
```java
CriteriaQuery<Tuple> cq = cb.createTupleQuery();
```

### 조회
- `select()` : 한건 지정
- `multiselect()` : 여러건 지정
- `cb.array()` : 여러 건 지정

### distinct : 중복 제거
- `distinct(true)`
```java
CriteriaQuery<Object[]> cq = cb.createQuery(Object[].class);
Root<Member> m = cq.from(Member.class);
cq.multiselect(m.get("username", m.get("age")).distinct(true);

List<Object[]> resultList = em.createQuery(cq).getResultList();
```

### new, construct()
```java
CriteriaQuery<MemberDTO> cq = cb.createQuery(MemberDTO.class);
Root<Member> m = cq.from(Member.class);
cq.select(cb.construct(MemberDTO.class, m.get("username"), m.get("age")));

List<MemberDTO> resultList = em.createQuery(cq).getResultList();
```

### 튜플
- 별칭을 꼭 사용해야한다. - alias()
  - 선언해둔 튜플 별칭으로 데이터를 조회할 수 있다.
- `이름 기반`이므로 `Object[]`보다 안전하다.
- `getElements()`로 현재 튜플의 별칭과 자바 타입도 조회할 수 있다.
- 엔티티도 조회할 수 있다.
```java
CriteriaQuery<Tuple> cq = cb.createTupleQuery();
Root<Member> m = cq.from(Member.class);
cq.multiselect(
	m.get("username").alias("username"),
	m.get("age").alias("age")
);

List<Tuple> resultList = em.createQuery(cq).getResultList();
for (Tuple tuple:resultList){
	String username = tuple.get("username", String.class);
	Integer age = tuple.get("age", Integer.class);
}
```

## 집합
- GROUP BY : 그룹화
- HAVING : 그룹화 결과 필터링
```java
cq.multiselect(m.get("team").get("name"), maxAge, minAge)
	.groupBy(m.get("team").get("name"))
	.having(cb.gt(minAge, 10));
```

## 정렬
- cb.desc() 또는 cb.asc()로 생성할 수 있다.
```java
cq.select(m)
	.where(ageGt)
	.orderBy(cb.desc(m.get("age")));
```

## 조인
- join()과 JoinType 클래스를 사용한다.
```java
public enum JoinType {
	INNER,
	LEFT,
	RIGHT
}
```

```java
Root<Member> m = cq.from(Member.class);
Join<Member, Team> t = m.join("team", JoinType.INNER); // 내부 조인
cq.multiselect(m,t)
	.where(cb.equal(t.get("name"), "팀A"));
);
```

### fetch join
```java
Root<Member> m = cq.from(Member.class);
m.fetch("team", JoinType.LEFT);

cq.select(m);
```

## 서브 쿼리
### 간단 서브 쿼리
```java
CriteriaBuilder cb = em.getCriteriaBuilder();
CriteriaQuery<Member> mainQuery = cb.createQuery(Member.class);

// 서브 쿼리 생성
SubQuery<Double> subQuery = mainQuery.subquery(Double.class);
Root<Member> m2 = subQuery.from(Member.class);
subQuery.select(cb.avg(`m2.<Integer>get("age")`));

// 메인 쿼리 생성
Root<Member> m = mainQuery.from(Member.class);
mainQuery.select(m)
	.where(cb.ge(m.<Integer>get("age"), subQuery));
```

### 상호 관련 서브 쿼리
- 메인 쿼리와 서브 쿼리간에 서로 관련이 있을 때
- 서브 쿼리에서 메인 쿼리의 정보를 사용하려면 메인 쿼리에서 사용한 `별칭`을 얻어야한다.
  - 메인 쿼리의 `Root`나 `Join`을 통해 생성된 별칭을 받는다.
```java
CriteriaBuilder cb = em.getCriteriaBuilder();
CriteriaQuery<Member> mainQuery = cb.createQuery(Member.class);
Root<Member> m = mainQuery.from(Member.class);

// 서브 쿼리 생성
SubQuery<Team> subQuery = mainQuery.subquery(Team.class);
Root<Member> subM = suqQuery.correlate(m); // 메인 쿼리의 별칭 가져옴
Join<Member, Team> t = subM.join("team");
subQuery.select(t)
	.where(cb.equal(t.get("name"), "팀A"));

// 메인 쿼리 생성
mainQuery.select(m)
	.where(cb.exists(subQuery));
List<Member> resultList = em.createQuery(mainQuery).getResultList();
```

## IN 식
```java
CriteriaBuilder cb = em.getCriteriaBuilder();
CriteriaQuery<Member> mainQuery = cb.createQuery(Member.class);
Root<Member> m = mainQuery.from(Member.class);

cq.select(m)
	.where(cb.in(m.get("username"))
	.value("회원1")
	.value("회원2"));
```

## CASE 식
- `selectCase()`, `when()`, `otherwise()` 사용한다.
```java
Root<Member> m = cq.from(Member.class);
cq.multiselect(
	m.get("username"),
	cb.selectCase()
		.when(cb.ge(m.<Integer>get("age"), 60), 600)
		.when(cb.le(m.<Integer>get("age"), 15), 500)
		.otherwise(1000)
);
```

## 파라미터 정의
- cb.parameter(타입, 파라미터 이름) : 파라미터 정의
- setParameter("usernameParam", "회원1") : 파라미터에 사용할 값 바인딩

## 네이티브 함수 호출
- `cb.function(...)` 메서드 사용

## 동적 쿼리
- 다양한 검색 조건에 따라 실행 시점에 쿼리 생성하는 것
- 코드 기반인 Criteria로 작성하는 것이 편리하다.
- 공백이나 where, and 위치로 인해 에러는 발생하지 않지만, 코드가 장황하고 복잡하여 읽기 힘들다.

## 함수 정리
- Criteria는 jpql 함수를 코드로 지원한다.
- 조건 함수
- 스칼라와 기타 함수
- 집합 함수
- 분기 함수

## Criteria 메타 모델 API
- 필드까지 코드로 작성하려면 메타 모델 API를 이용하면 된다.
  - 메타 모델 클래스 : 엔티티명_
- 코드 자동 생성기가 엔티티 클래스를 기반으로 메타 모델 클래스들을 만들어준다.

# 10.4 QueryDSL
- 문자가 아닌 코드로 JPQL을 작성하므로 문법 오류를 컴파일 단계에서 잡을 수 있다.
- 쉽고 간결하며 쿼리와 모양이 비슷하다.
- 오픈소스 프로젝트이다.

## QueryDsl 설정
- `querydsl-jpa` : QueryDsl JPA 라이브러리
- `querydsl-apt` : 쿼리 타입 생성시 필요한 라이브러리

## 시작
```java
public void queryDSL(){
	EntityManager em = emf.createEntityManager();
	
	JPAQuery query = new JPAQuery(em);
	QMember qMember = new QMember("m");
	List<Member> members = 
		query.from(qMember)
			.where(qMember.name.eq("회원1"))
			.orderBy(qMember.name.desc())
			.list(qMember);
}
```
- 엔티티 매니저를 생성자에 넘겨준다.
- 쿼리 타입을 생성하는데 생성자에는 별칭을 준다.

### 기본 Q 생성
- 같은 엔티티를 조인하거나 서브 쿼리에 사용하면 별칭을 사용하므로 직접 별칭을 지정해야한다.

### 검색 조건 쿼리
```java
JPQQuery query = new JPAQuery(em);
QItem item = QItem.item;
List<Item> list = query.from(item)
	.where(item.name.eq("좋은 상품").and(item.price.gt(20000)))
	.list(item);
```
- `where` 절에 `and` 나 `or`을 사용할 수 있다.
  - `between`, `contains`, `startsWith`도 사용할 수 있다.

## 결과 조회
- `uniqueResult()` : 조회 결과가 한 건일때 사용한다.
- `singleResult ()` : 결과가 하나 이상이면 처음 데이터를 반환한다.
- `list()` : 결과가 하나 이상일때 사용한다. 없으면 빈 컬렉션을 반환한다.

## 페이징과 정렬
- 정렬 : `orderBy`
  - `asc()`, `desc()`
- 페이징 : `offset`, `limit`, `restrict()`
```java
QItem item = QItem.item;

query.from(item)
	.where(item.price.gt(20000))
	.orderBy(item.price.desc(), item.stockQuantity.asc())
	.offset(10).limit(20)
	.list(item);
```

```java
QueryModifiers queryModifiers = new QueryModifiers(20L, 10L);
List<Item> list = query.from(item)
	.restrict(queryModifiers)
	.list(item);
```

- 실제 페이징 처리를 하려면 검색된 전체 데이터 수를 알아야한다.
  - listResults()를 사용한다.
```java
SearchResults<Item> result = 
	query.from(item)
		.where(item.price.gt(10000))
		.offset(10).limit(20)
		.listResults(item);
```
- `listResults()`를 사용하면 전체 데이터 조회를 위한 count 쿼리를 한번 더 실행한다.
  - 그리고 SearchResults를 반환하는데, 이 객체에서 전체 데이터 수를 조회할 수 있다.

## 그룹
- groupBy를 사용하고, 그룹화된 결과를 제한하려면 having을 사용한다.
```java
 query.from(item)
	 .groupBy(item.price)
	 .having(item.price.gt(1000))
	 .list(item);
```

## 조인
- 첫번째 파라미터에 조인 대상을 지정하고, 두번째 파라미터에 별칭으로 사용할 쿼리 타입을 지정한다.
```java
query.from(order)
	.join(order.member, member)
	.leftJoin(order.orderItems, orderItem)
	.list(order);
```

- join에 on을 사용할 수 있다.
```java
query.from(order)
	.leftJoin(order.orderItems, orderItem)
	.on(orderItem.count.gt(2))
	.list(order);
```

- fetch join 을 사용할 수 있다.
```java
query.from(order)
	.innerJoin(order.member, member).fetch()
	.leftJoin(order.orderItems, orderItem).fetch()
	.list(order);
```

- from 절에 여러 조건을 사용하는 세타 조인
```java
query.from(order, member)
	.where(order.member.eq(member))
	.list(order);
```

## 서브 쿼리
- JPASubQuery를 생성해서 사용한다.
- 결과가 하나면 unique(), 여러건이면 list()를 사용한다.
```java
query.from(item)
	.where(item.price.eq(
		new JPASubQuery().from(itemSub).unique(itemSub.price.max())
	))
	.list(item);
```

## 프로젝션과 결과 반환
- select 절에 조회 대상을 지정하는 것

### 프로젝션 대상이 하나
- 해당 타입으로 반환한다.
```java
QItem item = QItem.item;
List<String> result = query.from(item).list(item.name);
```

### 여러 컬럼 타입과 튜플
-  프로젝션 대상으로 여러 필드 선택시 Tuple을 사용한다.
```java
QItem item = QItem.item;

List<Tuple> result = query.from(item).list(item.name, item.price);

for (Tuple tuple:result){
	System.out.println("name = " + tuple.get(item.name));
	System.out.println("price = " + tuple.get(item.price));
}
```

### 빈 생성
- 프로퍼티 접근
- 필드 직접 접근
- 생성자 사용
```java
QItem item = QItem.item;
List<ItemDTO> result = query.from(item).list(
	Projections.bean(ItemDTO.class, item.name.as("username"), item.price));
)
```
> 쿼리 결과와 매핑할 프로퍼티 이름이 다르면 as를 사용해서 별칭을 주면 된다.

