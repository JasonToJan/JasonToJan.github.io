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

### 2.3 HandlerThread原理？
> 先说下是什么：HandlerThread是Google帮我们封装好的异步处理工具，可以用来执行多个耗时操作，而不需要多次开启线程，里面是采用Handler和Looper实现的。
1.首先它继承了线程，那么一定就不是在主线程了，且这里面创建了一个自己的looper，开启了循环。
2.原理跟Android的主线程创建的Looper比较相似，可以说是简化版。

### 2.4 IntentService原理？
> 最好先回答下是什么：IntentService是一种特殊的Service，它继承了Service，因此它自然会比单纯的线程优先级要高，然后它自己依旧是一个抽象类，需要创建自己的子类。 IntentService可用于执行后台耗时任务，执行完毕后会自动停止。
1.提一下在这个服务的onCreate里面，new了一个HandlerThread，然后start。
2.自定义Handler叫做ServiceHandler, 然后将前面工作线程的Looper传到这个Handler中。
3.然后在Hander的handleMessage中，定义了抽象方法onHandeIntent交给子类处理。
4.任务通过onStartCommand中依次通过hander发送消息到子线程里面去处理。

### 2.5 Kotlin协程怎么用？
> 首先一定要回答是什么：Kotlin 协程是一种轻量级的并发框架，可用于处理异步和非阻塞操作，而无需使用回调或显式线程管理。
然后提一下作用域，可以全局开，也可以在Activity里面自己创建一个作用域，或者使用ViewModel的作用域，通过launch函数即可开启协程。

### 2.6 为什么AsyncTask被遗弃了？
> 1.容易导致内存泄漏
2.并发性能，默认是顺序执行的。
3.有更好的替代方案，比如RxJava，协程等。

### 2.7 如何适配？
> 为什么要适配？需要回答下Android碎片化问题。
1.布局控件选择，LinearLayout权重，ConstraintLayout约束布局。
2.dimens动态化。
3.资源限定符布局文件。
4.使用三方库，如头条的AndroidAutoSize，最好能说下原理：通过修改设备的屏幕密度来实现适配。
最好是能简单说下dp是独立于设备屏幕密度的像素单位；dpi是每英寸的像素数；density是单位长度内的像素数；标准屏幕的dpi是160，xxhdpi对应的density为3，也就是1个dp约等于3个px；

### 2.8 Android进程优先级？
> 1.前台进程，正常和用户交互的进程。
2.可见进程，比如一个弹框弹出，后面的是可见的。
3.服务进程，已经启动的Service。
4.后台进程，切换到后台了。

### 2.9 Android进程间通信？
> 1.直接使用intent可携带数据。
2.文件共享，如sp文件。
3.Messenger，底层AIDL。
4.AIDL文件。
5.ContentProvider，底层Binder+AIDL。
6.Socket。
最好是提到Linux操作系统直接的IPC方案：可以共享内存，管道（亲缘关系），消息队列，信号量。

### 2.10 RecyclerView复用机制？
> 1.一定要提到ViewHolder，理论上可以说ViewHolder就是itemView，一个itemView对应一个ViewHolder，一对一的关系，而我们后面多级缓存复用的就是这个ViewHolder。
2.最好是能提到四级缓存，
Srap（ 缓存屏幕中可见范围的ViewHolder）
Cache（缓存滑动时即将与RecyclerView分离的ViewHolder）
ViewCacheExtension（开发者自行实现的缓存）
RecycledViewPool（ViewHolder缓存池，本质上是一个SparseArray，）
缓存对象都是ViewHolder。

## 3 白银
> 能够处理基本的性能优化，测试相关。

### 3.1 Android 事件分发机制？
> 先说下事件分发的本质：将点击事件向某个View进行传递并最终得到处理。
然后需要提到：在哪些对象传递，Activity->ViewGroup->View。
然后需要具体分析下三个重要方法：dispatchTouchEvent-> onInterceptTouchEvent->onTouchEvent。
最好分析下三个返回值：
dispatchTouchEvent，返回true，不传了；返回false，不传了，返回给上层onTouchEvent处理；默认走super.dispatchTouchEvent；
onInterceptTouchEvent，需要提到只有ViewGroup才有，主要作用是拦截，返回true，交给自己onTouchEvent处理；返回false，不拦截，加个子View处理。
onTouchEvent: 处理事件，返回true，消费；false回传给上层onTouchEvent。
可以幽默地说有点V。

