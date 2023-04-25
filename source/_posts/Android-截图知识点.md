---
title: Android 截图知识点
date: 2023-04-25 10:55:15
top: false
cover: false
toc: true
mathjax: true
tags:
- Android
categories:
- Android
---

## 1 快捷方式服务TileService

> 是什么？
TileService是Android中的一个服务（Service）类，它提供了一种可以在快速设置面板（Quick Settings Panel）中显示自定义操作（例如Wi-Fi开关、屏幕旋转锁定等）的方式。
TileService与其他服务类似，可以在后台长时间运行而不影响应用程序的性能，并且可以响应来自系统和用户的事件。
当用户打开快速设置面板时，系统会调用TileService的方法来获取操作的状态和图标。此外，当用户点击操作时，TileService还可以执行相应的操作。
要创建自己的TileService，需要继承TileService类并重写其中的一些方法，例如onCreate()、onStartListening()、onClick()等。然后在AndroidManifest.xml文件中声明该服务，以便系统可以找到它并将其添加到快速设置面板中。

> 怎么用？
若要使用TileService创建自定义操作并在快速设置面板中显示，需要按照以下步骤进行：
创建一个新类并继承TileService类。
在新类中重写onStartListening()方法，这将允许系统调用您的服务以更新状态和图标。
在新类中重写onClick()方法，以便在用户点击该操作时执行相应的操作。
在AndroidManifest.xml文件中声明您的新类作为服务。确保使用<service>标记指定您的新类，并在其中包含android.permission.BIND_QUICK_SETTINGS_TILE权限。
一旦完成了上述步骤，您的自定义操作将会出现在用户的快速设置面板中。用户可以通过长按操作以便编辑或删除它。
需要注意的是，TileService类仅适用于运行Android 5.0及以上版本的设备，因为这些版本中引入了快速设置面板功能。

### 1.1 一个简单的快捷方式服务

首先需要自定义一个TileService
```
class ScreenshotTileService: TileService() 
```

然后需要实现onStartCommand
```
override fun onStartCommand(intent: Intent?, flags: Int, startId: Int): Int {
    instance = this
    Log.e(TAG, "走到了ScreenshotTileService 的onStartCommand方法中")
    if (intent?.action == FOREGROUND_ON_START) {
        foreground()
    }
    /**
        * START_STICKY是一个用于指定服务启动方式的标志（flag）
        * 它可以在启动服务时作为参数传递给startService()方法。当使用START_STICKY标志启动服务时，
        * 如果该服务因系统资源不足而被关闭，系统会尝试重新启动该服务，并且保留其之前的Intent对象（即传递给startService()方法的参数），
        * 以便在系统有足够资源时能够重新启动并处理这些Intent。
        * 具体来说，如果服务因为某些原因（比如内存不足）而被系统关闭，那么系统会尝试重新启动该服务，
        * 并将之前的Intent对象传递给onStartCommand()方法，让服务能够继续处理之前未完成的任务。
        * 但是，如果在服务被关闭后没有任何未处理的Intent对象，系统就不会重新启动该服务，除非有新的Intent对象被传递进来。
        * 总之，使用START_STICKY标志启动服务可以保证服务在被关闭后能够自动重启，并且能够继续处理之前未完成的任务，从而提高应用程序的稳定性和可靠性。
        */
    return START_STICKY
}
```

然后可以再onClick方法中，处理点击快捷方式的逻辑：
```
 override fun onClick() {
        super.onClick()
        Log.e(TAG, "监听到了点击事件哦")

        foreground()
        Log.e(TAG, "前台服务开启成功")

        setState(Tile.STATE_ACTIVE)

        if (App.instance.prefManager.tileAction == getString(R.string.setting_tile_action_value_screenshot)) {
            Log.e(TAG, "开始截图，整页截图")
            App.instance.screenshot(this)
        } else {
            Log.e(TAG, "开始截图，区域截图")
            // 区域截图
//            App.instance.screenshotPartial(this)
        }
    }
```
ok，这里接受到点击事件，就开始执行截图的任务了，可以知道这个快捷方式的服务，我们的任务就是为了截图。

然后我们需要在AndroidManifest.xml中配置服务：
```
<service
        android:name=".service.ScreenshotTileService"
        android:exported="true"
        android:foregroundServiceType="mediaProjection"
        android:icon="@drawable/ic_stat_name"
        android:label="@string/tile_label"
        android:permission="android.permission.BIND_QUICK_SETTINGS_TILE"
        tools:targetApi="q">
        <intent-filter>
            <action android:name="android.service.quicksettings.action.QS_TILE" />
        </intent-filter>
    </service>
```
这个服务可以被外部使用，配置exported为true
另外需要配置foregroundServiceType为mediaProject
有如下几种类型可以选择：
```
   <!--快捷设置按钮服务配置-->
        <!-- android:foregroundServiceType=["camera" | "connectedDevice" |
               "dataSync" | "location" | "mediaPlayback" |
               "mediaProjection" | "microphone" | "phoneCall"]
            -->
```
这里我们用到的是屏幕录制，所以选择了mediaProjection

