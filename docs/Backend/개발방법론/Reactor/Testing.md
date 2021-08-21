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


