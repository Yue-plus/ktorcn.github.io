---
title: Docker
caption: 创建 Docker 容器
category: quickstart
permalink: /quickstart/quickstart/docker.html
redirect_from:
  - /quickstart/docker.html
---

[Docker](https://www.docker.com) 是一个容器平台：
它允许将软件打包为可以在任何所支持的操作系统中隔离运行的格式。

将 Ktor 应用发布到 docker 非常简单，只需以下几步：

* 安装 [Docker](https://www.docker.com)
* JAR 打包工具

在本页中，我们会指导你创建你一个 docker 镜像并将应用发布进去。

**目录：**

* TOC
{:toc}

## 使用 Gradle 打包应用

在本教程中，我们会使用 Gradle [shadow 插件](https://github.com/johnrengelman/shadow)。
它将所有输出的类、资源以及所有必需的依赖项打包到一个 JAR 文件中，
并追加一个清单文件来告诉 Java 哪个是包含 main 方法的入口点主类。

首先，需要在 `build.gradle` 文件中添加 shadow 插件依赖：

```groovy 
buildscript {
    ...
    repositories {
        ...
        maven { url "https://plugins.gradle.org/m2/" }
    }
    dependencies {
        ...
        classpath "com.github.jengelman.gradle.plugins:shadow:2.0.1"
    }
}
```

之后，必须应用它以及 application 插件：

```groovy
apply plugin: "com.github.johnrengelman.shadow"
apply plugin: 'application'
``` 

然后指定主类，这样它才知道当在 Docker 内部运行该 JAR 包时要运行什么：

```groovy
mainClassName = 'org.sample.ApplicationKt'
```

该字符串是包含 `main` 函数的类的完整限定名。 当 `main` 函数是<!--
-->文件中的顶层函数时，类名是带有 `Kt` 后缀的文件名。 在上述示例中，`main` 函数在
`org.sample` 包中的 `Application.kt` 文件中。

最后，必须配置 shadow 插件：

```groovy
shadowJar {
    baseName = 'my-application'
    classifier = null
    version = null
}
```

现在可以运行 `./gradlew build` 来构建并打包应用了。
应该可以在 `build/libs` 文件夹中获取 `my-application.jar`。

关于配置这个插件的更多信息，请参见[该插件的文档](http://imperceptiblethoughts.com/shadow/)

因此一个完整的 `build.gradle` 文件会如下所示：


{% capture build-gradle %}
```groovy
buildscript {
    ext.kotlin_version = '{{site.kotlin_version}}'
    ext.ktor_version = '{{site.ktor_version}}'
    ext.logback_version = '1.2.3'
    ext.slf4j_version = '1.7.25'
    repositories {
        jcenter()
        maven { url "https://plugins.gradle.org/m2/" }
    }
    dependencies {
        classpath "org.jetbrains.kotlin:kotlin-gradle-plugin:$kotlin_version"
        classpath "com.github.jengelman.gradle.plugins:shadow:2.0.1"
    }
}

apply plugin: 'kotlin'
apply plugin: "com.github.johnrengelman.shadow"
apply plugin: 'application'

mainClassName = "io.ktor.server.netty.EngineMain"

sourceSets {
    main.kotlin.srcDirs = [ 'src' ]
    main.resources.srcDirs = [ 'resources' ]
}

repositories {
    jcenter()
}

dependencies {
    compile "org.jetbrains.kotlin:kotlin-stdlib-jdk8:$kotlin_version"
    compile "io.ktor:ktor-server-netty:$ktor_version"
    compile "io.ktor:ktor-html-builder:$ktor_version"
    compile "ch.qos.logback:logback-classic:$logback_version"
}

kotlin.experimental.coroutines = 'enable'

shadowJar {
    baseName = 'my-application'
    classifier = null
    version = null
}
```
{% endcapture %}

{% capture resources-application-conf %}
```groovy
ktor {
    deployment {
        port = 8080
    }

    application {
        modules = [ io.ktor.samples.hello.HelloApplicationKt.main ]
    }
}
```
{% endcapture %}

{% capture src-hello-application-kt %}
```kotlin
package io.ktor.samples.hello

import io.ktor.application.*
import io.ktor.features.*
import io.ktor.html.*
import io.ktor.routing.*
import kotlinx.html.*

fun Application.main() {
    install(DefaultHeaders)
    install(CallLogging)
    routing {
        get("/") {
            call.respondHtml {
                head {
                    title { +"Ktor: jetty" }
                }
                body {
                    p {
                        +"Hello from Ktor Jetty engine sample application"
                    }
                }
            }
        }
    }
}
```
{% endcapture %}

{% include tabbed-code.html
    tab1-title="build.gradle" tab1-content=build-gradle
    tab2-title="resources/application.conf" tab2-content=resources-application-conf
    tab3-title="src/HelloApplication.kt" tab3-content=src-hello-application-kt
%}


可以在 ktor-samples 仓库中检出这个[完整示例](https://github.com/ktorio/ktor-samples/tree/master/deployment/docker)。
{: .note }

## 准备 Docker 镜像

在项目的根文件夹中创建一个名为 `Dockerfile` 的文件，其内容如下：

{% capture dockerfile %}{% include docker-sample.md %}{% endcapture %}
{% include tabbed-code.html
    tab1-title="Dockerfile" tab1-content=dockerfile
    no-height="true"
%}

我们来看下都是什么：

```dockerfile
FROM openjdk:8-jre-alpine
```

这行告诉 Docker 使用以 [Alpine Linux](https://alpinelinux.org/) 预构建的镜像作为基础镜像。也可以使用
[OpenJDK registry](https://hub.docker.com/_/openjdk/) 中的其他镜像。Alpine Linux 的好处是镜像非常小。
我们选择的也是只有 JRE 的镜像，因为我们并不需要在镜像中编译代码，只需要运行预编译的类。

```dockerfile
RUN mkdir /app
COPY ./build/libs/my-application.jar /app/my-application.jar
WORKDIR /app
```

这几行将已打包的应用复制到 Docker 镜像中，并将工作目录设置为复制后的位置。

```dockerfile
CMD ["java", "-server", "-XX:+UnlockExperimentalVMOptions", "-XX:+UseCGroupMemoryLimitForHeap", "-XX:InitialRAMFraction=2", "-XX:MinRAMFraction=2", "-XX:MaxRAMFraction=2", "-XX:+UseG1GC", "-XX:MaxGCPauseMillis=100", "-XX:+UseStringDeduplication", "-jar", "my-application.jar"]
```

最后一行指示 Docker 使用 G1 GC、4G 内存以及已打包的应用来运行 `java`。

## 构建并运行 Docker 镜像

构建应用包：

```bash
./gradlew build
```

构建并标记镜像：

```bash
docker build -t my-application .
```

启动镜像：

```bash
docker run -m512M --cpus 2 -it -p 8080:8080 --rm ktor-docker-sample-application
```

通过这个命令，我们启动 Docker 进入前台模式。 它会等待服务器退出，
也会响应 `Ctrl+C` 来停止服务。 `-it` 指示 Docker 分配一个终端（*tty*）来管理标准输出（stdout）
并响应中断键序列。

由于我们的服务器现在是在一个隔离的容器中运行，因此我们应该告诉 Docker 暴露一个端口，以便我们可以<!--
-->实际访问到服务器端口。 参数 `-p 8080:8080` 告诉 Docker 将容器内部的 8080 端口发布为本机的
8080 端口。 因此，当告诉浏览器访问 `localhost:8080` 时，它会首先连向 Docker，然后为应用将其桥接<!--
-->到内部端口 `8080`。

可以通过 `-m512M` 调整内存并通过  `--cpus 2` 调整所暴露 cpu 数。

默认情况下，容器的文件系统在容器退出后仍然存在，所以我们提供 `--rm` 选项以防<!--
-->垃圾堆积。

关于运行 docker 镜像的更多信息，请参考 [docker run](https://docs.docker.com/engine/reference/run)
文档。

## 上传 docker 镜像

一旦应用能在本地成功运行，那么可能就到部署它的时候了：

```bash
docker tag my-application hub.example.com/docker/registry/tag
docker push hub.example.com/docker/registry/tag
```
 
这两条命令会为 registry 创建应用的标记并将镜像上传。
当然，需要用你的 registry 的实际 URL 来替换 `hub.example.com/docker/registry/tag`。

我们这里不会详细展开，因为你的配置可能需要身份认证、特殊配置<!--
-->甚至特殊工具。请咨询你的组织或者云平台，或者<!--
-->查阅 [docker push](https://docs.docker.com/engine/reference/commandline/push/) 文档。

## 样例

可以在 ktor-samples 仓库中检出[完整样例](https://github.com/ktorio/ktor-samples/tree/master/deployment/docker)。
