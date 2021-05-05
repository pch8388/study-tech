# 1단계 - 지하철 노선 관리
## 요구사항
- [X]  지하철 노선 관련 기능의 인수 테스트 작성하기
- [X]  지하철 노선 관련 기능 구현 하기
- [X]  인수 테스트 리팩터링

### 지하철 노선 생성 request

```
POST /lines HTTP/1.1
accept: */*
content-type: application/json; charset=UTF-8

{
  "color": "bg-red-600",
  "name": "신분당선"
}
```

### 지하철 노선 생성 response

```
HTTP/1.1 201
Location: /lines/1
Content-Type: application/json
Date: Fri, 13 Nov 2020 00:11:51 GMT

{
  "id": 1,
  "name": "신분당선",
  "color": "bg-red-600",
  "createdDate": "2020-11-13T-0:11:51.997",
  "modifiedDate": "2020-11-13T09:11:51.997"
}
```

## 질문할 것
- request, response 미리 작성된 것을 보고 하였는 데, 실제 개발시에 설계 순서도 api 인터페이스 정의를 하고 그걸 바탕으로 인수테스트를 작성하는지?
- 인수테스트 작성시 feature 의 senario 들을 모두 테스트 생성 후 구현에 들어가는 것 vs senario 하나마다 테스트 구현 후 바로 성공하는 테스트를 작성
- 예외 처리 핸들러를 따로 구현하지 않았는 데, 이런 공통처리를 하는 시점이 언제인 것이 좋은지? 실무에서 초기에 미리 세팅을 하고 시작하는 지? 아니면 필요시 구현하는지?
- 단발성으로 쓸 상수를 메서드 내부에 선언해두고 사용하는지? ⇒ 메서드 내부에 둔다면 상수처럼 대문자 네이밍은 어떻게 생각하는지
- 사용자 입력으로 발생할 수 있는 모든 예외케이스에 대해 인수테스트 실패 케이스도 작성하는지?

## 피드백
- [path-parameter 가이드](https://github.com/rest-assured/rest-assured/wiki/Usage#path-parameters)
- [픽스처 관리](https://jobjava00.github.io/book/xunit/08/)
- [BDD 에 관한 마틴파울러의 글](https://martinfowler.com/bliki/GivenWhenThen.html)

## 3단계 질문
- 구간 등록 후 등록된 라인 정보를 그대로 돌려주었는데, 따로 구간 정보만 다시 주는 것이 좋은지? + 구간에 대한 request, response 를 따로 나누는 것이 좋다고 생각하는지?

### 3단계 피드백
- [Stream 반환에 대한 이펙티브 자바 글](https://jaehun2841.github.io/2019/02/16/effective-java-item47/)
- [Stream 반환에 대한 오라클 개발자 글](https://stackoverflow.com/questions/24676877/should-i-return-a-collection-or-a-stream/24679745#24679745)

[1주차 미션완료](https://github.com/next-step/atdd-subway-map/tree/pch8388)