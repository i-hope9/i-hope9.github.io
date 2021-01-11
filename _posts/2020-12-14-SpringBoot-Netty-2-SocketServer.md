---
layout: article
title: Spring Boot + Netty TCP 소켓 서버 (2) 기본 에코 서버 구현
tags: [java, spring, netty, tcp/ip]
aside:
    toc: true
---

## 프로젝트 생성하기
업무에서 구현한 서버는 gradle 멀티 프로젝트로 구현되어 있습니다. 그리고 네티 소켓 서버는 하위 프로젝트 중 하나입니다. 본 글에서는 단일 프로젝트로 생성하는 방법으로 작성하겠습니다.<br/>
스프링 프레임워크에 네티를 적용하는 전체 구조는  [만티스쿠바님의 깃허브](https://github.com/zbum/netty-spring-example)를 참고했습니다.
그 안에서 동작하는 네트의 구성 요소는 [Netty 공식 홈페이지](https://netty.io/wiki/user-guide-for-4.x.html)를 참고했습니다. <br/>

본 서버는 TCP 기반의 서버로, 스트림 기반(stream-base) 송수신입니다. 스트림 기반에서는 데이터가 소켓 내 `receive buffer`에 쌓입니다. 그리고 이 데이터는 패킷 단위로 쌓이지 않고, 바이트 배열로 쌓입니다.
즉, 클라이언트에서 두 개의 패킷을 따로 전송한다고 해도 **서버는 이를 하나의 바이트 배열**로 받습니다. 따라서 TCP 기반의 서버를 구현할 때는 **쌓여 있는 바이트 배열을 어떤 기준으로 나누어 읽을지 규칙**을 세워야 합니다.<br/>

네티를 사용하며 느낀 강점 중 하나가 바로 이 지점에서 나옵니다. 네티는 바이트 배열을 유의미한 단위로 끊어서 읽을 수 있도록 다양한 `decoder` 클래스를 제공합니다. `decoder`를 잘 구현하면 클라이언트와 합의한 프로토콜에 맞게 패킷을 읽어올 수 있습니다.<br/>
이 글에서는 클라이언트에서 정해진 길이(2048 bytes)를 하나의 패킷으로 읽어 오는 서버를 예시로 작성하였습니다. 또한, 읽어온 데이터를 다시 그대로 전송하는 에코 서버로 구현했습니다. (실제 업무에서는 이렇게 규약을 세우는 경우는 거의 없겠지만요😅) <br/>

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

### decoder package
#### TestDecoder
```java
import io.netty.buffer.ByteBuf;
import io.netty.channel.ChannelHandlerContext;
import io.netty.handler.codec.ByteToMessageDecoder;
import lombok.RequiredArgsConstructor;
import org.springframework.stereotype.Component;

import java.util.List;

@Component
@RequiredArgsConstructor
public class TestDecoder extends ByteToMessageDecoder {
    private int DATA_LENGTH = 2048;

    @Override
    protected void decode(ChannelHandlerContext ctx, ByteBuf in, List<Object> out) throws Exception {
        if (in.readableBytes() < DATA_LENGTH) {
            return;
        }

        out.add(in.readBytes(DATA_LENGTH));
    }
}
```
+ 정해진 길이(`DATA_LENGTH`)만큼 데이터가 들어올 때까지 기다립니다.

### handler package
#### TestHandler
```java
import io.netty.buffer.ByteBuf;
import io.netty.channel.ChannelFuture;
import io.netty.channel.ChannelFutureListener;
import io.netty.channel.ChannelHandler;
import io.netty.channel.ChannelHandlerContext;
import io.netty.channel.ChannelInboundHandlerAdapter;
import lombok.RequiredArgsConstructor;
import lombok.extern.slf4j.Slf4j;
import org.springframework.stereotype.Component;

@Slf4j
@Component
@ChannelHandler.Sharable
@RequiredArgsConstructor
public class TestHandler extends ChannelInboundHandlerAdapter {
    private int DATA_LENGTH = 2048;
        private ByteBuf buff;

        // 핸들러가 생성될 때 호출되는 메소드
        @Override
        public void handlerAdded(ChannelHandlerContext ctx) {
            buff = ctx.alloc().buffer(DATA_LENGTH);
        }

        // 핸들러가 제거될 때 호출되는 메소드
        @Override
        public void handlerRemoved(ChannelHandlerContext ctx) {
            buff = null;
        }

        // 클라이언트와 연결되어 트래픽을 생성할 준비가 되었을 때 호출되는 메소드
        @Override
        public void channelActive(ChannelHandlerContext ctx) {
            String remoteAddress = ctx.channel().remoteAddress().toString();
            log.info("Remote Address: " + remoteAddress);
        }


        @Override
        public void channelRead(ChannelHandlerContext ctx, Object msg){
            ByteBuf mBuf = (ByteBuf) msg;
            buff.writeBytes(mBuf);  // 클라이언트에서 보내는 데이터가 축적됨
            mBuf.release();

            final ChannelFuture f = ctx.writeAndFlush(buff);
            f.addListener(ChannelFutureListener.CLOSE);
        }

        @Override
        public void exceptionCaught(ChannelHandlerContext ctx, Throwable cause) {
            // Close the connection when an exception is raised.
            ctx.close();
            cause.printStackTrace();
        }

}
```

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

        // decoder는 @Sharable이 안 됨, Bean 객체 주입이 안 되고, 매번 새로운 객체 생성해야 함
        TestDecoder testDecoder = new TestDecoder();

        // 뒤이어 처리할 디코더 및 핸들러 추가
        pipeline.addLast(testDecoder);
        pipeline.addLast(testHandler);
    }
}
```
+ `initChannel()` 메소드는 클라이언트 소켓 채널이 생성될 때 호출됩니다.

## 테스트
네티 서버 구현을 완료했습니다. 이제 의도한 바와 같이 클라이언트와 통신이 되는지 확인해보겠습니다.<br/>
클라이언트는 네티를 사용하지 않고, 자바의 `Socket` 클래스로 개발했습니다. 테스트를 위해 최대한 간결하게 구현했습니다.
### 클라이언트 구조도
![Spring-Socket_클라이언트 구조도](/assets/images/structure/spring-netty-client-1.png)

### 클라이언트 소스 코드
#### ClientSocketApplication
```java
import com.ihope9.client.sockets.NonSslSocket;

