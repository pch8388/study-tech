# Webflux로 막힘없는 프로젝트 만들기
- spring mvc 가 아닌 webflux 를 선택하게 된 배경
  - 외부 리소스를 사용하는 로직이 있을 때 thread가 blocking 되어 자원낭비가 심하였고, 이러한 외부 리소스를 사용하는 경우가 많았음
- webflux는 event loop 를 이용해 thread를 효율적으로 사용한다
  - mvc처럼 thread pool 에 사용가능한 thread가 없는 상황이 발생하지 않음

## Event loop
- event -> event loop -> handler
- netty의 evnet loop는 single thread로 동작
- evnet는 queue로 처리

### Life cycle
event monitor -> event process -> runnable process(queue의 작업을 take)

## Reactor Meltdown
- event loop에 blocking call이 발생하면 single thread인 event loop가 병목의 원인이 될 수 있다
- blocking call에 대해 따로 스케줄러를 할당하여 event loop로부터 격리된 thread를 사용하여 해결한다
  - `Schedulers.boundedElastic`을 사용하는 것을 권장

## block hound
- blocking call 이 발생하는 것을 감지해주는 라이브러리

[발표 영상](https://if.kakao.com/session/107?t_src=tallkch_in_rcmd_sesssionclick)