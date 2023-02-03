---
title: iOS swift 3天30个Swift项目之第一天
date: 2023-02-03 09:44:26
op: false
cover: false
toc: true
mathjax: true
tags:
- 30个Swift项目
categories:
- iOS
---

> 参考项目：https://github.com/soapyigu/Swift-30-Projects 

## 1 计时器

首先看下效果吧。
<img src=01.gif>

### 1.1 主要功能
点击播放按钮，数字开始增加；
点击暂停按钮，数字停止增加；
点击“Reset”，数值置为0；

### 1.2 UI
这里UI采用storyboard来实现。因为比较简单，就4个控件，拖进去故事版即可。
<img src=1.2.png>

### 1.3 功能实现

ui声明，故事版可以直接拖进相关类中，自动生成：
```Swift
@IBOutlet weak var playBtn: UIButton!
@IBOutlet weak var pauseBtn: UIButton!
@IBOutlet weak var timeLabel: UILabel!
```
显示的数字声明：
```Swift
 // 浮点数默认是Double类型，若要使用Float，需要显示声明
    // var counter: Float = 0.0
    var counter: Float = 0.0 {
        // 属性观察器 
        didSet {
            timeLabel.text = String(format: "%.1f", counter)
        }
    }
```
这里的属性观察器是监听这个值变更情况，如果某个地方变更了这个值，那么会走里面的didSet回调。

```Swift
    // 知识点：存储属性和计算属性
    override var preferredStatusBarStyle: UIStatusBarStyle {
        // 只读计算属性，可以去掉get和花括号
//        get {
//            return UIStatusBarStyle.lightContent
//        }
        return UIStatusBarStyle.lightContent
    }
```
上面的代码是控制状态栏图标颜色为亮色（白色），默认是黑色。


```Swift
  // 给予timer一个默认值，这样timer就不会为Optional,
    // 后续可以不用再解包
    // var timer = Timer()
    
    // 这样定义可以在不用timer时回收内存
    var timer: Timer? = Timer()
    var isPlaying = false
```
这里定义了定时器，和定时器Flag。

```Swift
override func viewDidLoad() {
        super.viewDidLoad()
        // 符合LosslessStringConvertible协议的，
        // 都可以直接初始化一个String对象
        // timeLabel.text = String(counter)
        
        // 改成使用属性观察器监控和响应属性值的变化
        counter = 0.0
    }
```
这里在初始化时设置数值为0.0。

```Swift
  @IBAction func resetButtonDidTouch(_ sender: UIButton) {
        if let timerTemp = timer {
            timerTemp.invalidate()
        }
        timer = nil
        isPlaying = false
        counter = 0
        playBtn.isEnabled = true
        pauseBtn.isEnabled = true
    }
```
上面是重置按钮点击事件。

这个可以在故事版中右键视图，在Touch up inside中拖动到代码里面自动生成方法，具体实现逻辑由我们自行添加。这里是将定时器置为nil，然后将播放和暂停状态都设置为可点击状态。

```Swift
@IBAction func playButtonDidTouch(_ sender: UIButton) {
        playBtn.isEnabled = false
        pauseBtn.isEnabled = true
        // 调用实例的方法时建议用self.UpdateTimer,
        // 不建议使用ViewController.UpdateTimer
        // 因为若方法定义成了类方法，第二种方式编译器不会报错。
        timer = Timer.scheduledTimer(timeInterval: 0.1, target:self, selector: #selector(self.UpdateTimer), userInfo: nil, repeats: true)
        isPlaying = true
    }
```
这里是播放按钮的实现逻辑，这里面相当于new了一个Timer。间隔0.1s后刷新，然后会走一个selector里面的updateTimer方法更新数值。

```
   @objc func UpdateTimer() {
        counter = counter + 0.1
    }
```
这里变更数字，然后数值变化引起前面定义的didSet刷新，导致Lable数值更新。

```Swift
 @IBAction func pauseButtonDidTouch(_ sender: UIButton) {
        playBtn.isEnabled = true
        pauseBtn.isEnabled = false
        if let timerTemp = timer {
            timerTemp.invalidate()
        }
        timer = nil
        isPlaying = false
        
    }
```
这里是暂停按钮实现逻辑，将按钮置灰，定时器invalidate，然后定时器置空。

## 2 自定义字体

### 2.1 效果

<img src=Customfont.gif>

### 2.2 字体文件

首先放置在根目录下：
<img src=02_1.png>

这里XCode会自动识别出字体文件：
<img src=02_2.png>

### 2.3 实现细节

控制器可直接实现UITableViewDelegate, UITableViewDataSource,也可以扩展，最好是扩展，可读性好一点。
```Swift
class ViewController: UIViewController, UITableViewDelegate, UITableViewDataSource {
    
    static let identifier = "FontCell"
```

然后定义下数据和字体名称：
```Swift
   var data = ["MFTongXin 30 Days Swift", "MFJinHei 这些字体特别适合打「奋斗」和「理想」",
                "MFZhiHei 谢谢「造字工房」，本案例不涉及商业使用", "Zapfino 使用到造字工房劲黑体，致黑体，童心体",
                "Gaspar呵呵，再见🤗 See you next Project", "微博 @Allen朝辉",
                "测试测试测试测试测试测试", "123", "Alex", "@@@@@@"]
    
    var fontNames = ["MFTongXin_Noncommercial-Regular",
                     "MFJinHei_Noncommercial-Regular",
                     "MFZhiHei_Noncommercial-Regular",
                     "Zapfino",
                     "Gaspar Regular"]
```

然后定义下UI：
```Swift
  @IBOutlet weak var changeFontLabel: UILabel!
    @IBOutlet weak var fontTableView: UITableView!
```

然后定义下选择字体索引：
```
 var fontRowIndex = 0
```

生命周期初始化：
```Swift
   
    override func viewDidLoad() {
        super.viewDidLoad()
        // 在storyboard中直接设置了
        // fontTableView.dataSource = self
        // fontTableView.delegate = self
        
        // 使用手势加Label替换button
        changeFontLabel.layer.cornerRadius = 50
        changeFontLabel.layer.masksToBounds = true
        // 设置为true才能响应手势
        changeFontLabel.isUserInteractionEnabled = true
        let gesture = UITapGestureRecognizer(target: self,
                                             action: #selector(changeFontDidTouch(_:)))
        changeFontLabel.addGestureRecognizer(gesture)
    }

     @objc func changeFontDidTouch(_ sender: AnyObject) {
        
        fontRowIndex = (fontRowIndex + 1) % 5
        print(fontNames[fontRowIndex])
        fontTableView.reloadData()
        
    }
```
这里主要是给一个按钮添加了手势，手势设置了action，然后相当于给按钮添加点击事件，方法有很多，可以直接故事版添加事件，也可以这种手势添加点击事件。

