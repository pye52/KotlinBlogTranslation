# DSL In Action

> 伴随着Kotlin的发展，有一个神奇的框架`anko-layout`，一直存在于我们的视野却又一直因为各种原因无法用于生产环境中。最近在写项目时，再次拿出anko这个框架，思考它在UI小组件上的可用性。

> PS: Anko != Anko_Layouts ，但是为了表述方便，文中一部分Anko是代指这Anko Layouts框架，大家自己理解一下~

## 概述

关于`Anko-Layouts`框架的好处和局限性，网上已经有大部分文章在讲，它好在用DSL的方式来描述View，而缺点在于无法即时预览，在这方面导致Anko DSL的开发效率不及XML传统方式。经过大家的一些踩坑，以及开发上的试用，一致表示，Anko Layouts无法用在成熟的项目之中，还是老老实实用XML吧…

Anko Layouts的DSL设计那么棒… **就要这么放弃了吗**

## 大家眼里的Anko Layouts DSL

受官方文档的“**诱导**”，大家对于Anko Layouts DSL的印象大概是这样子的：

```kotlin
override fun onCreate(savedInstanceState: Bundle?) {
    super.onCreate(savedInstanceState)
    
    verticalLayout {
        padding = dip(30)
        editText {
            hint = "Name"
            textSize = 24f
        }
        editText {
            hint = "Password"
            textSize = 24f
        }
        button("Login") {
            textSize = 26f
        }
    }
}

val name: EditText = with(ankoContext) {
    editText {
        hint = "Name"
    }
}
```

官方的Demo中，将Activity的布局方式从`setContentView()`中传入Layout ID换到了直接的DSL，嗯… 看起来还不错，官方文档也提供了一个Anko View 组件化的方案：

```kotlin
class MyActivity : AppCompatActivity() {
    override fun onCreate(savedInstanceState: Bundle?, persistentState: PersistableBundle?) {
        super.onCreate(savedInstanceState, persistentState)
        MyActivityUI().setContentView(this)
    }
}

class MyActivityUI : AnkoComponent<MyActivity> {
    override fun createView(ui: AnkoContext<MyActivity>) = with(ui) {
        verticalLayout {
            val name = editText()
            button("Say Hello") {
                onClick { ctx.toast("Hello, ${name.text}!") }
            }
        }
    }
}
```

直戳XML的痛点，XML作为传统的View构建方式，复用的方式极其有限（比如说蛋疼的`include`)，而Anko可以在编程语言的层面来做View组件的复用，实在是棒… 

但是它背后做了啥？这些View是怎么被构造的？这些View是怎么被添加进去的？如果是复杂的参数又应该怎么办？
这些问题在你计划把Anko Layouts DSL 作为构建View的方式后，逐个浮出水面，然后开始劝退… QAQ

## Anko Layout DSL 到底在干什么

> 为什么我们可以用DSL来写界面？

Kotlin DSL本身就是语法糖而已，所以DSL背后就是使用Kotlin代码来自己初始化View，初始化LayoutParams，进行addView之类… 

而其实LayoutInflater它本身也只是在做相似的事情而已，LayoutInflater是根据XML文件里面的配置来通过反射初始化View，根据其他字段来填充View属性以及LayoutParams什么的。所以没有什么神秘的东西… 

> 我们梳理一下，其实在非XML代码中构建View的时候，无非就是 `new View(context) -> addView`

那么我们瞅瞅Anko的代码，是不是也有相似的逻辑（不用想也是啊）

```kotlin
inline fun ViewManager.textView(init: (@AnkoViewDslMarker android.widget.TextView).() -> Unit): android.widget.TextView {
    return ankoView(`$$Anko$Factories$Sdk25View`.TEXT_VIEW, theme = 0) { init() }
}

inline fun <T : View> ViewManager.ankoView(factory: (ctx: Context) -> T, theme: Int, init: T.() -> Unit): T {
    val ctx = AnkoInternals.wrapContextIfNeeded(AnkoInternals.getContext(this), theme)
    val view = factory(ctx)
    view.init()
    AnkoInternals.addView(this, view) // this.addView(view)
    return view
}

//自定义的View添加DSL支持的话 (ColorCircleView是我的一个自定义View)
//这里的代码比ViewManager.textView更容易理解
inline fun ViewManager.colorCircleView() = colorCircleView {}

inline fun ViewManager.colorCircleView(init: ColorCircleView.() -> Unit): ColorCircleView {
    return ankoView({ ColorCircleView(it) }, theme = 0, init = init)
}
```

