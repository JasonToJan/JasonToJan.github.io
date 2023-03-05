---
title: 面试 Java 通关秘籍
date: 2023-01-06 21:26:12
top: false
cover: false
toc: true
mathjax: true
tags:
- 面试题 Java
categories:
- 面试 
---

## 1 黑铁
> 对象，集合，泛型，反射的理解。

### 1.1 什么是多态？
> 面向对象的三大特性：封装、继承、多态。从一定角度来看，封装和继承几乎都是为多态而准备的。
多态的定义：指允许不同类的对象对同一消息做出响应。即同一消息可以根据发送对象的不同而采用多种不同的行为方式。（发送消息就是函数调用）

### 1.2 List和Set和Map区别？
> List：有序、可重复；索引查询速度快；插入、删除伴随数据移动，速度慢；
Set：无序，不可重复；(List,Set都是继承自Collection接口)
Map：键值对，键唯一，值多个；

### 1.3 线程安全的集合和非线程安全的集合？
> LinkedList、ArrayList、HashSet是非线程安全的，Vector是线程安全的;
HashMap是非线程安全的，HashTable是线程安全的;
StringBuilder是非线程安全的，StringBuffer是线程安的。

### 1.4 知道ArrayList的扩容机制吗？
> 默认长度要说下：10。
什么时候扩容：数组满的时候。
加多少：java1.6之后是1.5倍，之前是2倍。

### 1.5 ArrayList与LinkedList的区别和适用场景？
> 1.ArrayList
1.1 优点是实现了基于动态数组的数据结构,因地址连续，一旦数据存储好了，查询操作效率会比较高（在内存里是连着放的）。
1.2 因为地址连续，ArrayList要移动数据,所以插入和删除操作效率比较低。
2.LinkedList
2.1 LinkedList基于链表的数据结构，地址是任意的，其在开辟内存空间的时候不需要等一个连续的地址，对新增和删除操作add和remove，LinedList比较占优势。LikedList 适用于要头尾操作或插入指定位置的场景。
2.2 因为LinkedList要移动指针,所以查询操作性能比较低。

### 1.6 HashSet与TreeSet的区别和使用场景？
> 1.TreeSet 是二叉树（红黑树的树据结构）实现的，TreeSet中的数据是自动排好序的，不允许放入null值。
2.HashSet 是哈希表实现的，HashSet中的数据是无序的可以放入null，但只能放入一个null，两者中的值都不重复，就如数据库中唯一约束。
场景：
HashSet是基于Hash算法实现的，其性能通常都优于TreeSet。为快速查找而设计的Set，我们通常都应该使用HashSet，在我们需要排序的功能时，我们才使用TreeSet。

### 1.7 HashMap与TreeMap、HashTable的区别及适用场景？
> HashMap :非线程安全,基于哈希表(散列表)实现。适用于Map中插入、删除和定位元素。
TreeMap：非线程安全基于红黑树实现。适用于按自然顺序或自定义顺序遍历键(key)。
HashTable: 线程安全。Hashtable 的方法是Synchronized 的，底层实现和HashMap一致，都是数组+链表。

### 1.8 什么是Java反射？
> Java反射机制是在运行状态中，对于任意一个类，都能够知道这个类中的所有属性和方法；对于任意一个对象，都能够调用它的任意一个方法和属性；这种动态获取的信息以及动态调用对象的方法的功能称为java语言的反射机制。

### 1.9 类加载阶段？
> 1.加载；通过类的全限定名拿到class文件，在内存中生成一个代表此类的Class对象。
2.验证：验证类数据是否符合虚拟机的要求，确保不会危害虚拟机安全。
3.准备：为类的静态变量在方法区分配内存，赋初始值0或null。
4.解析：将类中的符号引用转换为直接引用。
5.初始化：为静态变量赋程序设定的初始值。
6.使用
7.卸载

### 1.10 抽象类和接口的区别？
> 1.抽象类和接口都不能直接实例化，如果要实例化，抽象类变量必须指向实现所有抽象方法的子类对象，接口变量必须指向实现所有接口方法的类对象。
2.接口可以被类多实现（被其他接口多继承），抽象类只能被单继承。
3.抽象类要被子类继承，接口要被类实现。

