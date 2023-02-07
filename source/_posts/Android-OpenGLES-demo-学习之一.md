---
title: Android OpenGLES demo 学习之一
date: 2023-02-07 12:03:07
top: false
cover: false
toc: true
mathjax: true
tags:
- Android OpenGLES
categories:
- Android
---

## 1 展示一个基本的红色三角形

### 1.1 效果
![](./Android-OpenGLES-demo-%E5%AD%A6%E4%B9%A0%E4%B9%8B%E4%B8%80/01.png)
<img src=01.png>

### 1.2 渲染页
```Kotlin
class NativeRenderActivity : Activity(), AudioCollector.Callback, SensorEventListener {

    private var mMinSetting = -1
    private var mMagSetting = -1

    companion object {
        private const val MIN_DIALOG = 1
        private const val CONTEXT_CLIENT_VERSION = 3
        private const val MAG_DIALOG = 2
        private const val MIN_SETTING = "min_setting"
        private const val MAG_SETTING = "mag_setting"
        private const val TAG: String = "NativeRenderActivity"

        private val REQUEST_PERMISSIONS = arrayOf(
            Manifest.permission.WRITE_EXTERNAL_STORAGE,
            Manifest.permission.RECORD_AUDIO
        )
        private const val PERMISSION_REQUEST_CODE = 1
    }

    private var mRootView: ViewGroup? = null

    /**
     * Hold a reference to our GLSurfaceView
     */
    private var mGLSurfaceView: MyCustomerGLSurfaceView? = null

    private var renderer: MyNativeRenderer? = null

    var type = IMyNativeRendererType.SAMPLE_TYPE


    private var mAudioCollector: AudioCollector? = null

    private var mSensorManager: SensorManager? = null
```
这里声明了必要的类。

MyCustomerGLSurfaceView应该是自定义的GLSurfaceView。

### 1.3 自定义渲染
引用自己写的so库。

```Kotlin
class MyNativeRenderer(activity: Activity) : GLSurfaceView.Renderer, RenderAction {
    private var mActivity: Activity = activity
    var mSampleType = 0

    init {
        System.loadLibrary("ouyangpeng-opengles-lib")
    }
```
这里的ouyangpeng-opengles-lib就是我们自己编写库，这里应该是cpp文件，但这里loadLibrary是load so库，编译时会打包成so库。

```Kotlin
////////////////////////////////// Native 方法///////////////////////////////////////
    // 通用的
    private external fun nativeSurfaceCreate(assetManager: AssetManager)
    private external fun nativeSurfaceChange(width: Int, height: Int)
    private external fun nativeDrawFrame()
    private external fun nativeSetRenderType(sampleCategoryType: Int, renderSampleType: Int)
    private external fun nativeOnDestroy()

    // 特定的方法
    private external fun nativeSwitchBlendingMode()

    // 特定的方法
    private external fun nativeSetDelta(x: Float, y: Float)
    private external fun nativeSetMinFilter(filter: Int)
    private external fun nativeSetMagFilter(filter: Int)

    private external fun nativeSetImageData(
        format: Int,
        width: Int,
        height: Int,
        imageData: ByteArray?
    )

    private external fun nativeSetImageDataWithIndex(
        index: Int,
        format: Int,
        width: Int,
        height: Int,
        imageData: ByteArray?
    )

    private external fun nativeUpdateTransformMatrix(
        rotateX: Float,
        rotateY: Float,
        scaleX: Float,
        scaleY: Float
    )

    private external fun nativeSetAudioData(audioData: ShortArray)

    private external fun nativeSetTouchLocation(x: Float, y: Float)

    private external fun nativeSetGravityXY(x: Float, y: Float)
```
这里定义了native方法。
具体实现的地方在cpp文件中。

```Kotlin
////////////////////////////////// Java 方法///////////////////////////////////////

/**
    * 当Surface被创建的时候，GLSurfaceView 会调用这个方法。
    * 这发送在应用程序第一次运行的时候，并且，当设备被唤醒或者用户从其他Activity切换换来时，
    * 这个方法也可能被调用。在实践中，这意味着，当应用程序运行时，本方法可能会被调用多次。
    *
    *
    * 为什么会有一个未被使用的参数类型GL10呢？
    * 它是OpenGL ES 1.0的API遗留下来的。如果要编写OpenGL ES 1.0的渲染器，就要用这个参数。
    * 但是，对应OpenGL ES 3.0，GLES20/GLEL30类提供了静态方法来读取。
    */
override fun onSurfaceCreated(gl: GL10, config: EGLConfig) {
    val assetManager: AssetManager = mActivity.assets
    nativeSurfaceCreate(assetManager)
}

/**
    * 当Surface被创建以后，每次Surface尺寸变化时，这个方法都会被 GLSurfaceView 调用到。
    * 在横屏、竖屏来回切换的时候，Surface尺寸会发生变化
    */
override fun onSurfaceChanged(gl: GL10, width: Int, height: Int) {
    nativeSurfaceChange(width, height)
}

/**
    * 当绘制一帧时，这个方法会被 GLSurfaceView 调用。
    * 在这个方法中，我们一定要绘制一些东西，即使只是清空屏幕。
    * 因为，在这个方法返回后，渲染缓冲区会被交换并显示在屏幕上，
    * 如果什么都没画，可能会看到糟糕的闪烁效果。
    */
override fun onDrawFrame(gl: GL10) {
    // 提前赋值，变化水印的bitmap
    if (mSampleType == IMyNativeRendererType.SAMPLE_TYPE_KEY_TIME_WATERMARK_STICKER) {
        // 获取时间水印的bitmap
        val mBitmap: Bitmap = CommonUtils.getTimeWaterBitmap()

        val bytes = mBitmap.byteCount
        val buf = ByteBuffer.allocate(bytes)
        mBitmap.copyPixelsToBuffer(buf)
        val byteArray = buf.array()
        setImageDataWithIndex(
            0,
            ImageFormat.IMAGE_FORMAT_RGBA,
            mBitmap.width,
            mBitmap.height,
            byteArray
        )
    }

    // native层去绘制
    nativeDrawFrame()
}

fun setRenderType(sampleCategoryType: Int, renderSampleType: Int) {
    if (sampleCategoryType == IMyNativeRendererType.SAMPLE_TYPE) {
        mSampleType = renderSampleType
    }
    nativeSetRenderType(sampleCategoryType, renderSampleType)
}

fun onDestroy() {
    nativeOnDestroy()
}

override fun switchBlendingMode() {
    nativeSwitchBlendingMode()
}

override fun setMinFilter(filter: Int) {
    nativeSetMinFilter(filter)
}

override fun setMagFilter(filter: Int) {
    nativeSetMagFilter(filter)
}

override fun setDelta(deltaX: Float, deltaY: Float) {
    nativeSetDelta(deltaX, deltaY)
}

override fun setImageData(
    format: Int,
    width: Int,
    height: Int,
    imageData: ByteArray
) {
    nativeSetImageData(format, width, height, imageData)
}

override fun setImageDataWithIndex(
    index: Int,
    format: Int,
    width: Int,
    height: Int,
    imageData: ByteArray
) {
    nativeSetImageDataWithIndex(index, format, width, height, imageData)
}

override fun updateTransformMatrix(
    rotateX: Float,
    rotateY: Float,
    scaleX: Float,
    scaleY: Float
) {
    nativeUpdateTransformMatrix(rotateX, rotateY, scaleX, scaleY)
}

override fun setAudioData(audioData: ShortArray) {
    nativeSetAudioData(audioData)
}

override fun setTouchLocation(x: Float, y: Float) {
    nativeSetTouchLocation(x,y)
}

override fun setGravityXY(x: Float, y: Float){
    nativeSetGravityXY(x,y)
}
```
主要渲染的方式应该是走这里。
onDrawFrame方法就是绘制每一帧。

