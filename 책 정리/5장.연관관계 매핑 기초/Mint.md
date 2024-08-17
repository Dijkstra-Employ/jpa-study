# 5장. 연관관계 매핑 기초

- 객체의 참조와 테이블의 외래키를 매핑하자
	- 객체는 참조(주소)를 사용해서 관계를 맺고, 테이블은 외래키를 사용해서 관계를 맺는다.

- 방향
	- 단방향
	- 양방향
> 방향은 객체 관계에만 존재하고, 테이블 관계는 항상 양방향이다.

- 다중성
	- 다대일
	- 일대다
	- 일대일
	- 다대다

- 연관관계 주인
	- 객체를 양방향 연관관계로 맺으면 연관관계의 주인을 정해야한다.

## 5.1 단방향 연관관계
### 객체 연관관계 vs 테이블 연관관계
- **객체 연관관계**: **참조**를 통해 설정하며 언제나 `단방향`이다.
	- 객체간의 연관관계를 양방향으로 만들고 싶으면 반대쪽에도 필드를 추가해서 참조를 보관해야 한다.
	- 양방향은 서로 다른 단방향 관계 2개이다.
- **테이블 연관관계**: **외래 키** 하나로 `양방향`으로 조인한다.

### 객체 그래프 탐색
```java
Team findTeam = member1.getTeam();
```
- 객체는 참조를 사용해서 연관관계를 탐색할 수 있다.

### 객체 관계 매핑
```java
@ManyToOne
@JoinColumn(name="TEAM_ID")
private Team team;
```
- 객체 연관관계 : 회원 객체의 `Member.team` 필드 사용
- 테이블 연관관계: 회원 테이블의 `@JoinColumn(name="TEAM_ID")` 외래키 컬럼 사용

#### @JoinColumn
- 외래키 매핑시 사용한다.
- @JoinColumn 생략시 기본 전략(필드명 + `_` + 참조하는 테이블 컬럼명)으로 외래키를 사용한다.
	- ex) team_TEAM_ID

#### @ManyToOne
- 다대일 관계에서 사용한다.
- `optional` : false로 설정하면 연관된 엔티티가 항상 있어야 한다.
- `fetch`: 글로벌 페치 전략을 설정한다.
- `cascade` : 영속성 전이 기능을 사용한다.
- `targetEntity` : 연관된 엔티티 타입 정보를 설정, 이 기능은 거의 사용하지 않는다.

## 5.2 연관관계 설정
### 연관관계 엔티티 저장
```java

Team team1 = new Team("team1", "팀1");
em.persist(team1);

Member member1 = new Member("member1", "회원1");
member1.setTeam(team1); // 연관관계 설정
em.persist(member1);

```
> JPA에서 엔티티를 저장할 때 연관된 모든 엔티티는 영속 상태여야 한다.

### 연관관계 엔티티 조회
1. **객체 그래프 탐색**
```java
Member member1 = em.find(Member.class, "member1");
Team team = member1.getTeam(); // 객체 그래프 탐색
System.out.println(team.getName());
```

2. **객체지향 쿼리(`JPQL`) 사용**
- 연관된 테이블을 `JOIN`하여 연관된 테이블을 조회한다.
```java
String jpql = "select m from Member m join m.team t where t.name = :teamName";

List<Member> resultList = em.createQuery(jpql, Member.class)
	.setParameter("teamName", "팀1") // 파라미터 바인딩
	.getResultList();

for (Member member: resultList) {
	System.out.println(member.getUsername);
}
```
> JPQL은 객체(엔티티)를 대상으로 SQL보다 간결하다.

### 연관관계 엔티티 수정
```java
Team team2 = new Team("team2", "팀2");
em.persist(team2);

Member member = em.find(Member.class, "member1");
member.setTeam(team2); // 연관관계 설정
```
- 엔티티의 값만 변경해두면 트랜잭션을 커밋할 때 `flush`가 일어나면서 **변경 감지 기능이 작동**한다. 변경사항이 db에 저장된다.
	- 수정은 `em.update()`와 같은 메서드가 없다. JPA가 자동으로 처리한다.

### 연관관계 엔티티 삭제(연관관계 제거)
```java
Member member1 = em.find(Member.class, "member1");
member1.setTeam(null); // 연관관계 제거
```

#### 연관관계 제거시 기존의 연관관계를 먼저 제거해야 한다.
```java
member1.setTeam(null); // 연관관계 제거
em.remove(team); // 팀 삭제
```

## 5.3 양방향 연관관계
- 일대다 연관관계는 여러 건과 연관관계를 맺을 수 있으므로 컬렉션을 사용해야 한다.
- 데이터베이스 테이블 : 외래 키 하나로 양방향으로 조회할 수 있다.

### 양방향 연관관계 매핑
```java
@Entity
public class Member {

	@Id
	@Column(name = "MEMBER_ID")
	private String id;

	private String username;

	@ManyToOne
	@JoinColumn(name="TEAM_ID")
	private Team team;

	// 연관관계 설정
	public void setTeam(Team team){
		this.team = team;
	}
}
```
> 회원 엔티티는 변경한 부분이 없다.

```java
@Entity
public class Team {

	@Id
	@Column(name = "TEAM_ID")
	private String id;

	private String name;

	@OneToMany(mappedBy = "team")
	private List<Member> members = new ArrayList<Member>();
}
```
- mappedBy 속성을 사용하여 양방향 매핑에서 반대쪽 매핑의 필드 이름을 값으로 준다.

