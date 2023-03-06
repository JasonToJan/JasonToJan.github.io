---
title: 面试 Flutter 通关秘籍
date: 2023-01-09 21:26:12
top: false
cover: false
toc: true
mathjax: true
tags:
- 面试题 Flutter
categories:
- 面试 
---

## 1 黑铁
> dart基础，flutter基础。

### 1.1 Dart语言有啥特性?
> 1.在Dart中，一切都是对象，所有的对象都是继承自Object
2.Dart是强类型语言，但可以用var或 dynamic来声明一个变量，Dart会自动推断其数据类型,dynamic类似c#<br>
3.没有赋初值的变量都会有默认值null<br>
4.Dart支持顶层方法，如main方法，可以在方法内部创建方法<br>
5.Dart支持顶层变量，也支持类变量或对象变量<br>
6.Dart没有public protected private等关键字，如果某个变量以下划线（_）开头，代表这个变量在库中是私有的（约定而已）

### 1.2 Dart中级联操作符？
> Dart 当中的 「..」意思是 「级联操作符」，为了方便配置而使用。「..」和「.」不同的是 调用「..」后返回的相当于是 this，而「.」返回的则是该方法返回的值 。<br>
级联操作符只能用于没有返回值的方法或属性。如果你需要获取对象的返回值，则必须使用单个.（点）符号来引用该对象。
```dart
void main() {
  var person = Person()
    ..name = "John"
    ..age = 30
    ..sayHello();
}
```

### 1.3 mixin，extends,implement的区别？
> 1.extends用于创建一个子类继承自一个父类，可以继承父类的属性和方法，并可以通过重写方法来修改或扩展父类的行为。例如：
```dart
class Animal {
  void makeSound() {
    print('...');
  }
}

class Cat extends Animal {
  @override
  void makeSound() {
    print('Meow!');
  }
}
```

> 2.implements用于创建一个类实现一个接口，一个接口定义了一个类应该实现哪些方法，但不提供具体的实现。一个类可以实现多个接口，并实现接口定义的所有方法。
```dart
abstract class Animal {
  int get preferredSize;
  void makeSound();
}

class Cat implements Animal {
  @override
  int get preferredSize => 50;

  @override
  void makeSound() {
    print('Meow!');
  }  
}
```

> 3.mixin用于在一个类中引入另一个类的功能，而不需要继承该类。Mixin类通常用于将一些通用的功能分离出来，并可以在多个类中复用。
```dart
mixin Swimming {
  void swim() {
    print('Swimming...');
  }
}

class Animal {}

class Fish extends Animal with Swimming {}
```

> 总结：mixin侧重点抽离公共方法，提升复用性，非强制要实现。implement是一种强制性行为，必须满足实现所有接口。继承注重类的结构，继承关系。

### 1.4 dart函数中对象作为参数传递，是值传递还是引用传递？
> 引用传递。
取决于这个不可变对象还是可变对象。
不可变（数字，字符串等）相等于值传递，如果是可变对象，相当于传的是指针，相当于引用传递。

### 1.5 statefulWidget生命周期和statelessWidget生命周期？
> 先说下无状态的Widget，只有一个build生命周期。<br>
然后重点说下有状态widget的生命周期：<br>
1.initState，初始化状态。<br>
2.didChangedependencies: 当 StatefulWidget 第一次创建的时候，didChangeDependencies 方法会在 initState 方法之后立即调用，之后当 StatefulWidget 刷新的时候，就不会调用了，除非你的 StatefulWidget 依赖的 InheritedWidget 发生变化之后，didChangeDependencies 才会调用，所以 didChangeDependencies 有可能会被调用多次。<br>
3.build: 每当 UI 需要重新渲染的时候，build 都会被调用。<br>
4.didUpdateWidget: 在Flutter中，当一个StatefulWidget所依赖的状态发生变化时，会触发didUpdateWidget()方法。
需要注意的是，当didUpdateWidget()方法被调用时，Widget树中的所有Widget都已经完成了build()方法的调用，因此我们应该避免在该方法中调用setState()方法或执行其他可能导致Widget树重建的操作，以避免陷入死循环。<br>
5.deactivate: 当要将 State 对象从渲染树中移除的时候，就会调用 deactivate 生命周期。暂时不会被销毁，会插入到渲染树中。<br>
6.dispose: 当 View 不需要再显示，从渲染树中移除的时候，State 就会永久的从渲染树中移除，就会调用 dispose 生命周期。

