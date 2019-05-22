---
title: 请求
caption: HTTP 客户端请求
category: clients
permalink: /clients/http-client/call/requests.html
redirect_from:
- /clients/http-client/calls/requests.html
ktor_version_review: 1.2.0
---

## 简单的请求

基本用法*超*简单：只需实例化一个 `HttpClient` 实例、
指定一个引擎——例如：
[`Apache`](/clients/http-client/engines.html#apache)、
[`OkHttp`](/clients/http-client/engines.html#okhttp)、
[`Android`](/clients/http-client/engines.html#android)、
[`Ios`](/clients/http-client/engines.html#ios)、
[`Js`](/clients/http-client/engines.html#js)、
[`Jetty`](/clients/http-client/engines.html#jetty)、
[`CIO`](/clients/http-client/engines.html#cio)
或者 [`Mock`](/clients/http-client/engines.html#mock)，
并使用众多可用的便利方法之一发出请求。

引擎可以省略，对于 JVM，Ktor 会使用 ServiceLoader 从所包含的构件中选择一个可用的<!--
-->引擎；对于其他平台 Ktor
会选用默认实现。

首先需要实例化客户端：

```kotlin
val client = HttpClient()
```

然后，执行读取完整 `String` 的 `GET` 请求：

```kotlin
val htmlContent = client.get<String>("https://en.wikipedia.org/wiki/Main_Page")
```

而如果对原始数据感兴趣，可以读取 `ByteArray`：

```kotlin
val bytes: ByteArray = client.get<ByteArray>("http://127.0.0.1:8080/")
```

可以对请求进行大量定制，并且流式传输请求与响应有效载荷：

```kotlin
val channel: ByteReadChannel = client.get<ByteReadChannel>("http://127.0.0.1:8080/")
```

使用完客户端之后，应该关闭它以彻底停止底层引擎。

```kotlin
client.close()
```

如果只使用客户端发出一个请求，考虑使用 `use`。一旦传入的块执行完，客户端会自动关闭。

```kotlin
val status = HttpClient().use { client ->
    client.get<HttpStatusCode>("http://127.0.0.1:8080/check")
}
```

## 定制请求

我们不能只进行 *get* 请求，Ktor 允许使用任何 HTTP 动词构件复杂请求，并且灵活地以多种方式处理响应。

### `call` 方法

{: #call-method }

The HttpClient `call` method, returns an `HttpClientCall` and allows you to perform simple untyped requests.

You can read the content using `response: HttpResponse`.
For further information, check out the [receiving content using HttpResponse](/clients/http-client/call/responses.html) section.

```kotlin
val call = client.call("http://127.0.0.1:8080/") {
    method = HttpMethod.Get
}
println(call.response.receive<String>())
```

### `request` 方法

{: #request-method }

In addition to call, there is a `request` method for performing a typed request,
[receiving a specific type](/clients/http-client/call/responses.html#receive) like String, HttpResponse, or an arbitrary class.
You have to specify the URL and the method when building the request.

```kotlin
val call = client.request<String> {
    url("http://127.0.0.1:8080/")
    method = HttpMethod.Get
}
```

### `get`、 `post`、 `put`、 `delete`、 `patch`、 `head` 以及 `options` 方法

{: #shortcut-methods }

Similar to `request`, there are several extension methods to perform requests
with the most common HTTP verbs: `GET`, `POST`, `PUT`, `DELETE`, `PATCH`, `HEAD` and `OPTIONS`.

```kotlin
val text = client.post<String>("http://127.0.0.1:8080/")
```

When calling request methods, you can provide a lambda to build the request
parameters like the URL, the HTTP method(verb), the body, or the headers.
The `HttpRequestBuilder` looks like this:

```kotlin
class HttpRequestBuilder : HttpMessageBuilder {
    var method: HttpMethod

    val url: URLBuilder
    fun url(block: URLBuilder.(URLBuilder) -> Unit)

    val headers: HeadersBuilder
    fun header(key: String, value: String)
    fun headers(block: HeadersBuilder.() -> Unit)

    var body: Any = EmptyContent

    val executionContext: CompletableDeferred<Unit>
    fun setAttributes(block: Attributes.() -> Unit)
    fun takeFrom(builder: HttpRequestBuilder): HttpRequestBuilder
}
```

The `HttpClient` class only offers some basic functionality, and all the methods for building requests are exposed as extensions.\\
You can check the standard available [HttpClient build extension methods](https://api.ktor.io/{{ site.ktor_version }}/io.ktor.client.request/).

{: .note.api}

### `submitForm` 与 `submitFormWithBinaryData` 方法

{: #submit-form }

There are a couple of convenience extension methods for submitting form information.
The detailed refrence is listed [here](https://api.ktor.io/{{ site.ktor_version }}/io.ktor.client.request.forms/).

`submitForm` 方法：

```kotlin
client.submitForm(
    formData: Parameters = Parameters.Empty,
    encodeInQuery: Boolean = false,
    block: HttpRequestBuilder.() -> Unit = {}
)
```

It allows requesting with the `Parameters` encoded in the query string(`GET` by default) or requesting with the `Parameters` encoded as multipart(`POST` by default) depending on the `encodeInQuery` parameter.

`submitFormWithBinaryData` 方法：

```kotlin
client.submitFormWithBinaryData(
    formData: List<PartData>,
    block: HttpRequestBuilder.() -> Unit = {}
): T
```

It allows to generate a multipart POST request from a list of `PartData`.
`PartData` can be `PartData.FormItem`, `PartData.BinaryItem` or `PartData.FileItem`.

To build a list of `PartData`, you can use the `formData` builder:

```kotlin
val data: List<PartData> = formData {
    // Can append: String, Number, ByteArray and Input.
    append("hello", "world")
    append("number", 10)
    append("ba", byteArrayOf(1, 2, 3, 4))
    append("input", inputStream.asInput())
    // Allow to set headers to the part:
    append("hello", "world", headersOf("X-My-Header" to "MyValue"))
}
```

### 指定自定义头

{: #custom-headers}

When building requests with `HttpRequestBuilder`, you can set custom headers.
There is a final property `val headers: HeadersBuilder` that inherits from `StringValuesBuilder`.
You can add or remove headers using it, or with the `header` convenience methods.

```kotlin
// this : HttpMessageBuilder

// Convenience method to add a header
header("My-Custom-Header", "HeaderValue")

// Calls methods from the headers: HeadersBuilder to manipulate the headers
headers.clear()
headers.append("My-Custom-Header", "HeaderValue")
headers.appendAll("My-Custom-Header", listOf("HeaderValue1", "HeaderValue2"))
headers.remove("My-Custom-Header")

// Applies the headers with the `headers` convenience method
headers { // this: HeadersBuilder
    clear()
    append("My-Custom-Header", "HeaderValue")
    appendAll("My-Custom-Header", listOf("HeaderValue1", "HeaderValue2"))
    remove("My-Custom-Header")
}
```

Complete `HeadersBuilder` API is listed [here](https://api.ktor.io/{{ site.ktor_version }}/io.ktor.http/-headers-builder/).

## 指定请求的正文

For `POST` and `PUT` requests, you can set the `body` property:

```kotlin
client.post<Unit> {
    url("http://127.0.0.1:8080/")
    body = // ...
}
```

The `HttpRequestBuilder.body` property can be a subtype of `OutgoingContent` as well as a `String` instance:

* `body = "HELLO WORLD!"`
* `body = TextContent("HELLO WORLD!", ContentType.Text.Plain)`
* `body = ByteArrayContent("HELLO WORLD!".toByteArray(Charsets.UTF_8))`
* `body = LocalFileContent(File("build.gradle"))`
* `body = JarFileContent(File("myjar.jar"), "test.txt", ContentType.fromFileExtension("txt").first())`
* `body = URIFileContent("https://en.wikipedia.org/wiki/Main_Page")`

If you install the *JsonFeature*, and set the content type to `application/json`
you can use arbitrary instances as the `body`, and they will be serialized as JSON:

```kotlin
data class HelloWorld(val hello: String)

val client = HttpClient(Apache) {
    install(JsonFeature) {
        serializer = GsonSerializer {
            // Configurable .GsonBuilder
            serializeNulls()
            disableHtmlEscaping()
        }
    }
}

client.post<Unit> {
    url("http://127.0.0.1:8080/")
    body = HelloWorld(hello = "world")
}
```

Alternatively(using the integrated `JsonSerializer`):

```kotlin
val json = io.ktor.client.features.json.defaultSerializer()
client.post<Unit>() {
    url("http://127.0.0.1:8080/")
    body = json.write(HelloWorld(hello = "world")) // Generates an OutgoingContent
}
```

Or using Jackson(JVM only):

```kotlin
val json = jacksonObjectMapper()
client.post<Unit> {
    url("http://127.0.0.1:8080/")
    body = TextContent(json.writeValueAsString(userData), contentType = ContentType.Application.Json)
}
```

Remember that your classes must be *top-level* to be recognized by `Gson`. \\
If you try to send a class that is inside a function, the feature will send a *null*.
{: .note}

## 上传 multipart/form-data

{: #multipart-form-data }

Ktor HTTP Client has support for making MultiPart requests.
The idea is to use the `MultiPartFormDataContent(parts: List<PartData>)` as `OutgoingContent` for the body of the request.

The easiest way is to use the [`submitFormWithBinaryData` method](#submit-form).

Alternatively, you can set the body directly:

```kotlin
val request = client.request {
    method = HttpMethod.Post
    body = MultiPartFormDataContent(formData {
        append("key", "value")
    })
}
```
