# Reactor Netty
- spring boot 에서 reactor netty 를 구동하면 기본으로 core * 2 의 thread 를 가진 EventLoopGroup 을 생성한다
  - reactor.netty.ioWorkerCount : worker thread 개수 지정
  - reactor.netty.ioSelectCount : selector thread 개수 지정 (지정하지 않음녀 worker count 와 똑같이 지정됨)
  - 지정하지 않으면 ```Math.max(Runtime.getRuntime().availableProcessors(), 4))``` 를 호출한다. 즉, 코어의 개수가 너무 작으면 최소 4개는 할당한다
    - `availableProcessors` 를 호출하면 core * 2 의 숫자를 반환한다(native method)

## Reactor resource 생성
- ReactorResourceFactory -> LoopResources -> DefaultLoopResources -> event loop thead 생성
- ReactorResourceFactory -> ConnectionProvider -> DefaultPooledConnectionProvider -> http connection pool 생성
