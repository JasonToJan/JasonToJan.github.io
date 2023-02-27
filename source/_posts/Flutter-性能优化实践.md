---
title: Flutter 性能优化实践
date: 2023-02-24 08:59:56
op: false
cover: false
toc: true
mathjax: true
tags:
- Flutter性能优化
categories:
- Flutter
---

## 1 简介
Flutter是谷歌的移动UI框架，可以快速在iOS和Android上构建高质量的原生用户界面。 Flutter可以与现有的代码一起工作。在全世界，Flutter正在被越来越多的开发者和组织使用，并且Flutter是完全免费、开源的，可以用一套代码同时构建Android和iOS应用，性能可以达到原生应用一样的性能。但是，在较为复杂的 App 中，使用 Flutter 开发也很难避免产生各种各样的性能问题。在这篇文章中，我将介绍一些 Flutter 性能优化方面的应用实践。

## 2 性能优化的检测工具

### 2.1 flutter编译模式
Flutter支持Release、Profile、Debug编译模式。

> Release模式,使用AOT预编译模式，预编译为机器码，通过编译生成对应架构的代码，在用户设备上直接运行对应的机器码，运行速度快，执行性能好；此模式关闭了所有调试工具，只支持真机。<br>
Profile模式，和Release模式类似，使用AOT预编译模式，此模式最重要的作用是可以用DevTools来检测应用的性能，做性能调试分析。<br>
Debug模式，使用JIT（Just in time）即时编译技术，支持常用的开发调试功能hot reload，在开发调试时使用，包括支持的调试信息、服务扩展、Observatory、DevTools等调试工具，支持模拟器和真机。

#### 2.1.1 什么是AOT预编译模式？
> 在Flutter中，AOT（Ahead-Of-Time）预编译模式是一种编译方式，可以将Dart代码提前编译成本机代码，以加快应用程序的启动时间和性能。<br>
在AOT模式下，Dart代码被编译成本机机器码，并在应用程序安装时生成本机库。这意味着，在应用程序启动时不需要JIT（Just-In-Time）编译器来将Dart代码转换为本机代码，而是可以直接加载本机代码，从而加快应用程序的启动速度。<br>
相比之下，在JIT模式下，Dart代码在应用程序运行时被编译为本机代码。这意味着，在应用程序启动时需要更多的时间来编译代码，因此启动速度可能会受到影响。但是，JIT模式具有更好的开发体验，因为可以在运行时动态更改代码，并且可以更快地进行开发迭代。<br>
总之，AOT预编译模式可以加快Flutter应用程序的启动时间和性能，但可能会对开发体验产生一些影响。开发人员应该根据自己的需求和应用程序的要求来选择合适的编译模式。

#### 2.1.2 profile模式有何作用？
> Flutter项目中的profile模式主要用于应用程序性能分析和调试。与debug模式相比，profile模式会应用一些额外的优化，以提高应用程序的性能，同时仍然保留了一些debug模式下的便利功能。以下是在Flutter项目中启用profile模式的一些好处：<br>
1.应用程序性能分析：在profile模式下，Flutter会进行一些额外的优化，例如代码预编译、冷启动优化、代码混淆和资源收缩等，以减少应用程序启动时间和内存占用。这使得你可以更轻松地对应用程序性能进行分析和优化。<br>
2.调试支持：与release模式相比，profile模式仍然保留了一些debug模式下的便利功能，例如支持调试和热重载等。这使得你可以在profile模式下进行调试，而不必牺牲应用程序性能。<br>
3.应用程序体积优化：在profile模式下，Flutter会进行代码混淆和资源收缩，以减少应用程序的体积。这使得你可以更轻松地优化应用程序体积，并在应用商店中更快地发布应用程序更新。<br>
总之，在Flutter项目中启用profile模式可以提高应用程序的性能，并帮助你更轻松地进行应用程序性能分析和优化。同时，profile模式还可以保留一些debug模式下的便利功能，使得你可以更轻松地进行调试和开发。

