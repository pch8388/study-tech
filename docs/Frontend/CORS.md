# 동일 출처 정책(same-origin policy)
- 자바스크립트에서 다른 웹페이지로 접근(XMLHttpRequest) 시 같은 출처의 페이지만 접근 가능한 데, 이러한 것을 동일 출처 정책(same-origin policy)라고 함
- 동일 출처 정책을 회피하기 위해 CORS 를 사용

# CORS (Cross-Origin Resouce Sharing)
- 웹 브라우저에서 외부 도메인 서버와 통신하기 위한 방식을 표준화한 스펙
## preflight(사전요청)
- 요청하려는 URL 이 외부 도메인이면 웹 브라우저는 preflight 요청을 먼저 보냄
    ```
    Request header
    OPTIONS / HTTP/1.1
    Host: http://domainA
    Origin: http://domainB

    Response header
    HTTP/1.1 200 OK
    Access-Control-Allow-Origin: http://domainA
    ```
- 서버측에서 추가적으로 처리해줘야 함
    - 응답헤더에 다음과 같은 프로퍼티들을 추가해준다. 
        - Access-Control-Allow-Origin: 허용할 도메인
        - Access-Control-Allow-Methods: 허용할 http 메소드
        - Access-Control-Max-Age: preflight request 에 대한 응답을 캐시할 시간
        - Access-Control-Allow-Headers: 허용가능한 헤더 목록

## 클라이언트에서 처리
- 클라이언트에서 회피해야한다면 jsonp 방식을 사용해야 함
- static 리소스 파일들은 동일 출처 정책의 영향을 받지 않기 때문에 js 파일을 json 으로 바꿔주는 일종의 편법
