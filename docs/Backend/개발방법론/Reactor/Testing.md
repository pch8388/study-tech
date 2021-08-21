# 개요
- `StepVerifier` : 시퀀스가 주어진 시나리오를 따르는지 단계별 테스트
- `TestPublisher` : 다운스트림 연산자를 테스트하기 위한 데이터 생산
- 선택 가능한 `Publisher`가 여럿 있는 시퀀스 검증

# Testing a Secnario with StepVerifier
- `Flux`나 `Mono`를 정의해두고 이를 구독하면 어떻게 동작하는 지 테스트
- 단계별 이벤트마다 기대치를 정의하는 테스트 시나리오를 만든다
  - 예를 들어 아래와 같은 질문들을 테스트 할 수 있다
    - 다음에 기대하는 이벤트는 무엇인지
    - `Flux`가 특정 값을 생산하기를 기대하는지
    - 다음 300ms 동안은 아무것도 하지 않기를 기대하는지

```java
public class Append {
  public static <T> Flux<T> appendBoomError(Flux<T> source) {
    // concatWith 는 stream 마지막에 other 을 이어 붙인다
	return source.concatWith(Mono.error(new IllegalArgumentException("boom")));
  }
}

@Test
public void testAppendBoomError() {
  Flux<String> source = Flux.just("thing1", "thing2");

  StepVerifier.create(
        Append.appendBoomError(source))
    .expectNext("thing1")
    .expectNext("thing2")
    .expectErrorMessage("boom")   // 마지막에 Mono.error 를 이어 붙였기 때문에 에러 메시지가 방출된다
    .verify();    // 테스트를 트리거한다
}
```
## StepVerifier 메소드
- `expectNext(T...), expectNextCount(long)` : 다음에 발생할 신호에 대한 기대값
- `consumeNextWith(Consumer<T>)` : 다음 신호를 컨슘. 일부 시퀀스를 넘어가고 싶거나 신호 컨텐츠에 커스텀한 `assertion`을 적용하고 싶을 때 사용
- `thenAwait(Duration), then(Runnable)` : 임의의 코드를 중단하거나 실행하는 등 다양한 조치를 취함
- `expectComplete(), expectError()` : 종료 이벤트
- `verify()` : 검증을 트리거 => 타임아웃이 없음(무한정 블로킹)
  - `verifyComplete(), verifyError(), verifyErrorMessage(String)` : `verify` + 종료 이벤트
  - `verify(Duration)` : 타임아웃 지정
- `StepVerifier.setDefualtTimeout(Duration)` : 전역 타임아웃 지정

## Better Identifying Test Failures
- `StepVerifier`는 테스트를 실패하게 만든 스텝을 찾아내기 위해 두가지 옵션을 제공
  - `as(String)`
    - `expect*` 메소드 뒤에 사용하면 이전 expectation에 대한 설명을 추가해준다. 
    - 해당 expectation이 실패하면 문자열을 포함하는 에러 메시지를 건네줌. 
    - 마지막 종료 expectation과 `verify`에는 사용 불가
  - `StepVerifierOptions.create().scenarioName(String)` : `StepVerifierOptions`로 `StepVerifier`를 만들면 `scenarioName`메소드로 전체 시나리오 이름을 지정할 수 있고, 이 이름은 assertion 에러 메시지에 사용

# Manipulating Time
`StepVerifier`를 시간 기반 연산자와 사용하면 오랜 시간이 걸리는 코드를 실제로 기다리지 않고 테스트 할 수 있다
```java 
// 항상 람다안에서 Flux(Mono)를 초기화하는 Supplier<Publisher<T>> 를 파라미터로 줘야 한다
StepVerifier.withVirtualTime(() -> Mono.delay(Duration.ofDays(1)))
// ... 또다른 expectation 들
```
- `Schedulers` 팩토리에 있는 커스텀 `Scheduler`를 연결
  - 시간 기반 연산자가 디폴트 `Schedulers.parallel()`스케줄러를 `VirtualTimeScheduler`로 변경
- 테스트할 Flux 인스턴스를 스케줄러가 세팅된 이후에 생성되어야 하기 때문에 인자로 `Supplier<Publisher<T>>`를 넘겨야하고, 해당 `Supplier`에서 `Flux or Mono`를 생성해야 한다
- 가상시간을 위한 expectation 메소드
  - `thenAwait(Duration)` : 잠시 스텝 검증을 멈춘다(몇 가지 신호가 발생하거나 지연되도록)
  - `expectNoEvent(Duration)` : 주어진 시간 동안 시퀀스를 재생하지만 그 시간 동안 신호가 하나라도 발생하면 테스트 실패
  - 위의 두 메소드는 classic 모드에선 주어진 시간 동안 스레드 중지, virtual 모드에선 가상 시계를 사용
> `expectNovent`는 `subscription`도 하나의 이벤트로 간주. 이 메소드를 첫 번째 스텝에 쓰면 구독 신호가 감지되어 보통 실패 => 대신 `expectSubscription().expectNoEvent(Duaration)` 사용
```java
@Test
void virtualTime() {
  StepVerifier.withVirtualTime(() -> Mono.delay(Duration.ofDays(1)))
    .expectSubscription()
    .expectNoEvent(Duration.ofDays(1))  // 하루동안 아무 일도 일어나지 않기를 기대함
    .expectNext(0L)
    .verifyComplete();
}

// 위의 코드와 같지만 thenAwait 은 아무 이벤트도 발생하지 않는 것을 보장하지 않는다
@Test
void thenAwait() {
  StepVerifier.withVirtualTime(() -> Mono.delay(Duration.ofDays(1)))
    .thenAwait(Duration.ofDays(1))
    .expectNext(0L)
    .verifyComplete();
}
```
> `verify()`는 `Duration`을 반환 => 전체 테스트를 하는 동안 걸린 실제 시간

