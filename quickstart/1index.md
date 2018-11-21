---
title: 快速入门
caption: 快速入门
category: quickstart
toc: true
permalink: /quickstart/index.html
children: /quickstart/quickstart/
ktor_version_review: 1.0.0
---

![Ktor logo](/assets/images/ktor_logo.svg){:style="width:134px;height:56px;"}
 
Ktor 是一个轻松构建联网应用（web 应用、 HTTP 服务、 移动应用以及浏览器应用）的框架。
现代的联网应用需要异步化来提供最佳的用户体验，而 Kotlin 协程为此提供了<!--
-->极其简便的方式。

虽然还没有完全实现，但是 Ktor 的目标是为联网应用提供端到端的多平台应用框架。
目前支持 JVM 客户端与服务器场景，以及 JavaScript、iOS 与 Android 客户端，而我们（官方）正努力将服务端支持引入到原生（native）
环境，并将客户端支持引入到其他原生平台。

**目录：**

* TOC
{:toc}

## Set up a Ktor project

You can set up a Ktor project using [Maven](/quickstart/quickstart/maven.html), [Gradle](/quickstart/quickstart/gradle.html), [start.ktor.io](/quickstart/generator.html#) and the [IntelliJ Plugin](/quickstart/quickstart/intellij-idea/plugin.html).

The plugin allows you to create a Ktor project as well as [start.ktor.io](/quickstart/generator.html#), but with the additional convenience of being fully integrated in the IDE.
If you don't have the plugin yet, there is a page about [how to install the plugin](/quickstart/quickstart/intellij-idea/plugin.html).

1) In a first step, you can configure the project to generate and select features to install:
![](/quickstart/quickstart/intellij-idea/plugin/ktor-plugin-1.png){: width="100%" }

2) In a second step, you can configure the project artifacts:
![](/quickstart/quickstart/intellij-idea/plugin/ktor-plugin-2.png){: width="100%" }

And that's it. A new project will be created and opened inside your IDE.

## Hello World

Ktor 的一个简单的 hello world 如下所示：

![Ktor Hello World](/quickstart/1/ktor_hello_world_main.png){: width="100%" }

1. 在这里定义一个寻常可调用的 *main 方法*。
2. 然后创建一个内嵌的*使用 Netty 的服务器*作为监听在*8080 端口*上的后端。
3. 通过一个代码块配置*路由特性*，在代码块中可以为指定的路径与 HTTP 方法定义路由。
4. 实际路由：在本例中，它会处理 `/demo` 路径的 *GET 请求*，并以一个 `HELLO WORLD!` 消息回复。
5. 实际*启动服务器*并等待连接。

{% capture main-kt %}
```kotlin
import io.ktor.application.*
import io.ktor.http.*
import io.ktor.response.*
import io.ktor.routing.*
import io.ktor.server.engine.*
import io.ktor.server.netty.*

fun main(args: Array<String>) {
    val server = embeddedServer(Netty, port = 8080) {
        routing {
            get("/") {
                call.respondText("Hello World!", ContentType.Text.Plain)
            }
            get("/demo") {
                call.respondText("HELLO WORLD!")
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


## 访问应用

由于有 main 方法，因此可以用 IDE 执行。这会打开一个 HTTP 服务器，
监听在 [http://127.0.0.1:8080](http://127.0.0.1:8080/)，可以尝试用你喜欢的 web 浏览器打开它。

如果这不起作用，可能是计算机已占用该端口。可以尝试修改
（在第 10 行的）8080 端口并按需调整。
{: .note}

![Ktor Hello World Browser](/quickstart/1/screenshot.png){: width="50%""}

此时应该有一个非常简单的 Web 后端在运行，所以你可以进行修改，
并在浏览器中查看结果。

由于已经为 Gradle 项目配置了 application 插件与 `mainClassName`，
因此可以在终端中运行，Linux/Mac 中使用 `./gradlew run`，Windows 中使用 `gradlew run`。
{:.note}

{::comment}
## 下一步

现在准备好下一步了。*你在开发哪种应用？*

1. [RESTful API: 让我们将*数据类*作为 JSON 提供服务](/quickstart/restful.html)
2. Web 应用：
    * [让我们使用完全静态类型的 kotlinx.html DSL 方式描述 HTML 并提供服务。](/quickstart/html-dsl.html)
    * [让我们使用 FreeMarker 模板引擎提供 HTML 服务。](/quickstart/html-freemarker.html)
    
{:/comment}

## 相关链接

{% include category-list.html %}
