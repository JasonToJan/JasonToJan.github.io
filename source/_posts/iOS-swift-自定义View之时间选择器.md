---
title: iOS-swift-自定义View之时间选择器
date: 2023-01-28 09:56:57
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
当我们选择购买一个东西时，有时需要预约发货，这时候需要用户手动选择某一天某一个时间，ui给的方案是这样的：
预约日期：
<img src=%E9%A2%84%E7%BA%A6%E6%97%A5%E6%9C%9F.png>

预约时间：
<img src=%E9%A2%84%E7%BA%A6%E6%97%B6%E9%97%B4.png>
这里是间隔了2个小时一个时间。

可以看到这个日期是以日历形式展现，可以选择未来某个日期；时间是以滚轮的方式展现，可以上下滑动到目标时间点。

对于这个需求，我们要如何实现呢？‘

答案当然是自定义View了。

## 2 结构分析
显而易见，这个因为设计到分页，我们可以考虑分页区域使用一个UIScrollView，水平滑动，宽度设置为屏幕的2倍，一半用来展示日历，一半用来展示预约时间的UIPickerView。

所以需要我们自定义一个UIView，这里面先绘制顶部分割线，再绘制标题，“预约日期”和“预约时间”和“取消”的文案。

然后是一个显示日历操作栏，包括左右箭头和中间的月份显示，这个要放到UIScrollView的左半边，包括星期视图和日历的UICollectionView的集合View，这里日历我们采用这个集合View很容易实现的。

然后右侧就是一个UIPickerView,这个系统封装好了，使用很方便，我们主要负责填充数据。

那这个结构大体出来了，一个是整个自定义View，暂且把它叫做CalenderView，还有一个就是日历内部的Cell，暂且叫做CalenderCell。

## 3 撸下CalenderView

### 3.1 全局变量

```
class CalendarView: UIView {
    
    /// 传出去给调用者
    typealias DoneBlock = (TimeModel)->()
    var doneHandle: DoneBlock!
    
    // 当月日期Model
    var modelArr: [TimeModel] = []
    
    // 动态调整上下月的时间变量
    var dynamicDate = Date()
    
    //默认选中的日期
    var timeModel: TimeModel = TimeModel()
    
    //时间时短数据
    static let range: String = "                      "
    private var timeArr: [String] = ["08:00\(range)10:00",
                                   "10:00\(range)12:00",
                                   "12:00\(range)14:00",
                                   "14:00\(range)16:00",
                                   "16:00\(range)18:00",
                                   "18:00\(range)20:00",
                                   "20:00\(range)22:00"]
    private var selectIndex = 0
    
```
这里定义了一个闭包返回，主要是用户选择后，回调给外部。

然后有一个TimeModel的数组， 也就是填充我们的日历视图。

timeModel是我们选择的日期，会高亮显示。

timeArr是我们右侧使用的UIPickerView要用的数据。

### 3.2 UI定义

这里我们大致会用到这些视图。
```
//白色背景底 整个半屏的视图
private lazy var acView: UIView = {
    let view = UIView()
    view.backgroundColor = .white
    return view
}()

//选中条
private lazy var lineView: UIView = {
    let view = UIView()
    view.backgroundColor = UIColor.init(hex: "#409EFF")
    return view
}()

/// 预约日期按钮
private lazy var dateBtn: UIButton = {
    let btn = UIButton()
    btn.setTitle("预约日期", for: .normal)
    btn.titleFont = .pingFangMedium(size: 16)
    btn.setTitleColor(UIColor.lightGray, for: .normal)
    btn.setTitleColor(UIColor.black, for: .selected)
    btn.addTarget(self, action: #selector(actionForChooseDate), for: .touchUpInside)
    btn.isSelected = true
    return btn
}()

/// 预约时间按钮
private lazy var timeBtn: UIButton = {
    let btn = UIButton()
    btn.setTitle("预约时间", for: .normal)
    btn.titleFont = .pingFangMedium(size: 16)
    btn.setTitleColor(UIColor.lightGray, for: .normal)
    btn.setTitleColor(UIColor.black, for: .selected)
    btn.addTarget(self, action: #selector(actionForChooseTime), for: .touchUpInside)
    return btn
}()

/// 取消按钮
private lazy var cancelBtn: UIButton = {
    let btn = UIButton()
    btn.setTitle("取消", for: .normal)
    btn.setTitleColor(UIColor.init(hex: "#409EFF"), for: .normal)
    btn.titleFont = .pingFangRegular(size: 16)
    btn.addTarget(self, action: #selector(actionForCancel), for: .touchUpInside)
    return btn
}()

//日历工具条
private lazy var dateToolView: UIView = {
    let view = UIView()
    return view
}()

//日历工具条显示当前月份标签
private lazy var monthLb: UILabel = {
    let lab = UILabel()
    lab.font = .pingFangMedium(size: 16)
    lab.textColor = UIColor.init(hex: "#3B4058")
    lab.text = ""
    lab.textAlignment = .center
    return lab
}()

//日历工具条左箭头
private lazy var toolLeftBtn: UIButton = {
    let btn = UIButton()
    btn.setImage(UIImage.init(named: "日历_左箭头"), for: .normal)
    btn.addTarget(self, action: #selector(actionForLeftEvent), for: .touchUpInside)
    btn.isHidden = true
    return btn
}()

//日历工具条右箭头
private lazy var toolRightBtn: UIButton =  {
    let btn = UIButton()
    btn.setImage(UIImage.init(named: "日历_右箭头"), for: .normal)
    btn.addTarget(self, action: #selector(actionForRightEvent), for: .touchUpInside)
    return btn
}()

//星期X显示条
private lazy var weeksView: UIView = {
    let view = UIView()
    view.backgroundColor = .white
    view.layer.shadowColor = UIColor.init(hex: "#7D7E80").cgColor
    view.layer.shadowOffset = CGSize.init(width: 0, height: 2)
    view.layer.shadowRadius = 10
    view.layer.shadowOpacity = 0.16
    return view
}()

//滚动视图 左右滚动
private lazy var scrollView: UIScrollView = {
    let _scrollView = UIScrollView()
    _scrollView.contentSize = CGSize.init(width: ScreenWidth*2, height: _scrollView.size.height)
    _scrollView.bounces = false
    _scrollView.isScrollEnabled = true
    _scrollView.isPagingEnabled = true
    _scrollView.showsVerticalScrollIndicator = false
    _scrollView.showsHorizontalScrollIndicator = false
    return _scrollView
}()

//月份水印背景
private lazy var watermarkView: CalendarWatermark = {
    let view = CalendarWatermark()
    view.backgroundColor = .white
    view.isHidden = true  //当前版本隐藏水印
    return view
}()

//日历展示表
    private lazy var collectionView: UICollectionView = {
        let layout = UICollectionViewFlowLayout()
        layout.scrollDirection = .vertical
        let collection = UICollectionView(frame: CGRect.init(x: 0, y: 34+40, width: ScreenWidth, height: ScreenHeight*0.57*0.57), collectionViewLayout: layout)
        collection.backgroundColor = .clear
        collection.delegate = self
        collection.dataSource = self
        collection.isPagingEnabled = true
        collection.showsHorizontalScrollIndicator = false
        collection.bounces = false
        collection.register(CalendarCell.self, forCellWithReuseIdentifier: CalendarCell.identifier)
        return collection
}()

//时间选择器 x轴以 屏幕宽度为起点，刚好在第二屏了
private lazy var pickerView: UIPickerView = {
    let picker = UIPickerView.init(frame: CGRect.init(x: ScreenWidth, y: 0, width: ScreenWidth, height: ScreenHeight*0.57*0.57+34+40))
    picker.delegate = self
    picker.dataSource = self
    return picker
}()
```

