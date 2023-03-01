---
title: Android 2022头条面经
date: 2023-01-04 18:47:45
top: false
cover: false
toc: true
mathjax: true
tags:
- Android 面试题
categories:
- Android
---

## 1 Android基础

### 1.1 请简述Android事件传递机制， ACTION_CANCEL事件何时触发？ 
> 本质： 将点击事件MotionEvent传递到某个具体的View，并消费事件。<br>
传递对象：Activity->ViewGroup->View。<br> 
在 Android 中，事件传递是指将触摸事件传递给正确的视图层级并执行相应的操作。Android 事件传递机制主要由以下三个方法组成：<br>
1.dispatchTouchEvent(MotionEvent event): 这个方法是 View 的一个方法，用于将 MotionEvent 事件分发给当前视图以及其子视图进行处理。这个方法是事件传递的第一步，事件从父视图传递到子视图，直到找到能够处理该事件的视图。<br>
在 Android 中，dispatchTouchEvent 方法是用于事件分发的一个重要方法，该方法的返回值可以影响事件分发的行为。dispatchTouchEvent 方法的返回值有以下几种情况：<br>
1).true：表示事件已被当前视图消费并处理，事件不再传递到其他视图。这个返回值通常用于拦截事件，防止事件继续向下传递。<br>
2).false：表示事件未被当前视图处理，事件将继续传递到父视图。这个返回值通常用于让父视图来处理事件，这里会调用父视图的onTouchEvent方法。<br>
3).super.dispatchTouchEvent(event)：表示调用父类的 dispatchTouchEvent 方法来处理事件。这个返回值通常用于将事件传递给子View来处理。<br>
2.onInterceptTouchEvent(MotionEvent event): 这个方法是 ViewGroup 的一个方法，用于拦截子视图的触摸事件。在 dispatchTouchEvent 方法中，如果父视图拦截了事件，那么事件将不会被传递给子视图。如果没有拦截，事件会继续向下传递。<br>
onInterceptTouchEvent() 是 Android View 体系中的一个方法，它用于拦截子 View 的触摸事件。该方法会在事件分发之前被调用，用于判断是否拦截子 View 的事件。如果返回值为 true，则表示拦截子 View 的事件，自身的 onTouchEvent() 方法会被调用；如果返回值为 false，则表示不拦截子 View 的事件，子 View 的 dispatchTouchEvent() 方法会被调用。<br>
3.onTouchEvent(MotionEvent event): 这个方法是 View 的一个方法，用于处理触摸事件。
当 onTouchevent() 返回 true 时，表示该事件已经被当前 View 处理并消耗掉了，其他的 View 不会再接收到该事件。当 onTouchevent() 返回 false 时，表示该事件没有被当前 View 处理，也会继续回传到父View的onTouchEvent。<br>
这三个方法一起构成了 Android 事件传递机制的基础。通过这些方法的重写和组合，可以实现复杂的触摸事件处理逻辑。<br>

### 1.2 简述Android的View绘制流程?
> 1.Measure：在measure阶段，View的尺寸会被计算出来。View会调用measure()方法，通过传入的MeasureSpec参数来确定自己的尺寸。MeasureSpec是由一个32位的int值组成，其中包含了一个2位的测试模式，也就是有3种，用来表示View的测量模式和测量值。根据测量模式的不同，View会有不同的行为。
1).UNSPECIFIED ：不对View进行任何限制，要多大给多大，一般用于系统内部
2).EXACTLY：对应LayoutParams中的match_parent和具体数值这两种模式。检测到View所需要的精确大小，这时候View的最终大小就是SpecSize所指定的值，
3).AT_MOST ：对应LayoutParams中的wrap_content。View的大小不能大于父容器的大小。<br>
如果父布局是精确模式，子View的布局方式如果为具体大小或者match_parent，都可以确定子View大小，那么子View对应EXACTLY。<br>
如果父布局是AT_MOST模式，就是说父View是wrap_content，那子View也无法确定，除非子View精确，其它情况都是AT_MOST模式。<br>
如果父布局是UNSPECIFIED，无限制的，比如ScrollView，除了子View精确，其它情况都是UNSPECIFIED。<br>
2.layout: 测量完View大小后，就需要将View布局在Window中，View的布局主要通过确定上下左右四个点来确定的。其中布局也是自上而下，不同的是ViewGroup先在layout()中确定自己的布局，然后在onLayout()方法中再调用子View的layout()方法，让子View布局。setFrame（）方法来确定自身的位置布局。<br>
3.draw：View的绘制过程遵循如下几步：
①绘制背景 background.draw(canvas)
②绘制自己（onDraw）
③绘制Children(dispatchDraw)
④绘制装饰（onDrawScrollBars）

