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

[예제](https://github.com/pch8388/study-java-base/blob/master/study-reactive/src/test/java/me/reactive/study/base/GenerateTest.java)

## create : Asynchronous and Multi-threaded
- create 를 사용하면 각 단계마다 값을 여러 개 생산하는 Flux 를 만들 수 있으며, 멀티스레드로도 가능
- generate 와는 다르게 별도 상태 기반 메소드는 없음
- 콜백에서 멀티 스레드 기반 이벤트 트리거 가능
- next, error, complete 메소드를 가지는 FluxSink 를 인자로 하는 Consumer 를 파라미터로 가진다
- 리스너 기반 비동기 api 등 기존 api 를 리액티브하게 연결할 수 있다
- create 람다 내에서 블로킹하면 교착 상태 등의 사이드 이팩트가 발생할 수 있다. 

[예제](https://github.com/pch8388/study-java-base/blob/master/study-reactive/src/test/java/me/reactive/study/base/SequenceCreatorTest.java)

## push : Asynchronous but single-threaded
- generate 와 create 중간 쯤이라고 생각할 수 있다 => 단일 생산자 이벤트 처리에 적합
- 비동기 지원
- 데이터를 생산하는 하나의 스레드에서만 next, complete, error 를 한번에 하나씩 실행 가능
- create 같은 리액터 연산자 대부분은 하이븨드 push/pull 모델
  - 대부분 비동기로 처리하더라도 요청과 관련해서 일부 컴포넌트가 pull 사용

## handle
- 인스턴스 메솓로 공통 연산자처럼 기존 소스에 연결 가능
- Mono, Flux 모두에 존재
- 각 소스에 있는 아이템 중 임의의 값만 생성하거나 일부 아이템을 스킵하는 식으로 활용
  - map + filter 의 느낌

[예제](https://github.com/pch8388/study-java-base/blob/master/study-reactive/src/test/java/me/reactive/study/base/SampleHandleTest.java)

# Threading and Schedulers
- 리액터는 동시성에 구애받지 않음
  - 특정 동시성 모델을 강요하지 않음
  - 연산자 대부분 이전 연산자를 실행햇던 Thread 에서 작업을 이어감
    - 지정하지 않았다면 맨 위에 있는 연산자 자체는 subscribe 를 호출한 Thread 에서 실행

```java
@Test
void thread() throws InterruptedException {
  System.out.println("main thread " + Thread.currentThread().getName());
  Mono<String> mono = Mono.just("hello ");

  Thread t = new Thread(() -> mono.map(msg -> msg + "thread ")
    .subscribe(v ->
      System.out.println(v + Thread.currentThread().getName())));

  t.start();
  t.join();
}

// 출력
// main thread main
// hello thread Thread-0
```

- 리액테에서 실행 모델과 실행 위치는 스케줄러에 의해 결정
  - 스케쥴러는 ExecutorService 와 유사하게 사용할 수도 있지만, 추상적인 사용을 통해 더 넓은 범위의 구현을 할 수도 있다
- 스케쥴러는 정적 메서드를 통해 아래의 실행 컨텍스트에 접급할 수 있다
  - 실행중이지 않은 컨텍스트 (`Schedulers.immediate()`) : 처리시간(processing time)에 제출된 Runnable 을 즉시 실행 => 효과적 실행을 위해 현재 Thread 사용(null object 패턴이나 스케쥴러가 아무것도 하지 않는 것처럼 보일 수 있다)
  - 하나의 재사용 Thread (`Schedulers.single()`) : 모든 caller 가 스케쥴러가 종료될때까지 같은 쓰레드를 재사용한다 => 호출마다 다른 쓰레드 사용을 원한다면 Schedulers.newSingle() 을 호출마다 실행해야 한다
  - unbounded elastic thread pool (`Schedulers.elastic()`) : 이 메소드는 backpressure 이슈를 감추고 너무 많은 스레드를 사용하는 경향이 있어서 잘 사용하지 않음
  - bounded elastic thread pool(`Schedulers.boundedElastic()`)
    - 새 워커 풀을 만들고, 여유 있는 스레드가 있으면 재사용
    - 너무 오랫동안 놀고 있는 워커풀이 있으면 폐기 (디폴트 60초)
    - 풀에 생성할 수 있는 스레드 수를 제한(디폴트 cpu 코어수 * 10)
    - 한도에 도달한 이후 제출한 태스ㅡ크는 100,000 개까지 큐에 담고, 스레드에 여유가 생기면 다시 스케줄링(지연 스케줄링을 사용하면, 스레드가 이용 가능해진 이후부터 지연시간 계산)
    - 블로킹 I/O가 필요할 때 좋은 선택 => 손쉽게 블로킹 프로세스에 별도 스레드를 할당 (너무 많은 스레드를 사용하여 시스템 오버헤드를 일으키지 않도록 주의)
  - 병렬 작업을 튜닝할 수 있는 워커의 고정 풀(`Schedulers.parallel()`) : cpu 코어수만큼 워커 생성
- 리액터는 리액티브 체인에서 실행 컨텍스트(또는 스케줄러)를 전환할 수 있는 `publishOn`과 `subscribeOn`을 제공
  - 두 메서드는 `Scheduler`를 받아 해당 스케줄러로 전환 가능
  - `publishOn`은 체인에서의 위치가 중요하지만 `subscribeOn`은 상관없음

## publishOn
`publishOn` 은 구독자 체인 중간에 적용할 수 있다. 해당 `Scheduler`의 워커에서 콜백을 실행하는 동안 업스트림에서 신호를 받아 다운스트림으로 재생산한다. `publishOn` 뒤에 이어지는 연산자의 실행 위치가 달라질 수 있다
- 실행위치에 대한 경우의 수
  - 실행 컨텍스트를 `Scheduler`가 고른 `Thread`로 변경
  - 리액티브 스트림 정의에 따라 `onNext`는 순차적으로 호출하기 때문에 단일   스레드 사용
  - 특정 `Scheduler`로 실행하지 않았다면 `publishOn` 이후의 연산자는   동일한 스레드에서 이어서 실행
```java
Scheduler scheduler = Schedulers.newParallel("parallel-scheduler", 4);

// publishOn 에서 지정한 스케줄러의 thread 를 사용한다
final Flux<String> flux = Flux.range(1, 2)
  .map(i -> Thread.currentThread().getName() + " " + (10 + i))
  .publishOn(scheduler)
  .map(i -> "value " + i + " " + Thread.currentThread().getName());

final Thread thread = new Thread(() -> flux.subscribe(System.out::println));

// value Thread-0 11 parallel-scheduler-1
// value Thread-0 12 parallel-scheduler-1
```

## subscribeOn
`subscribeOn`이 구독 프로세스에 적용되는 시점은 역방향 체인이 구성될 때이다. `subscribeOn` 위치에 상관없이 항상 데이터 소스를 방출하는 컨텍스트에 영향을 미친다. 하지만 `publishOn`을 호출하면 이후 동작에는 영향을 주지 않는다.
- `subscribeOn`은 전체 체인이 구독하는 Thread 를 변경한다
> 실제로는 체인의 가장 앞에 있는 `subscribeOn` 호출만 고려
```java
Scheduler scheduler = Schedulers.newParallel("parallel-scheduler", 4);

// subscribeOn 에서 지정한 스케줄러의 thread 를 전체 체인에서 사용한다
final Flux<String> flux = Flux.range(1, 2)
  .log()
  .map(i -> 10 + i)
  .subscribeOn(scheduler)
  .map(i -> "value " + i);

final Thread thread = new Thread(() -> flux.subscribe(System.out::println));
thread.start();
thread.join();
```

# Handling Errors
- 리액티브 스트림에서 에러는 종료 이벤트
  - 발생 즉시 시퀀스 종료
  - `Subscriber`의 `onError` 메소드에 에러 전파
- `onError`가 정의되어 있지 않으면 에러 발생시 `UnsupportedOperationException` 발생
  - 이런 예외는 `Exceptions.isErrorCallbackNotImplemented` 메소드로 알아내 분류할 수 있다
- 리액터는 에러 처리 연산자를 제공하기 때문에, 체인 중간에 다른 방식으로도 에러 처리 가능
  ```java
  Flux.just(1, 2, 0)
    .map(i -> "100 / " + i + " = " + (100 / i))
    .onErrorReturn("Divided by zero:(");
  ```
> 리액티브 시쿼스에서 발생하는 모든 에러는 종료 이벤트이기 때문에 에러 처리 연산자를 사용해도 기존 시퀀스를 유지할 수 없다. `onError` 연산자는 새로운 시퀀스(fallback)를 시작한다. 즉, 종료된 시퀀스의 업스트림을 대체한다.

## Error Handling Operators
구독할 때 체인 끝에 사용하는 `onError` 콜백은 `catch` 블록과 유사함
```java
Flux<String> s = Flux.range(1, 10)
  .map(this::doSomethingDangerous)
  .map(this::doSecondTransform);

s.subscribe(value -> System.out.println("RECEIVED " + value),
  error -> System.err.println("CAUGHT " + error) // error handle
);
```

### Static Fallback Value
정적인 값을 정해두고 반환시킨다
```java
final Flux<String> flux = Flux.range(1, 10)
  .map(this::doSomethingDangerous)
  .onErrorReturn("RECOVERED");

flux.subscribe(value -> System.out.println("RECEIVED " + value));

// 예외 발생시 시퀀스가 종료되고
// RECEIVED RECOVERED 가 반환된다
```
Predicate 을 사용하여 일치할때만 static value 를 반환하도록 할 수 있다
```java
final Flux<String> flux = Flux.range(1, 10)
  .map(this::doSomethingDangerous)
  .onErrorReturn(e -> e.getMessage().equals("doSomethingException"), "RECOVERED");

flux.subscribe(value -> System.out.println("RECEIVED " + value));
```
- 일치하지 않은 예외가 발생하면 `ErrorCallbackNotImplemented` 예외가 발생함 => `UnsupportedOperationException` 타입의 리액터에서 재구현한 예외 클래스임

### Fallback Method
캐치한 다음 fallback 메소드를 실행하는 형태
```java
Flux.just("key1", "key2")
  .flatMap(k -> callExternalService(k) 
      .onErrorResume(e -> getFromCache(k)) 
      // 데이터를 외부에서 가져오다가 에러 발생시 cache로부터 데이터를 가져오도록 구현함
  );
```
조건에 따라 다르게 처리할 수 있다
```java
Flux.just("timeout1", "unknown", "key2")
  .flatMap(k -> callExternalService(k)
      .onErrorResume(error -> { 
          if (error instanceof TimeoutException) 
              return getFromCache(k);
          else if (error instanceof UnknownKeyException)  
              return registerNewEntry(k, "DEFAULT");
          else
              return Flux.error(error);
      })
  );
```

### Dynamic Fallback Value
캐치한 다음 동적으로 fallback 값 계산
```java
erroringFlux.onErrorResume(error -> Mono.just(
  MyWrapper.fromError(error)
));
```

### catch and rethrow
```java
Flux.just("timeout1")
  .flatMap(k -> callExternalService(k))
  .onErrorResume(original -> Flux.error(
          new BusinessException("oops, SLA exceeded", original))
  );

// onErrorMap 사용하여 좀 더 직관적으로 표현
Flux.just("timeout1")
  .flatMap(k -> callExternalService(k))
  .onErrorMap(original -> new BusinessException("oops, SLA exceeded", original));
```

### Log or React on the Side
로깅 등의 에러를 전파하며 시퀀스를 수정하지 않는 다른 일을 할때는 `doOnError`연산자를 사용
> `doOn`으로 시작하는 연산자는 side-effect를 끼워 넣을 수 있다. 이를 활용하면 시퀀스는 수정하지 않고 시퀀스 이벤트 안에서 원하는 side-effect를 일으킬 수 있다
```java
LongAdder failureStat = new LongAdder();
Flux<String> flux = Flux.just("unknown")
  .flatMap(k -> callExternalService(k)
      .doOnError(e -> {
          // side-effect 를 일으키는 메소드 실행
          // log 통계를 1 증가시킴
          failureStat.increment();
          log("uh oh, falling back, service failed for key " + k);
      })
  );
```

### Using Resources and the Finally Block
`doFinally`는 시퀀스를 종료(`onComplete` 또는 `onError`)하거나 취소될 때마다 원하는 부수 효과를 실행시킬 수 있다
- `doFinally` 에서는 종료타입을 나타내는 `SignalType`을 consume 한다
```java
Stats stats = new Stats();
LongAdder statsCancel = new LongAdder();

Flux<String> flux = Flux.just("foo", "bar")
    .doOnSubscribe(s -> stats.startTimer())
    .doFinally(type -> { 
        stats.stopTimerAndRecordTiming(); 
        // 취소시마다 증가
        if (type == SignalType.CANCEL)
          statsCancel.increment();
    })
    .take(1); // 아이템을 하나 방출 후 종료
```

`using`은 `Flux`처리가 완료되면 `Flux`를 생산한 리소스로 어떤 처리를 해야 할 때 사용
```java
AtomicBoolean isDisposed = new AtomicBoolean();
Disposable disposableInstance = new Disposable() {
  @Override
  public void dispose() {
    isDisposed.set(true);
  }

  @Override
  public String toString() {
    return "DISPOSABLE";
  }
};

Flux<String> flux =
  Flux.using(
    // 리소스 생성
    () -> disposableInstance,
    // 리소스 처리하고 Flux<T> 리턴
    disposable -> Flux.just(disposable.toString()),
    // 리소스 정리 => Flux 종료 혹은 취소시 호출
    Disposable::dispose
  );

flux.subscribe(System.out::println);

// DISPOSABLE
```