#### 2.1.3 如何开启profile模式？
> 1.如果是独立flutter工程可以使用flutter run --profile启动。
2.如果是混合 Flutter 应用，flutter/packages/flutter_tools/gradle/flutter.gradle 的 buildModeFor 方法中将 debug 模式改为 profile即可。
在Flutter的SDK中，这个文件(根据自己配置的FlutterSDK路径来)，如下：
/Users/wangjianan/Documents/Tools/Deve/FlutterSDK_3.0/flutter/packages/flutter_tools/gradle
文件夹中的flutter.gradle文件下的方法：
private static String buildModeFor(buildType) {
    if (buildType.name == "profile") {
        return "profile"
    } else if (buildType.debuggable) {
        return "debug"
    }
    return "release"
}
这个方法中，将debug模式改为profile即可。

### 2.2 检测工具

#### 2.2.1 Flutter Inspector (debug模式下可用)
<img src=flutter01.png>
Flutter Inspector有很多功能，其中有两个功能更值得我们去关注，例如：“Select Widget Mode” 和 “Highlight Repaints”。

1.Select Widget Mode
Select Widget Mode点击 “Select Widget Mode” 图标，可以在手机上查看当前页面的布局框架与容器类型。
<img src=flutter02.png width=50%>

2.Highlight Repaints
点击右上角一个类似一直笔的图标：
<img src=flutter03.png>

点击到某个页面后就是这种效果：
<img src=flutter04.png>

这样做帮你找到 App 中频繁重绘导致性能消耗过大的部分。
例如：一个小动画可能会导致整个页面重绘，这个时候使用 RepaintBoundary Widget 包裹它，可以将重绘范围缩小至本身所占用的区域，这样就可以减少绘制消耗。

不同颜色代表不同的重绘区域和类型。以下是一些常见的颜色及其对应的意义：

紫色：重绘整个窗口或页面。
蓝色：重绘文本。
红色：重绘图像或图形。
绿色：重绘布局。
黄色：重绘绘制过程中的图形。
青色：重绘窗口的部分区域。
需要注意的是，这些颜色只是用于在Flutter Inspector中可视化重绘区域，它们并不会在应用程序中直接可见或影响应用程序的性能。在开发过程中，使用Highlight Repaints功能可以帮助开发者更好地了解应用程序的性能，并优化代码以减少重绘的次数和区域。

#### 2.2.2 Perfomance Overlay(性能图层)
在完成了应用启动之后，接下来我们就可以利用 Flutter 提供的渲染问题分析工具，即性能图层（Performance Overlay），来分析渲染问题了。
我们可以通过以下方式开启性能图层。
```dart
Widget appBuilder(Widget home) {
jhDebug.setGlobalKey = commonConfig.getGlobalKey;
appSetupInit();
return Consumer<ThemeStore>(
    builder: (context, themeStore, child) {
    return MaterialApp(
        navigatorKey: jhDebug.getNavigatorKey,
        showPerformanceOverlay: true,  // 这里为true就是开启
        locale: const Locale('zh', 'CH'),
        localizationsDelegates: const [
        GlobalMaterialLocalizations.delegate,
        GlobalWidgetsLocalizations.delegate,
        GlobalCupertinoLocalizations.delegate,
        ],
```
性能图层会在当前应用的最上层，以 Flutter 引擎自绘的方式展示 GPU 与 UI 线程的执行图表，而其中每一张图表都代表当前线程最近 300 帧的表现，如果 UI 产生了卡顿，这些图表可以帮助我们分析并找到原因。 下图演示了性能图层的展现样式。其中，GPU 线程的性能情况在上面，UI 线程的情况显示在下面，蓝色垂直的线条表示已执行的正常帧，绿色的线条代表的是当前帧：
| <img src=flutter05.png> | <img src=flutter06.png>  | <img src=flutter07.png>  |
| :---------------: | :---------------: | :---------------: |

如果红色竖条出现在 GPU 线程图表，意味着渲染的图形太复杂，导致无法快速渲染；
而如果是出现在了 UI 线程图表，则表示 Dart 代码消耗了大量资源，需要优化代码执行时间。

#### 2.2.3 CPU Profiler(UI线程问题定位)
在视图构建时，在 build 方法中使用了一些复杂的运算，或是在主 Isolate 中进行了同步的 I/O 操作。 我们可以使用 CPU Profiler 进行检测:

入口：
<img src=flutter08.png>

然后点击下面的Open DevTools会跳转到浏览器：
<img src=flutter09.png>

然后你需要手动点击 “Record” 按钮去主动触发，在完成信息的抽样采集后，点击 “Stop” 按钮结束录制。这时，你就可以得到在这期间应用的执行情况了。效果如下：
<img src=flutter10.png>

