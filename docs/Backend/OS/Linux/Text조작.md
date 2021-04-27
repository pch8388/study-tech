# Text 조작관련 명령어
## stdout (Standard Out)
- 기본 출력
- echo 의 out stream 을 input stream 으로 연결하여 file에 input

```bash
$ echo Hello World > peanuts.txt
```

- > 은 redirection operator 이고, standard output 을 리다이렉션 시킨다.
- 파일이 없으면 파일을 생성하고, 존재하면 덮어쓴다.
- >> 연산자를 사용하면 덮어쓰지 않고, 파일의 내용을 이어쓴다.(없으면 생성)

```bash
$ echo Hello World >> peanuts.txt
```

## stdin (Standard In)
- 기본 입력
- < 은 stdin redirection

- peanuts.txt 의 stdin 을 받아 banana.txt 로 stdout

```bash
$ cat < peanuts.txt > banana.txt
```

- 위와 같이 stream 을 리다이렉션하여 사용가능

## stderr (Standard Error)
- 기본 출력(stdout) 과는 달리 error message를 출력하는 stream이 따로 있다.

```bash
$ ls /fake/directory > peanuts.txt
```

- 위와 같이 존재하지 않는 경로의 ls(list) 출력 스트림을 file에 쓰려고 하면 아래와 같은 에러가 발생한다.

```bash
ls: cannot access /fake/directory: No such file or directory
```

- 하지만 peanuts.txt 의 내용을 확인하면 아무것도 쓰여있지 않다.
- file descriptor 를 사용하여 error 내용을 출력할 수 있다.
- file descriptor → 음수가 아닌 정수로 파일이나 stream에 접근 ( 0 : stdin , 1 : stdout , 2 : stderr )


- stderr 을 file 로 출력

```bash
$ ls /fake/directory 2> peanuts.txt
```