### 1.3 bundle的数据结构，如何存储？
> bundle的内部结构其实是Map，传递的数据可以是boolean、byte、int、long、float、double、string等基本类型或它们对应的数组，也可以是对象或对象数组。当Bundle传递的是对象或对象数组时，必须实现Serializable 或Parcelable接口。<br>
Bundle与Intent.putExtra()方法类似，但两者之间有一些区别。Intent.putExtra()方法主要用于将数据传递给其他组件，而Bundle通常用于在同一组件的不同方法之间传递数据。<br>
此外，Bundle还有一个非常方便的特性，即它可以保存和恢复Activity实例的状态。例如，当设备旋转或内存不足时，Android系统会销毁并重新创建Activity实例。为了保持用户的数据和状态不丢失，可以将这些数据存储在Bundle中，以便在Activity重新创建后恢复它们。<br>
因此，Bundle是一种非常有用的数据结构，它提供了一种灵活的方式来传递和保存数据，并且可以方便地与Android系统的组件生命周期集成。

### 1.4 主线程中的Looper.loop()一直无限循环为什么不会造成ANR？ 
> 因为Android 的是由事件驱动的，looper.loop() 不断地接收事件、处理事件，每一个点击触摸或者说Activity的生命周期都是运行在 Looper.loop() 的控制之下，如果它停止了，应用也就停止了。只能是某一个消息或者说对消息的处理阻塞了 Looper.loop()，而不是 Looper.loop() 阻塞它。也就说我们的代码其实就是在这个循环里面去执行的，当然不会阻塞了。

## 2 Android进阶

### 2.1 Android的进程间通信？Linux系统的进程间通信？
> Android 的进程间通信 (IPC)：
Android 中的进程间通信机制主要包括：Binder、Socket、Shared Memory、File、Message Queue 和 Content Provider 等方式。<br>
1.Binder：是 Android 独有的进程间通信方式，其实现了进程间对象的跨进程共享，常用于 Service 和 Activity 之间的通信。<br>
2.Socket：基于网络通信协议实现的进程间通信方式，可以用于不同设备之间的通信，也可以用于同一设备上的进程通信。<br>
3.Shared Memory：共享内存，多个进程共享同一块物理内存区域，可以快速地进行数据交换。<br>
4.File：通过文件共享数据，进程 A 将数据写入文件，进程 B 从文件中读取数据，这种方式实现简单，但效率较低。<br>
5.Message Queue：进程间通过消息传递进行通信，主要用于同一设备上的进程通信。<br>
6.Content Provider：提供数据共享的一种方式，常用于不同应用程序之间的数据共享。<br>
Linux系统的进程间通信 (IPC)：
Linux 中的进程间通信主要有以下几种方式：<br>
1.管道（Pipe）：一种半双工的通信方式，可以在具有亲缘关系的进程之间进行通信。<br>
2.命名管道（Named Pipe）：也是一种半双工的通信方式，但与管道不同的是，可以在不具有亲缘关系的进程之间进行通信。<br>
3.信号（Signal）：用于进程之间的简单通信，一个进程发送一个信号给另一个进程，另一个进程接收信号并处理。<br>
4.消息队列（Message Queue）：是一种进程间通信机制，可以实现不同进程之间的数据传递。<br>
5.共享内存（Shared Memory）：多个进程共享同一块物理内存区域，可以快速地进行数据交换。<br>
6.信号量（Semaphore）：一种进程间同步机制，可以用于解决进程之间的竞争问题。<br>
7.套接字（Socket）：是一种基于网络通信协议实现的进程间通信方式，可以用于不同设备之间的通信，也可以用于同一设备上的进程通信。

### 2.2 android的IPC通信方式有哪些？
> Binder IPC：是 Android 独有的 IPC 方式，也是 Android 中使用最广泛的 IPC 方式。Binder 是一个高效、安全的跨进程通信机制，可用于进程间数据传输和远程方法调用（RPC）。<br>
广播（Broadcast）：是一种异步的 IPC 通信方式，通过在 Intent 中包含信息，将信息广播给所有注册了相应 IntentFilter 的接收器。广播可以用于在应用程序内部或跨应用程序之间传递事件和信息。<br>
ContentProvider：是一种通过 URI（统一资源标识符）进行访问的共享数据集合，可用于在不同应用程序之间共享数据。ContentProvider 采用标准的 CRUD（创建、读取、更新、删除）操作，可以让多个应用程序共享同一个数据源。<br>
Messenger IPC：是一种基于 Binder 的封装，可用于进程间传递消息。Messenger 可以通过 Handler 向目标进程发送 Message，也可以通过 Bundle 传递数据。<br>
AIDL（Android 接口定义语言）：是一种用于生成客户端和服务器之间接口的工具。通过 AIDL，客户端可以调用服务器进程的方法，获取返回值或传递参数。

