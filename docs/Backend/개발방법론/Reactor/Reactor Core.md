# 개요
리액터를 사용하면 Publisher 를 구현하고 있는 리액티브 타입을 Flux, Mono 등으로 다양하게 구성할 수 있고, 풍부한 연산자를 제공한다.

# Flux
![Flux](../../../images/flux.png)
- Flux 는 0개 ~ N개 까지의 아이템을 생상하는 비동기 시퀀스를 나타내는 표준 Publisher<T> 로, 완료나 에러신호(onNext, onComplete, onError)로 종료된다.
  - 종료 이벤트를 포함한 모든 이벤트는 선택사항
  - 비어있는 유한한 시퀀스, 무한 시퀀스, 값이 있는 무한 시퀀스 등등..

# Mono
![Mono](../../../images/mono.png)
- Mono 는 0개 또는 1개의 아이템 생산에 특화된 Publisher<T> 로 onComplete 똔느 onError 신호로 종료되며 이러한 이벤트는 선택사항이다
  - Flux 가 지원하는 연산자를 일부 제공하며, 일부 연산자 중 Mono 와 다른 Publisher 를 합쳐 Flux 로 변환이 가능하다 => Mono 를 바로 Flux 로 변환도 가능
- Mono 값은 필요 없고 완료 개념만 있으면 되는 비동기 처리는 Mono<Void> 사용

# subscribe
- Flux 와 Mono 를 시작하는 가장 쉬운 방법은 팩토리 메소드 사용
```java
Flux<String> seq = Flux.just("foo", "bar", "foobar");

// 컬렉션 -> flux 변환
List<String> iterable = List.of("foo", "bar", "foobar");
Flux<String> seq2 = Flux.fromIterable(iterable);

// 팩토리 메소드는 값이 없어도 제네릭 타입이 필요함
Mono<String> emptyMono = Mono.empty();

// 첫번째 파라미터는 시작 값, 두 번째 파라미터는 생산할 아이템 수
Flux<Integer> numbersFromFiveToSeven = Flux.range(5, 3);
```