### 1.6 InheritedWidget是什么？
> 是一种特殊的Widget，可以将一些数据在Widget树中传递给子孙Widget，而不必通过回调等方式传递数据。<br>
最好举个例子：假设我们在应用中使用了一个ThemeWidget来设置应用的主题色。当我们在子Widget中使用了ThemeWidget并且依赖于它中的主题色时，这个子Widget就会依赖于ThemeWidget。如果我们在应用中修改了主题色，那么ThemeWidget中的数据就会发生变化，这时依赖于ThemeWidget的子Widget就会被通知，并且可以重新构建自己的Widget树，以便更新主题色等相关的样式。这个过程就是通过InheritedWidget来实现的。

### 1.7 dart中的isolate是什么东西？
> 在Dart语言中，Isolate是一个独立的运行环境，类似于操作系统中的进程。每个Isolate都有自己的内存空间和执行上下文，它们之间是相互独立的。因此，在Dart语言中，多个Isolate可以并行执行，从而充分利用多核CPU的性能优势。<br>
在Dart程序中，通常会有一个Main Isolate和若干个Worker Isolate。Main Isolate是Dart程序的主要执行环境，负责管理整个应用程序的生命周期，处理UI事件等。而Worker Isolate则负责执行耗时操作，例如网络请求、计算密集型任务等。<br>
每个Isolate都拥有自己的事件循环(Event Loop)和消息队列(Message Queue)，它们负责处理事件和消息，并按照一定的顺序执行任务。在Dart中，每个Isolate的执行优先级是相互独立的，也就是说，一个Isolate的执行不会受到其他Isolate的影响。<br>
一个应用程序对应一个Main Isolate，也就是一个主进程，当然也可以配置多进程。

### 1.8 如何使用Future?
> Future 实例有 3 个常用方法:<br>
then((value){...}): 正常运行时执行
catchError((err){...}): 出现错误时执行
whenComplete((){...}): 不管成功与否都会执行
Future.value(): 创建一个返回具体数据的 Future 实例
Future.error(): 创建一个返回错误的 Future 实例
Future.delayed(): 创建一个延时执行的 Future 实例

## 2 青铜
> dart高级用法。

### 2.1 Stream与Future是什么关系？
> 1、Future 表示稍后获得的一个数据，所有异步的操作的返回值都用 Future 来表示。
2、Future 只能表示一次异步获得的数据。
3、Stream 表示多次异步获得的数据。比如界面上的按钮可能会被用户点击多次，按钮上的点击事件（onClick）就是一个 Stream 。
4、Future将返回一个值，而Stream将返回多次值。
5、Dart 中统一使用 Stream 处理异步事件流。

### 2.2 await for和stream区别和联系？
> 首先请求务必回答下：在Flutter中，await for和stream都与异步流处理相关。<br>
await for是一个特殊的语法结构，用于从异步流中逐个获取元素并等待每个元素。它可以用于异步生成器或异步迭代器，其中每个元素都是通过异步计算得到的。await for在每次迭代时会等待下一个元素的到达，直到异步流关闭为止。
eg:
```dart
Stream<int> countStream() async* {
  for (var i = 1; i <= 5; i++) {
    yield i;
  }
}

await for (var count in countStream()) {
  print(count);
}
```

> stream是一个类，它表示一个异步流。使用stream类，你可以创建一个异步流并发送多个元素。
```dart
Stream<int> countStream() {
  return Stream.fromIterable([1, 2, 3, 4, 5]);
}
```