这些需要用代理和数据源的都设置self，后面再具体实现。

这里可以认为是我们搭建房子用到的一些素材。先写好，后期再拼接下。

### 3.3 生命周期函数

初始化看下，应该要先设置一个蒙层。
```
override init(frame: CGRect){
        super.init(frame: frame)
        /// 这里应该是蒙层
        self.backgroundColor = UIColor.init(r: 100, g: 1, b: 1, a: 0.3)
        self.alpha = 0
        
        let lastDyaDate = YSDateTool.lastDay()
        let com = YSDateTool.currentDateCom(date: lastDyaDate)
        watermarkView.monthStr = "\(com.month!)"
    }
    
    required init?(coder: NSCoder) {
        fatalError("init(coder:) has not been implemented")
    }
```
确实如此，这里就只加了个颜色。

### 3.4 定义方法，展示和隐藏选择器

```
extension CalendarView{
    /// 显示弹窗
    public func showAlertView() {
        let str = "\(timeModel.startTime)\(CalendarView.range)\(timeModel.endTime)"
        selectIndex = timeArr.firstIndex(of:str) ?? 0
        let timeStr = "\(timeModel.year)-\(timeModel.month)-\(timeModel.day)"
        let date = YSDateTool.dateStringToDate(timeStr)
        self.setDate(date: date)
        self.setUI()
        pickerView.selectRow(selectIndex, inComponent: 0, animated: false)
        UIView.animate(withDuration: 0.25) {
            self.alpha = 1
        }
    }

    /// 隐藏弹窗
    public func dismissAlertView() {
        self.animate(duration: 0.25) {
            self.alpha = 0
        } completion: { finish in
            self.removeFromSuperview()
        }
    }
}
```
这个方法很重要，主要提供给外部使用，这里帮助外部设置数据，因为外部调用者可能不知道内部细节，所以我们最好在这个类里面实现展示弹框的方法。

### 3.5 设置初始化日期

外部可能有选择好的日期，所以我们必须提供一个方法，设置数据。
```
extension CalendarView{
    /// 按照这个date获取这个月的数据
    func setDate(date: Date = Date()){
        modelArr.removeAll()
        let count = YSDateTool.countOfDaysInCurrentMonth(date: date)    //当月天数
        for i in 0..<count{
            let model = self.initializeModel(date: date, day: i+1)
            modelArr.append(model)
        }
        let firstWeekDay = YSDateTool.firstWeekDayInCurrentMonth(date: date)    //当月第一天周几
        
        //头部空缺数据填充 插入一个空的model
        if firstWeekDay != 7 {
            for _ in 0 ..< firstWeekDay{
                let model = self.initializeModel(date: date, day: 0)
                modelArr.insert(model, at: 0)
            }
        }
        //尾部空缺数据填充,插入一个空的model
        if modelArr.count < 35{
            for _ in modelArr.count ..< 35{
                let model = self.initializeModel(date: date, day: 0)
                modelArr.append(model)
            }
        }
        
        collectionView.reloadData()
    }
    
    /// 初始化数据，返回
    func initializeModel(date: Date, day: Int)->TimeModel{
        let calendar = NSCalendar.current
        let com = calendar.dateComponents([.year, .month, .day], from: date)
        let model = TimeModel()
        model.year = com.year!
        model.month = com.month!
        model.day = day
        
        // model指代当前时间
        let monthStr = com.month! < 10 ? "0\(model.month)" : "\(model.month)"
        let dayStr = com.day! < 10 ? "0\(model.day)" : "\(model.day)"
        model.dateStr = "\(com.year!)-\(monthStr)-\(dayStr)"
        return model
    }
}
```
这里也搞了2个扩展函数，会根据目标日期，获取到该日期的一个月的数据，然后会填充第一天非星期天的情况的数据，也会填最后几天空白区域的数据。

### 3.6 设置UI

有了日期就可以设置UI了，所以我们看下如何填充视图的。

