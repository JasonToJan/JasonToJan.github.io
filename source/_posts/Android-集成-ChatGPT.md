---
title: Android 集成 ChatGPT
date: 2023-02-10 08:41:21
top: false
cover: false
toc: true
mathjax: true
tags:
- Android Kotlin ChatGPT
categories:
- Android
---

## 1 效果

<img src=chatGPT.gif>

## 2 技术栈
Timber: 日志打印工具（[Github地址](https://github.com/JakeWharton/timber)）
Room：Jetpack包含的数据库（[最全面的Room数据库指南](https://juejin.cn/post/6844904200162246663)）
Splash Screens: 启动屏幕Android12支持（[Android 12 SplashScreen API快速入门](https://blog.csdn.net/guolin_blog/article/details/120275319)）
Coil：kotlin图片加载框架（[Github地址](https://github.com/coil-kt/coil)）
Coroutine：kotlin协程([官方文档](https://kotlinlang.org/docs/coroutines-overview.html#tutorials))
Retrofit：网络请求（[Github地址](https://github.com/square/retrofit)）

## 3 要点

### 3.1 Application初始化
```Kotlin
class App : Application() {
    
    init {
        instance = this
    }
    companion object {
        private var instance : App? = null
        fun context() : Context {
            return instance!!.applicationContext
        }
    }
    
    override fun onCreate() {
        super.onCreate()
        Timber.plant(Timber.DebugTree())
    }
}
```
这里初始化了Timer
> "Timber.plant(Timber.DebugTree())" 这个语句是 Android 中的一个日志框架 Timber 的语句，用于将 Timber 与一个 DebugTree 实例关联起来。
Timber 是一个 Android 日志框架，用于简化 Android 中的日志记录。它提供了一种更方便的方法来记录日志，而不需要繁琐的配置。
DebugTree 是 Timber 提供的一个简单的实现，它将日志信息打印到控制台，以便在开发期间进行调试。
通过调用 Timber.plant(Timber.DebugTree())，可以将 DebugTree 实例安装到 Timber 日志框架中，以便将日志记录到控制台。

### 3.2 启动页配置
```xml
<activity
        android:name=".view.MainActivity"
        android:theme="@style/Theme.Test.Splash"
        android:windowSoftInputMode="adjustResize"
        android:exported="true">
        <intent-filter>
            <action android:name="android.intent.action.MAIN" />

            <category android:name="android.intent.category.LAUNCHER" />
        </intent-filter>

        <meta-data
            android:name="android.app.lib_name"
            android:value="" />
    </activity>
```

关联的主体为：
```
<style name="Theme.Test.Splash" parent="Theme.SplashScreen.IconBackground">
    <item name="windowSplashScreenAnimatedIcon">@drawable/splash_layout</item>
    <item name="windowSplashScreenBackground">@color/black</item>
    <!-- splash delay -->
    <item name="windowSplashScreenAnimationDuration">2000</item>
    <!-- splash icon back color
    <item name="windowSplashScreenIconBackgroundColor">@color/white</item> -->
    <item name="postSplashScreenTheme">@style/Theme.Test</item>
</style>
```
这里的Theme.SplashScreen.IconBackground主题需要引入这个三方库：
```
implementation 'androidx.core:core-splashscreen:1.0.0'
```
然后还需要在首页onCreate最前面，执行一下：
```
installSplashScreen得以支持
```

这里直接跳转MainActivity。

### 3.3 首页

数据定义：
```Kotlin
private lateinit var binding : ActivityMainBinding
private val viewModel : MainViewModel by viewModels()
private var contentDataList = ArrayList<ContentEntity>()
```

ActivityMainBinding绑定的视图如下：
<img src=chat_01.png>

然后先走启动页，再设置布局：
```
installSplashScreen()

super.onCreate(savedInstanceState)
binding = ActivityMainBinding.inflate(layoutInflater)
setContentView(binding.root)
```

这里再配置下Coil的图片加载器：
```
val imageLoader = this?.let {
    ImageLoader.Builder(it)
        .components {
            if (SDK_INT >= 28) {
                add(ImageDecoderDecoder.Factory())
            } else {
                add(GifDecoder.Factory())
            }
        }
        .build()
}
if (imageLoader != null) {
    Coil.setImageLoader(imageLoader)
}

binding.loading.visibility = View.INVISIBLE
binding.loading.load(R.drawable.loading3)
```
这里用了一个扩展ImageView的load函数。

### 3.4 配置LiveData监听

```Kotlin
viewModel.contentList.observe(this, Observer {
    contentDataList.clear()
    for (entity in it) {
        contentDataList.add(entity)
    }
    setContentListRV()
})
```
上面是监听内容列表变更情况。如果有变化，就更新下contentDataList的全局变量。
看下setContentListRV()
```kotlin
private fun setContentListRV() {
    val contentAdapter = ContentAdapter(this, contentDataList)
    binding.RVContainer.adapter = contentAdapter
    binding.RVContainer.layoutManager = LinearLayoutManager(this).apply {
        stackFromEnd = true
    }

    viewModel.launch({
        delay(100)
        binding.SVContainer.fullScroll(ScrollView.FOCUS_DOWN)
    })

    contentAdapter.delChatLayoutClick = object : ContentAdapter.DelChatLayoutClick {
        override fun onLongClick(view : View, position: Int) {
            Timber.tag("TEST##").e("${contentDataList[position].id}")
            val builder = AlertDialog.Builder(this@MainActivity)
            builder.setTitle("清空")
                .setMessage("清空此条消息")
                .setPositiveButton("确定",
                    DialogInterface.OnClickListener { dialog, id ->
                        viewModel.deleteSelectedContent(contentDataList[position].id)
                    })
                .setNegativeButton("取消",
                    DialogInterface.OnClickListener { dialog, id ->
                    })
            builder.show()
        }
    }
}
```
这里每次都new了一个Adapter，然后走一个协程，滑动最底部。
同时也配置下长按点击事件，这里弹一个弹框，提示清空消息，如果要清空，就再走一次viewModel对数据进行删除。

第二个LiveData监听是：
```Kotlin
 viewModel.deleteCheck.observe(this, Observer {
    if (it == true) {
        viewModel.getContentData()
    }
})
```
这里监听删除是否成功，如果成功，就重新获取一次数据。

第三个LiveData监听是：
```Kotlin
 viewModel.gptInsertCheck.observe(this, Observer {
    if (it == true) {
        viewModel.getContentData()
        binding.loading.visibility = View.INVISIBLE
    }
})
```
插入数据监听，当用户发出消息成功后，需要插入数据库，插入后，这里更新列表，同时需要关闭加载动画。

### 3.5 其它点击事件
```Kotlin
 binding.sendBtn.setOnClickListener {
    binding.loading.visibility = View.VISIBLE

    if (binding.EDView.text.toString().isEmpty()) {
        return@setOnClickListener
    }

    viewModel.postResponse(binding.EDView.text.toString())
    viewModel.insertContent(binding.EDView.text.toString(), 2) // 1: Gpt, 2: User
    binding.EDView.setText("")
    viewModel.getContentData()
}
```
发送消息事件，这里先展示loading状态，然后通过viewModel层走接口发送数据。
同时需要插入用户发送数据到本地数据库。
然后理解刷新数据。

```Kotlin
 binding.backBtn.setOnClickListener {
    finish()
}
```
这里返回退出App。

## 3.6 消息Adapter
```Kotlin
class ContentAdapter(val context : Context, private val dataSet : List<ContentEntity>) : RecyclerView.Adapter<ContentAdapter.ViewHolder>() {

    companion object {
        private const val Gpt = 1
        private const val User = 2
    }

    interface DelChatLayoutClick {
        fun onLongClick(view : View, position: Int)
    }
    var delChatLayoutClick : DelChatLayoutClick? = null

    inner class ViewHolder(view : View) : RecyclerView.ViewHolder(view) {
        val contentTV : TextView = view.findViewById(R.id.rvItemTV)
        val delChatLayout : ConstraintLayout = view.findViewById(R.id.chatLayout)
        val idHolder : TextView = view.findViewById(R.id.holdingId)
    }

    override fun onCreateViewHolder(parent: ViewGroup, viewType: Int): ViewHolder {
        if (viewType == Gpt) {
            val view = LayoutInflater.from(parent.context).inflate(R.layout.gpt_content_item, parent, false)
            return ViewHolder(view)
        } else {
            val view = LayoutInflater.from(parent.context).inflate(R.layout.user_content_item, parent, false)
            return ViewHolder(view)
        }
    }

    override fun onBindViewHolder(holder: ViewHolder, position: Int) {
        holder.contentTV.text = dataSet[position].content
        holder.idHolder.text = dataSet[position].id.toString()

        holder.delChatLayout.setOnLongClickListener { view ->
            delChatLayoutClick?.onLongClick(view, position)
            return@setOnLongClickListener true
        }

    }

    override fun getItemCount(): Int {
        return dataSet.size
    }

    override fun getItemViewType(position: Int): Int {
        return if (dataSet[position].gptOrUser == 1) { // Gpt
            Gpt
        } else {
            User
        }
    }

}
```
这里分两种Type，一种是GPT，一种是用户，展示不同item。

item数据：
```Kotlin
@Entity(tableName = "ContentTable")
data class ContentEntity(

    @PrimaryKey(autoGenerate = true)
    @ColumnInfo(name = "id")
    var id : Int,
    @ColumnInfo(name = "Content")
    var content : String,
    @ColumnInfo(name = "gptOrUser")
    var gptOrUser : Int

)
```
这里是model层了，内容实体，有个int表示是人还是人工智能。

### 3.7 ViewModel层
```Kotlin
class MainViewModel : ViewModel() {

    private val databaseRepository = DatabaseRepository()
    private val netWorkRepository = NetWorkRepository()

    private var _contentList = MutableLiveData<List<ContentEntity>>()
    val contentList : LiveData<List<ContentEntity>>
        get() = _contentList

    private var _deleteCheck = MutableLiveData<Boolean>(false)
    val deleteCheck : LiveData<Boolean>
        get() = _deleteCheck

    private var _gptInsertCheck = MutableLiveData<Boolean>(false)
    val gptInsertCheck : LiveData<Boolean>
        get() = _gptInsertCheck
```
这里继承ViewModel，然后建立2个仓库。
* DatabaseRepository 数据仓库层
* NetworkRepository 网络仓库层

然后有3个LiveData监听内容，删除，插入数据的监听。

协程封装：
```Kotlin

/**
 * ViewModel扩展方法：启动协程
 * @param block 协程逻辑
 * @param onError 错误回调方法
 * @param onComplete 完成回调方法
 */
fun ViewModel.launch(
    block: suspend CoroutineScope.() -> Unit,
    onError: (e: Throwable) -> Unit = { _: Throwable -> },
    onComplete: () -> Unit = {}
) {
    viewModelScope.launch(
        CoroutineExceptionHandler { _, throwable ->
            run {
                if (throwable is Exception) {
                    onError(NetExceptionFilter.onFilter(throwable))
                } else {
                    onError(throwable)
                }
            }
        }
    ) {
        try {
            block.invoke(this)
        } finally {
            onComplete()
        }
    }
}
```
对ViewModel做了扩展，可以直接发起协程，并且内部捕获异常。
有个工具类可以对异常再继续处理：
```Kotlin
object NetExceptionFilter {
    @JvmStatic
    fun onFilter(e: Exception): Exception {
        e.printStackTrace()
        if (e is HttpException) {
            return e
        }
        if (e is SocketTimeoutException) {
            return Exception("已超时")
        }
        if (e is UnknownHostException || e is SocketException) {
            return Exception("socket异常")
        }
        return if (e is IOException) {
            return Exception("io异常")
        } else Exception("其它异常")
    }
}
```

ok。然后具体看下viewModel层中的耗时任务。

第一个最重要的当然是发起调用GPT接口了：
```Kotlin
fun postResponse(query : String) = launch({
    val jsonObject: JsonObject? = JsonObject().apply{
        // params
        addProperty("model", "text-davinci-003")
        addProperty("prompt", query)
        addProperty("temperature", 0)
        addProperty("max_tokens", 500)
        addProperty("top_p", 1)
        addProperty("frequency_penalty", 0.0)
        addProperty("presence_penalty", 0.0)
    }
    val response = netWorkRepository.postResponse(jsonObject!!)
    
    Timber.tag("TEST##").e("${response.choices.get(0)}")
    val gson = Gson()
    val tempjson = gson.toJson(response.choices.get(0))
    val tempgson = gson.fromJson(tempjson, GptText::class.java)
    Timber.tag("TEST##").e("${tempgson.text}")
    insertContent(tempgson.text.toString(), 1)
    
}, {
    Toast.makeText(App.context(), "${it.message}", Toast.LENGTH_SHORT).show()
    _gptInsertCheck.postValue(true)
})
```
这里配置GPT接口需要的参数定义，通过netWorkRepository发起请求，定义了接收数据的格式转换，然后拿到数据后，转换为GptText，并插入本地数据库。

有异常时，这里弹出一个toast。

第二个获取数据方法：
```Kotlin
fun getContentData() = viewModelScope.launch(Dispatchers.IO) {
    _contentList.postValue(databaseRepository.getContentData())
    _deleteCheck.postValue(false)
    _gptInsertCheck.postValue(false)
}
```
这里通过向IO线程，调用databaseRepository层的获取数据库的方法，拿到最新数据。

第三个插入数据：
```Kotlin
fun insertContent(content : String, gptOrUser : Int) = viewModelScope.launch(Dispatchers.IO) {
    databaseRepository.insertContent(content, gptOrUser)
    if (gptOrUser == 1) {
        _gptInsertCheck.postValue(true)
    }
}
```
这里也是调用了数据库层，插入数据。

第四个是删除数据：
```Kotlin
fun deleteSelectedContent(id : Int) = viewModelScope.launch(Dispatchers.IO) {
    databaseRepository.deleteSelectedContent(id)
    _deleteCheck.postValue(true)
}
```
同样走数据库层删除数据。

### 3.8 Model层的数据库层
先看下数据库层：
```Kotlin
class DatabaseRepository {

    private val context = App.context()
    private val database = ChatDatabase.getDatabase(context)

    fun getContentData() = database.contentDAO().getContentData()

    fun insertContent(content : String, gptOrUser : Int) = database.contentDAO().insertContent(ContentEntity(0, content, gptOrUser))

    fun deleteSelectedContent(id : Int) = database.contentDAO().deleteSelectedContent(id)

}
```
这是一个仓库类，声明了几个操作数据库的方法。

实体类声明：
```Kotlin
@Entity(tableName = "ContentTable")
data class ContentEntity(

    @PrimaryKey(autoGenerate = true)
    @ColumnInfo(name = "id")
    var id : Int,
    @ColumnInfo(name = "Content")
    var content : String,
    @ColumnInfo(name = "gptOrUser")
    var gptOrUser : Int
)
```
这里id为唯一索引。表名叫做ContentTable。

DAO接口声明：
```Kotlin
@Dao
interface ContentDAO {

    @Query("SELECT * FROM ContentTable")
    fun getContentData() : List<ContentEntity>

    @Insert(onConflict = OnConflictStrategy.IGNORE)
    fun insertContent(content : ContentEntity)

    @Query("DELETE FROM ContentTable WHERE id = :id")
    fun deleteSelectedContent(id : Int)

}
```
这里是操作数据库的底层方法。

数据库声明：
```Kotlin
@Database(entities = [ContentEntity::class], version = 2)
abstract class ChatDatabase : RoomDatabase() {

    abstract fun contentDAO() : ContentDAO

    companion object {
        @Volatile
        private var INSTANCE : ChatDatabase? = null

        fun getDatabase(
            context : Context
        ) : ChatDatabase {
            return INSTANCE ?: synchronized(this) {
                val instance = Room.databaseBuilder(
                    context.applicationContext,
                    ChatDatabase::class.java,
                    "chatDatabase"
                )
                    .fallbackToDestructiveMigration()
                    .build()
                INSTANCE = instance
                instance
            }
        }
    }
}
```
这里就是初始化Room数据库了。

### 3.9 Model层的网络层
首先声明Api接口：
```Kotlin
interface Apis {
    @Headers(
        "Content-Type:application/json",
        "Authorization:Bearer 自己的Key")
    @POST("v1/completions")
    suspend fun postRequest(
        @Body json : JsonObject
    ) : GptResponse

}
```
用了suspend，表示协程调用。
body发送的是Json对象，返回一个GptResponse对象。
这个是一个Json数组：
```Kotlin
import com.google.gson.JsonArray

data class GptResponse (
    val choices : JsonArray
)
```

然后是Retrofit初始化：
```Kotlin
object RetrofitInstance {
    private var okHttpClient = OkHttpClient
        .Builder()
        .connectTimeout(1, TimeUnit.MINUTES)
        .readTimeout(1, TimeUnit.MINUTES)
        .writeTimeout(1, TimeUnit.MINUTES)
        .build()

    private const val BASE_URL = "https://api.openai.com/"

    private val client = Retrofit
        .Builder()
        .baseUrl(BASE_URL)
        .client(okHttpClient)
        .addConverterFactory(GsonConverterFactory.create())
        .build()

    fun getInstance() : Retrofit {
        return client
    }
}
```
new了一个OkHttpClient对象，同时将他作为参数传到Retrofit里面。

然后就是声明一个网络仓库来调用接口：
```Kotlin
class NetWorkRepository {

    private val chatGPTClient = RetrofitInstance.getInstance().create(Apis::class.java)

    suspend fun postResponse(jsonData : JsonObject) = chatGPTClient.postRequest(jsonData)
}
```

## 4 总结

* 总体来说，集成GPT很简单，本质上就是联调一个接口，使用Retrofit通过传参即可拿到返回数据。
* 本篇文章主要是简单描述一个MVVM如何构造，主要是Activity和Adapter对应View层，自定义ViewModel层对应VM层，网络层和数据库层和实体 对应Model层。
* 其次，学会Kotlin协程用法，这里ViewModel中可以使用viewModelScope 来调用launch函数，实现协程，最好是要捕获一下异常，保证多种可能性。
* 另外，要了解Room数据库使用，通过执行协程，再调用数据库，拿到数据后再post发送给View层。