### 2.3 Stream有哪两种订阅模式？
> 简要回答下：两种，单订阅和广播。<br>
单订阅类似于点对点，在订阅者出现之前会持有数据，在订阅者出现之后就才转交给它。
而广播类似于发布订阅模式，可以同时有多个订阅者，当有数据时就会传递给所有的订阅者，而不管当前是否已有订阅者存在。<br>
默认单订阅模式，但是可以转换成多订阅模式，通过Stream.asBroadcastStream方法。

### 2.4 如何理解Future?
> Dart语言中，Future表示一个异步操作的结果，它可以看作是一个延迟计算的对象，用于处理一些需要耗费时间的操作，例如网络请求、文件读写等等。Future可以让我们在异步任务完成后进行回调处理，从而避免了阻塞主线程的情况。<br>
可以使用async和await关键字来简化异步编程。使用async标记一个函数为异步函数，该函数会返回一个Future对象；使用await关键字可以等待异步任务的结果，并返回结果。<br>
需要注意的是：Future并没有开启新的线程，而是在线程空闲的时候才去执行。

### 2.5 isolate是什么？
> 所有的 Dart 代码都是在 isolate 中运行的, 它就是机器上的一个小空间, 具有自己的私有内存块和一个运行着 Event Looper 的单个线程. 每个 isolate 都是相互隔离的, 并不像线程那样可以共享内存. 一般情况下, 一个 Dart 应用只会在一个 isolate 中运行所有代码, 但如果有特殊需要, 可以开启多个。
注意：Dart中没有线程概念，只有isolate，而这个又比较像Android的进程。

### 2.6 isolate之间如何通信？
> 唯一方法是通过来回传递消息。
般情况下, 子isolate 会将运行结果通过管道以消息的形式发送到 主isolate, 并在 主isolate 的 Event Looper 中处理该消息, 这时就需要借助 ReceivePort 来处理消息的传递了。

### 2.7 flutter中如何创建isolate?
> Flutter 提供了更为方便的开启 isolate 的 API: compute() 函数。

### 2.8 async+await+future，本质上还是在main isolate中处理的吧？
> 是的，使用 async/await 本质上仍然是在当前 isolate 中处理任务。因为在Dart中，async/await 是一种通过 Future 来管理异步任务的语法糖，其背后仍然是通过 Future 来实现的。在使用 async/await 时，代码的执行仍然是单线程的，只不过通过 Future 的事件循环机制来实现了异步效果，使得程序不会被阻塞。如果要在后台执行耗时任务，需要使用 Isolate 来开启新的 isolate。

### 2.9 immutable注解怎么用？
> 在 Dart 中，可以使用 @immutable 注解来标记一个类，使其成为不可变的。使用 @immutable 注解的类将保证其所有字段都是 final 的，并且所有的 getter 方法也都是不可变的，即它们的返回值也是不可变的。<br>
使用 @immutable 注解可以提高代码的可读性和可维护性，因为它明确表明了这个类是不可变的，因此在使用该类的过程中可以放心地假设其状态不会发生变化，从而避免一些意外的错误。

### 2.10 如何决策使用Future还是isolate?
> 1、如果一段代码不会被中断，那么就直接使用正常的同步执行就行。
2、如果代码段可以独立运行而不会影响应用程序的流畅性，建议使用 Future （需要花费几毫秒时间）
3、如果繁重的处理可能要花一些时间才能完成，而且会影响应用程序的流畅性，建议使用 isolate （需要几百毫秒）<br>
使用isolate的场景：
1、JSON解析: 解码JSON，这是HttpRequest的结果，可能需要一些时间，可以使用封装好的 isolate 的 compute 顶层方法。
2、加解密: 加解密过程比较耗时
3、图片处理: 比如裁剪图片比较耗时
4、从网络中加载大图

