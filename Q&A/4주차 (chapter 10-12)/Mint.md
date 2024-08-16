### [0724] 434~454

# QueryDsl

## 결과 조회

- uniqueResult() : 조회 결과가 한 건
- singleResult(): 결과가 하나 이상이면 처음 데이터 반환
- list(): 결과가 하나 이상일 때 사용

## 페이징과 정렬

- 정렬: orderBy() - asc(), desc()
- 페이징: offset(), limit() 또는 QueryModifiers에 restrict(), 전체 데이터 수 조회는 listResults()

## 그룹

- groupBy, 그룹화된 결과 제한: having

## 조인

- innerJoin, leftJoin, rightJoin, fullJoin
- join(조인 대상, 별칭으로 사용할 쿼리 타입)

## 서브 쿼리

- JPASubQuery 생성
- 결과가 하나면 unique(), 여러건이면 list()

## 프로젝션과 결과 반환

- 프로젝션: select절에 조회 대상 지정
- 프로젝션 대상이 하나: 해당 타입으로 반환
- 여러 컬럼 반환: Tuple 사용
- 빈 생성: 쿼리 결과를 엔티티가 아닌 특정 객체로 받고 싶을 경우
    - 프로퍼티 접근 Projections.bean(클래스, 프로퍼티..)
    - 필드 접근 Projections.fields(클래스, 필드...)
    - 생성자 사용 Projections.constructor(클래스, 파라미터...)

## Distinct

- 중복 제거 query.distinct().from(~~~)

## 수정, 삭제 배치 쿼리

- jpql 배치 쿼리와 같이 영속성 컨텍스트는 무시하고 데이터베이스를 직접 쿼리한다
- 수정 배치 쿼리: JpaUpdateClause
- 삭제 배치 쿼리: JpaDeleteClause

## 동적 쿼리

- BooleanBuilder를 사용하면 특정 조건에 따른 동적 쿼리 생성 가능

## 메서드 위임 @QueryDelegate(클래스)

- 메서드 위임 기능을 사용하면 쿼리 타입에 검색 조건 직접 정의 가능
- 정적 메서드를 만들기 - (대상 엔티티 쿼리타입, 필요한 파라미터들)
- 어노테이션에 속성으로 기능을 적용할 엔티티 지정
- 자바 기본 내장 타입에도 메서드 위임 기능 사용 가능

## QueryDsl 정리

- 문자가 아닌 코드로 안전하게 쿼리 작성 가능
- 복잡한 동적 쿼리를 쉽고 단순하게 작성 가능

# 네이티브 SQL

- JPQL은 대부분의 문법과 SQL 함수들을 지원하지만 특정 데이터베이스에 종속적인 기능은 지원하지 않는다.
- 아래와 같은 것들을 지원하지 않는다.
    - 특정 데이터베이스만 지원하는 함수, 문법, SQL 쿼리 힌트
    - 인라인 뷰, UNION, INTERSECT
    - 스토어드 프로시저
- JPQL을 사용할 수 없을때 JPA는 SQL을 직접 사용할 수 있는 기능을 지원하는데, 이를 네이티브 SQL 이라고 한다.

> JPQL : JPA가 SQL 생성
> 네이티브 SQL: SQL 직접 생성, 수동 모드
:star2: 네이티브 SQL을 사용하면 엔티티를 조회할 수 있고 JPA가 지원하는 영속성 컨텍스트의 기능을 그대로 사용할 수 있다.
> 그러나 JDBC API를 직접 사용하면 단순히 데이터의 나열을 조회할 뿐이다.

## JPA가 특정 데이터베이스에 종속적인 기능을 지원하는 방법

- 특정 데이터베이스만 사용하는 함수 => 네이티브 SQL 함수
- 특정 데이터베이스만 지원하는 SQL 쿼리 힌트 => hibernate 지원 O , 몇몇 JPA 구현체가 지원
- 인라인 뷰, UNION, INTERSECT => hibernate 지원 X, 몇몇 JPA 구현체가 지원
- 스토어 프로시저 => 호출 가능
- 특정 데이터베이스만 지원하는 문법 => 네이티브 SQL 사용하기

## 네이티브 SQL 사용

- 엔티티 조회
  em.createNativeQuery(sql, 결과 클래스), 위치 기반 파라미터만 지원한다.(hibernate는 이름 기반까지 지원)
- 값 조회
  em.createNativeQuery(sql), 반환 타입은 List<Object[]>
- 결과 매핑 사용
  엔티티와 스칼라를 함께 조회하는 경우
  em.createNativeQuery(sql, 결과 매핑 정보 이름)

## 결과 매핑 어노테이션

- @SqlResultSetMapping
- @EntityResult
- @FieldResult
- @ColumnResult

## 네임드 네이티브 SQL

- Native Sql에 이름 지정
- XML에 정의할 경우 멀티 라인을 지원하므로 완성한 SQL을 바로 집어넣을 수 있어 편리

## Native SQL 정리

- JPQL과 마찬가지로 Query, TypeQuery(Named 네이티브 쿼리인 경우에만)를 반환한다. 따라서 JPQL API를 그대로 사용할 수 있다.
- 네이티브 SQL은 자동 생성하는 SQL을 수동으로 직접 정의하는 것이다. 따라서 JPA가 제공하는 기능 대부분을 그대로 사용할 수 있다.
- 관리하기 쉽지 않고 자주 사용하면 특정 데이터베이스에 종속적인 쿼리가 증가하여 이식성이 떨어진다.
  =>
  될 수 있으면 표준 JPQL을 사용하자.
  기능이 부족하면 hibernate같은 JPA 구현체가 제공하는 기능을 사용하자.
  그래도 안되면 마지막 방법으로 네이티브 SQL을 사용하자.
  부족함을 느낀다면 MyBatis나 JdbcTemplate 같은 SQL 매퍼와 JPQ를 함께 사용하자.

