# docker 연습
### nginx 컨테이너 설치

```bash
# 이미지 다운로드
docker pull nginx
# 컨테이너 생성
docker create -p 80:80 --name nx nginx
# 확인
docker ps -a
# 실행
docker start [container id]
```

- docker run 으로 실행 시 pull + create + start
  - 이미지가 있을 경우 pull 되지 않음
  - 컨테이너가 이미 실행중이면 실행할때마다 생성함

### 컨테이너 삭제

```bash
# 먼저 정지시킨 후
docker stop [container name]
# 삭제
docker rm  [container name]
```

```bash
# 도커 정보 확인
docker info

# 이미지의 정보 확인
docker inspect nginx
```

### 포트포워딩

```bash
docker run -d --name tc -p 80:8080 tomcat
```

### 컨테이너 내부 쉘 실행

```bash
docker exec -it tc /bin/bash
```

### 컨테이너 로그 확인

```bash
docker logs tc
 # stdout, stderr
```

### 호스트 및 컨테이너 간 파일 복사

```bash
# local -> container
docker cp <path> <to container>:<path>
# container -> local
docker cp <from container>:<path> <path>
# container -> container
docker cp <from container>:<path> <to container>:<path>
```

### 임시 컨테이너 생성

```bash
docker run -d -p 80:8080 --rm --name tc tomcat
# d : 데몬
# p : 포트포워딩
# rm : 컨테이너를 내리면 바로 삭제
# name : 컨테이너 이름 지정
```

### 컨테이너 전체 삭제

```bash
docker rm `docker ps -a -q`
```

### 환경변수 사용

```bash
docker run -d --name nx -e env_name=test1234 nginx
# e : 환경변수 전달
```

### mysql 환경변수 사용(연습)

```bash
# mysql password 를 환경변수로 전달
docker run --name ms -e MYSQL_ROOT_PASSWORD='!qhdkscjfwj@' -d --rm mysql
# mysql 을 배시로 실행하면서 패스워드는 입력한다
docker exec -it ms mysql -u root -p
```

### 볼륨 마운트 옵션 사용해 로컬 파일 공유

```bash
docker run -v <호스트 경로>:<컨테이너내 경로>:<권한>
# 권한
# ro: 읽기전용
# rw: 읽기 및 쓰기

docker run -d -p 80:80 --rm -v ~/docker-test:/usr/share/nginx/html:ro nginx
```

- 로컬 파일을 공유 함으로써, 컨테이너의 존재 여부와 상관없이 파일을 유지시킬 수 있다

### 도커파일 빌드

```bash
# dockerfile 이라는 이름으로 도커 빌드에 전달할 내용을 작성
FROM pythod:3.7

RUN mkdir /echo
COPY test_server.py /echo

CMD ["python", "/echo/test_server.py"]

# 현재 디렉터리의 파일 도커 빌드
docker build -t echo_test .
```

### Spring boot 도커 빌드 및 실행

```bash
# dockerfile 작성
FROM openjdk:11
ARG JAR_FILE=*.jar
COPY ${JAR_FILE} app.jar
ENTRYPOINT [ "java", "-jar", "/app.jar" ]

# 도커파일과 같은 위치에 빌드된 jar 파일을 복사한다
cp ...생략

# 도커 빌드
docker build -t <생성할 이미지 이름> .
# 도커 실행
docker run -d -p 8080:8080 --name <컨테이너 이름> <이미지 이름>
```