### 2.11 Flutter是如何与原生Android,iOS通信的？
> Flutter 通过 PlatformChannel 与原生进行交互，其中 PlatformChannel 分为三种：
BasicMessageChannel ：用于传递字符串和半结构化的信息。
MethodChannel ：用于传递方法调用（method invocation）。
EventChannel : 用于数据流（event streams）的通信。

## 3 白银
> dart一些原理。

### 3.1 Dart的单线程模型是如何运行的？
> 务必回答下是：消息循环机制来运行的。包括两个任务队列。
Dart 在单线程中是以消息循环机制来运行的，包含两个任务队列，一个是“微任务队列” microtask queue，另一个叫做“事件队列” event queue。<br>
当Flutter应用启动后，消息循环机制便启动了。
按照先进先出的顺序逐个执行 微任务队列 中的任务，当所有 微任务队列 执行完后便开始执行 事件队列 中的任务，事件任务执行完毕后再去执行微任务，如此循环往复；

### 3.2 Widget，Sate，Context究竟是什么呢？为什么要设计这三个东西？
> 依次介绍作用：
Widget:
Widget是Flutter中构建用户界面的最基本单元，它是不可变的，描述了UI的一部分。Widget的构建是通过其他Widget的组合完成的，因此Widget树可以看做是由Widget组成的。<br>
State:
State是Widget的状态和数据，它是可变的，因此Flutter将State分离出来，以便可以在不重新创建Widget的情况下改变Widget的状态和数据。State存储了Widget的状态和数据，它通过setState方法来告诉Flutter框架当前Widget的状态和数据发生了变化。<br>
Context:
我们可以知道 BuildContext 实际上就是 Element 对象，主要是为了防止开发者直接操作 Element 对象。通过源码我们也可以看到 Element 是实现了 BuildContext 这个抽象类。Context是每个widget中的build方法参数，在flutter中context会把当前的widget在widgetTree中的具体位置和尺寸等相关信息都传递到build方法里，以此保证各个子widget和上级的相对位置等信息。<br>
总结下：
综上所述，Widget、State和Context是Flutter框架中非常重要的概念，它们的设计目的是为了简化UI的构建和管理。Widget是描述UI的最基本单元，State用于管理Widget的状态和数据，Context包装了Elements对象。通过这三个概念的协作，Flutter框架可以方便地构建出复杂的用户界面。

### 3.3 async和await的原理？
> 当我们使用async和await关键字执行异步操作时，实际上是利用了Dart的事件循环机制来实现异步操作的执行。事件循环是Dart中处理异步事件的核心机制，它是一个无限循环，在每个循环迭代中，Dart会检查是否有任何异步操作已经完成。如果有，则Dart会调用对应的回调函数，并执行下一步操作。<br>
这种事件循环机制使得我们可以执行耗时的异步操作，而不会阻塞主线程。例如，我们可以使用await关键字等待一个网络请求返回结果，而在等待期间，Dart会将控制权交回给事件循环，以允许其他异步操作继续执行。当网络请求完成并返回结果时，Dart会调用对应的回调函数，并执行下一步操作。<br>
需要注意的是，当我们执行耗时的异步操作时，实际上是将这些异步操作提交给Dart的事件循环机制进行处理。因此，如果我们提交的异步操作非常耗时，可能会导致事件循环的负载过重，从而影响其他异步操作的执行。为了避免这种情况，我们通常会将耗时的异步操作拆分成多个小的异步操作，并使用await关键字等待它们完成。这样可以将负载均匀分配到多个事件循环迭代中，从而保证整个应用程序的性能和稳定性。<br>
总之，Dart的async和await利用事件循环机制实现异步操作的执行，使得我们可以执行耗时的异步操作，而不会阻塞主线程。同时，为了保证整个应用程序的性能和稳定性，我们通常会将耗时的异步操作拆分成多个小的异步操作，并使用await关键字等待它们完成。

