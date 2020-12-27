스프링을 java single application 처럼 사용할 수 있음

```java
@SpringBootApplication
public class Application implements CommandLineRunner {

	private final ApiCallScheduler apiCallScheduler;

	public Application(ApiCallScheduler apiCallScheduler) {
		this.apiCallScheduler = apiCallScheduler;
	}

	public static void main(String[] args) {
		final SpringApplication app = new SpringApplication(Application.class);
		app.setBannerMode(Banner.Mode.OFF);  // banner off
		app.setWebApplicationType(WebApplicationType.NONE);  // was 를 띄우지 않음
		app.run(args);
	}

	@Override
	public void run(String... args) throws Exception {
        // 실행하고자 하는 명령
		apiCallScheduler.apiCall();
        // 시스템 종료(System.exit(0))
		exit(0);
	}
}
```

[참고](https://mkyong.com/spring-boot/spring-boot-non-web-application-example/)