---
title: 面试 iOS 通关秘籍
date: 2023-01-10 21:26:12
top: false
cover: false
toc: true
mathjax: true
tags:
- 面试题 iOS Swift
categories:
- 面试 
---

## 1 黑铁

### 1.1 单选题，常识问题？
1.iOS后台运行时在哪个版本才开始支持的:[B]
A、iOS3.0
B、iOS4.0 
C、iOS5.0
D、iOS6.0

2.下列UIView的方法中,哪一个在iOS5.0前后的系统调用机制不同:[B]
A、addSubView
B、layoutSubView 
C、drawRect
D、removeFromSuperView

3.关于iOS程序后台运行,下面说法正确的有:[ABC]
A、程序可以在后台播放音乐 
B、程序可以在后台收集用户位置信息
C、程序可以在后台运行VOIP服务
D、程序可以在后台发送HTTP通讯

4.关于iOS,以下说法正确的是?:[ABC]
A、iOS是Apple公司推出的一款操作系统，是用于Apple移动设备的移动操作系统。
B、由于最初是设计给iPhone使用的，所以该系统原名为iPhone OS 。即"iPhone 运行 OS X"。
C、iOS系统使用了和macOS一样的Unix内核。
D、iOS系统可以1应用在iWatch和iPod上。

5.用户可以通过Siri技术,使用语言提问的方式进行人机交互。Siri的引入是从哪个iOS版本开始的?:[B]
A、iOS 4.0
B、iOS 5.0
C、iOS 6.0
D、iOS 7.0

6.从哪个iOS版本开始,系统UI从拟物风格转换为扁平化风格:[C]
A、iOS 5
B、iOS 6
C、iOS 7
D、iOS 8

7.Apple Pay是在哪个版本开始和大家见面的?:[C]
A、iOS 6
B、iOS 7
C、iOS 8
D、iOS 9

8.从哪个iOS版本开始,苹果开放了对第三方输入的支持:[B]
A、iOS 7
B、iOS 8
C、iOS 9
D、iOS 10

9.Split View和画中画功能最早是在哪个iOS版本中引入的?:[C]
A、iOS 7
B、iOS 8
C、iOS 9
D、iOS 10

10.针对中国用户,苹果在哪个iOS版本中,开始对电话功能进入了十分体贴的优化。增加骚扰电话识别功能。:[C]
A、iOS 7
B、iOS 8
C、iOS 9
D、iOS 10

11.ARKit增强现实功能和CoreML机器学习功能在哪个iOS版本中引入的？:[A]
A、iOS 11
B、iOS 10
C、iOS 9
D、iOS 8

12.关于iOS开发,以下说法正确的是？:[ABCD]
A、采用iOS系统的iPhone屏幕较小，只是把需要现实给用户的内容合理地组织在一块小小的屏幕上，所有需要设计者进行精心的设计和排版。
B、iOS采用手指触摸的方式进行人机交互，所以要尽可能使按钮等交互控件的尺寸保持在44点以上，以避免误操作。
C、运行iOS系统的移动设备，通常内存在512MB~2GB之间。用户需要在应用中合理地使用多媒体素材，保证应用不会因太耗内存而被系统自动关掉。
D、作为运行在移动设备上的应用，需要尽可能降低电量的消耗。比如及时关闭地理定位服务，减少不必要的网络请求，尽量避免以轮询的方式工作。

13.关于iOS开发,以下说法正确的是？:[ABC]
A、一个App作为一个程序束bundle存在，App只可以访问其他资源束之内的文件夹或其他资源文件。
B、在iOS中运行的应用，可以访问移动设备自带的加速计、陀螺仪、地理定位设备、蓝牙、相机等。
C、iOS应用很少使用菜单进行页面之间的跳转，而是通常采用导航控制器或标签可控制器进行页面之间的导航。
D、iOS系统中的应用。没有最小化和关闭按钮。用户通过按下设备底部的Home键，退出正在运行的应用。应用退出后仍然在内存保存一段时间。

