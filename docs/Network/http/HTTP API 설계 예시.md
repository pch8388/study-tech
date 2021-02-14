# HTTP API 설계 예시
- HTTP API - 컬렉션
  - POST 기반 등록
- HTTP API - 스토어
  - PUT 기반 등록
- HTML FORM 사용

## 회원 관리 시스템 : POST 기반 등록
- 회원 목록 GET /members
- 회원 등록 POST /members
- 회원 조회 GET /members/{id}
- 회원 수정 PATCH,PUT,POST /members/{id}
  - PATCH : 수정할 데이터만 보내서 그 부분만 수정(대부분의 경우)
  - PUT : 데이터를 모두 보내서 완전히 덮어서 수정함
  - POST : 애매하다 싶으면 post
- 회원 삭제 DELETE /members/{id}
- 클라이언트는 등록될 리소스의 URI 를 모름
  - POST 등록시 resource uri 를 서버에서 만들어서 내려줌
    ```
    HTTP/1.1 201 Created
    Location: /members/100
    ```
- Collection
  - 서버가 관리하는 리소스 디렉터리
  - 서버가 리소스의 URI 를 생성하고 관리
  - 예제에서 컬렉션은 /members

## 파일 관리 시스템 : PUT 기반 등록
- 파일 목록 GET /files
- 파일 조회 GET /files/{filename}
- 파일 등록 PUT /files/{filename}
  - filename 이 있으면 수정하고, 없으면 등록한다 => PUT 이 의미적으로 맞음
- 파일 삭제 DELETE /files/{filename}
- 파일 대량등록 POST /files
  - 임의로 POST 로 지정할 수 있다
- 클라이언트가 리소스 URI 를 알고 있어야 한다.
- 클라이언트가 직접 리소스의 URI 를 지정
- Store
  - 클라이언트가 관리하는 리소스 저장소
  - 클라이언트가 리소스의 URI 를 알고 관리
  - 여기서 스토어는 /files