## subscribe example
[다양한 사용 예](https://github.com/pch8388/study-java-base/blob/master/study-reactive/src/test/java/me/reactive/study/base/SubscribeTests.java)

## subscribe 취소
- 람다 기반 subscribe() 메소드는 Disposable 타입 리턴
- Disposable 인터페이스는 dispose() 메소드 호출로 구독을 취소할 수 있음
- Flux, Mono 관점에서 취소는 소스가 데이터 생산을 중단해야 한다는 신호
  - 즉각적 중단을 보장하지 않음 => 취소 명령을 받기전 데이터 생산하고 완료 할 수도 있음

### Disposables
- `Disposable` 관련 유틸 클래스
- `Disposables.swap()` : `Disposable` 래퍼를 생성해서 기존 `Disposable` 을 자동으로 취소하고 다른 걸로 변경
  - 래퍼를 폐기하면 자체적으로 close => 현재 구현체와 이후에 대체하는 모든 구현쳬를 폐기
- `Disposables.composite()` : 여러 `Disposable` 을 수집해서 나중에 한번에 폐기할 수 있다
  - `composite` 의 `dispose()` 메소드를 호출하고 나면 다른 `Disposable` 을 추가할 때마다 즉시 폐기

## BaseSubscriber
- `subscribe` 메소드에 람다를 직접 조합하는 방법 대신, `Subscriber` 를 넘길 수 있음
- `BaseSubscriber` 는 `Subscriber` 를 확장
- 요청할 양을 커스텀 하는 등의 기능을 편리하게 구현할 수 있다. [예제](https://github.com/pch8388/study-java-base/blob/master/study-reactive/src/main/java/me/reactive/study/base/SampleSubscriber.java
)

## Backpressure
- 리액터에서는 `backpressure` 를 업스트림 연산자로 `request` 를 보내는 식으로 구현하였다
- 현재의 모든 요청을 합쳐서 `demand` 또는 `pending request` 라고 함
- `Demand` 는 언바운드 요청을 나타내는 Long.MAX_VALUE 로 제한 => 가능한 빨리 생산
  - 기본적으로 backpressure 비활성화
- 첫 요청은 구독하는 타이밍에 구독자 쪽에서는 보내는 데, 모든 데이터를 직접적으로 즉시 구독하면 Long.MAX_VALUE 언바운드 요청을 트리거링한다.
  - `subscribe()`및 람다 기반 오버로딩 메소드 대부분 (`Consumer<Subscription>` 을 받는 메서드 제외)
  - `block()`, `blockFirst()`, `blockLast()`
  - `toIterable()`, `toStream()`
- 요청 커스텀
  ```java
  Flux.range(1, 10)
			.doOnRequest(r -> System.out.println("request of " + r))
			.subscribe(new BaseSubscriber<>() {
				
        @Override
				protected void hookOnSubscribe(Subscription subscription) {
          // request 1 을 backpressure 로 upstream 에 보냄
					request(1);
				}

				@Override
				protected void hookOnNext(Integer value) {
          // next 가 호출 되며 cancle 이 실행되어 구독이 종료된다
					System.out.println("Cancelling after having received " + value);
          // BaseSubscriber 에서 구독을 취소할 수 있도록 함
					cancel();
				}
			});
  ```
  출력 결과
  ```
  request of 1
  Cancelling after having received 1
  ```
- 이와 같이 요청을 조작한다면 최소한 시퀀스를 진행할 수 있을 만큼 demand 를 생산하여야 하는데, 그렇지 않으면 Flux 가 오도 가도 못하는 상황이 벌어질 수 있다. `BaseSubscriber` 의 `hookOnSubscribe` 는 언바운드 요청을 기본으로하여, 이 훅을 `override` 하면 최소한 한번은 `request` 를 호출하여야 한다

# 이벤트를 프로그래밍 방식으로 정의
Sink : 관련 이벤트(onNext, onError 및 onComplete)를 프로그래밍 방식으로 정의하여 Flux 또는 Mono를 만드는 메소드 API

## generate : Synchronous
- 동기 및 일대일 방출을 위한 generate 메서드 사용
- 싱크는 SynchronousSink, next() 메서드는 콜백을 호출할 때마다 최대 한 번만 호출됨
- error(Throwable) 이나 complete() 를 호출할 수 있지만 필수는 아님
  - 호출하지 않으면 무한한 스트림
- 초기상태를 `Suppriler<S>` 로 제공, generator 함수는 각 단계마다 새로운 상태 반환

[예제](https://github.com/pch8388/study-java-base/blob/6a477534ec5e30cf40818245c17f46ebc01afd03/study-reactive/src/test/java/me/reactive/study/base/SinkTest.java)

## create : Asynchronous and Multi-threaded
- create 를 사용하면 각 단계마다 값을 여러 개 생산하는 Flux 를 만들 수 있으며, 멀티스레드로도 가능
- generate 와는 다르게 별도 상태 기반 메소드는 없음
- 콜백에서 멀티 스레드 기반 이벤트 트리거 가능
- next, error, complete 메소드를 가지는 FluxSink 를 인자로 하는 Consumer 를 파라미터로 가진다
- 리스너 기반 비동기 api 등 기존 api 를 리액티브하게 연결할 수 있다
- create 람다 내에서 블로킹하면 교착 상태 등의 사이드 이팩트가 발생할 수 있다. 

## push : Asynchronous but single-threaded
- generate 와 create 중간 쯤이라고 생각할 수 있다 => 단일 생산자 이벤트 처리에 적합
- 비동기 지원
- 데이터를 생산하는 하나의 스레드에서만 next, complete, error 를 한번에 하나씩 실행 가능
- create 같은 리액터 연산자 대부분은 하이븨드 push/pull 모델
  - 대부분 비동기로 처리하더라도 요청과 관련해서 일부 컴포넌트가 pull 사용
  