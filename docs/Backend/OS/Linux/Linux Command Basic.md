# Linux Command Basic
## echo
command line에 출력

```bash
$ echo Hello World
```

## pwd (Print Working Directory)
현재 위치를 알려줌

```bash
$ pwd
```

## cd (Change Directory)
디렉터리 위치 변경

```bash
$ cd .             # 현재 디렉터리
$ cd ..            # 한단계 상위
$ cd ~             # 현재 유저 home 디렉터리
$ cd -             # 이전에 있었던 디렉터리(previous)
```

## ls (List Directories)
디렉터리 및 파일 목록을 보여줌

```bash
$ ls                # 현재위치의 디렉터리 및 파일을 보여줌
$ ls /home/user     # 지정한 디렉터리의 파일 및 디렉터리를 보여줌
$ ls -a             # hidden file 까지 전부 보여줌
$ ls -l             # 디렉터리 및 파일의 상세정보를 리스트 형태로 보여줌(마지막 수정된 것이 제일 뒤로)
$ ls -al            # a + l 
$ ls -R             # 디렉터리 컨텐츠를 재귀적으로 보여줌(디렉터리 안까지 다 보기)
$ ls -r             # 정렬을 반전(reverse)해서 보여줌
$ ls -t             # 수정된 시간 순서로 정렬(최근의 것이 가장 앞)
```

## touch
파일 생성

```bash
$ touch myfile
```

## file
file 정보 확인 (리눅스는 확장자와 파일의 내용이 일치하도록 강제하지 않기 때문에 실제로는 확장자와 다른 내용이 들어있을 수 있음)

```bash
$ file banana.jpg
```

## cat
- file의 내용을 출력
- 여러개의 파일을 한번에 출력 가능
- 단순 출력이기 때문에 대용량의 파일을 읽기에는 적합하지 않음

```bash
$ cat dogfile birdfile
```

## less
- 대용량의 파일을 편하게 읽음
- more 와 비슷한 역할을 함
- page 에서 page 로 텍스트를 읽음

```bash
$ less /home/user/Documents/text1
```

- less 명령어로 file을 읽은 후 다음의 명령을 실행할 수 있다
  - q - 작업을 중단하고 shell 화면으로 복귀
  - Page up, Page down, up and down arrow - 화면을 위 아래로 이동
  - g - 시작점으로 이동
  - G - 끝으로 이동
  - /search - 검색하고 싶은 단어를 입력
  - h - help 창 불러오기

## history
명령어 history 보기

```bash
$ history

$ clear   # terminal commad line clear
```

command line 에서 ctrl - R 을 입력하면 history를 검색할 수 있다.(원하는 단어로)

## cp (Copy)
파일 복사

```bash
$ cp mycoolfile /home/user/Documents/cooldocs
```

- mycoolfile을 /home/user/Documents/cooldocs 로 복사한다.
- 와일드카드를 이용하여 여러파일을 복사할 수 있다.
  - \* : any string
  - ? : one character
  - [] : [] 안의 any character

현재 위치의 .jpg로 끝나는 모든 파일을 /home/user/Pictures 로 복사 : 

```bash
$ cp *.jpg /home/user/Pictures
```

-r 커맨드를 추가하여 recursively copy (디렉터리 안의 내용 모두 복사) :

```bash
$ cp -r Pumpkin/ /home/user/Documents
```

-i 커맨드 추가하여 동일한 파일명이 있으면 덮어쓸건지 (한번 더 확인)  :

```bash
$ cp -i mycoolfile /home/user/Pictures
```

## mv (Move)
file을 이동, cp와 유사한 command 와 기능

다음과 같이 file 이름 변경에 사용 가능 :

```bash
$ mv oldfile newfile
```

다른 디렉터리로 이동 :

```bash
$ mv file2 /home/user/Documents
```

하나 이상의 파일 이동 :

```bash
$ mv file_1 file_2 /somedirectory
```

디렉터리명 변경

```bash
$ mv directory1 directory2
```

-i 옵션으로 덮어쓰기  재확인

```bash
$ mv -i directory1 directory2
```

-b 옵션으로 동일 file 있을 경우 백업 후 move

```bash
$ mv -b directory1 directory2
```

## mkdir (Make Directory)
디렉터리 생성

동시에 여러개도 가능 :

```bash
$ mkdir mydir1 mydir2
```

-p 옵션으로 없는 디렉터리 모두 생성 :

```bash
$ mkdir -p /home/usr/mydir1/test
```

## rm (Remove)
파일삭제 - 리눅스는 윈도우의 휴지통 같은 기능이 없으므로 삭제시 주의해야함

```bash
$ rm file1
```

쓰기 권한이 없는 파일은 삭제시 재확인한다.

확인받지 않고 무조건 삭제하려면 -f 옵션을 추가 :

```bash
$ rm -f file1
```

확인받으려면 -i 옵션 추가 :

```bash
$ rm -i file1
```

디렉터리 삭제시에는 -r 옵션을 추가해야 삭제 가능 :

```bash
$ rm -r directory
```

또는 rmdir 명령을 통해 디렉터리 삭제 가능 :

```bash
$ rmdir directory
```

## find
파일 검색

-name 옵션으로 file 명 검색 :

```bash
$ find /home -name puppies.jpg
```

-type 옵션으로 file type 지정 검색 :

```bash
$ find /home -type d -name MyFolder   # -type d -> directory search
```

find 는 지정 디렉터리만이 아니라 서브 디렉터리까지 모두 검사한다.

- editor 로 찾은 파일 바로 열기

```bash
vim $(find /home/user -name "important*")
```

## help
도움말 검색

아래와 같은 형태로 bash command에 대해 도움말을 찾음 :

```bash
$ help pwd
$ pwd --help
```

## man

메뉴얼 페이지를 보여준다

```bash
$ man ls
```

## whatis

command 의 간략한 설명을 보여줌

```bash
$ whatis cat
```

## alias
자주 쓰는 키워드나 명령어등을 간략하게 표현할 수 있도록 별칭을 달아두는 명령

```bash
$ alias foobar='ls -al'
```

alias 는 reboot 하면 사라지기 때문에, reboot시에도 유지하고 싶으면 ***~/.bashrc*** 에 기록하면 된다.

alias를 삭제하려면 :

```bash
$ unalias foobar
```

## exit

shell 을 빠져나간다

```bash
$ exit
$ logout       # logout 한다.
```

[참고 사이트](https://linuxjourney.com/lesson/the-shell)