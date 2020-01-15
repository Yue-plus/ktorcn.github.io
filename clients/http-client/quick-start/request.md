---
title: 请求
caption: Preparing the request
category: clients
permalink: /clients/http-client/quick-start/requests.html
redirect_from:
- /clients/http-client/call/requests.html
ktor_version_review: 1.3.0
---

## Making request

After client configuration we're ready to perform our first request.
Most of the simple requests are made with pattern

```
val response = client.'http-method'<'ResponseType'>("url-string")
```

or even simpler form(due to kotlin generic type inference):

```
val response: ResponseType = client.'http-method'("url-string")
```

For example to perform a `GET` request fully reading a `String`:

```kotlin
val htmlContent = client.get<String>("https://en.wikipedia.org/wiki/Main_Page")
// same as
val content: String = client.get("https://en.wikipedia.org/wiki/Main_Page")
```

而如果对原始数据感兴趣，可以读取 `ByteArray`：

```kotlin
val channel: ByteArray = client.get("https://en.wikipedia.org/wiki/Main_Page")
```

Or get full [HttpResponse](https://api.ktor.io/{{ site.ktor_version }}/io.ktor.client.statement/-http-response/index.html):

```kotlin
val response: HttpResponse = client.get("https://en.wikipedia.org/wiki/Main_Page")
```

The [HttpResponse](https://api.ktor.io/{{ site.ktor_version }}/io.ktor.client.statement/-http-response/index.html) is downloaded in memory by default. To learn how to download response partially or work with a stream data consult with the [Streaming](/clients/http-client/quick-start/streaming.html) section.

And even your data class using [Json](/clients/http-client/features/json-feature.html) feature:

```kotlin
@Serializable
data class User(val id: Int)

val response: User = client.get("https://myapi.com/user?id=1")
```

Please note that some of response types are `Closeable` and can hold resources.

## Customizing requests

我们不能只进行 *get* 请求，Ktor 允许使用任何 HTTP 动词构件复杂请求，并且灵活地以多种方式处理响应。

### Default http methods

{: #shortcut-methods }

与 `request` 类似，还有几个可以<!--
-->使用最常见的 HTTP 动词（`GET`、 `POST`、 `PUT`、 `DELETE`、 `PATCH`、 `HEAD` 以及 `OPTIONS`）执行请求的扩展方法。

```kotlin
val text = client.post<String>("http://127.0.0.1:8080/")
```

When calling request methods, you can provide a lambda to build the request
parameters like the URL, the HTTP method(verb), the body, or the headers:

```kotlin
val text = client.post<String>("http://127.0.0.1:8080/") {
    header("Hello", "World")
}
```

The [HttpRequestBuilder](https://api.ktor.io/{{ site.ktor_version }}/io.ktor.client.request/-http-request-builder/) looks like this:

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

`HttpClient` 类只提供一些基本功能，而所有构建请求的方法都是扩展方法。
参见标准的 [HttpClient 构建扩展方法](https://api.ktor.io/{{ site.ktor_version }}/io.ktor.client.request/)。

### Customize method

In addition to call, there is a `request` method for performing a typed request,
[receiving a specific type](/clients/http-client/quick-start/responses.html#receive) like String, HttpResponse, or an arbitrary class.
You have to specify the URL and the method when building the request.

```kotlin
val call = client.request<String> {
    url("http://127.0.0.1:8080/")
    method = HttpMethod.Get
}
```

### Posting forms

{: #submit-form }

There are a couple of convenience extension methods for submitting form information.
The detailed refrence is listed [here](https://api.ktor.io/{{ site.ktor_version }}/io.ktor.client.request.forms/).

`submitForm` 方法：

```kotlin
client.submitForm(
    formParameters: Parameters = Parameters.Empty,
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
    appendInput("input", size = knownSize.orNull()) { openInputStream().asInput() }
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

Alternatively (using the integrated `JsonSerializer`):

```kotlin
val json = io.ktor.client.features.json.defaultSerializer()
client.post<Unit>() {
    url("http://127.0.0.1:8080/")
    body = json.write(HelloWorld(hello = "world")) // Generates an OutgoingContent
}
```

Or using Jackson (JVM only):

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
