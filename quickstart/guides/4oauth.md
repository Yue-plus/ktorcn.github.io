---
title: Google OAuth
caption: 指南：如何通过 Google 实现 OAuth 登录
category: quickstart
permalink: /quickstart/guides/oauth.html
ktor_version_review: 1.0.0
---

{::options toc_levels="1..2" /}

在本指南中，我们会使用 OAuth 实现登录。

This is an advanced tutorial and it assumes you have some basic knowledge about Ktor,
so you should follow the [guide about making a Website](/quickstart/guides/website.html) first.

**目录：**

* TOC
{:toc}

## 创建一个指向 127.0.0.1 的主机名条目

Google 的 OAuth 要求重定向 URL 不能是 IP 地址或者 localhost。
因此，出于开发目的，我们需要一个合适的主机名指向 127.0.0.1。
由于并不需要从计算机外部访问该主机名，因此可以只为本地主机设置。
有一个公网域名 <http://lvh.me/> 指向 localhost/127.0.0.1，但是出于安全原因考虑，你可能希望<!--
-->在本地提供自己的主机名。

为此，可以在计算机的 [host 文件](https://zh.wikipedia.org/wiki/Hosts%E6%96%87%E4%BB%B6)中添加一个条目。

对于本指南，我们会将 `me.mydomain.com` 关联到 `127.0.0.1`，不过你可以根据自己的需要进行更改，
只要它像公网顶级域名（.com、.org……）一样或者至少有两个组成部分即可。

```
127.0.0.1       me.mydomain.com
```

![](/quickstart/guides/oauth/etc_hosts.png){:.rounded-shadow}

这个文件的结构很简单：<kbd>#</kbd> 字符用于注释，
每个非空且非注释行都应该包含一个 IP 地址，后跟<!--
-->几个由空格或者制表符（tab）分隔的主机名。

### MacOS/Linux

在 MacOS 与 Linux（Unix）计算机中，可以在 `/etc/hosts` 处找到主机名文件。会需要 root 权限才能对其编辑。

```sudo nano /etc/hosts```
或者
```sudo vi /etc/hosts```

### Windows

在 Windows 中，主机名文件保存在 `%SystemRoot%\System32\drivers\etc\hosts`。需要管理员权限<!--
-->才能编辑这个文件。例如，可以使用以管理员身份打开的 [wxMEdit](https://wxmedit.github.io/zh_CN/) 来编辑。

还可以在 Windows 资源管理器中粘贴 `%SystemRoot%\System32\drivers\etc` 路径，然后右击
hosts 文件来编辑它。这个文件的结构与 MacOS/Linux 相同。

## Google 开发者控制台

为了能够使用任何提供商的 OAuth，会需要一个公开的 `clientId` 以及一个私有的 `clientSecret`。
对于 Google 登录，可以使用 Google 开发者控制台创建它：
<https://console.developers.google.com/>{:target="_blank"}

首先，必须在开发者控制台中创建一个新项目：

![](/quickstart/guides/oauth/1.png){:.rounded-shadow}
![](/quickstart/guides/oauth/2.png){:.rounded-shadow}

在 `API & Services` → `Credentials` 内部有一个带有 `OAuth Client Id` 选项的 `Create Credentials` 按钮：

![](/quickstart/guides/oauth/3.png){:.rounded-shadow}
![](/quickstart/guides/oauth/4.png){:.rounded-shadow}
![](/quickstart/guides/oauth/5.png){:.rounded-shadow}
![](/quickstart/guides/oauth/6.png){:.rounded-shadow}

不过首先，我们必须配置（Configure）OAuth consent screen：

![](/quickstart/guides/oauth/7.png){:.rounded-shadow}
![](/quickstart/guides/oauth/8.png){:.rounded-shadow}

现在，我们可以使用以下信息创建 OAuth 凭据：
* **Authorized JavaScript origins:** `http://me.mydomain.com:8080`
* **Authorized redirect URIs:** `http://me.mydomain.com:8080/login`

点击 `Create` 按钮。

![](/quickstart/guides/oauth/9.png){:.rounded-shadow}

可以稍后更改这些值，也可以通过编辑凭据添加其他授权的 URL。

会看到一个模式对话框，其中包含以下内容：

**OAuth client**
* `Here is your client ID: xxxxxxxxxxx.apps.googleusercontent.com`
* `Here is your client secret: yyyyyyyyyyy`

![](/quickstart/guides/oauth/10.png){:.rounded-shadow}

## 配置应用

首先，必须为 OAuth 提供商定义设置。必须用上一步获得的值替换 `clientId` 与 `clientSecret`
。根据我们对用户的需求，可以将 `defaultScopes`
列表调整为其他范围。`profile` 范围会访问用户 id、全名与图片，但不会访问电子邮件及其他任何内容：

```kotlin
val googleOauthProvider = OAuthServerSettings.OAuth2ServerSettings(
    name = "google",
    authorizeUrl = "https://accounts.google.com/o/oauth2/auth",
    accessTokenUrl = "https://www.googleapis.com/oauth2/v3/token",
    requestMethod = HttpMethod.Post,

    clientId = "xxxxxxxxxxx.apps.googleusercontent.com",
    clientSecret = "yyyyyyyyyyy",
    defaultScopes = listOf("profile") // 无电子邮件，但提供全名、图片与 id
)
```

请记住为了安全性、用户隐私与信任，请将 defaultScopes 调整为只请求你真正需要的内容。
{: .note}

我们还必须安装 OAuth 特性并进行配置。需要提供一个 HTTP 客户端实例、一个提供商查找程序——
由该调用确定提供商（不需要在这里输入逻辑，因为本指南只支持 Google）以及<!--
-->一个给出重定向 URL 的 `urlProvider`，必须匹配在 Google 开发者控制台中指定为 authorized redirection 的 url——在本例中是 `http://me.mydomain.com:8080/login`：

```kotlin
install(Authentication) {
    oauth("google-oauth") {
        client = HttpClient(Apache)
        providerLookup = { googleOauthProvider }
        urlProvider = { redirectUrl("/login") }
    }
}

private fun ApplicationCall.redirectUrl(path: String): String {
    val defaultPort = if (request.origin.scheme == "http") 80 else 443
    val hostPort = request.host()!! + request.port().let { port -> if (port == defaultPort) "" else ":$port" }
    val protocol = request.origin.scheme
    return "$protocol://$hostPort$path"
}

```

然后必须定义 `/login` 路由，该路由必须由我们的认证提供商进行认证。
当没有 get 参数传给该 URL 时，身份认证特性会 hook 住其处理程序，并会<!--
-->将我们重定向到 Google 的 OAuth Consent Screen，而它会重定向回我们的 `/login` 路由并带有
`status` 与 `code` 参数——会用于由认证提供方回调 Google 以获取<!--
-->`accessToken` 并将 `OAuthAccessTokenResponse.OAuth2` 身份附加到我们的调用中。而这一次，
我们的处理程序会执行。

我们可以通过获取生成的 `OAuthAccessTokenResponse.OAuth2` 身份与 `accessToken`
来取得 `accessToken`。然后我们可以使用  <https://www.googleapis.com/userinfo/v2/me>{:target="_blank"} URL
并传入 `accessToken` 作为 Authorization Bearer 来获取带有用户信息的 JSON。
可以使用 [Google OAuth playground](https://developers.google.com/oauthplayground){:target="_blank"} 检验该 JSON 的内容。

在本例中，一旦我们获取了用户 ID，我们就会将其存储在一个会话中，然后重定向到另一个地方。

```kotlin
class MySession(val userId: String)

authenticate("google-oauth") {
    route("/login") {
        handle {
            val principal = call.authentication.principal<OAuthAccessTokenResponse.OAuth2>()
                ?: error("No principal")

            val json = HttpClient(Apache).get<String>("https://www.googleapis.com/userinfo/v2/me") {
                header("Authorization", "Bearer ${principal.accessToken}")
            }

            val data = ObjectMapper().readValue<Map<String, Any?>>(json)
            val id = data["id"] as String?

            if (id != null) {
                call.sessions.set(MySession(id))
            }
            call.respondRedirect("/")
        }
    }
} 
```

必须先安装会话特性。详见[完整示例](#full-example)：
{: .note }

用户信息中的 ID 是一个看起来像数字的字符串。请记住 JSON 没有定义长整型，
并且对于 Twitter 或者 Google 来说，他们有大量的用户与实体，其 ID 可能会超过<!--
-->有符号整型的 31 比特，甚至超过标准 Double 的 51 比特精度。<br />
一般来说，如果你不需要对其进行算术运算，
那么应该将 ID 以及其他类似数字的值始终视为字符串。
{: .note }

## 完整示例
{: #full-example }

一个简单的嵌入式应用如下所示：

{% capture oauth-kt %}
```kotlin
val googleOauthProvider = OAuthServerSettings.OAuth2ServerSettings(
    name = "google",
    authorizeUrl = "https://accounts.google.com/o/oauth2/auth",
    accessTokenUrl = "https://www.googleapis.com/oauth2/v3/token",
    requestMethod = HttpMethod.Post,

    clientId = "xxxxxxxxxxx.apps.googleusercontent.com", // @TODO: 记得更改这个！
    clientSecret = "yyyyyyyyyyy", // @TODO: 记得更改这个！
    defaultScopes = listOf("profile") // no email, but gives full name, picture, and id
)

class MySession(val userId: String)

fun main(args: Array<String>) {
    embeddedServer(Netty, port = 8080) {
        install(WebSockets)
        install(Sessions) {
            cookie<MySession>("oauthSampleSessionId") {
                val secretSignKey = hex("000102030405060708090a0b0c0d0e0f") // @TODO: 记得更改这个！
                transform(SessionTransportTransformerMessageAuthentication(secretSignKey))
            }
        }
        install(Authentication) {
            oauth("google-oauth") {
                client = HttpClient(Apache)
                providerLookup = { googleOauthProvider }
                urlProvider = {
                    redirectUrl("/login")
                }
            }
        }
        routing {
            get("/") {
                val session = call.sessions.get<MySession>()
                call.respondText("HI ${session?.userId}")
            }
            authenticate("google-oauth") {
                route("/login") {
                    handle {
                        val principal = call.authentication.principal<OAuthAccessTokenResponse.OAuth2>()
                            ?: error("No principal")
            
                        val json = HttpClient(Apache).get<String>("https://www.googleapis.com/userinfo/v2/me") {
                            header("Authorization", "Bearer ${principal.accessToken}")
                        }
            
                        val data = ObjectMapper().readValue<Map<String, Any?>>(json)
                        val id = data["id"] as String?
            
                        if (id != null) {
                            call.sessions.set(MySession(id))
                        }
                        call.respondRedirect("/")
                    }
                }
            } 
        }
    }.start(wait = true)
}

private fun ApplicationCall.redirectUrl(path: String): String {
    val defaultPort = if (request.origin.scheme == "http") 80 else 443
    val hostPort = request.host()!! + request.port().let { port -> if (port == defaultPort) "" else ":$port" }
    val protocol = request.origin.scheme
    return "$protocol://$hostPort$path"
}
```
{% endcapture %}

{% include tabbed-code.html
    tab1-title="OAuthApp.kt" tab1-content=oauth-kt
%}

&nbsp;

## 测试

可以[提供一个测试 HttpClient 来测试 OAuth](https://github.com/ktorio/ktor-samples/commit/56119d2879d9300cf51d66ea7114ff815f7db752)。

## 补充资源

* Google OAuth playground：<https://developers.google.com/oauthplayground/>{:target="_blank"}
* 可用的谷歌 oauth 授权范围列表：<https://developers.google.com/identity/protocols/googlescopes>{:target="_blank"}
* 几个 oauth 提供商的示例：<https://github.com/ktorio/ktor-samples/blob/master/feature/auth/src/io/ktor/samples/auth/OAuthLoginApplication.kt>{:target="_blank"}
