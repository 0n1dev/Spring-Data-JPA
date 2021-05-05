# 스프링 데이터 JPA 소개 및 원리

- JpaRepository<Entity, Id> 인터페이스
    - 매직 인터페이스
    - @Repository가 없어도 빈으로 등록해줌

### 기존 Repository 변경

```java
import org.springframework.data.jpa.repository.JpaRepository;

public interface PostRepository extends JpaRepository<Post, Long> {
}
```

```java
@Component
@Transactional
public class JpaRunner implements ApplicationRunner {

    @Autowired
    PostRepository postRepository;

    @Override
    public void run(ApplicationArguments args) throws Exception {
        postRepository.findAll().forEach(System.out::println);
    }
}
```

-> 핵심은 ImportBeanDefinitionRegistrar