里面是用：
```Kotlin
/**
    * 当绘制一帧时，这个方法会被 GLSurfaceView 调用。
    * 在这个方法中，我们一定要绘制一些东西，即使只是清空屏幕。
    * 因为，在这个方法返回后，渲染缓冲区会被交换并显示在屏幕上，
    * 如果什么都没画，可能会看到糟糕的闪烁效果。
    */
override fun onDrawFrame(gl: GL10) {
    // 提前赋值，变化水印的bitmap
    if (mSampleType == IMyNativeRendererType.SAMPLE_TYPE_KEY_TIME_WATERMARK_STICKER) {
        // 获取时间水印的bitmap
        val mBitmap: Bitmap = CommonUtils.getTimeWaterBitmap()

        val bytes = mBitmap.byteCount
        val buf = ByteBuffer.allocate(bytes)
        mBitmap.copyPixelsToBuffer(buf)
        val byteArray = buf.array()
        setImageDataWithIndex(
            0,
            ImageFormat.IMAGE_FORMAT_RGBA,
            mBitmap.width,
            mBitmap.height,
            byteArray
        )
    }

    // native层去绘制
    nativeDrawFrame()
}
```
native层去绘制的。

### 1.4 布局
```Kotlin
setContentView(R.layout.activity_native_render)
mRootView = findViewById<View>(R.id.rootView) as ViewGroup

// Tell the surface view we want to create an OpenGL ES 3.0-compatible context,
// and set an OpenGL ES 3.0-compatible renderer.
mGLSurfaceView =
    MyCustomerGLSurfaceView(this, myNativeRenderer, CONTEXT_CLIENT_VERSION)

mGLSurfaceView?.let {
    val lp = RelativeLayout.LayoutParams(
        ViewGroup.LayoutParams.MATCH_PARENT, ViewGroup.LayoutParams.MATCH_PARENT
    )
    lp.addRule(RelativeLayout.CENTER_IN_PARENT)
    mRootView!!.addView(it, lp)

    if (mRootView!!.width != it.width
        || mRootView!!.height != it.height
    ) {
        it.setAspectRatio(mRootView!!.width, mRootView!!.height)
    }
    // 设置渲染模式
    setRenderMode(it)
    // 加载图片
    loadImageToGLSurfaceView()
    // 申请重新绘制
    it.requestRender()
}
```
这里先设置了下ContentView进去。
其实里面没啥的，就一个RelativeLayout：
```xml
<?xml version="1.0" encoding="utf-8"?>
<RelativeLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:id="@+id/rootView"
    android:layout_width="match_parent"
    android:layout_height="match_parent">

</RelativeLayout>
```

然后这里new了一个自定义的MyCustomerGLSurfaceView。
然后在RootView里面添加了这个自定义View。

然后有一个代码很重要：
```Kotlin
 mGLSurfaceView = MyCustomerGLSurfaceView(this, myNativeRenderer, CONTEXT_CLIENT_VERSION)
```
这里将 渲染器作为参数传到这个自定义View里面了。

### 1.5 自定义View
继承GLSurfaceView，来自android.opengl.GLSurfaceView。
```Kotlin
class MyCustomerGLSurfaceView : GLSurfaceView, ScaleGestureDetector.OnScaleGestureListener {
    private lateinit var mRenderer: MyNativeRenderer
    private lateinit var mScaleGestureDetector: ScaleGestureDetector

    private var mPreviousX = 0f
    private var mPreviousY = 0f

    private var mXAngle = 0f
    private var mYAngle = 0f

    private var mRatioWidth = 0
    private var mRatioHeight = 0

    private var mPreScale = 1.0f
    private var mCurScale = 1.0f

    private var mDensity = 0f

    private var mLastMultiTouchTime: Long = 0

    constructor(context: Context?) : super(context) {}

    constructor(context: Context?, attrs: AttributeSet?) : super(context, attrs) {}

    constructor(context: Context?, glRender: MyNativeRenderer, eglContextVersion: Int) : this(
        context,
        null,
        glRender,
        eglContextVersion
    )

    constructor(
        context: Context?,
        attrs: AttributeSet?,
        glRender: MyNativeRenderer,
        eglContextVersion: Int
    ) : super(context, attrs) {
        setEGLContextClientVersion(eglContextVersion)
        mRenderer = glRender

        /*If no setEGLConfigChooser method is called,
        then by default the view will choose an RGB_888 surface with a depth buffer depth of at least 16 bits.*/
        // 最后 2 个参数表示分别配置 16 位的深度缓冲区和模板缓冲区
        setEGLConfigChooser(8, 8, 8, 8, 16, 8)
        setRenderer(mRenderer)
        mScaleGestureDetector = ScaleGestureDetector(context, this)
    }
```

