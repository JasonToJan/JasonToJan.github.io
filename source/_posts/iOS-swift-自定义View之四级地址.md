---
title: iOS-swift-自定义View之四级地址
date: 2023-01-27 16:26:44
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
目标是可以选择四级地址，什么是四级地址呢？省市区街道。类似淘宝买东西时，支付订单时需要填写收货地址，这里就需要构造一个四级地址弹框，让用户去选择。具体效果如下：
<img src=%E5%9B%9B%E7%BA%A7%E5%9C%B0%E5%9D%80.gif>

这个我们可以单独将这个地址弹框单独封装一下，本身就是一个工具，很多地方可能会用的。
另外还需要提供传参，因为可能用户之前选择了某些地址，现在要更改，就需要跳转到已选择的四级地址，并且可以自己切换任何一个省市区，如果重新选择省，那么下级地址都需要清空；如果重新更新了市，那么下级清空，上级保留。
大概就是这样子。

所以我们就可以将四级地址视图用一个UIView来实现。

## 2 控件结构

首先我们分析下如何来完成这个需求，代码如何实现。
这里我们考虑将ui提供的四级地址，通过自定义View来实现。

顶部有一个列表，就是记录用户已选地址，省市区街道。
底部也有一个列表，而且有分组，以地址的拼音首字母分组，组下也可能有会有不定行数的cell。

这里我们就考虑 用两个UITableView来承载顶部和底部的列表。
两个UITableView都放在  AddressView 的自定义View里面吧。

所以会多出两个Cell，一个是顶部Cell，就叫做AddressCell吧；一个是底部Cell，承载底部列表，就叫做AddressSelectorCell吧。

另外还有数据层，就叫它 AddressModel吧，放地址数据的。

所以我们只需要4个文件就可以完成这个需求了。

* AddressView 四级地址自定义View
* AddressCell 顶部Cell
* AddressSelectorCell 底部Cell
* AddressModel 数据层

## 3 AddressView 实现逻辑

### 3.1 全局变量定义
```
class AddressView: UIView {
    
    typealias selectAddressBlock = (Array<RegionModel>)->()
    var selectAddressHandle: selectAddressBlock?
    var cancelAddressHandle: Handler?
    
    var modelArray: Array<Any> = []
    var addressArray: Array<(key: Swift.String, value: Array<RegionModel>)> = []
    var selectedIndex: Int = 0
```
这里定义一个闭包，就是用户勾选了4级地址后，需要将地址暴露个调用方，肯定需要一个方法发出去的。

cancelAddressHandle是一个取消函数，主要处理用户点击蒙层或者点击取消的事件。

modelArray是记录顶部列表数据的数组。

addressArray是记录底部列表数据的数组。

selectedIndex是记录顶部选择了第几项的全局变量记录。

然后是2个懒加载的UITableView，看下哈：
```
/// 顶部视图列表
private lazy var tblView: UITableView = {
    let tableView = UITableView.init(frame: CGRect(x: 0, y: 196, width: ScreenWidth, height: ScreenHeight - 196), style: .grouped)
    tableView.register(AddressCell.self, forCellReuseIdentifier: AddressCell.identifier)
    tableView.delegate = self
    tableView.dataSource = self
    tableView.separatorStyle = .none
    tableView.backgroundColor = .white
    tableView.showsVerticalScrollIndicator = false
    tableView.showsHorizontalScrollIndicator = false
    tableView.isScrollEnabled = false
    tableView.layer.cornerRadius = 14
    tableView.layer.maskedCorners = [.layerMinXMinYCorner,.layerMaxXMinYCorner]
    return tableView
}()
```

还有一个底部的列表：
```
/// 底部视图列表
private lazy var addressTblView: UITableView = {
    let tableView = UITableView.init(frame: CGRect.zero, style: .grouped)
    tableView.register(AddressSelectorCell.self, forCellReuseIdentifier: AddressSelectorCell.identifier)
    tableView.delegate = self
    tableView.dataSource = self
    tableView.backgroundColor = .white
    tableView.separatorStyle = .none
    tableView.showsVerticalScrollIndicator = false
    tableView.showsHorizontalScrollIndicator = false
    return tableView
}()
```
虽然是两个UITableView，但可以共享同一个代理和数据源，需要在里面另外加if else判断下。