### 3.2 滑动冲突处理？
> 先说下：一般是有两种方案，外部拦截法，内部拦截法。
1.外部拦截法，直接在指定ViewGroup中重写onInterceptTouchEvent，如果要拦截，返回true。
2.内部拦截费，在目标View的dispatchTouchEvent，禁止外部拦截，调用parent.requestDisallowInterceptTouchEvent(true);

### 3.3 Android 绘制原理？
> 重点提一下三个过程即可。
测量Measure: 必须必须提一下测量模式，用了2位表示3种测量模式，Exactly,精确的，对应matchparent和写死的长度；AT_MOST,不确定的，一般对应wrap_content；还有一种用得比较少，UNSPECIFIED对应无限制的，如ScrollView。
布局阶段layout: 重点提下是干啥的，确定四个坐标点，通过setFrame方法，自上而下确定自身位置。先确定自己，再确定子View。
绘制阶段onDraw: 重点提下过程即可。先绘制背景，再自己，在子View，最后装饰。

### 3.5 LruCache原理？
> 说下核心思想即可：
当缓存满的时候，会优先淘汰那些最近最少使用的缓存对象。
最好说下数据结构：LinkedHashMap是由数组+双向链表的数据结构来实现的.
然后就是维护一个缓存对象列表，其中对象的排列方式是按照访问顺序实现的。即一直没访问的对象，将放在队尾。

### 3.6 卡顿的根本原因？
> 主线程有耗时操作，导致doFrame方法没有在垂直同步信号发出后的16ms内完成.

### 3.7 UI优化？
> 1.布局优化，merge标签减少不必要的层级，ViewStub按需加载；include复用。
2.层级优化，可以用ConstraintLayout减少层级。
3.使用最新稳定的视图控件，比如ViewPage2，RecyclerView这种google比较推荐的控件。
4.如果是一些间隙，可以用Space代替View，因为这个draw为空了。

### 3.8 内存优化？
> 1.优化图片资源，这个最重要，一定要提到，图片占用空间比较大，可以采用压缩图片。
2.减少内存泄漏，可以通过leakCanary检测分析，弱引用等。
3.使用google推荐的数据库：如用SparseArray替代HashMap等，用Pair等。
4.少用枚举类，占用内存较大。
5.利用工具分析Android Profiler可以检测内存占用率。
6.减少内存抖动，会加剧系统GC，创建大量临时对象会导致内存抖动。
7.onDraw里面避免创建对象，会频繁调用。
8.大图局部加载方案，BitmapRegionDecoder。

### 3.9 内存泄漏原因？
> 本质原因：长生命周期中持有短生命周期引用，短生命周期销毁，长生命周期仍然持有短生命周期引用，如
场景说一下：
1.资源对象未关闭：Cursor，File对象。
2.注册的对象未销毁：广播，EventBus，回调监听。
3.类的静态变量持有大量数据。
4.非静态内部类，持有外部类引用。
5.Handler
6.容器的对象没有清理
7.WebView使用单进程。

### 3.10 你做过哪些性能优化？
> 从“快省稳小”四个角度出发。
快
1.1布局优化
1.2绘制优化
1.3内存优化
1.4启动速度优化
1.5线程优化
省
2.1省电
2.2省流量
稳
3.1ANR修复
3.2Crash修复
小
4.1安装包小

### 3.11 单元测试和自动化测试有了解过吗？
> 单元测试，
用过powermock对于静态方法的mock，
用过mocktio主要mock对象，模拟控制方法返回值，监控方法调用；
用过robolectric,模拟了Android系统和和核心库。
自动化测试：
用过uiAutomator
用过Espresso，google开源，体积小

## 4 黄金
> 了解系统API原理和解决项目难点和安全相关问题，音视频等。

