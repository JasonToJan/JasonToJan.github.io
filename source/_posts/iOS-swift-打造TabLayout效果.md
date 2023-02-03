---
title: iOS swift 打造TabLayout效果
date: 2023-02-02 16:15:27
op: false
cover: false
toc: true
mathjax: true
tags:
- TabLayout
categories:
- iOS
---

## 1 需求定义

在首页，我们需要实现一个多tab页，支持左右滑动，上滑吸顶的效果。
比如这样：
<img src=page1.gif width=50%>

需要保证良好的用户体验效果，每个页面都是独立的，可以上滑吸顶，左右滑动页面切换，而且还需要支持未读数字提示。

## 2 需求分析

针对这些需求，github上面有一个成熟的框架。
[JXsegementedView](https://github.com/pujiaxin33/JXSegmentedView)，github上面有2千多个stars，想必是深受大家喜欢，所以我们也可以考虑用起来。

> 它的特点是：
腾讯新闻、今日头条、QQ音乐、网易云音乐、京东、爱奇艺、腾讯视频、淘宝、天猫、简书、微博等
所有主流APP分类切换滚动视图
与其他的同类三方库对比的优点：
指示器逻辑面向协议编程，可以为所欲为的扩展指示器效果；
提供更加全面丰富效果，几乎支持所有主流APP效果；
使用子类化管理cell样式，逻辑更清晰，扩展更简单；
列表支持整个生命周期方法； 

另外一个与之协同的是 [JXPagingView](https://github.com/pujiaxin33/JXPagingView) ，注意这两个库不是替换的关系，而是互相协作的关系。
> JXPagingView: 类似微博主页、简书主页、QQ联系人页面等效果。多页面嵌套，既可以上下滑动，也可以左右滑动切换页面。支持HeaderView悬浮、支持下拉刷新、上拉加载更多。



## 3 使用步骤

### 3.1 引用依赖

这里其实可以远程依赖，但我们考虑到自身一些需求，直接拖进项目里面，将github上的source文件，拷贝到项目里面。

<img src=segment1.png>

注意这个Personal文件夹里面主要放的是JXPagingView相关代码。

ok，这样项目中可以自由使用了。

### 3.2 初始化视图相关

```Swift
extension JXPagingListContainerView: JXSegmentedViewListContainer {}
```
这里我们先扩展下这个协议，这个JXSegementedViewListContainer是框架里面的协议：
```Swift
public protocol JXSegmentedViewListContainer {
    var defaultSelectedIndex: Int { set get }
    func contentScrollView() -> UIScrollView
    func reloadData()
    func didClickSelectedItem(at index: Int)
}
```
协议可以理解为Android中的接口。

然后我们定义了几个tab，枚举类，姑且叫做这个吧：
```Swift
enum WorkListCellItem {
    /// 待收讫
    case waitingBalance
    /// 待开单
    case waitingForBilling
    /// 待送货
    case waitingForDelivery
    /// 待安装
    case waitingForInstallation
    /// 待出库
    case outOfWarehouse
    /// 待入库
    case toBeWarehoused
}
```

然后在控制器里面声明下全局变量：
```Swift
 var pagingView: JXPagingView!
var userHeaderView: WorkHeadView!
var segmentedViewDataSource: JXSegmentedTitleDataSource!
var segmentedView: JXSegmentedView!
var JXheightForHeaderInSection: Int = 44
var JXTableHeaderViewHeight: Int = 182{
    didSet{
        self.pagingView.reloadData()
    }
}
```
这里简单说下这几个成员作用：
* JXPagingView: 这个是最外层的容器，里面有一个mainTableView和listContainerView。
* WorkHeadView: 这个是我们自定义的头部，无需关注。
* JXSegemntedTitleDataSource: 这个主要是tab的数据。
* JXheightForHeaderInSection: 标签栏高度定义。
* JXTableHeaderViewHeight: 头部区域高度定义。

然后看下如何初始化视图：
```Swift
func setUI() {

        view.backgroundColor = UIColor(hex:"#3E4D89").withAlphaComponent(0.06)
        view.gm_addSubview(navView) { make in
            make.top.equalTo(0)
            make.left.right.equalToSuperview()
            make.height.equalTo(Height_NavBarAndStatusBar)
        }
        
        /// 头部区域
        userHeaderView = WorkHeadView(frame: CGRect(x: 0, y: 0, width: UIScreen.main.bounds.size.width, height: CGFloat(JXTableHeaderViewHeight)))
        userHeaderView.eventClosure = { [unowned self] (event) in
            self.workFunctionItemEventClick(event: event)
        }
        userHeaderView.clickMsgHandle = {[weak self] model in
            self?.requestForClickMsg(model: model)
        }
        // 配置指示器
        let indicator = JXSegmentedIndicatorLineView()
        // 标签栏下划线
        indicator.indicatorColor = UIColor(r: 64, g: 158, b: 255, alpha: 1)
        indicator.indicatorWidthIncrement = -15 /// 作用于增大可触区域
        /// 标签栏
        segmentedView = JXSegmentedView(frame: CGRect(x: 0, y: 0, width: UIScreen.main.bounds.size.width, height: CGFloat(JXheightForHeaderInSection)))
        segmentedView.backgroundColor = UIColor.white
        segmentedView.dataSource = dataSource
        segmentedView.isContentScrollViewClickTransitionAnimationEnabled = true
        segmentedView.indicators = [indicator]

        let lineWidth = 1/UIScreen.main.scale
        let lineLayer = CALayer()
        lineLayer.backgroundColor = UIColor.lightGray.cgColor
        lineLayer.frame = CGRect(x: 0, y: segmentedView.bounds.height - lineWidth, width: segmentedView.bounds.width, height: lineWidth)
        segmentedView.layer.addSublayer(lineLayer)

        pagingView = JXPagingView(delegate: self)
        let header = GMSAutoRefreshHeader()
        header.setRefreshingTarget(self, refreshingAction: #selector(defaultTableHeaderRefresh))
        pagingView.mainTableView.mj_header = header
        if #available(iOS 15.0, *) {
            pagingView.mainTableView.sectionHeaderTopPadding = 0
        } else {
            // Fallback on earlier ···versions
        }
        
        self.view.addSubview(pagingView)

        segmentedView.listContainer = pagingView.listContainerView
        
        self.view.gm_addSubview(stockInitializationTipsView) { make in
            make.top.equalTo(Height_NavBarAndStatusBar)
            make.width.equalTo(ScreenWidth)
            make.height.equalTo(0)
        }
        pagingView.snp_makeConstraints { make in
            make.top.equalTo(stockInitializationTipsView.snp_bottom).offset(0)
            make.width.equalTo(ScreenWidth)
            make.height.equalTo(ScreenHeight-Height_NavBarAndStatusBar-Height_TabBar)
        }
        
        /// 待开单列表--取消订单闭包回调
        self.waitingForBillingVC.cancelBlock = { [weak self] in
            self?.waitingForBillingVC.requestIntendedOrder()
        }
        
        self.setJXCountBolck()
        
        WatermarkView.showWindow()
    }
```
这里JXSegmentView主要是标签栏。

PageView是我们实现这效果的父布局。
不出意外，这个PageView里面应该会关联到标签栏。
这里有部分代码是这样的：
`
segmentedView.listContainer = pagingView.listContainerView
`
这里应该就是建立联系了。

### 3.3 配置代理和数据源

前面将delegate设置为self了，所以我们需要看下如何配置的。
先看下PageView设置的代理吧：
```swfit
extension NewWorkViewController: JXPagingViewDelegate {

    /// tableHeaderView高度
    func tableHeaderViewHeight(in pagingView: JXPagingView) -> Int {
        return JXTableHeaderViewHeight
    }

    /// tableHeaderView
    func tableHeaderView(in pagingView: JXPagingView) -> UIView {
        return userHeaderView
    }

    /// 吸顶标签栏的高度
    func heightForPinSectionHeader(in pagingView: JXPagingView) -> Int {
        return JXheightForHeaderInSection
    }

    /// 吸顶标签栏
    func viewForPinSectionHeader(in pagingView: JXPagingView) -> UIView {
        return segmentedView
    }
    
    /// 列表的数量
    func numberOfLists(in pagingView: JXPagingView) -> Int {
        return dataSource.attributedTitles.count
    }

    /// 列表
    func pagingView(_ pagingView: JXPagingView, initListAtIndex index: Int) -> JXPagingViewListViewDelegate {
        let flag: WorkListCellItem = self.listDataArray[index]
        
        if flag == .waitingBalance {
            /// 待收讫VC
            let list = waitingBalanceVC
            return list
        }else if flag == .waitingForBilling {
            /// 待开单VC
            let list = waitingForBillingVC
            return list
        }else if flag == .waitingForDelivery {
            /// 待送货VC
            let list = waitingForDeliveryVC
            return list
        }else if flag == .waitingForInstallation {
            /// 待安装VC
            let list = waitingForInstallationVC
            return list
        }else if flag == .outOfWarehouse {
            /// 待出库VC
            let list = outOfWarehouseVC
            return list
        }else {
            /// 待入库VC
            let list = toBeWarehousedVC
            return list
        }
    }

    func mainTableViewDidScroll(_ scrollView: UIScrollView) {
//        userHeaderView?.scrollViewDidScroll(contentOffsetY: scrollView.contentOffset.y)
    }
}
```
这里配置了头部，高度等。
主要的pagingView是需要我们自己实现的，这个就是Tab下方的列表展示了。
不出意外这个应该需要继承框架里面的某个View吧。

我们等下看下waitingForBillingVC吧。

另外看下标签栏的数据源是怎么设置的：
```Swift
 private lazy var dataSource : JXSegmentedTitleAttributeDataSource = {
        let dataSource = JXSegmentedTitleAttributeDataSource()
        /// 关闭标签栏等分宽度
        dataSource.isItemSpacingAverageEnabled = false
        dataSource.itemSpacing = 0
        return dataSource
    }()
```
这里只是定义了，还需要在需要的地方配置下数据：
```Swift
  dataSource.attributedTitles = dataTitles
  dataSource.selectedAttributedTitles = dataSelectedTitles
        
```

## 4 子列表

外层框架搭好了，需要看下每个子列表是如何塞进去的。
```Swift
// MARK: - 工作台（待开单）
class WorkWaitingForBillingVC: AutomaticDimensionTableViewController, JXPagingViewListViewDelegate {
```

这里其实本质上是一个控制器哦，只是实现了它定义的代理方法即可。
这里用了一个我们自己定义的自动高度的控制器，跟这个框架无关哈。
```Swift
/// 自动高度 TableView （避免 使用UITableView.automaticDimension 带来的弊端（某些时候 reloadData contentOffSet 会改变 导致闪跳））
/// 只实现了UITableViewDelegate和高度相关的代理方法，tableview也没有创建，自行按需创建
class AutomaticDimensionTableViewController: GMBaseViewController {
    private var rowHeights = [String: CGFloat]()
    private var headerHeights = [String: CGFloat]()
    private var footerHeights = [String: CGFloat]()
    
    /// 预估cell高度
    var rowEstimatedHeight: CGFloat = UITableView.automaticDimension
    /// 预估header高度
    var headerEstimatedHeight: CGFloat = 200
    /// 预估footer高度
    var footerEstimatedHeight: CGFloat = 200

}
private extension AutomaticDimensionTableViewController {
    func getRowKeyWithIndexPath(_ indexPath: IndexPath) -> String {
        let section = indexPath.section
        let row = indexPath.row
        let key = "\(section)-\(row)"
        return key
    }
}
extension AutomaticDimensionTableViewController: UITableViewDelegate {
    func tableView(_ tableView: UITableView, willDisplay cell: UITableViewCell, forRowAt indexPath: IndexPath) {
        let key = getRowKeyWithIndexPath(indexPath)
        let height = cell.frame.height
        rowHeights[key] = height
    }
    func tableView(_ tableView: UITableView, estimatedHeightForRowAt indexPath: IndexPath) -> CGFloat {
        let key = getRowKeyWithIndexPath(indexPath)
        guard let height = rowHeights[key] else { return rowEstimatedHeight }
        return height
    }
    func tableView(_ tableView: UITableView, heightForRowAt indexPath: IndexPath) -> CGFloat {
        return UITableView.automaticDimension
    }
    
    
    
    func tableView(_ tableView: UITableView, willDisplayHeaderView view: UIView, forSection section: Int) {
        let key = "\(section)"
        let height = view.frame.height
        headerHeights[key] = height
    }
    func tableView(_ tableView: UITableView, estimatedHeightForHeaderInSection section: Int) -> CGFloat {
        let key = "\(section)"
        guard let height = headerHeights[key] else { return headerEstimatedHeight }
        return height
    }
    func tableView(_ tableView: UITableView, heightForHeaderInSection section: Int) -> CGFloat {
        return UITableView.automaticDimension
    }
    
    
    func tableView(_ tableView: UITableView, willDisplayFooterView view: UIView, forSection section: Int) {
        let key = "\(section)"
        let height = view.frame.height
        footerHeights[key] = height
    }
    func tableView(_ tableView: UITableView, estimatedHeightForFooterInSection section: Int) -> CGFloat {
        let key = "\(section)"
        guard let height = footerHeights[key] else { return footerEstimatedHeight }
        return height
    }
    func tableView(_ tableView: UITableView, heightForFooterInSection section: Int) -> CGFloat {
        return UITableView.automaticDimension
    }
}
```

框架要求的协议需要看下：
```Swift
@objc
public protocol JXPagingViewListViewDelegate {
    /// 如果列表是VC，就返回VC.view
    /// 如果列表是View，就返回View自己
    ///
    /// - Returns: 返回列表视图
    func listView() -> UIView
    /// 返回listView内部持有的UIScrollView或UITableView或UICollectionView
    /// 主要用于mainTableView已经显示了header，listView的contentOffset需要重置时，内部需要访问到外部传入进来的listView内的scrollView
    ///
    /// - Returns: listView内部持有的UIScrollView或UITableView或UICollectionView
    func listScrollView() -> UIScrollView
    /// 当listView内部持有的UIScrollView或UITableView或UICollectionView的代理方法`scrollViewDidScroll`回调时，需要调用该代理方法传入的callback
    ///
    /// - Parameter callback: `scrollViewDidScroll`回调时调用的callback
    func listViewDidScrollCallback(callback: @escaping (UIScrollView)->())

    /// 将要重置listScrollView的contentOffset
    @objc optional func listScrollViewWillResetContentOffset()
    /// 可选实现，列表将要显示的时候调用
    @objc optional func listWillAppear()
    /// 可选实现，列表显示的时候调用
    @objc optional func listDidAppear()
    /// 可选实现，列表将要消失的时候调用
    @objc optional func listWillDisappear()
    /// 可选实现，列表消失的时候调用
    @objc optional func listDidDisappear()
}
```
这里要求我们可以实现这些方法。

```Swift
var listViewDidScrollCallback: ((UIScrollView) -> ())?
    
    deinit {
        listViewDidScrollCallback = nil
    }
    
    func listView() -> UIView {
        return self.view
    }
    
    func listScrollView() -> UIScrollView {
        return self.mainTableView
    }
    
    func listViewDidScrollCallback(callback: @escaping (UIScrollView) -> ()) {
        self.listViewDidScrollCallback = callback
    }

    /// 主列表
    lazy var mainTableView: UITableView = {
        let tableView = UITableView(frame: UIScreen.main.bounds, style: .grouped)
        if #available(iOS 15.0, *) {
            tableView.sectionHeaderTopPadding = 0
        } else {
            // Fallback on earlier ···versions
        }
        tableView.separatorStyle = .none
        tableView.delegate = self
        tableView.dataSource = self
        tableView.backgroundColor = .white
        tableView.showsVerticalScrollIndicator = false
        tableView.showsHorizontalScrollIndicator = false
        tableView.tableFooterView = UIView()
        
        let footer = GMSAutoRefreshFooter()
        footer.setRefreshingTarget(self, refreshingAction: #selector(defaultTableFooterRefresh))
        tableView.mj_footer = footer
       return tableView
   }()
```
这里我们直接把控制的view给他了。
这个滚动的视图就是我们自己定义的主列表。

大致就是这样，使用起来还是简单的。
后面有时间再详细看下源码才行。

## 5 总结
* 本文主要是针对多Tab页，吸顶效果和可分页可滑动的需求，利用一个三方框架完美解决。
* 主要通过最外层的JPagingView本质上是UIView来的，设置代理，配置头部区域和中间列表区域显示。通过一个listContainer关联起了标签栏和列表区，这样实现双向交互。
* 使用这个框架的工作就是创建这几个View，配置代理，实现代理，最后实现列表的控制器，控制器也需要实现它提供的代理，另外头部也需要我们按照ui要求自己编写即可。
* 使用任何一个三方库一定要懂得它的原理，一旦出问题，我们可以游刃而解。

