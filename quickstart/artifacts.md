---
title: 构件
caption: 构件列表
permalink: /quickstart/artifacts.html
category: quickstart
priority: 1
redirect_from:
  - /artifacts.html
---

Ktor 分为多个模块，以便根据所需功能实现细粒度的依赖关系。
典型的 Ktor 应用需要 `ktor-server-core` 以及一个相应的引擎，具体取决于它是自托管的<!--
-->还是使用应用服务器（Application Server）。

Ktor 中的所有构件都属于 `io.ktor` group 并且托管在 [Bintray](https://bintray.com/kotlin/ktor) 上。

[![Download](https://api.bintray.com/packages/kotlin/ktor/ktor/images/download.svg?version={{site.ktor_version}})](https://bintray.com/kotlin/ktor/ktor/{{site.ktor_version}})
    
Ktor 分为以下几组模块：

* `ktor-server` 包含支持使用不同引擎运行 Ktor 应用程序的模块：Netty、Jetty、Tomcat 以及<!--
-->通用 servlet。它还包含了一个用于设置应用测试而不真正启动服务器的 TestEngine
  * `ktor-server-core` 是大多数应用 API 与实现所在的核心包
  * `ktor-server-jetty` 支持部署版或内嵌版 Jetty 实例
  * `ktor-server-netty` 支持嵌入模式的 Netty
  * `ktor-server-tomcat` 支持 Tomcat 服务器
  * `ktor-server-servlet` 由 Jetty 与 Tomcat 使用，并允许在通用的 servlet 容器中运行
  * `ktor-server-test-host` 允许在不启动完整主机的情况下更快地运行应用测试
* `ktor-features` 为并非每个应用都需要的可选特性分组的模块
  * `ktor-auth` 为不同的[身份认证系统](/servers/features/authentication.html)如 Basic、 Digest、 表单、 OAuth 1a 与 OAuth 2 提供支持
  * `ktor-auth-jwt` 添加针对 [JWT](/servers/features/authentication/jwt.html) 认证的能力
  * `ktor-auth-ldap` 添加针对 [LDAP](/servers/features/authentication/ldap.html) 实例认证的能力
  * `ktor-freemarker` 将 Ktor 与 [Freemarker 模板](/servers/features/templates/freemarker.html)集成
  * `ktor-velocity` 将 Ktor 与 [Velocity 模板](/servers/features/templates/velocity.html)集成
  * `ktor-gson` 与 [Gson](/servers/features/content-negotiation/gson.html) 集成并添加 JSON 内容协商
  * `ktor-jackson` 与 [Jackson](/servers/features/content-negotiation/jackson.html) 集成并添加 JSON 内容协商
  * `ktor-html-builder` 将 Ktor 与 [kotlinx.html 构建器](/servers/features/templates/html-dsl.html)集成
  * `ktor-locations` 包含对[类型化 location](/servers/features/locations.html) 的实验性的支持
  * `ktor-metrics` 添加了为服务器添加一些[指标](/servers/features/metrics.html)的能力
  * `ktor-server-sessions` 添加使用[服务器存储的有状态会话](/servers/features/sessions.html)的能力
  * `ktor-websockets` 提供 [Websockets](/servers/features/websockets.html) 支持
* `ktor-client` 包含[执行 http 请求](/clients/http-client.html)的模块
  * `ktor-client-core` 是大多数 http HttpClient API 所在的核心包
  * `ktor-client-apache` 添加对 Apache 异步 HttpClient 的支持
  * `ktor-client-cio`  添加对纯 Kotlin 基于协程的 I/O 异步 HttpClient 的支持
  * `ktor-client-jetty` adds support
  * `ktor-client-auth-basic` 添加对[认证](/clients/http-client/features/basic-auth.html)的支持
  * `ktor-client-json` 添加对 [json 内容协商](/clients/http-client/features/json-feature.html)的支持
* `ktor-network` 包含用于客户端/服务器的[原始套接字](/servers/raw-sockets.html)与 TCP/UDP
  * `ktor-network-tls` 包含对原始套接字的 TLS 支持
 
参见使用以下这些搭建项目的说明

* [Maven](/quickstart/quickstart/maven.html)
* [Gradle](/quickstart/quickstart/gradle.html)
* [在线生成一个项目](/quickstart/generator.html)

