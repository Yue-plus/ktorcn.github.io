---
title: FAQ
caption: 常见问题
category: quickstart
permalink: /quickstart/faq.html
priority: 100
redirect_from:
  - /servers/faq.html
---

在本节中，我们（官方）为大家经常问我们的问题提供答案。

你的问题没有找到答案吗？请关注我们（官方）的 #ktor [Kotlin Slack](http://slack.kotlinlang.org/){: target="_blank" rel="noopener"} 频道，
我们会尽力帮助你。
{: .note}

**目录：**

* TOC
{:toc}

## ktor 的正确发音是什么？
{: #pronounce }

> `kay-tor`

## 如何提出问题/报告 bug/与你联系/做出贡献/提供反馈等？
{: #feedback }

根据具体内容，可以考虑以下几个渠道：

* **GitHub:** 功能请求、更改建议/提议、bug 以及 PR。
* **Slack:** 问题、疑难解决、指导等。
* **StackOverflow:** 问题。

**理由：**

对于问题或疑难解决，我们强烈建议使用 Slack 或者 StackOverflow。

考虑到使用 GitHub issues 会通知所有可能通过电子邮件订阅的人，
疑难解决通常需要几个问题与答案，这可能会一时产生很多电子邮件<!--
-->，也许订阅的人们希望了解 bug、修复、引入或提出的新功能相关内容<!--
-->，而对其他事情也许并不感兴趣。

如果有足够时间或者并不想加入 Slack，也可以在 StackOverflow 上提问。
鉴于 slack 是聊天与论坛的混合体，我们可以更快地相互联系并且在更短的时间内解决问题。

在进行疑难解决时，如果我们确定存在 bug 或需要改进的地方，可以在 GitHub 上报告。
当然，在 Slack 中进行 bug 报告（一旦确认）并不是一个好主意，因为可能会被遗忘，
所以我们将其放在 GitHub 中。

**Pull Requests：**

如果有一项你认为值得包含进 Ktor 的功能或者 bug 修复，可以创建 PR。

请记住，我们通常会批量评审与合并 PR，因此该 PR 可能会等待几周。
不过，如果可以，我们仍然鼓励你做出贡献！

如果你有一个需要立即使用的 bug 修复，我们建议你 fork Ktor、
自行编译，然后在自己的 artifactory、bintray 或者类似的地方临时发布打补丁后的版本，
直到它被合并并且随新版本或者预览版发布（因为时间安排可能不能满足你的需求）。

## CIO 是什么意思？
{: #cio }

CIO 代表基于协程的 I/O（Coroutine-based I/O）。 通常我们称之为引擎，它使用 Kotlin 以及协程来实现
IETF RFC 或者其他协议的逻辑，而不依赖外部 JVM 库。

## Ktor 导入不能解析。导入是红的。
{: #ktor-artifact }

> 请确保包含了 ktor 构件。例如，对于 gradle 及 Netty 引擎会是：
> ```kotlin
> dependencies {
>     compile "io.ktor:ktor-server-netty:$ktor_version"
> }
> ```
> * 对于 gradle，请查阅：<https://ktor.kotlincn.net/quickstart/gradle.html#engine>
> * 对于 maven，请查阅：<https://ktor.kotlincn.net/quickstart/maven.html>

## ktor 是否提供了捕获 ipc 信号（如 SIGTERM 或 SIGINT）的方法，以便可以优雅处理服务器关机？
{: #sigterm }

> 如果运行的是 `DevelopmentEngine`，那么会自动处理。
>
> 否则必须[手动处理](https://github.com/ktorio/ktor/blob/80f8c7bf352ac8075b8922b7f1aa94d7dc2ffdce/ktor-server/ktor-server-cio/src/io/ktor/server/cio/DevelopmentEngine.kt#L12)。
> 可以使用 JVM 的 `Runtime.getRuntime().addShutdownHook` 工具。

## 如何在代理服务器后获取客户端 IP？
{: #proxy-ip }

> 如果代理服务器提供了正确的头，并且已安装 `XForwardedHeaderSupport` 特性，
> 那么 `call.request.origin` 属性会提供原始调用者（代理）的连接信息。

## 出现错误“java.lang.IllegalStateException: No instance for key AttributeKey: Locations”
{: #no-attribute-key-locations }

> 如果试图使用 location 特性但没有实际安装它，就会出现这个错误。请查阅 location 特性：
> <https://ktor.kotlincn.net/features/locations.html>

## 客户端未发送 Accept 头时出现 406 错误。使用 WRK 在添加 JSON 支持后得到非 2xx 响应
{: #missing-accept-issue }

> 这是一个 Ktor <= 0.9.1 的[已知问题](https://github.com/ktorio/ktor/issues/38)，
> 当客户端未发送 Accept 头时，内容协商假定不接受任何内容。
> 自 0.9.2-alpha-1 起，如果未发送 Accept 头，ktor 假定接受所有内容。

## 如何测试 master 上的最新提交？
{: #bleeding-edge }

可以使用 jitpack 从 master 获取尚未发布的版本：
<https://jitpack.io/#ktorio/ktor>
也可以[从源代码构建 ktor](/advanced/building-from-source.html) 并使用 mavenLocal 仓库提供构件。

## 如何确定我用的是哪个版本的 ktor？
{: #ktor-version-used }

可以使用 [`DefaultHeaders` 特性](/features/default-headers.html)来发送一个<!--
-->包含 ktor 版本的 Server 头。
所发送的响应头中包含类似这样的内容： `Server: ktor-server-core/0.9.2-alpha-3 ktor-server-core/0.9.2-alpha-3`

## 网站辅助技巧与窍门
{: #website-tricks }

> 可以在文档网站的任何页面中<!--
> -->使用 <kbd>s</kbd> 键（搜索）、<kbd>t</kbd> 键（github 文件查找器风格）或者 <kbd>#</kbd> 键访问搜索。
> <kbd>#</kbd> 版限制为只在当前页的各节中搜索。

> 在搜索时，既可以使用鼠标或手指选择选项，也可以使用键盘的箭头 <kbd>↑</kbd> <kbd>↓</kbd>
> 与回车键 <kbd>⏎</kbd> 转到到当前所选页面。

> 这项搜索只使用页面标题与关键字搜索。也可以在
> `ktor.kotlincn.net` 域名下使用谷歌搜索对其所有内容进行全文检索。

> 折叠的大段代码，既可以在移动设备上可以通过点击<!--
> -->一直出现在左上角的 `'+'`/`'-'` 号来展开
> ~~、在有鼠标的设备通过鼠标悬停展开~~（译者注：实测不可行）。
> 也可以双击代码片段来展开。
> 除了展开之外，这一操作还会选中这段文本，于是可以轻松复制该代码段：
> 在 mac 上使用 <kbd>cmd</kbd> + <kbd>c</kbd>，而在其他操作系统中使用 <kbd>ctrl</kbd> + <kbd>c</kbd>。

> 可以点击标题以及一些注记来获取对应节的锚点链接。
> 点击后，可以在浏览器中复制包含 `#` 链向指定节的新 url。

## 我的路由没有执行到，该如何调试？
{: #route-not-executing }

Ktor 为路由功能提供了跟踪机制以帮助解决<!--
-->路由决策问题。请查阅路由页的[追踪路由决策](/features/routing.html#tracing)一节。

## 出现报错 `io.ktor.pipeline.InvalidPhaseException: Phase Phase('YourPhase') was not registered for this pipeline`.
{: #invalid-phase }

这意味着正在试图使用一个未注册为另一阶段（phase）引用的阶段。
例如，这可能发生在路由功能中，当尝试在一个节点内部注册阶段关系，
但是所引用的阶段在另一个祖先 Route 节点中定义。 由于路由阶段与拦截器稍后才<!--
-->合并，因此应该能生效，不过需要在 Route 节点中注册：

```kotlin
route.addPhase(PhaseDefinedInAncestor)
route.insertPhaseAfter(PhaseDefinedInAncestor, MyNodePhase)
```

## 出现报错 `io.ktor.server.engine.BaseApplicationResponse$ResponseAlreadySentException: Response has already been sent`
{: #response-already-sent }

这意味着你或者一个特性或拦截器已经调用了 `call.respond*` 函数，而你又再次调用了它<!--
-->。

## 如何订阅 ktor 事件？
{: #ktor-events }

这里有一个[解释 Ktor 应用级事件系统](/advanced/events.html)的页面。

## 出现 `Exception in thread "main" com.typesafe.config.ConfigException$Missing: No configuration setting found for key 'ktor'` 异常
{: #cannot-find-application-conf }

这意味着 Ktor 无法找到 `application.conf` 文件。请重新检查它是否在 `resources` 文件夹中<!--
-->并且该资源文件夹已正确标记。
可以考虑使用[项目生成器](/quickstart/generator.html)或者 [IntelliJ 插件](/quickstart/intellij-idea/plugin.html)<!--
-->搭建项目作为工作项目的基础。

## 可以在 Android 上使用 ktor 吗？
{: #android-support }

已知 Ktor 0.9.3 及以前版本适用于 Android 7 或更高版本（API 24）。会在 Android 5 等较低版本中失败。

在不支持的版本中会失败并出现类似以下异常：

```
E/AndroidRuntime: FATAL EXCEPTION: main Process: com.mypackage.example, PID: 4028 java.lang.NoClassDefFoundError:
io.ktor.application.ApplicationEvents$subscribe$1 at io.ktor.application.ApplicationEvents.subscribe(ApplicationEvents.kt:18) at
io.ktor.server.engine.BaseApplicationEngine.<init>(BaseApplicationEngine.kt:29) at
io.ktor.server.engine.BaseApplicationEngine.<init>(BaseApplicationEngine.kt:15) at
io.ktor.server.netty.NettyApplicationEngine.<init>(NettyApplicationEngine.kt:17) at io.ktor.server.netty.Netty.create(Embedded.kt:10) at
io.ktor.server.netty.Netty.create(Embedded.kt:8) at io.ktor.server.engine.EmbeddedServerKt.embeddedServer(EmbeddedServer.kt:50) at
io.ktor.server.engine.EmbeddedServerKt.embeddedServer(EmbeddedServer.kt:40) at
io.ktor.server.engine.EmbeddedServerKt.embeddedServer$default(EmbeddedServer.kt:27)
```

更多相关信息，请查阅[问题 #495](https://github.com/ktorio/ktor/issues/495) 以及 [StackOverflow
上的问题](https://stackoverflow.com/questions/49945584/attempting-to-run-an-embedded-ktor-http-server-on-android)

## CURL -I 返回 404 Not Found
{: #curl-head-not-found }

`CURL -I` 是执行 `HEAD` 请求的 `CURL --head` 的别名。
默认情况下，Ktor 不会处理对 `GET` 处理程序的 `HEAD` 请求，因此可能会遇到类似这样的内容：

```kotlin
curl -I http://localhost:8080
HTTP/1.1 404 Not Found
Content-Length: 0
```

对于：

```kotlin
routing {
    get("/") { call.respondText("HELLO") }
}
```

Ktor 可以自动处理 `HEAD` 请求，不过需要先安装 [`AutoHeadResponse` 特性](/features/autoheadresponse.html)。

## I get an infinite redirect when using the `HttpsRedirect` feature
{: #infinite-redirect }

The most probable cause is that your backend is behind a reverse-proxy or a load balancer, and that the reverse-proxy
is making normal HTTP requests to your backend, thus the HttpsRedirect feature inside your Ktor backend believes
that it is a normal HTTP request and responds with the redirect.

Normally, reverse-proxies send some headers describing the original request (like it was https, or the original IP address),
and there is a feature [`XForwardedHeaderSupport`](/features/forward-headers.html)
to parse those headers so the [`HttpsRedirect`](/features/https-redirect.html) feature knows that the original request was HTTPS.


