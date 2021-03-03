---
layout: article
title: JQuery Validation 나이 제한 유효성 검사를 하려면?
aside:
    toc: false
---

JQuery Validation으로 `<input type="date"/>` 필드의 유효성 검사를 해보았다.  <br/>
기본으로 제공하는 검사로는 한계가 있어서 추가 메소드를 구현해야 했다.

### custom-method.js 구현
```javascript
$.validator.addMethod("olderThan", function(value, element, param) {
    if (value.length === 0) {
        return true;
    }
    var today = new Date();
    var birthday = new Date(value);
    var age = today.getFullYear() - birthday.getFullYear();
    if (age > param + 1) {
        return true;
    }
    var gapMonth = today.getMonth() - birthday.getMonth();
    var gapDate = today.getDate() - birthday.getDate();
    if (gapMonth < 0 || (gapMonth === 0 && gapDate < 0)) {
        age--;
    }
    return age >= param;
}, $.validator.format("Only over {0} years old can join."));
```
+ 우리는 생년월일이 필수값이 아니라 `if (value.length === 0) return true;` 조건을 추가했다.
+ `addMethod()`에서 마지막에 메시지를 적는 부분은 포맷(`$.validator.format`)을 지정할 수 있다. 여기서 `{0}`은 `param`을 순서대로 넣어준다고 보면 된다.

### custom-method.js 적용
```html
<script type="text/javascript" th:src="@{/js/validation/custom-methods.js}"></script>
```
+ 파일을 적용 안 하면 함수를 찾을 수 없다는 에러가 뜬다.

### Jquery Validation rule에 적용
```javascript
$('#form-join').validate({
    onfocusout: function (element) {
        $(element).valid();
    },
    ignore: '.ignore',
    rules: {
        'birthday': {
            olderThan: minAge
        }
    },
    messages: {
        'birthday': errorCheckOverAge
    }
});
```
+ `minAge`는 다국어 처리에 따라 14세, 16세로 다르게 넘어 온다.


<!--more-->
### 참고자료
[jQuery 공식 사이트](https://jqueryvalidation.org/jQuery.validator.addMethod/) <br/>
[Check for a Minimum Age with jQuery Validation](https://petemall.com/validate-minimum-age-with-jquery-validate/)
