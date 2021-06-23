---
layout: article
title: 웹 개발 기초 02. JQuery, Ajax (feat. 스파르타 코딩 클럽)
tags: [web, html, css, javascript]
aside:
    toc: true
---

💡 본 글은 스파르타코딩클럽 '왕초보 시작반' 1주차 강의 내용을 바탕으로 정리하였습니다.

## JQuery 시작하기
### jQuery란?
* HTML의 요소들을 조작하는 편리한 Javascript 라이브러리
* 다른 사람이 짜둔 Javascript 코드, 반드시 `import`해야 함

> 📌 공식 홈페이지 소개: jQuery is a fast, small, and feature-rich JavaScript library. It makes things like HTML document traversal and manipulation, event handling, animation, and Ajax much simpler with an easy-to-use API that works across a multitude of browsers. With a combination of versatility and extensibility, jQuery has changed the way that millions of people write JavaScript.


## 자주 쓰는 jQuery
### `id`로 `input` 요소 값 가져오기, 수정하기
* 값을 가져오고 싶은 `input` 요소에 `id` 값을 줌
```html
<input id="post-url" type="email" class="form-control" aria-describedby="emailHelp" placeholder="">
```
* 아이디 값으로 `val()` 불러오기
```javascript
  $('#post-url').val();
  $('#post-url').val('바꾸고 싶은 값');
```

### `id`로 `div` 요소 보이기, 숨기기
```javascript
  $('#post-box').hide();
  $('#post-box').show();
```

### `id`로 css 값 가져오기
```javascript
  $('#post-box').css('width');
  $('#post-box').css('display');
```

### `id`로 (`input` 외) 요소 텍스트 수정하기
* 수정하고 싶은 요소에 `id` 값을 줌
```javascript
  <button id="btn-posting-box" type="button" class="btn btn-primary">포스팅 박스 열기</button>
```
* 아이디 값으로 수정
```javascript
  $('#btn-posting-box').text('포스팅 박스 닫기');
```

### `id`로 요소의 내부 태그 모두 지우기
```javascript
  $('#post-box').empty();
```

### `id`로 `img` 요소의 `src` 수정하기
```javascript
  let img_src = '';
  $('#img-cat').attr('src', img_src)
```

## 자바스크립트 함수
### 문자열에 문자 포함 여부 - `includes()`
```javascript
  string.includes('@');
```

### 페이지 로딩 하자마자 호출되는 함수
```javascript
  $(document).ready(function () {
      // code
  });
```

## 클라이언트-서버 통신 이해하기
### 서버 → 클라이언트: `JSON` 형태 응답
* Key - Value 형태
* Value에는 List, Dictionary가 들어올 수 있음
* 가볍고, 사람과 기계 둘 다 이해하기 쉬운 형태

### 클라이언트 → 서버: `GET` 요청
* Http Method 중 (통상적으로) 데이터를 읽어올 때 씀
  + `?`: query string, 클라이언트에서 서버로 전달할 데이터 작성
  + `&`: query를 연결 (전달할 데이터가 더 있다!)

## Ajax 시작하기
* Javascript 라이브러리 중 하나
* 페이지의 일부만 업데이트할 수 있는 기법

> 💡` Ajax`는 `jQuery`를 임포트한 상태에서만 작동

### Ajax 기본 골격
```javascript
$.ajax({
  type: "GET",
  url: "여기에URL을입력",
  data: {},
  success: function(response){
    console.log(response)
  }
})
```

***
느낀 점

<!--more-->
### 참고자료
+ [스파르타 코딩클럽](https://spartacodingclub.kr/)
+ [jQuery Document](https://jquery.com/)