下面是TableView的协议实现，这里Cell用了默认的，无需新建Cell。
```Swift
 func tableView(_ tableView: UITableView, heightForRowAt indexPath: IndexPath) -> CGFloat {
        return 35
    }
    
    func numberOfSections(in tableView: UITableView) -> Int {
        return 1
    }
    
    func tableView(_ tableView: UITableView, cellForRowAt indexPath: IndexPath) -> UITableViewCell {
//        let cell = fontTableView.dequeueReusableCell(withIdentifier: ViewController.identifier)
        let cell = fontTableView.dequeueReusableCell(withIdentifier: ViewController.identifier, for: indexPath)
        let text = data[indexPath.row]
       
        cell.textLabel?.text = text
        cell.textLabel?.textColor = indexPath.row != fontRowIndex ? UIColor.white : UIColor.blue
        cell.textLabel?.font = UIFont(name: self.fontNames[fontRowIndex], size:16)
        
        return cell
    
    }

    func tableView(_ tableView: UITableView, numberOfRowsInSection section: Int) -> Int {
        return data.count
    }
```
主要是在cellForRowAt显示了具体item效果，根据前面选择的字体索引，这里设置给cell了。

## 3 播放本地视频

### 3.1 效果
![](./iOS-swift-3%E5%A4%A930%E4%B8%AASwift%E9%A1%B9%E7%9B%AE%E4%B9%8B%E7%AC%AC%E4%B8%80%E5%A4%A9/playvideo.gif)
<img src=playvideo.gif>

### 3.2 本地视频

直接将mp4文件放置在根目录下。 
Xcode可自动识别到Movie文件夹下。

### 3.3 Core Data
这里播放本地视频并不依赖这个。
因为demo中有部分代码是这个，当然也是可以学习下的。
关于Core Data实际上是一个数据存储框架，类似Realm。
Ficow写的这篇文章还不错：
 [了解和采用 CoreData 框架 —— Ficow 陪你学 CoreData](https://blog.ficowshen.com/page/post/52)

 ### 3.4 UI定义

 ```Swift
  // Swift中mark的使用方式，效果等同OC重的 #pragma mark -
    //MARK:- Variables
    @IBOutlet weak var videoTableView: UITableView!
 ```

 这里就一个UITableView，展示一个列表。

```Swift   
  var playViewController = AVPlayerViewController()
  var playerView = AVPlayer()
```
然后这里定义并且初始化了视频播放关键类。
AVPlayerViewController是一个系统的控制器，用来播放视频的，这里播放本地视频其实也是跳转到系统默认的播放器来播放。

```Swift
var data = [
        // 给项目编译后属于同一个module，所以Video不需要import就可以使用
        Video(image: "videoScreenshot01",
              title: "Introduce 3DS Mario",
              source: "Youtube - 06:32"),
        Video(image: "videoScreenshot02",
              title: "Emoji Among Us",
              source: "Vimeo - 3:34"),
        Video(image: "videoScreenshot03",
              title: "Seals Documentary",
              source: "Vine - 00:06"),
        Video(image: "videoScreenshot04",
              title: "Adventure Time",
              source: "Youtube - 02:39"),
        Video(image: "videoScreenshot05",
              title: "Facebook HQ",
              source: "Facebook - 10:20"),
        Video(image: "videoScreenshot06",
              title: "Lijiang Lugu Lake",
              source: "Allen - 20:30")
    ]
```
上面是列表页数据定义。

```Swift
 //MARK:- View Life Cycle
    override func viewDidLoad() {
        super.viewDidLoad()
        
        videoTableView.dataSource = self
        videoTableView.delegate = self
        
    }
```
初始化里面设置了数据源和代理。

```Swift

//MARK:- UIViewTableView DataSource & Delegate
// 知识点：扩展
// 扩展和 Objective-C 中的分类类似，但没有名称
// 扩展可以为一个类型添加新的功能，但是不能重写已有的功能。
// 扩展可以添加新的计算型属性，但是不可以添加存储型属性，也不可以为已有属性添加属性观察器。
extension ViewController: UITableViewDataSource, UITableViewDelegate {
    // Extensions must not contain stored properties
    //var a = 1
    func tableView(_ tableView: UITableView, heightForRowAt indexPath: IndexPath) -> CGFloat {
    return 220
    }

    func numberOfSections(in tableView: UITableView) -> Int {
    return 2
    }

    func tableView(_ tableView: UITableView, numberOfRowsInSection section: Int) -> Int {
    return data.count
    }

    func tableView(_ tableView: UITableView, cellForRowAt indexPath: IndexPath) -> UITableViewCell {

    // 知识点：向下转型
    // as! 强制类型转换，无法转换时会抛出运行时异常
    // as？可选类型转换，无法转换时返回nil
    let cell = videoTableView.dequeueReusableCell(withIdentifier: "VideoCell", for: indexPath) as! VideoCell
    let video = data[indexPath.row]

    cell.videoScreenshot.image = UIImage(named: video.image)
    cell.videoTitleLabel.text = video.title
    cell.videoSourceLabel.text = video.source

    return cell

    }
}
```
这里扩展实现了UITableView的代理和数据源。

这里用到了一个Cell需要自定义。
```Swift

// 定义Video的结构体，属性初始化后不能被改变，因为结构体时值类型。
// 在你每次定义一个新类或者结构体的时候，实际上你是定义了一个新的 Swift 类型。
// 因此请使用UpperCamelCase这种方式来命名
struct Video {
    let image: String
    let title: String
    let source: String
}

class VideoCell: UITableViewCell {

    @IBOutlet weak var videoScreenshot: UIImageView!
    @IBOutlet weak var videoTitleLabel: UILabel!
    @IBOutlet weak var videoSourceLabel: UILabel!
    
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
这个VideoCell是怎么添加视图的呢，当然也是故事版的作用了。
全局搜索了下VideoCell，发现Main.storyboard里面用到了这个VideoCell，这里关联到了故事版。
<img src=03_1.png>
这里注意到右上角是一个Custom Class，说明故事版是可以关联自定义类的，这里就把Cell关联进来了。

然后还是在Main.storyboard中，给cell里面的botton添加了一个点击事件给控制器：
```Swift
@IBAction func playVideoButtonDidTouch(_ sender: AnyObject) {
        let path = Bundle.main.path(forResource: "emoji zone", ofType: "mp4")
        
        playerView = AVPlayer(url: URL(fileURLWithPath: path!))
        
        playViewController.player = playerView
        
        // 知识点：尾随闭包
        // 在使用尾随闭包时，你不用写出它的参数标签
        // 如果闭包表达式是函数或方法的唯一参数，则当你使用尾随闭包时，你甚至可以把 () 省略掉
        // 完整形式如下：
        //self.present(playViewController, animated: true, completion: {
        //    self.playViewController.player?.play()
        //})
        self.present(playViewController, animated: true) {
            self.playViewController.player?.play()
        }
    }
 }
