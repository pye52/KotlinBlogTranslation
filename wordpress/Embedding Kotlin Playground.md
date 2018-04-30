# 嵌入式Kotlin环境

```kotlin
fun main(args : Array<String>) {
    println("Hello Kotliner!")
    println("Click this green button at the top right!")
}
```

你想得没错，这是一段嵌入到博客中，可以独立运行的kotlin代码。请注意，你不仅可以运行它，还可以修改其中的代码。

```kotlin
// The code below doesn't compile. Add only one char to make it runnable again
fun main(args : Array<String>) {
    val fix = "Kotlin "
    val me = "is great"
    println(fixme)
}
```

这非常酷，不是吗？也请让上面这段代码顺利地工作起来吧。

你通常并不想显示所有代码，只想展示最关键最有趣的部分，这也是可以实现的。

```kotlin
val hint = "Click the plus button to see the full code"
println (hint.length)
```

你也可以添加测试：

```kotlin
// Implement extension functions Int.r() and Pair.r()
// and make them convert Int and Pair to RationalNumber.

fun Int.r(): RationalNumber = TODO()
fun Pair<Int, Int>.r(): RationalNumber = TODO()

data class RationalNumber(val numerator: Int, val denominator: Int)
```

你也可以将JavaScript设置为目标语言，甚至还能在canvas上绘制图案

```kotlin
val state = CanvasState(canvas).apply {
    addShape(Kotlin)
    addShape(Creature(size * 0.25, this))
    addShape(Creature(size * 0.75, this))
}
window.setTimeout({ state.valid = false }, 1000)
// You can drag this objects
```

有时你并不需要一个能运行的例子。在这种情况下，`highlight-only`属性允许你在相同的样式下显示代码片段，但无法运行。

## 嵌入式Kotlin环境工作原理

一直以来，成千上万的萌新通过[try.kotlinlang.org](https://try.kotlinlang.org/)网站上的交互方式来学习这个语言。尤其是[Kotlin Koans Online](https://try.kotlinlang.org/#/Kotlin%20Koans/Introduction/Hello,%20world!/Task.kt)更加受欢迎的现在。高级用户在编写小段代码的时候用到这个开发环境，而不必打开IDE，例如在StackOverflow上粘贴代码回答问题的时候。

[Embedded Kotlin Playground](https://github.com/JetBrains/kotlin-playground)在同样的原理下工作，但是允许你在网页中编写及运行示例。编译将在我们的后端服务器上进行，然后运行在你的浏览器上（如果平台是JS）或者运行在服务器上（如果平台是JVM）。

## 前端

只需要在header中添加简单的一行代码，即可将该环境嵌入到你的页面：

```html
<script src="https://unpkg.com/kotlin-playground@1" data-selector="code"></script>
```

现在页面上所有代码块都将被转换为能运行的Kotlin代码片段。当然可以自定义`data-selector`将脚本应用到某些特定的类，也可以手动配置Kotlin运行环境。

```html
<script src="https://unpkg.com/kotlin-playground@1"></script>

<script>
document.addEventListener('DOMContentLoaded', function() {
  KotlinPlayground('.code-blocks-selector');
});
</script>
```

除此之外，还有很多其他的安装及自定义选项，可以在[文档](https://github.com/JetBrains/kotlin-playground#installation)中查看。

## 后端

嵌入式环境将在后端编译代码，并高亮显示完成的信息。通常情况下，你不需要担心后端，直接使用我们的服务器即可，除非你想引用自定义的JVM库。

当编写的示例需要引用到外部的库，例如你在为自己的库编写可交互文档时，你需要配置及运行一个开发环境的实例。这是一件很容易的事：添加依赖，运行两个预定制的gradle task，再执行`docker-compose up`，看！服务器已经跑起来了。更多的细节请查看[指引](https://github.com/JetBrains/kotlin-web-demo#how-to-add-your-dependencies-to-kotlin-compiler-books)。

## 它正应用于这些地方

- 在官网的Kotlin文档中，我们已经大规模地去使用这个技术。所有新的文档，都含有可以运行的sample（参考[Basic syntax](http://kotlinlang.org/docs/reference/basic-syntax.html)，[1.1](http://kotlinlang.org/docs/reference/whatsnew11.html)、[1.2](http://kotlinlang.org/docs/reference/whatsnew12.html)、[lambdas](http://kotlinlang.org/docs/reference/lambdas.html)、[Coroutines](http://kotlinlang.org/docs/reference/lambdas.html)的更新内容）。对于标准库的一些函数，也带有这种形象的例子。

- [Kotlin By Example](http://kotlinbyexample.org/)也是采取Kotlin嵌入环境来编写示例

- 我们也发布了一个[Wordpress的插件](https://github.com/Kotlin/kotlin-playground-wp-plugin)，增加了`kotlin`标签，允许在文章中嵌入Kotlin可交互开发环境。该博客文章中所有示例都是在该插件的帮助下编写。

  ![](![preview](https://d3nmt5vlzunoa1.cloudfront.net/kotlin/files/2018/04/preview.gif))

- 在[Kotlin论坛](https://discuss.kotlinlang.org/)里，你可以通过`run-kotlin`标签，在markdown语法下编写kotlin代码回答问题，但请保证语法正确。

![](https://d3nmt5vlzunoa1.cloudfront.net/kotlin/files/2018/04/discuss3.gif)

## 应用场景

Kotlin嵌入环境改善了阅读体验，并增加了代码示例的表现力。读者不仅能看到代码，还可以运行它，更改它，并再次执行已更改的代码。特别是在以下场合中，我们鼓励所有作者都编写可运行的Kotlin代码片段：

- 课堂上
- 幻灯片和书籍上的补充资料
- 库及框架的文档
- 博客文章中的示例

之后Kotlin嵌入环境将支持脚本编写

```kotlin
println("请尽情享受Kotlin！")
```

原文链接：https://blog.jetbrains.com/kotlin/2018/04/embedding-kotlin-playground/