# Permissions
## File Permissions

```bash
# file 권한 확인
$ ls -l Desktop/
drwxr-xr-x 2 pete penguins 4096 Dec 1 11:45
```

- d | rwx | r-x | r-x
    - d → file type.     d 면 디렉터리 , - 면 일반 파일
    - 소유자 , 그룹, 그 외
    - r  : readable
    - w : writable
    - x : excutable
    - - : empty 권한없음

## Modifying Permissions
- chmod 로 권한 설정

```bash
# 권한 추가
$ chmod u+x myfile
# 권한 삭제
$ chmod u-x myfile
# 여러 그룹에 권한 추가
$ chmod ug+w
# 숫자로 권한 설정 ( 4: read, 2: write, 1: execute)
$ chmod 755 myfile
```

## Ownership Permissions
- file owner 설정

```bash
# 유저
$ sudo chown patty myfile
# 그룹
$ sudo chgrp whales myfile
# 유저+그룹
$ sudo chown patty:whales myfile
```

## Umask
- file 생성 시 기본 권한 설정

```bash
$ umask 021
# 대부분 022로 기본설정 되어있음
# permission numberic 설정에서 빼는 식으로 설정
```

- 설정을 유지하려면 .profile 을 수정해야함

## Setuid

```bash
$ ls -l /usr/bin/passwd
-rxsr-xr-x  1  root  root 47032 Dec 1 11:45 /usr/bin/passwd
# s -> SUID 파일을 실행한 사람이 root 권한을 가짐

# SUID 수정
$ sudo chmod u+s myfile
$ sudo chmod 4755 myfile
# numberic : 4를 권한앞에 붙이면 됨
# SUID 가 대문자 S 이면 실행권한 없음
```

## Setgid

```bash
$ ls -l /usr/bin/wall
-rwxr-sr-x 1 root  tty 19024 Dec 14 11:45 /usr/bin/wall
# s -> SGID 이 비트를 사용하면 프로그램이 해당 그룹의 구성원 인 것처럼 실행할 수 있습니다.

# SGID 수정
$ sudo chmod g+s myfile
$ sudo chmod 2555 myfile
# numberic : 2를 권한앞에 붙이면 됨
```

[참고](https://linuxjourney.com/lesson/file-permissions)