### 2.3 Android的多点触控如何传递?
> Android的多点触控是通过事件传递机制来实现的。当屏幕上发生触摸事件时，Android系统会将事件传递给当前Activity或View的事件处理程序。对于多点触控，Android会将每个触点的事件都传递给事件处理程序，以便应用程序能够根据这些事件来进行相应的操作。<br>
在Android中，多点触控事件由以下三个主要事件类型组成：<br>
ACTION_DOWN：当第一个手指按下屏幕时触发，通常用于开始处理多点触控事件。<br>
ACTION_POINTER_DOWN：当第二个及以后的手指按下屏幕时触发，每次按下都会触发一个ACTION_POINTER_DOWN事件，其中包含有关新按下手指的信息。<br>
ACTION_MOVE：当任何一个手指在屏幕上移动时触发，通常用于更新多点触控事件。<br>
每个事件都包含有关触摸点的坐标、事件类型和时间戳等信息，开发者可以根据这些信息来实现不同的多点触控操作。在事件处理程序中，可以使用MotionEvent类来获取和处理这些信息。

### 2.4 Asynctask的原理?
> Android中的AsyncTask是一种方便的异步处理方式，可以在UI线程之外执行一些耗时操作，然后将执行结果返回给UI线程。在实现上，AsyncTask基于Thread和Handler包装而成。<br>
具体来说，AsyncTask在执行时会创建一个新的线程（即Worker Thread），在这个线程中执行耗时操作。同时，它还会创建一个Handler对象，用于在UI线程中处理执行结果。<br>
AsyncTask的实现原理可以分为以下几个步骤：<br>
1.AsyncTask被调用时，首先会创建一个Worker Thread，然后在这个线程中执行doInBackground()方法。<br>
2.当doInBackground()方法执行完毕后，会将执行结果传递给onPostExecute()方法。<br>
3.在onPostExecute()方法中，会调用Handler对象的post()方法将结果发送到UI线程。<br>
4.在UI线程中，Handler会调用AsyncTask的onPostExecute()方法，将执行结果传递给它。<br>
5.最后，在onPostExecute()方法中，开发者可以处理执行结果，更新UI等操作。<br>
需要注意的是，AsyncTask还有其他一些回调方法，比如onPreExecute()、onProgressUpdate()等，这些方法可以用于在执行异步任务前、中、后执行一些操作，以及显示进度条等功能。<br>
总的来说，AsyncTask利用Thread和Handler机制，实现了方便的异步处理方式，帮助我们简化了一些耗时操作的实现。

### 2.5 Android模块化开发？
> Android 模块化开发是指将应用程序拆分为多个独立的模块，每个模块都具有自己的功能和职责。这种方式可以提高应用程序的可维护性、可测试性和可扩展性，也可以帮助多人协同开发。以下是可能的回答：<br>
什么是 Android 模块化开发？
Android 模块化开发是指将应用程序分解为多个独立的模块，每个模块都具有自己的功能和职责。这种方式可以提高应用程序的可维护性、可测试性和可扩展性，也可以帮助多人协同开发。<br>
Android 模块化开发的优点是什么？
Android 模块化开发可以提高应用程序的可维护性、可测试性和可扩展性。它可以帮助团队协同开发，因为每个模块都具有自己的功能和职责。此外，模块化开发还可以提高应用程序的性能和启动时间，因为只有需要的模块才会被加载。<br>
Android 模块化开发的实现方式有哪些？
实现 Android 模块化开发的方式有很多种。其中，使用 Android Studio 自带的 Module 功能是最常用的方式之一。使用 Module 功能可以将应用程序拆分为多个模块，并将每个模块作为一个独立的项目进行开发。另外，还可以使用 Gradle 的多项目构建功能，或者使用组件化开发框架，如阿里巴巴的 ARouter。

