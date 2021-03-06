# Spring Data JPA
---

- JDBC
- JPA (Hibernate)
- Spring Data JPA

---

# 관계형 데이터베이스와 자바
---

<details>
    <summary>펼치기</summary>

> 기본키나 외래키를 이용하여 데이터들을 식별하고 무결성을 관리

## 패러다임 불일치
---

> 객체를 릴레이션에 맵핑하려니 발생하는 문제들과 해결책

### 밀도(Granulartty) 문제

- 객체
    - 다양한 크기의 객체를 만들 수 있음
    - 커스텀한 타입 만들기 쉬움
- 릴레이션
    - 테이블
    - 기본 데이터 타입(User Define Type은 비추)

### 서브타입(SubType) 문제

- 객체
    - 상속 구조 만들기 쉬움
    - 다형성
- 릴레이션
    - 테이블 상속이라는게 없음
    - 상속 기능을 구현했다 하더라도 표준 기술이 아님
    - 다형적인 관계를 표현할 방법이 없음

### 식별성(Identity) 문제

- 객체
    - 레퍼런스 동일성(==)
    - 인스턴스 동일성(equals() 메서드)
- 릴레이션
    - 주키(Primary key)

### 관계(Association) 문제

- 객체
    - 객체 레퍼런스로 관계 표현
    - 근본적으로 "방향"이 존재
    - 다대다 관계를 가질 수 있음
- 릴레이션
    - 외래키(Foreign key)로 관계 표현
    - "방향"이라는 의미가 없음(Join으로 아무거나 묶음)
    - 태생적으로 다대다 관계를 못만들고, 조인 테이블 또는 링크 테이블을 사용해서 두개의 1대 다 관계로 풀어야함

### 데이터 네비게이션(Navigation)의 문제

- 객체
    - 레퍼런스를 이용해서 다른 객체로 이동 가능
    - 콜랙션을 순회할 수도 있음
- 릴레이션
    - 데이터는 요청을 적게 할 수록 성능이 좋음(그래서 Join을 사용)
    - 한번에 너무 많이 가져오는 것도 문제
    - lazy loading도 문제 (n + 1 select)
</details>


# JPA 프로젝트 세팅
---

<details>
    <summary>펼치기</summary>

- 프로젝트 환경
    - IntelliJ
    - JAVA 11
    - Gradle
    - Mysql
    - Docker

### 도커 세팅

> docker run -d -p 3316:3306 -e MYSQL_ROOT_PASSWORD=`<PASSWORD>` --name mysql mysql:latest --character-set-server=utf8mb4 --collation-server=utf8mb4_unicode_ci

### 의존성 추가

> implementation 'org.springframework.boot:spring-boot-starter-data-jpa'
> implementation group: 'mysql', name: 'mysql-connector-java', version: '8.0.24'

-> 의존성 추가하면 하이버네이트도 추가됨

### application.yml

```yaml
spring:
  datasource:
    url: jdbc:mysql://localhost:3316/springdata
    username: root
    password: 패스워드
    driver-class-name: com.mysql.cj.jdbc.Driver

  jpa:
    hibernate:
      ddl-auto: create # 개발할때만 사용하는게 좋음
```

### Account Entity

```java
package kr.spring.jpa;

import javax.persistence.Entity;
import javax.persistence.GeneratedValue;
import javax.persistence.Id;

@Entity // 해당 클래스가 데이터베이스에 맵핑 된다고 알려주는 어노테이션
public class Account {

    @Id // 주 키에 맵핑
    @GeneratedValue // 값을 자동으로 생성
    private Long id;

    private String username;

    private String password;

    public Long getId() {
        return id;
    }

    public void setId(Long id) {
        this.id = id;
    }

    public String getUsername() {
        return username;
    }

    public void setUsername(String username) {
        this.username = username;
    }

    public String getPassword() {
        return password;
    }

    public void setPassword(String password) {
        this.password = password;
    }
}
```

### JpaRunner

```java
@Component
@Transactional
public class JpaRunner implements ApplicationRunner {

    @PersistenceContext
    EntityManager entityManager; // 영속성 컨텍스트에 접근 및 관리 가능한 인터페이스 제공

    @Override
    public void run(ApplicationArguments args) throws Exception {
        Account account = new Account();
        account.setUsername("TEST");
        account.setPassword("PASSWORD");

        Session session = entityManager.unwrap(Session.class);
        session.save(account);

        // entityManager.persist(account); // 엔티티를 영속성 컨텍스트에 저장
    }
}
```

