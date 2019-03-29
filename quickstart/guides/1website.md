---
title: 网站
caption: 指南：如何使用 ktor 创建普通网站
category: quickstart
permalink: /quickstart/guides/website.html
ktor_version_review: 1.0.0
---

{::options toc_levels="1..2" /}

在本指南中，你会学习如何使用 Ktor 创建 HTML 网站。
我们会创建一个简单的网站，由后端渲染 HTML，有用户、登录表单<!--
-->以及保持持久会话。

为了实现这一点，我们会用到[路由]、 [状态页]、 [身份认证]、 [会话]、 [静态内容]、
[FreeMarker] 以及 [HTML DSL] 这些特性。

[路由]: /servers/features/routing.html
[状态页]: /servers/features/status-pages.html
[身份认证]: /servers/features/authentication.html
[会话]: /servers/features/sessions.html
[静态内容]: /servers/features/static-content.html
[FreeMarker]: /servers/features/templates/freemarker.html
[HTML DSL]: /servers/features/templates/html-dsl.html

**目录：**

* TOC
{:toc}

## 搭建项目

第一步是搭建一个项目。可以按照[快速入门](/quickstart/index.html)指南操作，或者使用以下表单创建：

{% include preconfigured-form.html hash="dependency=html-dsl&dependency=css-dsl&dependency=freemarker&dependency=static-content&dependency=auth&dependency=ktor-sessions&dependency=status-pages&dependency=routing&artifact-name=website-example" %}

## 简单路由

首先，我们要使用路由特性。 这个特性是 Ktor 核心的一部分，因此不需要<!--
-->包含任何其他构件。

这一特性在使用 `routing { }` 块时自动安装。

让我们开始创建一个响应为“OK”的简单 GET 路由：

```kotlin
fun Application.module() {
    routing {
        get("/") {
            call.respondText("OK")
        }
    }
}
```

## 以 FreeMarker 提供 HTML 服务

Apache FreeMarker 是一个JVM 上的模板引擎，因此可以将其与 Kotlin 一起使用。
有一个支持它的 Ktor 特性。

现在，我们会把作为资源的一部分而嵌入的模板存储在 `templates` 文件夹中。

创建一个名为 `resources/templates/index.ftl` 的文件，并输入以下内容来创建一个简单的 HTML 列表：

```freemarker
<#-- @ftlvariable name="data" type="com.example.IndexData" -->
<html>
	<body>
		<ul>
		<#list data.items as item>
			<li>${item}</li>
		</#list>
		</ul>
	</body>
</html>
```

IntelliJ IDEA Ultimate 具有带自动补全与变量提示功能的 FreeMarker 支持。
{:.note}

现在，我们来安装 FreeMarker 特性，然后创建一个以此模板提供服务的路由并为其传入一组值：

```kotlin
data class IndexData(val items: List<Int>)

fun Application.module() {
    install(FreeMarker) {
        templateLoader = ClassTemplateLoader(this::class.java.classLoader, "templates")
    }
    
    routing {
        get("/html-freemarker") {
            call.respond(FreeMarkerContent("index.ftl", mapOf("data" to IndexData(listOf(1, 2, 3))), ""))
        }
    }
}
```

现在可以运行服务器并用浏览器打开 <http://127.0.0.1:8080/html-freemarker>{:target="_blank"}  来查看结果：

![](/quickstart/guides/website/website1.png){:.rounded-shadow}

很好！

## 提供静态文件服务：样式、脚本、图片……

除了模板外，还会需要提供静态内容服务。
提供静态内容服务会更快，并且与一些其他特性（如部分内容）兼容，部分内容特性支持<!--
-->恢复文件下载或者部分下载文件）。

现在，我们将提供一个简单的 `styles.css` 文件来将样式应用到之前的简单页面中。

提供静态文件服务无需安装任何特性，而是一个简单路由处理程序。
如需以 `/resources/static` 在 `/static` url 提供静态文件服务，可以编写以下代码：

```kotlin
routing {
    // ……
    static("/static") {
        resources("static")
    }
}
```

现在我们来使用以下内容创建 `resources/static/styles.css` 文件：

```css
body {
    background: #B9D8FF;
}
```

除此之外，还必须更新模板以包含 `style.css` 文件：
```freemarker
<#-- @ftlvariable name="data" type="com.example.IndexData" -->
<html>
    <head>
        <link rel="stylesheet" href="/static/styles.css">
    </head>
    <body>
	<!-- …… -->
    </body>
</html>
```

其结果为：

![](/quickstart/guides/website/website2.png){:.rounded-shadow}

现在我们有一个来自 1990 年的彩色网站！

静态文件并非只有文本文件！试试向 `static` 文件夹中添加一个图片（花哨的动画闪烁 gif 文件怎么样？👩🏻‍🎨），并在 HTML 模板中包含一个 `<img src="……">` 标签。
{: .note.exercise}

## 启用部分内容特性：大文件与视频

