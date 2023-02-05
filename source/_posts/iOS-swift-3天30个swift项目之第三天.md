---
title: iOS swift 3天30个swift项目之第三天
date: 2023-02-05 09:32:39
op: false
cover: false
toc: true
mathjax: true
tags:
- 30个Swift项目
categories:
- iOS
---

## 1 滑动删除

### 1.1 效果
<img src=Swipeable%20Cell.gif>

## 1.2 列表实现
```Swift
class ViewController: UITableViewController {
```

数据定义：
```Swift
var data = [
    pattern(image: "1", name: "Pattern Building"),
    pattern(image: "2", name: "Joe Beez"),
    pattern(image: "3", name: "Car It's car"),
    pattern(image: "4", name: "Floral Kaleidoscopic"),
    pattern(image: "5", name: "Sprinkle Pattern"),
    pattern(image: "6", name: "Palitos de queso"),
    pattern(image: "7", name: "Ready to Go? Pattern"),
    pattern(image: "8", name: "Sets Seamless"),
]
```

数据绑定：
```Swift
override func tableView(_ tableView: UITableView, numberOfRowsInSection section: Int) -> Int {
        return data.count
    }
    
override func numberOfSections(in tableView: UITableView) -> Int {
    return 4
}

override func tableView(_ tableView: UITableView, heightForRowAt indexPath: IndexPath) -> CGFloat {
    return 60
}

override func tableView(_ tableView: UITableView, cellForRowAt indexPath: IndexPath) -> UITableViewCell {
    let cell = tableView.dequeueReusableCell(withIdentifier: "PatternCell", for: indexPath) as! PatternCell
    let pattern = data[indexPath.row]
    
    cell.patternImageView.image = UIImage(named: pattern.image)
    cell.patternNameLabel.text = pattern.name
    return cell
}

// 横滑 增加3个UITableViewRowAction 
override func tableView(_ tableView: UITableView, editActionsForRowAt indexPath: IndexPath) -> [UITableViewRowAction]? {
    let delete = UITableViewRowAction(style: .normal, title: "🗑\nDelete") { action, index in
        print("Delete button tapped")
    }
    delete.backgroundColor = UIColor.gray
    
    // 分析点击事件
    let share = UITableViewRowAction(style: .normal, title: "🤗\nShare") { (action, indexPath) in
        let firstActivityItem = self.data[indexPath.row]
        let activityViewController = UIActivityViewController(activityItems: [firstActivityItem.image as NSString], applicationActivities: nil)
        
        self.present(activityViewController, animated: true, completion: nil)
    }
    share.backgroundColor = UIColor.red
    
    let download = UITableViewRowAction(style: .normal, title: "⬇️\nDownload") { action, index in
        print("Download button tapped")
    }
    download.backgroundColor = UIColor.blue
    
    return [download, share, delete]
}

override func tableView(_ tableView: UITableView, canEditRowAt indexPath: IndexPath) -> Bool {
    return true
}

override func tableView(_ tableView: UITableView, commit editingStyle: UITableViewCellEditingStyle, forRowAt indexPath: IndexPath) {
    switch editingStyle {
    case .delete:
        print("Delete")
    case .insert:
        print("Insert")
    case .none:
        print("None")
    }
}
```

cell是自定义的，但主要是通过故事版拖动的，代码绑定里面没有做其它事情。
其实这个横滑效果，是系统支持的，我们只需要多实现一些协议方法即可。

## 2 3D触摸菜单

### 2.1 效果
<img src=3DTouchQuickAction.gif>

### 2.2 页面定义
首页很简单，这里是在故事版里面绑定的控制器。
就中间一个Label。

然后还有2个页面，都是一张图片，在故事版里面定义了，没有在代码里面声明。

### 2.3 配置3D菜单
先在info.plist中配置菜单，这里先不实现具体跳转：
<img src=22.png>

### 2.4 AppDelegate配置
```Swift
enum ShortcutIdentifier: String {
    
    case First
    case Second
    case Third
    
    init?(fullType: String) {
        guard let last = fullType.components(separatedBy: ".").last else {
            return nil
        }
        self.init(rawValue: last)
    }
    
    var type: String {
        return Bundle.main.bundleIdentifier! + ".\(self.rawValue)"
    }
}
```

这里先定义一个枚举，注意到这里的type为Bundle.main.bundleIdentifier，这里应该必须要为这个了。

然后这里配置application协议：
```Swift
func application(_ application: UIApplication, performActionFor shortcutItem: UIApplicationShortcutItem, completionHandler: @escaping (Bool) -> Void) {
    let handledShortCutItem = handleShortCutItem(shortcutItem)
    // 走回调
    completionHandler(handledShortCutItem)
}

func handleShortCutItem(_ shortcutItem: UIApplicationShortcutItem) -> Bool {
        
    var handled = false
    
    guard let _ = ShortcutIdentifier(fullType: shortcutItem.type) else {
        return false
    }
    
    guard let shortCutType = shortcutItem.type as String? else {
        return false
    }
    
    let storyboard = UIStoryboard(name: "Main", bundle: nil)
    var vc: UIViewController
    
    switch (shortCutType) {
    case ShortcutIdentifier.First.type:
        // Handle shortcut 1
        vc = storyboard.instantiateViewController(withIdentifier: "RunVC") as! RunViewController
        handled = true
    case ShortcutIdentifier.Second.type:
        // Handle shortcut 2
        vc = storyboard.instantiateViewController(withIdentifier: "ScanVC") as! ScanViewController
        handled = true
    case ShortcutIdentifier.Third.type:
        // Handle shortcut 3
        vc = storyboard.instantiateViewController(withIdentifier: "WiFiVC") as! SwitchWiFiViewController
        handled = true
    default:
        vc = UIViewController()
        break
    }
    
    // Display the selected view controller
    //
    var presentedVC: UIViewController = window!.rootViewController!
    while presentedVC.presentedViewController != nil {
        presentedVC = presentedVC.presentedViewController!
    }
    if !presentedVC.isMember(of: vc.classForCoder) {
        presentedVC.present(vc, animated: true, completion: nil)
    }
    
    return handled
}
```
这里定义了快捷键方式跳转方式，这里通过获取到Main的故事版，然后，故事版去instantiateViewController来获取其它的控制器，这里再通过present方法跳转到目标控制器。

