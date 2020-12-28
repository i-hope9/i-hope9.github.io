---
layout: article
title: Spring Boot + Netty TCP 소켓 서버 (2) 기본 구조 구현
tags: [java, spring, netty]
aside:
    toc: true
---

## 프로젝트 생성하기
업무에서 구현한 서버는 gradle 멀티 프로젝트로 구현되어 있습니다. 그리고 네티 소켓 서버는 하위 프로젝트 중 하나입니다. 본 글에서는 단일 프로젝트로 생성하는 방법으로 작성하겠습니다.<br/>
스프링 프레임워크에 네티를 적용하는 전체 구조는  [만티스쿠바님의 깃허브](https://github.com/zbum/netty-spring-example)를 참고했습니다.
그 안에서 동작하는 네트의 구성 요소는 [Netty 공식 홈페이지](https://netty.io/wiki/user-guide-for-4.x.html)를 참고했습니다. <br/>

### build.gradle

```groovy
dependencies {
    compile group: 'org.springframework.boot', name: 'spring-boot-starter', version: '2.4.0'
    compile group: 'io.netty', name: 'netty-all', version: '4.1.24.Final'

    testCompile group: 'junit', name: 'junit', version: '4.12'

    compileOnly 'org.projectlombok:lombok:1.18.12'
    annotationProcessor 'org.projectlombok:lombok:1.18.12'
    testCompileOnly 'org.projectlombok:lombok:1.18.12'
    testAnnotationProcessor 'org.projectlombok:lombok:1.18.12'
}
```

### 프로젝트 구조
![Spring-Netty_프로젝트_구조도](/assets/images/structure/spring-netty-basic.png)

## 소스 코드
본 서버는 TCP 기반의 서버로, 스트림 기반(stream-base) 송수신입니다. 이러한 데이터는 소켓 내 `receive buffer` 내에 쌓입니다. 스트림 기반 데이터는 패킷 단위로 데이터가 쌓이지 않고, 바이트로 쌓입니다.
즉, 클라이언트에서 두 개의 패킷을 따로 전송한다고 해도 서버는 이를 하나의 바이트 배열로 받습니다. 따라서 TCP 기반의 서버를 구현할 때는 **쌓여 있는 바이트 배열을 어떤 기준으로 나누어 읽을지 규칙**을 세워야 합니다.<br/>
네티를 사용하며 느낀 강점 중 하나가 바이트 배열을 유의미한 단위로 끊어서 읽을 수 있도록 많은 `decoder` 클래스를 제공한다는 점입니다. `decoder`를 잘 구현하면 클라이언트와 합의한 프로토콜에 맞게 패킷을 읽어올 수 있습니다.<br/>
이 글에서는 클라이언트에서 정해진 길이(2048 bytes)를 하나의 패킷으로 읽어 오는 서버를 예시로 작성하였습니다. (실제 업무에서는 이렇게 규약을 세우는 경우는 거의 없겠지만요😅) <br/>
네티 객체에 대한 설명은 코드 내 주석에 작성했습니다. 클래스는 참고하기 편하도록 구조도 내 패키지 순으로 정리하였습니다.<br/>
### root package
#### NettyServerApplication
```java
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

@SpringBootApplication
public class NettyServerApplication {
    public static void main(String[] args) {
        SpringApplication.run(NettyServerApplication.class, args);
    }
}
```
+ `main()` 메소드가 선언된 클래스로, 스프링 부트가 시작되는 지점입니다.

### config package
#### ApplicationStartupTask
```java
import lombok.RequiredArgsConstructor;
import org.springframework.boot.context.event.ApplicationReadyEvent;
import org.springframework.context.ApplicationListener;
import org.springframework.stereotype.Component;

@Component
@RequiredArgsConstructor
public class ApplicationStartupTask implements ApplicationListener<ApplicationReadyEvent> {

    private final NettyServerSocket nettyServerSocket;

    @Override
    public void onApplicationEvent(ApplicationReadyEvent event) {
        nettyServerSocket.start();
    }
}
```
+ `ApplicationReadyEvent`: 스프링 부트 서비스를 시작 시 초기화하는 코드를 Bean으로 만들 때 사용합니다. 여기서는 네티 서버 소켓을 실행하여 incoming connection을 받을 준비를 합니다.

#### NettyConfiguration
```java
import com.ihope9.netty.handler.NettyChannelInitializer;
import io.netty.bootstrap.ServerBootstrap;
import io.netty.channel.ChannelOption;
import io.netty.channel.nio.NioEventLoopGroup;
import io.netty.channel.socket.nio.NioServerSocketChannel;
import io.netty.handler.logging.LogLevel;
import io.netty.handler.logging.LoggingHandler;
import lombok.RequiredArgsConstructor;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

import java.net.InetSocketAddress;

@Configuration
@RequiredArgsConstructor
public class NettyConfiguration {

    @Value("${server.host}")
    private String host;
    @Value("${server.port}")
    private int port;
    @Value("${server.netty.boss-count}")
    private int bossCount;
    @Value("${server.netty.worker-count}")
    private int workerCount;
    @Value("${server.netty.keep-alive}")
    private boolean keepAlive;
    @Value("${server.netty.backlog}")
    private int backlog;

    @Bean
    public ServerBootstrap serverBootstrap(NettyChannelInitializer nettyChannelInitializer) {
        // ServerBootstrap: 서버 설정을 도와주는 class
        ServerBootstrap b = new ServerBootstrap();
        b.group(bossGroup(), workerGroup())
                // NioServerSocketChannel: incoming connections를 수락하기 위해 새로운 Channel을 객체화할 때 사용
                .channel(NioServerSocketChannel.class)
                .handler(new LoggingHandler(LogLevel.DEBUG))
                // ChannelInitializer: 새로운 Channel을 구성할 때 사용되는 특별한 handler. 주로 ChannelPipeline으로 구성
                .childHandler(nettyChannelInitializer);

        // ServerBootstarp에 다양한 Option 추가 가능
        // SO_BACKLOG: 동시에 수용 가능한 최대 incoming connections 개수
        // 이 외에도 SO_KEEPALIVE, TCP_NODELAY 등 옵션 제공
        b.option(ChannelOption.SO_BACKLOG, backlog);

        return b;
    }

    // boss: incoming connection을 수락하고, 수락한 connection을 worker에게 등록(register)
    @Bean(destroyMethod = "shutdownGracefully")
    public NioEventLoopGroup bossGroup() {
        return new NioEventLoopGroup(bossCount);
    }

    // worker: boss가 수락한 연결의 트래픽 관리
    @Bean(destroyMethod = "shutdownGracefully")
    public NioEventLoopGroup workerGroup() {
        return new NioEventLoopGroup(workerCount);
    }

    // IP 소켓 주소(IP 주소, Port 번호)를 구현
    // 도메인 이름으로 객체 생성 가능
    @Bean
    public InetSocketAddress inetSocketAddress() {
        return new InetSocketAddress(host, port);
    }
}
```
+ 네티 설정을 위한 클래스입니다.
+ `@Value` 어노테이션으로 스프링의 설정 파일(`application.yml` 혹은 `application.properties`)을 읽어 옵니다.

### socket package
#### NettyServerSocket
```java
import io.netty.bootstrap.ServerBootstrap;
import io.netty.channel.Channel;
import io.netty.channel.ChannelFuture;
import lombok.RequiredArgsConstructor;
import lombok.extern.slf4j.Slf4j;
import org.springframework.stereotype.Component;

import javax.annotation.PreDestroy;
import java.net.InetSocketAddress;

@Slf4j
@RequiredArgsConstructor
@Component
public class NettyServerSocket {
    private final ServerBootstrap serverBootstrap;
    private final InetSocketAddress tcpPort;
    private Channel serverChannel;

    public void start() {
        try {
            // ChannelFuture: I/O operation의 결과나 상태를 제공하는 객체
            // 지정한 host, port로 소켓을 바인딩하고 incoming connections을 받도록 준비함
            ChannelFuture serverChannelFuture = serverBootstrap.bind(tcpPort).sync();

            // 서버 소켓이 닫힐 때까지 기다림
            serverChannel = serverChannelFuture.channel().closeFuture().sync().channel();
        }
        catch (InterruptedException e) {
            e.printStackTrace();
        }
    }

    // Bean을 제거하기 전에 해야할 작업이 있을 때 설정
    @PreDestroy
    public void stop() {
        if (serverChannel != null) {
            serverChannel.close();
            serverChannel.parent().closeFuture();
        }
    }
}
```
+ 네티 서버를 실행하는 클래스입니다. 앞서 `ApplicationStartupTask` 클래스에서 스프링 부트 서비스를 시작할 때 본 클래스의 `start()` 메소드를 실행하도록 설정했습니다.

### handler package
#### NettyChannelInitializer
```java
import com.ihope9.netty.decoder.TestDecoder;
import io.netty.channel.ChannelInitializer;
import io.netty.channel.ChannelPipeline;
import io.netty.channel.socket.SocketChannel;
import lombok.RequiredArgsConstructor;
import org.springframework.stereotype.Component;

@Component
@RequiredArgsConstructor
public class NettyChannelInitializer extends ChannelInitializer<SocketChannel> {
    private final TestHandler testHandler;

    // 클라이언트 소켓 채널이 생성될 때 호출
    @Override
    protected void initChannel(SocketChannel ch) {
        ChannelPipeline pipeline = ch.pipeline();

        // decoder는 @sharable이 안 됨, 매번 새로운 객체 생성해야 함
        TestDecoder testDecoder = new TestDecoder();

        // 뒤이어 처리할 디코더 및 핸들러 추가
        pipeline.addLast(testDecoder);
        pipeline.addLast(testHandler);
    }
}
```


_⏳ 업데이트 예정입니다_
<!--more-->

---