```
这里给系统的控制器设置了一个AVPlayer，AVPlayer里面设置了URL，这样实现了播放视频效果。

## 4 摄像头

### 4.1 效果预览

![](./iOS-swift-3%E5%A4%A930%E4%B8%AASwift%E9%A1%B9%E7%9B%AE%E4%B9%8B%E7%AC%AC%E4%B8%80%E5%A4%A9/snapchatmenu.gif)
<img src=snapchatmenu.gif>

### 4.2 UI
这里主要是一个横滑的UIScrollView。
长度设置了屏幕的3倍，第一块放置一个UIImageView，中间放置摄像头，最后一块也放置了一个UIImageView。

这里左侧的视图，采用了xib的方式来创建。
我个人理解，xib的方式有点类似Android的xml，因为我本身是学Android，希望大家原谅我这样比喻。xib也是很直观,可以看到效果，而且也可以挂载到某个具体的UIView上。

左侧的xib是这样的：
![](./iOS-swift-3%E5%A4%A930%E4%B8%AASwift%E9%A1%B9%E7%9B%AE%E4%B9%8B%E7%AC%AC%E4%B8%80%E5%A4%A9/04_1.png)
<img src=04_1.png>

右侧的xib是这样的：
![](./iOS-swift-3%E5%A4%A930%E4%B8%AASwift%E9%A1%B9%E7%9B%AE%E4%B9%8B%E7%AC%AC%E4%B8%80%E5%A4%A9/04_2.png)
<img src=04_2.png>

中间的就是一个控制器来的,当然这个控制器也是可以关联xib的。
如何关联的？
这里有一个CameraView.xib，它的File's Owner是CameraView，这样就关联起来了。所以再CameraView.swift里面操作的都是针对这个xib的逻辑处理。

重点看下如何显示摄像头的吧：
首先是UI关联：
```
@IBOutlet weak var cameraView: UIView!
```
这个应该是拖进来的。

然后就是几个关键的类：
```Swift
 var captureSession : AVCaptureSession?
 var stillImageOutput : AVCaptureStillImageOutput?
 var previewLayer : AVCaptureVideoPreviewLayer?
```

然后是将要显示的时候逻辑处理：
```Swift
override func viewWillAppear(_ animated: Bool) {
        super.viewWillAppear(animated)
        
        captureSession = AVCaptureSession()
        // 已经不能再使用了
        // captureSession?.sessionPreset = AVCaptureSessionPreset1920x1080
        captureSession?.sessionPreset = AVCaptureSession.Preset.hd1920x1080
        let backCamera = AVCaptureDevice.devices(for: .video).first!
        // 已经不能再使用了
        // let backCamera = AVCaptureDevice.defaultDevice(withMediaType: AVMediaTypeVideo)
        var error : NSError?
        var input: AVCaptureDeviceInput!
        
        do {
            input = try AVCaptureDeviceInput(device: backCamera) }
        catch let error1 as NSError {
            error = error1
            input = nil
        }
        
        if (error == nil && captureSession?.canAddInput(input) != nil) {
            
                captureSession?.addInput(input)
                
                stillImageOutput = AVCaptureStillImageOutput()
                stillImageOutput?.outputSettings = [AVVideoCodecKey : AVVideoCodecJPEG]
                
            if let stillImageOutputTemp = stillImageOutput {
                if captureSession?.canAddOutput(stillImageOutputTemp) != nil {
                    captureSession?.addOutput(stillImageOutputTemp)
                    if let captureSessionTemp = captureSession {
                        previewLayer = AVCaptureVideoPreviewLayer(session: captureSessionTemp)
                        previewLayer?.videoGravity = AVLayerVideoGravity.resizeAspect
                        // 已经废弃不用了
                        // previewLayer?.videoGravity = AVLayerVideoGravityResizeAspect
                        previewLayer?.connection?.videoOrientation = AVCaptureVideoOrientation.portrait
                        cameraView.layer.addSublayer(previewLayer!)
                        captureSession?.startRunning()
                    }
                }
            }
        }
        
    }
```
主要是初始化了一个AVCaptureSession，然后又创建了一个AVCaptureDeviceInput来接收输入流，这样利用session.addInput，将输入流给input，然后搞一个输出流AVCaptureStillImageOutput，来接收captureSession.addOutPut，这样最后我们再创建一个 预览类叫做AVCaptureVideoPreviewLayer，里面放session，这样就得到一个Layer了。

这个Layer我们就可以加到普通的UIView上了。

### 4.3 根控制器

先定义一个UIScollView:
```Swift

class ViewController: UIViewController {
    

    @IBOutlet weak var scrollView: UIScrollView!
```
初始化添加子View，3块布局给它add进去：
```Swift
override func viewDidLoad() {
    super.viewDidLoad()
    
    // 已经废弃了，使用prefersStatusBarHidden属性返回设置的值
    // UIApplication.shared.isStatusBarHidden = true
    
    let screenWidth = UIScreen.main.bounds.width
    let screenHeight = UIScreen.main.bounds.height
    let leftView: UIViewController = UINib(nibName: "LeftView", bundle: nil).instantiate(withOwner: nil, options: nil)[0] as! UIViewController
    let centerView: CameraView = CameraView(nibName: "CameraView", bundle: nil)
    let rightView: RightView = RightView(nibName: "RightView", bundle: nil)
    
    leftView.view.frame = CGRect(x: 0, y: 0, width: screenWidth-200, height: screenHeight)
    centerView.view.frame = CGRect(x: screenWidth, y: 0, width: screenWidth, height: screenHeight)
    rightView.view.frame = CGRect(x: 2*screenWidth, y: 0, width: screenWidth, height: screenHeight)

    self.scrollView.addSubview(leftView.view)
    self.scrollView.addSubview(rightView.view)
    self.scrollView.addSubview(centerView.view)
    self.scrollView.contentSize = CGSize(width: screenWidth * 3, height: screenHeight)
}
```
frame要对应好屏幕坐标。
scrollView的contentSize刚好对应3个屏幕大小。

这里可以直接创建一个UINib作为控制器，传xib的名称给它就可以的。

另外这里用到了相机，需要相机的权限声明哦：
在info.plist文件下配置：
Privacy - Camera Usage Description 

然后Value自己随便填下就好了。

## 5 传送带效果

### 5.1 效果
![](./iOS-swift-3%E5%A4%A930%E4%B8%AASwift%E9%A1%B9%E7%9B%AE%E4%B9%8B%E7%AC%AC%E4%B8%80%E5%A4%A9/Carousel.gif)
<img src=Carousel.gif>

### 5.2 故事版关联

![](./iOS-swift-3%E5%A4%A930%E4%B8%AASwift%E9%A1%B9%E7%9B%AE%E4%B9%8B%E7%AC%AC%E4%B8%80%E5%A4%A9/5_1.png)
<img src=5_1.png>
首先Main.storyboard是在info.plist中注册的。
然后这个首页故事版配置的控制器的Class就是代码区域表示的控制器了。
层层关联环环相扣。

### 5.3 UI

这里其实就一个背景图片+一个UICollectionView。
```Swift
class HomeViewController: UIViewController {

    
    @IBOutlet weak var backgroundImageView: UIImageView!
    @IBOutlet weak var collectionView: UICollectionView!
```

### 5.4 Data
```Swift
// 访问权限分物种：private，fileprivate，internal，public 和 open
    // private：只能在本类的作用域且在当前文件内能访问
    // fileprivate：只能在当前文件内能访问
    // internal：本module内能访问
    // public：跨module访问但不能重写或继承
    // open：跨module访问并且能重写或继承
    fileprivate var interests = Interest.createInterests()
```

```Swift
class Interest
{
    // MARK: - Public API
    var title = ""
    var description = ""
    var numberOfMembers = 0
    var numberOfPosts = 0
    var featuredImage: UIImage!
    
