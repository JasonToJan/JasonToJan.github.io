---
title: iOS swift 实现简易地图功能
date: 2023-01-31 15:27:26
top: false
cover: false
toc: true
mathjax: true
tags:
- 地图
categories:
- iOS
---

## 1 需求定义

产品需要我们实现这样一个需求，就是根据一个地址（可能有经纬度也可能没有）,然后跳转到地图页面去，地图中间会显示这个地址。运行用户自行点击定位，这样中间会显示当前位置。允许左右滑动，地图中心也会随之变更。支持搜索功能，根据用户搜索关键词，获取一个搜索列表。然后就是一个提交功能，这样需要将经纬度提交到服务端。

大致是这样的需求。
实现效果是这样的：
<img src=map.png>

## 2 需求分析

根据产品的需求，我们考虑接入高德地图。所以我们需要事先调研下如何接入高德才行。

其实这个应该非常简单，高德提供了Map，我们只需要将中间的坐标设置为我们自己的坐标。如果没有坐标，我们也调研高德的搜索接口，将搜索结果默认第一条，这应该是最准的吧，然后设置给高德的中间坐标即可。

支持用户滑动，高德就应该支持的，这里应该会提供一个代理，我们在这个代理的回调类里面重新去调用下搜索接口，就能出现附近10条地址。

二指放大，高德应该也是支持的，不需要我们额外设置啥。

另外就是定位可能复杂一点，需要实现获取下定位权限，如果没有权限，需要跳转到系统设置。定位就不需要高德支持了，我们直接调用系统接口，获取经纬度，然后设置给高德就行了。

## 3 准备工作：了解高德

