---
title: 响应
caption: Reading the response
category: clients
permalink: /clients/http-client/quick-start/responses.html
redirect_from:
- /clients/http-client/call/responses.html
ktor_version_review: 1.2.0
---

## 接收响应的 body

{: #receive}

默认情况下，可以使用 `HttpResponse` 或者 `String` 作为类型化
HttpClient 请求的类型。例如：

```kotlin
val htmlContent = client.get<String>("https://en.wikipedia.org/wiki/Main_Page")
val response = client.get<HttpResponse>("https://en.wikipedia.org/wiki/Main_Page")
```

如果配置了 *JsonFeature*，并且服务端返回了头信息 `Content-Type: application/json`，
那么还可以指定一个类来反序列化。

```kotlin
val helloWorld = client.get<HelloWorld>("http://127.0.0.1:8080/")
```

### `HttpResponse` 类

{: #HttpResponse }

[这里](https://api.ktor.io/{{site.ktor_version}}/io.ktor.client.response/-http-response/)列出了 `HttpResponse` 的 API 参考。

通过 `HttpResponse` 可以轻松获取响应内容：

* `val readChannel: ByteReadChannel = response.content`
* `val bytes: ByteArray = response.readBytes()`
* `val text: String = response.readText()`
* `val readChannel = response.receive<ByteReadChannel>()`
* `val multiPart = response.receive<MultiPartData>()`
* `val inputStream = response.receive<InputStream>()` *请记住，InputStream API 是同步的！*
* `response.discardRemaining()`

还可以获取其他响应信息，例如响应状态码、响应头、内部状态等：

### *基本*

* `val status: HttpStatusCode = response.status`
* `val headers: Headers = response.headers`

### *高级*

* `val call: HttpClientCall = response.call`
* `val version: HttpProtocolVersion = response.version`
* `val requestTime: Date = response.requestTime`
* `val responseTime: Date = response.responseTime`
* `val executionContext: Job = response.executionContext`

### *响应头的扩展函数*

* `val contentType: ContentType? = response.contentType()`
* `val charset: Charset? = response.charset()`
* `val lastModified: Date? = response.lastModified()`
* `val etag: String? = response.etag()`
* `val expires: Date? = response.expires()`
* `val vary: List<String>? = response.vary()`
* `val contentLength: Int? = response.contentLength()`
* `val setCookie: List<Cookie> = response.setCookie()`
