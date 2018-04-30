# Extension In Action

去年的Google IO大会让Kotlin语言大火，大量开发者尝试使用Kotlin进行开发，然而很多人抱怨道：“Kotlin有什么好的啊？没有什么感觉啊？不就是一些语法糖吗？”… 究其原因，很多人依然在写着Java Style的Kotlin，并没有尝试着去感悟kotlin的语法糖背后的设计思维，以及如何将那些语法特性运用在生产环境的项目中。

最近的一些时间，我们在使用Kotlin对项目基础架构进行重新设计，于是在这篇文章写下我们的心得体会~

### 导读：

- Kotlin-Tips: https://github.com/heimashi/kotlin_tips 文章超棒
- 如何发挥拓展的魅力
- 待续… 会写代理，重载什么的

## 拓展

> 拓展干什么？

拓展功能是一个非常有魅力的功能，它可以让我们对已有的SDK进行功能的增强（在保证代码优雅的情况下）。

- **场景1：**在我们的项目中，为了做到APP的沉浸，需要获取到StatusBar的高度，在Java里面，你可以写一个某某Util来获取这个信息，而在Kotlin中，你可以使用拓展属性来为Activity进行拓展。

```kotlin
inline val Activity.statusBarHeight
    get() = resources.getIdentifier("status_bar_height", "dimen", "android")
            .takeIf { it > 0 }?.let {
                resources.getDimensionPixelSize(it)
            } ?: 0

inline val Activity.navigationBarHeight
    get() = resources.getIdentifier("navigation_bar_height", "dimen", "android")
            .takeIf { it > 0 }?.let {
                resources.getDimensionPixelSize(it)
            } ?: 0

```

在使用拓展属性的时候，就有着使用自己原生的属性一样的感觉。而我们还不算完，我们可以使用多个拓展属性，拓展方法的配合来让我们的拓展功能更加优雅。比如说，获取了statusBarHeight后，我们对外层view进行重新设置padding来让其达成沉浸式的效果。这时候我们就可以这样子来写：

```kotlin
fun Activity.fitSystemWindowWithStatusBar(view: View) = with(view) {
    setPadding(paddingLeft, paddingTop + statusBarHeight, paddingRight, paddingBottom)
    if (layoutParams.height > 0)
        layoutParams.height += statusBarHeight
}
```

在Activity里面对View带哦用这个方法，就像是Activity自己的方法一样。（Activity自己就支持了沉浸式耶）

```kotlin
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.gpa2_activity_evaluate)
        
        // init toolbar
        toolbar = findViewById<Toolbar>(R.id.toolbar).also {
            fitSystemWindowWithStatusBar(it)
            setSupportActionBar(it)
        }
```

