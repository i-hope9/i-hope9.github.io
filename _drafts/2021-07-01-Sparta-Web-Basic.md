---
layout: article
title: 웹 개발 기초 04. Flask 서버 만들기 (feat. 스파르타 코딩 클럽)
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


<!--more-->

## 참고자료
+ [스파르타 코딩클럽](https://spartacodingclub.kr/)
