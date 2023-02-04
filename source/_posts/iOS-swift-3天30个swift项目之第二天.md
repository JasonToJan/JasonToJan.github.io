---
title: iOS swift 3天30个swift项目之第二天
date: 2023-02-04 09:49:09
op: false
cover: false
toc: true
mathjax: true
tags:
- 30个Swift项目
categories:
- iOS
---

## 1 渐变TableView

### 1.1 效果
![](./iOS-swift-3%E5%A4%A930%E4%B8%AAswift%E9%A1%B9%E7%9B%AE%E4%B9%8B%E7%AC%AC%E4%BA%8C%E5%A4%A9/cleartableviewcell.gif)
<img src=cleartableviewcell.gif>

### 1.2 代码实现

直接是继承了UITableViewController这个控制器，这个应该是自带了一个列表。内部有一个tableView。
```
class ClearTableViewController: UITableViewController {
```

初始化：
```Swift
override func viewDidLoad() {
        super.viewDidLoad()

        self.view.backgroundColor = UIColor.black
        self.tableView.separatorStyle = UITableViewCellSeparatorStyle.none
        self.tableView.tableFooterView = UIView(frame: CGRect.zero)
        self.tableView.register(TableViewCell.self, forCellReuseIdentifier: "tableCell")        
}
```

这里绑定了TableViewCell。
```Swift
class TableViewCell: UITableViewCell {

    let gradientLayer = CAGradientLayer()
    
    override func awakeFromNib() {
        super.awakeFromNib()
    }
    
    override func setSelected(_ selected: Bool, animated: Bool) {
        super.setSelected(selected, animated: animated)
    }
    
    override init(style: UITableViewCellStyle, reuseIdentifier: String?) {
        super.init(style: style, reuseIdentifier: reuseIdentifier)
        
        gradientLayer.frame = self.bounds
        let color1 = UIColor(white: 1.0, alpha: 0.2).cgColor
        let color2 = UIColor(white: 1.0, alpha: 0.1).cgColor
        let color3 = UIColor.clear.cgColor
        let color4 = UIColor(white: 0.0, alpha: 0.05).cgColor
        
        gradientLayer.colors = [color1, color2, color3, color4]
        gradientLayer.locations = [0.0, 0.04, 0.95, 1.0]
    }
    
    override func layoutSubviews() {
        super.layoutSubviews()
        gradientLayer.frame = self.bounds
    }

    required init?(coder aDecoder: NSCoder) {
        fatalError("init(coder:) has not been implemented")
    }
}
```
这里把UITableViewCell的bounds给到了一个渐变层CAGradientLayer。

数据绑定：
```Swift
 var tableData = ["Read 3 article on Medium", "Cleanup bedroom", "Go for a run", "Hit the gym", "Build another swift project", "Movement training", "Fix the layout problem of a client project", "Write the experience of #30daysSwift", "Inbox Zero", "Booking the ticket to Chengdu", "Test the Adobe Project Comet", "Hop on a call to mom"]
```

然后因为这个是UITableViewController，就无需重复设置代理了：
```Swift
override func numberOfSections(in tableView: UITableView) -> Int {
        return 1
    }
    
    override func tableView(_ tableView: UITableView, numberOfRowsInSection section: Int) -> Int {
        return tableData.count
    }
    
    override func tableView(_ tableView: UITableView, heightForRowAt indexPath: IndexPath) -> CGFloat {
        return 60
    }
    
    override func tableView(_ tableView: UITableView, cellForRowAt indexPath: IndexPath) -> UITableViewCell {
        
        let cell = tableView.dequeueReusableCell(withIdentifier: "tableCell", for: indexPath) as! TableViewCell
        
        cell.textLabel?.text = tableData[indexPath.row]
        cell.textLabel?.textColor = UIColor.white
        cell.textLabel?.backgroundColor = UIColor.clear
        cell.textLabel?.font = UIFont(name: "Avenir Next", size: 18)
        cell.selectionStyle = UITableViewCellSelectionStyle.none
        return cell
    }
    
    override func tableView(_ tableView: UITableView, willDisplay cell: UITableViewCell, forRowAt indexPath: IndexPath) {
        cell.backgroundColor = colorforIndex(index: indexPath.row)
    }
    
    func colorforIndex(index: Int) -> UIColor {
        
        let itemCount = tableData.count - 1
        let color = (CGFloat(index) / CGFloat(itemCount)) * 0.6
        return UIColor(red: 1.0, green: color, blue: 0.0, alpha: 1.0)
    }
```
重点实现下这几个代理方法即可。颜色设置主要是通过willDisplay这个方法设置进去了。

## 2 登录动画

### 2.1 效果
![](./iOS-swift-3%E5%A4%A930%E4%B8%AAswift%E9%A1%B9%E7%9B%AE%E4%B9%8B%E7%AC%AC%E4%BA%8C%E5%A4%A9/simple%20login%20animation.gif)
<img src=simple%20login%20animation.gif>

### 2.2 故事版关系
![](./iOS-swift-3%E5%A4%A930%E4%B8%AAswift%E9%A1%B9%E7%9B%AE%E4%B9%8B%E7%AC%AC%E4%BA%8C%E5%A4%A9/21_1.png)
<img src=21_1.png>
这里是有两个场景Scene，每个Scene绑定了一个类，也就是左侧的两个Controller。
这里学会了不用代码，直接用storyboard直接实现控制器的跳转。

