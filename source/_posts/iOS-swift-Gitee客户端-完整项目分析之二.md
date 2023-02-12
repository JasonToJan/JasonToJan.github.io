---
title: iOS swift Gitee客户端 完整项目分析之二
date: 2023-02-12 09:41:35
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

## 2 首页其他列表

### 2.1 Issues
点击Issues这个item后，会跳转到所有Issures的列表页：
<img src=gitee_02_1.png>


```Swift
struct IssuesView: View {
    @State var issuesList: [IssueModel] = []
    @State var waitPlease = false
    @State var isRefreshing = false
    @State var isLoading: Bool = false
    @State var isModal: Bool = false
    @State var message: String = "数据加载中"
    @State var isFilterShow = false
    @State var page = 1
    @State var repoPath: String = ""
    @State var title: String? = "Issues"
    @State var isInserIssueShow:Bool = false
```
首先继承View，声明动态数据，因为swift是静态语言，这里通过注解@State实现动态监听。
> @State 是 SwiftUI 中的一个注解，它用于声明一个状态变量。SwiftUI 框架在渲染界面时会自动监测状态变量的变化，并且自动更新对应的界面元素。这样可以方便地管理界面状态和数据。

然后是body声明：
```Swift
var body: some View {
    LoadingView(isLoading:self.$isLoading,message: self.$message,isModal:self.$isModal) {
        VStack{
            ZStack{
                if issuesList.count == 0 && !isLoading {
                    VStack{
                        Image(systemName: "doc.text.magnifyingglass")
                            .scaleEffect(3, anchor: .center)
                        Text("暂无查询到的Issues").padding(.top,30)
                    }
                }
                RefreshView(refreshing: $isRefreshing, action: {
                    self.page = 1
                    self.getIssueList(page: self.page)
                }) {
                    LazyVStack{
                        ForEach(self.issuesList){ item in
                            NavigationLink(destination: IssueItemView(issueItem: item)) {
                                IssueItemView(issueItem: item)
                                    .onAppear(){
                                        if !waitPlease && item.id == issuesList[issuesList.count - 1].id {
                                            self.page = self.page + 1
                                            self.getIssueList(page: self.page)
                                        }
                                    }
                            }
                        }
                    }
                }
            }
            if self.repoPath != "" {
                Spacer()
                HStack{
                    Spacer()
                    Button {
                        self.isInserIssueShow = true
                    } label: {
                        Image(systemName: "square.and.pencil")
                            .scaleEffect(1.2, anchor: .center)
                    }
                    .padding(12)
                    .background(Color.yellow)
                    .foregroundColor(Color.black)
                    .cornerRadius(100)
                    .sheet(isPresented: $isInserIssueShow) {
                        let arr = self.repoPath.components(separatedBy: "/")
                        IssueInsertView(repoNamespacePath: arr[0], repoPath: arr[1])
                    }
                }
                .padding(.trailing,20)
            }
        }
    }
    .padding(.top,5)
    .navigationBarTitle(Text(title!), displayMode: .inline)
    .navigationBarItems(
        trailing:
            HStack {
                Button(action: {
                    self.isFilterShow.toggle()
                }) {
                    Image(systemName: "rectangle.and.text.magnifyingglass")
                        
                        .scaleEffect(1, anchor: .center)
                }
            }
            .sheet(isPresented: $isFilterShow,onDismiss: {
                self.isLoading = true
                self.page = 1
                self.getIssueList(page: self.page)
            }){
                IssueFilterView()
                    .modifier(DisableModalDismiss(disabled: false))
            }
    )
    .onAppear(){
        localConfig.setValue("all", forKey: giteeConfig.issue_filter)
        localConfig.setValue("open", forKey: giteeConfig.issue_state)
        localConfig.setValue("created", forKey: giteeConfig.issue_sort)
        localConfig.setValue("desc", forKey: giteeConfig.issue_direction)
        self.page = 1
        self.getIssueList(page: self.page)
    }
}
```

上面是定义了整个Issues列表的UI。
内部用了自定义的下拉刷新控件：RefreshView。
然后里面是通过一个ForEach循环，里面装载了IssueItemView，就是单个item布局效果。

这个跟前面分析的IssueItemView一致。

这里Issue页面，标题栏右边有个筛选按钮：
```Swift
.navigationBarItems(
        trailing:
            HStack {
                Button(action: {
                    self.isFilterShow.toggle()
                }) {
                    Image(systemName: "rectangle.and.text.magnifyingglass")
                        
                        .scaleEffect(1, anchor: .center)
                }
            }
            .sheet(isPresented: $isFilterShow,onDismiss: {
                self.isLoading = true
                self.page = 1
                self.getIssueList(page: self.page)
            }){
                IssueFilterView()
                    .modifier(DisableModalDismiss(disabled: false))
            }
    )
```
是通过给View扩展，添加trailing尾部，配置图标，以及点击后的响应view: IssureFilterView。

