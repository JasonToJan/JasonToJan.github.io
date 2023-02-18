---
title: iOS swift 码云客户端 完整项目分析之二
date: 2023-02-18 10:07:26
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


## 5 网络请求全过程

### 5.1 在ProjectsController的viewDidLoad生命周期
```Swift
override func viewDidLoad() {
        super.viewDidLoad()
        setupSubViews() { [weak self] indexPath in
            guard let project = self?.viewModel.itemsValue[indexPath.row] else { return }
            self?.delegate?.pushItemControllerWith(type: ItemType.projsDetails(project.id), item: project, navigation: self?.navigationController)
        }
    }
```
这里调用了setupSubViews，初始化配置tableView了。
而且在回调中，声明了点击item事件。这里通过自定义代理，跳转到了某个Item控制器里面了。

这个setupSubViews方法是定义在ItemsController中的，这个就是上方第4点讲述的一个配置了很多协议的主要用来展示列表的一个通用控制器协议。

### 5.2 通过ItemsController执行setupSubViews
```Swift
func setupSubViews(via requestParam: [String: Any]? = nil, itemSelected: @escaping (IndexPath)->() ) {
        
    if requestParam != nil {
        viewModel.requestParam = requestParam
    }
    
    if isRefreshable { setupRefreshHeader() }
    
    //viewConfig
    setupTableView()
    setupIndicatorWith(backgroundStyle: .normal)
    setupErrorView()
    setupTextHUD()
    request()
    
    /*--------------------binding-----------------------*/
    //标题
    viewModel.navigationTitle.drive(navigationItem.rx.title).disposed(by: disposeBag)
    //载入菊花是否显示
    viewModel.isLocalDataExistd.drive(indicator.rx.isHidden).disposed(by: disposeBag)
    //数据源
    viewModel.contents.drive(tableView.rx.items(dataSource: dataSource)).disposed(by: disposeBag)
    //cell高度
    viewModel.cellHeight.drive(tableView.rx.cellHeights(delegate: tableViewDelegate)).disposed(by: disposeBag)
    //cell点击
    tableView.rx.itemSelected.bind { [weak self] index in
        guard let self = self else { return }
        self.tableView.deselectRow(at: index, animated: true)
        itemSelected(index)
    }.disposed(by: disposeBag)
    
    //是否添加footer
    viewModel.dataSource.filter { $0.count == 20 }.asObservable().bind { [weak self] (_) in
        guard let self = self, self.isPageable else { return }
        self.setupRefreshFooter()
    }.disposed(by: disposeBag)
}
```
这里配置了很多关于Cell的属性，最关键的是一个 request方法，和 一个setupRefreshHeader方法。

首先看这里：
```Swift
if isRefreshable { setupRefreshHeader() }       
```
对应：
```Swift
private func setupRefreshHeader() {
    tableView.mj_header = MJRefreshNormalHeader()
    /*通过直接订阅刷新事件来保证每次网络请求都能重新订阅Observable*/
    //header
    tableView.mj_header?.rx.refreshing.bind { [weak self] in
        guard let s = self else { return }
        //print(RxSwift.Resources.total)
        s.bindRequestViews()
        s.viewModel.request.bind(to: s.tableView.mj_header!.rx.endRefreshing).disposed(by: s.disposeBag)
        }.disposed(by: disposeBag)
}
```
这里判断它能否支持刷新，支持的话，在header的refreshing状态下，会触发viewModel走request方法。
s就是自己，self的意思，通过自己的viewModel触发request。

什么时候触发这个header走refreshing，不出意外应该是上面的request()方法：
```Swift
extension ItemsController {
    
    func request() {
        if isRefreshable {
            tableView.mj_header?.beginRefreshing()
            return
        }
        bindRequestViews()
    }
```
果然，这里直接利用header走beginRefreshing方法，触发上面的bind，再触发viewModel走request。

### 5.3 viewModel走request
首先我们要明白这个viewModel是哪个具体的viewModel。
这个itemsController中定义的viewModel来自于这个NetAccessableControllerType协议：
```Swift
protocol BaseControllerType: ProtocolBindable, HasDelegate where Delegate == PushableControllerDelegate, Self: UIViewController { }

protocol NetAccessableControllerType: BaseControllerType, NetAccessable {
    /// 协议中使用泛型的解决方案
    associatedtype V: BaseViewMoelType
    var viewModel: V { get }
}
```

