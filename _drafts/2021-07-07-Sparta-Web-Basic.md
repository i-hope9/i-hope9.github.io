---
layout: article
title: 웹 개발 기초 05. 프로젝트 구현, AWS 서버 배포 (feat. 스파르타 코딩 클럽)
tags: [web, html, css, javascript]
aside:
    toc: true
---

💡 본 글은 스파르타코딩클럽 '왕초보 시작반' 5주차 강의 내용을 바탕으로 정리하였습니다.

## Python - MongoDB 연동
#### Read: `find`, `sort`
* 예시 프로젝트에서 사용된 영화 배우 목록은 좋아요(like) 기준 역순으로 정렬되어야 함
* `find()` 함수 뒤에 바로 `sort()` 호출

```python
mycol.find().sort("name", 1)  #ascending
mycol.find().sort("name", -1) #descending
```

> 🌐 [Pymongo_w3school](https://www.w3schools.com/python/python_mongodb_sort.asp)




***
그동안 배운 내용을 바탕으로 프로젝트를 진행해서 새롭게 적은 내용은 많이 없다. 강의에서는 수정도, 삭제도 POST 메서드로 했지만 PUT이랑 DELETE 메서드 정도는 소개해줘도 괜찮지 않았을까 싶다. URL에 행위(update, delete)을 나타내지 않는다거나 그정도 규칙은 소개해주면 너무 어려웠으려나...
<!--more-->

## 참고자료
+ [스파르타 코딩클럽](https://spartacodingclub.kr/)
