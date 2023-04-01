---
title: Android wear os 使用入门
date: 2023-02-11 08:41:21
top: false
cover: false
toc: true
mathjax: true
tags:
- Android wear
categories:
- Android
---

## 1 Wear OS 开发原则

### 1.1 针对关键任务进行设计
重点关注目标用户的一项或两项需求，而不是完整应用体验。不要迁移整个移动代码库，也不要将 Wear OS 界面放在顶层。

相反，您应寻找适合腕部佩戴的关键任务，并简化 Wear OS 的体验。

### 1.2 针对腕部佩戴进行优化
让用户可在几秒钟内完成手表上的任务，避免人体工学不适或手臂疲劳。

### 1.3 为任务使用适当的 surface
为了吸引用户，Wear OS 拥有比移动设备更多的 surface。应用应针对这些 surface 定制内容。

每个 surface 都有自己的使用场景。如果需要执行更多操作，请将用户引导至更完整的应用体验。

阅读并了解如何根据用户需求的优先级在各个 surface 上调整您的内容。以下是一个天气应用的优先级示例。

### 1.4 向其他 surface 添加通知
在 Wear OS API 级别 30 及更高版本中，将任何持续显示的通知与 OngoingActivity 配对，即可将该通知添加到 Wear OS 界面的其他 surface 中，以增强与长时间运行的 activity 的交互。

### 1.5 支持离线场景
虽然 Wear OS 设备通常支持蓝牙和 Wi-Fi，但可能不支持 LTE。针对不稳定的连接和离线使用场景（例如锻炼和通勤）进行设计，在这些场景中用户可能会将移动设备留在家中。

### 1.6 提供相关的内容
用户几乎总会佩戴手表。根据用户所处的环境（例如，时间、地点和活动）更新应用内容。

### 1.7 帮助用户通过其他设备完成任务
越来越多的用户拥有多台设备。手表可以帮助用户在分布式设备生态系统中完成任务。查看适合您的应用的使用场景。

### 1.8 改善应用冷启动期间的用户体验
为了改善应用冷启动期间的用户体验，请使用单独的主题创建启动 activity，并在清单文件中将其 windowBackground 设置为自定义启动可绘制对象。启动画面由层列表组成，层列表中包含两个元素：背景颜色和自定义可绘制对象（通常为应用图标）。 可绘制对象应为 48 x 48dp。

### 1.9 媒体应用的注意事项
启用手机中的音乐播放控件
如果您的应用同时安装在手机和手表上，用户希望能通过手表进行远程控制。例如，用户希望能够通过手表暂停、播放或跳过手机上的歌曲。

已下载的内容
如前所述，支持离线场景非常重要。这对于媒体应用尤为重要。对于媒体应用而言，先支持离线下载，然后根据需要添加在线播放功能，这样会更方便。

在设计时，请向用户明确说明哪些内容可离线使用。对于任何长时间运行的即时或定期任务，请使用 WorkManager。推迟下载，直到手表正在充电并连接到 Wi-Fi。

通过 LTE 在线播放
请考虑在具有 LTE 连接（媒体播放的常见使用场景）的设备上提供在线播放支持。借助在线播放功能，用户将其他设备留在家中时，仍能听音乐。当用户在线播放音乐和缓存在线播放的音频时，确保能够向用户直观地传达这些讯息。避免利用 LTE 执行任何可推迟的作业（例如发送日志记录和分析数据），以便在在线播放时优化耗电量。

支持蓝牙耳机
用户在外出跑步或散步可能会只带着手表和耳机。您可以通过支持与耳机配对，使用户获得真正的独立体验。如果在播放或恢复播放音乐时未连接耳机，请启动蓝牙设置，以便用户直接从应用连接到蓝牙耳机。

指明音乐来源
清楚地指明声音是来自手表还是手机。您可以使用来源图标来指示播放音乐的设备。默认来源应是用户开始播放音乐的设备。

使用扬声器
有些 Wear OS 设备配有内置扬声器，可用于提醒和闹钟等功能。避免使用内置扬声器播放媒体内容和音乐，因为用户希望通过耳机播放这些内容。如需了解详情，请参阅检测音频设备。

