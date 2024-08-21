# 7장. 고급 매핑
- 상속 관계 매핑
- `@MappedSuperClass`
- 복합키와 식별관계 매핑
- 조인 테이블
- 엔티티 하나에 여러 테이블 매핑

# 7.1 상속 관계 매핑
- 슈퍼 타입 서브 타입 관계 구현하는 방법
1. **각각의 테이블로 변환** : 각각을 모두 테이블로 만들고 조회할 때 조인 사용 (조인 전략)
2. **통합 테이블로 변환** : 테이블을 하나만 사용해서 통합 (단일 테이블 전략)
3. **서브타입 테이블로 변환** : 서브 타입마다 하나의 테이블 (구현 클래스마다 테이블 전략)

## 조인 전략
- 엔티티 각각을 모두 테이블로 만들고 자식 테이블이 부모 테이블의 기본키를 받아서 기본키 + 외래키로 사용하는 전략
- 테이블은 타입의 개념이 없으므로 타입을 구분하는 컬럼을 추가해야 한다.
- 매핑 정보
    - `@Inheritance(strategy = InheritanceType.JOINED)` : 조인 전략
    - `@DiscriminatorColumn(name = "DTYPE")` : 구분 컬럼 지정
    - `@DiscriminatorValue("A")` : 구분 컬럼 값
    - `@PrimaryKeyJoinColumn` : 자식 테이블의 기본 키 컬럼명 변경
```java
@Entity
@Inheritance(strategy = InheritanceType.JOINED)
@DiscriminatorColumn(name = "DTYPE")
public abstract class Item {

    @Id @GeneratedValue
    @Column(name = "ITEM_ID")
    private Long id;

    private String name;        //이름
    private int price;          //가격
    private int stockQuantity;  //재고수량

    @ManyToMany(mappedBy = "items")
    private List<Category> categories = new ArrayList<Category>();

    //Getter, Setter
    public Long getId() {
        return id;
    }

    public void setId(Long id) {
        this.id = id;
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public int getPrice() {
        return price;
    }

    public void setPrice(int price) {
        this.price = price;
    }

    public int getStockQuantity() {
        return stockQuantity;
    }

    public void setStockQuantity(int stockQuantity) {
        this.stockQuantity = stockQuantity;
    }

    public List<Category> getCategories() {
        return categories;
    }

    public void setCategories(List<Category> categories) {
        this.categories = categories;
    }

    @Override
    public String toString() {
        return "Item{" +
                "id=" + id +
                ", name='" + name + '\'' +
                ", price=" + price +
                '}';
    }
}
```

- Album
```java
@Entity
@DiscriminatorValue("A")
public class Album extends Item {

    private String artist;
    private String etc;

    public String getArtist() {
        return artist;
    }

    public void setArtist(String artist) {
        this.artist = artist;
    }

    public String getEtc() {
        return etc;
    }

    public void setEtc(String etc) {
        this.etc = etc;
    }

    @Override
    public String toString() {
        return "Album{" +
                "artist='" + artist + '\'' +
                ", etc='" + etc + '\'' +
                '}';
    }
}
```

```java
@Entity
@DiscriminatorValue("B")
public class Book extends Item {

    private String author;
    private String isbn;


    public String getAuthor() {
        return author;
    }

    public void setAuthor(String author) {
        this.author = author;
    }

    public String getIsbn() {
        return isbn;
    }

    public void setIsbn(String isbn) {
        this.isbn = isbn;
    }

    @Override
    public String toString() {
        return "Book{}";
    }
}
```

### 장점
- 테이블이 정규화된다.
- 외래 키 참조 무결성 제약조건을 활용할 수 있다.
- 저장공간을 효율적으로 사용한다.
### 단점
- 조회할 때 조인이 많이 사용되므로 성능이 저하될 수 있다.
- 조회 쿼리가 복잡하다.
- 데이터 등록시 INSERT SQL을 두번 실행한다. (부모 + 자식)