### 2.6 Android热更新有了解过吗？
> 热更新（Hotfix）是指在不重新发布应用程序的情况下，通过修补程序来修复应用程序中的问题。在 Android 应用程序中，常用的热更新技术包括代码注入、动态加载和差分包等。以下是可能的回答：<br>
什么是 Android 热更新？
Android 热更新是指在不重新发布应用程序的情况下，通过修补程序来修复应用程序中的问题。这种技术可以快速修复应用程序中的问题，避免用户等待应用程序的新版本发布。<br>
Android 热更新的实现方式有哪些？
实现 Android 热更新的方式有很多种。其中，比较常用的技术包括代码注入、动态加载和差分包等。<br>
动态加载：将应用程序的某些模块或代码动态加载到内存中，从而实现热更新的效果。这种方式的优点是比较安全稳定，但是也存在一定的限制，比如无法热更新应用程序的资源文件等。<br>
差分包：将应用程序的差异部分打包成一个差分包，然后在应用程序运行时下载并应用差分包。这种方式的优点是比较安全稳定，但是需要实现一定的逻辑来管理差分包的下载和应用。<br>
有哪些框架实现原理是什么？<br>
Tinker
Tinker 是腾讯开发的一种 Android 热更新框架。其实现原理是使用差分包技术，将应用程序的修改部分打包成一个差分包，然后在应用程序运行时下载并应用差分包。Tinker 的优点是可以实现类似于应用市场的增量更新功能，并且具有较高的兼容性和稳定性。<br>
AndFix
AndFix 是阿里巴巴开发的一种 Android 热更新框架。其实现原理是使用代码注入技术，在应用程序启动时动态修改需要修复的代码。AndFix 的优点是可以快速修复应用程序中的问题，但是也存在一定的风险，可能会影响应用程序的稳定性和安全性。<br>
Robust
Robust 是美团开发的一种 Android 热更新框架。其实现原理是使用动态加载技术，将需要修复的代码打包成一个补丁文件，在应用程序运行时动态加载补丁文件并应用补丁。Robust 的优点是可以实现灵活的热更新功能，并且具有较高的兼容性和稳定性。

### 2.7 Android热修复的底层原理？
> 热修复的原理就是将补丁 dex 文件放到 dexElements 数组靠前位置，这样在加载 class 时，优先找到补丁包中的 dex 文件，加载到 class 之后就不再寻找，从而原来的 apk 文件中同名的类就不会再使用，从而达到修复的目的。<br>
PathClassLoader只会加载已安装包中已经的dex文件，而DexClassLoader不仅仅可以加载 dex文件，还可以加载jar、apk、zip文件中的dex。而jar、apk、zip其实就是一些压缩格式，要拿到压缩包里面的dex文件就需要解压，所以，DexClassLoader在调用父类构造函数时会指定一个解压的目录。<br>
1:类加载器BaseDexClassLoader先将dex文件解析放到pathList到dexElements里面
2:加载类的时候从dexElements里面去遍历，看哪个dex里面有这个类就去加载，生成class对象

### 2.8 Android组件化开发？
> Android 组件化是一种将应用程序拆分为多个可独立开发、构建和部署的组件的架构设计方式，每个组件可以包含自己的业务逻辑、UI、数据和资源。组件可以在独立的开发环境中进行开发，也可以在集成环境中进行组合和测试。<br>
组件化架构可以带来多个好处，例如：<br>
1.模块化开发：将应用程序拆分成多个独立的组件，可以更容易地进行模块化开发，每个组件都可以独立进行开发和维护，同时也可以更好地重用已有的组件。<br>
2.提高开发效率：组件化架构可以将大型应用程序拆分成多个小模块，每个模块的编译、构建和测试可以独立进行，这样可以提高开发效率和代码质量。<br>
3.简化团队协作：组件化架构可以将应用程序拆分成多个小模块，每个模块的职责清晰，这样可以更好地划分团队的职责和任务，简化团队协作。<br>
4.提高应用程序的可维护性：组件化架构可以将应用程序拆分成多个小模块，每个模块的职责清晰，这样可以更容易地进行代码维护和升级。<br>
在实现组件化架构时，可以采用多种方式，例如使用 Android 原生的组件化框架，如组件化方案 ARouter、Dagger2、ButterKnife等等，也可以使用第三方的组件化框架，如阿里巴巴的ARouter。<br>
总之，组件化架构是一种很好的架构设计方式，可以帮助开发者更好地开发、构建和维护 Android 应用程序，同时也可以提高应用程序的可扩展性、可维护性和可重用性。

## 3 Java基础

### 3.1 VM虚拟机内存结构，以及它们的作用？
> JVM（Java Virtual Machine）虚拟机内存结构由以下几个部分组成：<br>
1.程序计数器（Program Counter Register）：程序计数器是一个较小的内存区域，它可以看作是当前线程所执行的字节码指令的地址指针。每个线程都有自己的程序计数器，因此它是线程私有的。当线程执行Java方法时，程序计数器会存储下一条指令的地址，当线程执行Native方法时，程序计数器的值为undefined。<br>
2.Java虚拟机栈（JVM Stack）：每个Java方法在执行的同时都会创建一个栈帧（Stack Frame），用于存储局部变量、操作数栈、方法出口等信息。栈帧随着方法的调用和返回而压入和弹出。Java虚拟机栈可以动态扩展或缩减，因此它的大小是可以调整的。<br>
3.本地方法栈（Native Method Stack）：与Java虚拟机栈类似，但用于执行Native方法。<br>
4.Java堆（Java Heap）：Java堆是Java虚拟机内存中最大的一块内存区域，用于存储对象实例和数组。Java堆是所有线程共享的，因此它也是线程安全的。Java堆的大小可以通过JVM启动参数进行调整。<br>
5.方法区/元空间（新）（Method Area）：方法区用于存储已被虚拟机加载的类信息、常量、静态变量等数据。与Java堆一样，方法区也是所有线程共享的。<br>


