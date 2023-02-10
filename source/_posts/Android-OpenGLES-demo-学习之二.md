---
title: Android OpenGLES demo 学习之二
date: 2023-02-08 09:56:49
top: false
cover: false
toc: true
mathjax: true
tags:
- Android OpenGLES
categories:
- Android
---

## 1 Demo2 展示一个蓝色三角形

### 1.1 效果
<img src=open_01.png width=50%>

### 1.2 首页传参
```Kotlin
val itemHelloTriangle2: MutableMap<String, Any?> = HashMap()
    itemHelloTriangle2[ITEM_IMAGE] = R.mipmap.ic_triangle2
    itemHelloTriangle2[ITEM_TITLE] = "展示一个基本的蓝色三角形"
    itemHelloTriangle2[ITEM_SUBTITLE] = "颜色由glVertexAttrib4fv传给片段着色器"
    data.add(itemHelloTriangle2)
    typeMapping.put(i++, IMyNativeRendererType.SAMPLE_TYPE_TRIANGLE2)
   
```

这里的type为IMyNativeRendererType.SAMPLE_TYPE_TRIANGLE2,实际上是101。

```Kotlin 
val launchIntent = Intent(this@MainActivity, NativeRenderActivity::class.java)
launchIntent.putExtra(IMyNativeRendererType.RENDER_TYPE, type)
startActivity(launchIntent)
```
这里把type传到NativeRenderActivity了。

### 1.3 展示页
内部onCreate中 new 了一个 MyNativeRenderer
```Kotlin
renderer = MyNativeRenderer(this)
renderer?.let { myNativeRenderer ->
        myNativeRenderer.setRenderType(IMyNativeRendererType.SAMPLE_TYPE, type)
```
这里传给MyNativeRenderer了。

然后通过一个自定义GLSurfaceView来展示我们的图案。

### 1.4 渲染类
然后这个渲染类先走：
```Kotlin
override fun onSurfaceCreated(gl: GL10, config: EGLConfig) {
    val assetManager: AssetManager = mActivity.assets
    nativeSurfaceCreate(assetManager)
}
```

再走这个：
```Kotlin
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
主要Java跟C的交互同时通过渲染类MyNativeRenderer来完成。

### 1.5 底层对接
首先type通过Java层定义的native函数，传过来。
```Kotlin
private external fun nativeSetRenderType(sampleCategoryType: Int, renderSampleType: Int)
```

然后走到cpp文件：
```C
extern "C"
JNIEXPORT void JNICALL
Java_com_oyp_openglesdemo_render_MyNativeRenderer_nativeSetRenderType(
        JNIEnv *env, jobject thiz, jint sampleCategoryType, jint renderSampleType) {
    MyGLRenderContext::GetInstance()->SetRenderType(sampleCategoryType, renderSampleType);
}
```

demo指针赋值：
```C
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
```

这里new了一个NativeTriangle2()对象。

### 1.6 demo类定义
首先是顶点定义：
```C
#include "NativeTriangle2.h"

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

// 设置顶点的颜色值  这里设置成蓝色
static GLfloat color[4] = {0.0f, 0.0f, 1.0f, 1.0f};
```

然后创建顶点和片段主色器
```C
void NativeTriangle2::Create() {
    GLUtils::printGLInfo();

    // 顶点着色器
    VERTEX_SHADER = GLUtils::openTextFile(
            "vertex/vertex_shader_hello_triangle2.glsl");
    // 片段着色器
    FRAGMENT_SHADER = GLUtils::openTextFile(
            "fragment/fragment_shader_hello_triangle2.glsl");
    m_ProgramObj = GLUtils::createProgram(&VERTEX_SHADER, &FRAGMENT_SHADER);
    if (!m_ProgramObj) {
        LOGD("Could not Create program")
        return;
    }
    // 设置清除颜色
    glClearColor(1.0f, 1.0f, 1.0f, 0.0f);
}
```

对应顶点着色器代码：
```glsl
#version 300 es
// 位置变量的属性位置值为 0
layout(location = 0) in vec4 a_position;
// 颜色变量的属性位置值为 1
layout(location = 1) in vec4 a_color;
// 向片段着色器输出一个颜色
out vec4 v_color;