这里看下筛选View:
<img src=gitee_02_filter.png>

对应的代码如下：
```Swift

struct IssueFilterView: View {
    @Environment(\.presentationMode) var mode
    @State var filter = 0;
    @State var filterList = ["所有来源的Issues","只显示指派给我的","只显示我创建的"];
    @State var filterKey = ["all","assigned","created"];
    @State var state = 1;
    @State var stateList = ["所有状态的Issues","只显示已开启的","只显示进行中的","只显示已完成的","只显示被拒绝的"];
    @State var stateKey = ["all","open","progressing","closed","rejected"];
    @State var sort = 0;
    @State var sortList = ["创建时间","更新时间"];
    @State var sortKey = ["created","updated"];
    @State var direction = 0;
    @State var directionList = ["从最新开始","从最早开始"];
    @State var directionKey = ["desc","asc"];
    var body: some View {
        NavigationView{
            Form{
                Section(header: Text("条件")) {
                    Picker(selection: self.$filter, label: Text("Issues来源")) {
                        ForEach(0 ..< self.filterList.count){
                            Text(self.filterList[$0]).tag($0)
                        }
                    }
                    Picker(selection: self.$state, label: Text("当前状态")) {
                        ForEach(0 ..< self.stateList.count){
                            Text(self.stateList[$0]).tag($0)
                        }
                    }
                }
                Section(header: Text("排序")) {
                    Picker(selection: self.$sort, label: Text("排序依据")) {
                        ForEach(0 ..< self.sortList.count){
                            Text(self.sortList[$0]).tag($0)
                        }
                    }
                    Picker(selection: self.$direction, label: Text("排序方式")) {
                        ForEach(0 ..< self.directionList.count){
                            Text(self.directionList[$0]).tag($0)
                        }
                    }
                }
            }
            .navigationBarTitle(Text("筛选Issues"), displayMode: .large)
            .navigationBarItems(trailing:
                                    Button(action: {
                                        // 存储这部分配置
                                        localConfig.setValue(self.filterKey[self.filter], forKey: giteeConfig.issue_filter)
                                        localConfig.setValue(self.stateKey[self.state], forKey: giteeConfig.issue_state)
                                        localConfig.setValue(self.sortKey[self.sort], forKey: giteeConfig.issue_sort)
                                        localConfig.setValue(self.directionKey[self.direction], forKey: giteeConfig.issue_direction)
                                        self.mode.wrappedValue.dismiss()
                                    }) {
                                        Text("筛选").foregroundColor(.yellow)
                                    })
        }
        .onAppear(){
            
        }
        .preferredColorScheme(.dark)
    }
}
```
这里是跳转到一个系统的Picker视图页：
<img src=gitee_02_02.png>

然后是右侧按钮点击事件：
```Swift
 .navigationBarItems(trailing:
        Button(action: {
            // 存储这部分配置
            localConfig.setValue(self.filterKey[self.filter], forKey: giteeConfig.issue_filter)
            localConfig.setValue(self.stateKey[self.state], forKey: giteeConfig.issue_state)
            localConfig.setValue(self.sortKey[self.sort], forKey: giteeConfig.issue_sort)
            localConfig.setValue(self.directionKey[self.direction], forKey: giteeConfig.issue_direction)
            self.mode.wrappedValue.dismiss()
        }) {
            Text("筛选").foregroundColor(.yellow)
        })
```
点击后，先存储配置，然后怎么通知列表更新的呢？

应该是这里：
```Swift
.navigationBarItems(
        trailing:
            HStack {
                Button(action: {
                    self.isFilterShow.toggle()
                }) {
                    Image(systemName: "rectangle.and.text.magnifyingglass")
                        
                        .scaleEffect(1, anchor: .center)
                }
            }
            .sheet(isPresented: $isFilterShow,onDismiss: {
                self.isLoading = true
                self.page = 1
                self.getIssueList(page: self.page)
            }){
                IssueFilterView()
                    .modifier(DisableModalDismiss(disabled: false))
            }
    )
```
通过配置sheet方法，在里面的view发送dissmiss后，会触发sheet里面的onDismiss回调，这里就重新走接口了。走接口的话应该会重新拿localConfig里面的值。
> sheet 方法是 SwiftUI 中的一个视图方法，用于从当前视图弹出一个新的视图作为模态视图。模态视图是一种浮现在当前界面之上的视图，通常用于呈现额外的内容或提供选项等。模态视图不会阻碍用户对当前界面的操作，但必须关闭模态视图才能继续对当前界面的操作。