另外权限需要配置：
```
   android:permission="android.permission.BIND_QUICK_SETTINGS_TILE"
```
表示拥有了快捷方式的权限， 具体含义是这个应用程序请求被授予连接到快速设置磁贴服务（Quick Settings Tile）的权限。

然后设置了一个快捷方式需要监听的广播：
```
 <intent-filter>
                <action android:name="android.service.quicksettings.action.QS_TILE" />
            </intent-filter>
```
这样快捷方式的服务就创建好了，我们可以通过下拉屏幕顶部，编辑，添加我们的快捷方式了。

## 2 屏幕录制实现截图

### 2.1 申请权限
这个需要在一个Activity中申请，但我们又不想跟业务耦合，所以单独创建了一个空的Activity来处理权限申请。
在这个空的Activity中：
```
(getSystemService(Context.MEDIA_PROJECTION_SERVICE) as? MediaProjectionManager)?.apply {
                Log.e(TAG, "这里设置了Manager")
                App.setMediaProjectionManager(this)
                try {
                    startActivityForResult(createScreenCaptureIntent(), SCREENSHOT_REQUEST_CODE)
                } catch(e: ActivityNotFoundException) {
                    Log.e(TAG, "startActivityForResult(createScreenCaptureIntent, ...) failed with", e)
                    toastMessage(getString(R.string.permission_missing_screen_capture), ToastType.ERROR)
                    finish()
                }
            }
```
这个createScreenCaptureIntent是MediaProjectionManager中的方法，这样就跳转到系统的屏幕录制和投射内容的权限申请框中了。

然后就是在onActivityResult中接收权限了：
```
    override fun onActivityResult(requestCode: Int, resultCode: Int, data: Intent?) {
        super.onActivityResult(requestCode, resultCode, data)
        if (SCREENSHOT_REQUEST_CODE == requestCode) {
            if (RESULT_OK == resultCode) {
                if (BuildConfig.DEBUG) Log.v(
                    TAG,
                    "onActivityResult() RESULT_OK"
                )
                data?.run {
                    (data.clone() as? Intent)?.apply {
                        App.setScreenshotPermission(this)
                    }
                }
            } else {
                App.setScreenshotPermission(null)
                Log.w(
                    TAG,
                    "onActivityResult() No screen capture permission: resultCode==$resultCode"
                )
                toastMessage(getString(R.string.permission_missing_screen_capture), ToastType.ERROR)
            }
        }
        finish()
    }
```
这里接收到权限后，如果是申请成功了，就将这个intent拷贝一份给App，暂存一下，然后回触发权限回调监听。
回调中可以处理我们自己的逻辑，比如去截图等。

### 2.2 屏幕录制截图方式
前面拿到权限后，通过在onClick方法中，我们可以去写截图相关的逻辑。
```
  val tileService =
                    if (context is ScreenshotTileService) context else ScreenshotTileService.instance!!
    // Open a activity to collapse notification bar, and wait for notification panel closing
    val intent: Intent
    if (alreadyCollapsed) {
        intent = NoDisplayActivity.newIntent(context, true)
    } else {
        tileService.takeScreenshotOnStopListening = true
        intent = NoDisplayActivity.newIntent(context, false)
    }
    intent.flags = Intent.FLAG_ACTIVITY_NEW_TASK
    try {
        Log.e(TAG, "通过tile服务的startActivity方法跳转到NoDisplayActivity")
        tileService.startActivityAndCollapse(intent)
    } catch (e: NullPointerException) {
        context.startActivity(intent)
    }
```
这里我们可以通过一个不显示的NoDisplayActivity来中转一下。
在清单里面我们这样配置这个NoDisplayActivity:
```
<activity
    android:name=".ui.other.NoDisplayActivity"
    android:excludeFromRecents="true"
    android:exported="false"
    android:theme="@android:style/Theme.NoDisplay" />

<style name="Theme.NoDisplay">
    <item name="windowBackground">@null</item>
    <item name="windowContentOverlay">@null</item>
    <item name="windowIsTranslucent">true</item>
    <item name="windowAnimationStyle">@null</item>
    <item name="windowDisablePreview">true</item>
    <item name="windowNoDisplay">true</item>
</style>
```
总之，这个页面应该是透明不可见，且在历史栈中不会留痕。

