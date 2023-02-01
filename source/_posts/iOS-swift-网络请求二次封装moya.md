---
title: iOS-swift-网络请求二次封装moya
date: 2023-02-01 16:44:45
top: false
cover: false
toc: true
tags:
- 网络请求
categories:
- iOS
---

## 1 github地址

> [https://github.com/chensx1993/moyaManager](https://github.com/chensx1993/moyaManager)

这个应该是一个小姐姐，我看到我们项目用了这个三方库，所以这边分析下封装细节。

简单介绍下吧：

moya是对Alamofire的再次封装。它可以实现各种自定义配置，真正实现了对网络层的高度抽象。

还有一个优秀的网络框架([github地址](https://github.com/mmoaay/Bamboots))，大家可以看看，跟`moya`对比一下。

有关moya的介绍可以看看: [Moya的使用](https://www.jianshu.com/p/2ee5258828ff)。

## 2 框架架构

<img src=moya1.png>

* Core应该是核心模块，存放网络状态管理员，Network.swift应该是最关键的暴露给开发者使用的类吧，所有的请求都是经过这层转发的，Request就是内部请求相关，比如get和post请求层的封装吧。

* Extension是扩展层，这里扩展了String，也扩展了网络请求基础返回体，这里可以根据项目来。

* Plugin是插件层，这个可以自由添加。

* Server是网络层，这里我们可以定义一些通过的请求参数，比如token啥的。

* API层，这就和我们自己定义的Api相关了，怎么请求，怎么传参这些定义。

* Error层，这里应该是异常层了。

## 3 实例分析

这里我们考虑用一个真实案例，来深入分析这个封装框架如何实现网络请求的。
这里就以登录接口为案例吧。

可以建立一个登录相关接口，比如修改密码啥的，跟登录有关的统一用一个Manager。
如下：
```
import Foundation
import Moya

// MARK: - 用户登录信息请求
let loginApiRequest = Networking<UserLoginAPIManagerService>()


enum UserLoginAPIManagerService {
    /// 账号密码登录
    case userAccountLogin(userName:String, passWord:String)
    /// 是否显示注册
    case isShowRegister
    /// 注册
    case register(_ params: [String: Any])
    /// 更新是否已经提醒用户修改密码标记
    case signNotFirstLogin(userId:String)
    /// 修改密码
    case changePassword(newPassword:String, oldPassword:String, username:String)
    /// 校验原密码
    case checkOldPassword(oldPassword:String, username:String)
    /// 登出
    case logOut

}

extension UserLoginAPIManagerService : MyServerType {
    
    //域名
    var baseURL: URL {
        switch self {
        case .register:
            return URL(string: mainHost)!
        case .isShowRegister:
            return URL(string: mainHost)!
        case .userAccountLogin:
            return URL(string: LoginHost)!
        case .signNotFirstLogin:
            return URL(string: mainHost)!
        case .changePassword,.checkOldPassword:
            return URL(string: mainHost)!
        default:
            return URL(string: LoginHost)!
        }
        
    }
    
    //接口路径
    public var path: String {
        switch self {
        case .userAccountLogin:
            return "oauth/token"
        case .register:
            return "v1/user/Register"
        case.isShowRegister:
            return "v1/user/iosIsEnabled"
        case .signNotFirstLogin:
            return "v1/exposure/password/flag"
        case .changePassword:
            return "v1/user/reset/myself"
        case .logOut:
            return "ssoLogout"
        case .checkOldPassword:
            return "v1/user/check/reset-password"
        }
    }
    
    //是否执行Alamofire验证
    var validate: Bool {
        return false
    }
    
    //验证方式
    var validationType: MyValidationType {
        return .none
    }
    
    //单元测试模拟的数据
    var sampleData: Data {
        return "{}".data(using: String.Encoding.utf8)!
    }
    
    //请求类型：get、post、delete、put
    var method: HTTPMethod {
        switch self {
        case .userAccountLogin(userName: _ , passWord: _),
                .register,
                .isShowRegister,
                .changePassword,
                .checkOldPassword:
            return .post
        default:
            return .get
        }
    }
    
    //请求任务:
    public var task: Task {
        switch self {
        case .userAccountLogin(let userName, let passWord):
            let secret = "1234".toMD5
            let params = ["client_id": clientId,
                          "client_secret":secret,
                          "username": userName,
                          "password": passWord] as [String : Any]
            return .requestParameters(parameters: params, encoding:URLEncoding.default)
        case .register(let params):
            return .requestParameters(parameters: params, encoding:JSONEncoding.default)
        case .isShowRegister:
            return .requestParameters(parameters: [: ], encoding: URLEncoding.default)
        case .logOut:
            return .requestPlain
        case .signNotFirstLogin(let userId):
            let params = ["userId": userId] as [String : Any]
            return .requestParameters(parameters: params, encoding:URLEncoding.default)
        case .changePassword(let newPassword, let oldPassword, let username):
            let params = ["newPassword":newPassword,
                          "oldPassword":oldPassword,
                          "username": username] as [String : Any]
            return .requestParameters(parameters: params, encoding: JSONEncoding.default)
        case .checkOldPassword(let oldPassword, let username):
            let params = ["oldPassword":oldPassword,
                          "username": username] as [String : Any]
            return .requestParameters(parameters: params, encoding: JSONEncoding.default)
        }
        
    }
    
    //请求头
    var appendHeaders: [String : String]? {
        switch self {
        case .register:
            return ["Content-Type": "application/json"]
        case .signNotFirstLogin,
                .checkOldPassword,
                .logOut:
            return [: ]
        case .changePassword:
            return ["menuPath": MenuPath.changePassword.info]
        default:
            return [: ]
        }
    }
    
}
```

首先是建立了一个常量，一个Networking包装了MyServerType，当然这个MyServerType和Networking都是封装好的。主要操作就是自定义这个MyServerType了。

这个接口基本就写好了。
请求头可配置，参数可配置，接口路径可配置，后端怎么改都不怕了。

然后就是我们调用的地方了。

在登录的地方这样用：
```
 loginApiRequest.requestJson(.userAccountLogin(userName: userName, passWord: safePassWord)) {result in
    let json = JSON.init(rawValue: result)
    if json?["code"].intValue == 1000 {
        MBProgressHUD.hide()
        guard let model = UserInformationData.deserialize(from: json?["data"].jsonString) else { return }
        self.saveLoginSuccessUserInformation(model: model, userName: userName, passWord: passWord)
        callBack(model)
        
        self.requestUploadLog(userName,1,"登录成功")
    }else{
        MBProgressHUD.hide()
        let str = json?["message"].stringValue ?? "登录失败"
        GMToast.showFailure(str)
        self.requestUploadLog(userName,0,str)
    }
    
} failure: { error in
    MBProgressHUD.hide()
    GMToast.showFailure()
}
```

loginApiRequest就是我们前面定义的登录类接口的常量。

因为这是一个Networking，所有它有requestJson方法，这个方法也是框架自己二次封装好的，等下具体分析下这里面的代码即可。

注意到这里有传参，需要关注下.userAccoutLogin是啥东西？
> 原来就是我们定义的泛型类，MyServerType，这里.userAccountLogin是一个枚举也是一个MyServerType，我们可以在这里面添加请求参数，这样就关联起来了。

```
enum UserLoginAPIManagerService {
    /// 账号密码登录
    case userAccountLogin(userName:String, passWord:String)
    /// 是否显示注册
    case isShowRegister
    /// 注册
    case register(_ params: [String: Any])
    /// 更新是否已经提醒用户修改密码标记
    case signNotFirstLogin(userId:String)
    /// 修改密码
    case changePassword(newPassword:String, oldPassword:String, username:String)
    /// 校验原密码
    case checkOldPassword(oldPassword:String, username:String)
    /// 登出
    case logOut

}
```
果然就是第一个“账号密码登录”。

所有对于使用还是相当方便的，只需要写一个Manager，就可以很方便请求接口了。

## 4 细节分析

继续上面的案例，当我们发送json请求时，走了一个框架封装好的方法，看下：
```
@discardableResult
    public func requestJson(_ target: T,
                            callbackQueue: DispatchQueue? = DispatchQueue.main,
                            progress: ProgressBlock? = .none,
                            success: @escaping JsonSuccess,
                            failure: @escaping Failure) -> Cancellable {
        return self.request(target, callbackQueue: callbackQueue, progress: progress, success: { (response) in
            do {
                let result = try JSON.init(data: response.data)
                let vc = UIViewController.current()
                if vc is LoginPageViewController {}else{
                    if response.statusCode == 403 || response.statusCode == 401 {
                        //token过期
                        if tokenInvalidHandle != nil {
                            tokenInvalidHandle()
                            return
                        }
                    }
                }
                #if DEBUG
                    print(result)
                #endif
                success(result)
            } catch (let error) {
                failure(error as? NetworkError ?? NetworkError.invalidURL)
            }
        }) { (error) in
            #if DEBUG
                print(error)
            #endif
            failure(error)
        }
    }
```
这里继续走了内部的self.request方法，T就是我们自定义的MyServerType。
里面回调是我们自己的逻辑，就是解析了下Json，然后就是403或401的时候，跳转了登录页，这里的逻辑不需要关注。

看下内部的request方法吧：
```
@discardableResult
    public func request(_ target: T,
                        callbackQueue: DispatchQueue? = DispatchQueue.main,
                        progress: ProgressBlock? = .none,
                        success: @escaping Success,
                        failure: @escaping Failure) -> Cancellable {
        return self.provider.request(target, callbackQueue: callbackQueue, progress: progress) { (result) in
            switch result {
            case let .success(response):
                #if DEBUG
                if let body = response.request?.httpBody,
                   let param = String(data: body, encoding: .utf8),
                   let interface = response.request?.url {
                    print("✅✅✅✅✅✅✅✅✅✅✅✅✅✅✅✅✅✅✅✅✅✅✅✅✅✅✅✅✅✅✅✅✅✅✅✅✅")
                    print("接口地址：\n\(interface)")
                    print("✅✅✅✅✅✅✅✅✅✅✅✅✅✅✅✅✅✅✅✅✅✅✅✅✅✅✅✅✅✅✅✅✅✅✅✅✅")
                    print("请求参数：\n\(param)")
                    print("✅✅✅✅✅✅✅✅✅✅✅✅✅✅✅✅✅✅✅✅✅✅✅✅✅✅✅✅✅✅✅✅✅✅✅✅✅")
                }
                #endif
                success(response);
            case let .failure(error):
                #if DEBUG
                print("❌❌❌❌❌❌❌❌❌❌❌❌❌❌❌❌❌❌❌❌❌❌❌❌❌❌❌❌❌❌❌❌❌❌❌❌❌❌❌❌❌❌❌❌❌❌❌❌❌❌❌❌❌❌❌❌")
                print(error)
                #endif
                let err = NetworkError.init(error: error)
                failure(err);
                break
            }
        }
    }
```
这里的回调队列，进度，都有默认实例，当然可以自由传递。
另外就是成功和失败的逃逸闭包了。

其实内部继续走了self.provider.request才是重中之重。


### 4.1 生产Moya Provider
首先看下provider怎么来的：
```
public struct Networking<T: MyServerType> {
    public let provider: MoyaProvider<T>
    
    public init(provider: MoyaProvider<T> = newDefaultProvider()) {
        self.provider = provider
    }
}

public static func newDefaultProvider() -> MoyaProvider<T> {
        return newProvider(plugins: plugins)
}

/// 如何新建MoyaProvider
func newProvider<T>(plugins: [PluginType],session: Session = newManager()) -> MoyaProvider<T> where T: MyServerType {
    
    return MoyaProvider(endpointClosure: Networking<T>.endpointsClosure(),
                        requestClosure: Networking<T>.endpointResolver(),
                        stubClosure: Networking<T>.APIKeysBasedStubBehaviour,
                        session: session,
                        plugins: plugins,
                        trackInflights: false
    )
}

/// 新建Provider需要的参数1
func newManager(delegate: SessionDelegate = SessionDelegate()) -> Session {
//    let configuration = URLSessionConfiguration.default
//    configuration.httpAdditionalHeaders = Alamofire.SessionManager.defaultHTTPHeaders
    let configuration = Alamofire.Session.default.session.configuration
    let session = Alamofire.Session(configuration: configuration, delegate: delegate, startRequestsImmediately: false)
    return session
}

/// 新建Provider需要的参数2
static var plugins: [PluginType] {
        let activityPlugin = NewNetworkActivityPlugin { (state, targetType) in
            switch state {
            case .began:
                if targetType.isShowLoading { //这是我扩展的协议
                    // 显示loading
                }
            case .ended:
                if targetType.isShowLoading { //这是我扩展的协议
                    // 关闭loading
                }
            }
        }
        
        return [
            activityPlugin, myLoggorPlugin
        ]
    }

```
这里创建的时候，就默认传了一个默认的Provider，这个是Moya官方的哦。

另外新建这个MoyaProvider的时候还需要3个闭包，我们通过Networking来生产：
```
static func endpointsClosure<T>() -> (T) -> Endpoint where T: MyServerType {
    return { target in
        var headers: [String: String] = target.headers ?? [:]
        if apiEnvironment == .product {
            //生产环境灰度
            if canary == true {
                headers["canary"] = "true"
            }
        }
        //生产环境api路径设置
        var str = ""
        str = URL(target: target).absoluteString
        if apiEnvironment != .test && apiEnvironment != .dev {
            str = str.replacingOccurrences(of: "-test", with: "", options: .literal, range: nil)
        }
        let absoluteString = str
        let defaultEndpoint = Endpoint(
            url: absoluteString,
            sampleResponseClosure: { target.sampleResponse },
            method: target.method,
            task: target.task,
            httpHeaderFields: headers
        )
        return defaultEndpoint;
    }
}

//测试网络错误,如超时等.
static func endpointResolver() -> MoyaProvider<T>.RequestClosure {
    return { (endpoint, closure) in
        do {
            var request = try endpoint.urlRequest()
            request.httpShouldHandleCookies = false
            request.timeoutInterval = WebService.shared.timeoutInterval
            closure(.success(request))
        } catch let error {
            closure(.failure(MoyaError.underlying(error, nil)))
        }
    }
}   

static func APIKeysBasedStubBehaviour<T>(_ target: T) -> Moya.StubBehavior where T: MyServerType {
        return target.stubBehavior;
    }
```
这里我们自己生产了3个闭包给moya，也是moya需要的3个闭包。

### 4.2 继续走Provider的request

```
/// Designated request-making method. Returns a `Cancellable` token to cancel the request later.
    @discardableResult
    open func request(_ target: Target,
                      callbackQueue: DispatchQueue? = .none,
                      progress: ProgressBlock? = .none,
                      completion: @escaping Completion) -> Cancellable {

        let callbackQueue = callbackQueue ?? self.callbackQueue
        return requestNormal(target, callbackQueue: callbackQueue, progress: progress, completion: completion)
    }
```
这个是moya三方库里面的代码了，简单看下就好，不是我们这期重点。

### 4.3 如何封装MyServerType

个人感觉这个也是非常关键，我们外部使用，需要建立一个枚举，同时也是一个MyServerType，这个携带了请求参数，请求方法之类的，总之它包装了一切我们请求需要的东西。

```
import Foundation
import Moya

public typealias HTTPMethod = Moya.Method
public typealias MyValidationType = Moya.ValidationType
public typealias MySampleResponse = Moya.EndpointSampleResponse
public typealias MyStubBehavior = Moya.StubBehavior

public protocol MyServerType: TargetType {
    var isShowLoading: Bool { get }
    /// 附加请求头（如需添加额外的，使用这个）
    var appendHeaders: [String : String]? { get }
    var parameters: [String: Any]? { get }
    var stubBehavior: MyStubBehavior { get }
    var sampleResponse: MySampleResponse { get }
}
```
首先，它这个继承了Moya自己的TargetType，还增加了自己额外的一些协议。

```
extension MyServerType {
    public var base: String { return WebService.shared.rootUrl}
    
    public var baseURL: URL { return URL(string: base)! }
    /// 请求头 （默认请求头 + appendHeaders），需求添加额外的改appendHeaders
    public var headers: [String : String]? {
        var result: [String : String]? = WebService().headers
        guard let appendHeaders = appendHeaders else { return result }
        result = WebService().headers?.merging(appendHeaders, uniquingKeysWith: { (_, new) in new })
        return result
    }
    public var appendHeaders: [String : String]? { return nil }
    public var parameters: [String: Any]? { return WebService.shared.parameters }
    
    public var isShowLoading: Bool { return false }
    
    public var task: Task {
        let encoding: ParameterEncoding
        switch self.method {
        case .post:
            encoding = JSONEncoding.default
        default:
            encoding = URLEncoding.default
        }
        if let requestParameters = parameters {
            return .requestParameters(parameters: requestParameters, encoding: encoding)
        }
        return .requestPlain
    }
    
    
    public var method: HTTPMethod {
        return .post
    }
    
    public var validationType: MyValidationType {
        return .successCodes
    }
    
    public var stubBehavior: StubBehavior {
        return .never
    }
    
    public var validate: Bool {
        return false
    }
    
    public var sampleData: Data {
        return "response: test data".data(using: String.Encoding.utf8)!
    }
    
    public var sampleResponse: MySampleResponse {
        return .networkResponse(200, self.sampleData)
    }
}
```
这里应该是实现了默认值的设定。
当然我们是可以更改的。

```
func myBaseUrl(_ path: String) -> String {
    if path.isCompleteUrl { return path }
    return WebService.shared.rootUrl;
}

func myPath(_ path: String) -> String {
    if path.isCompleteUrl { return "" }
    return path;
}

extension String {
    var isCompleteUrl: Bool {
        let scheme = self.lowercased()
        if scheme.contains("http") { return true }
        return false
    }
}
```
然后是其它工具方法。

这里有用到一个WebService类，这里面存放的也是一些默认值设定：
```
import Foundation
import UIKit
import AdSupport
import Alamofire
import Moya

class WebService: NSObject {
    
    var rootUrl: String = mainHost
    var headers: [String: String]? = defaultHeaders()
    var parameters: [String: Any]? = defaultParameters()
    var timeoutInterval: Double = 30.0
    //和服务器时间相差的时间戳:备注每次app恢复活跃状态需更新一次
    static var serviceTimeSpace: String?
    
    static let shared = WebService()
    override init() {}
    
    static func defaultHeaders() -> [String : String]? {
        var headers = ["x-flag": "iOS", "serverName": "APP", "clientId": clientId]
        if let token = Defaults[key: DefaultsKeys.access_token] {
            headers["Authorization"] = "Bearer" + token
        }
        let isMonopoly = Defaults[key: DefaultsKeys.isMonopoly] ?? false
        /// appVersion 字段，（1表示专业版，0表示简易版）
        headers["appVersion"] = isMonopoly ? "0" : "1"
        return headers
    }
    
    static func defaultParameters() -> [String : Any]? {
        return ["platform" : "ios",
            "version" : "1.2.3",
        ]
    }
    
    /**
     获取当前app版本号
     */
    static func getAppShortVersion() -> String {
        let infoDictionary = Bundle.main.infoDictionary!
        let version = infoDictionary["CFBundleShortVersionString"] as! String
        return version
    }
    
    /**
    获取当前请求User-Agent
     */
    static func getUserAgent() -> String {
        let infoDictionary = Bundle.main.infoDictionary!
        let version = infoDictionary["CFBundleShortVersionString"] as! String
        let userAgent = String(format: "GREEMall%@(%@; iOS %@; Scale/%.2f)", version, UIDevice.current.model, UIDevice.current.systemVersion, UIScreen.main.scale)
        
        return userAgent
    }
    
    /**
    获取服务器时间和计算时间差
     */
    static func setServiceTimeInterval(timeInterval:TimeInterval) {
        serviceTimeSpace = String(format: "%@", timeInterval)
    }
    
    /**
     获取服务器时间戳
     */
    static func getServiceTimeInterval() -> Int {
        return Int(serviceTimeSpace!)!
    }
    
    /**
     设置请求统一参数
     */
    static func getSystemParameterData() -> Dictionary<String, String> {
        var data = [String: String]()
        data["source"] = "iOS"
        var timeSpace: String = ""
        if serviceTimeSpace != nil {
            let now: TimeInterval = Date.init().timeIntervalSince1970
            let appTime: TimeInterval = now + Double(CLongLong(serviceTimeSpace!)!)
            timeSpace = String(format:"%@", appTime)
        } else {
            timeSpace = String(format:"%@", Date.init().timeIntervalSince1970)
        }
        data["t"] = timeSpace
        data["version"] = self.getAppShortVersion()
        data["ios_idfa"] = ASIdentifierManager.shared().advertisingIdentifier.uuidString
        return data
    }
    
    /**
     签名MD5处理
     */
    static func appendSign(paramterDic: Dictionary<String, String>, secrekey:String) -> String {
        var sign:String = ""
        var keys:Array = Array(paramterDic.keys)
        keys = keys.sorted()
        for key:String in keys {
            if key == "sign" {
                continue
            } else {
                let keyValue: String = String(format: "%@", paramterDic[key]!)
                sign.append(String(format: "%@%@", key, keyValue.EscapesValr))
            }
        }
        sign.append(secrekey)
        return sign.td.md5.uppercased()
    }
}
```

大致就是这样了。

## 5 总结

* 这个二次封装主要是基于Moya，简化moya流程，提供了一个MyServerType，封装了一些请求参数，请求方法，方便我们进行接口配置。

* 使用方法很简单，不过需要将一类接口单独放置，全部放一起不方便管理，这里比如登录相关接口统一用一个Manger配置，接口参数都写在这个Manager里面，其实也是由一定好处的。

* 这里将moya更加封装成一个系统了，我们需要的结果无非就是接口成功或失败，传参也是我们必要的，将必要的都封装起来了，总体上使用还是比较便捷。

* 如果有特殊请求，比如下载文件这种，需要我们单独抽一个方法处理，这里稍微增加了一点复杂度吧，一般的请求还是可以直接使用的。