# HTTP header
- header-field = field-name":"OWS field-value OWS (OWS : 띄어쓰기 허용)
- field-name 은 대소문자 구문 없음
```
GET /search?q=hello
Host: www.google.com
```

## 용도
- HTTP 전송에 필요한 모든 부가정보 기술
- 필요시 임의의 헤더 추가 가능

## 분류
- General 헤더 : 메시지 전체에 적용되는 정보
- Request 헤더 : 요청정보
- Response 헤더 : 응답정보
- Entity 헤더 : 엔티티 바디 정보

## RFC723x 변화
- 엔티티 -> 표현(Representation)
- 표현 : 표현 메타데이터 + 표현 데이터
- 메시지 본문(payload) 를 통해 표현 데이터 전달