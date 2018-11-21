---
title: WebSocket
category: clients
permalink: /clients/websockets.html
caption: WebSocket
feature:
    artifact: io.ktor:ktor-client-websocket:$ktor_version,io.ktor:ktor-client-cio:$ktor_version
    class: io.ktor.client.features.websocket.WebSockets
ktor_version_review: 1.0.0
---

{% include feature.html %}

除了支持[服务器端 WebSocket](/servers/features/websockets.html) 之外，Ktor 还提供仅支持 CIO 引擎的 WebSocket 客户端。

一旦连接后，客户端与服务器的 WebSocket 就共享相同的 [WebSocketSession](/servers/features/websockets.html#WebSocketSession)
接口进行通信。

目前，客户端 WebSocket 仅适用于 [CIO 客户端引擎](/clients/http-client/engines.html#cio)。

创建支持 WebSocket 的 HTTP 客户端的基本用法非常简单：

```kotlin
val client = HttpClient(CIO).config { install(WebSockets) }
```

一旦创建后就可以执行请求，启动一个 `WebSocketSession`：

```kotlin
client.ws(method = HttpMethod.Get, host = "127.0.0.1", port = 8080, path = "/route/path/to/ws") { // this: DefaultClientWebSocketSession
    send(Frame.Text("Hello World"))

    for (message in incoming.map { it as? Frame.Text }.filterNotNull()) {
        println(message.readText())
    }
}
```

可以配置超时与 ping 周期：

```kotlin
client.ws(……) { // this: DefaultClientWebSocketSession
    timeout = Duration.ofMinutes(10)
    pingInterval = Duration.ofMinutes(10) // null 则表示禁用
}
```

For more information about the WebSocketSession, check the [WebSocketSession page](/servers/features/websockets.html#WebSocketSession).