定义触摸事件：
```Kotlin
override fun onTouchEvent(event: MotionEvent): Boolean {
    Log.d(TAG, "onTouchEvent")
    if (event.pointerCount == 1) {
        Log.d(TAG, "event.pointerCount == 1")
        val currentTimeMillis = System.currentTimeMillis()
        if (currentTimeMillis - mLastMultiTouchTime > 200) {
            var x: Float = -1.0f
            var y: Float = -1.0f
            when (event.action) {
                MotionEvent.ACTION_DOWN -> {
                    if (mRenderer.mSampleType == IMyNativeRendererType.SAMPLE_TYPE_KEY_LESSON_FIVE) {
                        // Android的 GLSurfaceView 在后台线程中执行渲染，必须要小心，
                        // 只能在这个渲染线程中调用OpenGL，在Android主线程中使用UI(用户界面)相关的调用
                        // 两个线程之间的通信可以用如下方法：
                        // 在主线程中的 GLSurfaceView实例可以调用 queueEven() 方法传递一个Runnable给后台渲染线程
                        // 渲染线程可以调用Activity的runOnUIThread()来传递事件(event)给主线程

                        // Ensure we call switchMode() on the OpenGL thread.
                        // queueEvent() is a method of GLSurfaceView that will do this for us.
                        queueEvent { mRenderer.switchBlendingMode() }
                        return true
                    }
                }

                MotionEvent.ACTION_MOVE -> {
                    x = event.x
                    y = event.y
                    val deltaX = (x - mPreviousX) / mDensity / 2
                    val deltaY = (y - mPreviousY) / mDensity / 2
                    if (mRenderer.mSampleType == IMyNativeRendererType.SAMPLE_TYPE_KEY_LESSON_SIX) {
                        mRenderer.setDelta(deltaX, deltaY)
                    } else if (mRenderer.mSampleType == IMyNativeRendererType.SAMPLE_TYPE_KEY_SCRATCH_CARD) {
                        mRenderer.setTouchLocation(x, y)
                        // 重新请求绘制
                        requestRender()
                    }

                    val dy = y - mPreviousY
                    val dx = x - mPreviousX
                    mYAngle += (dx * TOUCH_SCALE_FACTOR).toInt()
                    mXAngle += (dy * TOUCH_SCALE_FACTOR).toInt()
                }

                MotionEvent.ACTION_CANCEL -> {
                    x = -1.0f
                    y = -1.0f
                }

                MotionEvent.ACTION_UP->{
                    if (mRenderer.mSampleType == IMyNativeRendererType.SAMPLE_TYPE_KEY_SHOCK_WAVE) {
                        mRenderer.setTouchLocation(event.x, event.y)
                    }
                }
            }

            mPreviousX = x
            mPreviousY = y

            when (mRenderer.mSampleType) {
                IMyNativeRendererType.SAMPLE_TYPE_KEY_FBO_LEG,
                IMyNativeRendererType.SAMPLE_TYPE_KEY_COORD_SYSTEM,
                IMyNativeRendererType.SAMPLE_TYPE_KEY_BASE_LIGHT,
                IMyNativeRendererType.SAMPLE_TYPE_KEY_MULTI_LIGHT,
                IMyNativeRendererType.SAMPLE_TYPE_KEY_STENCIL_TESTING,
                IMyNativeRendererType.SAMPLE_TYPE_KEY_PARTICLE_SYSTEM2,
                IMyNativeRendererType.SAMPLE_TYPE_KEY_INSTANCING,
                IMyNativeRendererType.SAMPLE_TYPE_KEY_PBO,
                IMyNativeRendererType.SAMPLE_TYPE_KEY_UBO,
                IMyNativeRendererType.SAMPLE_TYPE_KEY_TEXT_RENDER,
                IMyNativeRendererType.SAMPLE_TYPE_KEY_3D_MODEL,
                IMyNativeRendererType.SAMPLE_TYPE_KEY_3D_MODEL2,
                IMyNativeRendererType.SAMPLE_TYPE_KEY_BLENDING,
                IMyNativeRendererType.SAMPLE_TYPE_KEY_SKYBOX -> {
                    Log.d(TAG, "updateTransformMatrix")
                    mRenderer.updateTransformMatrix(mXAngle, mYAngle, mCurScale, mCurScale)
                    requestRender()
                }
            }

        }
    } else {
        Log.d(TAG, "event.pointerCount != 1")
        mScaleGestureDetector.onTouchEvent(event)
    }
    return true
}
```

测量方法：
```Kotlin
override fun onMeasure(widthMeasureSpec: Int, heightMeasureSpec: Int) {
    super.onMeasure(widthMeasureSpec, heightMeasureSpec)
    val width = MeasureSpec.getSize(widthMeasureSpec)
    val height = MeasureSpec.getSize(heightMeasureSpec)
    if (0 == mRatioWidth || 0 == mRatioHeight) {
        setMeasuredDimension(width, height)
    } else {
        if (width < height * mRatioWidth / mRatioHeight) {
            setMeasuredDimension(width, width * mRatioHeight / mRatioWidth)
        } else {
            setMeasuredDimension(height * mRatioWidth / mRatioHeight, height)
        }
    }
}
```

其实设置方法：
```Kotlin
fun setAspectRatio(width: Int, height: Int) {
    Log.d(TAG, "setAspectRatio() called with: width = [$width], height = [$height]")
    require(!(width < 0 || height < 0)) { "Size cannot be negative." }
    mRatioWidth = width
    mRatioHeight = height
    requestLayout()
}

// Hides superclass method.
fun setRenderer(renderer: Renderer, density: Float) {
    mRenderer = renderer as MyNativeRenderer
    mDensity = density
    super.setRenderer(renderer)
}
```

定义缩放方法：
```Kotlin
override fun onScale(detector: ScaleGestureDetector?): Boolean {
        Log.d(TAG, "onScale")
    when (mRenderer.mSampleType) {
        IMyNativeRendererType.SAMPLE_TYPE_KEY_COORD_SYSTEM,
        IMyNativeRendererType.SAMPLE_TYPE_KEY_BASE_LIGHT,
        IMyNativeRendererType.SAMPLE_TYPE_KEY_TEXT_RENDER,
        IMyNativeRendererType.SAMPLE_TYPE_KEY_3D_MODEL,
        IMyNativeRendererType.SAMPLE_TYPE_KEY_3D_MODEL2,
        IMyNativeRendererType.SAMPLE_TYPE_KEY_INSTANCING -> {
            val preSpan = detector!!.previousSpan
            val curSpan = detector.currentSpan
            mCurScale = if (curSpan < preSpan) {
                mPreScale - (preSpan - curSpan) / 200
            } else {
                mPreScale + (curSpan - preSpan) / 200
            }
            mCurScale = 0.05f.coerceAtLeast(mCurScale.coerceAtMost(80.0f))
            mRenderer.updateTransformMatrix(mXAngle, mYAngle, mCurScale, mCurScale)
            requestRender()
        }
        else -> {}
    }

    return false
}

override fun onScaleBegin(detector: ScaleGestureDetector?): Boolean {
    Log.d(TAG, "onScaleBegin")
    return true
}

override fun onScaleEnd(detector: ScaleGestureDetector?) {
    Log.d(TAG, "onScaleEnd")
    mPreScale = mCurScale
    mLastMultiTouchTime = System.currentTimeMillis()
}
```

常量定义：
```Kotlin
companion object {
    private const val TOUCH_SCALE_FACTOR = 180.0f / 320
    private const val TAG = "MyCustomerGLSurfaceView"
}
```