## 스토어드 프로시저

- `em.createStoredProcedureQuery("이름")`
- `registerStoredProcedureParameter(순서, 타입, 파라미터 모드)`

#### 질문

1. ❓ querydsl 페이징에서 listResults()의 결과는 전체 데이터인지, 페이징 결과 데이터인지 궁금하다. (435p)
2. ❓ 세타 조인이란? querydsl에서 from절에 여러 조건을 사용해서 작성하는지? (437p)
3. ❓ 이 쿼리는 어떤 쿼리를 나타내는지? (437p)
4. ❓ 네이티브 sql에서 결과 매핑을 사용할때 @FieldResult를 한번이라도 사용하면 전체 필드를 @FieldResult로 매핑해야한다는 의미? (448p)
5. ❓ 프로시저란? (453p)

### [0726]  455~475

### Named 스토어드 프로시저 사용 `@NamedStoredProcedureQuery`

- 스토어드 프로시저 쿼리에 이름을 부여해서 사용한다.

# 10.6 객체 지향 쿼리 심화

- 엔티티 수정시 영속성 컨텍스트의 변경 감지 기능이나 병합을 사용
- 엔티티 삭제시 EntityManager.remove() 사용

## 벌크 연산

- 여러 건을 한번에 수정하거나 삭제하는 연산
- executeUpdate() 메서드 사용

## 벌크 연산 주의점

- 벌크 연산은 영속성 컨텍스트를 무시하고 데이터베이스에 직접 쿼리한다.
  => 벌크 연산은 영속성 컨텍스트를 통하지 않고 데이터베이스에 직접 쿼리한다.
  그러므로 영속성 컨텍스트와 데이터베이스간에 데이터 차이가 발생할 수 있으므로 주의해야한다.

### 해결 방법

- em.refresh() 사용: db에서 다시 조회
- 벌크 연산 먼저 사용 (추천): 벌크 연산 후 조회하면 벌크 연산으로 이미 변한 엔티티를 조회하게 된다.
- 벌크 연산 수행 후 영속성 컨텍스트 초기화 (필요할 때 사용) : 영속성 컨텍스트의 엔티티를 제거한다.

## 영속성 컨텍스트와 JPQL

- jpql의 조회 대상은 엔티티, 임베디드 타입, 값 타입이 있다. 하지만 엔티티가 아니면 영속성 컨텍스트에서 관리되지 않는다.
- 영속성 컨텍스트에 엔티티가 있는데, jpql로 해당 엔티티를 다시 조회하면 db에서 조회한 결과를 버리고 영속성 컨텍스트에 있던 엔티티를 반환한다. (식별자 값으로 식별)
    - db에서 조회한 엔티티로 영속성 컨텍스트의 엔티티를 교체할 경우 기존에 수정 중인 데이터가 사라질 수 있으므로 위험하다.
- jpql로 조회한 엔티티가 영속성 컨텍스트에 없으면 영속성 컨텍스트에 추가한다.

### em.find() vs JPQL

- 두번 다 조회시 같은 인스턴스를 반환한다.
- 두번째 조회시 em.find()는 영속 컨텍스트에서 엔티티를 조회하고, jpql은 db에서 조회한 엔티티를 버리고 영속 컨텍스트에 있는 엔티티를 반환하기 때문이다.

#### jpql의 특징

- jpql은 항상 데이터베이스를 조회한다.
- jpql로 조회한 엔티티는 영속 상태이다.
- 영속성 컨텍스트에 이미 존재하는 엔티티가 있으면 기존 엔티티를 반환한다.

## JPQL과 플러시 모드

- 플러시 : 영속성 컨텍스트의 변경 내역을 데이터베이스에 동기화한다.
    - 플러시가 일어날 때 영속성 컨텍스트에 등록, 수정, 삭제한 엔티티를 찾아서 INSERT, UPDATE, DELETE SQL을 만들어 db에 반영한다.
- 플러시 모드
    - `FlushModeType.AUTO`: 커밋 또는 쿼리 실행시 플러시 (기본)
    - `FlushModeType.COMMIT` : 커밋 시에만 플러시 (성능 최적화시 사용, 주의해서 사용)
- `FlushModeType.COMMIT `사용시 주의
    - jpql 실행시 영속성 컨텍스트의 데이터의 수정 사항이 db에 반영되어 있지 않으므로(flush X) 갱신 전 결과가 조회된다.
    - 쿼리 전 em.flush()를 하거나 해당 쿼리에만 COMMIT 모드로 설정한다.
- `FlushModeType.COMMIT` 모드로 성능 최적화
    - 쿼리가 코드에 산재할때 쿼리마다 flush()하는 것을 막아 성능 최적화가 가능하다.
    - 하지만 주의해서 사용해야한다.
- JDBC를 사용하는 경우에 JPA가 해당 쿼리를 인식하지 못하므로 쿼리 실행 전 em.flush()를 호출하여 데이터베이스에 동기화해야한다.

## 정리

