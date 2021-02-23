---
layout: article
title: Spring Boot + Thymeleaf template + Mailjet 메일 전송(feat. noreply 계정)
tags: [java, spring, thymeleaf, mail]
aside:
    toc: true
---

Spring에서 메일을 보낸다고 하면 `JavaMailSender` 클래스로 보내는 방법이 가장 많이 나옵니다. 저 역시 개발 과정에서는 저의 구글 계정 설정을 변경하여 메일 전송을 테스트했습니다.
하지만 운영 환경에서도 제 계정으로 보낼 수는 없습니다. 우선, 보내는 계정이 `noreply@company.com`였으면 하고, 한꺼번에 다량을 메일을 보낼 수 있어야 하기 때문입니다. <br/>
그래서 오늘은 Mailjet으로 메일을 전송하는 방법을 찾아 적용했습니다.

## 사전 작업: Mailjet에서 계정 생성
### 왜 Mailjet을 선택했는가?
저희 회사는 Google Cloud Platform의 VM 인스턴스, 그리고 쿠버네티스를 사용합니다. 그래서 GCP와 계약이 되어 있는 메일 서비스 공급자를 이용하면 무료 등급을 사용할 수 있습니다. <br/>
[GCP 문서](https://cloud.google.com/compute/docs/tutorials/sending-mail?hl=ko)를 보면 다음 세 가지 메일 서비스를 이용할 수 있습니다.
> SendGrid, Mailgun, Mailjet은 Compute Engine 고객이 자사 서버를 통해 이메일을 설정하고 보낼 수 있는 무료 등급을 제공합니다.

이 중 하나를 택하면 되는데, 저희는 Mailjet을 선택했습니다. 사실 큰 이유는 없습니다. 세 서비스 모두 Java API 문서를 잘 제공하고 있고, 저희는 그렇게까지 대용량 메일을 보낼 예정은 아니라 서비스 간에 큰 차이는 없었습니다.
다만 Mailjet은 계정을 생성할 때 카드 정보를 입력하지 않아도 되길래 우선 이 서비스로 선택했습니다😅. <br/>
Mailjet 서비스의 특징은 [Mailjet으로 이메일 보내기](https://cloud.google.com/compute/docs/tutorials/sending-mail/using-mailjet?hl=ko) 페이지에 나와 있습니다.

### Mailjet 계정 생성
[Google-Mailjet 파트너 사이트](https://www.mailjet.com/integration/google/?p=google)에서 계정을 생성합니다. "Sign Up For Free" 버튼을 눌러 진행하면 됩니다.
> 이메일 검증을 하니, 실제로 이메일을 받을 수 있는 메일 주소로 가입해야 합니다.

### API Key 발급
메일 전송을 위한 Master API Key는 자동으로 발급됩니다. 확인을 위해 오른쪽 상단의 계정을 눌러, "Account settings"에 들어갑니다. 그 다음 아래의 "Master API Key & Sub API key management"에 들어갑니다.<br/>
![Account Setting](/assets/images/posts/2021-02-23_01_account.png) <br/><br/>
그러면 아래와 같이 API Key와 Secret Key를 확인할 수 있습니다. <br/>
![Account Setting](/assets/images/posts/2021-02-23_02_key.png) <br/>

### 보내는 사람, Sender 주소 추가 (`noreply@company.com` 계정을 위한 도메인 인증)
(가입한 계정 외에) 보내는 사람을 추가하고 싶은 분은 다음의 과정이 필요합니다. 만약 실제로 존재하는 계정을 개별로 추가하고 싶다면 Address 목록에 추가하면 됩니다. 이 부분은 계정 가입과 유사하므로 따로 설명하지 않겠습니다. <br/><br/>
만약 회사 계정에 발신 전용 **`noreply@company.com` 계정이 따로 없는데 가계정으로 전송하고 싶다면 Domain 인증**을 해야 합니다. 도메인 인증을 하면 `@company.com`처럼 끝나는 모든 이메일 주소를 별도의 인증 없이 다 Sender로 등록할 수 있습니다.
**따라서 실제로 `noreply@compan.com` 계정이 없다 하더라도, 별도의 메일 인증 없이 전송 가능합니다.** <br/>

"Account settings"를 눌러 이번에는 "Add Sender Domains & Addresses"를 클릭합니다. <br/>
![Account Setting](/assets/images/posts/2021-02-23_03_domain.png) <br/><br/>
그 다음 "Add a new domain"을 눌러 본인이 원하는 도메인을 입력하세요. 저는 회사 도메인을 이미 등록해서, 예시로 `naver.com` 도메인을 인증한다고 해보겠습니다. 도메인과 라벨을 넣고 "Next"를 누르면 다음의 화면이 나옵니다.<br/>
![Account Setting](/assets/images/posts/2021-02-23_04_domain.png) <br/>
저는 DNS 서버에 접속할 수 있어, 두 번째 방법을 택했습니다. 각 회사의 DNS 서버에 접속해서 해당 도메인의 TXT 부분에 레코드를 추가하면 됩니다. 그후 "Check now"를 하면 바로 인증됩니다. <br/>


여기까지 했다면 사전 작업은 완료입니다.

## JAVA API로 메일 전송
### build.gradle
`mailjet-client` 의존성을 추가합니다.
```groovy
dependencies {
    // ...
    implementation group: 'com.mailjet', name: 'mailjet-client', version: '4.2.0'
    //...
}
```

### 설정 추가
저는 설정 파일(`application.yml`)에 `api-key`와 `secret-key`를 입력했습니다. (꼭 아래와 같은 형식은 아니어도 됩니다!)
```yaml
spring:
    mail:
        username: noreply@i-hope.net
        mailjet:
          api-key: 0fcxxxx...
          secret-key: b15xxxx...
```

### 기본 메일 발송
메일 전송 코드는 단순합니다. 구글에서 제공하는 샘플 코드를 보고 작성하면, 쉽게 작성할 수 있습니다.<br/>
> 💥 참고로, Mailjet에서 제공하는 Getting Started 샘플 코드를 그대로 붙여 넣어도 401(Unauthorized You have specified an incorrect Api Key / API Secret Key.) 예외가 발생할 수 있습니다.<br/>
>이 경우 `ApiKey`와 `SecretKey`를 입력하는 코드 `new MailjetClient(Environment.GetEnvironmentVariable("xxxx"), Environment.GetEnvironmentVariable("yyyy")...`에서 **`Environment.GetEnvironmentVariable`은 제외하고 두 키를 문자열 그대로 `"xxxxxx"`** 넣어 주세요.

```java
@Component
@RequiredArgsConstructor
public class MailUtil {
    @Value("${spring.mail.username}")
    private String from;
    @Value("${spring.mail.mailjet.api-key}")
    private String apiKey;
    @Value("${spring.mail.mailjet.secret-key}")
    private String secretKey;

    public void sendMessage(String to) {
        MailjetClient client = new MailjetClient(apiKey, secretKey, new ClientOptions("v3.1"));
        MailjetRequest request = new MailjetRequest(Emailv31.resource)
                .property(Emailv31.MESSAGES, new JSONArray()
                        .put(new JSONObject()
                                .put(
                                    new JSONObject()
                                        .put(
                                            Emailv31.Message.FROM,
                                            new JSONObject().put("Email", from))
                                        .put(
                                            Emailv31.Message.TO,
                                            new JSONArray().put(new JSONObject().put("Email", to)))
                                        .put(Emailv31.Message.SUBJECT, "Your email flight plan!")
                                        .put(
                                            Emailv31.Message.TEXTPART,
                                            "Dear passenger, welcome to Mailjet!"
                                            + "May the delivery force be with you!")
                                        .put(
                                            Emailv31.Message.HTMLPART,
                                            "<h3>Dear passenger, welcome to Mailjet!</h3>"
                                            + "<br />May the delivery force be with you!"))));

    }
}
```
이렇게만 해도 단순한 메일은 보낼 수 있습니다. 그럼 이제 thymeleaf 템플릿을 적용하는 법을 알아 보겠습니다.

### Thymeleaf 템플릿을 적용한 메일 발송
찾아 보니 Mailjet에서 템플릿을 쉽게 제작하고, Thymeleaf처럼 메시지 변수를 Responsive하게 넣어 메일을 생성할 수 있도록 서비스를 제공합니다. 하지만 저는 이미 Thymeleaf로 템플릿을 만들어 둔 상태라, Thymeleaf를 적용하는 방법을 찾아보았습니다. <br/>
저는 [Mailjet and Spring / Thymeleaf](http://michalszalkowski.com/java/mailjet-and-spring.html) 페이지에서 많은 도움을 받았습니다. <br/><br/>
템플릿을 적용한 메일을 보내려면 `SpringTemplateEngine`을 빈으로 등록해야 합니다.
```java
@Component
public class MessageConfig {

    @Bean
    public SpringTemplateEngine templateEngine(ITemplateResolver templateResolver, SpringSecurityDialect sec) {
        final SpringTemplateEngine templateEngine = new SpringTemplateEngine();
        templateEngine.setTemplateResolver(templateResolver);
        templateEngine.addDialect(sec);
        templateEngine.setMessageSource(messageSource());
        return templateEngine;
    }
}
```

그 다음 위의 `MailUtil` 클래스를 수정합니다.
  ```java
@Component
@RequiredArgsConstructor
public class MailUtil {
private final TemplateEngine templateEngine;
private final MessageSource messageSource;

@Value("${spring.mail.username}")
private String from;
@Value("${spring.mail.mailjet.api-key}")
private String apiKey;
@Value("${spring.mail.mailjet.secret-key}")
private String secretKey;

public void sendMessage(String to, MailTypeEnum mailType, Object[] variables, Locale locale) {
  MailjetClient client = new MailjetClient(apiKey, secretKey, new ClientOptions("v3.1"));
  MailjetRequest request = new MailjetRequest(Emailv31.resource)
          .property(Emailv31.MESSAGES, new JSONArray()
                  .put(new JSONObject()
                          .put(
                              new JSONObject()
                                  .put(
                                      Emailv31.Message.FROM,
                                      new JSONObject().put("Email", from))
                                  .put(
                                      Emailv31.Message.TO,
                                      new JSONArray().put(new JSONObject().put("Email", to)))
                                  .put(Emailv31.Message.SUBJECT, createSubject(mailType, locale))
                                  .put(Emailv31.Message.HTMLPART, createText(mailType, variables, locale)))));

}

public String createSubject(MailType mailType, Locale locale) {

    return messageSource.getMessage("mail.subject." + mailType.getValue(), null, locale);
}

public String createText(MailTypeEnum mailType, Locale locale, Object[] variables) {
    Context context = new Context();
    String text = messageSource.getMessage("mail.text." + mailType.getValue(), variables, locale);
    context.setVariable("text", text);

    return templateEngine.process("mail", context);
}

  }
```
+ `SpringTemplateEngine`을 주입 받습니다.
+ 메시지를 생성하는 메소드 `createText()`를 선언합니다. 참고로 저는 메일의 타입을 나누는 `MailTypeEnum`을 선언했습니다. 그리고 enum의 값에 따라 `MessageSource`에서 `locale`에 맞는 본문을 불러옵니다.
+ `templateEngine.process("MAIL_TEMPLATE_PATH", context)`를 반환합니다. 저의 경우, template은 `src/resources/templates/mail.html`에 위치하고 있습니다.
+ 제목을 생성하는 메소드 `createSubject()`도 유사하게 선언합니다.


이와 같이 하면 기존의 `JavaMailSender`로 메일을 전송한 코드를 되살리면서 쉽게 메일 서버를 교체할 수 있습니다.


***

지금까지 Spring-Thymeleaf, 그리고 Mailjet으로 메일을 전송하는 법을 알아보았습니다. `MessageSource`와 관련하여 다국어 처리 방법은 TIL에 산발적으로 정리되어 있습니다😅. 혹시나 궁금하신 분은 TIL을 참고해주시기 바랍니다!

<!--more-->

---
