# Consumer Group Protocol
- 스토리지는 브로커가 처리, 컴퓨팅은 컨슈머나 컨슈머 프레임워크(kafka streams, ksqlDB 등)가 처리
- 컨슈머 그룹은 컨슈머의 효율성과 확장성에 핵심적 역할

# Consumer group
- group.id 를 통해 지정 => 설정하면 모든 새 인스턴스가 그룹에 추가
- 파티션이 그룹의 인스턴스에 고르게 분배되어, 파티션 단위의 병렬 처리가 가능해진다 (단, 하나의 파티션은 하나의 컨슈머에서만 처리 가능)

# Group Coordinator
- 내부 카프카 토픽(__consumer_offsets)을 이용하여 그룹 메타데이터 추적

## Group Startup
1. 브로커에게 코디네이터 찾아달라고 요청 보냄 `FindCoordinator(groupId)`
  - 다수의 브로커로 클러스터링 된 상태에서 해당 그룹의 코디네이터는 특정 브로커에 생성되기 때문
2. 브로커는 응답으로 코디네이터 엔드포인트 전달
3. 그룹 코디네이터에서 그룹에 포함되고 싶다고 요청 전달 `JoinGroupRequest(susbsription)`
4. 코디네이터는 정상적으로 그룹에 포함된걸 알려줌 `JoinGroupResponse(memberId)`
  - 그룹리더에게는 모든 멤버의 목록과 구독 정보도 넘겨주어 그룹 리더가 파티션 할당 전략에 따라 파티션 할당을 수행할 수 있도록 도움을 줌
5. 그룹 리더는 그룹 멤버에게 파티션을 할당하고, 코디네이터에게 할당된 파티션 정보를 전달 `SyncGroupRequest(memberId, group partition assignments)`
  - 그룹 리더가 아닌 컨슈머들은 자신의 memberId만 전달
6. 코디네이터는 리더에게 받은 정보를 바탕으로 실제 할당된 파티션 정보를 각 컨슈머에게 전달 `SysncGroupResponse(assignment)`

## Group Coordinator Failover
- 메타정보를 저장하는 `__consumer_offsets` 도 다른 브로커에 복제가 되기 때문에, 그룹 코디네이터가 사용할 수 없는 상황이 되어도, 복제본 중 하나를 새로운 그룹 코디네이터로 변경한다.

# Rebalance 
리밸런스가 필요하면 그룹 코디네이터가 각 컨슈머 인스턴스에게 알린다 (`HeartbeatResponse` or `OffsetFetchResponse`)

## Stop-the-World Rebalance 
리밸런스 정책을 `JoinGroupRequest`-> `JoinGroupResponse` -> `SyncGroupRequest` -> `SysncGroupResponse` 의 과정으로 수행하면 몇가지 문제가 생길 수 있다
  - 컨슈머가 파티션 할당을 취소하고, 재할당을 했는데, 같은 파티션이 할당되었다면, 상당한 리소스의 낭비이다.
    - 새롭게 할당되는 컨슈머가 있다면, 해당 컨슈머에게 할당할 파티션만 이동시켜 문제를 회피한다.(`StickyAssignor`)
  - 리밸런스가 진행되는 동안 모든 처리를 일시 중지해야한다.
    1. 어떤 파티션 할당을 취소할 지 결정하고 해당 파티션을 제외한 나머지 파티션을 정상 처리(`CooperativeStickyAssignor`)
    2. 새로운 컨슈머에 해지된 파티션 할당


[참고](https://developer.confluent.io/learn-kafka/architecture/consumer-group-protocol/)