### 4.1 Android Binder原理？
> 一定要说是什么先：Android独有的多进程相互通信。基于C/S架构。
前提最好说一下：Android 每个应用进程都有一个虚拟地址空间，虚拟地址空间又分为用户空间和内核空间，内核空间是共享的，为多进程通信提供了前提条件。
然后再提一下有几个重要角色：客户端，服务端，Binder驱动，Service Manger
然后分析下每个角色的作用：
1.客户端和服务端：这个相对的概念，客户端就是使用服务的进程，服务端就是提供服务的进程。
2.ServiceManager: 主要是将字符形式的Binder名字转换成客户端对该Binder的引用。
3.Binder驱动：负责进程之间Binder通信的建立，给数据交互提供底层支持。
最后分析一下Binder运行机制：
1.注册服务，就是服务进程需要注册服务到Service Manager。
2.客户端获取服务，客户端进程向Binder驱动发起获取服务请求，Binder驱动转发给ServiceManager，ServiceManager将Binder代理对象回传给Binder驱动，Binder驱动再回传给客户端。
3.使用服务，Binder驱动使用mmap函数实现内存映射，创建了接收缓冲区，数据传递：客户端进程拷贝数据到内核缓冲区，然后内核缓冲区和接收缓冲区存在映射关系，接收进程可以直接获取到缓冲区数据。

### 4.2 彻底理解ANR?
> 首先最好用比喻描述下：有点像埋一个定时炸弹，到时间了就爆炸，结果成功拆弹就不会爆炸了。
1.埋炸弹：在系统服务进程里面，启动一个倒计时，用户输入事件是5s，ContentProvider和广播是10s，服务是20s。
2.拆炸弹：在规定时间内处理完事件，并及时向系统服务进程报告，请求解除炸弹，就不会引爆了。
3.引爆炸弹：当没有在规定时间处理完事件，系统服务进程会搜集各个进程占用资源情况，已经相关日志，便于分析原因，然后就弹出ANR提示框。

### 4.3 Android Crash处理流程？
> 目标应用在创建的时候会有一个默认的未捕获异常的Handler，当然可以自定义处理，如果没有则会走默认实现。
1.调用ActivityManagerNative里面的一个handleApplicationCrash方法，通过binder ipc机制，传递到系统服务进程。
2.到了系统服务进程，继续走到AMS底handleApplicationCrash
3.然后此时AMS会忽略当前应用的广播，冻结该进程的屏幕消息。
4.然后通过UIHandler发送一个错误消息，弹出crash对话框。
5.此时系统服务进程完成。走到Crash进程中，它主要用来杀掉当前进程的操作。
6.当进程被杀，通过binder死亡通知，告知系统服务进程做一些回收工作。

### 4.4 Native崩溃如何处理？
> 先提一下过程：native crash时，操作系统会向进程发送一个signal信号，崩溃data下某个文件下面。
如何定位：可以使用ndk工具addr2line工具或者ndk-stack来定位。
xCrash: 捕获native崩溃。
addr2line可以干嘛：根据地址进行一个符号反解的过程。

### 4.5 三方库报错？
> 可以提一下用了一个弹框的三方库。
主要是分析源码，定位问题点，然后我们通过跳转弹出弹框时机解决了。（从Activity销毁导致宿主为空，然后延后调出时机）

### 4.6 网络超时30s，loading结束并提示？
> 因为我们loading用了一个三方库，每个请求都是单独弹出loading，而且设置okHttp超时时间不是很准。
我就自己封装了一个工具类，借鉴了Glide用一个空的Fragment绑定生命周期，内部包装了handler，每个请求计算时间，超过60s后，处理逻辑。

### 4.7 百度地图so库崩溃问题？
> 魅族的时候，快应用引擎内部接入了一个百度地图so库，某些手机一启动就崩溃。
每次修改底层源码，重新编译需要2个多小时，前前后后编译了10几次，不停尝试，发现是内部有一个ssl.so库跟系统的有冲突。（这个是有关安全校验方面的库）
网上有一个解决方案，就是将系统自身的ssl.so库公开，让所有的应用优先从这么加载，试了下，的确可以运行。但会有一个潜在的风险，就是其它第三方系统应用不排除也有用ssl.so，果然发现招商银行app启动就闪退，经过考量，我们决定将百度地图内部的ssl库去除，改为使用系统的，但是Android5的机型使用系统的也会加载不出地图来。后来我们决定放弃这部分低端机型，如果采用动态加载方案，到时候对用户体验不好。通过excludeArr技术将ssl.so去除了。