## 3 侧滑菜单

### 3.1 效果
<img src=SlideOutMenu.gif>

### 3.2 侧滑支持
首先需要引入SWRevealViewController.h和SWRevealViewController.m文件。
这个用官方的即可，文件比较长，就不贴进来了。

### 3.3 故事版定义
<img src=23_1.png>
这里定义好了跳转逻辑，直接在故事版里面操作的。

其它2个item也是如此。

主要是在Main.storyboard中定义了第一个控制为：SWRevealViewController
这个是oc写的。

故事版里面定义了这个控制器会指向一个BackTableVC。


### 3.4 控制器配置
第一个菜单控制器为：
```Swift
class Channel : UIViewController {
    
    override func viewDidLoad() {
        self.navigationController?.isNavigationBarHidden = true
        self.view.addGestureRecognizer(self.revealViewController().panGestureRecognizer())
    }
    
}
```

第二个菜单控制器为：
```Swift
class ReadLater : UIViewController {
    
    override func viewDidLoad() {
        self.navigationController?.isNavigationBarHidden = true
        self.view.addGestureRecognizer(self.revealViewController().panGestureRecognizer())
    }
    
}
```

第三个菜单控制为：
```Swift
class FriendRead : UIViewController {
    
    override func viewDidLoad() {
        self.navigationController?.isNavigationBarHidden = true
        self.view.addGestureRecognizer(self.revealViewController().panGestureRecognizer())   
    }   
}
```

这里必须配置：
```
 self.view.addGestureRecognizer(self.revealViewController().panGestureRecognizer())   
```
才能实现菜单左滑手势效果哦。

## 4 磁片效果

### 4.1 效果
<img src=MosaicLayouts.gif>

### 4.2 pod引入三方库
这里通过Pod引入依赖，如下图：
<img src=24_1.png>

这里引入了FMMosaicLayout库+AFNetworking+ORStackView+SwiftyJSON
用了这四个库。

### 4.3 页面定义
这里也是用了一个UICollectionView，单一个页面。
直接继承了这个：UICollectionViewController。

```Swift
override func numberOfSections(in collectionView: UICollectionView) -> Int {
        return 10
    }
    
override func collectionView(_ collectionView: UICollectionView, cellForItemAt indexPath: IndexPath) -> UICollectionViewCell {
    let cell = collectionView.dequeueReusableCell(withReuseIdentifier: "Cell", for: indexPath)
    cell.alpha = 0
    
    let imageView = cell.contentView.viewWithTag(2) as! UIImageView
    imageView.image = UIImage(named: imageArray[indexPath.row])
    
    let cellDelay = UInt64((arc4random() % 600 ) / 1000 )
    let cellDelayTime = DispatchTime(uptimeNanoseconds: cellDelay * NSEC_PER_SEC)
    DispatchQueue.main.asyncAfter(deadline: cellDelayTime) {
        UIView.animate(withDuration: 0.8, animations: {
            cell.alpha = 1.0
        })
    }
    
    return cell
}


override func collectionView(_ collectionView: UICollectionView, numberOfItemsInSection section: Int) -> Int {
    return imageArray.count
}
```
主要实现了这几个协议方法。
Cell用了系统的。

内容直接用UIImageView表示。

然后这里异步开启动画效果，alpha从0到1的变化效果。

### 4.4 初始化
```Swift
var imageArray = [String]()

override func viewDidLoad() {
    super.viewDidLoad()
    imageArray = ["1", "2", "3", "4", "5", "6", "7", "8", "9", "10", "11", "12", "13", "14", "15", "16", "17", "18", "19", "20", "21"]
    let mosaicLayout = FMMosaicLayout()
    self.collectionView?.collectionViewLayout = mosaicLayout
    if #available(iOS 11.0, *) {
        self.collectionView?.contentInsetAdjustmentBehavior = .never
    }
}
```
这里初始化的时候，将collectionView的collectionViewLayout设置为三方库的View。

如何设置为不同大小，这里就是三方库的作用了：
```Swift
extension ViewController : FMMosaicLayoutDelegate {

    func collectionView(_ collectionView: UICollectionView!, layout collectionViewLayout: FMMosaicLayout!, numberOfColumnsInSection section: Int) -> Int {
        return 3
    }

    func collectionView(_ collectionView: UICollectionView!, layout collectionViewLayout: FMMosaicLayout!, mosaicCellSizeForItemAt indexPath: IndexPath!) -> FMMosaicCellSize {
        return indexPath.item % 7 == 0 ? .big : .small
    }

    func collectionView(_ collectionView: UICollectionView!, layout collectionViewLayout: FMMosaicLayout!, interitemSpacingForSectionAt section: Int) -> CGFloat {
        return 1.0
    }

    func collectionView(_ collectionView: UICollectionView!, layout collectionViewLayout: FMMosaicLayout!, insetForSectionAt section: Int) -> UIEdgeInsets {
        return UIEdgeInsets(top: 1.0, left: 1.0, bottom: 1.0, right: 1.0)
    }
}
```
这里如果是7的倍数就大图，否则小图。有3种类型。

