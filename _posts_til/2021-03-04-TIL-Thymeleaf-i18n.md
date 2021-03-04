---
layout: article
title: Thymeleaf 다국어 처리 시 MessageResource에 arg를 전달하려면?
aside:
    toc: false
---

이제 Spring-Thymeleaf에서 다국어 처리하는 건 다 배웠다고 생각했는데 또 새로운 걸 배웠다.  <br/>
Thymeleaf에서 `MessageResource`를 불러올 때, `argument`를 어떻게 전달하면 되는지 알아 보았다.

### 기본
argument를 넘길 필요가 없을 때는 `[[#{MESSAGE_KEY}]]`과 같이 사용한다.
```html
<button class="btn btn-primary">[[#{button.yes}]]</button>
```

### 인자를 넘기려면?
만약에 MessageResource에 다음과 같은 `key-value`가 있다고 해보자. `{0}` 값은 상황에 따라 다른 값이 들어갈 수 있다.
```properties
label.subtitle.age = 만 {0}세 이상입니다.
```
서버 단에서 넣는 방법도 있지만, 찾아 보니 Thymeleaf에서 넣는 방법도 있었다. <br/>
메시지 키 뒤 `( )` 안에 인자를 넣어주면 된다. 여러 개면 `,`로 나열하면 된다.
```html
<p th:utext="#{label.subtitle.age(14)}"></p>
```
`( )` 안에 스프링 객체 값을 넣을 수도 있다.
```html
<p th:utext="#{label.subtitle.age(${userDto.ageLimit})}"></p>
```


<!--more-->
### 참고자료
[StackOverflow](https://stackoverflow.com/questions/20789441/how-to-show-localization-messages-with-parameters-in-spring-3-thymeleaf/) <br/>

