# G1 GC
Garbage First Garbage Collector
- 쓰레기로 가득찬 heap 영역을 집중적으로 수집
- 큰 메모리를 가진 멀티 프로세서 시스템에서 사용하기 위해 개발
- stop the world 시간을 최소화
- Java 9 부터 default gc
- realtime gc 가 아님. stop the world 를 최소화하긴 하지만 완전히 없애지 못함
- 통계를 계산해가며 gc 작업량 조절

## G1 GC 를 쓰면 좋은 상황
- Java heap 의 50% 이상이 라이브 데이터
- 시간이 흐르면서 객체 할당 비율과 프로모션 비율이 크게 달라진다
- GC 가 너무 오래 걸린다(0.5 ~ 1초)

## 다른 GC 와 비교
- Parallel GC
  - old gen 의 공간에서만 재확보와 조각 모음을 함
  - g1 은 이런 작업을 더 짧은 gc 작업들로 분배하여 수행, 전체적 처리량이 줄어드는 대신 stop the world 를 단축시킴
- CMS
  - g1 도 cms 처럼 old gen 영역을 동시에 작업
  - cms 는 old gen 조각 모음을 하지 않아 full gc 시간이 길어짐

## 작동 방식
G1 GC 의 힙 레이아웃은 다른 gc 와 좀 다르다. 전체 heap 을 여러 영역(region)으로 나누어 관리한다. G1 은 영역의 참조를 관리할 목적으로 remember set 을 만들어 사용한다. remember set 은 total heap 의 5% 미만 크기이다.
- 비어있는 영역에만 새로운 객체가 들어간다
- 쓰레기가 쌓여 꽉 찬 영역을 우선적으로 청소
- 꽉 찬 영역에서 라이브 객체를 다른 영역으로 옮기고, 꽉 찬 영역은 깨긋하게 비움
- 이렇게 옮기는 과정이 조각 모음 역할도 함
- 여러 영역을 차지하는 커다란 객체도 있음
- 병렬로 gc 작업 수행 => 각각의 스레드가 자신만의 영역을 잡고 작업



[참고](https://johngrib.github.io/wiki/java-g1gc/)