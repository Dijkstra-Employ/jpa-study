# 책 내용 정리

### [0718] 351~370

**QueryDsl**

* jpql 빌더, 코드 기반, 단순, 사용 쉬움

**Native Sql**

* sql 직접 사용, 특정 db에 의존
  jdbc 직접 사용, sql mapper framework
* jpa와 함께 사용할 경우 sql 실행 전 영속성 컨텍스트를 수동으로 flush하여 db와 동기화하기

**JPQL**

* 엔티티 객체를 대상으로 쿼리
* 특정 db의 sql에 의존하지 않는다.
* jpql은 결국 sql로 변환된다.

**select**

* 대소문자 구분
* 엔티티 이름
* 별칭 필수

**TypeQuery, Query**

* `TypeQuery` : 반환타입 명확하게 지정, Object 반환
  new 키워드를 지정하여 반환받을 클래스 지정 가능
* `Query` : 반환타입 지정 X ex) 여러 엔티티, 컬럼 선택할 경우, Object[] 반환
* 결과 조회
    * `query.getResultList()` : 결과를 반환
    * `query.getSingleResult()` : 결과가 정확히 하나일때 사용

**파라미터 바인딩**

* 이름 기준 파라미터(:) : jpql 지원
* 위치 기준 파라미터(?) : jdbc, jpql 지원

**프로젝션**

* select절에 조회할 대상을 지정
* 프로젝션 대상에는 엔티티, 임베디드 타입, 스칼라 타입이 있다.
* `Entity` 프로젝션 : 조회한 엔티티는 영속성 컨텍스트에서 관리
* `임베디드` 타입 프로젝션, `스칼라` 타입 프로젝션 : 영속성 컨텍스트에서 관리 X

**페이징 API**

* `getFirstResult` : 조회 시작 위치
* `getMaxResults` : 조회할 데이터 수

**집합과 정렬**

* COUNT, SUM, AVG, MAX, MIN
* null값 무시, DISTINCT로 중복 제거 가능
* GROUP BY : 그룹화
* HAVING : 그룹화된 데이터 필터링
* ORDER BY : 정렬

**JPQL 조인**

* 내부 조인 (INNER JOIN, JOIN) : sql 조인과 다르게 jpql 조인은 연관 필드를 사용한다.

❓ 임베디드와 스칼라 타입은 영속성 컨텍스트에서 왜 관리를 안할까? (361p)

❓ 데이터베이스 방언이란? (364p)

### [0719] 371~391

**외부 조인**

* outer 생략 가능하다.
  **컬렉션 조인**
* 컬렉션을 사용하는 곳에 조인한다.
* 컬렉션 조인시 JOIN과 같은 기능을 하는 IN을 사용할 수 있지만, 유물이므로 사용 X
  **세타 조인**
* 내부 조인만 지원한다.
* 전혀 관계없는 엔티티도 조인할 수 있다.
  **on**
* 조인 대상을 필터링하고 조인할 수 있다.

**fetch join**

* jpql에서** 성능 최적화**를 위해 제공하는 기능
* 연관된 엔티티나 컬렉션을 한번에 같이 조회하는 기능

* **엔티티 페치 조인**
    * 연관된 엔티티를 함께 조회한다.
    * fetch join은 **별칭을 사용할 수 없다**. (hibernate는 별칭 허용)
    * 객체 그래프를 유지하며 조회할 수 있다.
    * 지연 로딩이 일어나지 않는다.

* **컬렉션 페치 조인**
    * 연관된 일대다 관계의 컬렉션을 페치 조인한다.
    * 일대다 조인은 결과가 증가할 수 있지만, 일대일, 다대다 조인은 결과가 증가하지 않는다.

* `DISTINCT` : 중복 제거
    * sql에 DISTINCT를 추가하고(결과에서 중복 제거) 애플리케이션에서 한번 더 중복을 제거(조회 대상으로 중복 제거)한다.
    * `fetch join` vs `inner join`
        * 일반 조인은 fetch join과 다르게 **연관된 엔티티를 조회하지 않는다**.

