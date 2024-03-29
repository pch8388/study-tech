[참고](https://godekdls.github.io/Reactor%20Core/introductiontoreactiveprogramming/)
링크를 읽고 정리한 글

- 리액터는 리액티브 프로그래밍 패러다임의 구현체
# Reactive Programming
## 정의
리액티브 프로그래밍은 데이터 스트림과 변경 사항 전파에 초점을 둔 비동기 프로그래밍 패러다임

## 소개
- RxJava 는 JVM 위에서 실행하는 리액티브 프로그래밍 구현
- 자바 9 부터는 Flow 클래스로 통합
- 옵저버 패턴의 확장으로 사용되기도 함
- Iterable-Iterator 쌍과 성격이 유사 => pull 기반
- Publisher-Subscriber 쌍 => push 기반
  - 새로운 데이터가 있음을 Publisher 가 Subscriber 에게 통지(push)
  - push 받은 데이터에 적용할 연산은 선언형으로 표현
  - **프로그래머는 정확한 제어 흐름을 작성하는 대신 계산 논리를 표현한다**

```onNext x 0..N [onError | onComplete]```
- 이와 같은 접근법을 통해 0..N 개의 데이터를 유연하게 커버한다(N이 무한한 경우도 포함)

## Blocking 의 단점
- 블로킹은 리소스를 낭비
  - I/O 처리가 시작하면 스레드는 데이터를 기다리는 동안 아무일도 하지 않게 되고, 이로 인해 스레드를 많이 사용하는 방법을 사용해야하고, 스레드를 많이 사용하면 결국 컨텍스트 스위칭이 자주 일어나게 되어 cpu 오버헤드를 일으킨다
 
## 자바의 비동기 프로그래밍을 위한 모델
### Callbacks
- 리턴 값은 없지만 결과를 받으면 호출한 callback 파라미터를 추가로 받는 비동기 메소드
- 조합하기가 까다롭고 읽기가 힘들어 유지보수하기 어려운 코드를 만들 가능성이 높음

### Futures
- 곧바로 Future<T> 를 반환하는 비동기메소드. 비동기 프로세스는 T 값을 계산하고, 이를 래핑한 Future 객체로 접근
- Future 객체는 콜백보다 여러 면에서 낫다
  - 코드가 간결하여 알아보기 쉽고 행위를 추가하기도 콜백보다 쉽다
- Future 의 단점
  - get() 메소드를 호출하면 블로킹
  - lazy computation 미지원
  - 멀티 밸류에 대한 지원 부족, 에러 처리 커스텀 어려움

> 리액터는 조합을 위한 연산자가 Future 보다 많기 때문에 코드가 훨씬 간결해짐

## 리액티브 프로그래밍의 특징
- 쉽게 구성할 수 있고, 가독성이 좋음
- 데이터를 풍부한 연산자로 조작할 수 있는 플로우로 표현
- 구독하기 전에는 아무일도 일어나지 않음
- Backpressure : 컨슈머가 프로듀서에게 데이터 생산 속도가 너무 빠르다는 신호를 보낼 수 있음
- 고수준이면서 동시성에 구애받지 않을 정도의 높은 수준의 추상화

## Composability and Readability
- 여러 비동기 태스크를 쉽게 조율
  - 이전 태스크 결과는 다음 태스크의 입력으로 사용
  - 또는 fork-join 스타일로 여러 태스크 실행
  - 고수준 시스템에선 비동기 태스크를 별개의 컴포넌트로 재사용 가능
- 여러 태스크를 조율함으로써 가독성과 유지보수성 향상
  - 추상적인 처리 흐름을 반영할 수 있는 풍부한 구성 옵션 제공 => 중첩 최소화

## Operators
- 연산자는 Publisher 에 동작을 추가
- 이전 단계의 Publisher 를 새 인스턴스로 래핑
- 마지막엔 Subscriber 가 처리를 종료
- 연산자는 리액티브 스트림의 스펙이 아닌 리액터와 같은 라이브러리에서 제공하는 편의 기능이다

## Subscribe
- Publisher 체인을 구성해도 구독하지 않으면 아무일도 일어나지 않는다
- 구독을 해야 Publisher 와 Subscriber 가 연결되고, 전체 체인에 데이터 흐름이 트리거 된다
  - 내부적으로는 Subscriber 의 단일 요청 신호가 업스트림으로 전파돼 구독 중인 Publisher 로 전달

## Backpressure
- backpressure 를 구현할 때도 신호를 업스트림으로 전파
- Subscriber 는 언바운드 모드로 데이터 소스로부터 발생하는 모든 데이터를 제일 빠른 속도로 push 받거나, request 를 최대 n 개 처리할 준비가 되었다는 신호를 보낸다

### push - pull 모델
- 데이터가 있다면 다운스트림이 업스트림에 데이터 n 개를 pull 할 수 있음
- 요청 전 데이터를 미리 생성해 두는 비용이 크지 않으면 유리

## Hot, Cold 시퀀스
리액티브 스트림이 구독자에게 반응하는 방식으로 구분

[Hot vs Cold 자세한 설명](https://godekdls.github.io/Reactor%20Core/advancedfeaturesandconcepts/#92-hot-versus-cold)

### Cold 시퀀스
- 각 Subscriber 마다 데이터 소스를 포함해서 새로 시작
  - ex) 데이터 소스가 http request 를 래핑하고 있으면, 구독할 때마다 http request 를 새로 만듬
- 데이터 베이스에서 조회하는 것 같은 종류의 데이터

### Hot 시퀀스
- 각 Subscriber 마다 매번 처음부터 만들지 않음
- 나중에 구독한 구독자는 구독 이후 생산한 신호를 받음
- 일부 hot 리액티브 스트림은 생산 데이터 전체 혹은 일부를 캐시해두거나 이전 데이터를 재사용
- hot 시퀀스는 구독이 없을 때도 발생할 수 있음(이때는 예외적으로 구독하기 전에 아무일도 일어나지 않는다는 규칙이 적용되지 않음)
- 외부에서 발생하는 이벤트 정보 같은 것들