### 1.6 C层流程
首先是Java层：
```Kotlin
fun setRenderType(sampleCategoryType: Int, renderSampleType: Int) {
        if (sampleCategoryType == IMyNativeRendererType.SAMPLE_TYPE) {
            mSampleType = renderSampleType
        }
        nativeSetRenderType(sampleCategoryType, renderSampleType)
    }
```
这里走了native方法：
```C++
extern "C"
JNIEXPORT void JNICALL
Java_com_oyp_openglesdemo_render_MyNativeRenderer_nativeSetRenderType(
        JNIEnv *env, jobject thiz, jint sampleCategoryType, jint renderSampleType) {
    MyGLRenderContext::GetInstance()->SetRenderType(sampleCategoryType, renderSampleType);
}
```
这里是JniImpl.cpp类中定义的方法。

然后会走到MyGLRenderContext方法中，有个单例类,先看下怎么声明的类吧：
```C++

#ifndef OPENGLESDEMO_MYGLRENDERCONTEXT_H
#define OPENGLESDEMO_MYGLRENDERCONTEXT_H

#include <GLBaseSample.h>

class MyGLRenderContext
{
    MyGLRenderContext();

    ~MyGLRenderContext();

public:
    void SetRenderType(int sampleCategoryType, int renderSampleType);

    void OnSurfaceCreated(JNIEnv *env, jobject assetManager);

    void OnSurfaceChanged(int width, int height);

    void OnDrawFrame();

    static MyGLRenderContext* GetInstance();

    static void DestroyInstance();

    void SwitchBlendingMode();

    void SetDelta(float x, float y);

    void SetMinFilter(int filter);

    void SetMagFilter(int filter);

    void SetImageData(int format, int width, int height, uint8_t *pData);

    void SetImageDataWithIndex(int index, int format, int width, int height, uint8_t *pData);

    void UpdateTransformMatrix(float d, float d1, float d2, float d3);

    void SetAudioData(short *buffer, int len);

    void SetTouchLocation(float x, float y);

    void SetGravityXY(float x, float y);

private:
    static MyGLRenderContext *m_pContext;
    GLBaseSample *m_pBeforeSample;
    GLBaseSample *m_pCurSample;

    static NativeImage getImage(int format, int width, int height, uint8_t *pData) ;
};

#endif //OPENGLESDEMO_MYGLRENDERCONTEXT_H
```

具体怎么实现呢？
```C++
MyGLRenderContext *MyGLRenderContext::GetInstance() {
//    LOGD("MyGLRenderContext::GetInstance")
    if (m_pContext == nullptr) {
        m_pContext = new MyGLRenderContext();
    }
    return m_pContext;
}

void MyGLRenderContext::DestroyInstance() {
    LOGD("MyGLRenderContext::DestroyInstance")
    if (m_pContext) {
        delete m_pContext;
        m_pContext = nullptr;
    }

}
```
这里具体实现了GetInstance和DestoryInstance方法。

构造函数呢？
```C++
MyGLRenderContext::MyGLRenderContext() {
    LOGD("MyGLRenderContext::MyGLRenderContext")
    m_pCurSample = nullptr;
    m_pBeforeSample = nullptr;
}

MyGLRenderContext::~MyGLRenderContext() {
    LOGD("MyGLRenderContext::~MyGLRenderContext")

    if (m_pCurSample) {
        m_pCurSample->Shutdown();
        delete m_pCurSample;
        m_pCurSample = nullptr;
    }

    if (m_pBeforeSample) {
        m_pBeforeSample->Shutdown();
        delete m_pBeforeSample;
        m_pBeforeSample = nullptr;
    }
}
```

需要关注一下这里面有个指针变量，GLBaseSample，是什么呢？
目测是一个记录demo如何绘制顶点着色器的一个类。
```C++

// 注意，这个目录在java层创建，参考 com.oyp.openglesdemo.activity.NativeRenderActivity.onResume方法
#define DEFAULT_OGL_ASSETS_DIR "/data/data/com.oyp.openglesdemo/cache"

class GLBaseSample {
    
public:
    GLBaseSample() {
        VERTEX_SHADER = GL_NONE;
        FRAGMENT_SHADER = GL_NONE;
        m_ProgramObj = 0;
        m_Width = 0;
        m_Height = 0;
    }

    virtual ~GLBaseSample() {}

    virtual void Create() = 0;

    virtual void Change(int width, int height) {
        LOGD("Change() width = %d , height = %d\n", width, height)
        m_Width = width;
        m_Height = height;
        // Set the viewport
        // 通知OpenGL ES 用于绘制的2D渲染表面的原点、宽度和高度。
        // 在OpenGL ES 中，视口(Viewport) 定义所有OpenGL ES 渲染操作最终显示的2D矩形
        // 视口(Viewport) 由原点坐标(x,y)和宽度(width) 、高度(height)定义。
        glViewport(0, 0, m_Width, m_Height);
    }

    virtual void Draw() = 0;

    virtual void Shutdown(){
        if (m_ProgramObj) {
            glDeleteProgram(m_ProgramObj);
            m_ProgramObj = GL_NONE;
        }
        if(VERTEX_SHADER != nullptr){
            delete[] VERTEX_SHADER;
            VERTEX_SHADER = nullptr;
        }
        if(FRAGMENT_SHADER!= nullptr){
            delete[] FRAGMENT_SHADER;
            FRAGMENT_SHADER = nullptr;
        }
    }

    // 默认啥都不做，等待有需要的子类去重写
    virtual void SwitchBlendingMode() {}

    virtual void SetDelta(float x, float y) {}

    virtual void SetMinFilter(int filter) {}

    virtual void SetMagFilter(int filter) {}

    virtual void LoadImage(NativeImage *pImage) {};

    virtual void LoadMultiImageWithIndex(int index, NativeImage *pImage) {}

    virtual void UpdateTransformMatrix(float rotateX, float rotateY, float scaleX, float scaleY) {}

    virtual void LoadAudioData(short *buffer, int len) {}

    virtual void SetTouchLocation(float x, float y) {}

    virtual void SetGravityXY(float x, float y) {}

protected:
    /**
     * 程序对象
     */
    GLuint m_ProgramObj;

    /**
     * 顶点着色器
     */
    const char *VERTEX_SHADER;
    /**
     * 片段着色器脚本
     */
    const char *FRAGMENT_SHADER;

    /**
     * 屏幕宽度
     */
    int m_Width;
    /**
     * 屏幕高度
     */
    int m_Height;
};

#endif //OPENGLESDEMO_GLBASESAMPLE_H
```

