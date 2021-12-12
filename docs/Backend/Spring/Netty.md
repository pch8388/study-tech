# Spring Reactor Netty
- spring boot 에서 reactor netty 를 구동하면 기본으로 core * 2 의 thread 를 가진 EventLoopGroup 을 생성한다
  - reactor.netty.ioWorkerCount : worker thread 개수 지정
  - reactor.netty.ioSelectCount : selector thread 개수 지정 (지정하지 않음녀 worker count 와 똑같이 지정됨)
  - 지정하지 않으면 ```Math.max(Runtime.getRuntime().availableProcessors(), 4))``` 를 호출한다. 즉, 코어의 개수가 너무 작으면 최소 4개는 할당한다
    - `availableProcessors` 를 호출하면 core * 2 의 숫자를 반환한다(native method)

## Reactor resource 생성
- ReactorResourceFactory -> LoopResources -> DefaultLoopResources -> event loop thead 생성
- ReactorResourceFactory -> ConnectionProvider -> DefaultPooledConnectionProvider -> http connection pool 생성

# 이벤트 루프 동작 방식 
- 네티의 이벤트는 채널에서 발생
- 이벤트 루프 객체는 이벤트 큐를 가지고 있음
- 네티의 채널은 하나의 이벤트 루프에 등록됨
  - 하나의 이벤트 루프가 여러개의 채널을 처리할 수 있음 (채널:이벤트루프 = N:1)
  - 채널에서 발생한 이벤트는 항상 동일한 이벤트 루프에서 처리된다 => 이벤트의 처리 순서 보장

# 비동기 I/O 처리
- 네티는 비동기 I/O 처리를 위해 두 가지 패턴 제공
  1. 리액터 패턴의 구현체인 이벤트 핸들러
  2. Future 패턴