第一次跳转到这个NoDisplayActivity我们并不会去截图，原因是快捷方面面板没有收起，我们先调转到NoDisplayActivity中去收起面板。
然后监听面板收缩，在ScreenshotTileService中的onStopListening中：
```
override fun onStopListening() {
    super.onStopListening()
    Log.e(TAG, "上滑菜单时调用，调用一次 这里应该是判断是否要去截图了")

    // 这里是传统方式截图
    if (takeScreenshotOnStopListening) {
        takeScreenshotOnStopListening = false
        Handler(Looper.getMainLooper()).postDelayed({
            App.instance.takeScreenshotFromTileService(this)
        }, 700)
    } else {
        background()
    }

    setState(Tile.STATE_INACTIVE)

}
```
这里我们延迟去通过App.instance去发起截图。

具体截图代码如下：
```
  Log.e(TAG, "传统截图 走NoDisplayActivity")
            val intent = NoDisplayActivity.newIntent(context, true)
            intent.flags = Intent.FLAG_ACTIVITY_NEW_TASK
            context.startActivity(intent)
```
可以看到，还是交给NoDisplayActivity来处理，只是有个参数传了true。

最终会调用一个静态方法：
```
fun screenshot(context: Context, partial: Boolean = false) {
    Log.e("TEST##", "开始截图")
    if (partial || !tryNativeScreenshot()) {
        TakeScreenshotActivity.start(context, partial)
    }
}
```
委托给TakeScreenshotActivity去完成。

在onCreate里面，创建了一个ImageReader对象，然后将这个对象的surface获取到。
```
imageReader = ImageReader.newInstance(screenWidth, screenHeight, PixelFormat.RGBA_8888, 1)

surface = imageReader?.surface
```

然后再onCreate会走App的获取权限并截图的方法：
```
App.acquireScreenshotPermission(this, this)
```
底层如下：
```
if (null != mediaProjection) {
    mediaProjection!!.stop()
    mediaProjection = null
}

if (screenshotTileService != null) {
    screenshotTileService.foreground()
}

mediaProjection = mediaProjectionManager!!.getMediaProjection(
    Activity.RESULT_OK,
    (screenshotPermission!!.clone() as Intent)
)
if (onAcquireScreenshotPermissionListener != null) {
    onAcquireScreenshotPermissionListener!!.onAcquireScreenshotPermission(false)
}
```
这里创建了一个全局的MediaProjectManager对象，
然后直接触发到TakeScreenshotActivity的回调：
```
    override fun onAcquireScreenshotPermission(isNewPermission: Boolean) {
        Log.e(TAG, "这里拿到了权限$isNewPermission")

//        ScreenshotTileService.instance?.onAcquireScreenshotPermission(isNewPermission)
//        ScreenshotTileService.instance?.foreground()
//        BasicForegroundService.instance?.foreground()
        if (partial) {
            partialScreenshot()
        } else {
            if (isNewPermission) {
                // Wait a little bit, so the permission dialog can fully hide itself
                Handler(Looper.getMainLooper()).postDelayed({
                    prepareForScreenSharing()
                }, App.instance.prefManager.originalAfterPermissionDelay)
            } else {
                Handler(Looper.getMainLooper()).postDelayed({
                    prepareForScreenSharing()
                }, App.instance.prefManager.originalAfterPermissionDelay)
            }
        }
    }
```
这里就是延迟执行 prepareForScreenSharing，这里应该就是截图的主要代码了。
主要逻辑1：
```
   // 先拿到一个MediaProjection
        mediaProjection = try {
            App.createMediaProjection()
        } catch (e: SecurityException) {
            Log.e(TAG, "prepareForScreenSharing(): SecurityException 1")
            null
        }

        fun createMediaProjection(): MediaProjection? {
            if (BuildConfig.DEBUG) Log.v(TAG, "createMediaProjection()")
//            val basicForegroundService = BasicForegroundService.instance
            val screenshotTileService = ScreenshotTileService.instance
//            basicForegroundService?.foreground() ?: screenshotTileService?.foreground()
            if (mediaProjection == null) {
                if (screenshotPermission == null) {
                    screenshotPermission = ScreenshotTileService.screenshotPermission
                }
//                if (screenshotPermission == null && Build.VERSION.SDK_INT >= Build.VERSION_CODES.P) {
//                    screenshotPermission = ScreenshotAccessibilityService.screenshotPermission
//                }
                if (screenshotPermission == null) {
                    return null
                }
                mediaProjection = mediaProjectionManager!!.getMediaProjection(
                    Activity.RESULT_OK,
                    (screenshotPermission!!.clone() as Intent)
                )
            }
            return mediaProjection
        }
```

