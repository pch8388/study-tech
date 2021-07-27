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
- Disposable 관련 유틸 클래스
- Disposables.swap() : Disposable 래퍼를 생성해서 기존 Disposable 을 자동으로 취소하고 다른 걸로 변경
  - 래퍼를 폐기하면 자체적으로 close => 현재 구현체와 이후에 대체하는 모든 구현쳬를 폐기
- Disposables.composite() : 여러 Disposable 을 수집해서 나중에 한번에 폐기할 수 있다
  - composite 의 dispose() 메소드를 호출하고 나면 다른 Disposable 을 추가할 때마다 즉시 폐기

