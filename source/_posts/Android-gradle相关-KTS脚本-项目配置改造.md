---
title: Android gradle相关 KTS脚本 项目配置改造
top: false
cover: false
toc: true
mathjax: true
date: 2023-01-13 13:39:50
tags:
- Android
categories:
- Android
---

> googe samples之 nowinandroid项目地址：[https://github.com/android/nowinandroid](https://github.com/android/nowinandroid)

### 1.gradle.properties
这里面配置了 android.useAndroidX等常用的配置，这里应该耳熟能详了，这里就不过多介绍。下面这里有一块不是特别熟悉：
```
# Disable build features that are enabled by default,
# https://developer.android.com/studio/releases/gradle-plugin#buildFeatures
android.defaults.buildfeatures.buildconfig=false
android.defaults.buildfeatures.aidl=false
android.defaults.buildfeatures.renderscript=false
android.defaults.buildfeatures.resvalues=false
android.defaults.buildfeatures.shaders=false
```

通过关闭buildfeatures.buildconfig，来提升构建性能。有时候模块不需要生成buildconfig文件，那直接关闭即可。还有其它的aidl，renderscript，resvalues同理。


### 2.项目settings.gradle.kts
这里首先是一个pluginManagement闭包：
```
pluginManagement {
    includeBuild("build-logic") // 多模块共享模块
    repositories {
        google()
        mavenCentral()
        gradlePluginPortal()
    }
}
```
第一个问题是：pluginManagement是什么东西啊？没见过。
简单来说：dependencyManagement用于在父项目中统一对子项目依赖管理。这样就可以对所有模块的版本进行管理。

第二个问题是：includeBuild是什么？
相信大部分人都有用过buildSrc吧。如果没有，那config.gradle总用过吧，总之就是把不同模块之前相同依赖抽出来，同时引用。 buildSrc的弊端就是每次依赖更新都将重新构建整个项目，导致编译时间很长，这里google官方推荐使用includeBuild来解决这个问题。

然后gradlePluginProtal() 远程插件仓库，是google推荐的。

第二块代码是这样的：
```
dependencyResolutionManagement {
    repositoriesMode.set(RepositoriesMode.FAIL_ON_PROJECT_REPOS)
    repositories {
        google()
        mavenCentral()
    }
}
```

当您的依赖项不是本地库或文件树时，Gradle 会在 settings.gradle 文件的 dependencyResolutionManagement { repositories {...} } 代码块中指定的所有在线仓库中查找相关文件。 各个仓库的列出顺序决定了 Gradle 在这些仓库中搜索各个项目依赖项的顺序。例如，如果从仓库 A 和 B 均可获得某个依赖项，而您先列出了仓库 A，则 Gradle 会从仓库 A 下载该依赖项。说白了，就是有一个优先权，从上往下。

然后这个文件其它内容没啥参考价值了，都是include多个模块。
这个文件最重要也是最关键的地方就是 includeBuild("build-logic")


### 3.项目级build.gradle.kts
首先是一个buildscript闭包：
```
buildscript {
    repositories {
        maven { url = uri("https://maven.aliyun.com/repository/public") }
        maven { url = uri("https://maven.aliyun.com/repository/gradle-plugin") }
        maven { url = uri("https://maven.aliyun.com/repository/central") }
        maven { url = uri("https://maven.aliyun.com/repository/google") }

        mavenCentral()
        google()


        // Android Build Server
        maven { url = uri("../nowinandroid-prebuilts/m2repository") }
    }
}
```
其实这里引发了另外一个问题：
在项目根目录（root Project）下的build.gradle中：buildscript、allprojects、subprojects的repositories有什么区别？

1. buildScript块的repositories主要是为了Gradle脚本自身的执行，获取脚本依赖插件。也就是说，buildScript是用来加载Gradle脚本自身需要使用的资源，可以声明的资源包括依赖项、第三方插件、maven仓库地址等。
2. 根级别的repositories主要是为了当前项目提供所需依赖包，比如log4j、spring-core等依赖包可从mavenCentral仓库获得。
3. allprojects块的repositories用于多项目构建，为所有项目提供共同的所需依赖包。而子项目可以配置自己的repositories以获取自己独需的依赖包。
4. subprojects块的repositories用于配置这个项目的子项目。使用多模块项目时，不同模块之间有相同的配置，导致重复配置，可以将相同的部分抽取出来，使用配置注入的技术完成子项目的配置。根项目就像一个容器, subprojects 方法遍历这个容器的所有元素并且注入指定的配置。allprojects是对所有project的配置，包括Root Project。而subprojects是对所有Child Project的配置。

这里说明buildScript里面的仓库主要用于Gradle脚本自身执行，因为这里一直超时，我把阿里云镜像加进去就没问题了，估计是下载Gradle脚本自身相关依赖导致超时了。

然后下面还有一部分是plugins闭包：
```
plugins {
    alias(libs.plugins.android.application) apply false
    alias(libs.plugins.kotlin.jvm) apply false
    alias(libs.plugins.kotlin.serialization) apply false
    alias(libs.plugins.hilt) apply false
    alias(libs.plugins.secrets) apply false
}
```
这里的libs就是在 前面的build-logic模块下的 versionCatalogs创建的一个名称。
定义了后，这里就可以引用了。这里应该是引用插件，不过都是apply false，这里的作用个人还不是很清楚。

### 4.app模块下的build.gradle.kts
这里第一行是：
`import com.google.samples.apps.nowinandroid.NiaBuildType`
这里可以引用自己编写的类，
这个类定义再build-logic模块中，是一个枚举类，看一下吧：
```
package com.google.samples.apps.nowinandroid

/**
 * This is shared between :app and :benchmarks module to provide configurations type safety.
 */
@Suppress("unused")
enum class NiaBuildType(val applicationIdSuffix: String? = null) {
    DEBUG(".debug"),
    RELEASE,
    BENCHMARK(".benchmark")
}

```
估计是代表编译模式，debug还是release，或者其它模式。

然后继续回到build.gradle.kts文件下，
这里引入插件了：
```
plugins {
    id("nowinandroid.android.application")
    id("nowinandroid.android.application.compose")
    id("nowinandroid.android.application.jacoco")
    id("nowinandroid.android.hilt")
    id("jacoco")
    id("nowinandroid.firebase-perf")
}
```

注意这几个有nowinandroid前缀的应该都是我们自己编写的插件。
首先看下第一个：noewinandroid.android.application

当然还是先全局搜索下，发现在build-logic模块下注册了：
```
gradlePlugin {
    plugins {
        register("androidApplicationCompose") {
            id = "nowinandroid.android.application.compose"
            implementationClass = "AndroidApplicationComposeConventionPlugin"
        }
        register("androidApplication") {
            id = "nowinandroid.android.application"
            implementationClass = "AndroidApplicationConventionPlugin"
        }
```
实现类为：AndroidApplicationConventionPlugin
```
class AndroidApplicationConventionPlugin : Plugin<Project> {
    override fun apply(target: Project) {
        with(target) {
            with(pluginManager) {
                apply("com.android.application")
                apply("org.jetbrains.kotlin.android")
            }

            extensions.configure<ApplicationExtension> {
                configureKotlinAndroid(this)
                defaultConfig.targetSdk = 33
                configureFlavors(this)
            }
            extensions.configure<ApplicationAndroidComponentsExtension> {
                configurePrintApksTask(this)
            }
        }
    }
}
```
这里继承了Plugin<Project>，实现apply方法。
利用插件管理器pluginManager来引入"com.android.application"插件和“org.jetbrains.kotlin.android”插件。

可以看到这里面帮忙apply了com.android.application，这样就不用重复再build.gradle中引入了。

重点来看下引入的compose，说实话我还没用过google的jetpack compose工具包。
依然在build-logic下注册了这个插件：
```
gradlePlugin {
    plugins {
        register("androidApplicationCompose") {
            id = "nowinandroid.android.application.compose"
            implementationClass = "AndroidApplicationComposeConventionPlugin"
        }
```
实现类：AndroidApplicationComposeConventionPlugin
```
class AndroidApplicationComposeConventionPlugin : Plugin<Project> {
    override fun apply(target: Project) {
        with(target) {
            pluginManager.apply("com.android.application")
            val extension = extensions.getByType<ApplicationExtension>()
            configureAndroidCompose(extension)
        }
    }

}
```
非常简单，先apply了com.android.application
然后调用了一个 configureAndroidCompose 方法，这是一个扩展方法，也是在build-logic下的一个kt文件中对Project进行了扩展。
```
/**
 * Configure Compose-specific options
 */
internal fun Project.configureAndroidCompose(
    commonExtension: CommonExtension<*, *, *, *>,
) {
    val libs = extensions.getByType<VersionCatalogsExtension>().named("libs")

    commonExtension.apply {
        buildFeatures {
            compose = true
        }

        composeOptions {
            kotlinCompilerExtensionVersion = libs.findVersion("androidxComposeCompiler").get().toString()
        }

        kotlinOptions {
            freeCompilerArgs = freeCompilerArgs + buildComposeMetricsParameters()
        }

        dependencies {
            val bom = libs.findLibrary("androidx-compose-bom").get()
            add("implementation", platform(bom))
            add("androidTestImplementation", platform(bom))
        }
    }
}
```
这里编写了compose相关的必要的配置。buildFeaturs，composeOptions,kotlinOptions,dependencies这些都加上了。

再回到app模块下的build.gradle.kts文件，
还有一个插件是：nowinandroid.android.application.jacoco
这个是自定义的一个代码覆盖率的一个插件，基于jacoco的。
```
 register("androidApplicationJacoco") {
            id = "nowinandroid.android.application.jacoco"
            implementationClass = "AndroidApplicationJacocoConventionPlugin"
        }
```
先注册，再看下实现类：
```

class AndroidApplicationJacocoConventionPlugin : Plugin<Project> {
    override fun apply(target: Project) {
        with(target) {
            with(pluginManager) {
                apply("org.gradle.jacoco")
                apply("com.android.application")
            }
            val extension = extensions.getByType<ApplicationAndroidComponentsExtension>()
            configureJacoco(extension)
        }
    }

}
```
这里引入了gradle的jacoco插件，然后调用了configureJacoco方法。
具体插件使用方法可以这里看下：[https://www.w3cschool.cn/gradle/9pg71huj.html](https://www.w3cschool.cn/gradle/9pg71huj.html)

这个配置起来确实很麻烦，如果全部代码加到app模块下，很难进行维护，这样以插件形式确实方便很多。

这里继续看app模块引入的自定义插件：hilt
这个想必大家都不陌生了，是一个依赖注入的插件，基于Dragger，但比Dragger简洁很多了。
实现类为：
```
class AndroidHiltConventionPlugin : Plugin<Project> {
    override fun apply(target: Project) {
        with(target) {
            with(pluginManager) {
                apply("dagger.hilt.android.plugin")
                // KAPT must go last to avoid build warnings.
                // See: https://stackoverflow.com/questions/70550883/warning-the-following-options-were-not-recognized-by-any-processor-dagger-f
                apply("org.jetbrains.kotlin.kapt")
            }

            val libs = extensions.getByType<VersionCatalogsExtension>().named("libs")
            dependencies {
                "implementation"(libs.findLibrary("hilt.android").get())
                "kapt"(libs.findLibrary("hilt.compiler").get())
                "kaptAndroidTest"(libs.findLibrary("hilt.compiler").get())
            }

        }
    }
}
```
这样就配置好了hilt。

最后一个app模块引入的插件是firebase-perf
这个是google出品的性能监控插件。
具体用法参考：[https://blog.csdn.net/u010207898/article/details/125070024](https://blog.csdn.net/u010207898/article/details/125070024)


* * *
好的下面继续看app模块下的build.gradle.kts
android层级下基本没啥大问题，配置applicationId版本号之类的。
这里有个testInstrumentationRunner选择自定义了：
`testInstrumentationRunner = "com.google.samples.apps.nowinandroid.core.testing.NiaTestRunner"`

看下这里做了啥：
可以看到这个路径下是在testing模块下的，那应该就是测试相关的了。这里主要是用于单元测试的。这个NiaTestRunner继承了AndroidJunitRunner，这里代码如下：
```
class NiaTestRunner : AndroidJUnitRunner() {
    override fun newApplication(cl: ClassLoader?, name: String?, context: Context?): Application {
        return super.newApplication(cl, HiltTestApplication::class.java.name, context)
    }
}
```

这个hilt居然还有做单元测试的注入，还是不错，收藏了。

看下buildTypes里面的配置：
```
   buildTypes {
        val debug by getting {
            applicationIdSuffix = NiaBuildType.DEBUG.applicationIdSuffix
        }
```
这里val debug by gettting为啥这样写？
这里其实分了2步：
```
val release = getByName("release")
    release.apply {
```

等价于：
` val release by getting`

继续下面：
```
   packagingOptions {
        resources {
            excludes.add("/META-INF/{AL2.0,LGPL2.1}")
        }
    }
```
这个配置是什么呢：
打包配置，需要移除这个文件。

继续下面：
```
  testOptions {
        unitTests {
            isIncludeAndroidResources = true
        }
        // TODO: Convert it as a convention plugin once Flamingo goes out (https://github.com/android/nowinandroid/issues/523)
        managedDevices {
            devices {
                maybeCreate<com.android.build.api.dsl.ManagedVirtualDevice>("pixel4api30").apply {
                    device = "Pixel 4"
                    apiLevel = 30
                    // ATDs currently support only API level 30.
                    systemImageSource = "aosp-atd"
                }
            }
        }
    }
```
这里是一个testOptions，主要用于配置测试设备的一些属性。

这里关于其它配置可以参考这篇文档[https://blog.csdn.net/shulianghan/article/details/124919287](https://blog.csdn.net/shulianghan/article/details/124919287)

继续看android闭包最后一个配置:
`namespace = "com.google.samples.apps.nowinandroid"`

每个 Android 模块都有一个命名空间，此命名空间用作其生成的 R 和 BuildConfig 类的 Java 或 Kotlin 软件包名称。命名空间由模块的 build.gradle 文件中的 namespace 属性定义，如以下代码段所示。namespace 最初会设为您在创建项目时选择的软件包名称。

最下面就是我们熟悉的dependencies了，可以看到这里非常简练，没有任何版本号，全是通过lis引入的，看着很舒服。
```

dependencies {
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

    androidTestImplementation(project(":core:testing"))
    androidTestImplementation(project(":core:datastore-test"))
    androidTestImplementation(project(":core:data-test"))
    androidTestImplementation(project(":core:network"))
    androidTestImplementation(libs.androidx.navigation.testing)
    androidTestImplementation(libs.accompanist.testharness)
    androidTestImplementation(kotlin("test"))
    debugImplementation(libs.androidx.compose.ui.testManifest)
    debugImplementation(project(":ui-test-hilt-manifest"))

    implementation(libs.accompanist.systemuicontroller)
    implementation(libs.androidx.activity.compose)
    implementation(libs.androidx.appcompat)
    implementation(libs.androidx.core.ktx)
    implementation(libs.androidx.core.splashscreen)
    implementation(libs.androidx.compose.runtime)
    implementation(libs.androidx.lifecycle.runtimeCompose)
    implementation(libs.androidx.compose.runtime.tracing)
    implementation(libs.androidx.compose.material3.windowSizeClass)
    implementation(libs.androidx.hilt.navigation.compose)
    implementation(libs.androidx.navigation.compose)
    implementation(libs.androidx.window.manager)
    implementation(libs.androidx.profileinstaller)

    implementation(libs.coil.kt)
    implementation(libs.coil.kt.svg)
}

```

最下面是强制使用某个依赖：
```
// androidx.test is forcing JUnit, 4.12. This forces it to use 4.13
configurations.configureEach {
    resolutionStrategy {
        force(libs.junit4)
        // Temporary workaround for https://issuetracker.google.com/174733673
        force("org.objenesis:objenesis:2.6")
    }
}
```

### 5.build-logic 模块配置构建逻辑
这里完全封装了自定义插件，定义依赖，配置相关的逻辑主要在这个模块，其实这个模块感觉可以复用，就是说换一个项目，这个模块基本上可以通用，改改具体版本号或者属性即可。哪些要用，哪些不用，在app模块下按需注册插件即可。

首先看下这个模块的setttings.gradle.kts文件
很简单：
```
dependencyResolutionManagement {
    repositories {
        google()
        mavenCentral()
    }
    versionCatalogs {
        create("libs") {
            from(files("../gradle/libs.versions.toml"))
        }
    }
}

rootProject.name = "build-logic"
include(":convention")
```

这里最为关键的是用了一个 versionCatelogs，这里定义了我们所有三方依赖的地方。然后引入了这个模块下的convention的具体模块，这个模块就是编写自定义插件的地方。

简单看下这个libs.version.toml文件吧，它放在项目级下的gradle文件夹中了：
```
[versions]
accompanist = "0.28.0"
androidDesugarJdkLibs = "1.2.2"
androidGradlePlugin = "7.3.1"
androidxActivity = "1.6.1"
androidxAppCompat = "1.5.1"
androidxBrowser = "1.4.0"
androidxComposeBom = "2022.12.00"
androidxComposeCompiler = "1.4.0-alpha02"
androidxComposeRuntimeTracing = "1.0.0-alpha01"
androidxCore = "1.9.0"
androidxCoreSplashscreen = "1.0.0"
androidxDataStore = "1.0.0"
androidxEspresso = "3.5.0"
androidxHiltNavigationCompose = "1.0.0"
androidxLifecycle = "2.6.0-alpha03"
androidxMacroBenchmark = "1.1.1"
androidxMetrics = "1.0.0-alpha03"
androidxNavigation = "2.5.3"
androidxProfileinstaller = "1.2.1"
等等...
```

看下build-logic 模块下的build.gradle.kts文件：
首先引入了 kotlin-dsl 插件：
```
plugins {
    `kotlin-dsl`
}
```
这样就能写 java 闭包了，还有一些 compileOnly方法等。具体关于dsl使用方法可以参考这篇文章：[https://www.jianshu.com/p/461d4a249b71](https://www.jianshu.com/p/461d4a249b71)

下面配置了执行build-logic需要依赖的一些东西：
```
java {
    sourceCompatibility = JavaVersion.VERSION_1_8
    targetCompatibility = JavaVersion.VERSION_1_8
}

dependencies {
    compileOnly(libs.android.gradlePlugin)
    compileOnly(libs.kotlin.gradlePlugin)
}

```

这里需要编译依赖一些三方库。

然后下面是重点，gradlePlugin闭包下就是注册各种自定义插件了，并且实现类对应convention模块下的某个类，这个类继承了Plugin<Project>。



### 6.总结
1.单独抽出一个module作为封装自定义插件的module。在这个module里面添加自定义插件的逻辑，在这个module的settings.gradle.kts中创建一个versionCatelogs，指向所有依赖文件中。
2.项目级别的settings.gradle.kts用includeBuild来引入上述module。
3.gradle.properties文件可以配置编译时buildconfig，aidl等文件是否生成。
4.app级别的build.gradle.kts按需配置自定义插件，testInstrumentationRunnner可以配置测试类。