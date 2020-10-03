# 개요
- uri 는 정보의 자원을 표현
- 자원의 행위는 HTTP Method 로 표현

# 구성
- 자원(resource) - uri
- 행위(verb) - http method
- 표현(representations)

# 특징
1. Uniform(유니폼 인터페이스)
    - 메시지는 스스로 설명해야한다
    - HATEOAS : 애플리케이션 상태는 Hyperlink 를 이용해 전이되어야 함
2. Stateless(무상태성)
    - 상태정보를 따로 저장하거나 관리하지 않음
    - 서비스의 자유도가 높아지고, 구현이 단순해짐
3. cache
    - HTTP 가 가진 캐싱 기능 적용 가능
    - Last-Modified : 마지막 수정시간
    - E-Tag : 리소스에 임의의 태그값을 붙여 변경된 것을 확인
4. layered system(계층형 구조)
    - 다중 계층 구성 가능
5. self-descriptiveness(자체 표현 구조)
    - rest api 메시지만 보고도 이해 가능
6. client-server 구조
    - 클라이언트와 서버의 확실한 역할 분담

# uri 설계
- / 는 계층 관계를 나타낼 때 사용
- uri 마지막 문자로 / 를 포함하지 않음
- 불가피하게 긴 uri는 하이픈(-) 로 가독성 향상, _ 는 사용하지 않음
- 소문자 사용
- 파일확장자 포함하지 않음 (Accept head 에 사용)
- 리소스간의 관계표현
    ```
    소유(has) 관계 표현
    GET: http://test.com/users/{userId}/books
    서브 리소스에 명시적으로 표현이 필요할 때
    GET: http://test.com/users/{userId}/likes/books
    ```
- collection 은 복수, document(하나의 객체 혹은 문서) 는 단수로 표현

# HTTP Method
- GET : 리소스 조회
- POST : 리소스 생성
- PUT, PATCH : 리소스 수정, PUT(전체), PATCH(일부)
- DELETE : 리소스 삭제

# 상태코드
- 명확한 상태코드를 반환하여야 함
- 2xx : 정상수행
    - 200 ok
    - 201 created
- 3xx : 리다이렉트
    - 301 Moved Permanently
        - Location 에 리다이렉트 될 uri 를 포함해줘야 함
- 4xx : 요청 에러
    - 400 bad request
    - 401 unauthorized
    - 404 not found
- 5xx : 서버 에러
    - 500 internal server error