* **fetch join의 특징과 한계**
    * fetch join은 **글로벌 로딩 전략(FetchType)보다 우선**한다.
        * 최적화를 위해 글로벌 로딩 전략을 즉시 로딩으로 설정하면 애플리케이션 전체에 항상 즉시 로딩이 일어난다. 성능에 악영향을 끼친다.
          ** => 글로벌 로딩 전략은 되도록이면 지연 로딩을 사용하고, 최적화가 필요하면 fetch join을 적용하는 것이 효과적이다.***
* fetch join 대상에는 **별칭을 줄 수 없다.**
    * **둘 이상의 컬렉션을 fetch 할 수 없다.**
    * 컬렉션을 fetch join하면 **페이징 api를 사용할 수 없다.**

**경로 표현식**

* `.`을 찍어 객체 그래프를 탐색한다.
    * **상태** 필드 : 값 저장 필드, 더는 탐색할 수 없다.
    * **연관** 필드 : 연관관계 필드, 임베디드 타입 포함
        * **단일 값 연관 필드** : 대상이 엔티티, `@ManyToOne`, `@OneToOne` 묵시적 내부 조인(inner join), 계속 탐색이 가능하다.
        * 컬렉션 값 연관 필드 : 대상이 컬렉션, `@OneToMany`, `@ManyToMany` 묵시적 내부 조인(inner join), 별칭을 얻지 못하면 계속 탐색이 불가능하다.
* **묵시적 조인** : 경로 표현식에 의해 묵시적으로 조인 일어남
* **명시적 조인**: JOIN을 직접 적어주는 것

**서브 쿼리**

* where, having 절에서만 사용이 가능하다.
  **서브 쿼리 함수**
* exists: 서브 쿼리에 결과가 존재하면 참
* all, any, some : 비교연산자와 함께 사용
    * all : 조건 모두 만족
    * any, some : 조건 하나라도 만족
* in : 하나라도 같은 것이 있으면 참
  **조건식**
* 타입 표현 : 문자,숫자,날짜, Boolean,Enum, 엔티티 타입
* 연산자 우선순위: 경로 탐색 연산, 수학 연산, 비교 연산, 논리 연산
  **비교식**
* `Between` A AND B : A~B 값 사이(A,B 값 포함)
* X `IN` 예제: X에 예제와 같은 값이 하나라도 있으면 참
* 문자표현식 `LIKE` 패턴값 : 문자표현식과 패턴값 비교
* NULL 비교식 `IS NULL` : NULL이면 참
  컬렉션 식
* 빈 컬렉션 비교식 : IS EMPTY : 비어있으면 참

❓ right는 왜 fetch join 형식에 없을까? (373p)

❓ 영속성 컨텍스트에서 분리되어 준영속 상태가 되어도 연관된 엔티티를 조회할 수 있는 이유? (375p)

❓ 준영속 상태에서도 페치 조인을 이용하면 객체 그래프 탐색 가능한 이유? 별칭을 사용하면 연관된 데이터 수가 달라져서 무결성이 깨진다는 의미? 컬렉션 사용하면 페이징 api를 사용할 수 없는 이유? (381p)

❓ 컬렉션 값 경로 탐색시 별칭을 사용하면 탐색을 새로 시작할 수 있는 이유? (385p)

### [0721] 392~412

* 컬렉션의 멤버식 : 엔티티나 값 MEMBER [OF] 연관경로

**스칼라식**

* 스칼라 : 가장 기본적인 타입
* **수학 식** : +,-,*,/...
* **문자함수** : CONCAT, SUBSTRING..
* **수학함수** : ABS, SQRT..
* **날짜함수**: 데이터베이스의 현재 시간 조회(CURRENT_DATE,..)