</details>


# JPA 프로그래밍 - 엔티티 맵핑
---

<details>
    <summary>펼치기</summary>

- @Entity
    - 객체 세상에서 부르는 이름
    - 보통 클래스와 같은 이름을 사용하기 때문에 값을 변경하지 않음
    - 엔티티의 이름은 JQL에서 쓰임
- @Table
    - 릴레이션 세상에서 부르는 이름
    - @Entity의 이름이 기본값
    - 테이블의 이름은 SQL에서 쓰임
- @Id
    - 엔티티의 주키를 맵핑할 때 사용
    - 자바의 모든 Primitive 타입과 그 랩퍼 타입을 사용할 수 있음
        - Date랑 BigDecimal, BigInteger도 사용 가능
    - 복합키를 만드는 맵핑하는 방법도 있지만 그건 논외
- @GeneratedValue
    - 주키의 생성 방법을 맵핑하는 애노테이션
    - 생성 전략과 생성기를 설정
        - 기본 전략은 AUTO : 사용하는 DB에 따라 적절한 전략 선택
        - TABLE, SEQUENCE, IDENTITY 중 하나
- Column
    - 컬럼에 대한 설정
        - unique
        - nullable
        - length
        - ...
- Temporal
    - JPA 2.1까지는 Date와 Calendar까지 지원 추후 지원은 확인해봐야 함 
- Transient
    - 컬럼으로 맵핑하고 싶지 않은 멤버 변수에 사용


### 예제

```java
@Entity // 해당 클래스가 데이터베이스에 맵핑 된다고 알려주는 어노테이션
public class Account {

    @Id // 주 키에 맵핑
    @GeneratedValue // 값을 자동으로 생성
    private Long id;

    @Column(nullable = false, unique = true)
    private String username;

    private String password;

    @Temporal(TemporalType.TIMESTAMP)
    private Date created = new Date();

    @Transient
    private String yes;

    @Transient
    private String no;

    public Long getId() {
        return id;
    }

    public void setId(Long id) {
        this.id = id;
    }

    public String getUsername() {
        return username;
    }

    public void setUsername(String username) {
        this.username = username;
    }

    public String getPassword() {
        return password;
    }

    public void setPassword(String password) {
        this.password = password;
    }
}
```

-> application.yml에 jpa 설정을 `show-sql: true` 이거로 주면 쿼리문이 보임
</details>

# JPA 프로그래밍 - Value 타입 맵핑
---

<details>
    <summary>펼치기</summary>

### 엔티티 타입과 Value 타입 구분

- 식별자가 있어야 하는가 (엔티티)
- 독립적으로 존재해야 하는가 (엔티티)
- 종속적이면 Value 타입

### Value 타입 종류

- 기본 타입(String, Date, Boolean, ...)
- Composite Value 타입
- Collection Value 타입
    - 기본 타입의 콜렉션
    - 컴포짓 타입의 콜렉션

### Composite Value 기본 예제

```java
@Embeddable
public class Address {

    private String street;

    private String city;

    private String state;

    private String zipCode;
}
```

**Entity에 멤버 변수 추가**

```java
    @Embedded
    private Address address;
```
 
**같은 타입이 존재할 때 Override**

```java
    @Embedded
    @AttributeOverrides({
            @AttributeOverride(name = "street", column = @Column(name = "home.street"))
    })
    private Address address;
```

</details>

# JPA 프로그래밍 - 1대다 맵핑
---

<details>
    <summary>펼치기</summary>

- 관계에는 항상 두 엔티티가 존재
    - 둘 중 하나는 그 관계의 주인(Owning)
    - 다른쪽은 종속(non-owning)
    - 해당 관계의 반대쪽 레퍼런스를 가지고 있는 쪽이 주인
- 단뱡향에서의 관계의 주인은 명확


### Study Entity 생성 (단방향 N:1)

```java
@Entity
public class Study {

    @Id
    @GeneratedValue
    private Long id;

    private String name;

    @ManyToOne // 단방향 N:1 관계
    private Account owner;

    public Account getOwner() {
        return owner;
    }

    public void setOwner(Account owner) {
        this.owner = owner;
    }

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
}
```

