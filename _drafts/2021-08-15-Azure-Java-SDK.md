---
layout: article
title: Azure SDK for Java - Implementation
tags: [cloud, azure]
aside:
    toc: true
---

원문: [Azure SDKs - Java Guidelines](https://azure.github.io/azure-sdk/java_implementation.html)

## API Implementation
이 문서는 Azure SDK 클라이언트 라이브러리를 구현하기 위한 가이드라인입니다. 이 가이드라인의 일부는 코드 생성 도구(code generation tools)에 의해 자동으로 적용됩니다.

⛔️ 구현 코드(즉, public API의 일부를 구성하지 않은 코드)를 public API로 오인하지 않도록 하십시오. 구현 코드를 위한 두 가지 약정이 있는데, 우선 순위는 다음과 같습니다.
1. 구현 클래스를 package-private으로 만들고, 이를 사용하는 클래스(consuming class)와 같은 패키지 내에 배치할 수 있습니다.
2. 구현 클래스를 `implementation`으로 명명된 서브패키지 내에 배치할 수 있습니다.

CheckStyle 검사는 `implementation` 패키지 내의 클래스가 public API를 통해 노출되지 않도록 확인합니다. 하지만 처음부터 API를 public으로 구현하지 않는 것이 좋으므로, 가능하면 package-private를 적용하는 것이 더 나은 방법입니다.

### Service Client
#### Async Service Client
⛔️ 비동기 클라이언트 라이브러리 내에 blocking calls를 포함하지 마십시오. 비동기 API 안의 blocking calls를 감지하기 위해 [BlockHound](https://github.com/reactor/BlockHound)를 사용하십시오.

> 📌 BlockHound란?

#### Annotations
서비스 클라이언트 클래스에는 다음의 어노테이션을 포함하십시오. 아래 코드에서 다음 두 어노테이션을 사용하는 예시 클래스를 볼 수 있습니다.
```java
@ServiceClient(builder = ConfigurationAsyncClientBuilder.class, isAsync = true, service = ConfigurationService.class)
public final class ConfigurationAsyncClient {

    @ServiceMethod(returns = ReturnType.COLLECTION)
    public Mono<Response<ConfigurationSetting>> addSetting(String key, String value) {
        // ...
    }
}
```

| 어노테이션          | 위치  | 설명 |
|------------------|------|------|
| `@ServiceClient` |Service Client|서비스 클라이언트를 인스턴스화하는 빌더, API가 비동기인지 여부, 서비스 인터페이스(`@ServiceInterface` 어노테이션이 있는 인터페이스)에 대한 참조를 명시합니다.|
| `@ServiceMethod` |Service Method|클라이언트 클래스인지 여부에 관계 없이, 네트워크 작업을 수행하는 모든 메서드에 명시합니다.|

#### Service Client Builder
##### Annotations
`@ServiceClientBuilder` 어노테이션은 서비스 클라이언트 인스턴스화를 담당하는 클래스에 반드시 명시되어야 합니다. 즉, `@ServiceClient` 어노테이션이 적용된 클래스를 인스턴스화하는 클래스에 배치되어야 합니다. 예시는 다음과 같습니다.
```java
@ServiceClientBuilder(serviceClients = {ConfigurationClient.class, ConfigurationAsyncClient.class})
public final class ConfigurationClientBuilder { ... }
```
위의 빌더는 `ConfigurationClient` 및 `ConfigurationAsyncClient`의 인스턴스를 작성할 수 있다고 명시합니다.

### Supporting Types
#### Model Types
##### Annotations
다음 조건에 해당하는 경우, 모델 클래스에 적용해야 하는 두 가지 어노테이션이 있습니다.
* `@Fluent` 어노테이션은 최종 사용자에게 fluent API를 제공할 것으로 예상되는 모든 모델 클래스에 적용됩니다.
* `@Immutable` 어노테이션은 모든 불변(immutable) 클래스에 적용됩니다.

> 📌 Fluent API란? 사용자가 이해하고 사용하기 쉬운 API를 제공하기 위한 디자인 패턴. 메소드 체이닝을 지원하는 등의 방법으로 구현한다. (??)

## SDK Feature Implementation
### Logging

### Distributed tracing

## Testing

<!--more-->

---