14.ARC自动引用计数和iCloud是在哪个iOS版本中新增的？:[B]
A、iOS 7
B、iOS 6
C、iOS 5
D、iOS 4

15.哪个iOS版本增加了对Bit 64的支持 和引入了TextKit框架？:[B]
A、iOS 7
B、iOS 6
C、iOS 5
D、iOS 4

16.哪个iOS版带来了 Size Class和 Autolayout自动布局功能？:[B]
A、iOS 7
B、iOS 8
C、iOS 9
D、iOS 10

17.3D Touch和Ipad分屏是在哪个iOS版本开始引入的:[C]
A、iOS 7
B、iOS 8
C、iOS 9
D、iOS 10

18.苹果在哪个iOS版本中向开发者开放了SiriKit框架?:[C]
A、iOS 8
B、iOS 9
C、iOS 10
D、iOS 11

19.作为推广ApplePay的一种策略，苹果在哪个iOS版本中，向开发者开放了NFC(Near field communication)功能?:[D]
A、Xcode 8
B、Xcode 9
C、Xcode 10
D、Xcode 11

20.Core Image 图像处理框架是从哪个iOS版本起加入进来的?:[B]
A、iOS 5
B、iOS 6
C、iOS 7
D、iOS 8

21.自哪个版本的iOS开始,Apple为用户带来了炫酷的毛玻璃效果?:[B]
A、iOS 6.0
B、iOS 7.0
C、iOS 8.0
D、iOS 9.0

22.storyboard故事版功能是在哪个iOS版本发布的:[B]
A、iOS 4
B、iOS 5
C、iOS 6
D、iOS 7

23.苹果的iOS系统采用了哪些严格的安全机制:[ABCD]
A、代码签名
B、权限隔离
C、可信启动连
D、沙盒执行环境

24.为App设置关键词，如果关键词包含竞品的名称，则关键词会被屏蔽:[A]
A、正确
B、错误

25.为App设置关键词，关键字 不需要包含app的名称?[A]
A、正确
B、错误

