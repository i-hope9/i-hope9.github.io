---
layout: article
title: 웹 개발 기초 01. html, css, javascript 맛보기 (feat. 스파르타 코딩 클럽)
tags: [web, html, css, javascript]
aside:
    toc: true
---

💡 본 글은 스파르타코딩클럽 '왕초보 시작반' 1주차 강의 내용을 바탕으로 정리하였습니다.

## 웹브라우저 동작 원리
### ⭐브라우저의 역할
+ HTML을 받는 경우
  - 서버에게 요청 + 결과물(데이터)을 그려주는 역할
  - 브라우저에서 새로고침하면 API를 통해 서버에 요청하고, 응답을 받아서 새로 뿌려줌
+ 데이터만 받는 경우
  - HTML은 그대로 두고 데이터만 내려줄 때가 더 많음(사실 HTML도 따지고 보면 데이터!)
  - 데이터는 주로 JSON 형식으로 주고 받음

### 각 코드의 역할
+ HTML: 웹 페이지의 뼈대
+ CSS: 꾸미기
+ JS: 동작

## HTML 기초
### 주요 태그
> 🌮 모든 태그를 외워서 쓰지 말고, 필요할 때 찾아가며 쓰세요! <br/>
> 🌮 IDE에서 코드를 자동으로 정렬하고 싶다면? `ctrl(cmd) + alt + L`

+ `<head>`: 페이지 속성 정보. `<body>` 외에 필요한 것들. 페이지 타이틀, 파비콘, 구글 검색 엔진 설정, CSS 및 JS 등
+ `<body>`: 말 그대로 본문.
  - `<div>`: 구역을 나눠주는 태그
  - `<p>`: paragraph. 문단.
  - `<h1>`: 제목. 페이지마다 하나씩 꼭 써주면 좋음. 그래야 구글 검색이 잘 됨.
  - `<a>`: 하이퍼링크
  - `<img>`: 이미지. src 속성에 url 넣기
  - `<input>`, `<button>`, `<textarea>` 등

## CSS 기초
### 어떻게 사용하는가?
* `class="mytitle"`인 요소를 꾸며주고 싶다면?
``` html
  <head>
    <style>
      .mytitle {

      }
    </style>
  </head>
```

* 모든 영역에 적용하고 싶다면? `*`
``` html
  <head>
    <style>
      * {

      }
    </style>
  </head>
```

### 자주 쓰는 CSS 연습
* 매우 기본적인 것들
  width, height, color, text-align

* `<div>` 안에 이미지 넣기
아래 세 개는 세트로 자주 쓰임!
``` html
  background-image: url("PATH");
  background-size: cover;
  background-position: center;
```

* 여백
  + padding: 나의 안 쪽 여백
  + margin: 나의 바깥 쪽 여백

> 🌮 글 속성인 요소를 강제로 가로, 세로가 있는 박스로 바꿔주려면? `display: block` (여백 줄 때 필요할 수도!)
>
> 🌮 `margin: top right bottom left;` 시계 방향으로 생각하면 쉬움

### 웹 폰트 입히기
* [Google Fonts](https://fonts.google.com/?subset=korean) 원하는 폰트 선택 후, Use on the web 참고

### 부트스트랩 사용하기
* 부트스트랩이란? 미리 만들어둔 CSS, JS 모음
* 부트스트랩에서 제공하는 컴포넌트에 내가 원하는 CSS 코드를 중첩해서 사용 가능
* [Bootstrap Components](https://getbootstrap.com/docs/4.0/components/alerts/) 바로가기

## Javascript 기초
### Javascript란?
* 프로그래밍 언어 중 하나
* Java와 Javascript는 아무 연관 관계가 없음
> 🌮 Javascript는 브라우저가 알아들을 수 있는 유일한 언어. 역사적인 이유로 자리 잡은 표준이기 때문!

### Javascript - HTML 연결
* `<head>` 안에 `<script>` 추가
``` html
  <head>
    <script>
      alert('안녕!');
    </script>
  </head>
```

### 기초 문법 1
> 💡 간단한 실습은 크롬 검사 도구  console에서 가능!

#### 변수
* 변수는 값을 저장하는 박스
* 변수 이름은 다른 사람들이 알아 듣기 쉽게, 그리고 각 조직의 규칙에 맞게(ex. snake_case, camelCase)
* `let`으로 변수를 선언
``` javascript
  let num = 20
  num = 'Bob' // 문자열도 가능
```

#### 사칙 연산과 기본 함수
* 사칙연산, 문자열 더하기가 가능
* 부등호로 크기 비교 (결과: true, false)
* 문자열 나누기: `myname.split('@')`
* 리스트 속 요소를 특정 문자로 합치기: `names.join('>')`


#### 리스트
* 순서를 지켜서 갖고 있는 형태
* 0번 인덱스부터 저장
* 딕셔너리도 추가 가능
``` javascript
  let a_list = ['수박', '참외', '배']
  a_list[0]
  a_list.push('감')
```

#### 딕셔너리
* key-value를 저장할 수 있는 형태
* 순서 상관 없음
* value 안에 리스트를 담을 수도 있음
``` javascript
  let a_dict = {'name' : 'bob', 'age': 27}
  a_dict['name']  // 'name'을 키로 갖는 값 출력
  a_dict['height'] = 165  // 'height' 키에 164라는 값 저장
  a_dict['fruits'] = a_list // 앞서 선언한 리스트를 값에 저장 가능
```

### 기초 문법 2
#### 함수
* 호출하면 정해진 동작을 하는 것
``` javascript
  function sum(num1, num2) {
    return num1 + num2
  }
  function no_param() {
    alert('안녕!');
  }
```

#### 조건문
* `if`, `else if`, `else`
* AND (`&&`), OR (`||`)

#### 반복문
* `for (let i = 0; i < 100; i++)`: i는 0부터 시작, 1씩 추가, i가 100 미만일 때까지 반복
* 리스트/딕셔너리 + 반복문 조합을 잘 이해하기
``` javascript
  // 딕셔너리를 담은 리스트 선언
  let scores = [
    {'name':'철수', 'score':90},
    {'name':'영희', 'score':85},
    {'name':'민수', 'score':70},
    {'name':'형준', 'score':50},
    {'name':'기남', 'score':68},
    {'name':'동희', 'score':30},
  ]

  // 반복문으로 이름과 점수 출력
  for (let i = 0; i < scores.length; i++) {
    let name = scores[i]['name']
    let score = scores[i]['score']
    console.log(name + ' ' + score)
  }
```

***
나름 웹 개발을 해본 적은 있지만, 검색해가며 주먹구구 식으로 만든 경향이 있었다.
특히 CSS!!! 강의에 나온 것처럼 정렬과 배치가 어려운 편이다😂. 버튼을 페이지 가운데 정렬하기 위해 온갖 노력을 했던 기억이 난다.
이번 기회에 차근차근 배워 나갈 수 있어 좋다😊!

<!--more-->
### 참고자료
+ [스파르타 코딩클럽](https://spartacodingclub.kr/)