## 단일 테이블 전략
- 테이블을 하나만 사용한다.
- 구분 컬럼으로 어떤 자식 데이터가 저장되었는지 구분한다.
- 조인을 사용하지 않으므로 일반적으로 가장 빠르다.
- 매핑 정보
    - `@Inheritance(strategy = InheritanceType.SINGLE_TABLE)` : 단일 테이블 전략
    - 구분 컬럼 필수 사용(`@DiscriminatorColumn`, `@DiscriminatorValue("A")`)
>`@DiscriminatorValue는 기본값으로 엔티티 이름 사용
```java
@Entity
@Inheritance(strategy = InheritanceType.SINGLE_TABLE)
@DiscriminatorColumn(name = "DTYPE")
public abstract class Item {

    @Id @GeneratedValue
    @Column(name = "ITEM_ID")
    private Long id;

    private String name;        //이름
    private int price;          //가격
    private int stockQuantity;  //재고수량

    @ManyToMany(mappedBy = "items")
    private List<Category> categories = new ArrayList<Category>();

    //Getter, Setter
    public Long getId() {
        return id;
    }

    public void setId(Long id) {
        this.id = id;
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public int getPrice() {
        return price;
    }

    public void setPrice(int price) {
        this.price = price;
    }

    public int getStockQuantity() {
        return stockQuantity;
    }

    public void setStockQuantity(int stockQuantity) {
        this.stockQuantity = stockQuantity;
    }

    public List<Category> getCategories() {
        return categories;
    }

    public void setCategories(List<Category> categories) {
        this.categories = categories;
    }

    @Override
    public String toString() {
        return "Item{" +
                "id=" + id +
                ", name='" + name + '\'' +
                ", price=" + price +
                '}';
    }
}
```

```java
@Entity
@DiscriminatorValue("A")
public class Album extends Item {

    private String artist;
    private String etc;

    public String getArtist() {
        return artist;
    }

    public void setArtist(String artist) {
        this.artist = artist;
    }

    public String getEtc() {
        return etc;
    }

    public void setEtc(String etc) {
        this.etc = etc;
    }

    @Override
    public String toString() {
        return "Album{" +
                "artist='" + artist + '\'' +
                ", etc='" + etc + '\'' +
                '}';
    }
}
```

### 장점
- 조인이 필요 없으므로 일반적으로 조회 성능이 빠르다.
- 조회 쿼리가 단순하다.
### 단점
- 자식 엔티티가 매핑한 컬럼은 모두 `null을 허용`해야 한다.
- 단일 테이블에 모든 것을 저장하므로 테이블이 커질 수 있다.
    - 상황에 따라 오히려 조회 성능이 느려질 수 있다.

## 구현 클래스마다 테이블 전략 (추천 X)
- 자식 엔티티마다 테이블을 만든다.
- 자식 테이블 각각에 필요한 컬럼이 모두 있다.
    - 구분 컬럼을 사용하지 않는다.
```java
@Entity
@Inheritance(strategy = InheritanceType.TABLE_PER_CLASS)
public abstract class Item {

    @Id @GeneratedValue
    @Column(name = "ITEM_ID")
    private Long id;

    private String name;        //이름
    private int price;          //가격
    private int stockQuantity;  //재고수량

    @ManyToMany(mappedBy = "items")
    private List<Category> categories = new ArrayList<Category>();

    //Getter, Setter
    public Long getId() {
        return id;
    }

    public void setId(Long id) {
        this.id = id;
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public int getPrice() {
        return price;
    }

    public void setPrice(int price) {
        this.price = price;
    }

    public int getStockQuantity() {
        return stockQuantity;
    }

    public void setStockQuantity(int stockQuantity) {
        this.stockQuantity = stockQuantity;
    }

    public List<Category> getCategories() {
        return categories;
    }

    public void setCategories(List<Category> categories) {
        this.categories = categories;
    }

    @Override
    public String toString() {
        return "Item{" +
                "id=" + id +
                ", name='" + name + '\'' +
                ", price=" + price +
                '}';
    }
}
```

```java
@Entity
public class Album extends Item {