26.如果App审核被拒的原因是Meta信息造成的。则不需要重新提交IPA`文件吗?[A]
A、正确
B、错误

27.在iOS App中实体物品的购买可以使用支付宝?[B]
A、正确
B、错误

28.下载安装量无论是在App Store还是在Google Play，都是导致App排名 上升或者下跌的主要因素?[B]
A、正确
B、错误

29.在100字符长度的关键字列表中,越靠前的关键字权重越大?[B]
A、正确
B、错误

30.以下哪种情况会导致审核失败?[ABCD]
A、应用出现崩溃、加载失败等非常明显的Bug。
B、应用描述、截图等与应用功能严重不符。
C、错误使用抽奖、竞拍等促销方式。
D、包含虚假、误导用户的信息或功能。

31.在App的标题、子标题、描述文字等出现安卓或Android字样。有可能在审核导致App被拒吗？[B]
A、不可能
B、很有可能

32.个人开发者账号可以在App Store发布金融应用吗？[B]
A、可以
B、不可以

33.应用使用了私有API,会在审核时被拒吗？[A]
A、会
B、不会

34.应用名称、安装包等地方包含test、demo等字样，会在审核时被拒吗？[A]
A、会
B、不会

35.应用程序在审核时被拒，可以分哪两种情况？[BC]
A、Binary Rejected
B、App Rejected
C、Metadata Rejected
D、Game Rejected

36.如果应用程序审核被拒并显示 Binary Rejected,此时需要重新上传IPA文件吗？[A]
A、需要
B、不需要

37.如果应用程序审核被拒并显示 Metadata Rejected,此时需要重新上传IPA文件吗？[B]
A、需要
B、不需要

38.除了从App Store下载,我们还可从哪些渠道 安装一个App？[ABCD]
A、开发App时可以直接把开发中的应用安装进手机进行调试。
B、In-House 企业内部分发，可以直接安装企业正数签名后的APP。
C、AD-Hoc 相当于企业分发的限制版。
D、使用开发者证书打包，并将包安装在开发者证书指定的设备上。

39.苹果对连续订阅抽成15%[B]
A、正确
B、错误

40.开发者可以直接回复用户在App Store中的评论吗？[A]
A、可以
B、不可以

41.App名称、截图和预览中包含价格信息(免费、打折)将无法上架App Store?[B]
A、正确
B、错误

42.iOS11之前 导航栏的默认高度为:[B]
A、32Pt
B、48Pt
C、64Pt
D、96Pt

43.iOS11之后如果设置preferLargeTitles = YES,则导航栏的高度为:[C]
A、32Pt
B、48Pt
C、64Pt
D、96Pt

44.在iOS11上,如果App启动时图标的四周出现黑色,是因为图标的四角的圆角,并且周围为透明像素。:[B]
A、正确
B、错误

45.获取苹果推荐的App需要包含哪些要素:[ABCD]
A、质量为上：获得苹果推荐的首要的条件便是产品质量。
B、关注度：设计新颖，明确自己能传达给用户什么内容，同时具有独特的吸引力
C、商业模式：适当的商业模式和价格，最好是和同类游戏相比有着独具一格的商业模式
D、通用性：对于各种规格设备的支持，各个地区的本地化

### 1.2 String 与 NSString 的关系与区别？
> 在 Swift 中，String 和 NSString 是两个不同的类型，但它们之间可以互相转换。下面是它们之间的关系和区别：
1.String 和 NSString 都是表示字符串的类型，但它们是不同的类型。String 是 Swift 中的原生字符串类型，而 NSString 是 Foundation 框架中的 Objective-C 字符串类型。
2.String 是一个结构体类型，而 NSString 是一个类。因此，String 是值类型，而 NSString 是引用类型。
3.String 支持 Swift 语言的特性，例如可选值、泛型和模式匹配等，而 NSString 不支持这些特性。
4.在 Swift 中，可以使用 NSString 类型的值初始化 String 类型的值，并使用 as NSString 将 String 转换为 NSString。
5.String 和 NSString 有一些共同的方法和属性，例如 length、substring、isEqual 和 stringByAppendingString 等。

### 1.3 怎么获取一个 String 的长度
> 在 Swift 中，可以使用字符串的 count 属性来获取字符串的长度。count 属性返回字符串中字符的数量，不包括 Unicode 组合字符序列（例如重音符号和变音符号）。

### 1.4 如何截取 String 的某段字符串
> 1.在 Swift 中，可以使用字符串的 prefix(_:) 和 suffix(_:) 方法来截取字符串的某个子串。这两个方法分别用于从字符串的开头或结尾开始提取指定数量的字符。
2.另外，也可以使用字符串的 range(of:) 方法和下标来截取字符串的某个子串。range(of:) 方法返回包含子串的范围，然后可以使用下标来提取这个子串。

### 1.5 throws 和 rethrows 的用法与作用
> 1.throws 关键字用于在函数或方法的声明中标记该函数或方法可能会抛出错误。函数或方法使用 throw 关键字来抛出错误，并使用 try、try? 或 try! 关键字来调用抛出错误的函数或方法。
2.rethrows 关键字用于在函数或方法的声明中标记该函数或方法可能会调用抛出错误的函数或方法。这种函数或方法必须接受一个或多个抛出错误的函数或方法作为参数，并将它们标记为 throws。

### 1.6 try？ 和 try！是什么意思
> 1.try? 关键字表示将错误转换为可选值。使用 try? 关键字来调用可能抛出错误的函数或方法时，如果抛出了错误，则返回 nil，否则返回一个包含函数或方法返回值的可选值。
2.try! 关键字表示强制解包错误。使用 try! 关键字来调用可能抛出错误的函数或方法时，如果抛出了错误，则程序将崩溃，否则返回函数或方法的返回值。因此，使用 try! 关键字时必须确保调用的函数或方法不会抛出错误，否则将导致程序崩溃。

### 1.7 associatedtype 的作用
> associatedtype 是 Swift 中用于关联类型的关键字，它可以定义一个占位符类型，它的具体类型将在符合协议时或实现父类时被指定。
在协议中，associatedtype 可以用来指定一些类型的名称，但是这些类型的实际类型是不确定的，只有在符合协议时才会被具体化。

### 1.8 什么时候使用 final
> 在 Swift 中，final 是用于限制类、方法或属性继承和重写的关键字。当一个类、方法或属性被标记为 final 时，它将不能被继承或重写，从而防止其他人修改该类、方法或属性的实现。因此，final 可以用来提高代码的安全性、可靠性和性能。<br>
下面是一些使用 final 关键字的常见情况：
防止类被继承：当一个类被标记为 final 时，它将不能被其他类继承，从而保证该类的实现不会被修改或重写。例如，Foundation 框架中的 NSString 类就是一个被标记为 final 的类，因此它不能被继承。
防止方法被重写：当一个类的某个方法被标记为 final 时，它将不能被子类重写，从而保证该方法的实现不会被修改或覆盖。例如，iOS SDK 中的 UIViewController 类中的 viewDidLoad() 方法就是一个被标记为 final 的方法，它不能被子类重写。
提高性能：当一个方法被标记为 final 时，编译器可以优化该方法的调用，从而提高程序的性能。这是因为编译器可以在编译时确定该方法的实现，而不必在运行时进行动态派发。

### 1.9 public 和 open 的区别
> 在 Swift 中，public 和 open 都是访问控制关键字，用于控制模块之间的访问权限。它们的区别如下：<br>
public：表示一个实体可以被定义模块之外的模块访问，但是不能被继承和重写。这意味着，如果一个类、方法或属性被标记为 public，则其他模块可以访问该类、方法或属性，但是不能继承和重写该类、方法或属性。<br>
open：表示一个实体可以被定义模块之外的模块访问，并且可以被继承和重写。这意味着，如果一个类、方法或属性被标记为 open，则其他模块可以访问、继承和重写该类、方法或属性。<br>
因此，public 和 open 的区别在于是否允许其他模块继承和重写该实体。通常情况下，应该优先使用 public 关键字，只有在确实需要其他模块继承和重写该实体时，才使用 open 关键字。同时，使用 open 关键字也会带来一些安全风险，因为其他模块可以修改该实体的实现，从而破坏程序的正确性和稳定性。因此，应该谨慎使用 open 关键字，只在必要时才使用。

### 1.10 声明一个只有一个参数没有返回值闭包的别名
```swift
typealias MyClosure = (Int) -> Void