### 3.4 Dart事件循环机制？
> Dart 事件循环机制由 一个消息循环(Event Looper) 和 两个消息队列(Event Queue) 构成, 这两个消息队列分别是: 事件队列(Event queue) 和 微任务队列(MicroTask queue).<br>
然后需要简要说明下这三个对象的作用：
1.消息循环：
Dart 在执行完 main 函数后, Event Looper 就开始工作, Event Looper 优先全部执行完 Microtask Queue 中的 event, 直到 Microtask Queue 为空时, 才会执行 Event Looper 中的 event, Event Looper 为空时才可以退出循环。<br>
2.事件队列：
Event Queue 的 event 来源于 外部事件 和 Future。
外部事件: 例如输入/输出, 手势, 绘制, 计时器, Stream 等<br>
3.微任务队列
Microtask Queue 的优先级高于 Event Queue.
使用场景: 想要在稍后完成一些任务(microtask) 但又希望在执行下一个事件(event)之前执行。

### 3.5 Widget中的Key有什么用呢？
> 是一个唯一标识的作用。当需要在一个StatefulWidget集合中进行添加、删除、重排序等操作时，才是key登场的时候。<br>
Key有哪些类型：
ValueKey:以一个值为key。
ObjectKey:以一个对象为key。
UniqueKey:生成唯一的随机数作为key。
PageStorageKey:专用于存储页面滚动位置的key。
GlobalKey:每个globalkey都是一个在整个应用内唯一的key。
用途：
1.允许widget在应用程序中的任何位置更改其parent而不丢失其状态。应用场景：在两个不同的屏幕上显示相同的widget，并保持状态相同。
2.GlobalKey 能够跨 Widget 访问状态。 
```dart
class _ScreenState extends State<Screen> {
  final GlobalKey<SwitcherScreenState> key = GlobalKey<SwitcherScreenState>();
​
  @override
  Widget build(BuildContext context) {
    return Scaffold(
      body: SwitcherScreen(
        key: key,
      ),
      floatingActionButton: FloatingActionButton(onPressed: () {
        key.currentState.changeState();
      }),
    );
  }
}
```

### 3.6 Flutter的状态管理如何实现？
> Flutter的状态可以分为全局状态和局部状态两种。常用的状态管理有ScopedModel、BLoC、Redux / FishRedux和Provider。<br>
状态管理基本都是基于InheritedWidget封装的用于Widget树的数据传递与共享的的一套框架。<br>
Provider是继承于InheritProvider(框架内)，而InheritProvider本质上是一个InheritWidget(DartSDK内部)，所以Provider本质上是依托于InheritProvider的机制来实现的widget树的状态共享。

### 3.7 Provider如何使用？
> 需要提及3个重要对象。
1.ChangeNotifier: 被观察者对象，它用于向监听器发送通知，它内部有个notifyListener来发送通知。
2.Provider: 被观察者对象提供者，状态对象提供者，本质上是一个widget，提供可被监控的状态对象。
有多种Provider，如MultiProvider，StreamProvider流监听，FutureProvider等。
3.Consumer: 观察者，本质上也是一个widget，用于监听ChangeNotifier变化，并作出相应的UI处理。<br>
然后说下流程：
1.先创建一个ChangeNotifier，定义一些变量，和改变变量的方法。
2.创建一个provider作用域，就是在想要这个provider作用返回外层用各种类型的Provider包裹下。
3.获取一个ChangeNotifier，采用Provider.of方法读取。
4.在需要监听变更的地方，用Selector或者Consumer包一下，内部可读取最新值。
5.在触发变更的地方，调用ChangeNotifier的通知方法。

