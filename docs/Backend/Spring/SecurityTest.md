# Spring Security Test
참고 : [https://yoon0120.tistory.com/51]

- UserDetailService 를 커스텀하게 구현하면 @WithMockUser 어노테이션이 적용되지 않음
- 참고글의 WithSecurityContextFactory 를 통하여 SecurityContext 를 생성하여 넘겨주는 방법을 사용
- 생성된 User 의 id 가 필요하지만, private 필드라 가져오지 못하는 상황 발생
  - reflection 을 사용하여 id 를 임의로 주입하였다.
  - 통합테스트를 위해 임시방편으로 만들어 두었는데, 사실 문제의 소지는 많은거 같음
    1. 통합테스트를 테스트 환경에서 매번 데이터베이스를 초기화 하지 않으면, userId 가 미리 등록된 다른 User 와 겹칠 수 있음 -> 이로 인해 다른 테스트가 오염될 위험이 있음
    2. 진정한 의미의 통합테스트라고 할 수 있는 지에 대해 고민 필요

```java
public class WithMockCustomUserSecurityContextFactory
    implements WithSecurityContextFactory<WithMockCustomUser> {
    @SneakyThrows
    @Override
    public SecurityContext createSecurityContext(WithMockCustomUser customUser) {
        SecurityContext context = SecurityContextHolder.createEmptyContext();

        SimpleUser principal =
            new SimpleUser(User.create("sc", "test@test.com", "123"));

        Class<?> clazz = principal.getClass();
        Field field = clazz.getDeclaredField("userId");
        field.setAccessible(true);
        field.set(principal, 1L);

        Authentication auth =
            new UsernamePasswordAuthenticationToken(principal, "password", principal.getAuthorities());
        context.setAuthentication(auth);
        return context;
    }
}
```

# WebTestClient 에서의 시큐리티
- WebTestClient 에서는 시큐리티를 따로 추가 설정해주어야 작동한다
> [참고](https://godekdls.github.io/Spring%20Security/reactivetestsupport/#301-testing-reactive-method-security)
```java
@RunWith(SpringRunner.class)
@ContextConfiguration(classes = HelloWebfluxMethodApplication.class)
public class HelloWebfluxMethodApplicationTests {
    @Autowired
    ApplicationContext context;

    WebTestClient rest;

    @Before
    public void setup() {
        this.rest = WebTestClient
            .bindToApplicationContext(this.context)
            // add Spring Security test Support
            .apply(springSecurity())
            .configureClient()
            .filter(basicAuthentication())
            .build();
    }
    // ...
}
```

## Authentication
- WebTestClient 에서 시큐리티 기능 추가 후 기존의 어노테이션 방식을 사용하거나 mutateWith 를 사용하여 인증할 수 있다
```java
// 어노테이션
@Test
@WithMockUser
void testMethod() {
    client.get()
    // 내용...
}

// mutateWith
@Test
void testMethod() {
    client
      .mutateWith(mockUser())   // mock user 추가
      .get()
    // 내용...
}
```

## CSRF
```java
@Test
void testMethod() {
    client
      .mutateWith(csrf())   // CSRF 토큰 제공
      .get()
    // 내용...
}
```