虽然对于目前这个场景并非真正需要，但是如果启用部分内容支持，那么人们就能够<!--
-->在频繁出问题的连接上恢复较大的静态文件传输，也能够当提供视频服务并观看视频时<!--
-->支持拖动进度。

启用部分内容特性非常简单：

```kotlin
install(PartialContent) {
}
```

## 创建一个表单

现在要创建一个伪登录表单。为简单起见，我们会接受用户名与密码相同，
并且不会实现注册表单。

创建 `resources/templates/login.ftl`：

```kotlin
<html>
<head>
    <link rel="stylesheet" href="/static/styles.css">
</head>
<body>
<#if error??>
    <p style="color:red;">${error}</p>
</#if>
<form action="/login" method="post" enctype="application/x-www-form-urlencoded">
    <div>User:</div>
    <div><input type="text" name="username" /></div>
    <div>Password:</div>
    <div><input type="password" name="password" /></div>
    <div><input type="submit" value="Login" /></div>
</form>
</body>
</html>
```

除了模板，还需为其添加一些逻辑。在本例中，会在不同的代码块中处理 GET 与 POST 方法：

```kotlin
route("/login") {
    get {
        call.respond(FreeMarkerContent("login.ftl", null))
    }
    post {
        val post = call.receiveParameters()
        if (post["username"] != null && post["username"] == post["password"]) {
            call.respondText("OK")
        } else {
            call.respond(FreeMarkerContent("login.ftl", mapOf("error" to "Invalid login")))
        }
    }
}
```

如上所述，接受 `username` 与 `password` 相同，但不接受空值。
如果登录有效，现在只响应一个 OK，而如果登录失败，我们复用该模板<!--
-->以显示相同表单，只是带有错误信息。

## 重定向

在某些场景中，如路由重构或者表单提交，会希望执行（临时或永久）重定向。
在本例中，我们希望在成功登录后临时重定向到主页，而不是使用纯文本回复。

<table class="compare-table"><thead><tr><th>Original:</th><th>Change:</th></tr></thead><tbody><tr><td markdown="1">

```kotlin
call.respondText("OK")
```

</td><td markdown="1">

```kotlin
call.respondRedirect("/", permanent = false)
```

</td></tr></tbody></table>

## 使用表单认证

为了阐述如何接收 POST 参数，我们已手动处理登录，但是我们也可以使用以表单提供的身份认证<!--
-->特性：

```kotlin
install(Authentication) {
    form("login") {
        userParamName = "username"
        passwordParamName = "password"
        challenge = FormAuthChallenge.Unauthorized
        validate { credentials -> if (credentials.name == credentials.password) UserIdPrincipal(credentials.name) else null }
    }
}
route("/login") {
    get {
        // ……
    }
    authenticate("login") {
        post {
            val principal = call.principal<UserIdPrincipal>()
            call.respondRedirect("/", permanent = false)
        }
    }
}
```

## 会话

为了防止必须对所有页面都进行身份认证，我们会把用户存储在会话中，并<!--
-->使用会话 cookie 将该会话传播到所有页面中。

```kotlin
data class MySession(val username: String)

fun Application.module() {
    install(Sessions) {
        cookie<MySession>("SESSION")
    }
    routing {
        authenticate("login") {
            post {
                val principal = call.principal<UserIdPrincipal>() ?: error("No principal")
                call.sessions.set("SESSION", MySession(principal.name))
                call.respondRedirect("/", permanent = false)
            }
        }
    }
} 
```

在页面内部，我们可以尝试获取会话并产生不同的结果：

```kotlin
fun Application.module() {
    // ……
    get("/") {
        val session = call.sessions.get<MySession>()
        if (session != null) {
            call.respondText("User is logged")
        } else {
            call.respond(FreeMarkerContent("index.ftl", mapOf("data" to IndexData(listOf(1, 2, 3))), ""))
        }
    }
}
```

## 使用 HTML DSL 取代 FreeMarker

可以选择直接由代码生成 HTML 而不是使用模板引擎。
为此，可以使用 HTML DSL。这个 DSL 并不需要安装，但需要一个额外的构件。
这个构件提供了使用 HTML 块来响应的扩展：

```kotlin
get("/") { 
    val data = IndexData(listOf(1, 2, 3))
    call.respondHtml {
        head {
            link(rel = "stylesheet", href = "/static/styles.css")
        }
        body {
            ul {
                for (item in data.items) {
                    li { +"$item" }                
                }
            }
        }
    }
}
```

HTML DSL 的主要好处是可以对变量进行完全静态类型访问，并<!--
-->与代码完全整合。

所有这些的代价是必须重新编译才能更改 HTML，并且无法搜索完整的 HTML 块。
不过它快如闪电，还可以使用[自动加载特性](/servers/autoreload.html)<!--
-->在变更时重新编译并重新加载相关的 JVM 类。

## 练习

### 练习一

制作一个注册页并将用户名/密码数据源存储在内存中的 hashmap 中。

### 练习二

使用数据库存储用户。