然后创建一个虚拟设备：
```
 startVirtualDisplay()

    private fun startVirtualDisplay() {
        virtualDisplay = createVirtualDisplay()
        imageReader?.setOnImageAvailableListener({
            if (BuildConfig.DEBUG) Log.v(TAG, "startVirtualDisplay:onImageAvailable()")
            // Remove listener, after first image
            it.setOnImageAvailableListener(null, null)
            // Read and save image
            saveImage()
        }, null)
    }

      /**
     * 通过MediaProject创建一个虚拟设备
     */
    private fun createVirtualDisplay(): VirtualDisplay? {
        return mediaProjection?.createVirtualDisplay(
            "ScreenshotTaker",
            screenWidth, screenHeight, screenDensity,
            DisplayManager.VIRTUAL_DISPLAY_FLAG_AUTO_MIRROR,
            surface, null, null
        )
    }
```
这里注入截图后的回调，走saveImage方法。

然后后面的逻辑就是将imageReader读到的图片，保存起来。
```
   val image = try {
            imageReader?.acquireNextImage()
        } catch (e: UnsupportedOperationException) {
            stopScreenSharing()
            Log.e(TAG, "saveImage() acquireNextImage() UnsupportedOperationException", e)
            screenShotFailedToast("Could not acquire image.\nUnsupportedOperationException\nThis device is not supported.")
            finish()
            return
        }
```

## 3 模拟系统按键实现截图

这种方式就是利用无障碍实现屏幕点击，模拟按下截屏键，但弊端是无法拿到截图的图像，只能在图库里面找。

### 3.1 无障碍服务

是什么？
> AccessibilityService是Android中的一个服务（Service）类，它提供了一种可以帮助用户访问和使用设备功能的方式。AccessibilityService可以在后台长时间运行而不影响应用程序的性能，并且可以监视系统事件、应用程序界面等，并提供与这些事件交互的方法。当用户需要特殊辅助功能（如屏幕放大、语音输入等）时，AccessibilityService可以为其提供支持。<br>
AccessibilityService还提供了一些回调方法，例如onAccessibilityEvent()、onInterrupt()等，以便处理来自系统和用户的事件并执行相应操作。要创建自己的AccessibilityService，需要继承AccessibilityService类并重写其中的一些方法，例如onServiceConnected()、onAccessibilityEvent()等。然后在AndroidManifest.xml文件中声明该服务，以便系统可以找到它并将其注册为可用的辅助功能服务。<br>
需要注意的是，使用AccessibilityService需要用户授权，并且在实现过程中需要考虑到隐私和安全问题。

怎么用？
> 要使用AccessibilityService来实现辅助功能，需要按照以下步骤进行：
创建一个新类并继承AccessibilityService类。<br>
在新类中重写onAccessibilityEvent()方法和onInterrupt()方法，其中onAccessibilityEvent()方法用于处理系统事件（例如窗口内容变化、通知等）的回调，而onInterrupt()方法则是当服务被中断（例如由于设备休眠或设备重新启动时）时执行的回调。<br>
在新类中配置服务的特性，例如可以使用setServiceInfo()方法设置服务的描述信息和其他参数。<br>
在AndroidManifest.xml文件中声明您的新类作为服务。确保使用<service>标记指定您的新类，并在其中包含android.permission.BIND_ACCESSIBILITY_SERVICE权限。<br>
对于需要用户授权的辅助功能，可以通过发送Intent请求启动系统设置页面来让用户授权该服务。例如，可以使用ACTION_ACCESSIBILITY_SETTINGS Intent打开辅助功能设置页面，然后让用户启用您的服务。<br>
需要注意的是，使用AccessibilityService需要谨慎考虑隐私和安全问题，因为服务可以监视用户的操作和敏感信息。因此，在实现过程中要遵循最佳实践，例如只收集必要的数据、加密数据传输等。

### 3.2 申请无障碍权限

这个通过Intent跳转到无障碍列表：
```
  Intent(Settings.ACTION_ACCESSIBILITY_SETTINGS).apply {
                if (resolveActivity(context.packageManager) != null) {
                    if (returnTo != null) {
                        App.instance.prefManager.returnIfAccessibilityServiceEnabled = returnTo
                    }
                    context.startActivity(this)
                }
            }
```
<img src=snapshot1.png width=50%>
跳转到这个页面，需要手动开启无障碍权限。

开启后，会回调无障碍服务的 onServiceConnected方法：

