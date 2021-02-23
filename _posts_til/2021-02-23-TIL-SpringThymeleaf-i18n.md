---
layout: article
title: Spring Framework + Thymeleaf ENUM 다국어 처리, html select default 관련
aside:
    toc: false
---

TIL을 시작하고서 유난히 프론트 관련 글을 많이 쓰게 된다. <br/>
오늘은 아마도 다국어 처리의 마지막 글이 되지 않을까!

### Java enum 활용한 Thymeleaf에 다국어 처리
제목이 살짝 어색하지만, 오늘은 `enum`을 활용해서 다국어 처리를 하는 법을 배웠다.<br/>
엔티티에 `enum` 타입의 필드가 있을 때, Thymeleaf에서 `<select>`의 선택지 `<option>`에 `enum`을 넣어 준다.<br/>
가장 기본적인 사용법은 오늘도 역시 [Baeldung](https://www.baeldung.com/thymeleaf-enums).<br/>
아래와 같이 성별 `enum`을 선언했다고 하면, thymeleaf에서는 다음과 같이 `GenderEnum`의 필드를 선택지로 보여줄 수 있다.
```java
public enum GenderEnum {
    MALE, FEMALE, OTHER
}
```
```html
    <div class="form-group">
        <label for="gender">[[#{label.user.gender}]]</label>
        <select id="gender" class="custom-select d-block w-100 rounded-0" th:field="*{gender}">
            <option th:each="genderOp : ${T(net.ihope.sample.domain.model.enums.GenderEnum).values()}" th:value="${genderOp}" th:text="${genderOp}"></option>
        </select>
    </div>
```

여기서 내가 궁금했던 점은 **화면에서 선택지를 보여주는 `th:text="${genderOp}"` 속성에 다국어 처리를 하는 방법**이었다.<br/>
다행히 [Stackoverflow](https://stackoverflow.com/questions/40384453/localization-of-enum-in-spring-thymeleaf/40384732)에서 쉬운 방법을 찾았다. <br/>
`ResourceBundles`에 다음과 같이 메시지를 추가.
```properties
enum.gender.FEMALE	=	여성
enum.gender.MALE	=	남성
enum.gender.OTHER	=	그 외
```
주목할 thymeleaf 문법은 `th:text="#{'enum.gender.'+${genderOp}}"`
```html
<select id="gender" class="custom-select d-block w-100 rounded-0" th:field="*{gender}">
    <option th:each="genderOp : ${T(net.ihope.sample.domain.model.enums.GenderEnum).values()}" th:value="${genderOp}" th:text="#{'enum.gender.'+${genderOp}}"></option>
</select>
```

### html `<select>` default 값
우리 사이트는 성별이 필수 값이 아니다. 그래서 선택 안 함 선택지가 필요했다. 그리고 선택하라는 문구를 기본 default 선택지로 넣고 싶었다. <br/>
`ResourceBundles`에 다음과 같이 메시지를 추가.
```properties
select.option.guide	=	선택해주세요.
select.option.null	=	선택 안 함
```
html 파일은 다음처럼 수정. 기본 선택지에는 `hidden` 속성을 넣어준다. (`disabled selected`를 넣어주면 된다는 말도 있다.)
```html
<select id="gender" class="custom-select d-block w-100 rounded-0" th:field="*{gender}">
    <option class="text-secondary" value="" th:text="#{select.option.guide}" hidden></option>
    <option th:each="genderOp : ${T(${T(net.ihope.sample.domain.model.enums.GenderEnum).values()}}" th:value="${genderOp}" th:text="#{'enum.gender.'+${genderOp}}"></option>
    <option value="" th:text="#{select.option.null}"></option>
</select>
```
이렇게 하면 최종 결과는 요렇게! <br/>
![select-option-1](/assets/images/til/2021-02-23_1.png)
![select-option-2](/assets/images/til/2021-02-23_2.png) <br/>


<!--more-->
### 참고자료
본문 속 링크 참조
