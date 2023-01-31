---
title: iOS swift 混编Flutter
date: 2023-02-01 20:05:20
top: false
cover: false
toc: true
mathjax: true
tags:
- 混编Flutter
categories:
- iOS
---

## 1 集成Flutter原因

最除是项目人员调动，导致产品和开发数量不匹配。领导打算使用跨平台方案，但原有项目都是基于原生实现，如果想转Flutter，肯定得先考虑混编方案，因为需求也是持续迭代的，而且市场上也是由先例，可行性肯定是没问题的。

所以我们考虑使用阿里的Flutter Boost来实现混编，因为已经经过大型App测试过的，我们项目完全是可以用起来的。

## 2 Flutter Boost 初识

从0开始实现混编，首先当然是了解 Flutter Boost如何使用。
先看下官方文档：[https://github.com/alibaba/flutter_boost/blob/master/docs/install.md](https://github.com/alibaba/flutter_boost/blob/master/docs/install.md)。

文档也很清楚，其它博客没必要重复看了。

大概几个步骤吧：
* 引用三方库
* 新建一个代理类，继承FlutterBoostDelegate
* AppDelegate初始化引擎

## 3 项目结构和引入三方库

这里我们将原生的代码和Flutter代码分两个仓库。
比如在iOS项目的父文件夹下创建一个Flutter项目，这里面就放Flutter的代码。

首先要明确一点，Flutter Boost是 Flutter代码里面用的三方库。
这里配置项目结构是混编所需，不是因为依赖了Flutter Boost才这样引用。

为什么原生能用Flutter Boost里面的一些类呢？

这里并不是原生依赖了。

原因是原生项目通过在podFile文件里面配置Flutter项目路径，建立起了链接关系。
此时原生就可以正常访问Flutter项目里面依赖的三方库提供的一些类了。
此时就可以在AppDelegate用Flutter Boost提供的类了。

我们是这样配置的：
```
# flutter 集成
flutter_application_path = '../../flutter-pin-module'
load File.join(flutter_application_path, '.ios', 'Flutter', 'podhelper.rb')

use_frameworks!

target 'GreeSalesSystem' do

# flutter 集成
  install_all_flutter_pods(flutter_application_path)
```

这里路径为父文件的父文件下的“flutter-pin-module”，我们将flutter部分单独放置在这个文件夹下，其实这个Flutter部分也是可以单独编译，单独用Android Studio或者VS Code跑起来。

最后再走一次pod install。

这样就成功引入Flutter Boost，可以愉快地在原生项目进行Flutter配置了。

另外可能会有编译问题,没问题就不用加了。
在podFile文件夹最下面需要加上这段代码：
```
post_install do |installer|
    flutter_post_install(installer) if defined?(flutter_post_install)
    installer.pods_project.build_configurations.each do |config|
        config.build_settings["EXCLUDED_ARCHS[sdk=iphonesimulator*]"] = "arm64"
    end
end
```

## 4 原生项目配置Flutter

### 4.1 自定义FlutterBoostDelegate

```
// Flutter Boost 架构Delegate
// 参考： https://github.com/alibaba/flutter_boost/blob/master/docs/install.md
class GMFlutterBoostDelegate : NSObject, FlutterBoostDelegate {
    
    ///您用来push的导航栏
    var navigationController: UINavigationController?
    
    ///用来存返回flutter侧返回结果的表
    var resultTable: Dictionary<String,([AnyHashable:Any]?)->Void> = [:];
    
    /// Flutter 跳转 原生
    func pushNativeRoute(_ pageName: String!, arguments: [AnyHashable : Any]!) {
        GMFlutterNavigator.pushNative(pageName, arguments: arguments)
        return
    }
    
    /// 原生 跳转 Flutter
    func pushFlutterRoute(_ options: FlutterBoostRouteOptions!) {
        //对这个页面设置结果
        resultTable[options.pageName] = options.onPageFinished;
        
        //用参数来控制是push还是pop
        let isPresent = (options.arguments?["isPresent"] as? Bool)  ?? false
        var isAnimated = (options.arguments?["isAnimated"] as? Bool) ?? true
        
        // 分销入库成功   distributeWarehousingSuccessPage
        var vc: FBFlutterViewContainer
        if options.pageName == DistributeWarehousingSuccessPage.pageName {
            vc = DistributeWarehousingSuccessPage()
        } else {
            .......
        }
        
        vc.hidesBottomBarWhenPushed = true
        vc.setName(options.pageName, uniqueId: options.uniqueId, params: options.arguments,opaque: options.opaque)
        
        
        //如果是present模式 ，或者要不透明模式，那么就需要以present模式打开页面
        if(isPresent || !options.opaque){
            self.navigationController?.present(vc, animated: isAnimated, completion: nil)
        }else{
            self.navigationController?.pushViewController(vc, animated: isAnimated)
        }
    }
    
    /// 原生 pop Flutter
    func popRoute(_ options: FlutterBoostRouteOptions!) {
        var animated = true
        if options.pageName == DisPromotionChooseShopStoreViewController.pageName {
            animated = false
        }
        //如果当前被present的vc是container，那么就执行dismiss逻辑
        if let vc = self.navigationController?.presentedViewController as? FBFlutterViewContainer, vc.uniqueIDString() == options.uniqueId {
            
            //这里分为两种情况，由于UIModalPresentationOverFullScreen下，生命周期显示会有问题
            //所以需要手动调用的场景，从而使下面底部的vc调用viewAppear相关逻辑
            if vc.modalPresentationStyle == .overFullScreen {
                
                //这里手动beginAppearanceTransition触发页面生命周期
                self.navigationController?.topViewController?.beginAppearanceTransition(true, animated: false)
                
                
                vc.dismiss(animated: animated) {
                    self.navigationController?.topViewController?.endAppearanceTransition()
                }
            }else{
                //正常场景，直接dismiss
                vc.dismiss(animated: animated, completion: nil)
            }
        }else{
            self.navigationController?.popViewController(animated: animated)
        }
        //否则直接执行pop逻辑
        //这里在pop的时候将参数带出,并且从结果表中移除
        if let onPageFinshed = resultTable[options.pageName] {
            onPageFinshed(options.arguments)
            resultTable.removeValue(forKey: options.pageName)
        }
    }
    
}
```

Flutter 跳转 原生 主要用这个：pushNativeRoute 方法
原生 跳转 Flutter 主要用这个： pushFlutterRoute 方法

### 4.2 承载Flutter的容器

其实说白了，Flutter页面本质上还是Controller。
我们调整到任何一个Flutter，其实都是先跳自己的Controller。
如下：
```
class DisPromotionChooseShopStoreViewController: FBFlutterViewContainer {
    
    static var pageName = "disPromotionChooseShopStore"
    
    override func viewDidLoad() {
        super.viewDidLoad()
    }


}
```
这里就是跳转到Flutter的某个页面。
因为它继承了FBFlutterViewContainer，这个是阿里提供的。

实际上就是一个Controller呀。

### 4.3 应用启动初始化

这个需要再AppDelegate中完成。
可以搞个扩展方法：
```
extension AppDelegate {
    /// 设置flutter
    func configFlutterBoost(_ application: UIApplication) {
        //创建代理，做初始化操作
        let delegate = GMFlutterBoostDelegate()
        FlutterBoost.instance().setup(application, delegate: delegate) { engine in
            
        }
    }
}
```
这里面的delegate就是我们前面新建的一个Delegate。

就这样混编没啥问题了。

## 5 混编打包

这里可能还有几个坑要注意下。

在Flutter模块下执行：flutter build ios 报错
 
 首先我要确保Flutter模块能够在iOS真机上跑起来

### 5.1 真机运行报错

参考了这篇文章：
[https://honkersk.github.io/2020/07/02/Flutter-09-Flutter%E7%9C%9F%E6%9C%BA%E8%B0%83%E8%AF%95%E9%94%99%E8%AF%AF%E5%A4%84%E7%90%86/](https://honkersk.github.io/2020/07/02/Flutter-09-Flutter%E7%9C%9F%E6%9C%BA%E8%B0%83%E8%AF%95%E9%94%99%E8%AF%AF%E5%A4%84%E7%90%86/)

我这边发现用Xcode单独跑，总是说有些类not found，所以我用Android Studio跑好像可以了。

如果有些依赖有问题可以执行如下命令：
```
flutter pub upgrade
pod update
```

如果还不行可以将flutter项目clean下：
```
flutter clean
flutter pub get
```
这样是全新的系统创建的.iOS文件夹，基本没啥问题。

### 5.2 执行flutter build ios 报错

这里报错了的话，先别google。

可以先 flutter clean 再flutter pub get
然后走 flutter build ios --release --no-codesign

如果有一些shake 图标报错，可以运行这个指令看下：
`flutter build ios --release --no-tree-shake-icons`

啥都不用改，一次不行，多走几次，我这里试了3次，就成功了。
<img src=fluttererror2.png>
这里尝试了几次build，结果成功生成产物了。
生成路径在flutter模块下的build下的ios文件夹中。


### 5.3 build ios 产物分析
<table><thead><tr><th>产物</th><th>介绍</th></tr></thead><tbody><tr><td>App.framework</td><td>Dart业务源码相关文件，在Debug模式下就是一个很小的空壳，在Release模式下包含全部业务逻辑。</td></tr><tr><td>flutter_assets</td><td>Flutter依赖的静态资源，如字体、图片等。<br><strong>在Flutter 1.9.1这种已经被合并进App.framework</strong>。</td></tr><tr><td>FlutterPluginRegistrant</td><td>Flutter插件注册制，自动注册Flutter插件。</td></tr><tr><td>Flutter.framework</td><td>Flutter库和引擎。</td></tr><tr><td>.symlinks</td><td>指向Flutter插件依赖库的实际地址。</td></tr></tbody></table>

### 5.4 启动App闪退问题

解决方案很简单：在原生主工程配置Release模式 进行archive编译构建
<img src=fluttererror03.png>


## 6 总结

* 明确Flutter混编是系统支持，不是阿里提供的能力，Flutter Boost只是封装好的一个方便混编的工具，其实完全可以不用Flutter Boost，只是为了更好实现混编而已。

* 三部曲，先引入依赖，通过Flutter模块引入，然后通过链接关系，原生可以访问到Flutter依赖，此时再配置Delegate，最后再初始化引擎。

* 打包遇到问题不要急，先保证在真机上能运行起来。然后再走 flutter build ios命令，一次不行，走多次，有其它问题加后缀解决。