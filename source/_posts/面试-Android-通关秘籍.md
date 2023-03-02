---
title: 面试 Android 通关秘籍
date: 2023-01-05 21:26:12
top: false
cover: false
toc: true
mathjax: true
tags:
- 面试题 Android
categories:
- 面试 
---

## 1 黑铁
> 知道四大组件用法和生命周期。

### 1.1 Activity生命周期？
> onCreate->onStart->onResume->onPause->onStop->onDestory

### 1.2 Fragment生命周期？
> onAttach->onCreate->onCreateView->onActivityCreate (对应Activity的onCreate阶段)  
---> onStart->onResume->onStop   
---> onDestoryView->onDestory->onDetach (对应Activity的onDestroy阶段)

### 1.3 横竖屏切换对Activity生命周期的影响？
 > 1.不设置Activity的android:configChanges时，切屏会重新调用各个生命周期,测试不管怎么切换，都只走一次生命周期。具体生命周期如下：（使用Android12亲测，根网上说切横屏走2次不同，猜测可能跟系统版本有关）
 onPause()->onSaveInstanceState()-> onStop()->onDestroy()->
 onCreate()->onStart()->onRestoreInstanceState->onResume()<br>
 2.给Activity设置了android:configChanges="orientation|keyboardHidden|screenSize"这三个属性，切屏不会走生命周期，只会走onConfigurationChanged方法。<br>
 3.如果设置了固定屏幕方向，如android:screenOrientation="portrait"，将不会有任何生命周期回调。

### 1.4 按home键盘、切到桌面后再切回来？
> onPause()->onStop()->onRestart()->onStart()->onResume() 
这里需要提到Activity是否被回收，回收的话需要重新走onCreate（这样就不走onRestart了）

### 1.5 Android的启动模式？
> 必须提到4种启动模式。
1.标准模式standard：需要提到每次都重新创建一个Activity实例即可。（最好是能提到在服务中启动需要开启一个新的任务栈）<br>
2.栈顶复用模式(singleTop): 需要提到一定是栈顶复用，栈顶没有则新建Activity实例，跟栈内无关（最好是能提到一般可用于解决连续两次快速点击会跳转相同Activity）。<br>
3.栈内复用模式(singleTask): 需要提到当前栈内只要有这个Activity实例，都会复用，而且一定要提到清除上方所有的其他Activity实例。需要提到如果复用的话，会走onNewIntent回调方法。（最好是能提到taskAffinity,这个是表示当前Activity期望进入的栈，最好是能提到场景一般用于App首页）<br>
4.单例模式(singleInstance): 需要提到开启这个单例模式的Activity，会创建一个新的任务栈，将此Activity压入栈中。（最好是能提到手机的呼叫来电，就是使用的单例模式）

### 1.6 前台栈和后台栈交互逻辑？
> 如果前台栈是 AB，后台栈是CD（CD均为singleTask模式）。 
第一种情况，需要跳转D，此时任务栈会把C也带过来，任务栈变成：ABCD。
第二种情况，需要跳转C，此时会clear掉上方的D，任务栈变成：ABC。

### 1.7 服务可以直接执行耗时任务吗？
> 不可以哦。
服务虽然是后台运行的组件，但是它同样也是运行在UI线程中。当然它也可以开启子线程处理耗时任务，而且它很适合处理不需要跟用户交换而且要长期运行的任务，比如下载文件（最好是能提到IntentService，内部实现了任务排序，开启线程的工作，且任务执行完会停止服务）。

### 1.8 服务的分类？
> 1.按照运行地点分类：
1.1 本地服务，同进程的服务。
1.2 远程服务，不同进程的服务。<br>
2.按照运行类型分类：
2.1 前台服务，有通知栏提示。
2.2 后台服务，没有通知，用户无感知。

### 1.9 服务的生命周期？
> 1.startService启动的服务的生命周期
onCreate->onStartCommand->onDestroy   通过context的startService，传intent启动；通过context.stopService，传intent关闭服务。（最好是能提到重复启动不会走多次onCreate,会走onStartCommand）<br>
2.bindService启动的服务的生命周期
比如我们通过在Activity里面bindService开启服务：
2.1 onCreate->onBind ->然后走 我们在启动时注入的ServiceConection接口的 onServiceConnected回调函数
此时我们调用stopService是管不了服务的。
2.2 如果调用unBindService，此时会走：
onUnbind->onDestroy
(最好能提到服务通过bind开启，后续重复开启，将没有任何回调！)<br>
3.混合型
同时startService+bindService的生命周期：
onCreate->onStartCommand->onBind->走onServiceConnected回调
此时如果仅仅通过unBindService，服务会走onUnBind，但不会走onDestroy。
仅仅通过stopService，也无法关闭服务。
需要stopService+unbindService，才能停止服务。

### 1.10 如何创建前台服务？
> 调用服务的startForeground方法，里面有个参数可以传Notification。
如何关闭前台服务呢，先通过stopForeground，降级为后台服务，在通过stopService关闭服务。

### 1.11 服务的生命周期会受到Activity的生命周期影响吗？
> 如果Service是通过startService()方法启动的，那么它将继续运行，直到通过stopService()方法停止。此时，Service的生命周期不受Activity的生命周期影响。<br>
如果Service是通过bindService()方法绑定到Activity上的，那么它将会受到Activity的生命周期的影响。当Activity销毁时，它会解除与Service的绑定，从而导致Service的onUnbind()方法被调用，接着如果没有其他组件绑定到该Service，则Service的生命周期将终止，最终调用onDestroy()方法。

