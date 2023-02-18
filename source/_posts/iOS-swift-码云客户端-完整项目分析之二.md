---
title: iOS swift 码云客户端 完整项目分析之二
date: 2023-02-15 10:07:26
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

此篇文章讲解，如何构造一个3个Tab列表，列表请求网络，封装数据流对接RxSwift总体架构。
效果如下：
<img src=mayun_201.png width=50%>

## 2 入口
```Swift

/// Coordinator负责Controller之间的调度和为Controller提供ViewModel对象
final class Coordinator {

/// 我们需要在当前类中访问和修改这个属性的值，而在类外部只允许访问这个属性的值而不能修改它
private(set) var mRootController: MyTabBarController?

init() {
    // 项目tab
    let projrctsController = MyPageViewController(
        controllers: [
            getItemsControllerWith(type: .featuredProjs),
            getItemsControllerWith(type: .popularProjs),
            getItemsControllerWith(type: .latestProjs)],
        titles: [String.Local.featured, String.Local.popular, String.Local.latest])
    projrctsController.navigationItem.title = String.Local.projects
    let projrctsNaviController = UINavigationController(rootViewController: projrctsController)
    
    // 发现tab
    let discoverNaviController: UINavigationController = UINavigationController(rootViewController: getLoginSecondController())
    
    // 我的tab
    let mineNaviControlelr: UINavigationController = UINavigationController(rootViewController: getLoginController())
    
    mRootController = MyTabBarController(childControllers: [projrctsNaviController, discoverNaviController, mineNaviControlelr], titles: [String.Local.projects, String.Local.discover, String.Local.mine], normalImags: [#imageLiteral(resourceName: "projects"), #imageLiteral(resourceName: "discover"), #imageLiteral(resourceName: "mine")], selectedImgs: [#imageLiteral(resourceName: "projects_selected"), #imageLiteral(resourceName: "discover_selected"), #imageLiteral(resourceName: "mine_selected")])
}
```
首先是在AppDelegate里面new了一个这个对象，主要是统一管理控制器。
然后在它构造函数中，将项目的UINavigationController，和发现和我的添加到一个自定义的TabBarController。
App总体架构如上。
我们主要分析项目的UINavigationController怎么实现。

## 3 项目页构造
```Swift
class MyPageViewController: UIViewController, ProtocolBindable, ProtocolPageViewPresentable {
```
首先是自定义的UIViewController。

### 3.1 ProtocolBindable协议相关联
实现协议ProtocolBindable，这个很简单：
```Swift
protocol ProtocolBindable {
    var disposeBag: DisposeBag {get}
}
```
主要处理RxSwift的内存泄漏问题。

注意到这个协议做了扩展：
```Swift
extension ProtocolBindable where Self: NetAccessableControllerType {
    /// 网络请求相关的视图绑定
    func bindRequestViews() {
        let viewModel = self.viewModel
        if let self = self as? TextHUDPresentable & UIViewController {
            viewModel.textHUDConfig.drive(self.textHUD.rx.textConfig).disposed(by: disposeBag)
        }
        if let self = self as? ErrorViewPresentable & UIViewController & ProtocolBindable {
            viewModel.errorViewConfig.drive(self.errorView.rx.config).disposed(by: disposeBag)
        }
        if let self = self as? IndicatorPresentable & UIViewController {
            viewModel.request.mapAsVoid().bind(to: self.indicator.rx.hide).disposed(by: disposeBag)
        }
    }
}
```
这里对协议进行了约束
> 这个协议扩展有一个约束条件，即只有实现了NetAccessableControllerType协议的类（class）才可以使用它。