## 4 Java进阶

### 4.1 实现Java的生产者和消费者模型？
```Java
public class ProducerAndConsumer {

    public static void main(String[] args) {
        Person person = new Person();
        new Thread(new Producer("生产者1", person)).start();
        new Thread(new Producer("生产者2", person)).start();
        new Thread(new Producer("生产者3", person)).start();
        new Thread(new Producer("生产者4", person)).start();

        new Thread(new Consumer("消费者1", person)).start();
        new Thread(new Consumer("消费者2", person)).start();
        new Thread(new Consumer("消费者3", person)).start();

    }
}
 class Person {
    private int foodNum = 0;
    private Object lock = new Object();
    private final int MAX_NUM = 5;

    // 生产
    public void product() throws InterruptedException {
        synchronized(lock) {
            while (foodNum == MAX_NUM) {
                System.out.println("满了，需要消费哦");
                // 调用wait，可释放锁，等待唤醒
                lock.wait();
            }
            foodNum++;
            System.out.println("生产成功，当前食物数量："+foodNum);
            // 通知去消费
            lock.notifyAll();
        }
    }

    // 消费
    public void consume() throws InterruptedException {
        synchronized(lock) {
            while (foodNum == 0) {
                System.out.println("空了，需要生产哦");
                // 调用wait，可释放锁，等待唤醒
                lock.wait();
            }
            foodNum--;
            System.out.println("消费成功，当前食物数量："+foodNum);
            // 通知去生产
            lock.notifyAll();
        }
    }
}


// 生产线程
class Producer implements Runnable {
    private Person person;
    private String producerName;

    public Producer(String producerName, Person person){
        this.producerName = producerName;
        this.person = person;
    }

    @Override
    public void run() {
        while(true) {
            try {
                person.product();
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }
    }
}

/// 消费者
class Consumer implements Runnable {
    private Person person;
    private String consumerName;

    public Consumer(String consumerName, Person person) {
        this.consumerName = consumerName;
        this.person = person;
    }

    @Override
    public void run() {
        while(true) {
            try {
                person.consume();
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }
    }
}
```

### 4.2 HashMap的内部结构？ 内部原理? 
> HashMap是Java中最常用的集合之一，它实现了Map接口，提供了键值对存储和查找的功能。HashMap的内部结构是一个数组，每个数组元素是一个链表，也就是说它是基于数组和链表实现的。<br>
当我们向HashMap中添加一个键值对时，首先会根据键的hashcode值来确定在数组中的位置，然后将键值对插入到该位置的链表中。如果该位置已经存在键值对，那么新的键值对会被插入到链表的末尾。当我们通过键来查找值时，HashMap首先会根据键的hashcode值来确定在数组中的位置，然后遍历该位置的链表来查找对应的值。<br>
在实际使用中，为了提高HashMap的性能，我们需要合理设置HashMap的容量和负载因子。HashMap的容量指的是数组的长度，负载因子指的是数组中链表的平均长度。当链表长度超过负载因子时，HashMap会自动进行扩容，以保证性能的稳定和高效。同时，我们也需要注意HashMap中键的hashcode值的实现，以尽量避免哈希冲突，提高HashMap的性能。<br>
在Java 1.8中，HashMap的实现原理主要包括以下几个方面：<br>
数组+链表/红黑树结构：HashMap底层使用了一个数组来存储元素，每个数组元素又是一个链表或红黑树，链表主要用于解决哈希冲突，红黑树用于提高查询性能。<br>
哈希算法：HashMap使用了哈希算法来计算元素在数组中的位置，哈希算法主要是通过将键的哈希值进行计算得出元素在数组中的位置。<br>
阈值和负载因子：HashMap中还有一个负载因子和阈值的概念，当哈希表中的元素数量达到阈值时，哈希表就需要扩容，并重新计算每个元素在新数组中的位置，以保证哈希表的性能。<br>
扩容机制：HashMap的扩容机制是在扩容时将所有元素重新分配到一个更大的数组中，重新计算它们在新数组中的位置，这个过程是比较耗时的。<br>
线程安全：在Java 1.8中，HashMap不是线程安全的，如果多个线程同时访问HashMap，可能会出现数据不一致的情况，需要使用ConcurrentHashMap来解决线程安全问题。<br>
总之，在Java 1.8中，HashMap的实现原理主要是通过数组+链表/红黑树结构和哈希算法来实现元素的存储和查询，同时还使用了负载因子、阈值和扩容机制来保证哈希表的性能。

## 5 设计模式

