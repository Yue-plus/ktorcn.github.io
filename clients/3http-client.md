---
title: Http 客户端
category: clients
permalink: /clients/http-client.html
children: /clients/http-client/
caption: Http 客户端
ktor_version_review: 1.0.0
---

{::options toc_levels="1..2" /}

除了 HTTP 服务外，Ktor 还包含一个灵活的异步 HTTP 客户端。
这个客户端支持多个[可配置的引擎](/clients/http-client/engines.html)，并且有自己的一组[特性](/clients/http-client/features.html)。

其主要功能由 `io.ktor:ktor-client-core:$ktor_version` 构件提供。
而每个引擎都在[单独的构件](/clients/http-client/engines.html)中提供。
{: .note.artifact }

**目录：**

* TOC
{:toc}

## Call：Request 与 Response
{: #requests-responses }

可以在相应的章节中查看[如何发出请求](/clients/http-client/calls/requests.html)，
以及[如何接收响应](/clients/http-client/calls/responses.html)。

## 并发

请记住，请求是异步的，但是当执行请求时，该 API 会挂起后续请求，
并且所在函数会被挂起，直到请求完成。如果想要在同一块中一次<!--
-->执行多个请求，可以使用 `launch` 或 `async` 函数，之后获取各个结果。
例如：

### 顺序请求：

```kotlin
suspend fun sequentialRequests() {
    val client = HttpClient(Apache)
    
    // 获取一个 URL 的内容。
    val bytes1 = client.call("https://127.0.0.1:8080/a").response.readBytes() // 挂起点。
    
    // 完成上一个请求后，获取另一个 URL 的内容。
    val bytes2 = client.call("https://127.0.0.1:8080/b").response.readBytes() // 挂起点。

    client.close()
}
```

### 并行请求：

```kotlin
suspend fun parallelRequests() = coroutineScope<Unit> {
    val client1 = HttpClient(Apache)
    val client2 = HttpClient(Apache)
    
    // 异步启动两个请求。
    val req1 = async { client1.call("https://127.0.0.1:8080/a").response.readBytes() }
    val req2 = async { client2.call("https://127.0.0.1:8080/b").response.readBytes() }
    
    // 获取请求内容而不阻塞线程，只是挂起该函数直到两个
    // 请求都完成。
    val bytes1 = req1.await() // 挂起点。
    val bytes2 = req2.await() // 挂起点。

    client2.close()
    client1.close()
}
```

## 示例
{: #examples }

更多相关信息，请查看[示例页](/clients/http-client/examples.html)及一些示例。

## 特性
{: #features}

更多相关信息，请查看[特性页](/clients/http-client/features.html)以及所有可用特性。
