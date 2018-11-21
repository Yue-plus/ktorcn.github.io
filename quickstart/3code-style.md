---
title: 编码风格
caption: 编码风格
category: quickstart
permalink: /quickstart/code-style.html
toc: false
ktor_version_review: 1.0.0
---

## 官方编码规范

Ktor 以及其他官方 Kotlin 库都使用[官方 Kotlin 编码规范](https://www.kotlincn.net/docs/reference/coding-conventions.html)。

可以通过在 `gradle.properties` 文件中添加 `kotlin.code.style=official` 来使用官方编码标准。

## 使用星导入

官方编码规范并没有定义使用导入的推荐方式。
IntelliJ 默认在从一个包中导入至少 5 个符号后使用星（`*`）导入。但是在 Ktor 以及 JetBrains 的其他库中，我们总是使用并且建议使用星导入。

其背后的逻辑依据是，通常当你包含一个类时，你可能想要包含为该类声明的全部扩展方法与扩展属性。
这对操作符扩展方法尤其方便。

可以在 `Preferences... -> Editor -> Code Style -> Kotlin -> Imports` 中更改导入配置：

![](/quickstart/code-style/code-style-imports.png)