### 5.1 实现单例模式，饿汉式和饱汉式？
```Java
// 饿汉式
class SingleTon {
    private static final SingleTon instance = SingleTon();
    private SingleTon();

    public SingleTon getInstance() {
        return instance;
    }
}
```

```Java
// 懒汉式
class SingleTon2 {
    private static volatile SingleTon2 instance;

    private SingleTon2();
    
    public SingleTon2 getInstance() {
        if (instance == null) {
            synchronized(SingleTon2.class) {
                if (instance == null) {
                    return new SingleTon2();
                }
            }
        }
        return instance;
    }

}
```

## 6 三方库相关

### 6.1 Android图片加载框架对比
> Android 主流的图片加载框架有 Picasso、Glide、Fresco 等，它们都有各自的优缺点，下面是它们的比较：<br>
1.Picasso
优点：
体积小100k左右。
简单易用，上手容易。
内存占用低，处理速度快。
支持图片裁剪和压缩。<br>
缺点：
功能相对较少，只支持常见的图片加载和转换操作。
对于网络请求缓存策略的处理相对较弱。<br>
2.Glide
优点：
支持 GIF 图片播放。
支持缩略图的生成。
支持自定义的网络请求缓存策略。
能够有效地防止图片变形和拉伸。
缺点：
对于图片裁剪和旋转的处理不够灵活，需要自定义Transformation接口。
内存占用相对较高。<br>
Fresco
优点：
支持图片的多种格式，如 WebP 和 GIF 等。
支持图片的缩放、裁剪和旋转等操作。
支持自定义的网络请求缓存策略。
对于图片的加载和处理速度较快。
缺点：
包较大（2~3M） * 用法复杂 * 底层涉及c++领域。
内存占用相对较高，处理速度也会受到影响。<br>
综上所述，三个主流的图片加载框架各有优劣，我们需要根据具体的需求和项目特点来选择适合的框架。如果需要支持多种图片格式和处理操作，Fresco 可能更加适合；如果需要简单易用的框架，Picasso 是不错的选择；如果需要较为灵活的缓存策略和防止图片变形的功能，Glide 可以考虑使用。

### 6.2 深入分析下Glide框架实现原理？
> Glide 是一款用于 Android 平台的图片加载框架，它可以快速、高效地加载图片，并且提供了丰富的图片处理和缓存策略。下面是 Glide 的实现原理：<br>
请求管理
Glide 的请求管理模块负责管理图片加载请求，它包含了一个 RequestManager 类，用于管理所有的请求和生命周期。每个 RequestManager 实例都包含了一个唯一的 Fragment 或者 Activity 对象，并且可以根据这个对象的生命周期来自动取消请求，避免了内存泄漏的问题。<br>
图片加载
Glide 的图片加载模块负责从网络、本地文件、资源文件等不同的数据源中加载图片，并提供了图片转换和缩放等处理操作。Glide 的图片加载过程可以简化为以下几个步骤：
(1) 创建一个请求对象：通过 RequestBuilder 类创建一个请求对象，设置图片加载的地址、缩放比例、占位图、错误图等属性。
(2) 选择加载方式：根据图片的来源，选择不同的加载方式，如网络加载、本地文件加载或者资源文件加载等。
(3) 图片处理：对加载的图片进行处理，如压缩、缩放、旋转、裁剪等操作。
(4) 缓存处理：对加载的图片进行缓存，Glide 的缓存策略支持磁盘缓存和内存缓存，并且可以自定义缓存策略。
(5) 图片展示：将处理后的图片展示在 ImageView 控件上。<br>
缓存管理
Glide 的缓存管理模块负责管理图片的缓存，它提供了内存缓存和磁盘缓存两种缓存方式。内存缓存采用 LRU 算法来管理缓存，可以快速地从内存中读取缓存图片。磁盘缓存则采用 DiskLruCache 来实现，它可以将图片缓存到磁盘中，避免了频繁的网络请求，提高了图片加载的速度和效率。<br>
生命周期管理
Glide 的生命周期管理模块负责管理图片加载过程中的生命周期，避免了由于内存泄漏等问题导致的应用崩溃或者图片加载失败等问题。Glide 使用 RequestManagerFragment 和 RequestManagerRetriever 两个类来管理生命周期，当 Activity 或 Fragment 被销毁时，它会自动取消图片加载请求，避免了内存泄漏的问题。<br>
总的来说，Glide 的实现原理相对较为复杂，但是通过对请求管理、图片加载、缓存管理和生命周期管理等方面的优化，可以快速、高效地加载图片，同时避免了内存泄漏等常见问题。

