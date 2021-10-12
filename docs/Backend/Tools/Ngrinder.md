# Ngrinder
성능 측정 오픈소스

## 실행 방법
1. 컨트롤러 설치
    1. [다운로드](https://github.com/naver/ngrinder/releases) 페이지에서 war 파일을 다운받는다.
    2. 다운받은 war 실행   
    ```java -jar ngrinder-controller-{version}.war --port=8300```
    3. admin 페이지 접속 : localhost:8300 => 위에서 지정한 포트번호
        -  초기 접속정보 : admin/admin
2. agent 설치
    1. 설치한 admin 페이지에서 우측상단 admin 클릭 -> 에이전트 다운로드
    2. tar 파일이 다운되는데, 압축을 풀어준다.  
      ```tar -xvf ngrinder-agent-3.5.5-p1-localhost.tar```
    3. 압축풀린 디렉터리로 이동 후 +agent 실행  
        ```
        window : .\run_agent.bat
        mac/linux : ./run_agent.sh
        ```
3. 1에서 설치한 admin 페이지에서 admin -> 에이전트 관리 선택하면 2에서 띄워둔 에이전트가 보인다. 승인됨을 클릭해준다
4. 스크립트를 작성하고 성능테스트를 실시한다

> 로컬에서 테스트 시 ip 를 적어주자. 도메인 네임을 hosts 에 등록하는 등의 방식은 인식을 못함

[main page](https://naver.github.io/ngrinder/)