**CASE 식**

* **기본 CASE** : 조건식 사용 가능
* **심플 CASE** : 조건식 사용 불가
* **COALESCE **: 첫번째 값 null이면 두번째 값 반환, 아니면 첫번째값 반환
* **NULLIF**: 두 값이 같으면 null 반환, 다르면 첫번째 값 반환

**다형성 쿼리**

* jpql로 부모 엔티티를 조회하면 그 자식 엔티티도 함께 조회한다.
* `TYPE` : 엔티티의 상속 구조에서 조회 대상을 특정 자식 타입으로 한정할 때 사용
* `TREAT` : 부모 타입을 특정 자식 타입으로 다룰 때 사용, FROM,WHERE에서 사용가능

**사용자 정의 함수 호출**

* 방언 클래스를 상속해서 구현하고 사용할 데이터베이스 함수를 미리 등록한다.
* hibernate.dialect에 해당 방언을 등록한다.

**기타 정리**

* enum은 비교 연산만 정의한다.
* 임베디드 타입은 비교를 지원하지 않는다.
* EMPTY STRING: JPA 표준은 ''로 정했지만 데이터베이스에 따라 NULL로 사용하기도 한다,
* NULL 정의: 조건을 만족하는 데이터가 하나도 없으면 NULL이다.
    * Null == Null : 알수 없는 값
    * Null is Null은 True
    * FALSE AND NULL = FALSE
    * TRUE OR NULL = TRUE
    * NOT NULL = NULL

**엔티티 직접 사용**

* **기본키 값** : 객체 인스턴스는 참조 값으로 식별하고 테이블 로우는 기본키 값으로 식별한다.
  ***=> 엔티티 직접 사용하면 sql에서 엔티티의 기본키를 사용한다.***
* **외래키 값** : 연관 엔티티 직접 사용시 외래키 값을 사용한다. (묵시적 조인은 일어나지 않는다.)

**Named Query : 정적 쿼리, **`@NamedQuery`

* **동적 쿼리** : jpql을 문자로 완성해서 직접 넘기는 것, 런타임에 특정 조건에 따라 jpql을 동적으로 구성
* **정적 쿼리** : 미리 정의한 쿼리에 이름을 부여, 한번 정의하면 변경할 수 없는 정적 쿼리, 파싱된 결과를 재사용하므로 성능상 이점
* NamedQuery 여러개일 경우 `@NamedQueries` 사용
* 멀티 라인을 정의하기 위해 XML에 정의하는 것이 더 편리하다.(XML과 어노테이션에 같은 설정이 있으면 XML이 우선권을 가진다.)

**Criteria**

* jpql을 자바 코드로 작성하도록 도와주는 빌더 클래스 API
* 문법 오류를 컴파일 단계에서 잡을 수 있다.
* 코드가 복잡하고 장황해서 직관적으로 이해하기 어렵다.

**Criteria 작성 방법**

1. Criteria 쿼리를 생성하려면 먼저 Criteria 빌더를 얻어야한다.
2. Criteria 쿼리 빌더에서 Criteria 쿼리를 생성한다.
3. FROM 절을 생성한다. 반환된 값은 조회의 시작점이라는 의미로 쿼리 루트라 한다.
4. SELECT 절을 생성한다.

**Criteria 조회**

* 조회 대상을 하나만 지정 : `cq.select(m)`
* 조회 대상을 여러 건 지정:` cq.multiselect(m.get("username"), m.get("age"));`
  또는 `cq.select(cb.array(m.get("username"), m.get("age"));`
* 중복 제거 : `.distinct(true);`
* NEW, construct() : `cb.construct(클래스타입, ...)`

❓ 사용자 정의 함수 호출방법이 이해가 안간다. (398p)

❓ 왜 enum은 비교 연산만 지원하고, 임베디드 타입은 비교를 지원하지 않을까? (399p)

