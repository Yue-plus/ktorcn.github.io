---
title: Maven
caption: 搭建 Maven 构建版
category: quickstart
toc: true
permalink: /quickstart/quickstart/maven.html
redirect_from:
  - /quickstart/maven.html
---

在本指南中，我们会展示如何创建  Maven `pom.xml` 文件<!--
-->以及如何配置以支持 Ktor。

**目录：**

* TOC
{:toc}

## 基本的 Kotlin `pom.xml`文件（不带 Ktor）
{: #initial }

Maven 是一个主要用于 Java 项目的构建自动化工具。
它从 `pom.xml` 文件中读取项目配置。
以下是用于构建 Kotlin 应用的基本`pom.xml`文件：

{% capture pom-xml %}
```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/maven-v4_0_0.xsd">

    <modelVersion>4.0.0</modelVersion>

    <groupId>org.jetbrains</groupId>
    <artifactId>sample</artifactId>
    <version>1.0-SNAPSHOT</version>
    <packaging>jar</packaging>

    <name>org.jetbrains sample</name>

    <properties>
        <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
        <kotlin.version>{{site.kotlin_version}}</kotlin.version>
        <junit.version>4.12</junit.version>
    </properties>

    <dependencies>
        <dependency>
            <groupId>org.jetbrains.kotlin</groupId>
            <artifactId>kotlin-stdlib</artifactId>
            <version>${kotlin.version}</version>
        </dependency>
        <dependency>
            <groupId>org.jetbrains.kotlin</groupId>
            <artifactId>kotlin-test-junit</artifactId>
            <version>${kotlin.version}</version>
            <scope>test</scope>
        </dependency>
        <dependency>
            <groupId>junit</groupId>
            <artifactId>junit</artifactId>
            <version>${junit.version}</version>
            <scope>test</scope>
        </dependency>
    </dependencies>
    
    <build>
        <sourceDirectory>src/main/kotlin</sourceDirectory>
        <testSourceDirectory>src/test/kotlin</testSourceDirectory>

        <plugins>
            <plugin>
                <groupId>org.jetbrains.kotlin</groupId>
                <artifactId>kotlin-maven-plugin</artifactId>
                <version>${kotlin.version}</version>
                <executions>
                    <execution>
                        <id>compile</id>
                        <phase>compile</phase>
                        <goals>
                            <goal>compile</goal>
                        </goals>
                    </execution>
                    <execution>
                        <id>test-compile</id>
                        <phase>test-compile</phase>
                        <goals>
                            <goal>test-compile</goal>
                        </goals>
                    </execution>
                </executions>
            </plugin>
        </plugins>
    </build>
</project>
```
{% endcapture %}

{% include tabbed-code.html
    tab1-title="pom.xml" tab1-content=pom-xml
%}

## 添加 Ktor 依赖并配置构件设置
{: #ktor-dependencies}

Ktor 构件位于 bintray 的指定仓库中。
而其核心所依赖的 `kotlinx.coroutines` 库<!--
-->可以在 `jcenter` 上找到。

必须将两者都添加到 `pom.xml` 文件中的 `repositories` 块中。
   
```xml
<repositories>
    <repository>
        <id>jcenter</id>
        <url>http://jcenter.bintray.com</url>
    </repository>
</repositories>
``` 

访问 [Bintray](https://bintray.com/kotlin/ktor/ktor) 并确定 ktor 的最新版本。
在本例中是 `{{site.ktor_version}}`。

必须在每个 Ktor 构件的引用中指定该版本，
为避免重复，可以在 `properties` 块中
的附加属性中指定该版本以便后续使用：

```xml
<properties>
    <ktor.version>{{site.ktor_version}}</ktor.version>
</properties>
```

现在必须添加 `ktor-server-core` 构件了，可使用之前指定的 `ktor_version`：
 
```xml
<dependency>
    <groupId>io.ktor</groupId>
    <artifactId>ktor-server-core</artifactId>
    <version>${ktor.version}</version>
</dependency>
```

## 选择引擎并配置之
{: #engine}

Ktor 可以在很多环境中运行，例如 Netty、 Jetty 或者任何其他
Servlet 兼容的应用容器（Application Container）例如 Tomcat。

本例展示了如何配置 Ktor 使用 Netty。
对于其他引擎请参见[构件](/quickstart/artifacts.html)以查看<!--
-->可用构件的列表。

可使用之前创建的 `ktor.version` 属性添加依赖项 `ktor-server-netty`
。 这个模块提供了
Netty 作为 web 服务器以及运行基于其上的 Ktor
应用的所有必要代码：

```xml
<dependency>
    <groupId>io.ktor</groupId>
    <artifactId>ktor-server-netty</artifactId>
    <version>${ktor.version}</version>
</dependency>
```

## 最终版 `pom.xml`（带 Ktor）
{: #complete}

完成后的 `pom.xml` 文件应该如下所示：

{% capture pom-xml %}
```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/maven-v4_0_0.xsd">

    <modelVersion>4.0.0</modelVersion>

    <groupId>org.jetbrains</groupId>
    <artifactId>sample</artifactId>
    <version>1.0-SNAPSHOT</version>
    <packaging>jar</packaging>

    <name>org.jetbrains sample</name>

    <properties>
        <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
        <kotlin.version>{{site.kotlin_version}}</kotlin.version>
        <ktor.version>{{site.ktor_version}}</ktor.version>
        <junit.version>4.12</junit.version>
    </properties>

    <dependencies>
        <dependency>
            <groupId>org.jetbrains.kotlin</groupId>
            <artifactId>kotlin-stdlib</artifactId>
            <version>${kotlin.version}</version>
        </dependency>
        <dependency>
            <groupId>org.jetbrains.kotlin</groupId>
            <artifactId>kotlin-test-junit</artifactId>
            <version>${kotlin.version}</version>
            <scope>test</scope>
        </dependency>
        <dependency>
            <groupId>junit</groupId>
            <artifactId>junit</artifactId>
            <version>${junit.version}</version>
            <scope>test</scope>
        </dependency>

        <dependency>
            <groupId>io.ktor</groupId>
            <artifactId>ktor-server-netty</artifactId>
            <version>${ktor.version}</version>
        </dependency>

    </dependencies>

    <build>
        <sourceDirectory>src/main/kotlin</sourceDirectory>
        <testSourceDirectory>src/test/kotlin</testSourceDirectory>

        <plugins>
            <plugin>
                <groupId>org.jetbrains.kotlin</groupId>
                <artifactId>kotlin-maven-plugin</artifactId>
                <version>${kotlin.version}</version>
                <executions>
                    <execution>
                        <id>compile</id>
                        <phase>compile</phase>
                        <goals>
                            <goal>compile</goal>
                        </goals>
                    </execution>
                    <execution>
                        <id>test-compile</id>
                        <phase>test-compile</phase>
                        <goals>
                            <goal>test-compile</goal>
                        </goals>
                    </execution>
                </executions>
                <configuration>
                    <jvmTarget>1.8</jvmTarget>
                    <args>
                        <arg>-Xcoroutines=enable</arg>
                    </args>
                </configuration>
            </plugin>
        </plugins>
    </build>

    <repositories>
        <repository>
            <id>jcenter</id>
            <url>http://jcenter.bintray.com</url>
        </repository>
    </repositories>
</project>
```
{% endcapture %}

{% include tabbed-code.html
    tab1-title="pom.xml" tab1-content=pom-xml
%}

现在可以运行 `mvn package`来获取依赖并验证所有设置是否正确了。

## 配置日志
{: #logging}

如果想记录应用事件及有用信息的日志，
可以在[日志](/servers/logging.html)页中进一步阅读。


