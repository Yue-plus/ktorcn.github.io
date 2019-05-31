---
title: Application
category: servers
permalink: /servers/application.html
caption: Application 是什么？
ktor_version_review: 1.2.1
---

Ktor 服务器应用（Application）是使用[已配置的服务器引擎](/servers/configuration.html)监听一个或多个端口的自定义程序，
由带有应用逻辑的[模块](#modules)组成，其中安装了一系列[特性](#特性)，如[路由](/servers/features/routing.html)、
[会话](/servers/features/sessions.html)、 [压缩](/servers/features/compression.html) 等，以处理 HTTP/S 1.x/2.x 与 WebSocket 请求。

Ktor 还提供了[处理原始套接字](/servers/raw-sockets.html)的功能，不过并不作为 Application 及<!--
-->其流水线的一部分。
{: .note}

**目录：**

* TOC
{:toc}

## Application

`Application` 实例是 Ktor 应用的主要单元。当请求
（可以是 HTTP、 HTTP/2 或者 WebSocket 请求）进来时，将其转换为 `ApplicationCall`
并经过一个隶属于 `Application` 的流水线。该流水线由一到多个<!--
-->先前安装的拦截器组成，提供像路由、
压缩这样的结束请求处理的功能。

通常 Ktor 程序通过[模块](#modules)来配置 Application 的流水线，
在其中[安装并配置特性](#特性)。

关于流水线的更多信息可参阅[生命周期](/servers/lifecycle.html)一节。
{: .note}

## ApplicationCall

参见[关于 ApplicationCall 专有页](/servers/calls.html)。

## 特性

特性是可以为流水线安装并配置的单例（通常是伴生对象）。
Ktor 包含一些标准的特性，不过可以添加自己的或者社区中的其他特性。
任何流水线中都可以安装特性，例如在应用自身中，或者指定的路由中。

更多相关内容请参阅[特性](/servers/features.html)专有页。
{: .note}

## 模块
{: #modules}

Ktor 模块只是一个接收者为 `Application` 类的用户自定义函数，负责配置<!--
-->服务器流水线、安装特性、注册路由、处理请求等。

必须在 [`application.conf` 文件](/servers/configuration.html#hocon-file)中指定服务器启动时要加载的模块。

一个简单的模块函数如下所示：

{% capture main-kt %}
```kotlin
package com.example.myapp

fun Application.mymodule() {
    routing {
        get("/") {
            call.respondText("Hello World!")
        }
    }
}
```
{% endcapture %}

{% include tabbed-code.html
    tab1-title="Main.kt" tab1-content=main-kt
    no-height="true"
%}

当然，也可以将模块函数拆分为几个较小的函数或者类。

使用完整限定名来引用模块：类的完整限定名与方法名，
以点（`.`）分隔。

因此对于该示例，其模块的完整限定名为：

```
com.example.myapp.MainKt.mymodule
```

`mymodule` 是 `Application` 类的一个扩展方法（其中 `Application` 是 *接收者*）。
由于它已定义成顶层函数，Kotlin 会创建一个后缀为 `Kt` 的 JVM 类（`FileNameKt`），
并将扩展方法添加为该类的静态方法，其接收者作为该静态方法的第一个参数。
在本例中，类名是 `com.example.myapp` 包中的 `MainKt`，而其 Java 方法的签名为
`static public void mymodule(Application app)`。
{: .note}

## What's next

- [Application calls](/servers/calls.html)
- [Application lifecycle explanation](/servers/lifecycle.html)
- [Application configuration](/servers/configuration.html)
- [Pipelines exlained](/advanced/pipeline)
