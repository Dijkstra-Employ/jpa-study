## 2주차

📌 **244~264**
**상속관계를 데이터베이스에서는 어떻게 매핑할까?**
1. 조인전략
2. 단일 테이블 전략
3. 구현 클래스마다 테이블 전략

앞선 상속관계 매핑에서는 부모 클래스와 자식 클래스 모두 데이터베이스 테이블에 매핑하였지만 부모클래스는 매핑하지 않고 **자식클래스만 매핑할 수 있는** `@MappedSuperClass`를 제공한다.
따라서 단순 상속 목적만 있다면 위 어노테이션을 사용한다. (예, 등록일자, 수정일자, 등록자, 수정자 등등)

**식별관계와 비식별관계**
**식별관계**: 부모테이블의 기본키가 자식테이블로 내려와 자식테이블에서 **기본키 + (부모)외래키** 로 사용하는 관계
**비식별관계**: (보통 사용하는 방식) 부모 테이블의 기본키를 받아 자식테이블의 **외래 키**로만 사용하는 관계
이 부분을 공부하며 어렵기도 어렵지만 어떤 상황에서 사용할까에 대한 의문이 많이 든다.

📌 **264~285**
**식별 관계의 장단점**
`장점` : 상위 테이블들의 기본 키 컬럼을 자식, 손자 테이블들이 가지고 있어 특정 상황에서 **조인 없이 하위 테이블만으로 검색을 완료**할 수 있다.
`단점` : 기본 키로 비즈니스적 의미가 있는 **자연 키 컬럼 조합**으로 설정하는 경우가 많은데, **비즈니스 요구사항은 시간에 지남에 따라 변하니 **이러한 조합으로 키를 생성하면 변경하기 힘들다.

**비식별 관계의 장점**
`장점` : 비즈니스와 전혀 관계없는 **대리 키**를 주로 사용한다.
`@GenerateValue` 같은 대리 키 생성하기 위한 편리한 방법 또한 제공한다.
**선택적 비식별관계**보다 **필수적 비식별 관계**를 사용하는 것이 좋다 (NULL 문제)

사실 그래서 **비식별** 쓰는게 좋다~~~~~~

**조인테이블**
1. **조인 컬럼 사용** : 조인 컬럼의 외래키 컬럼을 사용하여 테이블 관계 관리
   조인 컬럼을 사용한 두 테이블이 아주 가끔 관계를 맺는다면 외래 키 대부분이 **null로 지정된다는 단점**이 있다.

2. **조인 테이블 사용** : 별도의 테이블을 사용하여 연관관계 관리 (각 테이블의 **외래키만을 관리하는 테이블**을 따로 만든다)
   관리해야하는 테이블이 하나 더 늘어난다는 단점이 있다.

📌 **288 ~ 308**
특정 엔티티의 정보만 필요하지만 연관관계로 인해 연관된 엔티티도 함께 조회를 하는 것은 **비효율적**일 수 있다.
따라서 **엔티티가 실제로 사용될 때까지 데이터베이스 조회를 지연**하는 `지연로딩`을 제공하는데 이 지연로딩을 사용하기 위해서는 `프록시` 가 필요하다.

`프록시 객체`는 **초기화가 된 이후 프록시 객체를 통해 실제 엔티티에 접근**할 수 있다.
하지만 만약 **영속성 컨텍스트에 찾는 엔티티가 이미 있다면** 프록시가 아닌 **실제 엔티티**를 반환한다.

위에서도 말했듯 프록시 객체는 주로 연관된 엔티티를 지연 로딩할 때 사용하는데 , JPA는 연관된 엔티티의 `조회 시점`을 선택할 수 있게 한다.
1. `지연 로딩` : **연관된 엔티티를 실제 사용할 때 조회**
2. `즉시 로딩` : **엔티티를 조회할 때 연관된 엔티티도 함께 조회**

보통 모든 연관관계에 **지연 로딩을 사용하는 것을 권장**하고 있지만 대부분의 로직에서 연관된 엔티티를 같이 사용한다면 SQL 조인을 사용하여 한번에 조회하는 것이 효율적이므로 상황에 따라 적절히 `지연로딩`과 `즉시로딩`을 사용하면 좋을 것 같다.

📌 **309 ~ 328**
**JPA에서 엔티티를 저장할 때 연관된 모든 엔티티는 영속 상태**여야 한다.
따라서 연관된 엔티티 하나하나 모두 영속 상태로 만들어줘야하는 **반복 작업**을 해야했지만
`Casacade` 영속성 전이를 사용하면 **부모만 영속 상태로 만들면 연관된 자식까지 한 번에 영속 상태**로 만들 수 있다.

`고아객체`는 부모 엔티티와 연관관계가 끊어진 자식 엔티티를 **자동으로 삭제**하는 기능이다.

`임베디드 타입`은 사용자가 **새로운 값 타입을 직접 정의**해서 사용할 수 있는 타입이다.
- `@Embeddable`: 값 타입을 정의하는 곳에 표시
- `@Embedded`: 값 타입을 사용하는 곳에 표시
  만약 임베디드 타입의 **컬럼 명이 중복**된다면 `@AttributeOverride`를 사용하여 속성을 **재정의** 해주면 된다.

📌 **329 ~ 350**
```java
Address a = new Address("Old");
Address b = a;
b.setCity("New")
```

자바는 객체의 값을 대입하면 참조값을 전달한다.
따라서 b.setCity("New")의 의도는 b의 city값만 new로 변경하려했지만 공유 참조로 인해 부작용이 발생하여 a.city 값도 변경되어버린다.
여기서 문제는 복사하지 않고 원본의 참조 값을 직접 넘기는 행위를 막을 방법이 없다는 것이다.
따라서 근본적인 해결책으로 객체의 값을 수정하지 못하게 막으면 되는데 이를 위해 setter와 같은 수정자 메소드를 모두 삭제하는 것이다.