func performClosure(closure: MyClosure) {
    closure(10)
}

let myClosure: MyClosure = { value in
    print("Value: \(value)")
}

performClosure(closure: myClosure) // 输出：Value: 10
```

### 1.11 iOS Swift 封装，多态，和嵌套类型？
> 1.封装：通常把隐藏属性、方法与方法实现细节的过程称为封装
1.1. public：从外部模块和本模块都能访问
1.2. internal：只有本模块能访问
1.3. private：只有本文件可以访问，本模块的其他文件不能访问<br>
2.多态：
多态是指允许使用一个父类类型的变量或者常量来引用一个子类类型的对象，根据被引用子类对象特征的不同，得到不同的运行结果，即使用父类的类型来调用子类的方法。
通过继承，两个冒号来实现。<br>
3.嵌套类型:
swift允许在一个类型中嵌套定义另一个类型，可以在枚举类型、类和结构体中定义支持嵌套的类型。

### 1.12 什么是Swift?
> Swift是苹果在2014年6月WWDC发布的全新编程语言，借鉴了JS,Python,C#,Ruby等语言特性,看上去偏脚本化,Swift 仍支持 cocoa touch 框架
他的优点:
1.Swift更加安全，它是类型安全的语言。
2.Swift容易阅读，语法和文件结构简易化。
3.Swift更易于维护，文件分离后结构更清晰。
4.Swift代码更少，简洁的语法，可以省去大量冗余代码。
5.Swift速度更快，运算性能更高。

### 1.13 Swift和OC如何相互调用？
> Swift 调用 OC代码
需要创建一个 Target-BriBridging-Header.h 的桥文件,在文件导入需要调用的OC代码头文件即可，可以使用 Swift 中的 Selector 类型来表示该方法的名称<br>
OC 调用 Swift代码
直接导入 Target-Swift.h文件即可, Swift如果需要被OC调用,需要先将Swift类或方法用@objc进行修饰

### 1.14 什么可选型(Optional)?
> 1.在 Swift 中,可选型是为了表达一个变量为空的情况,当一个变量为空,他的值就是 nil
2.在类型名称后面加个问号? 来定义一个可选型
3.值类型或者引用类型都可以是可选型变量

### 1.15 Swift,什么是泛型?
1.泛型主要是为增加代码的灵活性而生的,它可以是对应的代码满足任意类型的的变量或方法;
2.泛型可以将类型参数化，提高代码复用率，减少代码量

### 1.16 访问控制关键字 open, public, internal, fileprivate, private 的区别?
Swift 中有个5个级别的访问控制权限,从高到低依次是 open, public, internal, fileprivate, private
它们遵循的基本规则: 高级别的变量不允许被定义为低级别变量的成员变量,比如一个 private 的 class 内部允许包含 public的 String值,反之低级变量可以定义在高级别变量中;<br>
open: 具备最高访问权限,其修饰的类可以和方法,可以在任意 模块中被访问和重写.
public: 权限仅次于 open，和 open 唯一的区别是: 不允许其他模块进行继承、重写
internal: 默认权限, 只允许在当前的模块中访问，可以继承和重写,不允许在其他模块中访问
fileprivate: 修饰的对象只允许在当前的文件中访问;
private: 最低级别访问权限,只允许在定义的作用域内访问

### 1.17 关键字:Strong,Weak,Unowned 区别?
> Swift 的内存管理机制同OC一致,都是ARC管理机制; Strong,和 Weak用法同OC一样
Unowned(无主引用), 不会产生强引用，实例销毁后仍然存储着实例的内存地址(类似于OC中的unsafe_unretained), 试图在实例销毁后访问无主引用，会产生运行时错误(野指针)

### 1.18 如何理解copy-on-write?
> 值类型(比如:struct),在复制时,复制对象与原对象实际上在内存中指向同一个对象,当且仅当修改复制的对象时,才会在内存中创建一个新的对象<br>
为了提升性能，Struct, String、Array、Dictionary、Set采取了Copy On Write的技术
比如仅当有“写”操作时，才会真正执行拷贝操作
对于标准库值类型的赋值操作，Swift 能确保最佳性能，所有没必要为了保证最佳性能来避免赋值

### 1.19 什么是属性观察?
> 属性观察是指在当前类型内对特性属性进行监测,并作出响应,属性观察是 swift 中的特性,具有2种, willset 和 didset

### 1.20 如何将Swift 中的协议(protocol)中的部分方法设计为可选(optional)?
> 1.在协议和方法前面添加 @objc,然后在方法前面添加 optional关键字,改方式实际上是将协议转为了OC的方式
2.使用扩展(extension),来规定可选方法,在 swift 中,协议扩展可以定义部分方法的默认实现

### 1.21 什么是函数重载? swift 支不支持函数重载?
> 函数重载是指: 函数名称相同,函数的参数个数不同, 或者参数类型不同,或参数标签不同, 返回值类型与函数重载无关
swift 支持函数重载。

### 1.22 swift 中的枚举,关联值 和 原始值的区分?
> 1.关联值--有时会将枚举的成员值跟其他类型的变量关联存储在一起，会非常有用
2.原始值--枚举成员可以使用相同类型的默认值预先关联，这个默认值叫做:原始值

### 1.23 swift 中的闭包结构是什么样子的?
```Swift
{
    (参数列表) -> 返回值类型 in 函数体代码
}
```

### 1.24 什么是尾随闭包?
> 将一个很长的闭包表达式作为函数的最后一个实参
使用尾随闭包可以增强函数的可读性
尾随闭包是一个被书写在函数调用括号外面(后面)的闭包表达式
```swift
// fn 就是一个尾随闭包参数
func exec(v1: Int, v2: Int, fn: (Int, Int) -> Int) {
    print(fn(v1, v2))
}