## 5 基础动画

### 5.1 效果
<img src=BasicAnimation.gif>

### 5.2 故事版
第一个启动页首先是Main.storyboard故事版，然后这个故事版有一个Storyboard Entry Point就是第一个场景了，这里第一个场景是空的NavigationController Scene。

然后这里绑定了这个第一个场景跳转逻辑，底部这里有个：
<img src=25_1.png>

那如何修改这个启动路径呢？
<img src=25_2.png>
这里最右侧属性里面，先清除掉之前的，然后拖动小圆点，指向目标场景即可哦。

这里Navigation Controller场景指向了BasicAnimation场景，这里面是一个TableView。
这里配置的UITableView的代理和数据源都指向了Basic Animation。
<img src=25_3.png>
这里数据直接在故事版里面写好了。

然后item的跳转也是直接在故事版里面拖动的：
<img src=25_4.png>

这里其实对应的首页的Controller没有啥东西：
```Swift
class ViewController: UITableViewController {

    override func viewDidLoad() {
        super.viewDidLoad()
        self.tableView.tableFooterView = UIView(frame: .zero)
    }

    override func didReceiveMemoryWarning() {
        super.didReceiveMemoryWarning()
    }
}
```

### 5.3 Position动画
这里定义了3个方块。
```Swift
class PositionViewController: UIViewController {

    @IBOutlet weak var yellowSquareView: UIView!
    @IBOutlet weak var blueSquareView: UIView!
    @IBOutlet weak var mouseView: UIView!    
```

已经出现动画时执行动画：
```Swift
override func viewDidAppear(_ animated: Bool) {
        super.viewDidAppear(animated)
        
    UIView.animate(withDuration: 0.8, delay: 0.2, usingSpringWithDamping: 0.6, initialSpringVelocity: 0.8, options: .curveEaseInOut, animations: {
        self.yellowSquareView.center.x = self.view.bounds.width - self.yellowSquareView.center.x
        self.yellowSquareView.center.y = self.yellowSquareView.center.y + 30
        self.blueSquareView.center.x = self.view.bounds.width -  self.blueSquareView.center.x
        self.blueSquareView.center.y = self.blueSquareView.center.y + 30

        }, completion: nil )
    
    UIView.animate(withDuration: 0.6, delay: 0.4, usingSpringWithDamping: 0.6, initialSpringVelocity: 0.8, options: .curveEaseOut, animations: {
        self.setHeight(180)
        self.mouseView.center.y = self.view.bounds.height - self.mouseView.center.y
        }, completion: nil )
}

func setHeight(_ height: CGFloat) {
    
    var frame: CGRect = self.mouseView.frame
    frame.size.height = height
    
    self.mouseView.frame = frame
}
```

### 5.4 Opacity动画
```Swift
class OpacityViewController: UIViewController {

    @IBOutlet weak var exampleImageView: UIImageView!
    
    override func viewDidLoad() {
        super.viewDidLoad()

    }

    override func didReceiveMemoryWarning() {
        super.didReceiveMemoryWarning()
    }
    
    override func viewDidAppear(_ animated: Bool) {
        super.viewDidAppear(animated)
        
        UIView.animate(withDuration: 2) {
            self.exampleImageView.alpha = 0
        }
    }
}
```
这里控制图片alpha从1到0的动画。

### 5.5 Scale动画
```Swift
class ScaleViewController: UIViewController {

    @IBOutlet weak var scaleImageView: UIImageView!
    
    override func viewDidLoad() {
        super.viewDidLoad()
        self.scaleImageView.alpha = 0
    }

    override func didReceiveMemoryWarning() {
        super.didReceiveMemoryWarning()
    }
    
    override func viewDidAppear(_ animated: Bool) {
        super.viewDidAppear(animated)
        
        
        UIView.animate(withDuration: 0.8, delay: 0, usingSpringWithDamping: 0.7, initialSpringVelocity: 1, options: .curveEaseIn, animations: {
            self.scaleImageView.transform = CGAffineTransform(scaleX: 2, y: 2)
            self.scaleImageView.alpha = 1
            
            }, completion: nil )
    }
}
```
这里alpha从0到1,缩放动画从1到2。

### 5.6 Color动画
```Swift
class ColorViewController: UIViewController {

    @IBOutlet weak var bgColorView: UIView!
    @IBOutlet weak var numberLabel: UILabel!
    
    override func viewDidLoad() {
        super.viewDidLoad()

    }

    override func didReceiveMemoryWarning() {
        super.didReceiveMemoryWarning()
    }
    
    override func viewDidAppear(_ animated: Bool) {
        super.viewDidAppear(animated)
        
        
        UIView.animate(withDuration: 0.5, delay: 0.2, options: .curveEaseIn, animations: {
            self.bgColorView.backgroundColor = .black
            
            }, completion: nil )
        
        UIView.animate(withDuration: 0.5, delay: 0.8, options: .curveEaseOut, animations: {
            self.numberLabel.textColor = UIColor(red:0.959, green:0.937, blue:0.109, alpha:1)
            
            }, completion: nil)
    }
}
```
这里设置颜色从黄到黑，文字颜色也变化了。

