---
title: "[译]Kotlin 1.1-M02 is here!"
date: 2016-10-20 14:04:00
author: Denis Zharkov
tags:
keywords:
categories: 官方动态
reward: false
reward_title: Have a nice Kotlin!
reward_wechat:
reward_alipay:
source_url: https://blog.jetbrains.com/kotlin/2016/10/kotlin-1-1-m02-is-here/
translator:
translator_url:
---

我们很高兴地宣布 Kotlin 1.1 的第二个里程碑版本。这个版本带来了一个期待已久的新语言功能，在 lambdas**中进行破解，以及 1.1-M1 中引入的功能的许多改进，包括类型别名，协程和绑定引用。新版本还包括 Kotlin 1.0.4 和 1.0.5-eap-66 中引入的所有工具功能，并且与 IntelliJ IDEA 2016.3 EAP 和 Android Studio 2.2 完全兼容。
与 Kotlin 1.1-M01 一样，我们为新的语言和库功能提供**无后向兼容性保证**。在 1.1 版本的里程碑版本中引入的内容在最终 1.1 版本之前是**可能会更改**。
再次：请分享您关于新语言功能或您可能遇到的任何问题的反馈，通过此版本 [YouTrack](https://youtrack.jetbrains.com/issues/KT) ， [论坛](http://discuss.kotlinlang.org) 和 [松弛](https://kotlinlang.slack.com) 。
1.1-M02 的完整更新日期可用 [这里](https://github.com/JetBrains/kotlin/blob/1.1-M2/ChangeLog.md) 。

{% raw %}
<p><span id="more-4312"></span></p>
{% endraw %}

## 兰布达的破坏

Kotlin 1.0 支持 [解构声明](https://kotlinlang.org/docs/reference/multi-declarations.html) - 一个允许您“解包”复合值（如数据类）并将其组件分配给多个不同变量的功能。 Kotlin 1.1 将其扩展为**lambda 参数**，让您解压缩传递给 lambda 的复合变量，并以不同的名称访问其组件。例如，您可以使用它来遍历对列表：

{% raw %}
<p></p>
{% endraw %}

```kotlin
listOfPairs.map {
    (a, b) -> a + b
}
```

{% raw %}
<p></p>
{% endraw %}

你可以在里面找到更多的细节 [保持](https://github.com/Kotlin/KEEP/blob/master/proposals/destructuring-in-parameters.md) 。请注意，该功能目前仅支持 JVM 后端。嵌套解析以及传递给常规函数和构造函数的参数的解构目前不受支持。
## 标准库

Kotlin 1.1-M02 为标准库添加了几个新的 API：

* 不同的 AbstractCollection 和 AbstractMutableCollection 层次结构用作实现新 Kotlin 集合类（KEEP-53）的基类;
* 用于复制地图的 Map.toMap（）和 Map.toMutableMap（）扩展功能（KEEP-13）

## 反射

反思图书馆已经获得了大量新功能。您现在可以从 KType 实例获取更多有用的信息，并创建自定义 KType 实例，声明上的内省修饰符，获取超类或检查子类型等。要查看新功能，可以扫描 [以下提交](https://github.com/JetBrains/kotlin/commit/ed1490dbc43f88696f82e5307df43269ecbb32b1) 。
## IDE

IntelliJ IDEA 插件已经扩展到支持新的 1.1 语言功能，新的重构“引入类型别名”和“内联类型别名”，一种意图操作，用于从使用中创建一个类型别名，以及在 lambdas 中应用解构的 quickfix 自动。
## 脚本

从这个版本开始，Kotlin 支持 JSR-223（`javax.script` API），允许您从应用程序轻松运行 Kotlin 脚本，并使用 Kotlin 作为嵌入式脚本语言。它还继续支持在 Gradle 构建文件中支持 Kotlin 脚本的工作。
## JavaScript

1.1-M02 中的 JavaScript 支持已经扩展到支持**类型别名**和**类文字**（`Foo :: class`）。
除此之外，我们正在努力在多平台项目中提供更多的 Kotlin API。为此，我们在`kotlin`包中定义了所有标准异常类。当定位到 JVM 时，Kotlin 异常被定义为相应 Java 异常的类型别名，JS 后端提供了其完整的实现。我们还为标准集合类提供了一个完整的 Kotlin 实现，现在用于 JS 项目。 （Kotlin 在 JVM 上仍然使用标准的 Java 集合类。）
## 如何尝试

**在 Maven / Gradle 中：**将`http://dl.bintray.com/kotlin/kotlin-eap-1.1`添加为构建脚本和项目的存储库;使用 1.1-M02 作为编译器和标准库的版本号。
**在 IntelliJ IDEA 中：**转到*工具→Kotlin→配置 Kotlin 插件更新*，然后在*更新频道*下拉菜单中选择“Early Access Preview 1.1”下拉列表，然后按*检查更新*。
**<a href="http://try.kotlinlang.org/"> try.kotlinlang.org </a>**。使用右下角的下拉列表将编译器版本更改为 1.1-M02。
**使用 SDKMan**。运行`sdk install kotlin 1.1-M02`。
如果你正在使用 [kotlinx.coroutines](https://github.com/Kotlin/kotlinx.coroutines) 图书馆请使用更新版本的 0.1-alpha-2，它几乎相同，但是它与 1.1-M02 编译器重新编译。你可以跟随更新 [自述文件](https://github.com/Kotlin/kotlinx.coroutines/blob/master/README.md) 。
有一个漂亮的 Kotlin！
