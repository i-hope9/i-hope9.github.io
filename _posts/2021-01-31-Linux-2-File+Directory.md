---
layout: article
title: Linux 기초 (2) 파일, 디렉터리 명령어
tags: [os, linux]
aside:
    toc: true
---

> 💡 본 글은 솔데스크 '리눅스 시스템 디자인 및 고급 활용 - LEVEL1' 강의 수강 내용을 바탕으로 정리하였습니다.

## 단순한 명령어
+ 현재 로그인한 계정
``` shell
[user@host]$ whoami
```

+ 현재 날짜와 시간
``` shell
[user@host]$ date
[user@host]$ date +%R
12:25
[user@host]$ date +%x
01/31/2021
```
`date` 명령어는 다음과 같이 다른 명령어에서도 사용 가능하다.
``` shell
[user@host]$ touch file $(date +%H-%M).txt
```

## 파일 관리
### 파일 내용 보기
+ `cat`: 페이지 없이 간단한 열람
    - 옵션 `-n`: 줄 번호 표
``` shell
[user@host]$ cat /etc/password
```

+ `less`: 페이지 단위로 열람
    - 화살표 키로 스크롤 가능
    - q를 눌러서 명령 종료
    - 옵션 `+G`: 문서 끝부터 열람
``` shell
[user@host]$ less /etc/password
[user@host]$ less +G /etc/password
```

+ `head`, `tail`: 각각 문서 시작, 끝부터 열람
    - 옵션 `-숫자`: 지정된 숫자의 줄 열람(지정 안 하면 10줄)
    - 옵션 `-f`: `tail`과 함께 쓰면 실시간으로 파일 끝 부분 열람
``` shell
[user@host]$ head /etc/password
[user@host]$ tail /etc/password
[user@host]$ tail -9 /etc/password
```

### 절대 경로와 상대 경로
+ 절대 경로
    - 루트(/) 디렉터리부터 시작해서 특정 파일에 도달하기까지의 하위 디렉터리를 모두 지정하는 방식
    - 슬래시(/)로 시작하면 절대 경로로 인식

+ 상대 경로
    - 사용자의 현재 위치(작업 디렉터리)를 참조하여 파일에 도달하기까지 필요한 경로만 지정하는 방식
    - 슬래시(/)가 아닌 문자가 첫 문자로 사용되면 상대 경로로 인식

> 💡 쉘에서 공백은 기본적으로 옵션 및 인수를 구분하는 용도이다. 파일 명에 공백이 있는 경우, 쉘은 이를 새 파일 이름이나 인수로 인식할 수 있다.
>따라서 파일 명에는 가능하면 공백을 포함하지 않는 편이 좋다. 공백이 있는 경우, 따옴표로 파일 명을 묶어서 표현하면 된다.

### 경로 탐색
+ `pwd`: 현재 디렉토리
``` shell
[user@host]$ pwd
```

+ `ls`: 디렉터리 내용 나열
    - 옵션 `-l`: 긴 목록 형식(속성, 사용권한, facl설정유무, 링크수, 소유자, 그룹, 파일 크기, 수정일자, 파일 이름)
    - 옵션 `-a`: 숨겨진 파일 포함하여 표시
    - 옵션 `-R`: 모든 하위 디렉터리 내용을 포함하여 표시
    - 옵션 `-h`: file의 용량 값을 KB, MB, GB 등 사람이 읽기 좋게(human-readable) 표시

``` shell
[user@host]$ ls
[user@host]$ ls -al
```

+ `cd`: 디렉터리 이동
    - 옵션 `-`: 이전 디렉터리로 바로 이동
``` shell
[user@host]$ cd /home/user/Documents
[user@host]$ cd -
```

### 파일 관리
+ `mkdir`: 디렉터리 만들기
    - 중괄호 확장을 사용할 수 있음({file_1,file_2})
    - 옵션 `-p`: 현재 존재하지 않는 상위(parent) 디렉터리도 함께 생성
    - 옵션 `-v`: 디렉터리가 생성되었다는 메시지 출력
``` shell
[user@host]$ mkdir /home/user/Documents/new_directory
[user@host]$ mkdir -p /home/user/new_parent/new_directory
[user@host]$ mkdir -pv /sample/{file_1,file_2}
mkdir: created directory '/sample/file_1'
mkdir: created directory '/sample/file_2'
```

+ `cp`: 파일 복사
    - 대상 파일이 이미 있는 경우, 덮어 쓰여짐
    - 옵션 `-r`: 디렉터리와 디렉터리 내용을 다른 디렉터리에 복사할 때
``` shell
[user@host]$ cp file new_file
[user@host]$ cp -r directory new_directory
```

+ `mv`: 파일 이동 또는 이름 변경
``` shell
[user@host]$ mv file new_file
```

+ `rm`: 파일 및 디렉터리 삭제
    - 옵션 `-r`: 파일을 포함하는 디렉터리 제거
    - 옵션 `-i`: 파일을 제거하기 전에 대화형으로 확인 메시지 표시
    - 옵션 `-v`: 디렉터리가 삭제되었다는 메시지 출력(`-r`과 함께 쓸 때 유용)
``` shell
[user@host]$ rm file
[user@host]$ mkdir -p /nome/user/new_parent/new_directory
```
<!--more-->

---
