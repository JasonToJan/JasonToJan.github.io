---
title: iOS swift Gitee客户端 完整项目分析之一
date: 2023-02-06 14:47:06
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

## 2 项目截图
||||
|--|--|--|
| <img src=1.png> | <img src=2.png> | <img src=3.png> |
| <img src=4.png> | <img src=5.png> | <img src=6.png> |

## 3 首页架构

这里是底部3个Tab，首页展示一个工作台消息中心，你的团队的入库。应该是写死的布局。

第二个tab是好友动态，这里展示一个列表，展示动态列表。

第三个tab是设置页，上方是头像，然后是关注粉丝star啥的。

关于App和Scene代码结构，可以参考这篇文章：
[SwiftUI2.0 —— App、Scene及新的代码结构（一）](https://zhuanlan.zhihu.com/p/152624613)。

应用初始化代码：
```Swift
@main
struct GiteeApp: App {
    var body: some Scene {
        WindowGroup {
            TabBarView(selectedBarIndex: 0)
                .preferredColorScheme(.dark)
        }
    }
}
```
这里body就是首页了。
App，Scene，WindowGroup都是系统类。

里面的TabBarView是自定义View来的：
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
//                ExploreView()
//                .tabItem {
//                    Image(systemName:"opticaldisc")
//                    Text("发现")
//                }
//                    .tag(2)
                SettingView()
                .tabItem {
                    Image(systemName:"gearshape")
                    Text("设置")
                }
                    .tag(3)
            }
            .navigationBarTitle(getNavBarTitle(),displayMode: getNavBarModel())
            .navigationBarHidden(self.selectedBarIndex==3)
            .foregroundColor(.white)
            .accentColor(.white) //这里修改文字颜色
        }
        .navigationViewStyle(StackNavigationViewStyle())
        .accentColor(.white)
        .foregroundColor(.white)
        .preferredColorScheme(.dark)
    }
```
这里TabView其实就是定义的底部Tab，然后item点击后是配置在中间的。
比如HomeView就是首页了，ActivityView应该就是动态页了，SettingView就是设置页了。

```Swift

func getNavBarModel() -> NavigationBarItem.TitleDisplayMode{
    if self.selectedBarIndex == 0 {
        return .large
    }
    if self.selectedBarIndex == 1 {
        return .inline
    }
    if self.selectedBarIndex == 2 {
        return .large
    }
    if self.selectedBarIndex == 3 {
        return .large
    }
    return .inline
}
func getNavBarTitle() -> String{
    if self.selectedBarIndex == 0 {
        return "Gitee"
    }
    if self.selectedBarIndex == 1 {
        return "好友动态"
    }
    if self.selectedBarIndex == 2 {
        return "发现"
    }
    if self.selectedBarIndex == 3{
        return "设置"
    }
    return  ""
}
```
这里配置tab具体数据。
![](./iOS-swift-Gitee%E5%AE%A2%E6%88%B7%E7%AB%AF-%E5%AE%8C%E6%95%B4%E9%A1%B9%E7%9B%AE%E5%88%86%E6%9E%90%E4%B9%8B%E4%B8%80/tab1.png)
<img src=tab1.png>
这里Form是表格。
Section是组别。
这里有一个NavigationLink是item1就是第一个item，这里显示代码仓库。颜色和icon都可以设置的。

 但是这里的HomeListItem是自己定义的哦。
 ```Swift
struct HomeListItem : View {
    @State var title: String
    @State var icon: String
    @State var color: Color
    