然后回到MyGLRenderContext中。
具体看下setRenderType如何实现的吧？
```C++
void MyGLRenderContext::SetRenderType(int sampleCategoryType, int renderSampleType) {
    LOGD("MyGLRenderContext::SetRenderType sampleCategoryType = %d, renderSampleType = %d",
         sampleCategoryType, renderSampleType)

    if (sampleCategoryType == SAMPLE_TYPE) {
        m_pBeforeSample = m_pCurSample;

        LOGD("MyGLRenderContext::SetRenderType 0 m_pBeforeSample = %p", m_pBeforeSample)

        switch (renderSampleType) {
            case SAMPLE_TYPE_KEY_TRIANGLE:
                m_pCurSample = new NativeTriangle();
                break;
            case SAMPLE_TYPE_KEY_TRIANGLE2:
                m_pCurSample = new NativeTriangle2();
                break;
            case SAMPLE_TYPE_KEY_TRIANGLE3:
                m_pCurSample = new NativeTriangle3();
                break;
            case SAMPLE_TYPE_KEY_TRIANGLE_MAP_BUFFERS:
                m_pCurSample = new NativeTriangleMapBuffers();
                break;
            case SAMPLE_TYPE_KEY_TRIANGLE_VERTEX_ARRAY_OBJECT:
                m_pCurSample = new NativeTriangleVAO();
                break;
            case SAMPLE_TYPE_KEY_TRIANGLE_VERTEX_BUFFER_OBJECT:
                m_pCurSample = new NativeTriangleVBO();
                break;
            case SAMPLE_TYPE_KEY_CUBE_SIMPLE_VERTEX_SHADER:
                m_pCurSample = new NativeCubeSimpleVertexShader();
                break;
            case SAMPLE_TYPE_KEY_SIMPLE_TEXTURE_2D:
                m_pCurSample = new SimpleTexture2D();
                break;
            case SAMPLE_TYPE_KEY_SIMPLE_TEXTURE_CUBE_MAP:
                m_pCurSample = new SimpleTextureCubeMap();
                break;
            case SAMPLE_TYPE_KEY_MIPMAP_2D:
                m_pCurSample = new MipMap2D();
                break;
            case SAMPLE_TYPE_KEY_TEXTURE_WRAP:
                m_pCurSample = new TextureWrap();
                break;
            case SAMPLE_TYPE_KEY_MULTI_TEXTURE:
                m_pCurSample = new MultiTexture();
                break;
            case SAMPLE_TYPE_KEY_PARTICLE_SYSTEM:
                m_pCurSample = new ParticleSystem();
                break;
            case SAMPLE_TYPE_KEY_PARTICLE_SYSTEM_TRANSFORM_FEEDBACK:
                m_pCurSample = new ParticleSystemTransformFeedBack();
                break;
            case SAMPLE_TYPE_KEY_NOISE3D:
                m_pCurSample = new Noise3DRender();
                break;
            case SAMPLE_TYPE_KEY_MRT:
                m_pCurSample = new MRT();
                break;
            case SAMPLE_TYPE_KEY_TERRAIN_RENDER:
                m_pCurSample = new TerrainRender();
                break;
            case SAMPLE_TYPE_KEY_SHADOWS:
                m_pCurSample = new Shadows();
                break;
            case SAMPLE_TYPE_KEY_LESSON_ONE:
                m_pCurSample = new Native1Lesson();
                break;
            case SAMPLE_TYPE_KEY_LESSON_TWO:
                m_pCurSample = new Native2Lesson();
                break;
            case SAMPLE_TYPE_KEY_LESSON_THREE:
                m_pCurSample = new Native3Lesson();
                break;
            case SAMPLE_TYPE_KEY_LESSON_FOUR:
                m_pCurSample = new Native4Lesson();
                break;
            case SAMPLE_TYPE_KEY_LESSON_FIVE:
                m_pCurSample = new Native5Lesson();
                break;
            case SAMPLE_TYPE_KEY_LESSON_SIX:
                m_pCurSample = new Native6Lesson();
                break;
            case SAMPLE_TYPE_KEY_TEXTURE_MAP:
                m_pCurSample = new TextureMapSample();
                break;
            case SAMPLE_TYPE_KEY_YUV_RENDER:
                m_pCurSample = new NV21TextureMapSample();
                break;
            case SAMPLE_TYPE_KEY_FBO:
                m_pCurSample = new FBOSample();
                break;
            case SAMPLE_TYPE_KEY_FBO_LEG:
                m_pCurSample = new FBOLegLengthenSample();
                break;
            case SAMPLE_TYPE_COORD_SYSTEM:
                m_pCurSample = new CoordSystemSample();
                break;
            case SAMPLE_TYPE_KEY_BASE_LIGHT:
                m_pCurSample = new BasicLightingSample();
                break;
            case SAMPLE_TYPE_KEY_MULTI_LIGHT:
                m_pCurSample = new MultiLightingsSample();
                break;
            case SAMPLE_TYPE_KEY_INSTANCING:
                m_pCurSample = new Instancing3DSample();
                break;
            case SAMPLE_TYPE_KEY_STENCIL_TESTING:
                m_pCurSample = new StencilTestingSample();
                break;
            case SAMPLE_TYPE_KEY_BLENDING:
                m_pCurSample = new BlendingSample();
                break;
            case SAMPLE_TYPE_KEY_PARTICLE_SYSTEM2:
                m_pCurSample = new ParticlesSample2();
                break;
            case SAMPLE_TYPE_KEY_SKYBOX:
                m_pCurSample = new SkyBoxSample();
                break;
            case SAMPLE_TYPE_KEY_PBO:
                m_pCurSample = new PBOSample();
                break;
            case SAMPLE_TYPE_KEY_SHADER_TOY_BEATING_HEART:
                m_pCurSample = new BaseShaderToySimpleSample(
                        SAMPLE_TYPE_KEY_SHADER_TOY_BEATING_HEART);
                break;
            case SAMPLE_TYPE_KEY_SHADER_TOY_CLOUD:
                m_pCurSample = new BaseShaderToySimpleSample(SAMPLE_TYPE_KEY_SHADER_TOY_CLOUD);
                break;
            case SAMPLE_TYPE_KEY_SHADER_TOY_TIME_TUNNEL:
                m_pCurSample = new TimeTunnelSample();
                break;
            case SAMPLE_TYPE_KEY_SHADER_TOY_MAIN_SEQUENCE_STAR:
                m_pCurSample = new BaseShaderToySimpleSample(
                        SAMPLE_TYPE_KEY_SHADER_TOY_MAIN_SEQUENCE_STAR);
                break;
            case SAMPLE_TYPE_KEY_SHADER_TOY_SKY_PATH:
                m_pCurSample = new BaseShaderToySimpleSample(SAMPLE_TYPE_KEY_SHADER_TOY_SKY_PATH);
                break;
            case SAMPLE_TYPE_KEY_SHADER_TOY_A_DAY:
                m_pCurSample = new BaseShaderToySimpleSample(SAMPLE_TYPE_KEY_SHADER_TOY_A_DAY);
                break;
            case SAMPLE_TYPE_KEY_SHADER_TOY_ATMOSPHERE_SYSTEM_TEST:
                m_pCurSample = new BaseShaderToySimpleSample(
                        SAMPLE_TYPE_KEY_SHADER_TOY_ATMOSPHERE_SYSTEM_TEST);
                break;
            case SAMPLE_TYPE_KEY_BEZIER_CURVE:
                m_pCurSample = new BezierCurveSample();
                break;
            case SAMPLE_TYPE_KEY_BIG_EYES:
                m_pCurSample = new BigEyesSample();
                break;
            case SAMPLE_TYPE_KEY_FACE_SLENDER:
                m_pCurSample = new FaceSlenderSample();
                break;
            case SAMPLE_TYPE_KEY_BIG_HEAD:
                m_pCurSample = new BigHeadSample();
                break;
            case SAMPLE_TYPE_KEY_RATARY_HEAD:
                m_pCurSample = new RotaryHeadSample();
                break;
            case SAMPLE_TYPE_KEY_VISUALIZE_AUDIO:
                m_pCurSample = new VisualizeAudioSample();
                break;
            case SAMPLE_TYPE_KEY_SCRATCH_CARD:
                m_pCurSample = new ScratchCardSample();
                break;
            case SAMPLE_TYPE_KEY_AVATAR:
                m_pCurSample = new AvatarSample();
                break;
            case SAMPLE_TYPE_KEY_SHOCK_WAVE:
                m_pCurSample = new ShockWaveSample();
                break;
            case SAMPLE_TYPE_KEY_MRT2:
                m_pCurSample = new MRTSample();
                break;
            case SAMPLE_TYPE_KEY_FBO_BLIT:
                m_pCurSample = new FBOBlitSample();
                break;
            case SAMPLE_TYPE_KEY_UBO:
                m_pCurSample = new UniformBufferSample();
                break;
            case SAMPLE_TYPE_KEY_RGB2YUV:
                m_pCurSample = new RGB2YUVSample();
                break;
            case SAMPLE_TYPE_KEY_MULTI_THREAD_RENDER:
                m_pCurSample = new SharedEGLContextSample();
                break;
            case SAMPLE_TYPE_KEY_TEXT_RENDER:
                m_pCurSample = new TextRenderSample();
                break;
            case SAMPLE_TYPE_KEY_STAY_COLOR:
                m_pCurSample = new PortraitStayColorExample();
                break;
            case SAMPLE_TYPE_KEY_TRANSITIONS_1:
            case SAMPLE_TYPE_KEY_TRANSITIONS_2:
            case SAMPLE_TYPE_KEY_TRANSITIONS_3:
            case SAMPLE_TYPE_KEY_TRANSITIONS_4:
            case SAMPLE_TYPE_KEY_TRANSITIONS_5:
            case SAMPLE_TYPE_KEY_TRANSITIONS_6:
            case SAMPLE_TYPE_KEY_TRANSITIONS_7:
            case SAMPLE_TYPE_KEY_TRANSITIONS_8:
            case SAMPLE_TYPE_KEY_TRANSITIONS_9:
            case SAMPLE_TYPE_KEY_TRANSITIONS_10:
            case SAMPLE_TYPE_KEY_TRANSITIONS_11:
            case SAMPLE_TYPE_KEY_TRANSITIONS_12:
            case SAMPLE_TYPE_KEY_TRANSITIONS_13:
            case SAMPLE_TYPE_KEY_TRANSITIONS_14:
            case SAMPLE_TYPE_KEY_TRANSITIONS_15:
            case SAMPLE_TYPE_KEY_TRANSITIONS_16:
            case SAMPLE_TYPE_KEY_TRANSITIONS_17:
            case SAMPLE_TYPE_KEY_TRANSITIONS_18:
            case SAMPLE_TYPE_KEY_TRANSITIONS_19:
            case SAMPLE_TYPE_KEY_TRANSITIONS_20:
            case SAMPLE_TYPE_KEY_TRANSITIONS_21:
                m_pCurSample = new GLTransitionExample(renderSampleType);
                break;
            case SAMPLE_TYPE_KEY_3D_MODEL:
                m_pCurSample = new Model3DSample();
                break;
            case SAMPLE_TYPE_KEY_3D_MODEL2:
                m_pCurSample = new Model3DSample2();
                break;
            case SAMPLE_TYPE_KEY_AIR_HOCKEY:
                m_pCurSample = new AirHockeySample();
                break;

            case SAMPLE_TYPE_KEY_RECTANGLE:
                m_pCurSample = new NativeRectangle();
                break;

            case SAMPLE_TYPE_KEY_STICKER:
                m_pCurSample = new StickerSample();
                break;
            case SAMPLE_TYPE_KEY_TIME_WATERMARK_STICKER:
                m_pCurSample = new TimeWatermarkStickerSample();
                break;

            case SAMPLE_TYPE_KEY_GREEN_SCREEN_MATTING:
                m_pCurSample = new GreenScreenMatting();
                break;
            case SAMPLE_TYPE_KEY_GREEN_SCREEN_MATTING_MIX:
                m_pCurSample = new GreenScreenMattingMix();
                break;
            case SAMPLE_TYPE_KEY_ROTATE_TEXTURE:
                m_pCurSample = new RotateTexture();
                break;
            default:
                m_pCurSample = nullptr;
                break;
        }
        if (m_pCurSample == nullptr) {
            throw MyGLException(
                    "MyGLRenderContext::SetRenderType() 请注意：你应该忘记初始化你要展示的Sample类型 ，请补上初始化的代码，否则无法渲染");
        }
        LOGD("MyGLRenderContext::SetRenderType m_pBeforeSample = %p, m_pCurSample=%p",
             m_pBeforeSample, m_pCurSample)
    }
}
```
这里是所有类型，不过我们重点是看三角形。
三角形是这样赋值过去的。
```C++
switch (renderSampleType) {
    case SAMPLE_TYPE_KEY_TRIANGLE:
        m_pCurSample = new NativeTriangle();
        break;
```
这里new了一个NativeTriangle给到m_pCurSample指针变量。
这个m_pCurSample就是前面在MyGLRenderContext定义的两个指针变量的其中一个。
类型是GLBaseSample这个的指针类型。

