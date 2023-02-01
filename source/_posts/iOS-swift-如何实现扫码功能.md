---
title: iOS swift 如何实现扫码功能
date: 2023-01-29 08:13:37
top: false
cover: false
toc: true
mathjax: true
tags:
- 条形码识别 相机
categories:
- iOS
---

## 1 需求定义

这里有个需求，就是继承扫码能力，可以识别到条形码里面的内容。
首先我们需要下载一个离线库，这个库里面会包含很多条码Code，相机通过识别到条码跟离线库中的匹配，如果匹配上了，会提示用户。所以首先我们肯定要有一个识别能力，数据库先不管，后面会有专门的文章来写数据库相关的东西。

本篇文章主要是从0搭建一个可以识别条形码能力的Controller。
其它额外需求先忽略。

目标就是完成这样的效果：
<img src=scan1.png>
<img src=scan2.png>

## 2 分析需求

要具备扫码功能，肯定离不开相机，调用相机会设计到权限。
所以我们得考虑是否有相机权限。
其次，得考虑如何来识别条形码。
重复识别到的问题。
页面退出，相机资源怎么处理。
光线太暗了，是否需要打开手电筒。
条码太小，是否需要放大。
等等，这些事先都得考虑到。


## 3 打造页面

### 3.1 构建UI

首先把材料上齐。
```Swift
lazy var squareImgView: UIImageView = {
        let imgv = UIImageView()
        imgv.isUserInteractionEnabled = false
        imgv.image = UIImage(named: "扫码框")
        return imgv
    }()
    
    lazy var torchButton: UIButton = {
        let temp = UIButton(type: .custom)
        temp.setImage(UIImage(named: "scan_torch"), for: .normal)
        temp.setTitle("轻触照亮", for: .normal)
        temp.isHidden = true
        temp.titleLabel?.font = UIFont.regular(15)
        temp.ddy_SetStyle(.imgDown, padding: 10)
        temp.addTarget(self, action: #selector(torchButtonClick), for: .touchUpInside)
        return temp
    }()
    
    lazy var photoBtn: UIButton = {
        let btn = UIButton(type: .custom)
        btn.setImage(UIImage(named: "PhotoAlbum"), for: .normal)
        btn.addTarget(self, action: #selector(actionForPhoto), for: .touchUpInside)
        return btn
    }()
    
    lazy var tipsLabel: UILabel = {
        let temp = UILabel()
        temp.text = "扫一扫条形码"
        temp.textColor = .white
        temp.font = UIFont.regular(13)
        temp.sizeToFit()
        return temp
    }()
    
    lazy var commitButton: UIButton = {
        let btn: UIButton = UIButton(type: .custom)
        btn.backgroundColor = UIColor(hex: "#242424")
        btn.clipsToBounds = true
        btn.layer.cornerRadius = 22
        btn.layer.borderWidth = 1
        btn.layer.borderColor = UIColor.white.cgColor
        btn.setTitle("扫描完成", for: .normal)
        btn.addTarget(self, action: #selector(commitButtonClick), for: .touchUpInside)
        return btn
    }()
    
    lazy var inputButton: UIButton = {
        let btn: UIButton = UIButton(type: .custom)
        btn.backgroundColor = UIColor(hex: "#242424")
        btn.layer.borderColor = UIColor.white.cgColor
        btn.setTitle("手动输入条形码", for: .normal)
        btn.addTarget(self, action: #selector(actionForPushManuallyInput), for: .touchUpInside)
        return btn
    }()

    lazy var layerView: UIView = {
        let view = UIView(frame: CGRect(x: 0,
                                        y: STATUS_BAR_HEIGHT + 44,
                                        width: ScreenWidth,
                                        height: ScreenHeight - STATUS_BAR_HEIGHT - 44 - 162))
        return view
    }()
```
打开需要这几个View。

### 3.2 全局变量声明

想要具备扫码能力，有几个类是必须的。
```Swift
// AVC 相关  相机硬件接口相关
    private var videoDataOutput: AVCaptureVideoDataOutput?
    private var output: AVCaptureMetadataOutput?
    private var session: AVCaptureSession?
    private var videoPreviewLayer: AVCaptureVideoPreviewLayer?
```

其它成员不着急，后续需要再加上去。

### 3.3 生命周期之loadView

loadView执行时机为：访问视图控制器的view时候，如何view为nil，或者还没有加载完成就会调用loadView方法来创建view。可以理解成先于viewDidLoad，一般走一次，一般用于视图初始化。