### 3.2 生命周期函数
这里只有一个初始化，但其实没有做什么，也没有把那两个UITableView加进去，这里应该就是空白的区域。
```
  override init(frame: CGRect) {
        super.init(frame: frame)
        self.backgroundColor = UIColor.init(red: 0, green: 0, blue: 0, alpha: 0.6)
    }
    
    required init?(coder: NSCoder) {
        fatalError("init(coder:) has not been implemented")
    }
```

### 3.3 初始化数据
这里是另外提供的函数，来初始化，所以需要调用方额外调用这个方法。
也正是这个方法，才会去加载ui的。

```
func traverse(province: String, city: String, area: String, street: String){
        modelArray.removeAll()
        let dataArr: [RegionModel] = globalAddressModel?.data ?? AddressModel.init().data
        self.recursive(dataArr: dataArr, province: province, city: city, area: area, street: street)
        
        if globalAddressModel != nil {
            if modelArray.count == 0 {
                /// 这里就展示省列表
                self.setData(data: globalAddressModel!)
            }else if modelArray.count == 4{
                /// 这里展示街道列表
                let model = modelArray[2]
                self.setData(data: model)
            } else{
                /// 其它情况，就拿最后一个数据，在设置给底部内容tableView里面
                let model = modelArray.last as! RegionModel
                self.setData(data: model)
            }
        }
        if modelArray.count < 4 {
            modelArray.append("请选择")
        }
        
        /// 大概意思就是找到“请选择”在哪一个位置
        selectedIndex = modelArray.firstIndex{$0 is String} ?? modelArray.count-1
        /// 开始加载UI
        self.loadUI()
    }
```
globalAddressModel 实际上再AppDelegate初始化的时候就去异步加载json文件了，这个应该就是保存的所有的四级地址信息了。

然后这里用到了一个函数 recursive函数，是递归来寻找目标四级地址。
```
/// 递归获取目标地址
func recursive(dataArr: [RegionModel], province: String, city: String, area: String, street: String){
    for model in dataArr{
        if model.name.contains(province) ||
            model.name.contains(city) ||
            model.name.contains(area) ||
            model.name.contains(street){
            modelArray.append(model)
            self.recursive(dataArr: model.children, province: province, city: city, area: area, street: street)
            break
        }
    }
}
```
找到后，会将实体append到这个数组中。

然后上面初始化数据还用到一个函数：setData，作用是设置底部数据，因为这里底部自始至终都是同一个UITableView，会经常需要刷新数据。这里就是刷新底部数据的意思吧。

看下这个setData函数哈：
```
func setData(data: Any){
        var dataArr: [RegionModel] = []
        if data is AddressModel {
            let tmpData = data as! AddressModel
            dataArr = tmpData.data
        }else{
            let tmpData = data as! RegionModel
            dataArr = tmpData.children
        }
        /// 上面的代码目的是为了拿到 RegionModel数组
        
        if dataArr.count == 0 {
            return
        }
        
        var tmpDic: Dictionary<String,Array<RegionModel>> = Dictionary<String,Array<RegionModel>>()
        for model in dataArr{
            var key: String = ""
            key = self.findFirstLetterFromString(aString: model.name)
            var arr = tmpDic[key] ?? Array<RegionModel>()
            arr.append(model)
            tmpDic[key] = arr
        }
        
        //处理了第二个字首字母排序
        _ = tmpDic.keys.compactMap{ key in
            var tmpArr = tmpDic[key]!
            tmpArr = tmpArr.sorted{(model1,model2) in
                var key1 = ""
                var key2 = ""
                key1 = self.findFirstLetterFromString(aString: model1.name, isSecond: true)
                key2 = self.findFirstLetterFromString(aString: model2.name, isSecond: true)
                return key1 < key2
            }
            tmpDic[key] = tmpArr
        }
        addressArray = tmpDic.sorted{$0.key < $1.key}
        
        /// addressArray应该是拿到目前展示的内容区域数据 底部区域
        self.addressTblView.reloadData()
    }
```
这里代码虽然长，但实际上具体还是挺简单的，主要是生成一个字典的映射关系，key存放首字母，value存放同一个首字母下对应的地址列表。大概意思就是这个tmpDic是一个key value形式的map集合，key是这些比如省份首字母为G，那么value就对应广东省，甘肃省，广西，贵州省等下的所有地址。这里就对应tableView里面的每一组的数据了。然后处理下第二个首字母排序的问题。

