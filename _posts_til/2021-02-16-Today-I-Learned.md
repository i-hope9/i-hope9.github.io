---
layout: article
title: Java ClassNotFoundException javafx.util.Pair 해결
aside:
    toc: false
---

개발 과정에서 `javafx.util.Pair` 클래스를 사용했다. <br/>
로컬 개발 환경에서는 아무런 문제가 없었는데, 스테이징 서버에 올려 보니 애플리케이션이 시작 조차 못하고 있었다. <br/>
예외 로그는 다음과 같았다. (구체적인 클래스 이름은 @@@ 처리)

```shell
...
"Caused by: org.springframework.beans.factory.BeanCreationException: Error creating bean with name '@@@': Lookup method resolution failed; nested exception is java.lang.IllegalStateException: Failed to introspect Class [@@@] from ClassLoader [org.springframework.boot.loader.LaunchedURLClassLoader@5910e440] "
"Caused by: java.lang.IllegalStateException: Failed to introspect Class [@@@] from ClassLoader [org.springframework.boot.loader.LaunchedURLClassLoader@5910e440] "
"Caused by: java.lang.ClassNotFoundException: javafx.util.Pair "
...
```

로그가 매우 길게 찍혔지만, 결론은 `javafx.util.Pair` 클래스를 찾을 수 없어 빈 생성을 못했다는 예외. <br/>
다음의 [링크](https://gist.github.com/androidfred/bc64da9e6a355b984d37439ed63ae16b)에서 원인과 해결 방안을 찾아냈다😎 <br/>

### 원인
위 링크의 원문을 그대로 복사하자면, `javafx.util`이 OpenJDK에 포함되어 있지 않아 발생한 오류라고 한다.
> `javafx.util.Pair` and other classes from `javafx.util` are not included in OpenJDK.


OpenJDK와 Oracle JDK의 차이는 이 [블로그](https://jsonobject.tistory.com/395)를 보면 자세히 나와 있다. 읽어 보니 Oracle JDK는 일반적인 목적의 컴퓨팅에서는 무료며, 운영 환경에서는 구독형 라이센스를 구입해야 한다.
나의 로컬 컴퓨터에서는 Oracle JDK를 설치해두어서 `javafx.util.Pair` 클래스를 사용할 수 있던 반면에, Google Cloud Platform의 VM 인스턴스를 사용하고 있는 스테이징 환경에서는 Open JDK를 사용하고 있어 그렇지 못한 것으로 보인다. <br/>
> 💡 어떤 JDK가 설치되어 있는지 보려면 `java -version` 명령어를 실행하면 된다. Open JDK는 OpenJDK라는 문자열이 명시되어 있다.

### 해결책
위 링크에서 제시한 해결책은 다음과 같다. <br/>
**1. `javafx.util.Pair`  클래스를 유사한 클래스로 변경한다.** <br/>
이번에도 어김 없이 도움을 준 [Baeldung](https://www.baeldung.com/java-pairs)를 참고하면 `Pair` 대체재가 여럿 소개된다.
우리는 이 중 Apache Commons 라이브러리가 제공하는 클래스를 적용했다.

**2. `java-openjfx`를 설치한다.**

> 마지막 JDK를 교체한다는 해결책은 농담인 듯😂.


오늘도 전혀 예상치 못한 문제를 만났지만 이렇게 또 새로운 걸 배웠다😊.

<!--more-->
### 참고자료
본문 속 링크 참조!