然后就是如何获取网络层数据的？
```Swift
.onAppear(){
        print("这里appear了")
        localConfig.setValue("all", forKey: giteeConfig.issue_filter)
        localConfig.setValue("open", forKey: giteeConfig.issue_state)
        localConfig.setValue("created", forKey: giteeConfig.issue_sort)
        localConfig.setValue("desc", forKey: giteeConfig.issue_direction)
        self.page = 1
        self.getIssueList(page: self.page)
    }
```
主要还是这里，在onAppear的时候，去拿接口数据了。

```Swift
func getIssueList(page: Int){
    if self.waitPlease { return }
    self.waitPlease = true
    if issuesList.count == 0 {
        self.isLoading = true
    }
    let state = localConfig.string(forKey: giteeConfig.issue_state)
    let filter = localConfig.string(forKey: giteeConfig.issue_filter)
    let direction = localConfig.string(forKey: giteeConfig.issue_direction)
    let sort = localConfig.string(forKey: giteeConfig.issue_sort)
    var url = "user/issues?page=" + String(page)
    
    if self.repoPath != "" {
        url = "repos/" + self.repoPath + "/issues?page=" + String(page)
    }
    
    url = url + "&state=" + state!
    url = url + "&filter=" + filter!
    url = url + "&direction=" + direction!
    url = url + "&sort=" + sort!
    
    
    HttpRequest(url: url, withAccessToken: true)
        .doGet { (value) in
            let json = JSON(value)
            if json["message"].string != nil {
                print("error")
                DispatchQueue.main.async {
                    UIAlertController.confirm(message: json["message"].stringValue, title: "发生错误", confirmText: "重新登录", cancelText: "返回") { (action) in
                        Helper.relogin()
                    }
                }
            }else{
                var tempList = self.issuesList
                if page == 1{
                    tempList = []
                }
                for (_,subJson):(String, JSON) in json {
                    let repoNamespace = RepoNamespace(id: Int(subJson["repository"]["namespace"]["id"].stringValue)!, name: String(subJson["repository"]["namespace"]["name"].stringValue), path:  String(subJson["repository"]["namespace"]["path"].stringValue))
                    let repoInfo = RepoModel(id: Int(subJson["repository"]["id"].stringValue)!, repoName: String(subJson["repository"]["name"].stringValue),repoPath:  String(subJson["repository"]["path"].stringValue),repoNamespace: repoNamespace, repoDesc:  String(subJson["repository"]["description"].stringValue), repoForks:  String(subJson["repository"]["forks_count"].stringValue), repoStars:  String(subJson["repository"]["stargazers_count"].stringValue), repoWatches:  String(subJson["repository"]["watchers_count"].stringValue), repoLicense:  String(subJson["repository"]["license"].stringValue), repoLanguage:  String(subJson["repository"]["language"].stringValue), repoPushDate:  String(subJson["repository"]["pushed_at"].stringValue), repoIsFork:  Bool(subJson["repository"]["fork"].boolValue), repoIsOpenSource:  Bool(subJson["repository"]["public"].boolValue), repoIssues:  String(subJson["repository"]["open_issues_count"].stringValue), repoDefaultBranch:  String(subJson["repository"]["default_branch"].stringValue))
                    let userInfo = UserItemModel(id: Int(subJson["user"]["id"].stringValue)!, userHead: String(subJson["user"]["avatar_url"].stringValue), userName: String(subJson["user"]["name"].stringValue), userAccount: String(subJson["user"]["login"].stringValue))
                    let issueInfo = IssueModel(id: Int(subJson["id"].stringValue)!, issueId: String(subJson["number"].stringValue), issueTitle: String(subJson["title"].stringValue), issueTime: Helper.getDateFromString(str: String(subJson["created_at"].stringValue)), issueDesc: String(subJson["body"].stringValue), issueStatus: getIssueStatus(status: subJson["state"].stringValue), repoInfo:repoInfo, userInfo: userInfo)
                    tempList.append(
                        issueInfo
                    )
                }
                self.issuesList = tempList
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
这里利用localConfig拿到了存储的筛选数据，通过HttpRequest实现获取数据。

### 2.2 你的私信

```Swift
Section(header: Text("消息中心")) {
            NavigationLink(destination: MailView()) {
                HomeListItem(title: "你的私信",icon:"envelope.circle", color: Color(hex: 0x16c3fc))
            }
            NavigationLink(destination: NotificationView()) {
                HomeListItem(title: "你的通知",icon:"bell.circle", color: Color(hex: 0xf83860))
            }
        }