### 3.8 Redux如何使用？
> 先简单介绍： 是一个单项数据流架构。
原理：基于InheritedWidget封装的用于Widget树的数据传递与共享的的一套框架，它能高效的完成数据共享，进而达到ui及时更新等目的。<br>
主要流程：
1.我们在Redux中，所有的状态都储存在Store里。这个Store会放在App顶层。
2.widget拿到Store储存的状态(State)来用作ui处理。 widget还会与用户进行交互，用户点击按钮滑动屏幕等等，这时会因为交互需要数据发生改变。
3.Redux让我们不能让widget直接操作数据，而是通过发起一个action来告诉Reducer，状态得改变啦。
4.这时候Reducer接收到了这个action，他就回去遍历action表，然后找到那个匹配的action，根据action生成新的状态并把新的状态放到Store中,替换原有状态。
5.Store丢弃了老的状态对象，储存了新的状态对象后，就通知所有使用到了这个状态的widget更新（类似setState）。这样我们就能够同步不同view中的状态了。

## 4 黄金
> flutter一些渲染机制。

### 4.1 三棵树是什么？
> 第一棵树：Widget树，Widget树是Flutter中最基本的树形结构，用于表示Widget对象的层次结构。Widget树是由Widget构成的，Widget是Flutter中构建用户界面的最基本单元。Widget树是用来描述Flutter应用的整个用户界面的结构。<br>
第二棵树：元素树，Element树是Widget树的另一种表示形式，它是Widget树的实例化对象。在Flutter中，当Widget树被构建完成后，会生成对应的Element树，每个Widget都对应着一个Element。Element树是用来描述Flutter应用的整个用户界面的状态。<br>
第三棵树：渲染树，Render树是Element树的另一种表示形式，它是用来描述Flutter应用的整个用户界面的渲染信息。Render树是由RenderObject对象构成的，RenderObject对象是用来描述具体的绘制和布局信息的。<br>
最好总结下：总之，Widget树用于描述Flutter应用的用户界面结构，Element树用于描述Flutter应用的用户界面状态，Render树用于描述Flutter应用的用户界面渲染信息。这三棵树共同构成了Flutter应用的用户界面。

### 4.2 Flutter是怎么完成组件渲染的？
> 在计算机系统中，图像的显示需要CPU、GPU和显示器一起配合完成CPU负责图像数据计算，GPU负责图像数据渲染，而显示器则负责最终图像显示。CPU把计算好的、需要显示的内容交给GPU，由GPU完成渲染后放入帧缓冲区，随后视频控制器根据垂直同步信号以每秒60次的速度，从帧缓冲区读取帧数据交由显示器完成图像显示。操作系统在呈现图像时遵循了这种机制。<br>
而Flutter作为跨平台开发框架也采用了这种底层方案，UI线程使用Dart语言来构建视图结构数据，这些数据会在GPU线程进行图层合成，随后交给图像渲染引擎Skia加工成GPU数据，而这些数据会通过OpenGL最终提供给GPU渲染。<br>
可以看到Flutter用了计算机最基本的图像渲染技术，摒弃其他一些通道和过程，用最直接的方式完成了图形显示，自然性能也就得到了保障。

### 4.3 PlatformView以及其原理？
> Flutter 中通过 PlatformView 可以嵌套原生 View 到 Flutter UI 中，这里面其实是使用了 Presentation + VirtualDisplay + Surface 等实现的。<br>
大致原理：(深入回答)
使用了类似副屏显示的技术，VirtualDisplay 类代表一个虚拟显示器，调用 DisplayManager 的 createVirtualDisplay() 方法，将虚拟显示器的内容渲染在一个 Surface 控件上，然后将 Surface 的 id 通知给 Dart，让 engine 绘制时，在内存中找到对应的 Surface 画面内存数据，然后绘制出来，实时控件截图渲染显示技术。<br>
简单回答：
在 Flutter 中使用 PlatformView 的原理是通过创建一个 Flutter 的 PlatformView widget，然后将其与本地的 iOS 或 Android 原生视图相关联。Flutter 框架会将 PlatformView 作为一个整体插入到 widget 树中。在插入后，Flutter 和本地视图通过平台通道进行通信。<br>
具体来说，开发者需要在 Flutter 应用中创建一个 PlatformView widget，并将其添加到 widget 树中。然后，Flutter 框架将向原生层发送一个平台通道消息，请求原生层创建一个具体的视图。原生层收到请求后，会创建一个本地视图并返回其标识符。Flutter 框架将此标识符传递给 PlatformView widget，以便其可以与原生视图进行通信。