### 6.3 Glide的四级缓存原理？
> Glide 4.x 采用的是四级缓存的架构，相对于三级缓存多了一个资源类型的概念，这也是为了更加精细地管理图片资源。同时，Glide 4.x 中还引入了一个新的概念叫做资源池 (Resource Pool)，它用于管理正在使用的资源和可以被回收的资源，避免了过多的对象创建和销毁，提高了性能和效率。<br>
1.活动资源 (Active Resources)：当前正在使用的图片，指的是已经被解码成 Bitmap 对象并且正在被某个 ImageView 等控件使用的图片。<br>
2.内存缓存 (Memory cache)：指的是已经被解码成 Bitmap 对象并且保存在内存缓存中的图片。Glide 内存缓存使用 LruCache 来管理。在 Glide 中，内存缓存和活动资源缓存的区别在于内存缓存用于缓存未被使用的资源，而活动资源缓存用于缓存已经被使用的资源，以便于及时回收资源和避免重复加载。在<br>
3.资源类型 (Resource)：指的是磁盘缓存中的图片，经过了转换后可以被直接使用的图片。转换可以包括解码、裁剪、变换等操作。<br>
4.数据来源 (Data)：指的是磁盘缓存中的原始图片，即从网络或本地文件等数据源中下载的原始图片。数据来源可以被用来重新加载或者刷新缓存中的图片。<br>
主要流程如下：<br>
1.活动资源（Active Resources）缓存：Glide 会将最近使用过的资源缓存在活动资源缓存中，以便在下次请求时能够快速地访问到它们。如果所需的资源已经在活动资源缓存中，Glide 将直接使用它，不必再进行加载和解码。<br>
2.内存缓存（Memory Cache）：如果所需的资源不在活动资源缓存中，Glide 会尝试从内存缓存中查找。内存缓存是一个固定大小的 LRU 缓存，用于存储解码后的位图和其他昂贵的资源。如果资源在内存缓存中找到，Glide 将立即使用它。<br>
3.磁盘缓存（Disk Cache）：如果所需的资源不在内存缓存中，Glide 将尝试从磁盘缓存中查找。磁盘缓存是一个可配置的磁盘存储位置，用于存储经过解码的位图和其他资源。如果资源在磁盘缓存中找到，Glide 将加载它，并将其存储到内存缓存中，以便下次访问更快。<br>
4.数据源（Data Source）：如果资源不在活动资源、内存缓存或磁盘缓存中，Glide 将从数据源中加载它。数据源可能是网络、文件或任何其他类型的数据源。Glide 将下载资源、解码位图并将其存储到内存和磁盘缓存中，以便下次访问更快。

### 6.4 EventBus原理？
> EventBus是一个Android发布/订阅事件总线，简化了组件间的通信，让代码更加简介，但是如果滥用EventBus，也会让代码变得更加辅助。面试EventBus的时候一般会谈到如下几点：<br>
（1）EventBus是通过注解+反射来进行方法的获取的
注解的使用：@Retention(RetentionPolicy.RUNTIME)表示此注解在运行期可知，否则使用CLASS或者SOURCE在运行期间会被丢弃。 
 通过反射来获取类和方法：因为映射关系实际上是类映射到所有此类的对象的方法上的，所以应该通过反射来获取类以及被注解过的方法，并且将方法和对象保存为一个调用实体。<br>
（2）使用ConcurrentHashMap来保存映射关系
调用实体的构建：调用实体中对于Object，也就是实际执行方法的对象不应该使用强引用而是应该使用弱引用，因为Map的static的，生命周期有可能长于被调用的对象，如果使用强引用就会出现内存泄漏的问题。<br>
（3）方法的执行
使用Dispatcher进行方法的分派，异步则使用线程池来处理，同步就直接执行，而UI线程则使用MainLooper创建一个Handler，投递到主线程中去执行。

### 6.5 Retrofit原理？
> 最核心的就是动态代理技术。<br>
动态代理的实现机制实际上就是使用Proxy.newProxyInstance函数为动态代理对象A生成一个代理对象A*的类的字节码从而生成具体A*对象过程，这个A*类具有几个特点，一是它需要实现传入的接口，第二就是所有接口的实现中都会调用A的invoke方法，并且传入相应的调用实际方法（即接口中的方法）。<br>
Retrofit中使用了动态代理是不错，但是并不是为了真正的代理才使用的，它只是为了动态代理一个非常重要的功能，就是“拦截”功能。我们知道动态代理中自动生成的A*对象的所有方法执行都会调用实际代理类A中的invoke方法，再由我们在invoke中实现真正代理的逻辑，实际上也就是A*的所有方法都被A对象给拦截了。 <br>
Retrofit作用
Retrofit实际上是为了更方便的使用Okhttp，因为Okhttp的使用就是构建一个Call，而构建Call的大部分过程都是相似的，而Retrofit正是利用了代理机制带我们动态的创建Call，而Call的创建信息就来自于你的注解。