    init(title: String, description: String, featuredImage: UIImage!)
    {
        self.title = title
        self.description = description
        self.featuredImage = featuredImage
        numberOfMembers = 1
        numberOfPosts = 1
    }
    
    // MARK: - Private
    // dummy data
    static func createInterests() -> [Interest]
    {
        return [
            Interest(title: "Hello there, i miss u.", description: "We love backpack and adventures! We walked to Antartica yesterday, and camped with some cute pinguines, and talked about this wonderful app idea. 🐧⛺️✨", featuredImage: UIImage(named: "hello")!),
            Interest(title: "🐳🐳🐳🐳🐳", description: "We love romantic stories. We walked to Antartica yesterday, and camped with some cute pinguines, and talked about this wonderful app idea. 🐧⛺️✨", featuredImage: UIImage(named: "dudu")!),
            Interest(title: "Training like this, #bodyline", description: "Create beautiful apps. We walked to Antartica yesterday, and camped with some cute pinguines, and talked about this wonderful app idea. 🐧⛺️✨", featuredImage: UIImage(named: "bodyline")!),
            Interest(title: "I'm hungry, indeed.", description: "Cars and aircrafts and boats and sky. We walked to Antartica yesterday, and camped with some cute pinguines, and talked about this wonderful app idea. 🐧⛺️✨", featuredImage: UIImage(named: "wave")!),
            Interest(title: "Dark Varder, #emoji", description: "Meet life with full presence. We walked to Antartica yesterday, and camped with some cute pinguines, and talked about this wonderful app idea. 🐧⛺️✨", featuredImage: UIImage(named: "darkvarder")!),
            Interest(title: "I have no idea, bitch", description: "Get up to date with breaking-news. We walked to Antartica yesterday, and camped with some cute pinguines, and talked about this wonderful app idea. 🐧⛺️✨", featuredImage: UIImage(named: "hhhhh")!),
        ]
    }
}
```
这里创建了一个静态函数，就是数据定义了。

### 5.5 绑定数据
```Swift
extension HomeViewController : UICollectionViewDataSource {
    
    func numberOfSections(in collectionView: UICollectionView) -> Int {
        return 1
    }
    
    func collectionView(_ collectionView: UICollectionView, numberOfItemsInSection section: Int) -> Int {
        return interests.count
    }
    
    func collectionView(_ collectionView: UICollectionView, cellForItemAt indexPath: IndexPath) -> UICollectionViewCell {
        let cell = collectionView.dequeueReusableCell(withReuseIdentifier: Storyboard.CellIdentifier, for: indexPath) as! InterestCollectionViewCell
        
        cell.interest = self.interests[indexPath.item]
        
        return cell
        
    }
}
```
这里就是UICollectionView的数据源了。
对应的Cell为InterestCollectionViewCell
> 这里注意到这个UICollectionView好像没有设置dataSource和delegate，这里看了下Main.storyboard里面，果然在右侧看到Outlets设置了dataSource和delegate都给了控制器。所以说在故事版里面是可以设置数据源和代理的。

如下：
```Swift
class InterestCollectionViewCell: UICollectionViewCell {
    
    
    var interest: Interest! {
        didSet {
            updateUI()
        }
    }
    
    @IBOutlet weak var featuredImageView: UIImageView!
    @IBOutlet weak var interestTitleLabel: UILabel!
    
    fileprivate func updateUI() {
        interestTitleLabel?.text! = interest.title
        featuredImageView?.image! = interest.featuredImage
    }
    
    override func layoutSubviews() {
        super.layoutSubviews()
        
        self.layer.cornerRadius = 5.0
        self.clipsToBounds = true
    }
}
```
因为在Main.storyboard的已经关联了这个Cell，所以布局相关的都无需重新addView。

上面这些代码就可以实现一个传送带效果。

但是我们想要左滑，它自己能够刚好滑动到下一个屏幕，体验效果好一点的话要怎么办呢？

我们想到第一个优化点：
一张图能不能刚好占一个屏幕，增大一点padding。

可以的。

利用UICollectionView提供的另外一个协议可以实现。答案就是：UICollectionViewDelegateFlowLayout。

```Swift
extension HomeViewController : UICollectionViewDelegateFlowLayout {
    func collectionView(_ collectionView: UICollectionView, layout collectionViewLayout: UICollectionViewLayout, sizeForItemAt indexPath: IndexPath) -> CGSize {
        return CGSize(width: UIScreen.main.bounds.width - 2 * Storyboard.CellPadding, height: 450)
    }

    func collectionView(_ collectionView: UICollectionView, layout collectionViewLayout: UICollectionViewLayout, minimumLineSpacingForSectionAt section: Int) -> CGFloat {
        return 2 * Storyboard.CellPadding
    }

    func collectionView(_ collectionView: UICollectionView, layout collectionViewLayout: UICollectionViewLayout, insetForSectionAt section: Int) -> UIEdgeInsets {
        return UIEdgeInsetsMake(0, Storyboard.CellPadding, 0, Storyboard.CellPadding)
    }
}
```
这里第一个sizeForItemAt方法：单个item的大小，长度为屏幕宽度-左右两个padding，高度写死450。
这里第二个方法minimumLine: collectionView的滚动方向为水平时，这个就是item之间的最小水平间距。
这里的第三个方法insetForSectionAt: 单个item的padding，设置上，左，右，下。

第二个优化点：当滑动到一半多，自动切下一张，不足一半，切回上一张。
也好办。

```Swift
extension HomeViewController : UIScrollViewDelegate {
    func scrollViewWillEndDragging(_ scrollView: UIScrollView, withVelocity velocity: CGPoint, targetContentOffset: UnsafeMutablePointer<CGPoint>) {
        let originPoint = targetContentOffset.pointee;
        var index = Int(originPoint.x / UIScreen.main.bounds.width)
        let offset = Int(originPoint.x) % Int(UIScreen.main.bounds.width)
        index += (offset > Int(UIScreen.main.bounds.width/2) ? 1 : 0)
        targetContentOffset.pointee = CGPoint(x: index * Int(UIScreen.main.bounds.width) , y: 0)
    }
}
```
继续扩展下这个控制器，这里面，判断了offset在停止滑动后的逻辑。

## 6 定位

### 6.1 效果
![](./iOS-swift-3%E5%A4%A930%E4%B8%AASwift%E9%A1%B9%E7%9B%AE%E4%B9%8B%E7%AC%AC%E4%B8%80%E5%A4%A9/mylocation.gif)
<img src=mylocation.gif>

### 6.2 UI
这个UI比较简单，一个UILabel，一个按钮。

这个按钮的点击事件直接通过故事版链接到了控制器，所以控制器无需有这个UIView，有这个点击事件就行了。
![](./iOS-swift-3%E5%A4%A930%E4%B8%AASwift%E9%A1%B9%E7%9B%AE%E4%B9%8B%E7%AC%AC%E4%B8%80%E5%A4%A9/06_1.png)
<img src=06_1.png>

### 6.3 如何定位
首先声明一个全局变量：
```Swift
  // 强制自动解包，可以赋值为nil，为nil后再调用会报错
    // 建议定义为：
    // var locationManager: CLLocationManager
    var locationManager: CLLocationManager!
```

开始定位：
```Swift
@IBAction func myLocationButtonDidTouch(_ sender: AnyObject) { 
    locationManager = CLLocationManager()
    locationManager.delegate = self
    locationManager.desiredAccuracy = kCLLocationAccuracyBest
    locationManager.requestAlwaysAuthorization()
    locationManager.startUpdatingLocation()   
}
```

因为这里引入了CLLocationManager，那么在项目层级发现CoreLocation.framework自动引入进来了。

直接这样还不够，还需要设置下协议才行。因为这里我们delegate设置了self。
这里我们同样可以新建一个Delegate，本质上扩展控制器，当然也可以直接写在控制器里面，都可以，最好是分一个文件，可读性会强一点。

```Swift
import Foundation
import CoreLocation

extension ViewController : CLLocationManagerDelegate {
    
