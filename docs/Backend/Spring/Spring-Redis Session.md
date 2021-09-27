# Spring redis
- Spring 의 세션 클러스터링을 redis 로 관리하는 방법 정리

## redis
[참고영상](https://youtu.be/mPB2CZiAkKM)
- redis 는 싱글쓰레드
  - O(n) 명령어를 조심해야 함
    - KEYS
    - FLUSHALL, FLUSHDB => 어쩔 수 없이 써야하는 경우가 있음
    - Delete Collections
    - Get All Collections
  - 오래 걸리는 작업이 쓰레드를 점유하면 다른 작업들이 타임아웃에 걸려 서버가 다운될 수 있음
- KEYS 는 scan 명령을 사용하는 것으로 하나의 긴 명령을 짧은 여러번의 명령으로 바꿀 수 있음
  - KEYS : 저장된 모든 키 호출
  - scan : 커서 방식으로 짧게 여러번 스캔
- Collection 의 모든 item 을 가져와야 하면 Collection 의 일부만 가져오거나(Sorted Set 큰 Collection 을 여러개의 작은 Collection 으로 저장

## redis 설치 및 실행
[참고블로그](https://littleshark.tistory.com/68)
```shell
$ docker pull redis
$ docker run --name test-redis -d -p 6379:6379 redis

## 네트워크 구성
$ docker network create redis-net
## 생성된 네트워크 리스트 확인
$ docker network ls

## 레디스 컨테이너 구동
$ docker run -d --name redis -p 6379:6379 --network redis-net redis

## redis-cli 접속
$ docker run -it --network redis-net --rm redis redis-cli -h redis
```


## Redis 특징(document)
[커맨드 목록](https://redis.io/commands)
- in-memory data structure store
- database, cache, maessage broker 로 사용됨
- 제공하는 data structures : strings, hashes, lists, sets, sorted sets with range queires

### Collections
- Strings : key-value
- List
  - 데이터 중간 삽입시 매우 느림(사용하면 안됨)
  - 앞뒤로 넣을 수 있음
  - job queue 에서 많이 씀
- Set : 데이터가 있는지 없는지만 체크하는 용도, 특정 유저를 follow 하는 목록 저장등에 쓰임
- Sorted Set
  - 정렬되는 set
  - strings 와 함께 제일 많이 쓰임
  - 범위를 지정해서 가져올 수 있음(index 범위) => score 가 double 타입이기 때문에, 값이 정확하지 않을 수 있음
- hash
  - key 밑에 sub key 가 존재

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