这个ViewModel一定是BaseViewModelType类型的。
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
这个协议定义的request返回的是Observable对象，这个是RxSwift中的类。

回到具体的ProjectsController中，看到了如何定义ViewModel的：
```Swift
class ProjectsController: UIViewController, ItemsController {
    
    lazy var tableViewDelegate: RxTableViewSectionedReloadDelegate = .init(tableView: self.tableView)
    
    let isRefreshable: Bool = true
    
    let isPageable: Bool = true
    
    let tableView: UITableView = .init()
    
    let disposeBag: DisposeBag = DisposeBag()
    
    let cellInfo: [(String, RegisteredViewType)] = [("ProjectCell", .xib)]
    
    let viewModel: ItemsViewModel<ProjectsStore>
```
原来是ItemsViewModel，包了一个Store的实现类：ProjectsStore，这个ProjectsStore应该是用来本地网络缓存的，就是拿到网络数据后，磁盘缓存到本地。具体发起网络请求，应该是ItemsViewModel这个实现类。

首先看下这个ItemsViewModel的实现类中的request属性如何定义吧：
```Swift

final class ItemsViewModel<S: ItemsStoreType>: ItemsViewModelType {
    
    let initItems: BehaviorRelay<[S.O]>
    
    let uuid = UUID()
    
    lazy var requestParam: [String : Any]? = { return self.defaultParam }()
    
    let store: S
    
    lazy var page = BehaviorRelay(value: 2)
    
    let data = BehaviorSubject<[[Item]]>.init(value: [])
    
    /*---------observable---------------------*/

   //使用存储属性保证序列的唯一性
    lazy var request: Observable<()> = {return self.requestObservable() }()
```
这里搞了个闭包，调用了自身的requestObservable方法：
```Swift
///获取网络请求的序列
func requestObservable() -> Observable<()> {
    let paramaters = requestParam == nil ? defaultParam : requestParam
    return request(via: paramaters).share(replay: 1)
}
```
这里是拿到了请求参数，继续走request方法：
```Swift
func request(via param: [String: Any]?) -> Observable<()> {
    return Observable<()>.create { [weak self] (observer) in
        guard let self = self else {
            observer.onCompleted()
            return Disposables.create()
        }
        //model持有网络
        self.store.request(via: param, uuid: self.uuid, observer: observer)
        return Disposables.create()
    }
}
```
最终是走到了上面这个方法，参数为请求接口的参数。

最终将请求委托给自身的store来请求，看来这个store还是有用的，不仅仅用来缓存，网络请求也是它。

### 5.4 ItemsStoreType走request
上面因为ProjectController的viewModel是这样定义的：
```Swift
let viewModel: ItemsViewModel<ProjectsStore>
```

而这个ItemsViewModel中是这样要求Store的：
```Swift
final class ItemsViewModel<S: ItemsStoreType>: ItemsViewModelType {
```
所以这个Store应该是需要继承ItemsStoreType的。

如果request方法没有被ProjectsStore重写，那么request方法一定会走ItemsStoreType的request里面。

现在看下ItemsStoreType的request方法吧：
```Swift
//MARK:- 网络请求
func request(via param: [String: Any]?, uuid: UUID, observer: AnyObserver<()>) {
    HttpsManager.request(with: targetType, parameters: param).responseArray(completionHandler:
        ResponseHandler.handlerArrayResponse(via: observer, target: self.targetType, success: { [weak self] (newItems: [O]) in
            self?.handle(newItems: newItems, via: param, uuid: uuid)
            //发送完成事件避免资源一直被占用
            observer.onCompleted()
    }))
}
```
看了下ProjectsStore确实没有重写，而且这个ItemsStoreType自己扩展了这个request方法。

这里通过调用HttpsManager去发起请求了，请求参数通过request参数传进来了。
请求结果通过ResponseHandler工具类解析了。

然后通过自身handle方法，调用store的saveItem保存到本地，这个应该是交给store的实现类具体实现。
将json格式的数据转成了ItemsStoreType定义的items类型。