## 2 Wear OS 应用开发与移动应用开发的差异
Wear OS 以 Android 为基础，并且专门针对腕部佩戴进行了优化。如果您有 Android 应用开发经验，那么您可能使用过同时适用于这两类开发的许多功能和 API。不过，设计移动应用的方式与设计 Wear OS 应用的方式有一些区别。

功能、API 或最佳实践	Wear OS 应用与移动应用	Wear OS 详细信息
设计应用的界面或用户体验	不同	专注于几秒钟内就能完成的少数关键任务。
界面 surface	不同	不止包含各种活动和通知，还提供许多独特的 surface，包括应用、功能块、复杂功能、表盘主题等。
界面组件	移动应用等	包括移动设备组件和 Wear OS 专用组件，包括：BoxInsetLayout、SwipeDismissFrameLayout、WearableRecyclerView 等。
持续性活动	不同	将持续性通知添加到新的 Wear OS 应用 surface 中。
深色主题或模式	不同	仅提供深色模式以节省电量。
返回堆栈	不同	允许用户通过滑动关闭返回堆栈或在堆栈中上移。
实体按钮	不同	穿戴式设备通常包含一个或多个实体按钮。所有 Wear OS 设备都至少有一个按钮，即电源按钮。除此之外，手表上可能没有可在您的应用中使用的多功能按钮，也可能有多个此类按钮。
旋转输入	不同	某些 Wear OS 设备包含实体侧面旋钮或旋转输入钮。用户可以旋转此类按钮来向上或向下滚动您的应用的当前视图。
恢复应用运行	不同	允许用户点按两次辅助硬件按钮来使最近使用的应用恢复运行。当用户重新进入您的应用时，您的应用必须记住用户的滚动位置。
架构组件	相同	请参阅 Android 文档中的 Android 架构组件。
导航	不同	应用应采用浅层设计（避免超过两层）和线性设计（内联显示大部分内容和导航组件）。
与其他应用交互	相同	请参阅与其他应用交互。
与已配对设备交互	新增	可以通过 Wear 应用与已配对设备进行交互。如需了解详情，请参阅发送和同步数据。
intent 和 intent 过滤器	相同	请参阅 Android 文档中的 intent 和 intent 过滤器。
动画和过渡	相同	请参阅 Android 文档中的动画和过渡。
图片和图形	相同	请参阅 Android 文档中的图片和图形。
服务和后台任务	相同	请参阅 Android 文档中的服务概览。
后台任务	相同	请参阅 Android 文档中的后台工作概览。
权限	相同	请参阅 Android 文档中的 Android 中的权限。
应用数据和文件	相同	请参阅 Android 文档中的应用数据和文件。
用户数据和身份	类似	除了相关移动 API 之外，还有其他身份验证选项。详细了解在穿戴式设备上进行身份验证。
用户位置	相同	FusedLocationProvider 还利用手机的 GPS 芯片节省电池电量，并在 Wear OS 上检测位置。
轻触和输入	移动应用等	除标准触控输入外，还提供更多输入方式。如需了解详情，请参阅轻触和输入概览。
传感器	移动应用等	Wear OS 应用的开发与移动应用开发类似。如需简化 Wear OS 中的应用，请考虑使用能够为您处理这方面工作的 Health Services API。了解传感器。
健康服务	新增	提供由传感器、内容感知算法和全天健康状况监测功能生成的健身和健康数据。如需了解详情，请参阅 Wear OS 上的健康服务。
连接性	类似	大多数移动 API 均完全受支持，但也存在一些限制。例如，android.webkit API 不受支持。如需了解详情，请参阅 Wear OS 上的网络访问和同步。
Android App Bundle	相同	请参阅 Android 文档中的 Android App Bundle 简介。
依赖项注入	相同	请参阅 Android 文档中的 Android 中的依赖项注入。
测试	类似	请参阅 Android 文档中的在 Android 平台上测试应用。
性能	类似	请参阅 Android 文档中的应用性能指南。
无障碍功能	相同	请参阅 Android 文档中的打造无障碍应用。
隐私	相同	请参阅 Android 文档中的隐私设置最佳实践。
安全性	相同	请参阅 Android 文档中的应用安全性最佳实践。