好的，这里打住了，回到第一个函数 traverse方法中，里面根据modelArray的长度去决定刷新哪一个列表，因为调用方给的地址不一定是四级，可能是1级或2级或3级，这里也是需要考虑到的。能在哪个层级搜索到，就定位到哪个层级，那么最后一个层级这里使用 append一个“请选择”，这里说明如果是字符串了，那么这里就是最后一层级。感觉这还可以优化下，比较用字符串来判断到哪一层级不太合适的。

traverse最后一个方法是加载UI。
```
func loadUI(){
        /// 先加顶部
        self.addSubview(tblView)
        /// 顶部布局
        self.updateUI()
        
        /// 再加底部
        self.addSubview(addressTblView)
        /// 底部布局
        addressTblView.snp.remakeConstraints{make in
            make.leading.trailing.bottom.equalTo(0)
            make.top.equalTo(tblView.snp.bottom).offset(0)
        }
    }

func updateUI(){
        tblView.snp.remakeConstraints{make in
            make.leading.trailing.equalTo(0)
            make.top.equalTo(196)
            if modelArray.count == 1 {
                make.height.equalTo(96)
            }else{
                make.height.equalTo(96+modelArray.count*48)
            }
        }
        self.tblView.reloadData()
    }
```
大概意思就是把那两个UITableView加给父View。

然后需要用的扩展方法有下面这个：
```
extension AddressView{
    
    /// 找到中文对应的第一个字符对应的第一个字母拼音
    func findFirstLetterFromString(aString: String, isSecond : Bool = false) -> String {
        //转变成可变字符串
        let mutableString = NSMutableString.init(string: aString)
        
        //将中文转换成带声调的拼音
        CFStringTransform(mutableString as CFMutableString, nil,      kCFStringTransformToLatin, false)
        
        //去掉声调
        let pinyinString = mutableString.folding(options:          String.CompareOptions.diacriticInsensitive, locale:   NSLocale.current)
        
        //将拼音首字母换成大写
        let strPinYin = polyphoneStringHandle(nameString: aString,    pinyinString: pinyinString).uppercased()
        
        //截取大写首字母
        let arr = strPinYin.components(separatedBy: " ")
        let completeStr = isSecond == false ? arr[0] : arr[1]
        let firstString = completeStr.subScript(index: 0, length: 1)
        //判断首字母是否为大写
        let regexA = "^[A-Z]$"
        let predA = NSPredicate.init(format: "SELF MATCHES %@", regexA)
        return predA.evaluate(with: firstString) ? firstString : "#"
    }
    
    //多音字处理，根据需要添自行加
    func polyphoneStringHandle(nameString: String, pinyinString: String) -> String {
        //        if nameString.hasPrefix("长") {return "chang"}
        //        if nameString.hasPrefix("沈") {return "shen"}
        //        if nameString.hasPrefix("厦") {return "xia"}
        //        if nameString.hasPrefix("地") {return "di"}
        //        if nameString.hasPrefix("重") {return "chong"}
        return pinyinString
    }
}
```
这里主要是一个工具方法，内部使用，无需多讲。

好了，前面的ui基本就这些。
下面主要是讲解下UITableView的代理方法和数据源配置。

```
/// 设置顶部tableView 代理和数据
extension AddressView: UITableViewDelegate,UITableViewDataSource{
    func numberOfSections(in tableView: UITableView) -> Int {
        if tableView == tblView {
            return 1    /// 顶部tableView只有一组
        }else{
            return addressArray.count   /// 很多组 26个字母都有的话，就26组
        }
    }
```
这里扩展AddressView，去实现UITableViewDelegate和UITableViewDataSource协议。
这是第一个方法，numberOfSections，返回有多少组。这里因为顶部只有一种类型，也就只有一组，底部用了一个数组记录，组数就是数组的长度。

```
func tableView(_ tableView: UITableView, numberOfRowsInSection section: Int) -> Int {
        if tableView == tblView {
            /// 顶部，如果没有 历史地址，则为0，有则显示 一组多少个
            if modelArray.count == 1 && (modelArray.last is String) {
                return 0
            }else{
                return modelArray.count
            }
        }else{
            /// 取当前组里面多少个
            let dic = addressArray[section]
            let arr = dic.value
            return arr.count
        }
    }
```
这是第二个方法，numberOfRowsInSection，意思就是这个组下有多少行。