### 4.4 Flutter绘制流程
> Flutter只关心向 GPU提供视图数据，GPU的 VSync信号同步到 UI线程，UI线程使用 Dart来构建抽象的视图结构，这份数据结构在 GPU线程进行图层合成，视图数据提供给 Skia引擎渲染为 GPU数据，这些数据通过 OpenGL或者 Vulkan提供给 GPU。

## 5 铂金
> 高级原理。

### 5.1 Flutter的热重载原理？
> Flutter 的热重载是基于 JIT 编译模式的代码增量同步。由于 JIT 属于动态编译，能够将 Dart 代码编译成生成中间代码，让 Dart VM 在运行时解释执行，因此可以通过动态更新中间代码实现增量同步。<br>
热重载的流程可以分为 5 步，包括：扫描工程改动、增量编译、推送更新、代码合并、Widget 重建。Flutter 在接收到代码变更后，并不会让 App 重新启动执行，而只会触发 Widget 树的重新绘制，因此可以保持改动前的状态，大大缩短了从代码修改到看到修改产生的变化之间所需要的时间。<br>
另一方面，由于涉及到状态的保存与恢复，涉及状态兼容与状态初始化的场景，热重载是无法支持的，如改动前后 Widget 状态无法兼容、全局变量与静态属性的更改、main 方法里的更改、initState 方法里的更改、枚举和泛型的更改等。<br>
可以发现，热重载提高了调试 UI 的效率，非常适合写界面样式这样需要反复查看修改效果的场景。但由于其状态保存的机制所限，热重载本身也有一些无法支持的边界。

### 5.2 Flutter热更新？
> Android：
利用原生框架更新，实际上就是更新Flutter框架相关的二进制。Flutter应用发布出来的产物主要包括 libflutter.so，libapp.so，flutterAssets，这样，就可以通过Android端原生平台网络请求，动态下发并加载这些产物，从而实现热更新。<br>
iOS：苹果商店不允许动态下发和加载二进制产物，包括动态库之类的；

## 6 钻石

### 6.1 动态化方案了解过吗？
> 基本思路：
通过定义统一的描述语言(JSON来表示结构、样式和行为)，然后通过可视化平台来拖拽出JSON模板，最后将JSON模板下发到Flutter App，Flutter App内置了JS模板引擎以及DSL解析引擎，由它们将DSL解析映射为Flutter Widgets或者渲染对象。<br>
Flutter动态化整体架构
总体架构上分为四大部分。
第一部分是可视化搭建平台，负责开发DSL页面和配置数据。
第二部分是低代码服务中台，提供组件保存、页面发布和数据加工能力。
第三部分是面向端的接口服务，包括模板和数据接口。
第四部分是端，这块是核心重点，端上需要支持一整套DSL的解析和渲染映射，并且要做好相应的优化，以保证渲染性能和效率。

### 6.2 美团外卖Flutter动态化实践？
> 参考链接 [美团外卖Flutter动态化实践](https://tech.meituan.com/2020/06/23/meituan-flutter-flap.html)

### 6.3 阿里 Kraken?
> Kraken 是阿里开源的一款基于 W3C 标准的高性能渲染引擎。也是目前几个大厂框架内维护力度最高的库。
Kraken 的优势在于其能够基于 W3C 进行开发，而且引入 npm 生态，支持使用 Vue、React 框架开发，一般前端人员都能进行开发，学习成本很低。
github地址：[https://github.com/openkraken/kraken](https://github.com/openkraken/kraken)。

## 7 大师

## 8 宗师

## 9 王者