- JPQL은 SQL을 추상화해서 특정 데이터베이스 기술에 의존하지 않는다.
- Criteria나 QueryDsl은 jpql을 만들어주는 빌더 역할을 하므로 jpql을 잘 알아야한다.
- Criteria나 QueryDsl를 이용하여 동적 쿼리를 편리하게 작성할 수 있다.
- Criteria는 jpa가 공식적으로 지원하지만 사용하기 불편하고 직관적이지 않다. 반면에 QueryDsl은 jpa가 공식적으로 지원하지는 않지만 직관적이고 편리하다.
- jpa도 네이티브 sql을 제공하므로 직접 sql을 사용할 수 있다. 하지만 특정 db에 종속적인 sql 사용시 다른 데이터베이스로 변경하기 어려우므로 최대한 jpql을 사용하자.
- jpql은 대량으로 데이터를 수정하거나 삭제하는 벌크 연산을 지원한다.

### [0726] 476~496

# 11장 웹 애플리케이션 제작

## 프로젝트 환경설정

- 핵심 라이브러리
  스프링 MVC, 스프링 ORM, JPA,hibernate, H2 DB, Connection Pool, Web, Slf4J&LogBack, test
- 메이븐 dependency의 scope 설정
  compile, runtime, provided(외부에서 라이브러리 사용), test
- 의존성 전이
  의존관계가 있는 라이브러리도 함께 내려받아 라이브러리에 자동으로 추가한다.
- 스프링 프레임워크 설정
    - web.xml : 웹 애플리케이션 환경설정 파일
    - webAppConfig.xml: 스프링 웹 관련 환경설정 파일
    - appConfig.xml: 스프링 애플리케이션 관련 환경설정 파일

> 스프링 프레임워크를 설정할때 보통 웹 계층(webAppConfig)과 비즈니스 도메인 계층(appConfig)을 나누어 관리한다.

### webAppConfig

- <mvc:annotaion-driven> : 스프링 mvc 기능 활성화
- <context:component-scan> : basePackages를 포함한 하위 패지키를 검색하여 @Component, @Service, @Controller 어노테이션이 붙어 있는 클래스들을 스프링 빈으로
  자동 등록
- `<bean>` : 스프링 빈 등록

### appConfig

- `<tx:annotation-driven>` : 스프링 프레임워크가 제공하느 어노테이션 기반의 트랜잭션 관리자를 활성화한다. @Transactional이 붙은 곳에 트랜잭션을 적용한다.
- `<bean id="datasource" ~~>` : 데이터베이스에 접근할 데이터소스를 등록
- `<bean id="transactionManager" >` : 트랜잭션 관리자를 등록
- `<bean class="org.~~.PersistenceExceptionTranlationPostProcessor` : @Repository 어노테이션이 붙어 있는 스프링 빈에 예외 변환 AOP를 적용한다.

> AOP는 JPA예외를 스프링 프레임워크가 추상화한 예외로 변환해준다.

- `<bean id="entityManagerFactory" class="~~~.LocalContainerEntityManagerFactoryBean>"` : 엔티티 매니저 팩토리 등록

> 스프링 프레임워크에서 JPA를 사용하려면 스프링 프레임워크가 제공하는 LocalContainerEntityManagerFactoryBean을 스프링 빈으로 등록해야 한다.

- `LocalContainerEntityManagerFactoryBean` : JPA를 스프링 컨테이너에서 사용할 수 있도록 스프링 프레임워크가 제공하는 기능
- `dataSource`: 사용할 데이터소스 등록한다.
- `packagesToScan`: @Entity가 붙은 클래스 자동으로 검색하기 위한 시작점 지정
- `persistenceUnitName` : 영속성 유닛 이름 지정 (기본값은 default)
- `jpaVendorAdapter` : 사용할 JPA Vendor 지정 ex) HibernateJpaVendorAdapter

### [0728] 497~517

## 어노테이션

@Repository

- 스프링 빈 자동 등록
- JPA 전용 예외 발생시 스프링이 추상화한 예외로 변환
  @PersistenceContext
- 스프링이나 J2EE 컨테이너를 사용하면 컨테이너가 엔티티 매니저를 관리하고 제공해준다.
- 컨테이너가 관리하는 엔티티 매니저를 주입한다.
  @PersistenceUnit
- 엔티티 매니저 팩토리를 주입한다.
  @Service
- 스프링 빈으로 등록한다.
  @Transactional
- 트랜잭션을 적용한다. 메서드를 호출할 때 트랜잭션을 시작하고 메서드를 종료할 때 트랜잭션을 커밋한다.
- 예외가 발생하면 트랜잭션을 롤백한다(RuntimeException과 Unchecked Exception만 롤백한다.)
- 테스트에 적용시 테스트를 실행할 때마다 트랜잭션을 시작하고, 테스트가 끝나면 트랜잭션을 강제로 롤백한다.
  @Autowired
- 스프링 컨테이너가 적절한 스프링 빈을 주입한다.

## JPA 메서드

save()

- 저장 : 식별자 값이 없으면(null) 새로운 엔티티로 판단해서 persist()로 영속화한다.

> save() 호출시 식별자 값을 할당하여 persist()를 호출한다.

- 수정(병합) : 식별자 값이 있으면 이미 한번 영속화되었던 엔티티로 판단해서 merge()로 수정(병합)한다. 변경된 데이터를 저장한다.

