# Spring RestDocs
> [Document](https://docs.spring.io/spring-restdocs/docs/current/reference/html5)
- 테스트를 통과해야만 문서가 생성됨
  - 테스트를 강제할 수 있는 효과가 있음
- Swagger 나 RestDocs 나 문서를 친절하게 작성하면 코드가 지저분해지고 리팩터링 할 것이 많아지지만 RestDocs 는 Swagger 에 비해 프로덕션 코드에 침투력이 없고, 리팩터링해서 단순화할 수 있는 여지가 있다고 생각된다
- Swagger 는 외부에서 API 를 테스트하기가 편하다. 예를 들어, 각 팀에서 API 를 제공해서 기능을 개발할 때, 테스트 코드를 제공하는 것보다 Swagger API 를 제공하는 것이 편하다

## pom.xml
```xml
<build>
  <plugins>
    <plugin>
        <groupId>org.asciidoctor</groupId>
        <artifactId>asciidoctor-maven-plugin</artifactId>
        <version>1.5.8</version>
        <executions>
            <execution>
                <id>generate-docs</id>
                <phase>prepare-package</phase>
                <goals>
                    <goal>process-asciidoc</goal>
                </goals>
                <configuration>
                    <backend>html</backend>
                    <doctype>book</doctype>
                </configuration>
            </execution>
        </executions>
        <dependencies>
            <dependency>
                <groupId>org.springframework.restdocs</groupId>
                <artifactId>spring-restdocs-asciidoctor</artifactId>
                <version>${spring-restdocs.version}</version>
            </dependency>
        </dependencies>
    </plugin>
    <plugin>
        <groupId>org.apache.maven.plugins</groupId>
        <artifactId>maven-resources-plugin</artifactId>
        <version>2.7</version>
        <executions>
            <execution>
                <id>copy-resources</id>
                <phase>prepare-package</phase>
                <goals>
                    <goal>copy-resources</goal>
                </goals>
                <configuration>
                    <outputDirectory>src/main/resources/static/docs</outputDirectory>
                    <resources>
                        <resource>
                            <directory>target/generated-docs</directory>
                        </resource>
                    </resources>
                </configuration>
            </execution>
        </executions>
    </plugin>
  </plugins>
</build>
```

## Spring Webflux 에서 RestDocs
- Configuration
```java
@SpringBootTest(webEnvironment = SpringBootTest.WebEnvironment.RANDOM_PORT)
@AutoConfigureWebTestClient
@AutoConfigureRestDocs
public class AcceptanceTest {

	@Autowired
	protected WebTestClient client;
}
```

### RequestBody, ResponseBody
- requestFields 와 responseFields 를 통하여 Snippet 을 생성한다

- Test code
```java
// given, when
final Post responsePost = client.post().uri("/posts")
    .contentType(MediaType.APPLICATION_JSON)
    .accept(MediaType.APPLICATION_JSON)
    // BodyInserters.fromPublisher 로 Mono 를 제공
    .body(BodyInserters.fromPublisher(Mono.just(requestPost(title, content)), RequestPostDto.class))
    .exchange()
    .expectStatus().isCreated()
    .expectBody(Post.class)
    // consumeWith 메서드를 사용
    .consumeWith(document("post-new-item",
        preprocessRequest(prettyPrint()),
        preprocessResponse(prettyPrint()),
        requestFields(
            fieldWithPath("title").description("게시글 제목"),
            fieldWithPath("content").description("게시글 내용")
        ),
        responseFields(
            fieldWithPath("id").type(JsonFieldType.STRING).description("게시글 id"),
            fieldWithPath("title").type(JsonFieldType.STRING).description("게시글 제목"),
            fieldWithPath("content").type(JsonFieldType.STRING).description("게시글 내용")
        )))
    .returnResult()
    .getResponseBody();

// then
assertThat(responsePost).isNotNull();
assertThat(responsePost.getId()).isNotEmpty();
assertThat(responsePost.getTitle()).isEqualTo(title);
assertThat(responsePost.getContent()).isEqualTo(content);
```

### PathParameter 
- uri 부분에서 PathVariable (대괄호) 표현으로 작성해주어야 정상적으로 문서가 생성됨
```java
final ResponsePostDto responseDto = client.patch().uri("/posts/{postId}", postId)
			.contentType(APPLICATION_JSON)
			.accept(APPLICATION_JSON)
			.body(BodyInserters.fromPublisher(Mono.just(requestUpdatePost(updateContent)),
				RequestUpdatePostDto.class))
			.exchange()
			.expectStatus().isOk()
			.expectBody(ResponsePostDto.class)
			.consumeWith(document("post-update-item",
                preprocessRequest(prettyPrint()),
                preprocessResponse(prettyPrint()),
				pathParameters(parameterWithName("postId").description("게시글 id")),
				getUpdatePostRequestSnippet(), getPostResponseSnippet()))
			.returnResult()
			.getResponseBody();

		// then
		assertThat(responseDto).isNotNull();
		assertThat(responseDto.getId()).isNotEmpty();
		assertThat(responseDto.getContent()).isEqualTo(updateContent);
		assertThat(responseDto.getCreatedDate())
			.isNotEqualTo(responseDto.getModifiedDate());
```

## 문서 생성
- 테스트 코드 작성 후 maven package 나 명령어로 직접 prepare-package 를 실행하면 target/generated-docs 아래에 ascii doc snippet 이 생성된다
- 생성된 것을 바탕으로 main 디렉터리 바로 아래에 asciidoc/index.adoc 을 생성하여 문서를 만든다
  ```adoc
  = Spring boot Study

  == 게시글 등록

  === Request
  include::{snippets}/post-new-item/http-request.adoc[]

  === Request Parameter
  include::{snippets}/post-new-item/request-fields.adoc[]

  === Response
  include::{snippets}/post-new-item/http-response.adoc[]

  === Response Fields
  include::{snippets}/post-new-item/response-fields.adoc[]
  ```
- 다시 maven pakage 를 실행하면 resources/static/docs 아래에 index.html 이 생성된 것을 확인할 수 있다
- 생성된 것을 확인 후 ```{domain}/docs/index.html``` 에 접속하면 문서를 확인 할 수 있다