- ls 결과를 peanuts.txt file 로 보내고, stderr 을 2>&1 를 통해 stdout 으로 리다이렉션
- 순서가 중요 ( ls 결과를 stdout으로 출력 후, 2(stderr)을 > 1(stdout)으로 리다이렉션한다.

```bash
$ ls /fake/directory > peanuts.txt 2>&1
# 좀 더 줄여서 쓸 수 있음
$ ls /fake/directory &> peanuts.txt
# 출력된 에러를 버린다(/dev/null 로 리다이렉트하면 출력이 버려짐)
$ ls /fake/directory 2> /dev/null
# 표준출력과 표준에러를 분리하여 파일로 저장
$ ls /fake/directory 1>output.txt 2>error.txt
```

## pipe and tee
- 연속적인 작업을 pipeline 으로 이어서 실행하도록 할 수 있다.

```bash
# ls -al /etc의 stdout을 가져와서 less 명령으로 파이프 처리
$ ls -al /etc | less
```

- |  operator 는 표준출력을 가져와 표준입력을 다른 프로세스로 만들 수 있게 한다.
- 두 개의 다른 output stream 을 사용하려면 tee 명령어를 사용

```bash
# ls 결과가 화면에 출력되고, peanuts.txt 로 ls 의 결과가 출력됨
$ ls | tee peanuts.txt
# 두 개의 파일에 적용
$ ls | tee peanuts.txt banan.txt
```

## env (Environment)
- 환경설정에 관한 커맨드

```bash
# 설정된 환경변수 전부 출력
$ env
# 환경변수 확인 $변수명
$ echo $HOME
$ echo $PATH
```

## cut
- file 에서 text 일부를 추출

```bash
# 학습을 위해 먼저 text file 생성(lazy와 dog 사이에 tab을 넣는다)
$ echo 'The quick brown; fox jumps over the lazy  dog' > sample.txt

# 5번째 character 추출
$ cut -c 5 sample.txt
# 필드를 구분자로 구문을 나눈다(기본값이 tab)
$ cut -f 2 sample.txt
# -d 옵션으로 구분자 지정
$ cut -f 1 -d ";" sample.txt
# 가져올 범위지정
$ cut -c 5-10 sample.txt
$ cut -c 5- sample.txt
$ cut -c -5 sample.txt
```

## paste
- file 내용을 읽어 행을 병합하여 출력

```bash
# 하나의 행으로 출력 (각각의 행마다 tab으로 구분되어 보여짐)
$ paste -s sample2.txt
# -d 옵션으로 구분자를 변경
$ paste -d ' ' -s sample2.txt
```

## head
- 라인수를 지정하여 읽어올수 있다.(상위항목부터)

```bash
# 옵션없이 쓰면 위에서부터 10라인을 읽음
$ head /var/log/syslog
# -n 옵션으로 읽을 라인수 지정
$ head -n 15 /var/log/syslog
# -c 옵션으로 읽을 character 수 지정
$ head -c 15 /var/log/syslog
```

## tail
- 라인수를 지정하여 읽어올수 있다.(하위항목부터)

```bash
# 옵션없이 쓰면 아래에서부터 10라인을 읽음
$ tail /var/log/syslog
# -n 옵션으로 읽을 라인수 지정
$ tail -n 15 /var/log/syslog
# -f 옵션으로 file 이 수정될 때마다 출력
$ tail -f /var/log/syslog
```

## expand and unexpand
- TAB 으로 띄워진 공백을 일반 공백으로 변경

```bash
$ expand sample.txt
# 결과물 redirect
$ expand sample.txt > result.txt
# 일반 공백을 tab으로 변경
$ unexpand -a result.txt
```

## join and split
- join 은 여러개의 파일에서 text를 합쳐준다.

```bash
# test를 위해 2개의 file 생성
file1.txt
1 John
2 Jane
3 Mary

file2.txt
1 Doe
2 Doe
3 Sue

# 실행
$ join file1.txt file2.txt

# filed 1,2,3 을 통해 결합된다.
```

```bash
# 다른 유형의 file 생성
file1.txt
John 1
Jane 2
Mary 3

file2.txt
1 Doe
2 Doe
3 Sue

# 위와 같이 실행하면 에러가 발생한다
# join 할 필드를 정해줘야 한다.
$ join -1 2 -2 1 file1.txt file2.txt
```

- -1 은 file1.txt(첫번째 refer), -2는 file2.txt(두번째 refer) 에서 몇 번째 필드를 join할 것인지 정한다.

```bash
# file을 다른 file로 분할
$ split somefile
```

- 기본값으로 1000 line 에 도달하면 파일을 분할한다. file 명은 x**를 기본값으로 생성된다.

## sort
- 파일 내용 정렬

```bash
$ sort file.txt
# -r 옵션으로 역순 정렬 (reverse)
$ sort -r file.txt
# -k 옵션으로 정렬할 필드 지정(-k 2 -> 두번째 열을 기준으로 정렬)
$ sort -k 2 file.txt
```

## tr (Translate)
- 특정 형식의 텍스트를 다른 형식으로 변환

```bash
# 대문자를 소문자로 변환
$ tr a-z A-Z
hello
HELLO
# -d 옵션으로 삭제
$ tr -d ello
hello
h
```

## uniq (Unique)
- 중복되지 않은 유일한 텍스트를 볼 수 있게 도와주는 명령
- 인접하지 않은 텍스트에 대해서는 중복체크를 하지 않음

```bash
# test file 생성
test.txt
book
book
paper
paper
article
article
magazine

$ uniq test.txt
book
paper
article
magazine

# 중복 갯수 확인
$ uniq -c test.txt
2 book
2 paper
2 article
1 magazine

# 유니크한 값 출력
$ uniq -u test.txt
magazine

# 중복 값만 출력
$ uniq -d test.txt
book
paper
article

# 인접하지 않은 값이면 중복제거를 하지 않음
reading.txt
book
paper
book
paper
article
magazine
article
# uniq 를 써도 같은 값
# 정렬후 중복제거한다.
$ sort reading.txt | uniq
article
book
magazine
paper
```

## wc and nl
- wc (word count) 는 file안의 단어의 총 갯수를 보여준다.

```bash
$ wc /etc/passwd
96    265    2925   /etc/passwd
#라인수(-l), 단어수(-w), 바이트 수(-c) 괄호안의 옵션으로 해당 하는 것만 볼 수 있음
```

- nl (number lines) 는 file안의 라인 번호를 보여준다.

```bash
# test용 파일생성
file1.txt
i
like
turtles

# 출력
$ nl file1.txt
1. i
2. like
3. turtles
```

## grep
- 패턴 매칭을 통해 file 내용 검색

```bash
# fox가 있는 행을 반환
$ grep fox sample.txt
# -i 옵션 -> 대소문자 구분없이 매칭
$ env | grep -i User
# 특정 패턴 매칭
$ ls | grep '.txt$'
```

[참고](https://linuxjourney.com/lesson/stdout-standard-out-redirect)