void main()
{
    v_color = a_color;
    gl_Position = a_position;
}
```
其中，in关键字表示从应用程序中传递给着色器的输入变量；uniform关键字表示变量的值在渲染期间不会改变；out关键字表示从着色器传递给片段着色器的输出变量；main函数定义了着色器的主要处理逻辑。
layout(location = 0)是GLSL中的语法，用于定义顶点着色器的输入变量的位置。其中location参数指定了该输入变量的位置，即其在顶点数组中的索引。

GLSL顶点着色器的实现流程包括以下步骤：

1.定义顶点数据：定义每个顶点的位置、颜色、纹理坐标等数据。
2.编写顶点着色器代码：定义顶点坐标、颜色、纹理坐标等属性，并实现顶点变换。
3.传递数据到顶点着色器：使用OpenGL函数将顶点数据传递到顶点着色器。
4.进行顶点变换：使用顶点着色器代码实现顶点的变换。
5.输出顶点数据：顶点着色器输出变换后的顶点数据。
6.传递数据到片元着色器：使用OpenGL函数将顶点数据传递到片元着色器。
7.在片元着色器中渲染：使用片元着色器代码渲染图形。

这些步骤构成了一次完整的GLSL顶点着色器渲染过程。

对应片段着色器代码：
```glsl
#version 300 es
// 表示OpenGL ES着色器语言V3.00

// 声明着色器中浮点变量的默认精度
precision mediump float;

// 声明由上一步顶点着色器传入进来的颜色值
in vec4 v_color;

// 声明一个输出变量fragColor，这是一个4分量的向量，
// 写入这个变量的值将被输出到颜色缓冲器
out vec4 o_fragColor;

void main()
{
	o_fragColor = v_color;
}
```

如何绘制：
```C

