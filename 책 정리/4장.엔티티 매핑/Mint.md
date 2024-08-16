# 4장. 엔티티 매핑
- 엔티티와 테이블을 매핑하는 방법
    - 객체와 테이블 매핑: `@Entity`, `@Table`
    - 기본키 매핑: `@Id`
    - 필드와 컬럼 매핑: `@Column`
    - 연관관계 매핑: `@ManyToOne`, `@JoinColumn`

## 객체와 테이블 매핑
### @Entity
- 테이블과 매핑할 클래스
- 주의사항
    - 기본 생성자는 필수다(public, protected)
    - final 클래스, enum 클래스, inner 클래스 사용 불가
    - final 필드 불가
> Q&A에 주의사항 관련 대답 참고

### @Table
- 엔티티와 매핑할 테이블

### ddl_auto
- JPA는 매핑정보와 데이터베이스 방언을 사용하여 데이터베이스 스키마를 생성한다.
- ddl_auto를 사용하면 애플리케이션 실행 시점에 데이터베이스 테이블을 자동으로 생성한다.
    - show_sql을 true로 설정함면 DDL을 출력할 수 있다.
- 옵션
    - `create` : drop+create
    - `create-drop `: drop+create+drop
    - `update` : 변경사항만 수정
    - `validate` : db 테이블과 엔티티 매핑정보의 차이가 있으면 경고를 남기고 애플리케이션을 실행하지 않는다. (ddl 수정 X)
    - `none` : 자동 생성 기능을 사용하지 않으려면 옵션 값을 주지 않거나 유효하지 않은 옵션 값을 주면 된다.
> 스키마 자동 생성 기능이 만든 DDL은 운영 환경에서 사용할 만큼 완벽하지 않다.

> 운영 서버에는 validate와 none만 사용하자.


## 기본키 매핑 전략(`@Id`)
- 직접 할당
- 자동 생성: 대리키 사용
    - `IDENTITY` : 기본키 생성을 db에 위임
    - `SEQUENCE` : 데이터베이스 시퀀스를 사용하여 기본키 할당
    - `TABLE` : 키 생성 테이블을 사용
> `SEQUENCE`는 mysql에서 지원 X, TABLE은 모든 db에서 사용 가능

> 키 생성 전략 사용시 `new_generator_mappings = true`로 설정하여 효과적인 최신의 키 생성 전략을 사용하기

### 직접 할당
- `@Id`로 매핑 후 setId로 키 직접 할당하기
    - @Id는 java 기본형과 wrapper 타입 모두 가능

### IDENTITY 전략
- 기본키 생성을 db에 위임하는 전략
    - ex ) MY_SQL의 `auto_increment`
    - `@Id`와 `@GeneratedValue` 사용
- 엔티티를 영속화시키기 위해 식별자가 필요하므로 db에 insert 쿼리를 바로 날린다 (쓰기 지연 X)
- `@Id` `@GeneratedValue(strategy=GenerationType.IDENTITY)`
### SEQUENCE 전략
- 유일한 값을 순서대로 생성하는 db object
- 엔티티 저장을 위해 **db와 2번 통신**
    - **db에서 시퀀스를 조회**하여 엔티티에 할당한 후 영속성 컨텍스트에 저장한 뒤 `flush` 실행시 **엔티티가 db에 저장**된다.
- `@allocationSize`는 성능상의 이유로 기본 50이다.
    - 매번 db와 2번 통신하지 않고, 미리 50 단위로 시퀀스 값을 증가시켜 1~50까지는 메모리에서 식별자를 할당할 수 있다.
    - **db와 1번만 통신**하여 엔티티를 저장할 수 있다.
- `@SequenceGenerator`

### TABLE 전략
- 키 생성 전용 테이블을 만들어 이름과 값으로 사용할 컬럼을 만든다.
    - SEQUENCE 를 흉내낸다.
    - 모든 DB에 적용할 수 있다.
- 엔티티 저장을 위해 **db와 3번 통신**한다.
    - db에서 키를 **조회**하고, **update**한 후 db에 **엔티티를 저장**한다.
- `@TableGenerator`

### AUTO 전략
- 선택한 데이터베이스 방언에 따라 `IDENTITY`, `SEQUENCE`, `TABLE` 전략 중 하나를 자동으로 선택한다.
    - MySQL이면 `IDENTITY`를 사용한다.

#### 자연키와 대리키
- 자연키 : 비즈니스에 의미가 있는 키
    - ex) 주민등록번호
- 대리키(대체키) : 임의로 만들어진 키
    - ex) auto_increment, sequence, 키 생성 테이블
- 변경될 수 있는 자연키보다 대리키 사용을 권장한다.
- 자연키의 후보가 될 수 있는 컬럼들은 unique 인덱스를 사용하는 것을 권장한다.

## 필드와 컬럼 매핑(`@Column`)
- `@Column` : 컬럼 매핑
- `@Enumerated` : 자바의 enum 타입 매핑
- `@Temporal` : 날짜 타입 매핑
- `@Lob` : BLOB, CLOB 타입 매핑
- `@Transient` : 필드를 db에 매핑 X
- `@Access` : JPA가 엔티티에 매핑하는 방식 지정
    - 필드 접근
    - 프로퍼티 접근

### `@Column`
- nullable, unique, columnDefinition, length, precision, scale은 DDL 생성시 영향
- 기본 타입은 not null 제약조건이 자동으로 붙기 때문에, nullable=true(기본값)과 충돌이 일어날 수 있다.
    - 객체 타입을 사용하자.
    - 기본 타입 사용시 nullable=false 설정하기

### `@Enumerated`
- EnumType.ORDINAL : 순서를 db에 저장
    - 기본값이지만 권장 X, 순서는 바뀔 수 있음
- EnumType.STRING : enum 이름을 db에 저장

### `@Temporal`
- 날짜 타입, 거의 안씀

### `@Lob`
- 문자열이면 CLOB으로 매핑, 그외는 BLOB
- CLOB : String,char[]
- BLOB : byte[]
> 긴 문자열이나 데이터 저장시 사용

### `@Transient`
- db에 매핑하지 않는 필드

### `@Access`
- JPA가 엔티티 데이터에 접근하는 방식 지정
    - `@Id `위치에 따라 설정됨
- 필드 접근 : 필드에 직접 접근(`private`)도 가능
- 프로퍼티 접근 : 접근자(`Getter`) 사용
> 필드 접근  + 프로퍼티 접근 혼합 가능