```
 func tableView(_ tableView: UITableView, heightForRowAt indexPath: IndexPath) -> CGFloat {
        /// 上面下面都是48
        if tableView == tblView {
            return 48
        }else{
            return 48
        }
    }
```
这里是每行的高度。

```
/// 每个cell怎么显示
func tableView(_ tableView: UITableView, cellForRowAt indexPath: IndexPath) -> UITableViewCell {
    /// 如果是顶部
    if tblView == tableView {
        /// 顶部用AddressCell
        let cell = tableView.dequeueReusableCell(withIdentifier: AddressCell.identifier, for: indexPath) as! AddressCell
        let model = modelArray[indexPath.row]
        /// 数据用modelArray里面的数据
        cell.model = model
        /// 选中的话，高亮显示文字颜色
        if selectedIndex == indexPath.row {
            cell.titleLb.textColor = UIColor.init(hex: "#409EFF")
        }else{
            cell.titleLb.textColor = UIColor.init(hex: "#3B4058")
        }
        /// 这里是请选择什么
        if model is String {
            cell.updateUI(indexPath: indexPath)
        }
        
        /// 指示剂方向
        if indexPath.row == 0 {
            cell.position = .down // 第0行，只有下面的
        }else if indexPath.row == modelArray.count-1{
            cell.position = .top /// 最后一行，只有上面
        }else{
            cell.position = .middle /// 其它都有
        }
        
        return cell
    }else{
        /// 如果是底部，数据取addressArray里面的
        let dic = addressArray[indexPath.section]
        let arr = dic.value
        let model = arr[indexPath.row] // 这个字母类别里面的 第row行
        
        /// 这个cell用 AddressSelector来实现
        let cell = tableView.dequeueReusableCell(withIdentifier: AddressSelectorCell.identifier, for: indexPath) as! AddressSelectorCell
        
        /// 第1行才有字母
        if indexPath.row == 0 {
            cell.indexLb.text = dic.key
        }else{
            cell.indexLb.text = ""
        }
        
        /// 当前在那个层级 比如是广东省
        let obj = modelArray[selectedIndex]
        if obj is String { /// 请选择层级
            cell.checkImgView.isHidden = true
            cell.nameLb.textColor = UIColor.init(hex: "#3B4058")
        }else{
            /// 已选层级
            let selectedModel = obj as! RegionModel
            if selectedModel.name == model.name { /// 列表中的就是已选的，高亮一下
                cell.nameLb.textColor = UIColor.init(hex: "#409EFF")
                cell.checkImgView.isHidden = false
            }else{
                cell.nameLb.textColor = UIColor.init(hex: "#3B4058")
                cell.checkImgView.isHidden = true
            }
        }
        
        ///  显示目标地址名称
        cell.nameLb.text = model.name
        return cell
    }
}
```
这个方法就比较关键了，就是每行怎么展现的。
用哪个Cell，是否要高亮，指示剂怎么展示，这里有点像Android的adpter里面的bindAdapter实现逻辑。这里需要按照需求在item里面自行处理逻辑。

如果是底部，就需要考虑是否显示字母，是否高亮这些了。

```
/// 标题和单元格间隔
func tableView(_ tableView: UITableView, heightForHeaderInSection section: Int) -> CGFloat {
    /// 顶部头部预留48dp显示标题“请选择所在地区”
    if tableView == tblView {
        return 48
    }else{
        return 0.01
    }
}
```
顶部预留高度，用来展示自己的标题。

