# 6장. 다양한 연관관계 매핑
### 엔티티 연관관게 매핑
- 다중성
    - 다대일
    - 일대다
    - 일대일
    - 다대다 -> 실무에서 거의 안씀
- 단방향 vs 양방향
    - 객체 관계에서 한쪽만 참조하는지, 양쪽이 서로 참조하는지
- 연관관계 주인
    - 양방향 관계면 연관관계 주인(외래키 관리자) 정하기
    - 테이블은 외래키 하나로 연관관계를 관리하지만, 객체는 참조 2개로 연관관계를 관리한다.
    - JPA는 두 객체 연관관계 중 하나를 정해서 데이터베이스 외래 키를 관리한다
    - 보통 외래키를 가진 테이블과 매핑한 엔티티가 외래키를 관리하는 것이 효율적이다.
> 다대일,일대다에서 왼쪽을 연관관계의 주인으로 설정하고 설명한다.


## 6.1 **다**대일 (`@ManyToOne`)
- db 테이블의 일과 다 관계에서 외래키는 항상 다 쪽에 있다.
- 객체 양방향 관계에서 연관관계 주인은 항상 다 쪽에 있다.

### 다대일 단방향
- Member -> Team
- N : 1
```java
@Entity
public class Member {

	@Id @GeneratedValue
	@Column(name = "MEMBER_ID")
	private Long id;

	@ManyToOne
	@JoinColumn(name = "TEAM_ID")
	private Team team;
}
```
- `@JoinColumn(name = "TEAM_ID")` 을 이용해서 필드를 외래키와 매핑하여 관리한다.
    - 외래키를 가진 테이블에서 외래키를 관리하므로 편하다.

```java
@Entity
public class Team {

	@Id @GeneratedValue
	@Column(name="TEAM_ID")
	private Long id;

}
```

### 다대일 양방향
- **양방향은 외래키가 있는 쪽이 연관관계 주인이다.**
    - JPA는 외래키를 관리할때 연관관계 주인만 사용하고, 주인이 아닌 필드를 조회를 위한 JPQL이나 객체 그래프 탐색시 사용한다.
- **양방향 연관관게는 항상 서로를 참조해야한다.**
    - 연관관계 편의 메서드를 작성하는 것이 좋다.
    - 연관관계 편의 메서드를 양쪽에 작성할 경우 무한루프에 빠지므로 주의해야 한다.
```java
@Entity
public class Member {

	@Id @GeneratedValue
	@Column(name = "MEMBER_ID")
	private Long id;

	@ManyToOne
	@JoinColumn(name = "TEAM_ID")
	private Team team;

	public void setTeam(Team team){
		this.team = team;

		if (!team.getMembers().contains(this)){
			team.getMembers().add(this);
		}
	}
}
```

```java
@Entity
public class Team {

	@Id @GeneratedValue
	@Column(name="TEAM_ID")
	private Long id;

	@OneToMany(mappedBy = "team")
	private List<Member> members = new ArrayList<Member>();

	public void addMember(Member member){
		this.members.add(member);
		
		if (member.getTEam() != this){
			member.setTeam(this);
		}
	}
}
```

## 6.2 **일**대다 (`@OneToMany`)
- 엔티티를 하나 이상 참조할 수 있으므로 java collection을 사용한다.
- 연관관계 주인 : 일

### 일대다 단방향
- 외래키는 항상 다쪽 테이블에 있다.
- 연관관계 주인은 일쪽 테이블의 엔티티 필드에 있다.
- 반대편 테이블의 외래키를 관리하는 특이한 모습이 나타난다.

- `Team` -> `Member`
```java
@Entity
public class Team {

	@Id @GeneratedValue
	@Column(name="TEAM_ID")
	private Long id;

	@OneToMany
	@JoinColumn(name = "TEAM_ID")
	private List<Member> members = new ArrayList<Member>();

}
```
- `@JoinColumn(name = "TEAM_ID")` 으로 외래키 관리자(연관관계 주인)을 명시한다.