items类型就是前面声明的泛型O：
```Swift
protocol ItemsStoreType: StoreType where T == ItemsType {
    var items: [O] { get set }
    var targetType: ItemsType { get }
    func saveItems(via userInfo: [ResponseKey: Any])
    init(type: ItemsType)
}
```
这个泛型O,同样也是在ProjectsStore中进行具体实现：
```Swift
final class ProjectsStore: ItemsStoreType {
    
    typealias O = Project
```
这样，说明ProjectsStore中转换的json的目标实体就是Project实体。

必须要声明typealias哦：
```Swift
protocol StoreType: class {
    associatedtype O: Item
    associatedtype T: TargetType
    var targetType: T { get }
    func request(via param: [String: Any]?, uuid: UUID, observer: AnyObserver<()>)   
}
```
因为ProjectsStore继承ItemsStoreType集成StoreType，
这样ProjectsStore就必须实现associatedtype类型了。
> 是的，如果一个协议中声明了关联类型（associatedtype），
那么在实现该协议的类型中必须要声明一个类型别名（typealias）来具体化该关联类型。
这是因为协议中的关联类型是一个占位符，它表示该协议的某些方法或属性的返回值类型或参数类型可以是任何类型，具体的类型要由实现该协议的类型来指定。


这里json转换，主要利用了一个Objectmapper来帮忙转换：
```Swift

struct ResponseHandler {
    
    static func handlerArrayResponse<T: Mappable>(via observer: AnyObserver<()>, target: TargetType, success: @escaping ([T]) -> ()) -> (AFDataResponse<[T]>) -> Void {
        return { dataResponse in
            if let error = dataResponse.error {
                observer.onError(RequestError.requestFailed(error.localizedDescription))
                return
            }
            if let objects = dataResponse.value {
                objects.count == 0 ? observer.onError(RequestError.noData(target.noDataDescription)) : success(objects)
            } else {observer.onError(RequestError.noData(String.Local.noPermission))}
        }
    }
```
上面就是将一个数组，转换成一个目标T的数组：
ObjectMapper：github地址: [https://github.com/tristanhimmelman/ObjectMapper](https://github.com/tristanhimmelman/ObjectMapper)
Alamofire: github地址：[https://github.com/Alamofire/Alamofire](https://github.com/Alamofire/Alamofire)

如何使用：
要使用ObjectMapper将Alamofire获取的JSON数据转换为目标实体，您可以按照以下步骤进行操作：

1).确保您的项目已经集成了ObjectMapper和Alamofire库，并且您已经在文件顶部添加了所需的import语句。

2).定义您的目标实体，并使其符合Mappable协议。例如，如果您有一个名为Person的目标实体，则可以按照以下方式定义它：

```Swift
import ObjectMapper

class Person: Mappable {
    var name: String?
    var age: Int?
    
    required init?(map: Map) {}
    
    func mapping(map: Map) {
        name <- map["name"]
        age <- map["age"]
    }
}
```
在上面的代码中，我们定义了一个名为Person的类，并使其符合Mappable协议。该类包含name和age两个属性，它们将从JSON中映射到Person实例的相应属性中。

3).在您的Alamofire请求中，将返回的JSON数据转换为目标实体。您可以按照以下方式进行操作：
```Swift
import Alamofire
import ObjectMapper

Alamofire.request("https://example.com/person").responseJSON { response in
    guard let json = response.result.value as? [String: Any] else {
        return
    }
    if let person = Mapper<Person>().map(JSON: json) {
        // 将person实例用于您的应用程序逻辑
    }
}
```
在上面的代码中，我们使用Alamofire发出请求，并在响应中检查是否存在JSON数据。如果有数据，我们使用ObjectMapper将其映射到Person实例中。如果映射成功，我们可以使用person实例进行应用程序逻辑。

这就是如何使用ObjectMapper将Alamofire获取的JSON数据转换为目标实体的步骤。请注意，这只是一个简单的示例，您可能需要根据您的应用程序逻辑进行适当的更改。