> merge(수정, 병합)은 준영속 상태의 엔티티를 수정할 때 사용한다.
> 영속화된 엔티티는 변경 감지 기능이 동작해서 트랜잭션을 커밋할 때 자동으로 수정되므로 별도의 수정 메서드를 호출할 필요가 없다.

### [0729] 517-536

### 도메인 모델 패턴

- 비즈니스 로직 대부분이 엔티티에 존재
- 서비스 계층은 단순히 엔티티에 필요한 요청 위임

### 트랜잭션 스크립트 패턴

- 서비스 계층에서 대부분의 비즈니스 로직 처리

## 준영속 엔티티 수정

1. 변경 감지 기능 사용

- 영속성 컨텍스트에서 엔티티 다시 조회 후에 데이터 수정

2. 병합(merge) 사용

- 파라미터로 넘긴 준영속 엔티티의 식별자 값으로 엔티티를 조회한 후 영속 엔티티의 값을 준영속 엔티티의 값으로 채워넣는다.

> 변경 감지 기능을 사용하면 원하는 속성만 선택해서 변경할 수 있지만, 병합을 사용하면 모든 속성이 변경된다.

[0730] 538-558

# 12. 스프링 데이터 JPA

## 12.1 스프링 데이터 JPA 소개

- CRUD를 처리하기 위한 공통 인터페이스 제공
- 리포지토리 개발시 인터페이스만 작성하면 샐행 시점에 스프링 데이터 JPA가 구현 객체를 동적으로 생성해서 주입
- 데이터 접근 계층을 개발할 때 구현 클래스 없이 인터페이스만 작성해도 개발 완료 가능
- extends JpaRepository<Member, Long>

### 스프링 데이터 프로젝트

- 데이터 저장소에 대한 접근을 추상화해서 개발자 편의를 제공,하고 반복되는 데이터 접근 코드를 줄여 준다.

## 12.2 Spring Data Jpa 설정

- basePackages에는 리포지토리를 검색할 패키지 위치를 적는다.
- 스프링 데이터 JPA는 애플리케이션 실행시 basePackages에 있는 리포지토리 인터페이스들을 찾아서 해당 인터페이스를 구현한 클래스를 동적으로 생성하여 스프링 빈으로 등록한다.
- 개발자가 직접 구현 클래스를 만들지 않아도 된다.

## 12.3 공통 인터페이스 기능

- 스프링 데이터 JPA는 간단한 CRUD 기능을 공통으로 처리하는 JpaRepository 인터페이스를 제공한다.
- 단순히 이 인터페이스를 상속받고 제네릭에 엔티티 클래스와 엔티티 클래스가 사용하는 식별자 타입을 지정하면 된다.
- 스프링 데이터 : Repository <- CrudRepository <- PagingAndSortingRepository <- JpaRepository
- 스프링 데이터 Jpa는 JpaRepository이고, 그 외는 스프링 데이터 프로젝트가 공통으로 사용하는 인터페이스이다.
- save, delete, findOne, getOne(프록시로 조회), findAll 등이 있다.

## 12.4 쿼리 메서드 기능

- 메서드 이름만으로 쿼리를 생성하는 기능
- 인터페이스에 메서드만 선언하면 해당 메서드의 이름으로 적절한 jpql 쿼리를 생성하여 실행한다.

### 메서드 이름으로 쿼리 생성

- findByEmailAndName : 이메일과 이름으로 회원 조회

### JPA NamedQuery

- 메서드 이름으로 Jpa NamedQuery(쿼리에 이름을 부여해서 사용) 호출 가능
- `도메인 클래스.메서드 이름`으로 Named Query를 찾아서 실행한다.

### @Query

- 리포지토리 메서드에 직접 쿼리를 정의하려면 @Query를 사용하면 된다.
- 실행할 메서드에 정적 쿼리를 직접 작성하므로 이름 없는 NamedQuery라고 할 수 있다.
- 네이티브 sql을 사용하려면 @Query에 nativeQuery=true를 설정하면 된다.

### 파라미터 바인딩

- 위치 기반 파라미터 바인딩 ?1
- 이름 기반 파라미터 바인딩 :name을 사용하고 @Param("name") String username

> 코드 가독성과 유지보수성을 위해 이름 기반 파라미터 바인딩을 사용하자.

### 벌크업 수정 쿼리

- @Modifying 사용하면 된다.
- 벌크성 쿼리 실행 후 영속성 컨텍스트를 초기화하고 싶으면 @Modifying(clearAutomatically=true)를 하면 된다.

### 반환 타입

- 결과가 한 건 이상이면 컬렉션 인터페이스를 사용하고 단건이면 반환 타입을 지정한다.
- 조회 결과가 없을 경우 빈 컬렉션을 반환하고, 단건일 경우 null을 반환한다.
- 단건 반환 타입 지정했는데 결과가 2개 이상일 경우 NonUniqueResultException이 발생한다.

### 페이징과 정렬

- Pagable : 페이징 기능 (내부에 Sort 포함)
    - 파라미터에 Pageable을 사용하면 반환 타입으로 List나 Page를 사용할 수 있다.
    - 반환타입으로 Page를 사용할 경우 검색된 전체 데이터 건수를 조회하는 count 쿼리를 추가로 호출한다.
    - Pagable은 인터페이스이므로 실제 사용할 때는 PageRequest (구현체)를 사용한다.
- Sort : 정렬 기능

### 힌트

- JPA Query Hint를 사용하려면 @QueryHints를 사용하면 된다.

### Lock

- @Lock을 사용하면 된다.

## 12.5 명세