这里就负责 添加子View 的工作。
```Swift
private func setupUI() {
        view.addSubview(layerView)
        view.addSubview(squareImgView)
        view.addSubview(torchButton)
        view.addSubview(tipsLabel)
        view.addSubview(commitButton)
        view.addSubview(inputButton)
        view.addSubview(photoBtn)

        squareImgView.snp.makeConstraints { make in
            make.size.equalTo(320)
            make.centerX.equalToSuperview()
            make.top.equalTo(STATUS_BAR_HEIGHT + 44 + 76)
        }
        
        torchButton.snp.makeConstraints { make in
            make.centerX.equalToSuperview()
            make.bottom.equalTo(squareImgView.snp.bottom).offset(20)
            make.width.equalTo(320)
            make.height.equalTo(200)
        }
        
        tipsLabel.snp.makeConstraints { make in
            make.centerX.equalToSuperview()
            make.top.equalTo(squareImgView.snp.bottom).offset(12)
        }
        
        commitButton.snp.makeConstraints { make in
            make.top.equalTo(layerView.snp.bottom).offset(42)
            make.leading.equalTo(102)
            make.trailing.equalTo(-102)
            make.height.equalTo(44)
        }
        
        inputButton.snp.makeConstraints{ make in
            make.top.equalTo(commitButton.snp.bottom).offset(12)
            make.left.equalTo(commitButton)
            make.right.equalTo(commitButton)
            make.height.equalTo(36)
        }
        
        photoBtn.snp.makeConstraints{ make in
            make.centerY.equalTo(commitButton)
            make.leading.equalTo(16)
            make.width.height.equalTo(44)
        }
    }
```
可以看到，这里把架子搭上去了。

### 3.4 生命周期之 viewDidLoad

看下这里做了什么：
```Swift
 override func viewDidLoad() {
        super.viewDidLoad()
        // 标题文字大小
        navigationController?.navigationBar.navBarBackGroundColor(.black, image: UIImage(), isOpaque: false)
        navigationController?.navigationBar.titleTextAttributes = [NSAttributedString.Key.foregroundColor:UIColor.white, NSAttributedString.Key.font : UIFont.pingFangMedium(size: 18)]
        
        // 背景黑色
        view.backgroundColor = UIColor(hex: "#242424")

        // 添加手势，用来实现二指缩放效果
        addGesture()
        
        // 监听全局通知
        addNotification()
        
        // 自己定义的协议，去加载历史数据
        delegate?.loadHistoryData()
        
        // 手动输入条码闭包回调
        manuallyInputViewController.inputBlock = { [unowned self] (_, code) in
            self.delegate?.manuallyInput(code.replaceFlag)
        }
    }
```

如何添加手势呢？
```Swift
private func addGesture() {
        let prinGesture = UIPinchGestureRecognizer(target: self, action: #selector(pinch(gesture:)))
        layerView.addGestureRecognizer(prinGesture)
    }
```

操作后会触发：
```Swift
@objc func pinch(gesture: UIPinchGestureRecognizer) {
        guard let device = AVCaptureDevice.default(for: AVMediaType.video) else {
            print("get front video AVCaptureDeviceInput  failed!")
            return
        }
        var minZoomFactor = 1.0
        var maxZoomFactor = device.activeFormat.videoMaxZoomFactor
        
        if #available(iOS 11.0,*) {
            minZoomFactor = device.minAvailableVideoZoomFactor
            maxZoomFactor = device.maxAvailableVideoZoomFactor
        }
        
        if gesture.state == .began {
            lastZoomFactor = device.videoZoomFactor
            print("缩放比例  \(device.videoZoomFactor)")
            
        } else if gesture.state == .changed{
            var zoomFactor: Float = Float(lastZoomFactor * gesture.scale);
            zoomFactor = fmaxf(fminf(zoomFactor, Float(maxZoomFactor)), Float(minZoomFactor))
            print("缩放比例  \(zoomFactor)")
            try? device.lockForConfiguration()
            device.videoZoomFactor = CGFloat(zoomFactor)
            device.unlockForConfiguration()
        } else if gesture.state == .ended {
            print("最终缩放比例  \(device.videoZoomFactor)")
        }
    }
```

其它的没啥了。

### 3.5 生命周期之viewWillAppear

这个是将要显示，一般做轻量级操作。
```Swift
override func viewWillAppear(_ animated: Bool) {
        super.viewWillAppear(animated)
        setupRealWhiteClickCallBack()
        setupBlackNavWhiteTitleBarColor()
        isHiddenNavBar(isHidden: false)
    }
```
主要实现了状态栏和标题栏相关的。

### 3.6 生命周期之viewDidAppear

