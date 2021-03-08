---
layout: article
title: Spring @Controller와 @FrameworkEndpoint는 무슨 차이일까?
aside:
    toc: false
---

Spring Security를 이용해서 OAuth2.0을 구현하다 보니 `@FrameworkEndpoint`라는 어노테이션을 발견했다.  <br/>
`@Controller`와 차이가 없어 보여서, 오늘은 `@FrameworkEndpoint`가 뭔지 알아 보았다.

### `@FrameworkEndpoint`란?
패키지 자체가 `org.springframework.security.oauth2.provider.endpoint`다. Spring Security 중에서도 OAuth2.0을 위한 어노테이션으로 보인다. <br/>
[공식 문서](https://docs.spring.io/spring-security/oauth/apidocs/org/springframework/security/oauth2/provider/endpoint/FrameworkEndpoint.html)의 정의는 다음과 같다. <br/>
> Synonym for `@Controller` but only used for endpoints provided by the framework (so it never clashes with user's own endpoints defined with @Controller). <br/>
> Use with `@RequestMapping` and all the other `@Controller` features (and match with a FrameworkEndpointHandlerMapping in the servlet context).

정리하자면,
+ `@Controller`와 동의어
+ `@FrameworkEndpoint`는 프레임워크가 제공하는 엔드포인트에서만 사용. 따라서 사용자가 `@Controller`로 정의한 엔드포인트와는 충돌하지 않음.
+ `@RequestMapping`를 비롯하여 `@Controller`와 사용하는 모든 어노테이션을 그대로 쓸 수 있음

`@FrameworkEndpoint`와 같은 패키지에 속한 `FrameworkEndpointHandlerMapping` 클래스가 프레임워크 엔드포인트를 처리한다. <br/>
`FrameworkEndpointHandlerMapping`은 `RequestMappingHandlerMapping`를 상속 받은 클래스다. 해당 클래스가 `@FrameworkEndpoint`가 어노테이트된 프레임워크 엔드포인트를 `RequestMapping`하는 역할을 한다. <br/>
(참고로 `RequestMappingHandlerMapping`은 security가 아닌 `org.springframework.web.servlet.mvc.method.annotation` 소속 클래스다. `@Controller`의 `RequestMapping`을 등록한다.) <br/><br/>
결국 **`@FrameworkEndpoint`는 `FrameworkEndpointHandlerMapping` 클래스를 통해 `@Controller`와 같이 동작하는 셈**이다. <br/><br/>
`@FrameworkEndpoint`가 붙은 클래스를 보면 Oauth 2.0 동작을 커스터마이징하기 위해 어떤 클래스를 보면 되는지 참고할 수 있다. <br/>
![FrameworkEndpoint](/assets/images/til/2021-03-08.png)

***
결국 두 어노테이션은 차이가 없다는 결론을 얻었지만, 자세히 알아보면서 궁금증을 해소할 수 있어 좋았다😎.
<!--more-->
### 참고자료
본문 속 링크 참조
