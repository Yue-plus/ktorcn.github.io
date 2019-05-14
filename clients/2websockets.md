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

Ktor provides a WebSocket client for the following engines: CIO, OkHttp, Js. To get more information about the server side, follow this [section](/servers/features/websockets.html).

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

    // Send text frame.
    send("Hello, Text frame")

    // Send text frame.
    send(Frame.Text("Hello World"))

    // Send binary frame.
    send(Frame.Binary(...))

    // Receive frame.
    val frame = incomming.receive()
    when (frame) {
        is Frame.Text -> println(frame.readText())
        is Frame.Binary -> println(frame.readBytes())
    }
}
```

关于 WebSocketSession 的更多信息请参见 [WebSocketSession 页](/servers/features/websockets.html#WebSocketSession) and the [API reference](https://api.ktor.io/{{ site.ktor_version }}/io.ktor.client.features.websocket/).