这里是已经显示了，这里可以做稍微重量级代码。
```Swift
override func viewDidAppear(_ animated: Bool) {
        super.viewDidAppear(animated)
        if self.session == nil {
            self.scanQRCodePermission()
        } else {
            self.startScan()
        }
        UIApplication.shared.isIdleTimerDisabled = true
        setNeedsStatusBarAppearanceUpdate()
    }
```
很明显这里判断了下session是否为null。

如果session为nil，则先判断是否有相机权限：
```Swift
func scanQRCodePermission() {
        // 判断摄像头是否可用
        let available = UIImagePickerController.isSourceTypeAvailable(.camera)
        if available {
            
            // 判断是否有相机权限
            let authStatus: AVAuthorizationStatus = AVCaptureDevice.authorizationStatus(for: AVMediaType.video)
            if authStatus == .restricted {
                let alerVC: UIAlertController = .init(title: "提示", message: "由于系统原因, 无法访问相机", preferredStyle: UIAlertController.Style.alert)
                alerVC.addAction(UIAlertAction.init(title: "确定", style: .destructive) { (action) in
                    UIApplication.shared.open(NSURL(string: UIApplication.openSettingsURLString)! as URL, options: [:] ) { (success) in
                    }
                })
                present(alerVC, animated: true, completion: nil)
            } else if authStatus == .denied {
                let dic = Bundle.main.infoDictionary
                var appName = dic?["CFBundleDisplayName"]
                if appName == nil{
                    appName = dic?["CFBundleName"]
                }
                let str = String.localizedStringWithFormat("[前往：设置 - 隐私 - 相机 - %@] 允许应用访问", appName as! String)
                let alerVC = UIAlertController(title: "提示", message: str, preferredStyle: .alert)
                alerVC.addAction(UIAlertAction(title: "确定", style: UIAlertAction.Style.destructive) { (action) in
                    UIApplication.shared.open(NSURL(string: UIApplication.openSettingsURLString)! as URL, options: [:] ) { (success) in
                    }
                })
                present(alerVC, animated: true, completion: nil)
            } else if authStatus == .notDetermined {
                //用户未作出选择，申请权限 AVC直接requestAccess
                AVCaptureDevice.requestAccess(for: AVMediaType.video){
                    granted in
                    DispatchQueue.main.async {
                        if granted {
                            //初次授权成功
                            self.initCaptureSession()
                        } else {
                            //拒绝授权
                        }
                    }
                }
            } else {
                //已授权
                self.initCaptureSession()
            }
        } else {
            //摄像头可能已损坏
        }
    }
```
上面的代码讲述了如何申请权限，权限被拒绝如何跳转设置。

如果有权限了会走 initCaptureSession 来初始化session:
```Swift
func initCaptureSession() {
        //创建会话对象
        let `session` = AVCaptureSession()
        self.session = session
        session.sessionPreset = .high
        
        //获取设想设备
        guard let device = AVCaptureDevice.default(for: AVMediaType.video) else {
            print("get front video AVCaptureDeviceInput  failed!")
            return
        }
        
        //创建设想设备输入流
        guard let input = try? AVCaptureDeviceInput(device: device) else {
            print("get front video AVCaptureDeviceInput  failed!")
            return
        }
        
        //添加设想设备输入流到会话对象 输入流让session来分析
        if session.canAddInput(input) {
            session.addInput(input)
        }
        
        //创建元数据输出流
        let `output` = AVCaptureMetadataOutput()
        self.output = output
        output.setMetadataObjectsDelegate(self, queue: DispatchQueue.main)
                
        //添加元数据输出流到会话对象 给新建的一个Output对象添加到session,session等下会给output赋值
        session.addOutput(output)
        
        //创建摄像数据输出流并将其添加到会话对象上----->用户识别光线强弱
        let `videoDataOutput` = AVCaptureVideoDataOutput()
        self.videoDataOutput = videoDataOutput
        videoDataOutput.setSampleBufferDelegate(self, queue: DispatchQueue.main)
        session.addOutput(videoDataOutput)
        
        //设置数据输出类型(如下设置为条形码和二维码兼容)，需要将数据输出添加到会话后，才能指定元数据类型，否则会报错
        
        /*
         
         android
         
         Barcode.FORMAT_CODE_128,   *
         Barcode.FORMAT_CODE_39,    *
         Barcode.FORMAT_CODE_93,    *
         Barcode.FORMAT_CODABAR,    ios 15.4 +
         Barcode.FORMAT_EAN_13,     *
         Barcode.FORMAT_EAN_8,      *
         Barcode.FORMAT_UPC_A,      x
         Barcode.FORMAT_UPC_E,      *
         Barcode.TYPE_ISBN          x
         */
        output.metadataObjectTypes = [.code39,
                                      .code39Mod43,
                                      .code93,
                                      .code128,
                                      .ean8,
                                      .ean13,
                                      .upce,
                                      .pdf417] //itf14
        //实例化预览图层, 用于显示会话对象 session放到这里面，然后通过layer插入sublayer可以实现预览效果
        let `videoPreviewLayer` = AVCaptureVideoPreviewLayer(session: session)
        self.videoPreviewLayer = videoPreviewLayer
        videoPreviewLayer.videoGravity = .resizeAspectFill
        videoPreviewLayer.frame = layerView.bounds
        layerView.layer.insertSublayer(videoPreviewLayer, at: 0)
        
        // session开始run
        startScan()
    }
```

