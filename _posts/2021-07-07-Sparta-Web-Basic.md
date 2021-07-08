---
layout: article
title: 웹 개발 기초 05. 프로젝트 구현, AWS 서버 배포 (feat. 스파르타 코딩 클럽)
tags: [web, html, css, javascript]
aside:
    toc: true
---

💡 본 글은 스파르타코딩클럽 '왕초보 시작반' 5주차 강의 내용을 바탕으로 정리하였습니다.

## 서버 배포
### 왜 AWS EC2를 사는가?
* 서버는 클라이언트의 요청에 항상 응답할 수 있도록 프로그램을 항상 실행해주어야 함
* 프로그램을 항상 실행하기 위해서는?
  + 컴퓨터가 항상 켜져 있어야 하고
  + 모두가 접근할 수 있는 공개 주소인 공개 IP 주소로 나의 웹 서비에 접근할 수 있도록 해야 함
* 보안 상의 문제로 개인 컴퓨터를 서버로 잘 사용하지는 않음, 클라우드 컴퓨터에 배포

### AWS EC2에 배포하기
#### EC2 구매
* Ubuntu: Linux 기반 운영체제, 오픈소스

#### 로컬에서 AWS EC2 인스턴스 접속하기
* `SSH` 사용: 원격 접속, 파일 전송 등을 할 때 보안이 뛰어난 프로토콜, 기본 포트는 22

```shell
ssh -i [.pem 파일경로] ubuntu@[공개IP주소]
```

#### python 파일 실행
* python 파일 전송
  + FileZilla, MobaXterm 같은 프로그램을 이용하거나, `sftp` 명령어를 사용해도 됨
* AWS 인바운드 보안 그룹 규칙 수정하기
  + 80: HTTPS 기본 포트
  + 27017: MongoDB 포트 (로컬에서 내 인스턴스 DB에 접속해서 데이터를 봐야할 수도 있으므로)
* `nohup` 명령어 실행: SSH 연결이 끊겨도 서버가 돌도록 설정
```shell
nohup python app.py &
```

#### 프로세스 종료
* `kill` 명령어: 서버(프로세스) 종료

```shell
-- 아래 명령어로 미리 pid 값(프로세스 번호)을 본다
ps -ef | grep 'app.py'

-- 아래 명령어로 특정 프로세스를 죽인다
kill -9 [pid값]
```

#### og 태그 만들기
* 카카오톡, 페이스북 등에 링크를 공유했을 때 페이지 이미지, 설명 등이 예쁘게 나오도록 하는 태그
* html `<head>` 태그 사이에 붙여 넣기

```javascript
<meta property="og:title" content="내 사이트의 제목" />
<meta property="og:description" content="보고 있는 페이지의 내용 요약" />
<meta property="og:image" content="{{ url_for('static', filename='ogimage.png') }}" />
```


***

<!--more-->
5주차까지 강의 듣고 마무리했다! 교육에서는 가비아에서 도메인 구입해서 연결하기까지 해봤는데, 글에는 생략했다. <br/>
교육 수강 후기 글은 나중에 다시 작성해볼까 생각 중이다.

## 참고자료
+ [스파르타 코딩클럽](https://spartacodingclub.kr/)
