---
title: Auto.js 学习笔记1
date: 2023-04-14 20:51:59
top: false
cover: false
toc: true
mathjax: true
tags:
- Autojs
categories:
- Autojs
---

## 0 环境配置
下载4.1版本的Auto.js
因为AutojsPro现在无法注册，需要下载一个4.1版本无需更新的破解版app。
> 链接: https://pan.baidu.com/s/1hHDOw-pCPGROzNhqyuKWZQ 提取码: ceqa 


## 1 查找控件

toast("123456") 弹出toast

UiSelector.findOne()  返回一个 UIObject

UiSelector.text(str) 返回一个UISelector

text("微信").findOne()  找到微信控件 （根据屏幕上的显示进行寻找，一直找，会一直阻塞，无法执行后续代码）

text("微信").findOne(1000) 寻找1秒，找不到，会执行后续代码

text("微信").id("xxx") 通过两个条件去寻找

text("微信").findOnce() 只找一次

text("微信").findOnce(1) 对屏幕上的空间进行搜索，并返回第i+1个符合条件的控件

text("微信").find() 返回一个集合 UiCollection

text("微信").untilFind() 一直寻找，如果为空，则阻塞

text("微信").exists() 是否存在

text("微信").waitFor() 等待出现符合条件，如果没有，则阻塞

## 2 控件集合的操作方法

[”控件1“,”控件2“,”控件3“,”控件4“,”控件5“]
"控件1".click()

UiCollection

size(): 长度
get(i): 获取元素
each(func): 遍历
empty(): 是否为空
nonEmpty(): 是否非空

比如找出id为dux的控件数组
var 控件数量 = id("com.tencent.mm:id/dux").find().size()
id("").find().get(1) 获取集合第2个元素。

var 控件对象 = id("com.tencent.mm:id/dux").find().get(1)
控件对象.text()

触发点击：
click(控件对象.text())

快速遍历代码：点击布局范围分析->生成代码

```
id("f4").findOne().children().forEach(child => {

    var target = child.findOne(id("dux"));
    target.click();

})
```

或者：

```
id("com.tencent.mm:id/dux").find().forEach(child => {
    
    click(child.text())
    sleep(3000)
    back()
    sleep(1500)

})
```
实现朋友圈通讯录每个点击。

是否为空：
var bool = id("dux").find().empty()
log(bool)


UiCollection.find(selector)
UiCollection.findOne(selector)

如：
```
var myText = id("com.tencent.mm:id/dux").find().findOne(text("A")).text() // 不区分大小写
click(myText) // 点击
```

## 3 颜色相关

### 3.1 颜色简单api

> #AARRGGBB
透明度，红色FF，绿色FF，蓝色FF

一个简单的ui界面：
```
”ui“;
ui.layout(
    <vertical>
        <button bg="#22ff0000" text="第一个按钮"/>
        <button text="第二个按钮"/>
    </vertical>
)
```

可以通过colors.toString(),将
colors.toString(-16777206) 将颜色整数转换为字符串
colors.parseColor("#00000a")

demo:
colors.parseColor("#000032") 结果为 -16777166

colors.red(colorNum) 返回颜色的R通道的值  colors.red("#000032") 可以输入number或者字符串
colors.blue(colorNum) 返回颜色的G通道的值
colors.alpha(colorNum) 返回颜色的Alpha通道的值

colors.rgb(red, green, blue) 返回这些颜色通道构成的整数颜色值

colors.argb(alpha, red, green, blue) 返回这些颜色通道构成的整数颜色值，带了alpha


### 3.2 判断颜色相等或者相似

判断两个颜色是否相似
colors.isSimilar(num|str, num|str2[, thresholdNum, algorithm])
参数1： 颜色值1
参数2： 颜色值2
参数3： 颜色相似度临界值，默认为4，取值范围为0~255
参数4： 颜色匹配算法，默认为diff，还有 rgb, rgb+ ,hs