### 1.11 双亲委派机制？
> 可以类比，我碰到一个难题。先问问上司有没有解决方案，没有的话，再问问再上级，都没有，那就只能自己解决了。
1.当一个类加载器接收到类加载请求时，首先会检查它的父类加载器是否已经加载过这个类。
2.如果父类加载器已经加载了该类，那么直接返回父类加载器所加载的类。
3.如果父类加载器没有加载该类，则把类加载请求向上委托给父类加载器。
4.如果父类加载器还没有加载该类，则重复步骤3，直到达到了顶层的启动类加载器。
5.如果父类加载器都无法加载该类，那么该类加载器尝试自己去加载该类。
6.如果该类加载器自己也无法加载该类，那么会抛出ClassNotFoundException异常。<br>
使用双亲委派机制，可以保证Java虚拟机中所有的类都会被统一的父类加载器加载，并且不会出现同一个类被多个类加载器加载的情况。同时，也能防止用户自定义的类覆盖Java API中的核心类，从而提高了Java程序的稳定性和安全性。


## 2 青铜
> 多线程相关处理方案,异常处理，拷贝，引用。

### 2.1 ConcurrentHashMap保证线程安全的原因？
> 而采用了 CAS + synchronized 来保证并发安全性,
1.8 在 1.7 的数据结构上做了大的改动，采用红黑树之后可以保证查询效率（O(logn)），甚至取消了 ReentrantLock 改为了 synchronized。

### 2.2 线程池有哪几种，参数是怎样的？
> 先简单说下是哪个类：线程池是通过 ThreadPoolExecutor 类来实现的
1.参数1：核心线程数：当任务数少于核心线程数时，线程池会保持核心线程数的线程来处理任务。如果任务数超过核心线程数，线程池会根据情况创建新的线程来处理任务。
2.参数2：最大线程数：当任务数超过核心线程数时，线程池会创建新的线程来处理任务，直到线程数达到最大线程数。如果任务数继续增加，新的任务会被放到任务队列中等待处理。
3.参数3：空闲时间：如果线程池中的线程处于空闲状态超过该时间，线程池会将其销毁，直到线程数不超过核心线程数为止。
4.参数4：任务队列：用于存放等待执行的任务。Java 提供了多种类型的任务队列，包括 SynchronousQueue、LinkedBlockingQueue、ArrayBlockingQueue 等。
5.参数5：线程工厂：用于创建线程。
6.参数6：拒绝策略：当线程池中的线程都在执行任务时，如果任务队列已满，新的任务将无法提交。这时可以使用拒绝策略来处理这些任务。

### 2.3 Java的几种类型引用？
> 有4种。
1.强引用，通常我们用new创建的对象。
2.软引用：只有在内存不足的时候才回收，不期望gc回收的比较重要的对象，缓存。
3.弱引用：弱引用对象指向的对象在下一次垃圾回收时被回收，主要用于实现对象注册表、监听器和缓存等功能。
4.虚引用：虚引用对象指向的对象随时都可能被垃圾回收器回收，主要用于检测对象被垃圾回收器回收的情况。

### 2.4 Java创建线程池方法？
> 3种。
1.直接继承Thread，HandleThread就是继承Thread的。
2.实现Runnable接口，作为Thread的构造参数。
3.实现Callable接口，使用ExecutorService，OkHttp就是用的这个。

### 2.5 ThreadLocal是什么？
> ThreadLocal是Java中的一个线程本地变量，它提供了一种在多线程环境下，为每个线程都分配独立的变量副本的机制。
简单来说，它可以让多个线程访问同一个变量，但是每个线程都拥有自己独立的副本，互不影响。

### 2.6 Java内存模型？
> Java 内存模型是一种规范，定义了很多东西： 
1.所有的变量都存储在主内存(Main Memory)中。 
2.每个线程都有一个私有的本地内存(Local Memory)，本地内存中存储了该线程以读/写共享变量的拷贝副本。 
3.线程对变量的所有操作都必须在本地内存中进行，而不能直接读写主内存。