### 5.5 自定义封装的HttpsManager
```Swift

fileprivate let key_urlUpdateDic = "URLUpdateDictionary"

struct HttpsManager {
    /// 不自动产生缓存的SessionManager
    private static let defaultSession: Alamofire.Session = {
        let configuration = URLSessionConfiguration.ephemeral
        //更该其protocolClasses 才能拦截URLSession的网络请求
        configuration.protocolClasses = [HttpsURLProtocol.self]
        return Alamofire.Session(configuration: configuration)
    }()
    //https头
    private static var headers: Alamofire.HTTPHeaders {
        return Alamofire.HTTPHeaders(["User-Agent": userAgent])
    }
    
    private static var userAgent: String {
        let appVersion = Bundle.main.object(forInfoDictionaryKey: "CFBundleShortVersionString") as? String ?? ""
        let IDFV = UIDevice.current.identifierForVendor?.uuidString ?? ""
        return String.init(format: "git.OSChina.NET/git_%@/%@/%@/%@/%@", appVersion, UIDevice.current.systemName, UIDevice.current.systemVersion, UIDevice.current.model, IDFV)
    }
    
    static var urlUpdateDic: [String?: Date]? = {
        return UserDefaults.standard.dictionary(forKey: key_urlUpdateDic) as? [String : Date] ?? nil
    }()
    
    static func saveUpdateDic() {
        UserDefaults.standard.set(urlUpdateDic, forKey: key_urlUpdateDic)
    }
    
    static func request(with type: TargetType, parameters: [String: Any]? = nil) -> DataRequest {
        let param = parameters == nil ? type.parameters : parameters
        return HttpsManager.defaultSession.request(type.url, method: type.method, parameters: param)
    }
    
}
```

这里我们倒推一下：
外部触发了request方法：
```Swift
static func request(with type: TargetType, parameters: [String: Any]? = nil) -> DataRequest {
    let param = parameters == nil ? type.parameters : parameters
    return HttpsManager.defaultSession.request(type.url, method: type.method, parameters: param)
}
```
这里走了这个管理方法中的默认session发起请求。

这个session是这样定义的：
```Swift
/// 不自动产生缓存的SessionManager
private static let defaultSession: Alamofire.Session = {
    let configuration = URLSessionConfiguration.ephemeral
    //更该其protocolClasses 才能拦截URLSession的网络请求
    configuration.protocolClasses = [HttpsURLProtocol.self]
    return Alamofire.Session(configuration: configuration)
}()
```
是通过Alamofire类里面定义的一个Session。
通过自己配置URLSession，传给Alamofire中，就拿到一个不自动产生缓存的SessionManager对象了。

这个session走到了Alamofire中去请求了，下面就不涉及我们此篇文章的内容了。
这里通过Alamofire，我们得到一个DataRequest返回体。
```Swift
// MARK: - Subclasses

// MARK: - DataRequest

/// `Request` subclass which handles in-memory `Data` download using `URLSessionDataTask`.
public class DataRequest: Request {
    /// `URLRequestConvertible` value used to create `URLRequest`s for this instance.
    public let convertible: URLRequestConvertible
    /// `Data` read from the server so far.
    public var data: Data? { return protectedData.directValue }
```
可以看到，数据放在了data中。

然后我们利用实体类去继承Mapper，他们就能成功转成自己要的实体了。

## 6 磁盘缓存全过程

### 6.1 网络请求拿到数据了
在ItemsStoreType的request方法中，拿到数据后调用了一个handle方法：
```Swift
//MARK:- 网络请求
func request(via param: [String: Any]?, uuid: UUID, observer: AnyObserver<()>) {
    HttpsManager.request(with: targetType, parameters: param).responseArray(completionHandler:
        ResponseHandler.handlerArrayResponse(via: observer, target: self.targetType, success: { [weak self] (newItems: [O]) in
            self?.handle(newItems: newItems, via: param, uuid: uuid)
            //发送完成事件避免资源一直被占用
            observer.onCompleted()
    }))
}
```

这里继续走：
```Swift
func handle(newItems: [O], via param: [String: Any]?, uuid: UUID, sendNoti: Bool = true) {
    var userInfo: [ResponseKey: Any] = [.tragetType: self.targetType, .identity: uuid]
    //若参数存在page
    if let page = param?["page"] as? Int {
        var reason: ChangeReason
        if page == 1 {
            self.items = newItems
            reason = .request
        } else {
            self.items.append(contentsOf: newItems)
            reason = .loadMore
        }
        userInfo[.changeReason] = reason
    }
    else {
        userInfo[.changeReason] = ChangeReason.request
        self.items = newItems
    }
    //交由具体的StoreClasses实现持久化存储
    self.saveItems(via: userInfo)
    if sendNoti {
        NotificationCenter.default.post(name: Self.itemsChangedNoti, object: self
            .items, userInfo: userInfo)
    }
}
```
这里委托store的saveItems保存了。

