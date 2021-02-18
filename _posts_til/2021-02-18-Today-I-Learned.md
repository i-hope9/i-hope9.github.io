---
layout: article
title: Thymeleaf & 외부 javascript 파일에 다국어 적용(Message Resources 활용)
aside:
    toc: false
---

요즘 회원 관리(회원 가입, 로그인, 아이디/비밀번호 찾기 등)를 구현하고 있다. 어느 정도 구현은 다 되어서, 글로벌 서비스를 위해 다국어 처리를 알아 보는 중이다. <br/>
Thymeleaf에 Message Resources(.properties) 파일을 적용하여 다국어 처리를 하는 방법을 간략히 정리하고자 한다.

### html 페이지 내에 적용
며칠 전 소개한 자료[🔗](https://www.baeldung.com/spring-boot-internationalization)에서 쉽게 찾을 수 있다. <br/>
간략하게 소개하자면 아래의 두 방법 중 하나를 쓰면 된다. `#{}` 안에 들어가는 값은 properties 파일의 key 값이다.
```html
<label for="inputEmail">[[#{label.user.id}]]</label>
<label for="inputEmail" th:text="#{label.user.id}"></label>
```

### html script 내에 적용
html 파일 내 `<script> ... </script>` 에서 Message Resources 값을 가져오려면 다음과 같이 하면 된다. <br/>
중요한 건 **`th:inline="javascript"`**
```html
<script type="text/javascript" th:inline="javascript">
    var labelUserId = [#{label.user.id}]];
</script>
```

### 외부 js 파일에 적용
외부의 js 파일에는 Message Resources 값을 바로 가져오는 방법을 찾기가 어려웠다. <br/>
외부 js 파일은 유효성 검사를 구현해 놓았다. 현재 프론트 상의 유효성 검사를 Jquery Validation 플러그인을 사용 중이며, 에러 문구를 전부 커스텀했다. <br/>
Jquery Validation을 html 파일의 `<script></script>` 내부에 구현해도 되지만, html 파일과 분리하여 관리하고 싶었다. <br/>
결국 외부 js 파일에 바로 가져오는 방법은 찾지 못하고, 대신 다음의 방법을 사용했다. <br/>

우선 html의 `<script></script>` 내부에 Message Resources 값을 불러와 변수에 저장한다. 그리고 아래에 `th:src` 태그로 외부 js 파일을 불러온다.
```html
<script type="text/javascript" th:inline="javascript">
     var errorIdRequired = [[#{error.id.required}]];
     var errorPwPattern = [[#{error.pw.pattern}]];
     var errorPwConfirm = [[#{error.pw.confirm}]];
</script>
<script type="text/javascript" th:src="@{/js/account/join.js}"></script>
```
그러면 join.js 파일에서 다음과 같이 변수를 바로 사용할 수 있다.
```javascript
validateJoinForm : function() {
    $('#form-join').validate({
        rules: {
            'id': 'required',
            'password': {
                required: true,
                pattern: '^(?=.*[a-zA-Z])(?=.*\\d)(?=.*[@$!%*?&])[A-Za-z\\d@$!%*?&]{8,15}$'
            },
            're-password': {
                required: true,
                equalTo: '#password'
            }
        },
        messages: {
            'id': errorIdRequired,
            'password': errorPwPattern,
            're-password': errorPwConfirm
        },
        // ...
        // ...
    });
}
```

현재로써는 이게 최선의 방법으로 보인다. '다국어 처리는 나중에 적용하면 되겠지'라고 생각했는데... 미리 염두에 두고 작업할 걸 그랬다.
<!--more-->
### 참고자료
+ Thymeleaf 공식 문서(3.1 A multi-language welcome 중 Using th:text and externalizing text) [🔗](https://www.thymeleaf.org/doc/tutorials/2.1/usingthymeleaf.html#a-multi-language-welcome)
+ Stackoverflow 관련 질문 [🔗](https://stackoverflow.com/questions/27176640/how-to-put-code-thymeleaf-in-an-external-javascript-file)
