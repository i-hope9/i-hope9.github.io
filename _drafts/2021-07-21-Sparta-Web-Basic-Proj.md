---
layout: article
title: Python flask-restful 파일 분리, 커밋 시 API key 숨기기
tags: [web, html, css, javascript, python]
aside:
    toc: true
---

> 💡 본 글은 스파르타코딩클럽 '왕초보 시작반' 개인 프로젝트를 진행하며 작성한 내용입니다.

## 배경
교육 수강 개인 프로젝트 진행 중에 알게 된 정보.
Python으로 웹 서버 개발은 처음 해봐서 파일 분리 조차 처음 해봤습니다. <br/>
도움 받은 링크는 아래에 정리했습니다.

## `flask-restful`
교육에서는 flask 프레임워크를 사용했는데, 파일 분리를 위해 알아 보다 보니 flask-restful 있더라.
> 📌 flask-restx 도 있었는데, 이 라이브러리는 swagger 문서를 자동으로 만들어주는 기능이 있습니다. 저는 그 기능까지는 필요 없어서 flask-restful을 사용했습니다.

## 파일 분리 방법

## API 호출 Key 분리: config.py 파일 사용
깃허브에 올릴 때 알게 된 분리 방법.

> 왜 숨겨야 할까요? 누군가 내 API key로 호출을 남용할 수 있기 때문에 숨겨야.


***
<!--more-->

## 참고자료
+ [스파르타 코딩클럽](https://spartacodingclub.kr/)
+ [Flask-restful에서 REST API 처리를 여러 개의 소스 코드 파일로 분리하기](https://skylit.tistory.com/286)
+ [Flask로 REST API 구현하기 - 2. 파일 분리, 문서화](https://justkode.kr/python/flask-restapi-2)
+ [How to Hide Your API Keys in Python](https://medium.com/black-tech-diva/hide-your-api-keys-7635e181a06c)