这样开启后，我们设置无障碍的悬浮窗就不需要其他在顶层显示的权限了：
```
 /**
     * 获取悬浮窗的布局参数
     */
    private fun windowViewAbsoluteLayoutParams(x: Int, y: Int): WindowManager.LayoutParams {
        return WindowManager.LayoutParams().apply {
            type = WindowManager.LayoutParams.TYPE_ACCESSIBILITY_OVERLAY
            format = PixelFormat.TRANSLUCENT
            flags = flags or WindowManager.LayoutParams.FLAG_NOT_FOCUSABLE
            width = WindowManager.LayoutParams.WRAP_CONTENT
            height = WindowManager.LayoutParams.WRAP_CONTENT
            @SuppressLint("RtlHardcoded")
            gravity = Gravity.TOP or Gravity.LEFT
            this.x = x
            this.y = y
        }
    }
```
就是这个：WindowManager.LayoutParams.TYPE_ACCESSIBILITY_OVERLAY
有无障碍权限，就可以显示这个悬浮框。

### 3.3 大于等于Android11的截图
通过无障碍中的一个 takeScreenshot方法实现：
```
/**
     * 这段代码定义了一个名为 takeScreenshot() 的方法，该方法用于在 Android 平台上进行截屏操作。
     * 用于触发截屏操作并获取截屏结果。takeScreenshot() 方法的优点是使用简单、灵活性高，并且可以直接获取截屏结果，缺点是需要 Android 11（API 级别 30）或更高版本才能使用。
     */
    @RequiresApi(Build.VERSION_CODES.R)
    fun takeScreenshot() {
        super.takeScreenshot(Display.DEFAULT_DISPLAY, { r -> Thread(r).start() },
            object : TakeScreenshotCallback {
                override fun onSuccess(screenshot: ScreenshotResult) {
                    // 在这段代码中，一个名为 "screenshot" 的硬件缓冲区被用来创建位图。Bitmap.wrapHardwareBuffer() 函数用于包装硬件缓冲区并创建位图。
                    // screenshot.colorSpace 参数指定位图的颜色空间。
                    //然后使用 Bitmap.copy() 函数从包装的硬件缓冲区创建一个新位图。该函数创建具有指定配置（在本例中为 ARGB_8888）的新位图。false 参数指定新位图是否可变。
                    val bitmap = Bitmap.wrapHardwareBuffer(
                        screenshot.hardwareBuffer,
                        screenshot.colorSpace
                    )?.copy(Bitmap.Config.ARGB_8888, false)
                    screenshot.hardwareBuffer.close()

                    if (bitmap == null) {
                        Log.e(
                            TAG,
                            "takeScreenshot() bitmap == null, falling back to GLOBAL_ACTION_TAKE_SCREENSHOT"
                        )
                        // 重新调用截图
                        Handler(Looper.getMainLooper()).post {
                            fallbackToSimulateScreenshotButton()
                        }
                    } else {
                        // 保存图片
                        val saveImageResult = saveBitmapToFile(
                            this@ScreenshotAccessibilityService,
                            bitmap,
                            App.instance.prefManager.fileNamePattern,
                            compressionPreference(applicationContext),
                            null,
                            useAppData = "saveToStorage" !in App.instance.prefManager.postScreenshotActions,
                            directory = null
                        )
                        Handler(Looper.getMainLooper()).post {
                            onFileSaved(saveImageResult)
                        }
                    }
                }

                override fun onFailure(errorCode: Int) {
                    Log.e(
                        TAG,
                        "takeScreenshot() -> onFailure($errorCode), falling back to GLOBAL_ACTION_TAKE_SCREENSHOT"
                    )
                    Handler(Looper.getMainLooper()).post {
                        fallbackToSimulateScreenshotButton()
                    }
                }
            })
    }

```
通过回调方法传出的ScreenshotResult拿到结果。
然后利用Bitmap.wrapHardwareBuffer生成一个图片对象，然后保存到本地。

### 3.4 Android10及以下的截图方法(Android11以上也能用)
```
  success = try {
                Log.e("TEST##", "performGlobalAction 方法")
                // performGlobalAction(GLOBAL_ACTION_TAKE_SCREENSHOT) 是一个 Android 平台上的系统级方法，用于触发截屏操作。
                // 方法本身并不能直接获取截屏结果，因为它是一个系统级别的操作，是由辅助功能服务模拟用户执行操作的过程，因此并没有返回值或回调函数用于获取截屏结果。
                performGlobalAction(GLOBAL_ACTION_TAKE_SCREENSHOT)
            } catch (e: Exception) {
                Log.e(TAG, "Failed to performGlobalAction(GLOBAL_ACTION_TAKE_SCREENSHOT)", e)
                false
            }
```
直接调用performGlobalAction(GLOBAL_ACTION_TAKE_SCREENSHOT)
这句代码可以模拟原生截图。
这个方法是无障碍服务的方法。

## 4 长截图

### 4.1 申请悬浮窗权限