我们可以大概看到，在AnkoView中构造了一个View然后通过ViewManager添加到ViewGroup里面去。
那么，ViewManager是什么呢？

```java
package android.view;

/** Interface to let you add and remove child views to an Activity. To get an instance
  * of this class, call {@link android.content.Context#getSystemService(java.lang.String) Context.getSystemService()}.
  */
public interface ViewManager
{
    public void addView(View view, ViewGroup.LayoutParams params);
    public void updateViewLayout(View view, ViewGroup.LayoutParams params);
    public void removeView(View view);
}
```

所以ViewManager是管理着View的添加，修改以及删除的接口，不出意料，ViewGroup就实现了ViewManager
```java
public abstract class ViewGroup extends View implements ViewParent, ViewManager {
```

然后我们梳理一下，`textView`是一个拓展方法，拓展到了ViewManager接口里面，因此所有实现ViewManager接口的类都可以调用这个`textView`方法，而调用这个方法的结果就是把`textView`加入到此ViewGroup里面，比如说：

```kotlin
val frameLayout = findViewById<FrameLayout>(R.id.fl_container)

val view = frameLayout.textView {
    text = "辣鸡办公网😭🌚🌚🌚🌚🌚🌚"
    textColor = Color.BLACK
    textSize = 16f
}.apply {
    layoutParams = FrameLayout.LayoutParams(matchParent, wrapContent)
    visibility = View.GONE
}
```

效果就是，在FrameLayout里面添加了一个TextView，Textview拥有着DSL闭包里面的配置。

另外，我们构造View的方式还有，传入一个Context就可以构建出一个View，我们可以瞅瞅相关的代码：

```kotlin
inline fun Context.constraintLayout(): android.support.constraint.ConstraintLayout = constraintLayout() {}

inline fun Context.constraintLayout(init: (@AnkoViewDslMarker _ConstraintLayout).() -> Unit): android.support.constraint.ConstraintLayout {
    return ankoView(`$$Anko$Factories$ConstraintLayoutViewGroup`.CONSTRAINT_LAYOUT, theme = 0) { init() }
}
```

背后的实现我们不做深究，大概就是用Context来构建出一个View，然后拿到了View，我们就可以为所欲为了。

## 怎么把Anko灵活用起来

**简单回顾一下上面一节的内容**： 如果我们拥有一个ViewGroup或者拥有一个Context，就可以用来创建View

因此Anko的用法远要比你想象中的灵活 -> 可以拿到Context/ViewGroup的地方就可以使用Anko，而Anko的作用也就是简化初始化View + AddView的流程。

