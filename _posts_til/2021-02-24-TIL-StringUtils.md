---
layout: article
title: Spring StringUtils (null 과 isEmpty 체크를 한번에!)
aside:
    toc: false
---

회원 관리(회원가입, 아이디 찾기, 패스워드 찾기 등) 1차 개발이 어느 정도 마무리 되어 간다👏!!<br/>
오늘은 `reCaptcha` 서버 측 유효성 검사를 추가했는데, 이는 TIL 보다는 정식 글로 쓰려고 한다.<br/>
오늘의 TIL은 매우 소소한 발견을 적어야지.

### Spring Framework가 제공해주는 StringUtils
[StringUtils](https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/util/StringUtils.html)는 이름 그대로 `String`와 관련한 다양한 메소드를 제공한다.
문자열을 다루다 보면 은근 고려해야 하는 예외가 많다. 직접 구현할 수도 있지만, `StringUtils`를 사용하면 번거로운 작업들을 많이 해결할 수 있다.<br/>
예를 들어, 문자열이 `null` 이거나 `isEmpty`인지 확인하는 경우 그동안 다음과 같이 썼다.
```java
if (stringValue != null && !stringValue.isEmpty()) {
    /*...*/
}
```
`StringUtils`는 위의 조건식을 그대로 검사해주는 메소드를 제공한다.
```java
public static boolean hasLength(@Nullable String str) {
    return (str != null && !str.isEmpty());
}
```
다음과 같이 쓰면 됨!
```java
if(StringUtils.hasLength(stringValue)) {
    /*...*/
}
```

이 외에도 공식 문서를 살펴 보니 다양한 메소드를 제공한다. 눈에 띄는 메소드들만 정리해봤다.
+ `boolean hasText(String str)`: (공백을 제외한) 실제 문자를 포함하는지.
+ `String trimAllWhitespace(String str)`: trim()과 달리, 문자열 사이의 모든 공백을 제거한 후 반환.
+ `String getFilenameExtension(String path)`: 파일 경로 문자열을 주면, 파일의 확장자만 반환.

각 메소드의 내부 동작 구조를 살펴 보니 또 배울 점이 많다. 필요한 기능이 있을 때 하나씩 보면 좋겠다.
<!--more-->
### 참고자료
본문 속 링크 참조
