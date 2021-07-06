---
layout: article
title: 웹 개발 기초 04. Flask 서버 구현 (feat. 스파르타 코딩 클럽)
tags: [web, html, css, javascript]
aside:
    toc: true
---

💡 본 글은 스파르타코딩클럽 '왕초보 시작반' 4주차 강의 내용을 바탕으로 정리하였습니다.

## Flask 프레임워크
* 웹 프레임워크
* Django 보다 가벼운 대신 개발자가 많은 부분을 개발해야 함

> 🌐 [Flask 공식 문서](https://flask.palletsprojects.com/en/2.0.x/tutorial/)

### 기본 구조
* 프레임워크를 사용하려면 다음의 구조는 지켜주어야 함
* 서버 파일은 통상적으로 `app.py`라고 명명

```
📂project
  └📂static (image, css files)
  └📂templates (html files)
  └📄app.py
```

### 서버 구동을 위한 기본 코드
* 로컬의 5000 포트로 서버 시작
* http://0.0.0.0:8000 으로 This is home!이 화면이 출력되는지 확인

```python
from flask import Flask
app = Flask(__name__)

@app.route('/')
def home():
   return 'This is Home!'

if __name__ == '__main__':
   app.run('0.0.0.0',port=5000,debug=True)
```

### html 파일 불러오기
* Flask 내장 함수 `render_template()` 사용
* 📂templates 폴더 안의 html 파일을 읽어 반환

```python
from flask import Flask, render_template
app = Flask(__name__)

## URL 별로 함수명이 같거나,
## route('/') 등의 주소가 같으면 안됩니다.

@app.route('/')
def home():
   return render_template('index.html')

if __name__ == '__main__':
   app.run('0.0.0.0', port=5000, debug=True)
```

### REST API 작성
#### Get 메소드

```python
@app.route('/test', methods=['GET'])
def test_get():
   title_receive = request.args.get('title_give')
   print(title_receive)
   return jsonify({'result':'success', 'msg': '이 요청은 GET!'})
```

#### Post  메소드

```python
@app.route('/test', methods=['POST'])
def test_post():
   title_receive = request.form['title_give']
   print(title_receive)
   return jsonify({'result':'success', 'msg': '이 요청은 POST!'})
```

## 메타 태그(`meta tag`) 스크래핑
### 메타 태그란?
* HTML의 `<head></head>` 부분에 들어가는 정보
* 눈에 보이는 `<body>` 외에 사이트의 속성을 설명해주는 태그
* 메타 태그를 잘 활용하면 내 사이트 이미지 썸네일, 제목, 설명 등을 표시할 수 있음

### 메타 태그 스크래핑
* Beautiful Soup `soup.select_one(태그[속성]])`: 어떤 태그 중에서 특정 속성을 가진 요소를 가져오기

```python
# 메타태그 스크래핑하기 예시
title = soup.select_one('meta[property="og:title"]')
```

## 연습 프로젝트 진행
### API 설계
* 본격적인 개발을 하기 전, API를 설계
* 작성할 내용
  + 주요 기능
  + 요청 정보: 요청 URL, 요청 방식
  + 서버가 제공할 기능
  + 응답 데이터

### 개발 절차
* 서버-클라이언트 연동 먼저 확인
  + 클라이언트에서 서버 측 API가 호출 되는지
  + 클라이언트의 데이터가 서버로 잘 넘어 오는지
  + 서버의 응답이 클라이언트에게 잘 넘어 가는지
* 서버 개발
  + API 별로 해야하는 로직 구현
  + 필요하다면 DB와 연동, CRUD
* 클라이언트 개발
* 완성한 기능 확인

## 숙제
* 지난 번에 만든 쇼핑몰 프론트엔드에 서버 붙이기
* 주문 등록 API, 주문 목록 불러오기 API 개발
* 예시 페이지에 프론트엔드 유효성 검사도 있길래 구현했는데 이번 범위는 아니었나 봄

![스파르타코딩클럽_4주차_숙제](/assets/images/posts/2021-07-06_homework.png) <br/>


***
파이썬으로 웹 서버 개발은 처음 해봤는데, 확실히 가볍고 빠르게 서버를 구현할 수 있다고 느꼈다. 그래도 스프링으로 RESTful API를 구현해봐서 큰 틀은 쉽게 이해할 수 있었다. 같은 내용을 연습, 숙제하며 스파르타처럼 반복하다 보니 익숙해졌다😂
<!--more-->

## 참고자료
+ [스파르타 코딩클럽](https://spartacodingclub.kr/)
