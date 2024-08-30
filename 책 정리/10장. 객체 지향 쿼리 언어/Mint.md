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