```java
@Entity
public class Member {

	@Id @GeneratedValue
	@Column(name = "MEMBER_ID")
	private Long id;
}
```

- 일대다 단방향 관계 매핑시에는 `@JoinColumn`을 명시해야한다.
    - 그렇지 않으면 연결 테이블을 중간에 두고 연관관게를 관리하는 조인 테이블 전략을 기본으로 사용하여 매핑한다.

#### 일대다 단방향 매핑의 단점
- 매핑한 객체가 관리하는 외래키가 다른 테이블에 있으므로 추가 UPDATE SQL문이 실행된다.
    - Member와 Team 저장 INSERT SQL문이 실행될 때, Member는 Team을 알지 못하므로 외래키는 null로 저장된다 (단방향)
    - Team 저장시 연관관계를 설정하기 위해 Member 테이블 UPDATE SQL이 새로 추가된 연관관계 수만큼 실행된다.

#### 일대다 단방향 매핑보다는 다대일 양방향 매핑을 사용하자
- 엔티티를 매핑한 테이블이 아닌 다른 테이블에서 외래키를 관리해야한다.
    - 성능도 좋지 않고, 관리도 부담스럽다.
- 일대다 단뱡향 매핑 대신, 다대일 양방향 매핑을 사용하자.
    - 다대일 양방향 매핑은 관리해야 하는 외래키가 본인 테이블에 있다.

### 일대다 양방향
- 관계형 db에서 다쪽에 항상 외래키가 있으므로 @ManyToOne은 mappedBy 속성이 없다.
- 일대다 단방향 매핑 반대편에 같은 외래키를 사용하는 다대일 단방향 매핑을 읽기 전용으로 추가하면 구현할 수 있다.

- Team -> Member
- Team <- Member (읽기 전용)
```java
@Entity
public class Team {

	@Id @GeneratedValue
	@Column(name="TEAM_ID")
	private Long id;

	@OneToMany
	@JoinColumn(name = "TEAM_ID")
	private List<Member> members = new ArrayList<Member>();

}
```

```java
@Entity
public class Member {

	@Id @GeneratedValue
	@Column(name = "MEMBER_ID")
	private Long id;

	@ManyToOne
	@JoinColumn(name = "TEAM_ID", insertable = false, updatable = false) // 읽기 전용
	private Team team;
}
```
- 둘 다 같은 외래키를 관리하는 것은 문제가 될 수 있으므로, 다대일 쪽은 읽기만 가능하게 했다.
- 일대다 단방향 매핑의 단점을 그대로 가진다.
    - 될 수 있으면 다대일 양방향 매핑을 사용하자.

## 6.3 일대일 (`@OneToOne`)
- 양쪽이 서로 하나의 관계만 가진다.
- 일대일 관계는 그 반대도 일대일 관계다.
- 테이블 관계에서 일대다, 다대일은 항상 다쪽이 외래키를 가진다.
- 일대일 관계는 주 테이블이나 대사 테이블 둘 중 어느 곳이나 외래키를 가질 수 있다.

#### 주 테이블에 외래키
- 주 객체가 대상 객체(참조 대상)를 참조하는 것처럼, 주 테이블에 외래키를 두고 외래키를 통해 대상 테이블을 참조한다.
- 외래키를 객체 참조와 비슷하게 사용할 수 있으므로 객체지향 개발자들이 선호한다.
- 장점 : 주 테이블이 외래키를 가지고 있으므로 주 테이블만 확인해도 대상 테이블과 연관관계가 있는지 알 수 있다.

#### 대상 테이블에 외래키
- 전통적인 데이터베이스 개발자들은 보통 대상 테이블에 외래키를 두는 것을 선호한다.
- 테이블 관계를 일대일에서 일대다로 변경할 때 테이블 구조를 그대로 유지할 수 있다.
### 1. 주 테이블에 외래키
#### 일대일 단방향
- Member -> Locker
- 주 테이블인 `Member` 에 외래키 존재