```
/// 和上面对应，预留48个dp怎么显示
func tableView(_ tableView: UITableView, viewForHeaderInSection section: Int) -> UIView? {
    if tableView == tblView {
        let view = UIView.init(frame: CGRect.init(x: 0, y: 0, width: ScreenWidth, height: 48))
        let label = UILabel.init()
        label.font = .pingFangSemibold(size: 18)
        label.textColor = UIColor.init(hex: "#3B4058")
        label.text = "请选择所在地区"
        
        let closeBtn: UIButton = {
            let btn = UIButton()
            btn.setTitle("取消", for: .normal)
            btn.setTitleColor(UIColor.init(hex: "#409EFF"), for: .normal)
            btn.titleLabel?.font = .pingFangRegular(size: 16)
            btn.addTarget(self, action: #selector(actionForClose), for: .touchUpInside)
            return btn
        }()
        
        view.addSubview(label)
        view.addSubview(closeBtn)
        label.snp.makeConstraints{make in
            make.centerY.equalTo(view)
            make.centerX.equalTo(view)
            make.width.equalTo(130)
            make.height.equalTo(30)
        }
        closeBtn.snp.makeConstraints{make in
            make.trailing.equalTo(-16)
            make.centerY.equalTo(view)
            make.width.equalTo(48)
            make.height.equalTo(48)
        }
        return view
    }else{
        return UIView()
    }
}
```
标题和取消按钮显示逻辑。

```
/// 单元格尾巴怎么显示
func tableView(_ tableView: UITableView, heightForFooterInSection section: Int) -> CGFloat {
    /// 如果是顶部tableView，预留48个dp
    if tableView == tblView {
        return 48
    }else{
        return 0.01
    }
}
```
这里顶部有效，有48个单位长度，用来显示底部标题。

```
/// 尾部显示 选择下级地址
func tableView(_ tableView: UITableView, viewForFooterInSection section: Int) -> UIView? {
    if tableView == tblView {
        let view = UIView.init(frame: CGRect.init(x: 0, y: 0, width: ScreenWidth, height: 48))
        
        let lineView = UIView()
        lineView.backgroundColor = UIColor.init(hex: "#F6F6F6")
        let label = UILabel.init()
        label.font = .pingFangSemibold(size: 16)
        label.textColor = UIColor.init(hex: "#3B4058")
        
        if selectedIndex == 0 {
            label.text = "选择省份/地区"
        }else if selectedIndex == 1{
            label.text = "选择城市"
        }else if selectedIndex == 2{
            label.text = "选择区/县"
        }else{
            label.text = "选择街道/镇"
        }
        
        view.addSubview(lineView)
        view.addSubview(label)
        
        lineView.snp.makeConstraints{make in
            make.top.equalTo(4)
            make.leading.trailing.equalTo(0)
            make.height.equalTo(0.5)
        }
        label.snp.makeConstraints{make in
            make.top.equalTo(16)
            make.leading.equalTo(8)
            make.width.equalTo(150)
            make.height.equalTo(30)
        }
        return view
    }else{
        return UIView()
    }
}
```
这里就是确定底部标题的怎么显示的。

```
/// 点击了单元格
func tableView(_ tableView: UITableView, didSelectRowAt indexPath: IndexPath) {
    if tblView == tableView {
        /// 顶部点击了哪个单元格
        if indexPath.row == 0 {
            self.setData(data: globalAddressModel!)
        }else{
            let model = modelArray[indexPath.row-1]
            self.setData(data: model)
        }
        selectedIndex = indexPath.row
        /// 改变全局变量 selectedIndex，再去加载
        self.tblView.reloadData()
    }else{
        /// 当前是哪一组
        let dic = addressArray[indexPath.section]
        let arr = dic.value
        /// 当前是哪一行
        let model = arr[indexPath.row]
        
        /// 请选择是第几个
        let index = modelArray.firstIndex{$0 is String} ?? 0
        /// 顶部是请选择
        if modelArray[selectedIndex] is String && modelArray.count < 5{
            /// 插入目标点击的地址 给modelArray
            modelArray.insert(model, at: index)
            if modelArray.count == 5 {
                modelArray.removeLast()
            }
        }else{
            /// 顶部不是请选择,而已已选择的地址
            modelArray[selectedIndex] = model
            /// 下面有请选择，需要清空之前选择的值
            let isContain = modelArray.contains(where: {$0 is String})
            if isContain {
                for i in (selectedIndex+1 ... modelArray.count-1).reversed(){
                    if modelArray[i] is String {
                        continue
                    }
                    modelArray.remove(at: i)
                }
            } else if !isContain && selectedIndex != 3{
                for i in (selectedIndex+1 ... modelArray.count-1).reversed(){
                    modelArray.remove(at: i)
                }
            }
            
            if modelArray.count < 4 && !modelArray.contains(where: {$0 is String}){
                modelArray.append("请选择")
            }
        }
        
        /// 没有请选择了，会直接关闭弹框，执行回调
        if modelArray.count == 4 && !modelArray.contains(where: {$0 is String}){
            var arr: [RegionModel] = []
            for model in modelArray {
                let obj = model as! RegionModel
                arr.append(obj)
            }
            selectAddressHandle?(arr)
            self.removeFromSuperview()
        }else{
            /// 更新底部数据，以及顶部选择index
            self.setData(data: model)
            self.updateUI()
            selectedIndex += 1
        }
    }
}
```
上面的代码逻辑，主要就是处理点击顶部单元格，和底部单元格的逻辑。
顶部点击会调用setData,会同步刷新底部。

