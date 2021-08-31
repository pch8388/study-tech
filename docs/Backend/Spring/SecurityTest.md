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

# WebTestClient 에서의 시큐리티 (reactive)
- WebTestClient 에서는 시큐리티를 따로 추가 설정해주어야 작동한다
> [참고](https://godekdls.github.io/Spring%20Security/reactivetestsupport/#301-testing-reactive-method-security)

[예제] (https://github.com/pch8388/foo-board/blob/main/src/test/java/me/study/foostudy/board/api/PostApiTest.java)

```java
@SpringBootTest
@AutoConfigureWebTestClient
public class HelloWebfluxMethodApplicationTests {
    private PostService postService;
	WebTestClient client;

	@BeforeEach
	void setUp() {
		postService = Mockito.mock(PostService.class);
		client = WebTestClient
			.bindToController(new PostApi(postService))
			.apply(springSecurity())
			.configureClient()
            // filter 에 인증정보를 주면 인증정보를 생성해준다 => 즉, 따로 인증할 필요 없음
			.filter(basicAuthentication("test", "test"))
			.build();
	}
    // ...
}
```

## 인수테스트
- 인수테스트 작성시에는 로그인 정보도 같이 넘겨주어야 실제 실행하는 것과 같은 효과를 낼 수 있다.
- 위에서 소개한 controller 를 바인드하거나 하는 방법은 실제 환경과 또 차이가 있을 수 있기 때문에 최대한 실제상황과 비슷하게 테스트해야 한다

```java
@DisplayName("게시글을 등록한다")
@Test
void createPost() {
    게시글_등록_되어있음("새로운 게시글 제목", "새로운 게시글 내용을 등록합니다.");
}

private ResponsePostDto 게시글_등록(String title, String content) {
    return client.post().uri("/posts")
        // 헤더에 인증정보를 설정하여 넘겨준다
        .headers(headers -> headers.setBasicAuth("test", "test"))
        .contentType(APPLICATION_JSON)
        .accept(APPLICATION_JSON)
        .body(BodyInserters.fromPublisher(Mono.just(requestPost(title, content)),
            RequestPostDto.class))
        .exchange()
        .expectStatus().isCreated()
        .expectBody(ResponsePostDto.class)
        .consumeWith(getDocument("post-new-item",
            getNewPostRequestSnippet(), getPostResponseSnippet()))
        .returnResult()
        .getResponseBody();
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
