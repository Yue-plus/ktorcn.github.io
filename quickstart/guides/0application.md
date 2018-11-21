---
title: 第一个应用
caption: 创建第一个应用
category: quickstart
permalink: /quickstart/guides/application.html
redirect_from:
- /quickstart/application.html
ktor_version_review: 1.0.0
---

在本教程中你会学到如何创建简单自托管的 Ktor 服务器应用，该应用程序以 `Hello, World!` 响应 HTTP 请求。
Ktor 应用可以使用通用构建系统（如 [Maven](/quickstart/quickstart/maven.html) 或 [Gradle](/quickstart/quickstart/gradle.html)）构建。

**目录：**

* TOC
{:toc}

## 包含正确的依赖
{: #dependencies }

Ktor 分为几组构件，
允许只包含所需的功能。And thus reducing the size of a fat-jar containing all the code, and the startup time.

在本例中，只需包含构件 `ktor-server-netty`。
关于所有可用构件的列表，请查阅[构件](/quickstart/artifacts.html)页。

Release versions of these dependencies are available at jcenter and maven central.
For pre-releases we host them on [Bintray kotlin/ktor](https://bintray.com/kotlin/ktor).

关于使用不同构建系统设置构建文件的更详细指南，请查阅：

* [搭建 Gradle 构建版](/quickstart/quickstart/gradle.html)
* [搭建 Maven 构建版](/quickstart/quickstart/maven.html)

## 创建自托管的应用
{: #self-hosted}

Ktor 允许应用在兼容 Servlet 的应用服务器（如 Tomcat）中运行，
也可以使用 Jetty、Netty 或者 CIO 作为内嵌应用。

在本教程中，我们会创建一个使用 Netty 的自托管应用。

可以通过调用 `embeddedServer` 函数开始，向其传入引擎工厂作为第一个参数、
端口号作为第二个参数，而将实际应用代码作为<!--
-->第四个参数传入（第三个参数是主机名，默认取 `0.0.0.0`）。

下述代码定义了单个路由，它对 URL `/` 上的 `GET` 动词的响应是<!--
-->文本 `Hello, world!`。

定义该路由后，必须通过调用 `server.start` 方法来启动服务器，
同时传入一个布尔参数来指示是否希望应用的主线程阻塞。

{% capture main-kt %}
```kotlin
import io.ktor.application.*
import io.ktor.http.*
import io.ktor.response.*
import io.ktor.routing.*
import io.ktor.server.engine.*
import io.ktor.server.netty.*

fun main(args: Array<String>) {
    val server = embeddedServer(Netty, 8080) {
        routing {
            get("/") {
                call.respondText("Hello, world!", ContentType.Text.Html)
            }
        }
    }
    server.start(wait = true)
}
```
{% endcapture %}

{% include tabbed-code.html
    tab1-title="Main.kt" tab1-content=main-kt
    no-height="true"
%}

&nbsp;

如果服务器设置为只是监听 HTTP 请求而之后不想做任何事，
通常会以 `wait = true` 调用 server.start。
{: .note}

## 运行该应用
{: #running }

考虑到应用的入口是标准的 Kotlin `main` 函数，
直接运行即可有效启动服务器并监听指定端口。

在浏览器中查验 `localhost:8080` 页，会看到 `Hello, world!` 文本。

## 下一步
{: #next-steps }

这是编写并运行自托管 Ktor 应用的最简单示例。
继续学习 Ktor 服务端开发的旅程会是：

* [应用是什么？](/servers/application.html)
* [各种特性](/features)
* [应用结构](/servers/structure.html)
* [进行测试](/servers/testing.html)
