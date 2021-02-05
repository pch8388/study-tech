# HTTP 특징

## 클라이언트 서버 구조
- 클라이언트는 서버에 요청을 보내고, 서버는 요청에 대한 결과를 만들어 응답

## 무상태 프로토콜
- 상태를 유지하지 않음으로 확장성을 얻음
- 상태를 유지해야 하는 경우도 있음 -> 로그인이 필요한 경우 -> 상태 유지를 최소한만 사용

## 비연결성
- 일반적으로 초 단위 이하의 빠른 속도로 응답
- 1시간 동안 수천명이 서비스를 사용해도 실제 서버에서 동시 처리하는 요청은 수십개 이하
- 서버 자원을 효율적으로 사용
- 항상 TCP/IP 연결을 새로 맺어야 하는 단점(3 way handshake)
  - 지속 연결(Persistent Connections)로 문제 해결
  - HTTP/2, 3 에서 더욱 최적화 됨

## HTTP 메시지
- HTML, TEXT, IMAGE, 음성, 영상, 파일, JSON, XML 등 거의 모든 형태의 데이터 전송 가능
- 시작라인, 헤더, 공백라인, 메시지 바디로 구성
  - [공식스펙](https://tools.ietf.org/html/rfc7230#section-3)
  
### 시작라인 
- 요청 : `GET /search?q=hello&hl=ko HTTP/1.1`
  - Method SP(공백) request-target SP HTTP-version CRLF(엔터)
- 응답 : `HTTP/1.1 200 OK`
  - HTTP-version SP status-code SP reason-phrase CRLF
  
### 헤더
- field-name":" OWS(띄어쓰기 허용) field-value OWS
- field-name 은 대소문자 구분 없음
- field-name 뒤에 ":" 는 공백없이 써야함
- 용도 : HTTP 전송에 필요한 모든 부가정보
- 필요시 임의의 헤더 추가 가능

### 메시지 바디
- 실제 전송할 데이터
- HTML 문서, 이미지, 영상, JSON 등 byte로 표현할 수 있는 모든 데이터 전송 가능

## 단순함, 확장 가능
- 단순하기 때문에 확장이 쉬움
- 단순하고 확장이 가능하기 때문에 널리 쓰임