```
 fun start(view: View?) {
        if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.M) {
            // 动态申请悬浮窗权限
            if (!Settings.canDrawOverlays(this@LongSnapActivity)) {
                val intent = Intent(
                    Settings.ACTION_MANAGE_OVERLAY_PERMISSION,
                    Uri.parse("package:$packageName")
                )
                startActivityForResult(intent, REQUEST_WINDOW_GRANT)
            } else {
                initMediaProjectionManager()
            }
        } else {
            initMediaProjectionManager()
        }
    }
```
这里没有用无障碍，所以需要悬浮窗权限。

拿到权限后，初始化MediaProjection
```
 private fun initMediaProjectionManager() {
        if (mediaProjectionManager != null) {
            return
        }
        // How to use MediaProjectionManager
        // 1.get an instance of MediaProjectionManager
        mediaProjectionManager =
            getSystemService(MEDIA_PROJECTION_SERVICE) as MediaProjectionManager
        // 2.create the permissions intent and show it to user
        startActivityForResult(
            mediaProjectionManager!!.createScreenCaptureIntent(),
            REQUEST_MEDIA_PROJECTION
        )
    }
```
这里是直接点击了右下角的图片后并且有悬浮窗权限的时候，会startActivity到这个intent。

> 这段代码的作用是获取MediaProjection，以便在Android设备上进行屏幕录制或截图。具体来说，通过调用getSystemService()方法获取MediaProjectionManager服务，并使用createScreenCaptureIntent()方法创建一个屏幕捕获意图（Intent）。随后，将该意图传递给startActivityForResult()方法启动活动（Activity），并请求用户授权允许屏幕捕获操作。如果用户同意，则可以通过onActivityResult()方法获取MediaProjection对象，从而开始屏幕录制或截图操作。需要注意的是，使用MediaProjection需要用户授权，并且在实现过程中需要考虑到隐私和安全问题。

在开启意图后，会有一个onActivityResult的回调，这里开启一个悬浮窗服务：
```
    override fun onActivityResult(requestCode: Int, resultCode: Int, data: Intent?) {
        super.onActivityResult(requestCode, resultCode, data)
        when (requestCode) {
            REQUEST_MEDIA_PROJECTION -> if (resultCode == RESULT_OK && data != null) {
                try {
                    val mWindowManager = getSystemService(WINDOW_SERVICE) as WindowManager
                    val metrics = DisplayMetrics()
                    mWindowManager.defaultDisplay.getMetrics(metrics)
                } catch (e: Exception) {
                    Log.e(TAG, "MediaProjection error")
                }
                val service = Intent(this@LongSnapActivity
                    , FloatWindowsService::class.java)
                service.putExtra("data", resultCode)
                service.putExtra("data", data)
                if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.O) {
                    startForegroundService(service)
                } else {
                    startService(service)
                }
                moveTaskToBack(false)
            }
```

这个FloatWindowService是我们自己创建的悬浮窗服务，可以控制什么时候开始录屏，什么时候结束。当然需要再service的onStartCommand里面创建一个通知，Android8.0以上要求。


### 4.2 手势监听

然后就是点击浮动按钮事件了。
```
 private inner class FloatGestureTouchListener : GestureDetector.OnGestureListener {
        var lastX = 0
        var lastY = 0
        var paramX = 0
        var paramY = 0
        override fun onDown(event: MotionEvent): Boolean {
            lastX = event.rawX.toInt()
            lastY = event.rawY.toInt()
            paramX = mLayoutParams!!.x
            paramY = mLayoutParams!!.y
            return true
        }

        override fun onShowPress(e: MotionEvent) {}
        override fun onSingleTapUp(e: MotionEvent): Boolean {
            if (!isRunning) {
                virtualDisplay()
                isRunning = true
                isStop = false
                mFloatView!!.setImageBitmap(
                    BitmapFactory.decodeResource(
                        resources,
                        R.drawable.stop
                    )
                )
                touchWindow!!.show()
                startScreenShot()
            } else {
                isStopFlag = true
                isStop = true
                mFloatView!!.visibility = View.GONE
                mFloatView!!.setImageBitmap(
                    BitmapFactory.decodeResource(
                        resources,
                        R.drawable.start
                    )
                )
            }
            return true
        }

        override fun onScroll(
            e1: MotionEvent,
            e2: MotionEvent,
            distanceX: Float,
            distanceY: Float
        ): Boolean {
            val dx = e2.rawX.toInt() - lastX
            val dy = e2.rawY.toInt() - lastY
            mLayoutParams!!.x = paramX + dx
            mLayoutParams!!.y = paramY + dy
            return true
        }

        override fun onLongPress(e: MotionEvent) {}
        override fun onFling(
            e1: MotionEvent,
            e2: MotionEvent,
            velocityX: Float,
            velocityY: Float
        ): Boolean {
            return false
        }
    }
```
这里监听了手势。