### 1.7 三角形绘制方法
这里先声明头文件：
```C++
#pragma once

#include <GLBaseSample.h>

class NativeTriangle : public GLBaseSample {

#define VERTEX_POS_INDX       0

public:
    NativeTriangle() = default;

    virtual ~NativeTriangle() = default;

    virtual void Create();

    virtual void Draw();

    virtual void Shutdown();
};
```
这里看到了这个是必须继承我们的GLBaseSample，但不是绘制三角形必要，是我们为了统一管理demo，新建的一个Sample来，其它demo都继承这个，就比较好管理。

然后再写cpp文件，去实现这个三角形：
```C++
#include "NativeTriangle.h"

// 可以参考这篇讲解： https://learnopengl-cn.github.io/01%20Getting%20started/04%20Hello%20Triangle/
// 我们在OpenGL中指定的所有坐标都是3D坐标（x、y和z）
// 由于我们希望渲染一个三角形，我们一共要指定三个顶点，每个顶点都有一个3D位置。
// 我们会将它们以标准化设备坐标的形式（OpenGL的可见区域）定义为一个float数组。
// https://learnopengl-cn.github.io/img/01/04/ndc.png

// https://developer.android.com/guide/topics/graphics/opengl#kotlin
// 在 OpenGL 中，形状的面是由三维空间中的三个或更多点定义的表面。
// 一个包含三个或更多三维点（在 OpenGL 中被称为顶点）的集合具有一个正面和一个背面。
// 如何知道哪一面为正面，哪一面为背面呢？这个问题问得好！答案与环绕（即您定义形状的点的方向）有关。
// 查看图片 ： https://developer.android.com/images/opengl/ccw-winding.png
// 或者查看本地图片：Android_Java/Chapter_2/Hello_Triangle/ccw-winding.png
// 在此示例中，三角形的点按照使它们沿逆时针方向绘制的顺序定义。
// 这些坐标的绘制顺序定义了该形状的环绕方向。默认情况下，在 OpenGL 中，沿逆时针方向绘制的面为正面。
// 因此您看到的是该形状的正面（根据 OpenGL 解释），而另一面是背面。
//
// 知道形状的哪一面为正面为何如此重要呢？
// 答案与 OpenGL 的“面剔除”这一常用功能有关。
// 面剔除是 OpenGL 环境的一个选项，它允许渲染管道忽略（不计算或不绘制）形状的背面，从而节省时间和内存并缩短处理周期：
static GLfloat vVertices[] = {
        // 逆时针 三个顶点
        0.0f, 0.5f, 0.0f,            // 上角
        -0.5f, -0.5f, 0.0f,          // 左下角
        0.5f, -0.5f, 0.0f            // 右下角
};
```
这里先定义下3个坐标。这个坐标以屏幕中心为（0,0）,坐标跟常规的数学坐标一致。

