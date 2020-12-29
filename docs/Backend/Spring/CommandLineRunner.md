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

## Test
- POJO 테스트의 경우 문제가 없으나, Web 기술에 의존성이 있는 테스트를 할 경우, main 메서드에서 위와 같이 하면 오류가 발생함
  - 원인은 app 이 run 되자마자 종료를 시켜서 그런것으로 판단됨
- 이에 CommandLineRunner 를 따로 분리하여 Profile 설정을 해주었다.
```java
@Component
@Profile("!test")   // profile 이 test 가 아닐때만 실행
public class AppRunner implements CommandLineRunner {

	private static final Logger logger = LoggerFactory.getLogger(Application.class);

	private final ApiCallScheduler apiCallScheduler;
	private final ArgumentValidateUtils argumentValidateUtils;

	public AppRunner(ApiCallScheduler apiCallScheduler,
		ArgumentValidateUtils argumentValidateUtils) {
		this.apiCallScheduler = apiCallScheduler;
		this.argumentValidateUtils = argumentValidateUtils;
	}

	@Override
	public void run(String... args) throws Exception {
		logger.warn("data.api application start");
		apiCallScheduler.apiCall(argumentValidateUtils.argumentValidate(args));
		logger.warn("data.api application end");
		System.exit(0);
	}
}
```

[참고](https://mkyong.com/spring-boot/spring-boot-non-web-application-example/)
