# JDK 8 에서 Perm 영역 삭제
- JDK 8부터 Permanent Heap 영역 제거
  - Metaspace 영역이 native memory 에 추가
  - Perm 은 JVM 에 의해 크기가 강제되던 영역
- Metaspace 는 native memory 영역이기 때문에, OS 가 자동으로 크기 조절
  - 옵션으로 줄일 수 있음 `-XX:MaxMetaspaceSize`
  - 기존과 비교하여 메모리 영역을 많이 할당 받을 수 있음
    - Perm 영역의 크기로 인한 OOM 발생 감소

## Permanent 영역
- Class 메타정보나 메소드 메타정보, static 변수와 상소 정보 등이 저장되는 메타데이터 저장 영역
- Native 영역(Metaspace) 로 JDK 8 부터 이동되었지만, static object 는 GC의 대상이 될 수 있도록 Heap 영역으로 이동됨
- 이로 인해 JVM 메모리 옵션 튜닝은 Heap 영역만 신경쓰도록 되었음
  - JDK 7 : Heap, Perm 튜닝
  - JDK 8 : Heap 튜닝
- JDK 8 에서 Metaspace로 변경된 이유 : 필요한 크기를 예측하기가 매우 어려움