// 调用
exec(v1: 10, v2: 20) {
    $0 + $1
}
```

### 1.25 什么是逃逸闭包?
> 当闭包作为一个实际参数传递给一个函数或者变量的时候，我们就说这个闭包逃逸了，可以在形式参数前写 @escaping 来明确闭包是允许逃逸的。
非逃逸闭包、逃逸闭包，一般都是当做参数传递给函数
非逃逸闭包:闭包调用发生在函数结束前，闭包调用在函数作用域内
逃逸闭包:闭包有可能在函数结束后调用，闭包调用逃离了函数的作用域，需要通过@escaping声明
```Swift
// 定义一个数组用于存储闭包类型
var completionHandlers: [() -> Void] = []

//  在方法中将闭包当做实际参数,存储到外部变量中
func someFunctionWithEscapingClosure(completionHandler: @escaping () -> Void) {
    completionHandlers.append(completionHandler)
}
```

### 1.26 什么是自动闭包?
> 自动闭包是一种自动创建的  用来把作为实际参数传递给函数的表达式打包的闭包。  它不接受任何实际参数，并且当它被调用时，它会返回内部打包的表达式的值。
这个语法的好处在于通过写普通表达式代替显式闭包而使你省略包围函数形式参数的括号。
```Swift
func getFirstPositive(_ v1: Int, _ v2: @autoclosure () -> Int) -> Int? {
    return v1 > 0 ? v1 : v2()
}
getFirstPositive(10, 20)
```
为了避免与期望冲突，使用了@autoclosure的地方最好明确注释清楚:这个值会被推迟执行
@autoclosure 会自动将 20 封装成闭包 { 20 }
@autoclosure 只支持 () -> T 格式的参数
@autoclosure 并非只支持最后1个参数
有@autoclosure、无@autoclosure，构成了函数重载
如果你想要自动闭包允许逃逸，就同时使用 @autoclosure 和 @escaping 标志。

### 1.27 swift中, 存储属性和计算属性的区别?
> Swift中跟实例对象相关的属性可以分为2大类
存储属性(Stored Property)
类似于成员变量这个概念
存储在实例对象的内存中
结构体、类可以定义存储属性
枚举不可以定义存储属性<br>
计算属性(Computed Property)
本质就是方法(函数)
不占用实例对象的内存
枚举、结构体、类都可以定义计算属性
```Swift
struct Circle {
    // 存储属性
    var radius: Double
    // 计算属性
    var diameter: Double {
        set {
            radius = newValue / 2
        }
        get {
            return radius * 2
        }
    }
}
```

### 1.28 什么是延迟存储属性(Lazy Stored Property)?
> 使用lazy可以定义一个延迟存储属性，在第一次用到属性的时候才会进行初始化(类似OC中的懒加载)
lazy属性必须是var，不能是let
let必须在实例对象的初始化方法完成之前就拥有值
如果多条线程同时第一次访问lazy属性
无法保证属性只被初始化1次
```Swift
class PhotoView {
    // 延迟存储属性
    lazy var image: Image = {
        let url = "https://...x.png"        
        let data = Data(url: url)
        return Image(data: data)
    }() 
} 
```

### 1.29 单例模式？
> 可以通过类型属性+let+private 来写单例; 代码如下如下:
```Swift
 public class FileManager {
    public static let shared = {
        // ....
        // ....
        return FileManager()
}()
    private init() { }
}
```

### 1.30 简要说明Swift中的初始化器?
> 类、结构体、枚举都可以定义初始化器
类有2种初始化器: 指定初始化器(designated initializer)、便捷初始化器(convenience initializer)<br>
```Swift
 // 指定初始化器 