demo:
colors.isSimilar('#000000', '#000001')
colors.isSimilar('#000000', '#000000', 4)


判断两个颜色是否相等(忽略透明度)
colors.equals(color1, color2)

### 3.3 autojs内置颜色
```

#colors.BLACK
黑色，颜色值 #FF000000

#colors.DKGRAY
深灰色，颜色值 #FF444444

#colors.GRAY
灰色，颜色值 #FF888888

#colors.LTGRAY
亮灰色，颜色值 #FFCCCCCC

#colors.WHITE
白色，颜色值 #FFFFFFFF

#colors.RED
红色，颜色值 #FFFF0000

#colors.GREEN
绿色，颜色值 #FF00FF00

#colors.BLUE
蓝色，颜色值 #FF0000FF

#colors.YELLOW
黄色，颜色值 #FFFFFF00

#colors.CYAN
青色，颜色值 #FF00FFFF

#colors.MAGENTA
品红色，颜色值 #FFFF00FF

#colors.TRANSPARENT
透明，颜色值 #00000000
```

## 4 图片相关

### 4.1 图片回收机制

var img = images.read("./1.png");

不用的时候，需要调用回收方法
img.recycle();

截图生成一个对象：
var img2 = caputerScreen()

### 4.2 读取图片

通过路径加载：
images.read(path); 

通过链接加载：
images.load(url);

拷贝：
images.copy(img);

保存图片：
图片对象.saveTo(path);

保存网络图片到根目录下某个文件夹
var img = images.load('https://pro.autojs.org/docs/logo.png');
img.saveTo("/sdcard/download/脚本/auto.png")
这里应该只有根目录有权限。

拷贝图片：
var img = images.load('https://pro.autojs.org/docs/logo.png');
var img2 = images.copy(img)
img2.saveTo('/sdcard/download/脚本/copy.png');

最后别忘记回收。recycle();

### 4.3 image对象

getWidth()
getHeight()
saveTo(path)

```
var img = images.read('auto.png')

if (img) {
    var w = img.getWidth();
    var h = img.getHeight();

    log(w, h);

    toast("w="+w+" h="+h);
} else {
    toast("图片为空! ")
}
```

获取图片像素点：
image.pixel(x, y)

eg:
```
var img = images.read('auto.png');
if (img) {
    var color = img.pixel(100, 100);
  
    var colorStr = colors.toString(color);
    log(colorStr);
    toast(colorStr);
    
} else {
    toast("找不到图片")
}

// 输出 #ffadcf4e
```

### 4.4 图片对象保存方法
法1：
```
Image.saveTo(path)

var img = images.load('https://pro.autojs.org/docs/logo.png');
img.saveTo("/sdcard/logo2.png");

```

法2：
```
Image.save(image, path[format="png",quality=100])
// png一般底色可以透明， jpg没有透明

```

### 4.5 图片的编码转换

images.fromBase64(base64) // 返回img对象

images.toBase64(img[, format = "png", quality = 100]) // 返回base64对象

images.fromBytes(bytes) // 返回img对象

images.toBytes(img[, format = "png", quality = 100]) // 返回字节

