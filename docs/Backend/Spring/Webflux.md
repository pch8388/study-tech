# Mono
```java
@Slf4j
@RestController
public class MonoController {

  @GetMapping("/")
  Mono<String> hello() {
    log.info("pos1");
    
    Mono<String> m = Mono.fromSupplier(this::generateHello).doOnNext(log::info).log();
    
    // 명시적으로 구독하면 여기서 한번 실행하고, 스프링이 한번 실행시킨다
    // m.subscribe();

    // block : publisher 가 제공하는 컨테이너에서 값을 꺼낸다
    // String msg2 = m.block();

    log.info("pos2");
    // log.info("pos3: {}", msg2);
    return m;
  }

  private String generateHello() {
    log.info("method generateHello()");
    return "Hello Mono";
  }

}
```

- Mono 를 생성하는 부분 (just, fromSupplier 같은..)을 지연 시키려면 fromSupplier 사용
- Mono 는 build 시점에 생성되어있고, subscribe 되는 순간 실행된다
- fromSupplier 를 사용하면 subscribe 가 되어야 Mono 도 생성한다
- defer 로 생성하면 지연되긴 하는데, fromSupplier 와 동작방식이 다른 듯 함 => onSubscribe 되기 전에 실행
- Spring 이 subscribe 를 자동으로 해주긴 하지만 따로 subscribe 를 명시적으로 해주면 해당 구문에서 바로 subscribe 되어 실행된 후, Spring 이 subscribe 하여 한번 더 실행된다