底部点击也会setData,刷新底部数据，如果全部选择，则会走闭包回调，将用户选择的四级地址回调出去。

然后还有一些其它扩展方法，简单看下哈。
```
/// 扩展给外部
extension AddressView{
    /// 通知调用者当有一个或者多个手指触摸到了视图或者窗口时触发此方法。
    /// touches是UITouch的集合，通过UITouch我们可以检测触摸事件的属性，是单拍还是双拍，还有触摸的位置等。
    /// 这里应该是触摸到蒙层后关闭弹框
    override func touchesBegan(_ touches: Set<UITouch>, with event: UIEvent?) {
        guard let point = touches.first?.location(in: self) else { return }
        let p = self.layer.convert(point, from: self.layer)
        if self.layer.contains(p) {
            self.actionForClose()
        }
    }
    
    /// 移除自己，类似关闭弹框效果
    @objc func actionForClose(){
        cancelAddressHandle?()
        self.removeFromSuperview()
    }
}
```
这里actionForClose是UiButton设置的一个action，上面哪个是覆写的官方方法，应该是触摸到蒙层自动关闭弹框。

## 4 AddressCell 实现逻辑
这个可以理解成Android中的Adapter了。也就是item怎么显示。
```
/// 顶部cell
class AddressCell: UITableViewCell {
    
    enum LinePosition {
        case down
        case middle
        case top
    }
    
    static var identifier: String = "AddressCell"
```
这里需要继承UITableViewCell，必须要有一个identifier，这个主要是为了复用，因为列表可能会很长，为了合理复用，这里一般都需要一个标识符。

### 4.1 子View定义
```
 /// 安徽省
lazy var titleLb: UILabel = {
    let label = UILabel()
    label.font = .pingFangMedium(size: 16)
    label.textColor = UIColor.init(hex: "#3B4058")
    return label
}()

/// 实心圆
lazy var roundView: UIView = {
    let view = UIView()
    view.backgroundColor = UIColor.init(hex: "#409EFF")
    view.layer.cornerRadius = 3.5
    view.layer.borderWidth = 1
    view.layer.borderColor = UIColor.init(hex: "#409EFF").cgColor
    return view
}()

/// 圆上面的线条
lazy private var toplineView: UIView = {
    let view = UIView()
    view.backgroundColor = UIColor.init(hex: "#409EFF")
    return view
}()

/// 圆下面的线条
lazy private var downlineView: UIView = {
    let view = UIView()
    view.backgroundColor = UIColor.init(hex: "#409EFF")
    return view
}()

/// 右侧箭头
lazy private var rightImgView: UIImageView = {
    let imgView = UIImageView()
    imgView.image = UIImage.init(named: "客户信息_右箭头")
    return imgView
}()
```
这里是顶部item用的一些零部件。
比如上下线图，右侧箭头，中间文字。实心圆。
这里变空心，实际上只是设置了中间颜色为纯白。