❓ Named 쿼리는 영속성 유닛 단위로 관리된다고 하는데, 영속성 유닛이란? (403p)

❓ 언제 jpql에서 생성자를 쓸까? (412p)

### [0722] 413~433

**튜플**

* 튜플 전용 별칭을 필수로 할당
* 튜플 별칭으로 데이터 조회
* 이름 기반이므로 순서 기반인 `Object[]`보다 안전

**집합**

* 집합: GROUP BY, HAVING
* 정렬: cb.desc, cb,asc
* 조인: join(), JoinType
    * `JoinType.INNER` : 내부 조인
    * J`oinType.LEFT` : 외부 조인
* IN: `in(..) `사용

**서브 쿼리**

* Subquery 생성하고 메인 쿼리는 where에서 서브 쿼리를 사용한다.
* 상호 관련 서브쿼리시 `correlate` 메서드를 사용하여 메인 쿼리의 별칭을 서브 쿼리에서 사용한다.

**CASE 식**

* `selectCase() `메서드와 `when()`, `otherwise() `메서드 사용
  파라미터 정의
* `cb.parameter(타입, 파라미터 이름)`으로 파라미터 정의
* `setParameter`: 해당 파라미터에 사용할 값 바인딩

**네이티브 함수 호출**

* `cb.function `메서드 사용

**동적 쿼리**

* 다양한 검색 조건에 따라 실행 시점에 쿼리 생성
* 코드 기반인 Criteria로 작성하는 것이 편리(vs jpql)
* 하지만 Criteria는 장황하여 파악하기 어렵다.

**함수 정리**

* Criteria는 JPQL 빌더 역할을 하므로 JPQL 함수를 코드로 정의한다.
* 조건 함수, 스칼라와 기타 함수, 집합 함수, 분기 함수

**Criteria 메타 모델**

* 완전하게 코드 기반으로 사용하기 위해 **메타 모델**을 사용할 수 있다.
* 메타 모델 클래스를 생성하면 되는데, 코드 자동 생성기가 엔티티 클래스를 기반으로 메타 모델 클래스를 만들어 준다.

**QueryDsl**

* Criteria는 코드 기반이지만 너무 복잡하고 어렵다.
* 쿼리를 문자가 아닌 코드로 작성해도 쉽고 간결하며 모양도 쿼리와 비슷하다.
* JPQL 빌더 역할을 하는데, JPA Criteria를 대체할 수 있다.
* 오픈소스이며 데이터를 조회하는데 기능이 특화되어있다.

**시작**

1. JPAQuery 객체를 생성하는데, 엔티티 매니저를 생성자에 넘겨준다.
2. 쿼리 타입을 생성하는데 생성자에는 별칭을 주면 된다.
3. `from, where, orderBy, list` 등을 작성한다.

**기본 Q 생성**

* 쿼리 타입(Q)는 사용하기 편리하도록 기본 인스턴스를 보관하고 있다.
* 같은 엔티티를 조인하거나 같은 엔티티를 서브 쿼리에 사용하면 별칭을 다르게 지정해주어야한다.

**검색 조건 쿼리**

* QueryDSL의 `where` 절에는 `and`나 `or`을 사용할 수 있다.
* 여러 검색 조건을 사용해도 된다.
* `between, contains, startsWith` 등을 제공한다.

❓ JoinType.LEFT는 Left Outer Join을 의미하는 건지, 외부 조인만을 의미하는 것인지? (417p)

❓ 네이티브 함수란? (421p)

# Q&A

### ❓ 1. 임베디드와 스칼라 타입은 영속성 컨텍스트에서 왜 관리를 안할까?

- 이들은 **독립적인 생명주기가 없고 엔티티에 종속적**이기 때문이다. (고유한 식별자를 갖지 않아 개별적으로 추적할 필요가 없다.)
  엔티티의 일부로 취급되어 변경 감지나 지연 로딩 등이 필요 없어 관리 비용을 줄일 수 있다.