### 4.8 提高app安全性的方法？
> 1.代码混淆，一些密钥数据集成到so中。
2.完整性校验，校验签名，防止重打包。
3.接口重要数据加密。

### 4.9 谈谈你对安卓签名的理解？
> 提到v1和v2即可。
Android 7.0 引入了V2签名。
v1的话是通过zip条目进行验证（这样不安全，还是可以修改）
v2:是整个apk进行分开摘要加密，更加安全。

### 4.10 播放器原理？
> 视频播放原理：（mp4、flv）-> 解封装 -> （mp3/aac、h264/h265）-> 解码 -> （pcm、yuv）-> 音视频同步 -> 渲染播放
音视频同步：选择参考时钟源：音频时间戳、视频时间戳和外部时间三者选择一个作为参考时钟源（一般选择音频，因为人对音频更敏感，ijk 默认也是音频）
通过等待或丢帧将视频流与参考时钟源对齐，实现同步<br>
bilibili的ijkPlayer原理：
集成了 MediaPlayer、ExoPlayer 和 IjkPlayer 三种实现，其中 IjkPlayer 基于 FFmpeg 的 ffplay
音频输出方式：AudioTrack、OpenSL ES；视频输出方式：NativeWindow、OpenGL ES

### 4.11 有做过直播吗？
> 视频直播的流程可以分为如下几步： 采集 —>处理—>编码和封装—>推流到服务器—>服 务器流分发—>播放器流播放
如何保证流程效果：
1.编码优化，确保CodeC开启最低延迟设置。
2.传输协议优化，尽量使用RTMP而非HTTP的HLS协议。
3.传输网络优化
4.推流（推送视频流）和播放优化

## 5 铂金
> 对热修复，插件化，组件化有过深入分析。

### 5.1 MVP模式？
> 一定要说这是一种架构思想。
有3部分组成：View负责显示，Presenter负责逻辑处理，Model提供数据。
View:负责绘制UI元素、与用户进行交互(在Android中体现为Activity)。
Model:负责存储、检索、操纵数据(有时也实现一个Model interface用来降低耦合)。
Presenter:作为View与Model交互的中间纽带，处理与用户交互的逻辑。
View interface:需要View实现的接口，View通过View interface与Presenter进行交互，降低耦合，方便使用MOCK对Presenter进行单元测试。
缺点：接口写很多，不利于维护。

### 5.2 MVVM模式？
> 可以说是MVP模式升级版。
viewModel取代了Presenter。
并引入了DataBinding，可以将ViewModel中的数据同步到View层。对View层依赖更少。

### 5.3 热修复和插件化原理？
> 热修复主要是替换dex文件。
插件化主要是扩展应用程序功能。
热修复的原理：理就是将补丁 dex 文件放到 dexElements 数组靠前位置，这样在加载 class 时，优先找到补丁包中的 dex 文件，加载到 class 之后就不再寻找，从而原来的 apk 文件中同名的类就不会再使用，从而达到修复的目的。
插件化原理：PathClassLoader只会加载已安装包中已经的dex文件，而DexClassLoader不仅仅可以加载 dex文件，还可以加载jar、apk、zip文件中的dex。
资源的插件化和热修复的资源修复都借助了AssetManager。
so的插件化方案和so热修复的第一种方案类似，就是将so插件插入到NativelibraryElement数组中，并且将存储so插件的文件添加到nativeLibraryDirectories集合中就可以了。

### 5.4 有哪些热修复框架？
> 1.类加载方案
1.1 QQ空间的Nuwa是按照上面说的将补丁包放在Element数组的第一个元素得到优先加载。
1.2 微信的Tinker将新旧APK做了diff，得到path.dex，再将patch.dex与手机中APK的classes.dex做合并，生成新的classes.dex，然后在运行时通过反射将classes.dex放在Elements数组的第一个元素。
2.底层替换方案
2.1 AndFix采用的是替换ArtMethod结构体中的字段，这样会有兼容性问题，因为厂商可能会修改ArtMethod结构体
2.2 Sophix采用的是替换整个ArtMethod结构体，这样不会存在兼容问题。
3.Instant Run方案
3.1 美团的Robust: 在第一次构建APK时，利用ASM（java字节码操控框架），能够动态生成类或者增强现有类的功能。