### 4.2 数据定义
```
/// 监听数据变化 数据填充
var model: Any?{
    didSet{
        guard let _model = model else{return}
        if _model is RegionModel {
            let tmp = _model as! RegionModel
            titleLb.text = tmp.name
        }
    }
}
    
/// 线条位置
var position: LinePosition?{
    didSet{
        guard let _position = position else{return}
        /// 上面和下面的线条都先一出去
        toplineView.removeFromSuperview()
        downlineView.removeFromSuperview()
        
        /// 如果在下面 空心圆
        if _position == .down {
            contentView.addSubview(downlineView)
            downlineView.snp.makeConstraints{make in
                make.centerX.equalTo(roundView)
                make.top.equalTo(roundView.snp.top).offset(0)
                make.bottom.equalTo(0)
                make.width.equalTo(1)
            }
            roundView.backgroundColor = UIColor.init(hex: "#409EFF")
        }else if _position == .top{
            /// 如果在上面 空心圆
            contentView.addSubview(toplineView)
            toplineView.snp.makeConstraints{make in
                make.centerX.equalTo(roundView)
                make.top.equalTo(0)
                make.bottom.equalTo(roundView.snp.top)
                make.width.equalTo(1)
            }
            if model is String {
                roundView.backgroundColor = .white
            }else{
                roundView.backgroundColor = UIColor.init(hex: "#409EFF")
            }
        }else{
            /// 都不在，也是都存在的意思
            contentView.addSubview(toplineView)
            contentView.addSubview(downlineView)
            toplineView.snp.makeConstraints{make in
                make.centerX.equalTo(roundView)
                make.top.equalTo(0)
                make.bottom.equalTo(roundView.snp.top)
                make.width.equalTo(1)
            }
            downlineView.snp.remakeConstraints{make in
                make.centerX.equalTo(roundView)
                make.top.equalTo(roundView.snp.bottom)
                make.bottom.equalTo(0)
                make.width.equalTo(1)
            }
            roundView.backgroundColor = UIColor.init(hex: "#409EFF")
        }
    }
}
```
这里定义了2个数据，1个是具体显示什么的数据，一个是线条怎么展示，什么时候展示实心和空心圆的逻辑。

### 4.3 生命周期函数
首先看下初始化。
```
override init(style: UITableViewCell.CellStyle, reuseIdentifier: String?) {
        super.init(style: style, reuseIdentifier: reuseIdentifier)
        
        self.selectionStyle = .none
        
        /// 添加安徽省
        contentView.addSubview(titleLb)
        titleLb.snp.makeConstraints{make in
            make.centerY.equalToSuperview()
            make.leading.equalTo(roundView.snp.trailing).offset(22)
            make.trailing.equalTo(rightImgView.snp.leading).offset(10)
            make.height.equalTo(20)
        }
        
        /// 添加圆形指示器
        contentView.addSubview(roundView)
        roundView.snp.makeConstraints{make in
            make.centerY.equalToSuperview()
            make.leading.equalTo(14)
            make.width.equalTo(7)
            make.height.equalTo(7)
        }
        
        /// 添加右侧箭头
        contentView.addSubview(rightImgView)
        rightImgView.snp.makeConstraints{make in
            make.centerY.equalToSuperview()
            make.trailing.equalTo(-16)
            make.width.equalTo(19)
            make.height.equalTo(19)
        }
        
    }
    
    required init?(coder: NSCoder) {
        fatalError("init(coder:) has not been implemented")
    }
```
这里就加了3个视图，并且布局了。

### 4.4 其它扩展方法
```
extension AddressCell{
    func updateUI(indexPath: IndexPath){
        if indexPath.row == 1 {
            titleLb.text = "请选择城市"
        }else if indexPath.row == 2 {
            titleLb.text = "请选择县"
        }else if indexPath.row == 3{
            titleLb.text = "请选街道"
        }else{
            titleLb.text = ""//model as? String
        }
    }
}
```
这里就是显示item特殊情况的文案。

## 5 AddressSelectorCell 实现逻辑
这里也同上面的Cell，有一个标识符。
```
/// 底部Cell
class AddressSelectorCell: UITableViewCell {
    
    static var identifier: String = "UITableViewCell"
```

### 5.1 子View
这里需要哪些子View呢？
```
//索引 eg: A字母
lazy var indexLb: UILabel = {
    let label = UILabel()
    label.font = .pingFangMedium(size: 12)
    label.textColor = UIColor.init(hex: "#9B9DA7")
    return label
}()

//名称 eg: 安徽省
lazy var nameLb: UILabel = {
    let label = UILabel()
    label.font = .pingFangMedium(size: 14)
    label.textColor = UIColor.init(hex: "#3B4058")
    return label
}()

//勾选图标
lazy var checkImgView: UIImageView = {
    let imgView = UIImageView()
    imgView.isHidden = true
    imgView.image = UIImage.init(named: "勾选")
    return imgView
}()
```
这里非常简单，只有3个子视图，字母，中间名称，是否勾选。