### 일대다 컬렉션 조회
```java

Team team = em.find(Team.class, "team1");
List<Member> members = team.getMembers();

for (Member member:members){
	System.out.println(member.getUsername());
}
```


## 5.4 연관관계 주인
- 테이블은 외래키 하나로 두 테이블의 연관관계를 관리한다.
- 엔티티는 참조를 사용하면 단방향으로 매핑된다.
	- 양방향일 경우 2개의 참조를 사용한다.
- 객체의 참조는 2개인데, 외래 키는 하나이다.
	- 둘 중 어떤 엔티티에서 외래키를 관리해야할까?
- 연관관게 주인 : 두 객체 연관관계 중 하나를 정해서 테이블의 외래키를 관리하는데, 이를 연관관계 주인이라고 한다.

### 연관관계 주인
- 연관관계의 주인만이 데이터베이스 연관관계와 매핑되고 외래 키를 관리할 수 있다.
	- 연관관계 주인 = **외래키 관리자**, 테이블에 외래키가 있는 곳
- 주인이 아닌 쪽(`mappedBy`)은 읽기만 가능하다.
> 데이터베이스 테이블의 다대일, 일대다 관계에서는 항상 다 쪽이 외래키를 가진다.
> 다 쪽인 @ManyToOne은 항상 연관관게의 주인이 되므로, mappedBy를 설정할 수 없다.

## 5.5 양방향 연관관계 저장
```java
Team team1 = new Team("team1", "팀1");
em.persist(team1);

Member member1 = new Member("member1", "회원1");
member1.setTeam(team1); // 연관관계 설정
em.persist(member1);
```
- 양방향 연관관계는 연관관계 주인이 외래키를 관리한다.
- 주인이 아닌 방향은 값을 설정하지 않아도 db에 외래키 값이 정상 입력된다.
> 주인이 아닌 곳에 입력된 값은 외래키에 영향을 주지 않는다.

## 5.6 양방향 연관관계 주의점
- 연관관계 주인에 값을 입력하지 않고, 연관관계 주인이 아닌 곳에 값을 입력하면 외래키가 제대로 설정되지 않는다 .(`null` 저장)
	- **연관관계 주인만이 외래 키의 값을 변경할 수 있다.**

### 객체 관계까지 고려한 양방향 연관관계
- 외래키는 처음부터 양방향으로 설정되지만, 객체 관계는 기본적으로 단방향이다.
	- 객체 관점에서 양방향 설정을 하기 위해 양쪽 방향에 모두 값을 입력(참조를 가지는)하는 것이 좋다.
```java
Team team1 = new Team("team1", "팀1");
em.persist(team1);

Member member1 = new Member("member1", "회원1");

// 양방향 연관관계 설정
member1.setTeam(team1); // 연관관계 주인
team1.getMembers().add(member1); // 주인 X, db 저장시 사용 X
em.persist(member1);
``` 

### 연관관게 편의 메서드
- 양방향 연관관계에서 양쪽 모두 관계를 맺어주는 것을 잊지 않기 위해 연관관계 설정 코드를 한 메서드에 모아두자.
	- 연관관계 편의 메서드 = 한번에 양방향 관계를 설정하는 메서드
```java
public void setTeam(Team team){
	this.team = team;
	this.getMembers().add(this);
}
```
- 연관관계 메서드 사용
```java
Team team1 = new Team("team1", "팀1");
em.persist(team1);

Member member1 = new Member("member1", "회원1");

// 양방향 연관관계 설정
member1.setTeam(team1); 
em.persist(member1);
``` 

#### 연관관계 재설정시 기존 연관관계는 꼭 제거하자.
```java
public void setTeam(Team team){

	// 기존 연관관계 제거
	if (this.team != null){
		this.team.getMembers().remove(this);
	}
	
	this.team = team;
	this.getMembers().add(this);
}
```
> db에는 외래키가 새롭게 설정되므로 문제는 없지만, 관계 변경 후 영속성 컨텍스트에 엔티티가 계속 살아있고, 이를 조회한다면
> 제거되지 않은 기존 관계(기존 team)로 인해 문제가 발생할 수 있다.

## 5.7 정리
- 연관관계가 하나인 단방향 매핑은 언제나 연관관계의 주인이다.
- 양방향은 연관관계 주인인 단방향에 연관관계 주인이 아닌 연관관계를 하나 추가한 것이다.
	- 양방향은 반대 방향으로 객체 그래프 탐색 기능이 추가된다.
	- 주인의 반대편은 `mappedBy`로 주인을 지정해야 한다. 단순히 객체 그래프 탐색만 할 수 있다.
	- 객체에서 양쪽 방향을 모두 관리해야 한다. (ex) 연관관계 편의 메서드)

#### 연관관계 주인 설정
- 비즈니스 중요도와 상관없이 단순히 외래 키 관리자 정도의 의미를 부여해야 한다.
- 외래 키의 위치와 관련해서 정해야지 비즈니스 중요도로 접근하면 안된다.
- 될 수 있으면 외래 키가 있는 곳(다 테이블)을 연관관계 주인으로 선택하자.

#### 양방향 무한루프
- 양방향 매핑시 무한 루프에 빠지지 않도록 주의해야한다.
- JSON 변환시 발생할 수 있으므로 주의하자.

#### 단방향
- 참조 방향을 고려해서 단방향으로 설정할지, 양방향으로 설정할지 고려하자.
- 일대다 단방향보다 다대일 양방향이 낫다.