回到 生命周期 中，如果 session 不为空，会走上面的startScan 函数，这里就是让session继续工作的意思：
```Swift
func startScan() {
        session?.startRunning()
    }
```

在viewDidAppear中还做了什么呢？
```Swift
UIApplication.shared.isIdleTimerDisabled = true
setNeedsStatusBarAppearanceUpdate()
```

第一行：禁止该页面进入睡眠；
第二行：如果在viewController已经显示在当前页面，你可能还要在当前页面不时的更改statusBar的前景色，那么，你首先需要调用下面的setNeedsStatusBarAppearanceUpdate方法(这个方法会通知系统去调用当前UIViewController的preferredStatusBarStyle方法)，从而改变statusBar的statusBarStyle。

### 3.7 生命周期之viewDidDisappear

```Swift
 override func viewDidDisappear(_ animated: Bool) {
        super.viewDidDisappear(animated)
        stopScan()
        UIApplication.shared.isIdleTimerDisabled = false
        setNeedsStatusBarAppearanceUpdate()
    }
```
这里停止扫描，应该是停止session:
```Swift
func stopScan() {
        session?.stopRunning()
    }
```

其它上面讲过，不必多言了。

### 3.8 设置output代理

回到初始化session的部分代码中：
```Swift
//创建元数据输出流
let `output` = AVCaptureMetadataOutput()
self.output = output
output.setMetadataObjectsDelegate(self, queue: DispatchQueue.main)
```
这里设置了一个metadata的代理，所以这个controller必然扩展了这个代理。

具体如下：
```Swift
extension GMBaseScanViewController: AVCaptureMetadataOutputObjectsDelegate {
    func metadataOutput(_ output: AVCaptureMetadataOutput, didOutput metadataObjects: [AVMetadataObject], from connection: AVCaptureConnection) {
        if isCanScan {
            let isAchieve: Bool = metadataObjects.count == 1
            if isAchieve {
                // 扫到条码
                guard let device = AVCaptureDevice.default(for: AVMediaType.video) else { return }
                guard let codeObj = metadataObjects.last as? AVMetadataMachineReadableCodeObject else { return }
                print("扫到了条码 codeObj  ：  \(codeObj)")
                guard let str = codeObj.stringValue else { return }
                self.isCanScan = false
                print("扫到了条码 str：  \(str)")
                DispatchQueue.main.asyncAfter(deadline: .now() + scanInterval) {
                    try? device.lockForConfiguration()
                    device.videoZoomFactor = 1
                    device.unlockForConfiguration()
                    self.isCanScan = true
                }
                let barCode = barcodeProcessing(str)
                delegate?.scanBarcode(barCode)
            } else {
                MBProgressHUD.showTipsMessage("无法识别")
            }
        }
    }
    
    func barcodeProcessing(_ code: String) -> String {
        var codeStr: String = code.mutableCopy() as! String
        var newStr = code.uppercased()
        var index = 0
        newStr = newStr.replacingOccurrences(of: " ", with: "", options: .literal, range: nil)
        codeStr = codeStr.replacingOccurrences(of: " ", with: "", options: .literal, range: nil)
        if newStr.hasPrefix("GREE") {
            index = 3
        } else if newStr.hasPrefix("KINGHOME") {
            index = 7
        } else if newStr.hasPrefix("TOSOT") {
            index = 4
        }
        if index != 0 {
            let start = codeStr.index(codeStr.startIndex, offsetBy: 0)
            let end = codeStr.index(codeStr.startIndex, offsetBy: index)
            codeStr.removeSubrange(start...end)
        }
        return codeStr
    }
}
```
这里覆写了这协议方法，表示识别到内容了。然后会回调metadateOutput，原因是前面session里面设置了Output的代理为self,也就是这里，这里就关联起来了。
前面是这样的：
```Swift
 let `output` = AVCaptureMetadataOutput()
        self.output = output
        output.setMetadataObjectsDelegate(self, queue: DispatchQueue.main)
                
        //添加元数据输出流到会话对象 给新建的一个Output对象添加到session,session等下会给output赋值
        session.addOutput(output)
```
就是因为这个self，当session开始running后，就会走协议的回调方法，在那里面我们间隔3.5才让它进行下次扫描，也是在这里面分析这个条码是否正常之类的。

