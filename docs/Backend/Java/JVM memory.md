# JVM 구성요소
- 클래스 로더 시스템 : 컴파일된 자바바이트 코드를 읽어 메모리에 적재
  - 로딩 : 바이트 코드를 읽음
  - 링크 : 레퍼런스 연결
  - 초기화 : static 값 초기화 및 변수 할당
- 메모리
- 실행엔진
  - 인터프리터
  - JIT 컴파일러 : 반복되는 코드가 발견되면 모두 네이티브 코드로 바꿔둠
  - GC : throughput 위주, stop the world 를 줄이기 위해 최적화
- JNI : Java Native Interface, C 나 C++ 로 구현되어 있음
  - JNI 를 모은 라이브러리를 네이티브 메서드 라이브러리라고 함

## JVM 메모리 구조
- Stack : 메소드 콜 스택, thread 자원
- PC : PC register, 현재 실행위치를 나타냄, thread 자원
- native method stack : 네이티브 메소드 호출 스택, thread 자원
- heap : 객체정보, 공유자원
- method : 클래스 수준의 정보, 공유자원

## JDK 8 에서 Perm 영역 삭제
- JDK 8부터 Permanent Heap 영역 제거
  - Metaspace 영역이 native memory 에 추가
  - Perm 은 JVM 에 의해 크기가 강제되던 영역
- Metaspace 는 native memory 영역이기 때문에, OS 가 자동으로 크기 조절
  - 옵션으로 줄일 수 있음 `-XX:MaxMetaspaceSize`
  - 기존과 비교하여 메모리 영역을 많이 할당 받을 수 있음
    - Perm 영역의 크기로 인한 OOM 발생 감소

### Permanent 영역
- Class 메타정보나 메소드 메타정보, static 변수와 상소 정보 등이 저장되는 메타데이터 저장 영역
- Native 영역(Metaspace) 로 JDK 8 부터 이동되었지만, static object 는 GC의 대상이 될 수 있도록 Heap 영역으로 이동됨
- 이로 인해 JVM 메모리 옵션 튜닝은 Heap 영역만 신경쓰도록 되었음
  - JDK 7 : Heap, Perm 튜닝
  - JDK 8 : Heap 튜닝
- JDK 8 에서 Metaspace로 변경된 이유 : 필요한 크기를 예측하기가 매우 어려움