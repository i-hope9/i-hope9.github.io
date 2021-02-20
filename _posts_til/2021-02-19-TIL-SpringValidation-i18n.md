---
layout: article
title: Spring Validation 어노테이션 메시지에 다국어 적용(Message Resources 활용)
aside:
    toc: false
---

어제에 이어 계속 다국어 처리하는 중. <br/>
오늘은 서버 측 유효성 검사 시 message에 Message Resources를 적용하는 방법을 알아냈다.

### Bean 생성
다국어 지원을 위해 `MessageResource` 빈은 만든 상태였다. 하지만 서버 측 유효성 검사 어노테이션(예, `@NotNull`, `@Min`...)에 `MessageResource` 빈을 사용하려면 추가적인 설정이 필요하다. <br/>
바로 맨 아래에 있는 빈, **`LocalValidatorFactoryBean`** 요것! <br/>
클래스 생성 내에 `setValidationMessageSource(MessageSource messageSource)` 메소드로 커스텀한 `MessageSource`를 지정할 수 있다.
지정해주지 않으면 JSR-303의 디폴트 번들인 `ValidationMessages.properties`에 의존한다고 한다.
```java
@Component
public class MailConfig {

    /* ... */

    @Bean
    public SpringTemplateEngine templateEngine(ITemplateResolver templateResolver, SpringSecurityDialect sec) {
        final SpringTemplateEngine templateEngine = new SpringTemplateEngine();
        templateEngine.setTemplateResolver(templateResolver);
        templateEngine.addDialect(sec);
        templateEngine.setMessageSource(messageSource());
        return templateEngine;
    }

    @Bean
    public ResourceBundleMessageSource messageSource() {
        final ResourceBundleMessageSource messageSource = new ResourceBundleMessageSource();
        messageSource.setBasename("messages");
        messageSource.setDefaultEncoding("UTF-8");

        return messageSource;
    }

    @Bean
    public LocalValidatorFactoryBean getValidator() {
        LocalValidatorFactoryBean factoryBean = new LocalValidatorFactoryBean();
        factoryBean.setValidationMessageSource(messageSource());

        return factoryBean;
    }

}
```

### Validation 어노테이션에 적용
설정을 마치면 Validation을 위한 어노테이션에 적용하기는 매우 쉽다. <br/>
`message = {KEY}` 형태로 넣어주면 끝! (참고로 `@NotInUse`는 내가 커스텀한 어노테이션이다.)
```java
    @NotInUse(message = "{error.id.remote}", flag = NotInUse.Flag.USER_ID)
    @NotBlank(message = "{error.id.requiredJoin}")
    private String id;
```

오늘은 다국어 처리를 하느라 문자열을 정리하고, 번역하고, 기존에 하드 코딩으로 박아 둔 문자열을 모두 교체하느라 시간이 오래 걸렸다.
글로벌 서비스를 할 때는 미리 다국어 처리를 해가면서 구현하는 쪽이 낫겠다.

<!--more-->
### 참고자료
+ Custom Validation MessageSource in Spring Boot [🔗](https://www.baeldung.com/spring-custom-validation-message-source)