```
extension CalendarView {
    func setUI() {
        
        //日历顶部按钮布局
        self.addSubview(acView)
        acView.snp.makeConstraints{make in
            make.leading.trailing.bottom.equalTo(0)
            make.height.equalTo(ScreenHeight*0.57)
        }
        
        // 加顶部横线 左上角默认位置
        acView.addSubview(lineView)
        self.lineViewLayout()
        
        // 加预约日期
        acView.addSubview(dateBtn)
        dateBtn.snp.makeConstraints{make in
            make.top.equalTo(11)
            make.leading.equalTo(28)
            make.width.equalTo(66)
            make.height.equalTo(22)
        }
        
        // 预约时间
        acView.addSubview(timeBtn)
        timeBtn.snp.makeConstraints{make in
            make.top.equalTo(11)
            make.centerX.equalToSuperview()
            make.width.equalTo(66)
            make.height.equalTo(22)
        }
        
        // 加取消按钮
        acView.addSubview(cancelBtn)
        cancelBtn.snp.makeConstraints{make in
            make.top.equalTo(11)
            make.trailing.equalTo(-16)
            make.width.equalTo(66)
            make.height.equalTo(22)
        }
        
        
        // 继续加滚动视图
        acView.addSubview(scrollView)
        scrollView.snp.makeConstraints{make in
            make.top.equalTo(dateBtn.snp.bottom).offset(10)
            make.leading.trailing.equalToSuperview()
            make.bottom.equalToSuperview()
        }
        
        // acView添加完毕，现在到内部ScrollView层级了--------------
    
        // 加个集合View 集合View因为初始化确定了y轴高度，这里无需重复布局
        scrollView.addSubview(collectionView)
        
        // 再加个pickerView 时间滚轮，这里直接加到ScrollView里面了，也无需布局，直接覆盖到上层的
        scrollView.addSubview(pickerView)
        
        // 先加工具栏
        scrollView.addSubview(dateToolView)
        dateToolView.snp.makeConstraints{make in
            make.top.leading.equalTo(0)
            make.width.equalTo(ScreenWidth)
            make.height.equalTo(40)
        }
        
        // 滚动视图再加星期视图
        scrollView.addSubview(weeksView)
        weeksView.snp.makeConstraints{make in
            make.top.equalTo(dateToolView.snp.bottom).offset(4)
            make.leading.equalTo(0)
            make.width.equalTo(ScreenWidth)
            make.height.equalTo(30)
        }
        
        
        // 处理下工具栏内部布局
        dateToolView.addSubview(monthLb)
        monthLb.snp.makeConstraints{make in
            make.centerY.equalToSuperview()
            make.centerX.equalToSuperview()
            make.width.equalTo(100)
            make.height.equalTo(20)
        }
        
        // 日历工具条加左按钮
        dateToolView.addSubview(toolLeftBtn)
        toolLeftBtn.snp.makeConstraints{make in
            make.centerY.equalToSuperview()
            make.width.height.equalTo(40)
            make.leading.equalTo(16)
        }

        // 日历工具条加右侧箭头
        dateToolView.addSubview(toolRightBtn)
        toolRightBtn.snp.makeConstraints{make in
            make.centerY.equalToSuperview()
            make.width.height.equalTo(40)
            make.trailing.equalTo(-16)
        }
        
        // 设置星期几内部布局
        setWeekUiInner()
        
        // 地址底部按钮
        addBottomUI()
    
    }
```
中规中矩，老老实实从上到下，从左到右布局。

内部星期几看下如何布局的：
```
func setWeekUiInner() {
        let arr = ["日","一","二","三","四","五","六"]

        for i in 0 ..< arr.count {
            let width = ScreenWidth/7
            let lab = UILabel()
            lab.text = arr[i]
            lab.textColor = UIColor.init(hex: "#323233")
            lab.font = .pingFangRegular(size: 12)
            lab.textAlignment = .center
            weeksView.addSubview(lab)
            lab.snp.makeConstraints{make in
                make.leading.equalTo(width*CGFloat(i))
                make.centerY.equalToSuperview()
                make.width.equalTo(width)
                make.height.equalTo(20)
            }
        }
    }
```

还有底部的下一步和完成：
```
func addBottomUI() {
        let nextBtn: UIButton = UIButton.init(type: .custom)
        nextBtn.frame = CGRect.init(x: 16, y: ScreenHeight*0.57*0.57+8+34+40, width: (ScreenWidth-32), height: 44)
        nextBtn.titleFont = .pingFangMedium(size: 16)
        nextBtn.setTitleColor(.white, for: .normal)
        nextBtn.backgroundColor = UIColor.init(hex: "#409EFF")
        nextBtn.setTitle("下一步", for: .normal)
        nextBtn.layer.cornerRadius = 22
        nextBtn.addTarget(self, action: #selector(actionForNext), for: .touchUpInside)
        scrollView.addSubview(nextBtn)
        
        let doneBtn: UIButton = UIButton.init(type: .custom)
        doneBtn.frame = CGRect.init(x: ScreenWidth+16, y: ScreenHeight*0.57*0.57+8+34+40, width: (ScreenWidth-32), height: 44)
        doneBtn.titleFont = .pingFangMedium(size: 16)
        doneBtn.setTitleColor(.white, for: .normal)
        doneBtn.backgroundColor = UIColor.init(hex: "#409EFF")
        doneBtn.setTitle("完成", for: .normal)
        doneBtn.layer.cornerRadius = 22
        doneBtn.addTarget(self, action: #selector(actionForDone), for: .touchUpInside)
        scrollView.addSubview(doneBtn)
    }
```

### 3.7 填充数据

