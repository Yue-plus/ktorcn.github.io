---
title: 聊天
caption: 指南：如何使用 WebSocket 实现聊天
category: quickstart
permalink: /quickstart/guides/chat.html
ktor_version_review: 1.0.0
---

在本教程中，你会学习如何使用 Ktor 创建聊天应用。
我们会使用 WebSocket 进行实时双向通信。

为了实现这一点，我们会用到[路由]、 [WebSocket] 以及[会话]这些特性。

[路由]: /servers/features/routing.html
[WebSocket]: /servers/features/websockets.html
[会话]: /servers/features/sessions.html

这是一个篇级教程，假定你已经对 Ktor 有一些基本的了解，
因此你应该先看[关于制作网站的指南](/quickstart/guides/website.html)。

**目录：**

* TOC
{:toc}

## 搭建项目

第一步是搭建一个项目。可以按照[快速入门](/quickstart/index.html)指南操作，
或者使用以下表单创建：

{% include preconfigured-form.html hash="dependency=ktor-sessions&dependency=routing&dependency=ktor-websockets&artifact-name=chat" %}

## 理解 WebSocket

WebSocket 是 HTTP 的子协议。它以具有 upgrade 请求头的普通 HTTP 请求开始，
并且该连接会切换为双向通信取代请求响应通信。

可以作为 WebSocket 协议一部分发送的最小传输单元是 `Frame`（帧）。 WebSocket 帧定义了类型、长度以及可以是二进制或者文本的有效载荷。
在内部，这些帧可以透明地以多个 TCP 包发送。

可以将帧视为 WebSocket 消息。帧可以是以下类型：文本、 二进制、 关闭、 “乒”与“乓”。

你通常会处理 `Text` 与 `Binary` 帧，其他帧在大多数情况下会由 Ktor 处理
（虽然你可以使用原始模式，这样你可以自行处理那些额外的帧类型）。

可以在 [WebSocket 特性](/servers/features/websockets.html)页中查阅关于它的更多信息。

## WebSocket 路由

第一步是为 WebSocket 创建路由。在本例中，我们会定义 `/chat` 路由，
不过最初，我们会让该路由充当“回声”WebSocket 路由，它会向你发回与你发给它的内容相同的消息。

`webSocket` 路由是准备长期活跃的。由于它是一个挂起块并且使用轻量级 Kotlin 协程，
因此可以很好地同时处理数十万个连接（具体取决于计算机与复杂性）<!--
-->，同时保持代码易读易写。

```kotlin
routing {
    webSocket("/chat") { // this: DefaultWebSocketSession
        while (true) {
            val frame = incoming.receive() // suspend
            when (frame) {
                is Frame.Text -> {
                    val text = frame.readText()
                    outgoing.send(Frame.Text(text)) // suspend
                }
            }
        }
    }
}
```

## 保存一组打开的连接

我们可以使用 Set 来保存打开的连接列表。可以使用一个普通的 `try……finally` 来跟踪它们。
由于 Ktor 默认是多线程的，因此我们应该使用线程安全的集合或者[以 newSingleThreadContext 将代码体限制为单线程](https://github.com/Kotlin/kotlinx.coroutines/blob/master/coroutines-guide.md#coroutine-context-and-dispatchers){:target="_blank"}。

```kotlin
routing {
    val wsConnections = Collections.synchronizedSet(LinkedHashSet<DefaultWebSocketSession>())
    
    webSocket("/chat") { // this: DefaultWebSocketSession
        wsConnections += this
        try {
            while (true) {
                val frame = incoming.receive()
                // ……
            }
        } finally {
            wsConnections -= this
        }
    }
}
```

## 在所有连接之间广播消息

现在我们有了一组连接，可以对它们进行迭代并使用会话<!--
-->来发送需要的帧。
每当用户发送一条消息时，我们都会广播到所有已连接的客户端。

```kotlin
routing {
    val wsConnections = Collections.synchronizedSet(LinkedHashSet<DefaultWebSocketSession>())
    
    webSocket("/chat") { // this: DefaultWebSocketSession
        wsConnections += this
        try {
            while (true) {
                val frame = incoming.receive()
                when (frame) {
                    is Frame.Text -> {
                        val text = frame.readText()
                        // 迭代所有连接
                        for (conn in wsConnections) {
                            conn.outgoing.send(Frame.Text(text))
                        }
                    }
                }
            }
        } finally {
            wsConnections -= this
        }
    }
}
```

## 为用户/连接分配名称

我们可能希望将一些信息关联起来，如将名称与打开的连接关联，
可以创建一个包含 WebSocketSession 的对象并将其存储<!--
-->如下：

```kotlin
class ChatClient(val session: DefaultWebSocketSession) {
    companion object { var lastId = AtomicInteger(0) }
    val id = lastId.getAndIncrement()
    val name = "user$id"
}

routing {
    val clients = Collections.synchronizedSet(LinkedHashSet<ChatClient>())
    
    webSocket("/chat") { // this: DefaultWebSocketSession
        val client = ChatClient(this)
        clients += client
        try {
            while (true) {
                val frame = incoming.receive()
                when (frame) {
                    is Frame.Text -> {
                        val text = frame.readText()
                        // 迭代所有连接
                        val textToSend = "${client.name} said: $text"
                        for (other in clients.toList()) {
                            other.session.outgoing.send(Frame.Text(textToSend))
                        }
                    }
                }
            }
        } finally {
            clients -= client
        }
    }
}
```

## 练习

### 创建一个客户端

创建一个连接到这个端点的 JavaScript 客户端并以 ktor 为其提供服务。

### JSON

使用 [kotlinx.serialization](https://github.com/Kotlin/kotlinx.serialization) 来发送与接收 VO