## 3 Wear OS界面
Wear OS 让用户能够轻松与针对手表进行了优化的应用进行互动。确保内容显示在适当的 surface 上。

Wear OS 上的应用 surface 在设计时充分考虑了作业。例如，如果您有一个信息单元，并且用户可能希望每天查看多次该单元，您可以考虑提供一项复杂功能。如果您的内容具有非常高的价值，并且与情境高度相关，您可以考虑使用通知。

在 Wear OS 上直观地设计应用内容的另一种实用方式是，考虑信息在各个 surface 中的优先级，并将最有价值的内容提升到 Wear OS 的一目了然的 surface 中。

在复杂功能和通知中显示优先级最高的内容，并使用功能块和应用上的较大空间来适当地显示更多内容。

以下几个部分将更加详细地介绍以上各个 surface。

## 4 应用
将应用定义为 Wear OS 应用 
您必须在应用的 Android 清单文件中定义 <uses-feature> 标记。如要表明这是手表应用，请添加如下所示的条目：
```
  <manifest>
  ...
  <uses-feature android:name="android.hardware.type.watch" />
  ...
  </manifest>
  
```

### 4.1 请求权限
当 Wear 应用请求手机权限（例如，穿戴式应用想要访问其移动应用上的照片或其他敏感数据）时，Wear 应用必须提示用户在手机上授予此权限。在手机上，手机应用可以使用一个 activity 向用户提供更多信息。在 activity 中，添加两个按钮：一个用于授予权限，一个用于拒绝授予权限。

### 4.2 音频播放

检测音频设备：
Wear OS 应用首先必须检测穿戴式设备是否具有合适的音频输出方式。对开发者来说，穿戴式设备可能会具有以下音频输出方式中的一种或两种：

AudioDeviceInfo.TYPE_BUILTIN_SPEAKER：在具有内置扬声器的设备上
AudioDeviceInfo.TYPE_BLUETOOTH_A2DP：如果蓝牙耳机已配对并连接
```
AudioManager audioManager = (AudioManager) context.getSystemService(Context.AUDIO_SERVICE);

fun audioOutputAvailable(type: Int): Boolean {
    if (!packageManager.hasSystemFeature(PackageManager.FEATURE_AUDIO_OUTPUT)) {
        return false
    }
    return audioManager.getDevices(AudioManager.GET_DEVICES_OUTPUTS).any { it.type == type }
}

audioOutputAvailable(AudioDeviceInfo.TYPE_BUILTIN_SPEAKER) // True if the device has a speaker
audioOutputAvailable(AudioDeviceInfo.TYPE_BLUETOOTH_A2DP) // True if a Bluetooth headset is connected
```

播放音频：
检测到合适的音频输出方式后，在 Wear OS 上播放音频的过程和在移动设备或其他设备上播放音频相同。如需了解详情，请参阅 MediaPlayer 概览。如果想更轻松地使用更为高级的功能（例如在线播放和下载媒体内容），不妨使用 ExoPlayer。请务必遵循针对音频应用的最佳实践，例如管理音频焦点。

## 5 视图构建
Android Jetpack 包含 Wear OS 界面库。Wear OS 界面库包含以下类：

CurvedTextView：此组件可便于沿着视图中可内切的最大圆圈的弧线轻松显示文字。
DismissibleFrameLayout：此布局让用户可通过点按返回按钮或在屏幕从左向右滑动来关闭任何视图。Wear OS 用户希望通过从左向右滑动屏幕来实现返回操作。
WearableRecyclerView：此视图提供通过 WearableLinearLayoutManager 更新子布局的基本偏移逻辑。
AmbientModeSupport：与 AmbientModeSupport.AmbientCallbackProvider 接口一起使用的类，共同为氛围模式提供支持。

```
<androidx.wear.widget.DismissibleFrameLayout
    xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:id="@+id/swipe_dismiss_root" >

    <TextView
        android:id="@+id/test_content"
        android:layout_width="match_parent"
        android:layout_height="match_parent"
        android:gravity="center"
        android:text="Swipe the screen to dismiss me." />
</androidx.wear.widget.DismissibleFrameLayout>
```

### 5.1 处理不同的手表形状