将一个Base64的字符串，转换为图片；
先在 [站长工具base64](https://tool.chinaz.com/tools/imgtobase)

将一张图片转换为base64字符串：
```
var base64 = "data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAAZAAAABqCAYAAACSwQcHAAAJzWlDQ1BJQ0MgUHJvZmlsZQAASImVlndUFNcex+/M9kbbpbelN+BCPmWQylAAAAAElFTkSuQmCC";

var img = images.fromBase64(base64);

img.saveTo("/sdcard/111.png");
```
f5运行， 此时手机里面就有这个图片了。



图片转换为base64:
```
var img = images.read("/sdcard/111.png")
var base64 = images.toBase64(img);
log(base64);
```

写入文件操作：
```
var img = images.read("/sdcard/111.png")
var base64 = images.toBase64(img);
log(base64);

files.write("/sdcard/autoImage.txt", base64);
```
此时根目录会多一个这个txt文件。

base64字符串转换为图片：
```
var img = images.read("/sdcard/111.png")
var base64 = images.toBase64(img);

var img2 = images.fromBase64(base64);
img2.saveTo("/sdcard/112.png");
```

字节转换图片：
```
var res = http.get("https://pro.autojs.org/docs/logo.png");
var myBytes = res.body.bytes();

var img = images.fromBytes(myBytes);
img.saveTo("/sdcard/113.png");
```

图片转换为字节：
```
var res = http.get("https://pro.autojs.org/docs/logo.png");
var myBytes = res.body.bytes();

var img = images.fromBytes(myBytes);
img.saveTo("/sdcard/113.png");

var b = images.toBytes(img);
```


### 4.6 封装获取屏幕小图的函数

images.clip 剪切图片
参数 img, x, y, w, h

先申请截图权限：
```
if (!requestScreenCapture()) {
    toast("请求截图失败")
    eixt();
}

sleep(3000);

getImg(64, 195, 320, 200, "/sdcard/124.png")

function getImg(x1, y1, x2, y2, path) {
    var img = images.captureScreen();
    var imgS = images.clip(img, x1, y1, x2 - x1, y2 - y1);
    imgS.saveTo(path);
    imgS.recycle();
}
```

### 4.7 图片处理的函数

images.resize(img, size[, interpolation]); // 调整图片大小

images.scale(img, fx, fy[, interpolation]); //缩放图片，返回缩放后的图片

images.rotate(img, degree[, x, y]); // 将图片逆时针旋转degress度，返回旋转后的图片对象。

images.concat(img1, image2[, direction]); // 连接两张图片，返回连接后的图像，如果两张图片大小不一致，小的那张将适当居中。

images.grayScale(img); // 灰度化图片，并返回灰度化后的图片

images.threshold(img, threshold, maxVal[, type]); // 将图片阙值化

images.adaptiveThreshold(img, maxValue, adaptiveMethod, thresholdType, blockSize, C); // 对图片进行自适应阙值化处理

images.cvtColor(img, code[, dstCn]); //对图像进行颜色空间转换

images.inRange(img, lowerBound, upperBound); //将图片二值化

eg1:使用resize指南
```
var img = images.read('auto.png');

var imgRes = images.resize(img, [50, 100]); 

imgRes.saveTo('/sdcard/115.png');
```

eg2: 缩放图片：
```
var img = images.read('auto.png');

var imgRes = images.scale(img, 0.5, 0.5); 

imgRes.saveTo('/sdcard/116.png');
```

### 4.8 请求截图权限

images.requestScreenCapture([landscape])

```
threads.start(function() {

    while(true) {
        if (text('立即开始').findOnce()) {
            text('立即开始').findOnce().click()
            break;
        } else {
            sleep(3000);
        }
    }

});

if (!requestScreenCapture()) {
    toast("请求截图失败")
    eixt();
}

toast("请求截图成功")
```


### 4.9 截屏功能

images.captureScreen()

captureScreen()

区别，返回Image对象，保存到路径，不返回。


```
if (!requestScreenCapture()) {
    toast("请求截图失败")
    eixt();
}

toast("请求截图成功")

var img对象 = images.captureScreen()
img对象.saveTo("/sdcard/118.png");

var img对象2 = captureScreen();
```

截图，并保存到这里：
images.captureScreen("/sdcard/119.png")

captureScreen 不需要回收哦。

### 4.10 获取图片某点的颜色

images.pixel(image, x, y)

Image.pixel(x, y)

```

var img = images.read("auto.png")

var col = images.pixel(img, 100, 100);

log(col); // -5386418

var str = colors.toString(col);
toast(str);
```

### 4.11 在图片中寻找颜色以及Point对象讲解
images.findColor(image, color, options)











