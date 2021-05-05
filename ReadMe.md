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