### 5.7 Rotation动画
```Swift 
class RotationViewController: UIViewController {

    @IBOutlet weak var emojiLabel: UILabel!
    @IBOutlet weak var rotationImageView: UIImageView!
    @IBOutlet weak var trump2: UIImageView!
    @IBOutlet weak var trump3: UIImageView!
    @IBOutlet weak var trump4: UIImageView!
    @IBOutlet weak var trump5: UIImageView!
    @IBOutlet weak var trump6: UIImageView!
    @IBOutlet weak var trump7: UIImageView!
    @IBOutlet weak var trump8: UIImageView!
    
    func spin() {
        UIView.animate(withDuration: 0.8, delay: 0, options: .curveLinear, animations: {
            self.rotationImageView.transform = self.rotationImageView.transform.rotated(by: CGFloat(Double.pi))
            self.trump2.transform = self.trump2.transform.rotated(by: CGFloat(Double.pi))
            self.trump3.transform = self.trump3.transform.rotated(by: CGFloat(Double.pi))
            self.trump4.transform = self.trump4.transform.rotated(by: CGFloat(Double.pi))
            self.trump5.transform = self.trump5.transform.rotated(by: CGFloat(Double.pi))
            self.trump6.transform = self.trump6.transform.rotated(by: CGFloat(Double.pi))
            self.trump7.transform = self.trump7.transform.rotated(by: CGFloat(Double.pi))
            self.trump8.transform = self.trump8.transform.rotated(by: CGFloat(Double.pi))
            self.emojiLabel.transform = self.emojiLabel.transform.rotated(by: CGFloat(Double.pi))
            }) { (finished) -> Void in
                self.spin()
        }
    }
    
    override func viewDidAppear(_ animated: Bool) {
        super.viewDidAppear(animated)
        self.spin()
    }
}
```
这里让8个视图都开始旋转了。

## 6 CoreData使用

### 6.1 效果
<img src=CoreData.gif>

### 6.2 AddDelegate配置
```Swift
lazy var applicationDocumentsDirectory: NSURL = {
        // The directory the application uses to store the Core Data store file. This code uses a directory named "me.appkitchen.cd" in the application's documents Application Support directory.
        let urls = FileManager.default.urls(for: .documentDirectory, in: .userDomainMask)
        return urls[urls.count-1] as NSURL
    }()

lazy var managedObjectModel: NSManagedObjectModel = {
    // The managed object model for the application. This property is not optional. It is a fatal error for the application not to be able to find and load its model.
    let modelURL = Bundle.main.url(forResource: "cd", withExtension: "momd")!
    return NSManagedObjectModel(contentsOf: modelURL)!
}()

lazy var persistentStoreCoordinator: NSPersistentStoreCoordinator = {
    // The persistent store coordinator for the application. This implementation creates and returns a coordinator, having added the store for the application to it. This property is optional since there are legitimate error conditions that could cause the creation of the store to fail.
    // Create the coordinator and store
    let coordinator = NSPersistentStoreCoordinator(managedObjectModel: self.managedObjectModel)
    let url = self.applicationDocumentsDirectory.appendingPathComponent("SingleViewCoreData.sqlite")
    var failureReason = "There was an error creating or loading the application's saved data."
    do {
        try coordinator.addPersistentStore(ofType: NSSQLiteStoreType, configurationName: nil, at: url, options: nil)
    } catch {
        // Report any error we got.
        var dict = [String: AnyObject]()
        dict[NSLocalizedDescriptionKey] = "Failed to initialize the application's saved data" as AnyObject
        dict[NSLocalizedFailureReasonErrorKey] = failureReason as AnyObject

        dict[NSUnderlyingErrorKey] = error as NSError
        let wrappedError = NSError(domain: "YOUR_ERROR_DOMAIN", code: 9999, userInfo: dict)
        // Replace this with code to handle the error appropriately.
        // abort() causes the application to generate a crash log and terminate. You should not use this function in a shipping application, although it may be useful during development.
        NSLog("Unresolved error \(wrappedError), \(wrappedError.userInfo)")
        abort()
    }

    return coordinator
}()

lazy var managedObjectContext: NSManagedObjectContext = {
    // Returns the managed object context for the application (which is already bound to the persistent store coordinator for the application.) This property is optional since there are legitimate error conditions that could cause the creation of the context to fail.
    let coordinator = self.persistentStoreCoordinator
    var managedObjectContext = NSManagedObjectContext(concurrencyType: .mainQueueConcurrencyType)
    managedObjectContext.persistentStoreCoordinator = coordinator
    return managedObjectContext
}()

// MARK: - Core Data Saving support

func saveContext () {
    if managedObjectContext.hasChanges {
        do {
            try managedObjectContext.save()
        } catch {
            // Replace this implementation with code to handle the error appropriately.
            // abort() causes the application to generate a crash log and terminate. You should not use this function in a shipping application, although it may be useful during development.
            let nserror = error as NSError
            NSLog("Unresolved error \(nserror), \(nserror.userInfo)")
            abort()
        }
    }
}
```
因为代码比较久了，这个按照XCode指示修复后就是这个效果。
这里可以运行了。

主要是applicationWillTerminate方法中saveContext了。

