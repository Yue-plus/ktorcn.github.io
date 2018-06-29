---
title: REST API
caption: 如何使用 ktor 创建 API
category: quickstart
---

{::options toc_levels="1..2" /}

在本指南中，你会学习如何使用 ktor 创建 API。
我们会创建一个简单的 API 来存储简单的文本片段（就像一个小型的类 pastebin API）。

为了实现这一点，我们会用到[路由]、 [状态页]、 [身份认证]、 [JWT 认证]、
[CORS]、 [内容协商]以及 [Jackson] 这些特性。

[路由]: /features/routing.html
[状态页]: /features/status-pages.html
[身份认证]: /features/authentication.html
[JWT 认证]: /features/authentication/jwt.html
[CORS]: /features/cors.html
[内容协商]: /features/content-negotiation.html
[Jackson]: /features/content-negotiation/jackson.html

**目录：**

* TOC
{:toc}

## 搭建项目

第一步是搭建一个项目。可以按照[快速入门](/quickstart/index.html)指南操作，或者使用以下表单创建：

[**打开预先配置好的生成器表单**](javascript:$('#start_ktor_io_form').toggle())

<iframe src="{{ site.ktor_init_tools_url }}#dependency=auth&dependency=auth-jwt&dependency=ktor-jackson&dependency=cors&artifact-group=com.example&artifact-name=api-example" id="start_ktor_io_form" style="border:1px solid #343a40;width:100%;height:500px;display:none;"></iframe>

## 简单路由

首先，我们要使用路由特性。 这个特性是 Ktor 核心的一部分，因此不需要<!--
-->包含任何其他构件。

这一特性在使用 `routing { }` 块时自动安装。

让我们开始创建一个响应为“OK”的简单 GET 路由：

```kotlin
fun Application.module() {
    routing {
        get("/snippets") {
            call.respondText("OK")
        }
    }
}
```

## 提供 JSON 内容

REST API 通常以 JSON 响应。可以使用以 *Jackson* 进行*内容协商*这一特性来实现：

```kotlin
fun Application.module() {
    install(ContentNegotiation) {
        jackson {
        }
    }
    routing {
        // ……
    }
}
```

为了以 JSON 响应调用，必须调用 `call.respond` 方法并传入任意对象。

```kotlin
routing {
    get("/snippets") {
        call.respond(mapOf("OK" to true))
    }
}
```

现在浏览器应该以 `{"OK":true}` 来响应 `http://127.0.0.1:8080/snippets`

如果出现类似 `Response pipeline couldn't transform '...' to the OutgoingContent` 的错误，请检查是否已经<!--
-->安装了采用 Jackson 的内容协商特性。
{: .note}

也可以使用类型化的对象作为回复的一部分（但要确保该类不是在<!--
-->函数内部定义的，否则不能用）。例如：

```kotlin
data class Snippet(val text: String)

val snippets = Collections.synchronizedList(mutableListOf(
    Snippet("hello"),
    Snippet("world")
))

fun Application.module() {
    install(ContentNegotiation) {
        jackson {
            enable(SerializationFeature.INDENT_OUTPUT) // 美化输出 JSON
        }
    }
    routing {
        get("/snippets") {
            call.respond(mapOf("snippets" to synchronized(snippets) { snippets.toList() }))
        }
    }
}
```

会回复：

![](/quickstart/guides/api/snippets_get.png){:.rounded-shadow}

## 处理其他 HTTP 方法

REST API 使用大部分 HTTP 方法/动词（_HEAD_、 _GET_、 _POST_、 _PUT_、 _PATCH_、 _DELETE_、 _OPTIONS_）来执行操作。
我们来创建一个添加新片段的路由。为此，我们需要读取 POST 请求的 JSON 正文。
为此我们会使用 `call.receive<Type>()`：

```kotlin
data class PostSnippet(val snippet: PostSnippet.Text) {
    data class Text(val text: String)
}

routing {
    get("/snippets") {
        call.respond(mapOf("snippets" to synchronized(snippets) { snippets.toList() }))
    }
    post("/snippets") {
        val post = call.receive<PostSnippet>()
        snippets += Snippet(post.snippet.text)
        call.respond(mapOf("OK" to true))
    }
}
```

可以使用 postman 或 curl 轻松执行 POST 调用：

Postman：

![](/quickstart/guides/api/postman.png)`{:.rounded-shadow}`

CURL：

<table class="compare-table"><thead><tr><th>Bash:</th><th>Response:</th></tr></thead><tbody><tr><td markdown="1">

```bash
curl \
  --request POST \
  --header "Content-Type: application/json" \
  --data '{"snippet" : {"text" : "mysnippet"}}' \
  http://127.0.0.1:8080/snippets
```