### Account Entity 수정 (단방향 1:N)

-> Study Entity의 관계관련 멤버 변수 지움

```java
    @OneToMany // 단뱡향 1:N
    private Set<Study> studies = new HashSet<>();
```

### 양방향

- ManyToOne을 가지고 있는쪽이 주인
- OneToMany(mappedBy)
- 주인한테 관계를 설정해야 DB에 반영

-> Study와 Account 관계관련 멤버 변수 입력

```java
    @ManyToOne
    private Account owner;
```

```java
    @OneToMany(mappedBy = "owner")
    private Set<Study> studies = new HashSet<>();
```

양방향일때는 양쪽다 관계에 대해 넣어줘야함(주인은 필수 종속된 쪽은 옵셔널)

```java
account.getStudies().add(study);
study.setOwner(account); // 필수 (주인)
```

</details>

# JPA 프로그래밍 - Cascade 옵션
---

<details>
    <summary>펼치기</summary>

> 엔티티의 상태 변화를 전화 시키는 옵션

- Transient
    - JPA가 모르는 상태
- Persistent
    - JPA가 관리중인 상태
        - 1차 캐시
        - Dirty Checking
        - Write Behind
- Detached
    - JPA가 더 이상 관리하지 않는 상태
- Removed
    - JPA가 관리하긴 하지만 삭제하기로 한 상태

### POST 예제

```java
@Entity
public class Post {

    @Id
    @GeneratedValue
    private Long id;

    private String title;

    @OneToMany(mappedBy = "post", cascade = CascadeType.PERSIST)
    private Set<Comment> comments = new HashSet<>();

    public void addComment(Comment comment) {
        this.getComments().add(comment);
        comment.setPost(this);
    }

    public Long getId() {
        return id;
    }

    public void setId(Long id) {
        this.id = id;
    }

    public String getTitle() {
        return title;
    }

    public void setTitle(String title) {
        this.title = title;
    }

    public Set<Comment> getComments() {
        return comments;
    }

    public void setComments(Set<Comment> comments) {
        this.comments = comments;
    }
}
```

### Comment 예제

```java
@Entity
public class Comment {

    @Id
    @GeneratedValue
    private Long id;

    private String comment;

    @ManyToOne
    private Post post;

    public Long getId() {
        return id;
    }

    public void setId(Long id) {
        this.id = id;
    }

    public String getComment() {
        return comment;
    }

    public void setComment(String comment) {
        this.comment = comment;
    }

    public Post getPost() {
        return post;
    }

    public void setPost(Post post) {
        this.post = post;
    }
}
```

### Runner 예제

```java
@Component
@Transactional
public class JpaRunner implements ApplicationRunner {

    @PersistenceContext
    EntityManager entityManager; // 핵심

    @Override
    public void run(ApplicationArguments args) throws Exception {
        Post post = new Post();
        post.setTitle("아아아아아아아");

        Comment comment = new Comment();
        comment.setComment("언제 다하냐");
        post.addComment(comment);

        Comment comment2 = new Comment();
        comment2.setComment("내말이.....");
        post.addComment(comment2);

        Comment comment3 = new Comment();
        comment3.setComment("여긴 지옥이다!!");
        post.addComment(comment3);

        Session session = entityManager.unwrap(Session.class);
        session.save(post);
    }
}
```

</details>

# JPA 프로그래밍 - Query
---

<details>
    <summary>펼치기</summary>

### JPQL (HQL)

> JPA 또는 하이버네이트가 해당 쿼리를 SQL로 변환해서 실행

```java
        TypedQuery<Post> query = entityManager.createQuery("SELECT p FROM Post as p", Post.class);
        List<Post> posts = query.getResultList();
        posts.forEach(System.out::println);
```

### Criteria

> 타입 세이프

```java
        CriteriaBuilder builder = entityManager.getCriteriaBuilder();
        CriteriaQuery<Post> query = builder.createQuery(Post.class);
        Root<Post> root = query.from(Post.class);
        query.select(root);

        List<Post> posts = entityManager.createQuery(query).getResultList();
        posts.forEach(System.out::println);
```

### Native Query

```java
        List<Post> posts = entityManager.createNativeQuery("SELECT * FROM Post", Post.class).getResultList();
        posts.forEach(System.out::println);
```

</details>