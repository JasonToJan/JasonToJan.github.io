---
title: 玩Android Compose版本 项目分析
date: 2023-01-17 11:23:20
top: false
cover: false
toc: true
mathjax: true
tags:
- Android
categories:
- Android
---

> 玩Android(compose版本)项目地址：[https://github.com/yellowhai/PlayAndroid](https://github.com/yellowhai/PlayAndroid)

### 1.项目settings.gradle

```groovy
dependencyResolutionManagement {
    /**
     * 原文中说默认情况下，项目中的存储库会覆盖设置中的存储库 ，可以通过设置模式来更改这种行为
     设置存储库的方法：
     repositoriesMode.set(RepositoriesMode.FAIL_ON_PROJECT_REPOS)
     存储库模式：
     PREFER_PROJECT(true)--首选项目存储库
     PREFER_SETTINGS(false)--首选设置存储库
     FAIL_ON_PROJECT_REPOS(false)--强制设置存储库
     */
    repositoriesMode.set(RepositoriesMode.FAIL_ON_PROJECT_REPOS)
    repositories {
        google()
        mavenCentral()
        maven { url "https://jitpack.io" }
        maven {
            url 'https://maven.aliyun.com/repository/public/'
        }
        maven { url 'https://maven.aliyun.com/repository/public' }
        maven { url 'https://maven.aliyun.com/repository/central' }
        maven { url 'https://maven.aliyun.com/repository/google' }
        maven { url 'https://maven.aliyun.com/repository/public' }
        maven { url 'https://maven.aliyun.com/repository/gradle-plugin' }
    }
}
rootProject.name = "PlayAndroid"
include ':app'
include ':common'
include ':h_mine'
include ':toolkit'
```

还是很容易理解的，repositoriesMode.set(RepositoriesMode.FAIL_ON_PROJECT_REPOS)这个是新增的，简单看下就行。
引入了app模块(主模块)，common模块（通用模块，用于通用组件相关）,h_mine模块（我的模块，可以单独编译）。

### 2.config.gradle
这个文件相信大家都有用过的，配置远程依赖的一个文件，统一管理。
```groovy
ext{
    isRelease = true

    //android构建配置
    android_models = [
            compileSdk : 32,
            minSdk : 23,
            targetSdk : 32,
            versionCode : 1,
            versionName : "1.0"
    ]
    
    app_id = [
            mine      : 'com.hh.mine',
    ]

    core_ktx_version = '1.7.0'
    appcompat_version = '1.4.0'
    lifecycle_version = '2.4.0'
    material_version = '1.4.0'
    work_version = '2.7.1'
    gson_version = '2.8.9'
    litepal_version = '3.2.3'
    landscapist_version = '1.4.5'
    retrofit_version = '2.9.0'
    okhttp_version = '4.9.3'
    startup_version = '1.1.0'
    XXPermissions_version = '13.2'
    datastore_version = '1.0.0'
    materialDialog_version = '0.6.2'
    accompanist_version = '0.25.0'


    jetpack_compose = [
            material : "androidx.compose.material:material:$compose_version",
            activity : "androidx.activity:activity-compose:$appcompat_version",
    ]

    commonApi = [
            ktx_core        : "androidx.core:core-ktx:$core_ktx_version",
            lifecycle       : "androidx.lifecycle:lifecycle-runtime-ktx:$lifecycle_version",
            appcompat       : "androidx.appcompat:appcompat:$appcompat_version",
            material        : "com.google.android.material:material:$material_version",
            //数据存储
            datastore       : "androidx.datastore:datastore-preferences:$datastore_version",
            //权限处理
            XXPermissions   : "com.github.getActivity:XXPermissions:$XXPermissions_version",
    ]

    net = [
            retrofit    : "com.squareup.retrofit2:retrofit:$retrofit_version",
            converter   : "com.squareup.retrofit2:converter-gson:$retrofit_version",
            okhttp      : "com.squareup.okhttp3:logging-interceptor:$okhttp_version",
            gson        : "com.google.code.gson:gson:$gson_version"
    ]

    accompanist_ui = [
            insets_ui    : "com.google.accompanist:accompanist-insets-ui:$accompanist_version",
            navigation   : "com.google.accompanist:accompanist-navigation-animation:$accompanist_version",
            pager        : "com.google.accompanist:accompanist-pager:$accompanist_version",
            swiperefresh : "com.google.accompanist:accompanist-swiperefresh:$accompanist_version",
            flowlayout   : "com.google.accompanist:accompanist-flowlayout:$accompanist_version",
            systemUi     : "com.google.accompanist:accompanist-systemuicontroller:$accompanist_version",
            webview      : "com.google.accompanist:accompanist-webview:$accompanist_version"
    ]

    database =[
            litepal :  "org.litepal.guolindev:core:$litepal_version"
    ]

    skydoves_coil = [
            landscapist_coil :  "com.github.skydoves:landscapist-coil:$landscapist_version"
    ]

    workManager = [
            // Kotlin + coroutines
            work = "androidx.work:work-runtime-ktx:$work_version"
    ]

    material_dialog = [
            color : "io.github.vanpra.compose-material-dialogs:color:$materialDialog_version",
            core  : "io.github.vanpra.compose-material-dialogs:core:$materialDialog_version",
            datetime  : "io.github.vanpra.compose-material-dialogs:datetime:$materialDialog_version"
    ]

    paging = "androidx.paging:paging-compose:1.0.0-alpha14"

    jetpackComposeLibs = jetpack_compose.values()
    commonApiLibs = commonApi.values()
    netLibs = net.values()
    materialDialogLibs = material_dialog.values()
}
```
层次还是比较清晰，对于同一组单独建立数组存储，模块中只需要引入一个就将这组全部引入进去了。

### 3.app模块下的build.gradle

显示引入com.android.application+kotlin-android的插件。
下面配置android闭包，很简单。

其它关于compose的也需要配置下：
```groovy
 compileOptions {
        sourceCompatibility JavaVersion.VERSION_1_8
        targetCompatibility JavaVersion.VERSION_1_8
    }
    buildFeatures {
        compose true
    }
    composeOptions {
        kotlinCompilerExtensionVersion compose_version
        kotlinCompilerVersion kotlin_version
    }
```

然后是远程依赖：
```groovy
dependencies {
    // 如果是release模式，就把我的模块加进来
    if(isRelease){
        implementation project(':h_mine')
    }

    // 初始化组件
    implementation "androidx.startup:startup-runtime:$startup_version"
    implementation project(':common')

    // 测试相关
    testImplementation 'junit:junit:4.13.2'
    androidTestImplementation 'androidx.test.ext:junit:1.1.3'
    androidTestImplementation 'androidx.test.espresso:espresso-core:3.4.0'
    androidTestImplementation "androidx.compose.ui:ui-test-junit4:$compose_version"
    androidTestImplementation "androidx.compose.ui:ui-test-junit4:$compose_version"
    debugImplementation "androidx.compose.ui:ui-tooling:$compose_version"

}
```

isRelease定义在 config.gradle的顶部。如果是集成编译就为true，单独编译为false。

### 4.app模块的AndroidManifest.xml
* 配置Application
* 配置主页
* 配置provider，用于初始化sdk
    ```xml
      <!--  用provider初始化sdk-->
        <provider
            android:name="androidx.startup.InitializationProvider"
            android:authorities="${applicationId}.androidx-startup"
            android:exported="false"
            tools:node="merge">
            <meta-data
                android:name="com.hh.playandroid.base.BaseInitializer"
                android:value="androidx.startup" />
        </provider>
    ```

### 5.自定义Application
 ```
 class HhfApp : YshhApplication()
 ```   
 依赖common模块：
 ```Kotlin
 open class YshhApplication : Application() {

    lateinit var okbuilder: OkHttpClient

    companion object {
        /**
         * application context.
         */
        @SuppressLint("StaticFieldLeak")
        lateinit var context: Context

        @SuppressLint("StaticFieldLeak")
        lateinit var instance: YshhApplication

        /**
         * application级别的协程
         * 有时我们需要在协程上下文中定义多个元素，组合协程上下文中的元素，使用 + 操作符来创建 CoroutineScope .
         */
        val applicationScope = CoroutineScope(SupervisorJob() + Dispatchers.Main)
    }
 ```
 这里懒加载OkHttpClient
 定义了全局的协程，静态变量。

 ```Kotlin
   override fun onCreate() {
        super.onCreate()
        context = applicationContext
        instance = this
        initRetrofit()
    }
 ```
 初始化Retrofit：
 ```Kotlin
  /**
     * 初始化Retrofit
     */
    private fun initRetrofit(token : String = ""): OkHttpClient {
        //请求头
        val headerInterceptor = Interceptor { chain: Interceptor.Chain ->
            val orignaRequest = chain.request()
            val request = orignaRequest.newBuilder()
                .header("Authorization", "Bearer $token")
                .method(orignaRequest.method, orignaRequest.body)
                .build()
            chain.proceed(request)
        }
        val logInterceptor = LogInterceptor {
//            it.logE()
        }
        // 日志类别
        logInterceptor.setLevel(HttpLoggingInterceptor.Level.BODY);
        // 缓存相关
        val cacheFile =
            File(getExternalFilesDir(Environment.DIRECTORY_PICTURES).toString() + "http_cache")
        val cache = Cache(cacheFile, 104857600L) // 指定缓存大小100Mb
        val builder = OkHttpClient.Builder()
        //        builder.addInterceptor(addQueryParameterInterceptor);
        builder.addInterceptor(headerInterceptor)
        builder.cache(cache)
        builder.addNetworkInterceptor(logInterceptor)
        builder.cookieJar(cookieJar)
        builder.readTimeout(60000, TimeUnit.MILLISECONDS)
        //全局的写入超时时间60s
        builder.writeTimeout(60000, TimeUnit.MILLISECONDS)
        //全局的连接超时时间30s
        builder.connectTimeout(30000, TimeUnit.MILLISECONDS)
        okbuilder = builder.build()
        return builder.build()
    }

    private val cookieJar: PersistentCookieJar by lazy {
        PersistentCookieJar(SetCookieCache(), SharedPrefsCookiePersistor(context))
    }
    
    fun getOkBuilder(): OkHttpClient {
        return okbuilder
    }
 ```

 ### 6.首页 MainActivity
 继承BaseActivity:
 ```Kotlin

/**
 * 基类Activity 设置非沉浸式，因为要设置导航栏和状态栏颜色
 */
abstract class BaseActivity : AppCompatActivity() {

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        WindowCompat.setDecorFitsSystemWindows(window, false)
    }

}
 ```

onCreate生命周期：
```Kotlin
 @Suppress("DEPRECATED_IDENTITY_EQUALS")
    @OptIn(ExperimentalAnimationApi::class)
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        // 动画渐变
        overridePendingTransition(android.R.anim.fade_in, android.R.anim.fade_out)
        // 设置内容
        setContent {
            Log.e("TEST##", "开始setContent了")
            // 顶层为自定义主题
            HhfTheme(
                // 是否暗色或亮色，跟随系统，如果用户主动设置，跟随用户
                theme = if (isSystemInDarkTheme() || isNight) HhfTheme.Theme.Dark else HhfTheme.Theme.Light,
                // 用户自行设置的颜色主题
                colorTheme = appTheme
            ) {
                // 创建一个MainViewModel
                val viewModel : MainViewModel = viewModel()

                // 如果isSplash为true,就展示SplashView 启动页
                if (viewModel.isSplash) {
                    Log.e("TEST##", "加载闪屏了")
                    SplashView { viewModel.isSplash = false }
                } else {
                    Log.e("TEST##", "加载首页了")
                    // 全局静态变量存 底部控制器
                    CpNavigation.navHostController = rememberAnimatedNavController()
                    // 主页面
                    HhfNavigation()
                    // 进度条
                    DialogProgress()
                }
                // 是否纪念日，将App置灰
                if(isMourningDay()){
                    Canvas(modifier = Modifier.fillMaxSize()){
                        drawRect(color = Color.White,blendMode = BlendMode.Saturation)
                    }
                }
            }
        }
        addCallback()
    }
```
因为这个app就一个Activity，setContent中的内容就是展现给用户可以看到的所有东西了。
顶层是一个自定义主题：
```Kotlin
@Composable
fun HhfTheme(
    theme: HhfTheme.Theme = HhfTheme.Theme.Light,
    colorTheme: Color? = null,
    content: @Composable () -> Unit
) {
    val targetColors = if (theme == HhfTheme.Theme.Dark) {
        colorTheme?.let {
            rememberSystemUiController().setNavigationBarColor(colorTheme)
            DarkColorPalette.apply {
                themeColor = it
            }
        }?:DarkColorPalette
    } else {
        colorTheme?.let {
            rememberSystemUiController().setNavigationBarColor(colorTheme)
            LightColorPalette.apply {
                themeColor = it
            }
        }?:LightColorPalette
    }
    val themeColor = animateColorAsState(targetColors.themeColor, TweenSpec(600))
    val textColor = animateColorAsState(targetColors.textColor, TweenSpec(600))
    val background = animateColorAsState(targetColors.background,TweenSpec(600))
    val listItem =  animateColorAsState(targetColors.listItem,TweenSpec(600))
    val bottomBar =  animateColorAsState(targetColors.bottomBar,TweenSpec(600))
    val divider =  animateColorAsState(targetColors.divider,TweenSpec(600))
    val textFieldBackground =  animateColorAsState(targetColors.textFieldBackground,TweenSpec(600))
    val colors = HhfColors(themeColor.value,
        textColor.value,
        background = background.value,
        listItem = listItem.value,
        bottomBar = bottomBar.value,
        divider = divider.value,
        textFieldBackground = textFieldBackground.value,
        )
    CompositionLocalProvider(LocalHhfColors provides colors) {
        MaterialTheme(
            shapes = Shapes,
            typography = typography
        ) {
            content.invoke()
        }
    }
}
```
这里用到了一个 CompositionLocalProvider，具体用法可以参考 [https://juejin.cn/post/7097890697721675813](https://juejin.cn/post/7097890697721675813)

在Compose函数里，如果要用到函数外提供的值，一般都要通过函数定义好参数，外部调用时传入进去。
这种方法表面看没啥问题，但在一些场景下就不是很方便。比如一个函数A嵌套一个函数B，函数B嵌套函数C，并且这A、B、C的函数都用到这个参数，那对三个函数来说都需要定义这个参数，就不是那么方便。
那用个全局变量不就可以解决需要重复定义参数，确实可以。但全局变量有个副作用就是影响的范围比较大。本身我们定义的参数只会影响到函数内部。这个时候我们就可以使用CompositionLocal：它具有穿透函数功能的局部变量
CompositionLocal适用场景：用来提供上下文数据，不扩大影响范围。

然后最终主题呈现是用了 MaterialTheme这个类展现。

回到首页，内容区：
* 闪屏页展示逻辑
    通过一个变量：`  var isSplash by mutableStateOf(true)` 实现控制是否显示
*  加载首页
    先 展示主页面，覆盖一层 进度条
* 是否纪念日
    app置灰处理    
* 添加二次点击返回退出app逻辑

首页逻辑基本就这么多了。

### 7.闪屏ui -> SplashView
看下效果先：
<img src=%E9%97%AA%E5%B1%8F%E9%A1%B5.gif width = 50%>

可以看到进入app后有个启动页面，中间是logo。
那这个页面有个渐变动画，淡入淡出效果。

具体ui是这样的：
```Kotlin
@Composable
fun SplashView(startMain: () -> Unit) {
    /**
     * 辅助文字颜色变化 建立的一个mutableState包装的变量，实际上就是bool值
     */
    var enabled by remember { mutableStateOf(false) }

    /**
     * 背景颜色，从0.3alpha主题色变为纯主题色，2s的时间
     */
    val bgColor: Color by animateColorAsState(
        if (enabled) HhfTheme.colors.themeColor
        else HhfTheme.colors.themeColor.copy(alpha = 0.3f),
        animationSpec = tween(durationMillis = 2000)
    )

    /**
     * 中间文字颜色动画，如果enable为false，0.3alpha的颜色 到纯白色 2s的时间
     */
    val textColor: Color by animateColorAsState(
        if (enabled) Color.White
        else Color.White.copy(alpha = 0.3f),
        animationSpec = tween(durationMillis = 2000)
    )

    // 外层一个Box+中间一个Text
    Box(
        Modifier
            .fillMaxSize()
            .background(bgColor),
        contentAlignment = Alignment.Center
    ) {
        Text(text = "PlayAndroid", color = textColor,style = MaterialTheme.typography.h5)
    }

    // 开启协程 可以看出Compose里面开启协程的方法
    LaunchedEffect(Unit) {
        enabled = true
        delay(2000)
        // 2s后，设置 mutableStateOf 包装的bool变量 为false 会通知首页重新走setContent方法
        startMain.invoke()
    }
}
```

这里需要理解 animateColorAsState 是官方提供的一个动画，支持颜色变化的一个composable方法。

然后这里开启了2s延迟，怎么开延迟的，调用LaunchedEffect方法里面的delay，可以实现延迟效果。然后执行startMain回调。

这个回调很简单：viewModel.isSplash = false

虽然很简单，但实际上走了很多流程的。

```Kotlin
class MainViewModel : ViewModel() {
    var isSplash by mutableStateOf(true)
}
```
这里使用mutableStateOf包装了一个布尔值。 上面触发这改变时，会刷新composable注解声明的方法体。这里应该就是刷新MainActivity整个内容区了。


### 8.主页面外部架构
```Kotlin
@ExperimentalAnimationApi
@OptIn(ExperimentalPagerApi::class)
@Composable
fun HhfNavigation() {
    AnimatedNavHost(navController = CpNavigation.navHostController,
        /**
         * 首页默认页面
         */
        startDestination = ModelPath.Main.route,
        enterTransition = { fadeIn(animationSpec = tween(700), initialAlpha = 0f) },
        exitTransition = { fadeOut(animationSpec = tween(700), targetAlpha = 0f) }) {

        /**
         * 定义接收到 main 路由消息后->展示MainContent视图
         */
        composable(ModelPath.Main.route) {
            MainContent()
        }

        /**
         * 接收到 setting 路由消息后 -> 设置页面
         */
        composable(ModelPath.Setting.route) {
            CpSetting(Modifier.fillMaxSize())
        }
        ...
```

这里使用了一个AnimatedNavHost，这个类是官方提供的。
这个类里面定义了所有我们需要跳转的页面和初始页。
然后这个 需要传一个参数，也就是控制器：
` CpNavigation.navHostController = rememberAnimatedNavController()`
这里也是官方提供的remember包装的一个控制器。

这里的初始页为：`  startDestination = ModelPath.Main.route,`
本质上就是一个string，首页的路由。

这样会默认展示首页，怎么展示首页呢？
```Kotlin
composable(ModelPath.Main.route) {
            MainContent()
        }
```
那么这个MainContent就是我们的首页。

其它composable里面都是首页可能去哪些页面的路由定义。

那这个比如跳转到设置页面，怎么处理呢？
* 首先在这个AnimatedNavHost里面定义一个composable闭包
```Kotlin
 composable(ModelPath.Setting.route) {
            CpSetting(Modifier.fillMaxSize())
        }
```

* 然后再需要跳转的地方调用
`navHostController.navigate(“setting”)`
这个navHostController就是我们之前定义的那个控制器，setting就是路由名称。
这样子就可以跳转到设置页面了。

### 9.主页内部架构框
首先看下主页效果图
<img src=%E9%A6%96%E9%A1%B5.jpeg width=50%>
轮播图+中间列表+底部bar
轮播图+中间列表可以看成一个整体，那就是内容区+底部bar

架构怎么搭建呢？
```Kotlin
   Scaffold(bottomBar = {
        AnimatedVisibility(
            visible = bottomSwitch,
            enter = expandVertically() + fadeIn(), // 操作符重载了，第一个为空，就用第二个
            exit = shrinkVertically() + fadeOut()  // 操作符重载了，第一个为空，就用第二个
        ) {
            // 实际的底部bar 高度使用 固有特性测量 可以参考：https://juejin.cn/post/7068164264363556872
            MainBottomBar(
                Modifier
                    .fillMaxWidth()
                    .height(IntrinsicSize.Max),pagerState = pagerState) {
                pagerState.reenableScrolling(coroutineScope, it)
            }
        }
    }) {
        内容区...
```
最外层一个Scafffold脚手架包裹，有点像Flutter了啊。
然后是一个AnimatedVisibility，这个控制显示隐藏的动画组件。重点看里面的MainBottomBar，就找到了我们的底部bar了。

这个看起来像自定义的，具体怎么实现的呢？
```Kotlin
/**
 * App底部bar
 */
@OptIn(ExperimentalPagerApi::class)
@Composable
private fun MainBottomBar(
    modifier: Modifier = Modifier,
    pagerState: PagerState,
    currentChanged: (Int) -> Unit
) {
    /**
     * 这个BottomAppBar是官方的
     */
    BottomAppBar(
        modifier.navigationBarsPadding(),
        cutoutShape = CircleShape,
        backgroundColor = HhfTheme.colors.bottomBar,
        elevation = 5.dp
    ) {
        // 遍历4个tab
        bottomList.forEachIndexed { index, item ->
            // item单独设置 官方组件
            BottomNavigationItem(
                selected = pagerState.currentPage == index, onClick = {
                    /**
                     * item点击事件回调给上层
                     */
                    currentChanged(index)
                }, icon = {
                    /**
                     * icon为官方组件
                     */
                    Icon(
                        item.dashboardState.icon,
                        contentDescription = item.name,
                        tint = if (pagerState.currentPage == index) {
                            HhfTheme.colors.themeColor
                        } else {
                            LocalContentColor.current.copy(alpha = LocalContentAlpha.current)
                        }, modifier = Modifier.size(24.dp)
                    )
                }, label = {
                    Text(
                        text = item.name,
                        color = if (pagerState.currentPage == index) {
                            HhfTheme.colors.themeColor
                        } else {
                            LocalContentColor.current.copy(alpha = LocalContentAlpha.current)
                        },
                        fontSize = 12.sp
                    )
                },
                unselectedContentColor = LocalContentColor.current.copy(alpha = LocalContentAlpha.current),
                selectedContentColor = HhfTheme.colors.themeColor,
                alwaysShowLabel = true

            )
        }
    }
}
```
这里看出底部bar就是用了官方的BottomAppBar。
里面具体内容是我们开发者自行设定的，这里遍历了bottomList，这个集合就是我们自己定义的一个集合：
```Kotlin
/**
 * 底部List集合 这里定义了4块
 * 首页，项目，公众号，我的
 */
val bottomList = listOf(
    BottomBean(
        DashboardState.Home,
        YshhApplication.context.stringResource(R.string.main_title_home)
    ),
    BottomBean(
        DashboardState.Project,
        YshhApplication.context.stringResource(R.string.main_title_project)
    ),
    BottomBean(
        DashboardState.PubAccount,
        YshhApplication.context.stringResource(R.string.main_title_account)
    ),
    BottomBean(
        DashboardState.Mine,
        YshhApplication.context.stringResource(R.string.main_title_mine)
    )
)
```
就这几个模块。

底部bar的item就是 BottomNavigationitem里面的了，可以看到是一个icon+label构成。有个onClick的参数就是点击事件回调了，这里调用了 回调函数，传递给上层了，传出去的参数就一个int，表示第几页。

回调到哪里呢？
```Kotlin
 // 实际的底部bar 高度使用 固有特性测量 可以参考：https://juejin.cn/post/7068164264363556872
            MainBottomBar(
                Modifier
                    .fillMaxWidth()
                    .height(IntrinsicSize.Max),pagerState = pagerState) {
                pagerState.reenableScrolling(coroutineScope, it)
            }
```
这里通过调用rememberPagerState这个对象的`  pagerState.reenableScrolling(coroutineScope, it)`这个方法实现滚动到目标tab下。也就实现了页面切换。
最为关键的就是这个pagerState，同步页面数据和底部bar。

等下分析内容区也会用到。

```Kotlin
 // 水平分页
            HorizontalPager(
                count = bottomList.size,
                modifier.weight(1f), pagerState, userScrollEnabled = true
            ) { page ->
                when (page) {
                    // 首页tab
                    0 -> HomeView(
                        Modifier
                            .fillMaxSize()
                            .background(HhfTheme.colors.background)
                            .align(Alignment.CenterHorizontally)
                    )
                    // 项目tab
                    1 -> ProjectView(
                        Modifier
                            .fillMaxSize()
                            .align(Alignment.CenterHorizontally)
                    )
                    // 公众号tab
                    2 -> AccountView(
                        Modifier
                            .fillMaxSize()
                            .background(HhfTheme.colors.background)
                            .align(Alignment.CenterHorizontally)
                    )
                    // 我的tab
                    3 -> {
//                        if (isLogin) {
                            Mine(
                                Modifier
                                    .fillMaxSize()
                                    .background(HhfTheme.colors.background)
                                    .align(Alignment.CenterHorizontally)
                            )
//                        } else {
//                            ErrorBox(
//                                Modifier.fillMaxSize(),
//                                title = stringResource(id = R.string.sign_in)
//                            )
//                            { CpNavigation.to(ModelPath.Login) }
//                        }
                    }
                }
            }
```
这里是中间内容区。通过使用水平分页得以实现。可以看到传入了一个参数，也就是上面底部bar用到的一个pagerState，定义了不同索引下，中间内容区展示的内容。

* 首页Tab 对应 HomeView
* 项目Tab 对应 ProjectView
* 公众号Tab 对应 AccountView
* 我的Tab 对应 Mine

### 10.首页Tab-HomeView
首页效果图就是上面的那个效果。
这里再拆分下，就是轮播图+列表。
对应 HomeView。

```Kotlin
@CompoKotlinsable
fun HomeView(modifier: Modifier = Modifier) {
    val viewModel: HomeViewModel = viewModel()
    ColumnTopBarMain(modifier
        .background(HhfTheme.colors.background),
        stringResource(R.string.app_name)) {
        // 内容区域，传入HomeViewModel对象
        HomeContent(viewModel = viewModel)
    }
}
```
最外层是由顶部bar+内容区构成。

ColumnTabBarMain是自定义通用的一个App的组件，用于上面展示toolbar，中间区域自定义。
```Kotlin
@Composable
fun ColumnTopBarMain(
    modifier: Modifier = Modifier,
    title: String,
    content: @Composable ColumnScope.() -> Unit
) {
    Column(
        modifier.navigationBarsPadding(),
        horizontalAlignment = Alignment.CenterHorizontally
    ) {
        CpTopBar(
            Modifier.fillMaxWidth(),
            HhfTheme.colors.themeColor,
            title = title,
            actions = {
                IconButton(onClick = {
                    CpNavigation.to(ModelPath.Search)
                }) {
                    Icon(Icons.Filled.Search, contentDescription = "search", tint = Color.White)
                }
            },
        )
        content.invoke(this)
    }
}
```
这里具体是一个Column+CpTopBar构成

CpTopBar是这样的：
```Kotlin

@Composable
fun CpTopBar(
    modifier: Modifier = Modifier,
    backgroundColor: Color = Color.Transparent,
    title: String,
    actions: @Composable RowScope.() -> Unit = {},
    back: (() -> Unit)? = null
) {
    HhTopAppBar(
        {
            Text(title, color = Color.White)
        },
        modifier = modifier,
        backgroundColor = backgroundColor,
        contentPadding = WindowInsets.statusBars.only(WindowInsetsSides.Horizontal + WindowInsetsSides.Top).asPaddingValues(),
        navigationIcon = back?.run {
            {
                IconButton(
                    onClick = {
                        invoke()
                    }
                ) {
                    Icon(Icons.Filled.ArrowBack, contentDescription = "back", tint = Color.White)
                }
            }
        },
        actions = actions,
        elevation = 2.dp,
    )
}
```

里面还有自定义的HhTopAppBar
```Kotlin

@Composable
fun HhTopAppBar(
    title: @Composable () -> Unit,
    modifier: Modifier = Modifier,
    contentPadding: PaddingValues = PaddingValues(0.dp),
    navigationIcon: @Composable (() -> Unit)? = null,
    actions: @Composable RowScope.() -> Unit = {},
    backgroundColor: Color = MaterialTheme.colors.primarySurface,
    contentColor: Color = contentColorFor(backgroundColor),
    elevation: Dp = AppBarDefaults.TopAppBarElevation,
) {
    TopAppBarSurface(
        modifier = modifier,
        backgroundColor = backgroundColor,
        contentColor = contentColor,
        elevation = elevation
    ) {
        TopAppBarContent(
            title = title,
            navigationIcon = navigationIcon,
            actions = actions,
            modifier = Modifier.padding(contentPadding)
        )
    }
}
```

竟然还有自定义层：Surface是官方提供的了
```Kotlin
@Composable
fun TopAppBarSurface(
    modifier: Modifier = Modifier,
    backgroundColor: Color = MaterialTheme.colors.primarySurface,
    contentColor: Color = contentColorFor(backgroundColor),
    elevation: Dp = AppBarDefaults.TopAppBarElevation,
    content: @Composable () -> Unit,
) {
    Surface(
        color = backgroundColor,
        contentColor = contentColor,
        elevation = elevation,
        modifier = modifier,
        content = content
    )
}
```
还有标题栏内容区： 这个TopAppBar是官方提供的了
```Kotlin
@Composable
fun TopAppBarContent(
    title: @Composable () -> Unit,
    modifier: Modifier = Modifier,
    navigationIcon: @Composable (() -> Unit)? = null,
    actions: @Composable RowScope.() -> Unit = {},
) {
    TopAppBar(
        title = title,
        navigationIcon = navigationIcon,
        actions = actions,
        backgroundColor = Color.Transparent,
        elevation = 0.dp,
        modifier = modifier
    )
}
```

标题栏看完了，那就到内容区了。

```Kotlin

/**
 * 首页内容区
 */
@Composable
fun HomeContent(modifier: Modifier = Modifier, viewModel: HomeViewModel) {

    /**
     * list 为首页文章 列表数据；系统会对所有列表项进行组合和布局，无论它们是否可见，
     * 因此如果您需要显示大量列表项（或长度未知的列表），则使用 Column 等布局可能会导致性能问题。
     * Compose 提供了一组组件，这些组件只会对在组件视口中可见的列表项进行组合和布局。
     */
    val list = viewModel.viewStates.homeList.collectAsLazyPagingItems()

    /**
     * 下拉刷新组件
     */
    SwipeRefresh(
        /**
         * 状态，是否正在刷新
         */
        state = rememberSwipeRefreshState(viewModel.viewStates.isRefresh),
        /**
         * 下拉刷新回调，需要更新轮播+列表
         */
        onRefresh = {
            viewModel.dispatch(HomeAction.GetBanner)
            list.refresh()
        }) {
        // 内容区 使用LazyItem包裹
        PagingItem(modifier, list = list,
            errorBlock = {},
            successBlock = {},
            errorAndSuccessClick = {
                /**
                 * 错误后点击回调
                 */
                list.refresh()
            }) {
            /**
             * 仅组成和放置 当前可见项 的竖直滚动列表。它允许您放置不同类型的子级内容。
             * 例如，您可以使用LazyListScope.item添加单个项目，然后使用LazyListScope.items或者LazyListScope.itemsIndexed添加项目列表，后者有当前列表项的索引值。
             */
            LazyColumn{
                /**
                 * 单个item
                 */
                item {
                    BannerPager(Modifier.height(160.dp), viewModel = viewModel)
                }
                /**
                 * items集合
                 */
                items(it) { homeBean ->
                    homeBean?.apply {
                        /**
                         * 首页文字item
                         */
                        HomeListItem(homeBean = this) {
                            /**
                             * 处理点击收藏图标 触发viewModel层调用接口
                             */
                            if(CacheUtils.isLogin){
                                viewModel.dispatch(HomeAction.Collect(homeBean.id, it))
                            }
                            else{
                                CpNavigation.to(ModelPath.Login)
                            }
                        }
                    }
                }
            }
        }
    }
}
```
list 是专门用于懒加载列表，这个是存放可视区的列表。

数据来源是ViewModel层的viewStates实例。

```Kotlin
/**
 * 首页状态，包装首页需要的数据
 */
data class HomeState(
    /**
     * 是否正在刷新
     */
    val isRefresh : Boolean = false,

    /**
     * 轮播列表，数据源
     */
    val bannerList: List<BannerResponse> = listOf(),

    /**
     * 首页数据
     */
    val homeList: Flow<PagingData<ArticleBean>> = Pager(
        PagingConfig(PAGE_SIZE),
        pagingSourceFactory = { HomeSource() }).flow,
)
```
这个是存放到ViewModel层的首页动态数据。

```Kotlin
 // 首页数据监听 通过 mutableStateOf包装 HomeState
    var viewStates by mutableStateOf(
        HomeState(
            homeList = Pager(
                PagingConfig(PAGE_SIZE),
                pagingSourceFactory = { HomeSource() }).flow.cachedIn(viewModelScope)))
        private set
```
这里用一个mutableStateof包装下。


下面继续回到首页内容区：
然后是一个SwipeRefresh组件,官方提供的。
```Kotlin
    SwipeRefresh(
        /**
         * 状态，是否正在刷新
         */
        state = rememberSwipeRefreshState(viewModel.viewStates.isRefresh),
        /**
         * 下拉刷新回调，需要更新轮播+列表
         */
        onRefresh = {
            viewModel.dispatch(HomeAction.GetBanner)
            list.refresh()
        }) {
```
定义好下拉刷新触发事件。
注意这个list，实际上走的是LazyPagingItems里面的一个refresh方法。
我们在HomeState中定义的homeList是一个Flow包装列表集合。通过调用Flow中扩展的一个collectAsLazyPagingItems函数，就变成LazyPagingItems对象了。

这个list我们放在PagingItem里面。

然后首页内容区里面用了一个 LazyColumn，应该也是配合懒加载和复用官方实现的一个类。
然后里面第一个item就是我们的轮播图了。

```Kotlin
/**
 * 顶部轮播图
 */
@OptIn(ExperimentalPagerApi::class)
@Composable
fun BannerPager(modifier: Modifier = Modifier, viewModel: HomeViewModel) {
    val pagerState = rememberPagerState()

    /**
     * 开启协程 内部的闭包是协程 目的是获取轮播图数据
     */
    LaunchedEffect(viewModel){
        viewModel.dispatch(HomeAction.GetBanner)
    }

    /**
     * 如需了解来自父项的约束条件并相应地设计布局，您可以使用 BoxWithConstraints。
     * 就是外部传过来的modifier 决定子View 空间
     */
    BoxWithConstraints(modifier) {

        /**
         * 水平分页
         */
        HorizontalPager(
            count = viewModel.viewStates.bannerList.size, // 轮播总数
            state = pagerState // 轮播状态
        ) { page ->
            //  页面索引
            when (page) {
                viewModel.viewStates.bannerList[page].id -> {
                    /**
                     * 轮播图item
                     */
                    BannerItem(
                        Modifier
                            .height(maxHeight)
                            .fillMaxWidth(), viewModel.viewStates.bannerList[page]
                    )
                }
            }
        }

        /**
         * 轮播指示器
         */
        if (viewModel.viewStates.bannerList.isNotEmpty()) {
            BannerIndicator(
                Modifier
                    .align(Alignment.BottomCenter)
                    .fillMaxWidth()
                    .background(Color.Black.copy(0.2f))
                    .padding(8.dp),viewModel, pagerState )
        }


    }
}
```
这个轮播图里面开了个协程去获取轮播图数据，只会走一次。
然后一个水平分页里面存放所有轮播图，轮播item为BannerItem,这个是自定义的
```Kotlin
@Composable
fun BannerItem(modifier: Modifier = Modifier, data: BannerResponse) {
    data.apply {
        Box(modifier.clickable {
            /**
             * 点击图片，触发跳转WebView，通过bundle传递数据
             */
            CpNavigation.toBundle(ModelPath.WebView, Bundle().apply {
                putString(webTitle, data.title)
                putString(webUrl, data.url)
                putBoolean(webIsCollect, false)
                putInt(webCollectId, data.id)
                putInt(webCollectType, 1)
            })
        }) {
            /**
             * 自定义网络图片
             */
            NetworkImage(
                imagePath,
                Modifier.fillMaxWidth(),
                contentScale = ContentScale.FillBounds,
                defaultImg = R.mipmap.ic_default_round
            )
        }
    }
}
```

轮播完了，就是文章item了。
主要是为了实现这种效果
![](./%E7%8E%A9Android-Compose%E7%89%88%E6%9C%AC-%E9%A1%B9%E7%9B%AE%E5%88%86%E6%9E%90/home_item.jpeg)

```Kotlin
 /**
   * items集合
   */
items(it) { homeBean ->
    homeBean?.apply {
        /**
        * 首页文字item
        */
        HomeListItem(homeBean = this) {
            /**
            * 处理点击收藏图标 触发viewModel层调用接口
            */
            if(CacheUtils.isLogin){
                viewModel.dispatch(HomeAction.Collect(homeBean.id, it))
            }
            else{
                CpNavigation.to(ModelPath.Login)
            }
        }
    }
}
```
这里用了items，然后遍历了 list,每个item对应一个 HomeListItem
这里定义了一个函数，说明了点击收藏图标后的逻辑。

```Kotlin
  /**
     * 是否收藏了，用remember包装一下bool值
     */
    var isCollect by remember{ mutableStateOf(homeBean.collect)}

    /**
     * 卡片布局
     */
    Card(
        modifier
            .padding(8.dp) // 内padding 8个dp
            .clickable {
                /**
                 * 卡片点击事件，触发跳转webView
                 */
                CpNavigation.toBundle(ModelPath.WebView, Bundle().apply {
                    putString(webTitle, homeBean.title)
                    putString(webUrl, homeBean.link)
                    putBoolean(webIsCollect, isCollect)
                    putInt(webCollectId, homeBean.id)
                    putInt(webCollectType, 0)
                })
            },
        backgroundColor = HhfTheme.colors.listItem,
        elevation = 5.dp
    ) {
```
这里外层是用了Card,定义了点击item的逻辑。

里面是这样的：
```Kotlin
 Column(Modifier.padding(8.dp)) {
                /**
                 * 第一行 水平布局
                 */
                Row(verticalAlignment = Alignment.CenterVertically) {
                    /**
                     * 分享者没名字
                     */
                    Text(
                        text = if (author.isNotEmpty()) author else shareUser,
                        fontSize = 13.sp,
                        color = HhfTheme.colors.textColor
                    )
                    /**
                     * 显示标签
                     */
                    if(isShowLabel){
                        if (type == 1) {
                            Box(
                                Modifier
                                    .padding(start = 6.dp)
                                    .border(1.dp, Color.Red, RoundedCornerShape(5.dp))
                                    .padding(4.dp)
                            ) {
                                Text(stringResource(id = R.string.istop), Modifier, Color.Red, 10.sp)
                            }
                        }
                        if (fresh) {
                            Box(
                                Modifier
                                    .padding(start = 6.dp)
                                    .border(1.dp, Color.Red, RoundedCornerShape(5.dp))
                                    .padding(4.dp)
                            ) {
                                Text(stringResource(id = R.string.neww), Modifier, Color.Red, 10.sp)
                            }
                        }
                        if (tags.isNotEmpty()) {
                            Box(
                                Modifier
                                    .padding(start = 6.dp)
                                    .border(1.dp, Color(0xFF66BB6A), RoundedCornerShape(5.dp))
                                    .padding(4.dp)
                            ) {
                                Text(tags[0].name, Modifier, Color(0xFF66BB6A), 10.sp)
                            }
                        }
                    }
                    /**
                     * 发布日期
                     */
                    Text(
                        niceDate,
                        Modifier
                            .weight(1f)
                            .wrapContentWidth(
                                Alignment.End
                            )
                            .align(Alignment.CenterVertically),
                        HhfTheme.colors.textColor.copy(0.6f),
                        fontSize = 13.sp,
                    )
                }
                /**
                 * 有没有封面图
                 */
                if (TextUtils.isEmpty(envelopePic)) {
                    // 没有封面图
                    Text(
                        title.filterHtml(),
                        Modifier.padding(top = 12.dp),
                        HhfTheme.colors.textColor,
                        fontSize = 14.sp,
                        fontWeight = FontWeight.Bold
                    )
                } else {
                    // 有封面图时，中间区域展示
                    Row(Modifier.padding(top = 12.dp)) {
                        // 网络图片
                        NetworkImage(
                            envelopePic,
                            Modifier.size(100.dp),
                            contentScale = ContentScale.Crop,
                            defaultImg = R.mipmap.ic_default_round
                        )
                        // 垂直布局
                        Column(Modifier.padding(start = 8.dp)) {
                            // 标题
                            Text(
                                title.filterHtml(),
                                color = HhfTheme.colors.textColor,
                                fontSize = 14.sp,
                                maxLines = 3,
                                style = TextStyle(fontWeight = FontWeight.Bold)
                            )
                            // 摘要描述
                            Text(
                                desc,
                                color = HhfTheme.colors.textInactiveColor,
                                fontSize = 13.sp,
                                maxLines = 3,
                                overflow = TextOverflow.Ellipsis
                            )
                        }
                    }
                }
                // 底部水平布局
                Row {
                    // 文本
                    Text(
                        "$superChapterName / $chapterName",
                        Modifier.padding(top = 12.dp),
                        HhfTheme.colors.textColor,
                        fontSize = 13.sp
                    )
                    // 图标，是否收藏了
                    Icon(
                        if(isCollect) Icons.Filled.Favorite else Icons.Filled.FavoriteBorder, contentDescription = "",
                        Modifier
                            .wrapContentWidth(
                                Alignment.End
                            )
                            .weight(1f)
                            .clickable {
                                /**
                                 * 点击收藏，触发请求
                                 */
                                favoriteAction(isCollect)
                                isCollect = !isCollect
                            },
                        tint = HhfTheme.colors.themeColor
                    )
                }
            }
```

### 11.我的Tab-Mine
因为项目和公众号和首页基本一样，这里就不重复分析了。
看下我的页面的效果图：
<img src=mine.jpeg width=50%>
要实现这样的效果，怎么处理呢？

首先开启一个协程，获取用户sp数据，转成UserInfo对象
```Kotlin
   /**
     * 开启协程，只走一次
     */
    LaunchedEffect(CacheUtils.userInfo) {
        if (CacheUtils.userInfo != "") {
            /**
             * 根据sp文件，生产一个UserInfo对象，设置给这个我的页面使用
             */
            mineViewModel.dispatch(
                MineViewEvent.SetUserInfo(
                    Gson().fromJson(
                        CacheUtils.userInfo, // sp文件
                        UserInfo::class.java
                    )
                )
            )
            /**
             * 获取积分和排名
             */
            mineViewModel.dispatch(MineViewEvent.GetIntegral)
        }
    }
```
这里获取积分和排名是需要走接口的
这里会走到ViewModel层这个函数：
```Kotlin
    private fun getIntegral() {
        viewModelScope.launch {
            flow {
                emit(
                    TaskApi.create(ApiService::class.java).getIntegral()
                )
            }.map {
                if (it.errorCode == 0) {
                    it.data
                        ?: throw Exception("data null")
                } else {
                    throw Exception(it.errorMsg)
                }
            }.onEach {
                viewStates = viewStates.copy(integral = it)
            }.catch {
                viewStates = viewStates.copy(integral = null)
            }.collect()
        }
    }
```
这个retrofit中是这样定义的：
```Kotlin
interface ApiService {
    /**
     * 获取当前账户的个人积分
     */
    @GET("lg/coin/userinfo/json")
    suspend fun getIntegral(): ApiResponse<Integral>
```
这个就是一个suspend挂起函数。

顶层为Column:
```Kotlin
    /**
     * 顶层为列表视图
     */
    Column(modifier) {
        /**
         * 顶部👤
         */
        MineTopAvatar(
            Modifier
                .fillMaxWidth()
                .height(320.dp)
                .clip(QureytoImageShapes(160f)),
            mineViewModel.viewStates.userInfo?.run {
                nickname
            } ?: "avatar"
        ) {
```

头像区域，点击后可以弹pop，但是需要先获取权限，这样获取：
```Kotlin
// 图片点击事件 会去申请下权限
            XXPermissions
                .with(context)
                .permission(Permission.CAMERA)
                .permission(Permission.MANAGE_EXTERNAL_STORAGE)
                .request(object : OnPermissionCallback {
                    override fun onGranted(
                        granted: List<String>,
                        all: Boolean
                    ) {
                        if (all) {
                            if(isLogin){
                                mineViewModel.dispatch(MineViewEvent.ChangePopupState(true))
                            }
                            else{
                                CpNavigation.to(ModelPath.Login)
                            }
                        }
                    }

                    override fun onDenied(
                        denied: List<String>,
                        never: Boolean
                    ) {
                        if (never) {
                            context.showToast(context.stringResource(R.string.permissions_cm_error))
                            XXPermissions.startPermissionActivity(
                                context,
                                denied
                            )
                        } else {
                            context.showToast(context.stringResource(R.string.permissions_camera_error))
                        }
                    }
                })
```
用了一个三方库实现。

中间操作栏这样实现：
```Kotlin
  // 操作栏 有个外边框，包裹菜单项，底部有阴影
        Surface(
            Modifier
                .padding(start = 20.dp, end = 20.dp)
                .fillMaxWidth()
                .background(HhfTheme.colors.background),
            elevation = 2.dp,
            shape = RoundedCornerShape(8.dp),
            color = MaterialTheme.colors.surface, // color will be adjusted for elevation
        ) {
```

具体的菜单项也是用懒加载实现：
```Kotlin
LazyColumn(Modifier.background(HhfTheme.colors.listItem)) {
                itemsIndexed(list) { i, bean ->

                    /**
                     * 我的页面遍历菜单项
                     */
                    MineItem(
                        Modifier
                            .fillMaxWidth()
                            .clickable {
                                /**
                                 * 分发开发者行为
                                 */
                                mineViewModel.dispatch(MineViewEvent.ToComposable(bean.id))
                            }
                            .height(45.dp), bean.name, HhfTheme.colors.themeColor,
                        bean.icon)
                    if (i < list.size - 1) {
                        /**
                         * 非最后一行显示分割线
                         */
                        Divider(
                            modifier = Modifier
                                .fillMaxWidth()
                                .padding(start = 10.dp, end = 10.dp)
                        )
                    }
                }
            }
```

具体的菜单item,有图标+文字+右侧箭头实现：
```Kotlin

@Composable
fun MineItem(
    modifier: Modifier = Modifier, textName: String, iconColor: Color, icon: ImageVector
) {
    Row(
        modifier,
        verticalAlignment = Alignment.CenterVertically,
    ) {
        Icon(
            icon, "$textName icon",
            Modifier
                .size(28.dp)
                .padding(start = 10.dp), iconColor
        )
        Text(
            textName,
            Modifier.padding(start = 10.dp),
            fontSize = 14.sp,
            fontFamily = FontFamily.Serif,
            color = HhfTheme.colors.textColor
        )
        Icon(
            Icons.Filled.KeyboardArrowRight, textName,
            Modifier
                .weight(1f)
                .padding(end = 10.dp)
                .wrapContentWidth(Alignment.End),
            tint = HhfTheme.colors.textColor
        )
    }
}
```

UI搞定了，跳转逻辑怎么处理呢？
答案是viewModel层实现。

这里有个clickable点击闭包：
` mineViewModel.dispatch(MineViewEvent.ToComposable(bean.id))`


会委托给mineViewModel处理：
```Kotlin
    /**
     * 分发开发者行为
     */
    fun dispatch(action: MineViewEvent) {
        when (action) {
            is MineViewEvent.Blur -> bitmapBlur(action.s)
            /**
             * 用户点击菜单项
             */
            is MineViewEvent.ToComposable -> toComposable(action.type)
            is MineViewEvent.ChangePopupState -> {
                Log.e("TEST##", "这里触发了viewStates中 isShowpopup 变更为：${action.flag}")
                viewStates = viewStates.copy(isShowPopup = action.flag)
            }
            is MineViewEvent.SetUserInfo -> viewStates = viewStates.copy(userInfo = action.userInfo)
            is MineViewEvent.GetIntegral -> getIntegral()
        }
    }
```
这里分发开发者行为，其实更像用户行为，有可能不是用户触发，所以我这里称之为开发者行为。

点击菜单项继续分发：
```Kotlin
private fun toComposable(type: Int) {
        if(isLogin){
            when (type) {
                0 -> CpNavigation.to(ModelPath.Integral)
                1 -> CpNavigation.to(ModelPath.Collect)
                2 -> CpNavigation.to(ModelPath.Share)
                3 -> CpNavigation.to(ModelPath.Todo)
                4 -> CpNavigation.toBundle(ModelPath.WebView, Bundle().apply {
                    putString(webTitle, "PlayAndroid")
                    putString(webUrl, "https://github.com/yellowhai/PlayAndroid")
                    putBoolean(webIsCollect, false)
                    putInt(webCollectId, 998)
                    putInt(webCollectType, 1)
                })
                5 -> CpNavigation.to(ModelPath.Setting)
            }
        }
        else{
            when (type) {
                4 -> CpNavigation.toBundle(ModelPath.WebView, Bundle().apply {
                    putString(webTitle, "PlayAndroid")
                    putString(webUrl, "https://github.com/yellowhai/PlayAndroid")
                    putBoolean(webIsCollect, false)
                    putInt(webCollectId, 998)
                    putInt(webCollectType, 1)
                })
                5 -> CpNavigation.to(ModelPath.Setting)
                else -> CpNavigation.to(ModelPath.Login)
            }
        }
    }
```
这里定义了跳转逻辑，这里有回到 第8点主页外部架构 中的导航定义了。
实现跳转到其它页面了，这更像是在布局上叠加，不算是跳转吧。

这个项目还有很多细节值得一探究竟，不过对于初始compose，也差不多了，了解到这个程度也算入门了吧。