这里先看下日历数据如何使用代理和数据源的：
```
extension CalendarView: UICollectionViewDelegate,UICollectionViewDataSource,UICollectionViewDelegateFlowLayout{
    func collectionView(_ collectionView: UICollectionView, numberOfItemsInSection section: Int) -> Int {
        return modelArr.count
    }
    
    func collectionView(_ collectionView: UICollectionView, cellForItemAt indexPath: IndexPath) -> UICollectionViewCell {
        let cell = collectionView.dequeueReusableCell(withReuseIdentifier: CalendarCell.identifier, for: indexPath) as! CalendarCell
        cell.model = modelArr[indexPath.item]
        cell.selectedModel = timeModel
        return cell
    }

    // 定义每个Cell的大小
    func collectionView(_ collectionView: UICollectionView, layout collectionViewLayout: UICollectionViewLayout, sizeForItemAt indexPath: IndexPath) -> CGSize {
        let width = ScreenWidth/7
        let height = modelArr.count > 35 ? (ScreenHeight*0.57*0.57)/6 : (ScreenHeight*0.57*0.57)/5
        return CGSize(width: width-2, height: height-1)
    }
    
    // 定义每个Section的四边间距
    func collectionView(_ collectionView: UICollectionView, layout collectionViewLayout: UICollectionViewLayout, insetForSectionAt section: Int) -> UIEdgeInsets {
        return UIEdgeInsets(top: 0, left: 3.5, bottom: 0, right: 0)
    }
    
    // 这个是两行cell之间的间距（上下行cell的间距）
    func collectionView(_ collectionView: UICollectionView, layout collectionViewLayout: UICollectionViewLayout, minimumLineSpacingForSectionAt section: Int) -> CGFloat {
        return 1.0
    }
    
    // 两个cell之间的间距（同一行的cell的间距）
    func collectionView(_ collectionView: UICollectionView, layout collectionViewLayout: UICollectionViewLayout, minimumInteritemSpacingForSectionAt section: Int) -> CGFloat {
        return 1.0
    }
    
    // 选中某个ietm
    func collectionView(_ collectionView: UICollectionView, didSelectItemAt indexPath: IndexPath) {
        let model = modelArr[indexPath.item]
        let com = YSDateTool.currentDateCom()
        if com.year == model.year && com.month == model.month && model.day <= com.day! || model.day == 0{
            return
        }

        timeModel.year = model.year
        timeModel.month = model.month
        timeModel.day = model.day
        collectionView.reloadData()
    }
}
```
主要逻辑委托给Cell来实现了。后面再看下。

### 3.8 UIPickerView数据填充

然后看下预约时间如何填充数据的。
```
extension CalendarView:UIPickerViewDelegate,UIPickerViewDataSource {

    func numberOfComponents(in pickerView: UIPickerView) -> Int {
        return 1
    }
    
    func pickerView(_ pickerView: UIPickerView, numberOfRowsInComponent component: Int) -> Int {
        return timeArr.count
    }

    func pickerView(_ pickerView: UIPickerView, widthForComponent component: Int) -> CGFloat {
        return ScreenWidth
    }
    
    func pickerView(_ pickerView: UIPickerView, rowHeightForComponent component: Int) -> CGFloat {
        44
    }
    
    func pickerView(_ pickerView: UIPickerView, viewForRow row: Int, forComponent component: Int, reusing view: UIView?) -> UIView {
        
        //设置分割线 pickerView的subvews的第二个一般用来做分割线 subviews会有2个view
        for view in pickerView.subviews {
            print("view.height=\(view.size.height) 长度为：\(pickerView.subviews.count)")
            if view.size.height <= 50 {
                view.backgroundColor = .clear
                //自定义分割线
                let toplayer = CALayer()
                toplayer.frame = CGRect.init(x: 0, y: 0, width: ScreenWidth, height: 1)
                toplayer.backgroundColor = UIColor.init(hex: "#EBEDF0").cgColor
                view.layer.addSublayer(toplayer)

                let bottomlayer = CALayer()
                bottomlayer.frame = CGRect.init(x: 0, y: 44-1, width: ScreenWidth, height: 1)
                bottomlayer.backgroundColor = UIColor.init(hex: "#EBEDF0").cgColor
                view.layer.addSublayer(bottomlayer)
               
            }else{
                view.backgroundColor = .clear
            }
        }
        
        //设置内容样式
        var label = view as? UILabel
        if label == nil {
            label = UILabel.init()
            label?.backgroundColor = .clear
            label?.frame = CGRect.init(x: 0, y: 0, width:ScreenWidth, height:44)
        }
        if row == selectIndex {
            label?.textColor = UIColor.init(hex: "#409EFF")
        }else{
            label?.textColor = UIColor.init(hex: "#3B4058")
        }
        label?.textAlignment = .center
        label?.font = .pingFangRegular(size: 16)
        //label?.minimumScaleFactor = 0.5
        //label?.adjustsFontSizeToFitWidth = true
        label?.text = timeArr[row]
        return label!
    }
    
    func pickerView(_ pickerView: UIPickerView, didSelectRow row: Int, inComponent component: Int) {
        selectIndex = row
        pickerView.reloadAllComponents()
    }
}
```
主要逻辑是viewForRow覆写的方法里面。
交代了如何绘制分割线和选中高亮显示。
这里有个点要注意，这个分割线用到了 UIPickerView的subViews，这里通常设置分割线都要用到这个东西，具体细节我不是很清楚，预测这个subViews第二个就是居中的item。第一个item很长，不能用来分割线。

然后就是didSelectRow用来响应用户点击事件，需要刷新下components。

### 3.9 交互事件

