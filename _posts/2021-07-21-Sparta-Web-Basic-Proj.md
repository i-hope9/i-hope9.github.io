---
layout: article
title: Python flask-restful 파일 분리, 커밋 시 API key 숨기기
tags: [web, html, css, javascript, python]
aside:
    toc: true
---

## 배경
사내 교육을 수강하면서 처음으로 Python(flask framework)으로 웹 개발을 해보았습니다. <br/>
교육에서는 모든 api를 `app.py`라는 한 파일 안에 작성했지만, 개인 프로젝트를 하려고 보니 파일을 분리할 필요를 느꼈습니다. 저처럼 처음 프로젝트를 하시는 분들께는 꽤 유용한 정보라고 생각하여 정리합니다😉.

> 🔗 도움 받은 링크는 아래에 있습니다.

## `flask-restful` 프레임워크
파일 분리를 위해 검색해보니 Flask로 Restful api를 개발하기 위한 확장(extension) 프레임워크, [Flask-RESTful](https://flask-restful.readthedocs.io/en/latest/)을 알게 되었습니다. 파일 분리도 이 프레임워크로 쉽게 할 수 있어서 import 했습니다.

> 📌 `flask-restful` 외에 `flask-restx`도 있습니다. [이 글](https://stackoverflow.com/questions/40165665/flask-restful-vs-flask-restplus)에서 `flask-restful`과 `flask-restx`의 대표적인 차이점을 알아볼 수 있습니다. 저는 Swagger 문서를 생성할 필요는 없어 `flask-restful`을 사용했습니다.

## 파일 분리 방법
### 서버 실행을 위한 파일
서버를 실행하기 위한 기본 파일 `app.py`입니다.

```python
from flask import Flask, render_template, make_response
from flask_restful import Resource, Api

app = Flask(__name__)
api = Api(app)

# 메인 페이지
class MainApi(Resource):
    def get(self):
        return make_response(render_template('index.html'))

api.add_resource(MainApi, '/')

if __name__ == '__main__':
    app.run('0.0.0.0', port=5000, debug=True)
```

* <small> flask와 flask_restful을 이용합니다.</small>
* <small> `make_response`, `render_template`은 반환 결과를 json이 아닌 html 소스로 보여주기 위해 사용했습니다.</small>

### API 처리 파일 추가
만약에 책 장바구니 관련 API를 `basket.py` 파일로 분리했다면, `basket.py` 소스는 대략 이렇게 됩니다.

```python
from flask import Flask, jsonify
from flask_restful import Resource, reqparse

# parse request data(i.e. querystring or POST form encoded data)
parser = reqparse.RequestParser()
parser.add_argument('itemId')


class BookBasketApi(Resource):
    # GET: 책 장바구니 목록 불러오기
    def get(self):
        ...
        return jsonify({'books': books})

    # POST: 책 장바구니에 추가하기
    def post(self):
        ...
        return jsonify({'msg': '책바구니에 책을 담았습니다✨'})

    # DELETE: 책 장바구니에서 삭제하기
    def delete(self):
        itemId = request.form['itemId']
        ...
        return jsonify({'msg': '책바구니에서 책을 뺐습니다💨'})

```

* <small>`parser`는 request data(query parameter, form data 등)를 파싱하기 위한 라이브러리입니다. 자세한 설명은 [문서](https://flask-restful.readthedocs.io/en/latest/quickstart.html#argument-parsing)를 참고하세요! </small>

그 다음, `app.py` 상단과 하단에 다음의 코드를 추가합니다.
``` python
  from basket import BookBasketApi
  ...
  api.add_resource(BookBasketApi, '/basket')
```

이렇게 하면 `/basket`이라는 url에 각각 get, post, delete 메소드로 api가 매핑됩니다. 각 자원을 다루는 api 파일을 분리하면 코드 관리하기에 더 편하겠죠?

## API 호출 Key 분리: config.py 파일 사용
이번에는 커밋할 때 API Key는 숨기기 위해 파일을 분리하는 방법입니다. 매우 매우 간단합니다. <br/>
`config.py`와 같이 분리할 파일을 생성합니다. 해당 파일에 API 호출 키를 선언합니다.

``` python
aladin_api_key = "..."
national_lib_key = "..."
```

그 다음, 키가 필요한 파일에 해당 파일을 import 해줍니다. 아래 예시(`config.aladin_api_key`)처럼 `파일명.이름`을 입력하면 매핑됩니다.
``` python
import config
...

payload = { 'TTBKey': config.aladin_api_key,
            'Query': keyword,
            'QueryType': 'title',
            'start': page,
            'maxResults': 50,
            'cover': 'mini',
            'output': 'JS',
            'Omitkey': 1
            }
```
이제 커밋할 때 `config.py` 파일은 ignore하고 커밋하면 됩니다. 꼭 API Key만이 아니더라도 데이터베이스 정보(url, 사용자 이름, 비밀번호) 등 숨겨야 할 값에 적용하시면 됩니다.

***
이정도면 개인 토이 프로젝트를 진행하기 위한 구조 작업은 어느 정도 된 것으로 보입니다. 생각보다 쉽고 간편하게 해결할 수 있었습니다. 다른 분들께도 도움이 되기를 바랍니다😎!
<!--more-->

## 참고자료
+ [스파르타 코딩클럽](https://spartacodingclub.kr/)
+ [Flask-restful에서 REST API 처리를 여러 개의 소스 코드 파일로 분리하기](https://skylit.tistory.com/286)
+ [Flask로 REST API 구현하기 - 2. 파일 분리, 문서화](https://justkode.kr/python/flask-restapi-2)
+ [How to Hide Your API Keys in Python](https://medium.com/black-tech-diva/hide-your-api-keys-7635e181a06c)
