# Job
<img width="739" alt="스크린샷 2022-03-12 오후 11 36 49" src="https://user-images.githubusercontent.com/17218212/158022254-668b5609-1f52-4c15-8f5a-48ddd958eeae.png">

Job : 전체 배치 프로세스를 캡슐화한 엔티티

## JobInstance : 논리적 Job 실행
- `JobInstance` 는 재사용 될 수도 있고, 새로 만들수도 있다
  - 새로 만드는 것은 처음부터 시작을 의미하고, 재사용은 멈췄던 곳에서부터 시작을 의미

## JobExecution : Job 을 한번 실행
- 성공 / 실패의 경우가 있으며, Job 을 한번 실행시마다 생성될 수 있다
- `JobInstance` 는 `JobExecution` 이 성공적으로 종료되기 전까지는 완료되지 않은 것으로 간주

## JobParameters : 배치 Job 실행 시 사용하는 파라미터 셋을 가진 객체
- `JobInstance` = `Job` + `JobParameters`
  - `JobInstance` 를 구분짓는 단위로도 사용
- `Map<String, JobParameter>` 의 래퍼 클래스
- 전달 방법
  - CommandLineJobRunner 에 전달 (커맨드 라인으로 전달시 `--접두사` 의 형태로 전달하면 안됨)
    - `java -jar demo.jar name=param`
    - 타입 지정 : `java -jar demo.jar executionDate(date)=2020/12/27` => 소문자로 전달해야 함
- 접근 방법
  - ChunckContext
    ```java
    @Bean
    public Tasklet hello() {
        return (contribution, chunckContext) -> {
            String name = (String) chunckContext.getStepContext().getJobParameters().get("name");
            System.out.println(String.format("Hello, %s!", name);
            return RepeatStatus.FINISHED;
        };
    }
    ```
  - Late Binding
    ```java
    @StepScope // 스텝의 실행범위에 들어갈때까지 빈생성을 지연시킴
    @Bean
    public Tasklet hello(@Value("#{jobParameters['name']}") String name) {
        return (contribution, chunckContext) -> {ßßß
            System.out.println(String.format("Hello, %s!", name);
            return RepeatStatus.FINISHED;
        };
    }
    ```

## JobParameter
- `JobParametersValidator` 인터페이스를 구현하여 유효성 검증기를 만들 수 있다
  - 기본 구현체인 `DefaultJobParametersValidator` 사용하여 간단하게 쓸 수도 있음
### `JobParameter` 증가시키기
  - `JobParametersIncrementer` 사용
    ```java
    @Bean
    public Job job() {
        return this.jobBuilderFactory.get("basicJob")
                   .start(step1())
                   .validator(validator())
                   .incrementer(new RunIdIncrementer())
                   .build();
    }
    ```
  - `JobParametersIncrementer` 를 직접 구현하여 타임스탬프 등을 증가 시킬 수 있음

## Job 리스너 적용
- `JobExecutionListener` : 잡 실행과 연관
  - 알림, 초기화, 정리 등의 작업에 사용
- JobExecutionListener 구현체 사용
  ```java
  @Bean
  public Job job() {
      return this.jobBuilderFactory.get("basicJob")
                 .start(step1())
                 .validator(validator())
                 .incrementer(new RunIdIncrementer())
                 .listener(구현체)   // JobExecutionListener 구현체를 구현하여 넣어준다
                 .build();
  }
  ```
- 애노테이션 사용
  - `@BeforeJob`, `@AfterJob`
    ```java
    @Bean
    public Job job() {
      return this.jobBuilderFactory.get("basicJob")
                 .start(step1())
                 .validator(validator())
                 .incrementer(new RunIdIncrementer())
                 .listener(JobListenerFactoryBean.getListener(구현클래스))   // 애노테이션을 붙인 클래스를 넣으면 됨(인터페이스 구현 등의 행위 x)
                 .build();
    }
    ```

## ExecutionContext
- Job 의 상태 저장
- key-value store
- `JobExecution`, `StepExecution` 모두 `ExecutionContext`를 가진다

# Step
- Job의 구성요소를 담당
- Tasklet 처리와 Chunk 기반 처리가 있음

## Taklet
- `RepeatStatus.FINISHED` 반환전까지 반복적으로 실행되는 코드 블럭
- `Taklet` 인터페이스를 바로 구현할 수 있다(Functional interface)
- 스프링 배치는 기본 구현체들을 제공

### CallableTaskletAdapter
- `Callable<RepeatStatus>` 인터페이스 구현체를 구성할 수 있게 해주는 어댑터
  ```java
  @Bean
  public CallableTaskletAdapter tasklet() {
    CallableTaskletAdapter adapter = new CallableTaskletAdapter();
    adapter.setCallable(() -> {
       System.ount.println("This was executed in another thread");
       return RepeatStatus.FINISHED; 
    });
  }
  ```
- Step과 다른 thread 에서 실행
  - 하지만 step 과 병렬로 실행되는 것은 아님

### MethodInvokingTaskletAdapter
- 다른 클래스의 메서드를 Job 내의 Tasklet 처럼 사용
  ```java
  @StepScope
  @Bean
  public MethodInvokingTaskletAdapter methodInvokingTasklet(@Value("#{jobParameters['message']}") String message) {
    MethodInvokingTaskletAdapter adapter = new MethodInvokingTaskletAdapter();
    adapter.setTargetObject(new CustomService());
    adapter.setTargetMethod("serviceMethod");  // CustomService.serviceMethod 를 호출하도록 설정
    adapter.setArguments(new String[] {message});
    return adapter;
  }

  class CustomService {
    public void serviceMethod(String message) {
      System.out.println(message);    
    }
  }
  ```

### SystemCommandTasklet
- 시스템 명령을 실행할 때 사용
  ```java
  @Bean
  public SystemCommandTasklet tasklet() {
    SystemCommandTasklet tasklet = new SystemCommandTasklet();
    tasklet.setCommand("touch tmp.txt");
    tasklet.setTimeout(5000);
    tasklet.setInterruptOnCancel(true);
    return tasklet;
  }
  ```
- 환경변수 변경 등도 가능
- 이외에도 많은 기능을 수행할 수 있음

## Chunk
- 최소한 2~3개의 주요 컴포넌트로 구성(ItemReader, ItemProcessor(optional), ItemWriter)
- 레코드를 청크 또는 레코드 그룹 단위로 처리
- 각각의 청크는 자체 트랜잭션으로 실행되고, 처리에 실패하면 마지막으로 성공한 트랜잭션 이후부터 다시 시작 가능