首先附上高德官方文档：[https://lbs.amap.com/api/ios-sdk/summary](https://lbs.amap.com/api/ios-sdk/summary)

申请key这些工作就不用交给我们了，让产品自己申请吧。

接入高德地图，做很复杂的需求，可以参考这篇文档，写得还不错：
[https://www.jianshu.com/p/770728626874](https://www.jianshu.com/p/770728626874) 但是这个是objective-C，不过基本原理是类似的，还是有参考价值的。

另外附上高德地图可配置参数：
<table>
<thead>
<tr>
<th>参数</th>
<th>类型</th>
<th>说明</th>
</tr>
</thead>
<tbody>
<tr>
<td>logoCenter</td>
<td>CGPoint</td>
<td>可设置Logo的位置。必须在mapView.bounds之内，否则会被忽略。</td>
</tr>
<tr>
<td>showsCompass</td>
<td>Bool</td>
<td>是否显示指南针。</td>
</tr>
<tr>
<td>compassOrigin</td>
<td>CGPoint</td>
<td>设置指南针的位置。</td>
</tr>
<tr>
<td>showsScale</td>
<td>Bool</td>
<td>是否显示比例值。</td>
</tr>
<tr>
<td>scaleOrigin</td>
<td>CGPoint</td>
<td>设置比例尺的位置。</td>
</tr>
<tr>
<td>zoomEnabled</td>
<td>Bool</td>
<td>是否开启缩放手势，默认true。</td>
</tr>
<tr>
<td>scrollEnabled</td>
<td>Bool</td>
<td>是否开启滑动手势，默认true。</td>
</tr>
<tr>
<td>rotateEnabled</td>
<td>Bool</td>
<td>是否开启旋转手势，默认true。</td>
</tr>
<tr>
<td>rotateCameraEnabled</td>
<td>Bool</td>
<td>是否开启倾斜旋转手势，默认true。用户可以在地图上放置两个手指，移动它们一起向下或向上去增加或减小倾斜角。</td>
</tr>
<tr>
<td>setZoomLevel</td>
<td>CGFloat</td>
<td>改变地图缩放级别。范围从3级到19级，共17级。级别越高，展示的内容越细，例如街道等等。</td>
</tr>
<tr>
<td>setCenter</td>
<td>CLLocationCoordinate2D</td>
<td>改变地图中心的位置。传递具体的经纬度。</td>
</tr>
</tbody>
</table>

另外一个就是需要我们在AppDelegate中注册key：
```
 /// 设置高德地图
func configAMap() {
    AMapServices.shared().apiKey = "你申请的key"
    AMapServices.shared().enableHTTPS = true
}
```

这里，准备工作基本就完成了。

## 4 代码实现

### 4.1 新建两个实体类

```
class GMLocationPickerLocationResult: NSObject {
    
    //MARK: - property
    
    var coordinate: CLLocationCoordinate2D?     // 经纬度
    
    var poiName: String = ""                    // 名称
    
    var province: String = ""                   // 省
    var provinceCode: String = ""               // 省编码
    var city: String = ""                       // 市
    var citycode: String = ""                   // 市编码
    var district: String = ""                   // 区
    var adcode: String = ""                     // 区编码
    var township: String = ""                   // 乡镇街道
    var towncode: String = ""                   // 乡镇街道编码(截取前9位)
    
    var detailAddress: String = ""              // 详细地址
    var formattedAddress: String = ""           // 格式化全地址(4级+详细)
    
    var locationInfo: GMLocationPickerLocationInfo? //
    
    /// 模型转换
    convenience init(location: AMapGeoPoint, reGeocode: AMapReGeocode, locationInfo: GMLocationPickerLocationInfo?) {
        self.init()
        
        let latitude = location.latitude
        let longitude = location.longitude
        self.coordinate = CLLocationCoordinate2D(latitude: latitude, longitude: longitude)
        
        if let locationInfo = locationInfo {
            // 1.转换基础信息
            self.poiName = locationInfo.poiName
            self.province = locationInfo.province
            self.city = locationInfo.city
            self.district = locationInfo.district
            self.township = locationInfo.street
            self.detailAddress = locationInfo.address
            self.formattedAddress = locationInfo.province + locationInfo.city + locationInfo.district + locationInfo.address
        }
        if let addressComponent = reGeocode.addressComponent {
            // 2.转换所有信息
            var poiName = ""
            if let locationInfo = locationInfo {
                poiName = locationInfo.poiName
            } else if let aoisName = reGeocode.aois.first?.name {
                poiName = aoisName
            } else if let roadinterName = reGeocode.roadinters.first?.firstName {
                poiName = roadinterName
            } else {
                poiName = addressComponent.township ?? ""
            }
            self.poiName = self.poiName.ty_forEmpty(poiName)
            
            self.province = self.province.ty_forEmpty(addressComponent.province ?? "")
            self.city = self.city.ty_forEmpty(addressComponent.city ?? "")
            self.district = self.district.ty_forEmpty(addressComponent.district ?? "")
            self.township = self.township.ty_forEmpty(addressComponent.township ?? "")
            self.formattedAddress = self.formattedAddress.ty_forEmpty(reGeocode.formattedAddress ?? "")
        }
        if let addressComponent = reGeocode.addressComponent {
            // 设置编码
            self.citycode = addressComponent.citycode ?? ""
            self.adcode = addressComponent.adcode ?? ""
            self.towncode = String(addressComponent.towncode.prefix(9)) // 截取前九位
        }
        // 设置省份编码 用自带plist
        if let globalAddressModel = globalAddressModel {
            for currProvince in globalAddressModel.data {
                if currProvince.name == self.province {
                    self.provinceCode = currProvince.code
                    break
                }
            }
        }
        self.locationInfo = locationInfo
    }
}
```

这个用了一个convenience关键字来声明init，主要是覆写的作用，重载。可以参考这篇文章：[https://www.jianshu.com/p/09c6c88ed61e]()。这里面初始化已经用了高德的实体类AMapGeoPoint,这个包含了经纬度；AMapReGeocode，这个是逆地址编码；然后还有一个自定义实体，这个是Info，上面那个是result，info如下定义：

```

class GMLocationPickerLocationInfo: NSObject {
    
    //MARK: - property
    
    var coordinate: CLLocationCoordinate2D?     // 经纬度
    
    var poiName: String = ""                    // 名称
    var province: String = ""                   // 省
    var city: String = ""                       // 市
    var district: String = ""                   // 区
    var street: String = ""                     // 街道(*)
    var number: String = ""                     // 门牌号(*)
    
    var address: String = ""                    // 地址(街道 + 门牌)
    
    var distance: Int = 0                       // 与选中点距离
    
    // UI
    
    var ui_isSelected = false                   // cell是否选中中
    
    //MARK: - life cycle
    
    // ------------------------- map 内转换 -------------------------
    
    /// 模型转换 根据 [实时定位]获取的 经纬度 & 逆地理信息
    convenience init(coordinate: CLLocationCoordinate2D?, reGeocode: AMapLocationReGeocode?) {
        self.init()
        
        self.coordinate = coordinate
        
        self.poiName = reGeocode?.poiName ?? ""
        self.province = reGeocode?.province ?? ""
        self.city = reGeocode?.city ?? ""
        self.district = reGeocode?.district ?? ""
        self.street = reGeocode?.street ?? ""
        self.number = reGeocode?.number ?? ""
        self.address = (self.street + self.number)
        if self.street.isEmpty || self.number.isEmpty {
            let detailAddress = reGeocode?.formattedAddress ?? ""
            self.address = abbreviationAddressIn(detailAddress: detailAddress, province: province, city: city, district: district)
        }
        
        self.distance = 0
    }
    
    /// 模型转换 根据 [搜索{附近|关键词}]获取的 POI模型
    convenience init(mapPOI: AMapPOI) {
        self.init()
        
        if let location = mapPOI.location {
            let latitude = location.latitude
            let longitude = location.longitude
            self.coordinate = CLLocationCoordinate2D(latitude: latitude, longitude: longitude)
        }
        
        self.poiName = mapPOI.name
        self.province = mapPOI.province
        self.city = mapPOI.city
        self.district = mapPOI.district
        self.street = ""
        self.number = ""
        self.address = mapPOI.address
        
        self.distance = mapPOI.distance // 周边搜索可用
    }
    
    /// 模型转换 根据 [逆地理编码]获取的 经纬度 & 逆地理编码结果
    convenience init(location: AMapGeoPoint, reGeocode: AMapReGeocode) {
        self.init()
        
        let addressComponent = reGeocode.addressComponent
        
        let latitude = location.latitude
        let longitude = location.longitude
        self.coordinate = CLLocationCoordinate2D(latitude: latitude, longitude: longitude)
        
        if let aoisName = reGeocode.aois.first?.name {
            self.poiName = aoisName
        } else if let roadinterName = reGeocode.roadinters.first?.firstName {
            self.poiName = roadinterName
        } else {
            self.poiName = addressComponent?.township ?? ""
        }
        self.province = addressComponent?.province ?? ""
        self.city = addressComponent?.city ?? ""
        self.district = addressComponent?.district ?? ""
        self.street = addressComponent?.streetNumber?.street ?? ""
        self.number = addressComponent?.streetNumber?.number ?? ""
        self.address = (self.street + self.number)
        if self.street.isEmpty || self.number.isEmpty {
            let detailAddress = reGeocode.formattedAddress ?? ""
            self.address = abbreviationAddressIn(detailAddress: detailAddress, province: province, city: city, district: district)
        }
        self.distance = 0
    }
    
    /// 详细地址中获取缩略地址
    /// - Parameters:
    ///   - detailAddress: 详细地址
    ///   - province: 省份
    ///   - city: 城市
    ///   - district: 区域
    private func abbreviationAddressIn(detailAddress: String, province: String, city: String, district: String) -> String {
        var address = detailAddress
        address = address.replacingOccurrences(of: province, with: "")
        address = address.replacingOccurrences(of: city, with: "")
        address = address.replacingOccurrences(of: city, with: "")
        return address
    }
    
    // ------------------------- model 转换 -------------------------
    
    /// 模型转换 根据 地理信息结果模型
    convenience init(locationResult: GMLocationPickerLocationResult) {
        self.init()
        
        self.coordinate = locationResult.coordinate
        
        self.poiName = locationResult.poiName
        self.province = locationResult.province
        self.city = locationResult.city
        self.district = locationResult.district
        let street = locationResult.township
        self.address = street
        
        self.distance = 0
    }
    
    /// 模型转换 根据 仓库模型
    convenience init(warehouseInfo: GMWarehouseInfoHomeList) {
        self.init()
        
        self.coordinate = warehouseInfo.cus_coordinate
        
        self.poiName = warehouseInfo.name
        self.province = warehouseInfo.province
        self.city = warehouseInfo.city
        self.district = warehouseInfo.region
        self.street = ""
        self.number = ""
        self.address = warehouseInfo.address
        
        self.distance = 0
    }
    
    /// 模型转换 根据 门店模型
    convenience init(storeInfo: GMStoreInfoHomeModel) {
        self.init()
        
        self.coordinate = storeInfo.cus_coordinate
        
        self.poiName = storeInfo.shopName
        self.province = storeInfo.shopProvince
        self.city = storeInfo.shopCity
        self.district = storeInfo.shopCounty
        self.street = storeInfo.shopStreet
        self.number = ""
        self.address = storeInfo.shopAddress
        
        self.distance = 0
    }
}
```
这个是地图内部使用，上面那个实体，是为了更好对接服务端设定的一个实体。

### 4.2 外部类定义

```
/// 地图选择 闭包，向外吐出去
typealias GMLocationResultHandler = (GMLocationPickerLocationResult?) -> Swift.Void

protocol GMLocationPickerControllerDelegate: AnyObject {
    /// 提交所选位置
    func locationPickerController(_ collection: GMLocationPickerController, didSubmit locationInfo: GMLocationPickerLocationResult)
    
    /// 页面将要pop出
    func locationPickerControllerWillPop(_ collection: GMLocationPickerController)
}

/// 从哪里进来的 枚举类
enum GMLocationPickerFor {
    case warehouseCreate                                        // 仓库创建
    case warehouseEdit(warehouseInfo: GMWarehouseInfoHomeList)  // 仓库编辑
    case storeEdit(storeInfo: GMStoreInfoHomeModel)             // 门店编辑
}

/// 选择地理位置的原因 什么时候走高德接口
enum GMLocationPickerRegionChageFor {
    case located                                                // 实时定位
    case itemSelected                                           // 搜索项选择
    case keywordSearchSelected                                  // 关键词搜索 自动选择(第一个)
    case slideByCustomer                                        // 用户滑动逆地理
}
```
这里定义了可能需要的枚举类和协议。
枚举类主要跟我们自己业务有关系了，协议也是跟业务有关系。这里并不是实现地图的必须。

### 4.3 UI声明

```
class GMLocationPickerController: GMBaseViewController {

    //MARK: - property
    
    // UI
    
    private weak var btn_submit: UIButton!                      // 提交按钮
    private weak var mapView: MAMapView!                        // 地图
    private weak var textF_search: UITextField!                 // 搜索 输入栏
    private weak var tableView: TYTableView!                    // 搜索 列表
    private weak var ai_spinner: UIActivityIndicatorView!       // 加载loading
    private weak var btn_located: UIButton!                     // 定位按钮
    
    private weak var imgV_pointAnnotation: UIImageView!         // 自定义大头针
```

这里声明了实现地图用到的一些材料，当然最关键的是MAMapView了，这个View是高德提供的哦。

### 4.4 数据声明

```
// Data
var navTitle: String?                                       // Nav标题 (可选)
weak var delegate: GMLocationPickerControllerDelegate?      // 代理
var locationResultBlock: GMLocationResultHandler?

private var locationInfos = [GMLocationPickerLocationInfo]()// 搜索 地点信息
private var selectingLocationInfo: GMLocationPickerLocationInfo? // 选中地点信息

// Pre Data

var preLocationInfo: GMLocationPickerLocationInfo?          // 预地理位置信息
var preCoordinate: CLLocationCoordinate2D?                  // 预经纬度
var preKeyword: String?                                     // 预搜索关键词
var pickerFor: GMLocationPickerFor = .warehouseCreate       // 地图选择用于
```
这里大致列举了需要用的数据。
预数据，就是外部传过来的数据，然后我们新建了两个info实体报错搜索的地址和选中的地址。

### 4.5 其它成员声明

```
// Tool
private var search: AMapSearchAPI!                          // 地图 搜索功能
private var locationManager: AMapLocationManager!           // 地图 定位功能
private var authorizationManager = TYLocationAuthorization()// 权限管理

// flag

private var lastSearchKey = ""                              // 最后搜索字段
private var isRegionChangedFromCustomer = false             // 地图领域变化原因为用户手动滑动
private var isReGeoRequestForSubmit = false                 // 逆地理请求用于提交
private var viewDidAppear = false  
```

这里是一些工具和一些flag变量，工具包含地图搜索接口和地图定位功能工具，这些都是高德提供，另外权限是自定义的一个工具来处理的。

### 4.6 生命周期之viewDidLoad

这个走一次，看下做了什么：
```
override func viewDidLoad() {
        super.viewDidLoad()
        self.layoutSubviews()
        self.loadData()
    }
```
加载了子View，然后去加载数据。
```
private func layoutSubviews() {
        
        self.view.backgroundColor = .white
        
        // ------------------------- 导航栏 -------------------------
        let submitBtnF: UIFont = .pingFangRegular(size: 16.0)
        
        let btn_submit = UIButton()
        btn_submit.isEnabled = false
        btn_submit.ty_setButtonWith(text: "提交", font: submitBtnF, color: .ty_409EFF, state: .normal)
        btn_submit.ty_setButtonWith(text: "提交", font: submitBtnF, color: .ty_409EFF.alpha(0.5), state: .disabled)
        btn_submit.addTarget(self, action: #selector(act_submitBtnClicked(button:)), for: .touchUpInside)
        let item_submit = UIBarButtonItem(customView: btn_submit)
        self.navigationItem.rightBarButtonItem = item_submit
        self.btn_submit = btn_submit
        
        // ------------------------- 底部控件 -------------------------
        let searchListBGW: CGFloat = yScreenWidth
        let searchListBGH: CGFloat = 104.0
        let searchListBGR = CGRect(x: 0, y: 0, width: searchListBGW, height: searchListBGH)
        let searchListBGRadius = 16.0
        let searchListBGRadiusS = CGSize(width: searchListBGRadius, height: searchListBGRadius)
        
        let view_searchListBG = UIView.ty_viewWith(bgColor: .ty_ThemeWhite)
        view_searchListBG.ty_setCornerWith(rect: searchListBGR, corners: [.topLeft, .topRight], radii: searchListBGRadiusS)
        self.view.addSubview(view_searchListBG)
        view_searchListBG.snp.makeConstraints { make in
            make.bottom.equalTo(self.view.snp.bottom)
            make.centerX.equalTo(self.view)
            make.width.equalTo(searchListBGW)
            make.height.equalTo(searchListBGH)
        }
        
        // 搜索栏
        let searchBtnF: UIFont = .pingFangRegular(size: 14.0)
        let searchBtnW: CGFloat = 80.0
        let searchBtnH: CGFloat = 32.0
        let searchBarTextFF: UIFont = .pingFangRegular(size: 14.0)
        let searchBarTextFLeftImageS = CGSize(width: 16.0, height: 16.0)
        let searchBarTextFLeftViewS = CGSize(width: 32.0, height: 32.0)
        let searchBarTextFR: CGFloat = 10.0
        let searchBarTextFH: CGFloat = searchBtnH
        
        // 搜索按钮
        let btn_search = UIButton()
        btn_search.ty_setButtonWith(text: "搜索", font: searchBtnF, color: .ty_TextWhite, state: .normal)
        btn_search.ty_setBackgroundColor(.ty_409EFF, state: .normal)
        btn_search.ty_setCornerRadius(searchBtnH/2.0)
        btn_search.addTarget(self, action: #selector(act_searchBtnClicked(button:)), for: .touchUpInside)
        
        view_searchListBG.addSubview(btn_search)
        btn_search.snp.makeConstraints { make in
            make.top.equalTo(view_searchListBG).offset(ySuperVerPad)
            make.right.equalTo(view_searchListBG).offset(-ySuperHorPad)
            make.width.equalTo(searchBtnW)
            make.height.equalTo(searchBtnH)
        }
        
        // 搜索栏
        let textF_search = UITextField.ty_textFieldWith(font: searchBarTextFF, color: .ty_222222)
        textF_search.backgroundColor = .ty_F7F9FA
        textF_search.ty_setPlaceholderWith(placeholder: "请输入地址", color: .ty_9B9DA7, font: searchBarTextFF)
        textF_search.ty_setLeftPaddingImage(imageName: "locationPicker_img_search", imageSize: searchBarTextFLeftImageS, paddingSize: searchBarTextFLeftViewS)
        textF_search.ty_setCornerRadius(searchBarTextFH/2.0)
        
        textF_search.addTarget(self, action: #selector(act_searchTextFEditChange(textF:)), for: .editingChanged)
        textF_search.addObserver(self, forKeyPath: "text", options: [.new, .old], context: nil)
        
        textF_search.delegate = self
        textF_search.returnKeyType = .search
        
        view_searchListBG.addSubview(textF_search)
        self.textF_search = textF_search
        textF_search.snp.makeConstraints { make in
            make.top.equalTo(view_searchListBG).offset(ySuperVerPad)
            make.left.equalTo(view_searchListBG).offset(ySuperHorPad)
            make.right.equalTo(btn_search.snp.left).offset(-searchBarTextFR)
            make.height.equalTo(searchBarTextFH)
        }
        
        // TableView
        let tableViewT: CGFloat = 6.0
        
        let tableView = TYTableView(delegate: self, dataSource: self, style: .plain)
        tableView.showsVerticalScrollIndicator = false
        view_searchListBG.addSubview(tableView)
        self.tableView = tableView
        tableView.snp.makeConstraints { make in
            make.top.equalTo(textF_search.snp.bottom).offset(tableViewT)
            make.left.equalTo(view_searchListBG)
            make.right.equalTo(view_searchListBG)
            make.bottom.equalTo(view_searchListBG)
        }
        
        // loading 旋转图
        let indicatorStyle: UIActivityIndicatorView.Style
        if #available(iOS 13.0, * ) {
            indicatorStyle = .medium
        } else {
            indicatorStyle = .gray
        }
        let ai_spinner = UIActivityIndicatorView(style: indicatorStyle)
        ai_spinner.startAnimating()
        view_searchListBG.addSubview(ai_spinner)
        self.ai_spinner = ai_spinner
        ai_spinner.snp.makeConstraints { make in
            make.center.equalTo(view_searchListBG)
        }
        
        // ------------------------- mapView -------------------------
        let mapViewW: CGFloat = yScreenWidth
        let mapViewH: CGFloat = yScreenHeight - searchListBGH + searchListBGRadius/2.0
        let cus_mapViewR = CGRect(x: 0, y: 0, width: mapViewW, height: mapViewH)
        
        // MapView
        let mapView = MAMapView(frame: cus_mapViewR)
        mapView.zoomLevel = 13.0                // 缩放比例
        mapView.showsCompass = false            // 显示罗盘
        mapView.showsScale = false              // 显示比例尺
        mapView.delegate = self
        self.view.addSubview(mapView)
        self.mapView = mapView
        
        // 大头针
        let pointAnnotationImgVW: CGFloat = 40.0
        let pointAnnotationImgVH: CGFloat = 40.0
        let imgV_pointAnnotation = UIImageView.ty_imageViewWith(imageName: "locationPicker_icon_point_annotation")
        self.view.addSubview(imgV_pointAnnotation)
        self.imgV_pointAnnotation = imgV_pointAnnotation
        imgV_pointAnnotation.snp.makeConstraints { make in
            make.centerX.equalTo(mapView)
            make.centerY.equalTo(mapView).offset(-pointAnnotationImgVH/2.0)
            make.width.equalTo(pointAnnotationImgVW)
            make.height.equalTo(pointAnnotationImgVH)
        }
        
        // 回归定位按钮
        let locatedBtnWH: CGFloat = 46.0
        let locatedBtnR: CGFloat = 9.0
        let locatedBtnB: CGFloat = 9.0
        
        let btn_located = UIButton()
        btn_located.setImage(UIImage(named: "locationPicker_btn_located_Blue"), for: .selected)     // 在当前定位
        btn_located.setImage(UIImage(named: "locationPicker_btn_located_Black"), for: .normal)      // 不在当前定位
        btn_located.addTarget(self, action: #selector(act_locatedBtnClicked(button:)), for: .touchUpInside)
        self.view.addSubview(btn_located)
        self.btn_located = btn_located
        btn_located.snp.makeConstraints { make in
            make.right.equalTo(self.view).offset(-locatedBtnR)
            make.bottom.equalTo(view_searchListBG.snp.top).offset(-locatedBtnB)
            make.width.equalTo(locatedBtnWH)
            make.height.equalTo(locatedBtnWH)
        }
        
        self.view.bringSubviewToFront(view_searchListBG)
    }
```
逻辑也是非常清晰，一个一个把视图堆叠进去了。
这里大头针跟地图一点关系没有，这个只是把大头针放在中间而已。没有跟地图绑定任何关系。

然后去加载数据：
```
private func loadData() {
    // 设置标题
    if let navTitle = self.navTitle, !navTitle.isEmpty {
        self.title = "\(navTitle) 地图定位"
    } else {
        self.title = "地图定位"
    }
    
    // 定位功能
    // 更新App是否显示隐私弹窗的状态，隐私弹窗是否包含高德SDK隐私协议内容的状态
    AMapLocationManager.updatePrivacyShow(.didShow, privacyInfo: .didContain)
    // 更新用户授权高德SDK隐私协议状态
    AMapLocationManager.updatePrivacyAgree(.didAgree)
    let locationManager = AMapLocationManager()
    locationManager.desiredAccuracy = kCLLocationAccuracyHundredMeters
    if #available(iOS 14.0, *) {
        locationManager.locationAccuracyMode = .fullAccuracy
    }
    self.locationManager = locationManager
    
    // 搜索
    let search = AMapSearchAPI()
    search?.timeout = 10
    search?.delegate = self
    self.search = search
    
    // 预地理位置信息处理
    self.data_preLocatedHandle(locationInfo: self.preLocationInfo,
                                preCoordinate: self.preCoordinate,
                                preKeyword: self.preKeyword)
}
```
这里设置了标题，然后就去走预地理位置信息处理的方法。这里的pre相关的数据，在跳转前，就给这个controller赋值了，这里跟Android不太一样，Android还要通过intent来传，它这个居然可以直接用controller的实例传值。

具体预制方法为：
```
/// 根据预地理位置信息 按照优先级 决定预定位方式
func data_preLocatedHandle(locationInfo: GMLocationPickerLocationInfo? = nil,
                            preCoordinate: CLLocationCoordinate2D? = nil, preKeyword: String? = nil) {
    
    // 预定位处理规则
    // 完整数据 - 经纬度 - 位置关键词 - 实时定位 - 默认'北京'定位
    if let preLocationInfo = locationInfo {
        // 1.完整信息
        if let coordinate = preLocationInfo.coordinate,
            coordinate.latitude != 0, coordinate.longitude != 0 {
            // 1.1 经纬度信息完整
            self.data_changeRegionTo(preLocationInfo, for: .itemSelected)
            self.map_searchLocationForNearby()  // 搜索附近
        } else if !preLocationInfo.address.isEmpty {
            // 1.2 无经纬度、有地名：搜索地名
            let searchKey = "\(preLocationInfo.city)\(preLocationInfo.district)\(preLocationInfo.address)"
            self.data_preLocatedHandle(preKeyword: searchKey)
        } else {
            // 1.3 都没有：默认操作
            self.data_preLocatedHandle()
        }
    } else if let preCoordinate = preCoordinate {
        // 2.显示经纬度位置
        self.mapView.centerCoordinate = preCoordinate
        self.map_searchLocationForReGeo(isForSubmit: false) // 搜索逆地理编码
    } else if let preKeyword = preKeyword, preKeyword.count > 0 {
        // 3.关键词配置
        self.textF_search.text = preKeyword
        self.act_searchBtnClicked(button: nil)
    } else {
        var isLocationEnable = self.authorizationManager.isLocationAuthroizationEnable()
        if #available(iOS 14.0, *) {
            isLocationEnable = isLocationEnable && self.authorizationManager.isAccuracyAuthorizationFull()
        }
        if isLocationEnable {
            // 4.实时定位
            self.act_locatedBtnClicked(button: nil)
        } else {
            // 5.默认北京
            self.mapView.centerCoordinate = CLLocationCoordinate2D(latitude: 39.909115, longitude: 116.397535)
            self.map_searchLocationForReGeo(isForSubmit: false) // 搜索逆地理编码
        }
    }
}
```
这里就是一段逻辑，如果之前有地址，然后如何去显示的逻辑。如何去移动地图。
主要是用了一个 self.mapView.centerCoordinate来设置地图中心坐标，然后这里调用搜索词是走了这个方法：
```
/// 根据当前地图中间位置 搜索逆地理
/// - Parameter isForSubmit: 用于提交 or 用户滑动地理获取(顺带搜索附近)
private func map_searchLocationForReGeo(isForSubmit: Bool) {
    self.isReGeoRequestForSubmit = isForSubmit
    if isForSubmit {
        self.ty_showLoadingHUD()
    } else {
        // 清空列表数据
        self.data_reloadTableViewToEmpty(forRequest: true)
    }
    
    // 取消上一个请求
    self.search.cancelAllRequests()
    
    // 逆地理位置请求
    let coordinate = self.mapView.centerCoordinate
    let regeo = AMapReGeocodeSearchRequest()
    regeo.location = AMapGeoPoint.location(withLatitude: coordinate.latitude, longitude: coordinate.longitude)
    regeo.requireExtension = true
    self.search.aMapReGoecodeSearch(regeo)
}
```
这里根据地图中心位置调用了搜索接口,具体请求在这里：
```
self.search.aMapReGoecodeSearch(regeo)
```

如何搜索附近地址呢？
```
/// 根据选中位置 搜索附近POI
private func map_searchLocationForNearby() {
    guard let coordinate = self.selectingLocationInfo?.coordinate else { return }
    // 清空列表数据
    self.data_reloadTableViewToEmpty(forRequest: true)
    
    // 取消上一个请求
    self.search.cancelAllRequests()
    
    // 附近搜索请求
    let aroundRequest = AMapPOIAroundSearchRequest()
    let geoPoint = AMapGeoPoint.location(withLatitude: coordinate.latitude,
                                            longitude: coordinate.longitude)
    aroundRequest.location = geoPoint
//        aroundRequest.special = true
    self.search.aMapPOIAroundSearch(aroundRequest)
}
```
跟上面方法也比较类似。

### 4.7 生命周期之viewWillAppear

```
override func viewWillAppear(_ animated: Bool) {
        super.viewWillAppear(animated)
        setupWhiteNavBarColor()
        setupWhiteClickCallBack()
        
        isHiddenNavBar(isHidden: false)
    }
```
主要处理了导航栏相关的。

### 4.8 生命周期之viewDidAppear

```
override func viewDidAppear(_ animated: Bool) {
        super.viewDidAppear(animated)
        self.viewDidAppear = true
    }
```
这里没干啥，就设置了一个flag。

### 4.9 析构函数

```
deinit {
        self.tydev_logDeinit()
    }
```
这里也没干啥，打印下日志：
```
/// 输出对象销毁日志
func tydev_logDeinit() {
    self.tydev_logDeinit(nil)
}

/// 输出对象销毁日志
/// - Parameter mark: 自定义标识
func tydev_logDeinit(_ mark: String?) {
    if let mark = mark {
        print(">>>>>>>>>>>>>>> deinit view [\(self)] (\(mark) <<<<<<<<<<<<<<<")
    } else {
        print(">>>>>>>>>>>>>>> deinit view [\(self)] <<<<<<<<<<<<<<<")
    }
}
```

### 4.10 地图代理

前面我们设置了代理为自己，需要处理下：
```
// 地图Delegate - Map
extension GMLocationPickerController : MAMapViewDelegate {
    
    /// 地图区域改变完成后会调用此接口
    func mapView(_ mapView: MAMapView!, regionDidChangeAnimated animated: Bool) {
        if !self.viewDidAppear { return }
        
        // 大头针动画
        self.ui_pointAnnotationAnimate()
        if self.isRegionChangedFromCustomer, self.mapView.userTrackingMode == .none {
            // 用户手动滑动模式
            // 搜索逆地理编码(顺带搜索附近)
            self.map_searchLocationForReGeo(isForSubmit: false)
        }
        self.isRegionChangedFromCustomer = true
    }
}
```
这里是高德自己调用的，估计是滑动后区域改变，这里会走。
这里我们走完后需要顺带搜索下附近地址，所以需要加上我们自己逻辑哦。

### 4.11 搜索代理

```
// 地图Delegate - Search
extension GMLocationPickerController : AMapSearchDelegate {
    /// POI查询回调函数 [定位获取附近POI、搜索POI]
    func onPOISearchDone(_ request: AMapPOISearchBaseRequest!, response: AMapPOISearchResponse!) {
//        if response.pois.count <= 0 {
//            GMBannerTips.showWarningTips("没有找到定位")
//            data_reloadTableViewToEmpty(forRequest: false)
//            return
//        }
        
        let prefixPois = response.pois.prefix(20)     // 最多获取20个数据
        var locationInfos = prefixPois.map({ GMLocationPickerLocationInfo(mapPOI: $0) })
        
        if request.isKind(of: AMapPOIKeywordsSearchRequest.self) {
            // 1. 搜索关键词(搜索关键词)
            
            // 没有搜索到则提示
            if response.pois.count <= 0 {
                GMBannerTips.showWarningTips("没有找到定位")
                data_reloadTableViewToEmpty(forRequest: false)
                return
            }
            
            // 自动选择第一个位置
            var selectingInfo = self.selectingLocationInfo
            if let firstInfo = locationInfos.first {
                selectingInfo = firstInfo
                // 移动地图位置
                self.data_changeRegionTo(firstInfo, for: .keywordSearchSelected)
            }
            // 搜索POI距离补全
            if let selectingInfo = selectingInfo, let selectingCoordinate = selectingInfo.coordinate {
                let selectingPoint = MAMapPointForCoordinate(selectingCoordinate)
                for currLocationInfo in locationInfos {
                    if selectingInfo == currLocationInfo { continue }
                    if let currCoordinate = currLocationInfo.coordinate {
                        let currPoint = MAMapPointForCoordinate(currCoordinate)
                        currLocationInfo.distance = Int(MAMetersBetweenMapPoints(selectingPoint,currPoint))
                    }
                }
            }
            
        } else if request.isKind(of: AMapPOIAroundSearchRequest.self) {
            // 2. 搜索附近位置(定位or选中)
            
            // 没有找到定位 则选中当前选中的定位
            if prefixPois.count <= 0, let selectingInfo = self.selectingLocationInfo {
                locationInfos.insert(selectingInfo, at: 0)
            }
            
            // 则确保第一个选项为当前地理位置（不匹配则手动添加）
            if let firstInfo = locationInfos.first, let selectingInfo = self.selectingLocationInfo,
               firstInfo.poiName != selectingInfo.poiName, firstInfo.address != selectingInfo.address {
                // 当前第一个位置 非选中中位置
                locationInfos.insert(selectingInfo, at: 0)
            }
            if let firstInfo = locationInfos.first {
                // 纠正/同步选择中地理位置
                self.selectingLocationInfo = firstInfo
            }
        }
        
        // 刷新TableView数据
        self.data_reloadTableViewWith(locationInfos: locationInfos)
    }
    
    /// 逆地理编码回调 [手动滑动地图时用]
    func onReGeocodeSearchDone(_ request: AMapReGeocodeSearchRequest!, response: AMapReGeocodeSearchResponse!) {
        if self.isReGeoRequestForSubmit {
            // 1. 用于提交的逆地理编码搜索
            self.ty_hideLoadingHUD()
            self.isReGeoRequestForSubmit = false
            
            guard let location = request.location, let regeocode = response.regeocode else { return }
            
            self.view.endEditing(true)
            let selectingInfo = GMLocationPickerLocationResult(location: location, reGeocode: regeocode, locationInfo: self.selectingLocationInfo)
            
            // 提交地址
            self.delegate?.locationPickerController(self, didSubmit: selectingInfo)
            locationResultBlock?(selectingInfo)
            self.popViewController()
        } else {
            // 2. 用于用户滑动的逆地理编码搜索
            guard let location = request.location, let regeocode = response.regeocode else { return }
            
            // 设置 locationInfos
            let prefixPois = regeocode.pois.prefix(20)     // 最多获取20个数据
            var locationInfos = prefixPois.map({ GMLocationPickerLocationInfo(mapPOI: $0) })
            
            let selectingInfo = GMLocationPickerLocationInfo(location: location, reGeocode: regeocode)
            // 移动地图位置
            self.data_changeRegionTo(selectingInfo, for: .slideByCustomer)
            locationInfos.insert(selectingInfo, at: 0)
            
            // 刷新TableView数据
            self.data_reloadTableViewWith(locationInfos: locationInfos)
        }
    }
    
    /// 当请求发生错误 (timeout)
    func aMapSearchRequest(_ request: Any!, didFailWithError error: Error!) {
        let errorCode = error._code
        
        var errMsg = ""
        if let searchErrorCode = AMapSearchErrorCode(rawValue: errorCode) {
            switch searchErrorCode {
            case .notConnectedToInternet:   break// 网络连接异常
                //errMsg = AMapSearchError.errorInfo(with: searchErrorCode)
            default:
                break
            }
        } else {
            print("asdf")
        }
        if !errMsg.isEmpty {
            self.ty_showMsgHud(errMsg)
        }
    }
    
}
```
这里是高德提供的搜索代理。
一个是手动滑动，一个是自行搜索，走接口不一样哦。

### 4.12 其它代理

```

// TableView Delegate
extension GMLocationPickerController: UITableViewDelegate, UITableViewDataSource {
    
    func tableView(_ tableView: UITableView, numberOfRowsInSection section: Int) -> Int {
        return self.locationInfos.count
    }

    func tableView(_ tableView: UITableView, cellForRowAt indexPath: IndexPath) -> UITableViewCell {
        let cell = GMLocationPickerCell.cellWithTableView(tableView)
        cell.locationInfo = self.locationInfos[indexPath.row]
        return cell
    }
    
    func tableView(_ tableView: UITableView, heightForRowAt indexPath: IndexPath) -> CGFloat {
        return GMLocationPickerCell.cellHeight()
    }
    
    func tableView(_ tableView: UITableView, didSelectRowAt indexPath: IndexPath) {
        let locationInfo = self.locationInfos[indexPath.row]
        // 清除搜索栏内容
        self.ui_clearSearchTextF()
        // 移动地图位置
        self.data_changeRegionTo(locationInfo, for: .itemSelected)
        // 搜索附近
        self.map_searchLocationForNearby()
    }
}

// 输入框 & 自定义 Delegate
extension GMLocationPickerController : UITextFieldDelegate
{
    func textFieldShouldReturn(_ textField: UITextField) -> Bool {
        // 搜索
        self.act_searchBtnClicked(button: nil)
        return true
    }
}
```
这里还用了2个代理，一个是搜索列表的tableView的列表展示，一个是搜索框代理。

### 4.13 如何定位

```
/// 定位当前位置
private func map_requestLocation(_ completion: ((CLLocation?, AMapLocationReGeocode?) -> ())?) {
    // 定位请求
    func requestLocation(_ completion: ((CLLocation?, AMapLocationReGeocode?) -> ())?) {
        // 单次定位
        self.locationManager.requestLocation(withReGeocode: true, completionBlock: { (location: CLLocation?, reGeocode: AMapLocationReGeocode?, error: Error?) in
            // 定位回调
            if let completion = completion { completion(location, reGeocode) }
        })
    }
    
    let manager = self.authorizationManager
    manager.verifyLocationAuthroization(showAlertIn: self, completion: {
        if #available(iOS 14.0, *) {
            manager.verifyAccuracyAuthroization(completion: {
                requestLocation(completion)
            }, failure: nil)
        } else {
            requestLocation(completion)
        }
    }, failure: nil)
}
```
别慌，高德也是提供了方法，支持去定位。

### 4.14 地址权限工具

```
/// 地址权限管理器 配置
class TYLocationAuthorizationConfig: NSObject {
    
    // info.plist 上配置的 精读权限获取描述
    // <key>NSLocationTemporaryUsageDescriptionDictionary</key>
    var accuracyPurposeKey = "AccuracyUsageDescription"
    
    var locationAlertTitle = "您需要开启GPS权限"
    var locationAlertMessage = "地图定位需要获取您的GPS权限才能正常使用"
}

/// 地址权限管理器
class TYLocationAuthorization: NSObject {
    
    //MARK: - property
    
    // Data
    
    private var locationManager: CLLocationManager!             // 定位权限管理
    
    private var locationAuthVerifyCompletion: (() -> ())?       // 定位权限获取完成Block (用于权限首次询问)
    private var locationAuthVerifyFailure: (() -> ())?          // 定位权限获取失败Block (用于权限首次请求)
    
    private var config = TYLocationAuthorizationConfig()              // 文本相关配置
    
    // State
    
    /// 定位权限状态
    var locationAuthorization: CLAuthorizationStatus {
        if #available(iOS 14.0, *) {
            return self.locationManager.authorizationStatus
        } else {
            return CLLocationManager.authorizationStatus()
        }
    }
    
    /// 精读权限状态
    @available(iOS 14.0, *)
    var accuracyAuthorization: CLAccuracyAuthorization {
        return self.locationManager.accuracyAuthorization
    }
    
    
    //MARK: - life cycle
    
    convenience init(config: TYLocationAuthorizationConfig) {
        self.init()
        self.config = config
    }
    
    override init() {
        super.init()
        self.loadData()
    }
    
    private func loadData() {
        // 定位权限管理
        let locationManager = CLLocationManager()
        locationManager.delegate = self
        self.locationManager = locationManager
    }
    
    deinit {
        print(">>>>>>>>>>>>>>> deinit view [\(self)] <<<<<<<<<<<<<<<")
    }
    
    //MARK: - public methods
    
    // ------------------------- 权限请求封装 -------------------------
    
    /// 定位权限判断
    func verifyLocationAuthroization(showAlertIn controller: UIViewController? = nil,
                                     completion: (() -> ())? = nil,
                                     failure: (() -> ())? = nil) {

        switch self.locationAuthorization {
        case .notDetermined:                            // 1. 首次打开App时
            self.locationAuthVerifyCompletion = completion
            self.locationAuthVerifyFailure = failure
            self.locationManager.requestWhenInUseAuthorization()    // 使用时 *
        case .restricted, .denied:                      // 2. 父母限制 or 拒绝
            self.ui_showAlertForDeniedLocationAuthorization(in: controller)
            if let failure = failure { failure() }
        case .authorizedAlways, .authorizedWhenInUse:   // 3. 已授权
            if let completion  = completion { completion() }
        @unknown default:                               // 4. 未知情况
            self.ui_showAlertForDeniedLocationAuthorization(in: controller)
            if let failure = failure { failure() }
        }
    }
    
    /// 精读权限判断(高精读)
    @available(iOS 14.0, *)
    func verifyAccuracyAuthroization(completion: (() -> ())? = nil,
                                     failure: (() -> ())? = nil) {
        if self.isAccuracyAuthorizationFull() {
            if let completion = completion { completion() }
        } else {
            let key = self.config.accuracyPurposeKey
            self.locationManager.requestTemporaryFullAccuracyAuthorization(withPurposeKey: key) { err in
                if self.isAccuracyAuthorizationFull() {
                    if let completion = completion { completion() }
                } else {
                    if let failure = failure { failure() }
                }
            }
        }
    }
    
    // ------------------------- 权限状态 -------------------------
    
    /// 定位权限是否被拒绝
    func isLocationAuthroizationDenied() -> Bool {
        switch self.locationAuthorization {
        case .notDetermined:                            // 1. 首次打开App时
            return false
        case .restricted, .denied:                      // 2. 父母限制 or 拒绝
            return true
        case .authorizedAlways, .authorizedWhenInUse:   // 3. 已授权
            return false
        @unknown default:                               // 4. 未知情况
            return true
        }
    }
    
    /// 定位权限是否被允许
    func isLocationAuthroizationEnable() -> Bool {
        switch self.locationAuthorization {
        case .notDetermined:                            // 1. 首次打开App时
            return false
        case .restricted, .denied:                      // 2. 父母限制 or 拒绝
            return false
        case .authorizedAlways, .authorizedWhenInUse:   // 3. 已授权
            return true
        @unknown default:                               // 4. 未知情况
            return false
        }
    }
    
    /// 精准定位
    @available(iOS 14.0, *)
    func isAccuracyAuthorizationFull() -> Bool {
        return self.locationManager.accuracyAuthorization == .fullAccuracy
    }
    
    //MARK: - private methods
    
    /// 显示定位权限拒绝 Alert
    private func ui_showAlertForDeniedLocationAuthorization(in controller: UIViewController?) {
        guard let controller = controller else { return }

        let title = self.config.locationAlertTitle
        let message = self.config.locationAlertMessage
        let alertVC = UIAlertController.ty_doubleOptionAlertWith(title: title, message: message, optionTitle1: "去设置", action1: {
            if let settingURL = URL(string: UIApplication.openSettingsURLString),
                UIApplication.shared.canOpenURL(settingURL) {
                UIApplication.shared.open(settingURL, options: [:], completionHandler: nil)
            }
        }, optionTitle2: "取消", action2: nil)
        controller.present(alertVC, animated: true, completion: nil)
    }
    
    
    /// 清空 定位权限获取Block
    private func clearLocationAuthorizationBlock() {
        self.locationAuthVerifyCompletion = nil
        self.locationAuthVerifyFailure = nil
    }
}

extension TYLocationAuthorization : CLLocationManagerDelegate {
    
    /// 定位权限修改 (iOS 4.2 - 14.0)
    func locationManager(_ manager: CLLocationManager, didChangeAuthorization status: CLAuthorizationStatus) {
        self.data_locationAuthorizationDidChange()
    }
    
    /// 定位权限修改 (iOS 14.0+)
    func locationManagerDidChangeAuthorization(_ manager: CLLocationManager) {
        self.data_locationAuthorizationDidChange()
    }
    
    /// 权限状态修改
    private func data_locationAuthorizationDidChange() {
        if self.locationAuthVerifyCompletion == nil, self.locationAuthVerifyFailure == nil { return }
              
        switch self.locationAuthorization {
        case .notDetermined:                            // 1. 首次打开App时
            break
        case .restricted, .denied:                      // 2. 父母限制 or 拒绝
            if let failure = self.locationAuthVerifyFailure { failure() }
            self.clearLocationAuthorizationBlock()
        case .authorizedAlways, .authorizedWhenInUse:   // 3. 已授权
            if let completion = self.locationAuthVerifyCompletion { completion() }
            self.clearLocationAuthorizationBlock()
        @unknown default:                               // 4. 未知情况
            if let failure = self.locationAuthVerifyFailure { failure() }
            self.clearLocationAuthorizationBlock()
        }
        
    }
}
```
这里需要考虑到多种情况，是否拒绝权限，是否拥有了权限以及不同手机系统的问题。
用法也是相当简单：
```
let manager = self.authorizationManager
manager.verifyLocationAuthroization(showAlertIn: self, completion: {
    if #available(iOS 14.0, *) {
        manager.verifyAccuracyAuthroization(completion: {
            requestLocation(completion)
        }, failure: nil)
    } else {
        requestLocation(completion)
    }
}, failure: nil)
```
这个manager就是我们的工具类。

大致就是这么多了。

## 5 总结

* 如果要做一个需要三方支持的东西，请务必要提前调研下，如地图，推送，埋点等，这些最后提前了解下三方的接入指南，这样更好去实现需求。

* 高德地图对接非常简单，拿到key后，主要工作就是对Map的处理，用户行为二指缩放这种高德自行实现，我们只需要设置坐标给中间的位置即可。

* 对于地图，一定要考虑是否有权限的情况，没有权限的情况也需要跳转到系统设置，这样才能加强用户体验。万万不可出现没有权限直接闪退的问题。

* 一般做地图都会有一个搜索能力，这样就需要高德搜索接口了，需要了解AMapSearchAPI 这个接口的用法，定位的话就用到AMapLocationManager。