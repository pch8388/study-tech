# B마트 전시 도메인 CQRS 적용하기
- 정규화된 테이블을 전시에서 보여줄 때는 비정규화하여 보여주어야 한다
- CQRS : 명령과 조회의 책임을 분리
- 외부의 요소들로 인해 도메인 모델은 점점 복잡해진다
  - 내부 관리용 데이터
  - 정책, 외부 주입 데이터

## 생성, 저장과 조회 시점 나누기
- 조회 모델은 대부분 한정적인 정보 => DTO 생성
- 조회는 바로 가져다 쓸 수 있는 비정규화 된 데이터를 사용하는 것이 좋다
  - 명령 모델 변경시 조회 모델을 생성한다
  - 명령 모델보다 조회 모델이 더 많이 사용될 때 이득이 크다
- fallback method 사용을 위해 기존 데이터베이스에도 저장
- 이렇게만 나누면 조회 모델이 생성, 저장되는 책임이 명령 모델에 있기 때문에 이벤트 소싱 패턴을 통해 분리할 수 있다
  - 이벤트만 발행하고 컨슈머가 조회 모델을 생성하고 저장하는 역할을 맡는다
  - 세가지 영역으로 분리됨 => 명령 모델 애플리케이션, 조회 모델 생성/저장 애플리케이션, 조회 모델 애플리케이션

## CQRS 적용하기
- 모델을 명확히 나눠야 한다
- 변경감지대상과 이벤트를 정의해야 한다 
  - 엔티티 변경에 따라 갱신할 대상 id
  - 변경 감지할 property
  - 실행 method
  - 변경에 대한 log 정보
- 갱신대상의 id만 persistence 데이터에 접근해서 직접 조회해서 조회모델에 반영하도록 구성

### 변경감지 방법
- JPA EntityListeners
  - @EntityListeners 어노테이션과 함께 callback listeners 사용 가능
- Hibernate - EventListener
  - EventListenerRegistry 를 통해 26가지 디테일한 상황에 콜백
  - 이력관리에 유용한 정보가 있음
- Hibernate Interceptor
  - Session 혹은 SessionFactory에 Interceptor 등록 가능
  - 저장될 데이터 조작가능
- Spring AOP
  - Method에만 설정 가능
  - Pointcut 문법으로 동작
  - 특정 케이스에만 사용하는 것이 좋다

> Hibernate - EventListener, Spring AOP 가 사용하기 가장 좋다

- Kafka connect를 통해 CDC를 적용하는 것이 가장 활용도가 높을 것 같고, 앞으로 도입할 예정

### 이벤트 발행하기
Amazon SNS, Amazon SQS 사용

### 이벤트 처리하기
- 단건으로 여러 차례 데이터가 변경된다면 컨슈머가 비효율적으로 작동할 수 있다
- elk : 이벤트 로깅
- redis : 변경 대상 버퍼에 저장
- spring @Scheduled : 스케줄러로 이벤트 대상 동기화 batch 처리
- 주기적으로 Full Batch 실행하여 조회모델 동기화