### 1.12 广播是干什么用的？
> 能简单提一下原理：采用了观察者模式，基于消息的发布/订阅事件模型。
主要用于不同组件之间（能提到应用内或者跨进程）相互通信，也能接收到系统的发出广播。
如果能提到原理是最佳的：
1.广播接收者 通过 Binder 机制在 AMS 注册
2.广播发送者 通过 Binder 机制向 AMS 发送广播
3.AMS 根据 广播发送者 要求，在已注册列表中，寻找合适的广播接收者
  寻找依据：IntentFilter / Permission
4.AMS将广播发送到合适的广播接收者相应的消息循环队列中；
5.广播接收者通过 消息循环 拿到此广播，并回调 onReceive()

### 1.13 广播的onReceive方法可以处理耗时任务吗？
> 不建议在这里处理耗时任务，很容易导致ANR。如果真的需要，建议在onReceive方法里面开启一个IntentService来处理。

### 1.14 广播有哪几种注册方式呢？
> 能提到静态广播和动态广播即可。
最好是能提到Android8.0对静态广播的限制：1.取消了大部分静态注册广播 2.无法接收隐式启动的广播。

### 1.15 使用广播注意点？
> 一定要注意在onPause或者onStop中unRegisterReceiver。

### 1.16 广播有哪些类型？
> 1.自定义广播： 开发者自己定义
2.系统广播：系统定义的，比如分钟变更啥的
3.有序广播：发送时采用sendOrderedBroadcast
4.应用内本地广播：不需要跨进程的话，建议使用这个LocalBroadcastManager类处理。

### 1.17 ContentProvider怎么使用？
> 能提到底层Binder最佳。
1.自定义一个ContentProvider，内部先通过UriMatcher注册一个唯一标识。
2.ContentProvider提供了增删改查的接口方法，我们需要在这里面实现自己的增删改查（可以委托给SQLiteOpenHelper来处理）。
3.在清单文件注册
4.使用时，通过getContentResolver来访问。

### 1.18 Activity怎么和Fragment通信？
> 先提一下如果这个Fragment是依附这个Activity，那么可以直接通过引用直接访问public方法（没有引用也可以通过Tag获取到实例）。
但为了降低耦合性和增加复用性，建议可以统一以下方法优化：
1.自定义接口
2.Bundle: 通过getArguments接收。
3.广播或者EventBus。
4.通过ViewModel（建议）：可以实现共享。

### 1.19 kotlin let also apply with run怎么用？
> let: let it go,就是让它来，扩展函数，括号里面就是让它来做什么事情，可以执行它自己的函数，返回值为最后一行。
also: also let it go，也让它来，扩展函数，也让它来，说明它很牛逼，返回值就是它自己了。
with: with this object，和这个对象，非扩展函数，里面可以直接访问这个对象的成员变量，返回值为最后一行。
run: let it come, run with this object, 让她来，和这个对象一起跑， 扩展函数，里面可以直接访问成员变量，返回值为最后一行。
apply: apply run with this object，扩展函数，与run类似，但最终结果不同，返回的是他自己，对象跑了，留他孤身走暗巷。


## 2 青铜
> 多线程编程，会用三方库，懂得适配，了解常用UI组件。

### 2.1 Android消息机制原理？
> 首先一定要解释下是什么：Android多线程相互通信。
一定要从4个角色上进行分析：
Handler: 消息机制的调度器，可用于发送消息和处理消息。
Message: 消息对象，在多线程之间传递的消息对象。
MessageQueue: 消息队列，一个具有优先级的消息队列，以单链表形式存在。
Looper: 通过looper函数创建一个死循环，不断轮训消息，分发到消息处理器中。

### 2.2 为什么Looper一直循环，不会卡死？
> 当没有消息的时候，会阻塞在queue.next()里面的nativePollOnce方法
这个方法从native层将线程挂起，释放CPU资源进入休眠状态。
另外，Android本身就是事件驱动，Looper.loop不断接收和处理事件，Looper就是支持Android应用能跑起来的关键，一般而言，不会说是looper循环导致卡顿，而是在消息处理里面做耗时操作，才会导致卡顿。

### 2.3 IntentService原理？

### 2.4 HandlerThread原理？

### 2.5 Kotlin协程怎么用？
> 首先一定要回答是什么：Kotlin 协程是一种轻量级的并发框架，可用于处理异步和非阻塞操作，而无需使用回调或显式线程管理。
然后提一下作用域，可以全局开，也可以在Activity里面自己创建一个作用域，或者使用ViewModel的作用域，通过launch函数即可开启协程。


## 3 白银
> 能够处理基本的性能优化。

### 3.1 Android 事件分发机制？

### 3.2 Android 绘制原理？

### 3.3 Android进程优先级？内存不足时怎么回收？

### 3.4 LruCache原理？

## 4 黄金
> 了解系统API原理。

### 4.1 Android Binder原理？

### 4.2 Android IPC方案和Linux IPC方案？

## 5 铂金
> 对热修复，插件化，组件化有过深入分析。

## 6 钻石
> 熟悉常用三方库原理。

## 7 大师
> 对AMS,PMS有一定深入分析。

## 8 宗师
> 熟悉Framework。

## 9 王者
> 对ndk，openGL，framework有自己独特理解。