### ❓ 2. 데이터베이스 방언이란?

데이터베이스 방언은 **JPA가 다양한 데이터베이스와 호환되게 해주는 기능**이다.
**각 데이터베이스는 서로 다른 SQL 문법, 함수, 데이터 타입**을 가지고 있다.
예를 들어, 페이징 처리나 고유 함수 등을 데이터베이스별로 적절하게 변환해준다.

### ❓ 3. right는 왜 fetch join 형식에 없을까?

**right fetch join은 주 엔티티가 없는 결과도 포함할 수 있어, 결과 처리가 복잡해질 수 있다.**

### ❓ 4. 영속성 컨텍스트에서 분리되어 준영속 상태가 되어도 연관된 엔티티를 조회할 수 있는 이유?

- 이미 로드된 데이터는 **객체 그래프를 통해 접근 가능**하기 때문이다.
    - 단, 지연 로딩은 동작하지 않는다. 연관된 엔티티가 로드되지 않기 때문이다.
- **준영속 상태가 되면 해당 엔티티는 영속성 컨텍스트에서 제거**되지만, 엔티티 객체 자체는 메모리에 남아있기 때문이다.
    - 단, 영속성 컨텍스트의 관리 대상에서 벗어난 상태가 된다. ( **변경 감지, 지연 로딩 등 영속성 컨텍스트의 기능을 사용할 수 없게 된다**.)

#### 준영속 상태

**한 번 영속 상태였다가 영속성 컨텍스트에서 분리된 상태**의 엔티티를 말한다.
=> 준영속 상태는 영**속성 컨텍스트의 관리를 받지 않으면서도 식별자를 가진 상태**로, **필요에 따라 다시 영속 상태로 전환할 수 있는 중간 상태**라고 볼 수 있다.

- 특징
    - 영속성 컨텍스트가 제공하는 기능(변경 감지 등)을 사용할 수 없다.
    - 식별자 값을 가지고 있다.
    - 지연 로딩(Lazy Loading)을 할 수 없다.

- 관련메서드
    - 엔티티 매니저의 detach() 메소드 호출
    - 영속성 컨텍스트를 초기화하는 clear() 메소드 호출
    - 영속성 컨텍스트를 종료하는 close() 메소드 호출

- 사용 상황
    - **영속성 컨텍스트의 메모리를 최적화**할 때 사용한다.
    - 애플리케이션의 다른 계층에 엔티티를 전달할 때 사용할 수 있다.

> **준영속 상태의 엔티티는 변경 사항이 자동으로 데이터베이스에 반영되지 않는다**.
> 필요시 **merge()** 메소드를 통해 다시 영속 상태로 변경할 수 있다.

#### 연관된 엔티티

- **엔티티가 준영속 상태가 되면, 그와 직접 연관된 엔티티들도 일반적으로 준영속 상태가 된다.**
    - 엔티티가 준영속 상태가 될 때 연관 엔티티의 상태는 설정과 로딩 방식에 따라 달라질 수 있다.
      대체로 직접 연관되고 이미 로드된 엔티티들은 함께 준영속 상태가 되는 경향이 있다.

- Cascade
    - CascadeType.DETACH가 설정된 경우: 연관 엔티티도 함께 준영속 상태가 된다.
    - 그렇지 않은 경우: 직접 관련된 엔티티만 준영속 상태가 되고, 나머지는 영속 상태를 유지할 수 있다.
- 로딩 방식
    - **즉시 로딩(Eager Loading)된 연관 엔티티: 함께 준영속 상태가 된다.**
    - 지연 로딩(Lazy Loading)된 연관 엔티티: 아직 로드되지 않았다면 영향을 받지 않는다.
