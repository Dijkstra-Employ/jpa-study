# 11장. 웹 애플리케이션 제작
## 프로젝트 환경설정
- **핵심 라이브러리**
  스프링 MVC, 스프링 ORM, JPA,hibernate, H2 DB, Connection Pool, Web, Slf4J&LogBack, test
- **메이븐 dependency의 scope 설정**
  compile, runtime, provided(외부에서 라이브러리 사용), test
- **의존성 전이**
  의존관계가 있는 라이브러리도 함께 내려받아 라이브러리에 자동으로 추가한다.
- **스프링 프레임워크 설정**
    - `web.xml` : 웹 애플리케이션 환경설정 파일
    - `webAppConfig.xml`: 스프링 웹 관련 환경설정 파일
    - `appConfig.xm`l: 스프링 애플리케이션 관련 환경설정 파일
> 스프링 프레임워크를 설정할때 보통 웹 계층(webAppConfig)과 비즈니스 도메인 계층(appConfig)을 나누어 관리한다.

### webAppConfig
- `<mvc:annotaion-driven>` : 스프링 mvc 기능 활성화
- `<context:component-scan>` : basePackages를 포함한 하위 패지키를 검색하여 `@Component`, `@Service`, `@Controller` 어노테이션이 붙어 있는 클래스들을 스프링 빈으로 자동 등록
- `<bean>` : 스프링 빈 등록

### appConfig
- `<tx:annotation-driven>` : 스프링 프레임워크가 제공하느 어노테이션 기반의 트랜잭션 관리자를 활성화한다. `@Transactional`이 붙은 곳에 트랜잭션을 적용한다.
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

### 하이버네이트 속성 설정
- `hibernate.dialect` : 사용할 데이터베이스 방언 설정
- `hibernate.show_sql` : 실행하는 sql 출력
- `hibernate.formate_sql` : SQL을 보기 좋게 출력
- `hibernate.user_sql_comments` : 코멘트를 남긴다.
- `hibernate.id.new_generator_mappings` : JPA에 맞춘 새로운 id 생성 방법 사용
- `hibernate.hbm2ddl_auto` :
    -  `create` : 기존 ddl을 제거하고 새로 생성
    - `create-drop` : create와 같은데 애플리케이션을 종료할 때 생성한 ddl을 제거
    - `update` : 현재 db를 비교해서 변경사항만 수정
    - `validate` : 현재 엔티티 매핑 정보와 데이터베이스 스키마가 같은지 비교한다. 다르면 경고를 남기고, 애플리케이션을 실행하지 않는다. ddl을 변경하지 않는다.

## 어노테이션
`@Repository`
- 스프링 빈 자동 등록
- JPA 전용 예외 발생시 스프링이 추상화한 예외로 변환
  `@PersistenceContext`
- 스프링이나 J2EE 컨테이너를 사용하면 컨테이너가 엔티티 매니저를 관리하고 제공해준다.
- 컨테이너가 관리하는 엔티티 매니저를 주입한다.
  `@PersistenceUnit`
- 엔티티 매니저 팩토리를 주입한다.
  `@Service`
- 스프링 빈으로 등록한다.
  `@Transactional`
- 트랜잭션을 적용한다. 메서드를 호출할 때 트랜잭션을 시작하고 메서드를 종료할 때 트랜잭션을 커밋한다.
- 예외가 발생하면 트랜잭션을 롤백한다(`RuntimeException`과 그 자식들인`Unchecked Exception`만 롤백한다.)
- 테스트에 적용시 테스트를 실행할 때마다 트랜잭션을 시작하고, 테스트가 끝나면 트랜잭션을 강제로 롤백한다.
  `@Autowired`
- 스프링 컨테이너가 적절한 스프링 빈을 주입한다.

## JPA 메서드
### `save()`
```java
public void save(Item item){
	if (item.getId() == null){
		em.persist(item);
	} else {
		em.merge(item);
	}
}
```
- 저장 : 식별자 값이 없으면(null) 새로운 엔티티로 판단해서 `persist()`로 영속화한다.
> save() 호출시 식별자 값을 할당하여 persist()를 호출한다.
- 수정(병합) : 식별자 값이 있으면 이미 한번 영속화되었던 엔티티로 판단해서 `merge()`로 수정(병합)한다. 변경된 데이터를 저장한다.
> merge(수정, 병합)은 준영속 상태의 엔티티를 수정할 때 사용한다.
> 영속화된 엔티티는 변경 감지 기능이 동작해서 트랜잭션을  커밋할 때 자동으로 수정되므로 별도의 수정 메서드를 호출할 필요가 없다.

### 도메인 모델 패턴
- 비즈니스 로직 대부분이 엔티티에 존재
- 서비스 계층은 단순히 엔티티에 필요한 요청 위임
### 트랜잭션 스크립트 패턴
- 서비스 계층에서 대부분의 비즈니스 로직 처리

## 준영속 엔티티 수정
1. `변경 감지 기능` 사용
- 영속성 컨텍스트에서 엔티티 다시 조회 후에 데이터 수정
2. `병합(merge)` 사용
- 파라미터로 넘긴 준영속 엔티티의 식별자 값으로 엔티티를 조회한 후 영속 엔티티의 값을 준영속 엔티티의 값으로 채워넣는다.
> 변경 감지 기능을 사용하면 원하는 속성만 선택해서 변경할 수 있지만, 병합을 사용하면 모든 속성이 변경된다.