![举个栗子？](http://img3.tupian1.com/uploads/allimg/170122/1-1F122232127.jpg)

比如说我已经用XML写好了页面的布局，然后我们需要根据代码在其中一个FrameLayout中动态添加一些东西。我们就可以拿到这个FrameLayout的引用，然后就可以用anko大展拳脚了。

```kotlin
val frameLayout = findViewById<FrameLayout>(R.id.fl_container)
val view = frameLayout.textView {
    text = "辣鸡办公网😭🌚🌚🌚🌚🌚🌚"
    textColor = Color.BLACK
    textSize = 16f
}.apply {
    layoutParams = FrameLayout.LayoutParams(matchParent, wrapContent)
    visibility = View.GONE
}

frameLayout.verticalLayout { 
    
}
```

摸着良心说，是不是比自己创建View(不管是从Inflater还是java code方式)都要简单太多。

再举一个例子，在BottomSheetDialogFragment中，我们拿到Dialog后，需要通过setContView的方式来给它设置有个View进去，而我们一般会在XML写好然后Inflater获得View加载进去，或者自己一个一个new。有了Anko后，你可以随手写起DSL。

```kotlin
    override fun setupDialog(dialog: Dialog?, style: Int) {
        if (dialog == null) return
        val context = dialog.context
        
        val view = context.nestedScrollView {
            verticalLayout {
                constraintLayout {
                    backgroundColor = getColorCompat(R.color.colorPrimary)

                    val titleText = textView {
                        text = "课程表设置"
                        id = View.generateViewId()
                        textSize = 20f
                        textColor = Color.WHITE
                    }.lparams(width = wrapContent, height = wrapContent) {
                        startToStart = ConstraintLayout.LayoutParams.PARENT_ID
                        topToTop = ConstraintLayout.LayoutParams.PARENT_ID
                        bottomToBottom = ConstraintLayout.LayoutParams.PARENT_ID
                        margin = dip(16)
                    }

                }

                indicator("课程表界面设置")

                constraintLayout {
                    backgroundColor = Color.WHITE

                    textView {
                        text = "自动隐藏周六日"
                        textSize = 14f
                        textColor = Color.BLACK
                    }.lparams(width = wrapContent, height = wrapContent) {
                        startToStart = PARENT_ID
                        topToTop = PARENT_ID
                        bottomToBottom = PARENT_ID
                        leftMargin = dip(16)
                    }

                    switch {
                        isChecked = SchedulePref.autoCollapseSchedule
                        onCheckedChange { _, isChecked ->
                            SchedulePref.autoCollapseSchedule = isChecked
                        }
                    }.lparams {
                        topToTop = PARENT_ID
                        bottomToBottom = PARENT_ID
                        endToEnd = PARENT_ID
                        rightMargin = dip(16)
                    }
                }.lparams(width = matchParent, height = dip(48))

                indicator("主题设置(课程表试点)")

            }
        }

        dialog.setContentView(view)
    }

```

你甚至可以像函数一样去封装，给LinearLayout做拓展后，就可以包装添加固定风格TextView的操作了（这个封装是不是就很好写 就贼tm方便）

```kotlin
fun _LinearLayout.indicator(indicatorText: String) = frameLayout {
        textView {
            text = indicatorText
            textColor = getColorCompat(R.color.colorPrimary)
            textSize = 12f
            typeface = Typeface.create("sans-serif-medium", Typeface.NORMAL)
        }.lparams(width = matchParent, height = wrapContent) {
            leftMargin = dip(8)
            topMargin = dip(8)
        }
    }.lparams(width = matchParent, height = wrapContent)

    fun lollipop(block: () -> Unit) {
        if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.LOLLIPOP) {
            block()
        }
    }
```

你甚至可以用for循环来做类似于适配器的事情（当然缓存是不会有缓存的 这辈子都没有做的）

```kotlin
spreadChainLayout {
                    listOf(
                            "佩奇粉" to Color.parseColor("#EDC6CD"),
                            "乔治蓝" to Color.parseColor("#6595D9"),
                            "猪妈黄" to Color.parseColor("#F4B17F"),
                            "猪爸绿" to Color.parseColor("#6FC6C5"),
                            "基佬紫" to Color.parseColor("#9C26B0")
                    ).forEachIndexed { index, (name, color) ->

                        verticalLayout {
                            colorCircleView {
                                this.color = color
                            }.lparams {
                                width = dip(24)
                                height = dip(24)
                                gravity = Gravity.CENTER_HORIZONTAL
                            }
                            textView {
                                text = name
                                textColor = color
                                textSize = 12f
                                typeface = Typeface.create("sans-serif-medium", Typeface.NORMAL)
                            }.lparams(width = wrapContent, height = wrapContent) {
                                topMargin = dip(6)
                            }
                        }.lparams {
                            width = wrapContent
                            height = wrapContent
                            horizontalPadding = dip(16)
                        }.setOnClickListener {
                            val theme = CustomTheme.themeList[index]
                            Log.e(TAG, "custom theme: $theme")
                            val activity = this@CustomSettingBottomFragment.activity
                            Colorful().edit()
                                    .setPrimaryColor(theme)
                                    .setAccentColor(theme)
                                    .apply(context) {
                                        activity?.recreate()
                                    }
                        }


                    }
                }.lparams(width = matchParent, height = wrapContent) {
                    topMargin = dip(12)
                    bottomMargin = dip(12)
                }
```

在一个Layout的闭包里面写循环，填充数据，然后addView，有了Kotlin的语法糖 + Anko变得很舒服。

你甚至可以在Recyclerview里面写Anko

```kotlin

class Item(var text: String, var builder: (TextView.() -> Unit)? = null)

class ViewHolder(itemView: View?, val textView: TextView) : RecyclerView.ViewHolder(itemView)

override fun onCreateViewHolder(parent: ViewGroup): RecyclerView.ViewHolder {
    lateinit var textView: TextView
    val view = parent.context.constraintLayout {
         textView = textView {
            text = "课程表设置"
            id = View.generateViewId()
            textSize = 16f
            textColor = Color.BLACK
        }.lparams(width = wrapContent, height = wrapContent) {
            startToStart = PARENT_ID
            topToTop = PARENT_ID
            bottomToBottom = PARENT_ID
            margin = dip(16)
        }
        imageView {
            backgroundColor = getColorCompat(R.color.common_lv4_divider)
        }.lparams(width = matchParent, height = dip(1)) {
            bottomToBottom = PARENT_ID
        }
    }.apply {
        layoutParams = RecyclerView.LayoutParams(matchParent, wrapContent)
    }
    return ViewHolder(view,textView)
}

override fun onBindViewHolder(holder: RecyclerView.ViewHolder, item: Item) {
    item as SingleTextItem
    holder as ViewHolder
    holder.textView.text = item.text
    item.builder?.invoke(holder.textView)
}

```

在数据里面附着上一个闭包，便可以实现TextView的自定义（把逻辑从onBindViewHolder里面抽离出来），我们的项目中Recyclerview Adapter做了DSL风格的二次封装，目前处于测试阶段，稳定了之后会分享在博客里面。

DSL大概是这样子的：

```kotlin
        recyclerView.withItems {
            courseInfo(course = course)
            indicatorText("上课信息")

            val week = course.week
            course.arrangeBackup.forEach {
                iconLabel(CourseDetailViewModel(R.drawable.ic_schedule_location, "${week.start}-${week.end}周，${it.week}上课，每周${getChineseCharacter(it.day)}第${it.start}-${it.end}节\n${it.room}"))
            }

            indicatorText("其他信息")
            iconLabel(CourseDetailViewModel(R.drawable.ic_schedule_other, "逻辑班号：${course.classid}\n课程编号：${course.courseid}"))
            indicatorText("自定义（开发中 敬请期待）")
            iconLabel(CourseDetailViewModel(R.drawable.ic_schedule_search, "在蹭课功能中搜索相似课程"))
            iconLabel(CourseDetailViewModel(R.drawable.ic_schedule_event, "添加自定义课程/事件"))
            iconLabel(CourseDetailViewModel(R.drawable.ic_schedule_homework, "添加课程作业/考试"))
            indicatorText("帮助")
        }
```

## 做个总结？

**DSL最吸引人的地方就在于**，它可以在布局上加入逻辑，对于布局过程，它有着编程语言级别的控制，比如说封装成类，封装成函数什么的。这些东西在XML里面都是无法做到的，因为aapt工具的局限性，XML只能按照固定的格式写布局 + 代码控制来提供动态性，反正就很蛋疼。

**而DSL可以解决很多问题**，比如说用一个for循环来取代Adapter填充View功能，避免了很多无用的操作。比如说在布局里面加一个if就可以来操作一个控件的布局与否，而不是在findView之后控制Visibility，可以用Kotlin的闭包来封装一个View的初始化操作什么的，重复的操作就可以封装起来，再比如XML只能设置paddingLeft/paddingRight，在Anko DSL / 自定义DSL里面就可以很轻易的封装出一个horizontalPadding。当然Anko因为避免了反射，提高了大量的性能。

**DSL和XML并不是冲突的**，DSL用于解决布局中细碎和动态的部分，而XML用于单页布局，复杂布局。同时DSL和XML也可以无缝嵌合在一起，所以两者并不是冲突的关系，也没有必要去选择“我到底该用DSL写还是XML写”，两者各有优点，了解Anko DSL并且与XML活用起来才是最优解。XML可以拿到ViewGroup的应用然后用DSL做骚操作，DSL也可以动态添加Inflate出来的XML来实现复杂页面布局的添加

**DSL和XML各有所长**，DSL更适合用于页面模块的解耦，XML更多用于单页构建 / 复杂布局，两者相互结合相互服务。

## 还想说的

Anko DSL让人望而却步的部分就是它不能支持即时预览，所以这个局限性也就导致Anko无法构建大型复杂的页面。而当你的设计图可以精确到dp的时候，完全可以用DSL来描述UI的各个小组件，因此DSL在这里不应该被一棒子打死，DSL在目前的项目中，可以很好的替代手工`new View, add view`的部分，以及小规模的View控制。

如果你认真看了上面的内容，并且有自己的体会，可以在已有的UI构架中很快的用上Anko Layout来解决一些轻量级UI的构建。比如说List中的一个Item，或者一个小Dialog之类。

没有所谓的“最佳实践”，对于业务与技术的一步步探索才是最重要的。