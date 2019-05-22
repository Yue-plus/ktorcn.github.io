---
title: WebSocket
category: clients
permalink: /clients/websockets.html
caption: WebSocket
feature:
    artifact: io.ktor:ktor-client-websocket:$ktor_version,io.ktor:ktor-client-cio:$ktor_version,io.ktor:ktor-client-js:$ktor_version,io.ktor:ktor-client-okhttp:$ktor_version
    class: io.ktor.client.features.websocket.WebSockets
ktor_version_review: 1.2.0
---

{% include feature.html %}

Ktor 为 CIO、OkHttp、Js 这几个引擎提供了 WebSocket 客户端。如需了解服务端的相关信息，请参见[这一节](/servers/features/websockets.html)。

一旦连接后，客户端与服务器的 WebSocket 就共享相同的 [WebSocketSession](/servers/features/websockets.html#WebSocketSession)
接口进行通信。

创建支持 WebSocket 的 HTTP 客户端的基本用法非常简单：

```kotlin
val client = HttpClient {
    install(WebSockets)
}
```

一旦创建后就可以执行请求，启动一个 `WebSocketSession`：

```kotlin
client.ws(
    method = HttpMethod.Get,
    host = "127.0.0.1",
    port = 8080, path = "/route/path/to/ws"
) { // this: DefaultClientWebSocketSession

    // 发送文本帧。
    send("Hello, Text frame")

    // 发送文本帧。
    send(Frame.Text("Hello World"))

    // 发送二进制帧。
    send(Frame.Binary(……))

    // 接收帧。
    val frame = incoming.receive()
    when (frame) {
        is Frame.Text -> println(frame.readText())
        is Frame.Binary -> println(frame.readBytes())
    }
}
```

关于 WebSocketSession 的更多信息请参见 [WebSocketSession 页](/servers/features/websockets.html#WebSocketSession)及其 [API 参考](https://api.ktor.io/{{ site.ktor_version }}/io.ktor.client.features.websocket/)。