- 객체 그래프 탐색
    - **이미 로드된 연관 엔티티는 준영속 상태에서도 접근 가능하다.**
        - Java의 Heap 메모리에 엔티티는 여전히 존재한다. (영속성 컨텍스트에서만 제거된 상태이다.)
    - 그러나 새로운 지연 로딩은 발생하지 않는다.

> 복잡한 객체 그래프에서는 일부 엔티티만 준영속 상태가 될 수 있다.

#### 영속 컨텍스트 != 메모리(Java의 Heap Memory)

- **영속성 컨텍스트:** J**PA가 엔티티를 관리**하는 논리적인 공간이다.
- 메모리: 실제 객체가 저장되는 물리적인 공간이다.

#### 준영속 상태란?

- 해당 엔티티는 영속성 컨텍스트에서 제거되고, 더이상 JPA가 해당 엔티티의 변경 사항을 추적하지 않는다.
- 엔티티 객체 자체는 여전히 Java의 heap 메모리에 존재하기 때문에, Java에서 여전히 해당 객체를 참조하고 데이터를 읽을 수 있다.
    - **엔티티가 영속성 컨텍스트에서 분리되었다는 것이지, 메모리에서 제거된 것은 아니다.**
- 준영속 상태가 된 이후에는, 해당 엔티티의 변경사항이 자동으로 db에 반영되지 않는다.
  **=> JPA의 관리대상에서 제거되어 변경 감지나 지연 로딩 등의 기능을 사용할 수 없지만, Java 객체로서의 존재는 유지된다.**

## ❓ 5. 준영속 상태, 페치 조인, 별칭 사용, 컬렉션과 페이징 API 관련 질문:

### 5.1 페치 조인 대상에는 별칭을 줄 수 없는데, 몇몇 구현체는 별칭을 지원한다. 하지만 별칭을 사용하면 연관된 데이터 수가 달라져서 데이터 무결성이 깨질 수 있으므로 조심해서 사용해야한다. => ??

   ```sql
   SELECT m
   FROM Member m
            JOIN FETCH m.orders
   ```

모든 회원과 그들의 모든 주문을 가져오는 쿼리이다.

- 별칭 사용

   ```sql
   SELECT m FROM Member m JOIN FETCH m.orders o WHERE o.status = 'COMPLETED'
   ```

  별칭을 사용하여 조건을 추가하여 쿼리를 수행하면, '완료된 주문'만 가져오게 되어 **일부 회원의 주문 데이터가 불완전해진다.**
  완료된 주문만 가져오므로 완료되지 않은 주문을 가진 회원에 대해서는 불완전한 주문 데이터를 갖게 된다.

### 2차 캐시란? 연관된 데이터 수가 달라진 상태에서 2차 캐시에 저장되면 다른 곳에서 조회할때도 연관된 데이터 수가 달라질 수 있다. => ??

**2차 캐시는 여러 세션에서 공유하는 캐시**이다.
위와 같은 불완전한 데이터가 2차 캐시에 저장되면, 다른 세션에서 이 데이터를 조회할 때도 불완전한 상태로 조회된다.

예를 들어, 위의 쿼리로 인해 **"완료된 주문만 있는 회원 객체"가 2차 캐시에 저장됐다고 가정**한다면, 이후 다른 세션에서 이 회원의 모든 주문을 보려고 해도, 캐시된 불완전한 데이터만 보게 된다. 데이터
일관성 해침)

### 하이버네이트에서 컬렉션을 페치 조인하고 페이징 api를 사용하면 경고를 띄우며 메모리에서 페이징 처리를 한다. 그 이유는?

- 회원(Member)과 주문(Order)이 일대다 관계라고 가정했을때 fetch join 쿼리는 아래와 같다.

   ```sql
   SELECT m FROM Member m JOIN FETCH m.orders
   ```

이 쿼리는 모든 회원과 그들의 모든 주문을 함께 가져온다.

