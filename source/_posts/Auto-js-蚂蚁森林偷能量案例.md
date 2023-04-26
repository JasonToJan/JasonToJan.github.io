---
title: Auto.js 蚂蚁森林偷能量案例
date: 2023-04-26 18:34:22
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

## 1 效果图
<img src=autojs.gif>

## 2 解锁
手机处于锁屏状态是很常见的，所以需要考虑一下手机锁屏时，如何解锁。
目前考虑数字密码，滑动的相对复杂一点，后面再研究下。

### 2.1 创建一个机器类
这个是处理一些屏幕的坐标点击相关事件。
```

/**
 * 机器人工厂
 * @param {int} max_retry_times 最大尝试次数
 * @author ridersam <e1399579@gmail.com>
 */
function Robot(max_retry_times) {
    this.robot = (device.sdkInt < 24) ? new LollipopRobot(max_retry_times) : new NougatRobot(max_retry_times);

    this.click = function (x, y) {
        return this.robot.click(x, y);
    };

    this.clickCenter = function (b) {
        var rect = b.bounds();
        return this.robot.click(rect.centerX(), rect.centerY());
    };

    this.swipe = function (x1, y1, x2, y2, duration) {
        this.robot.swipe(x1, y1, x2, y2, duration);
    };

    this.back = function () {
        Back();
    };

    this.kill = function (package_name) {
        shell("am force-stop " + package_name, true);
    };

    this.clickMultiCenter = function (collection) {
        var points = [];
        collection.forEach(function(o) {
            var rect = o.bounds();
            points.push([rect.centerX(), rect.centerY()]);
        });
        this.robot.clickMulti(points);
    };

    this.clickMulti = function (points) {
        this.robot.clickMulti(points);
    };
}

module.exports = Robot;
```

