# DNS
- Domain Name Server
- IP 주소는 사용자에게 친화적이지 않기 때문에, 사람이 기억하기 쉬운 domain name 을 IP 주소와 따로 mapping
- DNS 는 계층구조로 되어 있고, 캐시 기능 등을 가지고 있음
- 가까운 DNS 부터 UDP 통신을 통해 요청하는 도메인의 IP 주소를 가지고 있는 지 확인
- IP 주소는 변할 수 있고, 클라이언트는 도메인 주소로만 접속하면 되고, 도메인 주소에 바뀐 IP를 mapping 하기만 하면 됨

[참고강의](https://www.inflearn.com/course/http-%EC%9B%B9-%EB%84%A4%ED%8A%B8%EC%9B%8C%ED%81%AC)