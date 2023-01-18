# 카프카 프로듀서 파티션
- 최신버전 (3.3.X) 에서는 기본 파티셔너들을 사용하지 않음 => default partitioner class = null
- 기존에는 default partitioner 에 의해 파티션을 지정했지만 이제는 지정된 파티셔너가 없다면 + 키가 없다면 RecordMetadata.UNKNOWN_PARTITION (-1) 를 반환하고, accumulator 가 어떤 파티션으로 보낼지 결정하는 로직을 직접(BuiltInPartitioner) 가지고 있음
- 키가 있다면 해쉬함수에 의해 파티션 지정(이전과 같은 방식)
- 파티션 로드 통계를 기반으로 토픽의 다음 파티션 계산 => 로드된 파티션이 있는지 체크 => 클러스터 정보로부터 사용 가능한 파티션이 있는 지 확인하여 있으면 그 중에 하나(랜덤) / 없으면 파티션 중 랜덤
- deprecated : DefaultPartitioner / UniformStickyPartitioner 

## 변경된 원인
UniformStickyPartitioner 는 시간기반(linger.ms) 혹은 배치사이즈(batch.size)에 의해 선택된 파티션이 요건에 만족하면 전송을 하는 형태이다. 문서에서는 특히 linger.ms=0 으로 설정된 경우에 대해 설명한다.
문제는 특정 브로커에 latency 가 생긴 경우(리더 변경이나 일시적 네트워크 문제일 수도 있고, 여타 다른 문제로 지연이 발생할 수 있음)인데, 프로듀서가 해당 브로커가 사용 가능해질때까지 해당 파티션에 대한 배치를 보유해야한다.
해당 배치를 유지하는 동안 추가 배치가 쌓이기 시작한다. 

느린 파티션과 빠른 파티션의 차이로 생각해보면 느린 파티션은 선택될때마다 배치로그가 쌓일 것이고, 빠른 파티션은 계속 정상 처리된다. 따라서 파티션끼리의 불균형이 발생하고, 쌓인 배치를 처리하는 동안에 이러한 불균형은 해결되지 않는다.

UniformStickyPartitioner 는 배치사이즈를 바이트로 조절하여 균일한 처리량을 달성하려고 했던 것인데, 특정 브로커가 느려지면 레코드가 accumaulator 에 쌓이고 결국 버퍼 풀 메모리가 소진되어 프로듀서가 느려진다.

## 변경된 구성
|옵션|기본값|설명|
|--|--|--|
|partitioner.class|null|기본파티셔너 지정|
|partitioner.ignore.keys|false|true 설정하면, 키가 있어도 기본파티셔너 동작이 수행된다|
|partitioner.adaptive.partitioning.enable|true|브로커에서 가용 파티션 정보를 받아서 더 빠른 파티션에 할당하도록 한다. false 면 임의의 파티션 선정|
|partitioner.availability.timeout.ms|0|지정된 시간동안 파티션에 대한 프로듀서의 요청을 수락할 수 없는 경우 해당 파티션을 사용할 수 없는 것으로 판단. 값이 0이면 비활성화. partitioner.adaptive.partitioning.enable=false 설정하여 임의의 파티션 설정을 한 경우에는 이 옵션을 무시한다|

> partitioner.class 를 지정하여 사용하면 위의 구성들을 무시한다

[참고](https://cwiki.apache.org/confluence/display/KAFKA/KIP-794%3A+Strictly+Uniform+Sticky+Partitioner)