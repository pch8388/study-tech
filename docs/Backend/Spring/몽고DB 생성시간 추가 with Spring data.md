Spring Webflux 프로젝트 기준

---

# 개요
- Spring Webflux + MongoDB 에서 생성시간, 생성자, 수정시간, 수정자 등의 Spring data 에서 제공하는 Auditing 기능을 사용하여 자동으로 세팅할 수 있다
- 생성시간과 수정시간만 넣을 때와 생성자 수정자를 추가할 때는 조금 조건이 다르다
  -  생성시간과 수정시간만 넣을 때는 AuditorAware 를 정의하지 않아도 된다
  -  생성자와 수정자를 추가하려면 [AuditorAware](https://docs.spring.io/spring-data/mongodb/docs/current/reference/html/#auditing.auditor-aware) 구현 필요

# 구현방법
1. Configuration 클래스에 어노테이션을 추가
  ```java
  @SpringBootApplication
  // 아래 어노테이션을 추가 => 몽고디비+리액티브 조합일 경우
  @EnableReactiveMongoAuditing
  public class FooStudyApplication {

	public static void main(String[] args) {
		SpringApplication.run(FooStudyApplication.class, args);
	}

  }
  ```
2. 수정시간 및 등록시간을 구현하고자 하는 엔티티에 어노테이션 추가
  ```java
  @Getter
  public class Post {

	@Id
	private String id;
	private String title;
	private String content;

	@CreatedDate
	private LocalDateTime createdDate;

	@LastModifiedDate
	private LocalDateTime modifiedDate;

	@Builder
	public Post(String title, String content) {
		this.title = title;
		this.content = content;
	}
  }
  ```
  - @CreatedDate, @LastModifiedDate 어노테이션만 추가해도 Spring data 에서 처리해준다

TODO : 생성자와 수정자를 자동으로 추가하도록 구현하면 이 문서에 내용을 추가