其中：
x 轴：表示单位时间，一个函数在 x 轴占据的宽度越宽，就表示它被采样到的次数越多，即执行时间越长。
y 轴：表示调用栈，其每一层都是一个函数。调用栈越深，火焰就越高，底部就是正在执行的函数，上方都是它的父函数。

通过上述CPU帧图我们可以大概分析出哪些方法存在耗时操作,针对性的进行优化
一般的耗时问题，我们通常可以 使用 Isolate（或 compute）将这些耗时的操作通过子线程完成。

例如:复杂JSON解析子线程化
Flutter的isolate默认是单线程模型，而所有的UI操作又都是在UI线程进行的，想应用多线程的并发优势需新开isolate 或compute。无论如何await，scheduleTask 都只是延后任务的调用时机，仍然会占用“UI线程”， 所以在大Json解析或大量的channel调用时，一定要观测对UI线程的消耗情况。
> 在 Flutter 中，UI 线程是指负责处理界面更新和渲染的线程。当我们在 UI 线程上执行一些耗时操作时，会导致 UI 线程被阻塞，从而导致应用程序的卡顿或者无响应。<br>
使用 await 关键字可以将一个耗时操作包装成一个异步操作，从而避免阻塞 UI 线程。await 会挂起当前的异步函数，等待异步操作完成后再继续执行下面的代码。<br>
而 scheduleTask 方法可以将一个任务推迟到当前事件队列的末尾执行，从而延迟任务的调用时机。这个方法通常用于将某些不紧急的任务放到事件队列的末尾，从而让 UI 线程先处理一些紧急的任务，以提高应用程序的响应速度。<br>
然而，无论如何使用 await 或者 scheduleTask，它们都不会在 UI 线程之外执行。它们仅仅是将任务的调用时机推迟到将来的某个时间点。因此，它们仍然会占用 UI 线程，从而可能导致 UI 线程被阻塞，应用程序的响应速度变慢。<br>
因此，在 Flutter 中，我们需要遵循一些最佳实践来避免在 UI 线程上执行耗时操作。比如，我们可以将一些计算密集型的操作放到后台 Isolate 中执行，或者使用 Flutter 提供的异步 API 来执行一些耗时的 IO 操作，从而避免阻塞 UI 线程。

## 3 布局优化

Flutter 使用了声明式的 UI 编写方式，而不是 Android 和 iOS 中的命令式编写方式。
当然，Android后面出的Compose,iOS的SwiftUI也是支持声明式的。

声明式:简单的说，你只需要告诉计算机，你要得到什么样的结果，计算机则会完成你想要的结果，声明式更注重结果。
命令式:用详细的命令机器怎么去处理一件事情以达到你想要的结果，命令式更注重执行过程。

