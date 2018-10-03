---
title: Gradle
caption: 在 IntelliJ IDEA（Gradle）中搭建项目
category: quickstart
toc: true 
permalink: /quickstart/quickstart/intellij-idea/gradle.html
priority: 0
---

本教程会指导你从最基本的搭建到可用于开始开发应用的全<!--
-->功能搭建。

**目录：**

* TOC
{:toc}

## 前提条件

1.  最新版的 IntelliJ IDEA
2.  已启用 Kotlin 与 Gradle 插件（它们应该都已默认启用）

可以在 IntelliJ IDEA 中通过以下主菜单查验：
* Windows: `File -> Settings -> Plugins`
* Mac: `IntelliJ IDEA -> Settings -> Plugins`

## 开始一个项目

1.  `File -> New -> Project`:

    ![Ktor IntelliJ: File New Project](/quickstart/intellij-idea/file-new-project.png)

2.  选择 Gradle，并且在 Additional Libraries and Frameworks 下选中 Java 与 Kotlin (Java)。 确认 Project SDK 已配置好并点击 `Next`：

    ![Ktor IntelliJ: Gradle Kotlin JVM](/quickstart/intellij-idea/gradle-kotlin-jvm.png)

3.  输入 GroupId: `Example`
    以及 ArtifactId: `Example`
    并点击 Next:

    ![Ktor IntelliJ: GroupId](/quickstart/intellij-idea/groupid.png)

4.  选中复选框 `Use auto-import` 以及 `Create separate module per source set`。 确认选中单选框 Use default gradle wrapper 并且 Gradle JVM 已有值，然后点击 `Next`：

    ![Ktor IntelliJ: Gradle Config](/quickstart/intellij-idea/gradle-config.png)

5.  填写项目名： `Example`
    与项目位置： `a/path/on/your/filesystem`
    并点击 `Finish`：

    ![Ktor IntelliJ: Project Location Name](/quickstart/intellij-idea/project-location-name.png)

6.  等待 Gradle 运行几秒，应该能看到以下项目结构（以及其他几个文件与目录）：

    ![Ktor IntelliJ: Project Structure](/quickstart/intellij-idea/project-structure.png)

7.  更新 `build.gradle` 文件，添加使相关类可用的构件与仓库：
    * 将 `compile("io.ktor:ktor-server-netty:$ktor_version")` 包含到 `build.gradle` 的 `dependencies` 块中
    * 将  `maven { url = uri("http://kotlin.bintray.com/ktor") }` 与 `jcenter()` 包含到 `repositories` 块中

    ![Ktor IntelliJ: Build Gradle](/quickstart/intellij-idea/build-gradle.png)

关于配置 `build.gradle` 文件的更详细的指南，请查阅[以 Gradle 入门](/quickstart/quickstart/gradle.html)一节。
{: .note}

## 创建应用

选择 `src/main/kotlin` 目录并创建一个新包。 我们称之为 `blog`。

选择该目录并在其下创建一个名为 `BlogApp` 的新的 kotlin 文件

![Ktor IntelliJ: Create Kotlin File](/quickstart/intellij-idea/create-kotlin-file.png)

![Ktor IntelliJ: Create Kotlin File Name](/quickstart/intellij-idea/create-kotlin-file-name.png)

复制并粘贴应用的最基本设置，使其看起来像：


```kotlin
package blog

import io.ktor.application.*
import io.ktor.http.*
import io.ktor.response.*
import io.ktor.routing.*
import io.ktor.server.engine.*
import io.ktor.server.netty.*

fun main(args: Array<String>) {
    embeddedServer(Netty, 8080) {
        routing {
            get("/") {
                call.respondText("My Example Blog", ContentType.Text.Html)
            }
        }
    }.start(wait = true)
}
```

![Ktor IntelliJ: Program](/quickstart/intellij-idea/program.png)

现在可以运行“`blog.BlogAppKt`”了。 可以这样做，点击边框中的三角图标并选择带有 **🐞**{: style="transform:rotate(90deg);display:inline-block;"} 符号的 `Debug 'blog.BlogAppKt'`：

![Ktor IntelliJ: Program Run](/quickstart/intellij-idea/program-run.png)

这也会在 IntelliJ 的右上角创建一个运行配置，会使再次运行<!--
-->该配置很容易：

![Ktor IntelliJ: Program Run Config](/quickstart/intellij-idea/program-run-config.png)

这会启动该 Netty web 服务器。
在浏览器中输入其 URL： localhost:8080
然后应该就能看到示例博客页了。

![Ktor IntelliJ: Website](/quickstart/intellij-idea/website.png)

## 使用 Application 对象改进应用

上述设置中有很多嵌套的块，而这不适合开始<!--
-->向应用中添加功能。 我们可以这样改进之：通过使用 Application 对象<!--
-->并在 main 函数中的 embeddedServer 调用引用该对象。

将 BlogApp.kt 中的代码改为以下这样来尝试这点：

```kotlin
package blog

import io.ktor.application.*
import io.ktor.features.*
import io.ktor.http.*
import io.ktor.response.*
import io.ktor.routing.*
import io.ktor.server.engine.*
import io.ktor.server.netty.*

fun Application.module() {
    install(DefaultHeaders)
    install(CallLogging)
    install(Routing) {
        get("/") {
            call.respondText("My Example Blog2", ContentType.Text.Html)
        }
    }
}

fun main(args: Array<String>) {
    embeddedServer(Netty, 8080, watchPaths = listOf("BlogAppKt"), module = Application::module).start()
}
```

## 提取配置数据

虽然可以在主函数的 embeddedServer 调用中指定一些应用配置数据，但是要为将来的部署与更改提供更多的灵活性的话，可以将其提取到一个独立的配置文件中。在 `src/main/resources` 目录中创建一个新的文本文件名为 `application.conf`，其内容如下：

```kotlin
ktor {
    deployment {
        port = 8080
    }

    application {
        modules = [ blog.BlogAppKt.main ]
    }
}
```

然后我们在 `BlogApp.kt` 中删除主函数并将 `fun Application.module()` 改为 `fun Application.main()`。 然而，如果我们现在运行该应用，它会失败并且有像“Top-level function 'main' not found in package blog.”这样的错误信息。 我们的 `Application.main()` 函数现在是一个扩展函数而不具备顶层主函数的作用。

因为 IntelliJ IDEA 不能自动找到主类，这需要我们指定一个新的主类。在 `build.gradle` 添加：

{% capture gradle-groovy-build %}
```groovy
// build.gradle

apply plugin: 'application'

mainClassName = 'io.ktor.server.netty.DevelopmentEngine'
```
{% endcapture %}

{% capture gradle-kotlin-build %}
```kotlin
// build.gradle.kts

plugins {
    application
    // ...
}

application {
    mainClassName = "io.ktor.server.netty.DevelopmentEngine"
}
```
{% endcapture %}

{% include gradle.html gradle-kotlin=gradle-kotlin-build gradle-groovy=gradle-groovy-build %}

然后转到 `Run -> Edit Configurations` 选择 `blog.BlogAppKt` 配置并将其 Main class 修改为：
`io.ktor.server.netty.DevelopmentEngine`

现在，当我们运行新的配置时，应用程序会再次启动。