使用 BoxInsetLayout可以适配方形和圆形的屏幕。
```
<androidx.wear.widget.BoxInsetLayout
    xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    android:layout_height="match_parent"
    android:layout_width="match_parent"
    android:padding="15dp">

    <androidx.constraintlayout.widget.ConstraintLayout
        android:layout_width="match_parent"
        android:layout_height="match_parent"
        android:padding="5dp"
        app:layout_boxedEdges="all">

        <TextView
            android:layout_height="wrap_content"
            android:layout_width="match_parent"
            android:text="@string/sometext"
            android:textAlignment="center"
            app:layout_constraintEnd_toEndOf="parent"
            app:layout_constraintStart_toStartOf="parent"
            app:layout_constraintTop_toTopOf="parent" />

        <ImageButton
            android:background="@android:color/transparent"
            android:layout_height="50dp"
            android:layout_width="50dp"
            app:layout_constraintBottom_toBottomOf="parent"
            app:layout_constraintStart_toStartOf="parent"
            android:src="@drawable/cancel" />

        <ImageButton
            android:background="@android:color/transparent"
            android:layout_height="50dp"
            android:layout_width="50dp"
            app:layout_constraintBottom_toBottomOf="parent"
            app:layout_constraintEnd_toEndOf="parent"
            android:src="@drawable/ok" />

    </androidx.constraintlayout.widget.ConstraintLayout>

</androidx.wear.widget.BoxInsetLayout>
```
请注意该布局中以粗体标记的部分：

android:padding="15dp"

此行用于为 <BoxInsetLayout> 元素指定内边距。

android:padding="5dp"

此行用于为内部 ConstraintLayout 元素指定内边距。

app:layout_boxedEdges="all"

此行可确保 ConstraintLayout 元素及其子项位于圆形屏幕上的窗口边衬区所定义的区域内。此行对方形屏幕没有影响。

### 5.2 使用曲线布局
您可以通过 Wear OS 界面库中的 WearableRecyclerView 类选择使用针对圆形屏幕进行了优化的曲线布局。如需为应用中的可滚动列表启用曲线布局。

### 5.3 对方形屏幕和圆形屏幕使用不同的布局 
同时支持方形屏幕和圆形屏幕的另一种方法是为不同的屏幕形状提供备用资源。对于布局、尺寸或其他资源类型，将资源限定符设置为 round 或 notround。

例如，考虑按如下方式组织布局：

如需同时适用于圆形和方形手表的布局，请使用 layout/ 目录。
如需针对特定屏幕形状的专用布局，请使用 layout-round/ 和 layout-notround/ 目录。

您还可以使用 res/values、res/values-round 和 res/values-notround 资源目录。以这种方式组织资源，您可以共用一个布局，仅需根据设备类型更改特定属性即可。

使用 values/dimens.xml 和 values-round/dimens.xml 可为圆形手表和方形手表构建布局。通过指定不同的内边距设置，您可以使用单个 layout.xml 文件和两个 dimens.xml 文件创建以下布局：

### 5.4 使用 XML 填补下巴
某些手表的圆形屏幕上有边衬区（也称为“下巴”）。如果不填补下巴，部分设计可能会被它遮挡住。

```
<FrameLayout
  ...
  <androidx.wear.widget.RoundedDrawable
    android:id="@+id/androidbtn"
    android:src="@drawable/ic_android"
    .../>
   <ImageButton
    android:id="@+id/lovebtn"
    android:src="@drawable/ic_favorite"
    android:paddingTop="5dp"
    android:paddingBottom="5dp"
    android:layout_gravity="bottom"
    .../>
</FrameLayout>
```
您可以使用 fitsSystemWindows 属性设置内边距来避开下巴。以下 activity_main.xml 代码段展示了 fitsSystemWindows 的用法：
```
<ImageButton
  android:id="@+id/lovebtn"
  android:src="@drawable/ic_favorite"
  android:paddingTop="5dp"
  android:paddingBottom="5dp"
  android:fitsSystemWindows="true"
  .../>
```