### 2.7 Java内存分布？
> 1.程序计数器（Program Counter Register）：记录当前线程执行的字节码指令的地址。
2.Java虚拟机栈（Java Virtual Machine Stacks）：保存每个线程执行方法时的局部变量表、操作数栈、动态链接、方法出口等信息。
3.本地方法栈（Native Method Stack）：与Java虚拟机栈类似，为Native方法服务的。
4.Java堆（Java Heap）：Java虚拟机中最大的一块内存，用于存放对象实例。
5.（Java8后用元空间代替）方法区（Method Area）：存储类的元数据信息，包括类的名称、方法信息、字段信息、常量池等。

### 2.8 Java垃圾回收机制？
> 1.新生代中通常采用复制算法（Copying），将堆空间分为两部分，一部分为存活对象，一部分为空闲空间。在垃圾收集时，将存活对象复制到另一部分空间，然后清空原来的空间。
2.老年代中通常采用标记-清除算法（Mark-Sweep）或标记-整理算法（Mark-Compact），对不再被引用的对象进行回收。


### 2.9 synchronized底层原理？
> synchronized 关键字是用来实现同步的，主要是通过对对象的监视器（monitor）进行加锁和解锁来实现的。在 Java 中，每个对象都有一个监视器，它可以被用来实现对象的同步。<br>
Java 中 synchronized 的底层实现是通过 Java 对象头的 Mark Word 来实现的。每个 Java 对象头都有一个 Mark Word，其中包含了对象的状态信息以及锁信息。当一个线程尝试获取一个被 synchronized 修饰的代码块的锁时，它会通过修改对象头中的锁信息来获得锁。当线程释放锁时，它会再次修改对象头中的锁信息来释放锁。<br>
，synchronized 关键字仅能保证代码块的原子性和可见性，但不能保证执行顺序。

### 2.10 volatile的作用和原理？
> 一定要明确两个作用。
保证变量的可见性和禁止指令重排序。但不能保证原子性。
然后具体分析下为什么能保证这两点：
能保证可见性的原因是：，当一个变量被声明为 volatile 后，它的值在每次被读取时都会被强制从主内存中读取，而不是从线程的本地缓存中读取。同时，在写入 volatile 变量时，会立即将新值刷新回主内存，而不是等到该线程退出同步块时再刷新。<br>
能禁止指令重排序的原因是：volatile 会使用一种特殊的内存屏障来实现内存同步。读的时候，会确保写操作已经完成。写的时候，确保之前的存储操作已完成。

### 2.11 什么是CAS操作？
> CAS 是一种无锁的原子操作，它可以在并发场景下安全地更新共享变量的值。CAS 操作包括三个操作数：内存地址 V，旧的预期值 A 和新值 B。CAS 操作的语义是，只有当 V 的值等于 A 时，才将 V 的值设置为 B，否则不进行任何操作。

### 2.12 AtomicInteger底层原理？
> AtomicInteger 是 Java 提供的一个原子类，它提供了一些原子操作，例如对整数变量的原子加减、原子赋值、原子比较等操作。AtomicInteger 通过使用 CAS（Compare And Swap）操作实现了对整数变量的原子操作。

### 2.13 synchronized修饰static方法，普通方法，类和方法块的区别？
> 1.修饰静态方法（static synchronized 方法）：在多线程环境中，只有一个线程能够访问该静态方法
2.修饰实例方法（synchronized 方法）：但同一时刻只能有一个线程访问该实例方法。（同一个实例不可同时访问）
3.修饰代码块（synchronized 块）：对象级别的，同一个实例不可同时访问。
4.修饰类：同静态方法，不可多个线程同时访问，尽管是不同实例。

### 2.14 Java多线程如何通信？
> 1.共享内存：多个线程可以访问同一个共享内存区域，通过读写共享内存中的变量来进行通信。但需要注意的是，共享内存可能会出现竞态条件问题，需要使用同步机制来保证线程安全。(我们声明的全局变量即为共享)
2.管道通信：管道是一种单向通信机制，一个线程可以向管道中写入数据，另一个线程可以从管道中读取数据。Java中通过PipedOutputStream和PipedInputStream类来实现管道通信。
3.消息传递：可以使用Java中的wait、notify和notifyAll方法来实现。一个线程可以通过调用wait方法来等待另一个线程发送消息，当另一个线程发送消息时，可以调用notify或notifyAll方法来唤醒等待的线程。
4.信号量：可以通过Java中的Semaphore类来实现。Semaphore中有一个计数器，每当一个线程获取一个许可时，计数器就会减少，当计数器为0时，其他线程就需要等待。通过调整Semaphore的许可数量，可以控制多个线程之间的同步行为。
5.屏障：CyclicBarrier类来实现屏障。每个线程在到达屏障前都需要等待其他线程，当所有线程都到达屏障后，屏障才会打开，所有线程可以继续执行。
6.同步工具类：CountDownLatch，假设某个线程在执行任务之前，需要等待其它线程完成一些前置任务，必须等所有的前置任务都完成，才能开始执行本线程的任务。

