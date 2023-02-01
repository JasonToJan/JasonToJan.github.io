---
title: Android google samples nowinandroid app模块流程分析
date: 2023-01-15 11:53:19
top: false
cover: false
toc: true
mathjax: true
tags:
- Android
categories:
- Android
---

> googe samples之 nowinandroid项目地址：[https://github.com/android/nowinandroid](https://github.com/android/nowinandroid)

### 1.app模块build.gradle文件配置
```Groovy
plugins {
    id("nowinandroid.android.application")
    id("nowinandroid.android.application.compose")
    id("nowinandroid.android.application.jacoco")
    id("nowinandroid.android.hilt")
    id("jacoco")
    id("nowinandroid.firebase-perf")
}
```
引入了application插件，compose插件，jacoco插件，hilt插件，firebase插件

依赖项：
```Groovy
    implementation(project(":feature:interests"))
    implementation(project(":feature:foryou"))
    implementation(project(":feature:bookmarks"))
    implementation(project(":feature:topic"))
    implementation(project(":feature:settings"))

    implementation(project(":core:common"))
    implementation(project(":core:ui"))
    implementation(project(":core:designsystem"))
    implementation(project(":core:data"))
    implementation(project(":core:model"))

    implementation(project(":sync:work"))
```
feature文件夹下是app模块的界面上拆分组件。
如feature_interests 就是兴趣tab
feature_foryou 就是第一个tab
feature_bookmarks 是书签模块
等等

下面的core就是核心模块了，有通用组件，ui组件，设计系统，数据区

然后sync_work是一些工具module了。

### 2.清单文件
```xml
    <application
        android:name=".NiaApplication"
        android:allowBackup="true"
        android:enableOnBackInvokedCallback="true"
        android:icon="@mipmap/ic_launcher"
        android:label="@string/app_name"
        android:roundIcon="@mipmap/ic_launcher_round"
        android:supportsRtl="true"
        android:theme="@style/Theme.Nia.Splash">
<!--        Profileable 是 Android 10 中引入的清单配置，可用于 CPU 和内存分析任务。使用 profileable 标志而不是 debuggable 标志具有降低性能开销的关键优势；-->
        <profileable android:shell="true" tools:targetApi="q" />

        <activity
            android:name=".MainActivity"
            android:exported="true">
            <intent-filter>
                <action android:name="android.intent.action.MAIN" />

                <category android:name="android.intent.category.LAUNCHER" />
            </intent-filter>
        </activity>
    </application>

```
这里很简单，就一个Application自定义的，一个首页，说明这个App就一个Activity。
这里引入了一个profileable标签，是Android10后引入的，可以用于CPU和内存分析任务。

### 3.自定义Application
这里App开始启动了。
首先是用了一个HiltAndroidApp的注解。
这个是类似Dragger的依赖注解框架，这里要求在Application用这个注解。
关于Hilt用法可参考：[https://developer.android.com/training/dependency-injection/hilt-android?hl=zh-cn](https://developer.android.com/training/dependency-injection/hilt-android?hl=zh-cn)

这里在Application里面初始化了一个东西：
`Sync.initialize(context = this)`
作用就是初始化Sync三方库。

里面是这样走的：
```Kotlin
  AppInitializer.getInstance(context)
            .initializeComponent(SyncInitializer::class.java)
```
AppInitializer是google的Jetpack下的startup中的类成员，关于这个用法可以参考下郭霖的博客：[https://blog.csdn.net/guolin_blog/article/details/108026357](https://blog.csdn.net/guolin_blog/article/details/108026357)

为什么要引入这个同步的库呢？
/**
 * Registers work to sync the data layer periodically on app startup.
 * 初始化三方库 这里是WorkManager，主要用于
 * Android出于设备电量的考虑，为开发者提供了WorkManager，旨在将一些不需要及时完成的任务交给它来完成。
 * 虽然WorkManager宣称，能够保证任务得到执行，但我在真实设备中，发现应用程序彻底退出与重启设备，任务都没有再次执行。
 * 查阅了相关资料，发现这应该与系统有关系。我们前面也提到了，WorkManager会根据系统的版本，
 * 决定采用JobScheduler或是AlarmManager+Broadcast Receivers来完成任务。
 * 但是这些API很可能会受到OEM系统的影响。比如，假设某个系统不允许AlarmManager自动唤起，那么WorkManager很可能就无法正常使用。
 */
 
 ### 4.MainActivity 参数定义
 Application走完，就到了Activity了。
 首先是两个注解：

1.  @OptIn(ExperimentalMaterial3WindowSizeClassApi::class)
 Opt-in 是 Kotlin 标准库中的一个方法，用于声明使用某些 API 需要明确的同意。
  该功能可以让开发者告知 API 使用者使用某些 API 需要一些特定的条件，如果使用者已经知晓则需要明确声明依旧使用（Opt-in）才能继续使用该 API。
  例如，某些 API 尚处于测试阶段，未来可能会发生变化；亦或是我前言中提到的场景，都非常适合使用该方法。
  如果我们声明了某个方法（functiuon）或类(class)需要 Opt-in ，则IDE或编译器会发出警告，要求使用者明确标注需要使用（Opt-in）。
  参考文档：[https://juejin.cn/post/7157950373250990093](https://juejin.cn/post/7157950373250990093)

2. @AndroidEntryPoint
在要使用依赖注入的类上方添加@AndroidEntryPoint注解
参考文档：[https://www.jianshu.com/p/22a36660a656](https://www.jianshu.com/p/22a36660a656)

然后声明了两个注入的变量：
```Kotlin
/**
     * Lazily inject [JankStats], which is used to track jank throughout the app.
     * JankStats 是首个专为在用户设备上检测及报告应用的性能问题而构建的 AndroidX 库。
     * JankStats 是占用空间相对较小的 API，主要有三大目标: 捕获每帧的性能信息、在用户设备 (不仅是开发设备) 上运行、以及在应用出现性能问题时启用检测，并报告所发生的情况
     *
     * 在Dagger中Lazy可以将要注入的依赖项转变为懒加载模式，这样注入的依赖项，只有在需要使用时，
     * 才会调用对应的依赖生成方法。懒加载的实现很简单，只需要在注入位置将依赖项的类型设置为Lazy接口的类型参数即可。
     *
     * 构造函数在di 文件夹下的 JankStatsModule 下
     */
    @Inject
    lateinit var lazyStats: dagger.Lazy<JankStats>

    /**
     * 自定义网络监视器 这个是一个接口 应该也是测试用的
     */
    @Inject
    lateinit var networkMonitor: NetworkMonitor

```
 
 关于lazyStats怎么初始化的？在项目di文件夹下有个JankStatsModule
 
```Kotlin
/**
 * 这里的目标是生产一个JankStats对象
 *
 * 怎么生产呢，需要2个构造参数
 * 1.JankStats.OnFrameListener
 * 2.Window
 *
 * window需要依赖activity，这里就需要 ActivityComponent了
 * 这个JankStats对象的生命周期设定为Activity级别
 */
@Module
@InstallIn(ActivityComponent::class)
object JankStatsModule {
    @Provides
    fun providesOnFrameListener(): JankStats.OnFrameListener {
        return JankStats.OnFrameListener { frameData ->
            // Make sure to only log janky frames.
            if (frameData.isJank) {
                // We're currently logging this but would better report it to a backend.
                Log.v("NiA Jank", frameData.toString())
            }
        }
    }

    @Provides
    fun providesWindow(activity: Activity): Window {
        return activity.window
    }

    @Provides
    fun providesJankStats(
        window: Window,
        frameListener: JankStats.OnFrameListener
    ): JankStats {
        return JankStats.createAndTrack(window, frameListener)
    }
}
```
这里生产了  一个JankStats对象。

关于网络监视器如何初始化的呢？
这个网络监视器是一个接口来的：
```Kotlin
/**
 * Utility for reporting app connectivity status
 */
interface NetworkMonitor {
    val isOnline: Flow<Boolean>
}

```
一直没找到如何实例化的地方。
不过再其它模块，core下的data.di文件夹下发现了另外一个DataModule下有如下代码：
```Kotlin

    @Binds
    fun bindsNetworkMonitor(
        networkMonitor: ConnectivityManagerNetworkMonitor
    ): NetworkMonitor
```
这里貌似可以生产一个NetworkMonitor对象。
但怎么关联起来的呢？

这个我确实不清楚hilt的用法，而且这个DataModule理应需要在Application配置，但也没找到相关依赖，可能hilt对加上这种注解能全局自动生成依赖代码也说不定。反正这里的网络监视器应该就是这个ConnectivityManagerNetworkMonitor 主要逻辑也在这里。

第三个变量是viewModel
` val viewModel: MainActivityViewModel by viewModels()`
这个用法类似：
`ViewModelProvider(this).get(MainActivityViewModel::class.java)`


### 5.首页生命周期-onCreate
继续走，第一行是 适配闪屏页 installSplashScreen()
关于闪屏页可参考文档：[https://blog.csdn.net/guolin_blog/article/details/120275319](https://blog.csdn.net/guolin_blog/article/details/120275319)（郭霖）

然后这里监听了一个密封接口中的 Loading 状态，
密封接口定义如下：
```Kotlin
/**
 * 首页ui 状态的密封类，有加载中和成功的状态
 * 密封接口 类似枚举类型，说明有Loading状态和成功状态
 * 表示受限制的类层次结构
 */
sealed interface MainActivityUiState {
    object Loading : MainActivityUiState
    data class Success(val userData: UserData) : MainActivityUiState
}
```
很明显这里Loading实现了这个接口，Success也实现了这个接口。

在onCreate监听了这个Loading类：
```Kotlin

        /**
         * mutableStateOf 表明某个变量是有状态的，对变量进行监听，当状态改变时，触发重绘。
         * remember --- 记录变量的值，使得下次使用改变量时不进行初始化。
         * 使用 remember 存储对象的可组合项会创建内部状态，使该可组合项有状态。
         * remember 会为函数提供存储空间，将 remember 计算的值储存，当 remember 的键改变的时候会进行重新计算值并储存。
         * rememberSaveable --- 在这里顺带提一下,rememberSaveable 可以在重组后保持状态,也可以在重新创建 activity 和进程后保持状态。
         */
        var uiState: MainActivityUiState by mutableStateOf(Loading)
```

然后开启一个携程，拿用户数据：
```Kotlin
 // Activity生命周期内使用携程
        lifecycleScope.launch {
            // Lifecycle.State.STARTED: 表示只有在Activity处于Started状态的情况下，协程中的代码才会执行。
            lifecycle.repeatOnLifecycle(Lifecycle.State.STARTED) {
                viewModel.uiState
                    .onEach {
                        uiState = it
                    }
                    .collect() // 走collect方法才代表执行携程
            }
        }
```
拿到数据后，才回去更新uiState对象。

就是说uiState会有上面密封接口定义的两种类型。

当监听到uiState变化后会走：
```Kotlin
 /**
         * 如果网络请求正在请求loading状态，就还是显示闪屏，如果拿到数据了就不需要闪屏了
         */
        splashScreen.setKeepOnScreenCondition {
            when (uiState) {
                Loading -> true
                is Success -> false
            }
        }
```
闪屏会去除掉。

下面开启沉浸式状态栏：
```Kotlin
  /**
         * 第二参数decorFitsSystemWindows表示是否沉浸，false 表示沉浸，true表示不沉浸
         */
        WindowCompat.setDecorFitsSystemWindows(window, false)
```

然后就是内容区，compose设置内容区了
一个setContent的函数，太长了，就不全部贴图，挨个分析。

1.首先获取一个systemUIController
```Kotlin
 /**
             * 开发者若想在 Compose 布局中控制 System UI，就必须获取 SystemUiController 对象。
             * 通过该库提供 rememberSystemUiController 函数，开发者可以获取当前操作系统（目前仅支持 Android 系统）的  SystemUiController 对象。
             */
            val systemUiController = rememberSystemUiController()
```

2.是否使用深色主题，用户可以自己选
```Kotlin
 /**
             * 是否使用深色主题，入参当前ui状态
             */
            val darkTheme = shouldUseDarkTheme(uiState)
            
```

内部实现：
```Kotlin
/**
 * Returns `true` if dark theme should be used, as a function of the [uiState] and the
 * current system context.
 * 调用@Composable 函数的函数必须用@Composable 注解标记
 */
@Composable
private fun shouldUseDarkTheme(
    uiState: MainActivityUiState,
): Boolean = when (uiState) {
    /**
     * isSystemInDarkTheme 根据系统来，加载中状态
     */
    Loading -> isSystemInDarkTheme()
    is Success -> when (uiState.userData.darkThemeConfig) {
        /**
         * 成功状态下，根据用户配置的数据来展示，如果跟随系统那就跟随，如果用户主动设置了某一种，那就成功时选择
         */
        DarkThemeConfig.FOLLOW_SYSTEM -> isSystemInDarkTheme()
        DarkThemeConfig.LIGHT -> false
        DarkThemeConfig.DARK -> true
    }
}
```

然后开启一个DisposableEffect来关联上面这两个对象：
```Kotlin
  /**
             * disposable，顾名思义，这就是一个自带清理作用的Effect
             */
            DisposableEffect(systemUiController, darkTheme) {
                // 否与导航栏图标+内容是否“黑暗"
                systemUiController.systemBarsDarkContentEnabled = !darkTheme
                onDispose {}
            }
```

最后，开始设置App主题了：
```Kotlin
  /**
             * 主题配置，是否暗色，判断用户是否有自己选择
             * 是否用Android主题
             * 下面这些代码才是内容区的布局，上面MainActivityUiState 这个只是拿用户的持久化数据，槽了
             */
            NiaTheme(
                darkTheme = darkTheme,
                androidTheme = shouldUseAndroidTheme(uiState)
            ) {
                // NiaApp是一个闭包，作为第三个参数
                NiaApp(
                    /**
                     * 窗口大小类，获取一个WindowSizeClass 这个实例
                     */
                    windowSizeClass = calculateWindowSizeClass(this),
                    /**
                     * 网络监视器，前面注入的
                     */
                    networkMonitor = networkMonitor,
                )
            }
```

里面的闭包 NiaApp也就是开启了我们的App。

### 6.首页其它生命周期
```Kotlin
  override fun onResume() {
        super.onResume()
        /**
         * 跟踪性能
         */
        lazyStats.get().isTrackingEnabled = true
    }

    override fun onPause() {
        super.onPause()
        /**
         * 去除跟踪
         */
        lazyStats.get().isTrackingEnabled = false
    }
```

MainActivity内容就这么多了，非常指简洁。
布局交给NiaApp类处理了。

其它对象初始化交给hilt处理了。

主要就只开了个协程获取用户数据，而且优化闪屏页和loading的显示。

### 7.NiaTheme主题设置
先是主题颜色：
```Kotlin
 // 主题颜色
    val colorScheme = when {
        // 判断当前 android主题是暗色还是浅色
        androidTheme -> if (darkTheme) DarkAndroidColorScheme else LightAndroidColorScheme
        // 没有禁用动态主题并且支持动态主题
        !disableDynamicTheming && supportsDynamicTheming() -> {
            val context = LocalContext.current
            if (darkTheme) dynamicDarkColorScheme(context) else dynamicLightColorScheme(context)
        }
        // 其它情况
        else -> if (darkTheme) DarkDefaultColorScheme else LightDefaultColorScheme
    }
```

渐变色:
```Kotlin
    // Gradient colors 渐变色
    // 空渐变色
    val emptyGradientColors = GradientColors(container = colorScheme.surfaceColorAtElevation(2.dp))
    // 默认渐变色
    val defaultGradientColors = GradientColors(
        top = colorScheme.inverseOnSurface,
        bottom = colorScheme.primaryContainer,
        container = colorScheme.surface
    )
    // 渐变色
    val gradientColors = when {
        androidTheme -> if (darkTheme) DarkAndroidGradientColors else LightAndroidGradientColors
        !disableDynamicTheming && supportsDynamicTheming() -> emptyGradientColors
        else -> defaultGradientColors
    }
```

最终设置到MaterialTheme里面：
```Kotlin
   // Composition locals
    CompositionLocalProvider(
        LocalGradientColors provides gradientColors,
        LocalBackgroundTheme provides backgroundTheme
    ) {
        MaterialTheme(
            colorScheme = colorScheme,
            typography = NiaTypography,
            content = content
        )
    }
```

### 8.NiaApp状态定义

首先创建一个NiaApp实例。
入参：窗口大小类，网络监视器
```Kotlin
 NiaApp(
                    /**
                     * 窗口大小类，获取一个WindowSizeClass 这个实例
                     */
                    windowSizeClass = calculateWindowSizeClass(this),
                    /**
                     * 网络监视器，前面注入的
                     */
                    networkMonitor = networkMonitor,
                )
```

这里是App首页ui样式。

这里用到的状态作为第三个参数，作为app状态记录。

函数定义如下：
```Kotlin
@OptIn(
    ExperimentalMaterial3Api::class,
    ExperimentalLayoutApi::class,
    ExperimentalComposeUiApi::class,
    ExperimentalLifecycleComposeApi::class
)
@Composable
fun NiaApp(
    windowSizeClass: WindowSizeClass,
    networkMonitor: NetworkMonitor,
    /**
     * app状态记录
     */
    appState: NiaAppState = rememberNiaAppState(
        networkMonitor = networkMonitor,
        windowSizeClass = windowSizeClass
    ),
) 
```
这里第三个参数已经默认实现了。

就是用于app状态记录，调用了一个自定义的rememberNiaAppState方法。
```Kotlin
@Composable
fun rememberNiaAppState(
    windowSizeClass: WindowSizeClass,
    networkMonitor: NetworkMonitor,
    coroutineScope: CoroutineScope = rememberCoroutineScope(),
    /**
     * 导航控制器
     */
    navController: NavHostController = rememberNavController()
): NiaAppState {
    NavigationTrackingSideEffect(navController)
    /**
     * remember 里面参数可以任意，闭包里面的东西，就是remember返回的东西
     */
    return remember(navController, coroutineScope, windowSizeClass, networkMonitor) {
        NiaAppState(navController, coroutineScope, windowSizeClass, networkMonitor)
    }
}
```
这里默认实现了协程作用域，导航控制器，需要返回一个NiaAppState变量。

这里通过remember闭包来返回这个变量。

这个NiaAppState定义如下：
```Kotlin
/**
 * 主要用于类似LiveData保存的数据
 * App状态记录，需要传入导航控制器，携程作用范围，window窗口大小
 */
@Stable
class NiaAppState(
    val navController: NavHostController,
    val coroutineScope: CoroutineScope,
    val windowSizeClass: WindowSizeClass,
    networkMonitor: NetworkMonitor,
) {
```
类似LiveData，保存了首页必要的一些数据。

```Kotlin
 /**
     * 当前在那个tab
     */
    val currentDestination: NavDestination?
        @Composable get() = navController
            .currentBackStackEntryAsState().value?.destination
```

```Kotlin
 /**
     * 通过在哪个tab，决定用哪个枚举类， TopLevelDestination是自己定义的枚举类
     */
    val currentTopLevelDestination: TopLevelDestination?
        @Composable get() = when (currentDestination?.route) {
            forYouNavigationRoute -> FOR_YOU
            bookmarksRoute -> BOOKMARKS
            interestsRoute -> INTERESTS
            else -> null
        }
```

```Kotlin
 /**
     * 是否展示设置弹框
     */
    var shouldShowSettingsDialog by mutableStateOf(false)
        private set
```

```Kotlin
 /**
     * 是否展示底部bar
     */
    val shouldShowBottomBar: Boolean
        get() = windowSizeClass.widthSizeClass == WindowWidthSizeClass.Compact ||
            windowSizeClass.heightSizeClass == WindowHeightSizeClass.Compact
```

```Kotlin
  /**
     * 是否展示水平左侧导航
     */
    val shouldShowNavRail: Boolean
        get() = !shouldShowBottomBar
```

```Kotlin
 /**
     * 监控网络，是否断网
     */
    val isOffline = networkMonitor.isOnline
        .map(Boolean::not)
        .stateIn(
            scope = coroutineScope,
            started = SharingStarted.WhileSubscribed(5_000),
            initialValue = false
        )

```

```Kotlin
   /**
     * 这里是一个枚举类，表示底部tab集合
     * Map of top level destinations to be used in the TopBar, BottomBar and NavRail. The key is the
     * route.
     */
    val topLevelDestinations: List<TopLevelDestination> = TopLevelDestination.values().asList()
```

```Kotlin
 /**
     * 点击tab事件传递
     *
     * @param topLevelDestination: The destination the app needs to navigate to.
     */
    fun navigateToTopLevelDestination(topLevelDestination: TopLevelDestination) {
        trace("Navigation: ${topLevelDestination.name}") {
            val topLevelNavOptions = navOptions {
                // Pop up to the start destination of the graph to
                // avoid building up a large stack of destinations
                // on the back stack as users select items
                popUpTo(navController.graph.findStartDestination().id) {
                    saveState = true
                }
                // Avoid multiple copies of the same destination when
                // reselecting the same item
                launchSingleTop = true
                // Restore state when reselecting a previously selected item
                restoreState = true
            }

            when (topLevelDestination) {
                FOR_YOU -> navController.navigateToForYou(topLevelNavOptions)
                BOOKMARKS -> navController.navigateToBookmarks(topLevelNavOptions)
                INTERESTS -> navController.navigateToInterestsGraph(topLevelNavOptions)
            }
        }
    }

```

然后定义了一些方法：
```Kotlin
 /**
     * 返回点击
     */
    fun onBackClick() {
        navController.popBackStack()
    }

    /**
     * 改变设置弹框状态
     */
    fun setShowSettingsDialog(shouldShow: Boolean) {
        shouldShowSettingsDialog = shouldShow
    }
```

### 9.NiaApp内容区

App背景色
```Kotlin
@Composable
fun NiaBackground(
    modifier: Modifier = Modifier,
    content: @Composable () -> Unit
) {
    val color = LocalBackgroundTheme.current.color
    val tonalElevation = LocalBackgroundTheme.current.tonalElevation
    Surface(
        color = if (color == Color.Unspecified) Color.Transparent else color,
        tonalElevation = if (tonalElevation == Dp.Unspecified) 0.dp else tonalElevation,
        modifier = modifier.fillMaxSize(),
    ) {
        CompositionLocalProvider(LocalAbsoluteTonalElevation provides 0.dp) {
            content()
        }
    }
}
```

数据源使用 LocalBackgroundTheme.current.color

内容区继续渐变背景：
```Kotlin
@Composable
fun NiaGradientBackground(
    modifier: Modifier = Modifier,
    gradientColors: GradientColors = LocalGradientColors.current,
    content: @Composable () -> Unit
) {
    val currentTopColor by rememberUpdatedState(gradientColors.top)
    val currentBottomColor by rememberUpdatedState(gradientColors.bottom)
    Surface(
        color = if (gradientColors.container == Color.Unspecified) {
            Color.Transparent
        } else {
            gradientColors.container
        },
        modifier = modifier.fillMaxSize()
    ) {
        Box(
            Modifier
                .fillMaxSize()
                .drawWithCache {
                    // Compute the start and end coordinates such that the gradients are angled 11.06
                    // degrees off the vertical axis
                    val offset = size.height * tan(
                        Math
                            .toRadians(11.06)
                            .toFloat()
                    )

                    val start = Offset(size.width / 2 + offset / 2, 0f)
                    val end = Offset(size.width / 2 - offset / 2, size.height)

                    // Create the top gradient that fades out after the halfway point vertically
                    val topGradient = Brush.linearGradient(
                        0f to if (currentTopColor == Color.Unspecified) {
                            Color.Transparent
                        } else {
                            currentTopColor
                        },
                        0.724f to Color.Transparent,
                        start = start,
                        end = end,
                    )
                    // Create the bottom gradient that fades in before the halfway point vertically
                    val bottomGradient = Brush.linearGradient(
                        0.2552f to Color.Transparent,
                        1f to if (currentBottomColor == Color.Unspecified) {
                            Color.Transparent
                        } else {
                            currentBottomColor
                        },
                        start = start,
                        end = end,
                    )

                    onDrawBehind {
                        // There is overlap here, so order is important
                        drawRect(topGradient)
                        drawRect(bottomGradient)
                    }
                }
        ) {
            content()
        }
    }
}
```
这里在上面0到0.724区间设置了topGradient
在0.2552f到1设置了 bottomGradient

### 10.业务内容区

```Kotlin
   // 记录下snackbar状态，后续可能会刷新
            val snackbarHostState = remember { SnackbarHostState() }
```
这个用于网络监听，如果没有网络，底部会常驻一个snackbar

```Kotlin
            // 是否离线
            val isOffline by appState.isOffline.collectAsStateWithLifecycle()

            // If user is not connected to the internet show a snack bar to inform them.
            // 如果没有网络，底部展示Snackbar
            val notConnectedMessage = stringResource(R.string.not_connected)
            // 这个是Compose里面的launch，compose开启携程的方法
            LaunchedEffect(isOffline) {
                if (isOffline) snackbarHostState.showSnackbar(
                    message = notConnectedMessage,
                    duration = Indefinite
                )
            }
```

Kotlin
            // 是否应该展示设置弹框，当点击右上角设置后，会弹出设置弹框
            if (appState.shouldShowSettingsDialog) {
                SettingsDialog(
                    onDismiss = { appState.setShowSettingsDialog(false) }
                )
            }
```

然后就是脚手架了：
Scaffold，这里可以配置底部bar
```Kotlin
  Scaffold(
                modifier = Modifier.semantics {
                    testTagsAsResourceId = true
                },
                containerColor = Color.Transparent,
                contentColor = MaterialTheme.colorScheme.onBackground,
                contentWindowInsets = WindowInsets(0, 0, 0, 0),
                snackbarHost = { SnackbarHost(snackbarHostState) },
                // 底部bar 设置destinations，就是底部tab数据
                bottomBar = {
                    if (appState.shouldShowBottomBar) {
                        NiaBottomBar(
                            destinations = appState.topLevelDestinations,
                            onNavigateToDestination = appState::navigateToTopLevelDestination,
                            currentDestination = appState.currentDestination,
                            modifier = Modifier.testTag("NiaBottomBar")
                        )
                    }
                }
            )
```

这里snackbarHost也是脚手架里面的内容。

然后里面是一个Row，为什么是一个水平布局，因为考虑到兼容电视，平板，这里将导航页设置了一个水平的左侧导航。
```Kotlin
  // 平板 水平布局 左侧导航
                    if (appState.shouldShowNavRail) {
                        NiaNavRail(
                            destinations = appState.topLevelDestinations,
                            onNavigateToDestination = appState::navigateToTopLevelDestination,
                            currentDestination = appState.currentDestination,
                            modifier = Modifier
                                .testTag("NiaNavRail")
                                .safeDrawingPadding()
                        )
                    }
```

然后就是实际内容区域了：
设置+主页构成。
```Kotlin
// 垂直布局，状态栏+内容区
                    /**
                     * 以前，我们在布局中去设置一个控件的大小，间距，点击事件，宽高，背景等属性值。
                     * 而在Compose中我们是通过Modifie去设置，Modifie相当于一个控件的属性配置的工具类。
                     */
                    Column(Modifier.fillMaxSize()) {
                        // Show the top app bar on top level destinations.
                        val destination = appState.currentTopLevelDestination
                        if (destination != null) {
                            NiaTopAppBar(
                                titleRes = destination.titleTextId,
                                actionIcon = NiaIcons.Settings,
                                actionIconContentDescription = stringResource(
                                    id = settingsR.string.top_app_bar_action_icon_description
                                ),
                                colors = TopAppBarDefaults.centerAlignedTopAppBarColors(
                                    containerColor = Color.Transparent
                                ),
                                /**
                                 * 设置点击事件触发appState中的 mutableStateOf变量发生更改
                                 */
                                onActionClick = { appState.setShowSettingsDialog(true) }
                            )
                        }

                        // 主页面
                        NiaNavHost(
                            navController = appState.navController,
                            onBackClick = appState::onBackClick
                        )
                    }
```

NiaTopAppBar是自定义顶部标题栏
NiaNavHost是主页面

主页面也是在App模块下的navigation文件夹中定义。

```Kotlin
@Composable
fun NiaNavHost(
    /**
     * 导航控制器
     */
    navController: NavHostController,
    /**
     * 返回点击
     */
    onBackClick: () -> Unit,
    /**
     * 属性配置工具
     */
    modifier: Modifier = Modifier,
    /**
     * 开始路由设置
     */
    startDestination: String = forYouNavigationRoute
) {
    NavHost(
        navController = navController,
        startDestination = startDestination,
        modifier = modifier,
    ) {
        forYouScreen()
        bookmarksScreen()
        interestsGraph(
            navigateToTopic = { topicId ->
                navController.navigateToTopic(topicId)
            },
            nestedGraphs = {
                topicScreen(onBackClick)
            }
        )
    }
}
```
这里使用了一个NavHost 的compose官方组件。

tab实现为：
forYouScreen 你的模块
bookmarksScreen 书签模块
interestsGraph 兴趣模块

这几个tab都是依赖于其它module了。
app自身module大致内容就这么多了。