    private String artist;
    private String etc;

    public String getArtist() {
        return artist;
    }

    public void setArtist(String artist) {
        this.artist = artist;
    }

    public String getEtc() {
        return etc;
    }

    public void setEtc(String etc) {
        this.etc = etc;
    }

    @Override
    public String toString() {
        return "Album{" +
                "artist='" + artist + '\'' +
                ", etc='" + etc + '\'' +
                '}';
    }
}
```

### 장점
- 서브 타입을 구분해서 처리할 때 효과적이다.
- not null 제약조건을 사용할 수 있다.
### 단점
- 여러 자식 테이블을 함께 조회할 때 성능이 느리다(UNION 사용)
- 자식 테이블을 통합해서 쿼리하기 어렵다.

# 7.2 `@MappedSuperclass`
- 부모 클래스는 테이블과 매핑하지 않고 **부모 클래스를 상속받는 자식 클래스에게 매핑 정보만 제공**
- 추상 클래스와 비슷하며 실제 테이블과는 매핑되지 않는다.
    - 단순히 매핑 정보를 상속할 목적으로만 사용된다.
    - 엔티티가 아니므로 `em.find()`나 `JPQL`에서 사용할 수 없다.
    - 클래스를 직접 생성해서 사용할 일이 거의 없으므로 `추상 클래스`를 만드는 것을 권장한다.

### 공통 속성 상속
- 테이블은 그대로 두고 객체 모델의 `id`, `name` 두 공통 속성을 부모 클래스로 모으고 객체 상속 관계 만들기
- 부모 클래스
    - 공통 매핑 정보를 정의한다.
```java
@MappedSuperclass
public class BaseEntity {

    private Date createdDate;       //등록일
    private Date lastModifiedDate;  //수정일

    //Getter, Setter
    public Date getCreatedDate() {
        return createdDate;
    }

    public void setCreatedDate(Date createdDate) {
        this.createdDate = createdDate;
    }

    public Date getLastModifiedDate() {
        return lastModifiedDate;
    }

    public void setLastModifiedDate(Date lastModifiedDate) {
        this.lastModifiedDate = lastModifiedDate;
    }
}
```
- 자식 클래스
    - 상속을 통해 부모 클래스의 매핑 정보를 물려받는다.
    - `@AttributeOverride`, `@AttributeOverrides`  : 부모로부터 물려받은 `매핑정보` 재정의
    - `@AssociationOverride`, `@AssociationOverrides` : `연관관계` 재정의
```java
@Entity
@AttributeOverride(name = "id", column = @Column(name="MEMBER_ID"))
public class Member extends BaseEntity {

    @Id @GeneratedValue
    @Column(name = "MEMBER_ID")
    private Long id;

    private String name;

    private String city;
    private String street;
    private String zipcode;

    @OneToMany(mappedBy = "member")
    private List<Order> orders = new ArrayList<Order>();

    //Getter, Setter

    public Long getId() {
        return id;
    }