## 3 白银
> 常用API原理。

### 3.1 HashMap原理？
> 底层基于数组+链表，java1.8后了，如果链表过程，会将链表转换为红黑树。
提一下关键方法，有put和get。原理主要分析这两个方法：<br>
put方法：
1.判断当前桶是否为空，空的就需要初始化（resize 中会判断是否进行初始化）。
2.根据当前 key 的 hashcode 定位到具体的桶中并判断是否为空，为空表明没有 Hash 冲突就直接在当前位置创建一个新桶即可。
3.如果当前桶有值（ Hash 冲突），那么就要比较当前桶中的 key、key 的 hashcode 与写入的 key 是否相等，相等就赋值给 e,在第 8 步的时候会统一进行赋值及返回。
4.如果当前桶为红黑树，那就要按照红黑树的方式写入数据。
5.如果是个链表，就需要将当前的 key、value 封装成一个新节点写入到当前桶的后面（形成链表）。
6.接着判断当前链表的大小是否大于预设的阈值，大于时就要转换为红黑树。
7.如果在遍历过程中找到 key 相同时直接退出遍历。
8.如果 e != null 就相当于存在相同的 key,那就需要将值覆盖。
9.最后判断是否需要进行扩容。<br>
get方法：
1.首先将 key hash 之后取得所定位的桶。
2.如果桶为空则直接返回 null 。
3.否则判断桶的第一个位置(有可能是链表、红黑树)的 key 是否为查询的 key，是就直接返回 value。
4.如果第一个不匹配，则判断它的下一个是红黑树还是链表。
5.红黑树就按照树的查找方式返回值。
6.不然就按照链表的方式遍历匹配返回值。

### 3.2 ConcurrentHashMap原理？
> 先简单介绍底层结构：ConcurrentHashMap采用 分段锁的机制，实现并发的更新操作，底层采用数组+链表+红黑树的存储结构。
再说下核心内部类：Segment和HashEntry。
1.Segment:Segment继承ReentrantLock用来充当锁的角色，每个 Segment 对象守护每个散列映射表的若干个桶。
2.HashEntry: 用来封装映射表的键 / 值对；
3.每个桶是由若干个 HashEntry 对象链接起来的链表。<br>
最后再说下最新jdk1.8的实现：利用CAS+Synchronized来保证并发更新的安全，不用之前分段锁机制了。

### 3.3 深入理解ReectrantLock?
> 1.公平性： ReentrantLock 支持公平锁和非公平锁，默认是非公平锁，而 synchronized 只支持非公平锁。
2.条件变量： ReentrantLock 提供了 Condition 接口，可以用于实现精确唤醒和等待，而 synchronized 不支持这种方式。
3.可中断性： ReentrantLock 提供了可中断的锁获取方式，通过 lockInterruptibly() 方法，可以在等待锁的过程中响应中断请求，而 synchronized 不支持这种方式。
4.实现方式： synchronized 是 JVM 内置的关键字，它通过监视对象来实现同步。而 ReentrantLock 是通过 Java 代码实现的类，它使用了一些底层的硬件支持。
总结：综上所述，ReentrantLock 和 synchronized 都可以用于线程同步，但 ReentrantLock 更加灵活，提供了更多的控制和功能，可以满足更多的需求，但使用也更加复杂。

### 3.4 ArrayBlockingQueue队列原理？
> 是什么一定要说：是一个大小固定的阻塞队列。底层是数组维护，队列元素先进先出。新元素总是先插入到队列尾部。
一旦创建，大小不能改变，队列满的时候执行put操作，会阻塞线程，队列为空，执行take，也会阻塞。
支持公平策略，底层基于ReentrantLock公平锁实现。

