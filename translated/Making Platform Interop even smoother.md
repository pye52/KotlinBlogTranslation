---
title: "[译]Making Platform Interop even smoother"
date: 2014-10-06 13:20:00
author: Hadi Hariri
tags:
keywords:
categories: 官方动态
reward: false
reward_title: Have a nice Kotlin!
reward_wechat:
reward_alipay:
source_url: https://blog.jetbrains.com/kotlin/2014/10/making-platform-interop-even-smoother/
translator:
translator_url:
---

与 JVM 100％可互操作，随后使用 JavaScript，一直是 Kotlin 的首要任务。随着现有代码的数量和丰富的 JVM 生态系统，具有互操作性和有效性的功能至关重要。<span id =“more-1616”> </span>
随着即将发布的 M9，我们已经改进了这一点，使集成更加无缝。
## 平台类型

处理空值是涉及 Java 互操作性的最大问题之一。几乎任何 Java 方法可能会返回 null，所以 Kotlin 将 Java 类型视为可以为空的，我们还是要求使用安全调用（？。）运算符或 notnull-assertion（!!）来避免编译错误：

{% raw %}
<p></p>
{% endraw %}

```kotlin
val x = javaCanReturnNull() // Java call that can return null
```

{% raw %}
<p></p>
{% endraw %}

尝试将值*x*传递给以下函数：

{% raw %}
<p></p>
{% endraw %}

```kotlin
fun nullNotAllowed(value: String) {}
fun nullAllowed(value: String?) {}
```

{% raw %}
<p></p>
{% endraw %}

在第一种情况下，Kotlin 编译器会发出错误。这意味着对*nullNotAllowed*的调用将需要：

{% raw %}
<p></p>
{% endraw %}

```kotlin
val x = javaCanReturnNull()
nullNotAllowed(x!!)
```

{% raw %}
<p></p>
{% endraw %}

从 M9 开始，情况就不一样了。这允许更清洁的代码，并避免过度使用？和!!操作员在与 Java 进行交互时。
同样的方式，在实现 Java 接口时，可能具有潜在的空参数的方法不再需要在 Kotlin 中将这些声明声明为可空。例如，给出：

{% raw %}
<p></p>
{% endraw %}

```kotlin
public interface SomeFoo {
    void foo(String input, String data);
}
```

{% raw %}
<p></p>
{% endraw %}

当在 Kotlin 实施这个时：

{% raw %}
<p></p>
{% endraw %}

```kotlin
public class SomeInterestingFoo(): SomeFoo {
    override fun foo(input: String, data: String?) { // String? no longer required
    ...
    }
}
```

{% raw %}
<p></p>
{% endraw %}

参数*输入*不再需要类型为*String？*。您可以选择使用*String*还是*String？*  - 取决于其实际含义。在这个例子中，我们选择*输入*不为空，并且*数据*为空。
## 使用 platformStatic 的注释方法

Kotlin 有*对象声明*，可以看作单身人士。这些都是从 Java 中消耗的，尽管不是最好的语法。给定：

{% raw %}
<p></p>
{% endraw %}

```kotlin
object Foo {
    fun bar() {
    }
}
```

{% raw %}
<p></p>
{% endraw %}

从 Kotlin 消费，将是：

{% raw %}
<p></p>
{% endraw %}

```kotlin
Foo.bar()
```

{% raw %}
<p></p>
{% endraw %}

而从 Java 它将看起来像这样：

{% raw %}
<p></p>
{% endraw %}

```kotlin
Foo.INSTANCE$.bar()
```

{% raw %}
<p></p>
{% endraw %}

随着下一个版本，我们可以通过简单地向函数声明添加一个注释，从与 Kotlin 相同的方式从 Java 调用该方法：

{% raw %}
<p></p>
{% endraw %}

```kotlin
object Foo {
   platformStatic bar() {
   }
}
```

{% raw %}
<p></p>
{% endraw %}

