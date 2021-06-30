---
layout: article
title: 웹 개발 기초 03. 파이썬 기초, 웹 스크래핑 (feat. 스파르타 코딩 클럽)
tags: [web, html, css, javascript]
aside:
    toc: true
---

💡 본 글은 스파르타코딩클럽 '왕초보 시작반' 3주차 강의 내용을 바탕으로 정리하였습니다.

## Python 기초
### 변수 선언
변수형을 따로 선언할 필요 없이 변수명으로만 바로 선언 가능

```python
first_name = 'taco'
last_name = 'kim'
```

### 리스트, 딕셔너리
```python
# 리스트
fruits = ['사과', '배', '수박']
fruits.append('귤')

# 딕셔너리
a_dictionary = {'name':'bob', 'age': 27}
print(a_dictionary['name'])
a_dictionary['height'] = 178
```

### 함수
```python
def 함수이름(파라미터, ...):
  return 반환값
```

### 조건문
```python
def is_adult(age):
    if age > 20:
        print('성인입니다')
    else:
        print('청소년입니다')

is_adult(30)
is_adult(15)
```

### 반복문
```python
# 배열 안의 요소를 하나씩 꺼내서 사용
fruits = ['사과','배','배','감','수박','귤','딸기','사과','배','수박']

for fruit in fruits:
    print(fruit)

# 딕셔너리 안의 요소를 하나씩 꺼내서 사용
people = [{'name': 'bob', 'age': 20},
          {'name': 'carry', 'age': 38},
          {'name': 'john', 'age': 7},
          {'name': 'smith', 'age': 17},
          {'name': 'ben', 'age': 27}]

for person in people:
    print(person['name'], person['age'])
```

## 파이썬 패키지 설치하기
### 패키지와 라이브러리
* Python에서 패키지는 모듈(일종의 기능들 묶음)을 모아 놓은 단위
* 패키지의 묶음이 라이브러리

### 가상환경(Virtual Environment)이란?
* 같은 시스템에서 프로젝트마다 다른 패키지, 라이브러리를 써야한다고 할 때 프로젝트마다 격리된 실행 환경을 지원하는 기능

> 📌 자세한 사항은 [Pyhton doc 내용](https://docs.python.org/ko/3/tutorial/venv.html) 참고!

### 패키지 설치하기
* `🛠Settings` 들어가서 아래 `+` 버튼 누르고 원하는 패키지 설치

![스파르타코딩클럽_2주차_숙제](/assets/images/posts/2021-06-25_py_package.png) <br/>

### `requests` 패키지 사용
> 📌 [공식 문서](https://docs.python-requests.org/en/master/)

```python
import requests # requests 라이브러리 설치 필요

r = requests.get('http://openapi.seoul.go.kr:8088/6d4d776b466c656533356a4b4b5872/json/RealtimeCityAir/1/99')
rjson = r.json()

gus = rjson['RealtimeCityAir']['row']

for gu in gus:
    gu_name = gu['MSRSTE_NM']
    gu_mise = gu['IDEX_MVL']
    if gu_mise > 50:
        print(gu_name, gu_mise)
```

### `bs4` 패키지 사용: 웹스크래핑(크롤링)
> 📌 [공식 문서](https://www.crummy.com/software/BeautifulSoup/bs4/doc//)

* `request` 보내서 페이지 가져오기
* 가져온 페이지에서 원하는 정보 솎아내기

```python
import requests
from bs4 import BeautifulSoup

headers = {'User-Agent' : 'Mozilla/5.0 (Windows NT 10.0; Win64; x64)AppleWebKit/537.36 (KHTML, like Gecko) Chrome/73.0.3683.86 Safari/537.36'}
data = requests.get('https://movie.naver.com/movie/sdb/rank/rmovie.nhn?sel=pnt&date=20200303',headers=headers)

soup = BeautifulSoup(data.text, 'html.parser')

# 하나 가져오기
# select 경로는? 크롬 검사에서 오른쪽 마우스 copy selector
title = soup.select_one('#old_content > table > tbody > tr:nth-child(2) > td.title > div > a')

# 태그의 텍스트 가져오기
print(title.text)

# 태그의 속성 가져오기
print(title['href'])

# 여러 개 가져오기
trs = soup.select('#old_content > table > tbody > tr')
for tr in trs:
    a_tag = tr.select_one('td.title > div > a')
    if a_tag is not None:
        rank = tr.select_one('img')['alt']
        title = a_tag.text
        point = tr.select_one('td.point').text
        print(rank, title, point)
```

> 💡 스크래핑 해오는 방식은 각자 다 다름. 어떻게든 효율적으로 원하는 정보를 가져온다면 됨!

## Python - MongoDB 연동
### DB 접속을 위한 기본 코드
```python
from pymongo import MongoClient

client = MongoClient('localhost', 27017)  # 'localhost:27017' DB에 연결
db = client.dbsparta  # 이 이름의 DB에 접속, 없으면 생성
```

### DB CRUD
#### Create: `insert`
```python
# insert - db 안의 'users' collection 안에 하나 삽입
doc = {'name': 'jane', 'age': 21}
db.users.insert_one(doc)
```

#### Read: `find`

```python
# find
same_ages = list(db.users.find({'age':21},{'_id':False}))
for person in same_ages:
    print(person)

user = db.users.find_one({'name':'bobby'})
```

#### Update: `update`

```python
# update
db.users.update_one({'name': 'bobby'}, {'$set': {'age':19}})
db.users.update_many({'name': 'bobby'}, {'$set': {'age': 19}})
```

#### Delete: `delete`
```python
db.users.delete_one({'name':'bobby'})
```

## 숙제
* 지니 뮤직 순위 크롤링하기
* 크롤링 결과 내가 원하는 정보를 얻기 위해서는 문자열 다루는 함수를 활용해야 함

### Python 문자열 다루기
#### 문자열에서 특정 문자 인덱스 찾기
* `index('문자')`

```python
my_string = "Hello, World!"
my_index = my_string.index(',')
```

#### 문자열 자르기(Slicing)
* `문자열[시작인덱스:끝인덱스]`, 끝인덱스는 미포함
* 시작인덱스, 끝인덱스를 생략하면 각각 0, 문자열 마지막 인덱스로 잘라옴

```python
b = "Hello, World!"
print(b[2:5])
```

#### 문자열 공백 제거(Remove Whitespace)
* `strip()`: 양옆 공백 제거
* `rstrip()`: 오른쪽 공백 제거
* `lstrip()`: 왼쪽 공백 제거

```python
a = " Hello, World! "
print(a.strip()) # returns "Hello, World!"
```


<!--more-->

## 참고자료
+ [스파르타 코딩클럽](https://spartacodingclub.kr/)
+ [W3Schools Python String Ref.](https://www.w3schools.com/python/python_ref_string.asp)