import java.util.Scanner;


public class ClientSocketApplication {

    public static void main(String[] args) throws InterruptedException {
        String host = "127.0.0.1";
        int port = 9999;
        try {
            System.out.println("Enter message length: ");
            Scanner sc = new Scanner(System.in);
            int messageLength = Integer.parseInt(sc.nextLine());

            NonSslSocket socket = new NonSslSocket(host, port);
            socket.run(messageLength);
        }
        catch (Exception e) {
            e.printStackTrace();
        }
    }
}
```
+ 현재 서버에 쓰인 디코더는 정해진 길이만큼 데이터가 들어오기를 기다리도록 구현되어 있습니다. 이를 테스트하기 위해 전송할 메시지 길이를 입력하도록 구현했습니다.

#### ClientSocket
```java
import com.google.common.io.ByteStreams;
import lombok.AllArgsConstructor;

import java.io.IOException;
import java.io.InputStream;
import java.io.OutputStream;
import java.net.Socket;
import java.util.Arrays;

@AllArgsConstructor
public class ClientSocket {
    private Socket socket;

    public void sendFixedLength(int messageLength) {
        int delimiterLength = 256;

        StringBuilder stringBuilder = new StringBuilder();
        for (int i = 0; i < messageLength; i++)
            stringBuilder.append("A");
        byte[] totalData = stringBuilder.toString().getBytes();

        System.out.println("Sending message");
        try {
            OutputStream os = socket.getOutputStream();

            for (int i = 0; i < messageLength / delimiterLength; i++) {
                byte[] sending = Arrays.copyOfRange(totalData, i * delimiterLength, (i + 1) * delimiterLength);
                System.out.println("sending... " + (i + 1));
                os.write(sending);
                os.flush();
                Thread.sleep(500);
            }
        }
        catch (InterruptedException | IOException e) {
            e.printStackTrace();
        }

        System.out.println("Receiving message");
        try {
            InputStream is = socket.getInputStream();

            byte[] reply = new byte[messageLength];
            ByteStreams.read(is, reply, 0, reply.length);

            System.out.println(new String(reply));
        }
        catch (IOException e) {
            e.printStackTrace();
        }
    }
}
```
+ `sendFixedLength()`: 콘솔에서 입력한 길이만큼 문자 'A'를 버퍼에 가득 채워 전송합니다.
+ 서버가 정해진 길이만큼 데이터가 들어오기를 기다리는지 확인하기 위해 바이트 배열을 `delimiterLength`씩 나누어 전송합니다.
+ 보낸 데이터를 그대로 전송하는지 확인하기 위해 `InputStream`으로 데이터를 받습니다.

#### NonSslSocket
```java
import lombok.AllArgsConstructor;

