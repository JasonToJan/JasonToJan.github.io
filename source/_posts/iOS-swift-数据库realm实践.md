---
title: iOS swift 数据库realm实践
date: 2023-01-30 16:38:21
top: false
cover: false
toc: true
mathjax: true
tags:
- 数据库 iOS
categories:
- iOS
---

## 1 需求定义

需要获取服务端的条码数据库，存放到本地数据库。

背景：集成了扫码功能，扫到的条码头需要匹配数据库中的条码列表，可以认为这个是一个离线数据库。不会实时判断，因为条码很多，这里先获取所有的条码，存放到本地数据库。然后扫码就按照数据库中的关系表来匹配就好了。

这里我们有两个表，一个商品库表，一个是关系表。为什么有2个表呢？因为我们有配套的关系，2个零件可以看做单个商品，他们组合了也可以看成单独商品哦，这种就是套机了。想必做个电商的人都懂这个道理。比如可以买一件上衣，一条裤子，它们都是独立商品，也可以配套组合一套商品，而且这一套也可以认为是一个商品。


## 2 需求分析

这里我们采用 realm数据库。
realm非常简介，效率也是不错的。
github地址：[https://github.com/realm/realm-swift](https://github.com/realm/realm-swift)

简单描述下realm吧：Realm 是一个跨平台的移动数据库引擎，其性能要优于 Core Data 和 FMDB - 移动端数据库性能比较, 我们可以在 Android 端 realm-java，iOS端:Realm-Cocoa，同时支持 OC 和 Swift两种语言开发。其使用简单，免费，性能优异，跨平台的特点广受关注。

这里我们会用到2个表。

因为这是个离线数据库，我们需要考虑数据库变更，与服务端同步问题。
所以我们需要一个接口，判断有无新增或者变更的数据。

需要考虑到增量更新，同时如果数据变更大于一个阙值，比如1000条数据，我们可以认为是全量更新。这时候全量更新的时间和增量插入数据时间几乎相同。

另外如果第一次获取数据，数据量可能会很大，我们需要考虑压缩，与服务端商议，将json数据压缩，然后我们自行解析为实体类。

另外增量更新的时候，要特别注意改数据是删除了还是新增的，还是变更的。
如果商品删除了，关系表也是要删除的哦。这点很重要，不然后期会导致匹配异常。


## 3 准备工作，会用Realm

因为是从0开始，建议熟悉一下Realm使用方法。
可以参考下这篇博客：[https://juejin.cn/post/6844904117442215944](https://juejin.cn/post/6844904117442215944),写得还是相当不错，有Demo，讲述得非常清晰易懂。

这里就不讲解如何引用这个Realm了。

这里需要会用一个工具：Realm Studio，
下载地址：[https://www.mongodb.com/docs/realm/studio/install/](https://www.mongodb.com/docs/realm/studio/install/)
这里按照自己电脑版本下载相应软件就好了，这个也是免费的。

如何分析真机里面的realm数据库呢？
可以参考下：[https://blog.csdn.net/asfasnjn/article/details/124714242](https://blog.csdn.net/asfasnjn/article/details/124714242) 很简单，打开XCode的window下的设备和模拟器，再 点击更多图标，再下载Container，然后下载后，查看包内容，就可以看到这个应用的realm文件了。然后可以右键，用我们下载好的Realm Studio来查看这个文件，文件以表格形式展示，清晰直观。

## 4 初始化配置数据库

这里需要在AppDelegate中配置下realm数据库。
很简单的。
不过我们没有在AppDelegate初始化，因为我们这个绑定了userId，等用户登录后才初始化，当然也是可以的。

重点是初始化这个动作要先于使用就行了。
```
func configRealm(userID: String?,
                        keyWord: String? = nil,
                        schemaVersion: UInt64 = 2, migrationBlock: MigrationBlock? = nil) {
    // 加密串128位结果为：464e5774625e64306771702463336e316a4074487442325145766477335e21346b715169364c406c6a4976346d695958396245346e356f6a62256d2637566126
    // let key: Data = "FNWtb^d0gqp$c3n1j@tHtB2QEvdw3^!4kqQi6L@ljIv4miYX9bE4n5ojb%m&7Va&".data(using: .utf8, allowLossyConversion: false)!
    // 加密的key
    // let key: Data = keyWord.data(using: .utf8, allowLossyConversion: false)!
    // 打开加密文件
    // (encryptionKey: key)
    // 使用默认的目录，但是使用用户 ID 来替换默认的文件名
    
    //schemaVersion 数据库版本号从0开始
    let manager = FileManager.default
    let document = manager.urls(for: .documentDirectory, in: .userDomainMask)
    let url = document[0] as URL
    let urlStr = url.absoluteString
    let path = urlStr + "/" + ("\(userID!).realm")
    let config = Realm.Configuration(fileURL: URL(string: path), schemaVersion: schemaVersion, migrationBlock: { (migration, oldSchemaVersion) in
        // 目前我们还未进行数据迁移，因此 oldSchemaVersion == 0
        if oldSchemaVersion < 100 {
            // 什么都不要做！Realm 会自行检测新增和需要移除的属性，然后自动更新硬盘上的数据库架构
            // 属性重命名  migration.renameProperty(onType: Person.className(), from: "yearsSinceBirth", to: "age")
            print(oldSchemaVersion)
            print("end")
        }
        // 低版本的数据库迁移......
        if migrationBlock != nil {
            migrationBlock!(migration, oldSchemaVersion)
        }
    })
    // 告诉 Realm 为默认的 Realm 数据库使用这个新的配置对象
    Realm.Configuration.defaultConfiguration = config
    guard let realm = try? Realm(configuration: config) else {
        return
    }
    currentSchemaVersion = schemaVersion
    currentRealm = realm
    currentKeyWord = keyWord
}
```
这个realm实际上以userId为名称了。
这样就有个弊端了，切换用户后，需要重新下载数据库，特别是数据库特别大的情况下，这对用户体验不是特别好。


## 5 走接口拿数据

这里就是拿商品数据了，这里服务端提供的接口，返回的是加密后的字符串。
类似这样：
```
{
  "message" : "请求成功",
  "code" : 1000,
  "data" : "H4sIAAAAAAAAAKtWSiwoCCjKTylNLglKzQnz98ksLlGyio7VQZJAFk3OzytJzQ
  NyDA2NdJRKC1IS\nS1JDMnNTlayUjAyMjHUNDHWNLBUMTaxMzaxMDJVqAQ0XaZphAAAA"
}
```

比如我们接口是这样请求的，这里我们先扩展了一个方法给控制器：
```
extension GMBaseViewController {
    
    open func updateOfflinePackageData(tips: String = "", successful: @escaping Handler) {
        MBProgressHUD.showMessage(tips)
        OfflinePackageManager.update {
            successful()
            MBProgressHUD.hide()
            MBProgressHUD.hide()
        } failure: {
            MBProgressHUD.hide()
            MBProgressHUD.hide()
            let alerVC = UIAlertController(
                title: "",
                message: "检测到商品信息变动，系统为您自动更新商品信息失败，请在网络良好的环境下，手动更新商品信息",
                preferredStyle: .alert
            )
            alerVC.addAction(
                UIAlertAction(
                    title: "下载商品信息",
                    style: .default
                ) { (action) in
                    self.updateOfflinePackageData(successful: successful)
                }
            )
            self.present(alerVC, animated: true, completion:nil)
        }
        
    }
    
}
```
这里我们的OfflinePackageManager就是我们封装好的专门去下载离线库的工具类。
成功后走successful的逃逸闭包，失败弹一个弹框。

继续走进去：
```
class OfflinePackageManager: NSObject {
    
    open class func update(
        updateTime: String = Defaults[key: DefaultsKeys.updateTime] ?? "",
        successful: @escaping Handler,
        failure: @escaping Handler
    ) {
        let endTimeStr = Defaults[key: DefaultsKeys.updateTime]
        let updateTimeStr = updateTime
       
        productProvider.requestJson(.getProductMetadataForAPP(updateTime: updateTimeStr)) { response in
            let jsonString = JSON(rawValue: response).jsonString
            let result = GMBaseRequestModel.deserialize(from: jsonString)
            if result?.code == 1000 {
                guard let data = result?.data?.gunzip?.data(using: .utf8) else { return }
                
                let file = "zpi_offline_package_data.txt"
                let text = result?.data?.gunzip ?? ""
                
                if let dir = FileManager.default.urls(for: .documentDirectory, in: .userDomainMask).first {
                    let fileURL = dir.appendingPathComponent(file)
                    do {
                        try text.write(to: fileURL, atomically: false, encoding: .utf8)
                    } catch {}
                }
                
                guard let model = try? JSONDecoder().decode(StageOfflinePackageData.self, from: data) else { return }
                
                if endTimeStr != nil && (model.appProductVOList.count >= 1000 || model.appProductRelVOList.count >= 1000) {
                    Defaults[key:DefaultsKeys.updateTime] = nil
                    OfflinePackageManager.update(updateTime: "", successful: successful, failure: failure)
                } else {
                    if endTimeStr == nil {
                        OfflinePackageManager.fullUpdate(model: model)
                    } else {
                        OfflinePackageManager.incrementalUpdate(model: model)
                    }
                    Defaults[key: DefaultsKeys.updateTime] = model.updateTime
                    successful()
                }
            } else {
                failure()
            }
        } failure: { _ in
            failure()
        }
    }
```

## 6 解压缩字符串

这里服务端返回的一段加密后的字符串：
```
{
  "message" : "请求成功",
  "code" : 1000,
  "data" : "H4sIAAAAAAAAAKtWSiwoCCjKTylNLglKzQnz98ksLlGyio7VQZJAFk3OzytJzQNyDA2NdJRKC1IS\nS1JDMnNTlayUjAyMjHUNDHWNLBUMTaxMzaxMDJVqAQ0XaZphAAAA"
}
```
不过返回体还是json，我们先拿data里面的字符串。
再去解压缩。

字符串解压缩可以参考这篇文章：[https://www.cnblogs.com/strengthen/p/10844466.html](https://www.cnblogs.com/strengthen/p/10844466.html)

这里我们用到了GzipSwift框架。

这个框架地址为：[https://github.com/1024jp/GzipSwift](https://github.com/1024jp/GzipSwift) 还是有挺多人star的。

具体细节就不讲解了。

这里我们自己给String做了一个扩展，这样直接能拿到解压缩后的字符串：
```
extension String {
    var gunzip: String? {
        if let data = Data(base64Encoded: self, options: .ignoreUnknownCharacters) {
            if data.isGzipped {
                if let gun = try? data.gunzipped() {
                    return String(data: gun, encoding: String.Encoding.utf8)
                }
            }
        }
        return nil
    }
}
```

这里解析后的字符串为：
```
{
    "appProductRelVOList":[],
    "appProductVOList":[],
    "content":112,
    "updateTime":"2023-01-29 14:56:41"
}
```

同样也是一个json数组哦。
所以我们还是得用Json解析下：
```
guard let model = try? JSONDecoder().decode(StageOfflinePackageData.self, from: data)
                else { return }
```

这时候我们需要一个实体来接收下：
```
class StageOfflinePackageData:Codable{
    
    var appProductVOList: [StageGoodsPackgeModelData] = []
    
    var appProductRelVOList: [StageGoodsSuitPackgeModelData] = []
    
    var updateTime: String = ""
    
    
    private enum Codingkeys: String, CodingKey{
        case appProductVOList,appProductRelVOList,updateTime
    }
    
    func encode(to encoder: Encoder) throws {}
    
    required init(from decoder: Decoder) throws {
        let values = try decoder.container(keyedBy: Codingkeys.self)
        appProductVOList = try values.decode(Array.self, forKey: .appProductVOList)
        appProductRelVOList = try values.decode(Array.self, forKey: .appProductRelVOList)
        updateTime = try values.decode(String.self, forKey: .updateTime)
    }
}
```
为啥要继承Codable呢？
Codable 协议在 Swift4.0 开始被引入，目标是取代现有的 NSCoding 协议，它对结构体，枚举和类都支持。Codable 的引入简化了JSON 和 Swift 类型之间相互转换的难度，能够把 JSON 这种弱类型数据转换成代码中使用的强类型数据。具体可以看这篇文章：[https://juejin.cn/post/6971997599725256734](https://juejin.cn/post/6971997599725256734)。


解析后就拿到服务端的数据了，可能是增量数据，可能是全量数据。后面我们具体分析下。


## 7 数据处理

解析完毕后，我们有一段逻辑是这样的：
```
if endTimeStr != nil && (model.appProductVOList.count >= 1000 || model.appProductRelVOList.count >= 1000) {
                    Defaults[key:DefaultsKeys.updateTime] = nil
                    OfflinePackageManager.update(updateTime: "", successful: successful, failure: failure)
                } else {
                    if endTimeStr == nil {
                        OfflinePackageManager.fullUpdate(model: model)
                    } else {
                        OfflinePackageManager.incrementalUpdate(model: model)
                    }
                    Defaults[key: DefaultsKeys.updateTime] = model.updateTime
                    successful()
                }
```

这里如果数据大于1000条，我们重新走全量更新接口。
如果上次更新时间为空，我们默认为全量更新，将所有数据插入数据库。
如果上次更新时间不为空，而且小于1000条，我们就走增量更新逻辑。

### 7.1 Model层定义

这里是定义的Realm的基础模型。

我们需要定义两个实体，一个是商品模型，一个是关系模型。

商品模型为：
```

class StageGoodsPackgeModelData: Object,Codable{
    
    @objc dynamic var barcode: String = ""
    @objc dynamic var brand: String = ""
    @objc dynamic var brandId: String = ""
    @objc dynamic var category: String = ""
    @objc dynamic var categoryId: Int = 0
    @objc dynamic var createBy: String = ""
    @objc dynamic var createTime: String = ""
    @objc dynamic var delFlag: Int = 0
    @objc dynamic var groupNum: Int = 1         //返回为""时服务器需要传1
    @objc dynamic var id: String = ""
    @objc dynamic var isGreeProduct: Bool = false
    @objc dynamic var isGroupedProduct: Bool = false
    @objc dynamic var isSaleProduct: Bool = false
    @objc dynamic var length: Double = 0.0
    @objc dynamic var name: String = ""
    @objc dynamic var orgId: String = ""
    @objc dynamic var skuCode: String = ""
    @objc dynamic var tall: Double = 0
    @objc dynamic var unit: String = ""
    @objc dynamic var updateBy: String = ""
    @objc dynamic var updateTime: String = ""
    @objc dynamic var volume: Double = 0.0
    @objc dynamic var weight: Double = 0.0
    @objc dynamic var wide: Double = 0.0
    @objc dynamic var isMarketingProduct :Bool = false
    
    private enum Codingkeys: String, CodingKey{
        case barcode
        ,brand
        ,brandId
        ,category
        ,categoryId
        ,createBy
        ,createTime
        ,delFlag
        ,groupNum
        ,id
        ,isGreeProduct
        ,isGroupedProduct
        ,isSaleProduct
        ,length
        ,name
        ,orgId
        ,skuCode
        ,tall
        ,unit
        ,updateBy
        ,updateTime
        ,volume
        ,weight
        ,wide,
        isMarketingProduct
    }
    
    
    required init(from decoder: Decoder) throws {
        let values = try decoder.container(keyedBy: Codingkeys.self)
        
        do {
            barcode = try values.decode(String.self, forKey: .barcode)
        } catch {
            barcode = ""
        }
        do {
            brand = try values.decode(String.self, forKey: .brand)
        } catch {
            brand = ""
        }
        do {
            brandId = try values.decode(String.self, forKey: .brandId)
        } catch {
            brandId = ""
        }
        do {
            category = try values.decode(String.self, forKey: .category)
        } catch {
            category = ""
        }
        do {
            categoryId = try values.decode(Int.self, forKey: .categoryId)
        } catch {
            categoryId = 0
        }
        do {
            createBy = try values.decode(String.self, forKey: .createBy)
        } catch {
            createBy = ""
        }
        do {
            createTime = try values.decode(String.self, forKey: .createTime)
        } catch {
            createTime = ""
        }
        do {
            delFlag = try values.decode(Int.self, forKey: .delFlag)
        } catch {
            delFlag = 0
        }
        do {
            groupNum = try values.decode(Int.self, forKey: .groupNum)
        } catch {
            groupNum = 1
        }
        do {
            id = try values.decode(String.self, forKey: .id)
        } catch {
            id = ""
        }
        do {
            isGreeProduct = try values.decode(Bool.self, forKey: .isGreeProduct)
        } catch {
            isGreeProduct = false
        }
        do {
            isGroupedProduct = try values.decode(Bool.self, forKey: .isGroupedProduct)
        } catch {
            isGroupedProduct = false
        }
        do {
            isSaleProduct = try values.decode(Bool.self, forKey: .isSaleProduct)
        } catch {
            isSaleProduct = false
        }
        do {
            length = try values.decode(CGFloat.self, forKey: .length)
        } catch {
            length = 0.0
        }
        do {
            name = try values.decode(String.self, forKey: .name)
        } catch {
            name = ""
        }
        do {
            orgId = try values.decode(String.self, forKey: .orgId)
        } catch {
            orgId = ""
        }
        do {
            skuCode = try values.decode(String.self, forKey: .skuCode)
        } catch {
            skuCode = ""
        }
        do {
            tall = try values.decode(Double.self, forKey: .tall)
        } catch {
            tall = 0
        }
        do {
            unit = try values.decode(String.self, forKey: .unit)
        } catch {
            unit = ""
        }
        do {
            updateBy =  try values.decode(String.self, forKey: .updateBy)
        } catch {
            updateBy = ""
        }
        do {
            updateTime = try values.decode(String.self, forKey: .updateTime)
        } catch {
            updateTime = ""
        }
        do {
            volume = try values.decode(Double.self, forKey: .volume)
        } catch {
            volume = 0.0
        }
        do {
            weight = try values.decode(Double.self, forKey: .weight)
        } catch {
            weight = 0.0
        }
        do {
            wide = try values.decode(Double.self, forKey: .wide)
        } catch {
            wide = 0.0
        }
        do {
            isMarketingProduct = try values.decode(Bool.self, forKey: .isMarketingProduct)
        } catch {
            isMarketingProduct = false
        }
    }
    
    func encode(to encoder: Encoder) throws {}
    
    override required init(){}
}
```

另外一个为：
```

class StageGoodsSuitPackgeModelData: Object,Codable{

    @objc dynamic var groupNum: Int = 0
    @objc dynamic var id: String = ""
    @objc dynamic var orgId: String = ""
    @objc dynamic var productId: String = ""
    @objc dynamic var subProductId: String = ""
    
    private enum Codingkeys: String, CodingKey {
        case groupNum
        case id
        case orgId
        case productId
        case subProductId
    }

    required init(from decoder: Decoder) throws {
        let values = try decoder.container(keyedBy: Codingkeys.self)
        
        do {
            groupNum = try values.decode(Int.self, forKey: .groupNum)
        } catch {
            groupNum = 0
        }
        do {
            id = try values.decode(String.self, forKey: .id)
        } catch {
            id = ""
        }
        do {
            orgId = try values.decode(String.self, forKey: .orgId)
        } catch {
            orgId = ""
        }
        do {
            productId = try values.decode(String.self, forKey: .productId)
        } catch {
            productId = ""
        }
        do {
            subProductId = try values.decode(String.self, forKey: .subProductId)
        } catch {
            subProductId = ""
        }
    }

    func encode(to encoder: Encoder) throws {}
    
    override required init(){}
}
```

### 7.2 全量更新

```
private class func fullUpdate(model: StageOfflinePackageData) {
        let suitResult = objectWithAll(objct: StageGoodsPackgeModelData.self)
        if suitResult.count != 0 {
            deleteList(suitResult) {
                print("StageGoodsPackgeModelData表数据清除成功")
            }
        }
        addList(model.appProductVOList) {
            print("StageGoodsPackgeModelData 插入成功")
        }
        
        let relationResult = objectWithAll(objct: StageGoodsSuitPackgeModelData.self)
        if relationResult.count != 0 {
            deleteList(relationResult) {
                print("StageGoodsSuitPackgeModelData表数据清除成功")
            }
        }
        addList(model.appProductRelVOList) {
            print("StageGoodsSuitPackgeModelData 插入成功")
        }
    }
```
如果是全量更新就比较简单了。
先把之前的那两个删除掉，然后新增新的model就好了。

### 7.3 增量更新

先声明一下sql
```
    private class func incrementalUpdate(model: StageOfflinePackageData) {
        print("增量处理开始时间---->", Date().currentTime(0))
        var addGoodsArr: [StageGoodsPackgeModelData] = []
        var addRelationsArr: [StageGoodsSuitPackgeModelData] = []
        var relationidArr: [String] = []
        
        var goodsSql = ""
        var relationSql = ""
        var deleteGoodsSql = ""
        var deleteRelationSql = ""
```

遍历下服务端返回的增量数据库：
```
for (_,suitModel) in model.appProductVOList.enumerated() {
    // delFlag为0表示，不是删除哦，这里是新增商品
    if suitModel.delFlag == 0 {
        if goodsSql == "" {
            let str1 = String(format:"id = '%@'", suitModel.id)
            let str2 = String(format:"productId = '%@'", suitModel.id)
            goodsSql.append(str1)
            relationSql.append(str2)
        } else {
            let str1 = String(format:"|| id = '%@'", suitModel.id)
            let str2 = String(format:"|| productId = '%@'", suitModel.id)
            goodsSql.append(str1)
            relationSql.append(str2)
        }
        addGoodsArr.append(suitModel)
        relationidArr.append(suitModel.id)
    } else {
        // delFlag为1表示删除了商品
        if deleteGoodsSql == "" {
            let str1 = String(format:"id = '%@'", suitModel.id)
            let str2 = String(format:"productId = '%@'", suitModel.id)
            deleteGoodsSql.append(str1)
            deleteRelationSql.append(str2)
        } else {
            let str1 = String(format:"|| id = '%@'", suitModel.id)
            let str2 = String(format:"|| productId = '%@'", suitModel.id)
            deleteGoodsSql.append(str1)
            deleteRelationSql.append(str2)
        }
    }
}
```
有个一个delFlag变量，就是表示是否删除的意思。
如果是删除的话，将sql组合一下，就成为 id = '12' || id = '13' || id = '14' 这种。关系表就是 productId = '12' || productId = '14' 这种。


这里就知道哪些商品要删除，哪些要加。哪些关系表要删除，哪些要加。

这里先处理下新增的商品：
```
if goodsSql != "" {
    let predicate = NSPredicate(format:goodsSql)
    let suitArr = objectsWithPredicate(object: StageGoodsPackgeModelData.self, predicate: predicate)
    // 这里需要把可能之前存在相同的去掉
    deleteList(suitArr){
        print("删除商品成功")
    }
    if addGoodsArr.count != 0 {
        addList(addGoodsArr){
            print("增量更新商品数据成功")
        }
    }
}
```
这里用了系统的NSPredicate类来组合sql语句。它其实是用来过滤数据的类。
使用方法可以参考这篇文章：[https://juejin.cn/post/6844903540805074951](https://juejin.cn/post/6844903540805074951)。

然后处理下要删除的商品：
```
 if deleteGoodsSql != "" {
    // NSPredicate是swift 过滤数据的类，类似数据库中的where字段
    let predicate = NSPredicate(format:deleteGoodsSql)
    let suitArr = objectsWithPredicate(object: StageGoodsPackgeModelData.self, predicate: predicate)
    deleteList(suitArr){
        print("删除商品成功")
    }
}
```

商品处理完成，剩下就是关系表了。

这里先处理要新增的关系：
```
// 这里是新增的关系表
for idStr in relationidArr {
    for relationModel in model.appProductRelVOList{
        if idStr == relationModel.productId {
            addRelationsArr.append(relationModel)
        }
    }
}

// 新增的关系表
if relationSql != "" {
    let relationPredicate = NSPredicate(format:relationSql)
    let relationArr = objectsWithPredicate(object: StageGoodsSuitPackgeModelData.self, predicate: relationPredicate)
    deleteList(relationArr){
        print("删除套机关系模型成功")
    }
    if addRelationsArr.count != 0 {
        addList(addRelationsArr){
            print("增量更新套机关系模型成功")
        }
    }
}
```

然后处理下要删除的关系表：
```
 if deleteRelationSql != "" {
    let relationPredicate = NSPredicate(format:deleteRelationSql)
    let relationArr = objectsWithPredicate(object: StageGoodsSuitPackgeModelData.self, predicate: relationPredicate)
    deleteList(relationArr){
        print("删除套机关系模型成功")
    }
}
```

这样增量数据就成功插入了。

## 8 预览realm数据库

插入数据成功后，我们可以用 Realm Studio 来预览数据。
上面有讲解如何使用这个可视化工具。

这里我们就看下最终的结果吧：
<img src=realm1.png>
<img src=realm2.png>

## 9 总结

* iOS数据库可以考虑使用realm数据库，这个是跨平台，Android也有相应的三方库，使用非常简洁，可以考虑。

* 对于大量数据，我们务必要注意数据的修改，新增，最好采用增量更新的方式，而且如果全量，最好是先压缩下，减小对网络依赖。

* realm数据库要注意数据库迁移的问题，升级的问题，使用前需要configRealm。

* 真机预览realm数据库可以使用Realm Studio。

* 增量更新一定要注意删除之前的记录，否则数据会很紊乱。
