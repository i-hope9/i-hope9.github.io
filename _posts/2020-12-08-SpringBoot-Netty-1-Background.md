---
layout: article
title: Spring Boot + Netty TCP 소켓 서버 (1) 프레임워크 선택 배경
tags: [java, spring, netty]
aside:
    toc: true
---

## 배경
회사에서 데이터 동기화를 위한 RESTful API를 개발하고 있습니다. 모바일이나 PC 애플리케이션에서 API를 활용할 때는 큰 이슈는 없었습니다. 하지만 LTE 망을 사용하는 기기에서 HTTPS 프로토콜 기반의 기존 API를 사용하려고 하니 문제가 생깁니다.
우선 해당 기기는 **한 번 연결을 하기가 너무 어렵**습니다. LTE 특성 때문인지 연결하기가 힘들어 한 번의 요청-응답으로 연결이 종료되는 HTTP(S) 프로토콜은 적절하지 않았습니다.
또, 이 기기는 **한 번에 보낼 수 있는 데이터의 양이 한정**되어 있습니다. 필수 데이터만 보낸다 해도 여러 조각으로 나누어 보내야 합니다. 기존의 API는 데이터를 JSON 형식으로 주고 받는데, JSON 형식을 위한 `""`나 `{}`와 같은 기호를 넣기에는 낭비였습니다.

이러한 배경에서 우리만의 프로토콜을 만들고, 프로토콜을 따르는 서버와 클라이언트를 개발하기로 했습니다.

## 요구사항
+ Spring Boot 위에서 동작하는 서버
    - 기존 서버를 Spring Boot 기반의 멀티 프로젝트로 개발, Spring Boot에 익숙함
    - 주요 로직이 core 프로젝트에 구현됨. TCP 소켓으로 데이터를 받아서 core 프로젝트의 `Bean`을 주입 받아 활용할 수 있다면? 개발 시간이 많이 단축되는 효과
+ 데이터 순서가 보장되는 TCP 기반
+ 보안을 위해 SSL 지원
+ 한 번 연결되면 (일부러 연결을 끊지 않는 한) 끊기지 않아야 함
+ 데이터를 최적화해서 통신 (필요한 경우 압축)
+ 한 번에 여러 기기와 통신이 가능해야 함
+ 한 연결 안에 세션이 있고, 로그인-로그아웃 개념으로 세션을 관리해야 함

## 왜 Netty인가
Netty는 위의 요구사항을 충족하기에 적합했어요.
+ Spring Boot에서 Netty 서버를 구현할 수 있음 (참고 자료도 많음)
+ TCP, SSL 지원. 특히 byte stream 데이터를 쉽게 읽어올 수 있도록 제공하는 `ByteBuf` 클래스가 아주 편리함!
+ 자바 NIO 사용을 쉽게 할 수 있도록 지원하여 한 번에 여러 기기와 통신 가능
+ 통신(`Channel`) 내 `Context` 유지 가능, 즉 세션 관리 가능
처음에는 일반적인 자바 소켓 프로그래밍으로 TCP 서버를 멀티 쓰레드로 구현할까 했습니다. 이번 프로젝트는 데모이기도 했고, 새로운 프레임워크를 학습하려면 시간이 필요하니까요.
하지만 위의 장점 때문에 Netty를 선택했고, 학습하고 나니 오히려 개발 시간을 단축시켜 주었습니다.

하면서 겪은 시행착오를 하나씩 블로그에 기록해둘까 합니다.

## 참고자료
+ 역시 공식 문서가 최고입니다. A4 8장 분량의 User guide만 꼼꼼히 읽어도 많은 부분을 파악할 수 있습니다. [🔗](https://netty.io/wiki/user-guide-for-4.x.html, "Netty.docs: User guide")
+ Spring Boot로 TCP 서버 구축하는 방법을 다양하게 다룹니다. 왜 Netty가 편리한지 BIO, NIO 서버 구축부터 차근차근 설명해줍니다. [🔗](https://programmer.help/blogs/spring-boot-build-tcp-server.html, "Spring Boot Build TCP Server")
+ Spring Boot+Netty로 구현한 채팅 서버입니다. 깃허브에 올려주신 코드를 보며 매우 큰 도움을 받았습니다.  [🔗](https://www.manty.co.kr/bbs/detail/develop?id=69, "Spring Boot 와 Netty 의 아슬아슬한 동거")

<!--more-->

---
