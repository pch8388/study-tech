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

## 표현(Representation)
- 전송, 응답에 둘다 사용
- 서버에서 어떤 형태로 데이터를 내려줄지 알려줌
- Content-Type : 표현 데이터 형식 (ex: application/json)
- Content-Encoding : 표현 데이터의 압축 방식 (ex: gzip)
- Content-Language : 표현 데이터의 자연어 (ex: en)
- Content-Length : 표현 데이터의 길이(바이트 단위), Transfer-Encoding 사용시 같이 쓰면 안됨

## Content negotiation
- 클라이언트가 선호하는 표현 요청
- Accept : 미디어 타입
- Accept-Charset : 문자 인코딩
- Accept-Encoding : 압축 인코딩
- Accept-Language : 언어
- Content negotiation 헤더는 요청시에만 사용

### 우선순위
- Quality Values(q) 값 사용
- 0 ~ 1 클수록 높은 우선 순위를 가짐(생략하면 1)
- Accept-Language: ko-KR,ko;q=0.9,en-US;q=0.8
  - ko-KR => Quality Values 1 (생략)
  - ko => Quality Values 0.9
  - en-US => Quality Values 0.8
- 우선순위에 따라 서버에서 지원하는 언어 중에 포함된 것을 반환할 수 있게 요청한다.
- 구체적인 것을 우선함 => 좀 더 나타내는 폭이 좁은(명시적인) 요청을 우선순위로 한다.

## 전송방식
- 단순전송 : 요청 <-> 응답(Content-Length 를 지정 => 컨텐츠에 대한 길이를 알때), 한번에 요청하고 한번에 받음
- 압축전송 : 서버에서 gzip 등으로 응답을 압축(Content-Encoding: gzip 과 같이 압축 인코딩을 응답)
- 분할전송 : 서버에서 전송시 응답을 나눠서 전송 (Transfer-Encoding: chunked) => Content-Length 를 보내면 안됨
- 범위전송 : 범위를 지정해서 요청(Range) <-> 범위에 대해 응답(Content-Range)

## 일반정보
- From : 유저 에이전트의 이메일 정보 => 잘 사용하지 않음, 검색엔진 등에 정보 제공을 위해 사용
- Referer : 이전 페이지 정보, 유입경로 분석 등에 이용한다
- User-Agent : 클라이언트 애플리케이션 정보
- Server : 요청을 처리하는 origin(실제 응답을 요청하는) 서버의 소프트웨어 정보

## 특별한 정보
- Host : 요청한 호스트 정보(도메인) => 실제로 어떤 도메인에 요청을 한 건지에 대한 정보를 가지고 있어야 한다
- Location : 페이지 리다리렉션, 300번대 응답이면 이 헤더로 어디로 리다이렉션 해야할지 알려줌
- Allow: 405 응답을 내리면서 어떤 메소드를 응답하는 지 알려줌
  - 405 : 해당 메소드가 서버에서 지원하지 않을 때 응답하는 http status
- Retry-After : 503 응답일때 서비스가 언제부터 이용가능한지 알려주는 헤더
  - 503 : 서비스가 불가할 때 응답해주는 http status
  
## 인증
- Authorization : 클라이언트의 인증정보를 서버로 전달
- XXX-Authenticate : 401응답과 함께 어떤 인증방식을 사용해야 하는지 알려줌
  - 401: 인증이 되지 않았을 때 응답하는 http status
  [참고](https://github.com/pch8388/til/blob/master/docs/read-book/%EB%A6%AC%EC%96%BC%EC%9B%94%EB%93%9Chttp/2%EC%9E%A5.md#%EC%9D%B8%EC%A6%9D%EA%B3%BC-%EC%84%B8%EC%85%98)

[참고강의](https://www.inflearn.com/course/http-%EC%9B%B9-%EB%84%A4%ED%8A%B8%EC%9B%8C%ED%81%AC)