init(parameters) {
    statements 
}
// 便捷初始化器
convenience init(parameters) {
    statements 
} 
```

### 1.31 什么可选链?
> 可选链是一个调用和查询可选属性、方法和下标的过程，它可能为 nil 。如果可选项包含值，属性、方法或者下标的调用成功；如果可选项是 nil ，属性、方法或者下标的调用会返回 nil 。多个查询可以链接在一起，如果链中任何一个节点是 nil ，那么整个链就会得体地失败。<br>
多个?可以链接在一起
如果链中任何一个节点是nil，那么整个链就会调用失败

## 2 青铜
> 底层必会问题，三方库的使用。

### 2.1 什么情况使用weak关键字，相比assign有什么不同？
> 情况1：在 ARC 中，在有可能出现循环引用的时候，往往要通过让其中一端使用 weak 来解决，比如: delegate、block。
情况2：自身已经对它进行一次强引用，没有必要再强引用一次，此时也会使用 weak。<br>
weak 和 assign 的不同点：
weak、assign 修饰的属性指向一个对象时都不会增加对象的引用计数。然而在所指的对象被释放时，weak 属性值会被置为 nil，而 assign 属性不会。
assign 可以用非 OC 对象以及基本类型，而 weak 必须用于 OC 对象。

### 2.2 iOS 类（class）和结构体（struct）有什么区别？
> Swift 中，类是引用类型，结构体是值类型。值类型在传递和赋值时将进行复制，而引用类型则只会使用引用对象的一个"指向"。所以他们两者之间的区别就是两个类型的区别。

### 2.3 class 和 struct 比较,优缺点?
> class 有以下功能,struct 是没有的:
1.class可以继承,子类可以使用父类的特性和方法
2.类型转换可以在运行时检查和解释一个实例对象
3.class可以用 deinit来释放资源
4.一个类可以被多次引用 <br>
struct 优势:
1.结构较小,适用于复制操作,相比较一个class 实例被多次引用,struct 更安全
2.无需担心内存泄露问题

### 2.4 swift 为什么将 String,Array,Dictionary设计为值类型?
> 值类型和引用类型相比,最大优势可以高效的使用内存,值类型在栈上操作,引用类型在堆上操作,栈上操作仅仅是单个指针的移动,而堆上操作牵涉到合并,位移,重链接,Swift 这样设计减少了堆上内存分配和回收次数,使用 copy-on-write将值传递与复制开销降到最低。

### 2.5 比较Swift 和OC中的初始化方法 (init) 有什么不同?
> swift 的初始化方法,更加严格和准确, swift初始化方法需要保证所有的非optional的成员变量都完成初始化, 同时 swfit 新增了convenience和 required两个修饰初始化器的关键字
1.convenience只提供一种方便的初始化器,必须通过一个指定初始化器来完成初始化
2.required是强制子类重写父类中所修饰的初始化方法

### 2.6 比较 Swift和OC中的 protocol 有什么不同?
> 1.Swift 和OC中的 protocol相同点在于: 两者都可以被用作代理;
2.不同点: Swift中的 protocol还可以对接口进行抽象,可以实现面向协议,从而大大提高编程效率,Swift中的protocol可以用于值类型,结构体,枚举;

### 2.7 如何使用SanpKit?
> 地址：[https://github.com/SnapKit/SnapKit](https://github.com/SnapKit/SnapKit)。
是什么？
SnapKit 是一个用于 iOS 开发的流行的第三方自动布局库，可以帮助 iOS 开发者更轻松地实现自动布局。
```Swfit
redView.snp.makeConstraints { (make) in
    make.center.equalToSuperview()
    make.width.height.equalTo(100)
}
```

### 2.8 如何使用Alamofire?
> 地址：[https://github.com/Alamofire/Alamofire](https://github.com/Alamofire/Alamofire)
是什么？
Alamofire 是一个基于 Swift 语言的 HTTP 网络请求库，它提供了简单易用的接口和强大的功能，可以帮助 iOS 开发者更方便地实现网络请求。
配置链接，AF.request发送请求，response闭包处理结果。

### 2.9 如何使用Moya?
> 地址：[https://github.com/Moya/Moya](https://github.com/Moya/Moya)
是什么？
Moya 是一个基于 Swift 语言的网络抽象库，它使用 Alamofire 库来处理网络请求，并提供了简单易用的接口和强大的功能，可以帮助 iOS 开发者更方便地实现网络请求。
先定义枚举，扩展枚举继承TargetType。通过provider调用request方法发送请求。

### 2.10 如何使用Realmswift?
> 地址：[https://github.com/realm/realm-swift](https://github.com/realm/realm-swift)
是什么？
RealmSwift 是一种移动端数据库，用于在 iOS、iPadOS 和 macOS 应用程序中存储和管理数据。使用 RealmSwift，您可以轻松地在应用程序中创建、读取、更新和删除对象，而无需编写 SQL 查询语句。<br>
先定义实体，继承Object，然后获取Realm实例，通过这个实例，配置filter来过滤符合条件的对象。

### 2.11 如何使用MJRefresh?
> 地址：[https://github.com/CoderMJLee/MJRefresh](https://github.com/CoderMJLee/MJRefresh)
是什么？
MJRefresh是一个iOS开发中用于下拉刷新和上拉加载更多的第三方库，可以快速集成到iOS项目中。<br>
这个是objc写的，需要配置桥接文件。
扩展了tableView，直接通过扩展方法即可访问。

### 2.12 如何使用RxSwift?
> 地址：[https://github.com/ReactiveX/RxSwift](https://github.com/ReactiveX/RxSwift)
是什么？
RxSwift是一个基于Rx标准的Swift版本，它提供了一套函数响应式编程的框架，可以帮助我们更好地处理异步事件。<br>
1.创建Observable
2.创建Observer
3.应用操作符，如过滤、转换、合并、聚合，map等。
map：将Observable的事件序列中的每个元素都映射成另一个元素。
filter：过滤Observable的事件序列中的元素，只保留符合条件的元素。
flatMap：将Observable的事件序列中的每个元素都转换成一个新的Observable，并将这些Observable合并成一个新的Observable。
scan: 对Observable的事件序列中的元素进行累加操作，并将每一步的累加结果发送出去。
distinctUntilChanged：去除Observable的事件序列中连续重复的元素。
throttle：在一定时间内只发送Observable事件序列中的第一个元素，忽略其余的元素。

## 3 白银
> 多线程，网络能力。

### 3.1 什么是GCD？
GCD 是 Grand Central Dispatch 的缩写。
Grand Central Dispatch (GCD)是 Apple 开发的一个多核编程的较新的解决方法。在 Mac OS X 10.6 雪豹中首次推出，并在最近引入到了 iOS4.0。
GCD 是一个替代诸如 NSThread 等技术的很高效和强大的技术。GCD 完全可以处理诸如数据锁定和资源泄漏等复杂的异步编程问题。
GCD 可以完成很多事情，但是这里仅关注在 iOS 应用中实现多线程所需的一些基础知识。

## 4 黄金
> 性能优化架构能力。

### 4.1 造成tableView卡顿的原因有哪些？

### 4.2 如何提升 tableview 的流畅度？

### 4.3 APP启动时间应从哪些方面优化？

### 4.4 如何降低APP包的大小？

### 4.5 日常如何检查内存泄露？

### 4.6 iOS有哪些常见的设计模式？

### 4.7 单例会有什么弊端？

### 4.8 MVC、MVP、MVVM模式？

### 4.9 编程中的六大设计原则？

## 5 铂金

### 5.1 

### 5.1 

## 6 钻石

## 7 大师

## 8 宗师

## 9 王者
