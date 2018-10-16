---
title: 快速入门
caption: 快速入门
category: quickstart
toc: true
permalink: /quickstart/index.html
children: /quickstart/quickstart/
priority: -100
---

![Ktor logo](/assets/images/ktor_logo.svg){:style="width:134px;height:56px;"}
 
Ktor 是一个轻松构建联网应用（web 应用、 HTTP 服务、 移动应用以及浏览器应用）的框架。
现代的联网应用需要异步化来提供最佳的用户体验，而 Kotlin 协程为此提供了<!--
-->极其简便的方式。

虽然还没有完全实现，但是 Ktor 的目标是为联网应用提供端到端的多平台应用框架。
目前支持 JVM 客户端与服务器场景，而我们（官方）正努力将服务端支持引入到原生（native）
环境，并将客户端支持引入到原生与 JavaScript 环境。

{::comment}
Ktor embraces the strongly typed nature of the Kotlin programming language and provides [strongly typed end-points (Locations)](/servers/features/locations.html) and
the ability to exchange data with classes shared across platforms.
{:/comment}

**目录：**

* TOC
{:toc}

{::comment}
## start.ktor.io

Ktor has a [start.ktor.io](https://soywiz.github.io/start-ktor-io-proposal/) website to quickly generate a ZIP with a skeleton for your application:

<iframe src="https://soywiz.github.io/start-ktor-io-proposal/" style="border:1px solid #aaa;width:100%;height:450px;"></iframe>
{:/comment}

## Gradle 设置

本节假定你有一些 Gradle 的基本知识。如果你从未使用过 Gradle，
那么可参考 gradle.org 提供的[一些指南](https://guides.gradle.org/building-java-applications/)来帮你入门。
{: .note}

你可以像这样使用 Gradle 搭建一个简单的 Ktor 应用：

![Ktor Build with Gradle](/quickstart/1/ktor_build_gradle.png)

{% capture gradle-kotlin-build %}
```kotlin
// build.gradle.kts

import org.jetbrains.kotlin.gradle.dsl.Coroutines
import org.jetbrains.kotlin.gradle.tasks.KotlinCompile

group = "Example"
version = "1.0-SNAPSHOT"

val ktor_version = "{{ site.ktor_version }}"

plugins {
    application
    kotlin("jvm") version "{{ site.kotlin_version }}"
}

kotlin.experimental.coroutines = Coroutines.ENABLE

repositories {
    mavenCentral()
    jcenter()
    maven { url = uri("https://dl.bintray.com/kotlin/ktor") }
    maven { url = uri("https://dl.bintray.com/kotlin/kotlin-eap") }
}

java {
    sourceCompatibility = JavaVersion.VERSION_1_8
}

tasks.withType<KotlinCompile>().all {
    kotlinOptions.jvmTarget = "1.8"
}

application {
    mainClassName = "MainKt"
}

dependencies {
    compile(kotlin("stdlib-jdk8"))
    compile("io.ktor:ktor-server-netty:$ktor_version")
    compile("ch.qos.logback:logback-classic:1.2.3")
    testCompile(group = "junit", name = "junit", version = "4.12")
}
```
{: .compact}
{% endcapture %}

{% capture gradle-groovy-build %}
```groovy
// build.gradle

group 'Example'
version '1.0-SNAPSHOT'

buildscript {
    ext.kotlin_version = '{{ site.kotlin_version }}'
    ext.ktor_version = '{{ site.ktor_version }}'

    repositories {
        mavenCentral()
        maven { url "https://dl.bintray.com/kotlin/kotlin-eap" }
    }
    dependencies {
        classpath "org.jetbrains.kotlin:kotlin-gradle-plugin:$kotlin_version"
    }
}

apply plugin: 'java'
apply plugin: 'kotlin'
apply plugin: 'application'

mainClassName = 'MainKt'

sourceCompatibility = 1.8
compileKotlin { kotlinOptions.jvmTarget = "1.8" }
compileTestKotlin { kotlinOptions.jvmTarget = "1.8" }

kotlin { experimental { coroutines "enable" } }

repositories {
    mavenCentral()
    jcenter()
    maven { url "https://dl.bintray.com/kotlin/ktor" }
    maven { url "https://dl.bintray.com/kotlin/kotlin-eap" }
}

dependencies {
    compile "org.jetbrains.kotlin:kotlin-stdlib-jdk8:$kotlin_version"
    compile "io.ktor:ktor-server-netty:$ktor_version"
    compile "ch.qos.logback:logback-classic:1.2.3"
    testCompile group: 'junit', name: 'junit', version: '4.12'
}
```
{: .compact}
{% endcapture %}

文本版：
{% include gradle.html gradle-kotlin=gradle-kotlin-build gradle-groovy=gradle-groovy-build %}

由于 Ktor 还没到 1.0，我们（官方）用自定义的 Maven 仓库来分发早期预览版构件。
必须设置如下所示的几个仓库，以便工具可以找到 Ktor 构件及其依赖。

当然，不要忘记包含实际的构件！对于快速入门，我们使用 `ktor-server-netty` 构件。
这包括 Ktor 的核心、netty 以及 ktor-netty 连接器作为传递依赖。
当然你还可以包含任何其他所需要的额外依赖。

由于 Ktor 设计为模块化的，对于指定特性会需要额外的构件并且有可能需要其他仓库<!--
-->。 可以在指定特性的文档中找到每个特性所需的构件（以及所需仓库）<!--
-->。
{:.note}

## Hello World

Ktor 的一个简单的 hello world 如下所示：

![Ktor Hello World](/quickstart/1/ktor_hello_world_main.png)

1. 在这里定义一个寻常可调用的 *main 方法*。
2. 然后创建一个内嵌的*使用 Netty 的服务器*作为监听在*8080 端口*上的后端。
3. 通过一个代码块配置*路由特性*，在代码块中可以为指定的路径与 HTTP 方法定义路由。
4. 实际路由：在本例中，它会处理 `/demo` 路径的 *GET 请求*，并以一个 `HELLO WORLD!` 消息回复。
5. 实际*启动服务器*并等待连接。

文本版：
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
{: .compact}

## 访问应用

由于有 main 方法，因此可以用 IDE 执行。这会打开一个 HTTP 服务器，
监听在 [http://127.0.0.1:8080](http://127.0.0.1:8080/)，可以尝试用你喜欢的 web 浏览器打开它。

如果这不起作用，可能是该端口已被占用。可以尝试修改
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