### 3.5 LinkedBlockingQueue队列原理？
> 是什么： ，它是由一个基于链表的阻塞队列。
底层也是用ReentrantLock实现公平锁。
主要从入队列操作分析原理：其主要是通过获取到putLock锁来完成，当队列的数量达到最大值，此时会导致线程处于阻塞状态或者返回false(根据具体的方法来看)；如果队列还有剩余的空间，那么此时会新创建出一个Node对象，将其设置到队列的尾部，作为LinkedBlockingQueue的last元素。

### 3.6 生产者消费者模式？
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

## 4 黄金
> 了解虚拟机相关知识。

### 4.1 对象的创建，内存布局，和访问定位？
> 1.对象的创建：
1.1 虚拟机遇到一个new指令时，首先将去检查这个指令的参数是否能在常量池中定位到一个类的符号引用；
1.2 检查这个符号引用代表的类是否已经被加载，解析和初始化过。如果没有，那必须先执行响应的类加载过程；
1.3 在类加载检查功通过后，为新生对象分配内存。对象所需的内存大小在类加载完成后便可完全确定。<br>
2.对象的内存布局：
2.1 对象头：对象自身的运行时数据，如线程持有的锁，synchronized原理基于这个。+第二部分是类型指针。
2.2 实例数据：真正存储的有效数据。
2.3 对象填充： 刚好8个字节整数倍。<br>
3.对象的访问定位：
3.1 句柄访问
3.2 直接指针，存的就是对象的地址。

### 4.2 Java类加载机制详解？
> 首先最好说下生命周期。
加载->验证->准备->解析->初始化->使用->卸载。<br>
最好说下什么时候触发类加载：
1.遇到new，getstatic,putstatic,invokestaic这4个字节码指令时，主要就是new一个对象，读取或设置一个类的静态字段，调用静态方法，也会触发。
2.反射也会触发。
3.初始化一个子类，会先初始化父类。
4.虚拟机启动，用户指定要执行的main函数对应的类。<br>
然后类加载器的分类可以说下：
1.启动类加载器：负责加载Java的核心类。
2.扩展类加载器，负责加载Java扩展的核心类之外的类。
3.应用程序类加载器：没有自定义加载器都是使用这个加载器。
4.自定义加载器
<br>
最好再补仓下双亲委派机制：
双亲委派模型的工作流程是：如果一个类加载器收到了类加载的请求，它首先不会自己去尝试加载这个类，而是把请求委托给父加载器去完成，依次向上，
因此，所有的类加载请求最终都应该被传递到顶层的启动类加载器中，只有当父加载器在它的搜索范围中没有找到所需的类时，即无法完成该加载，子加载器才会尝试自己去加载该类。

### 4.3 Java垃圾回收算法详解？
> 1.标记清除算法
①首先标记出所有需要回收的对象
②在标记完成后统一回收所有被标记的对象。
不足：1.效率低，生成内存碎片。<br>
2.复制算法
将可用内存分层大小相等的两块，每次只使用其中一块。
不足：将内存缩小了一半。<br>
3.标记整理算法
先走标记清除算法逻辑，再将存活对象向一端移动，再统一清除。<br>
4.分代收集算法
在新生代中，每次垃圾收集时都发现有大批对象死去，只有少量存活，那就选用复制算法。
在老年代中，因为对象存活率高、没有额外空间对它进行分配担保，就必须采用“标记-清除”或“标记-整理”算法来进行回收。

### 4.4 你知道哪些垃圾收集器？
> 1.Serial收集器：单线程收集器。简单而高效。
2.ParNew收集器：Serial的多线程版本。
3.Parallel Scavenge收集器： 新生代收集器，并行多线程收集器。
4.Serial Old收集器：使用了标记整理算法。

### 4.5 JVM怎么判断对象已经死亡？
> 采用了可达性分析算法。
通过一些列的称为“GC Roots”的对象作为起始点，从这些节点开始向下搜索，搜索所走过的路径称为引用链，当一个对象到GC Roots没有任何引用链相连时（就是从GC Roots 到这个对象是不可达），则证明此对象是不可用的。

## 5 铂金

## 6 钻石

## 7 大师

## 8 宗师

## 9 王者
