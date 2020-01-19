---
title: 客户端
caption: 配置客户端
category: clients
permalink: /clients/http-client/quick-start/client.html
redirect_from:
- /clients/http-client/calls/client.html
ktor_version_review: 1.3.0
---

## 添加引擎依赖

在使用客户端之前需要做的第一件事就是添加客户端引擎依赖。客户端引擎是执行来自 ktor API 请求的请求执行器。有许多开箱即用的客户端引擎分别用于各个平台： [`Apache`](/clients/http-client/engines.html#apache)、
[`OkHttp`](/clients/http-client/engines.html#okhttp)、
[`Android`](/clients/http-client/engines.html#android)、
[`Ios`](/clients/http-client/engines.html#ios)、
[`Js`](/clients/http-client/engines.html#js-javascript)、
[`Jetty`](/clients/http-client/engines.html#jetty)、
[`CIO`](/clients/http-client/engines.html#cio) 以及 [`Mock`](/clients/http-client/testing.html)。 更多内容请参见[多平台](/clients/http-client/multiplatform.html)部分。

例如可以在 `build.gradle` 添加 `CIO` 引擎依赖，如下所示：

```kotlin
dependencies {
    implementation("io.ktor:ktor-client-cio:$ktor_version")
}
```

## 创建客户端

接下来可以按以下方式创建客户端：

```kotlin
val client = HttpClient(CIO)
```

其中 `CIO` 是引擎类。如果你不确定该使用哪个引擎类，请考虑使用 `CIO`。

如果用于多平台，那么可以省略引擎：

```kotlin
val client = HttpClient()
```

Ktor 会使用 JVM 上的 `ServiceLoader` 或者其他平台上的类似方式从所包含的构件中选择一个可用的引擎。如果依赖中有多个引擎，那么 Ktor 会按引擎名称的字母顺序选择第一个。

创建多个客户端实例或者将同一客户端用于多次请求都是安全的。

## 释放资源

Ktor 客户端持有一些资源：已就绪的线程、协程以及连接。使用完客户端后，可能希望通过调用 `close` 将其关闭：

```kotlin
client.close()
```

如果只想使用客户端发出一个请求，那么可以考虑使用 `use`。一旦传入的代码块执行完毕，客户端会自动关闭：

```kotlin
val status = HttpClient().use { client ->
    ……
}
```

`close` 方法会发出信号以停止执行新的请求。它并不会阻止任何当前请求并会等待所有当前请求成功完成，然后释放资源。也可以使用 `join` 方法等待关闭，或者使用 `cancel` 方法停止任何活动。例如：

```kotlin
try {
    // 关闭并等待 3 秒钟。
    withTimeout(3000) {
        client.close()
        client.join()
    }
} catch (timeout: TimeoutCancellationException) {
    // 超时后取消
    client.cancel()
}
```

Ktor HttpClient 遵循 `CoroutineScope` 生命周期。可查阅[协程指南](https://kotlin.github.io/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines/-coroutine-scope/)了解更多内容。

## 客户端配置

如需配置客户端，可以将附加功能参数传给客户端构造函数。客户端以 [HttpClientEngineConfig](https://api.ktor.io/{{ site.ktor_version }}/io.ktor.client.engine/-http-client-engine-config/index.html) 配置。

例如可以限制线程数（`threadCount`）或者设置[代理服务器](/clients/http-client/features/proxy.html)：

```kotlin
val client = HttpClient(CIO) {
    threadCount = 2
}
```

还可以在代码块中使用 `engine` 方法配置引擎：

```kotlin
val client = HttpClient(CIO) {
    engine {
        // 引擎配置
    }
}
```

更多细节请参见[引擎](/clients/http-client/engines.html)部分。

接下来是[准备请求](/clients/http-client/quick-start/requests.html)。