需要继续看下NetAccessableControllerType协议：
```Swift
protocol NetAccessableControllerType: BaseControllerType, NetAccessable {
    /// 协议中使用泛型的解决方案
    associatedtype V: BaseViewMoelType
    var viewModel: V { get }
}
```
这里定义了ViewModel了。
这个协议又继承了子协议，需要继续看：
```Swift
protocol BaseControllerType: ProtocolBindable, HasDelegate where Delegate == PushableControllerDelegate, Self: UIViewController { }
```
> 这段代码定义了一个名为BaseControllerType的协议，它包含了以下几个特性：
继承了ProtocolBindable协议：即实现了这个协议的类必须符合ProtocolBindable协议的要求。
继承了HasDelegate协议：即实现了这个协议的类必须符合HasDelegate协议的要求，其中Delegate必须是PushableControllerDelegate类型。
继承了UIViewController类：即实现了这个协议的类必须是UIViewController或其子类。
通过这个协议的定义，我们可以得出以下理解：
实现了BaseControllerType协议的类，必须是继承自UIViewController的类，并且同时实现了ProtocolBindable协议和HasDelegate协议，其中HasDelegate协议要求这个类的Delegate属性的类型必须是PushableControllerDelegate类型。
因此，这个协议可以用于约束那些需要同时具有ProtocolBindable和HasDelegate特性，并且必须是UIViewController或其子类的类。通常情况下，实现了BaseControllerType协议的类是视图控制器（ViewController），因为它们通常需要具有与导航和推送相关的特性。

HasDelegate是RxCocoa三方库中的协议。
需要看下PushableControllerDelegate协议：
```Swift
protocol PushableControllerDelegate: class {
    
    func pushUserControllerWith(userId: Int64, userName: String, navigation: UINavigationController?)
    
    func pushItemsControllerWith(type: ItemsType, item: Item?, navigation: UINavigationController?)
    
    func pushItemControllerWith(type: ItemType, item: Item?, navigation: UINavigationController?)
    
    func pushImageControllerWith(type: BlobFileType, navigation: UINavigationController?)
    
    func pushHtmlControllerWith(type: BlobFileType, navigation: UINavigationController?)
    
    func pushLoginControllerWith(navigation: UINavigationController?)
    
    func pushProjectsSearchControllerWith(navigation: UINavigationController?)
    
    func pushSettingControlelrWith(navigation: UINavigationController?)
    
    func pushMineControllerWith(navigation: UINavigationController?)
    
    func pushIssueCreateControllerWith(projectID: Int64?, navigation: UINavigationController?)
}
```
这个应该是跳转逻辑。

另外那个网络协议为：
```Swift
protocol NetAccessable {
    func request()
}
```
应该是发起网络。

然后是关注下ProtocalBindable

需要看下这里：
```Swift
protocol NetAccessableControllerType: BaseControllerType, NetAccessable {
    /// 协议中使用泛型的解决方案
    associatedtype V: BaseViewMoelType
    var viewModel: V { get }
}
```
定义了ViewModel。
这个ViewModel需要继承BaseViewModelType。
```Swift
protocol BaseViewMoelType: class {
    
    associatedtype E
    
    var errorViewConfig: Driver<(Bool, (String?, UIImage?))> { get }
    
    var textHUDConfig: Driver<(Bool, String?)> { get }
    
    var navigationTitle: Driver<String?> { get }
    
    /// request是指控制器在window显示后默认产生的网络请求数据流
    var request: Observable<(E)> {get set}
    
    ///actionRequest是指与UI交互产生的次要网络请求数据流
    func actionRequestWith<T>(api: ActionAPI) -> Observable<T>?
}
```
这里定义ViewModel基础行为。可以驱动显示异常View，标题名称，请求，交互行为，这里交互行为用到了一个ActionAPI，需要看下：
```Swift
enum ActionAPI {
    case starProject(String)
    case unstarProject(String)
    case watchProject(String)
    case unwatchProject(String)
    case createIssue(proID: Int64, title: String, description: String)
    case login(email: String?, password: String?)
    case none
}
```
基础行为，这里可以start或者watch一个project。
另外BaseViewMoelType扩展了下，其实就是默认实现了：
```Swift

/// 协议默认实现
extension BaseViewMoelType {
    
    func actionRequestWith<T>(api: ActionAPI) -> Observable<T>? { return nil }
    
    var errorViewConfig: Driver<(Bool, (String?, UIImage?))> {
        return request.asErrorViewConfig()
    }
    var textHUDConfig: Driver<(Bool, String?)> {
        return request.asTextHUDConfig()
    }
    func setActionRequestAsMain(_ actionApi: ActionAPI) {
        self.request = actionRequestWith(api: actionApi)!
    }
    
    func createActionRequest<T>(with api: ActionAPI, creation: @escaping (AnyObserver<T>)->() ) -> Observable<T>? {
        switch api {
        case .none: return Observable.error(RequestError.requestFailed(nil))
        default: return Observable<T>.create({ (observer) -> Disposable in
            creation(observer)
            return Disposables.create()
        }).share(replay: 1)
    }
        
    }
}
```