然后是一个Create方法，具体会在MyGLRenderContext的onSurfaceCreated中会执行
```C++
void MyGLRenderContext::OnSurfaceCreated(JNIEnv *env, jobject assetManager) {
    LOGD("MyGLRenderContext::OnSurfaceCreated")

    // 初始化设置assetManager  一定要记得初始化，否则会报空指针异常
    GLUtils::setEnvAndAssetManager(env, assetManager);

    if (m_pBeforeSample) {
        m_pBeforeSample->Shutdown();
        delete m_pBeforeSample;
        m_pBeforeSample = nullptr;
    }

    // 就是这里会走Create
    if (m_pCurSample) {
        m_pCurSample->Create();
    }
}
```

三角形的Create方法怎么写呢？
```C++
void NativeTriangle::Create() {
    GLUtils::printGLInfo();

    // Main Program
    // 顶点着色器
    VERTEX_SHADER = GLUtils::openTextFile(
            "vertex/vertex_shader_hello_triangle.glsl");
    // 片段着色器
    FRAGMENT_SHADER = GLUtils::openTextFile(
            "fragment/fragment_shader_hello_triangle.glsl");

    m_ProgramObj = GLUtils::createProgram(&VERTEX_SHADER, &FRAGMENT_SHADER);
    if (!m_ProgramObj) {
        LOGD("Could not Create program")
        return;
    }
    // 设置清除颜色
    glClearColor(1.0f, 1.0f, 1.0f, 0.0f);
}
```
这里主要是初始化，并没有绘制相关信息。

主要绘制发生于Draw方法：
```C++
void NativeTriangle::Draw() {
    // Clear the color buffer
    // 清除屏幕
    // 在OpenGL ES中，绘图中涉及多种缓冲区类型：颜色、深度、模板。
    // 这个例子，绘制三角形，只向颜色缓冲区中绘制图形。在每个帧的开始，我们用glClear函数清除颜色缓冲区
    // 缓冲区将用glClearColor指定的颜色清除。
    // 这个例子，我们调用了GLES30.glClearColor(1.0f, 1.0f, 1.0f, 0.0f); 因此屏幕清为白色。
    // 清除颜色应该由应用程序在调用颜色缓冲区的glClear之前设置。
    glClear(GL_COLOR_BUFFER_BIT);

    // Use the program object
    // 在glUseProgram函数调用之后，每个着色器调用和渲染调用都会使用这个程序对象（也就是之前写的着色器)了。
    // 当我们渲染一个物体时要使用着色器程序 , 将其设置为活动程序。这样就可以开始渲染了
    glUseProgram(m_ProgramObj);

    // Load the vertex data
    //  顶点着色器允许我们指定任何以顶点属性为形式的输入。这使其具有很强的灵活性的同时，
    //  它还的确意味着我们必须手动指定输入数据的哪一个部分对应顶点着色器的哪一个顶点属性。
    //  所以，我们必须在渲染前指定OpenGL该如何解释顶点数据。

    //  我们的顶点缓冲数据会被解析为下面这样子：https://learnopengl-cn.github.io/img/01/04/vertex_attribute_pointer.png
    //   . 位置数据被储存为32位（4字节）浮点值。
    //   . 每个位置包含3个这样的值。
    //   . 在这3个值之间没有空隙（或其他值）。这几个值在数组中紧密排列(Tightly Packed)。
    //   . 数据中第一个值在缓冲开始的位置。

    // 有了这些信息我们就可以使用glVertexAttribPointer函数告诉OpenGL该如何解析顶点数据（应用到逐个顶点属性上）了：
    // Load the vertex data

    // 第一个参数指定我们要配置的顶点属性。因为我们希望把数据传递到这一个顶点属性中，所以这里我们传入0。
    // 第二个参数指定顶点属性的大小。顶点属性是一个vec3，它由3个值组成，所以大小是3。
    // 第三个参数指定数据的类型，这里是GL_FLOAT(GLSL中vec*都是由浮点数值组成的)。
    // 第四个参数定义我们是否希望数据被标准化(Normalize)。如果我们设置为GL_TRUE，所有数据都会被映射到0（对于有符号型signed数据是-1）到1之间。我们把它设置为GL_FALSE。
    // 第五个参数叫做步长(Stride)，它告诉我们在连续的顶点属性组之间的间隔。我们设置为0来让OpenGL决定具体步长是多少（只有当数值是紧密排列时才可用）。
    //      一旦我们有更多的顶点属性，我们就必须更小心地定义每个顶点属性之间的间隔，
    //      （译注: 这个参数的意思简单说就是从这个属性第二次出现的地方到整个数组0位置之间有多少字节）。
    // 最后一个参数的类型是void*，所以需要我们进行这个奇怪的强制类型转换。它表示位置数据在缓冲中起始位置的偏移量(Offset)。
    glVertexAttribPointer(VERTEX_POS_INDX, 3, GL_FLOAT, GL_FALSE, 0, vVertices);

    // 现在我们已经定义了OpenGL该如何解释顶点数据，
    // 我们现在应该使用glEnableVertexAttribArray，以顶点属性位置值作为参数，启用顶点属性；顶点属性默认是禁用的。
    glEnableVertexAttribArray(VERTEX_POS_INDX);

    // glDrawArrays函数第一个参数是我们打算绘制的OpenGL图元的类型。我们希望绘制的是一个三角形，这里传递GL_TRIANGLES给它。
    // 第二个参数指定了顶点数组的起始索引，我们这里填0。
    // 最后一个参数指定我们打算绘制多少个顶点，这里是3（我们只从我们的数据中渲染一个三角形，它只有3个顶点长）。
    //        public static final int GL_POINTS                                  = 0x0000;
    //        public static final int GL_LINES                                   = 0x0001;
    //        public static final int GL_LINE_LOOP                               = 0x0002;
    //        public static final int GL_LINE_STRIP                              = 0x0003;
    //        public static final int GL_TRIANGLES                               = 0x0004;
    //        public static final int GL_TRIANGLE_STRIP                          = 0x0005;
    //        public static final int GL_TRIANGLE_FAN                            = 0x0006;
    glDrawArrays(GL_TRIANGLES, 0, 3);

    // 禁用 通用顶点属性数组
    glDisableVertexAttribArray(0);
}
```