### 5.2 生命周期函数
看下初始化吧。
```
override init(style: UITableViewCell.CellStyle, reuseIdentifier: String?) {
        super.init(style: style, reuseIdentifier: reuseIdentifier)
        self.selectionStyle = .none
        
        /// 添加索引，并且布局
        contentView.addSubview(indexLb)
        indexLb.snp.makeConstraints{make in
            make.leading.equalTo(8)
            make.centerY.equalToSuperview()
            make.width.equalTo(10)
            make.height.equalTo(20)
        }
        
        /// 添加名称，并且位置确定
        contentView.addSubview(nameLb)
        nameLb.snp.makeConstraints{make in
            make.leading.equalTo(indexLb.snp.trailing).offset(16)
            make.trailing.equalTo(-16)
            make.centerY.equalToSuperview()
            make.height.equalTo(20)
        }
        
        /// 确定勾选图标
        contentView.addSubview(checkImgView)
        checkImgView.snp.makeConstraints{make in
            make.trailing.equalTo(-16)
            make.centerY.equalToSuperview()
            make.width.equalTo(16)
            make.height.equalTo(16)
        }
    }
    
    required init?(coder: NSCoder) {
        fatalError("init(coder:) has not been implemented")
    }
```
也是非常简单，添加了3个子视图，并且确定位置了。

## 6 AddressModel 实现逻辑
这个是四级地址的数据模型，可以看下：
```
import Foundation
import HandyJSON

class AddressModel: HandyJSON {
    var data: [RegionModel] = []
    required init() {}
}

class RegionModel: HandyJSON {
    var code: String = ""
    var name: String = ""
    var children: [RegionModel] = []
    required init() {}
}
```
继承了HandyJSON，方便解析。
这里AddressModel实际上内部也是由 RegionModel数组构成的。
这个AddressModel用来存放省列表比较合适。

RegionModel可以存放市，区，街道，都没问题。

封装四级地址基本上就这些了。

## 7 外部调用方式

### 7.1 先声明一个四级地址View
```
lazy private var addressView: AddressView = {
        let tmpview = AddressView(frame: CGRect.init(x: 0, y: 0, width: ScreenWidth, height: ScreenHeight))
        return tmpview
    }()
```
这里设定了它的frame，也就是大小位置。

### 7.2  弹出四级地址
```
func showAddressSelector(){
        self.view.addSubview(self.addressView)
        self.addressView.traverse(province: self.orderAddressModel?.province ?? "", city: self.orderAddressModel?.city ?? "", area: self.orderAddressModel?.region ?? "", street: self.orderAddressModel?.street ?? "")
        addressView.selectAddressHandle = {[weak self] (arr)in
            guard let weakSelf = self else{return}
            let pModel = arr[0]
            let cModel = arr[1]
            let aModel = arr[2]
            let tModel = arr[3]
            ...
            
        }
    }
```
这里通过addSubview的方式，将这个四级地址添加到这个UIViewController的根布局上。类似叠加布局的方式。

然后调用了一个关键的函数，traverse，将初始地址设置进去。

然后设置了选择回调函数，这里用户选择了四级地址后，会回调到这个handle里面哦。

## 8 总结

* 做一个复杂的自定义View，一定要化繁为简，将复杂的结构一步一步拆分成一个一个基本视图，这样能够很好规划实现方案。

* 任何一个自定义View实现方案可能会有多种，可以选择自己最熟悉的一种，然后进行研究，觉得不合适可以换其它方案来实现。

* 四级地址最关键的应该是数据了，这里我们实现方案是单页面，数据同步刷新的方式，其实也可以考虑多页面，可自由切换，毕竟地址层级不会太多。

* 需要考虑到用户之前填写的地址，可能只填了1到3级，所以一定要考虑全面一点，作为封装视图者，不能只考虑到最完美的情况。

* 多个UITableView是可以共用同一个代理和数据源的，只需要再代理里面区分下是那个UITableView即可。

* UITableView有很多代理协议，这里可以设置多个组类，一个组多少行，头部View，尾部View，头部高度，尾部高度，点击item，等多种方法，可以实现很复杂的逻辑。

* UITableView的Cell一定要分配一个标识符，用来复用，增加程序稳定性。