import java.io.IOException;
import java.net.InetSocketAddress;
import java.net.Socket;
import java.net.SocketAddress;

@AllArgsConstructor
public class NonSslSocket{
    private String host;
    private int port;

    public void run(int messageLength) {
        try {
            Socket socket = new Socket();
            SocketAddress address = new InetSocketAddress(host, port);
            socket.connect(address);

            ClientSocket clientSocket = new ClientSocket(socket);
            clientSocket.sendFixedLength(messageLength);
        } catch (IOException e) {
            e.printStackTrace();
        }
    }
}
```
+ SSL을 적용하지 않은 클라이언트 소켓입니다.

### 서버-클라이언트 통신
이제 모든 구현이 끝났습니다. 테스트를 진행해보겠습니다. <br/>

서버 `NettyServerApplication`의 `main()` 메소드를 먼저 실행해봅니다. 콘솔에 `Started NettyServerApplication in ... ` 메세지가 뜬다면 성공적으로 실행됐다는 뜻입니다. <br/>
마찬가지로 `ClientSocketApplication`의 `main()` 메소드를 실행합니다. 전송하려는 메시지 길이를 입력하라는 문구가 뜹니다.
![Spring-Netty_기초_테스트_1](/assets/images/test-console/spring-netty-basic-test-1.png) <br/>
서버 디코더에 설정한 고정된 길이, 2048을 입력해봅니다.
![Spring-Netty_기초_테스트_2](/assets/images/test-console/spring-netty-basic-test-2.png) <br/>
잘 전송되었네요! 에코 서버로부터 다시 받은 메세지도 처음 보낸 그대로(AAAA...)입니다.
![Spring-Netty_기초_테스트_3](/assets/images/test-console/spring-netty-basic-test-3.png) <br/>
서버에서도 클라이언트와 연결되었을 때 호출되는 `channelActive()` 메소드의 로그가 잘 찍혀 있습니다.


만약 정해진 길이 2048 보다 짧은 길이를 입력하면 어떻게 될까요?
![Spring-Netty_기초_테스트_3](/assets/images/test-console/spring-netty-basic-test-4.png) <br/>
서버에서 클라이언트와 연결도 잘 되었고, 클라이언트는 데이터 전송을 성공했지만 응답을 받지 못합니다. 즉, **서버의 디코더에 아직 2048 바이트가 쌓이지 않아** 서버에서 아무런 응답을 하지 않고 있음을 확인할 수 있습니다.

***
지금까지 스프링-네티를 활용한 에코 서버를 구현했습니다. 또한 이를 테스트하기 위한 클라이언트도 간단히 구현해보았습니다.<br/>
스프링-네티 서버의 기본적인 구조를 파악하고, 매우 기본이 되는 디코더를 이해하는 데 도움이 되셨기를 바랍니다🤗!

<!--more-->

---