- 페이징을 적용하여 첫 5명의 회원만 가져올 경우
    - **회원과 주문이 조인된 결과 집합**이 생성된다. 이를 페이징 대상으로 한다.
    - 이 결과 집합에서 '5개 행'을 가져오는 것은 '5명의 회원'을 의미하지 않을 수 있다. (한 회원이 여러 주문을 가지기 때문이다.)
    - **5명의 Member를 가져오기 위해, 하이버네이트는 모든 데이터를 한번에 메모리로 가져오고, 메모리 상에서 페이징 처리를 수행**한다.
    - 위의 과정을 통해 페이징시 정확한 결과를 보장하지만, 대량의 데이터를 다룰 때 **심각한 성능 문제**를 일으킬 수 있다.

**=> 컬렉션 fetch join과 페이징을 함께 사용하지 말자.**

- 대안
    - **페치 조인 대신 batch size를 설정하면 N+1 문제를 해결**하면서 페이징을 사용할 수 있다.
        - fetch join은 연관된 데이터를 한번에 다 가져오지만(조인), 사용하지 않으면 필요할때 연관된 엔티티를 가져올 수 있다.
        - **5명의 member를 조회하고, 연관된 order에 접근할때 batch size만큼만 Order를 조회한다. (n+1 문제 완화)**
    - 또는 단순히 회원만 페이징하여 가져오고, 필요할 때 주문을 별도로 조회하는 방식을 사용할 수도 있다.

#### batch size로 N+1 문제 해결하기

- batch size: 연관된 엔티티들을 조회할 때 한 번에 몇 개의 엔티티를 가져올지 지정하는 값
    - '조회할 엔티티의 ID 개수'를 의미**
- **지연 로딩 시 연관된 엔티티를 조회할 때, 지정된 batch size만큼 한 번에 조회**
- 이를 통해 여러 번의 쿼리를 **하나의 IN 쿼리로 줄일 수 있다.**
- @BatchSize 어노테이션 사용 또는 XML 설정으로 지정할 수 있다.
    - @BatchSize(size = 100)

- fetch join

```java

@Entity
public class Member {
    @Id
    @GeneratedValue
    private Long id;
    private String name;

    @OneToMany(mappedBy = "member")
    private List<Order> orders = new ArrayList<>();
}

@Entity
public class Order {
    @Id
    @GeneratedValue
    private Long id;

    @ManyToOne(fetch = FetchType.LAZY)
    private Member member;
}

    public List<Member> findMembersWithOrders() {
        return em.createQuery(
                        "SELECT DISTINCT m FROM Member m JOIN FETCH m.orders", Member.class)
                .getResultList();
    }
```

> 페이징을 위해 연관된 모든 엔티티를 메모리에 가져오므로 성능에 좋지 않다.

- batch size

```java

@Entity
public class Member {
    @Id
    @GeneratedValue
    private Long id;
    private String name;

    @OneToMany(mappedBy = "member")
    @BatchSize(size = 100)  // Batch Size 설정
    private List<Order> orders = new ArrayList<>();
}

@Entity
public class Order {
    @Id
    @GeneratedValue
    private Long id;

    @ManyToOne(fetch = FetchType.LAZY)
    private Member member;
}

@Service
public class MemberService {

    private final MemberRepository memberRepository;

    @Autowired
    public MemberService(MemberRepository memberRepository) {
        this.memberRepository = memberRepository;
    }

    @Transactional(readOnly = true) // 한 트랜잭션 안에서 실행되므로 괜찮다.
    public List<MemberDto> findMembersWithOrderCount(int pageNumber, int pageSize) {
        Pageable pageable = PageRequest.of(pageNumber - 1, pageSize);
        Page<Member> memberPage = memberRepository.findAllMembers(pageable);

        return memberPage.getContent().stream()
                .map(m -> new MemberDto(m.getName(), m.getOrders().size()))
                .collect(Collectors.toList());
    }
}

@Repository
public interface MemberRepository extends JpaRepository<Member, Long> {
    @Query("SELECT m FROM Member m")
    Page<Member> findAllMembers(Pageable pageable);
}
```