    public void setId(Long id) {
        this.id = id;
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public String getCity() {
        return city;
    }

    public void setCity(String city) {
        this.city = city;
    }

    public String getStreet() {
        return street;
    }

    public void setStreet(String street) {
        this.street = street;
    }

    public String getZipcode() {
        return zipcode;
    }

    public void setZipcode(String zipcode) {
        this.zipcode = zipcode;
    }

    public List<Order> getOrders() {
        return orders;
    }

    public void setOrders(List<Order> orders) {
        this.orders = orders;
    }
}
```

## 7.3 복합키와 식별 관계 매핑
### 식별 관계 vs 비식별 관계
- 식별 관계: 부모 테이블의 기본키를 내려받아서 **자식 테이블의 기본키+외래키**로 사용하는 관계
- 비식별 관계: 부모 테이블의 기본키를 받아서 **자식 테이블의 외래키**로만 사용하는 관계
    - 필수적 비식별 관계: 외래키에 `null을 허용하지 않는다`. **연관관계 필수**
    - 선택적 비식별 관계: 외래키에 `null을 허용`한다. **연관관게 선택**
> 비식별 관계를 주로 사용하며,  **꼭 필요한 곳에만 식별 관계 사용하기**

### 복합키: 비식별관계 매핑
- JPA에서 식별자를 둘 이상 사용하려면 **별도의 `식별자 클래스`를 만들어야 한다.**
    - 영속성 컨텍스트에 엔티티를 보관할때 `엔티티의 식별자`를 키로 사용한다.
    - `equals`와 `hashCode`를 사용해서 동등성을 비교하므로 해당 식별자 클래스에 꼭 구현해야한다.
        - 기본은 동일성 비교(`\==`)이기 때문이다.
        - 모든 필드를 사용하여 식별자 클래스의 `equals` 와 `hashCode` 를 구현한다.
> 복합키에는 @GeneratedValue를 사용할 수 없다.
> 복합키를 구성하는 여러 컬럼 중 하나에도 사용할 수 없다.

### 1. @IdClass
- 관계형 데이터베이스에 가까운 방법

예시
- **비식별 관계**, `PARENT`는 복합 기본키를 사용
- Parent
```java
@Entity
@IdClass(ParentId.class) // 식별자 클래스 명시
public class Parent {

