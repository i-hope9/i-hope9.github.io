---
layout: article
title: Spring Framework query parameter로 배열 받기(@RequestParam, dto 객체)
aside:
    toc: false
---

참으로 오랜만에 쓰는 TIL... 그동안 회사 일에 치여서 배우는 건 많았는데... 다시 열심히 써봐야지. <br/>
오늘은 Query parameter로 배열이 넘어올 때 Spring Framework에서 어떻게 처리할 수 있는지 배웠다.

### 요청 URL
Query parameter로 같은 Key에 Value를 다음과 같이 요청하면 배열과 같이 처리된다. <br/>
```
http://www.example.com:80/profile?student=1&student=2
```

### Spring Framework에서 처리하기
#### `@RequestParam`으로 바로 받기
배열을 사용하거나, `Collection<E>`를 사용할 수 있다. <br/>
다른 `@RequestParam`과 마찬가지로 `required = false` 옵션을 주면 해당 Key를 Query param에 포함하지 않아도 된다. 이 경우, 배열과 `Collection<E>` 모두 `null`로 매핑된다.

```java
@GetMapping(value = "/profile")
public String getProfile(@RequestParam(value = "student") int[] studentArray){
    ....
}
```

```java
@GetMapping(value = "/profile")
public String getProfile(@RequestParam(required = false) Set<String> student){
    ....
}
```

#### DTO 객체 안에 받기
다른 Query parameter와 함께 요청을 받으려면 보통 객체로 받는 경우가 많다. 이 경우에도 객체 안에 해당 parameter를 배열 혹은 `Colleciton<E>`로 선언해두면 된다. <br/>
`@NotEmpty` 어노테이션을 붙이지 않으면 `required = false`와 같은 효과가 나며, Query Parameter에 해당 Key가 없으면 `null`로 매핑된다.

```java
public class ProfileRequestDto {
    private String tag;
    @NotEmpty
    private List<Integer> student;
    ...
}
```
```java
@GetMapping(value = "/profile")
public String getProfile(@Valid ProfileRequestDto reqDto){
    ....
}
```

***
생각보다 매우 간단하게 매핑이 되어 손쉽게 처리했다😎!

<!--more-->
### 참고자료
[StackOverflow](https://stackoverflow.com/questions/9768509/can-spring-mvc-handle-multivalue-query-parameter)