- DDD의 명세 개념을 스프링 데이터 JPA에서 사용가능하다.
- 술어 : 참이나 거짓으로 평가, AND, OR 같은 데이터를 검색하기 위한 제약조건 하나하나
- 컴포지트 패턴, 여러 명세 조합 가능
- JpaSpecificationExecutor 인터페이스 상속받으면 된다. Specification을 파라미터로 받아서 검색 조건으로 사용한다.
- where(), and(), or(), not()을 제공한다.

## 12.6 사용자 정의 레포지토리 구현

- 필요한 메서드를 구현할 수 있도록 방법을 지원한다.
- 인터페이스를 작성한 후 구현한 클래스를 작성한다. 구현한 클래스의 이름은 끝에 Impl이 붙어야 JPA가 사용자 정의 클래스로 인식한다.
- 리포지토리 인터페이스에서 사용자 정의 인터페이스르 상속받으면 된다.

## 12.7 Web 확장

- 도메인 클래스 컨버터 기능 : 도메인 클래스 바로 바인딩
- 페이징과 정렬 기능

❓ 힌트는 언제 쓸까? (553p)
❓ Lock은 어떻게 언제 사용할까? 사용 예시와 종류? (554p)
❓ DDD에서 명세는 무엇이고 왜 사용할까? 잘 이해가 안된다. (554p)
❓ 사용자 정의 인터페이스는 왜 사용할까? 네이티브 쿼리로도 해결이 되지 않을때 사용할까? (558p)

## ❓ 질문 모음 및 답변

### 1. ❓ querydsl 페이징에서 listResults()의 결과는 전체 데이터인지, 페이징 결과 데이터인지 궁금하다. (435p)

페이징 결과 데이터와 전체 데이터 수를 함께 반환한다.

### 2. ❓ 세타 조인이란? querydsl에서 from절에 여러 조건을 사용해서 작성하는지? (437p)

```sql
query.from(order, member)
    .where(order.member.eq(member))
    .list(order);

query.from(order, member)
	 .where(order.amount.gt(member.creditLimit))
	 .list(order);
```

세타 조인은 내부 조인의 한 종류이며, **조인 조건이 동등(=)을 포함한 모든 다른 비교 연산자를 사용하는 조인**이다.
QueryDsl에서는 from절에 여러 엔티티를 나열하고 **where절에 조인 조건을 명시**하여 세타 조인을 구현한다.

### 3. ❓ 이 쿼리는 어떤 쿼리를 나타내는지? (437p)

```sql
query.from(item)
    .where(new JPASubQuery()
	    .from(itemSub)
	    .where(item.name.eq(itemSub.name))
	    .list(itemSub))
	)
    .list(item);
```

모든 item을 반환한다 ?

### 4. ❓ 네이티브 sql에서 결과 매핑을 사용할때 @FieldResult를 한번이라도 사용하면 전체 필드를 @FieldResult로 매핑해야한다는 의미? (448p)

`@SqlResultSetMapping`: JPA에서 **네이티브 SQL 쿼리의 결과를 엔티티에 매핑**할 때 사용하는 어노테이션
이 매핑 과정에서 `@FieldResult`를 사용할 수 있는데, SQL 쿼리 결과의 컬럼을 엔티티의 **필드에 직접 매핑하는데 사용**된다.

```sql
@SqlResultSetMapping(
    name = "BookMapping",
    entities = {
        @EntityResult(
            entityClass = Book.class,
            fields = {
                @FieldResult(name = "id", column = "book_id"),
                @FieldResult(name = "title", column = "book_title"),
                @FieldResult(name = "author", column = "book_author"),
            }
        )
    }
)
```

만약 `@SqlResultSetMapping` 내에서 `@FieldResult`를 한 번이라도 사용하면, 해당 엔티티의 모든 필드에 대해 `@FieldResult`를 사용하여 매핑을 명시해야한다.
> 일부 필드만 @FieldResult로 매핑하고 나머지는 자동으로 매핑될 수 없다.

### 5. ❓ 프로시저란? (453p)

프로시저: 데이터베이스 내에 저장되고 이름이 부여된 PL/SQL 프로그램 단위
> 여러 SQL 문과 프로그래밍 로직을 포함한다.

```sql
CREATE PROCEDURE GetEmployeesByDepartment(
    @DepartmentID INT,
    @EmployeeCount INT OUTPUT
)
    AS
BEGIN
SELECT *
FROM Employees
WHERE DepartmentID = @DepartmentID;

SET @EmployeeCount = @@ROWCOUNT;
END
```

구성요소

- **선언부**: 변수, 상수, 커서 등을 선언
- **실행부**: SQL문과 프로그래밍 로직 (IF문, 루프 등)
- **예외 처리부**: 오류 처리 로직

특징

- **매개변수**: 입력, 출력, 입출력 매개변수를 가질 수 있음
- **반환 값**: 하나 이상의 값을 반환할 수 있음
- **트랜잭션**: 여러 SQL문을 하나의 트랜잭션으로 처리 가능

장점

- **성능 향상**: 미리 컴파일되어 저장되므로 실행 속도가 빠름
- **네트워크 트래픽 감소**: 여러 SQL문을 한 번의 호출로 실행
- **보안 강화**: 데이터베이스 객체에 대한 직접 접근을 제한할 수 있음
- **재사용성**: 여러 애플리케이션에서 동일한 프로시저 사용 가능
  단점
