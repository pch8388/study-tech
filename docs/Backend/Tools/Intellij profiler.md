# 이슈
Profiler - CPU and Memory Live Charts 를 보려고 하면 connection 을 못하는 문제 발생

## 에러 메시지
RemoteService: Failed to retrieve application JMX service URL  

## 원인 분석
- JMX 에 연결할 수 없는 이슈가 발생
- Spring boot 의 경우 actuator 엔드포인트 검색을 위해 JMX 를 사용하는 데, profiler 에서도 JMX connector 를 찾으려 하기 때문에 연결하지 못하는 것으로 생각됨

[참고 이슈](https://youtrack.jetbrains.com/issue/IDEA-210665#focus=Comments-27-3544328.0-0)

## 해결
JVM 옵션 추가  
```-Djava.rmi.server.hostname=localhost```