> 参考代码：[UISupport.kt](https://github.com/twtstudio/WePeiYang-Android/blob/refactor/WePeiYang/commons/src/main/java/com/twt/wepeiyang/commons/experimental/extensions/UISupport.kt)

- **场景2：**OkHttp封装网络框架的时候，比如说我们需要对不同的请求使用不同的Ineceptor（对于某个域名下的请求要加入Token，对于非该域名的访问不加入任何Token和UA）。应该怎么做？

或者应该问… 怎么样才是优雅的做法？你的大脑闪过：那我创建两个Client好了… （极其暴力）
这个时候，我们就需要用到Kotlin的拓展属性来做这件事情。你也许会疑惑… 这个怎么用在这里？让我们来慢慢看~

> 预备知识：[OkHttp的Interceptor设计理念与使用](https://www.jianshu.com/p/2710ed1e6b48)

 ![OkHttp拦截器](https://upload-images.jianshu.io/upload_images/1504154-8daf5fd9540545d3.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/700)

```kotlin
object ServiceFactory {

    internal const val TRUSTED_HOST = "open.twtstudio.com"
    internal const val BASE_URL = "https://$TRUSTED_HOST/api/"

    internal const val APP_KEY = "xxxxxxx"
    internal const val APP_SECRET = "xxxxxx"

    private val loggingInterceptor = HttpLoggingInterceptor()
            .apply { level = HttpLoggingInterceptor.Level.BODY }

    val client = OkHttpClient.Builder()
            .addInterceptor(UserAgentInterceptor.forTrusted)
            .addInterceptor(SignatureInterceptor.forTrusted)
            .addInterceptor(AuthorizationInterceptor.forTrusted)
            .authenticator(RealAuthenticator)
            .addNetworkInterceptor(loggingInterceptor)
            .addNetworkInterceptor(CodeCorrectionInterceptor.forTrusted)
            .build()

	//省略其他代码...
}
```

你会发现，每一个Interceptor的后面都会加上一个`.forTrusted`，这里就是对于拓展属性的运用。那么我们如何设计这里？

```kotlin
internal inline val Request.isTrusted
    get() = url().host() == ServiceFactory.TRUSTED_HOST

/**
 * A wrapped interceptor only applied to trusted request.
 *
 * @see ServiceFactory
 */
internal val Interceptor.forTrusted
    get() = Interceptor {
        if (it.request().isTrusted) intercept(it)
        else it.proceed(it.request())
    }
```

为所有的Interceptor类型添加`forTrusted`拓展属性。在取得这个属性的时候，返回的是一个Interceptor的实现，因为Interceptor接口只有一个方法，所以可以直接用lambda表达式来写，表达式大括号里面便是对`Response intercept(Chain chain) throws IOException`这个方法的实现，其实就是在这里加了一层AOP，如果要拦截的Request是`Trusted`，我们就调用被拓展对象的`intercept`方法来拦截它，否则直接放行，传到下一层chain。

在这里我们也对于`Request`进行了`isTrusted`属性的拓展，这种属性拓展，可以大幅度提高代码的可读性，因为你发现，即使我不去看`isTrusted`拓展的实现，我也可以推测出这个属性的意思。这样子就很可读… 

![已经可以了大佬](image-201803172217104-1296235.png)

多说几句，这种使用拓展属性的写法，对于基础库封装和其他的场景，可以通过拓展属性/拓展方法通俗易懂的命名来大幅度提高代码的可读性 —— Rickygao同学（大意是这样子的）

> 如果你要抬杠，可以说java里面封装静态Util也可以做到… 但代码可读性还是会有一层的差距的 （逃

再举一个这方面的例子，我们写了一个Interceptor来处理OkHttp请求的签名。我们先看一段代码

```kotlin
internal object SignatureInterceptor : Interceptor {
    override fun intercept(chain: Interceptor.Chain): Response =
            chain.proceed(chain.request().signed)
}
```

我们只看到这一部分，看到`chain.request().signed`的时候，你就应该想到：“哦，这个地方是发了一个Request的签名后处理版”。如果我们只是为了梳理代码逻辑，看到这里就已经足够了… 我们已经知道了这个Interceptor是签名了网络请求（就可以溜了 

如果我们要详细看签名逻辑，我们可以点进去`signed`拓展看它的逻辑。

```kotlin
internal inline val Request.signed
    get() = when (method()) {
        "HEAD" -> this
        "GET" -> newBuilder().url(url().signed).build()
        "POST", "PUT", "PATCH", "DELETE" -> newBuilder().method(method(), body()?.signed).build()
        else -> this
    }
```

这里我们可以看到，`signed`对于不同请求类型进行处理（when简直好用到爆炸），然后我们发现对于不同的请求，有着不同分支处理，比如说GET它sign了url，POST什么的，对body做sign… (因为是inline拓展，所以不用担心性能 逃

这块我们就看到这里。你会发现，通过拓展属性的抽离，你可以在不把实现一层层看完的情况下，对于代码组织依然有着清晰的理解… Kotlin拓展的魅力。