- 데이터베이스 **의존성**: DBMS별로 문법이 다를 수 있음
- **디버깅 어려움**: 데이터베이스 내부에서 실행되어 디버깅이 복잡할 수 있음
- 리소스 사용: 복잡한 로직의 경우 데이터베이스 서버에 부하를 줄 수 있음

- JPA/Hibernate에서 사용하기

```sql
StoredProcedureQuery query = em.createStoredProcedureQuery("GetEmployeesByDepartment");
query.registerStoredProcedureParameter("DepartmentID", Integer.class, ParameterMode.IN);
query.registerStoredProcedureParameter("EmployeeCount", Integer.class, ParameterMode.OUT);

query.setParameter("DepartmentID", 1);

List<Employee> result = query.getResultList();
int employeeCount = (int) query.getOutputParameterValue("EmployeeCount");
```

### 6. ❓ JPA 쿼리 힌트는 언제 쓸까? (553p)

> 여기서 말하는 힌트는 sql 힌트가 아닌 jpa 힌트

힌트는 JPA 구현체(예: Hibernate)에게 **특정 쿼리를 어떻게 처리해야 하는지 제안**하는 방법이다.
성능 최적화, 읽기 전용 쿼리 지정, 특정 데이터베이스 기능 활용 등의 목적으로 사용된다.

1. **읽기 전용 쿼리** (org.hibernate.readOnly)
    - 사용 시기: 데이터를 조회만 하고 수정하지 않을 때
    - 장점:
        * 불필요한 스냅샷 생성을 방지하여 메모리 사용량 감소
        * 성능 향상 (변경 감지 작업 불필요)

      ```java
      @QueryHints(value = @QueryHint(name = "org.hibernate.readOnly", value = "true"))
      List<User> findByUsername(String username);
      ```

2. **쿼리 캐시** (org.hibernate.cacheable)
    - 사용 시기: 자주 실행되는 동일한 쿼리의 결과를 캐시하고 싶을 때
    - 장점: 반복적인 쿼리 실행 시 성능 향상
      ```java
      @QueryHints({@QueryHint(name = "org.hibernate.cacheable", value = "true")})
      User findById(Long id);
      ```

3. **페치 크기 설정** (org.hibernate.fetchSize)
    - 사용 시기: 대량의 데이터를 조회할 때 메모리 사용을 최적화하고 싶은 경우
    - 장점: 네트워크 통신 횟수 감소, 메모리 사용 최적화
      ```java
      @QueryHints(value = @QueryHint(name = "org.hibernate.fetchSize", value = "50"))
      List<User> findAll();
      ```

4. **타임아웃 설정** (javax.persistence.query.timeout)
    - 사용 시기: 쿼리 실행 시간을 제한하고 싶을 때
    - 장점: 장시간 실행되는 쿼리로 인한 시스템 부하 방지
      ```java
      @QueryHints(value = @QueryHint(name = "javax.persistence.query.timeout", value = "5000"))
      List<User> findByComplexCondition();
      ```

5. **플러시 모드 설정** (org.hibernate.flushMode)
    - 사용 시기: 특정 쿼리에 대해 플러시 동작을 제어하고 싶을 때
    - 장점: 불필요한 플러시 작업 방지, 성능 최적화
      ```java
      @QueryHints(value = @QueryHint(name = "org.hibernate.flushMode", value = "COMMIT"))
      List<User> findForReport();
      ```

6. **락 모드 설정** (javax.persistence.lock.timeout)
    - 사용 시기: 동시성 제어가 필요한 상황에서 락 획득 대기 시간을 설정하고 싶을 때
    - 장점: 데드락 방지, 동시성 제어 강화
      ```java
      @QueryHints(@QueryHint(name = "javax.persistence.lock.timeout", value = "3000"))
      @Lock(LockModeType.PESSIMISTIC_WRITE)
      User findByIdForUpdate(Long id);
      ```

7. **comment 추가** (org.hibernate.comment)
    - 사용 시기: SQL 로그에 특정 커멘트를 추가하고 싶을 때 (디버깅, 모니터링 목적)
    - 장점: SQL 로그 분석 용이
      ```java
      @QueryHints(value = @QueryHint(name = "org.hibernate.comment", value = "This query is for daily report"))
      List<Sale> findDailySales();
      ```

JPA 쿼리 힌트는 애플리케이션의 성능을 최적화하고, 특정 요구사항을 만족시키는 데 도움이 된다.

> 하지만 과도한 힌트 사용은 코드의 복잡성을 증가시키므로 실제 성능 개선이 필요한 경우에 신중하게 사용해야 한다.
> 또한, 일부 힌트는 특정 JPA 구현체에 종속적일 수 있으므로 이식성을 고려해야 한다.

### 7. ❓ Lock은 어떻게 언제 사용할까? 사용 예시와 종류? (554p)

Lock은 **동시성 제어**를 위해 사용된다. 주로 동시 수정 방지나 일관성 보장을 위해 사용한다.
**낙관적 락**(Optimistic Lock)과 **비관적 락**(Pessimistic Lock)이 있다.
**@Lock** 어노테이션을 통해 사용한다.

1. Lock의 목적
    - **동시성 제어**: 여러 트랜잭션이 동시에 같은 데이터에 접근할 때 데이터의 일관성을 유지
    - **데이터 무결성 보장**: **동시 수정으로 인한 데이터 손상 방지**
    - **일관된 읽기 보장**: 특정 트랜잭션 동안 데이터가 변경되지 않도록 보장

