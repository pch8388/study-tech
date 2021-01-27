# Spring redis
- Spring 의 세션 클러스터링을 redis 로 관리하는 방법 정리

## 디펜던시 추가
- maven
```xml
<dependency>
  <groupId>org.springframework.session</groupId>
  <artifactId>spring-session-data-redis</artifactId>
</dependency>
```
- gradle
```gradle
// spring-data-redis
compile('org.springframework.boot:spring-boot-starter-data-redis')
//embedded-redis (H2 와 같은 내장 redis 데몬)
compile group: 'it.ozimov', name: 'embedded-redis', version: '0.7.2'
```

## redis 설치
- 외부의 redis 를 사용하려면 미리 설치를 해야함
- docker 로 설치할 수 있음 
  ```shell
  $ docker pull redis
  $ docker run --name some-redis -d -p 6379:6379 redis
  $ docker exec -it 컨테이너id redis-cli # redis cli 접근
  ```
[redis docker 설치 참고](https://jistol.github.io/docker/2017/09/01/docker-redis/)

## application.properties 설정
```properties
spring.session.store-type=redis
server.servlet.session.timeout= # 세션타임아웃 설정(접미사 없으면 초단위)
spring.session.redis.flush-mode=on_save # 세션이 언제 저장소에 저장될지를 결정
spring.session.redis.namespace=spring:session # 세션을 저장하는데 사용할 키(네임스페이스)
```

## redis connection 설정
- 스프링 부트는 RedisConnectionFactory 를 자동생성 : 기본포트 6379 의 localhost 서버
- 환경설정
  ```properties
  spring.redis.host=localhost # Redis server host.
  spring.redis.password= # Login password of the redis server.
  spring.redis.port=6379 # Redis server port.
  ```

## 서블릿 컨테이너 Initialization
- 스프링 부트는 springSessionRepositoryFilter 라는 이름의 빈을 생성하는데(Filter 구현체)
이 빈은 HttpSession 을 Spring Session 의 커스텀 구현체로 대체 시켜준다.

[참고예제](https://www.baeldung.com/spring-session)

[참고예제](https://github.com/eugenp/tutorials/tree/master/spring-security-modules/spring-session)

[세션 클러스터링 예제](https://skasha.tistory.com/29)