可以参考下这篇文章：[Xcode新建View Controller Scene并实现界面间跳转的方法](https://blog.csdn.net/Sherlooock/article/details/106825134)。

### 2.3 启动页
```Swift
class SplasViewController: UIViewController {
    
    @IBOutlet weak var signupButton: UIButton!
    @IBOutlet weak var loginButton: UIButton!
    
    override func viewDidLoad() {
        super.viewDidLoad()
        
        signupButton.layer.cornerRadius = 5
        loginButton.layer.cornerRadius = 5 
    }
    
    override var preferredStatusBarStyle: UIStatusBarStyle {
        return UIStatusBarStyle.lightContent
    }
}
这里设置了按钮圆角。
```

### 2.4 登录页
UI定义：
```Swift
@IBOutlet weak var uesernameTextField: UITextField!
@IBOutlet weak var passwordTextField: UITextField!

@IBOutlet weak var centerAlignUsername: NSLayoutConstraint!
@IBOutlet weak var centerAlignPassword: NSLayoutConstraint!

@IBOutlet weak var loginButton: UIButton!
```

初始化：
```Swift
override func viewDidLoad() {
    super.viewDidLoad()
    
    uesernameTextField.layer.cornerRadius = 5
    passwordTextField.layer.cornerRadius = 5
    uesernameTextField.delegate = self
    passwordTextField.delegate = self
    loginButton.layer.cornerRadius = 5
}
```
这里设置编辑框代理，设置按钮圆角。

将要显示：
```Swift
override func viewWillAppear(_ animated: Bool) {
    super.viewWillAppear(animated)
    
    centerAlignUsername.constant -= view.bounds.width
    centerAlignPassword.constant -= view.bounds.width
    loginButton.alpha = 0
}
```
这里设置刚进页面x轴减去控件宽度，实现效果就是从左边滑出来。

已经显示了：
```Swift
override func viewDidAppear(_ animated: Bool) {
    super.viewDidAppear(animated)
    
    UIView.animate(withDuration: 0.5, delay: 0.00, options: .curveEaseOut, animations: {
        
        self.centerAlignUsername.constant += self.view.bounds.width
        self.view.layoutIfNeeded()
    
        }, completion: nil)
    
    UIView.animate(withDuration: 0.5, delay: 0.10, options: .curveEaseOut, animations: {
        
        self.centerAlignPassword.constant += self.view.bounds.width
        self.view.layoutIfNeeded()
        
        }, completion: nil)
    
    UIView.animate(withDuration: 0.5, delay: 0.20, options: .curveEaseOut, animations: {
        
        self.loginButton.alpha = 1
        
        }, completion: nil)

}
```
这是设置3个动画，将三个视图从左边滑出。

## 3 列表动画

### 3.1 效果
![](./iOS-swift-3%E5%A4%A930%E4%B8%AAswift%E9%A1%B9%E7%9B%AE%E4%B9%8B%E7%AC%AC%E4%BA%8C%E5%A4%A9/AnimateTabel.gif)
<img src=AnimateTabel.gif>

### 3.2 第一个列表
初始化：
```Swift
override func viewDidLoad() {
    super.viewDidLoad()    
    self.view.backgroundColor = UIColor.black
    self.tableView.separatorStyle = UITableViewCellSeparatorStyle.none
    self.tableView.tableFooterView = UIView(frame: CGRect.zero)
    self.tableView.register(FirstTableCell.self, forCellReuseIdentifier: "tableCell")
}
```
这里绑定了第一个Table的Cell：
```Swift
class FirstTableCell: UITableViewCell {

    let gradientLayer = CAGradientLayer()
    
    override func awakeFromNib() {
        super.awakeFromNib()
    }
    
    override func setSelected(_ selected: Bool, animated: Bool) {
        super.setSelected(selected, animated: animated)
    }
    
    
    override init(style: UITableViewCellStyle, reuseIdentifier: String?) {
        super.init(style: style, reuseIdentifier: reuseIdentifier)
        
        gradientLayer.frame = self.bounds
        let color1 = UIColor(white: 1.0, alpha: 0.2).cgColor as CGColor
        let color2 = UIColor(white: 1.0, alpha: 0.1).cgColor as CGColor
        let color3 = UIColor.clear.cgColor as CGColor
        let color4 = UIColor(white: 0.0, alpha: 0.05).cgColor as CGColor
        
        gradientLayer.colors = [color1, color2, color3, color4]
        gradientLayer.locations = [0.0, 0.04, 0.95, 1.0]
        layer.insertSublayer(gradientLayer, at: 0)
    }
    
    override func layoutSubviews() {
        super.layoutSubviews()
        gradientLayer.frame = self.bounds
    }
    
    required init?(coder aDecoder: NSCoder) {
        fatalError("init(coder:) has not been implemented")
    }
}
```
这里跟前面类似，设置了渐变背景。通过layer层insertSublayer设置了渐变层实现。

如何实现动画呢？
继续看第一个控制器。

将要的生命周期：
```Swift
override func viewWillAppear(_ animated: Bool) {
    animateTable()
}

func animateTable() {
        self.tableView.reloadData()
        let cells = tableView.visibleCells
        let tableHeight: CGFloat = tableView.bounds.size.height
        
        for i in cells {
            let cell: UITableViewCell = i as UITableViewCell
            cell.transform = CGAffineTransform(translationX: 0, y: tableHeight)
        }
        
        var index = 0
        for a in cells {
            
            let cell: UITableViewCell = a as UITableViewCell
            
            UIView.animate(withDuration: 1.5, delay: 0.05 * Double(index), usingSpringWithDamping: 0.8, initialSpringVelocity: 0, options: [], animations: {
                
                cell.transform = CGAffineTransform(translationX: 0, y: 0);
                
            }, completion: nil)
            
            index += 1
        }
    }
```
这里一个设置了Cell的transform，第一个是从底部初始位置，第二个循环是恢复到目标位置。

第一个table绑定的代理和数据源：
```Swift
override func numberOfSections(in tableView: UITableView) -> Int {
        return 1
    }
    
override func tableView(_ tableView: UITableView, numberOfRowsInSection section: Int) -> Int {
    return tableData.count
}

override func tableView(_ tableView: UITableView, heightForRowAt indexPath: IndexPath) -> CGFloat {
    return 60
}

override func tableView(_ tableView: UITableView, cellForRowAt indexPath: IndexPath) -> UITableViewCell {
    
    let cell = tableView.dequeueReusableCell(withIdentifier: "tableCell", for: indexPath)
    
    cell.textLabel?.text = tableData[indexPath.row]
    cell.textLabel?.textColor = UIColor.white
    cell.textLabel?.backgroundColor = UIColor.clear
    cell.textLabel?.font = UIFont(name: "Avenir Next", size: 18)
    cell.selectionStyle = UITableViewCellSelectionStyle.none
    return cell
}

override func tableView(_ tableView: UITableView, willDisplay cell: UITableViewCell, forRowAt indexPath: IndexPath) {
    cell.backgroundColor =  colorforIndex(indexPath.row)
}

override func tableView(_ tableView: UITableView, didSelectRowAt indexPath: IndexPath) {
    performSegue(withIdentifier: "ShowAnimateTableViewController", sender: nil)
}

func colorforIndex(_ index: Int) -> UIColor {
    let itemCount = tableData.count - 1
    let color = (CGFloat(index) / CGFloat(itemCount)) * 0.6
    return UIColor(red: color, green: 0.0, blue: 1.0, alpha: 1.0)
}
```
这里设置了渐变色，和前面一样。

### 3.3 第2个列表

继承UITableViewController:
```Swift
class AnimateTableViewController: UITableViewController {
```

初始化：
```Swift
 override func viewDidLoad() {
    super.viewDidLoad()
    
    self.view.backgroundColor = UIColor.black
    self.tableView.separatorStyle = UITableViewCellSeparatorStyle.none
    self.tableView.tableFooterView = UIView(frame: CGRect.zero)
    self.tableView.register(SecondTableCell.self, forCellReuseIdentifier: "SecondTableCell")
}
```

将要显示：
```Swift
override func viewWillAppear(_ animated: Bool) {
    animateTable()
}
    
func animateTable() {
    
    self.tableView.reloadData()
    
    let cells = tableView.visibleCells
    let tableHeight: CGFloat = tableView.bounds.size.height
    
    for (index, cell) in cells.enumerated() {
        cell.transform = CGAffineTransform(translationX: 0, y: tableHeight)
        UIView.animate(withDuration: 1.0, delay: 0.05 * Double(index), usingSpringWithDamping: 0.8, initialSpringVelocity: 0, options: [], animations: {
                cell.transform = CGAffineTransform(translationX: 0, y: 0);
            }, completion: nil)
    }
}
```
这里设置了显示动画，从底部弹出效果。

```Swift
override func numberOfSections(in tableView: UITableView) -> Int {
        return 1
    }
    
override func tableView(_ tableView: UITableView, numberOfRowsInSection section: Int) -> Int {
    return tableData.count
}

override func tableView(_ tableView: UITableView, heightForRowAt indexPath: IndexPath) -> CGFloat {
    return 60
}

override func tableView(_ tableView: UITableView, cellForRowAt indexPath: IndexPath) -> UITableViewCell {
    
    let cell = tableView.dequeueReusableCell(withIdentifier: "SecondTableCell", for: indexPath)
    
    cell.textLabel?.text = tableData[indexPath.row]
    cell.textLabel?.textColor = UIColor.white
    cell.textLabel?.backgroundColor = UIColor.clear
    cell.textLabel?.font = UIFont(name: "Avenir Next", size: 18)
    cell.selectionStyle = UITableViewCellSelectionStyle.none
    return cell
    
}

override func tableView(_ tableView: UITableView, willDisplay cell: UITableViewCell, forRowAt indexPath: IndexPath) {
    cell.backgroundColor =  colorforIndex(indexPath.row)
}

func colorforIndex(_ index: Int) -> UIColor {
    let itemCount = tableData.count - 1
    let color = (CGFloat(index) / CGFloat(itemCount)) * 0.6
    return UIColor(red: 1.0, green: color, blue: 0.0, alpha: 1.0)
}
```
这里数据源绑定。还需要一个Cell支持下。
SecondTableCell:
```Swift
class SecondTableCell: UITableViewCell {
    
    let gradientLayer = CAGradientLayer()
    
    override func awakeFromNib() {
        super.awakeFromNib()
    }
    
    override func setSelected(_ selected: Bool, animated: Bool) {
        super.setSelected(selected, animated: animated)
    }

    override init(style: UITableViewCellStyle, reuseIdentifier: String?) {
        super.init(style: style, reuseIdentifier: reuseIdentifier)
        
        gradientLayer.frame = self.bounds
        let color1 = UIColor(white: 1.0, alpha: 0.2).cgColor as CGColor
        let color2 = UIColor(white: 1.0, alpha: 0.1).cgColor as CGColor
        let color3 = UIColor.clear.cgColor as CGColor
        let color4 = UIColor(white: 0.0, alpha: 0.05).cgColor as CGColor
        
        gradientLayer.colors = [color1, color2, color3, color4]
        gradientLayer.locations = [0.0, 0.04, 0.95, 1.0]
        layer.insertSublayer(gradientLayer, at: 0)
    }
    
    override func layoutSubviews() {
        super.layoutSubviews()
        gradientLayer.frame = self.bounds
    }
    
    required init?(coder aDecoder: NSCoder) {
        fatalError("init(coder:) has not been implemented")
    }
}
```
这里和第一个Cell类似。

## 4 游戏抽奖滚动图案

### 4.1 效果
![](./iOS-swift-3%E5%A4%A930%E4%B8%AAswift%E9%A1%B9%E7%9B%AE%E4%B9%8B%E7%AC%AC%E4%BA%8C%E5%A4%A9/emoji%20spin.gif)
<img src=emoji%20spin.gif>

### 4.2 UI
这里一个背景图片
go按钮
中间是UIPickerView，有点像时间滚动条
底部模式，结果文案。

直接在故事版里面拖好了。

然后拖动3个View到控制器：
```Swift
class ViewController: UIViewController {

    @IBOutlet weak var emojiPickerView: UIPickerView!
    @IBOutlet weak var goButton: UIButton!
    @IBOutlet weak var resultLabel: UILabel!
```

### 4.3 Data
```Swift
 var imageArray = [String]()
    var dataArray1 = [Int]()
    var dataArray2 = [Int]()
    var dataArray3 = [Int]()
    var amazingFlag = false
    var bounds: CGRect = CGRect.zero
```
变量声明，等下作为UIPickerView的数据。

### 4.3 生命周期之viewDidLoad
```Swift
override func viewDidLoad() {
    super.viewDidLoad()
    
    bounds = goButton.bounds
    imageArray = ["👻","👸","💩","😘","🍔","🤖","🍟","🐼","🚖","🐷"]
    
    for _ in 0...100 {
        self.dataArray1.append((Int)(arc4random() % 10 ))
        self.dataArray2.append((Int)(arc4random() % 10 ))
        self.dataArray3.append((Int)(arc4random() % 10 ))
    }
    
    resultLabel.text = ""
    
    emojiPickerView.delegate = self
    emojiPickerView.dataSource = self
    
    goButton.layer.cornerRadius = 6
    goButton.layer.masksToBounds = true
}
```
这里生成长度为100的3个数组。本质上存放的0到9个数字。

### 4.4 将要可见
```Swift
override func viewWillAppear(_ animated: Bool) {
    super.viewWillAppear(animated)
    
    goButton.alpha = 0   
}
```

### 4.5 已经可见
```Swift
override func viewDidAppear(_ animated: Bool) {
    super.viewDidAppear(animated)
    
    UIView.animate(withDuration: 0.5, delay: 0.3, options: .curveEaseOut, animations: {
        
        self.goButton.alpha = 1
        
        }, completion: nil)
}
```
底部按钮逐渐显示。

### 4.6 点击事件
```Swift
@IBAction func amazingButtonDidTouch(_ sender: UIButton) {
    amazingFlag = !amazingFlag;
    sender.setTitle(amazingFlag ? "开挂模式":"常规模式", for: .normal)
}

@IBAction func goButtoDidTouch(_ sender: AnyObject) {
    let index1: Int
    let index2: Int
    let index3: Int
    if amazingFlag {
        index1 = Int(arc4random()) % 90 + 3
        index2 = dataArray2.firstIndex(of: dataArray1[index1])!
        index3 = dataArray3.lastIndex(of: dataArray1[index1])!
    } else {
        index1 = Int(arc4random()) % 90 + 3
        index2 = Int(arc4random()) % 90 + 3
        index3 = Int(arc4random()) % 90 + 3
    }
    
    /// 下次多少行，开启动画效果
    emojiPickerView.selectRow(index1, inComponent: 0, animated: true)
    emojiPickerView.selectRow(index2, inComponent: 1, animated: true)
    emojiPickerView.selectRow(index3, inComponent: 2, animated: true)
    
    /// 结果显示
    if(dataArray1[emojiPickerView.selectedRow(inComponent: 0)] == dataArray2[emojiPickerView.selectedRow(inComponent: 1)] && dataArray2[emojiPickerView.selectedRow(inComponent: 1)] == dataArray3[emojiPickerView.selectedRow(inComponent: 2)]) {
        
        resultLabel.text = "Bingo!"
        
    } else {
        resultLabel.text = "💔"
    }
    
    /// 底部Go抖动效果
    UIView.animate(withDuration: 0.5, delay: 0.0, usingSpringWithDamping: 0.1, initialSpringVelocity: 5, options: .curveLinear, animations: {
        
        self.goButton.bounds = CGRect(x: self.bounds.origin.x, y: self.bounds.origin.y, width: self.bounds.size.width - 20, height: self.bounds.size.height)
        
    }, completion: { (compelete: Bool) in
        
        UIView.animate(withDuration: 0.1, delay: 0.0, options: UIViewAnimationOptions(), animations: {
            
            self.goButton.bounds = CGRect(x: self.bounds.origin.x, y: self.bounds.origin.y, width: self.bounds.size.width, height: self.bounds.size.height)
            
        }, completion: nil)
        
    })
}
```

## 5 启动动画

### 5.1 效果
![](./iOS-swift-3%E5%A4%A930%E4%B8%AAswift%E9%A1%B9%E7%9B%AE%E4%B9%8B%E7%AC%AC%E4%BA%8C%E5%A4%A9/splash.gif)
<img src=splash.gif>

### 5.2 启动页
启动页需要再info.plist中配置
如下：
![](./iOS-swift-3%E5%A4%A930%E4%B8%AAswift%E9%A1%B9%E7%9B%AE%E4%B9%8B%E7%AC%AC%E4%BA%8C%E5%A4%A9/25_1.png)
<img src=25_1.png>

这里我们在LaunchScreen.storyboard中设置了一个背景图片。
这里面设置了一张白色的🕊没有效果哦。但是之前有，不知道为啥。

### 5.3 首页
首页其实加了一个同启动页的蓝色背景，中间减掉一个白鸽，然后白鸽再扩展的动画。
这个背景是在故事版的View右侧属性里面的Background中设置的。

设置蒙层：
```Swift
override func viewDidLoad() {
    super.viewDidLoad()
    // Do any additional setup after loading the view, typically from a nib.
    
    self.mask = CALayer()
    self.mask!.contents = UIImage(named: "twitter")?.cgImage
    self.mask!.contentsGravity = kCAGravityResizeAspect
    self.mask!.bounds = CGRect(x: 0, y: 0, width: 100, height: 81)
    self.mask!.anchorPoint = CGPoint(x: 0.5, y: 0.5)
    self.imageView.setNeedsLayout()
    self.imageView.layoutIfNeeded()
    self.mask!.position = CGPoint(x: self.imageView.frame.size.width / 2, y: self.imageView.frame.size.height / 2)
    self.imageView.layer.mask = mask
    
    animateMask()
}
```
这里给到imageView的layer的mask，这样就相当于在蓝色背景中镂空可以看到里面的东西了。

```Swift
func animateMask() {
    let keyFrameAnimation = CAKeyframeAnimation(keyPath: "bounds")
    keyFrameAnimation.delegate = self
    keyFrameAnimation.duration = 0.6
    keyFrameAnimation.beginTime = CACurrentMediaTime() + 0.5
    keyFrameAnimation.timingFunctions = [CAMediaTimingFunction(name: kCAMediaTimingFunctionEaseInEaseOut), CAMediaTimingFunction(name: kCAMediaTimingFunctionEaseInEaseOut)]
    do {
        // 动画需要加上这段代码，否则会造成页面闪一下
        keyFrameAnimation.fillMode = kCAFillModeForwards
        keyFrameAnimation.isRemovedOnCompletion = false
    }
    let initalBounds = NSValue(cgRect: mask!.bounds)
    let secondBounds = NSValue(cgRect: CGRect(x: 0, y: 0, width: 90*0.9, height: 73*0.9))
    let finalBounds = NSValue(cgRect: CGRect(x: 0, y: 0, width: 1600, height: 1300))
    keyFrameAnimation.values = [initalBounds, secondBounds, finalBounds]
    keyFrameAnimation.keyTimes = [0, 0.3, 1]
    self.mask!.add(keyFrameAnimation, forKey: "bounds")
}
```
这个动画的作用是将镂空效果爆炸，显示出主页面。

动画停止后需要去除蒙层哦：
```Swift
extension ViewController : CAAnimationDelegate {
    func animationDidStop(_ anim: CAAnimation, finished flag: Bool) {
        self.imageView.layer.mask = nil
    }
}
```
动画结束，需要将蒙层去掉，不然屏幕上方会多出一块多余的遮挡视图。

## 6 滑动菜单

### 6.1 效果
![](./iOS-swift-3%E5%A4%A930%E4%B8%AAswift%E9%A1%B9%E7%9B%AE%E4%B9%8B%E7%AC%AC%E4%BA%8C%E5%A4%A9/SlideMenu.gif)
<img src=SlideMenu.gif>

### 6.2 首页
```Swift
class NewsTableViewController: BaseTableViewController {
    let menuTransitionManager = MenuTransitionManager()
    
    override func viewDidLoad() {
        super.viewDidLoad()

        self.title = "Everyday Moments"
        self.view.backgroundColor = UIColor(red:0.062, green:0.062, blue:0.07, alpha:1)
    }
    
    override func prepare(for segue: UIStoryboardSegue, sender: Any?) {
        let menuTableViewController = segue.destination as! MenuTableViewController
        menuTableViewController.currentItem = self.title!
        menuTableViewController.transitioningDelegate = menuTransitionManager
        menuTransitionManager.delegate = self
    }
}
```
这里有一个prepare方法，实际上是跳转到其它页面前会执行。
我们在故事版里面定义了跳转关系，这里就可以通过prepare方法拿到跳转的目标控制器了。

这是一个新闻的列表页。
这里首先继承了BaseTableViewController:
```Swift
class BaseTableViewController: UITableViewController {
    
    override var preferredStatusBarStyle: UIStatusBarStyle {
        return UIStatusBarStyle.lightContent
    }
    
    override func viewDidLoad() {
        super.viewDidLoad()
        self.tableView.separatorStyle = UITableViewCellSeparatorStyle.none
    }
}
```

然后这里有一个MenuTransitionManager,是我们自定义的一个类：
个人猜测这个Transition相关的应该是为了实现一个转场动画，实现菜单能够丝滑地顶下来吧。
里面放了一个自己定义的协议MenuTransitionManagerDelegate:
```Swift
@objc protocol MenuTransitionManagerDelegate {
    func dismiss()
}

class MenuTransitionManager: NSObject {
    var duration = 0.5
    var isPresenting = false
    var delegate:MenuTransitionManagerDelegate?
    var snapshot:UIView? {
        didSet {
            if let _delegate = delegate {
                let tapGestureRecognizer = UITapGestureRecognizer(target: _delegate, action: #selector(MenuTransitionManagerDelegate.dismiss))
                snapshot?.addGestureRecognizer(tapGestureRecognizer)
            }
        }
    }
}
```

然后扩展了一个UIViewControllerAnimatedTransitioning，
这个应该是系统的协议：
```Swift
extension MenuTransitionManager : UIViewControllerAnimatedTransitioning {
    func transitionDuration(using transitionContext: UIViewControllerContextTransitioning?) -> TimeInterval {
        return duration
    }
    
    func animateTransition(using transitionContext: UIViewControllerContextTransitioning) {
        let fromView = transitionContext.view(forKey: .from)!
        let toView = transitionContext.view(forKey: .to)!
        let container = transitionContext.containerView
        let moveLeft = CGAffineTransform(translationX: 250, y: 0)
        let moveRight = CGAffineTransform(translationX: 0, y: 0)
        
        if isPresenting {
            snapshot = fromView.snapshotView(afterScreenUpdates: true)
            container.addSubview(toView)
            container.addSubview(snapshot!)
        }
        
        UIView.animate(withDuration: duration, delay: 0.0, usingSpringWithDamping: 0.9, initialSpringVelocity: 0.3, options: .curveEaseInOut, animations: {
            if self.isPresenting {
                self.snapshot?.transform = moveLeft
                toView.transform = .identity
            } else {
                self.snapshot?.transform = .identity
                fromView.transform = moveRight
            }
            
        }, completion: { finished in
            transitionContext.completeTransition(true)
            if !self.isPresenting {
                self.snapshot?.removeFromSuperview()
            }
        })
    }
}
```
同时也扩展了UIViewControllerTransitioningDelegate,也是系统的：
```Swift
extension MenuTransitionManager : UIViewControllerTransitioningDelegate {
    func animationController(forDismissed dismissed: UIViewController) -> UIViewControllerAnimatedTransitioning? {
        isPresenting = false
        return self
    }
    
    func animationController(forPresented presented: UIViewController, presenting: UIViewController, source: UIViewController) -> UIViewControllerAnimatedTransitioning? {
        isPresenting = true
        return self
    }
}
```

再看下首页新闻展示的代理和数据源：
```Swift
extension NewsTableViewController {
    // MARK: UITableViewDataSource
    override func numberOfSections(in tableView: UITableView) -> Int {
        return 3
    }
    
    override func tableView(_ tableView: UITableView, numberOfRowsInSection section: Int) -> Int {
        return 4
    }
    
    override func tableView(_ tableView: UITableView, cellForRowAt indexPath: IndexPath) -> UITableViewCell {
        let cell = tableView.dequeueReusableCell(withIdentifier: "Cell", for: indexPath) as! NewsTableViewCell
        cell.backgroundColor = UIColor.clear
        
        if indexPath.row == 0 {
            cell.postImageView.image = UIImage(named: "1")
            cell.postTitle.text = "Love mountain."
            cell.postAuthor.text = "Allen Wang"
            cell.authorImageView.image = UIImage(named: "a")
            
        } else if indexPath.row == 1 {
            cell.postImageView.image = UIImage(named: "2")
            cell.postTitle.text = "New graphic design - LIVE FREE"
            cell.postAuthor.text = "Cole"
            cell.authorImageView.image = UIImage(named: "b")
            
        } else if indexPath.row == 2 {
            cell.postImageView.image = UIImage(named: "3")
            cell.postTitle.text = "Summer sand"
            cell.postAuthor.text = "Daniel Hooper"
            cell.authorImageView.image = UIImage(named: "c")
            
        } else {
            cell.postImageView.image = UIImage(named: "4")
            cell.postTitle.text = "Seeking for signal"
            cell.postAuthor.text = "Noby-Wan Kenobi"
            cell.authorImageView.image = UIImage(named: "d")
            
        }
        
        return cell
    }
}
```
这里每行能够正常展示了。Cell用了自定义的NewsTableViewCell:
```Swift
class NewsTableViewCell: UITableViewCell {
    @IBOutlet weak var postImageView:UIImageView!
    @IBOutlet weak var postTitle:UILabel!
    @IBOutlet weak var postAuthor:UILabel!
    @IBOutlet weak var authorImageView:UIImageView!

    override func awakeFromNib() {
        super.awakeFromNib()
        authorImageView.layer.cornerRadius = authorImageView.frame.width / 2
        authorImageView.layer.masksToBounds = true
    }

    override func setSelected(_ selected: Bool, animated: Bool) {
        super.setSelected(selected, animated: animated)

        // Configure the view for the selected state
    }
}
```
具体布局还是故事版里面处理的布局。

然后让首页也实现我们自定义的MenuTransitionManagerDelegate 转场代理吧：
```Swift
extension NewsTableViewController : MenuTransitionManagerDelegate {
    func dismiss() {
        dismiss(animated: true, completion: nil)
    }
}
```
这里猜测是点击空白区域，菜单dismiss吧。

### 6.3 菜单控制器
```Swift
class MenuTableViewController: BaseTableViewController {
    var menuItems = ["Everyday Moments", "Popular", "Editors", "Upcoming", "Fresh", "Stock-photos", "Trending"]
    var currentItem = "Everyday Moments"
    
    override func viewDidLoad() {
        super.viewDidLoad()
        self.view.backgroundColor = UIColor(red:0.109, green:0.114, blue:0.128, alpha:1)
    }
    
    // 这里prepare去掉效果一样
    override func prepare(for segue: UIStoryboardSegue, sender: Any?) {
        let menuTableViewController = segue.source as! MenuTableViewController
        
        if let selectedRow = menuTableViewController.tableView.indexPathForSelectedRow?.row {
            currentItem = menuItems[selectedRow]
        }
    }
}

extension MenuTableViewController {
    // MARK: UITableViewDataSource
    override func numberOfSections(in tableView: UITableView) -> Int {
        return 1
    }
    
    override func tableView(_ tableView: UITableView, numberOfRowsInSection section: Int) -> Int {
        return menuItems.count
    }
    
    override func tableView(_ tableView: UITableView, cellForRowAt indexPath: IndexPath) -> UITableViewCell {
        let cell = tableView.dequeueReusableCell(withIdentifier: "Cell", for: indexPath) as! MenuTableViewCell
        
        cell.titleLabel.text = menuItems[indexPath.row]
        cell.titleLabel.textColor = (menuItems[indexPath.row] == currentItem) ? UIColor.white : UIColor.gray
        cell.backgroundColor = UIColor.clear
        return cell
    }
}
```
这里实现了菜单列表显示。

菜单Cell如下：
```Swift
class MenuTableViewCell: UITableViewCell {

    @IBOutlet weak var titleLabel:UILabel!
    
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

## 7 酷炫左右缩放菜单

### 7.1 效果
![](./iOS-swift-3%E5%A4%A930%E4%B8%AAswift%E9%A1%B9%E7%9B%AE%E4%B9%8B%E7%AC%AC%E4%BA%8C%E5%A4%A9/TumblrMenu.gif)
<img src=TumblrMenu.gif>

### 7.2 首页
```Swift
class MainViewController: BaseViewController {

    override func viewDidLoad() {
        super.viewDidLoad()
        self.navigationController?.isNavigationBarHidden = true
    }

    override func didReceiveMemoryWarning() {
        super.didReceiveMemoryWarning()
    }
    
    @IBAction func unwindToMainViewController (_ sender: UIStoryboardSegue){
        self.dismiss(animated: true, completion: nil)
    }
}
```
这里的unwindToMainViewController 是我们自行在故事版拖进来的，这样子在这个控制器里面的点击事件也可以处理的。
这里就是dismiss弹框。
其它ui是我们自己拖到故事版里面的。

### 7.3 菜单页
菜单页是由一个UIVisualEffectView的父布局包裹的。
这个故事版的Scene绑定的class为MenuViewController。
```Swift
class MenuViewController: BaseViewController {
    
    let transitionManager = MenuTransitionManager()
    
    @IBOutlet weak var textButton: UIButton!
    @IBOutlet weak var textLabel: UILabel!
    
    @IBOutlet weak var photoButton: UIButton!
    @IBOutlet weak var photoLabel: UILabel!
    
    @IBOutlet weak var quoteButton: UIButton!
    @IBOutlet weak var quoteLabel: UILabel!
    
    @IBOutlet weak var linkButton: UIButton!
    @IBOutlet weak var linkLabel: UILabel!
    
    @IBOutlet weak var chatButton: UIButton!
    @IBOutlet weak var chatLabel: UILabel!
    
    @IBOutlet weak var audioButton: UIButton!
    @IBOutlet weak var audioLabel: UILabel!
    
    override func viewDidLoad() {
        super.viewDidLoad()
        self.transitioningDelegate = self.transitionManager
    }

    override func didReceiveMemoryWarning() {
        super.didReceiveMemoryWarning()
    }
}
```
这里声明了，布局效果在故事版里面拖动的。

注意了这里self的transitiningDelegate设置了转场动画。

也是我们自己定义的一个类：
```Swift

class MenuTransitionManager: NSObject {

    private var presenting = false

    func offstage(_ amount: CGFloat) -> CGAffineTransform {
        return CGAffineTransform(translationX: amount, y: 0)
    }
    
    func offStageMenuController(_ menuViewController: MenuViewController) {
        if !presenting{
            menuViewController.view.alpha = 0
        }
        let topRowOffset  : CGFloat = 300
        let middleRowOffset : CGFloat = 150
        let bottomRowOffset  : CGFloat = 50
        
        menuViewController.textButton.transform = self.offstage(-topRowOffset)
        menuViewController.textLabel.transform = self.offstage(-topRowOffset)
        
        menuViewController.quoteButton.transform = self.offstage(-middleRowOffset)
        menuViewController.quoteLabel.transform = self.offstage(-middleRowOffset)
        
        menuViewController.chatButton.transform = self.offstage(-bottomRowOffset)
        menuViewController.chatLabel.transform = self.offstage(-bottomRowOffset)
        
        menuViewController.photoButton.transform = self.offstage(topRowOffset)
        menuViewController.photoLabel.transform = self.offstage(topRowOffset)
        
        menuViewController.linkButton.transform = self.offstage(middleRowOffset)
        menuViewController.linkLabel.transform = self.offstage(middleRowOffset)
        
        menuViewController.audioButton.transform = self.offstage(bottomRowOffset)
        menuViewController.audioLabel.transform = self.offstage(bottomRowOffset)
        
    }
    
    func onStageMenuController(_ menuViewController: MenuViewController) {
        menuViewController.view.alpha = 1
        
        menuViewController.textButton.transform = .identity
        menuViewController.textLabel.transform = .identity
        menuViewController.quoteButton.transform = .identity
        menuViewController.quoteLabel.transform = .identity
        menuViewController.chatButton.transform = .identity
        menuViewController.chatLabel.transform = .identity
        menuViewController.photoButton.transform = .identity
        menuViewController.photoLabel.transform = .identity
        menuViewController.linkButton.transform = .identity
        menuViewController.linkLabel.transform = .identity
        menuViewController.audioButton.transform = .identity
        menuViewController.audioLabel.transform = .identity
        
    }
}
```
可以看到这里主要是针对里面控制器的ui的transform做了一些配置。

然后这个管理员实现了UIViewControllerTransitioningDelegate代理：
```Swift
extension MenuTransitionManager : UIViewControllerTransitioningDelegate {
    func animationController(forPresented presented: UIViewController, presenting: UIViewController, source: UIViewController) -> UIViewControllerAnimatedTransitioning? {
        self.presenting = true
        return self
    }
    
    func animationController(forDismissed dismissed: UIViewController) -> UIViewControllerAnimatedTransitioning? {
        self.presenting = false
        return self
    }
}
```

同样也实现了UIViewControllerAnimatedTransitioning这个协议：
```Swift
extension MenuTransitionManager : UIViewControllerAnimatedTransitioning {
    
    func transitionDuration(using transitionContext: UIViewControllerContextTransitioning?) -> TimeInterval {
        return 0.5
    }
    
    func animateTransition(using transitionContext: UIViewControllerContextTransitioning) {
        let container = transitionContext.containerView
        let screens: (from:UIViewController, to:UIViewController) = (transitionContext.viewController(forKey: .from)!, transitionContext.viewController(forKey: .to)!)
        
        let menuViewController = !self.presenting ? screens.from as! MenuViewController : screens.to as! MenuViewController
        let bottomViewController = !self.presenting ? screens.to as UIViewController : screens.from as UIViewController
        
        let menuView: UIView! = menuViewController.view
        let bottomView: UIView! = bottomViewController.view
        
        if (self.presenting) {
             self.offStageMenuController(menuViewController)
        }
        container.addSubview(bottomView)
        container.addSubview(menuView)
        
        let duration = self.transitionDuration(using: transitionContext)
        
        UIView.animate(withDuration: duration, delay: 0.0, usingSpringWithDamping: 0.7, initialSpringVelocity: 0.8, options: [], animations: {
            if (self.presenting) {
                 self.onStageMenuController(menuViewController)
            } else {
                self.offStageMenuController(menuViewController)
            }
        }, completion: { finished in
            transitionContext.completeTransition(true)
            UIApplication.shared.keyWindow!.addSubview(screens.to.view)
        })
    }
}
```
主要是动画执行的时候，走了self.onStageMenuController方法或者self.offStageMenuController这个方法实现动画效果。

## 8 限制字符串

### 8.1 效果
![](./iOS-swift-3%E5%A4%A930%E4%B8%AAswift%E9%A1%B9%E7%9B%AE%E4%B9%8B%E7%AC%AC%E4%BA%8C%E5%A4%A9/Limit.gif)
<img src=Limit.gif>

### 8.2 UI
这里也是先构造一个故事版。
左上角：Close按钮。
右上角：Tweet按钮。
头像：Avatar Image VIew。
底部View: 4个Button
编辑框：UITextView

![](./iOS-swift-3%E5%A4%A930%E4%B8%AAswift%E9%A1%B9%E7%9B%AE%E4%B9%8B%E7%AC%AC%E4%BA%8C%E5%A4%A9/18_1.png)
<img src=18_1.png>

控制器要用的：
```Swift
class ViewController: UIViewController {

    @IBOutlet weak var tweetTextView: UITextView!
    @IBOutlet weak var bottomUIView: UIView!
    @IBOutlet weak var avatarImageView: UIImageView!
    @IBOutlet weak var characterCountLabel: UILabel!
```

初始化生命周期：
```Swift
    override func viewDidLoad() {
        super.viewDidLoad()
        tweetTextView.delegate = self
        avatarImageView.layer.cornerRadius = avatarImageView.frame.width / 2
        tweetTextView.backgroundColor = UIColor.clear
        
        NotificationCenter.default.addObserver(self, selector:#selector(ViewController.keyBoardWillShow(_:)), name:UIResponder.keyboardWillShowNotification, object: nil)
        NotificationCenter.default.addObserver(self, selector:#selector(ViewController.keyBoardWillHide(_:)), name:UIResponder.keyboardWillHideNotification, object: nil)
    }
```
这里配置了UITextView的代理为自己，后面一定会实现相关方法的。

然后这里监听了键盘显示和隐藏哦。
主要是将底部的控制栏跟随键盘上下顶起来。

```Swift
@objc func keyBoardWillShow(_ note:NSNotification) {
    let userInfo  = note.userInfo
    let keyBoardBounds = (userInfo![UIResponder.keyboardFrameEndUserInfoKey] as! NSValue).cgRectValue
    let duration = (userInfo![UIResponder.keyboardAnimationDurationUserInfoKey] as! NSNumber).doubleValue
    let deltaY = keyBoardBounds.size.height
    let animations = {
        self.bottomUIView.transform = CGAffineTransform(translationX: 0, y: -deltaY)
    }
    
    if duration > 0 {
        // 莫名其妙的一段代码, 左移16位能看出来是个啥值吗
        // let options = UIView.AnimationOptions(rawValue: UInt((userInfo![UIResponder.keyboardAnimationCurveUserInfoKey] as! NSNumber).intValue << 16))
        UIView.animate(withDuration: duration, delay: 0, options:[.beginFromCurrentState, .curveLinear], animations: animations, completion: nil)
    }else {
        animations()
    }
}
```
上面的代码是为了在键盘弹出的时候，底部bottomUIView跟随键盘顶起来。

```Swift
@objc func keyBoardWillHide(_ note:NSNotification) {
    let userInfo  = note.userInfo
    let duration = (userInfo![UIResponder.keyboardAnimationDurationUserInfoKey] as! NSNumber).doubleValue
    
    let animations = {
        self.bottomUIView.transform = .identity
    }
    
    if duration > 0 {
        // let options = UIView.AnimationOptions(rawValue: UInt((userInfo![UIResponder.keyboardAnimationCurveUserInfoKey] as! NSNumber).intValue << 16))
        UIView.animate(withDuration: duration, delay: 0, options:[.beginFromCurrentState, .curveLinear], animations: animations, completion: nil)
    }else{
        animations()
    }
    
}
```
上面的代码是为了在键盘隐藏的时候，底部bottomUIView也随之落下去。

然后编辑框的代理如下：
```Swift
extension ViewController : UITextViewDelegate {
    // MARK:UITextViewDelegate
    func textView(_ textView: UITextView, shouldChangeTextIn range: NSRange, replacementText text: String) -> Bool {
        let inputTextLength = text.count - range.length + tweetTextView.text.count
        if inputTextLength > 140 {
            return false
        }
        characterCountLabel.text = "\(140 - inputTextLength)"
        return true
    }
}
```
这里是为了统计当前剩余字符，如果不足，则无法继续输入。

## 9 自定义下拉刷新

### 9.1 效果
![](./iOS-swift-3%E5%A4%A930%E4%B8%AAswift%E9%A1%B9%E7%9B%AE%E4%B9%8B%E7%AC%AC%E4%BA%8C%E5%A4%A9/CustomPullToRefresh.gif)
<img src=CustomPullToRefresh.gif>

### 9.2 变量定义
```Swift
    var refreshController: UIRefreshControl!
    var customView: UIView!
    var labelsArray: Array<UILabel> = []
    var isAnimating = false
    var currentColorIndex = 0
    var currentLabelIndex = 0
    var timer: Timer!
    var dataArray: Array<String> = ["😂", "🤗", "😳", "😌", "😊"]
```
这里第一个就是刷新控制器了，应该主要就是往这里面加逻辑。

### 9.3 初始化
```Swift
override func viewDidLoad() {
    super.viewDidLoad()
    
    tblDemo.delegate = self
    tblDemo.dataSource = self
    refreshController = UIRefreshControl()
    refreshController.backgroundColor = UIColor.clear
    refreshController.tintColor = UIColor.clear
    tblDemo.addSubview(refreshController)
    
    loadCustomRefreshContents()
}
```
这里设置了代理和数据源，new了一个刷新控制器，清空背景，给tableView添加了一个子View。

下面加载自定义xib文件：
```Swift
 func loadCustomRefreshContents() {
        
    let refreshContents = Bundle.main.loadNibNamed("RefreshContents", owner: self, options: nil)
    
    customView = refreshContents![0] as? UIView
    customView.frame = refreshController.bounds
    
    for i in 0..<customView.subviews.count {
        labelsArray.append(customView.viewWithTag(i + 1) as! UILabel)
    }
    
    refreshController.addSubview(customView)
}
```
这个布局里面主要是存放了我们自定义头部的View。

### 9.4 数据源和代理
```Swift
extension ViewController : UITableViewDelegate {
    func tableView(_ tableView: UITableView, didSelectRowAt indexPath: IndexPath) {
        
    }
    
    func tableView(_ tableView: UITableView, heightForRowAt indexPath: IndexPath) -> CGFloat {
        return 80
    }
}

extension ViewController : UITableViewDataSource {
    
    func numberOfSectionsInTableView(tableView: UITableView) -> Int {
        return 1
    }
    
    func tableView(_ tableView: UITableView, numberOfRowsInSection section: Int) -> Int {
        return dataArray.count
    }
    
    func tableView(_ tableView: UITableView, cellForRowAt indexPath: IndexPath) -> UITableViewCell {
        let cell = tableView.dequeueReusableCell(withIdentifier: "idCell", for: indexPath)
        
        cell.textLabel!.text = dataArray[indexPath.row]
        cell.textLabel!.font = UIFont(name: "Apple Color Emoji", size: 40)
        cell.textLabel!.textAlignment = .center
        
        return cell
    }
}
```
这里cell用默认的。

### 9.5 下拉动画实现
首先实现UIScrollViewDelegate代理：
```Swift
extension ViewController : UIScrollViewDelegate {
    func scrollViewDidEndDecelerating(_ scrollView: UIScrollView) {
        if refreshController.isRefreshing {
            if !isAnimating {
                doSomething()
                animateRefreshStep1()
            }
        }
    }
}
```
如果没有在刷新中，没有在动画中，就做点失去，然后走动画第一步。

```Swift 
func doSomething() {
    timer = Timer.scheduledTimer(timeInterval: 5, target: self, selector: #selector(ViewController.endedOfWork), userInfo: nil, repeats: true)
}
```
这里开启定时器，5s后才走endedofWork。
```Swift
@objc func endedOfWork() {
    refreshController.endRefreshing()
    timer.invalidate()
    timer = nil
}
```
这里停止刷新，定时器结束。
说明上面的doSomething只是开启一个定时器，模拟进行网络请求，然后加载动画而已。

动画第一步：
```Swift
func animateRefreshStep1() {
    isAnimating = true
    
    UIView.animate(withDuration: 0.1, delay: 0.0, options: .curveLinear, animations: {
        self.labelsArray[self.currentLabelIndex].transform = CGAffineTransform(rotationAngle: CGFloat(Double.pi/4))
        self.labelsArray[self.currentLabelIndex].textColor = self.getNextColor()
        }, completion: { _ in
            UIView.animate(withDuration: 0.05, delay: 0.0, options: .curveLinear, animations: {
                self.labelsArray[self.currentLabelIndex].transform = .identity
                self.labelsArray[self.currentLabelIndex].textColor = UIColor.black
                }, completion: { _ in
                    self.currentLabelIndex += 1
                    if self.currentLabelIndex < self.labelsArray.count {
                        self.animateRefreshStep1()
                    }else {
                        self.animateRefreshStep2()
                    }
            })
    })
}

func getNextColor() -> UIColor {
    var colorsArray: Array<UIColor> = [.magenta, .brown, .yellow,
                                        .red, .green, .blue, .orange]
    if currentColorIndex == colorsArray.count {
        currentColorIndex = 0
    }
    let returnColor = colorsArray[currentColorIndex]
    currentColorIndex += 1
    return returnColor
}
```
这里开启动画了，给子View配置transform和textColor。
递归走每一个子View动画，走完后走动画第二步：
```Swift
func animateRefreshStep2() {
    UIView.animate(withDuration: 0.40, delay: 0.0, options: .curveLinear, animations: {
        let scale = CGAffineTransform(scaleX: 1.5, y: 1.5)
        self.labelsArray[1].transform = scale
        self.labelsArray[2].transform = scale
        self.labelsArray[3].transform = scale
        self.labelsArray[4].transform = scale
        self.labelsArray[5].transform = scale
        self.labelsArray[6].transform = scale
        self.labelsArray[7].transform = scale
        self.labelsArray[8].transform = scale
        self.labelsArray[9].transform = scale
        self.labelsArray[10].transform = scale
        self.labelsArray[11].transform = scale
        
        }, completion: { _ in
            UIView.animate(withDuration: 0.25, delay: 0.0, options: .curveLinear, animations: {
                self.labelsArray[0].transform = .identity
                self.labelsArray[1].transform = .identity
                self.labelsArray[2].transform = .identity
                self.labelsArray[3].transform = .identity
                self.labelsArray[4].transform = .identity
                self.labelsArray[5].transform = .identity
                self.labelsArray[6].transform = .identity
                self.labelsArray[7].transform = .identity
                self.labelsArray[8].transform = .identity
                self.labelsArray[9].transform = .identity
                self.labelsArray[10].transform = .identity
                self.labelsArray[11].transform = .identity
                
                }, completion: { _ in
                    if self.refreshController.isRefreshing {
                        self.currentLabelIndex = 0
                        self.animateRefreshStep1()
                    } else {
                        self.isAnimating = false
                        self.currentLabelIndex = 0
                        for i in 0 ..< self.labelsArray.count {
                            self.labelsArray[i].textColor = UIColor.black
                            self.labelsArray[i].transform = .identity
                        }
                    }
            })
    })
}
```

## 10 CollectionView动画

### 10.1 效果
![](./iOS-swift-3%E5%A4%A930%E4%B8%AAswift%E9%A1%B9%E7%9B%AE%E4%B9%8B%E7%AC%AC%E4%BA%8C%E5%A4%A9/CollectionViewAnimation.gif)
<img src=CollectionViewAnimation.gif>

### 10.2 首页
Main.storyboard里面放置了一个UICollectionView。

里面放置了2个结构体：
```Swift
private struct Storyboard {
    static let CellIdentifier = "AnimationCollectionViewCell"
    static let NibName = "AnimationCollectionViewCell"
}
    
private struct Constants {
    static let AnimationDuration: Double = 0.5
    static let AnimationDelay: Double = 0.0
    static let AnimationSpringDamping: CGFloat = 1.0
    static let AnimationInitialSpringVelocity: CGFloat = 1.0
}
```

成员定义：
```Swift
@IBOutlet var testCollectionView: UICollectionView!

var imageCollection: AnimationImageCollection!
```
第一个是拖过来的，第二个是自定义的：
```Swift
struct AnimationImageCollection {
    private let imagePaths = ["1", "2", "3", "4", "5"]
    var images: [AnimationCellModel]
    
    init() {
        images = imagePaths.map { AnimationCellModel(imagePath: $0) }
    }
}
```
这个是一个结构体，里面的实体是这个：
```Swift
struct AnimationCellModel {
    let imagePath: String
    
    init(imagePath: String?) {
        self.imagePath = imagePath ?? ""
    }
}
```
很简单，放置了一个图片路径而已。

回到第一个控制器：
```Swift
override func viewDidLoad() {
    super.viewDidLoad()
    
    imageCollection = AnimationImageCollection()
    testCollectionView.register(UINib(nibName: Storyboard.NibName, bundle: nil), forCellWithReuseIdentifier: Storyboard.CellIdentifier)
}
```
这里集合View注册了一个Cell，这个Cell是我们建立的xib文件，名称叫做AnimationCollectionViewCell.

### 10.3 子item
应该就是这个itemCell了，还是看下吧：
```Swift
class AnimationCollectionViewCell: UICollectionViewCell {
    
    @IBOutlet weak var backButton: UIButton!
    @IBOutlet weak var animationImageView: UIImageView!
    @IBOutlet weak var animationTextView: UITextView!
    
    var backButtonTapped: (() -> Void)?
    
    func prepareCell(_ viewModel: AnimationCellModel) {
        animationImageView.image = UIImage(named: viewModel.imagePath)
        animationTextView.isScrollEnabled = false
        backButton.isHidden = true
        addTapEventHandler()
    }
    
    func handleCellSelected() {
        animationTextView.isScrollEnabled = false
        backButton.isHidden = false
        self.superview?.bringSubview(toFront: self)
    }
    
    private func addTapEventHandler() {
        backButton.addTarget(self, action: #selector(backButtonDidTouch(_:)), for: .touchUpInside)
    }
    
    @objc func backButtonDidTouch(_ sender: UIGestureRecognizer) {
        backButtonTapped?()
    }
}
```
item布局就是返回按钮，图片，和文案。
这个返回按钮默认应该是隐藏的，选中才给它显示。选中的时候走handleCellSelected方法，
这里走了一个
```
  self.superview?.bringSubview(toFront: self)
```
这个作用应该就是把item移动到最前面了。

```Swift
 private func addTapEventHandler() {
    backButton.addTarget(self, action: #selector(backButtonDidTouch(_:)), for: .touchUpInside)
}

@objc func backButtonDidTouch(_ sender: UIGestureRecognizer) {
    backButtonTapped?()
}
```
这里将返回的点击事件暴露出去了。

回调给首页里面了：
```Swift
// MARK: 按钮事件
func backButtonDidTouch() {
    guard let indexPaths = self.collectionView!.indexPathsForSelectedItems else {
        return
    }

    collectionView!.isScrollEnabled = true
    collectionView!.reloadItems(at: indexPaths)
}
```

### 10.4 代理设置和数据绑定
```Swift
 // MARK: UICollectionViewDataSource
override func collectionView(_ collectionView: UICollectionView, cellForItemAt indexPath: IndexPath) -> UICollectionViewCell {
    
    guard let cell = collectionView.dequeueReusableCell(withReuseIdentifier: Storyboard.CellIdentifier, for: indexPath) as? AnimationCollectionViewCell,
        let viewModel = imageCollection.images.safeIndex(indexPath.item) else {
        return UICollectionViewCell()
    }
    // 这里是自己定义的方法哦
    cell.prepareCell(viewModel)
    return cell
}

override func collectionView(_ collectionView: UICollectionView, numberOfItemsInSection section: Int) -> Int {
    return imageCollection.images.count
}

// MARK: UICollectionViewDelegate
override func collectionView(_ collectionView: UICollectionView, didSelectItemAt indexPath: IndexPath) {
    guard let cell = collectionView.cellForItem(at: indexPath) as? AnimationCollectionViewCell else {
        return
    }
    
    self.handleAnimationCellSelected(collectionView, cell: cell)
}
```
这里选中的时候走了handleAnimationCellSelected这个方法，才实现动画效果的：
```Swift
private func handleAnimationCellSelected(_ collectionView: UICollectionView, cell: AnimationCollectionViewCell) {
        
    cell.handleCellSelected()
    cell.backButtonTapped = self.backButtonDidTouch
    
    let animations = {
        cell.frame = self.view.bounds
    }

    let completion: (_ finished: Bool) -> () = { _ in
        collectionView.isScrollEnabled = false
    }

    UIView.animate(withDuration: Constants.AnimationDuration, delay: Constants.AnimationDelay, usingSpringWithDamping: Constants.AnimationSpringDamping, initialSpringVelocity: Constants.AnimationInitialSpringVelocity, options: [], animations: animations, completion: completion)
}
```
动画主要是设置了cell的frame吧。


## 11 总结
* 1. 渐变TableView主要实现的是每个cell颜色不同，实现渐变效果，主要是在Cell里面定义了一个CAGradientlayer，在tableView的代理方法为willDisplay中给cell设置了背景色。背景色+渐变层实现了这种效果。

* 2. 登录动画的实现方案，就是在视图将要可见，将左侧约束减掉视图宽度，然后已经可见再走UIView的动画函数，再恢复约束，实现动画效果。

* 3. 列表动画，主要也是在将要可见的生命周期，遍历cell，修改cell的transform为CGAffineTransform，改变y值初始值为整个tableView的高度，然后动画设置恢复，从而实现进入动画效果。

* 4. 游戏抽奖滚动动画，主要用了UIPickerView来实现。点击后设置pickerView的selectRow，开启动画效果，即可实现改效果。

* 5. 启动动画，这里主要是加了一个蒙层，通过控制器的mask，设置未CALayer，然后将png镂空图标设置给mask的conents，然后可以通过CAKeyframeAnimation给mask设置动画，这样可以实现启动展开效果。

* 6. 滑动菜单，主要是在prepare方法中，定义了场景的transitioningDelegate为自定义效果，通过扩展UIViewControllerAnimatedTransitioning的系统协议，在animateTransition方法里面加入我们自己定义好的动画，可以实现滑动菜单效果。

* 7. 酷炫左右缩放菜单效果，跳转的逻辑可以在故事版里面写，这里只是菜单里面配置了一个自定义的transitioningDelegate得以实现，具体动画在UIViewControllerAnimatedTransitioning这个协议里面的animateTransition方法中处理，这里面可以拿到跳转的控制器，控制器可以再拿里面的ui。

* 8. 限制字符串，这个比较简单，就是UITextView的用法，一个是监听键盘收起和弹出，给底部栏加动画，另一个是实现UITextViewDelegate，可以拿到当前长度。

* 9. 自定义下拉刷新，主要是对UIRefreshControl做处理，这个控制器可以添加子View，子View可以通过Bundle.main.loadNibNamed加载xib文件。这样可以实现自定义下拉刷新效果。

* 10. 列表转场动画，这个主要就是走didSelectItemAt的协议方法中，实现动画，关键逻辑就是cell.frame赋予self.view.bounds，相当于放大了cell，达成目标效果。