### 5.5 有哪些插件化框架？
> 主要是利用DexClassLoader可以加载dex文件的机制。（最好提一下双亲委派模式）
1.1 360的RePlugin和DroidPlugin: hook系统实现插件化
1.2 滴滴的VirtualApk: Hook IActivityManager和Hook Instrumentation。主要方案就是先用一个在AndroidManifest.xml中注册的Activity来进行占坑，用来通过AMS的校验，接着在合适的时机用插件Activity替换占坑的Activity
1.3 腾讯Shadow: 零反射，本质上使用代理分发生命周期实现四大组件动态化。使插件框架的代码成为了插件的一部分。
如果加载的插件不需要和宿主有任何耦合，也无须和宿主进行通信，比如加载第三方App，那么推荐使用RePlugin，其他情况推荐使用Shadow。

### 5.6 组件化方案？
> 每个模块作为1个module，通过配置不同Manifest清单文件，可以单独跑起来，也可集成编译。
节省开发成本，分工明确，提升开发效率，方便管理，耦合性低。
跨组件通信可以采用Arouter:
ARouter维护了一个路由表Warehouse，其中保存着全部的模块跳转关系，ARouter路由跳转实际上还是调用了startActivity的跳转，使用了原生的Framework机制，只是通过apt注解的形式制造出跳转规则，并人为地拦截跳转和设置跳转条件。

### 5.7 了解过编译插桩技术吗？
> 是一种在编译期间向代码中插入特定代码的技术，以实现某些功能或分析和监视代码执行情况。该技术通常用于调试、性能分析、错误检测、代码覆盖率分析、安全检测等领域。
Android可以用来 按钮防止抖动，通过自定义Gradle插件来实现。

### 5.8 如何提升编译速度？
> 1.使用最新的gradle插件
2.减少打包的资源文件
3.禁用PNG图片，开发模式下禁用
4.开启org.gradle.caching = true (利用缓存)

## 6 钻石
> 熟悉常用三方库原理。

### 6.1 OkHttp原理？
> 一定要说是干啥的先：Android常用的网络请求库，封装多种请求方式，支持多种场景。
优点可以提一下：
1.提供了Http2和SPDY的支持。（SPDY是google设计的单个TCP连接实现多个http请求和响应的一个协议，HTTP2是基于SPDY衍生的）
2.OkHttp使用连接池复用连接以提升效率。
3.提供对GZIP（数据压缩算法）的默认支持来降低传输内容大小。
4.提供了Http相应的缓存机制，避免不必要的请求。
5.网络出现问题，会自动重试一个主机的多个IP地址。<br>
请求流程（必须说）：
使用OkHttp会在请求的时候初始化一个Call的实例，然后执行它的execute(同步)方法或enqueue(异步)方法，
内部最后都会执行到getResponseWithInterceptorChain()方法，
这个方法里面通过拦截器组成的责任链，依次经过用户自定义普通拦截器、重试拦截器、桥接拦截器、缓存拦截器、连接拦截器和用户自定义网络拦截器以及访问服务器拦截器等拦截处理过程，来获取到一个响应并交给用户。<br>
然后最后简明地说下这几个拦截器作用：
1.interceptors：用户自定义拦截器
2.retryAndFollowUpInterceptor：负责失败重试以及重定向
3.BridgeInterceptor：请求时，对必要的Header进行一些添加，接收响应时，移除必要的Header
4.CacheInterceptor：负责读取缓存直接返回（根据请求的信息和缓存的响应的信息来判断是否存在缓存可用）、更新缓存
5.ConnectInterceptor：负责和服务器建立连接
6.networkInterceptors：用户定义网络拦截器
7.CallServerInterceptor：负责向服务器发送请求数据、从服务器读取响应数据<br>
连接池工作机制（可以说下）：
1.判断连接是否可用，不可用则从ConnectionPool获取连接，ConnectionPool无连接，创建新连接，握手，放入ConnectionPool。
2.使用连接复用省去了进行 TCP 和 TLS 握手的一个过程。