### 5.2 saveItems
这里是交给了具体的Store去实现了：
这里是ProjectsStore的saveItems方法：
```Swift
func saveItems(via userInfo: [ResponseKey : Any]) {
    //只存储第一页的数据
    if let changedReason = userInfo[ResponseKey.changeReason] as? ChangeReason, changedReason == .request, !(realm?.isInWriteTransaction ?? false) {
        switch self.targetType {
        case .userProjs(_): saveNormalItems()
        case .staredProjs(_), .watchedProjs(_): saveOwnerdItems()
        case .latestProjs, .popularProjs, .featuredProjs, .languagedProjs(_, _):
            saveOrderedItems()
        default: break
        }
    }
}
```
这里根据类型，继续走到了saveOrderedItems：
```Swift
///定制的存储方法
private func saveOrderedItems() {
    //本地没有数据存在
    if queryAllItems()?.count == 0 {
        var index = 0
        for item in items {
            custom(item: item, index: index)
            index += 1
        }
        update(items: items)
    }
    //本地存在数据
    else {
        var index = items.count - 1
        var diff  = 0
        var turn  = true
        var newItems = [Project]()
        //所有请求到的数据都是新数据（本地不存在）
        //取本地第一条数据的index-新数据的count作为差值
        if object(of: items[items.count - 1].id) == nil, let firstIndex = queryAllItems()?.first?.index(of: targetType) {
            diff = firstIndex - items.count
            turn = false
        }
        for _ in items {
            let item = items[index]
            //已经存在拥有序列的该类型数据(忽略，并计算新的index差)
            if turn, let obj = object(of: item.id), let storedIndex = obj.index(of: targetType) {
                diff = storedIndex - index
            } else {
                //不存在就新定制
                custom(item: item, index: index + diff)
                newItems.append(item)
            }
            index -= 1
        }
        update(items: newItems)
    }
}
```

这里先看下queryAllItems吧。

### 5.3 ItemsStoreType的queryAllItems
这里走到了ItemsStoreType方法中：
```Swift
func queryAllItems(limited: Int = 20) -> [O]? {
    switch targetType {
    case .featuredProjs:
        return self.query(via: NSPredicate(format: "featuredIndex != nil"), sortedKey: "featuredIndex").toArray(limited: limited)
    case .popularProjs:
        return self.query(via: NSPredicate(format: "popularIndex != nil"), sortedKey: "popularIndex").toArray(limited: limited)
    case .latestProjs:
        return self.query(via: NSPredicate(format: "latestIndex != nil"), sortedKey: "latestIndex").toArray(limited: limited)
    case .languagedProjs(_, let language):
        return self.query(via: NSPredicate(format: "languageIndex != nil && language = %@", language), sortedKey: "languageIndex").toArray(limited: limited)
    case .userEvents(let id):
        return self.query(via: NSPredicate(format: "authorId = %ld", id), sortedKey: "createdDate", ascending: false).toArray(limited: limited)
    case .selfEvents:
        return self.query(via: NSPredicate(format: "authorId = %ld", CurrentUserManager.id), sortedKey: "createdDate", ascending: false).toArray(limited: limited)
    case .userProjs(let id):
        return self.query(via: NSPredicate(format: "owner.id = %ld", id), sortedKey: "createdAt", ascending: true).toArray(limited: limited)
    case .staredProjs(let id):
        return UserStore(type: .user(id)).item?.startedProjects.toArray(limited: 20) as? [Self.O]
    case .watchedProjs(let id):
        return UserStore(type: .user(id)).item?.watchedProjects.toArray(limited: 20) as? [Self.O]
    default: return nil
    }
}
```

这里通过NSPredicate数据库语法，走query方法：

### 5.4 StoreType的query方法
因为这个走到了底层，所以应该在底层封装：
```Swift
func query(via predicate: NSPredicate, sortedKey: String? = nil, ascending: Bool = true) -> Results<O> {
    if let key = sortedKey {
        return realm.objects(O.self).filter(predicate).sorted(byKeyPath: key, ascending: ascending)
    }
    
    return realm.objects(O.self).filter(predicate)
}
```
这里终于找到罪魁祸首了，就是realm数据库。

