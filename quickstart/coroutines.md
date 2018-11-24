---
title: 协程
category: quickstart
permalink: /quickstart/coroutines.html
caption: 协程
redirect_from:
  - /advanced/kotlinx.coroutines.html
ktor_version_review: 1.0.0
---

Ktor 大量使用 Kotlin 1.3 稳定版的协程。

协程是一种基本的 Kotlin 机制（也称为 `suspend` 函数），除了其他方面外，还让异步编程能像普通代码一样线性编写，
而不是使用传统基于回调的方法。

其他的现代语言暴露了一种类似但更具体的机制，称为 await-async。Kotlin 的方法更加通用与灵活，而且更简洁、更不易出错，
因为调用异步（`suspend`）函数时的默认行为也是挂起调用方。

Ktor 使用源自 JetBrains 的一个标准库，称为 [kotlinx.coroutines](/kotlinx/coroutines.html)。

因为 Ktor 完全异步且大量使用协程，所以最好熟悉下这些概念。