    func locationManager(_ manager: CLLocationManager, didFailWithError error: Error) {
        
        self.locationLabel.text = "Error while updating location " + error.localizedDescription
        
    }
    
    func locationManager(_ manager: CLLocationManager, didUpdateLocations locations: [CLLocation]) {
        CLGeocoder().reverseGeocodeLocation(locations.first!) { (placemarks, error) in
            guard error == nil else {
                self.locationLabel.text = "Reverse geocoder failed with error" + error!.localizedDescription
                return
            }
            if placemarks!.count > 0 {
                let pm = placemarks!.first
                self.displayLocationInfo(pm)
            } else {
                self.locationLabel.text = "Problem with the data received from geocoder"
            }
        }
    }
    
    func displayLocationInfo(_ placemark: CLPlacemark?) {
        if let containsPlacemark = placemark {
            //stop updating location to save battery life
            locationManager.stopUpdatingLocation()
            
            let locality = (containsPlacemark.locality != nil) ? containsPlacemark.locality : ""
            let postalCode = (containsPlacemark.postalCode != nil) ? containsPlacemark.postalCode : ""
            let administrativeArea = (containsPlacemark.administrativeArea != nil) ? containsPlacemark.administrativeArea : ""
            let country = (containsPlacemark.country != nil) ? containsPlacemark.country : ""
            
            self.locationLabel.text = postalCode! + " " + locality!
            
            self.locationLabel.text?.append("\n" + administrativeArea! + ", " + country!)
        }
        
    }
}
```

## 7 下拉刷新

### 7.1 效果
![](./iOS-swift-3%E5%A4%A930%E4%B8%AASwift%E9%A1%B9%E7%9B%AE%E4%B9%8B%E7%AC%AC%E4%B8%80%E5%A4%A9/07.gif)
<img src=07.gif>

### 7.2 变量声明
```Swift
class ViewController: UIViewController, UITableViewDataSource, UITableViewDelegate {
    
    var index = 0
    let cellIdentifer = "NewCellIdentifier"
    
    let favoriteEmoji = ["🤗🤗🤗🤗🤗", "😅😅😅😅😅", "😆😆😆😆😆"]
    let newFavoriteEmoji = ["🏃🏃🏃🏃🏃", "💩💩💩💩💩", "👸👸👸👸👸", "🤗🤗🤗🤗🤗", "😅😅😅😅😅", "😆😆😆😆😆" ]
    var emojiData = [String]()
    var tableView: UITableView!
    