```
这里是首页的入口。

点击你的私信会跳转到MailView页面。
首先看下声明的状态变量：
```Swift
struct MailView: View {
    @State var mailList: [MailModel] = []
    @State var waitPlease = false
    @State var isRefreshing = false
    @State var isLoading: Bool = false
    @State var isModal: Bool = false
    @State var message: String = "数据加载中"
    @State var page = 1
```

然后是body声明：
```Swift
var body: some View {
    LoadingView(isLoading:self.$isLoading,message: self.$message,isModal:self.$isModal) {
        ZStack{
            if mailList.count == 0 && !isLoading {
                VStack{
                    Image(systemName: "doc.text.magnifyingglass")
                        .scaleEffect(3, anchor: .center)
                    Text("暂无查询到的私信").padding(.top,30)
                }
            }
            RefreshView(refreshing: $isRefreshing, action: {
                self.page = 1
                self.getMailList(page: self.page)
            }) {
                LazyVStack{
                    ForEach(self.mailList){ item in
                        MailItemView(mailItem: item)
                            .onAppear(){
                                if !waitPlease && item.id == mailList[mailList.count - 1].id {
                                    self.page = self.page + 1
                                    self.getMailList(page: self.page)
                                }
                            }
                    }
                }
            }
        }
    }
    .padding(.top,5)
    .navigationBarTitle(Text("你的私信"), displayMode: .inline)
    .onAppear(){
        self.page = 1
        self.getMailList(page: self.page)
    }
}
```
也是一个LoadingView负责加载loading的封装视图，然后是一个刷新RefreshView。
里面是循环的item。
可见的时候走getMailList接口。

先看下item吧：
```Swift
struct MailItemView:View{
    @State var mailItem: MailModel
    @State var placeholderImage = UIImage(named: "Logo")!
    @State var isMessageReplyShow:Bool = false
    var body: some View{
        ZStack{
            VStack{

            }
            .frame(width: 0, height: 0)
            .opacity(0)
            
            VStack{
                HStack(alignment: .top) {
                    Image(uiImage: placeholderImage)
                        .resizable()
                        .scaledToFit()
                        .frame(
                            width:40,height:40,
                            alignment: .center
                        )
                        .cornerRadius(5)
                        .onAppear(){
                            guard let url = URL(string: mailItem.user.userHead) else {
                                return
                            }
                            URLSession.shared.dataTask(with: url) { (data, response, error) in
                                if let data = data, let image = UIImage(data: data) {
                                    placeholderImage = image
                                }
                            }.resume()
                        }
                    VStack(alignment: .leading){
                        HStack(alignment:.top){
                            Text(mailItem.user.userName)
                                .padding(0)
                            Spacer()
                            Text(mailItem.time).font(.system(size:12)).foregroundColor(.gray)
                        }
                        Text(mailItem.message).font(.system(size:14)).foregroundColor(.gray)
                            .padding(.top,1)
                            .lineLimit(10)
                            .fixedSize(horizontal: false, vertical: true)
                    }
                }
                .padding(10)
            }
            .background(Color(red: 1, green: 1, blue: 1, opacity: 0.1))
            .cornerRadius(10)
            .padding(.horizontal,5)
            .padding(.bottom,-3)
            .onTapGesture {
                self.isMessageReplyShow = true
            }
            .sheet(isPresented: $isMessageReplyShow,onDismiss: {
                //TODO
            }){
                MailReplyView(user: mailItem.user)
            }
        }
    }
}
```

这里点击item后会展示MailReplyView回复私信界面：
<img src=gitee_02_04.png>

对应代码如下：
```Swift
struct MailReplyView: View {
    @Environment(\.presentationMode) var mode
    @State var user: UserItemModel
    @State var content: String = ""
    @State var isLoading: Bool = false
    @State var isModal: Bool = false
    @State var message: String = "回复中"
    