这里交互事件统一用一个扩展类来实现。
```
extension CalendarView {
    
    @objc func actionForChooseDate(){
        self.lineViewLayout()
        self.scrollDate()
    }
    
    @objc func actionForChooseTime(){
        self.lineViewLayout(leading: 1)
        self.scrollTime()
    }
    
    @objc func actionForCancel(){
        self.dismissAlertView()
    }
    
    @objc func actionForLeftEvent(){
        dynamicDate = YSDateTool.lastMonth(date: dynamicDate)
        self.isRefrenshMonth()
    }
    
    @objc func actionForRightEvent(){
        dynamicDate = YSDateTool.nextMonth(date: dynamicDate)
        self.isRefrenshMonth()
    }
    
    @objc func actionForNext(){
        self.lineViewLayout(leading: 1)
        self.scrollTime()
    }
    
    @objc func actionForDone(){
        if doneHandle != nil {
            let str = timeArr[selectIndex]
            let arr = str.components(separatedBy: CalendarView.range)
            timeModel.startTime = arr[0]
            timeModel.endTime = arr[1]
            
            let monthStr = timeModel.month < 10 ? "0\(timeModel.month)" : "\(timeModel.month)"
            let dayStr = timeModel.day < 10 ? "0\(timeModel.day)" : "\(timeModel.day)"
            timeModel.dateStr = "\(timeModel.year)-\(monthStr)-\(dayStr)"
            
            doneHandle(timeModel)
        }
        self.dismissAlertView()
    }
    
    // 滑动到日期
    func scrollDate(){
        dateBtn.isSelected = true
        timeBtn.isSelected = false
        scrollView.setContentOffset(CGPoint.init(x: 0, y: 0), animated: true)
    }
    
    // 滑动到时间
    func scrollTime(){
        dateBtn.isSelected = false
        timeBtn.isSelected = true
        scrollView.setContentOffset(CGPoint.init(x: ScreenWidth, y: 0), animated: true)
    }
    
    func isRefrenshMonth(){
        self.setDate(date: dynamicDate)
        let com = YSDateTool.currentDateCom(date: dynamicDate)
        monthLb.text = "\(com.year!)年\(com.month!)月"
        watermarkView.monthStr = "\(com.month!)"
        
        let currentCom = YSDateTool.currentDateCom()
        if (com.year! == currentCom.year!  && com.month! > currentCom.month!) || (com.year! > currentCom.year!){
            toolLeftBtn.isHidden = false
        }else{
            toolLeftBtn.isHidden = true
        }
    }
    
    func lineViewLayout(leading: Int = 0){
        lineView.snp.remakeConstraints{make in
            make.top.equalTo(0)
            if leading == 0 {
                make.leading.equalTo(leading)
            }else{
                make.centerX.equalToSuperview() // 居中了
            }
            make.width.equalTo(66+56) // 宽度
            make.height.equalTo(2)  // 高度
        }
    }
}
```

## 4 撸下时间工具