### 5.5 以编程方式管理下巴
如果您需要获得比使用 XML 以声明方式定义布局时更多的控制权限，可以通过编程方式调整布局。如需获取下巴的尺寸，请将 View.OnApplyWindowInsetsListener 附加到布局的最外层视图。
```
private var chinSize: Int = 0

override fun onCreate(savedInstanceState: Bundle?) {
    super.onCreate(savedInstanceState)
    setContentView(R.layout.activity_main)
    // Find the outermost element
    findViewById<View>(R.id.outer_container).apply {
        // Attach a View.OnApplyWindowInsetsListener
        setOnApplyWindowInsetsListener { v, insets ->
            chinSize = insets.systemWindowInsetBottom
            // The following line is important for inner elements that react to insets
            v.onApplyWindowInsets(insets)
            insets
        }
    }
}
```

### 5.6 创建列表
在 Wear OS 设备上，用户可通过列表轻松地从一组选项中选择一个项目。

穿戴式设备界面库包含 WearableRecyclerView 类，它是 RecyclerView 的实现，用于创建针对穿戴式设备进行了优化的列表。您可以通过创建一个新的 WearableRecyclerView 容器，在穿戴式应用中使用此界面。

对于简单项目的长列表（例如应用启动器或联系人列表），请使用 WearableRecyclerView。每个项目可能具有一个简短的字符串和一个关联的图标。或者，每个项目也可能只有一个字符串或一个图标。
```
<androidx.wear.widget.WearableRecyclerView
    xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:tools="http://schemas.android.com/tools"
    android:id="@+id/recycler_launcher_view"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:scrollbars="vertical" />
```
创建曲线布局

如需为穿戴式应用中的可滚动项目创建曲线布局，请执行以下操作：

在相关的 XML 布局中，将 WearableRecyclerView 用作主容器。
将 setEdgeItemsCenteringEnabled(boolean) 方法设置为 true。这会使列表中的第一个项目和最后一个项目在屏幕中垂直居中。
使用 WearableRecyclerView.setLayoutManager() 方法设置项目在屏幕上的布局。

```
wearableRecyclerView.apply {
    // To align the edge children (first and last) with the center of the screen.
    isEdgeItemsCenteringEnabled = true
    ...
    layoutManager = WearableLinearLayoutManager(this@MainActivity)
}
```

如果您的应用在自定义滚动时的子视图外观方面有特定要求（例如，在项目从中心向外滚动时缩放图标和文本），请扩展 WearableLinearLayoutManager.LayoutCallback 类并替换 onLayoutFinished 方法。

以下代码段举例说明了如何通过扩展 WearableLinearLayoutManager.LayoutCallback 类来自定义随着项目向离中心越来越远的位置滚动而进行缩放：
```
/** How much icons should scale, at most.  */
private const val MAX_ICON_PROGRESS = 0.65f

class CustomScrollingLayoutCallback : WearableLinearLayoutManager.LayoutCallback() {

    private var progressToCenter: Float = 0f

    override fun onLayoutFinished(child: View, parent: RecyclerView) {
        child.apply {
            // Figure out % progress from top to bottom.
            val centerOffset = height.toFloat() / 2.0f / parent.height.toFloat()
            val yRelativeToCenterOffset = y / parent.height + centerOffset

            // Normalize for center.
            progressToCenter = Math.abs(0.5f - yRelativeToCenterOffset)
            // Adjust to the maximum scale.
            progressToCenter = Math.min(progressToCenter, MAX_ICON_PROGRESS)

            scaleX = 1 - progressToCenter
            scaleY = 1 - progressToCenter
        }
    }
}
```

最后：
```
wearableRecyclerView.layoutManager =
        WearableLinearLayoutManager(this, CustomScrollingLayoutCallback())
```

### 5.7 滑动关闭
滑动关闭手势
用户从左向右滑动以关闭当前屏幕。因此，我们建议您使用以下内容：

垂直布局
内容容器
我们还建议您的应用不要包含水平滑动手势。

水平滚动视图 
在某些情况下，例如在包含支持平移的地图的视图中，用户界面无法阻止水平滑动。在这种情况下，有两种选择：