    var body: some View {
        HStack(spacing: 15){
            Image(systemName:icon)
                .scaleEffect(1.5, anchor: .center)
                .foregroundColor(color)
            Text(title).font(.system(size:16))
        }
        .padding(.horizontal, 5)
        .frame( height: 48)
        .cornerRadius(10)
    }
}
 ```

这个HStack是系统的，估计类似于Flutter里面的Stack吧。
这里右侧箭头是咋来的呢？
原来是
```Swift
var body: some View {
    Form {
        Section(header: Text("工作台")) {
            NavigationLink(destination: RepoView()) {
                HomeListItem(title: "代码仓库",icon:"archivebox.circle", color: Color(hex: 0x7699ec))
            }
//                NavigationLink(destination:  PullRequestView()) {
//                    HomeListItem(title: "Pull Request",icon:"shuffle.circle", color: Color(hex: 0x00b392))
//                }
            NavigationLink(destination:  IssuesView()) {
                HomeListItem(title: "Issues",icon:"exclamationmark.circle", color: Color(hex: 0xfe665b))
            }
        }
```
这里面的NavigationLink自带的箭头哦。

### 3.2 仓库页面
前面用了一个NavigationLink包裹，那么点击这个之后应该是跳转到目标页面。

这里是先跳转到你的仓库页面。
先看看哈：
```Swift
var body: some View {
    LoadingView(isLoading:self.$isLoading,message: self.$message,isModal:self.$isModal) {
        ZStack{
            if repoList.count == 0 && !isLoading {
                VStack{
                    Image(systemName: "doc.text.magnifyingglass")
                        .scaleEffect(3, anchor: .center)
                    Text("暂无查询到的仓库").padding(.top,30)
                }
            }
            RefreshView(refreshing: $isRefreshing, action: {
                self.page = 1
                self.getRepoList(page: self.page)
            }) {
                LazyVStack{
                    ForEach(self.repoList){ item in
                        NavigationLink(destination: RepoDetailView(repoFullPath: item.repoNamespace.path + "/" + item.repoPath)) {
                            RepoItemView(repoItem: item)
                                .onAppear(){
                                    if !waitPlease && item.id == repoList[repoList.count - 1].id {
                                        self.page = self.page + 1
                                        self.getRepoList(page: self.page)
                                    }
                                }
                        }
                    }
                }
            }
        }
    }
    .sheet(isPresented: $isLoginShow,onDismiss: {
        UserModel().getMyInfo { (userInfo) in
            self.page = 1
            self.getRepoList(page: self.page)
        } error: {
            self.isLoginShow.toggle()
        }
    }){
        LoginView()
            .modifier(DisableModalDismiss(disabled: true))
    }
    .padding(.top,5)
    .navigationBarTitle(Text(naviTitle), displayMode: .inline)
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
                self.getRepoList(page: self.page)
            }){
                RepoFilterView(showListFrom: self.showListFrom)
                    .modifier(DisableModalDismiss(disabled: false))
            }
    )
    .onAppear(){
        switch self.showListFrom {
        case ShowRepoListFrom.fromWatches:
            self.naviTitle = "Watch的仓库"
        case ShowRepoListFrom.fromStars:
            self.naviTitle = "Star的仓库"
        default:
            self.naviTitle = "你的仓库"
        }
        
        localConfig.setValue("all", forKey: giteeConfig.repo_type)
        localConfig.setValue("pushed", forKey: giteeConfig.repo_sort)
        localConfig.setValue("desc", forKey: giteeConfig.repo_direction)
        self.page = 1
        self.getRepoList( page: self.page)
    }
}
```

loading视图：
```Swift
struct LoadingView<Content>: View where Content: View {
    
    @Binding var isLoading: Bool
    @Binding var message:String
    @Binding var isModal: Bool
    @State var isAnimating: Bool = true
    var content: () -> Content
    
    var body: some View {
        GeometryReader { geometry in
            ZStack(alignment: .center) {
                self.content()
                    .disabled(self.isModal)
                    .blur(radius: self.isModal ? 10 : 0)
                VStack {
                    Text(self.message)
                        .padding(.bottom,20)
                    ActivityIndicatorLoading(isAnimating: self.$isAnimating, style: .large)
                }
                    
                .frame(width: geometry.size.width / 2,
                       height: geometry.size.height / 5)
                    .background(Color.secondary.colorInvert())
                    .foregroundColor(Color.primary)
                    .cornerRadius(20)
                    .opacity(self.isLoading ? 1 : 0)
            }
        }
    }
}
```
这里的Loading效果是ActivityIndicatorLoading这个实现的。这个是自定义的哦。
继续走这个：
```Swift
struct ActivityIndicatorLoading: UIViewRepresentable {
    
    @Binding var isAnimating: Bool
    let style: UIActivityIndicatorView.Style
    
    func makeUIView(context: UIViewRepresentableContext<ActivityIndicatorLoading>) -> UIActivityIndicatorView {
        return UIActivityIndicatorView(style: style)
    }
    