# Performing Post-execution Assertions with StepVerifier
`verifyTenAssertThat()` : verify()`를 트리거하는 대신 다른 assertion API 로 전환하여 검증할 수 있다

# Testing the Context
`StepVerifier`에서 `Context`를 전파하는 두 가지 expectation
- `expectAccessibleContext` : 전파한 `Context` 관련 expectation을 세팅할 수 있는 `ContextExpectations` 객체 반환. 시퀀스 expectation 셋으로 돌아가려면 `then()` 호출
- `expectNoAccessibleContext` : 테스트하는 동안 연산자 체인에서 `Context`가 전파되지 않는다는 expectation 세팅. 테스트에서 사용하는 `Publisher`가 리액터의 publisher가 아니거나 `Context`를 전파할 연산자가 없는 경우 주로 사용
> verifier를 만들때 `StepVerifierOptions`를 사용하면 `StepVerifier`에 테스트 환경에서 필요한 초기 `Context` wndlq
```java
@Test
void context() {
   StepVerifier.create(Mono.just(1).map(i -> i + 10),
        StepVerifierOptions.create().withInitialContext(Context.of("thing1", "thing2")))
    .expectAccessibleContext()
    .contains("thing1", "thing2")  // Context 를 위한 expectation
    .assertThat(context -> assertEquals(context.get("thing1"), "thing2")) // 이런 형태로 Context 내용 검증 가능
    .then()
    .expectNext(11)
    .verifyComplete();
}
```

# Manually Emitting with TestPublisher
`TestPublisher`는 데이터 소스를 완전히 통제해서 직접 테스트하고 싶은 상황과 거의 근접한 신호를 트리거한다
- 연산자를 직접 구현하거나 리액티브 스트림 스펙을 잘 따르는지, 데이터 소스가 잘 동작하지 않는 상황을 검증하기도 한다
- 다양한 신호를 프로그래밍 방식으로 트리거 할 수 있는 `Publisher<T>`이다
  - `next(T), next(T, T...)` :  1~n `onNext` 신호를 트리거
  - `emit(T...)` : 1~n `onNext` 신호를 트리거 + `complete()`
  - `complete()` : `onComplete` 신호와 함께 종료
  - `error(Throwable)` : `onError` 신호와 함께 종료

## TestPublisher 팩토리 메소드
- `create` : 잘 동작하는 `TestPublisher`를 얻음
- `createNonCompliant` : 제대로 동작하지 않는 `TestPublisher`를 얻음
  - `TestPublisher.Violation` 열거형 값을 하나 이상 인자로 받음 => 이 값으로 publisher가 따르지 않을 스펙 정의
    - `REQUEST_OVERFLOW` : 요청이 충분하지 않을 때도 `IllegalStateException`을 트리거하는 대신 `next`호출 허용
    - `ALLOW_NULL` : null 값이 들어오면 `NullPointException`을 트리거하는 대신 `next`호출 허용
    - `CLEANUP_ON_TERMINATE` : row 하나에서 종료 신호를 여러번 허용 (`complete(), error(), emit()`)
    - `DEFER_CANCELLATION` : `TestPublisher`가 취소 신호를 무시하고, 이전 신호한테 밀린 것처럼 계속해서 신호를 방출하도록 허용
- `flux(), mono()` 를 사용하여 `Flux, Mono` 로 전환 가능

# Checking the Execution Path with PublisherProbe
연산자 체인 중 실행경로가 여러 곳으로 나뉘는 별도 하위 시퀀스에 대해 테스트 하는 방법
- source 가 빈 경우 `switchIfEmpty`를 사용하여 fallback 하는 메소드 테스트
```java
@Test
void switchIfEmpty() {
  StepVerifier.create(processOrFallback(Mono.empty(), Mono.just("EMPTY_PHRASE")))
    .expectNext("EMPTY_PHRASE")
    .verifyComplete();
}

static Flux<String> processOrFallback(Mono<String> source, Publisher<String> fallback) {
  return source.flatMapMany(phrase -> Flux.fromArray(phrase.split("\\s+")))
    .switchIfEmpty(fallback);
}
```
- `Mono<Void>`가 생산되는 경우, 소스가 완료되길 기다렸다가 추가 작업을 수행 뒤 완료
```java
@Test
void testCommandEmptyPathIsUsed() {
  PublisherProbe<Void> probe = PublisherProbe.empty();

  StepVerifier.create(processOrFallback(Mono.empty(), probe.mono()))
    .verifyComplete();

  probe.assertWasSubscribed();
  probe.assertWasRequested();
  probe.assertWasNotCancelled();
}

static Mono<Void> processOrFallback(Mono<String> commandSource, Mono<Void> doWhenEmpty) {
  return commandSource.flatMap(command -> executeCommand(command).then())
    .switchIfEmpty(doWhenEmpty);
}

private static Mono<String> executeCommand(String command) {
  return Mono.just(command + " DONE");
}
```