### 6.3 可见数据操作
这里代码使用还有点问题，暂时无法贴最新代码。
关于CoreData的使用还是建议参考这篇文章：
[了解和采用 CoreData 框架 —— Ficow 陪你学 CoreData](https://blog.ficowshen.com/page/post/52)。

## 7 底部Bar动画

### 7.1 效果
<img src=TapBarAnimation.gif>

### 7.2 故事版添加导航item
<img src=27_1.png>

这里应该是关联到了这3个item。
这样子，就可以展示底部导航栏效果。
然后item也定义了跳转的目标场景。

### 7.3 第一个Tab页
```Swift
class FirstTabViewController: UIViewController, UITableViewDataSource, UITableViewDelegate {

    @IBOutlet weak var articleTableView: UITableView!
    
    var data = [
    
        article(avatarImage: "allen", sharedName: "Allen Wang", actionType: "Read Later", articleTitle: "Giphy Cam Lets You Create And Share Homemade Gifs", articleCoverImage: "giphy", articleSouce: "TheNextWeb", articleTime: "5min  •  13:20"),
        article(avatarImage: "Daniel Hooper", sharedName: "Daniel Hooper", actionType: "Shared on Twitter", articleTitle: "Principle. The Sketch of Prototyping Tools", articleCoverImage: "my workflow flow", articleSouce: "SketchTalk", articleTime: "3min  •  12:57"),
        article(avatarImage: "davidbeckham", sharedName: "David Beckham", actionType: "Shared on Facebook", articleTitle: "Ohlala, An Uber For Escorts, Launches Its ‘Paid Dating’ Service In NYC", articleCoverImage: "Ohlala", articleSouce: "TechCrunch", articleTime: "1min  •  12:59"),
        article(avatarImage: "bruce", sharedName: "Bruce Fan", actionType: "Shared on Weibo", articleTitle: "Lonely Planet’s new mobile app helps you explore major cities like a pro", articleCoverImage: "Lonely Planet", articleSouce: "36Kr", articleTime: "5min  •  11:21"),

    ]
```
上面是数据定义。

然后设置代理和数据源：
```Swift
override func viewDidLoad() {
    super.viewDidLoad()
    
    articleTableView.dataSource = self
    articleTableView.delegate = self
    articleTableView.separatorStyle = UITableViewCell.SeparatorStyle.none
    articleTableView.tableFooterView = UIView(frame: .zero)
}

 
func numberOfSectionsInTableView(tableView: UITableView) -> Int {
    return 10
}

func tableView(_ tableView: UITableView, numberOfRowsInSection section: Int) -> Int {
    return data.count
}

func tableView(_ tableView: UITableView, heightForRowAt indexPath: IndexPath) -> CGFloat {
    return 165
}

func tableView(_ tableView: UITableView, cellForRowAt indexPath: IndexPath) -> UITableViewCell {
    let cell = articleTableView.dequeueReusableCell(withIdentifier: "Cell", for: indexPath) as! ArticleTableViewCell
    let article = data[indexPath.row]
    
    cell.avatarImage.image = UIImage(named: article.avatarImage)
    cell.articleCoverImage.image = UIImage(named: article.articleCoverImage)
    cell.sharedNameLabel.text = article.sharedName
    cell.actionTypeLabel.text = article.actionType
    cell.articleTitleLabel.text = article.articleTitle
    cell.articleSouceLabel.text = article.articleSouce
    cell.articelCreatedAtLabel.text = article.articleTime
    cell.selectionStyle = UITableViewCell.SelectionStyle.none
    
    return cell   
}
```
这里配置了Cell:
```Swift
struct article {
    let avatarImage: String
    let sharedName: String
    let actionType: String
    let articleTitle: String
    let articleCoverImage: String
    let articleSouce: String
    let articleTime: String
}

class ArticleTableViewCell: UITableViewCell {

    @IBOutlet weak var avatarImage: UIImageView!
    @IBOutlet weak var sharedNameLabel: UILabel!
    @IBOutlet weak var actionTypeLabel: UILabel!
    @IBOutlet weak var articleCoverImage: UIImageView!
    
    @IBOutlet weak var articleTitleLabel: UILabel!
    @IBOutlet weak var articleSouceLabel: UILabel!
    @IBOutlet weak var articelCreatedAtLabel: UILabel!
    
    override func awakeFromNib() {
        super.awakeFromNib()
        // Initialization code

    }

    override func setSelected(_ selected: Bool, animated: Bool) {
        super.setSelected(selected, animated: animated)

        // Configure the view for the selected state
    }
}
```
这里应该就是item配置了，具体布局是用故事版里面的配置的。

可见的时候，执行动画效果：
```Swift
override func viewWillAppear(_ animated: Bool) {   
    animateTable()
}

func animateTable() {
    
    self.articleTableView.reloadData()
    
    let cells = articleTableView.visibleCells
    let tableHeight: CGFloat = articleTableView.bounds.size.height
    
    for i in cells {
        let cell: UITableViewCell = i as UITableViewCell
        cell.transform = CGAffineTransform(translationX: 0, y: tableHeight)
    }
    
    var index = 0
    
    for a in cells {
        let cell: UITableViewCell = a as UITableViewCell
        UIView.animate(withDuration: 1.0, delay: 0.05 * Double(index), usingSpringWithDamping: 0.8, initialSpringVelocity: 0, options: [], animations: {
            cell.transform = CGAffineTransform(translationX: 0, y: 0);
            }, completion: nil)
        
        index += 1
    }
}
```
从下往上的动画效果。


### 7.4 第二个Tab的动画
```Swift
override func viewDidAppear(_ animated: Bool) {
    super.viewDidAppear(animated)
    
    UIView.animate(withDuration: 0.5, delay: 0.1, usingSpringWithDamping: 0.7, initialSpringVelocity: 1, options: .curveEaseIn, animations: {
        self.exploreImageView.transform = CGAffineTransform(scaleX: 1, y: 1)
        self.exploreImageView.alpha = 1
        
        }, completion: nil )
}

override func viewDidDisappear(_ animated: Bool) {
    super.viewDidDisappear(animated)
    resetViewTransform()
}

// MARK:
func resetViewTransform() {
    self.exploreImageView.alpha = 0
    self.exploreImageView.transform = CGAffineTransform(scaleX: 0.5, y: 0.5)
}
```

### 7.5 第三个Tab的动画
```Swift
override func viewDidAppear(_ animated: Bool) {
        super.viewDidAppear(animated)
    
    UIView.animate(withDuration: 0.5, delay: 0.1, usingSpringWithDamping: 0.7, initialSpringVelocity: 1, options: .curveEaseIn, animations: {
        self.profileImageView.transform = CGAffineTransform(scaleX: 1, y: 1)
        self.profileImageView.alpha = 1
        
        }, completion: nil )
}

override func viewDidDisappear(_ animated: Bool) {
    super.viewDidDisappear(animated)
    resetViewTransform()
}

// MARK:
func resetViewTransform() {
    self.profileImageView.alpha = 0
    self.profileImageView.transform = CGAffineTransform(scaleX: 0.5, y: 0.5)
}
```

## 8 系统搜索

### 8.1 效果
<img src=Spotlight%20Search.gif>

### 8.2 首页
数据设置：
<img src=28_1.png>
这里新建了一个电影数据，用key-value形式保存了。

可见时处理：
```Swift
override func viewDidLoad() {
    super.viewDidLoad()
    loadMoviesInfo()
    configureTableView()
    navigationItem.title = "Movies"
    setupSearchableContent()
}

// 加载根目录文件数据，拿到数组
func loadMoviesInfo() {
    if let path = Bundle.main.path(forResource: "MoviesData", ofType: "plist") {
        moviesInfo = NSMutableArray(contentsOfFile: path)
    }
}

// 配置代理和数据源
func configureTableView() {
    tblMovies.delegate = self
    tblMovies.dataSource = self
    tblMovies.tableFooterView = UIView(frame: CGRect.zero)
    tblMovies.register(UINib(nibName: "MovieSummaryCell", bundle: nil), forCellReuseIdentifier: "idCellMovieSummary")
}

// 搜索数据装载，使用了系统的类  CSSearchableItem
func setupSearchableContent() {
    var searchableItems = [CSSearchableItem]()
    
    for i in 0...(moviesInfo.count - 1) {
        
        let movie = moviesInfo[i] as! [String: String]
        let searchableItemAttributeSet = CSSearchableItemAttributeSet(itemContentType: kUTTypeText as String)
        
        //set the title
        searchableItemAttributeSet.title = movie["Title"]!
        
        //set the image
        let imagePathParts = movie["Image"]!.components(separatedBy: ".")
        searchableItemAttributeSet.thumbnailURL = Bundle.main.url(forResource: imagePathParts[0], withExtension: imagePathParts[1])
        
        // Set the description.
        searchableItemAttributeSet.contentDescription = movie["Description"]!
        
        var keywords = [String]()
        let movieCategories = movie["Category"]!.components(separatedBy: ", ")
        for movieCategory in movieCategories {
            keywords.append(movieCategory)
        }
        
        let stars = movie["Stars"]!.components(separatedBy: ", ")
        for star in stars {
            keywords.append(star)
        }
        
        searchableItemAttributeSet.keywords = keywords
        
        let searchableItem = CSSearchableItem(uniqueIdentifier: "com.appcoda.SpotIt.\(i)", domainIdentifier: "movies", attributeSet: searchableItemAttributeSet)
        
        searchableItems.append(searchableItem)
        
        CSSearchableIndex.default().indexSearchableItems(searchableItems) {
            if $0 != nil {
                print($0!.localizedDescription)
            }
        }
    }
}
```

监听系统搜索后跳转逻辑：
```Swift
override func restoreUserActivityState(_ activity: NSUserActivity) {
    if activity.activityType == CSSearchableItemActionType {
        if let userInfo = activity.userInfo {
            let selectedMovie = userInfo[CSSearchableItemActivityIdentifier] as! String
            selectedMovieIndex = Int(selectedMovie.components(separatedBy: ".").last!)
            performSegue(withIdentifier: "idSegueShowMovieDetails", sender: self)
        }
    }
}
```

然后是代理和数据源配置：
```Swift
 func numberOfSections(in tableView: UITableView) -> Int {
        return 1
    }
    
func tableView(_ tableView: UITableView, numberOfRowsInSection section: Int) -> Int {
    if moviesInfo != nil {
        return moviesInfo.count
    }
    return 0
}

func tableView(_ tableView: UITableView, cellForRowAt indexPath: IndexPath) -> UITableViewCell {
    let cell = tableView.dequeueReusableCell(withIdentifier: "idCellMovieSummary", for: indexPath) as! MovieSummaryCell
    let currentMovieInfo = moviesInfo[(indexPath as NSIndexPath).row] as! [String: String]
    
    cell.lblTitle.text = currentMovieInfo["Title"]!
    cell.lblDescription.text = currentMovieInfo["Description"]!
    cell.lblRating.text = currentMovieInfo["Rating"]!
    cell.imgMovieImage.image = UIImage(named: currentMovieInfo["Image"]!)
    
    return cell
}

func tableView(_ tableView: UITableView, heightForRowAt indexPath: IndexPath) -> CGFloat {
    return 100.0
}

func tableView(_ tableView: UITableView, didSelectRowAt indexPath: IndexPath) {
    selectedMovieIndex = (indexPath as NSIndexPath).row
    performSegue(withIdentifier: "idSegueShowMovieDetails", sender: self)
}

override func prepare(for segue: UIStoryboardSegue, sender: Any?) {
    if let identifier = segue.identifier, identifier == "idSegueShowMovieDetails" {
        let movieDetailsViewController = segue.destination as! MovieDetailsViewController
        movieDetailsViewController.movieInfo = moviesInfo[selectedMovieIndex] as? [String: String]
    }
}
```
这里didSelectRowAt，通过调用系统的performSegue决定跳转到某个场景。

### 8.3 详情页
```Swift
class MovieDetailsViewController: UIViewController {

    @IBOutlet weak var imgMovieImage: UIImageView!
    @IBOutlet weak var lblTitle: UILabel!
    @IBOutlet weak var lblCategory: UILabel!
    @IBOutlet weak var lblDescription: UILabel!
    @IBOutlet weak var lblDirector: UILabel!
    @IBOutlet weak var lblStars: UILabel!
    @IBOutlet weak var lblRating: UILabel!
    
    var movieInfo: [String: String]!
    
    override func viewDidLoad() {
        super.viewDidLoad()
    }
    
    override func viewWillAppear(_ animated: Bool) {
        super.viewWillAppear(animated)
        
        lblRating.layer.cornerRadius = lblRating.frame.size.width/2
        lblRating.layer.masksToBounds = true
        
        if movieInfo != nil {
            populateMovieInfo()
        }
    }
    
    
    override func didReceiveMemoryWarning() {
        super.didReceiveMemoryWarning()
    }
    
    func populateMovieInfo() {
        
        lblTitle.text = movieInfo["Title"]!
        lblCategory.text = movieInfo["Category"]!
        lblDescription.text = movieInfo["Description"]!
        lblDirector.text = movieInfo["Director"]!
        lblStars.text = movieInfo["Stars"]!
        lblRating.text = movieInfo["Rating"]!
        imgMovieImage.image = UIImage(named: movieInfo["Image"]!)
        
    }
}
```
这里配置了movieInfo，外部设置进来，然后回显进去。

## 9 选择头像

### 9.1 效果
<img src=AvatarPicker.gif>

### 9.2 点击头像
```Swift
@IBAction func pickProfileImage(_ tap: UITapGestureRecognizer) {
    let authorization = PHPhotoLibrary.authorizationStatus()
    
    if authorization == .notDetermined {
        PHPhotoLibrary.requestAuthorization { _ in
            DispatchQueue.main.async {
                self.pickProfileImage(tap)
            }
        }
    }
    
    if authorization == .authorized {
        let controller = ImagePickerSheetController()
        controller.addAction(action: ImageAction(title: NSLocalizedString("Take Photo or Video", comment: "Action Title"), secondaryTitle: NSLocalizedString("Use this one", comment: "Action Title"), handler: { _ in
            self.presentCamera()
        }, secondaryHandler: { (action, numberOfPhotos) in
            controller.getSelectedImagesWithCompletion(completion: { images in
                self.profileImage = images[0]
                self.userProfileImageView.image = self.profileImage
            })
        }))
                    
        controller.addAction(action: ImageAction(title: NSLocalizedString("Cancel", comment: "Action Title"), style: .Cancel, handler: nil, secondaryHandler: nil))
        
        self.present(controller, animated: true, completion: nil)
    }
    
    
}

func presentCamera()
{
    print("拍照")
}
```
这里先判断有无权限，没有权限继续执行，有权限再跳转控制器。

## 10 wiki-Face

### 10.1 效果
<img src=wikiFace.gif>

### 10.2 搜索处理
```Swift
func textFieldShouldReturn(_ textField: UITextField) -> Bool {
        
    textField.resignFirstResponder()
    
    if let textFieldContent = textField.text {
        do {
            try WikiFace.faceForPerson(textFieldContent, size: CGSize(width: 300, height: 400), completion: { (image:UIImage?, imageFound:Bool) -> () in
                if imageFound == true {
                    DispatchQueue.main.async {
                        self.faceImageView.image = image
                        
                        UIView.animate(withDuration: 0.8, delay: 0, usingSpringWithDamping: 0.7, initialSpringVelocity: 1, options: .curveEaseIn, animations: { () -> Void in
                            
                            self.faceImageView.transform = CGAffineTransform(scaleX: 1, y: 1)
                            self.faceImageView.alpha = 1
                            
                            //Fuck! Useless...LOL
                            self.faceImageView.layer.shadowOpacity = 0.4
                            self.faceImageView.layer.shadowOffset = CGSize(width: 3.0, height: 2.0)
                            self.faceImageView.layer.shadowRadius = 15.0
                            self.faceImageView.layer.shadowColor = UIColor.black.cgColor
                            
                        }, completion: nil )
                        
                        WikiFace.centerImageViewOnFace(self.faceImageView)
                    }
                }

            })
        }catch WikiFace.WikiFaceError.CouldNotDownloadImage{
            print("Could not access wikipedia for downloading an image")
        } catch {
            print(error)
        }
    }
    
    return true
    
}
```

使用工具类：
```Swift
import UIKit
import ImageIO

class WikiFace: NSObject {
    
    enum WikiFaceError: Error {
        case CouldNotDownloadImage
    }
    
    class func faceForPerson(_ person: String, size: CGSize, completion:@escaping (_ image: UIImage? ,_ imageFound: Bool) -> ()) throws {
        
        let escapedString = person.addingPercentEncoding(withAllowedCharacters: CharacterSet.urlHostAllowed)
        let pixelsForAPIRequest = Int(max(size.width, size.height)) * 2
        
        let url = URL(string: "https://en.wikipedia.org/w/api.php?action=query&titles=\(escapedString!)&prop=pageimages&format=json&pithumbsize=\(pixelsForAPIRequest)")
        
        let task: URLSessionTask = URLSession.shared.dataTask(with: url!, completionHandler: {
            (data: Data?, response: URLResponse?, error: Error?) in
            if error == nil {
                let wikiDict = try! JSONSerialization.jsonObject(with: data!, options: JSONSerialization.ReadingOptions.allowFragments) as! NSDictionary
        
                if let query = wikiDict.object(forKey: "query") as? NSDictionary {
                    if let pages = query.object(forKey: "pages") as? NSDictionary {
                        if let pageContent = pages.allValues.first as? NSDictionary {
                            if let thumbnail = pageContent.object(forKey: "thumbnail") as? NSDictionary {
                                if let thumbURL = thumbnail.object(forKey: "source") as? String {
                                    let faceImage = UIImage(data: try! Data(contentsOf: URL(string: thumbURL)!))
                                    completion(faceImage, true)
                                }
                            }else{
                                completion(nil, false)
                            }
                        }
                    }
                }
            }
        })
        task.resume()
    }
    
    class func centerImageViewOnFace (_ imageView: UIImageView) {
        
        let context = CIContext(options: nil)
        let options = [CIDetectorAccuracy:CIDetectorAccuracyHigh]
        let detector = CIDetector(ofType: CIDetectorTypeFace, context: context, options: options)
        
        let faceImage = imageView.image
        let ciImage = CIImage(cgImage: faceImage!.cgImage!)
        
        let features = detector?.features(in: ciImage)
        
        if (features?.count)! > 0 {
            
            var face:CIFaceFeature!
            
            for rect in features! {
                face = rect as? CIFaceFeature
            }
            
            var faceRectWithExtendedBounds = face.bounds
            faceRectWithExtendedBounds.origin.x -= 20
            faceRectWithExtendedBounds.origin.y -= 30
            
            faceRectWithExtendedBounds.size.width += 40
            faceRectWithExtendedBounds.size.height += 60
            
            let x = faceRectWithExtendedBounds.origin.x / faceImage!.size.width
            let y = (faceImage!.size.height - faceRectWithExtendedBounds.origin.y - faceRectWithExtendedBounds.size.height) / faceImage!.size.height
            
            let widthFace = faceRectWithExtendedBounds.size.width / faceImage!.size.width
            let heightFace = faceRectWithExtendedBounds.size.height  / faceImage!.size.height
            
            imageView.layer.contentsRect = CGRect(x: x, y: y, width: widthFace, height: heightFace)
        }
    }
}
```
这里应该是走异步接口，然后将网络图片设置给UIImageVIew了。

## 11 总结

* 滑动删除效果其实是UITableView里面自带的一个方法editActionsForRowAt方法，里面配置的UITableViewRowAction实现的。

* 3D触摸效果，首先需要在info.plist中配置菜单，然后在AppDelegate中定义菜单的跳转逻辑，可以present方式跳转到目标页面。

* 侧滑菜单，这个需要引入一下SWRevealViewController，这个是oc写的，这里面配置一下菜单控制器，这样就可以实现侧滑效果，主要工作量在SWRevealViewController里面。

* 磁片效果，这个主要是引入了一个三方库，FMMosaicLayout库，需要实现一下FMMosaicLayoutDelegate这个方法，这样就可以实现不同item的大小。

* 基础动画，这里定义了各种各样的基础动画使用方法，Position动画就是height高度区别，Opacity效果是配置alpah，Scale效果是CGAffineTransform这个配置，Color动画就改变一下颜色值，Rotation动画就是设置transform的rotated方法。

* CoreData的使用主要是现在appDelegate中设置saveContext，其它操作类似操作数据库。

* 底部Bar动画，主要是在可见的时候，对每个UITableViewCell里面做了一个动画效果，里面通过设置CGAffineTransform这个实现。

* 系统搜索，这个首先需要配置一个数据源，可以再plist文件里面写一个数组，然后通过系统的搜索类来实现，CSSearchableIndex这个来注入数据。然后配置restoreUserActivityState这个方法，可以决定item跳转目标类。

* 选择头像，这里主要是用了一个工具类，然后通过PHPotoLibrary获取权限，有权限就跳转，没有权限申请，有权限跳转到自定义的ImagePickerSheetController，然后会present这个类。

* wiki-Face，其实是显示一个网络图片的工具类。当我们编辑框结束，点击return后，这里利用WikiFace静态方法去加载网络接口，然后给图片的image设置进去。