flutter声明式的布局方式如何构建布局的：
可以参考下这篇文章：[一文读懂Flutter的三棵树渲染机制和原理](https://cloud.tencent.com/developer/article/1771780)。
> 当我们在使用 Flutter 构建应用时，Flutter 会维护三棵树：widget 树、元素树和渲染对象树。这三棵树分别代表了不同的概念和职责。<br>
Widget 树：Widget 树是 Flutter 构建 UI 的基础，它是由 widget 组成的层次结构，用于描述我们应用的界面。Widget 是不可变的，所以我们无法直接修改 Widget 树。当我们需要更新 UI 时，我们会创建一个新的 Widget 树，Flutter 会比较新旧两棵树之间的差异，并根据差异来更新界面。<br>
元素树：元素树是在构建 Widget 树的过程中创建的。元素是 Widget 树中的一个实例，它是可变的，因为它会持有一个指向渲染对象的引用。在构建元素树时，Flutter 会根据 Widget 树创建出对应的元素树，并在元素树上记录需要更新的位置和操作，以便后续的渲染。<br>
渲染对象树：渲染对象树是由 Flutter 引擎构建和维护的，它描述了我们应用的实际渲染内容。渲染对象是与平台相关的，因此它们可能会被实现为 OpenGL、Skia 或其他图形库。当我们需要更新界面时，Flutter 会比较新旧两棵元素树之间的差异，并根据差异来更新渲染对象树，最终实现视图的更新。<br>
总之，Widget 树、元素树和渲染对象树是 Flutter 构建 UI 的核心，它们分别代表了 UI 的描述、状态和实际渲染内容。Flutter 利用这三棵树的结构和关系，实现了高效的界面更新和渲染机制，使得我们能够快速地构建高性能、流畅的应用。

### 3.1 常规优化
常规优化即针对 build() 进行优化，build() 方法中的性能问题一般有两种：耗时操作和 Widget 层叠。

建议1：不要在build()方法中执行了耗时操作
> 我们应该尽量避免在 build() 中执行耗时操作，因为 build() 会被频繁地调用，尤其是当 Widget 重建的时候。 此外，我们不要在代码中进行阻塞式操作，可以将一般耗时操作等通过 Future 来转换成异步方式来完成。 对于 CPU 计算频繁的操作，例如图片压缩，可以使用 isolate 来充分利用多核心 CPU。

建议2：不要在build()方法中堆叠了大量的Widget
> 这将会导致三个问题：
1、代码可读性差：画界面时需要一个 Widget 嵌套一个 Widget，但如果 Widget 嵌套太深，就会导致代码的可读性变差，也不利于后期的维护和扩展。<br>
2、复用难：由于所有的代码都在一个 build()，会导致无法将公共的 UI 代码复用到其它的页面或模块。<br>
3、影响性能：我们在 State 上调用 setState() 时，所有 build() 中的 Widget 都将被重建，因此 build() 中返回的 Widget 树越大，那么需要重建的 Widget 就越多，也就会对性能越不利。<br>
所以，你需要 控制 build 方法耗时，将 Widget 拆小，避免直接返回一个巨大的 Widget，这样 Widget 会享有更细粒度的重建和复用。

建议3：尽可能地使用const构造器
> 当构建你自己的 Widget 或者使用 Flutter 的 Widget 时，这将会帮助 Flutter 仅仅去 rebuild 那些应当被更新的 Widget。 因此，你应该尽量多用 const 组件，这样即使父组件更新了，子组件也不会重新进行 rebuild 操作。特别是针对一些长期不修改的组件，例如通用报错组件和通用 loading 组件等。

建议4：对列表进行优化
尽量避免使用 ListView默认构造方法。不管列表内容是否可见，会导致列表中所有的数据都会被一次性绘制出来
建议使用 ListView 和 GridView 的 builder 方法
它们只会绘制可见的列表内容，类似于 Android 的 RecyclerView。
```dart
Widget _list() {
return Expanded(
    child: ListView.builder(
    padding: const EdgeInsets.all(0),
    shrinkWrap: true,
    itemBuilder: (context, index) {
    return UnmatchedProductItem(mProduct: mList?[index]);
    },
    itemCount: mList?.length ?? 0,
));
}
```
其实，本质上，就是对列表采用了懒加载而不是直接一次性创建所有的子 Widget，这样视图的初始化时间就减少了。

### 3.2 深入光栅化优化
屏幕显示器一般以60Hz的固定频率刷新，每一帧图像绘制完成后，会继续绘制下一帧，这时显示器就会发出一个Vsync信号，按60Hz计算，屏幕每秒会发出60次这样的信号。CPU计算好显示内容提交给GPU，GPU渲染好传递给显示器显示。 Flutter遵循了这种模式，渲染流程如图：
<img src=flutter11.png>

flutter通过native获取屏幕刷新信号通过engine层传递给flutter framework:
所有的 Flutter 应用至少都会运行在两个并行的线程上：UI 线程和 Raster 线程。
•	UI 线程
构建 Widgets 和运行应用逻辑的地方。
•	Raster 线程
用来光栅化应用。它从 UI 线程获取指令将其转换成为GPU命令并发送到GPU。

我们通常可以使用Flutter DevTools-Performance 进行检测，步骤如下：

* 在 Performance Overlay 中，查看光栅线程和 UI 线程哪个负载过重。
* 在 Timeline Events 中，找到那些耗费时间最长的事件，例如常见的 SkCanvas::Flush，它负责解决所有待处理的 GPU 操作。
* 找到对应的代码区域，通过删除 Widgets 或方法的方式来看对性能的影响。


## 4 内存优化

### 4.1 const 实例化
const 对象只会创建一个编译时的常量值。在代码被加载进 Dart Vm 时，在编译时会存储在一个特殊的查询表里，仅仅只分配一次内存给当前实例。
我们可以使用 flutter_lints 库对我们的代码进行检测提示。

### 4.2 检测消耗多余内存的图片
Flutter Inspector：点击 “Highlight Oversizeded Images”，它会识别出那些解码大小超过展示大小的图片，并且系统会将其倒置，这些你就能更容易在 App 页面中找到它。

如果发现有些图片被倒置了，
针对这些图片，你可以指定 cacheWidth 和 cacheHeight 为展示大小，这样可以让 flutter 引擎以指定大小解析图片，减少内存消耗
```dart
  @override
  Widget build(BuildContext context) {
    return Image.asset('test.png', cacheHeight: 200, cacheWidth: 200);
  }
```

### 4.3 针对 ListView item 中有 image 的情况来优化内存
ListView 不会销毁那些在屏幕可视范围之外的那些 item，如果 item 使用了高分辨率的图片，那么它将会消耗非常多的内存。

ListView 在默认情况下会在整个滑动/不滑动的过程中让子 Widget 保持活动状态，这一点是通过 AutomaticKeepAlive 来保证，在默认情况下，每个子 Widget 都会被这个 Widget 包裹，以使被包裹的子 Widget 保持活跃。 
其次，如果用户向后滚动，则不会再次重新绘制子 Widget，这一点是通过 RepaintBoundaries 来保证，在默认情况下，每个子 Widget 都会被这个 Widget 包裹，它会让被包裹的子 Widget 仅仅绘制一次，以此获得更高的性能。 但，这样的问题在于，如果加载大量的图片，则会消耗大量的内存，最终可能使 App 崩溃。

```dart
@override
Widget build(BuildContext context) {
    return ListView.builder(
        itemBuilder: (context, index) {
        return Image.asset('test.png', cacheWidth: 200, cacheHeight: 200);
        },
        addAutomaticKeepAlives: false,
        addRepaintBoundaries: false,
    );
}
```
通过将这两个选项置为 false 来禁用它们，这样不可见的子元素就会被自动处理和 GC。

### 4.4 多变图层与不变图层分离
在日常开发中，会经常遇到页面中大部分元素不变，某个元素实时变化。如Gif，动画。这时我们就需要RepaintBoundary，不过独立图层合成也是有消耗，这块需实测把握。
这会导致页面同一图层重新Paint。此时可以用RepaintBoundary包裹该多变的Gif组件，让其处在单独的图层，待最终再一块图层合成上屏。
```dart
@override
Widget build(BuildContext context) {
    return Container(
        child: Row(
        mainAxisSize: MainAxisSize.min,
        crossAxisAlignment: CrossAxisAlignment.center,
        mainAxisAlignment: MainAxisAlignment.center,
        children: [
            RepaintBoundary(child: loadingGifImage()),
            Container(height: 20, width: 20, color: Colors.red)
        ]),
    );
}
```

### 4.5 降级CustomScrollView,ListView等预渲染区域为合理值
默认情况下，CustomScrollView除了渲染屏幕内的内容，还会渲染上下各250区域的组件内容，例如当前屏幕可显示4个组件，实际仍有上下共4个组件在显示状态，如果setState()，则会进行8个组件重绘。实际用户只看到4个，其实应该也只需渲染4个， 且上下滑动也会触发屏幕外的Widget创建销毁，造成滚动卡顿。高性能的手机可预渲染，在低端机降级该区域距离为0或较小值。

```Dart
@override
Widget build(BuildContext context) {
    return CustomScrollView(
        controller: _scrollController,
        slivers: detailContent,
        cacheExtent: isLowDeviceOptimizer() ? 60: null,
    );
}
```
这里根据是否低端机来设置cacheExtent的值来决定是否预加载。

## 5 总结

为什么Flutter会出现卡顿，帧率低？
根本原因只有两个：
* UI线程慢了–>渲染指令出的慢
* GPU线程慢了–>光栅化慢、图层合成慢、像素上屏慢，所以我们一般使用flutter布局尽量按照以下原则
> •	尽量不要为 Widget 设置半透明效果，而是考虑用图片的形式代替，这样被遮挡的 Widget 部分区域就不需要绘制了；
•	控制 build 方法耗时，将 Widget 拆小，避免直接返回一个巨大的 Widget，这样 Widget 会享有更细粒度的重建和复用；
•	对列表采用懒加载而不是直接一次性创建所有的子 Widget，这样视图的初始化时间就减少了。