```java
@Entity
public class Member {

	@Id @GeneratedValue
	@Column(name = "MEMBER_ID")
	private Long id;

	@OneToOne
	@JoinColumn(name = "LOCKER_ID") 
	private Locker locker;
}
```

```java
@Entity
public class Locker {

	@Id @GeneratedValue
	@Column(name = "MEMBER_ID")
	private Long id;
	
}
```

#### 일대일 양방향
- 주 테이블인 Member에 외래키 존재
```java
@Entity
public class Member {

	@Id @GeneratedValue
	@Column(name = "MEMBER_ID")
	private Long id;

	@OneToOne
	@JoinColumn(name = "LOCKER_ID") 
	private Locker locker;
}
```

```java
@Entity
public class Locker {

	@Id @GeneratedValue
	@Column(name = "MEMBER_ID")
	private Long id;

	@OneToOne(mappedBy = "locker")
	private Member member;
}
```

### 2. 대상 테이블에 외래키
#### 단방향
- 일대일 관계중 대상 테이블에 외래키가 있는 단방향은 JPA에서 지원하지 않는다.
- Member->Locker로 참조할 때 Member는 무조건 연관관계의 주인이기 때문이다.

#### 양방향
- Member -> Locker : 읽기 전용
- Member <- Locker : 연관관계 주인, 외래키 존재

```java
@Entity
public class Member {

	@Id @GeneratedValue
	@Column(name = "MEMBER_ID")
	private Long id;

	@OneToOne(mappedBy = "member")
	private Locker locker;
}
```

```java
@Entity
public class Locker {

	@Id @GeneratedValue
	@Column(name = "MEMBER_ID")
	private Long id;

	@OneToOne(name = "MEMBER_ID")
	private Member member;
}
```

> 프록시를 사용할 때 외래키를 직접 관리하지 않는 일대일 관계는 지연 로딩으로 설정해도 즉시 로딩된다.


## 6.4 다대다 (@ManyToMany)
- 관계형 db는 정규화된 테이블 2개로 다대다 관계를 표현할 수 없다.
    - 보통 다대다 관계를 일대다, 다대일 관계로 풀어내는 연결 테이블을 사용한다.
- 객체는 테이블과 다르게 객체 2개로 다대다 관계를 만들 수 있다.
    - @ManyToMany 로 매핑이 가능하다.
    - 키 이외의 필드는 추가할 수 없다. (추가하고 싶다면 새로운 엔티티를 생성해야 한다)
### 다대다 단방향
```java
@Entity
public class Member {

	@Id @GeneratedValue
	@Column(name = "MEMBER_ID")
	private Long id;

	@ManyToMany
	@JoinTable(name="MEMBER_PRODUCT",
		joinColumns= @JoinColumn(name = "MEMBER_ID"),
		inverseJoinColumns = @JoinColumn(name = "PRODUCT_ID")
	private List<Product> products = new ArrayList<Product>();
}
```

```java
@Entity
public class Product {
	@Id @GeneratedValue
	@Column(name = "PRODUCT_ID")
	private Long id;

}
```
- `@ManyToMany`와 `@JoinTable`을 이용해 연결 테이블을 바로 매핑한다.
- 회원 정보 저장시 연결 테이블에도 값이 저장된다.

#### @JoinTable
- `name` : 연결 테이블 지정
- `joinColumns`: 현재 방향인 조인 컬럼 정보 지정
- `inverseJoinColumns` : 반대 방향인 상품과 매핑할 조인 컬럼 정보 지정