	@Id
	@Column(name = "PARENT_ID1)
	private String id1; // ParentId.id1과 연결

	@Id
	@Column(name = "PARENT_ID2)
	private String id2; // ParentId.id2과 연결
}
```
- 식별자 클래스
```java
public class ParentId implements Serializable {

	private String id1; // Parent.id1
	private String id2; // Parent.id2

	public ParentId(){}

	public ParentId(String id1, String id2){
		this.id1 = id1;
		this.id2 = id2;
	}

	@Override
	public boolean equals(Object o){...}

	@Overridc
	public int hashCode() {...}
}
```

#### @IdClass 식별자 클래스 조건
- `식별자 클래스의 속성명`과 `엔티티에서 사용하는 식별자의 속성명`이 같아야 한다.
- `Serializable` 인터페이스를 구현해야 한다.
- `equals`. `hashCode를` 구현해야 한다.
- `기본 생성자`가 있어야 한다.
- 식별자 클래스는 `public` 이어야 한다.

#### @IdClass 복합키 가진 엔티티 저장하기
```java
Parent parent = new Parent();
parent.setId1("myId1");
parent.setId2("myId2");
parent.setName("parentName");
em.persist(parent);
```
> 복합키 클래스 자동 생성

#### @IdClass 복합키 가진 엔티티 조회하기
```java
ParentId parentId = new ParentID("myId1", "myId2");
Parent parent = em.find(Parent.class, parentId);
```
> 식별자 클래스로 엔티티 조회

- 자식 클래스
```java
@Entity
public class Child{
	
	@Id
	private String id;

	@ManyToOne
	@JoinColumns({
		@JoinColumn(name = "PARENT_ID1",
			referencedColumnName = "PARENT_ID1"),
		@JoinColumn(name = "PARENT_ID2",
			referencedColumnName = "PARENT_ID2")
	})
	private Parent parent;
}
```
> JoinColumn의 name 속성과 referencedColumnName 속성의 값이 같으면 생략해도 된다.

### 2. @EmbeddedId
- 객체지향에 가까운 방법
- Parent
    - 식별자 클래스에 기본키를 직접 매핑한다.
```java
@Entity
public class Parent {

	@EmbeddedId
	private ParentId id;
}
```

- 식별자 클래스
```java
@Embeddable
public class ParentId implements Serializable {

	@Column(name = "PARENT_ID1")
	private String id1;
	@Column(name = "PARENT_ID2")
	private String id2;
}
```

#### @EmbeddedId 식별자 클래스 조건
- `@Embeddable` 어노테이션을 붙여주어야 한다.
- `Serializable` 인터페이스를 구현해야 한다.
- `equals`,`hashCode를` 구현해야 한다.
- `기본 생성자`가 있어야 한다.
- 식별자 클래스는 `public` 이어야 한다.

#### @IdClass 복합키 가진 엔티티 저장하기
```java
Parent parent = new Parent();
ParentId parentId = new ParentId("myId1", "myId2");
parent.setId(parentId);
parent.setName("parentName");
em.persist(parent);
```
> 식별자 클래스 parentId를 직접 생성해서 사용한다.

#### @IdClass 복합키 가진 엔티티 조회하기
```java
ParentId parentId = new ParentID("myId1", "myId2");
Parent parent = em.find(Parent.class, parentId);
```
> 식별자 클래스로 엔티티 조회

### @IdClass vs @EmbeddedId
- `@EmbeddedId` 가 `@IdClass`에 비해 더 객체지향적이고 중복이 없어 좋아보이지만, 특정 상황에 `JPQL`이 조금 더 길어질 수 있다.
```java
em.createQuery("select p.id.id1, p.id.id2 from Parent p") // @EmbeddedId
em.createQuery("select p.id1, p.id2 from Parent p"); //@IdClass
```

### 복합키: 식별관계 매핑
- 부모, 자식, 손자까지 기본키를 전달하는 식별 관계이다.

#### @IdClass와 식별 관계
- Parent
```java
@Entity
public class Parent {

	@Id
	@Column(name = "PARENT_ID")
	private String id; 
}
```

```java
@Entity
@IdClass(ChildId.class) // 식별자 클래스 명시
public class Child {

	@Id
	@ManyToOne
	@JoinColumn(name = "PARENT_ID1")
	private Parent parent; 

	@Id
	@Column(name = "CHILD_ID)
	private String childId; 
```
- 식별자 클래스
```java
public class ChildId implements Serializable {

	private String parent; // Child.parent 매핑
	private String childId; // Child.childId 매핑

	@Override
	public boolean equals(Object o){...}

	@Override
	public int hashCode() {...}
}
```

- 자식 클래스
```java
@Entity
@IdClass(GrandChildId.class) // 식별자 클래스 명시
public class GrandChild {
	
	@Id
	@ManyToOne
	@JoinColumns({
		@JoinColumn(name = "PARENT_ID"),
		@JoinColumn(name = "CHILD_ID")
	})
	private Child child;

	@Id @Column(name = "GRANDCHILD_ID")
	private String id;
}
```

```java
public class GrandChildId implements Serializable {

	private ChildId child; // GrandChild.child 매핑
	private String id; // GrandChild.id 매핑

	@Override
	public boolean equals(Object o){...}

	@Override
	public int hashCode() {...}
}
```

### @EmbeddedId와 식별 관계
- 식별관계 구성시 `@MapsId`를 사용해야 한다.
    - `@MapsId` : 외래키와 매핑한 연관관계를 기본키에도 매핑한다.
        - 식별자 클래스의 기본 키 필드를 지정하면 된다.
- Parent
```java
@Entity
public class Parent {

	@Id
	@Column(name = "PARENT_ID")
	private String id; 
}
```

```java
@Entity
public class Child {

	@EmbeddedId
	private ChildId id; 

	@MapsId("parentId") // ChildId.parentId 매핑
	@ManyToOne
	@JoinColumn(name = "PARENT_ID")
	private Parent parent; 

```
> 상속받은키 컬럼 만들어서 명시

- 식별자 클래스
```java
@Embeddable
public class ChildId implements Serializable {

	private String parentId; // @MapsId("parentId")로 매핑

	@Column(name = "CHILD_ID")
	private String id; 
	
	@Override
	public boolean equals(Object o){...}

	@Override
	public int hashCode() {...}
}
```

- 자식 클래스
```java
@Entity
public class GrandChild {

	@EmbeddedId
	private GrandChildId id;
	
	@MapsId("childId") // GrandChildId.childId 매핑
	@ManyToOne
	@JoinColumns({
		@JoinColumn(name = "PARENT_ID"),
		@JoinColumn(name = "CHILD_ID")
	})
	private Child child;

}
```

```java
@Embeddable
public class GrandChildId implements Serializable {

	private ChildId child; // @MapsId("childId")로 매핑

	@Column(name = "GRANDCHILD_ID")
	private String id; 

	@Override
	public boolean equals(Object o){...}

	@Override
	public int hashCode() {...}
}
```

### 비식별 관계로 변경하기
```java
@Entity
public class Parent {

	@Id @GeneratedValue
	@Column(name = "PARENT_ID")
	private Long id; 
	private String name;
}
```

```java
@Entity
public class Child {

	@Id @GeneratedValue
	@Column(name = "CHILD_ID")
	private Long id;

	@ManyToOne
	@JoinColumn(name = "PARENT_ID")
	private Parent parent; 

```

```java
@Entity
public class GrandChild {

	@Id @GeneratedValue
	@Column(name = "GRANDCHILD_ID")
	private Long id;

	@ManyToOne
	@JoinColumn(name = "CHILD_ID")
	private Child child; 

```

> 매핑도 쉽고 코드도 단순하다.

### 일대일 식별관계
- 자식 테이블의 기본키 값으로 부모 테이블의 기본키 값만 사용한다.
    - 부모 테이블의 기본키가 복합키가 아니면 자식 테이블의 기본키는 복합키로 구성하지 않아도 된다.
```java
@Entity
public class Board {

	@Id @GeneratedValue
	@Column(name = "BOARD_ID")
	private Long id; 

	@OneToOne(mappedBy = "board")
	private BoardDetail boardDetail;
}
```

```java
@Entity
public class BoardDetail {

	@Id 
	private Long boardId;

	@MapsId //BoardDetail.boardId 매핑
	@OneToOne
	@JoinColumn(name = "BOARD_ID")
	private Board board; 
```
> 식별자가 단순히 컬럼 하나면 @MapsId를 사용하고 속성값은 비워두면 된다.

### 식별, 비식별 관계의 장단점
#### 데이터베이스 설계 관점에서 **식별 관계보다는 비식별 관계를 선호**한다.
- 식별 관계: 부모 테이블의 기본키를 자식 테이블로 전파하면서 **자식 테이블의 기본 키 컬럼이 점점 늘어난다.**
    - 조인할 때 SQL이 복잡해지고 기본키 인덱스가 불필요하게 커질 수 있다.
    - 테이블 구조가 유연하지 못하다.
- 식별 관계는 `2개 이상의 컬럼`을 합해서 복합 기본키를 만들어야하는 경우가 많다.
- 식별 관계를 사용할 때 비즈니스 의미가 있는 `자연키 컬럼을 기본키`를 조합하는 경우가 많다.
    - 비즈니스 요구사항은 시간이 지남에 따라 변하므로 식별 관계의 자연 키 컬럼들이 자식에 손자까지 전파되면 변경하기 힘들다.
- 비식별 관계의 기본키는 비즈니스와 전혀 관계없는 `대리키`를 주로 사용한다.

#### 객체 매핑 관점에서 **식별 관계보다는 비식별 관계를 선호**한다.
- JPA에서 복합 키는 별도의 복합 키 클래스를 만들어서 사용해야 한다.
- @GeneratedValue를 통해 대리키를 손쉽게 생성할 수 있다.

#### 식별 관계의 장점
- 기본키 인덱스를 활용하기 좋다.
- 조인 없이 하위 테이블만으로 검색을 완료할 수 있따.
#### 추천 방법
- 가능하면 **비식별 관계를 사용**하고 기본키를 **Long 타입의 대리키**를 사용하자.
    - 대리키는 비즈니스가 변경되어도 유연한 대처가 가능하다.
    - 식별자 컬럼이 하나여서 쉽게 매핑할 수 있다.
- 선택적 비식별 관계보다 **필수적 비식별 관계를 사용**하는 것이 좋다.
    - `선택적인 비식별 관계`는 NULL을 허용하므로 `외부 조인을 사용`한다.
    - `필수적 비식별 관계`는 NOT NULL로 항상 관계가 있음을 보장하므로 **내부 조인만 사용해도 된다.**