这个只是个单纯工具，不必重复造轮子。
```
class YSDateTool {
    // MARK: - 当前时间组件
    static func currentDateCom(date: Date = Date()) -> DateComponents{
        let calendar = Calendar.current
        let com = calendar.dateComponents([.year, .month, .day], from: date)
        return com
    }
    // MARK: - 今年
    static func currentYear(date: Date = Date()) -> Int {
        let calendar = NSCalendar.current
        let com = calendar.dateComponents([.year, .month, .day], from: date)
        return com.year!
    }
  
    // MARK: - 今月
    static func currentMonth(date: Date = Date()) -> Int {
        let calendar = NSCalendar.current
        let com = calendar.dateComponents([.year, .month, .day], from: date)
        return com.month!
    }
  
    // MARK: - 今日
    static func currentDay(date: Date = Date()) -> Int {
        let calendar = NSCalendar.current
        let com = calendar.dateComponents([.year, .month, .day], from: date)
        return com.day!
    }
    
    // MARK: - 今天星期几
    static func currentWeekDay(date: Date = Date()) -> Int {
        let calendar = NSCalendar.current
        let com = calendar.dateComponents([.weekday], from: date)
        return com.weekday!
    }
    
    // MARK: - 农历今年
    static func currentChineseYear(date: Date = Date()) -> String {
        let calendar = NSCalendar.init(calendarIdentifier: .chinese)
        let com = calendar?.components([.year], from: date)
        return numberToChina(yearNum: (com?.year)!) + "年"
    }
    
    // MARK: - 农历今月
    static func currentChineseMonth(date: Date = Date()) -> String {
        let calendar = NSCalendar.init(calendarIdentifier: .chinese)
        let com = calendar?.components([.month], from: date)
        return numberToChina(monthNum: (com?.month)!)
    }

    // MARK: - 农历今日
    static func currentChineseDay(date: Date = Date()) -> String {
        let calendar = NSCalendar.init(calendarIdentifier: .chinese)
        let com = calendar?.components([.day], from: date)
        return numberToChina(dayNum: (com?.day)!)
    }
    
    // MARK: - 农历今日
    static func currentChineseDayInt(date: Date = Date()) -> Int {
        let calendar = NSCalendar.init(calendarIdentifier: .chinese)
        let com = calendar?.components([.day], from: date)
        return (com?.day)!
    }
    
    // MARK: - 农历星期
    static func currentChineseWeekYear(date: Date = Date()) -> String {
        let calendar = NSCalendar.init(calendarIdentifier: .chinese)
        let com = calendar?.components([.weekday], from: date)
        return numberToChina(weekNum: (com?.weekday)!)
    }
  
    // MARK: - 下个月
    static func nextMonth(date: Date = Date()) -> Date {
        let calendar = NSCalendar.current
        let nDate = calendar.date(byAdding: DateComponents(month: 1), to: date)
        return nDate!
    }
    
    // MARK: - 上个月
    static func lastMonth(date: Date = Date()) -> Date {
        let calendar = NSCalendar.current
        let lDate = calendar.date(byAdding: DateComponents(month: -1), to: date)
        return lDate!
    }
    
    // MARK: - 下一天
    static func lastDay(date: Date = Date()) -> Date{
        let calendar = NSCalendar.current
        let lDate = calendar.date(byAdding: DateComponents(day: 1), to: date)
        return lDate!
    }
  
    // MARK: - 本月天数
    static func countOfDaysInCurrentMonth(date: Date = Date()) -> Int {
        let calendar = Calendar(identifier: Calendar.Identifier.gregorian)
        let range = (calendar as NSCalendar?)?.range(of: NSCalendar.Unit.day, in: NSCalendar.Unit.month, for: date)
        return (range?.length)!
    }
 
    // MARK: - 当月第一天是星期几
    static func firstWeekDayInCurrentMonth(date: Date = Date()) -> Int {
        // 星期和数字一一对应 星期日：7
        let dateFormatter = DateFormatter()
        dateFormatter.dateFormat = "yyyy-MM"
        let date = dateFormatter.date(from: String(date.xj.year)+"-"+String(date.xj.month))
        let calender = Calendar(identifier: Calendar.Identifier.gregorian)
        let comps = (calender as NSCalendar?)?.components(NSCalendar.Unit.weekday, from: date!)
        var week = comps?.weekday
        if week == 1 {
            week = 8
        }
        return week! - 1
    }

    // MARK: - - 获取指定日期各种值
    // 根据年月得到某月天数
    static func getCountOfDaysInMonth(year: Int, month: Int) -> Int{
        let dateFormatter = DateFormatter()
        dateFormatter.dateFormat = "yyyy-MM"
        let date = dateFormatter.date(from: String(year)+"-"+String(month))
        let calendar = Calendar(identifier: Calendar.Identifier.gregorian)
        let range = (calendar as NSCalendar?)?.range(of: NSCalendar.Unit.day, in: NSCalendar.Unit.month, for: date!)
        return (range?.length)!
    }
    
    // MARK: - 根据年月得到某月第一天是周几
    static func getfirstWeekDayInMonth(year: Int, month: Int) -> Int{
        let dateFormatter = DateFormatter()
        dateFormatter.dateFormat = "yyyy-MM"
        let date = dateFormatter.date(from: String(year)+"-"+String(month))
        let calendar = Calendar(identifier: Calendar.Identifier.gregorian)
        let comps = (calendar as NSCalendar?)?.components(NSCalendar.Unit.weekday, from: date!)
        let week = comps?.weekday
        return week! - 1
    }

    // MARK: - date转日期字符串
    static func dateToDateString(_ date: Date, dateFormat: String) -> String {
        let timeZone = NSTimeZone.default
        let formatter = DateFormatter()
        formatter.timeZone = timeZone
        formatter.dateFormat = dateFormat
        let date = formatter.string(from: date)
        return date
    }

    // MARK: - 日期字符串转date
    static func dateStringToDate(_ dateStr: String) -> Date {
        let dateFormatter = DateFormatter()
        dateFormatter.timeZone = TimeZone(abbreviation: "UTC+8")
        dateFormatter.dateFormat = "yyyy-MM-dd"
        let date = dateFormatter.date(from: dateStr)
        return date!
    }
    
    // MARK: - 时间字符串转date
    static func timeStringToDate(_ dateStr: String) -> Date {
        let dateFormatter = DateFormatter()
        dateFormatter.timeZone = TimeZone(abbreviation: "UTC+8")
        dateFormatter.dateFormat = "yyyy-MM-dd HH:mm:ss"
        let date = dateFormatter.date(from: dateStr)
        return date!
    }

    // MARK: - 计算天数差
    static func dateDifference(_ dateA: Date, from dateB: Date) -> Double {
        let interval = dateA.timeIntervalSince(dateB)
        return interval/86400
    }

    // MARK: - 比较时间先后
    static func compareOneDay(oneDay: Date, withAnotherDay anotherDay: Date) -> Int {
        let dateFormatter: DateFormatter = DateFormatter()
        dateFormatter.dateFormat = "yyyy-MM-dd"
        let oneDayStr: String = dateFormatter.string(from: oneDay)
        let anotherDayStr: String = dateFormatter.string(from: anotherDay)
        let dateA = dateFormatter.date(from: oneDayStr)
        let dateB = dateFormatter.date(from: anotherDayStr)
        let result: ComparisonResult = (dateA?.compare(dateB!))!
        // Date1 is in the future
        if(result == ComparisonResult.orderedDescending ) {
            return 1
        }
        // Date1 is in the past
        else if(result == ComparisonResult.orderedAscending) {
            return 2
        }
            // Both dates are the same
        else {
            return 0
        }
    }

    // MARK: - 时间与时间戳之间的转化
    // 将时间转换为时间戳
    static func stringToTimeStamp(_ stringTime: String) -> Int {
        let dfmatter = DateFormatter()
        dfmatter.dateFormat = "yyyy-MM-dd HH: mm: ss"
        dfmatter.locale = Locale.current
        let date = dfmatter.date(from: stringTime)
        let dateStamp: TimeInterval = date!.timeIntervalSince1970
        let dateSt: Int = Int(dateStamp)
        return dateSt
    }
    
    // 将时间戳转换为年月日
    static func timeStampToString(_ timeStamp: String) -> String {
        let string = NSString(string: timeStamp)
        let timeSta: TimeInterval = string.doubleValue
        let dfmatter = DateFormatter()
        dfmatter.dateFormat="yyyy年MM月dd日"
        let date = Date(timeIntervalSince1970: timeSta)
        return dfmatter.string(from: date)
    }
    
    // 将时间戳转换为具体时间
    static func timeStampToStringDetail(_ timeStamp: String) -> String {
        let string = NSString(string: timeStamp)
        let timeSta: TimeInterval = string.doubleValue
        let dfmatter = DateFormatter()
        dfmatter.dateFormat="yyyy年MM月dd日HH: mm: ss"
        let date = Date(timeIntervalSince1970: timeSta)
        return dfmatter.string(from: date)
    }
    
    // 将时间戳转换为时分秒
    static func timeStampToHHMMSS(_ timeStamp: String) -> String {
        let string = NSString(string: timeStamp)
        let timeSta: TimeInterval = string.doubleValue
        let dfmatter = DateFormatter()
        dfmatter.dateFormat="HH: mm: ss"
        let date = Date(timeIntervalSince1970: timeSta)
        return dfmatter.string(from: date)
    }
    
    // 将时间戳转换为时分
    static func timeStampToHHMM(_ timeStamp: String) -> String {
        let string = NSString(string: timeStamp)
        let timeSta: TimeInterval = string.doubleValue
        let dfmatter = DateFormatter()
        dfmatter.dateFormat="HH: mm"
        let date = Date(timeIntervalSince1970: timeSta)
        return dfmatter.string(from: date)
    }
    
    // 获取系统的当前时间戳
    static func getStamp(date: Date = Date()) -> Int{
        // 获取当前时间戳
        let timeInterval: Int = Int(date.timeIntervalSince1970)
        return timeInterval
    }
    
    // 日数字转汉字
    static func numberToChina(yearNum: Int) -> String {
        // 以一个天干和一个地支相配，排列起来，天干在前，地支在后，天干由甲起，地支由子起，阳干对阳支，阴干对阴支（阳干不配阴支，阴干不配阳支）
        // 就会得到六十年一周期的甲子回圈，一般称为“六十甲子”或“花甲子”
        // 十天干： 甲（jiǎ）、乙（yǐ）、丙（bǐng）、丁（dīng）、戊（wù）、己（jǐ）、庚（gēng）、辛（xīn）、壬（rén）、癸（guǐ）
        // 十二地支： 子（zǐ）、丑（chǒu）、寅（yín）、卯（mǎo）、辰（chén）、巳（sì）、午（wǔ）、未（wèi）、申（shēn）、酉（yǒu）、戌（xū）、亥（hài）
        let ChinaArray = [ "甲子", "乙丑", "丙寅", "丁卯", "戊辰", "己巳", "庚午", "辛未", "壬申", "癸酉",
                         "甲戌", "乙亥", "丙子", "丁丑", "戊寅", "己卯", "庚辰", "辛己", "壬午", "癸未",
                         "甲申", "乙酉", "丙戌", "丁亥", "戊子", "己丑", "庚寅", "辛卯", "壬辰", "癸巳",
                         "甲午", "乙未", "丙申", "丁酉", "戊戌", "己亥", "庚子", "辛丑", "壬寅", "癸丑",
                         "甲辰", "乙巳", "丙午", "丁未", "戊申", "己酉", "庚戌", "辛亥", "壬子", "癸丑",
                         "甲寅", "乙卯", "丙辰", "丁巳", "戊午", "己未", "庚申", "辛酉", "壬戌", "癸亥"]
        let ChinaStr: String = ChinaArray[yearNum - 1]
        return ChinaStr
    }
    
    // 月份数字转汉字
    static func numberToChina(monthNum: Int) -> String {
        let ChinaArray = ["一月", "二月", "三月", "四月", "五月", "六月", "七月", "八月", "九月", "十月", "十一月", "十二月"]
        let ChinaStr: String = ChinaArray[monthNum - 1]
        return ChinaStr
    }
    
    // 日数字转汉字
    static func numberToChina(dayNum: Int) -> String {
        let ChinaArray = [ "初一", "初二", "初三", "初四", "初五", "初六", "初七", "初八", "初九", "初十",
                         "十一", "十二", "十三", "十四", "十五", "十六", "十七", "十八", "十九", "二十",
                         "廿一", "廿二", "廿三", "廿四", "廿五", "廿六", "廿七", "廿八", "廿九", "三十"]
        let ChinaStr: String = ChinaArray[dayNum - 1]
        return ChinaStr
    }
      
    // 星期数字转汉字
    static func numberToChina(weekNum: Int) -> String {
        let ChinaArray = ["一月", "二月", "三月", "四月", "五月", "六月", "七月", "八月", "九月", "十月", "十一月", "十二月"]
        let ChinaStr: String = ChinaArray[weekNum - 1]
        return ChinaStr
    }
    
    // MARK: - 数字前补0
    static func add0BeforeNumber(_ number: Int) -> String {
        if number >= 10 {
            return String(number)
        }else{
            return "0" + String(number)
        }
    }
    
    static func getCurrentSystemDate() -> Date{
        let date = Date() // 获得时间对象
        let zone = NSTimeZone.system // 获得系统的时区
        let time = zone.secondsFromGMT(for: date)// 以秒为单位返回当前时间与系统格林尼治时间的差
        return date.addingTimeInterval(TimeInterval(time))// 然后把差的时间加上,就是当前系统准确的时间
    }
    
    static func date(_ date: String?, dateFormat: String = "yyyy-MM-dd") -> Date? {
        guard let date = date else {
            return nil
        }
        let dateformatter = DateFormatter()
        dateformatter.timeZone = TimeZone.init(secondsFromGMT: 0)
        dateformatter.dateFormat = dateFormat
        return dateformatter.date(from: date)
    }

    // MARK: - 将时间显示为（几分钟前，几小时前，几天前）
    static func compareCurrentTime(str: String) -> String {
        //let currentDateStr = dateToDateString(Date(), dateFormat: "yyyy-MM-dd hh:mm:ss")
        let currentDate = getCurrentSystemDate()
        let sendTimeDate = date(str,dateFormat: "yyyy-MM-dd HH:mm:ss")!//self.timeStringToDate(str)
        let sendTimeInterval = Double(getStamp(date: sendTimeDate))
        let timeInterval = currentDate.timeIntervalSince(sendTimeDate)

        var todayStr = dateToDateString(Date(), dateFormat: "yyyy-MM-dd")
        todayStr = todayStr+" 00:00"
        let todayTimeInterval = Double(Date().stringToSecondTimeStamp(todayStr)) ?? 0
        let yestDayTimeInterval = Double(todayTimeInterval - 86400)
        let nextDayTimeInterval = Double(todayTimeInterval + 86400)

        var temp: Double = 0
        var result: String = ""
        if timeInterval/60 < 1 {
            result = "刚刚"
        }else if (timeInterval/60) < 60{
            temp = timeInterval/60
            result = "\(Int(temp))分钟前"
        }else if sendTimeInterval > todayTimeInterval && sendTimeInterval < nextDayTimeInterval{

            //let interval = nextDayTimeInterval - sendTimeInterval
            temp = timeInterval/60/60
            result = "\(Int(temp))小时前"
        }else if sendTimeInterval > yestDayTimeInterval && sendTimeInterval < todayTimeInterval{
            //let str = timeStampToHHMM(String.init(format: "%d", sendTimeInterval))
            let timeStr = str.components(separatedBy: " ").last ?? ""
            let str = timeStr.prefix(5)
            result = "昨天\(str)"
        }else{
            let timeStr = str.components(separatedBy: " ").first ?? ""
            let timeArr = timeStr.components(separatedBy: "-")
            if !timeArr.isEmpty {
                result = "\(timeArr[0])年\(timeArr[1])月\(timeArr[2])日"
            }
            //result = timeStampToString(String.init(format: "%d", sendTimeInterval))
        }
        
//        else if timeInterval/(30 * 24 * 60 * 60) < 12 {
//            temp = timeInterval/(30 * 24 * 60 * 60)
//            result = "\(Int(temp))个月前"
//        }else{
//            temp = timeInterval/(12 * 30 * 24 * 60 * 60)
//            result = "\(Int(temp))年前"
//        }
        return result
    }
}

```