2. Lock의 주요 종류
   **A. 낙관적 락 (Optimistic Lock)**
    - 원리: 데이터 충돌이 드물게 일어난다고 가정하고, **실제 데이터베이스에 반영할 때 충돌을 감지**
    - 구현 방식:
        * 버전 관리: 엔티티에 버전 필드를 추가하여 수정할 때마다 버전을 증가
        * 타임스탬프: 최종 수정 시간을 이용하여 충돌 감지
    - 사용 예시:
      ```java
      @Entity
      @OptimisticLocking(type = OptimisticLockType.VERSION)
      public class Product {
          @Id
          private Long id;
          
          @Version
          private Long version;
      }
      ```
    - 장점: **동시성이 높은 환경에서 성능이 좋음, 데이터베이스 리소스를 적게 사용**
    - 단점: **충돌 발생 시 애플리케이션에서 직접 해결**해야 함

   **B. 비관적 락 (Pessimistic Lock)**
    - 원리: 데이터 충돌이 자주 일어난다고 가정하고, **데이터를 읽는 시점에 락을 걸어 다른 트랜잭션의 접근을 차단**
    - 주요 모드
        * **PESSIMISTIC_READ**: 공유 락, 다른 트랜잭션의 읽기는 허용하지만 쓰기는 차단
        * **PESSIMISTIC_WRITE**: 배타적 락, 다른 트랜잭션의 읽기와 쓰기 모두 차단
        * **PESSIMISTIC_FORCE_INCREMENT**: 버전 정보를 강제로 증가시키는 락
    - 사용 예시
      ```java
      @Lock(LockModeType.PESSIMISTIC_WRITE)
      @Query("SELECT p FROM Product p WHERE p.id = :id")
      Product findByIdForUpdate(@Param("id") Long id);
      ```
    - 장점: 데이터 일관성을 강하게 보장, 충돌 처리가 간단함
    - 단점: **동시성이 떨어지고 데드락 가능성 있음**, 성능 저하 가능성

3. 사용 시나리오
    - 낙관적 락
        * **읽기가 쓰기보다 많은 경우**
        * 짧은 트랜잭션 또는 충돌 가능성이 낮은 경우
    - 비관적 락
        * **동시 수정 가능성이 높은 경우**
            * **온라인 쇼핑몰에서 상품의 재고를 관리**할 때 사용

```java

@Repository
public interface ProductRepository extends JpaRepository<Product, Long> {
    @Lock(LockModeType.PESSIMISTIC_WRITE)
    @Query("SELECT p FROM Product p WHERE p.id = :id")
    Product findByIdForUpdate(@Param("id") Long id);
}

@Service
public class OrderService {
    @Transactional
    public void createOrder(Long productId, int quantity) {
        Product product = productRepository.findByIdForUpdate(productId);
        if (product.getStock() >= quantity) {
            product.decreaseStock(quantity);
        } else {
            throw new InsufficientStockException();
        }
    }
}
```

> 여러 사용자가 동시에 주문을 시도할 때 재고가 정확하게 관리된다.

* 데이터 일관성이 중요한 금융 거래: 계좌 이체 같은 경우, **출금 계좌와 입금 계좌의 잔액을 동시에 업데이트**해야한다.

```java

@Repository
public interface AccountRepository extends JpaRepository<Account, Long> {
    @Lock(LockModeType.PESSIMISTIC_WRITE)
    @Query("SELECT a FROM Account a WHERE a.id = :id")
    Account findByIdForUpdate(@Param("id") Long id);
}

@Service
public class TransferService {
    @Transactional
    public void transfer(Long fromId, Long toId, BigDecimal amount) {
        Account fromAccount = accountRepository.findByIdForUpdate(fromId);
        Account toAccount = accountRepository.findByIdForUpdate(toId);

        if (fromAccount.getBalance().compareTo(amount) >= 0) {
            fromAccount.withdraw(amount);
            toAccount.deposit(amount);
        } else {
            throw new InsufficientFundsException();
        }
    }
}
```

- 동시성이 높은 예약 시스템: 공연 티켓 예약 시스템에서 특정 좌석의 예약 상태를 관리할 때 사용한다.

```java

@Repository
public interface SeatRepository extends JpaRepository<Seat, Long> {
    @Lock(LockModeType.PESSIMISTIC_WRITE)
    @Query("SELECT s FROM Seat s WHERE s.id = :id AND s.status = 'AVAILABLE'")
    Optional<Seat> findAvailableSeatForBooking(@Param("id") Long id);
}

@Service
public class BookingService {
    @Transactional
    public void bookSeat(Long seatId, Long userId) {
        Seat seat = seatRepository.findAvailableSeatForBooking(seatId)
                .orElseThrow(() -> new SeatNotAvailableException());

        seat.book(userId);
    }
}
```

#### @Lock 사용 장단점

- 장점

1. **동시 접근으로 인한 데이터 불일치를 방지**한다.
2. 트랜잭션 격리 수준을 높여 **데이터 정합성을 보장**한다.
3. race condition을 방지하여 비즈니스 로직의 정확성을 유지한다.

- 단점
    - **성능 저하**가 발생할 수 있으므로, 꼭 필요한 경우에만 사용해야 한다
    - **락의 범위를 최소화**하여 동시성을 최대한 유지해야 한다.
    - 데드락 가능성을 고려하여 **락 획득 순서 등을 신중히 설계**해야 한다.

### 8. ❓ DDD에서 명세는 무엇이고 왜 사용할까? 잘 이해가 안된다. (554p)