void NativeTriangle2::Draw() {
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

    //  指定通用顶点属性数组
    // 第一个参数指定我们要配置的顶点属性。因为我们希望把数据传递到这一个顶点属性中，所以这里我们传入0。
    // 第二个参数指定顶点属性的大小。顶点属性是一个vec3，它由3个值组成，所以大小是3。
    // 第三个参数指定数据的类型，这里是GL_FLOAT(GLSL中vec*都是由浮点数值组成的)。
    // 第四个参数定义我们是否希望数据被标准化(Normalize)。如果我们设置为GL_TRUE，所有数据都会被映射到0（对于有符号型signed数据是-1）到1之间。我们把它设置为GL_FALSE。
    // 第五个参数叫做步长(Stride)，它告诉我们在连续的顶点属性组之间的间隔。我们设置为0来让OpenGL决定具体步长是多少（只有当数值是紧密排列时才可用）。
    //      一旦我们有更多的顶点属性，我们就必须更小心地定义每个顶点属性之间的间隔，
    //      （译注: 这个参数的意思简单说就是从这个属性第二次出现的地方到整个数组0位置之间有多少字节）。
    // 最后一个参数的类型是void*，所以需要我们进行这个奇怪的强制类型转换。它表示位置数据在缓冲中起始位置的偏移量(Offset)。
    glVertexAttribPointer(VERTEX_POS_INDX, VERTEX_POS_SIZE, GL_FLOAT, GL_FALSE, 0, vVertices);

    // 现在我们已经定义了OpenGL该如何解释顶点数据，
    // 我们现在应该使用glEnableVertexAttribArray，以顶点属性位置值作为参数，启用顶点属性；顶点属性默认是禁用的。
    glEnableVertexAttribArray(VERTEX_POS_INDX);

    // Set the vertex color to red
    // 设置顶点的颜色值
    // 加载index指定的通用顶点属性，加载(x,y,z,w)
    // opengl各个坐标系理解与转换公式 https://blog.csdn.net/grace_yi/article/details/109341926
    // x，y，z，w：指的不是四维，其中w指的是缩放因子
    // X轴为水平方向，Y轴为垂直方向，X和Y相互垂直
    // Z轴同时垂直于X和Y轴。Z轴的实际意义代表着三维物体的深度
    glVertexAttrib4fv(VERTEX_COLOR_INDX, color);
    // 相对于下面这句   这里设置成蓝色
//    glVertexAttrib4f(1,0.0,0.0,1.0f,1.0f);

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
颜色是通过glVertexAttrib4fv 这个方法，将color设置进去。

glVertexAttrib4fv 函数是OpenGL函数，用于向顶点着色器传递数据。它有以下参数：
index：指定要修改的顶点属性的索引。
v：指向长度为4的浮点数数组的指针，表示要传递的数据。

glVertexAttrib4fv 函数用于向顶点着色器传递一个4维向量。在顶点着色器中，通过索引值可以访问这个4维向量，并对其进行处理。例如，可以用它来定义一个顶点的位置，并在顶点着色器中对其进行变换。

其实这个方法并不是专门设置颜色，而是前面的顶点着色器文件中定义了参数：
```glsl
#version 300 es
// 位置变量的属性位置值为 0
layout(location = 0) in vec4 a_position;
// 颜色变量的属性位置值为 1
layout(location = 1) in vec4 a_color;
```

然后我们就可以通过：
```C
glVertexAttrib4fv(1, color);
```
这个1就是颜色变量的属性位置值为1，就是设置的这个。

## 2 Demo3 展示红绿蓝三角形

### 2.1 效果
<img src=demo3_1.png width=50%>

### 2.2 Kotlin层
```Kotlin
val itemHelloTriangle3: MutableMap<String, Any?> = HashMap()
itemHelloTriangle3[ITEM_IMAGE] = R.mipmap.ic_triangle3
itemHelloTriangle3[ITEM_TITLE] = "展示一个基本的由红、绿、蓝三种颜色绘制而成的三角形"
itemHelloTriangle3[ITEM_SUBTITLE] = "使用了顶点缓冲对象(Vertex Buffer Objects, VBO) 和 EBO 技术"
data.add(itemHelloTriangle3)
typeMapping.put(i++, IMyNativeRendererType.SAMPLE_TYPE_TRIANGLE3)
```
数据定义，这里传值为 102

然后初始化调用：
```
 myNativeRenderer.setRenderType(IMyNativeRendererType.SAMPLE_TYPE, type)
```

自定义View调用：
```kotlin
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
这里setRenderer。因为继承了GLSurfaceView，其实setRenderer走的是GLSurfaceView的方法。

所以mRender必须实现这个接口：
```
 public interface Renderer {
       
        void onSurfaceCreated(GL10 gl, EGLConfig config);

        void onSurfaceChanged(GL10 gl, int width, int height);

        void onDrawFrame(GL10 gl);
    }
```

然后再这几个方法，我们调用了C层。

### 2.3 C层
主要是先在MyGLRenderContext中走一个setRenderType方法：
```C
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
```
这里的demo指针是 NativeTriangle3，主要看这个类即可。

顶点坐标声明：
```C
#include "NativeTriangle3.h"

// 可以参考这篇讲解： https://learnopengl-cn.github.io/01%20Getting%20started/04%20Hello%20Triangle/

// 3 vertices, with (x,y,z) ,(r, g, b, a) per-vertex
static GLfloat vertexPos[3 * VERTEX_POS_SIZE] =
        {
                // 逆时针 三个顶点
                0.0f, 0.5f, 0.0f,            // 上角
                -0.5f, -0.5f, 0.0f,          // 左下角
                0.5f, -0.5f, 0.0f            // 右下角
        };
```

顶点坐标颜色值声明：
```C
// 设置顶点的颜色值
static GLfloat color[4 * VERTEX_COLOR_SIZE] =
        {
                1.0f, 0.0f, 0.0f, 1.0f,   // c0
                0.0f, 1.0f, 0.0f, 1.0f,   // c1
                0.0f, 0.0f, 1.0f, 1.0f    // c2
        };
```

其它static变量声明：
```C
static GLint vtxStrides[2] =
        {
                VERTEX_POS_SIZE * sizeof(GLfloat),
                VERTEX_COLOR_SIZE * sizeof(GLfloat)
        };

// Index buffer data
// 注意索引从0开始!
static GLushort indices[3] = {
        0, 1, 2
};

static GLfloat *vtxBuf[2] = {vertexPos, color};
```

Surface初始化调用：
```C
void NativeTriangle3::Create() {
    GLUtils::printGLInfo();

    // 顶点着色器
    VERTEX_SHADER = GLUtils::openTextFile(
            "vertex/vertex_shader_hello_triangle2.glsl");
    // 片段着色器
    FRAGMENT_SHADER = GLUtils::openTextFile(
            "fragment/fragment_shader_hello_triangle2.glsl");
    m_ProgramObj = GLUtils::createProgram(&VERTEX_SHADER, &FRAGMENT_SHADER);

    if (!m_ProgramObj) {
        LOGD("Could not Create program")
        return;
    }

    vboIds[0] = 0;
    vboIds[1] = 0;
    vboIds[2] = 0;

    // 设置清除颜色
    glClearColor(1.0f, 1.0f, 1.0f, 0.0f);
}
```

顶点着色器代码块：
```glsl
#version 300 es
// 位置变量的属性位置值为 0
layout(location = 0) in vec4 a_position;
// 颜色变量的属性位置值为 1
layout(location = 1) in vec4 a_color;
// 向片段着色器输出一个颜色
out vec4 v_color;

void main()
{
    v_color = a_color;
    gl_Position = a_position;
}
```

输出颜色值： v_color,但这个未初始化，默认是黑色，需要后面走绘制方式时再调用赋值参数。
gl_Position对应三角形坐标位置，系统规定这个gl_Position就对应坐标位置。

片段主色器代码块
```glsl
#version 300 es
// 表示OpenGL ES着色器语言V3.00

// 声明着色器中浮点变量的默认精度
precision mediump float;

// 声明由上一步顶点着色器传入进来的颜色值
in vec4 v_color;

// 声明一个输出变量fragColor，这是一个4分量的向量，
// 写入这个变量的值将被输出到颜色缓冲器
out vec4 o_fragColor;

void main()
{
	o_fragColor = v_color;
}
```

具体绘制还是得看Draw方法：

先走 glClear清屏；

再走 glUseProgram 执行程序；

然后是
```C
if (vboIds[0] == 0 && vboIds[1] == 0 && vboIds[2] == 0) {
    // Only allocate on the first Draw

    // 使用glGenBuffers函数生成3个VBO对象
    glGenBuffers(3, vboIds);

    // OpenGL有很多缓冲对象类型，顶点缓冲对象的缓冲类型是GL_ARRAY_BUFFER。
    // OpenGL允许我们同时绑定多个缓冲，只要它们是不同的缓冲类型。
    // 我们可以使用glBindBuffer函数把新创建的缓冲绑定到GL_ARRAY_BUFFER目标上
    // 从这一刻起，我们使用的任何（在GL_ARRAY_BUFFER目标上的）缓冲调用都会用来配置当前绑定的缓冲(VBO)。

    // VBO 0 是 缓冲 顶点
    // 复制顶点数组到缓冲中供OpenGL使用
    glBindBuffer(GL_ARRAY_BUFFER, vboIds[0]);
    /**
        * 然后我们可以调用glBufferData函数，它会把之前定义的顶点数据复制到缓冲的内存中。
        * glBufferData是一个专门用来把用户定义的数据复制到当前绑定缓冲的函数。
        *  它的第一个参数是目标缓冲的类型：顶点缓冲对象当前绑定到GL_ARRAY_BUFFER目标上。
        *  第二个参数指定传输数据的大小(以字节为单位)；用一个简单的sizeof计算出顶点数据大小就行。
        *  第三个参数是我们希望发送的实际数据。
        *  第四个参数指定了我们希望显卡如何管理给定的数据。它有三种形式：
                GL_STATIC_DRAW ：数据不会或几乎不会改变。
                GL_DYNAMIC_DRAW：数据会被改变很多。
                GL_STREAM_DRAW ：数据每次绘制时都会改变。

                三角形的位置数据不会改变，每次渲染调用时都保持原样，所以它的使用类型最好是GL_STATIC_DRAW。
                如果，比如说一个缓冲中的数据将频繁被改变，那么使用的类型就是GL_DYNAMIC_DRAW或GL_STREAM_DRAW，
                这样就能确保显卡把数据放在能够高速写入的内存部分。
        */
    glBufferData(GL_ARRAY_BUFFER, vtxStrides[0] * 3, vtxBuf[0], GL_STATIC_DRAW);

    // VBO 1 是 缓冲 颜色
    glBindBuffer(GL_ARRAY_BUFFER, vboIds[1]);
    glBufferData(GL_ARRAY_BUFFER, vtxStrides[1] * 3, vtxBuf[1], GL_STATIC_DRAW);

    // VBO 2 实际上是 EBO， 缓冲 indices
    // 和顶点缓冲对象一样，EBO也是一个缓冲，它专门储存索引，OpenGL调用这些顶点的索引来决定该绘制哪个顶点。
    // 所谓的索引绘制(Indexed Drawing)正是我们问题的解决方案。

    // 与VBO类似，我们先绑定EBO然后用glBufferData把索引复制到缓冲里。
    // 同样，和VBO类似，我们会把这些函数调用放在绑定和解绑函数调用之间，只不过这次我们把缓冲的类型定义为GL_ELEMENT_ARRAY_BUFFER。
    glBindBuffer(GL_ELEMENT_ARRAY_BUFFER, vboIds[2]);
    glBufferData(GL_ELEMENT_ARRAY_BUFFER, sizeof(GLushort) * 4, indices, GL_STATIC_DRAW);
}
```
这里有个if，应该是第一次才走进来，或者vboIds的值未生效时走。

```
glGenBuffers(3, vboIds);
```
这个函数是啥用？
是 OpenGL 库中的一个函数，用于生成一个或多个缓冲对象标识符。
缓冲对象是存储在图形卡内存中的数据结构，用于存储顶点数据、纹理数据等，通过调用 glGenBuffers 函数可以为这些数据创建一个唯一的标识符。

然后走
```
glBindBuffer(GL_ARRAY_BUFFER, vboIds[0]);
```
这个函数有啥用？
glBindBuffer 函数的作用是将缓冲区对象绑定到指定的缓冲区类型上。
这样，OpenGL 就知道后续的操作该作用于哪个缓冲区。它常用于顶点缓冲区对象 (VBO) 和索引缓冲区对象 (IBO)。

然后走
```
 glBufferData(GL_ARRAY_BUFFER, vtxStrides[0] * 3, vtxBuf[0], GL_STATIC_DRAW);
```
这个函数有啥用？
glBufferData 函数是OpenGL中用来分配内存并存储顶点数据的一个函数。
它接受一个缓冲对象标识符，一个指向顶点数据的指针，以及该数据的大小。
调用这个函数后，OpenGL将在内存中为缓冲对象分配一块连续的内存，并将顶点数据复制到该内存中。在后续的图形绘制中，OpenGL将从该缓冲对象中读取顶点数据。

所以先调用glGenBuffers生产缓冲区标识符，然后调用glBindBuffer绑定缓冲区对象，然后glBufferData分配缓冲区数据。

这里的demo，绑定了3组缓冲区对象，分别是：三角形坐标点，顶点颜色值，索引值。

然后程序继续：
```
glEnableVertexAttribArray(VERTEX_POS_INDX);
```
glEnableVertexAttribArray 函数是用于在 OpenGL 中启用顶点属性数组。
通过使用该函数，我们可以告诉 OpenGL 要读取顶点属性数组中的数据。在绘制图形之前，必须启用顶点属性数组，才能够使用它们来描述图形。
这里启用了顶点数组。

然后设置顶点属性指针：
```
glVertexAttribPointer(VERTEX_POS_INDX, VERTEX_POS_SIZE, GL_FLOAT,
                          GL_FALSE, vtxStrides[0], nullptr);
```

> glVertexAttribPointer函数是OpenGL中用于设置顶点属性指针的函数，其参数如下：
index：顶点属性的索引值，表示该属性数组的编号。
size：每个顶点属性的分量数量，可以为1,2,3,4中的任意一个值。
type：数据类型，可以是GL_BYTE, GL_UNSIGNED_BYTE, GL_SHORT, GL_UNSIGNED_SHORT, GL_INT, GL_UNSIGNED_INT, GL_FLOAT等。
normalized：是否将数据归一化，如果为GL_TRUE，则数据会被归一化到0~1的范围内；如果为GL_FALSE，则数据会按原始值保存。
stride：每个顶点的步长，即顶点属性的数据在内存中相邻两个顶点属性之间的字节数。
pointer：顶点属性数组的首地址。

这里前面通过glBufferData给vboIds数组赋值了，所以之前vboIds的值全为0，这里现在变成1,2,3了。

后面继续走glBindBuffer:
```
glBindBuffer(GL_ARRAY_BUFFER, vboIds[1]);
```
其实就是走
glBindBuffer(GL_ARRAY_BUFFER, 1);

然后重复走glEnablevertexAttribArray和glVertexAttribPointer方法。

然后if外面，其实就是拿缓存区里面的东西了。
```
glBindBuffer(GL_ARRAY_BUFFER, vboIds[0]);
glEnableVertexAttribArray(VERTEX_POS_INDX);
glVertexAttribPointer(VERTEX_POS_INDX, VERTEX_POS_SIZE, GL_FLOAT,
                          GL_FALSE, vtxStrides[0], nullptr);
```
上面是拿第一个缓冲区数据，然后立即将绑定的缓冲数据映射到顶点着色器中的顶点属性变量。
> 在使用OpenGL中的glVertexAttribPointer和glBindBuffer函数时，通常需要先绑定需要操作的缓冲对象，再调用glVertexAttribPointer函数。
具体地，通常会进行如下操作：
使用glGenBuffers函数创建一个缓冲对象。
使用glBindBuffer函数绑定刚创建的缓冲对象。
使用glBufferData函数将顶点数据复制到缓冲对象中。
调用glVertexAttribPointer函数指定顶点属性数组。
绑定缓冲对象后调用glVertexAttribPointer函数，表示将当前绑定的缓冲对象映射到顶点着色器中的顶点属性变量。因此，绑定和映射是两个独立的操作，需要按照正确的顺序进行。

最后，再先拿glBindBuffer，再走glDrawElements，为何要按照这个顺序？
> glBindBuffer 用于绑定缓冲区，指定下一次操作的缓冲区对象。在使用 glDrawElements 绘制图形之前，需要通过 glBindBuffer 将顶点数据缓冲区绑定到 OpenGL，以便它能够使用这些数据进行渲染。
因此，在执行 glDrawElements 之前，必须首先通过执行 glBindBuffer 来绑定包含顶点数据的缓冲区，以确保 OpenGL 能够访问这些数据并正确渲染图形。

那glDrawElements函数到底怎么用呢？
> glDrawElements是OpenGL中的一个函数，用于通过索引来渲染几何图形。它用于绘制一系列的三角形或其他图元，基于一个索引数组和一组顶点数组。索引用于访问存储在一个或多个顶点数组中的顶点数据，这些数组可用于指定顶点的位置、颜色、纹理坐标或其他属性。
在这种情况下，传递了nullptr，表示未提供索引数据，顶点将按照它们在顶点缓冲区中存储的顺序绘制。


最后将顶点disable:
```
glDisableVertexAttribArray(VERTEX_POS_INDX);
glDisableVertexAttribArray(VERTEX_COLOR_INDX);
```

然后清空缓存：
```
glBindBuffer(GL_ARRAY_BUFFER, 0);
glBindBuffer(GL_ELEMENT_ARRAY_BUFFER, 0);
```

然后析构函数注意shutdown回收：
```
 glDeleteBuffers(3, &vboIds[0]);
```
这里把第一个地址给它即可。

## 3 总结

* 绘制三角形方法1，定义 in vec4 a_position位置信息，in vec4 a_color颜色信息，位置给gl_position表示顶点位置，a_color作为输出。绘制的时候通过glVertextAttribPointer方法可以直接将坐标点传进去，然后通过glVertexAttrib4fv给顶点设置颜色。然后直接用glDrawArrays开始绘制。

* 使用缓冲区绘制三角形，着色器代码不改，本地建立一个int指针，第一个存顶点，第二个存颜色，第3个存3个点索引。然后通过glVertexAttribPointer，再拿第二个缓冲区再设置一次glVertexAttribPointer，相当于给顶点着色器中的数值赋值了。最后通过再拿索引，走glDrawElements绘制。记得需要disable和清缓存，并且销毁的时候需要将指针delete。

