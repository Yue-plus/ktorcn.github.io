---
title: 客户端
category: clients
permalink: /clients/index.html
caption: 客户端
ktor_version_review: 1.0.0
---

除了服务器应用外，Ktor 还支持连接到其他服务的<!--
-->异步客户端。可以使用它创建纯客户端应用，
也可以创建混合的复杂应用：这类应用不仅提供资源服务，还与<!--
-->外部服务进行通信。

Ktor directly supports [Raw Sockets](/clients/raw-sockets.html), [HTTP Clients](/clients/http-client.html) and [WebSockets](/clients/websockets.html).
Using Raw Sockets it is possible to implement clients with higher level protocols.

{% include category-list.html %}
