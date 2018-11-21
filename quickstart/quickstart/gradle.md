---
title: Gradle
caption: 搭建 Gradle 构建版
category: quickstart
toc: true
permalink: /quickstart/quickstart/gradle.html
redirect_from:
  - /quickstart/gradle.html
  - /quickstart/quickstart/intellij-idea/gradle.html
  - /quickstart/quickstart/intellij-idea/plugin.html
---

在本指南中，我们会展示如何创建 `build.gradle` 文件<!--
-->以及如何配置以支持 Ktor。

**目录：**

* TOC
{:toc}

## 基本的 Kotlin `build.gradle`文件（不带 Ktor）
{: #initial }

首先，需要一个包含 Kotlin 的骨架 `build.gradle` 文件。
可以使用任何文本编辑器创建它，也可以使用 IntelliJ 按照
[IntelliJ 指南](/quickstart/quickstart/intellij-idea.html)来创建。

初始文件如下所示：

{% capture build-gradle %}
```groovy
group 'Example'
version '1.0-SNAPSHOT'

buildscript {
    ext.kotlin_version = '{{site.kotlin_version}}'

    repositories {
        mavenCentral()
    }
    dependencies {
        classpath "org.jetbrains.kotlin:kotlin-gradle-plugin:$kotlin_version"
    }
}

apply plugin: 'java'
apply plugin: 'kotlin'

sourceCompatibility = 1.8

repositories {
    jcenter()
}

dependencies {
    compile "org.jetbrains.kotlin:kotlin-stdlib-jdk8:$kotlin_version"
    testCompile group: 'junit', name: 'junit', version: '4.12'
}
```
{% endcapture %}

{% include tabbed-code.html
    tab1-title="build.gradle" tab1-content=build-gradle
%}

## 添加 Ktor 依赖并配置构件设置
{: #ktor-dependencies}

Ktor 构件位于 bintray 的指定仓库中。
而其核心所依赖的 `kotlinx.coroutines` 库<!--
-->可以在 `jcenter` 上找到。

必须将两者都添加到 `build.gradle` 文件中的 `repositories` 块中。

```groovy
jcenter()
```

必须在每个 Ktor 构件的引用中指定该版本，
为避免重复，可以在 `buildscript` 块中
（或者在 `gradle.properties` 文件中）的附加属性中指定该版本以便后续使用：

```groovy
ext.ktor_version = '{{site.ktor_version}}'
```

现在必须添加 `ktor-server-core` 构件了，可引用之前指定的 `ktor_version`：

```groovy
compile "io.ktor:ktor-server-core:$ktor_version"
```

在 groovy 中，有单引号括起的字符串（而不是字符）
与双引号括起的字符串，为了能够内插像版本号这样的变量，必须使用双引号<!--
-->括起的字符串。
{: .note.tip }

需要告诉 Kotlin 编译器生成与 Java 8 兼容的字节码：
{: #java8}

```groovy
compileKotlin {
    kotlinOptions.jvmTarget = "1.8"
}
compileTestKotlin {
    kotlinOptions.jvmTarget = "1.8"
}
```

## 选择引擎并配置之
{: #engine}

Ktor 可以在很多环境中运行，例如 Netty、 Jetty 或者任何其他
Servlet 兼容的应用容器（Application Container）例如 Tomcat。

本例展示了如何配置 Ktor 使用 Netty。
对于其他引擎请参见[构件](/quickstart/artifacts.html)以查看<!--
-->可用构件的列表。

可使用之前创建的 `ktor_version` 属性添加依赖项 `ktor-server-netty`
。 这个模块提供了一个
Netty web 服务器以及运行基于其上的 Ktor
应用的所有必要代码：

```groovy
compile "io.ktor:ktor-server-netty:$ktor_version"
```

## 最终版 `build.gradle`（带 Ktor）
{: #complete}

完成后的 `build.gradle` 文件应该如下所示：

{% capture build-gradle %}
```groovy
group 'Example'
version '1.0-SNAPSHOT'

buildscript {
    ext.kotlin_version = '{{site.kotlin_version}}'
    ext.ktor_version = '{{site.ktor_version}}'

    repositories {
        mavenCentral()
    }
    dependencies {
        classpath "org.jetbrains.kotlin:kotlin-gradle-plugin:$kotlin_version"
    }
}

apply plugin: 'java'
apply plugin: 'kotlin'

sourceCompatibility = 1.8
compileKotlin {
    kotlinOptions.jvmTarget = "1.8"
}
compileTestKotlin {
    kotlinOptions.jvmTarget = "1.8"
}

kotlin {
    experimental {
        coroutines "enable"
    }
}

repositories {
    jcenter()
}

dependencies {
    compile "org.jetbrains.kotlin:kotlin-stdlib-jdk8:$kotlin_version"
    compile "io.ktor:ktor-server-netty:$ktor_version"
    testCompile group: 'junit', name: 'junit', version: '4.12'
}
```
{% endcapture %}

{% include tabbed-code.html
    tab1-title="build.gradle" tab1-content=build-gradle
%}

现在可以运行 Gradle（只需 `gradle`，或者 `./gradlew` 如果使用包装器的话）
来获取依赖并验证所有设置是否正确了。

本教程会指导你从最基本的搭建到可用于开始开发应用的全<!--
-->功能搭建。

## IntelliJ：前提条件

1.  最新版的 IntelliJ IDEA
2.  已启用 Kotlin 与 Gradle 插件（它们应该都已默认启用）

可以在 IntelliJ IDEA 中通过以下主菜单查验：
* Windows：`File -> Settings -> Plugins`
* Mac：`IntelliJ IDEA -> Settings -> Plugins`

## IntelliJ：开始一个项目

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

## IntelliJ：Gradle 设置

本节假定你有一些 Gradle 的基本知识。如果你从未使用过 Gradle，
那么可参考 gradle.org 提供的[一些指南](https://guides.gradle.org/building-java-applications/)来帮你入门。
{: .note}

可以像这样使用 Gradle 搭建一个简单的 Ktor 应用：

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

repositories {
    mavenCentral()
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

repositories {
    mavenCentral()
}

dependencies {
    compile "org.jetbrains.kotlin:kotlin-stdlib-jdk8:$kotlin_version"
    compile "io.ktor:ktor-server-netty:$ktor_version"
    compile "ch.qos.logback:logback-classic:1.2.3"
    testCompile group: 'junit', name: 'junit', version: '4.12'
}
```
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

## IntelliJ：创建应用

选择 `src/main/kotlin` 目录并创建一个新包。 我们称之为 `blog`。

选择该目录并在其下创建一个名为 `BlogApp` 的新的 kotlin 文件

![Ktor IntelliJ: Create Kotlin File](/quickstart/intellij-idea/create-kotlin-file.png)

![Ktor IntelliJ: Create Kotlin File Name](/quickstart/intellij-idea/create-kotlin-file-name.png)

复制并粘贴应用的最基本设置，使其看起来像：

{% capture blog-app %}
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
{% endcapture %}

{% include tabbed-code.html
    tab1-title="BlogApp.kt" tab1-content=blog-app
    no-height="true"
%}

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

## IntelliJ：使用 Application 对象改进应用

上述设置中有很多嵌套的块，而这不适合开始<!--
-->向应用中添加功能。 我们可以这样改进之：通过使用 Application 对象<!--
-->并在 main 函数中的 embeddedServer 调用引用该对象。

将 BlogApp.kt 中的代码改为以下这样来尝试这点：

{% capture blog-app %}
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
{% endcapture %}

{% include tabbed-code.html
    tab1-title="BlogApp.kt" tab1-content=blog-app
    no-height="true"
%}

## IntelliJ：提取配置数据

虽然可以在主函数的 embeddedServer 调用中指定一些应用配置数据，但是要为将来的部署与更改提供更多的灵活性的话，可以将其提取到一个独立的配置文件中。在 `src/main/resources` 目录中创建一个新的文本文件名为 `application.conf`，其内容如下：

{% capture application-conf %}
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
{% endcapture %}

{% include tabbed-code.html
    tab1-title="application.conf" tab1-content=application-conf
    no-height="true"
%}

然后我们在 `BlogApp.kt` 中删除主函数并将 `fun Application.module()` 改为 `fun Application.main()`。 然而，如果我们现在运行该应用，它会失败并且有像“Top-level function 'main' not found in package blog.”这样的错误信息。 我们的 `Application.main()` 函数现在是一个扩展函数而不具备顶层主函数的作用。

因为 IntelliJ IDEA 不能自动找到主类，这需要我们指定一个新的主类。在 `build.gradle` 添加：

{% capture gradle-groovy-build %}
```groovy
// build.gradle

apply plugin: 'application'

//mainClassName = 'io.ktor.server.netty.DevelopmentEngine' // 对于版本 < 1.0.0-beta-3
mainClassName = 'io.ktor.server.netty.EngineMain' // 1.0.0-beta-3 起
```
{% endcapture %}

{% capture gradle-kotlin-build %}
```kotlin
// build.gradle.kts

plugins {
    application
    // ……
}

application {
    mainClassName = "io.ktor.server.netty.EngineMain"
}
```
{% endcapture %}

{% include gradle.html gradle-kotlin=gradle-kotlin-build gradle-groovy=gradle-groovy-build %}

然后转到 `Run -> Edit Configurations` 选择 `blog.BlogAppKt` 配置并将其 Main class 修改为：
`io.ktor.server.netty.EngineMain`

现在，当我们运行新的配置时，应用程序会再次启动。

## 配置日志
{: #logging}

如果想记录应用事件及有用信息的日志，
可以在[日志](/servers/logging.html)页中进一步阅读。
