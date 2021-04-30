# User Management
## Users and Groups
- linux 의 유저와 그룹은 권한관리를 위해 존재한다.
- 유저는 각각의 home 디렉터리를 가진다. ( 일반적으로, /home/username 의 형태 )
- 시스템 데몬 등도 모두 user 이다.
- User name은 유저를 식별하기 친숙한 식별자이지만, 시스템은 UID(user ids) 를 사용하여 유저를 관리한다.
- 시스템은 groups 를 사용하여 권한을 관리하고, 유저를 포함시킬 수 있다. 시스템은 group 을 GID(group id)로 식별한다.
- root 권한은 시스템의 모든 파일을 제어하고, 프로세스 작동을 멈추는 등의 모든 권한을 갖는 데, 이것을 root 사용자로만 한정지은 것은 안정성의 보장 때문이다.
- sudo 커맨드와 함께 root 권한을 사용할 수 있다. (superuser do)

## root
- su( substitue users ) 는 유저를 변경하는 데, 유저이름을 입력하지 않으면 root로 간주 → 해당 유저의 패스워드를 알아야 한다.
- su 로 root 권한을 받아 작업하면,  중대한 실수를 할수도 있고, 누가 접속해서 권한을 받은지 알 수 없기 때문에, sudo로 일시적인 권한 부여를 해서 사용하는 것이 안전하다.
- /etc/sudoers 파일에 어떤 유저가 sudo 커맨드를 실행했는지 기록된다. visudo 커맨드를 이용하여 해당 파일을 수정할 수 있다.

## /etc/passwd
- 유저 이름은 실제로 유저를 구분하는 식별자가 아닌 UID로 구분된다. 어떤 UID가 맵핑되었는지 확인하려면 /etc/passwd 파일을 확인한다.
    1. 유저 상세정보 확인 ( 하나의 라인에 하나의 유저)
    2. 콜론( : ) 으로 각 필드를 구분
    3. user-name : password : user-id : group-id : GECOS field : home directory : user's shell
        - root:x:0:0:root:/root:/bin/bash
        - password 부가 설명
            - "x" - password 는 실제로는 /etc/shadow 파일에 저장됨
            - "*" - access 권한이 없음
            - " " - 비밀번호 지정되지 않음
        - user-id : 부여된 UID , root 유저는 기본적으로 0
        - GECOS field :  실제 이름이나 전화번호 등의 코맨트를 달기 위해 있음. 콤마로 구분

## /etc/shadow
- 유저 인증정보가 담겨 있음
- 읽기 위해 root 권한 필요

```bash
$ sudo cat /etc/shadow
```

- root : MyEPTEa$6Nonsense : 15000 : 0 : 99999 : 7 : : :
    1. username
    2. 암호화된 password
    3. 마지막 비밀번호 변경일 - 1970년 1월1일부터 일자로 계산, 0이면 다음 로그인때 암호 변경필요
    4. 사용자가 암호 변경을 위해 대기해야 하는 일자
    5. 사용자가 암호 변경을 해야하는 최대 일자
    6. 암호 만료되기 전의 일수
    7. 암호가 만료된 일수
    8. 계정 만료일
- 최근 배포판에서는 해당 파일에만 의존하지 않고 PAM(Pluggable Authentication Modules) 와 같은 것을 사용

## /etc/group
- 그룹 인증정보

```bash
$ cat /etc/group
```

- root : * : 0 : pete
    1. group name
    2. group password
    3. GID ( group ID)
    4. 유저 목록

## User Management Tools

```bash
# user 추가
$ sudo useradd bob
# user 삭제
$ sudo userdel bob
# password 변경
$ passwd bob
```

[참고](https://linuxjourney.com/lesson/users-and-groups)