### 다대다 양방향
- `mappedBy` 로 역방향에 설정한다.
```java
@Entity
public class Product {
	@Id @GeneratedValue
	@Column(name = "PRODUCT_ID")
	private Long id;

	@ManyToMany(mappedBy = "products")
	private List<Member> members;
}
```
- 연관관계 편의 메서드를 추가하여 양방향 연관관계를 관리하는 것이 편하다.
```java
@Entity
public class Member {

	@Id @GeneratedValue
	@Column(name = "MEMBER_ID")
	private Long id;

	@ManyToMany
	@JoinTable(name="MEMBER_PRODUCT",
		joinColumns= @JoinColumn(name = "MEMBER_ID"),
		inverseJoinColumns = @JoinColumn(name = "PRODUCT_ID")
	private List<Product> products = new ArrayList<Product>();

	public void addProduct(Product product){
		products.add(product);
		product.getMembers().add(this);
	}
}
```

#### 다대다 매핑의 한계
- 다대다 매핑을 실무에서 사용하는 것은 한계가 있다.
- 연결 테이블에 키 이외의 컬럼을 추가하고 싶을 경우 `@ManyToMany`를 사용할 수 없다.
    - 그 대신 연결 테이블을 매핑하는 연결 엔티티를 만들고 추가한 컬럼들을 매핑해야 한다.
### 연결 엔티티 사용
- `Member` <-> `MemberProduct` -> `Product`
```java
@Entity
public class Member {

	@Id @GeneratedValue
	@Column(name = "MEMBER_ID")
	private String id;

	@OneToMany(mappedBy = "member")
	private List<MemberProduct> memberProducts;
}
```

```java
@Entity
public class Product {
	@Id @GeneratedValue
	@Column(name = "PRODUCT_ID")
	private String id;
}
```

- `MemberProduct` : 연결 테이블
```java
@Entity
@IdClass(MemberProductId.class)
public class MemberProduct {

	@Id
	@ManyToOne
	@JoinColumn(name = "MEMBER_ID")
	private Member member;

	@Id
	@ManyToOne
	@JoinColumn(name = "PRODUCT_ID")
	private Product product;
}
```

```java
public class MemberProductId implements Serializable {

	private String member;
	private String product;
}
```

#### 복합 기본키
- 별도의 식별자 클래스르 만들어야 한다.
- Serializable을 구현해야 한다.
- equals와 hashCode 메서드를 구현해야 한다.
- 기본 생성자가 있어야 한다.
- 식별자 클래스는 public 이어야 한다.
- `@IdClass`를 사용하는 방법 외에 `@EmbeddedId`를 사용한다.
> 식별자 클래스로 엔티티를 조회한다.
#### 식별 관계
- 부모 테이블의 기본키를 받아서 자신의 기본키 + 외래키르 사용한다.

### 기본키 생성 전략
- 데이터베이스에서 자동으로 생성해주는 대리키를 `Long` 값으로 사용한다.
    - 간편하고 거의 영구히 쓸 수 있으며 비즈니스에 의존하지 않는다.
    - 복합키를 만들지 않아도 되므로 매핑을 완성할 수 있다.
```java
@Entity
public class Order {

	@Id @GeneratedValue
	@Column(name = "ORDER_ID")
	private Long id;

	@ManyToOne
	@JoinColumn(name = "MEMBER_ID")
	private Member member;

	@ManyToOne
	@JoinColumn(name = "PRODUCT_ID")
	private Product product;
}
```

```java
@Entity
public class Member {

	@Id @GeneratedValue
	@Column(name = "MEMBER_ID")
	private String id;

	@OneToMany(mappedBy = "member")
	private List<Order> orders = new ArrayList<Order>();
}
```

```java
@Entity
public class Product {

	@Id @Column(name = "PRODUCT_ID")
	private String id;
}
```

### 다대다 연관관계 정리
- 식별 관계 : 식별자를 기본키 + 외래키로 사용한다.
    - 부모 테이블의 기본키를 받아서 자식 테이블의 기본키 + 외래키로 사용
- 비식별 관게: 식별자를 외래키로만 사용하고 새로운 식별자를 추가한다.
    - 부모 테이블의 기본키를 외래키로만 사용
