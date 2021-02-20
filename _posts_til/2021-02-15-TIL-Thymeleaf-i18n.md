---
layout: article
title: Spring Framework + Thymeleaf 메일 전송 시 다국어 처리하기
aside:
    toc: false
---

회원 관리를 구현하다 보니 메일 전송할 일이 많다🤓. <br/>
스프링에 타임리프 메일 템플릿을 적용해서 메일 전송하기까지는 해봤는데, 오늘은 다국어 처리를 해보았다.

### Locale 관련 Config
미리 만들어 두었던 `WebMvcConfig` 클래스에 다음의 빈, 인터셉터 추가
```java
   @Configuration
   public class WebMvcConfig implements WebMvcConfigurer {

      /* ... */

       @Bean
       public LocaleResolver localeResolver() {
           SessionLocaleResolver sessionLocaleResolver = new SessionLocaleResolver();
           sessionLocaleResolver.setDefaultLocale(Locale.KOREAN);
           return sessionLocaleResolver;
       }

       @Override
       public void addInterceptors(InterceptorRegistry registry) {
           LocaleChangeInterceptor localeChangeInterceptor = new LocaleChangeInterceptor();
           localeChangeInterceptor.setParamName("lang");
           registry.addInterceptor(localeChangeInterceptor);
       }

        /* ... */

   }
```

### Mail 관련 Config
이미 메일 전송 테스트는 해본 후라 `JavaMailSender`, `SpringTemplateEngine` 빈은 만들어져 있었음. <br/>
눈 여겨 볼 부분은
+ `ResourceBundleMessageSource` 빈을 추가
+ `SpringTemplateEngine` 빈에서 `setMessageSource(messageSource())` 설정 추가

```java
@Component
public class MailConfig {
    @Value("${spring.mail.host}")
    private String host;
    @Value("${spring.mail.port}")
    private int port;
    @Value("${spring.mail.username}")
    private String username;
    @Value("${spring.mail.password}")
    private String password;

    @Bean
    public JavaMailSender javaMailSender() {
        final JavaMailSenderImpl mailSender = new JavaMailSenderImpl();
        mailSender.setHost(host);
        mailSender.setPort(port);
        mailSender.setUsername(username);
        mailSender.setPassword(password);

        Properties props = mailSender.getJavaMailProperties();
        props.put("mail.transport.protocol", "smtp");
        props.put("mail.smtp.auth", "true");
        props.put("mail.smtp.starttls.enable", "true");
        props.put("mail.debug", "false");

        return mailSender;
    }

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

}
```

### Message Sources 정의
`/src/main/resources` 폴더 안에 다국어 지원을 위한 key, value를 담은 messages.properties 파일 정의. <br/>
인텔리제이에서 ResourceBundle 만들기로 해도 좋고, 파일 명 앞 부분만 통일해줘도 알아서 ResourceBundle로 만들어 줌. <br/>
**중요한 점은 앞서 `MailConfig`에서 `ResourceBundleMessageSource`의 `setBaseName()`과 ResourceBundle 이름은 일치해야 함. <br/>**
> 💡 `No message found under code for locale` 예외가 뜬다면 base 이름을 잘 설정했는지, properties 파일에 `ko_KR`이라고 명시해줬는지 등을 확인!

참고로 인텔리제이에서 locale code를 붙이지 않은 기본 properties를 생성하면, 아래와 같이 한번에 여러 언어를 편집할 수 있음! <br/>
![messages.properties](/assets/images/til/2021-02-15_1.png) <br/>

메세지에 개행을 넣고 싶으면 아래와 같이 `<br><br>` 추가(인터넷에는 `\r\n`, `\\n`과 같은 방법을 사용할 수 있다고 하는데... 난 안 됨.)
```properties
mail.text.changePw = <p>본인 요청이 맞다면 아래 링크를 클릭해주세요. <br><br></p>
```
메세지에 `argument`를 넣고 싶을 수 있는데, 뒤에 나오는 `messageSource` 빈에서 배열(`Object[]`)로 넘겨줄 수 있음. <br/>
인자는 아래 `{숫자}` 안에 배열 인덱스 대로 들어감. (아래와 같이 링크를 넣을 수도 있음)
```properties
mail.link.changePw = <p><a href="{0}"><span>Link</span></a></p>
```

### 메일을 전송하는 클래스, MailService
앞서 config 파일에 추가해 둔 빈들을 주입 받고, 메시지를 생성하는 부분에서 `messageSource` 빈을 이용 <br/>
`String getMessage(String code, @Nullable Object[] args, Locale locale) throws NoSuchMessageException;` 메소드 참고!

```java
@Component
@RequiredArgsConstructor
public class MailUtil {
    private final JavaMailSender javaMailSender;
    private final TemplateEngine templateEngine;
    private final MessageSource messageSource;

    /* ... */

    public String createText(MailType mailType, Locale locale, Object[] variables) {
        Context context = new Context();
        String title = messageSource.getMessage("mail.title." + mailType.getValue(), variables, locale);
        String text = messageSource.getMessage("mail.text." + mailType.getValue(), variables, locale);

        context.setVariable("title", title);
        context.setVariable("text", text);

        return templateEngine.process("mail", context);
    }

    /* ... */
}
```

<!--more-->

### 참고자료
+ 늘 많은 도움을 받는 Baeldung 사이트 [🔗](https://www.baeldung.com/spring-boot-internationalization)
+ Thymeleaf 공식 문서 [🔗](https://www.thymeleaf.org/doc/articles/springmail.html)