DDD에서 명세(Specification)는 **도메인 객체가 특정 조건을 만족하는지 검사하는 predicate**(참 또는 거짓을 반환하는 함수)이다.
> 도메인 로직을 캡슐화하는 객체이다.

2. 명세를 사용하는 이유
   a) **비즈니스 규칙의 명확한 표현**
    - 복잡한 비즈니스 규칙을 코드로 명확하게 표현할 수 있다.
    - 도메인 전문가와 개발자 간의 의사소통을 돕는다.

   b) **재사용성**
    - 동일한 규칙을 여러 곳에서 일관되게 적용할 수 있다.
    - 비즈니스 로직의 중복을 줄인다.

   c) **조합** 가능성
    - 여러 명세를 AND, OR 등으로 조합하여 복잡한 규칙을 표현할 수 있다.

   d) 관심사의 **분리**
    - 비즈니스 규칙을 별도의 객체로 분리하여 코드의 구조를 개선한다.

3. JPA
   복잡한 쿼리 조건을 객체지향적으로 표현할 수 있다.

   예시
```java
   public interface ProductRepository extends JpaRepository<Product, Long>, JpaSpecificationExecutor<Product> {
   }

   public class ProductSpecs {
       public static Specification<Product> isPriceGreaterThan(BigDecimal price) {
           return (root, query, criteriaBuilder) ->
               criteriaBuilder.greaterThan(root.get("price"), price);
       }

       public static Specification<Product> isInCategory(String category) {
           return (root, query, criteriaBuilder) ->
               criteriaBuilder.equal(root.get("category"), category);
       }
   }

    // 예시
    List<Product> products = productRepository.findAll(
    Specification.where(ProductSpecs.isPriceGreaterThan(new BigDecimal("100"))
    .and(ProductSpecs.isInCategory("Electronics"))
    );

```

> - `isPriceGreaterThan`: 가격이 특정 값보다 큰 제품을 찾는 조건
> - `isInCategory`: 특정 카테고리에 속한 제품을 찾는 조건

> 명세 패턴을 통해 복잡한 도메인 로직을 명확하고 재사용 가능한 형태로 표현할 수 있다.
>  특히 JPA와 함께 사용할 때, 복잡한 쿼리 조건을 객체지향적이고 도메인 중심적으로 표현할 수 있다.

### 9. ❓ 사용자 정의 인터페이스는 왜 사용할까? 네이티브 쿼리로도 해결이 되지 않을때 사용할까? (558p)

   - **Spring Data JPA가 제공하는 기본 기능을 넘어서는 복잡한 작업을 수행**할 때 사용한다.
   - 레포지토리의 기능을 확장하면서도 **Spring Data JPA의 이점을 유지**할 수 있게 해준다.

   a) 복잡한 비즈니스 로직:
      - 단순 CRUD 연산으로 해결할 수 없는 **복잡한 도메인 로직**이 필요한 경우
   
   b) 특정 기술에 종속적인 기능:
      - **JPA나 Hibernate의 특정 기능을 직접 사용**해야 하는 경우
   
   c) 동적 쿼리 생성:
      - 런타임에 조건에 따라 **동적으로 쿼리를 생성**해야 하는 경우
   
   d) 성능 최적화:
      - 특정 쿼리의 성능을 극대화하기 위해 **저수준 최적화가 필요한 경우**

   ```java
   // 사용자 정의 인터페이스
   public interface CustomProductRepository {
       List<Product> findProductsOnSale();
       void updateProductPrices(String category, double increaseRate);
   }

   // 구현 클래스
   public class CustomProductRepositoryImpl implements CustomProductRepository {
       @PersistenceContext
       private EntityManager em;

       @Override
       public List<Product> findProductsOnSale() {
           // 복잡한 JPQL 또는 Criteria API 사용
           return em.createQuery("SELECT p FROM Product p WHERE p.onSale = true", Product.class)
                    .getResultList();
       }

       @Override
       public void updateProductPrices(String category, double increaseRate) {
           // 벌크 연산 수행
           em.createQuery("UPDATE Product p SET p.price = p.price * :rate WHERE p.category = :category")
             .setParameter("rate", 1 + increaseRate)
             .setParameter("category", category)
             .executeUpdate();
       }
   }

   // 메인 레포지토리 인터페이스
   public interface ProductRepository extends JpaRepository<Product, Long>, CustomProductRepository {
   }
   ```

네이티브 쿼리와의 차이점

- 네이티브 쿼리는 SQL을 직접 작성하는 반면, **사용자 정의 인터페이스는 JPA/Hibernate의 기능을 활용**할 수 있다.
- 네이티브 쿼리는 데이터베이스에 종속적이지만, 사용자 정의 인터페이스는 JPA를 통해 **데이터베이스 독립성을 유지**할 수 있다.

네이티브 쿼리 vs 사용자 정의 인터페이스:

- 네이티브 쿼리: **특정 데이터베이스의 고유 기능**이 필요하거나 **매우 복잡한 SQL이 필요한 경우**
- 사용자 정의 인터페이스: **JPA/Hibernate의 기능을 활용하면서 복잡한 로직을 구현해야 할 때**

> - **성능에 민감한 작업의 경우, 네이티브 쿼리가 더 적합할 수 있다.**
    >     네이티브 쿼리를 사용하면 특정 데이터베이스의 고유 기능과 최적화 기법을 직접 활용할 수 있기 때문이다.