主要是glVertexAttribPointer中传入了顶点坐标。
但这里还不能显示，需要解释一下顶点数据，怎么绘制顶点等。
这里传入了一个GL_TRIANGLES，系统就知道这三个点用来绘制三角形哦。

### 1.8 三角形颜色怎么设置
其实有多种方法。
方法1：可以在顶点后面直接加红绿蓝颜色值。
方法2：在片段着色器里面加。
方法3：可以通过特殊手段传值给着色器。

这里我们看下第2种，直接在片段着色器中如何修改颜色。

首先确认下在哪里加载片段主色器的。
```C++
void NativeRectangle::Create() {
    GLUtils::printGLInfo();

    // Main Program
    // 顶点着色器
    VERTEX_SHADER = GLUtils::openTextFile(
            "vertex/vertex_shader_hello_triangle.glsl");
    // 片段着色器
    FRAGMENT_SHADER = GLUtils::openTextFile(
            "fragment/fragment_shader_hello_triangle.glsl");

    m_ProgramObj = GLUtils::createProgram(&VERTEX_SHADER, &FRAGMENT_SHADER);
    if (!m_ProgramObj) {
        LOGD("Could not Create program")
        return;
    }
```
soga，原来在这里，create方法中，定义了片段着色器fragment/fragment_shader_hello_triangle.glsl。
顶点主色器是同理，应该是顶点的颜色。

目标位置在app/src/main/assets/fragment文件夹下，原来藏这里了。
```glsl
#version 300 es
// 表示OpenGL ES着色器语言V3.00

// 声明着色器中浮点变量的默认精度
precision mediump float;
// 声明一个输出变量fragColor，这是一个4分量的向量，
// 写入这个变量的值将被输出到颜色缓冲器
out vec4 fragColor;

void main()
{
	//在计算机图形中颜色被表示为有4个元素的数组：红色、绿色、蓝色和alpha(透明度)分量，
	//通常缩写为RGBA。当在OpenGL或GLSL中定义一个颜色的时候，
	//我们把颜色每个分量的强度设置在0.0到1.0之间。

	//比如说我们设置红为1.0f，绿为1.0f，我们会得到两个颜色的混合色，即黄色。
	//这三种颜色分量的不同调配可以生成超过1600万种不同的颜色！

	// 所有片段的着色器输出都是红色( 1.0, 0.0, 0.0, 1.0 )
	fragColor = vec4 ( 1.0, 0.0, 0.0, 1.0 );

	// 会输出橘黄色
	// fragColor = vec4(1.0f, 0.5f, 0.2f, 1.0f);
}
```
这个是着色器语言，决定怎么显示颜色。
这个着色器语言这么加载到程序的呢？
继续看上面的Create方法：
```C++
  m_ProgramObj = GLUtils::createProgram(&VERTEX_SHADER, &FRAGMENT_SHADER);
    if (!m_ProgramObj) {
        LOGD("Could not Create program")
        return;
    }
```
这里就加载到当前程序了。包括了顶点着色器和片段着色器。

整个流程基本就是这样了，另外别忘了一个指针置空的问题。
我们这里每个demo会走一个Shutdown方法，这里就是关闭的时候进行一些数据回收操作。


## 2 总结
* 首先Java层需要自定义一个View，继承GLSurfaceView，需要实现必须实现的方法。主要先在构造函数中进行必要的初始化，然后再设置Renderer，这个的Renderer是我们自己定义的一个封装好的类，这里面加载so库，方法c层的外观者类。

* 在外观者类定义好Surface初始化和DrawFrame的底层方法，定义好怎么初始化，怎么绘制的方法。再自定义View中调用，实现在底层实现。

* 可以通过alt+enter一键生成一个生成名字的方法，参数有JNIEnv的指针变量，可以访问jni的关键实例。然后这里面走我们自定义单例MyGLRenderContext的onDrawFrame或者其它方法。

* 然后我们在MyGLRenderContext声明了一个demo指针变量，可以让这个demo执行目标函数，比如onSurfaceCreated方法，就命令里面的m_pCurSample走Create方法。

* 因为外部先走setRenderType，所以我们在这里给m_pCurSample指针new对象。

* 比如绘制三角形，就创建一个NativeTriangle,当然继承demo类，然后定义一个坐标集合，三角形就按照数学坐标定义3个点。然后create方法里面加载我们的着色器，着色器代码可以放在项目的assets文件夹下。然后让GLUtils创建程序。

* 最后就是在自定义View的onDrawFrame绘制每一帧了。这里如果是三角形，我们就按照前面定义的坐标集合来绘制三角形即可。