### 6.6 OkHttp3原理？
> Okhttp底层是采用Socket建立流连接，而连接如果不手动close掉，就会造成内存泄漏，那我们使用Okhttp时也没有做close操作，其实是Okhttp自己来进行连接池的维护的。<br>
Okhttp使用分发器Dispatcher来维护一个正在运行任务队列和一个等待队列。如果当前并发任务数量小于64，就放入执行队列中并且放入线程池中执行。而如果当前并发数量大于64就放入等待队列中。<br>
OkHttpClient通过创建一个newCall方法生成一个Request对象，然后它会陆续通过这几个拦截器：<br>
(1)、在配置OkHttpClient时设置的interceptors。
(2)、负责失败重试以及重定向的RetryAndFollowUpInterceptor。
(3)、负责把用户构造的请求转换为发送到服务器的请求、把服务器返回的响应转换为用户友好的响应的BridgeInterceptor。
(4)、负责读取缓存直接返回、更新缓存的CacheInterceptor。
(5)、负责和服务器建立连接的ConnectInterceptor。
(6)、配置OkHttpClient时设置的networkInterceptors。
(7)、负责向服务器发送请求数据、从服务器读取响应数据的 CallServerInterceptor。

### 6.7 RxJava实现原理？
> RxJava 是一个 基于事件流、实现异步操作的库。
RxJava原理可总结为：
被观察者 （Observable） 通过 订阅（Subscribe） 按顺序发送事件 给观察者 （Observer）
观察者（Observer） 按顺序接收事件 & 作出对应的响应动作。<br>
RxJava2 是一个响应式编程库，它通过使用可观察序列（Observable）和操作符（Operators）来简化异步编程。RxJava2 基于 ReactiveX 规范，可以让开发者使用一致的 API 来处理异步任务和事件流。<br>
RxJava2 的实现原理涉及到几个核心概念，包括 Observable、Observer、Subscriber、Operator 和 Scheduler。<br>
1.Observable
Observable 是 RxJava2 中的核心概念之一，它代表着一系列的事件流。Observable 可以被观察者订阅（subscribe），以便在产生事件时通知观察者。<br>
2.Observer / Subscriber
Observer 或 Subscriber 是观察者的概念，它们会订阅 Observable，并在 Observable 发出事件时得到通知。
Observer 和 Subscriber 的区别在于，Observer 通常用于 RxJava2 的非背压（non-backpressure）场景下，而 Subscriber 则是用于背压场景下的观察者。<br>
3.Operator
Operator 是 RxJava2 中的操作符，它们用于对 Observable 发出的事件流进行转换和过滤。RxJava2 中内置了许多常用的操作符，如 map、filter、take、zip 等。
Operator 可以组合使用，形成复杂的操作链，来完成更复杂的业务需求。<br>
4.Scheduler
Scheduler 是 RxJava2 中的调度器，它用于指定 Observable 和 Observer 所在的线程。RxJava2 内置了多个 Scheduler，如 io、computation、newThread、single 等。<br>
RxJava2 的基本实现流程如下：<br>
1.创建 Observable 对象，定义事件流的产生方式。
2.在 Observable 对象上应用操作符，对事件流进行转换和过滤。
3.调用 subscribe 方法，让 Observer 或 Subscriber 订阅 Observable，开始接收事件。
4.当事件产生时，Observable 将事件通知给 Observer 或 Subscriber。
5.在 Subscriber 中实现相应的处理逻辑。<br>
具体实现中，RxJava2 通过调用 subscribe 方法，将 Observer 或 Subscriber 注册到 Observable 中。Observable 在产生事件时，会通过调用 Observer 或 Subscriber 的相应方法，将事件通知给它们。在通知事件时，RxJava2 可以通过 Scheduler 来指定事件在哪个线程中处理，从而实现异步处理的功能。<br>
RxJava2 实现了多种常用的异步编程模式，如单一返回值、多返回值、数据流、事件流等，是 Android 开发中常用的异步编程库之一。

## 7 算法相关

### 7.1 负数移动左侧？
有一个整形数组，包含正数和负数，然后要求把数组内的所有负数移至正数的左边，且保证相对位置不变，要求时间复杂度为O(n), 空间复杂度为O(1)。例如，{10, -2, 5, 8, -4, 2, -3, 7, 12, -88, -23, 35}变化后是{-2, -4，-3, -88, -23,5, 8 ,10, 2, 7, 12, 35}。
```Java
 public void setParted(int[] a){  
    int temp=0;  
    int border=-1;  

    for(int i=0;i<a.length;i++){  
        if(a[i]<0){  
            temp=a[i];  
            a[i]=a[border+1];  
            a[border+1]=temp;  
            border++;  
        }  
    }  
    for(int j=0;j<a.length;j++){  
        System.out.println(a[j]);  
    }  
}
```
有点像快速排序二分法，选择的基准元素为0而已。遍历整个列表，每次将小于0的移动到依次递增的左侧。