单独点击后，执行开始录制事件（startScreenShot）
```
   private fun startCapture() {
        // 这个方法已经被调用过，在获取另外一个新的image之前，请先关闭原有有的image
        val image = mImageReader!!.acquireLatestImage()
        if (image == null) {
            startScreenShot()
        } else {
            val mSaveTask = SaveTask()
            mSaveTask.execute(image)
        }
    }
```
这里不为空，就开启一个异步任务，imageReader可以通过acquireLatestImage获取最新图片。

### 4.3 录制流程一：创建虚拟设备
在上方的手势监听中，首先是一个virtualDisplay方法：
```
    /**
     * 最终得到当前屏幕的内容，注意这里mImageReader.getSurface()被传入，屏幕的数据也将会在ImageReader中的Surface中
     */
    private fun virtualDisplay() {
        mVirtualDisplay = mediaProjection!!.createVirtualDisplay(
            "screen-mirror",
            mScreenWidth,
            mScreenHeight,
            mScreenDensity,
            DisplayManager.VIRTUAL_DISPLAY_FLAG_AUTO_MIRROR,
            mImageReader!!.surface,
            null,
            null
        )
    }
```
这里通过mediaProjection创建了一个虚拟显示器。

这个mediaProjection来自这里：
```
override fun onStartCommand(intent: Intent, flags: Int, startId: Int): Int {
    createNotificationChannel()
    mResultCode = intent.getIntExtra("code", -1)
    mResultData = intent.getParcelableExtra("data")
    //mResultData = intent.getSelector();
    mediaProjection = LongSnapActivity.mediaProjectionManager!!
        .getMediaProjection(mResultCode, mResultData!!)
    //mMediaProjection =  ((MediaProjectionManager) Objects.requireNonNull(getSystemService(Context.MEDIA_PROJECTION_SERVICE))).getMediaProjection(mResultCode, mResultData);
    Log.e(TAG, "mMediaProjection created: " + mediaProjection)
    return super.onStartCommand(intent, flags, startId)
}
```
是因为在这个LongSnapActivity中，拿到了悬浮窗权限后，初始化了mediaProjectManager，然后通过startActivityResult传入这个manager的一个createScreenCaptureIntent，跳转到一个录制屏幕和投射的页面，拿到权限后才开启这个悬浮窗服务。

然后再这个服务的onCreate中初始化了ImageReader:
```
  private fun createImageReader() {
        // 设置截屏的宽高
        mImageReader =
            ImageReader.newInstance(mScreenWidth, mScreenHeight, PixelFormat.RGBA_8888, 1)
    }
```

### 4.4 第一次捕获
首先利用imageReader.acquireLatestImage()方法：
> 这段代码使用ImageReader类的acquireLatestImage()方法来获取最新的屏幕图像。具体来说，ImageReader是一个用于捕获屏幕内容的类，它可以创建一个Surface对象，用于接收显示器输出的帧缓冲区，并提供了一些方法来获取这些帧缓冲区中的像素数据。acquireLatestImage()方法将返回最新的可用图像，如果没有可用的图像，则返回null。一旦获取到图像，就可以通过其getWidth()、getHeight()和getPlanes()等方法获取图像的大小和像素数据。需要注意的是，使用ImageReader需要考虑到内存占用和性能问题，因为它会在后台持续捕获屏幕图像并缓存数据。

```
    private fun startCapture() {
        // 这个方法已经被调用过，在获取另外一个新的image之前，请先关闭原有有的image
        val image = mImageReader!!.acquireLatestImage()
        if (image == null) {
            Log.e(TAG, "开始截图")
            startScreenShot()
```
这里如果image为null，则开始截图：
```
  private fun startScreenShot() {
        handler.postDelayed({ startCapture() }, 30)
    }
```
这里延迟执行startCapture方法，相当于如果为空，就延迟30毫秒，继续走该保存方法。
然后如果不为空：
```
  Log.e(TAG, "执行保存任务")
            val mSaveTask = SaveTask()
            mSaveTask.execute(image)
```
这里拿到ImageReader中读取的最新的Bitmap。

通过读取这个image的planes，来生成一个新的Bitmap对象：
```
 val image = params[0]
            val width = image?.width
            val height = image?.height
            val planes = image?.planes
            val buffer = planes!![0].buffer
            // 每个像素的间距
            val pixelStride = planes[0].pixelStride
            // 总的间距
            val rowStride = planes[0].rowStride
            val rowPadding = rowStride - pixelStride * width!!
            var bitmap = Bitmap.createBitmap(
                width + rowPadding / pixelStride,
                height!!,
                Bitmap.Config.ARGB_8888
            )
            bitmap!!.copyPixelsFromBuffer(buffer)
```