</td><td markdown="1">

```json
{
  "OK" : true
}
```

</td></tr></tbody></table>

我们再次执行 GET 请求：

![](/quickstart/guides/api/snippets_get_new.png){:.rounded-shadow}

很好！

## 将路由分组合并

现在我们有两个分开的路由共享同一路径（但不共享方法），而我们不希望自我重复。

我们可以使用 `route(path) { }` 块将具有相同前缀的路由分组。对于每个 HTTP 方法，都有一个<!--
-->不带路由路径参数的重载，可以用于路由的叶子节点：

```kotlin
routing {
    route("/snippets") {
        get {
            call.respond(mapOf("snippets" to synchronized(snippets) { snippets.toList() }))
        }
        post {
            val post = call.receive<PostSnippet>()
            snippets += Snippet(post.snippet.text)
            call.respond(mapOf("OK" to true))
        }
    }
}
```

## 身份认证

防止每个人都发布片段是一个好主意。现在，我们将使用
http 的基本身份认证用固定的用户名与密码来限制。为做到这一点，我们会使用身份认证特性。

```kotlin
fun Application.module() {
    install(Authentication) {
        basic {
            realm = "myrealm" 
            validate { if (it.name == "user" && it.password == "password") UserIdPrincipal("user") else null }
        }
    }
    // ……
}
```

安装并配置该特性之后，我们可以将一些路由组合到一起，以便<!--
-->使用 `authenticate { }` 块进行身份认证。

在本例中，我们保持 get 调用无需认证，而要对 post 调用进行身份认证：

```kotlin
routing {
    route("/snippets") {
        get {
            call.respond(mapOf("snippets" to synchronized(snippets) { snippets.toList() }))
        }
        authenticate {
            post {
                val post = call.receive<PostSnippet>()
                snippets += Snippet(post.snippet.text)
                call.respond(mapOf("OK" to true))
            }        
        }
    }
}
```

## JWT 认证

我们接下来使用 JWT 令牌取代固定身份认证。

我们会添加一个登录注册路由。 如果指定用户不存在就会注册一个用户，
而对于有效登录或注册会返回一个 JWT 令牌。
JWT 令牌会保存用户名，而发布过程会将片段链接到用户。

我们需要安装并配置 JWT（取代基本身份认证 basic auth）：

```kotlin
open class SimpleJWT(val secret: String) {
    private val algorithm = Algorithm.HMAC256(secret)
    val verifier = JWT.require(algorithm).build()
    fun sign(name: String): String = JWT.create().withClaim("name", name).sign(algorithm)
}

fun Application.module() {
    val jwt = SimpleJWT("my-super-secret-for-jwt")
    install(Authentication) {
        jwt {
            verifier(jwt.verifier)
            validate {
                UserIdPrincipal(it.payload.getClaim("name").asString())
            }
        }
    }
    // ……
}
```

我们还需要一个保存用户名与密码的数据源。一个简单的选择是：

```kotlin
class User(val name: String, val password: String)

val users = Collections.synchronizedMap(
    listOf(User("test", "test"))
        .associateBy { it.name }
        .toMutableMap()
)
```

有了这些，我们就可以创建一个用来登录或者注册用户的路由：

```kotlin
routing {
    post("/login-register") {
        val post = call.receive<LoginRegister>()
        val user = users.getOrPut(post.user) { User(post.user, post.password) }
        if (user.password != post.password) error("Invalid credentials")
        call.respond(mapOf("token" to jwt.sign(user.name)))
    }
}
```

有了这些，我们就可以尝试获取用户的 JWT 令牌：

<table class="compare-table"><thead><tr><th>Bash:</th><th>Response:</th></tr></thead><tbody><tr><td markdown="1">

```bash
curl -v \
  --request POST \
  --header "Content-Type: application/json" \
  --data '{"user" : "test", "password" : "test"}' \
  http://127.0.0.1:8080/login-register
```

</td><td markdown="1">

```json
{
  "token" : "eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJuYW1lIjoidGVzdCJ9.96At6bwFhxebk4xk4tpkOFj-3ThxkLFNHkHaKoedOfA"
}
```

</td></tr></tbody></table>

使用这个令牌，我们就可以发布片段了：

<table class="compare-table"><thead><tr><th>Bash:</th><th>Response:</th></tr></thead><tbody><tr><td markdown="1">

```bash
curl -v \
  --request POST \
  --header "Content-Type: application/json" \
  --header "Authorization: Bearer eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJuYW1lIjoidGVzdCJ9.96At6bwFhxebk4xk4tpkOFj-3ThxkLFNHkHaKoedOfA" \
  --data '{"snippet" : {"text": "hello-world-jwt"}}' \
  http://127.0.0.1:8080/snippets
```