    @State private var alertShow = false
    @State var alertTitle:String = ""
    @State var alertMessage:String = ""
    func startAlert(title:String,message:String){
        self.alertTitle = title
        self.alertMessage = message
        self.alertShow = true
    }
    var body: some View {
        LoadingView(isLoading:self.$isLoading,message: self.$message,isModal:self.$isModal) {
            NavigationView{
                VStack{
                    Form{
                        Section(header: Text("你正在回复 @" + user.userName)) {
                            TextEditor(text: $content)
                                .foregroundColor(.white)
                                .font(.system(size:16))
                                .lineLimit(5)
                        }
                    }
                    Spacer()
                }
                .navigationBarTitle(Text("回复私信"), displayMode: .inline)
                .navigationBarItems(
                    trailing:
                        Button(action: {
                            if self.content.count < 1 {
                                self.startAlert(title: "回复失败", message: "你确定不说点什么吗???")
                                return
                            }
                            self.isLoading = true
                            self.isModal = true
                            HttpRequest(url: "notifications/messages",withAccessToken: true).doPost(postData: ["username":user.userAccount,"content":content]) { (result) in
                                let json = JSON(result)
                                self.isLoading = false
                                self.isModal = false
                                if json["message"].string != nil{
                                    self.startAlert(title: "回复失败", message: json["message"].stringValue)
                                }else{
                                    self.content = ""
                                    self.startAlert(title: "回复成功", message: "你的消息回复成功")
//                                    self.mode.wrappedValue.dismiss()
                                }
                            } errorCallback: {
                                self.isLoading = false
                                self.isModal = false
                                self.startAlert(title: "回复失败", message:"发生了一点小错误,请稍候再试")
                            }
                        }) {
                            Text("回复").foregroundColor(.yellow)
                        })
                
            }
            .preferredColorScheme(.dark)
            .alert(isPresented: $alertShow) {
                Alert(title: Text(alertTitle),message: Text(alertMessage),dismissButton: .default(Text("好的")))
            }
        }
    }
}
```

注意到这里有句代码：
```
@Environment(\.presentationMode) var mode
```
是什么意思呢？
> @Environment(\.presentationMode) var mode 这句代码用于声明一个环境变量，它使用 @Environment 声明，后面跟着的是环境变量的键值：\.presentationMode。
环境变量是 SwiftUI 中的一个特殊类型，它允许我们在不同的视图层次结构中访问一些全局的环境值。在这种情况下，我们声明的环境变量 mode 包含了一个名为 presentationMode 的环境值。这个环境值可以被用于管理当前视图的呈现模式，以便控制其是否显示或被关闭。
例如，如果您在模态视图中提供一个“取消”按钮，您可以在按钮的单击事件中使用 mode.dismiss 来关闭模态视图。

### 2.3 消息通知

入口如下：
```Swift
 Section(header: Text("消息中心")) {
    NavigationLink(destination: MailView()) {
        HomeListItem(title: "你的私信",icon:"envelope.circle", color: Color(hex: 0x16c3fc))
    }
    NavigationLink(destination: NotificationView()) {
        HomeListItem(title: "你的通知",icon:"bell.circle", color: Color(hex: 0xf83860))
    }
}
```

这里点击你的通知后，会跳转到NotificationView。

先声明几个状态变量：
```Swift
struct NotificationView: View {
    @State var notificationList: [NotificationModel] = []
    @State var waitPlease = false
    @State var isRefreshing = false
    @State var isLoading: Bool = false
    @State var isModal: Bool = false
    @State var message: String = "数据加载中"
    @State var page = 1
```

然后是视图结构：
```Swift 
var body: some View {
    LoadingView(isLoading:self.$isLoading,message: self.$message,isModal:self.$isModal) {
        ZStack{
            if notificationList.count == 0 && !isLoading {
                VStack{
                    Image(systemName: "doc.text.magnifyingglass")
                        .scaleEffect(3, anchor: .center)
                    Text("暂无查询到的通知").padding(.top,30)
                }
            }
            RefreshView(refreshing: $isRefreshing, action: {
                self.page = 1
                self.getNotificationList(page: self.page)
            }) {
                LazyVStack{
                    ForEach(self.notificationList){ item in
                        NotificationItemView(notificationItem: item)
                            .onAppear(){
                                if !waitPlease && item.id == notificationList[notificationList.count - 1].id {
                                    self.page = self.page + 1
                                    self.getNotificationList(page: self.page)
                                }
                            }
                    }
                }
            }
        }
    }
    .padding(.top,5)
    .navigationBarTitle(Text("消息通知"), displayMode: .inline)
    .onAppear(){
        self.page = 1
        self.getNotificationList(page: self.page)
    }
}
```

注意到在onAppear的生命周期中调用了接口。

然后对应的消息通知的item视图结构如下：
```Swift 
struct NotificationItemView:View{
    @State var notificationItem: NotificationModel
    @State var placeholderImage = UIImage(named: "Logo")!
    @State var isRepoDetailShow:Bool = false
    @State var isPullRequestShow:Bool = false
    @State var isCommitShow:Bool = false
    @State var isIssueShow:Bool = false
    var body: some View{
        ZStack{
            VStack{
                NavigationLink(destination: RepoDetailView(repoFullPath: self.notificationItem.repo.fullName), isActive: $isRepoDetailShow) { EmptyView() }
                NavigationLink(destination: PullRequestView(repoFullPath: self.notificationItem.repo.fullName), isActive: $isPullRequestShow) { EmptyView() }
                NavigationLink(destination: CommitView(repoFullPath: self.notificationItem.repo.fullName, repoDefaultBranch: "master"), isActive: $isCommitShow) { EmptyView() }
                NavigationLink(destination:  IssuesView(repoPath: self.notificationItem.repo.fullName), isActive: $isIssueShow) { EmptyView() }

            }
            .frame(width: 0, height: 0)
            .opacity(0)
            VStack{
                HStack(alignment: .top) {
                    Image(uiImage: placeholderImage)
                        .resizable()
                        .scaledToFit()
                        .frame(
                            width:40,height:40,
                            alignment: .center
                        )
                        .cornerRadius(5)
                        .onAppear(){
                            guard let url = URL(string: notificationItem.user.userHead) else {
                                return
                            }
                            URLSession.shared.dataTask(with: url) { (data, response, error) in
                                if let data = data, let image = UIImage(data: data) {
                                    placeholderImage = image
                                }
                            }.resume()
                        }
                    VStack(alignment: .leading){
                        HStack(alignment:.top){
                            Text(notificationItem.user.userName)
                                .padding(0)
                            Spacer()
                            Text(notificationItem.time).font(.system(size:12)).foregroundColor(.gray)
                        }
                        Text(notificationItem.message).font(.system(size:14)).foregroundColor(.gray)
                            .padding(.top,1)
                            .lineLimit(10)
                            .fixedSize(horizontal: false, vertical: true)
                    }
                }
                .padding(10)
            }
            .background(Color(red: 1, green: 1, blue: 1, opacity: 0.1))
            .cornerRadius(10)
            .padding(.horizontal,5)
            .padding(.bottom,-3)
            .onTapGesture {
            }
            .contextMenu(ContextMenu {
                Button(action: {
                    self.isRepoDetailShow = true
                }) {
                    HStack{
                        Image(systemName: "archivebox.circle").scaleEffect(1, anchor: .center)
                        Spacer()
                        Text("进入仓库")
                    }
                }
                Divider()
                Button(action: {
                    self.isCommitShow = true
                }) {
                    HStack{
                        Image(systemName: "icloud.and.arrow.up").scaleEffect(1, anchor: .center)
                        Spacer()
                        Text("查看提交")
                    }
                }
                Button(action: {
                    self.isPullRequestShow = true
                }) {
                    HStack{
                        Image(systemName: "shuffle.circle").scaleEffect(1, anchor: .center)
                        Spacer()
                        Text("查看Pull Request")
                    }
                }
                Button(action: {
                    self.isIssueShow = true
                }) {
                    HStack{
                        Image(systemName: "exclamationmark.circle").scaleEffect(1, anchor: .center)
                        Spacer()
                        Text("查看Issue")
                    }
                }
                Divider()
                Button(action: {
                }) {
                    HStack{
                        Image(systemName: "person.circle").scaleEffect(1, anchor: .center)
                        Spacer()
                        Text("用户资料")
                    }
                }
            })
        }
    }
}
```

这里长按item后会弹出一个menu弹框：
<img src=gitee_02_05.png>

这里怎么弹出来的呢？
答案是通过View的扩展函数contextMenu方法：
> contextMenu 方法是 SwiftUI 中的一个视图方法，可以在视图上创建一个上下文菜单。上下文菜单是当用户右键单击视图时显示的菜单。
您可以使用 contextMenu 方法创建一个上下文菜单，并在该菜单中显示选项。

然后点击事件呢？
发现这里只是改变了一个状态变量而已哦。
```Swift
Button(action: {
    self.isRepoDetailShow = true
}) {
    HStack{
        Image(systemName: "archivebox.circle").scaleEffect(1, anchor: .center)
        Spacer()
        Text("进入仓库")
    }
}
```
改变了` self.isRepoDetailShow = true`
这个变量就跳转了？

原来答案在前面哦：
```Swift
var body: some View{
    ZStack{
        VStack{
            NavigationLink(destination: RepoDetailView(repoFullPath: self.notificationItem.repo.fullName), isActive: $isRepoDetailShow) { EmptyView() }
            NavigationLink(destination: PullRequestView(repoFullPath: self.notificationItem.repo.fullName), isActive: $isPullRequestShow) { EmptyView() }
            NavigationLink(destination: CommitView(repoFullPath: self.notificationItem.repo.fullName, repoDefaultBranch: "master"), isActive: $isCommitShow) { EmptyView() }
            NavigationLink(destination:  IssuesView(repoPath: self.notificationItem.repo.fullName), isActive: $isIssueShow) { EmptyView() }

        }
        .frame(width: 0, height: 0)
```
这里相当于监听了 isRepoDetailShow变量，如果监听到了，需要跳转到：
RepoDetailView界面。

其它item类似的。

### 2.4 加入的组织

入口在这里：
```Swift
Section(header: Text("你的团队")) {
            NavigationLink(destination:  OrganizationView()) {
                HomeListItem(title: "加入的组织",icon:"person.2.circle", color: Color(hex: 0xb76dda))
            }
//                NavigationLink(destination:  RepoView()) {
//                    HomeListItem(title: "所在的企业",icon:"paperplane.circle", color: Color(hex: 0xffc55a))
//                }
        }
```

这里会直接跳转到OrganizationView的页面。

先看下状态变量：
```Swift
struct OrganizationView: View {
    @State var orgList: [OrgModel] = []
    @State var waitPlease = false
    @State var isRefreshing = false
    @State var isLoading: Bool = false
    @State var isModal: Bool = false
    @State var message: String = "数据加载中"
    @State var page = 1
```

然后是视图结构：
```Swift
var body: some View {
    LoadingView(isLoading:self.$isLoading,message: self.$message,isModal:self.$isModal) {
        ZStack{
            if orgList.count == 0 && !isLoading {
                VStack{
                    Image(systemName: "doc.text.magnifyingglass")
                        .scaleEffect(3, anchor: .center)
                    Text("暂无查询到的组织").padding(.top,30)
                }
            }
            RefreshView(refreshing: $isRefreshing, action: {
                self.page = 1
                self.getOrgList(page: self.page)
            }) {
                LazyVStack{
                    ForEach(self.orgList){ item in
                        OrgItemView(orgItem: item)
                            .onAppear(){
                                if !waitPlease && item.id == orgList[orgList.count - 1].id {
                                    self.page = self.page + 1
                                    self.getOrgList(page: self.page)
                                }
                            }
                    }
                }
            }
        }
    }
    .padding(.top,5)
    .navigationBarTitle(Text("你的组织"), displayMode: .inline)
    .onAppear(){
        self.page = 1
        self.getOrgList(page: self.page)
    }
}
```

item定义的视图：
```Swift
struct OrgItemView:View{
    @State var orgItem: OrgModel
    @State var placeholderImage = UIImage(named: "Logo")!
    var body: some View{
        ZStack{
            VStack{

            }
            .frame(width: 0, height: 0)
            .opacity(0)
            VStack{
                HStack(alignment: .top) {
                    Image(uiImage: placeholderImage)
                        .resizable()
                        .scaledToFit()
                        .frame(
                            width:80,height:80,
                            alignment: .center
                        )
                        .cornerRadius(10)
                        .onAppear(){
                            guard let url = URL(string: orgItem.head) else {
                                return
                            }
                            URLSession.shared.dataTask(with: url) { (data, response, error) in
                                if let data = data, let image = UIImage(data: data) {
                                    placeholderImage = image
                                }
                            }.resume()
                        }
                    VStack(alignment: .leading){
                        HStack(alignment:.top){
                            Text(orgItem.name)
                                .padding(0)
                            Spacer()
                            Text("粉丝:" + String(orgItem.fans)).font(.system(size:12)).foregroundColor(.gray)
                        }
                        Text(orgItem.desc).font(.system(size:14)).foregroundColor(.gray)
                            .padding(.top,1)
                            .lineLimit(10)
                            .fixedSize(horizontal: false, vertical: true)
                    }
                }
                .padding(10)
            }
            .background(Color(red: 1, green: 1, blue: 1, opacity: 0.1))
            .cornerRadius(10)
            .padding(.horizontal,5)
            .padding(.bottom,-3)
            
```

这个和其他列表页也比较相似。不再过多讲述了。

## 3 动态
这是首页的第二个Tab页。
首先在这里定义：
```Swift 
struct TabBarView: View {
    @State var selectedBarIndex:Int
    
    var body: some View {
        NavigationView{
            TabView(selection: $selectedBarIndex) {
                HomeView()
                .tabItem {
                    Image(systemName:"briefcase")
                    Text("首页")
                }
                    .tag(0)
                ActivityView()
                .tabItem {
                    Image(systemName:"stopwatch")
                    Text("动态")
                }
                    .tag(1)
```
这里叫做ActivityView。

先声明动态变量：
```Swift
struct ActivityView: View {
    @State var activityList: [ActivityModel] = []
    @State var waitPlease = false
    @State var isRefreshing = false
    @State var isLoading: Bool = false
    @State var isModal: Bool = false
    @State var message: String = "数据加载中"
    @State var isLoginShow = false
    @State var lastId = 0 //331672539
    
    @State var myInfoString:JSON? = nil;
```

然后是视图结构定义：
```Swift
var body: some View {
    LoadingView(isLoading:self.$isLoading,message: self.$message,isModal:self.$isModal) {
        ZStack{
            if activityList.count == 0 && !isLoading {
                VStack{
                    Image(systemName: "doc.text.magnifyingglass")
                        .scaleEffect(3, anchor: .center)
                    Text("动态读取中").padding(.top,30)
                }
            }
            RefreshView(refreshing: $isRefreshing, action: {
                self.lastId = 0
                self.getActivityList()
            }) {
                LazyVStack{
                    ForEach(self.activityList){ item in
                        ActivityItemView(activity: item)
                        .onAppear(){
                            if !waitPlease && item.id == activityList[activityList.count - 1].id {
                                self.getActivityList()
                            }
                        }
                    }
                }
            }
        }
        .sheet(isPresented: $isLoginShow,onDismiss: {
            self.lastId = 0
            self.getActivityList()
        }){
            LoginView()
                .modifier(DisableModalDismiss(disabled: true))
        }
        .onAppear(){
            self.lastId = 0
            self.getActivityList()
        }
    }
}
```

具体看下item:
```Swift
struct ActivityItemView:View{
    @State var activity: ActivityModel
    @State var placeholderImage = UIImage(named: "nohead")!
    @State var isActivePullRequest:Bool = false
    @State var isActiveRepoDetail:Bool = false
    @State var isActiveCommit:Bool = false
    @State var isActiveIssues:Bool = false
    var body: some View{
        NavigationLink(destination: PullRequestView(repoFullPath: activity.repoInfo.fullName), isActive: $isActivePullRequest) { EmptyView() }
        NavigationLink(destination: RepoDetailView(repoFullPath: activity.repoInfo.fullName), isActive: $isActiveRepoDetail) { EmptyView() }
        NavigationLink(destination:
                        CommitView(repoFullPath: activity.repoInfo.fullName, repoDefaultBranch: "master"), isActive: $isActiveCommit) { EmptyView() }
        NavigationLink(destination:  IssuesView(repoPath:activity.repoInfo.fullName), isActive: $isActiveIssues) { EmptyView() }
        
        VStack(alignment: .leading){
            VStack(alignment: .leading){
                HStack(alignment: .top) {
                    Image(uiImage: placeholderImage)
                        .resizable()
                        .scaledToFit()
                        .frame(
                            width:40,height:40,
                            alignment: .center
                        )
                        .cornerRadius(5)
                        .onAppear(){
                            guard let url = URL(string: activity.userInfo.userHead) else {
                                return
                            }
                            URLSession.shared.dataTask(with: url) { (data, response, error) in
                                if let data = data, let image = UIImage(data: data) {
                                    placeholderImage = image
                                }
                            }.resume()
                        }
                    VStack(alignment: .leading){
                        HStack(alignment:.top){
                            Text(activity.userInfo.userName)
                                .padding(0)
                            Spacer()
                            Text(activity.createTime).font(.system(size:12)).foregroundColor(.gray)
                        }
                        Text(getActivityContent(activity:activity)).font(.system(size:14)).foregroundColor(.gray)
                            .padding(.top,1)
                            .lineLimit(10)
                            .fixedSize(horizontal: false, vertical: true)
                    }
                }
            }
            .padding(10)
            Spacer()
        }
//        .frame(height:100)
        .background(Color(red: 1, green: 1, blue: 1, opacity: 0.1))
        .cornerRadius(10)
        .padding(.horizontal,5)
        .padding(.top,5)
```

## 4 总结

* 需要理解下@State注解用法，这个主要是一个状态变量，可以理解成 didSet，就是有一个回调作用，在Swift UI中会通知更新界面元素。 或者可以理解成Android中的LiveData变量。

* View里面扩展了一个方法navigationBarItems，是标题栏的右侧点击事件，然后有个sheet方法，是类似模态框，里面的闭包View就是要弹出的视图。

* Appear声明周期，就是首次可见的生命周期，一般可以在这里面请求网络。同时最好配合LoadingView进行网络加载，一般是用一个加载圈包裹列表视图，通过isLoading变量控制加载器的显示或隐藏。

* 这里有个Environment的注解，主要是定义了全局的环境值，我们可以用来管理当前视图的显示或关闭。

* View里面扩展了一个contextMenu方法，可以用来显示上下文菜单。然后通过在顶部配置NavigationLink来监听isActive定义的变量，当点击某个上下文菜单，我们改变这个状态变量，然后通知到这里isActive，然后就可以跳转到destination定义的View页面了。