### 3.9 设置videoDataOuput代理

这个代理，目的是为了识别光线强弱。
```Swift
extension GMBaseScanViewController: AVCaptureVideoDataOutputSampleBufferDelegate {
    
    func captureOutput(_ output: AVCaptureOutput, didOutput sampleBuffer: CMSampleBuffer, from connection: AVCaptureConnection) {
        // 获取光照亮度
        guard let metadataCFDict: CFDictionary = CMCopyDictionaryOfAttachments(allocator: nil, target: sampleBuffer, attachmentMode: 1) else { return }
        guard let metadataDic: [String: AnyObject] = metadataCFDict as? [String: AnyObject] else { return }
        guard let exifMetadata: [String: AnyObject] = metadataDic[kCGImagePropertyExifDictionary as String] as? [String : AnyObject] else { return }
        guard let brightnessValue: CGFloat = exifMetadata[kCGImagePropertyExifBrightnessValue as String] as? CGFloat else { return }
        // 根据光照亮度展示按钮
        if brightnessValue < -1 {
            torchButton.isHidden = false
        } else {
            guard let device = AVCaptureDevice.default(for: AVMediaType.video) else { return }
            if device.hasTorch && device.isTorchAvailable {
                try? device.lockForConfiguration()
                torchButton.isHidden = !(device.torchMode == .on)
                device.unlockForConfiguration()
                
            }
        }
        
    }
    
    private func convert(cmage: CIImage) -> UIImage {
         let context = CIContext(options: nil)
         let cgImage = context.createCGImage(cmage, from: cmage.extent)!
         let image = UIImage(cgImage: cgImage)
         return image
    }
    
}
```
在代理 captureOutput的回调方法中可以获取光线强弱信息。

也是在初始化session的时候设置了这个代理。

### 3.10 识别到后播放声音

这里我们额外增加一个方法，主要给外面调用，因为我们会提供一个代理，如果识别到后，通过代理内部方法将识别的条码给外面。这时候外面要震动或声音效果，只需要调用我们内部的一个方法即可。

```Swift
func playAudio(success code: Int) {
        if isPlay == false {
            return
        }
        if code == 1 {
            var soundID: SystemSoundID = 0
            guard let path = Bundle.main.path(forResource: "scanSuccess", ofType: "mp3") else { return }
            let url = URL(fileURLWithPath: path)
            AudioServicesCreateSystemSoundID(url as CFURL, &soundID)
            AudioServicesPlaySystemSound(soundID) //声音
        } else {
            var soundID: SystemSoundID = 0
            guard let path = Bundle.main.path(forResource: "scanFail", ofType: "mp3") else { return }
            let url = URL(fileURLWithPath: path)
            AudioServicesCreateSystemSoundID(url as CFURL, &soundID)
            AudioServicesPlayAlertSound(soundID) //震动+播放声音
        }
    }
```
也是非常简单，只需要先拿到声音路径，转换url，再通过AudioServicesCreateSystemSoundID包装一下，再通过AudioServicesPlaySystemSound来播放这个soundID即可。

就这样，一个具有扫码能力的Controller就出来了哦。

## 4 总结

* 如果用到扫码，一定需要申请权限的，必须考虑到是否有权限，如果拒绝了权限，或其它原因导致没有权限，都需要考虑。

* 扫码关键的几个类，AVCaptureSession控制数据流关键类；AVCaptureMetadataOutput接收流，代理方法来处理识别到条码后的逻辑；AVCaptureVideoDataOutput接收流，可以处理光线强弱；AVCaptureVideoPreviewLayer用来预览相机的视图层，可显示在UIView的layer上。

* 一般扫码需要具备的二指缩放，光线太暗打开闪光灯，这些可以算作基本能能力，务必要实现，否则体验很不佳。

* 合理控制识别到条码间隔时间，该停止session的时候务必停止，为了防止频繁识别到相同条码，可以设置间隔时间。

* 可以先定义一个播放声音效果的方法，方便外部直接调用方法播放成功或者失败的声音。声音主要用到AudioServices相关方法。

* 系统类大多都是提供了代理，暴露了一些方法主要都是定义好的方法，我们业务开发也只需要接收和处理这些定义好的类就可以了。