    var refreshControl = UIRefreshControl()
    var navBar: UINavigationBar = UINavigationBar(frame: CGRect(x: 0, y: 0, width: 375, height: 64))
```
这里继承了UIViewController，然后再Main.storyboard中配置了首页的控制器就是这个，这样就直接启动这个控制器了。

这里定义了初始化数据3条，更新后的6条。
模拟刷新后的数据变化。

这里不是再故事版里面加视图，直接在代码里面new了。

### 7.3 viewDidLoad初始化
```Swift
 override func viewDidLoad() {
        super.viewDidLoad()
        // 这里new一个UITableView
        tableView = UITableView(frame: self.view.bounds, style: .plain)
        
        // 初始数据
        emojiData = favoriteEmoji
        let emojiTableView = tableView
        
        // 数据源设置，注册Cell，这里的Cell比较简单，直接用了UITableViewCell可满足
        emojiTableView?.backgroundColor = UIColor(red:0.092, green:0.096, blue:0.116, alpha:1)
        emojiTableView?.dataSource = self
        emojiTableView?.register(UITableViewCell.self, forCellReuseIdentifier: cellIdentifer)
        
        // 因为TableView是UIScrollViw，这里也有这个refreshControl的属性
        tableView.refreshControl = self.refreshControl
        self.refreshControl.addTarget(self, action: #selector(ViewController.didRoadEmoji), for: .valueChanged)
        
        // 自定义刷新器 这里的loading效果猜测是refreshControl自带的
        self.refreshControl.backgroundColor = UIColor(red:0.113, green:0.113, blue:0.145, alpha:1)
        let attributes = [NSAttributedStringKey.foregroundColor: UIColor.white]
        // 这里是loading下方的文案
        self.refreshControl.attributedTitle = NSAttributedString(string: "Last updated on \(Date())", attributes: attributes)
        self.refreshControl.tintColor = UIColor.white
        
        self.title = "emoji"
        self.navBar.barStyle = UIBarStyle.blackTranslucent
        
        emojiTableView?.rowHeight = UITableViewAutomaticDimension
        emojiTableView?.estimatedRowHeight = 60.0
        emojiTableView?.tableFooterView = UIView(frame: CGRect.zero)
        emojiTableView?.separatorStyle = UITableViewCellSeparatorStyle.none
        
        // 代码add子View
        self.view.addSubview(emojiTableView!)
        self.view.addSubview(navBar)
    }
```
这里通过在代码里面new一个UITableView方式，也是可以实现UI效果。

### 7.4 设置代理和数据源

```Swift
func tableView(_ tableView: UITableView, numberOfRowsInSection section: Int) -> Int {
        return emojiData.count
    }
    
func numberOfSections(in tableView: UITableView) -> Int {
    return 1
}


func tableView(_ tableView: UITableView, cellForRowAt indexPath: IndexPath) -> UITableViewCell {
    let cell = tableView.dequeueReusableCell(withIdentifier: cellIdentifer)! as UITableViewCell
    cell.textLabel!.text = self.emojiData[indexPath.row]
    cell.textLabel!.textAlignment = NSTextAlignment.center
    cell.textLabel!.font = UIFont.systemFont(ofSize: 50)
    cell.backgroundColor = UIColor.clear
    cell.selectionStyle = UITableViewCellSelectionStyle.none

    return cell
}
```
这里给Cell设置数据了。

### 7.5 下拉监听

```Swift
@objc func didRoadEmoji() {
    DispatchQueue.main.asyncAfter(deadline:DispatchTime.now() + 3 ) {
        self.emojiData = [self.newFavoriteEmoji,self.favoriteEmoji][self.index]
        self.tableView.reloadData()
        self.refreshControl.endRefreshing()
        self.index = (self.index + 1) % 2
    }
}
```
DispatchQueue.main.asyncAfter： 主线程执行延迟任务方法。
这里延迟3s后，去更新数据。然后控制器取消刷新。

## 8 炫彩音乐

### 8.1 效果
![](./iOS-swift-3%E5%A4%A930%E4%B8%AASwift%E9%A1%B9%E7%9B%AE%E4%B9%8B%E7%AC%AC%E4%B8%80%E5%A4%A9/randomMusicColor.gif)
<img src=randomMusicColor.gif>

### 8.2 全局变量定义

```Swift
var audioPlayer = AVAudioPlayer()
    
let gradientLayer = CAGradientLayer()

var timer : Timer?

var backgroundColor: (red: CGFloat, green: CGFloat,blue: CGFloat,alpha: CGFloat)! {
    didSet {
        let color1 = UIColor(red: backgroundColor.blue,
                                green: backgroundColor.green,
                                blue: 0,
                                alpha: backgroundColor.alpha).cgColor
        let color2 = UIColor(red: backgroundColor.red,
                                green: backgroundColor.green,
                                blue: backgroundColor.blue,
                                alpha: backgroundColor.alpha).cgColor
        gradientLayer.colors = [color1, color2]
    }
}
```
这里第一个是AVAudioPlayer，负责音频播放。
这里第二个类是CAGradientLayer，负责渐变色渲染。
timer是定时器。
backgroundColor背景色，如果任何地方更改后，触发渐变色颜色变化。

### 8.3 播放音乐
```Swift
@IBAction func playMusicButtonDidTouch(_ sender: AnyObject) {
        
    //play bg music
    let bgMusic = URL(fileURLWithPath: Bundle.main.path(forResource: "Ecstasy", ofType: "mp3")!)
    
    do {
        try AVAudioSession.sharedInstance().setCategory(AVAudioSessionCategoryPlayback)
        try AVAudioSession.sharedInstance().setActive(true)
        try audioPlayer = AVAudioPlayer(contentsOf: bgMusic)
        
        audioPlayer.prepareToPlay()
        audioPlayer.play()
        
    }
    catch let audioError as NSError {
        print(audioError)
    }
    
    if (timer == nil) {
        timer = Timer.scheduledTimer(timeInterval: 0.2, target: self, selector: #selector(ViewController.randomColor), userInfo: nil, repeats: true)
    }
    
    let redValue = CGFloat(drand48())
    let blueValue =  CGFloat(drand48())
    let greenValue = CGFloat(drand48())
    
    self.view.backgroundColor = UIColor(red: redValue, green: greenValue, blue: blueValue, alpha: 1.0)
    
    //graditent color
    gradientLayer.frame = view.bounds
    gradientLayer.startPoint = CGPoint(x: 0, y: 0)
    gradientLayer.endPoint = CGPoint(x: 1, y: 1)
    
    self.view.layer.addSublayer(gradientLayer)
    
}
```
这里先利用AVAudioSession的单例类设置Active，然后利用AVAudioPlayer播放本地音乐。

然后后面搞了0.2s刷新的周期定时器。每次会执行这个函数：
```Swift
@objc func randomColor() {
    
    let redValue = CGFloat(drand48())
    let blueValue =  CGFloat(drand48())
    let greenValue = CGFloat(drand48())
    
    
    backgroundColor = (redValue, blueValue, greenValue, 1)
    
}
```
这里红蓝绿会随机获取一个值，生成一个颜色值。同时会更新渐变层变更，这样背景就随机变化了。

## 9 图片缩放效果

### 9.1 效果
![](./iOS-swift-3%E5%A4%A930%E4%B8%AASwift%E9%A1%B9%E7%9B%AE%E4%B9%8B%E7%AC%AC%E4%B8%80%E5%A4%A9/09.gif)
<img src=09.gif>

### 9.2 布局构造
这里看了下Main.storyboard
貌似是在ScrollView里面放了一个ImageView哦。

这里有个细节，就是背景有点毛玻璃效果。
其实是这样的，它在Main.storyboard底部设置了一张UIImageView，其实就是这张图片。
然后再搞了一个UIVisualEffectView覆盖再上面，就形成了一种毛玻璃效果。

回到控制器里面，这里无非就是两个视图，并且把这个Constraint约束也加进来了，有4个约束，顶部约束，头部约束，底部约束和尾部约束。

```Swift
@IBOutlet weak var scrollView: UIScrollView!
@IBOutlet weak var imageView: UIImageView!
@IBOutlet weak var imageViewTopConstraint: NSLayoutConstraint!
@IBOutlet weak var imageViewTrailingConstraint: NSLayoutConstraint!
@IBOutlet weak var imageViewBottomConstraint: NSLayoutConstraint!
@IBOutlet weak var imageViewLeadingConstraint: NSLayoutConstraint!
```

### 9.3 代码实现

首先在初始化里面配置了一个mask:
```Swift
 scrollView.autoresizingMask = [.flexibleWidth, .flexibleHeight]
```
这个是自动调整尺寸设置。

其实ScrollView本来就支持缩放的：
> 在iOS中，滚动视图UIScrollView用于查看大于屏幕的内容。Scroll View有两个主要目的：
让用户拖动视图以显示更多内容区域。
让用户使用捏合手势放大或缩小所显示的内容。

这里需要关注一个生命周期函数：viewWillLayoutSubviews
> 有以下几种情况会调用（init初始化不会触发layoutSubviews）
1、addSubview会触发viewWillLayoutSubviews
2、设置self.view及子视图的frame.size会触发layoutSubviews，当然前提是frame.size的值设置前后发生了变化,注意，此处不是origin，呼应官方文档上的边界发生变化
3、滚动一个UIScrollView(该scrollview有子视图的时候)会触发layoutSubviews
4、横竖屏幕切换会触发

这里最为关键的就是这个 viewWillLayoutSubviews函数和viewForZooming了,如果没有这个代码，无法支持缩放效果!
```Swift
//每次控制器更新其子视图时，更新最小缩放比例
override func viewWillLayoutSubviews() {
    super.viewWillLayoutSubviews()
    updateMinZoomScaleForSize(view.bounds.size)
}

//计算scrollView的缩放比例，缩放比例为1表示内容以正常大小显示；缩放比例小于1表示容器内的内容缩小，缩放比例大于1表示放大容器内的内容
fileprivate func updateMinZoomScaleForSize(_ size: CGSize)
{
    //要获得最小的缩放比例，首先计算所需的缩放比例，以便根据其宽度在scrollView中紧贴imageView
    let widthScale = size.width / imageView.bounds.width
    let heightScale = size.height / imageView.bounds.height
    //选取宽度和高度比例中最小的那个,设置为minimumZoomScale
    let minScale = min(widthScale,heightScale)
    
    scrollView.minimumZoomScale = minScale
    scrollView.maximumZoomScale = 3.0
    scrollView.zoomScale = minScale
}
```
这样ScrollView就有一个最小和最大缩放比例了。

另外还有一个协议：
```Swift
func viewForZooming(in scrollView: UIScrollView) -> UIView? {
    //当手势动作发生时，scrollView告诉控制器要放大或缩小子视图imageView
    return imageView
}
```
必须要有viewForZooming，这个作用就是谁可以放大或缩小。

viewForZooming+viewWillLayoutSubviews，两者结合才能实现子View缩放效果。

两位还有一个scrollViewDidZoom方法, 当scrollView缩放时调用,在缩放过程中会被多次调用：
```Swift
func scrollViewDidZoom(_ scrollView: UIScrollView) {
    updateConstraintsForSize(view.bounds.size)
}

//当scrollView的内容大小小于边界时，内容将放置在左上角而不是中心，updateConstraintForSize方法处理这个问题；通过调整图像视图的布局约束。
fileprivate func updateConstraintsForSize(_ size: CGSize) {
    
    ////将图像垂直居中，从视图高度减去imageView的高度并分成两半，这个值用作顶部和底部imageView的约束
    let yOffset = max(0, (size.height - imageView.frame.height) / 2)
    imageViewTopConstraint.constant = yOffset
    imageViewBottomConstraint.constant = yOffset
    
    ////根据宽度计算imageView前后约束的偏移量
    let xOffset = max(0, (size.width - imageView.frame.width) / 2)
    imageViewLeadingConstraint.constant = xOffset
    imageViewTrailingConstraint.constant = xOffset
    
    view.layoutIfNeeded()
}
```

另外还有其它的代理方法如下：
> func scrollViewDidScroll(_ scrollView: UIScrollView)
scrollView滚动时调用，在滚动过程中会多次调用

> func scrollViewWillBeginDragging(_ scrollView: UIScrollView)
将要开始拖拽时调用

> func scrollViewWillEndDragging(_ scrollView: UIScrollView, withVelocity velocity: CGPoint, targetContentOffset: UnsafeMutablePointer<CGPoint>)
将要停止拖拽时 velocity:加速度 向左滑动 x为负值，否则为正值 向上滚动为y为负值否则为正值；targetContentOffset:滚动停止时的ContentOffset

> func scrollViewDidEndDragging(_ scrollView: UIScrollView, willDecelerate decelerate: Bool)
停止拖拽时调用， willDecelerate:停止拖拽时是否要减速，若值为false表示已经停止减速，也就意味着滚动已停止，此时不会调用scrollViewWillBeginDecelerating和scrollViewDidEndDecelerating;若值为true，则代表scrollView正在减速滚动

> func scrollViewWillBeginDecelerating(_ scrollView: UIScrollView)
开始减速的时候调用(也就是松开手指时)，在拖拽滚动的时候，如果松手时已经停止滚动则不会调用

> func scrollViewDidEndDecelerating(_ scrollView: UIScrollView)
停止减速的时候调用（也就是停止滚动的时候调用），在拖拽滚动的时候，如果松手时已经停止滚动则不会调用

> func scrollViewDidEndScrollingAnimation(_ scrollView: UIScrollView)
当调用setContentOffset(_ contentOffset: CGPoint, animated: Bool)/scrollRectToVisible(_ rect: CGRect, animated: Bool)API并且animated参数为true时,会在scrollView滚动结束时调用。若是UITableView或者UICollectionView,调用scrollToRow也和上面一样

> func viewForZooming(in scrollView: UIScrollView) -> UIView?
放回要缩放的view，此view必须是scrollView的subview

> func scrollViewDidZoom(_ scrollView: UIScrollView)
当scrollView缩放时调用,在缩放过程中会被多次调用

> func scrollViewWillBeginZooming(_ scrollView: UIScrollView, with view: UIView?)
scrollView开始缩放时调用

> func scrollViewDidEndZooming(_ scrollView: UIScrollView, with view: UIView?, atScale scale: CGFloat)
scrollView结束缩放时调用

> func scrollViewShouldScrollToTop(_ scrollView: UIScrollView) -> Bool
是否允许点击scrollview的头部，让其滚动到最上面,若不实现此代理，则默认为true

> func scrollViewDidScrollToTop(_ scrollView: UIScrollView)
当滚动到最上面时调用

## 10 视频背景

### 10.1 效果
![](./iOS-swift-3%E5%A4%A930%E4%B8%AASwift%E9%A1%B9%E7%9B%AE%E4%B9%8B%E7%AC%AC%E4%B8%80%E5%A4%A9/videobg.gif)
<img src=videobg.gif>

### 10.2 定义一个基类播放控制器

```Swift
public enum ScalingMode {
  case resize
  case resizeAspect
  case resizeAspectFill
}
```
定义了3种缩放模式。

再定义有关视频的属性：
```Swift
// 视频播放器
fileprivate let moviePlayer = AVPlayerViewController()
     
  // 声音   
  fileprivate var moviePlayerSoundLevel: Float = 1.0

   // 播放url  
   var contentURL: URL? {
    didSet {
      if let _contentURL = contentURL {
      setMoviePlayer(_contentURL)
      }
    }
  }

   var videoFrame: CGRect = CGRect()
   var startTime: CGFloat = 0.0
   var duration: CGFloat = 0.0

   // 背景
   var backgroundColor: UIColor = UIColor.black {
    didSet {
      view.backgroundColor = backgroundColor
    }
  }

  // 声音
   var sound: Bool = true {
    didSet {
      if sound {
        moviePlayerSoundLevel = 1.0
      }else{
        moviePlayerSoundLevel = 0.0
      }
    }
  }
  
  // alpha
   var alpha: CGFloat = CGFloat() {
    didSet {
      moviePlayer.view.alpha = alpha
    }
  }

  // 是否重复 这里发送一个全局通知
   var alwaysRepeat: Bool = true {
    didSet {
      if alwaysRepeat {
        NotificationCenter.default.addObserver(self,
          selector: #selector(VideoSplashViewController.playerItemDidReachEnd),
          name: NSNotification.Name.AVPlayerItemDidPlayToEndTime,
          object: moviePlayer.player?.currentItem)
      }
    }
  }

  // 填充模式
   var fillMode: ScalingMode = .resizeAspectFill {
    didSet {
      switch fillMode {
      case .resize:
          moviePlayer.videoGravity = AVLayerVideoGravity.resize.rawValue
      case .resizeAspect:
          moviePlayer.videoGravity = AVLayerVideoGravity.resizeAspect.rawValue
      case .resizeAspectFill:
          moviePlayer.videoGravity = AVLayerVideoGravity.resizeAspectFill.rawValue
      }
    }
  }
```

生命周期已经显示,添加子View和视频view：
```Swift
  override func viewDidAppear(_ animated: Bool) {
    moviePlayer.view.frame = videoFrame
    moviePlayer.showsPlaybackControls = false
    view.addSubview(moviePlayer.view)
    view.sendSubview(toBack: moviePlayer.view)
  }
```

生命周期将要消失，移除通知：
```Swift
  override func viewWillDisappear(_ animated: Bool) {
    super.viewDidDisappear(animated)
    NotificationCenter.default.removeObserver(self)
  }
```

设置播放器：
```Swift
 fileprivate func setMoviePlayer(_ url: URL){
    let videoCutter = VideoCutter()
    videoCutter.cropVideoWithUrl(videoUrl: url, startTime: startTime, duration: duration) { (videoPath, error) -> Void in
      if let path = videoPath as URL? {
        DispatchQueue.main.async {
            self.moviePlayer.player = AVPlayer(url: path)
            self.moviePlayer.player?.play()
            self.moviePlayer.player?.volume = self.moviePlayerSoundLevel
        }
      }
    }
  }
```

播放视频，前面监听视频播放完成消息，然后走这个方法，可实现重复播放效果：
```Swift
  @objc func playerItemDidReachEnd() {
    moviePlayer.player?.seek(to: kCMTimeZero)
    moviePlayer.player?.play()
  }
```

### 10.3 实现视频背景效果控制器

```Swift
 @IBOutlet weak var loginButton: UIButton!
    @IBOutlet weak var signupButton: UIButton!
```
这里搞2个View。

```Swift
override func viewDidLoad() {
    super.viewDidLoad()
    setupVideoBackground()
    
    loginButton.layer.cornerRadius = 4
    signupButton.layer.cornerRadius = 4
    
}

func setupVideoBackground() {
    
    let url = URL(fileURLWithPath: Bundle.main.path(forResource: "moments", ofType: "mp4")!)
    
    videoFrame = view.frame
    fillMode = .resizeAspectFill
    alwaysRepeat = true
    sound = true
    startTime = 2.0
    alpha = 0.8
    
    contentURL = url
    // view.isUserInteractionEnabled = false
    
}
```
这里初始化，设置视频背景，设置按钮圆角。

### 10.4 视频剪切工具
前面设置播放器，用到了一个VideoCutter类，
这里：
```Swift
 fileprivate func setMoviePlayer(_ url: URL){
    let videoCutter = VideoCutter()
    videoCutter.cropVideoWithUrl(videoUrl: url, startTime: startTime, duration: duration) { (videoPath, error) -> Void in
      if let path = videoPath as URL? {
        DispatchQueue.main.async {
            self.moviePlayer.player = AVPlayer(url: path)
            self.moviePlayer.player?.play()
            self.moviePlayer.player?.volume = self.moviePlayerSoundLevel
        }
      }
    }
  }
```
这里其实是根据startTime和duration来剪切目标视频url，返回一个新的videoPath，这里再重新播放的。

需要看下如何剪切的：
```Swift
extension String {
  var convert: NSString { return (self as NSString) }
}

 class VideoCutter: NSObject {

  /**
  Block based method for crop video url
  
  @param videoUrl Video url
  @param startTime The starting point of the video segments
  @param duration Total time, video length

  */
   func cropVideoWithUrl(videoUrl url: URL, startTime: CGFloat, duration: CGFloat, completion: ((_ videoPath: URL?, _ error: NSError?) -> Void)?) {
    DispatchQueue.global(qos: DispatchQoS.QoSClass.default).async {
        let asset = AVURLAsset(url: url, options: nil)
        let exportSession = AVAssetExportSession(asset: asset, presetName: "AVAssetExportPresetHighestQuality")
        let paths: NSArray = NSSearchPathForDirectoriesInDomains(.documentDirectory, .userDomainMask, true) as NSArray
        var outputURL = paths.object(at: 0) as! String
        let manager = FileManager.default
        // 异常处理：
        // try?: 将错误转化成可选异常,有错误发生时返回nil
        // try!: 禁用错误传递，当确认无异常发生时使用，否则可能会发生运行时异常
        // defer: 在即将离开当前代码块时执行一系列语句,延迟执行的语句不能包含任何控制转移语句，例如break、return语句，或是抛出一个错误。
        // defer语句是从后往前执行
        do {
            try manager.createDirectory(atPath: outputURL, withIntermediateDirectories: true, attributes: nil)
        } catch _ {
        }
        outputURL = outputURL.convert.appendingPathComponent("output.mp4")
        do {
            try manager.removeItem(atPath: outputURL)
        } catch _ {
        }
        if let exportSession = exportSession as AVAssetExportSession? {
            exportSession.outputURL = URL(fileURLWithPath: outputURL)
            exportSession.shouldOptimizeForNetworkUse = true
            exportSession.outputFileType = AVFileType.mp4
            let start = CMTimeMakeWithSeconds(Float64(startTime), 600)
            let duration = CMTimeMakeWithSeconds(Float64(duration), 600)
            let range = CMTimeRangeMake(start, duration)
            exportSession.timeRange = range
           
            exportSession.exportAsynchronously {
                switch exportSession.status {
                case AVAssetExportSessionStatus.completed:
                    completion?(exportSession.outputURL, nil)
                case AVAssetExportSessionStatus.failed:
                    print("Failed: \(String(describing: exportSession.error))")
                case AVAssetExportSessionStatus.cancelled:
                    print("Failed: \(String(describing: exportSession.error))")
                default:
                    print("default case")
                }
            }
        }
    }
  }
}
```
这是个工具，简单看下就行。


## 11 总结
* 1.第一个是计时器demo，界面很简单，主要学会Timer用法，每次继续其实也是new了一个Timer，Timer的作用只是更新，数值是我们自己记录的。
* 2.第二个改变字体，主要学会字体文件放在根目录，xCode会自动识别，然后设置字体可以通过UILabel的Font属性，其次学会UiTableView用法，懂得设置数据源和代理。
* 3.本地视频，当然就是学会AVPlayer的用法，还要用到系统的一个控制器。
* 4.摄像头，主要学会Session用法，input，outPut设置，通过设置AV预览图设置给UIView，达成预览效果，另外学会水平ScrollView可实现类似分页效果，当然坐标要按照屏幕宽度设置下。
* 5.传送带效果，学会设置UICollectionView和UIScrollView的代理，做到良好的用户体验效果，滑动到半屏和不足半屏的考虑。另外学会了xib布局方式，可以绑定控制器。
* 6.定位，学会使用系统提供的定位Api，在代理里面处理自己的逻辑。
* 7.下拉刷新，是UIScrollView里面的refreshControl提供的功能，自带了loading效果，可以自行添加文案。
* 8.炫彩音乐，主要学会音频播放，以及随机函数的用法，还有渐变色调用方式。
* 9.图片缩放效果，这个主要是UIScrollView的子View这种方式提供的缩放效果，主要还是针对代理方法做一些调整。
* 10.视频背景，这个主要就是学会构造一个基类控制器，可实现播放视频效果，通过配置不同参数丰富这个基类，然后另外就是学会视频剪切。










