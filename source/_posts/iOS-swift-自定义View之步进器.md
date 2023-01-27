---
title: iOS swift 自定义View之步进器
date: 2023-01-26 10:54:54
top: false
cover: false
toc: true
mathjax: true
tags:
- iOS 自定义View
categories:
- iOS
---

## 1 需求分析
 需要实现一个步进器效果，比如我们在购买商品时，需要添加这个商品，可以点击+或者-，也可以手动输入购买数量。
 效果如下：
 
 <img src="stepper0.png">
 <img src="stepper5.png">

 基本交互很简单，有最小值和最大值，点击减号就将数量减1，点击加号就将数量加1，如果是最大值或者最小值，就将图标置灰不可点击。

## 2 代码实现

### 2.1 类外声明

```
/// 定义一个闭包，主要是需要暴露里面的数字给外面，可以理解成定义一个接口类型
public typealias ResultClosure = (_ number: String)->()
```

这个主要是给调用步进器的地方使用，当用户行为导致数量变更，肯定要通知外部处理自己逻辑，这是封装自定义View的基本常识，一定要给外部一定的可操作空间。

### 2.2 创建步进器类
```
/// 这里是自定义步进器了  @IBDesignable关键字用来声明一个类是可以被设计的，可以实时渲染在interface builder 上
/// @IBInspectable关键字用来声明一个属性，可以在interface builder上修改该属性，就可以实时渲染border的变化
/// open 修饰的 class 在 Module 内部和外部都可以被访问和继承
@IBDesignable open class PPNumberButton: UIView {
```
这里继承了UIView，这个注解只是用来实时预览，这里不要也行。如果想了解自定义View实时预览功能，可以参考下这篇文档：[https://blog.wangruofeng007.com/posts/56184/]()

### 2.3 定义类属性
```
    /// 0为默认，1为零售开单
    var documentType:Int = 0
    
    /// 结果闭包，用自定义的闭包类型
    var NumberResultClosure: ResultClosure?
    
    var decreaseBtn: UIButton!     // 减按钮
    var increaseBtn: UIButton!     // 加按钮
    var textField: UITextField!    // 数量展示/输入框
    var timer: Timer!              // 快速加减定时器
    public var _minValue = 1                 // 最小值
    public var _maxValue = Int.max           // 最大值
    public var shakeAnimation: Bool = false  // 是否打开抖动动画
    
    var decreaseBgBtn:UIButton!  //减按钮(增大可触面积)
    var increaseBgBtn:UIButton!  //加按钮(增大可触面积)
    
    /// rxSwift生命的袋子
    let bag = DisposeBag()
```

这里documentType是自己的业务逻辑，忽略。结果闭包2.1声明的那个类型。然后就是几个基本视图了。这里还有个timer，主要用途是长按实现持续增加数字的功能，体验会好一点。
最大值最小值就是步进器的最大值和最小值。
抖动动画，是如果已经到最大值了，用户还在按加，就弹一个动画效果。
然后还有两个UIButton覆盖在增加和减号的图标上面，只是为了扩大点击热区。

### 2.4 生命周期函数

```
/// UIView的初始化函数，这里一般会覆写这个方法
override public init(frame: CGRect) {
    super.init(frame: frame)
    
    setupUI()
    
    //整个控件的默认尺寸
    if frame.isEmpty {
        self.frame = CGRect(x: 0, y: 0, width: 110, height: 30)
    }
    
}

required public init?(coder aDecoder: NSCoder) {
    super.init(coder: aDecoder)

}
```
一般情况，自定义View都需要覆写下这两个init函数。用来初始化UI。

```
//设置UI布局
fileprivate func setupUI() {
    /// 背景颜色清空
    backgroundColor = UIColor.clear

    /// 加图标和热区加按钮
    decreaseBtn = setupButton(title: "reduce_available")
    decreaseBgBtn = setupButton(title: " ")
    increaseBtn = setupButton(title: "increase_available")
    increaseBgBtn = setupButton(title: " ")
    
    decreaseBtn.backgroundColor = .clear
    increaseBtn.backgroundColor = .clear
    //减按钮(增大可触面积)
    decreaseBgBtn.backgroundColor = .clear
    //加按钮(增大可触面积)
    increaseBgBtn.backgroundColor = .clear
    
    /// 中间区域的编辑框
    textField = UITextField.init()
    textField.font = UIFont.pingFangMedium(size: 16)
    textField.layer.borderColor = UIColor(red: 0.79, green: 0.80, blue: 0.83, alpha: 1.00).cgColor
    textField.layer.borderWidth = 1
    textField.backgroundColor = UIColor(red: 0.97, green: 0.97, blue: 0.99, alpha: 1.00)
    textField.layer.cornerRadius = 8
    textField.textColor = UIColor(hex:"#3B4058")
    textField.delegate = self
    textField.adjustsFontSizeToFitWidth = true        //当文字超出文本框宽度时，自动调整文字大小
    textField.minimumFontSize = 14                   //最小可缩小的字号
    textField.keyboardType = UIKeyboardType.numberPad
    textField.textAlignment = NSTextAlignment.center
    self.addSubview(textField)
}
```

内部用了一个setupButton给按钮增加背景，看下哈：
```
//设置加减按钮的公共方法 设置action
fileprivate func setupButton(title:String) -> UIButton {
    let button = UIButton.init();
    button.setBackgroundImage(UIImage.init(named: title), for: .normal)
    button.setTitleColor(UIColor.gray, for: .normal)
    button.addTarget(self, action:#selector(self.touchDown(_:)) , for: .touchDown)
    button.addTarget(self, action:#selector(self.touchUp) , for:.touchUpOutside)
    button.addTarget(self, action:#selector(self.touchUp) , for:.touchUpInside)
    button.addTarget(self, action:#selector(self.touchUp) , for:.touchCancel)
    self.addSubview(button)
    
    return button;
}
```

这里给按钮增加了一个action，主要是处理用户行为点击加号或者减号的逻辑：
```
@objc fileprivate func touchDown(_ button: UIButton) {
        textField.endEditing(false)
        if button == decreaseBtn || button == decreaseBgBtn {
            timer = Timer.scheduledTimer(timeInterval: 0.15, target: self, selector: #selector(self.decrease), userInfo: nil, repeats: true)
        } else {
            timer = Timer.scheduledTimer(timeInterval: 0.15, target: self, selector: #selector(self.increase), userInfo: nil, repeats: true)
        }
        timer.fire()
    }
```
这里给按钮添加的action的函数，都需要@objc注解，原因是swift是静态编程语言，objc是动态的，所以这里需要转换为动态效果，所以要声明这个注解，不知道我理解得对不对。

这里只有点击了就去执行加或者减的逻辑，用了一个定时器，保证150ms才走一次。
加的逻辑看下哈：
```
@objc fileprivate func increase() {
        if (textField.text?.count)! == 0 || Int(textField.text!)! <= _minValue {
            textField.text = "\(_minValue)"
        }
        
        let number = Int(textField.text!)! + 1;
        
        if number <= _maxValue {
            textField.text = "\(number)";
            //闭包回调
            NumberResultClosure?("\(number)")
        } else {
            //添加抖动动画
            if shakeAnimation {shakeAnimationFunc()}
            print("已超过最大数量\(_maxValue)");
            if documentType == 1 {
                MBProgressHUD.showTipsMessage("数量已超出零售上限")
            }
        }
        /// 触发rxSwift监听处更新
        textField.sendActions(for: .valueChanged)
        
        if Int(textField.text!)! >= maxValue {
            decreaseBtn.setBackgroundImage(UIImage.init(named: "reduce_available"), for: .normal)
            increaseBtn.setBackgroundImage(UIImage.init(named: "increase_Disable"), for: .normal)
        }else{
            decreaseBtn.setBackgroundImage(UIImage.init(named: "reduce_available"), for: .normal)
            increaseBtn.setBackgroundImage(UIImage.init(named: "increase_available"), for: .normal)
        }
    }
```
这里主要就是最大值和最小值判断，这里有个sendActions的方法，个人不是很理解。猜测是用了RxSwift的话，内部编辑框的值变更，会通知到rx的回调处，其实也是通知外部同步刷新。

减的逻辑基本一致，这里就不再看了。

上面还遗漏了一个抬起手后，将定时器取消，这点很重要，如果不及时处理，很容易造成内存泄漏。看下上面给touchCancel和其它按钮状态绑定的事件：
```
//松开按钮:清除定时器
@objc fileprivate func touchUp()  {
    cleanTimer()
}

/// 内部实现
fileprivate func cleanTimer() {
        if ((timer?.isValid) != nil) {
            timer.invalidate()
            timer = nil;
        }
    }
    
/// 析构函数    
deinit {
    cleanTimer()
}
```

这里还有一个非常关键的生命周期函数：
```
// MARK: - 重新布局UI
    /// https://juejin.cn/post/6984250995874365448 layoutSubviews调用时机
    /// 改变一个UIView的Frame会触发layoutSubviews
    /// layoutSubviews, 当我们在某个类的内部调整子视图位置时，需要调用。反过来的意思就是说：如果你想要在外部设置subviews的位置，就不要重写。
    /// 这个方法，默认没有做任何事情，需要子类进行重写
    /// 如果没有这个方法，那么结果会是空白的，看不到任何东西
    override open func layoutSubviews() {
        super.layoutSubviews()
        
        let height = frame.size.height
        let width = frame.size.width
        let textFieldWidth:CGFloat = 90
        let BgBtnWidth:CGFloat = 42
        let btnSize:CGFloat = 28
        decreaseBtn.frame = CGRect(x: 0, y: (height-btnSize)/2, width: btnSize, height: btnSize)
        //减按钮(增大可触面积)
        decreaseBgBtn.frame = CGRect(x: 0, y: 0, width: BgBtnWidth, height: height)
        increaseBtn.frame = CGRect(x: width-btnSize, y: (height-btnSize)/2, width: btnSize, height: btnSize)
        //加按钮(增大可触面积)
        increaseBgBtn.frame = CGRect(x: BgBtnWidth + textFieldWidth, y: 0, width: BgBtnWidth, height: height)
        textField.frame = CGRect(x: BgBtnWidth, y: 0, width: textFieldWidth, height: height)
    }
```
这个方法必须要实现，确定视图frame，才可以真正显示到屏幕上。

### 2.5 给TextField绑定代理
在前面初始化UI里面有句代码是这样的：
`textField.delegate = self`
这里需要新建一个代理类来支持一下：
```
extension PPNumberButton: UITextFieldDelegate {
    
    // MARK: - UITextFieldDelegate
    public func textFieldDidEndEditing(_ textField: UITextField) {
        if (textField.text?.count)! == 0 || Int(textField.text!) ?? 999999999 < _minValue {
            textField.text = "\(_minValue)"
        }
        if Int(textField.text!) ?? 999999999 > _maxValue {
            if documentType == 1 {
                MBProgressHUD.showTipsMessage("数量已超出零售上限")
            }
        }
        if Int(textField.text!) ?? 999999999 >= _maxValue {
            textField.text = "\(_maxValue)"
        }
        //闭包回调，传递值给外部
        NumberResultClosure?("\(textField.text!)")
        //delegate的回调，暴露内部值给外部
//        delegate2?.numberButtonResult(self, number: "\(textField.text!)")
        
        print("当前值:   \(textField.text?.int ?? 0)")
        
        if (textField.text?.int ?? 0) <= 0 {
            decreaseBtn.setBackgroundImage(UIImage.init(named: "reduce_Disable"), for: .normal)
            increaseBtn.setBackgroundImage(UIImage.init(named: "increase_available"), for: .normal)
        }else if (textField.text?.int ?? 0) >= maxValue {
            decreaseBtn.setBackgroundImage(UIImage.init(named: "reduce_available"), for: .normal)
            increaseBtn.setBackgroundImage(UIImage.init(named: "increase_Disable"), for: .normal)
        }else{
            decreaseBtn.setBackgroundImage(UIImage.init(named: "reduce_available"), for: .normal)
            increaseBtn.setBackgroundImage(UIImage.init(named: "increase_available"), for: .normal)
        }
    }
    
    public func textField(_ textField: UITextField, shouldChangeCharactersIn range: NSRange, replacementString string: String) -> Bool {
        
        if range.location <= 8 {
            return true
        }else{
            return false
        }
    }
    
}
```
这里重写了官方文本代理的两个函数：
· textFieldDidEndEditing 结束编辑调用
· textField 实时调用，能否显示

### 2.6 自定义扩展函数
原因是每个地方使用场景不同，这里需要通过扩展函数，可以自己添加一些逻辑，方便调用处能够实现效果。
```
public extension PPNumberButton {
    
    /**
     输入框中的内容
     */
    var currentNumber: String? {
        get {
            return (textField.text!)
        }
        set {
            textField.text = newValue
            if isUserInteractionEnabled {
                if (newValue?.int ?? 0) <= 0 {
                    decreaseBtn.setBackgroundImage(UIImage.init(named: "reduce_Disable"), for: .normal)
                    increaseBtn.setBackgroundImage(UIImage.init(named: "increase_available"), for: .normal)
                } else if (newValue?.int ?? 0) >= maxValue {
                    decreaseBtn.setBackgroundImage(UIImage.init(named: "reduce_available"), for: .normal)
                    increaseBtn.setBackgroundImage(UIImage.init(named: "increase_Disable"), for: .normal)
                }else{
                    decreaseBtn.setBackgroundImage(UIImage.init(named: "reduce_available"), for: .normal)
                    increaseBtn.setBackgroundImage(UIImage.init(named: "increase_available"), for: .normal)
                }
            }
        }
    }
    /**
     设置最小值
     */
    var minValue: Int {
        get {
            return _minValue
        }
        set {
            _minValue = newValue
            textField.text = "\(newValue)"
        }
    }
    /**
     设置最大值
     */
    var maxValue: Int {
        get {
            return _maxValue
        }
        set {
            _maxValue = newValue
        }
    }
    
    /**
     * 加减按钮的响应闭包回调
     * 当数字变化后，暴露给外部，执行自己逻辑
     */
    func numberResult(_ finished: @escaping ResultClosure) {
        NumberResultClosure = finished
    }
    
    /**
     输入框中的字体属性
     */
    func inputFieldFont(_ inputFieldFont: UIFont) {
        textField.font = inputFieldFont;
    }
    
    /**
     加减按钮的字体属性
     */
    func buttonTitleFont(_ buttonTitleFont: UIFont) {
        increaseBtn.titleLabel!.font = buttonTitleFont;
        decreaseBtn.titleLabel!.font = buttonTitleFont;
    }
    
    /**
     设置按钮的边框颜色
     */
    func borderColor(_ borderColor: UIColor) {
//        layer.borderColor = borderColor.cgColor;
        decreaseBtn.layer.borderColor = borderColor.cgColor;
        increaseBtn.layer.borderColor = borderColor.cgColor;
        
//        layer.borderWidth = 0.5;
        decreaseBtn.layer.borderWidth = 0.5;
        increaseBtn.layer.borderWidth = 0.5;
    }
    
    //注意:加减号按钮的标题和背景图片只能设置其中一个,若全部设置,则以最后设置的类型为准
    
    /**
     设置加/减按钮的标题
     
     - parameter decreaseTitle: 减按钮标题
     - parameter increaseTitle: 加按钮标题
     */
    func setTitle(decreaseTitle: String, increaseTitle: String) {
        decreaseBtn.setBackgroundImage(nil, for: .normal)
        increaseBtn.setBackgroundImage(nil, for: .normal)
        
        decreaseBtn.setTitle(decreaseTitle, for: .normal)
        increaseBtn.setTitle(increaseTitle, for: .normal)
    }
    
    /**
     设置加/减按钮的背景图片
     
     - parameter decreaseImage: 减按钮背景图片
     - parameter increaseImage: 加按钮背景图片
     */
    func setImage(decreaseImage: UIImage, increaseImage: UIImage) {
        decreaseBtn.setTitle(nil, for: .normal)
        increaseBtn.setTitle(nil, for: .normal)
        
        decreaseBtn.setBackgroundImage(decreaseImage, for: .normal)
        increaseBtn.setBackgroundImage(increaseImage, for: .normal)
    }
    
}

```
这些都是暴露给外部调用者的函数，外部可以轻松改变背景，改变边框颜色等。

```
public extension PPNumberButton {
    
    /// 是否开启交互 这里主要是设置能否点击，业务需求是某些情况没有超过最大值或低于最小值也不可点击哦
    var enabled: Bool {
        get {
            return isUserInteractionEnabled
        }
        set {
            isUserInteractionEnabled = newValue
            if isUserInteractionEnabled {
                let text = textField.text
                currentNumber = text
            } else {
                decreaseBtn.setBackgroundImage(UIImage.init(named: "reduce_Disable"), for: .normal)
                increaseBtn.setBackgroundImage(UIImage.init(named: "increase_Disable"), for: .normal)
            }
        }
    }
    
}
```
上面的代码也是因为增加了 特殊需求，添加的一个扩展方法。

另外还有一部分代码是使用RxSwift，增加了扩展方法，看下哈：
```
extension Reactive where Base: PPNumberButton {
    var gmMaxValue: Binder<Int> {
        return Binder(self.base) { (pp, max) in
            pp.maxValue = max
        }
    }
}
```
这里应该是获取最大值，pp指代这个步进器，max就是最大值。

### 2.7 调用者如何使用
上面基本就把步进器实现完了。
下面看看调用者如何来使用步进器，这里简单示例下：

先声明一个步进器
```
private weak var cus_stepper: PPNumberButton!
```

然后目标地方创建一个步进器，并且初始化步进器相关属性
```
let stepperViewW: CGFloat = 178.0
let stepperViewH: CGFloat = 40.0
let stepperViewB: CGFloat = 24.0

/// 创建步进器并初始化
let cus_stepper = PPNumberButton()
cus_stepper.shakeAnimation = true
cus_stepper.maxValue = 999999999
cus_stepper.minValue = 0

/// 暴露回调给外部，里面就是自己逻辑了
cus_stepper.numberResult { [weak self] number in
    guard let self = self else { return }
    self.act_stepperCountChnage(number)
}

/// 添加到目标contentView里面
contentView.addSubview(cus_stepper)
self.cus_stepper = cus_stepper

/// 布局设置
cus_stepper.snp.makeConstraints { make in
    make.top.equalTo(imgV_photo.snp.bottom).offset(ySuperVerPad)
    make.right.equalTo(contentView).offset(-ySuperHorPad)
    make.width.equalTo(stepperViewW)
    make.height.equalTo(stepperViewH)
}

cus_stepper.snp.makeConstraints { make in
    make.bottom.equalTo(contentView).offset(-stepperViewB)
}
```

基本就这样。

## 3 总结

1.自定义View需要先分析下交互，确定下如何拆分这个自定义View，再复杂的视图都是一块砖一块瓦拼接起来的，其实这个步进器就3个结构，左右图标，中间编辑框。
2.iOS自定义View一般都是继承UIView，重写两个init函数，layoutSubviews这个方法确定视图frame，才能显示到屏幕上。
3.给UIButton添加action一定要用@objc注解，表示可以使用动态方法。
4.对于封装自定义View一定要严格将某些方法暴露出去，某些方法私有，因为调用者不知道具体细节，所以封装者就一定要注意扩展性，一方面提供好用的扩展方法出去，能够方便调用者去设置各种属性，另一方面要注意类安全性，以及解耦性，尽量不要跟数据太耦合了。
