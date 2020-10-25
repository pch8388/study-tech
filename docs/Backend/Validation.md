## Java Bean Validation

- 일반적으로 validation은 전 계층에 걸치지만 검증 로직의 불일치 등의 문제가 발생할 수 있고, 코드가 지저분하고 산만해질 가능성이 있다.
- 데이터에 대한 검증로직을 도메인 모델 자체에 묶어서 표한한다 ⇒ Bean Validation 명세

### Hibernate Validator

- Bean Validation 명세의 구현체
- Spring Boot 2.0 이상의 버전에서는 Hibernate Validator 6.0.1 Final 사용
- 어노테이션을 사용하여 제약조건을 명시한다

    [Hibernate Validator 6.1.5.Final - Jakarta Bean Validation Reference Implementation: Reference Guide](http://docs.jboss.org/hibernate/stable/validator/reference/en-US/html_single/#section-builtin-constraints)

- javax.validation.Validator 를 사용하여 검증
    - 제약조건 위반시 ConstraintViolation 인터페이스에 내용을 담아 반환

    ```java
    ValidatorFactory factory = Validation.buildDefaultValidatorFactory();
    Validator validator = factory.getValidator();

    Set<ConstraintViolation<Car>> constraintViolation = validator.validate(car);
    ```

- 그룹핑 제약조건

    ```java
    public interface Group {}

    public class Person {
      @NotEmpty
      private String name;

      @Positive
      @Min(value = 18, groups = Group.class)
    	private int age;

      @NotEmpty(groups = Group.class)
      private String x;
    }

    // ... client class
    // group 제약조건만 검사
    Set<ConstraintViolation<Person>> constraintViolations = 
    															validator.validate(person, Group.class);

    // default 조건과 group 제약조건 검사
    Set<ConstraintViolation<Person>> constraintViolations = 
    									validator.validate(person, Default.class, Group.class);
    ```

- 필드제약조건 : 인스턴스 변수
- 속성제약조건 : 메서드 리턴 값(getXX, isXX 이름의 메소드)

    ```java
    class Person {
    	@Size(min = 1, max = 5)   //필드제약
    	private String name;
    	
    	@Size(min = 1, max = 5)   // 속성제약
    	public String getNAme() {
    		return "이름은 " + name;
    	}
    }
    ```

- 객체 그래프 - cascaded validation (@Valid)
    - 해당하는 필드에 애노테이션 추가

    ```java
    public class Car {
      @NotNull(message ="driver is not null")
      @Valid
    	private Person driver;
    }

    public Person {
      @NotNull
      private String name;
    }
    ```

- 컨테이너 요소 유효성 검사 : Iterable, Map, Optional

    ```java
    public class Car {
    	@NotEmpty
    	private String model;

    	@NotEmpty
    	private List<@NotNull @Valid Person> drivers;
    }
    ```

- 매개변수 : ExecutableValidator
- 응답값 : ExecutableValidator
- 복수의 매개변수 유효성 검사
    - 사용자 정의 애노테이션 + ConstraintValidator
- 스크립트 사용(@ScriptAssert)

### Spring Validation

- AOP 등을 사용
- @Validated : 유효성 검사 진입점
    - MethodValidationPostProcessor 빈 정의 필요
        - 제약조건 위반 시 ConstraintViolationException 발생 ⇒ @ExceptionHanlder 로 처리할 수 있음
- Spring 은 Java Bean Validation 을 지원하는 LocalValidatorFactoryBean 과 SpringValidatorAdaptor 제공
- 바인딩 실패시 MethodArgumentNotValidException 발생
- 사용자 정의 Validator를 만들어 처리 가능(@InitBinder)
- 자바의 @Valid 는 그룹처리가 불가능
- @Validated 는 Java 스펙을 스프링이 확장

### Spring life cycle
- @Lazy 어노테이션은 빈을 lazy 하게 생성한다 ⇒ 사용할 때 생성
- MethodValidationPostProcessor 를 스프링 부트에서는 ValidationAutoConfiguration 에서 자동 설정해준다.(Validator 를 파라미터로 받아서 set 한다. ⇒ Validator를 Lazy하게 받음)
- WebMvcConfigurationSupport 의 requestMappingHandlerAdapter 메서드에서 validator 빈을 받아 RequestMappingHandlerAdapter 에 주입시킨다.
    - 이 validator 는 같은 클래스에 빈으로 정의된 mvcValidator 메서드에서 반환되는 데, 이 mvcValidator메서드는 WebMvcAutoConfiguration 클래스에서 오버라이딩 한다.
    - WebMvcAutoConfiguration  mvcValidator 메서드는 클래스 로더에서 javax.validation.Validator 라는 이름으로 밸리데이터를 검색하여 없으면 상위클래스에게 위임하고 (super.mvcValidator() 호출) 있으면 applicationContext 에서 찾는다(즉, 빈으로 생성된 validator를 찾는다. ⇒ 찾아서 있으면 ValidatorAdapter 에서 SpringValidatorAdaper class로 wrapping 하여 반환한다)