</td><td markdown="1">

```json
{
  "OK" : true
}
```

</td></tr></tbody></table>

## 将用户与片段关联

由于我们使用有认证的路由发布片段，因此我们可以访问所生成的包含用户名的 `Principal`
。所以我们应该能够访问该用户并将其关联到相应片段。

首先，我们需要将用户信息关联到片段：

```kotlin
data class Snippet(val user: String, val text: String)

val snippets = Collections.synchronizedList(mutableListOf(
    Snippet(user = "test", text = "hello"),
    Snippet(user = "test", text = "world")
))
```

现在我们可以在插入新片段时<!--
-->使用（在 JWT 认证时由身份认证特性生成的）当事人信息了：

```kotlin
routing {
    // ……
    route("/snippets") {
        // ……
        authenticate {
            post {
                val post = call.receive<PostSnippet>()
                val principal = call.principal<UserIdPrincipal>() ?: error("No principal")
                snippets += Snippet(principal.name, post.snippet.text)
                call.respond(mapOf("OK" to true))
            }
        }
    }
}
```

我们来试试这个：

<table class="compare-table"><thead><tr><th>Bash:</th><th>Response:</th></tr></thead><tbody><tr><td markdown="1">

```bash
curl -v \
  --request GET \
  http://127.0.0.1:8080/snippets
```

</td><td markdown="1">

```json
{
  "snippets" : [ {
    "user" : "test",
    "text" : "hello"
  }, {
    "user" : "test",
    "text" : "world"
  }, {
    "user" : "test",
    "text" : "hello-world-jwt"
  } ]
}
```

</td></tr></tbody></table>

好极了！

## 状态页

现在我们来细化一下。REST API 应该使用 Http 状态码来提供错误相关的语义信息。
现在，当异常抛出时（例如当试图用已存在的用户获取 JWT 令牌，
但密码错误时），会返回 500 服务器错误。 我们可以做的更好，并且状态页特性<!--
-->会允许通过捕获指定的异常并生成结果来实现这个功能。

我们来创建一个新的异常类型：

```kotlin
class InvalidCredentialsException(message: String) : RuntimeException(message)
```

现在，我们来安装状态页特性、注册这个异常类型，并生成一个未授权（Unauthorized）页：

```kotlin
fun Application.module() {
    install(StatusPages) {
        exception<InvalidCredentialsException> { exception ->
            call.respond(HttpStatusCode.Unauthorized, mapOf("OK" to false, "error" to (exception.message ?: "")))
        }
    }
    // ……
}
```

我们也需要更新登录注册页来引发这个异常：

```kotlin
routing {
    post("/login-register") {
        val post = call.receive<LoginRegister>()
        val user = users.getOrPut(post.user) { User(post.user, post.password) }
        if (user.password != post.password) throw InvalidCredentialsException("Invalid credentials")
        call.respond(mapOf("token" to jwt.sign(user.name)))
    }
}
```

我们来试试这个：

<table class="compare-table"><thead><tr><th>Bash:</th><th>Response:</th></tr></thead><tbody><tr><td markdown="1">

```bash
curl -v \
  --request POST \
  --header "Content-Type: application/json" \
  --data '{"user" : "test", "password" : "invalid-password"}' \
  http://127.0.0.1:8080/login-register
```

</td><td markdown="1">

```bash
< HTTP/1.1 401 Unauthorized
< Content-Length: 53
< Content-Type: application/json; charset=UTF-8
```
```json
{
  "OK" : false,
  "error" : "Invalid credentials"
}
```

</td></tr></tbody></table>

看起来好多了！

## CORS

现在假定我们需要从另一个域通过 JavaScript 访问这个 API。我们需要配置 CORS。
而 Ktor 有一个特性来配置这个功能：

```kotlin
fun Application.module() {
    install(CORS) {
        method(HttpMethod.Options)
        method(HttpMethod.Get)
        method(HttpMethod.Post)
        method(HttpMethod.Put)
        method(HttpMethod.Delete)
        method(HttpMethod.Patch)
        header(HttpHeaders.Authorization)
        allowCredentials = true
        anyHost()
    }
    // ……
}
```

现在我们的 API 可以从任何主机访问了 :)

## 练习

学完本指南，作为练习，可以尝试以下题目：

### 练习一

为每个片段添加唯一 id，并为 /snippets 添加一个 DELETE http 动词，以允许通过身份认证的用户删除<!--
-->自己的片段。

### 练习二

将用户与片段存储到数据库中。