### 2.2 解锁类
最好是单独封装一下，因为解锁相对比较独立，其它项目可能也会用到。
这里单独用一个Security类封装：
```
/**
 * 安全相关
 * @param {Robot} robot 机器人对象
 * @param {int} max_retry_times 最大尝试次数
 * @author ridersam <e1399579@gmail.com>
 */
function Secure(robot, max_retry_times) {
    this.robot = robot;
    this.max_retry_times = max_retry_times || 10;
    this.km = context.getSystemService(context.KEYGUARD_SERVICE);

    log("走了Secure的构造函数")
    this.secure = (function () {
        var secure;

        var miui_match = shell("getprop ro.miui.ui.version.name").result.match(/\d+/);
        switch (true) {
            case (miui_match !== null):
                if (miui_match[0] === '10') {
                    secure = new MIUI10Secure(this);
                } else {
                    secure = new MIUISecure(this);
                }
                break;
            default:
                secure = new NativeSecure(this);
                break;
        }

        return secure;
    }.bind(this))();

    this.isLocked = function () {
        return this.km.inKeyguardRestrictedInputMode();
    };

    this.openLock = function (password, pattern_size) {

        var isLocked = this.isLocked(); // 是否已经上锁
        var isSecure = this.km.isKeyguardSecure(); // 是否设置了密码
        pattern_size = pattern_size || 3;
        log({
            isLocked: isLocked,
            isSecure: isSecure
        });

        // 有些手机需要上滑一下，才能输入密码
        var height = device.height; //设定高度值=设备高度
        var width = device.width; //设定宽度值=设备宽度
        swipe(width / 2, height - 500, width / 2, 0, 500);

        var i = 0;
        while (this.secure.hasLayer()) {
            if (!this.isLocked()) return true;

            if (i >= this.max_retry_times) {
                toastLog("打开上滑图层失败");
                return this.failed();
            }
            log("向上滑动");
            this.openLayer();
            i++;
        }

        if (!(isLocked && isSecure)) return true;
        log("开始解锁.........");
        for (var i = 0; i < this.max_retry_times; i++) {
            if (this.unlock(password, pattern_size)) {
                return true;
            } else {
                toastLog("解锁失败，重试");
            }
        }

        toastLog("解锁失败，不再重试");
        return this.failed();
    };

    this.failed = function () {
        KeyCode("KEYCODE_POWER");
        engines.stopAll();
        exit();
        return false;
    };

    this.openLayer = function () {
        var x = WIDTH / 2;
        var y = HEIGHT - 300;
        this.robot.swipe(x, y, x, HEIGHT / 2, 500);
        sleep(1500); // 等待动画
    };

    this.unlock = function (password, pattern_size) {
        log("password="+password)
        var len = password.length;

        if (len < 4) {
            throw new Error("密码至少4位");
        }

        return this.secure.unlock(password, pattern_size);
    };

    this.gestureUnlock = function (pattern, password, len, pattern_size) {
        var rect = pattern.bounds();
        // 使用坐标查找按键
        var oX = rect.left, oY = rect.top; // 第一个点位置
        var w = (rect.right - rect.left) / pattern_size, h = (rect.bottom - rect.top) / pattern_size; // 2点之单间隔为边框的1/3
        var points = [];

        points[0] = {
            x: 0,
            y: 0
        };
        // 初始化每个点的坐标
        for (var i = 1; i <= pattern_size; i++) {
            for (var j = 1; j <= pattern_size; j++) {
                var row = i - 1;
                var col = j - 1;
                var index = pattern_size * (i - 1) + j; // 序号，从1开始
                points[index] = {
                    x: oX + col * w + w / 2,
                    y: oY + row * h + h / 2
                };
            }
        }

        // 使用手势解锁
        var gestureParam = [100 * len];
        for (var i = 0; i < len; i++) {
            var point = points[password[i]];

            gestureParam.push([point.x, point.y]);
        }
        gestures(gestureParam);

        return this.checkUnlock();
    };

    this.unlockPassword = function (password) {
        if (typeof password !== "string") {
            password = password.join("");
        }
        setText(0, password); // 输入密码
        var confirm;
        if (confirm = text("确认").findOnce()) {
            confirm.click();
        } else {
            KeyCode("KEYCODE_ENTER"); // 按Enter
        }

        sleep(1500);
        return this.checkUnlock();
    };
}

function NativeSecure(secure) {
    this.__proto__ = secure;

    this.hasLayer = function () {
        return id("com.android.systemui:id/preview_container").visibleToUser(true).exists(); // 是否有上滑图层
    };

    this.unlock = function (password, pattern_size) {
        var len = password.length;

        if (id("com.android.systemui:id/lockPatternView").exists()) {
            return this.unlockPattern(password, len, pattern_size);
        } else if (id("com.android.systemui:id/passwordEntry").exists()) {
            return this.unlockPassword(password);
        } else if (id("com.android.systemui:id/pinEntry").exists()) {
            return this.unlockKey(password, len);
        } else {
            log("识别锁定方式失败22222，型号：" + device.brand + " " + device.product + " " + device.release);
            return this.mockClickPassowrd(password);
        }
    };

    this.mockClickPassowrd = function (password) {
        var len = password.length;
        for (var j = 0; j < len; j++) {
            log("模拟点击"+password[j])
            desc(password[j]).findOne().click()
            sleep(500)
        }
        return this.checkUnlock();
    }

    this.unlockKey = function (password, len) {
        for (var j = 0; j < len; j++) {
            var key_id = "com.android.systemui:id/key" + password[j];
            if (!id(key_id).exists()) {
                return false;
            }
            id(key_id).findOne(1000).click();
        }
        if (id("com.android.systemui:id/key_enter").exists()) {
            id("com.android.systemui:id/key_enter").findOne(1000).click();
        }

        return this.checkUnlock();
    };

    this.unlockPattern = function (password, len, pattern_size) {
        var pattern = id("com.android.systemui:id/lockPatternView").findOne(1000);
        return this.gestureUnlock(pattern, password, len, pattern_size);
    };

    this.checkUnlock = function () {
        sleep(1500); // 等待动画
        if (id("android:id/message").textContains("重试").exists()) {
            toastLog("密码错误");
            return this.failed();
        }

        return !this.isLocked();
    };
}

function MIUISecure(secure) {
    this.__proto__ = secure;

    this.hasLayer = function () {
        return id("com.android.keyguard:id/unlock_screen_sim_card_info").exists() 
        || id("com.android.keyguard:id/miui_unlock_screen_digital_clock").exists() 
        || id("com.android.keyguard:id/miui_porch_notification_and_music_control_container").exists()
        || id("com.android.keyguard:id/notification_message_view").exists();
    };

    this.unlock = function (password, pattern_size) {
        var len = password.length;
        
        if (id("com.android.keyguard:id/lockPattern").exists()) {
            return this.unlockPattern(password, len, pattern_size);
        } else if (id("com.android.keyguard:id/miui_mixed_password_input_field").exists()) {
            return this.unlockPassword(password);
        } else if (id("com.android.keyguard:id/numeric_inputview").exists()) {
            return this.unlockKey(password, len);
        } else {
            log("识别锁定方式失败111，型号：" + device.brand + " " + device.product + " " + device.release);
            return this.mockClickPassowrd(password);
        }
    };

    this.mockClickPassowrd = function (password) {
        var len = password.length;
        for (var j = 0; j < len; j++) {
            log("模拟点击"+password[j])
            desc(password[j]).findOne().click()
            sleep(500)
        }
        return this.checkUnlock();
    }

    this.unlockKey = function (password, len) {
        for (var j = 0; j < len; j++) {
            var btn = id("com.android.keyguard:id/numeric_inputview").findOne(1000).findOne(text(password[j]));
            if (btn) {
                this.robot.clickCenter(btn);
            } else {
                return false;
            }
        }

        return this.checkUnlock();
    };

    this.unlockPattern = function (password, len, pattern_size) {
        var pattern = id("com.android.keyguard:id/lockPattern").findOne(1000);
        return this.gestureUnlock(pattern, password, len, pattern_size);
    };

    this.checkUnlock = function () {
        sleep(1500); // 等待动画
        if (id("com.android.keyguard:id/phone_locked_textview").exists()) {
            toastLog("密码错误");
            return this.failed();
        }

        return !this.isLocked();
    };
}

function MIUI10Secure(secure) {
    this.__proto__ = secure;
    this.secure = new NativeSecure(secure);

    this.hasLayer = function () {
        return id("com.android.systemui:id/awesome_lock_screen_container").exists() 
        || id("com.android.systemui:id/notification_container_parent").exists() 
        || id("com.android.systemui:id/keyguard_header").exists()
        || id("com.android.systemui:id/keyguard_carrier_text").exists()
        || id("com.android.systemui:id/notification_panel").exists();
    };

    this.unlock = function (password, pattern_size) {
        return this.secure.unlock(password, pattern_size);
    };
}

module.exports = Secure;
```

