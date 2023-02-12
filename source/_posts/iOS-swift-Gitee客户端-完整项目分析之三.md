---
title: iOS swift Gitee客户端 完整项目分析之三
date: 2023-02-12 15:39:14
op: false
cover: false
toc: true
mathjax: true
tags:
- 完整项目 SwiftUI
categories:
- iOS
---

## 1 项目地址
> [https://gitee.com/open-gitee/gitee_ios](https://gitee.com/open-gitee/gitee_ios) 
Gitee基于SwiftUI和OpenApi的iOS客户端项目。


## 2 设置页面

首先看下设置tab页效果：
<img src=gitee_03_01.png width=50%>

### 2.1 入口

```Swift
SettingView()
            .tabItem {
                Image(systemName:"gearshape")
                Text("设置")
            }
```

这里是首页的tabView里面配置的。

### 2.2 SettingView
首先是定义的变量：
```Swift
struct SettingView: View {
    @State var isQrcodeShow = false
    @State var isLoginShow = false
    @State var isModal = false
    @State var userAccount = "hamm"
    @State var userName = "请先登录"
    @State var userBio = "你需要登录后才能访问"
    @State var userFollowers = "0"
    @State var userFollowing = "0"
    @State var userStars = "0"
    @State var userWatches = "0"
    @State var userHead:UIImage? = nil
    let placeholderImage = UIImage(named: "nohead")!
    
    @State var isActiveFollowing:Bool = false
    @State var isActiveFollowers:Bool = false
    @State var isActiveStars:Bool = false
    @State var isActiveWatches:Bool = false
```

然后是body区：
```Swift
var body: some View {
    ScrollView{
        NavigationLink(destination: UserFollowersView(), isActive: $isActiveFollowers) { EmptyView() }
        NavigationLink(destination: UserFollowingView(), isActive: $isActiveFollowing) { EmptyView() }
        NavigationLink(destination: RepoView(urlPath: self.userAccount + "/starred", showListFrom: ShowRepoListFrom.fromStars), isActive: $isActiveStars) { EmptyView() }
        NavigationLink(destination: RepoView(urlPath: self.userAccount + "/subscriptions", showListFrom: ShowRepoListFrom.fromWatches), isActive: $isActiveWatches) { EmptyView() }
        NavigationLink(destination:
                        SettingQrcode(userAccount:  self.userAccount), isActive: $isQrcodeShow) { EmptyView() }
```
这里定义了跳转逻辑。

首先顶层是一个VStack:
``` 
Vstack{

}
```
然后是一个HStack:
```
HStack(alignment: .center){
        Image(uiImage: self.userHead ?? placeholderImage)
            .resizable()
            .scaledToFit()
            .frame(
                width:80,height:80,
                alignment: .center
            )
            .cornerRadius(80)
        VStack(alignment: .leading){
            Text(self.userName).foregroundColor(Color(hex: 0xFFFFFF))
                .padding(.bottom, 2).font(.system(size:24)).lineLimit(1)
            Text(self.userBio).foregroundColor(Color(hex: 0x999999)).lineLimit(2)
                .font(.system(size:14))
        }
        .padding(.horizontal, 5.0)
        Spacer()
        Image(systemName: "qrcode")
            .scaleEffect(1.2, anchor: .center)
            .padding(.top,10)
            .onTapGesture {
                self.isQrcodeShow.toggle()
            }
}
.padding(.horizontal, 10.0)
.padding(.vertical, 10)
```
这个是顶部用户图像和名称显示。

然后是中间4个区块：
```Swift
HStack{
    VStack{
        Text(self.userFollowing).font(.system(size:20))
            .fontWeight(.bold)
        Text("关注").foregroundColor(Color(hex: 0x999999)).font(.system(size:12))
    }
    .onTapGesture {
        self.isActiveFollowing = true
    }
    Spacer()
    VStack{
        Text(self.userFollowers).font(.system(size:20))
            .fontWeight(.bold)
        Text("粉丝").foregroundColor(Color(hex: 0x999999)).font(.system(size:12))
    }
    .onTapGesture {
        self.isActiveFollowers = true
    }
    Spacer()
    VStack{
        Text(self.userStars).font(.system(size:20))
            .fontWeight(.bold)
        Text("Star").foregroundColor(Color(hex: 0x999999)).font(.system(size:12))
    }
    .onTapGesture {
        self.isActiveStars = true
    }
    Spacer()
    VStack{
        Text(self.userWatches).font(.system(size:20))
            .fontWeight(.bold)
        Text("Watch").foregroundColor(Color(hex: 0x999999)).font(.system(size:12))
    }
    .onTapGesture {
        self.isActiveWatches = true
    }
}
.padding(.vertical,10)
.padding(.horizontal,20)
```

然后是下方的列表：
```Swift
 VStack{
    NavigationLink(destination:  PubKeysView()) {
        SettingItemView(title: "公钥管理",icon:"lock.circle")
    }
    Divider().background(Color(hex: 0x111111))
        .padding(.leading,20)
    NavigationLink(destination:  IssuesView(repoPath: "gitee_ios/feedback", title: "反馈建议")) {
        SettingItemView(title: "反馈与建议",icon:"envelope.circle")
    }
    Divider().background(Color(hex: 0x111111))
        .padding(.leading,20)
    NavigationLink(destination:  AboutView()) {
        SettingItemView(title: "关于项目和源码",icon:"paperplane.circle")
    }
}
.background(Color(red: 1, green: 1, blue: 1, opacity: 0.1))
.cornerRadius(10)
.padding(.horizontal,5)
.padding(.top,10)
```

最下方是一个退出登录按钮：
```Swift
VStack{
    SettingItemView(title: "退出登录",icon:"eject.circle")
        .foregroundColor(.red)
        .background(Color(hex: 0x1c1c1e))
        .onTapGesture {
            UIAlertController.danger(message: "是否确认退出当前登录的账号？", title: "退出提醒", confirmText: "退出", cancelText: "暂不") { (action) in
                localConfig.setValue("", forKey: giteeConfig.access_token)
                self.reloadMyInfo()
            }
        }
}
.background(Color(red: 1, green: 1, blue: 1, opacity: 0.1))
.cornerRadius(10)
.padding(.horizontal,5)
.padding(.top,10)
```

这里有个问题，就是如果用户么有登录，那肯定先要弹登录按钮呀，怎么处理呢？
答案是用sheet扩展函数：
```Swift
.sheet(isPresented: $isLoginShow,onDismiss: {
            self.reloadMyInfo();
}){
    LoginView()
        .modifier(DisableModalDismiss(disabled: true))
}
```

可见的时候需求去调用接口拿个人数据，拿不到再跳登录页：
```Swift
.navigationBarHidden(true)
.edgesIgnoringSafeArea(.top)
.padding(.horizontal,10)
.padding(.top,50)
.preferredColorScheme(.dark)
.onAppear(){
    self.reloadMyInfo();
}
```

具体请求如下：
```Swift
func reloadMyInfo() {
    UserModel().getMyInfo { (userInfo) in
        self.showMyInfo(userInfo: userInfo)
    } error: {
        self.userName = "请先登录"
        self.userBio = "你需要登录后才能访问"
        self.userFollowers = "0"
        self.userFollowing = "0"
        self.userStars = "0"
        self.userWatches = "0"
        self.userHead = nil
        self.isLoginShow.toggle()
    }
}

func showMyInfo(userInfo: JSON){
    self.userAccount = userInfo["login"].string!
    self.userName = userInfo["name"].string!
    self.userBio = userInfo["bio"].string ?? ""
    self.userFollowers = String(userInfo["followers"].int!)
    self.userFollowing = String(userInfo["following"].int!)
    self.userStars = String(userInfo["stared"].int!)
    self.userWatches = String(userInfo["watched"].int!)
    self.downloadWebImage(url: userInfo["avatar_url"].string!)
}
```

然后去加载头像：
```Swift
func downloadWebImage(url:String) {
    guard let url = URL(string: url) else {
        return
    }
    URLSession.shared.dataTask(with: url) { (data, response, error) in
        if let data = data, let image = UIImage(data: data) {
            self.userHead = image
        }
    }.resume()
}
```

对应的菜单item为：
```Swift
struct SettingItemView : View {
    @State var title: String
    @State var icon: String
    
    var body: some View {
        VStack{
            HStack(alignment: .center){
                Image(systemName:icon)
                    .scaleEffect(1.5, anchor: .center)
                Text(title).font(.system(size:16)).padding(.leading,10)
                Spacer()
                HStack{
                    Image(systemName: "chevron.forward")
                        .scaleEffect(0.8)
                }
                .foregroundColor(.gray)
            }
            .padding(.bottom,15)
            .padding(.top,15)
            .padding(.leading,20)
            .padding(.trailing,15)
        }
    }
}
```

### 2.3 Gitee码

对应页面为：
<img src=gitee_03_02.png width=50%>

代码为：
```Swift
struct SettingQrcode: View {
    @State var qrcodeImage:UIImage? = nil
    let placeholderImage = UIImage(named: "nohead")!
    @State var userAccount:String
    var body: some View {
        VStack{
            VStack{
                Image(uiImage: self.qrcodeImage ?? placeholderImage)
                    .resizable()
                    .scaledToFit()
                    .frame(
                        width:200,height:200,
                        alignment: .center
                    )
            }
            .padding(5)
            .background(Color(.white))
            .cornerRadius(10)
            Text("扫描二维码关注我的开源项目").padding(.top,20)
        }
        .navigationBarTitle(Text("你的Gitee码"), displayMode: .inline)
        .preferredColorScheme(.dark)
        .onAppear(){
            print("Creating qrcode for " + userAccount)
            qrcodeImage = Qrcode.setQRCodeToImageView(url: "https://gitee.com/" + userAccount)
        }
    }
}
```

### 2.4 关注列表
效果如下：
<img src=gitee_03_03 width=50%>

代码如下：
```Swift
struct UserFollowersView: View {
    @State var userList: [UserItemModel] = []
    @State var repoPath :String = ""
    @State var waitPlease = false
    @State var isRefreshing = false
    @State var isLoading: Bool = false
    @State var isModal: Bool = false
    @State var message: String = "数据加载中"
    @State var isLoginShow = false
    @State var page = 1
```
上方是变量声明。

```Swift
var body: some View {
    LoadingView(isLoading:self.$isLoading,message: self.$message,isModal:self.$isModal) {
        ZStack{
            if userList.count == 0 && !isLoading {
                VStack{
                    Image(systemName: "doc.text.magnifyingglass")
                        .scaleEffect(3, anchor: .center)
                    Text("Ta还没有粉丝").padding(.top,30)
                }
            }
            RefreshView(refreshing: $isRefreshing, action: {
                self.page = 1
                self.getUserList(page: self.page)
            }) {
                LazyVStack{
                    ForEach(self.userList){ item in
                        NavigationLink(destination: UserProfileView(userInfo: item)) {
                            UserItemView(userItem:  item)
                                .onAppear(){
                                    if !waitPlease && item.id == userList[userList.count - 1].id {
                                        self.page = self.page + 1
                                        self.getUserList(page: self.page)
                                    }
                                }
                        }
                    }
                }
            }
        }
    }
    .padding(.top,5)
    .sheet(isPresented: $isLoginShow,onDismiss: {
        UserModel().getMyInfo { (userInfo) in
            self.isLoading = false
            self.page = 1
            self.getUserList(page: self.page)
        } error: {
            self.isLoginShow.toggle()
        }
    }){
        LoginView()
            .modifier(DisableModalDismiss(disabled: true))
    }
    .navigationBarTitle(Text("粉丝列表"), displayMode: .inline)
    .onAppear(){
        self.page = 1
        self.getUserList( page: self.page)
    }
}
```

这里对应的item如下：
```Swift

struct UserItemView:View{
    @State var userItem: UserItemModel
    @State var placeholderImage = UIImage(named: "nohead")!
    var body: some View{
        VStack(alignment: .leading){
            HStack(){
                Image(uiImage: placeholderImage)
                    .resizable()
                    .scaledToFit()
                    .frame(
                        width:50,height:50,
                        alignment: .center
                    )
                    .cornerRadius(5)
                    .onAppear(){
                        guard let url = URL(string: userItem.userHead) else {
                            return
                        }
                        URLSession.shared.dataTask(with: url) { (data, response, error) in
                            if let data = data, let image = UIImage(data: data) {
                                placeholderImage = image
                            }
                        }.resume()
                    }
                VStack(alignment: .leading){
                    Text(userItem.userName).font(.system(size: 16))
                    Text("@" + userItem.userAccount).font(.system(size: 14))
                        .foregroundColor(Color(hex: 0xaaaaaa))
                        .lineLimit(1)
                        .padding(.top,1)
                }
                Spacer()
            }
            .padding(10)
        }
        .background(Color(red: 1, green: 1, blue: 1, opacity: 0.1))
        .cornerRadius(10)
        .padding(.horizontal,5)
        .padding(.bottom,-3)
    }
}
```

其它列表类似，不做过多讲述。

### 2.5 公钥管理页面
效果如下：
<img src=gitee_03_04.png width=50%>

因为前面定义了跳转：
```
  VStack{
    NavigationLink(destination:  PubKeysView()) {
        SettingItemView(title: "公钥管理",icon:"lock.circle")
    }
```
这里对应View应该就是PubKeysView。

声明变量如下：
```Swift
struct PubKeysView: View {
    @State var publicKeyList:[PublicKeyModel] = []
    @State var waitPlease = false
    @State var isRefreshing = false
    @State var isLoading: Bool = false
    @State var isModal: Bool = false
    @State var message: String = "数据加载中"
    @State var isLoginShow = false
    @State var isFormShow = false
```


对应的body区：
```Swift
var body: some View {
    LoadingView(isLoading:self.$isLoading,message: self.$message,isModal:self.$isModal) {
        ZStack{
            if publicKeyList.count == 0 && !isLoading {
                VStack{
                    Image(systemName: "doc.text.magnifyingglass")
                        .scaleEffect(3, anchor: .center)
                    Text("暂无查询到的公钥").padding(.top,30)
                }
            }
            RefreshView(refreshing: $isRefreshing, action: {
                self.getPublicKeyList()
            }) {
                LazyVStack{
                    ForEach(self.publicKeyList){ item in
                        PubKeysItemView(publicKeyItem: item)
                    }
                }
            }
        }
    }
    .sheet(isPresented: $isLoginShow,onDismiss: {
        UserModel().getMyInfo { (userInfo) in
            self.isLoading = true
            self.getPublicKeyList()
        } error: {
            self.isLoginShow.toggle()
        }
    }){
        LoginView()
            .modifier(DisableModalDismiss(disabled: true))
    }
    .padding(.top,5)
    .navigationBarTitle(Text("你的公钥"), displayMode: .inline)
    .navigationBarItems(
        trailing:
            HStack {
                Button(action: {
                    self.isFormShow.toggle()
                }) {
                    Image(systemName: "plus.circle")
                        .scaleEffect(1, anchor: .center)
                }
            }
            .sheet(isPresented: $isFormShow,onDismiss: {
                self.isLoading = true
                self.getPublicKeyList()
            }){
                PublicKeyInsertView()
                    .modifier(DisableModalDismiss(disabled: false))
            }
    )
    .onAppear(){
        self.getPublicKeyList()
    }
}
```

这里对应的公钥item:
```Swift
struct PubKeysItemView:View{
    @State var publicKeyItem:PublicKeyModel
    var body: some View{
        VStack{
            VStack{
                HStack(alignment: .top) {
                    VStack(alignment: .leading){
                        Text(publicKeyItem.title)
                            .font(.system(size: 16))
                            .lineLimit(1)
                    }
                    Spacer()
                    Text(Helper.getDateFromString(str: publicKeyItem.createTime))
                        .padding(.vertical,1)
                        .padding(.horizontal,3)
                        .font(.system(size: 12))
                        .foregroundColor(.gray)
                        .cornerRadius(3)
                    
                }
                .padding(5)
                Text(publicKeyItem.key)
                    .font(.system(size: 14))
                    .foregroundColor(.gray)
                    .lineLimit(5)
                    .padding(.top,10)
                    .fixedSize(horizontal: false, vertical: true)
            }
            .padding(10)
        }
        .background(Color(red: 1, green: 1, blue: 1, opacity: 0.1))
        .cornerRadius(10)
        .padding(.horizontal,5)
        .padding(.bottom,-5)
        .contextMenu(ContextMenu {
            Button(action: {
            }) {
                HStack{
                    Image(systemName: "doc.on.doc").scaleEffect(1, anchor: .center)
                    Spacer()
                    Text("复制公钥")
                }
            }
            Divider()
            Button(action: {
            }) {
                HStack{
                    Image(systemName: "trash").scaleEffect(1, anchor: .center)
                    Spacer()
                    Text("删除公钥")
                }
            }
        })
    }
}
```

然后是请求公钥列表：
```Swift
func getPublicKeyList(){
    if self.waitPlease { return }
    self.waitPlease = true
    if publicKeyList.count == 0 {
        self.isLoading = true
    }
    HttpRequest(url: "user/keys", withAccessToken: true)
        .doGet { (value) in
            let json = JSON(value)
            if json["message"].string != nil {
                print("error")
                self.isLoginShow.toggle()
            }else{
                var tempList:[PublicKeyModel] = []
                for (_,subJson):(String, JSON) in json {
                    tempList.append(PublicKeyModel(id: subJson["id"].intValue,title: subJson["title"].stringValue,key:subJson["key"].stringValue,createTime: subJson["created_at"].stringValue))
                }
                publicKeyList = tempList
            }
            self.isRefreshing = false
            self.isLoading = false
            self.waitPlease = false
        } errorCallback: {
            self.isRefreshing = false
            self.isLoading = false
            self.waitPlease = false
        }
}
```

## 3 仓库详情

首先看下效果图：
<img src=gitee_03_05.png width=50%>

### 3.1 变量声明
```Swift
struct RepoDetailView: View {
    @State var size:CGSize = .zero
    @State var isLoading: Bool = false
    @State var isModal: Bool = false
    @State var message: String = "数据加载中"
    @State var repoItem: RepoModel?
    @State var repoReadme = ""
    @State var repoIsStarred:Bool = false
    @State var repoIsWatched:Bool = false
    @State var isLoginShow = false
    @State var isIssueInsertShow = false
    @State var repoFullPath: String
```

### 3.2 视图构造
body定义如下，首先是一个LoadingView+VStack+ScrollView:
```Swift
var body: some View {
    LoadingView(isLoading:self.$isLoading,message: self.$message,isModal:self.$isModal) {
        VStack{
            if self.repoItem != nil {
                
                ScrollView(.vertical, showsIndicators: false) {
```

然后第一部分是第一个VStack:
```Swift
VStack{
    RepoDetailInfoView(repoItem: self.repoItem!)
    HStack{
        NavigationLink(destination: UserStarsView(repoPath: self.repoFullPath)) {
            VStack{
                Text(self.repoItem!.repoStars).font(.system(size:20))
                    .fontWeight(.bold)
                Text("Stars").foregroundColor(Color(hex: 0x999999)).font(.system(size:12))
            }
        }
        Spacer()
        NavigationLink(destination: UserWatchesView(repoPath:self.repoFullPath)) {
            VStack{
                Text(self.repoItem!.repoWatches).font(.system(size:20))
                    .fontWeight(.bold)
                Text("Watches").foregroundColor(Color(hex: 0x999999)).font(.system(size:12))
            }
        }
        Spacer()
        NavigationLink(destination: UserForksView(repoPath:self.repoFullPath)) {
            VStack{
                Text(self.repoItem!.repoForks).font(.system(size:20))
                    .fontWeight(.bold)
                Text("Forks").foregroundColor(Color(hex: 0x999999)).font(.system(size:12))
            }
        }
        Spacer()
        NavigationLink(destination: IssuesView(repoPath:self.repoFullPath)) {
            VStack{
                Text(self.repoItem!.repoIssues).font(.system(size:20))
                    .fontWeight(.bold)
                Text("Issues").foregroundColor(Color(hex: 0x999999)).font(.system(size:12))
            }
        }
    }
    .padding(.top,10)
    .padding(.horizontal,20)
    .padding(.bottom,20)
}
.background(Color(red: 1, green: 1, blue: 1, opacity: 0.1))
.cornerRadius(10)
.padding(5)
```

上面对应顶部区域，和标星，Watches,Forks,Issures这4个区块。

上方的详细信息呢？
```Swift
struct RepoDetailInfoView:View {
    @State var repoItem:RepoModel
    var body: some View {
        VStack(alignment: .leading){
            HStack(alignment: .top) {
                if !self.repoItem.repoIsOpenSource {
                    Image(systemName: "lock.square.fill")
                        .foregroundColor(Color(hex: 0xffc55a))
                        .padding(.trailing,-5)
                        .scaleEffect(1, anchor: .center)
                }
                VStack(alignment: .leading){
                    Text(self.repoItem.repoName)
                        .font(.system(size: 18))
                        .lineLimit(1)
                }
                Spacer()
            }
            .padding(5)
            Text(self.repoItem.repoDesc == "" ? "很尴尬,该项目暂无介绍..." : self.repoItem.repoDesc)
                .font(.system(size: 14))
                .foregroundColor(.gray)
                .multilineTextAlignment(.leading)
                .padding(.top,10)
                .padding(.leading,5)
            HStack{
                if self.repoItem.repoLanguage != "" {
                    VStack{
                        Text(self.repoItem.repoLanguage)
                            .padding(.vertical,1)
                            .padding(.horizontal,3)
                    }
                    .font(.system(size: 12))
                    .background(Color(red: 1, green: 1, blue: 1, opacity: 0.1))
                    .foregroundColor(.gray)
                    .cornerRadius(3)
                }
                if self.repoItem.repoLicense != "" {
                    VStack{
                        Text(self.repoItem.repoLicense)
                            .padding(.vertical,1)
                            .padding(.horizontal,3)
                    }
                    .font(.system(size: 12))
                    .background(Color(red: 1, green: 1, blue: 1, opacity: 0.1))
                    .foregroundColor(.gray)
                    .cornerRadius(3)
                }
            }
            .padding(.top,10)
        }
        .padding(10)
    }
}
```
这里描述了仓库名称和语言。

然后是下方4个item:
```Swift
VStack{
    VStack{
        NavigationLink(destination: SettingQrcode(userAccount: "hamm")) {
            HStack{
                Text("代码").font(.system(size:16))
                Spacer()
                HStack{
                    //                                            Text("master").font(.system(size:14))
                    Image(systemName: "chevron.forward")
                }
                .foregroundColor(.gray)
            }
            .padding(.vertical,12)
            .padding(.leading,20)
            .padding(.trailing,10)
        }
        Divider().background(Color(hex: 0x111111))
        NavigationLink(destination: CommitView(repoFullPath: self.repoItem!.repoNamespace.path + "/" + self.repoItem!.repoPath,repoDefaultBranch: self.repoItem!.repoDefaultBranch)) {
            HStack{
                Text("提交").font(.system(size:16))
                Spacer()
                HStack{
                    Text(self.repoItem!.repoPushDate).font(.system(size:14))
                    Image(systemName: "chevron.forward")
                }
                .foregroundColor(.gray)
            }
            .padding(.vertical,12)
            .padding(.leading,20)
            .padding(.trailing,10)
        }
        NavigationLink(destination: PullRequestView(repoFullPath: self.repoItem!.repoNamespace.path + "/" + self.repoItem!.repoPath)) {
            HStack{
                Text("Pull Requests").font(.system(size:16))
                Spacer()
                HStack{
                    Image(systemName: "chevron.forward")
                }
                .foregroundColor(.gray)
            }
            .padding(.vertical,12)
            .padding(.leading,20)
            .padding(.trailing,10)
        }
        Divider().background(Color(hex: 0x111111))
        NavigationLink(destination: RepoMembersView(repoPath:self.repoFullPath)) {
            HStack{
                Text("仓库成员").font(.system(size:16))
                Spacer()
                HStack{
                    Image(systemName: "chevron.forward")
                }
                .foregroundColor(.gray)
            }
            .padding(.vertical,12)
            .padding(.leading,20)
            .padding(.trailing,10)
        }
    }
    .padding(.vertical,10)
}
.background(Color(red: 1, green: 1, blue: 1, opacity: 0.1))
.cornerRadius(10)
.padding(.horizontal,5)
.padding(.vertical,-10)
```

然后下方展示Readme:
```Swift
 if repoReadme != "" {
    VStack(alignment: .leading){
        VStack{
            MDText(markdown: repoReadme).padding()
        }
        .padding(.top,5)
        .padding(.horizontal,5)
        .padding(.bottom,10)
    }
    .background(Color(red: 1, green: 1, blue: 1, opacity: 0.1))
    .cornerRadius(10)
    .padding(5)
}
```
这里通过三方库MDText来展示。

然后最下方是4个按钮，可以started,watched,forck,Issue:
```Swift
 Spacer()
VStack{
    HStack(alignment: .center){
        Button {
            let url = "user/starred/" + self.repoFullPath
            self.isLoading = true
            self.message = "Loading"
            if repoIsStarred {
                HttpRequest(url: url,withAccessToken: true).doDelete(postData: ["a":"b"]) { (result) in
                    self.isLoading = false
                    self.getRepoInfo()
                    getIsStarred()
                } errorCallback: {
                    self.isLoading = false
                    getIsStarred()
                }
            }else{
                HttpRequest(url: url,withAccessToken: true).doPut(postData: ["a":"b"]) { (result) in
                    self.isLoading = false
                    self.getRepoInfo()
                    getIsStarred()
                } errorCallback: {
                    self.isLoading = false
                    getIsStarred()
                }
            }
        } label: {
            VStack(alignment: .center){
                Image(systemName: "star")
                    .scaleEffect(1.2, anchor: .center)
                    .padding(.bottom,5).foregroundColor(repoIsStarred ? .yellow : .gray)
                Text(repoIsStarred ? "Starred" : "Star").font(.system(size:12)).foregroundColor(repoIsStarred ? .yellow : .gray)
            }
            .frame(width: 60)
        }
        Spacer()
        Button {
            let url = "user/subscriptions/" + self.repoFullPath
            self.isLoading = true
            self.message = "Loading"
            if repoIsWatched {
                HttpRequest(url: url,withAccessToken: true).doDelete(postData: ["a":"b"]) { (result) in
                    self.isLoading = false
                    self.getRepoInfo()
                    getIsWatched()
                } errorCallback: {
                    self.isLoading = false
                    getIsWatched()
                }
            }else{
                HttpRequest(url: url,withAccessToken: true).doPut(postData: ["watch_type":"watching"]) { (result) in
                    self.isLoading = false
                    self.getRepoInfo()
                    getIsWatched()
                } errorCallback: {
                    self.isLoading = false
                    getIsWatched()
                }
            }
        } label: {
            VStack(alignment: .center){
                Image(systemName: "eye")
                    .scaleEffect(1.2, anchor: .center)
                    .padding(.bottom,5).foregroundColor(repoIsWatched ? .yellow : .gray)
                Text(repoIsWatched ? "Watched" : "Watch").font(.system(size:12)).foregroundColor(repoIsWatched ? .yellow : .gray)
            }
            .frame(width: 60)
        }
        Spacer()
        Button {
            UIAlertController.confirm(message: "是否确认Fork这个仓库到你个人仓库？", title:"Fork仓库提醒", confirmText: "确认Fork", cancelText: "取消") { (action) in
                let url = "repos/" + self.repoFullPath + "/forks"
                self.isLoading = true
                self.message = "Loading"
                HttpRequest(url: url,withAccessToken: true).doPost(postData: ["a":"b"]) { (result) in
                    let json = JSON(result)
                    if json["message"].string != nil{
                        DispatchQueue.main.async{
                            UIAlertController.alert(message: json["message"].string!, title: "Fork失败", confirmText: "好吧")
                        }
                    }else{
                        DispatchQueue.main.async{
                            UIAlertController.alert(message: "你已经成功将该仓库Fork到自己仓库", title: "Fork成功", confirmText: "好吧")
                        }
                    }
                    self.isLoading = false
                } errorCallback: {
                    self.isLoading = false
                }
            }
        } label: {
            VStack(alignment: .center){
                Image(systemName: "arrowshape.turn.up.backward.2.fill")
                    .scaleEffect(1.2, anchor: .center)
                    .padding(.bottom,5).foregroundColor(.gray)
                Text("Fork").font(.system(size:12)).foregroundColor(.gray)
            }
            .frame(width: 60)
        }
        Spacer()
        Button {
            self.isIssueInsertShow.toggle()
        } label: {
            VStack(alignment: .center){
                Image(systemName: "exclamationmark.circle.fill")
                    .scaleEffect(1.2, anchor: .center)
                    .padding(.bottom,5).foregroundColor(.gray)
                Text("Issue").font(.system(size:12)).foregroundColor(.gray)
            }
            .frame(width: 60)
            .sheet(isPresented: $isIssueInsertShow,onDismiss: {
                self.getRepoInfo()
            }){
                IssueInsertView(repoNamespacePath: self.repoItem!.repoNamespace.path, repoPath: self.repoItem!.repoPath)
            }
        }
    }
    .padding(.vertical,20)
    .padding(.horizontal,30)
    .background(Color(red: 1, green: 1, blue: 1, opacity: 0.1))
    .cornerRadius(10)
}
.padding(.horizontal,5)
}}
```

然后可见的时候，需要拿数据：
```Swift
.onAppear(){
    self.getRepoInfo()
    self.getRepoReadme()
    self.getIsStarred()
    self.getIsWatched()
}
```

### 3.3 详情网络请求
然后就是走了这几个接口：
```Swift
func getRepoInfo(){
    let url = "repos/" +  self.repoFullPath
    print(url)
    HttpRequest(url: url,withAccessToken: true).doGet { (result) in
        let json = JSON(result)
        print(json)
        if json["message"].string == nil{
            let repoNamespace = RepoNamespace(id: Int(json["namespace"]["id"].stringValue)!, name: String(json["namespace"]["name"].stringValue), path: String(json["namespace"]["path"].stringValue))
            let repoItem = RepoModel(id: Int(json["id"].int!) , repoName:  String(json["name"].stringValue), repoPath:   String(json["path"].stringValue), repoNamespace: repoNamespace, repoDesc: String(json["description"].stringValue), repoForks: String(json["forks_count"].stringValue), repoStars: String(json["stargazers_count"].stringValue), repoWatches: String(json["watchers_count"].stringValue), repoLicense: String(json["license"].stringValue), repoLanguage: String(json["language"].stringValue), repoPushDate: Helper.getDateFromString(str: String(json["pushed_at"].stringValue)), repoIsFork: json["fork"].bool ?? false, repoIsOpenSource: json["public"].bool ?? false,repoIssues: String(json["open_issues_count"].stringValue),repoDefaultBranch:  String(json["default_branch"].stringValue))
            self.repoItem = repoItem
        }
    } errorCallback: {
    }
}
func getIsWatched(){
    let url = "user/subscriptions/" +  self.repoFullPath
    HttpRequest(url: url,withAccessToken: true).doGet { (result) in
        let json = JSON(result)
        if json["message"].stringValue != ""{
            self.repoIsWatched = false
        }else{
            self.repoIsWatched = true
        }
    } errorCallback: {
        self.repoIsWatched = false
    }
}
func getIsStarred(){
    let url = "user/starred/" + self.repoFullPath
    HttpRequest(url: url,withAccessToken: true).doGet { (result) in
        let json = JSON(result)
        if json["message"].stringValue != ""{
            self.repoIsStarred = false
        }else{
            self.repoIsStarred = true
        }
    } errorCallback: {
        self.repoIsStarred = false
    }
}
func getRepoReadme(){
    self.isLoading = true
    self.message = "Load ReadMe"
    let url = "repos/" +  self.repoFullPath + "/readme"
    HttpRequest(url: url,withAccessToken: true).doGet { (result) in
        let json = JSON(result)
        if json["content"].stringValue != "" {
            self.repoReadme = Helper.base64Decoding(encodedString: String(json["content"].stringValue))
            self.isLoading = false
        }else{
            self.isLoading = false
        }
    } errorCallback: {
        self.repoReadme = ""
        self.isLoading = false
    }
}
```

## 4 网络层

前面讲的都是利用一个HttpRequest来进行调用接口，我们需要了解具体网络层做了什么。

### 4.1 以获取仓库readme为例

```Swift
func getRepoReadme(){
    self.isLoading = true
    self.message = "Load ReadMe"
    let url = "repos/" +  self.repoFullPath + "/readme"
    HttpRequest(url: url,withAccessToken: true).doGet { (result) in
        let json = JSON(result)
        if json["content"].stringValue != "" {
            self.repoReadme = Helper.base64Decoding(encodedString: String(json["content"].stringValue))
            self.isLoading = false
        }else{
            self.isLoading = false
        }
    } errorCallback: {
        self.repoReadme = ""
        self.isLoading = false
    }
}
```

这里通过HttpRequest，调用了url，拿到json数据后，通过SwiftJSON三方库完成数据转换。

### 4.2 网络封装工具

下面封装了网络层请求工具，可以执行各种请求。
```Swift
import Foundation
class HttpRequest {
    var url:String = ""
    var baseUrl:String = "https://gitee.com/api/v5/"
    //    var url:String = "https://api.bbbug.com/api/"
    var access_token:String = "";
    
    init(url: String) {
        self.url = self.baseUrl
        if url.contains("http://") || url.contains("https://"){
            self.url = url
        }else {
            self.url = self.baseUrl + url
        }
    }
    init(url: String, withAccessToken: Bool) {
        self.url = self.baseUrl
        if url.contains("http://") || url.contains("https://"){
            self.url = url
        }else {
            self.url = self.baseUrl + url
        }
        if withAccessToken {
            let access_token = localConfig.string(forKey: giteeConfig.access_token)
            if access_token != nil{
                self.access_token = access_token!
            }
        }
        print(self.url)
    }
    public func doPost(postData:[String:String],successCallback:@escaping((Data)->Void),errorCallback:@escaping(()->Void)){
        if self.access_token != ""{
            if self.url.contains("?") {
                self.url = self.url + "&access_token=" + self.access_token
            }else{
                self.url = self.url + "?access_token=" + self.access_token
            }
        }
        print(self.url)
        var urlSession:URLSession = URLSession(configuration: .default)
        var urlRequest:URLRequest = URLRequest(url: URL(string: self.url)!)
        urlSession = URLSession(configuration: .default)
        urlRequest.setValue("application/x-www-form-urlencoded", forHTTPHeaderField: "Content-Type")
        urlRequest.httpMethod = "POST"
        let postString = postData.compactMap({ (key, value) -> String in
            return "\(key)=\(value)"
        }).joined(separator: "&")
        urlRequest.httpBody = postString.data(using: .utf8)
        let task = urlSession.dataTask(with: urlRequest) {(data, response, error) in
            if data != nil{
                successCallback(data!)
            }else{
                errorCallback()
            }
        }
        task.resume()
    }
    public func doPostJson(postJson:String,successCallback:@escaping((Data)->Void),errorCallback:@escaping(()->Void)){
        print(self.url)
        var urlSession:URLSession = URLSession(configuration: .default)
        var urlRequest:URLRequest = URLRequest(url: URL(string: self.url)!)
        urlSession = URLSession(configuration: .default)
        urlRequest.setValue("application/json", forHTTPHeaderField: "Content-Type")
        urlRequest.httpMethod = "POST"
        urlRequest.httpBody = postJson.data(using: .utf8)
        let task = urlSession.dataTask(with: urlRequest) {(data, response, error) in
            if data != nil{
                successCallback(data!)
            }else{
                errorCallback()
            }
        }
        task.resume()
    }
    public func doGet(successCallback:@escaping((Data)->Void),errorCallback:@escaping(()->Void)){
        if self.access_token != ""{
            if self.url.contains("?") {
                self.url = self.url + "&access_token=" + self.access_token
            }else{
                self.url = self.url + "?access_token=" + self.access_token
            }
        }
        print(self.url)
        var urlSession:URLSession = URLSession(configuration: .default)
        var urlRequest:URLRequest = URLRequest(url: URL(string: self.url)!)
        urlSession = URLSession(configuration: .default)
        urlRequest.httpMethod = "GET"
        let task = urlSession.dataTask(with: urlRequest) {(data, response, error) in
            if data != nil{
                successCallback(data!)
            }else{
                errorCallback()
            }
        }
        task.resume()
    }
    public func doPut(postData:[String:String],successCallback:@escaping((Data)->Void),errorCallback:@escaping(()->Void)){
        if self.access_token != ""{
            if self.url.contains("?") {
                self.url = self.url + "&access_token=" + self.access_token
            }else{
                self.url = self.url + "?access_token=" + self.access_token
            }
        }
        print(self.url)
        var urlSession:URLSession = URLSession(configuration: .default)
        var urlRequest:URLRequest = URLRequest(url: URL(string: self.url)!)
        urlSession = URLSession(configuration: .default)
        urlRequest.setValue("application/x-www-form-urlencoded", forHTTPHeaderField: "Content-Type")
        urlRequest.httpMethod = "PUT"
        let postString = postData.compactMap({ (key, value) -> String in
            return "\(key)=\(value)"
        }).joined(separator: "&")
        urlRequest.httpBody = postString.data(using: .utf8)
        let task = urlSession.dataTask(with: urlRequest) {(data, response, error) in
            if data != nil{
                successCallback(data!)
            }else{
                errorCallback()
            }
        }
        task.resume()
    }
    public func doDelete(postData:[String:String],successCallback:@escaping((Data)->Void),errorCallback:@escaping(()->Void)){
        if self.access_token != ""{
            if self.url.contains("?") {
                self.url = self.url + "&access_token=" + self.access_token
            }else{
                self.url = self.url + "?access_token=" + self.access_token
            }
        }
        print(self.url)
        var urlSession:URLSession = URLSession(configuration: .default)
        var urlRequest:URLRequest = URLRequest(url: URL(string: self.url)!)
        urlSession = URLSession(configuration: .default)
        urlRequest.setValue("application/x-www-form-urlencoded", forHTTPHeaderField: "Content-Type")
        urlRequest.httpMethod = "DELETE"
        let postString = postData.compactMap({ (key, value) -> String in
            return "\(key)=\(value)"
        }).joined(separator: "&")
        urlRequest.httpBody = postString.data(using: .utf8)
        let task = urlSession.dataTask(with: urlRequest) {(data, response, error) in
            if data != nil{
                successCallback(data!)
            }else{
                errorCallback()
            }
        }
        task.resume()
    }
    public func doPatch(postData:[String:String],successCallback:@escaping((Data)->Void),errorCallback:@escaping(()->Void)){
        if self.access_token != ""{
            if self.url.contains("?") {
                self.url = self.url + "&access_token=" + self.access_token
            }else{
                self.url = self.url + "?access_token=" + self.access_token
            }
        }
        print(self.url)
        var urlSession:URLSession = URLSession(configuration: .default)
        var urlRequest:URLRequest = URLRequest(url: URL(string: self.url)!)
        urlSession = URLSession(configuration: .default)
        urlRequest.setValue("application/x-www-form-urlencoded", forHTTPHeaderField: "Content-Type")
        urlRequest.httpMethod = "PATCH"
        let postString = postData.compactMap({ (key, value) -> String in
            return "\(key)=\(value)"
        }).joined(separator: "&")
        urlRequest.httpBody = postString.data(using: .utf8)
        let task = urlSession.dataTask(with: urlRequest) {(data, response, error) in
            if data != nil{
                successCallback(data!)
            }else{
                errorCallback()
            }
        }
        task.resume()
    }
}
```

在调用请求的地方这样使用：
```Swift
HttpRequest(url: url,withAccessToken: true).doPost(postData: ["a":"b"]) { (result) in
    let json = JSON(result)
    if json["message"].string != nil{
        DispatchQueue.main.async{
            UIAlertController.alert(message: json["message"].string!, title: "Fork失败", confirmText: "好吧")
        }
    }else{
        DispatchQueue.main.async{
            UIAlertController.alert(message: "你已经成功将该仓库Fork到自己仓库", title: "Fork成功", confirmText: "好吧")
        }
    }
    self.isLoading = false
} errorCallback: {
    self.isLoading = false
}
```

它这个底层是用的URLSession+URLRequest实现的。
通过urlSession执行一个异步任务发起请求。

## 5 总结

* 这个项目总体讲述了SwiftUI的基本使用，个人首先是学会了如何构建布局，感觉有点像Flutter，都是声明式布局，所以上手还是挺轻松的。主要是学会VStack,HStack,Button,Text,Image,Spacer,ScrollView的用法就行。

* 另外需要学会自定义View，其实还是很简单，通过继承View where Content: View来实现，主要是自己实现一个body就行了。

* 响应事件有多种方法可以实现，直接通过定义Button实现闭包，或者通过NavigationLink设置跳转指定页面。或者直接onTapGesture闭包。

* 另外学会View的生命周期，可以在onAppear里面做初始化工作，可以去请求网络，onDisappear做一些销毁工作。

* 还有些扩展函数，比如弹出上下文菜单，contextMenu,sheet弹出模态框等等。

* 另外是网络层相关的请求，学会封装网络层，定义各种参数请求，方便更换地调用接口，接收参数。