    func updateUIView(_ uiView: UIActivityIndicatorView, context: UIViewRepresentableContext<ActivityIndicatorLoading>) {
        isAnimating ? uiView.startAnimating() : uiView.stopAnimating()
    }
}
```
原来是UIActivityIndicatorView这个类来实现loading效果的。

然后这里有个自定义刷新视图。
可以参考下这篇文章：[https://swiftui-lab.com/scrollview-pull-to-refresh/](https://swiftui-lab.com/scrollview-pull-to-refresh/)。

```Swift
LazyVStack{
    ForEach(self.repoList){ item in
        NavigationLink(destination: RepoDetailView(repoFullPath: item.repoNamespace.path + "/" + item.repoPath)) {
            RepoItemView(repoItem: item)
                .onAppear(){
                    if !waitPlease && item.id == repoList[repoList.count - 1].id {
                        self.page = self.page + 1
                        self.getRepoList(page: self.page)
                    }
                }
        }
    }
}
```
这里就是刷新器里面展示列表。

列表的item是这个RepoItemView:
```Swift
struct RepoItemView:View{
    @State var repoItem: RepoModel
    @State private var isActiveCommit:Bool = false
    @State private var isActivePullRequest:Bool = false
    @State private var isActiveIssues:Bool = false
    
    var body: some View{
        VStack{
            VStack{
                NavigationLink(destination: CommitView(repoFullPath: repoItem.repoNamespace.path + "/" + repoItem.repoPath,repoDefaultBranch: repoItem.repoDefaultBranch), isActive: $isActiveCommit) { EmptyView() }
                NavigationLink(destination: PullRequestView(repoFullPath: repoItem.repoNamespace.path + "/" + repoItem.repoPath), isActive: $isActivePullRequest) { EmptyView() }
                NavigationLink(destination: IssuesView(repoPath: repoItem.repoNamespace.path + "/" + repoItem.repoPath),isActive: $isActiveIssues) { EmptyView() }
            }
            .frame(width: 0, height: 0)
            .opacity(0)
            VStack(alignment: .leading){
                HStack(alignment: .top) {
                    if !self.repoItem.repoIsOpenSource {
                        Image(systemName: "lock.square.fill")
                            .foregroundColor(Color(hex: 0xffc55a))
                            .padding(.trailing,-5)
                            .scaleEffect(1, anchor: .center)
                    }
                    VStack(alignment: .leading){
                        Text(self.repoItem.repoNamespace.name + "/" + self.repoItem.repoName)
                            .font(.system(size: 16))
                            .lineLimit(1)
                    }
                    Spacer()
                    Text(self.repoItem.repoPushDate)
                        .font(.system(size: 12))
                        .foregroundColor(.gray)
                    
                }
                .padding(5)
                Text(self.repoItem.repoDesc == "" ? "很尴尬,该项目暂无介绍..." : self.repoItem.repoDesc)
                    .font(.system(size: 14))
                    .foregroundColor(.gray)
                    .lineLimit(3)
                    .multilineTextAlignment(.leading)
                    .padding(.top,10)
                    .padding(.leading,5)
                    .fixedSize(horizontal: false, vertical: true)
                Spacer()
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
                    Spacer()
                    HStack{
                        Image(systemName: "star.fill")
                            .foregroundColor(.gray)
                            .scaleEffect(0.6, anchor: .center)
                            .padding(0)
                        Text(self.repoItem.repoStars)
                            .font(.system(size: 12))
                            .foregroundColor(.gray)
                            .padding(0)
                            .padding(.leading,-12)
                    }
                    HStack{
                        Image(systemName: "eye.fill")
                            .foregroundColor(.gray)
                            .scaleEffect(0.6, anchor: .center)
                            .padding(0)
                        Text(self.repoItem.repoWatches)
                            .font(.system(size: 12))
                            .foregroundColor(.gray)
                            .padding(0)
                            .padding(.leading,-12)
                    }
                    .padding(.horizontal,-8)
                    HStack{
                        Image(systemName: "arrowshape.turn.up.backward.2.fill")
                            .foregroundColor(.gray)
                            .scaleEffect(0.7, anchor: .center)
                            .padding(0)
                        Text(self.repoItem.repoForks)
                            .font(.system(size: 12))
                            .foregroundColor(.gray)
                            .padding(0)
                            .padding(.leading,-12)
                    }
                }
                .padding(.top,5)
            }
            .padding(10)
        }
        .background(Color(red: 1, green: 1, blue: 1, opacity: 0.1))
        .cornerRadius(10)
        .padding(.horizontal,5)
        .padding(.bottom,5)
        //        .swipeCell(
        //            cellPosition: .both,
        //            leftSlot: nil,
        //            rightSlot: SwipeCellSlot(
        //                slots:
        //                    [
        //                        SwipeCellButton(
        //                            buttonStyle: .titleAndImage,
        //                            title: "Mark",
        //                            systemImage: "bookmark",
        //                            titleColor: .white,
        //                            imageColor: .white,
        //                            view: nil,
        //                            backgroundColor: .green,
        //                            action: {
        //                                print("123")
        //                            },
        //                            feedback:true
        //                        )
        //                    ]
        //            )
        //        )
        .contextMenu(ContextMenu {
            Button(action: {
                self.isActiveCommit = true
            }) {
                HStack{
                    Image(systemName: "icloud.and.arrow.up").scaleEffect(1, anchor: .center)
                    Spacer()
                    Text("提交记录")
                }
            }
            Divider()
            Button(action: {
                self.isActivePullRequest = true
            }) {
                HStack{
                    Image(systemName: "shuffle.circle").scaleEffect(1, anchor: .center)
                    Spacer()
                    Text("Pull Requests")
                }
            }
            Button(action: {
                self.isActiveIssues = true
            }) {
                HStack{
                    Image(systemName: "exclamationmark.circle").scaleEffect(1, anchor: .center)
                    Spacer()
                    Text("查看Issues")
                }
            }
        })
    }
}
```

长按会弹出菜单，提交记录，Pull Requests Issues都是菜单项。
注意到这里的菜单是一个Button，里面配置了action，然后在顶部配置了NavigationLink，实现跳转。

### 3.3 提交记录 

比如点击了提交记录：
这里显示body:
```Swift
var body: some View {
    LoadingView(isLoading:self.$isLoading,message: self.$message,isModal:self.$isModal) {
        ZStack{
            if commitList.count == 0 && !isLoading {
                VStack{
                    Image(systemName: "doc.text.magnifyingglass")
                        .scaleEffect(3, anchor: .center)
                    Text("暂无查询到的提交").padding(.top,30)
                }
            }
            RefreshView(refreshing: $isRefreshing, action: {
                self.page = 1
                self.getCommitList(page: self.page)
            }) {
                LazyVStack{
                    ForEach(self.commitList){ item in
                        CommitItemView(commitItem: item, repoFullPath: self.repoFullPath!)
                            .onAppear(){
                                if !waitPlease && item.id == commitList[commitList.count - 1].id {
                                    self.page = self.page + 1
                                    self.getCommitList(page: self.page)
                                }
                            }
                    }
                }
            }
        }
        .padding(.top,5)
        .navigationBarTitle(Text(self.showCommitFrom == CommitFromModel.fromRepo ? branch : "包含的提交"), displayMode: .inline)
        .navigationBarItems(
            trailing:
                HStack {
                    if self.showCommitFrom == CommitFromModel.fromRepo{
                        Button {
                        } label: {
                            Menu {
                                ForEach (0 ..< self.branchList.count, id: \.self) {index in
                                    Button(self.branchList[index], action: {
                                        self.branch = self.branchList[index]
                                        self.page = 1
                                        self.isLoading = true
                                        self.getCommitList(page: self.page)
                                    })
                                }
                            } label: {
                                VStack{
                                    Text("分支").foregroundColor(.yellow)
                                }
                            }
                        }
                    }
                }
        )
        .onAppear(){
            if self.showCommitFrom == CommitFromModel.fromRepo{
                self.branch = self.repoDefaultBranch!
            }else{
                self.branch = "master"
            }
            self.page = 1
            self.getCommitList( page: self.page)
            self.getBranchList()
        }
    }
}
```

onAppear的时候，走接口。
```Swift
 func getCommitList(page: Int){
        if self.waitPlease { return }
        self.waitPlease = true
        if commitList.count == 0 {
            self.isLoading = true
        }
        var url = "repos/"
        if self.showCommitFrom == CommitFromModel.fromRepo{
            url = url + self.repoFullPath! + "/commits?page=" + String(page)
            url = url + "&sha=" + self.branch
        }else if self.showCommitFrom == CommitFromModel.fromPullRequest{
            url = url + (self.pullRequestItem?.prTo.repoNamespace.path)! + "/"
            url = url + (self.pullRequestItem?.prTo.repoPath)! + "/pulls/"
            url = url + String(self.pullRequestItem!.prId) + "/commits";
        }
        
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
                    var tempList = self.commitList
                    if page == 1{
                        tempList = []
                    }
                    for (_,subJson):(String, JSON) in json {
                        let author = subJson["commit"]["author"]["name"].stringValue
                        let userInfo = UserItemModel(id: Int(subJson["author"]["id"].intValue), userHead: String(subJson["author"]["avatar_url"].stringValue), userName: String(subJson["author"]["name"].stringValue), userAccount: String(subJson["author"]["login"].stringValue))
                        tempList.append(CommitModel(id: subJson["sha"].stringValue, sha: String(subJson["sha"].stringValue), author: author, commitTime: String(subJson["commit"]["author"]["date"].stringValue), message: String(subJson["commit"]["message"].stringValue), addCount:  String(subJson["stats"]["additions"].stringValue), deleteCount: String(subJson["stats"]["deletions"].stringValue), totalCount: String(subJson["stats"]["total"].stringValue), user: userInfo))
                    }
                    self.commitList = tempList
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

这里获取分支：
```Swift
func getBranchList(){
        var url = "repos/"
        url = url + self.repoFullPath! + "/branches";
        HttpRequest(url: url, withAccessToken: true)
            .doGet { (value) in
                let json = JSON(value)
                if json["message"].string != nil {
                    DispatchQueue.main.async {
                        UIAlertController.confirm(message: json["message"].stringValue, title: "发生错误", confirmText: "重新登录", cancelText: "返回") { (action) in
                            Helper.relogin()
                        }
                    }
                }else{
                    if page == 1{
                        branchList = []
                    }
                    for (_,subJson):(String, JSON) in json {
                        branchList.append(subJson["name"].stringValue)
                    }
                }
            } errorCallback: {
                
            }
    }
```

注意到分支列表里面用了分支的item：
```Swift
struct CommitItemView:View{
    @State var commitItem: CommitModel
    @State var repoFullPath: String
    @State var userHead:UIImage? = nil
    let placeholderImage = UIImage(named: "nohead")!
    @State var isCommitChangeShow: Bool = false
    
    var body: some View{
        VStack{
            VStack(alignment: .leading){
                VStack(alignment: .leading){
                    HStack(alignment: .top) {
                        Image(uiImage: self.userHead ?? placeholderImage)
                            .resizable()
                            .scaledToFit()
                            .frame(
                                width:20,height:20,
                                alignment: .center
                            )
                            .cornerRadius(5)
                            .onAppear(){
                                guard let url = URL(string: commitItem.user.userHead) else {
                                    return
                                }
                                URLSession.shared.dataTask(with: url) { (data, response, error) in
                                    if let data = data, let image = UIImage(data: data) {
                                        self.userHead = image
                                    }
                                }.resume()
                            }
                        VStack(alignment: .leading){
                            Text(commitItem.message)
                                .font(.system(size: 16))
                                .lineLimit(1)
                        }
                        Spacer()
                        Text("+" + commitItem.addCount).foregroundColor(.green).font(.system(size:14)).fontWeight(.bold).padding(.leading,0)
                        Text("-" + commitItem.deleteCount).foregroundColor(.red).font(.system(size:14)).fontWeight(.bold).padding(.leading,0)
                        
                    }
                }
                .padding(.vertical,5)
                HStack{
                    Text(Helper.getDateFromString(str: commitItem.commitTime)).font(.system(size:14)).foregroundColor(.gray)
                    Text("由 ").font(.system(size:14)).foregroundColor(.gray).padding(.leading,-5)
                    Text(commitItem.author + "(" + commitItem.user.userAccount + ")").font(.system(size:14)).foregroundColor(Color(hex: 0xaaaaaa)).padding(.leading,0).padding(.leading,-5)
                    Text(" 提交").font(.system(size:14)).foregroundColor(.gray).padding(.leading,0).padding(.leading,-5)
                    Spacer()
                }
            }
            .padding(10)
        }
        .onTapGesture {
            self.isCommitChangeShow = true
        }
        .sheet(isPresented: $isCommitChangeShow,onDismiss: {
            
        }){
            CommitChangesView(sha: self.commitItem.sha, repoFullPath:
                                self.repoFullPath)
        }
        .background(Color(red: 1, green: 1, blue: 1, opacity: 0.1))
        .cornerRadius(10)
        .padding(.horizontal,5)
        .padding(.bottom,-5)
    }
}
```

### 3.4 Pull Request

整个body定义：
```Swift
var body: some View {
    LoadingView(isLoading:self.$isLoading,message: self.$message,isModal:self.$isModal) {
        ZStack{
            if pullRequestList.count == 0 && !isLoading {
                VStack{
                    Image(systemName: "doc.text.magnifyingglass")
                        .scaleEffect(3, anchor: .center)
                    Text("暂无查询到的Pull Requests").padding(.top,30)
                }
            }
            RefreshView(refreshing: $isRefreshing, action: {
                self.page = 1
                self.getPullRequestList(page: self.page)
            }) {
                LazyVStack{
                    ForEach(self.pullRequestList){ item in
                        PullRequestItemView(pullRequestItem: item)
                            .onAppear(){
                                if !waitPlease && item.id == pullRequestList[pullRequestList.count - 1].id {
                                    self.page = self.page + 1
                                    self.getPullRequestList(page: self.page)
                                }
                            }
                    }
                }
            }
        }
    }
    .navigationBarTitle(Text("Pull Requests"), displayMode: .inline)
    .navigationBarItems(trailing:
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
                                self.getPullRequestList(page: self.page)
                            }){
                                PullRequestFilterView()
                                    .modifier(DisableModalDismiss(disabled: false))
                            }
    )
    .onAppear(){
        localConfig.setValue("open", forKey: giteeConfig.pull_request_state)
        localConfig.setValue("updated", forKey: giteeConfig.pull_request_sort)
        localConfig.setValue("desc", forKey: giteeConfig.pull_request_direction)
        self.page = 1
        self.getPullRequestList( page: self.page)
    }
}
```

可见时请求接口：
```Swift
func getPullRequestList(page: Int){
    if self.waitPlease { return }
    self.waitPlease = true
    if pullRequestList.count == 0 {
        self.isLoading = true
    }
    let state = localConfig.string(forKey: giteeConfig.pull_request_state)
    let sort = localConfig.string(forKey: giteeConfig.pull_request_sort)
    let direction = localConfig.string(forKey: giteeConfig.pull_request_direction)
    var url = "repos/"
    url = url + self.repoFullPath + "/pulls?page=" + String(page);
    
    url = url + "&state=" + state!
    url = url + "&sort=" + sort!
    url = url + "&direction=" + direction!
    
    
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
                var tempList = self.pullRequestList
                if page == 1{
                    tempList = []
                }
                for (_,subJson):(String, JSON) in json {
                    let userInfo = UserItemModel(id: Int(subJson["user"]["id"].intValue), userHead: String(subJson["user"]["avatar_url"].stringValue), userName: String(subJson["user"]["name"].stringValue), userAccount: String(subJson["user"]["login"].stringValue))
                    
                    let fromRepo = RepoModel(id: Int(subJson["head"]["repo"]["id"].intValue), repoName: String(subJson["head"]["repo"]["name"].stringValue), repoPath: String(subJson["head"]["repo"]["path"].stringValue), repoNamespace: RepoNamespace(id: Int(subJson["head"]["repo"]["namespace"]["id"].intValue), name: String(subJson["head"]["repo"]["namespace"]["name"].stringValue), path: String(subJson["head"]["repo"]["namespace"]["path"].stringValue)), repoDesc: String(subJson["head"]["repo"]["description"].stringValue), repoForks: "", repoStars: "",repoWatches:"", repoLicense:"", repoLanguage: "", repoPushDate:"", repoIsFork: false, repoIsOpenSource: Bool(subJson["head"]["repo"]["public"].boolValue), repoIssues:"", repoDefaultBranch:"")
                    
                    let toRepo = RepoModel(id: Int(subJson["base"]["repo"]["id"].intValue), repoName: String(subJson["base"]["repo"]["name"].stringValue), repoPath: String(subJson["base"]["repo"]["path"].stringValue), repoNamespace: RepoNamespace(id: Int(subJson["base"]["repo"]["namespace"]["id"].intValue), name: String(subJson["base"]["repo"]["namespace"]["name"].stringValue), path: String(subJson["base"]["repo"]["namespace"]["path"].stringValue)), repoDesc: String(subJson["base"]["repo"]["description"].stringValue), repoForks: "", repoStars: "",repoWatches:"", repoLicense:"", repoLanguage: "", repoPushDate:"", repoIsFork: false, repoIsOpenSource: Bool(subJson["base"]["repo"]["public"].boolValue), repoIssues:"", repoDefaultBranch:"")
                    
                    tempList.append(PullRequestModel(id: Int(subJson["id"].intValue), prId: Int(subJson["number"].intValue), prStatus: getPullRequestStatus(status: String(subJson["state"].stringValue)), prTitle: String(subJson["title"].stringValue), prBody: String(subJson["body"].stringValue), prUser: userInfo, prFrom: fromRepo, prTo: toRepo, prTime: Helper.getDateFromString(str: String(subJson["updated_at"].stringValue)), prAuthMerge: Bool(subJson["mergeable"].boolValue),prFromBranch: String(subJson["head"]["ref"].stringValue),prToBranch: String(subJson["base"]["ref"].stringValue)))
                }
                self.pullRequestList = tempList
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

然后还是pull Request的item展示了：
```Swift
struct PullRequestItemView:View{
    @State var pullRequestItem: PullRequestModel
    @State var prFromToString:String = ""
    @State var prFromToString2:String = ""
    @State private var isActiveCommit:Bool = false
    @State private var isActivePullRequest:Bool = false
    var body: some View{
        VStack{
            VStack(alignment: .leading){
                HStack(alignment: .top) {
                    VStack{
                        Text(getPullRequestStatusStringShow(pullRequestItem:pullRequestItem))
                            .foregroundColor(getPullRequestColor(pullRequestItem:pullRequestItem))
                            .padding(.vertical,1)
                            .padding(.horizontal,3)
                    }
                    .font(.system(size: 12))
                    .background(Color(red: 1, green: 1, blue: 1, opacity: 0.1))
                    .foregroundColor(.gray)
                    .cornerRadius(3)
                    VStack(alignment: .leading){
                        Text(self.pullRequestItem.prTitle)
                            .font(.system(size: 16))
                            .lineLimit(1)
                    }
                    .padding(.leading,0)
                    Spacer()
                    Text(self.pullRequestItem.prTime)
                        .padding(.vertical,1)
                        .padding(.horizontal,3)
                        .font(.system(size: 12))
                        .foregroundColor(.gray)
                }
                .padding(5)
                HStack(alignment: .center){
                    Text(self.pullRequestItem.prBody)
                        .font(.system(size: 14))
                        .foregroundColor(.gray)
                        .lineLimit(1)
                        .padding(.top,10)
                    Spacer()
                }
                HStack{
                    VStack(alignment: .leading){
                        Text(prFromToString)
                            .padding(.vertical,0)
                            .padding(.horizontal,3)
                            .font(.system(size: 12))
                            .foregroundColor(.gray)
                            .lineLimit(1)
                        
                        Text(prFromToString2)
                            .padding(.vertical,0)
                            .padding(.horizontal,3)
                            .font(.system(size: 12))
                            .foregroundColor(.gray)
                            .lineLimit(1)
                    }
                    Spacer()
                }
            }
            .padding(10)
        }
        .background(Color(red: 1, green: 1, blue: 1, opacity: 0.1))
        .cornerRadius(10)
        .padding(.horizontal,5)
        .sheet(isPresented: $isActivePullRequest,onDismiss: {
        }){
            PullRequestDetailView(pullRequestItem: $pullRequestItem).foregroundColor(.white)
        }
        .onTapGesture {
            self.isActivePullRequest = true
        }
        .contextMenu(ContextMenu {
            if pullRequestItem.prAutoMerge && pullRequestItem.prStatus == PullRequestStatus.open {
                Button(action: {
                }) {
                    HStack{
                        Image(systemName: "mail.and.text.magnifyingglass").scaleEffect(1, anchor: .center)
                        Spacer()
                        Text("审查通过")
                    }
                }
                Button(action: {
                }) {
                    HStack{
                        Image(systemName: "wrench.and.screwdriver").scaleEffect(1, anchor: .center)
                        Spacer()
                        Text("测试通过")
                    }
                }
                Button(action: {
                }) {
                    HStack{
                        Image(systemName: "shuffle").scaleEffect(1, anchor: .center)
                        Spacer()
                        Text("确认合并")
                    }
                }
            }
            Divider()
            Button(action: {
                self.isActiveCommit = true
            }) {
                HStack{
                    Image(systemName: "icloud.and.arrow.up").scaleEffect(1, anchor: .center)
                    Spacer()
                    Text("提交记录")
                }
            }
            .sheet(isPresented: $isActiveCommit,onDismiss: {
            }){
                CommitView(pullRequestItem: pullRequestItem).foregroundColor(.white)
            }
        })
        .onAppear(){
            if pullRequestItem.prFrom.repoNamespace.path == pullRequestItem.prTo.repoNamespace.path {
                self.prFromToString = "从 " + pullRequestItem.prFromBranch + "分支 到 " + pullRequestItem.prToBranch + "分支"
                self.prFromToString2 = "这是你的自己的合并请求"
            }else{
                self.prFromToString = "从 " + pullRequestItem.prFrom.repoNamespace.name + "/" + pullRequestItem.prFrom.repoName + ":" + pullRequestItem.prFromBranch
                self.prFromToString2 = "到 " + pullRequestItem.prTo.repoNamespace.name + "/" + pullRequestItem.prTo.repoName + ":" + pullRequestItem.prToBranch
            }
        }
    }
}
```

### 3.5 查看Issues

body定义：
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

可见时请求列表：
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

最后就是具体的IssueItemView了。
```Swift
struct IssueItemView:View{
    @State var issueItem: IssueModel
    @State private var isActiveIsssueDetail:Bool = false
    @State private var isActiveRepoDetail:Bool = false
    var body: some View{
        ZStack{
            VStack{
                NavigationLink(destination: RepoDetailView(repoFullPath: issueItem.repoInfo.repoNamespace.path + "/" + issueItem.repoInfo.repoPath), isActive: $isActiveRepoDetail) { EmptyView() }
            }
            .frame(width: 0, height: 0)
            .opacity(0)
            VStack{
                VStack(alignment: .leading){
                    HStack(alignment: .top) {
                        Image(systemName:getIssueIcon(status: issueItem.issueStatus))
                            .scaleEffect(1, anchor: .center)
                            .foregroundColor(getIssueColor(status: issueItem.issueStatus))
                        VStack(alignment: .leading){
                            Text(issueItem.issueTitle)
                                .font(.system(size: 16))
                                .lineLimit(1)
                        }
                        Spacer()
                        Text(issueItem.issueTime)
                            .padding(.vertical,1)
                            .font(.system(size: 12))
                            .foregroundColor(.gray)
                        
                    }
                    .padding(5)
                    HStack{
                        Text(issueItem.repoInfo.repoName)
                            .padding(.leading,35)
                            .font(.system(size: 12))
                            .foregroundColor(Color(hex: 0xCCCCCC))
                        Text(issueItem.issueDesc)
                            .font(.system(size: 12))
                            .foregroundColor(.gray)
                            .lineLimit(1)
                    }
                    HStack{
                        Image(systemName:"person.circle")
                            .scaleEffect(0.7, anchor: .center)
                            .foregroundColor(.gray)
                            .padding(.leading,30)
                        Text(issueItem.userInfo.userName)
                            .font(.system(size: 12))
                            .foregroundColor(.gray)
                            .padding(.leading,-8)
                        Spacer()
                    }
                    .padding(.top,5)
                }
                .padding(10)
            }
            .background(Color(red: 1, green: 1, blue: 1, opacity: 0.1))
            .cornerRadius(10)
            .padding(.horizontal,5)
            .padding(.bottom,-3)
            .sheet(isPresented: self.$isActiveIsssueDetail) {
                self.reloadIssue()
            } content: {
                IssuesDetailView(issueItem: $issueItem)
                    .foregroundColor(.white)
            }
            .onTapGesture {
                self.isActiveIsssueDetail = true
            }
            .contextMenu(ContextMenu {
                if issueItem.issueStatus != IssueStatus.open {
                    Button(action: {
                        self.changeIssueStatus(issueItem: issueItem,  issueStatus: IssueStatus.open)
                    }) {
                        HStack{
                            Image(systemName: "moon.circle").scaleEffect(1, anchor: .center)
                            Spacer()
                            Text("标记为已开启")
                        }
                    }
                }
                if issueItem.issueStatus != IssueStatus.progressing {
                    Button(action: {
                        self.changeIssueStatus(issueItem: issueItem,  issueStatus: IssueStatus.progressing)
                    }) {
                        HStack{
                            Image(systemName: "timer").scaleEffect(1, anchor: .center)
                            Spacer()
                            Text("标记为进行中")
                        }
                    }
                }
                if issueItem.issueStatus != IssueStatus.rejected {
                    Button(action: {
                        UIAlertController.alert(message: "请期待Gitee开放这个API吧~", title: "即将上线", confirmText: "安排")
                    }) {
                        HStack{
                            Image(systemName: "xmark.circle").scaleEffect(1, anchor: .center)
                            Spacer()
                            Text("标记为已拒绝")
                        }
                    }
                }
                if issueItem.issueStatus != IssueStatus.closed {
                    Button(action: {
                        self.changeIssueStatus(issueItem: issueItem, issueStatus: IssueStatus.closed)
                    }) {
                        HStack{
                            Image(systemName: "checkmark.circle").scaleEffect(1, anchor: .center)
                            Spacer()
                            Text("标记为完成")
                        }
                    }
                }
                Divider()
                Button(action: {
                    self.isActiveRepoDetail = true
                }) {
                    HStack{
                        Image(systemName: "archivebox.circle").scaleEffect(1, anchor: .center)
                        Spacer()
                        Text("进入所属仓库")
                    }
                }
            })
        }
    }
```

## 4 总结

* 这里学会了Swift初始化配置，可以简化其它流程，只需要继承App类。

* 然后这里学会使用类似Flutter的组件，HStack,VStack,ZSack，Spacer,Text，Image这一类基础组件。

* 网络请求的最好自己封装一个HttpRequest类，动态配置各种参数。






