## 5 撸下最后的日历Cell

这个就是日历里面的item，还是相当简单的。

### 5.1 全局变量
```
class CalendarCell: UICollectionViewCell {
    
    static var identifier = "UICollectionViewCell"
    
    var isGreaterThan: Bool = false     //是否取大于今天的值
    
    var isShowPassTime: Int = 0         //是否设定禁选
    
    var model:TimeModel?{
        didSet{
            guard let _model = model else {return}
            if _model.day != 0 {
                titleLab.text = "\(_model.day)"
            }else{
                titleLab.text = "" // 为0，啥也不显示
            }
            
            let com = YSDateTool.currentDateCom()
            
            // 是否是选中的日子
            if com.year == _model.year && com.month == _model.month && com.day == _model.day {
                titleLab.textColor = UIColor.init(hex: "#409EFF")
            }else{
                titleLab.textColor = UIColor.init(hex: "#323233")
            }
            
            // 禁用还是可用
            if isShowPassTime == 0 {
                if isGreaterThan {
                    if (com.year == _model.year && com.month == _model.month && _model.day > com.day!) || (_model.year == com.year! && _model.month > com.month!) || (_model.year > com.year!){
                        titleLab.textColor = UIColor.init(hex: "#C8C9CC")
                    }
                }else{
                    // 如果要大于今天，那么前面的时间就的置灰了 前一个月的点不过去，不考虑了
                    if com.year == _model.year && com.month == _model.month && _model.day < com.day!{
                        titleLab.textColor = UIColor.init(hex: "#C8C9CC")
                    }
                }
            }
        }
    }
    
    // 选中的item，背景颜色高亮
    var selectedModel: TimeModel?{
        didSet{
            guard let _model = selectedModel else{return}
            if _model.year == model?.year && _model.month == model?.month && _model.day == model?.day{
                titleLab.backgroundColor = UIColor.init(hex: "#409EFF")
                titleLab.textColor = UIColor.white
            }else{
                titleLab.backgroundColor = UIColor.clear
            }
        }
    }
```
比较重要的是model，表示当前时间实体类。这里监听变量改变，利用didSet实现动态改变文案效果。