### 6.2 Retrofit原理？
> 一定要说是干啥的先：基于RESTful的网络请求框架，底层请求交给okhttp，自己专注于上层接口封装。
优点可以提一下：
1.支持RxJava，支持多种数据格式解析
2.简单一用，通过注解配置参数
3.可扩展性好，可自定义Converters<br>
核心（务必要说）： 采用了动态代理技术，将注解解析成参数，将接口转换成一个Request。
create方法中采用动态代理模式（通过访问代理对象的方式来间接访问目标对象）实现接口方法，这个过程构建了一个ServiceMethod对象，根据方法注解获取请求方式，参数类型和参数注解拼接请求的链接，当一切都准备好之后会把数据添加到Retrofit的RequestBuilder中。然后当我们主动发起网络请求的时候会调用okhttp发起网络请求，okhttp的配置包括请求方式，URL等在Retrofit的RequestBuilder的build()方法中实现，并发起真正的网络请求。<br>

### 6.3 EventBus原理？
> 干啥的： EventBus是一个Android发布/订阅事件总线，简化了组件间的通信。
优点可以提一下：解耦。
核心逻辑：一定要说：
EventBus最核心的逻辑就是利用了 subscriptionsByEventType 这个重要的列表，将订阅对象，即接收事件的方法存储在这个列表，发布事件的时候在列表中查询出相对应的方法并执行。<br>
然后可以添油加醋：
（1）EventBus是通过注解+反射来进行方法的获取的
注解的使用：@Retention𝑅𝑒𝑡𝑒𝑛𝑡𝑖𝑜𝑛𝑃𝑜𝑙𝑖𝑐𝑦.𝑅𝑈𝑁𝑇𝐼𝑀𝐸
表示此注解在运行期可知，否则使用CLASS或者SOURCE在运行期间会被丢弃。
通过反射来获取类和方法：因为映射关系实际上是类映射到所有此类的对象的方法上的，所以应该通过反射来获取类以及被注解过的方法，并且将方法和对象保存为一个调用实体。
2）使用ConcurrentHashMap来保存映射关系

### 6.4 Glide原理？
> 干啥的：图片加载框架。还有哪些（picasso(小)，fresco(大)）
优点可以提一下：
支持 GIF 图片播放。
支持缩略图的生成。
支持自定义的网络请求缓存策略。
能够有效地防止图片变形和拉伸。<br>
然后才说原理：
1.Glide的with方法
1.1 初始化各式各样的配置信息（包括缓存，请求线程池，大小，图片格式等等）以及glide对象。
1.2 将Glide请求和某个Fragment生命周期绑定<br>
2.Glide.load方法
主要是设置请求url。
3.Glide.into方法
3.1 首先根据转码类transcodeClass类型返回不同的ImageViewTarget：BitmapImageViewTarget、DrawableImageViewTarget。
3.2 递归建立缩略图请求，没有缩略图请求，则直接进行正常请求。
3.3 如果没指定宽高，会根据ImageView的宽高计算出图片宽高，最终执行到onSizeReay()方法中的engine.load()方法。
3.4 engine是一个负责加载和管理缓存资源的类<br>
三级缓存最好说一下：
1.LRU算法缓存内存，强引用，最近最少使用淘汰算法，拿的时候先从这里拿，拿不到就继续，拿到了会从LRU列表中删除，放到activeResource里面。（这里存放的是目前没有正在使用的资源）
2.弱引用，叫做activeResource（活动资源）的弱引用，判断这里有没有，有的话，会将这个引用计数器+1，如果还是拿不到，就继续走磁盘，当引用计数器为0的时候，会移动到LRU里面，（这个列表存放的是正在使用的资源）
3.磁盘缓存，disLRUCahce
4.都没有，就请求网络，然后先存弱引用，再存LRU，最后磁盘。

