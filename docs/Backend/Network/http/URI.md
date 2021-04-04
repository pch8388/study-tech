# URI
- Uniform Resource Identifier
- URL 과 URN 을 포함하는 개념
- Uniform : 리소스를 식벽하는 통일된 방식
- Resource : 자원, URI 로 식별할 수 있는 모든 것(제한 없음)
- Identifier : 다른 항목과 구분하는 데 필요한 정보

## URL
- Uniform Resource Locator
- 리소스가 있는 위치를 지정
- 변경가능
- URL 은 거의 URI 와 같은 의미로 사용되고, 많은 프로그래밍 언어에서 관련 인터페이스를 정의할 때, 섞어 쓰기도 함

### URL 문법
- scheme://[userinfo@]host[:port][/path][?query][#fragment]
- https://www.google.com:443/search?q=hello&hl=ko
  - 프로토콜 : https
  - 호스트명 : www.google.com
  - 포트번호 : 443 => 일반적으로 생략 (well known port)
  - 패스 : /search
  - 쿼리 파라미터 : q=hello&hl=ko
- 리소스 경로(path)는 계층적 구조를 가짐
- 쿼리 파라미터는 key=value 의 형태로 정의하며, 문자의 형태로 전송됨
- fragment : html 내부 북마크 등에 사용되며 서버에 전송되지 않음

## URN
- Uniform Resource Name
- 리소스에 이름을 부여
- 변하지 않음
- urn:isbn:12312190008 (어떤 책의 isbn URN)
- 이름만으로 실제의 리소스를 찾을 수 있는 방법이 보편화 되지 않음

[참고강의](https://www.inflearn.com/course/http-%EC%9B%B9-%EB%84%A4%ED%8A%B8%EC%9B%8C%ED%81%AC)