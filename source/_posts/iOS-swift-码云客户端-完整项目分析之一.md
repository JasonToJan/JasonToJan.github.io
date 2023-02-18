---
title: iOS swift 码云客户端 完整项目分析之一
date: 2023-02-13 17:37:25
op: false
cover: false
toc: true
mathjax: true
tags:
- 完整项目 RxSwift 
categories:
- iOS
---

## 1 项目地址
> [https://gitee.com/oschina/git-osc-iphone](https://gitee.com/oschina/git-osc-iphone) 
使用Swift语言重构的码云iOS客户端，采用MVVM设计模式与POP(面向协议编程)，核心框架为RxSwift。 

效果如下：
| <img src=mayun_01.png> | <img src=mayun_02.png>  | <img src=mayun_03.png>  |
| :---------------: | :---------------: | :---------------: |


## 2 项目配置

### 2.1 info.plist
> 在Xcode中，每个项目都必须有一个名为info.plist的文件。这个文件包含了关于项目的重要配置信息，例如应用程序的名称、图标、版本号等。它还可以用于控制应用程序的行为，例如定义应用程序支持的屏幕方向、使用的隐私权限等。因此，info.plist是项目的一个关键文件，不能缺少。

| key  | value |
| ---- | ----- |
| Bundle Identifier    | (com.example.app) - 此字符串唯一标识您的应用程序     |
| Bundle Name    | 应用程序的内部名称，它是用于程序内部的引用，如在代码中的识别。     |
| Bundle display name    | 是应用程序的用户可见名称，它在iOS设备的应用程序图标下显示。   |
| Bundle Version   | (1.0) - 此数字表示应用程序的版本号。     |
| Bundle Version String    | (1.0.0) - 此字符串是人类可读的版本号。     |
| Bundle Executable     | (MyApp) - 此字符串是主可执行文件的名称。     |
| Main Storyboard file base name    | (Main) - 此字符串是主要的Storyboard文件的名称。  |
| Launch Screen File    |(LaunchScreen) - 此字符串是应用程序启动时显示的屏幕文件的名称。 |
| Icon files    | (AppIcon) - 此键包含您的应用程序图标的文件名。     |
| Status bar style     | (UIStatusBarStyleLightContent) - 此键控制状态栏的样式，例如：浅色或深色。     |
| Required device capabilities    | (armv7) - 此键指定您的应用程序需要的设备功能，例如：armv7或arm64。     |
| Application requires full screen    | (false) - 此键指示应用程序是否需要全屏显示。     |
| Initial interface orientation    | (UIInterfaceOrientationPortrait) - 此键指定应用程序启动时的初始界面方向，例如：纵向或横向。     |
| Supported interface orientations     | (UIInterfaceOrientationMaskAllButUpsideDown) - 此键指定支持的界面方向。     |
| UIStatusBarStyle    | 控制状态栏的样式，可以选择黑色或白色     |
| UILaunchStoryboardName    | 指定启动时加载的故事板文件     |
| UIApplicationExitsOnSuspend    | 控制应用程序在挂起时是否退出     |
| UIRequiredDeviceCapabilities    | 指定应用程序需要的设备功能     |
| UISupportedInterfaceOrientations    | 指定应用程序支持的接口方向     |
| UIPrerenderedIcon    | 指示是否在应用图标上预渲染阴影     |
| CFBundleDisplayName    | 指定应用程序的显示名称，该名称将在用户的设备上显示。     |
| NSCameraUsageDescription    | 描述应用程序使用相机的原因     |
| NSPhotoLibraryUsageDescription    | 描述应用程序使用相册的原因     |
| NSBluetoothPeripheralUsageDescription    | 描述应用程序使用蓝牙外围设备的原因     |
| NSLocationWhenInUseUsageDescription    | 描述应用程序使用位置服务的原因，当应用程序在使用时  |
| NSMicrophoneUsageDescription    | 描述应用程序使用麦克风的原因    |
| NSAppTransportSecurity    | 用于控制应用程序的网络安全性     |
| LSApplicationQueriesSchemes    | 定义应用程序能够查询的其他应用程序的 URL Schemes。    |
| UIApplicationShortcutItems    |  用于定义3D Touch快捷操作。     |
| UIMainStoryboardFile    |  指定主故事板文件的名称。     |
| UIBackgroundModes    | 指定应用程序支持的后台模式，例如处理音频或位置更新。     |

另外在info.plist中，可以会用到外部定义的常量，以${}这种方式定义：
> 使用该形式的常量是Xcode中的环境变量。这些常量定义在项目的Build Settings中，可以在info.plist中引用。
要在Build Settings中定义环境变量，请执行以下操作：
选择项目文件，然后单击“Build Settings”标签。
在搜索框中输入“User-Defined”。
单击“+”按钮以添加新的用户定义的变量。
在“Key”字段中输入变量名称，在“Value”字段中输入变量值。
然后，您可以在info.plist中使用“$ {}”语法来引用该变量。例如，如果您已定义了一个叫做"APP_NAME"的变量，则可以在info.plist中使用"${APP_NAME}"来引用该变量。

### 2.2 新增Localizable.strings文件

> 在 Swift 项目中，你可以使用 Xcode 的国际化功能来生成 Localizable.strings 文件。
步骤如下：
选择你的项目文件，然后点击 "Editor" -> "Export for Localization"
在弹出的文件选择器中，选择一个目录并保存。
右键点击刚刚生成的文件夹，选择 "New File"
选择 "Strings File" 模板，然后单击 "Next"
命名文件为 "Localizable.strings"，然后单击 "Create"
这将生成一个名为 "Localizable.strings" 的文件，并且你可以开始使用它在你的项目中管理本地化字符串。

对于每种语言，请创建一个以语言代码命名的文件夹，并在其中复制 "Localizable.strings" 文件。例如，如果您要支持英语和中文，请创建 "en.lproj" 和 "zh-Hans.lproj" 两个文件夹。

其它国家如下：
> 对于其它国家的语言，命名规则也是类似的。通常采用语言代码 + 国家/地区代码的形式，如德语的文件夹命名为 de.lproj，德语（德国）的文件夹命名为 de-DE.lproj。具体的语言代码和国家/地区代码可以查询 ISO 639 和 ISO 3166 标准。
常见的语言代码有：
zh-Hans：简体中文
en：英语
de：德语
fr：法语
es：西班牙语
ja：日语
ru：俄语
常见的国家/地区代码有：
CN：中国
US：美国
DE：德国
FR：法国
ES：西班牙
JP：日本
RU：俄罗斯
最终的命名方式应该是语言代码 + 国家/地区代码，如中文（中国）的文件夹命名为 zh-Hans-CN.lproj。

用的时候这样引用下就行：
```Swift
let str =  NSLocalizedString("test", comment: "")
print(str)
```

国际化方面，可以参考下这篇文章：
[https://blog.csdn.net/M316625387/article/details/123083923](https://blog.csdn.net/M316625387/article/details/123083923)

### 2.3 Swift Compiler-General
注意到这里有配置Objective-C Bridging Header 到某一个头文件。
<img src=mayun_04.png>

文件具体内容是这样的：
```c
#ifndef Git_OSC_Bridging_Header_h
#define Git_OSC_Bridging_Header_h

#import <MJRefresh/MJRefresh.h>
#import <MBProgressHUD/MBProgressHUD.h>
#import <SDWebImage/SDWebImage.h>
#import "HCDropdownView.h"
//#import "HttpsURLProtocol.h"


#endif /* Git_OSC_Bridging_Header_h */
```

这个文件具体有什么作用呢？
> Objective-C Bridging Header 文件配置的作用是将 Objective-C 代码与 Swift 代码进行混编。它允许 Swift 代码访问 Objective-C 的类、方法和属性，并且可以使用 Objective-C 的第三方库。这种混编的能力对于逐步将现有的 Objective-C 代码迁移到 Swift 中非常有用。在 Bridging Header 文件中，可以使用 #import 或者 #include 指令来引入 Objective-C 的头文件，这些头文件中包含了需要在 Swift 代码中使用的 Objective-C 类和方法的声明。

## 3 项目入口

首先是AppDelegate.swift文件：
```Swift
import UIKit
import RxSwift
import RxCocoa
import Foundation
import MonkeyKing
import Alamofire

@UIApplicationMain
class AppDelegate: UIResponder, UIApplicationDelegate {

    var window: UIWindow?
    var coordinator: Coordinator?
    private let bag = DisposeBag()
    
    private func setupBar() {
        let bar = UINavigationBar.appearance()
        bar.isTranslucent = false
        bar.barTintColor = .theme
        bar.tintColor = .white
        bar.titleTextAttributes = [NSAttributedString.Key.foregroundColor: UIColor.white]
    }
    
    private func registerMonkeyKingAccount() {
        let wcAccount = MonkeyKing.Account.weChat(appID: ShareConfigs.WeChat.appID, appKey: ShareConfigs.WeChat.appKey, miniAppID: nil, universalLink: nil)
        MonkeyKing.registerAccount(wcAccount)
        
        let qqAccount = MonkeyKing.Account.qq(appID: ShareConfigs.QQ.appID, universalLink: nil)
        MonkeyKing.registerAccount(qqAccount)
        
        let wbAccount = MonkeyKing.Account.weibo(appID: ShareConfigs.Weibo.appID, appKey: ShareConfigs.Weibo.appKey, redirectURL: "http://sns.whalecloud.com/sina2/callback")
        MonkeyKing.registerAccount(wbAccount)
    }
    
    func application(_ application: UIApplication, didFinishLaunchingWithOptions launchOptions: [UIApplication.LaunchOptionsKey: Any]?) -> Bool {
        // Override point for customization after application launch.
        setupBar()
        registerMonkeyKingAccount()
        URLCache.shared = URLCache(memoryCapacity: 10 * 1024 * 1024, diskCapacity: 10 * 1024 * 1024, diskPath: nil)
        
        URLProtocol.registerClass(HttpsURLProtocol.self)
        
        CurrentUserManger.isLogin.bind { [weak self] _ in
            self?.coordinator = Coordinator()
            self?.window = UIWindow(frame: UIScreen.main.bounds)
            self?.window?.rootViewController = self?.coordinator?.rootController
            self?.window?.makeKeyAndVisible()
        }.disposed(by: bag)
    
        return true
    }
    
    func application(_ app: UIApplication, open url: URL, options: [UIApplication.OpenURLOptionsKey : Any] = [:]) -> Bool {
        if MonkeyKing.handleOpenURL(url) {
            return true
        }
        return false
    }

    func applicationWillResignActive(_ application: UIApplication) {
        // Sent when the application is about to move from active to inactive state. This can occur for certain types of temporary interruptions (such as an incoming phone call or SMS message) or when the user quits the application and it begins the transition to the background state.
        // Use this method to pause ongoing tasks, disable timers, and invalidate graphics rendering callbacks. Games should use this method to pause the game.
    }

    func applicationDidEnterBackground(_ application: UIApplication) {
        // Use this method to release shared resources, save user data, invalidate timers, and store enough application state information to restore your application to its current state in case it is terminated later.
        // If your application supports background execution, this method is called instead of applicationWillTerminate: when the user quits.
    }

    func applicationWillEnterForeground(_ application: UIApplication) {
        // Called as part of the transition from the background to the active state; here you can undo many of the changes made on entering the background.
    }

    func applicationDidBecomeActive(_ application: UIApplication) {
        // Restart any tasks that were paused (or not yet started) while the application was inactive. If the application was previously in the background, optionally refresh the user interface.
    }

    func applicationWillTerminate(_ application: UIApplication) {
        // Called when the application is about to terminate. Save data if appropriate. See also applicationDidEnterBackground:.
    }

}
```

### 3.1 @UIApplicationMain注解

> "@UIApplicationMain" 注解是 iOS 开发中的 Swift 语言功能。它用于指定带有此注解的 Swift 类是应用程序的主入口点。"UIApplicationMain" 类负责管理应用程序的主运行循环，这是将事件分派给应用程序的 UI 和其他对象的主事件循环。
当 Swift 应用程序启动时，系统会寻找 "main.swift" 文件作为执行的起点。但是，对于 iOS 应用程序，通常不使用 "main.swift" 文件，而是使用 "@UIApplicationMain" 注解定义主入口点。通过在类上标记此注解，您告诉 Swift 编译器此类是应用程序的主入口点，它应该为您生成 "main.swift" 文件。
在大多数情况下，您将使用 "UIApplicationMain" 类作为应用程序的主入口点，因为它提供了 iOS 应用程序所需的大部分基本功能，例如事件处理和管理应用程序的状态。但是，如果需要对主入口点进行更多控制，您也可以创建自己的自定义类并在其上标记 "@UIApplicationMain" 注解。

### 3.2 继承UIResponder

> 在 iOS 中，AppDelegate 通常是 UIResponder 类的子类。这是因为 UIResponder 类为 iOS 中响应和处理事件提供了基本的基础设施，例如触摸事件和其他用户生成的事件。
AppDelegate 是 iOS 应用程序的重要组成部分，负责处理各种应用级事件，例如应用程序启动、应用程序终止和应用程序状态的更改。通过从 UIResponder 类继承，AppDelegate 可以接收并响应这些事件，以符合 iOS 中整体事件处理机制。
此外，UIResponder 类提供了许多可以重写来处理特定类型事件的方法。例如，可以重写 touchesBegan 方法来响应触摸事件，而可以重写 applicationWillResignActive 方法来处理对应用程序活动状态的更改。
通过从 UIResponder 类继承，AppDelegate 可以提供一个集中和一致的处理这些应用级事件的方式，并使得开发人员更容易编写与 iOS 系统和用户交互的代码。

主要是因为AppDelegate一般作为应用入口，而UIResponser类为iOS提供响应事件提供了基础。这样可以更容易处理交互问题。

### 3.3 didFinishLaunchingWithOptions方法

> didFinishLaunchingWithOptions 方法是 iOS 中 UIApplicationDelegate 协议的代理方法。当应用程序启动完成时，它将被调用，用于执行应用程序所需的任何必要的设置或配置。
这个方法通常是应用程序启动时第一个被调用的方法之一，它传递一个包含应用程序启动选项的 NSDictionary 对象。这些选项可以包含诸如应用程序启动的原因（例如，用户点击了通知）或需要传递给应用程序的数据等信息。
在 didFinishLaunchingWithOptions 方法中，您可以执行各种任务，例如设置应用程序的用户界面、初始化必要的数据结构或配置应用程序的行为。例如，您可以使用此方法设置应用程序的根视图控制器，或注册推送通知。
总的来说，didFinishLaunchingWithOptions 方法是 iOS 应用程序启动过程的关键组成部分，它为您提供了在应用程序完全激活之前执行任何必要的设置和配置的方便地点。

通常，我们会将一些初始化代码放这里。可以配置根控制器。

```Swift
func application(_ application: UIApplication, didFinishLaunchingWithOptions launchOptions: [UIApplication.LaunchOptionsKey: Any]?) -> Bool {
    // Override point for customization after application launch.
    setupBar()
    registerMonkeyKingAccount()
    URLCache.shared = URLCache(memoryCapacity: 10 * 1024 * 1024, diskCapacity: 10 * 1024 * 1024, diskPath: nil)
    
    URLProtocol.registerClass(HttpsURLProtocol.self)

    UIApplication.shared.statusBarStyle = .lightContent
    
    CurrentUserManger.isLogin.bind { [weak self] _ in
        self?.coordinator = Coordinator()
        self?.window = UIWindow(frame: UIScreen.main.bounds)
        self?.window?.rootViewController = self?.coordinator?.rootController
        self?.window?.makeKeyAndVisible()
    }.disposed(by: bag)

    return true
}
```

首先执行自定义函数setupBar:
```Swift
private func setupBar() {
    let bar = UINavigationBar.appearance()
    bar.isTranslucent = false
    bar.barTintColor = .theme
    bar.tintColor = .white
    bar.titleTextAttributes = [NSAttributedString.Key.foregroundColor: UIColor.white]
}
```
UINavigationBar是导航栏，属于顶部导航栏。
> 这段代码是用来设置导航栏的外观。它首先创建了一个 UINavigationBar 对象，并将该对象的外观设置为全局生效。
bar.isTranslucent 设置为 false 表示导航栏不透明。
bar.barTintColor 设置导航栏的背景颜色。
bar.tintColor 设置导航栏元素（如按钮）的颜色。
bar.titleTextAttributes 设置导航栏标题的字体颜色。

然后这里注册了社交平台信息：
```Swift
private func registerMonkeyKingAccount() {
    let wcAccount = MonkeyKing.Account.weChat(appID: ShareConfigs.WeChat.appID, appKey: ShareConfigs.WeChat.appKey, miniAppID: nil, universalLink: nil)
    MonkeyKing.registerAccount(wcAccount)
    
    let qqAccount = MonkeyKing.Account.qq(appID: ShareConfigs.QQ.appID, universalLink: nil)
    MonkeyKing.registerAccount(qqAccount)
    
    let wbAccount = MonkeyKing.Account.weibo(appID: ShareConfigs.Weibo.appID, appKey: ShareConfigs.Weibo.appKey, redirectURL: "http://sns.whalecloud.com/sina2/callback")
    MonkeyKing.registerAccount(wbAccount)
}

struct ShareConfigs {
    struct QQ {
        static let appID = "1101982202"
        static let appKey = "c7394704798a158208a74ab60104f0ba"
    }
    struct WeChat {
        static let appID = "wx850b854f6aad6764"
        static let appKey = "39859316eb9e664168d2af929e46f971"
    }
    struct Weibo {
        static let appID = "3645105737"
        static let appKey = "3fbd38f46f9a2dd0207160c4a8d82149"
    }
}
```

> 这段代码是在 Swift 中使用 MonkeyKing 库来注册三个社交平台的账号信息。
首先，创建了一个微信账号，并使用 MonkeyKing.Account.weChat 方法进行注册。该方法需要传入微信的 appID，appKey 和 miniAppID（如果使用小程序）。
然后，创建了一个 QQ 账号，并使用 MonkeyKing.Account.qq 方法进行注册。该方法需要传入 QQ 的 appID。
最后，创建了一个微博账号，并使用 MonkeyKing.Account.weibo 方法进行注册。该方法需要传入微博的 appID，appKey 和 redirectURL。
这些注册账号的信息将在以后使用 MonkeyKing 进行分享时使用。请注意，这段代码仅是示例代码，你需要替换成你自己的账号信息。

```Swift
 URLCache.shared = URLCache(memoryCapacity: 10 * 1024 * 1024, diskCapacity: 10 * 1024 * 1024, diskPath: nil)
```
> 这段代码是创建并设置了一个 URLCache 对象，并将其赋值给 URLCache.shared 单例。
URLCache 是 iOS 中的一个类，用于缓存网络请求的数据。它可以加速网络请求的响应速度，降低网络流量消耗。
这段代码创建了一个 URLCache 对象，它具有 10 MB 的内存容量和 10 MB 的磁盘容量，不使用磁盘路径。然后，该对象将作为全局的缓存对象，使用 URLCache.shared 单例进行引用。

```
URLProtocol.registerClass(HttpsURLProtocol.self)
```
> 这行代码注册了一个类，使其作为一个 URL 协议处理器。
URLProtocol 是 iOS 中的一个类，用于提供对 URL 请求的控制。它允许你拦截并修改 URL 请求，以便对请求进行自定义处理。
在这行代码中，HttpsURLProtocol.self 类被注册为 URL 协议处理器。因此，任何使用 https 协议的 URL 请求都将由 HttpsURLProtocol 类处理。

```Swift
CurrentUserManger.isLogin.bind { [weak self] _ in
    self?.coordinator = Coordinator()
    self?.window = UIWindow(frame: UIScreen.main.bounds)
    self?.window?.rootViewController = self?.coordinator?.rootController
    self?.window?.makeKeyAndVisible()
}.disposed(by: bag)
```
> 这是一个使用RxSwift（ReactiveX）的代码片段。它主要实现了在用户登录状态发生变化时，更新UI界面的功能。
BehaviorRelay 是一个可观察的可变的值。它可以在任何时候通过访问value属性获取当前的值。
bind 方法用于将值的变化与特定的操作绑定，当值发生变化时，该操作会自动触发。
disposed(by:) 方法用于在不再需要绑定时销毁绑定。
在这个代码片段中，当 BehaviorRelay 的值发生变化时，就会触发更新UI界面的操作。窗口是由一个名为Coordinator的类生成的，并将其作为窗口的根视图控制器，并使窗口变为可见。

大致意思就是检查isLogin变量，发生变化，都将通知这里切换响应的控制器。
注意下，首次执行代码，也是会执行到闭包中的。
> 这个代码是创建一个 BehaviorRelay 对象并绑定一个闭包，闭包内的代码会在绑定时立刻执行一次，因为初始化时 isLogin 值被传入到了 BehaviorRelay 对象中。如果 isLogin 的值变化了，BehaviorRelay 对象会再次执行闭包代码，以更新界面。


然后注意到这里有个代码片段：
```Swift
UIApplication.shared.statusBarStyle = .lightContent
```
> 如果你想将 iOS 状态栏图标和系统时间显示的颜色设置为白色，可以使用以下方法：
在 Info.plist 文件中添加 "View controller-based status bar appearance" 选项，并将其设置为 "NO"。这样可以确保应用程序中所有视图控制器的状态栏样式都一致。
然后在应用启动方法didFinishLaunchingWithOptions中设置UIApplication的statusBarStye为亮色，这样状态栏时间wifi都显示为白色了。

### 3.4 open url 
```Swift
func application(_ app: UIApplication, open url: URL, options: [UIApplication.OpenURLOptionsKey : Any] = [:]) -> Bool {
    if MonkeyKing.handleOpenURL(url) {
        return true
    }
    return false
}
```
> 这是一个 iOS 的系统回调方法，它被称为 open URL 方法，主要是用于处理应用间跳转（第三方应用通过 URL 跳转到当前应用）。
在这个方法中，通过调用 MonkeyKing.handleOpenURL 方法，可以判断是否是第三方应用跳转到当前应用。如果是，则通过这个方法处理相关的跳转事件，并返回 true；否则返回 false。
为了方便使用，MonkeyKing.handleOpenURL 方法封装了对 URL 的处理过程，使得开发人员可以更简单地处理应用间的跳转。

### 3.5 applicationWillResignActive
```Swift
func applicationWillResignActive(_ application: UIApplication) {

}
```
> applicationWillResignActive方法是在应用程序即将从活动状态切换到非活动状态时调用。这可能是因为某些类型的临时中断（例如来电或短信）或用户退出应用程序，并开始向后台状态转换时。
你可以使用这个方法暂停正在进行的任务，禁用定时器并使图形渲染回调失效。游戏应该使用此方法暂停游戏。

### 3.6 applicationDidEnterBackground
```Swift
func applicationWillEnterForeground(_ application: UIApplication) {

}
```
> applicationWillEnterForeground 方法是在应用程序从后台进入前台的时候被调用的。这个方法的主要作用是恢复应用程序的状态，例如重新启动定时器、恢复用户界面等，以便准备显示给用户。

### 3.7 applicationDidBecomeActive
```Swift
func applicationDidBecomeActive(_ application: UIApplication) {

}
```
> applicationDidBecomeActive(_:)方法是在应用程序重新进入活动状态时调用的方法。这可能是由于以下原因：
当应用程序从后台回到前台时
当应用程序从挂起状态恢复时
当应用程序从挂起状态转到活动状态时（例如，当某个音频处于播放状态时）
在此方法中，你可以执行一些重要的初始化任务，例如启动网络连接，注册通知，重新加载数据等。

### 3.8 applicationWillTerminate
```Swift
func applicationWillTerminate(_ application: UIApplication) {

}
```
> applicationWillTerminate(_:) 方法是 iOS 程序生命周期中的一个方法，它在应用程序即将终止时调用。这通常是在用户退出应用程序或系统终止应用程序，因为它占用了过多内存等原因。在这个方法中，你可以保存应用程序的当前状态，以便在下次启动应用程序时使用，并且还可以执行其他必要的清理操作。

## 4 Coordinator 选择首页

```Swift
/// Coordinator负责Controller之间的调度和为Controller提供ViewModel对象
final class Coordinator {
    private(set) var rootController: RootViewController?
    
    init() {
        
        let projectsController = PageViewController(controllers: [getItemsControllerWith(type: .featuredProjs) , getItemsControllerWith(type: .popularProjs), getItemsControllerWith(type: .latestProjs)], titles: [String.Local.featured, String.Local.popular, String.Local.latest])
        projectsController.navigationItem.title = String.Local.projects
        
        let projrctsNaviController = UINavigationController(rootViewController: projectsController)
        let discoverNaviController = UINavigationController(rootViewController: LanguageListController.init(viewModel: .init(store: .init(type: .languageList)), delegate: self))
        
        let mineNaviControlelr: UINavigationController
        
        if CurrentUserManger.isLogin.value {
            mineNaviControlelr = UINavigationController(rootViewController: getMinePageController())
        }
        
        else {
            mineNaviControlelr = UINavigationController(rootViewController: getLoginController())
        }
        
        rootController = RootViewController(childControllers: [projrctsNaviController, discoverNaviController, mineNaviControlelr], titles: [String.Local.projects, String.Local.discover, String.Local.mine], normalImgs: [#imageLiteral(resourceName: "projects"), #imageLiteral(resourceName: "discover"), #imageLiteral(resourceName: "mine")], selectedImgs: [#imageLiteral(resourceName: "projects_selected"), #imageLiteral(resourceName: "discover_selected"), #imageLiteral(resourceName: "mine_selected")])
    }
```
> 这是一段 Swift 代码，它用于初始化一个 Coordinator 对象的属性。它实例化了三个导航控制器，分别代表项目，发现，和我的页面。
在实例化控制器之前，使用 CurrentUserManger.isLogin.value 检查用户是否登录。如果用户已经登录，则创建一个带有“我的页面”控制器的导航控制器；如果用户没有登录，则创建一个带有“登录”控制器的导航控制器。
最后，使用这三个导航控制器，创建一个 RootViewController，并将其作为根控制器赋值给 rootController 属性。

注意到这里用了一个#imageLiteral:
> #imageLiteral(resourceName: ...) 是 Swift 中一种快捷方式，可以将图片资源直接嵌入程序包中。它的作用是引用工程中的图片资源。

### 4.1 获取登录控制器
```Swift
private func getLoginController() -> UIViewController {
    let sb = UIStoryboard(name: "Login", bundle: nil)
    let controller: LoginController = castOrFatalError(sb.instantiateInitialViewController())
    controller.delegate = self
    return controller
}
```
通过故事版获取到UIStoryboard，然后强制转换为LoginController。
首先看下这个故事版怎么构造的：
<img src=mayun_05.png>

### 4.2 获取我的控制器
```Swift
private func getMinePageController() -> UIViewController {
    let userId = CurrentUserManger.id
    return MinePageController(controllers: [getItemsControllerWith(type: .selfEvents), getItemsControllerWith(type: .userProjs(id: userId)), getItemsControllerWith(type: .staredProjs(id: userId)), getItemsControllerWith(type: .watchedProjs(id: userId))], titles: [String.Local.events, String.Local.projects, "Star", "Watch"], delegate: self)
}
```
> 这是一个名为 "getMinePageController" 的私有函数，返回一个 UIViewController 对象。它创建了一个 "MinePageController" 类的实例，使用不同的 "type" 参数调用 "getItemsControllerWith" 函数获取一些视图控制器并将它们放入一个数组中。同时，也创建了一个包含一些字符串的标题数组。这些标题将与每个视图控制器对应。 "delegate" 参数被设置为 "self"，表示当前对象符合 "MinePageController" 类的委托协议。最后，该函数返回创建的 "MinePageController" 对象。

### 4.3 获取其他控制器
这个getItemsControllerWith，主要是通过type枚举，返回相应的控制器：
```Swift
private func getItemsControllerWith(type: ItemsType, item: Item? = nil) -> UIViewController {
    switch type {
    case .featuredProjs, .popularProjs, .latestProjs, .userProjs(_), .languagedProjs(_), .staredProjs(_), .watchedProjs(_):
        return ProjectsController(viewModel: .init(store: .init(type: type)), delegate: self)
    case .userEvents(_), .selfEvents:
        return EventsController(viewModel: .init(store: .init(type: type)), delegate: self)
    case .files(let info):
        return FileTreeController(viewModel: .init(store: .init(type: type)), fileInfo: info, delegate: self)
    case .projectCommits(_):
        return ProjectCommitsController(viewModel: .init(store: .init(type: type)), delegate: self)
    case .commitsDetails(_, _):
        guard let item = item as? ProjectCommit else  { break }
        return ProjectCommitDetailsController(viewModel: .init(store: .init(type: type), commit: item), delegate: self)
    case .projectIssues(_):
        return IssuesController(viewModel: .init(store: .init(type: type)), delegate: self)
    case .issueNotes(_, _):
        guard let item = item as? Issue else  { break }
        return IssueNotesController(viewModel: .init(store: .init(type: type), issue: item), delegate: self)
    default: break
    }
    fatalError("Unexpected controller type")
}

enum ItemsType {
    case featuredProjs
    case popularProjs
    case latestProjs
    case userProjs(id: Int64)
    case languagedProjs(id: Int32, language: String)
    case userEvents(id: Int64)
    case selfEvents
    case files(info: [String: String])
    case staredProjs(id: Int64)
    case watchedProjs(id: Int64)
    case projectCommits(id: Int64)
    case projectBanrches(id: Int64)
    case projectIssues(id: Int64)
    case projectsSearch(query: String)
    case commitsDetails(proId: Int64, comId: String)
    case issueNotes(proId: Int64, issueId: Int64)
    case languageList
}
```
上面是type对应关系。

### 4.4 根控制器
前面4.1中，将3个控制器加入根控制器中，这里根控制器实际上也是一个UITabBarController的自定义。
```Swift
class RootViewController: UITabBarController, Bindable  {
    var disposeBag: DisposeBag = .init()
    
    init(childControllers: [UIViewController], titles: [String], normalImgs:[UIImage?], selectedImgs: [UIImage?]) {
        super.init(nibName: nil, bundle: nil)
        var i = 0
        for _ in childControllers {
            add(childController: childControllers[i], normalImage: normalImgs[i] ?? UIImage(), selectedImage: selectedImgs[i] ?? UIImage(), title: titles[i])
            i += 1
        }
        tabBar.isTranslucent = false
        
        NightModeViewModel.shared.normalBackgroud.bind(to: self.tabBar.rx.barTintColor).disposed(by: disposeBag)
    }
    
    required init?(coder aDecoder: NSCoder) {
        fatalError("init(coder:) has not been implemented")
    }

    func add(childController: UIViewController, normalImage: UIImage, selectedImage: UIImage, title: String) {
        self.addChild(childController)
        childController.tabBarItem.image = normalImage.withRenderingMode(.alwaysOriginal)
        childController.tabBarItem.selectedImage = selectedImage.withRenderingMode(.alwaysOriginal)
        childController.tabBarItem.title = title
    }
}
```

这里设置了根控制器，本质上是一个UITabBarController。
通过遍历add子控制器，将3个控制器加进去了。

## 5 RxSwift 夜间模式实现

### 5.1 绑定可变变量给视图某个属性
可以看到在init方法中有部分是这个代码：
```Swift
 NightModeViewModel.shared.normalBackgroud.bind(to: self.tabBar.rx.barTintColor).disposed(by: disposeBag)
```

> 这段代码使用了 RxSwift 框架来绑定一个背景颜色属性到一个 UITabBar 控件的 barTintColor 属性上，使得背景颜色可以动态改变。
具体来说，NightModeViewModel.shared.normalBackgroud 是一个 Observable 序列，它会不断发出背景颜色属性的新值。bind(to: self.tabBar.rx.barTintColor) 将这个序列与 UITabBar 控件的 barTintColor 属性进行双向绑定，这样当序列发出新值时，UITabBar 控件的背景颜色也会自动更新。
最后，disposed(by: disposeBag) 表示将这个订阅添加到 disposeBag 中，以便在需要时可以统一取消这个订阅，避免内存泄漏。

### 5.2 新建DisposeBag防止内存泄漏
首先注意到这个根控制器实现了一个协议：
```Swift
protocol Bindable {
    var disposeBag: DisposeBag { get }
}
```
具体有什么作用呢？
> 这个 Bindable 协议定义了一个 disposeBag 属性，它要求遵守这个协议的类型必须提供一个 DisposeBag 实例，通常用于管理 RxSwift 中的订阅。
在实现 Bindable 协议的类型中，可以在初始化方法中创建一个 DisposeBag 实例，并将其赋值给 disposeBag 属性。在使用 RxSwift 进行订阅时，可以将订阅对象添加到 disposeBag 中，这样当这个类型被释放时，disposeBag 中的订阅也会被自动取消，避免内存泄漏。
通过这个协议，我们可以更方便地管理 RxSwift 中的订阅，避免手动管理造成的繁琐问题，提高代码的可读性和可维护性。

因为实现了这个协议，所以在类里面必须对他赋值：
```Swift
 var disposeBag: DisposeBag = .init()
```
> 这行代码定义了一个 DisposeBag 类型的属性 disposeBag，并给它赋了一个默认值 DisposeBag()，表示如果没有显式地给 disposeBag 赋值，那么就会使用这个默认的 DisposeBag 实例来管理 RxSwift 中的订阅。
具体来说，DisposeBag 是 RxSwift 中的一个类，用于管理 Observable 订阅。当一个对象需要监听一个 Observable 时，可以将这个订阅添加到一个 DisposeBag 实例中，当这个对象被释放时，这个 DisposeBag 实例会自动取消其中的所有订阅，避免内存泄漏。因此，使用 DisposeBag 来管理订阅是 RxSwift 中的一个常见做法。
这里的 var disposeBag: DisposeBag = .init() 表示在定义 disposeBag 属性时，就给它赋上了一个默认的 DisposeBag 实例，这个实例会在初始化对象时被创建。这样，使用 disposeBag 时，就不需要每次都手动创建一个 DisposeBag 实例了。当然，如果需要的话，也可以手动赋值一个新的 DisposeBag 实例给 disposeBag 属性，来重新管理订阅。

### 5.3 夜间模式ViewModel层
```Swift
final class NightModeViewModel {
    
   private let bag = DisposeBag()
    
    static let shared = NightModeViewModel()
    
    let normalBackgroud = BehaviorSubject<UIColor>(value: .normalBackgroud)
```
> 这段代码定义了一个 NightModeViewModel 类，其中：
final 关键字表示这个类是一个最终类，不能被继承。
private let bag = DisposeBag() 定义了一个私有的 DisposeBag 实例，用于管理 RxSwift 中的订阅。
static let shared = NightModeViewModel() 定义了一个静态属性 shared，用于创建一个全局唯一的 NightModeViewModel 实例。
let normalBackgroud = BehaviorSubject<UIColor>(value: .normalBackgroud) 定义了一个 BehaviorSubject 属性 normalBackgroud，用于表示正常模式下的背景颜色。BehaviorSubject 是 RxSwift 中的一个类型，它既可以作为 Observable，也可以作为 Observer，可以用来发出事件和订阅事件。这里通过传入一个初始值 value: .normalBackgroud 来创建了一个带有默认值的 BehaviorSubject，这个默认值表示正常模式下的背景颜色。
综合起来，这段代码定义了一个全局唯一的 NightModeViewModel 单例，其中包含了一个用于管理 RxSwift 订阅的私有 DisposeBag 实例，以及一个表示正常模式下背景颜色的 BehaviorSubject 属性。通过这种方式，可以在整个应用程序中方便地访问同一个 NightModeViewModel 实例，并通过 BehaviorSubject 属性来获取和设置正常模式下的背景颜色。


前面是通过监听normalBackground的值，来绑定底部tabBar的颜色，这样子normalBackground只要改变，会同步到底部tabBar。

所以现在还有个点需要注意，哪里来修改normalBackground的值呢？

这里发现在这个夜间模式的构造函数有如下代码：
```Swift
 CurrentUserManger.isNightMode.bind { (_) in
    self.normalBackgroud.onNext(.normalBackgroud)
    self.deepBackgroud.onNext(.deepBackgroud)
    self.normalTextColor.onNext(.normalText)
    self.selectedColor.onNext(.selected)
    self.pageViewStyle.onNext(DNSPageStyle.custom)
    
}.disposed(by: bag)
```
这里就说明了，只有监听到夜间模式，就驱动所有的BehaiorSubject对象发送事件给观察者，然后就会通知到底部Tabbar的颜色了。

那这个CurrentUserManager.isNightMode的被观察者有是怎样的呢？

### 5.4 夜间模式被观察者对象
```Swift
class CurrentUserManger {
    
    static let isNightMode = BehaviorRelay(value: _isNightMode)

    static private var _isNightMode: Bool {
        if _isLogin == false {
            return false
        }
        return UserDefaults.standard.bool(forKey: Key.nightMode.rawValue + String(id))
    }   
```
找到始发地了，这里是用了持久化方法，是轻量级本地存储库：
> UserDefaults.standard.bool(forKey:) 是一个 UserDefaults 类的实例方法，用于读取 UserDefaults 中指定键对应的布尔值。UserDefaults 是一个轻量级的本地存储库，可以用于保存应用程序的配置信息、用户偏好设置、应用程序状态等，它是一个键值存储系统，可以将基本数据类型（如整数、浮点数、布尔值、字符串等）存储到本地文件系统中。
UserDefaults.standard.bool(forKey:) 方法用于读取 UserDefaults 中指定键对应的布尔值，它是一个持久化操作，读取到的值将一直保存在本地，即使应用程序被关闭或设备被重启也不会丢失。因此，可以使用它来保存应用程序的状态或用户的偏好设置等。

哪里触发isNightMode改变的地方，就是事件发起的地方。
有很多地方会触发isNightMode改变，但是如果从0开始安装打开app，会有一个默认值，默认值就是先走上面那个_isNightMode里面拿bool值，这里判断是否登录，没有登录直接返回false。
然后，一层一层发到这里：
```Swift
private init() {
    CurrentUserManger.isNightMode.bind { (_) in
        print("这里走到夜间模式了")
        self.normalBackgroud.onNext(.normalBackgroud)
        self.deepBackgroud.onNext(.deepBackgroud)
        self.normalTextColor.onNext(.normalText)
        self.selectedColor.onNext(.selected)
        self.pageViewStyle.onNext(DNSPageStyle.custom)
        
    }.disposed(by: bag)
}
```
然后触发了底部tabBar颜色变化。

### 5.5 其它地方触发夜间模式变量更新
```Swift
static func saveNightMode<O: ObservableType>() -> (_ source: O) -> Disposable where O.Element == Bool {
    return { source in
        source.bind(onNext: { (isNightMode) in
            CurrentUserManger.isNightMode.accept(isNightMode)
            UserDefaults.standard.set(isNightMode, forKey: Key.nightMode.rawValue + String(id))
        })
    }
}
```
在CurrentUserManager中，有上面这个方法，调用的地方再设置页面。
在这个回调中，发现会改变夜间模式变量，同样也会触发某些颜色整体变化。

> 这段代码是一个 Swift 函数，它是一个工厂函数，用于创建并返回另一个函数。
该函数定义了一个泛型参数 O，它是 ObservableType 的子类型。这意味着，在使用该函数时，可以将任意类型的 Observable 作为参数，只要该类型的元素是 Bool 值即可。
该函数返回的内部函数是一个参数为 Observable 的函数，并且返回一个 Disposable 对象。该内部函数使用 bind 方法绑定给定的 Observable，并在收到新的 Bool 值时执行一些操作：
将当前的 Bool 值传递给 CurrentUserManger.isNightMode 变量。
将该 Bool 值存储在 UserDefaults 中，以便在下次使用时可以读取该值。
因此，该函数的作用是保存夜间模式的状态。

这里有些同学可能不了解Where关键字的用法：
在Swift中，where关键字通常用于函数和泛型类型的定义，用于添加额外的条件限制。具体来说，可以在函数签名的参数列表后面使用where关键字来添加一个或多个条件，以限制该函数仅在满足这些条件时才可用。例如：
```Swift
func myFunction<T>(someParameter: T) where T: SomeProtocol, T: SomeOtherProtocol {
    // 函数实现
}
```
上面的函数myFunction使用where关键字来指定了一个名为T的泛型参数，同时添加了两个条件：T必须符合SomeProtocol协议，并且还必须符合SomeOtherProtocol协议。这意味着只有当传递给函数的参数类型符合这些条件时，该函数才能被调用。

除了在函数定义中使用where关键字外，还可以在泛型类型的定义中使用它，以类似的方式添加约束条件。


OK。
现在就剩哪里调用它了，怎么调用它了。
发现只有一个地方，就是在设置页面的夜间模式点击了开关会触发：
```Swift
 nightModeSwitch.rx.isOn.bind(to: CurrentUserManger.saveNightMode()).disposed(by: disposeBag)
```
这就好理解了，当用户点击开关的时候，触发CurrentUserManager走saveNightMode函数。
这里用了一个三方库的bind函数：
```Swift
/**
Subscribes to observable sequence using custom binder function.

- parameter binder: Function used to bind elements from `self`.
- returns: Object representing subscription.
*/
public func bind<Result>(to binder: (Self) -> Result) -> Result {
    binder(self)
}
```
该函数是对ObservableType做的扩展。
> 这段Swift代码定义了一个公共方法 bind，它接受一个函数类型的参数 binder，并返回调用 binder 函数后的结果。
该方法的泛型参数 Result 用于指定 binder 函数的返回类型。binder 函数使用 Self 类型的参数（即调用 bind 方法的对象自身）并返回一个 Result 类型的值。
最后，bind 方法将调用 binder 函数并传入 self 对象，然后返回 binder 函数的结果，该结果的类型是 Result。
因此，bind 方法的主要作用是将对象本身绑定到一个闭包中，并将闭包的返回值返回。在使用 bind 方法时，需要传递一个闭包作为参数，该闭包的类型必须与 bind 方法中的 Result 类型相同。

所以再回到上面：
```
static func saveNightMode<O: ObservableType>() -> (_ source: O) -> Disposable where O.Element == Bool {
    return { source in
        source.bind(onNext: { (isNightMode) in
            CurrentUserManger.isNightMode.accept(isNightMode)
            UserDefaults.standard.set(isNightMode, forKey: Key.nightMode.rawValue + String(id))
        })
    }
}
```
就是savaNightModel函数，这个函数没有参数。返回一个函数。
说明返回的这个函数，就是三方库中的`to binder: (Self)->Result`中的 （Self）-> Result这个函数。
定义为（_source: O）-> Disposable
> return { source in
就是定义了 回调，同时也是作为上方的参数

> -> source.bind 
这里返回的就是Disposable对象

所以上面的saveNightMode不用参数，调用这个方法，直接返回一个 （_source: O）-> Disposable 这种类型的函数。定义了入参和返回值的函数。

再回到上面：
```
 nightModeSwitch.rx.isOn.bind(to: CurrentUserManger.saveNightMode()).disposed(by: disposeBag)
```
这里开关通过扩展rx，这里三方库帮忙做的，通过监听isOn的Bool值，绑定了一个函数，什么函数呢？
就是一个入参是source，其实就是ObservableType,返回一个Disposable的函数。
怎么来返回这个Disposasble呢，答案就是根据入参source执行bind函数，这样来返回Disposable对象。

## 6 总结

* 熟悉info.plist文件定义，每个项目缺一不可，类似Android中的清单注册文件，可以在里面配置很多东西，应用名称，版本号，图标，隐私权限等。右键add Row，可以添加一列，带自动提示。

* 了解iOS国际化如何处理，主要是多文件字符串的形式，首先按照需求创建 ”zh-Hans.lproj“文件夹表示支持中文，如果支持其它国家就按照语言代码和区域代码新建lproj文件，然后新建Localizable.strings文件，将字符串添加进去。

* 了解如何在Swift项目中引用oc的三方库，需要在Swift Compile General中配置头文件。

* 熟悉应用全局入口，需要继承UIResponder接收用户响应，实现协议UIApplicationDelegate，然后需要在didFinishLaunchingWithOptions 这个方法里面，做一些初始化工作。

* 了解如何全局配置URLCache和URLProtocal代理协议，协议需要通过registerClass注册一个自定义协议类来实现。

* 学会使用RxSwift，通过 BehaviorRelay对象的bind函数，实现对一个值的监听，值变化时，会执行bind里面的闭包回调。然后需要统一管理DisposeBag()。

* 学会开关中的isOn对象绑定一个函数，函数入参是一个Source对象，返回source.bind的Dispose对象。这个bind可以监听到开关的状态，然后我们可以在不同状态下写我们自己的逻辑。