### 6.5 RxJava原理？
> 一定要说是干啥的先：RxJava2是一个基于观察者模式和迭代器模式的响应式编程库，它提供了一种基于事件流的编程方式，可以方便地处理异步、并发和事件驱动的编程场景。
实现与原理一定要说：被观察者 （Observable） 通过 订阅（Subscribe） 按顺序发送事件 给观察者 （Observer）， 观察者（Observer） 按顺序接收事件 & 作出对应的响应动作。
说下这几个角色意义就差不多了：
1.Observable：俗称被订阅者，被订阅者是事件的来源，接收订阅者(Observer)的订阅，然后通过发射器(Emitter)发射数据给订阅者。
2.Observer：俗称订阅者，注册过程传给被订阅者，订阅者监听开始订阅，监听订阅过程中会把Disposable传给订阅者，然后在被订阅者中的发射器(Emitter)发射数据给订阅者(Observer)。
3.Emitter：俗称发射器，在发射器中会接收下游的订阅者(Observer)，然后在发射器相应的方法把数据传给订阅者(Observer)。
4.Consumer：俗称消费器，这是RxJava2.0才出来的，在RxJava1.0中用Action来表示，消费器其实是Observer的一种变体，Observer的每一个方法都会对应一个Consumer，比如Observer的onNext、onError、onComplete、onSubscribe都会对应一个Consumer。
5.Disposable：是释放器，通常有两种方式会返回Disposable，一个是在Observer的onSubscribe方法回调回来，第二个是在subscribe订阅方法传consumer的时候会返回。<br>
一般会问：RxJava中map、flatMap的区别，你还用过其他哪些操作符?
map是通过原始数据类型返回另外一种数据类型，而flatMap是通过原始数据类型返回另外一种被观察者。
1.创建型操作符：create,just,fromArray,defer,range
2.变换操作符：map,flatMap,concatMap,buffer(定期从被观察者发送的事件中获取一定数量的事件并放到缓存区中，然后把这些数据集合打包发射)
3.合并操作符：concat,merge,zip,combine
4.功能性操作符：delay,do,retry,repeat
5.过滤操作符：filter,take,debounce,elementAt
6.条件操作符：exist,contains,skipWhile等

### 6.6 LeakCanary原理？
> 是什么必须说：内存泄漏检测工具。<br>
流程一定要说清楚：
1、RefWatcher.watch()创建了一个KeyedWeakReference弱引用用于去观察对象。LeakCanary通过在应用程序中注入一个监听器，监测对象的引用关系。
2、然后，在后台线程中，它会检测引用是否被清除了，并且是否没有触发GC。
3、如果引用仍然没有被清除，那么它将会把堆栈信息保存在文件系统中的.hprof文件里。
4、HeapAnalyzerService被开启在一个独立的进程中，并且HeapAnalyzer使用了HAHA开源库解析了指定时刻的堆栈快照文件heap dump。
5、从heap dump中，HeapAnalyzer根据一个独特的引用key找到了KeyedWeakReference，并且定位了泄露的引用。
6、HeapAnalyzer为了确定是否有泄露，计算了到GC Roots的最短强引用路径，然后建立了导致泄露的链式引用。
7、这个结果被传回到app进程中的DisplayLeakService，然后一个泄露通知便展现出来了。<br>
简单来说就是：
通过将Activity包装到WeakReference中，被WeakReference包装过的Activity对象如果能够被回收，则说明引用可达，垃圾回收器就会将该WeakReference引用存放到ReferenceQueue中。
假如我们要监视某个activity对象，LeakCanary就会去ReferenceQueue找这个对象的引用，如果找到了，说明该对象是引用可达的，能被GC回收，如果没有找到，说明该对象有可能发生了内存泄漏。

