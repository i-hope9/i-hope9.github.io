---
layout: article
title: Spring Boot + Netty TCP 소켓 서버 (4) Channel Attribute로 Context 유지
tags: [java, spring, netty, tcp/ip]
aside:
    toc: true
---

[네티 프레임워크 선택 배경]({% post_url 2020-12-08-SpringBoot-Netty-1-Background %}) 글에서 **네티가 채널 내에 컨텍스트 유지가 가능**하다고 소개했습니다. <br/>
이는 네티 프레임워크가 제공하는 [Attribute](https://netty.io/4.1/api/io/netty/util/Attribute.html, "Interface Attribute<T>") 인터페이스 덕분입니다.
`Attribute`는 `value reference`를 저장할 수 있도록 합니다. 서버와 클라이언트에 연결이 성사되어 채널이 생성되면, 채널에 `Attribute`를 저장할 수 있습니다.
`Attribute`를 저장하고 접근할 때는 [AttributeKey](https://netty.io/4.1/api/io/netty/util/AttributeKey.html, "Class AttributeKey<T>") 클래스를 사용합니다.


## 사전 작업: SSL 인증서 발급 및 추가
### 서버에 인증서 적용
#### 무료 인증서 발급


***

<!--more-->

---