如果返回堆栈很短，用户可以通过按下电源按钮关闭应用程序并返回到表盘主屏幕。
如果您希望用户向下返回堆栈，您可以将视图包装在一个SwipeDismissFrameLayout支持边缘滑动的对象中。当视图或其子视图 true从 调用返回时启用边缘滑动。边缘轻扫让用户可以通过从屏幕最左侧 10% 的区域（而不是视图中的任何位置）轻扫来关闭视图。 canScrollHorizontally()

```
<androidx.wear.widget.SwipeDismissFrameLayout
    xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:id="@+id/swipe_dismiss_root" >

    <TextView
        android:id="@+id/test_content"
        android:layout_width="match_parent"
        android:layout_height="match_parent"
        android:gravity="center"
        android:text="Swipe me to dismiss me." />
</androidx.wear.widget.SwipeDismissFrameLayout>
```

### 5.8 显示确认动画
确认动画会在用户完成操作时向其提供视觉反馈。 它们会覆盖整个屏幕，确保用户可以一目了然地查看这些确认信息。

在大多数情况下，您无需使用单独的确认动画。请查看设计原则了解详情。

Jetpack Wear 界面库提供了 ConfirmationActivity，用于在应用中显示确认动画。

显示确认动画
ConfirmationActivity 用于当用户在穿戴式设备上完成某个操作后显示确认动画。

确认内容有三种类型：

成功：已成功在穿戴式设备上完成操作。
失败：未能完成操作。
在手机上打开：操作导致在手机上显示某些内容，或者为了完成该操作，用户需要转到手机上继续操作。

```
<manifest>
  <application>
    ...
    <activity
        android:name="androidx.wear.activity.ConfirmationActivity">
    </activity>
  </application>
</manifest>
```

### 5.9 让应用始终显示在Wear上

当用户不再使用手表时，Wear OS 会自动使处于活跃状态的应用进入低功耗模式。这称为“系统氛围模式”。如果用户在一定时间范围内再次与手表进行互动，Wear OS 会使用户重新进入应用中上次离开时的位置。

对于特定用例，例如用户想在跑步期间查看心率和步速，您还可以控制在低功耗氛围模式下显示哪些内容。在氛围模式和互动模式下均处于运行状态的 Wear OS 应用称为始终开启的应用。

让应用始终可见会影响电池续航时间，因此在为应用添加这项功能时，应该考虑这个影响。

配置项目 
如需支持氛围模式，请按以下步骤操作：

根据创建和运行穿戴式应用页面上的配置，创建或更新您的项目。
向 Android 清单文件添加 WAKE_LOCK 权限：
```
<uses-permission android:name="android.permission.WAKE_LOCK" />
```


使用 AmbientModeSupport 类支持氛围模式 
如需使用 AmbientModeSupport 类，请执行以下操作：

创建 FragmentActivity 的子类，即创建其任一子类。
实现 AmbientCallbackProvider 接口，如以下示例所示。替换 getAmbientCallback() 方法，以提供对来自 Android 系统的氛围事件做出反应所需的回调。在后面的一个步骤中，您将创建自定义回调类。
```
public class MainActivity extends AppCompatActivity implements AmbientModeSupport.AmbientCallbackProvider {
    ...
    @Override
    public AmbientModeSupport.AmbientCallback getAmbientCallback() {
        return new MyAmbientCallback();
    }
    ...
}
```

使用 WearableActivity 类支持氛围模式 
对于新项目和现有项目，您可以通过更新项目配置，为 Wear 应用添加氛围模式支持。

创建一个支持氛围模式的 activity 
您可以使用 WearableActivity 类在 activity 中启用氛围模式，如下所示：

创建扩展 WearableActivity 的 activity。
在您的 activity 的 onCreate() 方法中，调用 setAmbientEnabled() 方法。
```
public class MainActivity extends WearableActivity {
    @Override
    public void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);

        setAmbientEnabled();
        ...
    }
```

当 activity 切换到氛围模式时，系统会调用氛围回调的 onEnterAmbient() 方法。以下代码段展示了如何在系统切换到氛围模式后将文本颜色更改为白色并停用抗锯齿功能：
```
@Override
public void onEnterAmbient(Bundle ambientDetails) {
    super.onEnterAmbient(ambientDetails);

    stateTextView.setTextColor(Color.WHITE);
    stateTextView.getPaint().setAntiAlias(false);
}
```