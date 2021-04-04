# Port
- 하나의 ip 로 여러 프로세스의 네트워크와 연결하기 위해 port 번호를 서비스마다 할당
- TCP 헤더에 송/수신 port 번호가 첨부되어 있음
- 0 ~ 65535 할당 가능 (2^16)
- 0 ~ 1023 : well known port
  - FTP : 20, 21
  - TELNET : 23
  - HTTP : 80
  - HTTPS : 443

[참고강의](https://www.inflearn.com/course/http-%EC%9B%B9-%EB%84%A4%ED%8A%B8%EC%9B%8C%ED%81%AC)