### 3.2 ProtocolPageViewPresentable协议

另外一个协议：
```Swift
protocol ProtocolPageViewPresentable {
    var titles: [String] {get}
    var pageViewManager: DNSPageViewManager { get }
}
```
主要是配置入参，顶部标题列表，和一个控制器。

这个DNSPageViewManager，其实用到了一个三方库，这里只是把它copy到本地了。
> [https://github.com/Danie1s/DNSPageView](https://github.com/Danie1s/DNSPageView) 现在还有人维护呢。

另外它扩展了：
```Swift

/// 它通过 where 子句指定了扩展的约束条件，即 PageViewPresentable 协议同时也是 Bindable 和 UIViewController 的子协议。
/// 意思就是在 UIViewController+Bindable 的类中才可使用这个扩展方法
extension ProtocolPageViewPresentable where Self: ProtocolBindable & UIViewController {
    
    func setupNavigationTitleStyle() {
        let titleView = pageViewManager.titleView
        titleView.frame = CGRect.init(x: 0, y: 0, width: .screenWidth, height: 84)
        view.addSubview(titleView)
        view.addSubview(pageViewManager.contentView)
        
        navigationItem.titleView = titleView
        
        pageViewManager.contentView.snp.makeConstraints {[weak self] (maker) in
            maker.edgesEqualTo(view: self?.view, with: self)
        }
    }
    
    func setupCustomStyle() {
        let titleView = pageViewManager.titleView
        titleView.frame = CGRect.init(x: 0, y: 0, width: .screenWidth, height: 44)
        view.addSubview(titleView)
        view.addSubview(pageViewManager.contentView)
        
        NightModeViewModel.shared.pageViewStyle.bind(to: pageViewManager.rx.titleStyle).disposed(by: disposeBag)
        
        pageViewManager.contentView.snp.makeConstraints { (maker) in
            maker.top.equalToSuperview().offset(40)
            if #available(iOS 11, *) {
                maker.leading.equalTo(view.safeAreaLayoutGuide.snp.leading)
                maker.trailing.equalTo(view.safeAreaLayoutGuide.snp.trailing)
                maker.bottom.equalTo(view.safeAreaLayoutGuide.snp.bottom)
            }
            else {
                maker.leading.trailing.equalToSuperview()
                maker.bottom.equalTo(bottomLayoutGuide.snp.top)
            }
        }
    }
}
```
这里setupNavigationTitleStyle 方法非常关键，将pageViewManger的contentView加进去了。
这里才可正常显示。
调用地方再MyPageViewController中的viewDidLoad()。
```
override func viewDidLoad() {
    super.viewDidLoad()
    setupNavigationTitleStyle()
}
```

### 3.3 构造函数
变量声明：
```Swift
    var disposeBag: DisposeBag = .init()
    
    let titles: [String]
    
    let pageViewManager: DNSPageViewManager
    
```

构造函数：
```Swift
init(controllers: [UIViewController], titles: [String]) {
    pageViewManager = .init(style: .navigationTitle, titles: titles, childViewControllers: controllers)
    self.titles = titles
    super.init(nibName: nil, bundle: nil)
    for vc in controllers {
        self.addChild(vc)
    }
}
```
这里传3个Tab的UIViewController，和标题名称。

### 3.4 第一次加载
```Swift
override func viewDidLoad() {
    super.viewDidLoad()
    setupNavigationTitleStyle()
}
```
这里是生命周期，将视图添加进去了，使用方法为 ProtocalPageViewPresetable中的协议扩展方法，就是上面3.2所示。

## 4 单个列表控制器ProjectsController

### 4.1 入口
在Coordinator中：
```Swift
/// Coordinator负责Controller之间的调度和为Controller提供ViewModel对象
final class Coordinator {
    
    /// 我们需要在当前类中访问和修改这个属性的值，而在类外部只允许访问这个属性的值而不能修改它
    private(set) var mRootController: MyTabBarController?
    
    init() {
        // 项目tab
        let projrctsController = MyPageViewController(
            controllers: [
                getItemsControllerWith(type: .featuredProjs),
                getItemsControllerWith(type: .popularProjs),
                getItemsControllerWith(type: .latestProjs)],
```
这里其实推荐，热门，最近更新的这三个tab全是这个通用的ItemsController。

```Swift
private func getItemsControllerWith(type: ItemsType, item: Item? = nil) -> UIViewController {
    switch type {
    case .featuredProjs, .popularProjs, .latestProjs, .userProjs(_), .languagedProjs(_), .staredProjs(_), .watchedProjs(_):
        return ProjectsController(viewModel: .init(store: .init(type: type)), delegate: self)
```
这里return 一个ProjectsController()。

### 4.2 继承关系和全局变量
```Swift
class ProjectsController: UIViewController, ItemsController {
```
这里继承了UIViewController。
同时也实现了ItemsController协议。

```Swift
protocol ItemsController: NetAccessableControllerType, TableViewPresentable, HintViewsPresentable where V: ItemsViewModelType {
    
    var isRefreshable: Bool { get }
    
    var isPageable: Bool { get }
    
}
```
这里定义了三个约束条件。
> 这段代码是一个协议（protocol）的声明，其中定义了三个约束条件，分别是：
NetAccessableControllerType: 表示实现这个协议的类型必须能够访问网络，即实现了相关的网络请求方法和属性。
TableViewPresentable: 表示实现这个协议的类型必须能够在界面上呈现一个表格（UITableView），即实现了相关的UITableViewDataSource和UITableViewDelegate协议的方法。
HintViewsPresentable: 表示实现这个协议的类型必须能够在界面上呈现提示视图（hint views），即实现了相关的提示视图显示和隐藏的方法和属性。
在这个协议声明中还使用了一个泛型（generic）类型参数V，它是一个符合ItemsViewModelType协议的类型。这个协议的实现者需要提供一个实现了ItemsViewModelType协议的属性viewModel，以便在协议中使用。
因此，这个协议的意义是：实现它的类型需要满足上述三个约束条件，同时提供一个符合ItemsViewModelType协议的viewModel属性，以便协议中可以调用它所提供的方法和属性。

Item的控制器的协议需要实现1：
```Swift
protocol NetAccessableControllerType: BaseControllerType, NetAccessable {
    /// 协议中使用泛型的解决方案
    associatedtype V: BaseViewMoelType
    var viewModel: V { get }
}
```
这个应该是用ViewModel的。

Item的控制器的协议需要实现2：
```Swift
protocol TableViewPresentable: NightModeChangable, UITableViewDelegate {
    var tableView: UITableView { get }
    var cellInfo: [(String, RegisteredViewType)] { get }
    /// 需要在调用协议代理之前初始化
    var tableViewDelegate: RxTableViewSectionedReloadDelegate { get }
    func cellReuseIdentifier(for indexPath: IndexPath) -> String
    func setupCell(_ cell: UITableViewCell, with indexPath: IndexPath)
}
```
这里应该是用来展示TableView的。并且内部继承夜间模式和UiTableViewDelegate。

Item的控制器的协议需要实现3：
```Swift
typealias HintViewsPresentable = TextHUDPresentable & IndicatorPresentable & ErrorViewPresentable
```
这里是一个显示视图的协议。

看下TextHUDPresentable：
```Swift
protocol TextHUDPresentable: class { }
```
虽然是空协议，但是下面扩展了：
```Swift
private var associatedKey = "TextHUD"

//MARK:- TextHUDPresentable
protocol TextHUDPresentable: class { }


extension TextHUDPresentable where Self: UIViewController {
    var textHUD: MBProgressHUD {
        get {
            guard let res = objc_getAssociatedObject(self, &associatedKey) as? MBProgressHUD else {
                fatalError("It should be set before used")
            }
            return res
        }
        
        set {
            objc_setAssociatedObject(self, &associatedKey, newValue, .OBJC_ASSOCIATION_RETAIN_NONATOMIC)
        }
    }
    
    func setupTextHUD() {
        textHUD = MBProgressHUD()
        textHUD.label.adjustsFontSizeToFitWidth = true
        textHUD.mode = .text
        view.addSubview(textHUD)
    }
}
```

还有一个是协议：
```Swift
protocol IndicatorPresentable: DNSPageEventHandleable { }
```
这个应该是DNSPage事件相关的。
```Swift
/// DNSPageView的事件回调，如果有需要，请让对应的childViewController遵守这个协议
 public protocol DNSPageEventHandleable: class {
    
    /// 重复点击pageTitleView后调用
    func titleViewDidSelectSameTitle()
    
    /// pageContentView的上一页消失的时候，上一页对应的controller调用
    func contentViewDidDisappear()
    
    /// pageContentView滚动停止的时候，当前页对应的controller调用
    func contentViewDidEndScroll()
    
    func contentViewDidFirstLoad()
    
    func contentViewWillAppear()
}
```
这个定义在三方库里面了。

最后一个是：
```Swift
protocol ErrorViewPresentable: class {}
```

应该是用来展示异常视图的：
```Swift
extension ErrorViewPresentable where Self: UIViewController & ProtocolBindable {
    
    var errorView: ErrorView {
        set {
            
            objc_setAssociatedObject(self, &associatedKey, newValue, .OBJC_ASSOCIATION_RETAIN_NONATOMIC)
        }
        
        get {
            return castOrFatalError(objc_getAssociatedObject(self, &associatedKey))
        }
        
    }
    
    func setupErrorView() {
        errorView = ErrorView()
        view.addSubview(errorView)
        errorView.rx.refreshing.startWith(()).bind(to: errorView.rx.hide).disposed(by: disposeBag)
        errorView.snp.makeConstraints { (maker) in
            maker.edges.equalToSuperview()
        }
        NightModeViewModel.shared.normalBackgroud.bind(to: errorView.rx.backgroundColor).disposed(by: disposeBag)
    }
}
```

回到ItemsController。
这个V应该是NetAccessableControllerType 这里的V。

### 4.3 ViewModel中的泛型

```Swift
protocol ItemsController: NetAccessableControllerType, TableViewPresentable, HintViewsPresentable where V: ItemsViewModelType {
    
    var isRefreshable: Bool { get }
    
    var isPageable: Bool { get }
    
}
```
这里用到了一个V。

```Swift
protocol NetAccessableControllerType: BaseControllerType, NetAccessable {
    /// 协议中使用泛型的解决方案
    associatedtype V: BaseViewMoelType
    var viewModel: V { get }
}
```
V指向了这里。

上面V必须是ItemsViewModelType这种类型哦：
```Swift
protocol ItemsViewModelType: NormalViewModelType where S: ItemsStoreType {
    var initItems: BehaviorRelay<[S.O]> { get }
    var page: BehaviorRelay<Int> { get }
    var loadMore: Observable<()> { get  set }
    ///数据源
    var dataSource: Driver<[S.O]> { get }
    
    var contents: Driver<[Section]> { get }
    
    var cellHeight: Driver<[[CGFloat]]> { get }

}
```

哎，这里又得去继承NormalViewModelType了：
```Swift
/// 遵循MVVM设计模式的ViewModel
protocol NormalViewModelType: BaseViewMoelType where E == () {
    
    associatedtype S: StoreType
    
    var store: S { get }

    /// 用来保证ViewModel和其接受的Notification一一对应
    var uuid: UUID { get }
    
    /// 网络请求的参数
    var requestParam: [String: Any]? { get set }
    
    var isLocalDataExistd: Driver<Bool> { get }
    
    func reset(requestParam: [String: Any]?)
    
}
```
这里是一个MVVM模式封装的一个协议。

这里又得去看BaseViewModelType了：
```Swift
protocol BaseViewMoelType: class {
    
    associatedtype E
    
    var errorViewConfig: Driver<(Bool, (String?, UIImage?))> { get }
    
    var textHUDConfig: Driver<(Bool, String?)> { get }
    
    var navigationTitle: Driver<String?> { get }
    
    /// request是指控制器在window显示后默认产生的网络请求数据流
    var request: Observable<(E)> {get set}
    
    ///actionRequest是指与UI交互产生的次要网络请求数据流
    func actionRequestWith<T>(api: ActionAPI) -> Observable<T>?
}
```


可以看到，这里定义了Driver对象，封装请求方法。

回到上面的NormalViewModelType，
需要看下里面的一个S: StoreTYpe类型：
```Swift
protocol StoreType: class {
    associatedtype O: Item
    associatedtype T: TargetType
    var targetType: T { get }
    func request(via param: [String: Any]?, uuid: UUID, observer: AnyObserver<()>)
    
}
```
这里又定义了两种泛型，O和T， T应该是目标类，O指定item。

需要看下O到底是什么：
```Swift
typealias Item = Object & Mappable & RenderableObject
```
> Item 类型可以被视为是一个遵循了 Object、Mappable 和 RenderableObject 这三个协议的对象类型。

T是啥？
```Swift
protocol TargetType {
    //net
    var baseURL: URL { get }
    var path: String { get }
    var method: HTTPMethod { get }
    var parameters: [String: Any]? { get }
    
    //title for the controller
    var title: String? { get }
    var noDataDescription: String { get }
}
```
这里应该是请求类型协议封装。
这个Store应该是拿数据的，要么从数据库拿，要么从网络拿。

再回到上面：
```Swift
protocol ItemsController: NetAccessableControllerType, TableViewPresentable, HintViewsPresentable where V: ItemsViewModelType {
    
    var isRefreshable: Bool { get }
    
    var isPageable: Bool { get }
    
}
```
这里V来自NetAccessableControllerType已经分析了。

注意到：
```Swift
protocol ItemsViewModelType: NormalViewModelType where S: ItemsStoreType {
    var initItems: BehaviorRelay<[S.O]> { get }
    var page: BehaviorRelay<Int> { get }
    var loadMore: Observable<()> { get  set }
    ///数据源
    var dataSource: Driver<[S.O]> { get }
    
    var contents: Driver<[Section]> { get }
    
    var cellHeight: Driver<[[CGFloat]]> { get }

}
```
最右边还有一个S: ItemsToreType，
S来自哪里呢？
```Swift
/// 遵循MVVM设计模式的ViewModel
protocol NormalViewModelType: BaseViewMoelType where E == () {
    
    associatedtype S: StoreType
```
这里。

上面的
```Swift
 var initItems: BehaviorRelay<[S.O]> { get }
```
怎么理解？
```Swift
protocol StoreType: class {
    associatedtype O: Item
    associatedtype T: TargetType
    var targetType: T { get }
    func request(via param: [String: Any]?, uuid: UUID, observer: AnyObserver<()>)
    
}
```
应该是用的Item哦。
```Swift
typealias Item = Object & Mappable & RenderableObject
```
这样就串联起来了。

### 4.4 关系图
太多协议了，需要一个图来理解下。
<img src=mayun_202.png>