再看下update:
```Swift
func update(items: [O]) {
        if realm.isInWriteTransaction { return }
        do {
            //save
            try realm.write {
                realm.add(items, update: .modified)
                //realm.add(items, update: true)
            }
            
        } catch(_) { return }
        
    }
```

realm来自于下方这里，当然可以在自己的实现Store中自定义，这里是默认实现。

```Swift
extension StoreType {
var realm: Realm {
    return Store.shared.realm
}
```
应该是在Store的单例类中找到了realm。

### 5.5 Store底层
```Swift
final class Store {
    static let shared = Store()
    let realm: Realm
    private init() {
        Store.set(realmName: "gitosc")
        realm = try! Realm()
    }
    
    static func deleteRealm(with realm: Realm = Store.shared.realm) {
        if realm.isInWriteTransaction { return }
        autoreleasepool {
            do {
                try realm.write {
                    
                    realm.deleteAll()
                }
            } catch(_) { return }
        }
        let realmURL = Realm.Configuration.defaultConfiguration.fileURL!
        let realmURLs: [URL] = [
            realmURL.appendingPathExtension("lock"),
            realmURL.appendingPathExtension("note"),
            realmURL.appendingPathExtension("management")
        ]
        for URL in realmURLs {
            do {
                try FileManager.default.removeItem(at: URL)
            } catch {
                // 错误处理
            }
        }
    }
    
    private static func set(realmName: String) {
        
        var config = Realm.Configuration()
        // 使用默认的目录，但是请将文件名替换为用户名
        config.fileURL = config.fileURL!.deletingLastPathComponent().appendingPathComponent("\(realmName).realm")
    
        // 将该配置设置为默认 Realm 配置
        Realm.Configuration.defaultConfiguration = config
    }
}
```
这里搞了一个单例类，存放了realm类，可以操作数据库。

其实在我们自己实现的ProjectsStore也声明了realm:
```Swift
final class ProjectsStore: ItemsStoreType {
    
    typealias O = Project
    
    var items: [Project] = []
    
    var targetType: ItemsType
    
    var realm: Realm? = Store.shared.realm
```
其实不声明也可以，反正都是用的同一个单例类。

就这样，使用realm把数据转换成功数据库了，持久化在本地了。

## 6 总结

* 本篇文章主要是窥探码云这个客户端网络请求和缓存的主流程，揭秘如何封装，一层一层环环相扣，将数据完美展现在UItableView里面。

* 首先我们如果是一个上方多tab，每个tab有一个列表，这种架构，可以用一个DNSPageView来构造，只需要在这里添加tab的几个控制器，titles，其它工作很少。

* 然后封装下子控制器，通过让这个子控制器继承UIViewController，和一个关键的ItemsController协议，这个协议我们自己定义的，主要用来请求网络，填充数据的。

* 这个子控制器里面，主要做一个tableView相关的绑定工作，这里我们会在子控制器里面添加一个tableView。主要在viewDidLoad里面配置。通过view.addSubView一个table来实现。

* 然后在第一次执行下下拉刷新事件，这个事件通过rxSwift绑定关系，来触发viewModel层走request。

* 因为要viewModel，所以我们需要继续封装一下ItemsController，将网络相关的全部给它处理，viewModel也放在它里面。

* 这里网络相关我们单独抽一个协议NetAccessableControllerType，这个叫网络可达，这里存放ViewModel，我们可以定义ViewModel基本协议方法，用一个BaseViewModelType来规范吧。

* 然后还有一个TableViewPresentable就主要用来填充数据吧，还有一个HintViewsPresentable也处理UI方面的东西。

* 然后的的ViewModel去继承ItemsViewModelType，上面定义的是BaseViewModelType是这个类型，这里需要声明继承关系。

* 这个ItemsViewModelType，其实就配置了很多Driver事件驱动，在setupView的初始化配置中，利用viewModel.contents来驱动tableView.rx.item数据填充。这个数据源我们在TableVIewPresentable中配置。

* 在网络请求成功后，给了ItemsStoreType中的items数据为网络数据，然后其实是给了ViewModel中的Store，就是说store有数据了，然后监听了dataSource其实是S.o，也就是items，这样就相当于监听了网络数据。

* 拿到数据后，驱动ItemsViewModel中的dataSource，走dataSource方法，在setupViewModel中监听了数据源，实现tableView赋值过程。