另外一个是selectedModel，这个是当前选中的item，会比对item是否为选中，选中样式会有所不同。

### 5.2 UI子View

这里只用到一个子View。
```
var titleLab: UILabel = {
        let lan = UILabel()
        lan.font = .pingFangRegular(size: 16)
        lan.textAlignment = .center
        lan.textColor = UIColor.init(hex: "#323233")
        
        lan.layer.cornerRadius = 4
        lan.layer.masksToBounds = true
        return lan
    }()
```

### 5.3 生命周期函数

```
override init(frame: CGRect) {
        super.init(frame: frame)
        contentView.backgroundColor = .clear
        contentView.addSubview(titleLab)
        titleLab.snp.makeConstraints{make in
            make.top.bottom.leading.trailing.equalToSuperview()
        }
    }
    
    required init?(coder aDecoder: NSCoder) {
        fatalError("init(coder:) has not been implemented")
    }
```
把那个子View添加到Cell里面去了。

大概就是这样子。

## 6 总结

* 如果UI图有那种Tab页，就可以考虑使用多种方案来实现，iOS直接用简单的Button也是可以实现的，分割线也是可以用最简单的UIView来实现，不一定要用封装好的框架，谁用挖掘机来搬小石头呢，大材小用了。

* 分页效果可以使用UIScrollView来实现，把它宽度设置为屏幕的n倍，这样最有滑动可以很好控制的，不要动画也可以设置的，办法总是就很多的。

* 日历效果可以用UICollectionView来实现，看着很复杂，其实很简单，主要逻辑也不复杂，控制好数据刷新就好了。UICollectionView可以实现类似网格的效果，我们只需要在Cell里面绘制一个一个Cell，空的部分，我们啥不展示就好了。

* 系统的UIPickerView设置分割线很简单，利用UIPickerView下有一个subViews，这里寻找高度为item高度大小的，然后给view设置一个Layer即可。