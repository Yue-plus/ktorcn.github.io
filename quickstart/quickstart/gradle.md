---
title: Gradle
caption: 搭建 Gradle 构建版
category: quickstart
toc: true
permalink: /quickstart/quickstart/gradle.html
redirect_from:
  - /quickstart/gradle.html
priority: 0
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

```groovy
group 'Example'
version '1.0-SNAPSHOT'

buildscript {
    ext.kotlin_version = '{{site.kotlin_version}}'

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

sourceCompatibility = 1.8

repositories {
    jcenter()
    maven { url "https://dl.bintray.com/kotlin/kotlin-eap" }
}

dependencies {
    compile "org.jetbrains.kotlin:kotlin-stdlib-jdk8:$kotlin_version"
    testCompile group: 'junit', name: 'junit', version: '4.12'
}
```
{: .compact}

## 添加 Ktor 依赖并配置构件设置
{: #ktor-dependencies}

Ktor 构件位于 bintray 的指定仓库中。
而其核心所依赖的 `kotlinx.coroutines` 库<!--
-->可以在 `jcenter` 上找到。

必须将两者都添加到 `build.gradle` 文件中的 `repositories` 块中。

```groovy
jcenter()
maven { url "https://dl.bintray.com/kotlin/ktor" }
```

访问 [Bintray](https://bintray.com/kotlin/ktor/ktor) 并确定 ktor 的最新版本。
在本例中是 `{{site.ktor_version}}`。

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

截止到 Kotlin 1.2x，协程仍然是一项实验性的功能，
所以需要告诉编译器可以<!--
-->使用它们以避免警告：

```groovy
kotlin {
    experimental {
        coroutines "enable"
    }
}
```

还需要告诉 Kotlin 编译器生成<!--
-->与 Java 8 兼容的字节码：
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

```groovy
group 'Example'
version '1.0-SNAPSHOT'

buildscript {
    ext.kotlin_version = '{{site.kotlin_version}}'
    ext.ktor_version = '{{site.ktor_version}}'

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
    maven { url "https://dl.bintray.com/kotlin/ktor" }
    maven { url "https://dl.bintray.com/kotlin/kotlin-eap" }
}

dependencies {
    compile "org.jetbrains.kotlin:kotlin-stdlib-jdk8:$kotlin_version"
    compile "io.ktor:ktor-server-netty:$ktor_version"
    testCompile group: 'junit', name: 'junit', version: '4.12'
}
```

现在可以运行 Gradle（只需 `gradle`，或者 `./gradlew` 如果使用包装器的话）
来获取依赖并验证所有设置是否正确了。

## 配置日志
{: #logging}

如果想记录应用事件及有用信息的日志，
可以在[日志](/servers/logging.html)页中进一步阅读。