然后利用工具来生成新的 Bitmap:
```
 fun screenShotBitmap(context: Context?, bitmap: Bitmap?, finish: Boolean): Bitmap {
        var bitmap = bitmap
        var scope = 0
        if (!finish) {
            scope = ScreenHeight(context!!) / 4
        }
        val statusHeight = getStatusHeight(context!!)
        val cropRetX = 0
        val cropWidth = ScreenWidth(context)
        val cropHeight = ScreenHeight(context) - statusHeight - scope
        val result = Bitmap.createBitmap(
            bitmap!!,
            cropRetX,
            statusHeight,
            cropWidth,
            cropHeight,
            null,
            false
        )
        bitmap.recycle()
        bitmap = null
        return result
    }
```
这里应该是裁剪图片，只取底部的1/4拼接。

### 4.5 拼图
然后建立一个全局的finalImage，记录拼接中的图片。
每次如果还在录制中，就持续的拼接底部1/4,拼好后，再将这个拼好的图片给全局的finalImage。
拼接图片方法采用了一个工具类：
```
   fun merge(bmp1: Bitmap?, bmp2: Bitmap?): Bitmap? {
        var bmp1 = bmp1
        var bmp2 = bmp2
        val samePart = compare(bmp1!!, bmp2!!)
        val cropHeight = bmp2.height - samePart
        val result: Bitmap
        if (cropHeight > 0) {
            val len = bmp1.width
            val h0 = bmp1.height + cropHeight
            result = Bitmap.createBitmap(len, h0, Bitmap.Config.ARGB_8888)
            merge(result, bmp1, bmp2, h0, bmp1.height, bmp2.height, samePart, len)
        } else {
            return bmp1
        }
        bmp1.recycle()
        bmp2.recycle()
        bmp1 = null
        bmp2 = null
        return result
    }
```
底层采用了c，提升效率。
```

JNIEXPORT void JNICALL Java_com_ishuyun_demo_utils_SewUtils_merge(
        JNIEnv *env, jobject thiz, jobject bmp0, jobject bmp1, jobject bmp2, int h0, int h1, int h2, int samePart, int len) {

    int *pixels_0 = lockPixel(env, bmp0);
    int *pixels_1 = lockPixel(env, bmp1);
    int *pixels_2 = lockPixel(env, bmp2);
    /* -------------------- merge the difference ----------------------- */
    int index = 0;
    while(index < h0) {
        if(index < h1) {
            getRowPixels(pixels_0, index, pixels_1, index, len);
        } else {
            getRowPixels(pixels_0, index, pixels_2, index - h1 + samePart, len);
        }
        index++;
    }
    /* -------------------- merge the difference ----------------------- */
    unlockPixel(env, bmp0);
    unlockPixel(env, bmp1);
    unlockPixel(env, bmp2);
}
```
这里merge两张图片。

## 5 总结

1.首先对截图有一定的了解，截图大致上分两种类型，一种应用内截图，一种应用外，截其他应用的图片。还可以分为普通截图和长截图。还可以分为传统截图和原生截图，传统截图是利用屏幕录制实现，原生截图是利用模拟原生按钮实现。
2.传统截图是利用屏幕录制，mediaProjectManager来实现，需要授予屏幕录制和投射相关权限，入口可以利用快捷方式服务来实现一键截图也就是TileService，这个只是一个工具，方便用户截图，如果不想用这个服务，可以自己定义个悬浮窗按钮，用户点击时截图。这个录制权限主要是通过MediaProjectionManager开启一个Activity，传入createScreenCaptureIntent来实现。
3.传统截图拿到权限后，首先需要创建一个ImageReader对象，然后需要拿到这个对象的surface，后面创建虚拟设备的时候，需要用到这个surface对象，然后就可以利用imageReader的setOnImageAvailableListener回调方法中来保存截图了。
4.原生截图，在Android11之前，通常是采用无障碍服务中的一个 performGlobalAction(GLOBAL_ACTION_TAKE_SCREENSHOT)方法来实现系统截图，但这个方法监听不到返回值，所以Android11后，出了一个 无障碍方法中的takeScreenshot(Display.DEFAULT_DISPLAY)方法可以监听到返回的Bitmap。
5.长截图，目前主流方案还是使用屏幕录制，然后在某个时间节点保存图片，然后拼接，但是弊端是有点难控制完美拼接，总是有点误差。另外系统方面其实提供了长截图，我们可以通过模拟按键，然后无障碍模式去滚动，也能实现长截图。但是结果还是存在相册，可能需要再打开相册，然后上传到服务端。






