### 2.3 前置条件
判断一些模块是否引入。
主要这些模块需要对应手机里面脚本的目录下
如果用VS code开发，先安装相关autojs插件，然后 command+shift+P 输入autojs
然后保存到指定设备，这样就同步到手机里面了。

```
auto(); // 自动打开无障碍服务

/**
 * 判断当前目录下是否存在名为"config.js"的文件，如果存在则使用require()方法加载该文件并将其内容赋值给变量config，否则创建空对象{}并赋值给config。
 */
var config = files.isFile("config.js") ? require("config.js") : {};

/**
 * 最后判断config是否为一个对象，如果不是则重新将空对象{}赋值给config。
 */
if (typeof config !== "object") {
    config = {};
}

// 配置参数
var options = Object.assign({
    password: "123456",
    pattern_size: 3
}, config); // 用户配置合并

// 所有操作都是竖屏
const WIDTH = Math.min(device.width, device.height);
const HEIGHT = Math.max(device.width, device.height);
const IS_ROOT = files.exists("/sbin/su") || files.exists("/system/xbin/su") || files.exists("/system/bin/su");

setScreenMetrics(WIDTH, HEIGHT);

// 开始
start(options);

```

### 2.4 手机屏幕息屏处理
继续走到start里面：
```
  checkModule();

    var Robot = require("Robot.js");
    var robot = new Robot(options.max_retry_times);

    // 创建一个蚂蚁森林的实例对象
    var antForest = new AntForest(robot, options);
    antForest.saveState();

    while (!device.isScreenOn()) {
        device.wakeUp();
        sleep(1000); // 等待屏幕亮起
        log("唤醒屏幕了")
    }
```
我们先创建一些必要实例，然后通过device.wakeUp唤醒屏幕。

### 2.5 解锁
```
    // 解决锁屏问题
    if (files.exists("Secure.js")) {
        var Secure = require("Secure.js");
        var secure = new Secure(robot, options.max_retry_times);
        // 这里应该是解锁
        secure.openLock(options.password, options.pattern_size);
        // 拉起到前台界面
        antForest.openApp();
    }
```
这里判断Secure.js是否存在，存在就解锁。

## 3 打开支付宝页面
```
this.openApp = function () {
        launch(this.package);
    };

    this.closeApp = function () {
        this.robot.kill(this.package);
    };

    this.launch = function () {
        var times = 0;
        do {
            if (this.doLaunch()) {
                return;
            } else {
                times++;
                this.back();
                sleep(1500);
                this.openApp();
            }
        } while (times < this.options.max_retry_times);

        throw new Error("运行失败");
    };

    this.doLaunch = function () {
        // 可能出现的红包弹框，点击取消
        var timeout = this.options.timeout;
        threads.start(function () {
            var cancelBtn;
            if (cancelBtn = id("com.alipay.mobile.accountauthbiz:id/update_cancel_tv").findOne(timeout)) {
                cancelBtn.click();
            }
            if (cancelBtn = id("com.alipay.android.phone.wallet.sharetoken:id/btn1").findOne(timeout)) {
                cancelBtn.click();
            }
        });

        log("打开蚂蚁森林2");
        var ant = text("蚂蚁森林").findOne();
        log("找到了蚂蚁森林");
        ant.parent().parent().click();

        return true;
    };
```
主要是通过launch函数，传入支付宝包名，这样就能打开了。

