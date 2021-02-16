---
layout: article
title: Spring Boot + Netty TCP 소켓 서버 (3) SSL 적용한 에코 서버 구현
tags: [java, spring, netty, tcp/ip]
aside:
    toc: true
---

[지난 글]({% post_url 2020-12-14-SpringBoot-Netty-2-SocketServer %})에서는 스프링에 네티를 적용한 기본 에코 서버를 구현해보았습니다.
이제 이 프로젝트에 SSL 인증서를 적용하여 클라이언트와 통신할 수 있도록 구현해보겠습니다.
큰 구조는 [지난 글]({% post_url 2020-12-14-SpringBoot-Netty-2-SocketServer %})의 코드를 따르며, 본 글에서는 추가되는 부분만 다루겠습니다.

## 사전 작업: SSL 인증서 발급 및 추가
### 서버에 인증서 적용
#### 무료 인증서 발급
SSL 인증을 적용하려면 SSL 인증서를 발급 받아야 합니다. 운영 환경에서는 공인된 기관에서 발급 받은 유료 인증서를 사용해야 하지만, 본 글에서는 테스트를 위해 90일 무료로 사용할 수 있는 인증서를 사용하겠습니다.
> 💡 무료 인증서는 [ZeroSSL](https://zerossl.com/) 사이트에서 발급 받았습니다. UI가 사용하기 매우 쉽게 되어 있으므로 자세한 설명은 생략하겠습니다. <br/>

#### PKC8로 변환
[네티 공식 문서](https://netty.io/wiki/sslcontextbuilder-and-private-key.html)에 따르면 PKCS8 keys만 지원합니다.
ZeroSSL에서 다운 받은 파일 중 `private.key` 파일을 PKCS8로 바꾸는 작업을 해줍니다. 명령 프롬프트(cmd)에서 [네티 공식 문서](https://netty.io/wiki/sslcontextbuilder-and-private-key.html)에 나와 있는 아래의 명령어를 입력합니다.
```shell
openssl pkcs8 -topk8 -nocrypt -in private.key -out pkcs8_key.pem
```

#### 서버 프로젝트에 추가
위의 작업을 마치면 `.crt` 파일과 `.pem` 파일이 생성됩니다. 이 두 파일을 서버 프로젝트의 `src/main/resources` 폴더에 넣어줍니다. 그 다음, 설정 파일(`application.yml`)에 `.crt`와 `.pem`을 추가해줍니다.
(꼭 아래와 같은 형식은 아니어도 됩니다!)
```yaml
ssl:
  cert-name: sample_net.crt
  key-name: sample_net.pem
```

### truststore에 인증서 등록
`truststore`는 클라이언트에서 서버의 인증서를 확인하기 위해 사용합니다. SSL 인증서를 `truststore`에 등록하는 방법은 [무중력 물고기](https://zero-gravity.tistory.com/199)님의 글을 보고 큰 도움 받았습니다.
앞서 저희는 인증서를 다운 받아 두었으므로, 글의 **3. cacerts 파일에 keystore를 추가하는 방법**을 참고하여 keystore를 추가합니다.
```shell
keytool -import -alias <alias> -file <cerFileName>.cer -keystore cacerts –storepass changeit
```
> 💡 참고로 위 명령어에서 cacerts 파일은 `C:\Program Files\Java\jre{version}\lib\security` 경로에 있습니다. 제일 마지막 `changeit`은 디폴트 비밀번호입니다.

여기까지 완료했다면, 사전 작업은 끝입니다.

## SSL 인증서 적용 에코 서버 구현
### 프로젝트 구조

![Spring-Netty_프로젝트_구조도](/assets/images/structure/netty/spring-netty-ssl.png) <br/>
기본 구조에서 크게 달라진 점은 없습니다. 보안 설정을 위한 `SecurityConfiguration` 클래스를 추가했고, 위의 사전 작업에서 `resources` 디렉토리에 인증서가 추가되었습니다.

## 소스 코드
### config package
#### SecurityConfiguration
```java
import io.netty.handler.ssl.SslContext;
import io.netty.handler.ssl.SslContextBuilder;
import lombok.RequiredArgsConstructor;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.core.io.ClassPathResource;

import java.io.IOException;

@RequiredArgsConstructor
@Configuration
public class SecurityConfiguration {
    @Value("${ssl.cert-name}")
    private String certName;
    @Value("${ssl.key-name}")
    private String keyName;

    @Bean
    public SslContext sslContext() {
        ClassPathResource certPath = new ClassPathResource(certName);
        ClassPathResource keyPath = new ClassPathResource(keyName);

        SslContext sslContext = null;
        try {
            sslContext = SslContextBuilder.forServer(certPath.getInputStream(), keyPath.getInputStream()).build();
        }
        catch (IOException e) {
            e.printStackTrace();
        }
        return sslContext;
    }
}
```
+ `@Value` 어노테이션으로 스프링의 설정 파일(`application.yml` 혹은 `application.properties`)에 설정한 인증서 파일 이름을 불러옵니다.
+ `SslContext`는 네티에서 `SSLEngine`, `SslHandler`의 팩토리로 동작하는 구현체입니다. 사전에 준비한 인증서로 `SslContext` 객체를 만들어 빈으로 등록해줍니다.
+ `ClassPathResource` 클래스로 스프링의 resource(여기서는 인증서)에 접근합니다. `getFile()` 메소드를 사용해도 되지만, resource 파일이 jar 안에 있으면 제대로 동작하지 않을 수 있습니다. 운영 환경을 생각하여 `getInputStream()`을 사용하겠습니다.

### socket package
#### NettyChannelInitializer
```java
import com.ihope9.netty.decoder.TestDecoder;
import com.ihope9.netty.handler.TestHandler;
import io.netty.channel.ChannelInitializer;
import io.netty.channel.ChannelPipeline;
import io.netty.channel.socket.SocketChannel;
import io.netty.handler.ssl.SslContext;
import io.netty.handler.ssl.SslHandler;
import lombok.RequiredArgsConstructor;
import org.springframework.stereotype.Component;

import java.util.concurrent.TimeUnit;

@Component
@RequiredArgsConstructor
public class NettyChannelInitializer extends ChannelInitializer<SocketChannel> {
    private final TestHandler testHandler;
    private final SslContext sslContext;

    @Override
    protected void initChannel(SocketChannel ch) {
        ChannelPipeline pipeline = ch.pipeline();

        // decoder는 @sharable이 안 됨, 매번 새로운 객체 생성해야 함
        TestDecoder testDecoder = new TestDecoder();

        // SSL 적용
        SslHandler sslHandler = sslContext.newHandler(ch.alloc());
        sslHandler.setHandshakeTimeout(60000, TimeUnit.MILLISECONDS);
        pipeline.addLast(sslHandler);

        // 뒤이어 처리할 디코더 및 핸들러 추가
        pipeline.addLast(testDecoder);
        pipeline.addLast(testHandler);
    }
}
```
+ `SecurityConfiguration`에서 빈으로 등록한 `SslContext`를 주입 받습니다. `SslContext`에서 핸들러를 생성하여 `ChannelPipeline`에 등록해줍니다.

## 테스트
SSL 인증을 적용한 네티 서버 구현을 완료했습니다. 이제 의도한 바와 같이 클라이언트와 통신이 되는지 확인해보겠습니다.<br/>
클라이언트 역시 지난 글에서 구현한 소스를 바탕으로 구현합니다.
### SSL 미적용 클라이언트
우선 테스트를 위해 SSL 인증을 적용하지 않은 기존 클라이언트로 데이터를 전송해보겠습니다.<br/>
![Spring-Netty_SSL_테스트_1](/assets/images/test-console/netty/spring-netty-ssl-test-1.png) <br/>
네티 서버의 핸들러가 클라이언트가 전송한 패킷에서 SSL/TLS 레코드를 디코딩할 수 없다는 오류가 나타납니다.

### SSL 적용 클라이언트
[지난 글]({% post_url 2020-12-14-SpringBoot-Netty-2-SocketServer %})을 토대로 SSL을 적용한 클라이언트를 구현해보겠습니다.
#### 클라이언트 구조도
![Spring-Socket_클라이언트 구조도](/assets/images/structure/netty/spring-netty-client-2.png)<br/>
지난 구조에서 `SslSocket` 클래스를 추가했습니다.

#### 클라이언트 소스 코드
클라이언트 코드는 자바에서 제공하는 `SSLSocket` 클래스로 구현했습니다. 이미 많은 블로그에서 자세하게 다루고 있으니 별도의 설명은 하지 않겠습니다.
##### ClientSocketApplication
```java
import com.ihope9.client.sockets.SslSocket;

import java.util.Scanner;


public class ClientSocketApplication {

    public static void main(String[] args) throws InterruptedException {
        String host = "127.0.0.1";
        int port = 9999;
        try {
            System.out.println("Enter message length: ");
            Scanner sc = new Scanner(System.in);
            int messageLength = Integer.parseInt(sc.nextLine());

//            NonSslSocket socket = new NonSslSocket(host, port);
            SslSocket socket = new SslSocket(host, port);
            socket.run(messageLength);
        }
        catch (Exception e) {
            e.printStackTrace();
        }
    }
}
```

##### SslSocket
```java
import lombok.AllArgsConstructor;

import javax.net.ssl.SSLSocket;
import javax.net.ssl.SSLSocketFactory;
import java.io.IOException;

@AllArgsConstructor
public class SslSocket {
    private String host;
    private int port;

    public void run(int messageLength) {
        try {
            System.setProperty("javax.net.ssl.trustStore", "C:\\Program Files\\Java\\jre1.8.0_271\\lib\\security\\cacerts");
            System.setProperty("javax.net.ssl.trustStorePassword", "changeit");

            SSLSocketFactory factory = (SSLSocketFactory) SSLSocketFactory.getDefault();
            SSLSocket sslSocket = (SSLSocket) factory.createSocket(host, port);
            ClientSocket clientSocket = new ClientSocket(sslSocket);

            clientSocket.sendFixedLength(messageLength);
        } catch (IOException e) {
            e.printStackTrace();
        }
    }
}
```

#### 서버-클라이언트 통신
이제 통신을 해봅니다!! <br/>
![Spring-Netty_SSL_테스트_2](/assets/images/test-console/netty/spring-netty-ssl-test-2.png) <br/>
서버와 클라이언트 양측 다 오류 없이 성공적으로 통신했습니다👏. 위의 콘솔은 클라이언트가 서버로부터 응답 받은 데이터입니다.

> 💥 만약 서버에서 `javax.net.ssl.SSLHandshakeException: Received fatal alert: certificate_unknown`, 클라이언트에서 `un.security.provider.certpath.SunCertPathBuilderException: unable to find valid certification path to requested target`와 같은 예외가 뜬다면,
> + cacerts에 인증서를 제대로 추가했는지 확인해봅니다. 뒤에 `findstr` 명령을 연결하면 특정 이름의 인증서만 찾아볼 수 있습니다. <br/>
> `keytool -keystore cacerts -storepass changeit -list -v | findstr "{CERT_NAME}"`
> + 클라이언트 `System.setProperty()` 경로를 확인합니다. `cacert`와 `cacerts`는 다른 파일이니 잘 확인해줍니다.


***

지금까지 스프링-네티 에코 서버에 SSL 인증을 적용해보았습니다. 네티에서 제공하는 `SslContext`를 활용하여 쉽게 구현할 수 있습니다. 그럼에도 개인적으로 어려움을 많이 겪었던 부분이라😅 많은 분들께 도움이 되기를 바랍니다.

<!--more-->

---