> batch size를 너무 크게 지정할 경우 오히려 성능이 나빠질 수 있다.

### ❓ 6. 컬렉션 값 경로 탐색시 별칭 사용 이유:

- **별칭을 통해 새로운 탐색 시작점을 제공**하여 더 **복잡한 쿼리 구성**이 가능해진다.

```sql
SELECT m
FROM Member m
         JOIN m.orders o
WHERE o.status = 'COMPLETED'
```

### ❓ 7. 사용자 정의 함수 호출 방법:

- 데이터베이스 방언 클래스를 확장하고 사용자 정의 함수를 등록한 후 JPQL에서 사용

1. 데이터베이스 방언 클래스 확장
   ```java
   public class MyDialect extends H2Dialect {
       public MyDialect() {
           registerFunction("my_custom_function", new StandardSQLFunction("my_custom_function", StandardBasicTypes.STRING));
       }
   }
   ```
2. 설정 파일에 방언 등록
   ```xml
   <property name="hibernate.dialect" value="com.example.MyDialect"/>
   ```
3. JPQL에서 사용
   ```java
   SELECT my_custom_function(m.name) FROM Member m
   ```

### ❓ 8. 왜 enum은 비교 연산만 지원하고, 임베디드 타입은 비교를 지원하지 않을까? (399p)

- **enum은 고정된 값 집합이라 비교가 간단**하지만, **임베디드 타입은 복잡한 구조로 단순 비교가 어렵다**.

- enum: 미리 정의된 상수 집합으로, 쉽게 비교 가능
  ```java
  @Enumerated(EnumType.STRING)
  private OrderStatus status;
  // JPQL: WHERE m.status = 'COMPLETED'
  ```
- 임베디드 타입: 여러 필드로 구성된 복합 타입으로, 단순 비교가 어렵다.
  ```java
  @Embeddable
  public class Address {
      private String street;
      private String city;
  }
  // 직접 비교 불가: WHERE m.address = :address
  // 대신: WHERE m.address.street = :street AND m.address.city = :city
  ```

### ❓ 9. Named 쿼리는 영속성 유닛 단위로 관리된다고 하는데, 영속성 유닛이란? (403p)

- 연관된 엔티티 클래스들의 집합으로, 하나의 데이터베이스에 매핑된다.
    - **하나의 데이터베이스에 매핑되는 엔티티들의 그룹**이다.
- persistence.xml 파일에 정의된다.

  ```xml
  <persistence-unit name="myPU" transaction-type="RESOURCE_LOCAL">
      <class>com.example.Member</class>
      <class>com.example.Order</class>
      <properties>
      </properties>
  </persistence-unit>
  ```

### ❓ 10. JPQL에서 생성자 사용하는 이유?

**복잡한 쿼리 결과를 DTO로 직접 매핑**할 때 유용하다.

```java
public class MemberDTO {
    public MemberDTO(String name, int orderCount) { ...}
}

// JPQL
SELECT new com.example.MemberDTO(m.name,SIZE(m.orders))FROM Member m
```

> 패키지 포함 전체 클래스 이름을 명시해야한다.

### ❓ 11. JoinType.LEFT:

Left Outer Join을 의미한다.
왼쪽 엔티티는 모두 포함하고, 오른쪽 엔티티는 조건에 맞는 것만 포함

```java
SELECT m FROM Member m LEFT JOIN m.orders o
```

> 주문이 없는 회원도 포함하여 결과를 반환

### ❓ 12. 네이티브 함수:

**특정 데이터베이스에만 존재하는 함수**
JPA에서 직접 지원하지 않지만, 사용자 정의 함수로 등록하여 사용할 수 있다.

- MySQL의 JSON 관련 함수

```sql
SELECT JSON_EXTRACT(m.attributes, '$.key')
FROM Member m
```
