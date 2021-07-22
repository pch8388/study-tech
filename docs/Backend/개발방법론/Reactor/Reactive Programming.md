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