打开后，通过 text("蚂蚁森林")找到控件，然后用父控件走点击事件，这样就到蚂蚁森林页面了。

## 4 跳转找能量页面
难点是这个找能量是一张图片，如何识别图片，然后准确点击呢。
那就用到了截图，截图需要申请权限，这个是一个录制权限。
```
   // 找能量图片
        var icon_list = [];
        var icon = images.read(this.options.findImg);
        if (null === icon) {
            throw new Error("缺少图片文件，请仔细查看使用方法的第一条！！！");
        }
        icon_list = [icon];
```
这里读取了找能量的图片，后面会用到。

申请截图权限：
```
 // 截图权限申请
threads.start(function () {
    var beginBtn;
    if (beginBtn = classNameContains("Button").textContains("立即开始").findOne(timeout)) {
        log("找到了哦")
        sleep(1000)
        beginBtn.click();
    } else {
        log("没有找到哦")
    }
});
if (!requestScreenCapture(false)) {
    throw new Error("请求截图失败");
}
```
这里是requestScreenCapture去申请，然后开了个子线程，去点击同意权限。

## 5 肆无忌惮地偷能量吧
```
   /**
     * 去偷能量
     * @param icon 
     */
    this.goToStealEnergy = function (iconList) {
        while (1) {
            if (this.findNextImage(iconList[0])) {
                
                // 开始收取能量了
                while (1) {
                    if (this.handleEnergy(iconList[1])) {
                        // 找到有能量的人
                    } else {
                        break;
                    }
                   
                }

            } else {
                break;
            }
            
        }
    }
```
这里第一层循环，是持续寻找找能量图片，因为一个人收完后，右下角可以继续点击找能量切换到下一个人。

第二层循环，就是去持续的点击能量球，所以我们也需要制造一张很小的能量球的图片（裁剪）。

### 5.1 识别“找能量”图片
```
 /**
     * 找下一个有能量的
     * @param  icon 
     */
    this.findNextImage = function (icon) {
        var point;
        var total = 0;
        var times = 0;
        var x = WIDTH / 2;
        var offset = icon.getHeight() / 2;

        log("开始寻找find图片了" + offset)

        while (times < this.options.max_retry_times) {
            // 截图
            this.capture = captureScreen();
            if (null === this.capture) {
                toastLog("截图失败");
                times++;
                sleep(200);
                continue;
            }

            point = findImage(this.capture, icon, 0.95);
            if (null === point) {
                log("未找到匹配的图片1")
                times++;
                continue;
            } else {
                log("找到了匹配的图片")
            }
            log(point)

            this.robot.click(point.x, point.y);
            return true;
        }

        return false;
    }
```
主要核心是 findImage函数，第一个参数是截图，第二个参数是小图，小图自己先截图，然后利用一些裁剪工具截取核心部分，然后我们就是在截图里面寻找这个小图的坐标位置，然后利用机器人去点击。

### 5.2 偷能量
这里我们需要识别能量球，具体逻辑和上面类似。
```
 /**
    * 抓能量
    * @param  icon 
    */
this.handleEnergy = function (icon) {
    log("开始寻找有能量的人了")
    sleep(1000);
    var times = 0;
    while (times < this.options.max_retry_times) {
        // 截图
        this.capture = captureScreen();
        if (null === this.capture) {
            toastLog("截图失败");
            times++;
            sleep(200);
            continue;
        }

        point = findImage(this.capture, icon, 0.9);

        if (null === point) {
            log("未找到匹配的图片")
            times++;
            continue;
        } else {
            log("找到了匹配的图片")
        }
        log(point)

        this.robot.click(point.x, point.y);
        return true;
    }

    log("没有了哦")
    return false;
}
```
主要也是通过findImage识别到能量球的坐标位置，然后点击，最好是有个一秒延迟，更加流畅。

## 6 总结
1.这里通过一个蚂蚁森林偷取能量的简单案例，清晰地认识到自动化脚本的基本流程，以及相关知识点。
2.锁屏处理一般会根据机型可能会采用不同方案，密码的话，可以通过desc识别数字，然后模拟点击。
3.图片识别的话，需要截图权限，一般流程是截图后，然后再截图里面选择小图的大致坐标位置，然后点击坐标实现点击图片效果。
4.然后整体流程有了一定认识，主要就是通过一些方法函数，找到控件，然后模拟点击，可以用控件点击也可以用机器模拟点击坐标，最后一定要注意图片的回收，防止内存溢出。