使代码更清洁。同样适用于*类对象*。
### 去除包版广告

*platformStatic*的另一个好处是在使用某些 Java 库（例如 JUnit）时删除了一些存在的 showstoppers。特别地，后者在使用时需要 Java 中的静态方法 [理论](https://github.com/junit-team/junit/wiki/Theories) 。对此的解决方法相当乏味。幸运的是，情况已经不再如此。我们可以使用*platformStatic*注释来解决这个问题 [问题](https://youtrack.jetbrains.com/issue/KT-4861) 。

{% raw %}
<p></p>
{% endraw %}

```kotlin
RunWith(javaClass<Theories>())
public class TestDoubling {
       class object {
           platformStatic DataPoints
               public fun values(): IntArray {
                    return intArray(-1, 0, 1, 2, 3, 4, 5, 6, 7, 8, 9)
               }
         }

         Theory
         public fun testDoubling(a: Int?) {
                  Assert.assertThat<Int>(doubling(a!!), `is`<Int>(a * 2))
         }

         public fun doubling(value: Int): Int {
                  return value * 2
         }
}
```

{% raw %}
<p></p>
{% endraw %}

## 利用 platformName 重载函数

当有重载的方法使用泛型，如：

{% raw %}
<p></p>
{% endraw %}

```kotlin
fun Iterable<Long>.average(): Double {
...
}
 
fun Iterable<Int>.average(): Double {
...
}
```

{% raw %}
<p></p>
{% endraw %}

当从 Kotlin 调用这些可能时，尝试从 Java 调用它们是由于类型擦除而有问题的。类似于*platformStatic*，我们引入了允许重命名该函数的*platformName*注释，以便在从 Java 调用时，使用新的给定名称：

{% raw %}
<p></p>
{% endraw %}

```kotlin
fun Iterable<Long>.average(): Long {
...
}
 
platformName("averageOfInt") fun Iterable<Int>.average(): Int {
...
}
```

{% raw %}
<p></p>
{% endraw %}

现在可以从 Java 调用如下：

{% raw %}
<p></p>
{% endraw %}

```kotlin
averageOfInt(numbers)
```

{% raw %}
<p></p>
{% endraw %}

请注意，这是 [不是唯一的用例](http://blog.jetbrains.com/kotlin/2014/07/m8-is-out/#platformName) for *platformName*。
## 私人财产存取者

Kotlin 的私人私人财产不再生成属性访问器，这意味着 [冲突](http://blog.jetbrains.com/kotlin/2014/07/m8-is-out/#platformName) 使用现有的*getXYZ*方法
不必要地发生。在 Java 中执行以下界面：

{% raw %}
<p></p>
{% endraw %}

```kotlin
public interface SomeFoo {
    String getBar();
}
```

{% raw %}
<p></p>
{% endraw %}

如果我们要在 Kotlin 中实现这个接口，而我们的类有一个名为*bar*的私有属性，在 M8 和以前的版本中它会引起冲突，我们必须将属性命名为与< em> bar*。从 M9 开始，情况就不一样了。因此，以下代码完全有效：

{% raw %}
<p></p>
{% endraw %}

```kotlin
public class SomeInterestingFoo(private val bar: String): SomeFoo {
    override fun getBar(): String {
          ...
    }
}
```

{% raw %}
<p></p>
{% endraw %}

## 概要

随着 M9 中的这些变化，我们已经删除了一些障碍，更重要的是使 Java 和 Kotlin 之间的互操作性更加清晰。虽然这些改进可以使消费现有的 Java 代码更加愉快，但它也允许在 Kotlin 中编写新的库和功能以及从 Java 中消耗这些库的更好的体验。
和往常一样，我们会喜欢反馈。让我们知道你的想法。
注意：M9 尚未发布，但您可以在<a href="https://teamcity.jetbrains.com/project.html?projectId=Kotlin&amp;tab=projectOverview">夜间版本中找到这些功能</a>*
