# Port
- 하나의 ip 로 여러 프로세스의 네트워크와 연결하기 위해 port 번호를 서비스마다 할당
- TCP 헤더에 송/수신 port 번호가 첨부되어 있음
- 0 ~ 65535 할당 가능 (2^16)
- 0 ~ 1023 : well known port
  - FTP : 20, 21
  - TELNET : 23
  - HTTP : 80
  - HTTPS : 443