### 6.7 GreenDao实现原理？
> 是什么必须说：是一个轻便快捷的ORM(对象映射)框架，可以将Java对象映射到数据库中。
优点可以提一下：
对Android进行了高度优化，最小的内存开销，依赖体积小，支持数据库加密。<br>
一定要说这几个类的作用，使用的时候动态生成：
DaoMaster：所有Dao类的主人，负责整个库的运行，内部的静态抽象子类DevOpenHelper继承并重写了Android的SqliteOpenHelper。
DaoSession：作为一个会话层的角色，用于生成相应的Dao对象、Dao对象的注册，操作Dao的具体对象。
xxDao（HistoryDataDao）：生成的Dao对象，用于进行具体的数据库操作。<br>
首先，它通过使用自身的插件配套相应的freemarker模板生成所需的静态代码，避免了反射等消耗性能的操作。
其次，它内部提供了实体数据的映射缓存机制，能够进一步加快查询速度。对于不同数据库对应的SQL语句，也使用了不同的DataBaseStatement实现类结合代理模式进行了封装，屏蔽了数据库操作等繁琐的细节。
最后，它使用了sqlcipher提供了加密数据库的功能，在一定程度确保了安全性，同时，结合RxJava，我们便能更简洁地实现异步的数据库操作。


## 7 大师
> 对AMS,PMS有一定深入分析。

### 7.1 Android 系统启动流程？
> 1、启动电源以及系统启动：当电源按下时引导芯片从预定义的地方（固化在ROM）开始执行，加载引导程序BootLoader到RAM，然后执行。
2、引导程序BootLoader：BootLoader是在Android系统开始运行前的一个小程序，主要用于把系统OS拉起来并运行。
3、Linux内核启动：当内核启动时，设置缓存、被保护存储器、计划列表、加载驱动。当其完成系统设置时，会先在系统文件中寻找init.rc文件，并启动init进程。
4、init进程启动：初始化和启动属性服务，并且启动Zygote进程。
5、Zygote进程启动：创建JVM并为其注册JNI方法，创建服务器端Socket，启动SystemServer进程。
6、SystemServer进程启动：启动Binder线程池和SystemServiceManager，并且启动各种系统服务。
7、Launcher启动：被SystemServer进程启动的AMS会启动Launcher，Launcher启动后会将已安装应用的快捷图标显示到系统桌面上。

### 7.2 App启动流程？
> 1、点击桌面应用图标，Launcher进程将启动Activity（MainActivity）的请求以Binder的方式发送给了AMS。
2、AMS接收到启动请求后，交付ActivityStarter处理Intent和Flag等信息，然后再交给ActivityStackSupervisior/ActivityStack 处理Activity进栈相关流程。同时以Socket方式请求Zygote进程fork新进程。
3、Zygote接收到新进程创建请求后fork出新进程。
4、在新进程里创建ActivityThread对象，新创建的进程就是应用的主线程，在主线程里开启Looper消息循环，开始处理创建Activity。
5、ActivityThread利用ClassLoader去加载Activity、创建Activity实例，并回调Activity的onCreate()方法，这样便完成了Activity的启动。

### 7.3 apk安装过程？
> 1.复制APK到/data/app目录下，解压并扫描安装包
2.将APP的dex文件拷贝到/data/dalvik-cache目录，再在/data/data/目录下创建应用程序的数据目录（以应用包名命令），用来存放应用的数据库、xml文件、cache、二进制的so动态库等
3.解析apk的AndroidManifest.xml文件，注册四大组件，将apk的权限、应用包名、apk的安装位置、版本、userID等重要信息保存在/data/system/packages.xml文件中。这些操作都是在PackageManagerService中完成。
4.资源管理器解析APK里的资源文件。
5.dex2oat操作，对dex文件进行优化，并保存在dalvik-cache目录下。
6.更新权限信息。
7.安装完成后，发送Intent.ACTION_PACKAGE_ADDED广播。
8.显示icon图标，应用程序经过PMS中的逻辑处理后，相当于已经注册好了，如果想要在Android桌面上看到icon图标，则需要Launcher将系统中已经安装的程序展现在桌面上。

### 7.4 apk打包流程？
> 1.打包资源文件，生成R.java文件（aapt工具，生成resource.arsc）
2.处理aidl文件，生成相应Java文件
3.编译java文件，生成class文件（java编译器）
4.将class文件生成dex文件（dx工具）
5.将资源文件和dex文件合并打包，生成apk文件（apkBuilder工具）
6.对apk文件进行签名
7.对前面后的文件进行对齐（zipalign,作用是减少运行时内存）

## 8 宗师
> 熟悉Framework。

## 9 王者
> 对ndk，openGL，framework有自己独特理解。
