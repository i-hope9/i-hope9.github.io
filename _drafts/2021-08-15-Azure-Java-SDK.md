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

CheckStyle 검사는 `implementation` 패키지 내의 클래스가 public API를 통해 노출되지 않도록 확인합니다. 하지만 우선 API를 public으로 구현하지 않는 것이 좋으므로, 가능하면 package-private를 적용하는 것이 더 낫습니다.

### Service Client
#### Async Service Client
⛔️ 비동기 클라이언트 라이브러리 내에 blocking calls를 포함하지 마십시오. 비동기 API 안의 blocking calls를 감지하기 위해 [BlockHound](https://github.com/reactor/BlockHound)를 사용하십시오.

> 📌 BlockHound란? Non-blocking 쓰레드에 blocking calls가 쓰였는지 감지하는 자바 에이전트. Reactor에서 제공.

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
조건에 해당하는 경우, 모델 클래스에 적용해야 하는 두 가지 어노테이션이 있습니다.
* `@Fluent` 어노테이션은 최종 사용자에게 유연한(fluent) API를 제공할 것으로 예상되는 모든 모델 클래스에 적용됩니다.
* `@Immutable` 어노테이션은 모든 불변(immutable) 클래스에 적용됩니다.

> 📌 Fluent API란? 사용자가 이해하고 사용하기 쉬운 API를 제공하기 위한 디자인 패턴. 메소드 체이닝을 지원하는 등의 방법으로 구현한다. (??)

## SDK Feature Implementation
### Logging
클라이언트 라이브러리는 azure core의 강력한 로깅 메커니즘을 사용해야 합니다. 이는 사용자(consumer)가 메서드 호출의 문제를 적절하게 진단하고, 그 문제가 개발자 코드, 클라이언트 라이브러리 코드, 서비스 중 어디에서 발생했는지 확인할 수 있도록 하기 위함입니다.

✅ 모든 클라이언트 라이브러리에서는 Azure Core가 제공하는 `ClientLogger` API를 유일한 로깅 API로 사용하십시오. 내부적으로 `ClientLogger`는 SLF4J를 래핑하므로, SLF4J를 통해 제공되는 모든 외부 구성(configuration)이 유효합니다. 최종 사용자에게 SLF4J 구성을 노출하는 것이 좋습니다. 자세한 내용은 [SLF4J](http://www.slf4j.org/manual.html) 설명서를 참조하십시오.

✅ 모든 관련 클래스에 `ClientLogger` 인스턴스를 생성하십시오. 단, 성능이 매우 중요하거나, 인스턴스의 수명이 짧아 고유한 로거를 사용하기에 비용이 과하거나, 클래스 인스턴스화가 허용되지 않는 static-only 클래스는 예외입니다. 이러한 예외의 경우, 공유 또는 static 로거 인스턴스가 허용됩니다. 예를 들어, 아래 코드는 `ConfigurationAsyncClient`에 대한 `ClientLogger`를 생성합니다.

```java
public final class ConfigurationAsyncClient {
    private final ClientLogger logger = new ClientLogger(ConfigurationAsyncClient.class);

    // example async call to a service that uses the Project Reactor APIs to log request, success, and error
    // information out to the service logger instance
    public Mono<Response<ConfigurationSetting>> setSetting(ConfigurationSetting setting) {
        return service.setKey(serviceEndpoint, setting.key(), setting.label(), setting, getETagValue(setting.etag()), null)
            .doOnRequest(ignoredValue -> logger.info("Setting ConfigurationSetting - {}", setting))
            .doOnSuccess(response -> logger.info("Set ConfigurationSetting - {}", response.value()))
            .doOnError(error -> logger.warning("Failed to set ConfigurationSetting - {}", setting, error));
    }
}
```

static 로거는 JVM 인스턴스에서 실행되는 모든 클라이언트 라이브러리 인스턴스 간에 공유됩니다. static 로거는 인스턴스 수명이 짧은 경우에만 신중히 사용해야 합니다.

✅ 로그를 내보낼 때는 다음 로그 레벨 중 하나를 사용하십시오: `Verbose`(상세 정보), `Informational`(발생한 상황), `Warning`(문제일 수 있는 상황), `Error`.

✅ `Error` 레벨은 응용 프로그램이 복구할 가능성이 거의 없는 오류(메모리 부족 등)에 사용하십시오.

✅ `Warning` 레벨은 함수가 의도한 작업을 수행하지 못한 경우 사용하십시오. 이는 일반적으로 함수가 예외를 발생시킨다는 것을 의미합니다. self-healing 이벤트 발생(예: 요청이 자동으로 재시도되는 경우)은 포함하지 마십시오.

✔️ 요청/응답 주기(응답 body 시작까지)가 서비스 정의 임계값(service-defined threshold)을 초과하면 `Warning` 수준에서 요청 및 응답을 기록할 수 있습니다. false-positives(*실제로는 문제가 아닌데 문제로 진단되는 경우)를 최소화하고, 서비스 문제를 식별하려면 임계값을 선택해야 합니다.

✅ `Informational` 레벨은 함수가 정상적으로 작동할 때 사용하십시오.

✅ `Verbose` 레벨은 자세한 문제 해결 시나리오를 위해 사용하십시오. 이는 주로 개발자 혹은 시스템 관리자가 특정 오류를 진단하기 위한 것입니다.

✅ 승인된 헤더와 쿼리 파라미터의 서비스 제공 "허용 목록(allow-list)"에 있는 헤더 및 쿼리 파라미터만 로그로 기록하십시오. 다른 모든 헤더와 쿼리 파라미터는 해당 값을 수정해야 합니다.

✅ `Informational` 메세지로 요청 라인과 헤더를 로그에 남기십시오. 로그는 다음의 정보를 포함해야 합니다:
* HTTP 메서드
* URL
* 쿼리 파라미터(허용 목록에 없는 경우 수정)
* 요청 헤더(허용 목록에 없는 경우 수정)
* SDK가 제공하는 (correlation purpose를 위한) 요청 ID
* 요청이 시도된 횟수

이는 기본적으로 azure-core 내에서 수행하지만, 사용자가 `httpLogOptions` 구성(configuration) 빌더를 통해 설정할 수 있습니다.

✅ `Informational` 메세지로 응답 라인과 헤더를 로그에 남기십시오. 로그 형식은 다음과 같아야 합니다:
* SDK에서 제공한 요청 ID(위 참조)
* 상태 코드
* 상태 코드와 함께 제공된 메세지
* 응답 헤더(허용 목록에 없는 경우 수정)
* 요청의 첫 번째 시도와 본문의 첫 번째 바이트 사이의 기간

✅ 서비스 요청이 취소된 경우 `Informational` 로그를 기록하십시오. 로그에는 다음이 포함되어야 합니다:
* SDK에서 제공한 요청 ID(위 참조)
* 취소 사유(가능한 경우)

✅ 예외(exception thrown)는 `Warning` 레벨 메지지로 기록하십시오. 로그 레벨이 `Verbose`로 설정된 경우, stack trace 정보를 메세지에 포함하십시오.
(_DO log exceptions thrown as a Warning level message. If the log level set to Verbose, append stack trace information to the message._)

✅ 클라이언트 라이브러리 코드 내에서 발생한 모든 예외는 다음 로거 API 중 하나를 통해 발생시킵시오:
* `ClientLogger.logThrowableAsError()`
* `ClientLogger.logThrowableAsWarning()`
* `ClientLogger.logExceptionAsError()`
* `ClientLogger.logExceptionAsWarning()`

아래 예시가 있습니다.
```java
// ⛔️ NO!!!!
if (priority != null && priority < 0) {
    throw new IllegalArgumentException("'priority' cannot be a negative value. Please specify a zero or positive long value.");
}

// ✅ Good

// Log any Throwable as error and throw the exception
if (!file.exists()) {
    throw logger.logThrowableAsError(new IOException("File does not exist " + file.getName()));
}

// Log any Throwable as warning and throw the exception
if (!file.exists()) {
    throw logger.logThrowableAsWarning(new IOException("File does not exist " + file.getName()));
}

// Log a RuntimeException as error and throw the exception
if (priority != null && priority < 0) {
    throw logger.logExceptionAsError(new IllegalArgumentException("'priority' cannot be a negative value. Please specify a zero or positive long value."));
}

// Log a RuntimeException as warning and throw the exception
if (numberOfAttempts < retryPolicy.getMaxRetryCount()) {
    throw logger.logExceptionAsWarning(new RetryableException("A transient error occurred. Another attempt will be made after " + retryPolicy.getDelay()));
}
```

### Distributed tracing
분석 추적 메커니즘(Distributed tracing mechanisms)을 통해 사용자는 그들의 코드를 프론트엔드부터 백엔드까지 추적할 수 있습니다. 분산 추적 라이브러리는 고유한 작업 단위인 범위(span)를 생성합니다. 각각의 범위는 부모-자식 관계에 있습니다. 코드 계층 구조에 더 깊이 들어갈수록 더 많은 범위가 생성됩니다. 그 다음, 필요에 따라 이러한 범위를 적합한 수신자(receiver)로 내보낼 수 있습니다. 범위를 추적하기 위해, _distributed tracing context_(분산 추적 컨텍스트, 이하 컨텍스트)는 각 연속 계층으로 전달됩니다. 자세한 정보는 [OpenTelemetry](https://opentelemetry.io)의 추적 항목을 참조하십히오.

✅ HTTP 파이프라인 인스턴스화의 일부로써 플러그형 파이프라인 정책을 지원하십시오. 이를 통해 개발자는 추적 플러그인을 포함하고, 해당 파이프라인 정책을 사용 중인 모든 클라이언트 라이브러리에 자동으로 주입할 수 있습니다.

아래의 예시 코드를 검토해보십시오. 이 코드에서는 서비스 클라이언트 빌더가 해당 정책에 `HttpPipeline`을 생성하고 있습니다. 동시에, 빌더는 `HttpPolicyProviders.addBeforeRetryPolicies(policies)` 및 `HttpPolicyProviders.addAfterRetryPolicies(policies)` 행을 사용하여 플러그인이 ‘before retry’와 ‘after retry’ 정책을 추가할 수 있도록 허용합니다.

```java
public ConfigurationAsyncClient build() {
    ...

    // Closest to API goes first, closest to wire goes last.
    final List<HttpPipelinePolicy> policies = new ArrayList<>();
    policies.add(new UserAgentPolicy(AzureConfiguration.NAME, AzureConfiguration.VERSION, buildConfiguration));
    policies.add(new RequestIdPolicy());
    policies.add(new AddHeadersPolicy(headers));
    policies.add(new AddDatePolicy());
    policies.add(new ConfigurationCredentialsPolicy(buildCredentials));
    HttpPolicyProviders.addBeforeRetryPolicies(policies);
    policies.add(retryPolicy);
    policies.addAll(this.policies);
    HttpPolicyProviders.addAfterRetryPolicies(policies);
    policies.add(new HttpLoggingPolicy(httpLogDetailLevel));

    ...
}
```

✅ 추적 범위와 함께 제공해야 하는 추가 메타데이터를 설정하려면 Azure core `TracerProxy` API를 사용하십시오. 특히 `setAttribute(String key, String value, Context context)` 메서드를 사용하여 추적 컨텍스트에서 새 키/값 쌍을 설정합니다.
> 📌 [TracerPolicy](https://github.com/Azure/azure-sdk-for-java/blob/main/sdk/core/azure-core/src/main/java/com/azure/core/util/tracing/TracerProxy.java)

✅ 서비스 메서드 인수로 받은 Context 객체 전부를 모든 경우에 생성된 서비스 인터페이스 메서드로 전달하십시오.

## Testing
✅ 사용 가능한 모든 HTTP 클라이언트와 서비스 버전을 사용하려면, 적용 가능한 모든 단위 테스트를 매개변수화(parameterize)하십시오. 모든 테스트의 매개변수화된 실행은 라이브 테스트의 일부로 이루어져야 합니다. Netty와 최신 서비스 버전으로 구성된 짧은 실행은 PR 유효성 검사가 발생할 때마다 실행할 수 있습니다. (??)
> 📌 `@ParameterizedTest` 어노테이션을 적용한 단위 테스트를 의미하는 것으로 보임.

_이하 TODO_
<!--more-->

---
