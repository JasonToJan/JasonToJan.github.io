---
title: Android 集成 高德地图API
date: 2023-02-10 10:59:57
top: false
cover: false
toc: true
mathjax: true
tags:
- Android 高德地图
categories:
- Android
---

## 1 参考文档
> Github源码地址：[https://github.com/lilongweidev/GaodeMapDemo](https://github.com/lilongweidev/GaodeMapDemo)
博主博客地址: 
[Android 高德地图API（详细步骤+源码）7](https://cloud.tencent.com/developer/article/1797654?areaSource=104001.113&traceId=nfv8q8EbnuT49lmQX8eK1)
[Android 高德地图API（详细步骤+源码）6](https://cloud.tencent.com/developer/article/1797651?areaSource=104001.114&traceId=nfv8q8EbnuT49lmQX8eK1)
[Android 高德地图API（详细步骤+源码）5](https://cloud.tencent.com/developer/article/1797622?areaSource=104001.115&traceId=nfv8q8EbnuT49lmQX8eK1)
[Android 高德地图API（详细步骤+源码）4](https://cloud.tencent.com/developer/article/1797621?areaSource=104001.116&traceId=nfv8q8EbnuT49lmQX8eK1)
[Android 高德地图API（详细步骤+源码）3](https://cloud.tencent.com/developer/article/1792227?areaSource=104001.121&traceId=nfv8q8EbnuT49lmQX8eK1)
[Android 高德地图API（详细步骤+源码）2](https://cloud.tencent.com/developer/article/1789741?areaSource=104001.122&traceId=nfv8q8EbnuT49lmQX8eK1)
[Android 高德地图API（详细步骤+源码）1](https://cloud.tencent.com/developer/article/1789740?areaSource=104001.123&traceId=nfv8q8EbnuT49lmQX8eK1)

## 2 集成步骤

### 2.1 创建开发者账号配置应用
直接在 [高德开放平台](https://lbs.amap.com/) 注册一个账号，新建一个应用。

这里添加应用的key的时候，需要两个SHA1，怎么获取呢？
* 获取调试版安全码SHA1
AS的右侧边栏，点击Gradle → app → android → signingReport，最后双击signingReport。

* 获取发布版安全帽SHA1
这个需要根据发布签名来，比如我们拿到一个jks文件，通过以下命令来获取：
```
keytool -list -v -keystore 后面加个空格 再跟上你打正式包后的 jks 文件完整地址
```

### 2.2 本地集成方式
首先在这里下载sdk
[Android地图SDK 一键下载](https://a.amap.com/lbs/static/zip/AMap_Android_SDK_All.zip)
这个包括：开发包（2D地图包、3D地图包、搜索包）、示例代码、开发文档（2D地图、3D地图、搜索服务）等。

下载好后放在app模块的libs文件夹下，没有则新建。

然后打开你的app下的build.gradle文件，在android闭包下添加：
```
sourceSets {
    main{
        jniLibs.srcDirs = ['libs']
    }
}
```
这样程序就能访问到这个文件夹下的sdk了。

然后配置下AndroidManifest.xml文件：
打开AndroidManifest.xml，首先在application标签下添加定位服务：
```
<!--定位service-->
<service android:name="com.amap.api.location.APSService"/>
```

然后添加权限,有的则无需重复添加：
```
<!--用于访问网络，网络定位需要上网-->
<uses-permission android:name="android.permission.INTERNET" />
<!--用于读取手机当前的状态-->
<uses-permission android:name="android.permission.READ_PHONE_STATE" />
<!--用于写入缓存数据到扩展存储卡-->
<uses-permission android:name="android.permission.WRITE_EXTERNAL_STORAGE" />
<!--用于申请调用A-GPS模块-->
<uses-permission android:name="android.permission.ACCESS_LOCATION_EXTRA_COMMANDS" />
<!--用于进行网络定位-->
<uses-permission android:name="android.permission.ACCESS_COARSE_LOCATION" />
<!--用于访问GPS定位-->
<uses-permission android:name="android.permission.ACCESS_FINE_LOCATION" />
<!--用于获取运营商信息，用于支持提供运营商信息相关的接口-->
<uses-permission android:name="android.permission.ACCESS_NETWORK_STATE" />
<!--用于访问wifi网络信息，wifi信息会用于进行网络定位-->
<uses-permission android:name="android.permission.ACCESS_WIFI_STATE" />
<!--用于获取wifi的获取权限，wifi信息会用来进行网络定位-->
<uses-permission android:name="android.permission.CHANGE_WIFI_STATE" />
```

然后在application标签下添加高德的key：
```
 <meta-data
            android:name="com.amap.api.v2.apikey"
            android:value="d3347ee0f2928f9a0c199cae009ae7f7" />
```

### 2.3 远程集成方式
这里通过远程依赖直接下载依赖包，可以在app模块的dependencies下方填写需要的依赖：
|  SDK   | 引入代码  |
|  ----  | ----  |
| 3D地图  | 'com.amap.api:3dmap:latest.integration' |
| 2D地图  | 'com.amap.api:map2d:latest.integration' |
| 导航  | 'com.amap.api:navi-3dmap:latest.integration‘ |
| 搜索  | 'com.amap.api:search:latest.integration' |
| 定位  | 'com.amap.api:location:latest.integration'|

然后最好设置下ndk:
```
ndk {
    //设置支持的SO库架构（开发者可以根据需要，选择一个或多个平台的so）
    abiFilters "armeabi", "armeabi-v7a", "arm64-v8a", "x86","x86_64"
}
```
一般情况，设置一个arm64-v8a就可以了，目前主流手机都支持。

注意：
> 1、3D地图 SDK 和导航 SDK，5.0.0 版本以后全面支持多平台 so 库(armeabi、armeabi-v7a、arm64-v8a、x86、x86_64)，开发者可以根据需要选择。同时还需要注意的是：如果您涉及到新旧版本更替请移除旧版本的 so 库之后替换新版本 so 库到工程中。
2、navi导航SDK 5.0.0以后版本包含了3D地图SDK，所以请不要同时引入 map3d 和 navi SDK。
3、如果build失败提示com.amap.api:XXX:X.X.X 找不到，请确认拼写及版本号是否正确，如果访问不到maven仓库尝试一下配置下阿里云镜像。
4、依照上述方法引入 SDK 以后，不需要在libs文件夹下导入对应SDK的 so 和 jar 包，会有冲突。

## 3 获取当前定位信息

### 3.1 申请定位权限
首先可以考虑引入官方的
[easypermissions](https://github.com/googlesamples/easypermissions) 进行动态权限申请。
需要引入此依赖：
```
implementation 'pub.devrel:easypermissions:3.0.0'
```

申请权限：
```Java
/**
    * 动态请求权限
    */
@AfterPermissionGranted(REQUEST_PERMISSIONS)
private void requestPermission() {
    String[] permissions = {
            Manifest.permission.ACCESS_COARSE_LOCATION,
            Manifest.permission.ACCESS_FINE_LOCATION,
            Manifest.permission.READ_PHONE_STATE,
            Manifest.permission.WRITE_EXTERNAL_STORAGE
    };

    if (EasyPermissions.hasPermissions(this, permissions)) {
        //true 有权限 开始定位
        //showMsg("已获得权限，可以定位啦！");
        Log.d(TAG, "已获得权限，可以定位啦！");
        //启动定位
        mLocationClient.startLocation();
    } else {
        //false 无权限
        EasyPermissions.requestPermissions(this, "需要权限", REQUEST_PERMISSIONS, permissions);
    }
}
```

然后需要在Activity的onRequestPermissionsResult获取请求权限回调：
```Java
/**
    * 请求权限结果
    * @param requestCode
    * @param permissions
    * @param grantResults
    */
@Override
public void onRequestPermissionsResult(int requestCode, @NonNull String[] permissions, @NonNull int[] grantResults) {
    super.onRequestPermissionsResult(requestCode, permissions, grantResults);
    //设置权限请求结果
    EasyPermissions.onRequestPermissionsResult(requestCode, permissions, grantResults, this);
}
```

### 3.2 初始化定位

变量声明：
```Java
//声明AMapLocationClient类对象
public AMapLocationClient mLocationClient = null;
//声明AMapLocationClientOption对象
public AMapLocationClientOption mLocationOption = null;
```

变量初始化
```Java
/**
    * 初始化定位
    */
private void initLocation() {
    //初始化定位
    mLocationClient = new AMapLocationClient(getApplicationContext());
    //设置定位回调监听
    mLocationClient.setLocationListener(this);
    //初始化AMapLocationClientOption对象
    mLocationOption = new AMapLocationClientOption();
    //设置定位模式为AMapLocationMode.Hight_Accuracy，高精度模式。
    mLocationOption.setLocationMode(AMapLocationClientOption.AMapLocationMode.Hight_Accuracy);
    //获取最近3s内精度最高的一次定位结果：
    //设置setOnceLocationLatest(boolean b)接口为true，启动定位时SDK会返回最近3s内精度最高的一次定位结果。如果设置其为true，setOnceLocation(boolean b)接口也会被设置为true，反之不会，默认为false。
    mLocationOption.setOnceLocationLatest(true);
    //设置是否返回地址信息（默认返回地址信息）
    mLocationOption.setNeedAddress(true);
    //设置定位请求超时时间，单位是毫秒，默认30000毫秒，建议超时时间不要低于8000毫秒。
    mLocationOption.setHttpTimeOut(20000);
    //关闭缓存机制，高精度定位会产生缓存。
    mLocationOption.setLocationCacheEnable(false);
    //给定位客户端对象设置定位参数
    mLocationClient.setLocationOption(mLocationOption);
}
```

开始定位：
这里通过 `mLocationClient.startLocation();` 去开始定位，结果会通过回调传给activity。

```Java 
@Override
public void onLocationChanged(AMapLocation aMapLocation) {
    if (aMapLocation != null) {
        if (aMapLocation.getErrorCode() == 0) {
            //地址
            String address = aMapLocation.getAddress();
            //城市赋值
            city = aMapLocation.getCity();
            //获取纬度
            double latitude = aMapLocation.getLatitude();
            //获取经度
            double longitude = aMapLocation.getLongitude();

            Log.d("MainActivity", aMapLocation.getCity());
            showMsg(address);

            //停止定位后，本地定位服务并不会被销毁
            mLocationClient.stopLocation();

            //显示地图定位结果
            if (mListener != null) {
                // 显示系统图标
                mListener.onLocationChanged(aMapLocation);
            }

            //显示浮动按钮
            fabPOI.show();
            //赋值
            cityCode = aMapLocation.getCityCode();
        } else {
            //定位失败时，可通过ErrCode（错误码）信息来确定失败的原因，errInfo是错误信息，详见错误码表。
            Log.e("AmapError", "location Error, ErrCode:"
                    + aMapLocation.getErrorCode() + ", errInfo:"
                    + aMapLocation.getErrorInfo());
        }
    }
}
```
获取到定位结果后，需要我们解析一下。
同时需要停止定位。

同时需要再onDestory里面进行销毁引用：
```Java
@Override
protected void onDestroy() {
    super.onDestroy();
    //销毁定位客户端，同时销毁本地定位服务。
    if (mLocationClient != null) {
        mLocationClient.onDestroy();
    }
    //在activity执行onDestroy时执行mMapView.onDestroy()，销毁地图
    mapView.onDestroy();
}
```

## 4 显示地图

### 4.1 添加MapView
```
<?xml version="1.0" encoding="utf-8"?>
<RelativeLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:gravity="center"
    tools:context=".MainActivity">

    <!--地图-->
    <com.amap.api.maps.MapView
        android:id="@+id/map_view"
        android:layout_width="match_parent"
        android:layout_height="match_parent"/>

</RelativeLayout>
```

简单看下源码：
```Java
public class MapView extends FrameLayout implements BaseMapView {
    private IMapFragmentDelegate mapFragmentDelegate;
    private AMap aMap;
    private int visibility = 0;

    public MapView(Context var1) {
        super(var1);
        this.getMapFragmentDelegate().setContext(var1);
    }

    public MapView(Context var1, AttributeSet var2) {
```
没想到地图是用FrameLayout，我以为是用GLSurfaceView绘制的呢。
主要逻辑它通过这个代理来完成的。

### 4.2 同步生命周期
然后需要在activity的生命周期里面同步下mapView的生命周期。
```Java
  @Override
    protected void onDestroy() {
        super.onDestroy();
        //销毁定位客户端，同时销毁本地定位服务。
        if (mLocationClient != null) {
            mLocationClient.onDestroy();
        }
        //在activity执行onDestroy时执行mMapView.onDestroy()，销毁地图
        mapView.onDestroy();
    }

    @Override
    protected void onResume() {
        super.onResume();
        //在activity执行onResume时执行mMapView.onResume ()，重新绘制加载地图
        mapView.onResume();
    }

    @Override
    protected void onPause() {
        super.onPause();
        //在activity执行onPause时执行mMapView.onPause ()，暂停地图的绘制
        mapView.onPause();
    }

    @Override
    protected void onSaveInstanceState(Bundle outState) {
        super.onSaveInstanceState(outState);
        //在activity执行onSaveInstanceState时执行mMapView.onSaveInstanceState (outState)，保存地图当前的状态
        mapView.onSaveInstanceState(outState);
    }
```

此时可以显示北京地图了。


## 5 显示当前定位地图

### 5.1 相关成员变量
```Java
//地图控制器
private AMap aMap = null;
//位置更改监听
private OnLocationChangedListener mListener;
```

### 5.2 初始化地图控制器
在onCreate中执行如下代码：
```Java
private void initMap(Bundle savedInstanceState) {
    mapView = findViewById(R.id.map_view);
    //在activity执行onCreate时执行mMapView.onCreate(savedInstanceState)，创建地图
    mapView.onCreate(savedInstanceState);
    //初始化地图控制器对象
    aMap = mapView.getMap();

    // 设置定位监听
    aMap.setLocationSource(this);
    // 设置为true表示显示定位层并可触发定位，false表示隐藏定位层并不可触发定位，默认是false
    aMap.setMyLocationEnabled(true);
}
```

### 5.3 给地图注入监听
注意到上面的 `aMap = aMap.setLocationSource(this);`
这个this说明这个Activity要实现回调方法：
```Java
public interface LocationSource {
    void activate(LocationSource.OnLocationChangedListener var1);

    void deactivate();

    public interface OnLocationChangedListener {
        void onLocationChanged(Location var1);
    }
}
```
这个是高德提供的定位接口。

我们需要在activity里面去实现这个接口：
```Java
/**
 * 激活定位
 */
@Override
public void activate(OnLocationChangedListener onLocationChangedListener) {
    mListener = onLocationChangedListener;
    if (mLocationClient == null) {
        mLocationClient.startLocation();//启动定位
    }
}

/**
 * 停止定位
 */
@Override 
public void deactivate() {
    mListener = null;
    if (mLocationClient != null) {
        mLocationClient.stopLocation();
        mLocationClient.onDestroy();
    }
    mLocationClient = null;
}
```

这里主要是在activate里面接收回调，拿到地址后，通过这个回调给map。这样map就指向定位地址了。

## 6 地图设置

### 6.1 修改自定义定位图标
```Java
//定位样式
private MyLocationStyle myLocationStyle = new MyLocationStyle();
```

初始化地图：
```Java
// 自定义定位蓝点图标
myLocationStyle.myLocationIcon(BitmapDescriptorFactory.fromResource(R.drawable.gps_point));
// 自定义精度范围的圆形边框颜色  都为0则透明
myLocationStyle.strokeColor(Color.argb(0, 0, 0, 0));
// 自定义精度范围的圆形边框宽度  0 无宽度
myLocationStyle.strokeWidth(0);
// 设置圆形的填充颜色  都为0则透明
myLocationStyle.radiusFillColor(Color.argb(0, 0, 0, 0));

//设置定位蓝点的Style
aMap.setMyLocationStyle(myLocationStyle);
```

### 6.2 设置缩放等级
```Java
//设置最小缩放等级为16 ，缩放级别范围为[3, 20]
aMap.setMinZoomLevel(12);
```
### 6.3 开启室内地图
```Java
//开启室内地图
aMap.showIndoorMap(true);
```

### 6.4 地图控件设置
```Java
//定义一个UiSettings对象
private UiSettings mUiSettings;
```

然后可以隐藏缩放按钮：
```Java
//实例化UiSettings类对象
mUiSettings = aMap.getUiSettings();
//隐藏缩放按钮
mUiSettings.setZoomControlsEnabled(false);
```

可以设置比例尺控件：
```Java
//显示比例尺 默认不显示
mUiSettings.setScaleControlsEnabled(true);
```

## 7 获取POI数据

POI是什么呢？
> POI (Point of Interest，兴趣点)。在地图表达中，一个 POI 可代表一栋大厦、一家商铺、一处景点等等。通过POI搜索，完成找餐馆、找景点、找厕所等等的功能。

回到首页，声明几个POI相关变量：
```Java
//POI查询对象
private PoiSearch.Query query;
//POI搜索对象
private PoiSearch poiSearch;
//城市码
private String cityCode = null;
//浮动按钮
private FloatingActionButton fabPOI;
```

在定位回调中，成功后，可显示浮动按钮。

我们设定，点击浮动按钮，去查询附近POI数据，
这里点击事件走一下逻辑：
```Java
public void queryPOI(View view) {
    //构造query对象
    query = new PoiSearch.Query("购物", "", cityCode);
    // 设置每页最多返回多少条poiitem
    query.setPageSize(10);
    //设置查询页码
    query.setPageNum(1);
    //构造 PoiSearch 对象
    try {
        poiSearch = new PoiSearch(this, query);
    } catch (AMapException e) {
        e.printStackTrace();
    }
    //设置搜索回调监听
    poiSearch.setOnPoiSearchListener(this);
    //发起搜索附近POI异步请求
    poiSearch.searchPOIAsyn();
}
```
这里设置了回调，所以在activity里面，应该要实现搜索回调接口。

该接口为：
```Java
public interface OnPoiSearchListener {
    void onPoiSearched(PoiResult var1, int var2);

    void onPoiItemSearched(PoiItem var1, int var2);
}
```

所以我们在首页里面去实现：
```Java
/**
    * POI搜索返回
    *
    * @param poiResult POI所有数据
    * @param i
    */
@Override
public void onPoiSearched(PoiResult poiResult, int i) {
    //解析result获取POI信息

    //获取POI组数列表
    ArrayList<PoiItem> poiItems = poiResult.getPois();
    for (PoiItem poiItem : poiItems) {
        Log.d("MainActivity", " Title：" + poiItem.getTitle() + " Snippet：" + poiItem.getSnippet());
    }
}

/**
    * POI中的项目搜索返回
    *
    * @param poiItem 获取POI item
    * @param i
    */
@Override
public void onPoiItemSearched(PoiItem poiItem, int i) {

}
```
这里我们只是打印了附近POI数据。

## 8 地图点击事件

### 8.1 点击事件
```Java
//设置地图点击事件
aMap.setOnMapClickListener(this);
```

这里需要Activity实现一下接口 AMap.OnMapClickListener
```Java
/**
    * 地图单击事件
    *
    * @param latLng
    */
@Override
public void onMapClick(LatLng latLng) {
    //通过经纬度获取地址
    //latlonToAddress(latLng);
    //添加标点
    addMarker(latLng);
    //改变地图中心点
    updateMapCenter(latLng);
}
```

```Java
/**
    * 添加地图标点
    *
    * @param latLng
    */
private void addMarker(LatLng latLng) {
    //显示浮动按钮
    fabClearMarker.show();
    //添加标点
    Marker marker = aMap.addMarker(new MarkerOptions()
            .draggable(true)//可拖动
            .position(latLng)
            .title("标题")
            .snippet("详细信息"));

    //绘制Marker时显示InfoWindow
    //marker.showInfoWindow();

    //设置标点的绘制动画效果
    Animation animation = new RotateAnimation(marker.getRotateAngle(), marker.getRotateAngle() + 180, 0, 0, 0);
    long duration = 1000L;
    animation.setDuration(duration);
    animation.setInterpolator(new LinearInterpolator());

    marker.setAnimation(animation);
    marker.startAnimation();

    markerList.add(marker);
}
```
上面添加了一个Marker标点，然后设置了动画旋转效果。

然后地图好像以marker居中了：
```Java
/**
    * 改变地图中心位置
    * @param latLng 位置
    */
private void updateMapCenter(LatLng latLng) {
    // CameraPosition 第一个参数： 目标位置的屏幕中心点经纬度坐标。
    // CameraPosition 第二个参数： 目标可视区域的缩放级别
    // CameraPosition 第三个参数： 目标可视区域的倾斜度，以角度为单位。
    // CameraPosition 第四个参数： 可视区域指向的方向，以角度为单位，从正北向顺时针方向计算，从0度到360度
    CameraPosition cameraPosition = new CameraPosition(latLng, 16, 30, 0);
    //位置变更
    CameraUpdate cameraUpdate = CameraUpdateFactory.newCameraPosition(cameraPosition);
    //改变位置
    //aMap.moveCamera(cameraUpdate);
    //带动画的移动
    aMap.animateCamera(cameraUpdate);

}
```

### 8.2 设置地图长按事件

```Java
//设置地图长按事件
aMap.setOnMapLongClickListener(this);
```

所以需要Activity实现接口：
```Java
public interface OnMapLongClickListener {
    void onMapLongClick(LatLng var1);
}
```

具体实现如下：
```Java
/**
    * 地图长按事件
    *
    * @param latLng
    */
@Override
public void onMapLongClick(LatLng latLng) {
    //通过经纬度获取地址
    latlonToAddress(latLng);
}
```
这里通过点击的经纬度获取地址：
```Java
/**
    * 通过经纬度获取地址
    *
    * @param latLng
    */
private void latlonToAddress(LatLng latLng) {
    //位置点  通过经纬度进行构建
    LatLonPoint latLonPoint = new LatLonPoint(latLng.latitude, latLng.longitude);
    //逆编码查询  第一个参数表示一个Latlng，第二参数表示范围多少米，第三个参数表示是火系坐标系还是GPS原生坐标系
    RegeocodeQuery query = new RegeocodeQuery(latLonPoint, 20, GeocodeSearch.AMAP);
    //异步获取地址信息
    geocodeSearch.getFromLocationAsyn(query);
}
```

因为前面初始化的时候给gecodeSearch设置了监听
```Java
geocodeSearch.setOnGeocodeSearchListener(this);
```

所以当我们走异步获取地址信息的时候，结果会回调到这个接口上：
```Java
public interface OnGeocodeSearchListener {
    void onRegeocodeSearched(RegeocodeResult var1, int var2);

    void onGeocodeSearched(GeocodeResult var1, int var2);
}
```

所以我们Activity也需要实现这两个方法：
```Java
/**
    * 坐标转地址
    *
    * @param regeocodeResult
    * @param rCode
    */
@Override
public void onRegeocodeSearched(RegeocodeResult regeocodeResult, int rCode) {
    //解析result获取地址描述信息
    if (rCode == PARSE_SUCCESS_CODE) {
        RegeocodeAddress regeocodeAddress = regeocodeResult.getRegeocodeAddress();
        //显示解析后的地址
        Log.d("MainActivity", regeocodeAddress.getFormatAddress());
        //showMsg("地址：" + regeocodeAddress.getFormatAddress());

        LatLonPoint latLonPoint = regeocodeResult.getRegeocodeQuery().getPoint();
        LatLng latLng = new LatLng(latLonPoint.getLatitude(), latLonPoint.getLongitude());
        addMarker(latLng);
    } else {
        showMsg("获取地址失败");
    }

}

/**
    * 地址转坐标
    *
    * @param geocodeResult
    * @param rCode
    */
@Override
public void onGeocodeSearched(GeocodeResult geocodeResult, int rCode) {
    if (rCode == PARSE_SUCCESS_CODE) {
        List<GeocodeAddress> geocodeAddressList = geocodeResult.getGeocodeAddressList();
        if (geocodeAddressList != null && geocodeAddressList.size() > 0) {
            LatLonPoint latLonPoint = geocodeAddressList.get(0).getLatLonPoint();
            //显示解析后的坐标
            showMsg("坐标：" + latLonPoint.getLongitude() + "，" + latLonPoint.getLatitude());
        }

    } else {
        showMsg("获取坐标失败");
    }
}
```
这里获取到地址后，打印下来。

这个GeocodeSearch主要是用来逆地址编码的，因为搜索结果只返回经纬度，我们需要通过经纬度获取具体地址名称，这时候就需要使用这个GeocodeSearch类来帮忙处理了。

### 8.3 地址编码

地址编码，就是地址转坐标。
> 比如说你到一个景点去游玩，不知道路线只知道景点名，那么这个时候通常你会在导航软件中输入这个景点名，然后搜索出前往的路线及搭乘的交通工具。此时，导航软件会将你输入的地址转成经纬度坐标，然后通过你当前的所在地坐标计算距离，获取两点之间的交通情况，然后规划路线。

所以要搞一个搜索框，这里直接搞一个EditText进去。

设置点击事件，以及键盘监听 EditText.OnKeyListener。
```Java
/**
    * 键盘点击
    *
    * @param v
    * @param keyCode
    * @param event
    * @return
    */
@Override
public boolean onKey(View v, int keyCode, KeyEvent event) {
    if (keyCode == KeyEvent.KEYCODE_ENTER && event.getAction() == KeyEvent.ACTION_UP) {
        //获取输入框的值
        String address = etAddress.getText().toString().trim();
        if (address == null || address.isEmpty()) {
            showMsg("请输入地址");
        } else {
            InputMethodManager imm = (InputMethodManager) getSystemService(Context.INPUT_METHOD_SERVICE);
            //隐藏软键盘
            imm.hideSoftInputFromWindow(getWindow().getDecorView().getWindowToken(), 0);

            // name表示地址，第二个参数表示查询城市，中文或者中文全拼，citycode、adcode
            GeocodeQuery query = new GeocodeQuery(address, city);
            geocodeSearch.getFromLocationNameAsyn(query);
        }
        return true;
    }
    return false;
}
```
这里点击搜索时，判空，然后通过 GeocodeSearch实例的getFromLocationNameAsyn来通过名称搜索地址。

### 8.4 清空marker
```Java
/**
    * 清空地图Marker
    *
    * @param view
    */
public void clearAllMarker(View view) {
    if (markerList != null && markerList.size()>0){
        for (Marker markerItem : markerList) {
            markerItem.remove();
        }
    }
    fabClearMarker.hide();
}
```
这里是存储了Marker到一个集合，清除的时候，调用marker的remove方法即可。

### 8.5 Marker的点击和拖拽

需要实现下 Amap.OnMarkerClickListener 接口。

然后需要配置下map的marker点击事件：
```Java
//设置地图Marker点击事件
aMap.setOnMarkerClickListener(this);
```

具体实现如下：
```Java
/**
    * Marker点击事件
    *
    * @param marker
    * @return
    */
@Override
public boolean onMarkerClick(Marker marker) {
    //showMsg("点击了标点");
    //显示InfoWindow
    if (!marker.isInfoWindowShown()) {
        //显示
        marker.showInfoWindow();
    } else {
        //隐藏
        marker.hideInfoWindow();
    }
    return true;
}
```

拖拽比较类似，
先配置map的markerDragListener事件：
```Java
//设置地图Marker拖拽事件
aMap.setOnMarkerDragListener(this);
```

然后实现这个接口：
```Java
/**
    * 开始拖动
    *
    * @param marker
    */
@Override
public void onMarkerDragStart(Marker marker) {
    Log.d(TAG, "开始拖动");
}

/**
    * 拖动中
    *
    * @param marker
    */
@Override
public void onMarkerDrag(Marker marker) {
    Log.d(TAG, "拖动中");
}

/**
    * 拖动完成
    *
    * @param marker
    */
@Override
public void onMarkerDragEnd(Marker marker) {
    Log.d(TAG, "拖动完成");
}
```

但是实现需要配置marker可点拖动状态：
```Java
Marker marker = aMap.addMarker(new MarkerOptions()
            .draggable(true)//可拖动
            .position(latLng)
            .title("标题")
            .snippet("详细信息"));
```

### 8.6 绘制InfoWindow
```Java
Marker marker = aMap.addMarker(new MarkerOptions()
            .draggable(true)//可拖动
            .position(latLng)
            .title("标题")
            .snippet("详细信息"));
```
这里配置title和snippet，其实就是一个窗体。

然后通过marker.showInfoWindow或者hideInfoWindow方法实现显示或隐藏。

如何自定义InfoWindow呢？
首先需要在Activity中实现 AMAP.InfoWindowAdapter接口。
```Java
/**
    * 修改背景
    *
    * @param marker
    */
@Override
public View getInfoWindow(Marker marker) {
    View infoWindow = getLayoutInflater().inflate(
            R.layout.custom_info_window, null);

    render(marker, infoWindow);
    return infoWindow;
}

/**
    * 修改内容
    *
    * @param marker
    * @return
    */
@Override
public View getInfoContents(Marker marker) {
    View infoContent = getLayoutInflater().inflate(
            R.layout.custom_info_contents, null);
    render(marker, infoContent);
    return infoContent;
}
```
通过这两个方法可以自定义infoWindow。

其次infoWindow也是可以点击的，
我们需要再给Activity实现一个接口，AMap.OnInfoWindowClickListener,
```Java
//设置InfoWindow点击事件
aMap.setOnInfoWindowClickListener(this);
```

然后继续实现接口：
```Java
@Override
public void onInfoWindowClick(Marker marker) {
    showMsg("弹窗内容：标题：" + marker.getTitle() + "\n片段：" + marker.getSnippet());
}
```

### 8.7 如何改变地图中心点
希望点击一下就可以移动到所在地，这其实是比较容易做到的，回顾我们现在是一进入地图就会定位到当前所在地，而当我点击地图上其他位置时，会增加一个标点，而我们要做的就是把这个标点作为地图中心，然后移动地图位置即可。

```Java
/**
    * 改变地图中心位置
    * @param latLng 位置
    */
private void updateMapCenter(LatLng latLng) {
    // CameraPosition 第一个参数： 目标位置的屏幕中心点经纬度坐标。
    // CameraPosition 第二个参数： 目标可视区域的缩放级别
    // CameraPosition 第三个参数： 目标可视区域的倾斜度，以角度为单位。
    // CameraPosition 第四个参数： 可视区域指向的方向，以角度为单位，从正北向顺时针方向计算，从0度到360度
    CameraPosition cameraPosition = new CameraPosition(latLng, 16, 30, 0);
    //位置变更
    CameraUpdate cameraUpdate = CameraUpdateFactory.newCameraPosition(cameraPosition);
    //改变位置
    aMap.moveCamera(cameraUpdate);
}
```
用这个方法即可。

## 9 出行路线规划

### 9.1 步行路线规划
```Java
//起点
private LatLonPoint mStartPoint;
//终点
private LatLonPoint mEndPoint;
```

帮助类：
```Java
package com.llw.mapdemo.util;

public class ChString {
    public static final String Kilometer = "\u516c\u91cc";// "公里";
    public static final String Meter = "\u7c73";// "米";
    public static final String ByFoot = "\u6b65\u884c";// "步行";
    public static final String To = "\u53bb\u5f80";// "去往";
    public static final String Station = "\u8f66\u7ad9";// "车站";
    public static final String TargetPlace = "\u76ee\u7684\u5730";// "目的地";
    public static final String StartPlace = "\u51fa\u53d1\u5730";// "出发地";
    public static final String About = "\u5927\u7ea6";// "大约";
    public static final String Direction = "\u65b9\u5411";// "方向";

    public static final String GetOn = "\u4e0a\u8f66";// "上车";
    public static final String GetOff = "\u4e0b\u8f66";// "下车";
    public static final String Zhan = "\u7ad9";// "站";

    public static final String cross = "\u4ea4\u53c9\u8def\u53e3"; // 交叉路口
    public static final String type = "\u7c7b\u522b"; // 类别
    public static final String address = "\u5730\u5740"; // 地址
    public static final String PrevStep = "\u4e0a\u4e00\u6b65";
    public static final String NextStep = "\u4e0b\u4e00\u6b65";
    public static final String Gong = "\u516c\u4ea4";
    public static final String ByBus = "\u4e58\u8f66";
    public static final String Arrive = "\u5230\u8FBE";// 到达
}
```

然后下面是地图帮助类：
```Java
package com.llw.mapdemo.util;

import com.amap.api.maps.model.LatLng;
import com.amap.api.services.core.LatLonPoint;

import java.text.DecimalFormat;

/**
 * 地图帮助类
 * @author llw
 */
public class MapUtil {

    /**
     * 把LatLng对象转化为LatLonPoint对象
     */
    public static LatLonPoint convertToLatLonPoint(LatLng latLng) {
        return new LatLonPoint(latLng.latitude, latLng.longitude);
    }

    /**
     * 把LatLonPoint对象转化为LatLon对象
     */
    public static LatLng convertToLatLng(LatLonPoint latLonPoint) {
        return new LatLng(latLonPoint.getLatitude(), latLonPoint.getLongitude());
    }

    public static String getFriendlyTime(int second) {
        if (second > 3600) {
            int hour = second / 3600;
            int miniate = (second % 3600) / 60;
            return hour + "小时" + miniate + "分钟";
        }
        if (second >= 60) {
            int miniate = second / 60;
            return miniate + "分钟";
        }
        return second + "秒";
    }

    public static String getFriendlyLength(int lenMeter) {
        if (lenMeter > 10000) // 10 km
        {
            int dis = lenMeter / 1000;
            return dis + ChString.Kilometer;
        }

        if (lenMeter > 1000) {
            float dis = (float) lenMeter / 1000;
            DecimalFormat fnum = new DecimalFormat("##0.0");
            String dstr = fnum.format(dis);
            return dstr + ChString.Kilometer;
        }

        if (lenMeter > 100) {
            int dis = lenMeter / 50 * 50;
            return dis + ChString.Meter;
        }

        int dis = lenMeter / 10 * 10;
        if (dis == 0) {
            dis = 10;
        }

        return dis + ChString.Meter;
    }

}
```

这里定位之后，设置一个起点。
```Java
//设置起点
mStartPoint = convertToLatLonPoint(new LatLng(latitude, longitude));
```
通过经纬度，构建LatLng对象，然后将LatLng转为LatLonPoint。最后赋值给起点mStartPoint。

点击地图，设置终点：
```Java
/**
    * 点击地图
    */
@Override
public void onMapClick(LatLng latLng) {
    //终点
    mEndPoint = convertToLatLonPoint(latLng);
}
```

搜索路线需要建立一个RouteSearch对象：
```Java
//路线搜索对象
private RouteSearch routeSearch;
```

然后初始化：
```Java
/**
    * 初始化路线
    */
private void initRoute() {
    try {
        routeSearch = new RouteSearch(this);
    } catch (AMapException e) {
        e.printStackTrace();
    }
    routeSearch.setRouteSearchListener(this);
}
```

这里设置了监听，所以需要重写几个路线规划的方法：


步行规划路径结果：
```Java
/**
    * 步行规划路径结果
    *
    * @param walkRouteResult 结果
    * @param code            结果码
    */
@Override
public void onWalkRouteSearched(WalkRouteResult walkRouteResult, int code) {
    aMap.clear();// 清理地图上的所有覆盖物
    if (code == AMapException.CODE_AMAP_SUCCESS) {
        if (walkRouteResult != null && walkRouteResult.getPaths() != null) {
            if (walkRouteResult.getPaths().size() > 0) {
                final WalkPath walkPath = walkRouteResult.getPaths().get(0);
                if (walkPath == null) {
                    return;
                }
                //绘制路线
                WalkRouteOverlay walkRouteOverlay = new WalkRouteOverlay(
                        this, aMap, walkPath,
                        walkRouteResult.getStartPos(),
                        walkRouteResult.getTargetPos());
                walkRouteOverlay.removeFromMap();
                walkRouteOverlay.addToMap();
                walkRouteOverlay.zoomToSpan();

                int dis = (int) walkPath.getDistance();
                int dur = (int) walkPath.getDuration();
                String des = MapUtil.getFriendlyTime(dur) + "(" + MapUtil.getFriendlyLength(dis) + ")";
                //显示步行花费时间
                tvTime.setText(des);
                bottomLayout.setVisibility(View.VISIBLE);
                //跳转到路线详情页面
                bottomLayout.setOnClickListener(new View.OnClickListener() {
                    @Override
                    public void onClick(View v) {
                        Intent intent = new Intent(RouteActivity.this,
                                RouteDetailActivity.class);
                        intent.putExtra("type", 0);
                        intent.putExtra("path", walkPath);
                        startActivity(intent);
                    }
                });
            } else if (walkRouteResult.getPaths() == null) {
                showMsg("对不起，没有搜索到相关数据！");
            }
        } else {
            showMsg("对不起，没有搜索到相关数据！");
        }
    } else {
        showMsg("错误码；" + code);
    }
}
```

怎么触发路线搜索：
当起点和终点通过地址转坐标方式，拿到地址后，就可以开始路线搜索了
```Java
if (mStartPoint != null && mEndPoint != null) {
    //开始路线搜索
    startRouteSearch();
}
```

然后开始路线搜索：
```Java
/**
    * 开始路线搜索
    */
private void startRouteSearch() {
    //在地图上添加起点Marker
    aMap.addMarker(new MarkerOptions()
            .position(convertToLatLng(mStartPoint))
            .icon(BitmapDescriptorFactory.fromResource(R.drawable.start)));
    //在地图上添加终点Marker
    aMap.addMarker(new MarkerOptions()
            .position(convertToLatLng(mEndPoint))
            .icon(BitmapDescriptorFactory.fromResource(R.drawable.end)));

    //搜索路线 构建路径的起终点
    final RouteSearch.FromAndTo fromAndTo = new RouteSearch.FromAndTo(
            mStartPoint, mEndPoint);

    //出行方式判断
    switch (TRAVEL_MODE) {
        case 0://步行
            //构建步行路线搜索对象
            RouteSearch.WalkRouteQuery query = new RouteSearch.WalkRouteQuery(fromAndTo, RouteSearch.WalkDefault);
            // 异步路径规划步行模式查询
            routeSearch.calculateWalkRouteAsyn(query);
            break;
        case 1://骑行
            //构建骑行路线搜索对象
            RouteSearch.RideRouteQuery rideQuery = new RouteSearch.RideRouteQuery(fromAndTo, RouteSearch.WalkDefault);
            //骑行规划路径计算
            routeSearch.calculateRideRouteAsyn(rideQuery);
            break;
        case 2://驾车
            //构建驾车路线搜索对象  剩余三个参数分别是：途经点、避让区域、避让道路
            RouteSearch.DriveRouteQuery driveQuery = new RouteSearch.DriveRouteQuery(fromAndTo, RouteSearch.WalkDefault, null, null, "");
            //驾车规划路径计算
            routeSearch.calculateDriveRouteAsyn(driveQuery);
            break;
        case 3://公交
            //构建驾车路线搜索对象 第三个参数表示公交查询城市区号，第四个参数表示是否计算夜班车，0表示不计算,1表示计算
            RouteSearch.BusRouteQuery busQuery = new RouteSearch.BusRouteQuery(fromAndTo, RouteSearch.BusLeaseWalk, city, 0);
            //公交规划路径计算
            routeSearch.calculateBusRouteAsyn(busQuery);
            break;
        default:
            break;
    }
}
```

好的，现在通过接口，拿到路线返回的结果了。现在需要对路线进行绘制。

需要先安排一个工具类：
```Java

import android.graphics.Bitmap;

import com.amap.api.maps.model.LatLng;
import com.amap.api.services.core.LatLonPoint;

import java.io.ByteArrayOutputStream;
import java.io.IOException;
import java.io.InputStream;
import java.util.ArrayList;
import java.util.List;

/**
 * 地图服务工具类
 */
class AMapServicesUtil {
	public static int BUFFER_SIZE = 2048;

	public static byte[] inputStreamToByte(InputStream in) throws IOException {

		ByteArrayOutputStream outStream = new ByteArrayOutputStream();
		byte[] data = new byte[BUFFER_SIZE];
		int count = -1;
		while ((count = in.read(data, 0, BUFFER_SIZE)) != -1){
			outStream.write(data, 0, count);
		}

		data = null;
		return outStream.toByteArray();
	}
	public static LatLonPoint convertToLatLonPoint(LatLng latlon) {
		return new LatLonPoint(latlon.latitude, latlon.longitude);
	}
	public static LatLng convertToLatLng(LatLonPoint latLonPoint) {
		return new LatLng(latLonPoint.getLatitude(), latLonPoint.getLongitude());
	}
	public static ArrayList<LatLng> convertArrList(List<LatLonPoint> shapes) {
		ArrayList<LatLng> lineShapes = new ArrayList<LatLng>();
		for (LatLonPoint point : shapes) {
			LatLng latLngTemp = AMapServicesUtil.convertToLatLng(point);
			lineShapes.add(latLngTemp);
		}
		return lineShapes;
	}
	public static Bitmap zoomBitmap(Bitmap bitmap, float res) {
		if (bitmap == null) {
			return null;
		}
		int width, height;
		width = (int) (bitmap.getWidth() * res);
		height = (int) (bitmap.getHeight() * res);
		Bitmap newbmp = Bitmap.createScaledBitmap(bitmap, width, height, true);
		return newbmp;
	}

}
```

图层叠加工具类：
```Java
import java.util.ArrayList;
import java.util.List;

import android.content.Context;
import android.graphics.Bitmap;
import android.graphics.Color;

import com.amap.api.maps.AMap;
import com.amap.api.maps.CameraUpdateFactory;
import com.amap.api.maps.model.BitmapDescriptor;
import com.amap.api.maps.model.BitmapDescriptorFactory;
import com.amap.api.maps.model.LatLng;
import com.amap.api.maps.model.LatLngBounds;
import com.amap.api.maps.model.Marker;
import com.amap.api.maps.model.MarkerOptions;
import com.amap.api.maps.model.Polyline;
import com.amap.api.maps.model.PolylineOptions;
import com.llw.mapdemo.R;

/**
 * 路线图层叠加
 */
public class RouteOverlay {
	protected List<Marker> stationMarkers = new ArrayList<Marker>();
	protected List<Polyline> allPolyLines = new ArrayList<Polyline>();
	protected Marker startMarker;
	protected Marker endMarker;
	protected LatLng startPoint;
	protected LatLng endPoint;
	protected AMap mAMap;
	private Context mContext;
	private Bitmap startBit, endBit, busBit, walkBit, driveBit;
	protected boolean nodeIconVisible = true;

	public RouteOverlay(Context context) {
		mContext = context;
	}

	/**
	 * 去掉BusRouteOverlay上所有的Marker。
	 * @since V2.1.0
	 */
	public void removeFromMap() {
		if (startMarker != null) {
			startMarker.remove();

		}
		if (endMarker != null) {
			endMarker.remove();
		}
		for (Marker marker : stationMarkers) {
			marker.remove();
		}
		for (Polyline line : allPolyLines) {
			line.remove();
		}
		destroyBit();
	}

	private void destroyBit() {
		if (startBit != null) {
			startBit.recycle();
			startBit = null;
		}
		if (endBit != null) {
			endBit.recycle();
			endBit = null;
		}
		if (busBit != null) {
			busBit.recycle();
			busBit = null;
		}
		if (walkBit != null) {
			walkBit.recycle();
			walkBit = null;
		}
		if (driveBit != null) {
			driveBit.recycle();
			driveBit = null;
		}
	}
	/**
	 * 给起点Marker设置图标，并返回更换图标的图片。如不用默认图片，需要重写此方法。
	 * @return 更换的Marker图片。
	 * @since V2.1.0
	 */
	protected BitmapDescriptor getStartBitmapDescriptor() {
		return BitmapDescriptorFactory.fromResource(R.drawable.amap_start);
	}
	/**
	 * 给终点Marker设置图标，并返回更换图标的图片。如不用默认图片，需要重写此方法。
	 * @return 更换的Marker图片。
	 * @since V2.1.0
	 */
	protected BitmapDescriptor getEndBitmapDescriptor() {
		return BitmapDescriptorFactory.fromResource(R.drawable.amap_end);
	}
	/**
	 * 给公交Marker设置图标，并返回更换图标的图片。如不用默认图片，需要重写此方法。
	 * @return 更换的Marker图片。
	 * @since V2.1.0
	 */
	protected BitmapDescriptor getBusBitmapDescriptor() {
		return BitmapDescriptorFactory.fromResource(R.drawable.amap_bus);
	}
	/**
	 * 给步行Marker设置图标，并返回更换图标的图片。如不用默认图片，需要重写此方法。
	 * @return 更换的Marker图片。
	 * @since V2.1.0
	 */
	protected BitmapDescriptor getWalkBitmapDescriptor() {
		return BitmapDescriptorFactory.fromResource(R.drawable.amap_man);
	}

	protected BitmapDescriptor getDriveBitmapDescriptor() {
		return BitmapDescriptorFactory.fromResource(R.drawable.amap_car);
	}

	protected void addStartAndEndMarker() {
		startMarker = mAMap.addMarker((new MarkerOptions())
				.position(startPoint).icon(getStartBitmapDescriptor())
				.title("\u8D77\u70B9"));
		// startMarker.showInfoWindow();

		endMarker = mAMap.addMarker((new MarkerOptions()).position(endPoint)
				.icon(getEndBitmapDescriptor()).title("\u7EC8\u70B9"));
		// mAMap.moveCamera(CameraUpdateFactory.newLatLngZoom(startPoint,
		// getShowRouteZoom()));
	}
	/**
	 * 移动镜头到当前的视角。
	 * @since V2.1.0
	 */
	public void zoomToSpan() {
		if (startPoint != null) {
			if (mAMap == null) {
				return;
			}
			try {
				LatLngBounds bounds = getLatLngBounds();
				mAMap.animateCamera(CameraUpdateFactory
						.newLatLngBounds(bounds, 100));
			} catch (Throwable e) {
				e.printStackTrace();
			}
		}
	}

	protected LatLngBounds getLatLngBounds() {
		LatLngBounds.Builder b = LatLngBounds.builder();
		b.include(new LatLng(startPoint.latitude, startPoint.longitude));
		b.include(new LatLng(endPoint.latitude, endPoint.longitude));
		return b.build();
	}
	/**
	 * 路段节点图标控制显示接口。
	 * @param visible true为显示节点图标，false为不显示。
	 * @since V2.3.1
	 */
	public void setNodeIconVisibility(boolean visible) {
		try {
			nodeIconVisible = visible;
			if (this.stationMarkers != null && this.stationMarkers.size() > 0) {
				for (int i = 0; i < this.stationMarkers.size(); i++) {
					this.stationMarkers.get(i).setVisible(visible);
				}
			}
		} catch (Throwable e) {
			e.printStackTrace();
		}
	}
	
	protected void addStationMarker(MarkerOptions options) {
		if(options == null) {
			return;
		}
		Marker marker = mAMap.addMarker(options);
		if(marker != null) {
			stationMarkers.add(marker);
		}
		
	}

	protected void addPolyLine(PolylineOptions options) {
		if(options == null) {
			return;
		}
		Polyline polyline = mAMap.addPolyline(options);
		if(polyline != null) {
			allPolyLines.add(polyline);
		}
	}
	
	protected float getRouteWidth() {
		return 18f;
	}

	protected int getWalkColor() {
		return Color.parseColor("#6db74d");
	}

	/**
	 * 自定义路线颜色。
	 * return 自定义路线颜色。
	 * @since V2.2.1
	 */
	protected int getBusColor() {
		return Color.parseColor("#537edc");
	}

	protected int getDriveColor() {
		return Color.parseColor("#537edc");
	}

	// protected int getShowRouteZoom() {
	// return 15;
	// }
}
```

步行路线图层类：
```Java
package com.llw.mapdemo.overlay;

import java.util.List;

import com.amap.api.maps.AMap;
import com.amap.api.maps.model.BitmapDescriptor;
import com.amap.api.maps.model.LatLng;
import com.amap.api.maps.model.MarkerOptions;
import com.amap.api.maps.model.PolylineOptions;
import com.amap.api.services.core.LatLonPoint;
import com.amap.api.services.route.WalkPath;
import com.amap.api.services.route.WalkStep;

import android.content.Context;

/**
 * 步行路线图层类。在高德地图API里，如果要显示步行路线规划，可以用此类来创建步行路线图层。如不满足需求，也可以自己创建自定义的步行路线图层。
 * @since V2.1.0
 */
public class WalkRouteOverlay extends RouteOverlay {

    private PolylineOptions mPolylineOptions;

    private BitmapDescriptor walkStationDescriptor= null;

    private WalkPath walkPath;
	/**
	 * 通过此构造函数创建步行路线图层。
	 * @param context 当前activity。
	 * @param amap 地图对象。
	 * @param path 步行路线规划的一个方案。详见搜索服务模块的路径查询包（com.amap.api.services.route）中的类 <strong><a href="../../../../../../Search/com/amap/api/services/route/WalkStep.html" title="com.amap.api.services.route中的类">WalkStep</a></strong>。
	 * @param start 起点。详见搜索服务模块的核心基础包（com.amap.api.services.core）中的类<strong><a href="../../../../../../Search/com/amap/api/services/core/LatLonPoint.html" title="com.amap.api.services.core中的类">LatLonPoint</a></strong>。
	 * @param end 终点。详见搜索服务模块的核心基础包（com.amap.api.services.core）中的类<strong><a href="../../../../../../Search/com/amap/api/services/core/LatLonPoint.html" title="com.amap.api.services.core中的类">LatLonPoint</a></strong>。
	 * @since V2.1.0
	 */
	public WalkRouteOverlay(Context context, AMap amap, WalkPath path,
			LatLonPoint start, LatLonPoint end) {
		super(context);
		this.mAMap = amap;
		this.walkPath = path;
		startPoint = AMapServicesUtil.convertToLatLng(start);
		endPoint = AMapServicesUtil.convertToLatLng(end);
	}
	/**
	 * 添加步行路线到地图中。
	 * @since V2.1.0
	 */
    public void addToMap() {

        initPolylineOptions();
        try {
            List<WalkStep> walkPaths = walkPath.getSteps();
            for (int i = 0; i < walkPaths.size(); i++) {
                WalkStep walkStep = walkPaths.get(i);
                LatLng latLng = AMapServicesUtil.convertToLatLng(walkStep
                        .getPolyline().get(0));
                
				addWalkStationMarkers(walkStep, latLng);
                addWalkPolyLines(walkStep);
               
            }
            addStartAndEndMarker();

            showPolyline();
        } catch (Throwable e) {
            e.printStackTrace();
        }
    }
	
	/**
	 * 检查这一步的最后一点和下一步的起始点之间是否存在空隙
	 */
	private void checkDistanceToNextStep(WalkStep walkStep,
			WalkStep walkStep1) {
		LatLonPoint lastPoint = getLastWalkPoint(walkStep);
		LatLonPoint nextFirstPoint = getFirstWalkPoint(walkStep1);
		if (!(lastPoint.equals(nextFirstPoint))) {
			addWalkPolyLine(lastPoint, nextFirstPoint);
		}
	}

	/**
	 * @param walkStep
	 * @return
	 */
	private LatLonPoint getLastWalkPoint(WalkStep walkStep) {
		return walkStep.getPolyline().get(walkStep.getPolyline().size() - 1);
	}

	/**
	 * @param walkStep
	 * @return
	 */
	private LatLonPoint getFirstWalkPoint(WalkStep walkStep) {
		return walkStep.getPolyline().get(0);
	}


    private void addWalkPolyLine(LatLonPoint pointFrom, LatLonPoint pointTo) {
        addWalkPolyLine(AMapServicesUtil.convertToLatLng(pointFrom), AMapServicesUtil.convertToLatLng(pointTo));
    }

    private void addWalkPolyLine(LatLng latLngFrom, LatLng latLngTo) {
        mPolylineOptions.add(latLngFrom, latLngTo);
    }

    /**
     * @param walkStep
     */
    private void addWalkPolyLines(WalkStep walkStep) {
        mPolylineOptions.addAll(AMapServicesUtil.convertArrList(walkStep.getPolyline()));
    }

    /**
     * @param walkStep
     * @param position
     */
    private void addWalkStationMarkers(WalkStep walkStep, LatLng position) {
        addStationMarker(new MarkerOptions()
                .position(position)
                .title("\u65B9\u5411:" + walkStep.getAction()
                        + "\n\u9053\u8DEF:" + walkStep.getRoad())
                .snippet(walkStep.getInstruction()).visible(nodeIconVisible)
                .anchor(0.5f, 0.5f).icon(walkStationDescriptor));
    }

    /**
     * 初始化线段属性
     */
    private void initPolylineOptions() {

        if(walkStationDescriptor == null) {
            walkStationDescriptor = getWalkBitmapDescriptor();
        }

        mPolylineOptions = null;

        mPolylineOptions = new PolylineOptions();
        mPolylineOptions.color(getWalkColor()).width(getRouteWidth());
    }


    private void showPolyline() {
        addPolyLine(mPolylineOptions);
    }
}
```

通过这三个类，完成步行路线绘制。
主要是地图的View，通过参数，传递给WalkRouteOverLay中，实现路线绘制。

### 9.2 骑行路线规划

骑行其实和步行差不多，只是路线限制时图层不同而已，其他的都类似，写起来也是比较简单的。

在startRouteSearch方法中：
```Java
 //出行方式判断
switch (TRAVEL_MODE) {
    case 0://步行
        //构建步行路线搜索对象
        RouteSearch.WalkRouteQuery query = new RouteSearch.WalkRouteQuery(fromAndTo, RouteSearch.WalkDefault);
        // 异步路径规划步行模式查询
        routeSearch.calculateWalkRouteAsyn(query);
        break;
    case 1://骑行
        //构建骑行路线搜索对象
        RouteSearch.RideRouteQuery rideQuery = new RouteSearch.RideRouteQuery(fromAndTo, RouteSearch.WalkDefault);
        //骑行规划路径计算
        routeSearch.calculateRideRouteAsyn(rideQuery);
        break;
```
这里通过routeSearch来计算骑行路线。

然后再MapUtil工具中，新增一个方法：
```Java
/**
    * 把集合体的LatLonPoint转化为集合体的LatLng
    */
public static ArrayList<LatLng> convertArrList(List<LatLonPoint> shapes) {
    ArrayList<LatLng> lineShapes = new ArrayList<LatLng>();
    for (LatLonPoint point : shapes) {
        LatLng latLngTemp = convertToLatLng(point);
        lineShapes.add(latLngTemp);
    }
    return lineShapes;
}
```

然后新增一个RideRouteOverlay，用于在地图上绘制骑行的图层：
```Java
import android.content.Context;

import com.amap.api.maps.AMap;
import com.amap.api.maps.model.BitmapDescriptor;
import com.amap.api.maps.model.BitmapDescriptorFactory;
import com.amap.api.maps.model.LatLng;
import com.amap.api.maps.model.MarkerOptions;
import com.amap.api.maps.model.PolylineOptions;
import com.amap.api.services.core.LatLonPoint;
import com.amap.api.services.route.RidePath;
import com.amap.api.services.route.RideStep;
import com.llw.mapdemo.R;
import com.llw.mapdemo.util.MapUtil;

import java.util.List;

/**
 * 骑行路线图层类。在高德地图API里，如果要显示步行路线规划，可以用此类来创建骑行路线图层。如不满足需求，也可以自己创建自定义的骑行路线图层。
 * @since V3.5.0
 */
public class RideRouteOverlay extends RouteOverlay {

	private PolylineOptions mPolylineOptions;
	
	private BitmapDescriptor walkStationDescriptor= null;

	private RidePath ridePath;
	/**
	 * 通过此构造函数创建骑行路线图层。
	 * @param context 当前activity。
	 * @param amap 地图对象。
	 * @param path 骑行路线规划的一个方案。详见搜索服务模块的路径查询包（com.amap.api.services.route）中的类 <strong><a href="../../../../../../Search/com/amap/api/services/route/WalkStep.html" title="com.amap.api.services.route中的类">WalkStep</a></strong>。
	 * @param start 起点。详见搜索服务模块的核心基础包（com.amap.api.services.core）中的类<strong><a href="../../../../../../Search/com/amap/api/services/core/LatLonPoint.html" title="com.amap.api.services.core中的类">LatLonPoint</a></strong>。
	 * @param end 终点。详见搜索服务模块的核心基础包（com.amap.api.services.core）中的类<strong><a href="../../../../../../Search/com/amap/api/services/core/LatLonPoint.html" title="com.amap.api.services.core中的类">LatLonPoint</a></strong>。
	 * @since V3.5.0
	 */
	public RideRouteOverlay(Context context, AMap amap, RidePath path,
                            LatLonPoint start, LatLonPoint end) {
		super(context);
		this.mAMap = amap;
		this.ridePath = path;
		startPoint = MapUtil.convertToLatLng(start);
		endPoint = MapUtil.convertToLatLng(end);
	}
	/**
	 * 添加骑行路线到地图中。
	 * @since V3.5.0
	 */
	public void addToMap() {
		
		initPolylineOptions();
		try {
			List<RideStep> ridePaths = ridePath.getSteps();
			for (int i = 0; i < ridePaths.size(); i++) {
				RideStep rideStep = ridePaths.get(i);
				LatLng latLng = MapUtil.convertToLatLng(rideStep
						.getPolyline().get(0));
				
				addRideStationMarkers(rideStep, latLng);
				addRidePolyLines(rideStep);
			}
			addStartAndEndMarker();
			
			showPolyline();
		} catch (Throwable e) {
			e.printStackTrace();
		}
	}


	/**
	 * @param rideStep
	 */
	private void addRidePolyLines(RideStep rideStep) {
		mPolylineOptions.addAll(MapUtil.convertArrList(rideStep.getPolyline()));
	}
	/**
	 * @param rideStep
	 * @param position
	 */
	private void addRideStationMarkers(RideStep rideStep, LatLng position) {
		addStationMarker(new MarkerOptions()
				.position(position)
				.title("\u65B9\u5411:" + rideStep.getAction()
						+ "\n\u9053\u8DEF:" + rideStep.getRoad())
				.snippet(rideStep.getInstruction()).visible(nodeIconVisible)
				.anchor(0.5f, 0.5f).icon(walkStationDescriptor));
	}
	
	 /**
     * 初始化线段属性
     */
    private void initPolylineOptions() {
    	
    	if(walkStationDescriptor == null) {
    		walkStationDescriptor = BitmapDescriptorFactory.fromResource(R.drawable.amap_ride);
    	}
        mPolylineOptions = null;
        mPolylineOptions = new PolylineOptions();
        mPolylineOptions.color(getDriveColor()).width(getRouteWidth());
    }
	 private void showPolyline() {
	        addPolyLine(mPolylineOptions);
	    }
}
```

然后在骑行路径规划的接口回调中使用这个图层绘制：
```Java
/**
    * 骑行规划路径结果
    *
    * @param rideRouteResult 结果
    * @param code            结果码
    */
@Override
public void onRideRouteSearched(final RideRouteResult rideRouteResult, int code) {
    aMap.clear();// 清理地图上的所有覆盖物
    if (code == AMapException.CODE_AMAP_SUCCESS) {
        if (rideRouteResult != null && rideRouteResult.getPaths() != null) {
            if (rideRouteResult.getPaths().size() > 0) {
                final RidePath ridePath = rideRouteResult.getPaths()
                        .get(0);
                if(ridePath == null) {
                    return;
                }
                RideRouteOverlay rideRouteOverlay = new RideRouteOverlay(
                        this, aMap, ridePath,
                        rideRouteResult.getStartPos(),
                        rideRouteResult.getTargetPos());
                rideRouteOverlay.removeFromMap();
                rideRouteOverlay.addToMap();
                rideRouteOverlay.zoomToSpan();

                int dis = (int) ridePath.getDistance();
                int dur = (int) ridePath.getDuration();
                String des = MapUtil.getFriendlyTime(dur)+"("+MapUtil.getFriendlyLength(dis)+")";
                Log.d(TAG, des);

            } else if (rideRouteResult.getPaths() == null) {
                showMsg("对不起，没有搜索到相关数据！");
            }
        } else {
            showMsg("对不起，没有搜索到相关数据！");
        }
    } else {
        showMsg("错误码；" + code);
    }
}
```

### 9.3 驾车路线规划

先增加计算驾车接口调用：
```Java
    case 2://驾车
        //构建驾车路线搜索对象  剩余三个参数分别是：途经点、避让区域、避让道路
        RouteSearch.DriveRouteQuery driveQuery = new RouteSearch.DriveRouteQuery(fromAndTo, RouteSearch.WalkDefault, null, null, "");
        //驾车规划路径计算
        routeSearch.calculateDriveRouteAsyn(driveQuery);
        break;
```

然后增加图层工具：
```Java
import android.content.Context;
import android.graphics.Color;

import com.amap.api.maps.AMap;
import com.amap.api.maps.model.BitmapDescriptor;
import com.amap.api.maps.model.BitmapDescriptorFactory;
import com.amap.api.maps.model.LatLng;
import com.amap.api.maps.model.LatLngBounds;
import com.amap.api.maps.model.Marker;
import com.amap.api.maps.model.MarkerOptions;
import com.amap.api.maps.model.PolylineOptions;
import com.amap.api.services.core.LatLonPoint;
import com.amap.api.services.route.DrivePath;
import com.amap.api.services.route.DriveStep;
import com.amap.api.services.route.TMC;
import com.llw.mapdemo.R;
import com.llw.mapdemo.util.MapUtil;

import java.util.ArrayList;
import java.util.List;


/**
 * 驾车路线图层类
 */
public class DrivingRouteOverlay extends RouteOverlay{

	private DrivePath drivePath;
    private List<LatLonPoint> throughPointList;
    private List<Marker> throughPointMarkerList = new ArrayList<Marker>();
    private boolean throughPointMarkerVisible = true;
    private List<TMC> tmcs;
    private PolylineOptions mPolylineOptions;
    private PolylineOptions mPolylineOptionscolor;
    private Context mContext;
    private boolean isColorfulline = true;
    private float mWidth = 25;
    private List<LatLng> mLatLngsOfPath;

	public void setIsColorfulline(boolean iscolorfulline) {
		this.isColorfulline = iscolorfulline;
	}

	/**
     * 根据给定的参数，构造一个导航路线图层类对象。
     *
     * @param amap      地图对象。
     * @param path 导航路线规划方案。
     * @param context   当前的activity对象。
     */
    public DrivingRouteOverlay(Context context, AMap amap, DrivePath path,
                               LatLonPoint start, LatLonPoint end, List<LatLonPoint> throughPointList) {
    	super(context);
    	mContext = context; 
        mAMap = amap; 
        this.drivePath = path;
        startPoint = MapUtil.convertToLatLng(start);
        endPoint = MapUtil.convertToLatLng(end);
        this.throughPointList = throughPointList;
    }

    @Override
    public float getRouteWidth() {
        return mWidth;
    }

    /**
     * 设置路线宽度
     *
     * @param mWidth 路线宽度，取值范围：大于0
     */
    public void setRouteWidth(float mWidth) {
        this.mWidth = mWidth;
    }

    /**
     * 添加驾车路线添加到地图上显示。
     */
	public void addToMap() {
		initPolylineOptions();
        try {
            if (mAMap == null) {
                return;
            }

            if (mWidth == 0 || drivePath == null) {
                return;
            }
            mLatLngsOfPath = new ArrayList<LatLng>();
            tmcs = new ArrayList<TMC>();
            List<DriveStep> drivePaths = drivePath.getSteps();
            for (DriveStep step : drivePaths) {
                List<LatLonPoint> latlonPoints = step.getPolyline();
                List<TMC> tmclist = step.getTMCs();
                tmcs.addAll(tmclist);
                addDrivingStationMarkers(step, convertToLatLng(latlonPoints.get(0)));
                for (LatLonPoint latlonpoint : latlonPoints) {
                	mPolylineOptions.add(convertToLatLng(latlonpoint));
                	mLatLngsOfPath.add(convertToLatLng(latlonpoint));
				}
            }
            if (startMarker != null) {
                startMarker.remove();
                startMarker = null;
            }
            if (endMarker != null) {
                endMarker.remove();
                endMarker = null;
            }
            addStartAndEndMarker();
            addThroughPointMarker();
            if (isColorfulline && tmcs.size()>0 ) {
            	colorWayUpdate(tmcs);
            	showcolorPolyline();
			}else {
				showPolyline();
			}            
            
        } catch (Throwable e) {
        	e.printStackTrace();
        }
    }

	/**
     * 初始化线段属性
     */
    private void initPolylineOptions() {

        mPolylineOptions = null;

        mPolylineOptions = new PolylineOptions();
        mPolylineOptions.color(getDriveColor()).width(getRouteWidth());
    }

    private void showPolyline() {
        addPolyLine(mPolylineOptions);
    }
    
    private void showcolorPolyline() {
    	addPolyLine(mPolylineOptionscolor);
		
	}

    /**
     * 根据不同的路段拥堵情况展示不同的颜色
     *
     * @param tmcSection
     */
    private void colorWayUpdate(List<TMC> tmcSection) {
        if (mAMap == null) {
            return;
        }
        if (tmcSection == null || tmcSection.size() <= 0) {
            return;
        }
        TMC segmentTrafficStatus;
        mPolylineOptionscolor = null;
        mPolylineOptionscolor = new PolylineOptions();
        mPolylineOptionscolor.width(getRouteWidth());
        List<Integer> colorList = new ArrayList<Integer>();
        mPolylineOptionscolor.add(MapUtil.convertToLatLng(tmcSection.get(0).getPolyline().get(0)));
        colorList.add(getDriveColor());
        for (int i = 0; i < tmcSection.size(); i++) {
        	segmentTrafficStatus = tmcSection.get(i);
        	int color = getcolor(segmentTrafficStatus.getStatus());
        	List<LatLonPoint> mployline = segmentTrafficStatus.getPolyline();
			for (int j = 1; j < mployline.size(); j++) {
				mPolylineOptionscolor.add(MapUtil.convertToLatLng(mployline.get(j)));
				colorList.add(color);
			}
		}
        colorList.add(getDriveColor());
        mPolylineOptionscolor.colorValues(colorList);
    }
    
    private int getcolor(String status) {

    	if (status.equals("畅通")) {
    		return Color.GREEN;
		} else if (status.equals("缓行")) {
			 return Color.YELLOW;
		} else if (status.equals("拥堵")) {
			return Color.RED;
		} else if (status.equals("严重拥堵")) {
			return Color.parseColor("#990033");
		} else {
			return Color.parseColor("#537edc");
		}	
	}

	public LatLng convertToLatLng(LatLonPoint point) {
        return new LatLng(point.getLatitude(),point.getLongitude());
  }
    
    /**
     * @param driveStep
     * @param latLng
     */
    private void addDrivingStationMarkers(DriveStep driveStep, LatLng latLng) {
        addStationMarker(new MarkerOptions()
                .position(latLng)
                .title("\u65B9\u5411:" + driveStep.getAction()
                        + "\n\u9053\u8DEF:" + driveStep.getRoad())
                .snippet(driveStep.getInstruction()).visible(nodeIconVisible)
                .anchor(0.5f, 0.5f).icon(getDriveBitmapDescriptor()));
    }

    @Override
    protected LatLngBounds getLatLngBounds() {
        LatLngBounds.Builder b = LatLngBounds.builder();
        b.include(new LatLng(startPoint.latitude, startPoint.longitude));
        b.include(new LatLng(endPoint.latitude, endPoint.longitude));
        if (this.throughPointList != null && this.throughPointList.size() > 0) {
            for (int i = 0; i < this.throughPointList.size(); i++) {
                b.include(new LatLng(
                        this.throughPointList.get(i).getLatitude(),
                        this.throughPointList.get(i).getLongitude()));
            }
        }
        return b.build();
    }

    public void setThroughPointIconVisibility(boolean visible) {
        try {
            throughPointMarkerVisible = visible;
            if (this.throughPointMarkerList != null
                    && this.throughPointMarkerList.size() > 0) {
                for (int i = 0; i < this.throughPointMarkerList.size(); i++) {
                    this.throughPointMarkerList.get(i).setVisible(visible);
                }
            }
        } catch (Throwable e) {
            e.printStackTrace();
        }
    }
    
    private void addThroughPointMarker() {
        if (this.throughPointList != null && this.throughPointList.size() > 0) {
            LatLonPoint latLonPoint = null;
            for (int i = 0; i < this.throughPointList.size(); i++) {
                latLonPoint = this.throughPointList.get(i);
                if (latLonPoint != null) {
                    throughPointMarkerList.add(mAMap
                            .addMarker((new MarkerOptions())
                                    .position(
                                            new LatLng(latLonPoint
                                                    .getLatitude(), latLonPoint
                                                    .getLongitude()))
                                    .visible(throughPointMarkerVisible)
                                    .icon(getThroughPointBitDes())
                                    .title("\u9014\u7ECF\u70B9")));
                }
            }
        }
    }
    
    private BitmapDescriptor getThroughPointBitDes() {
    	return BitmapDescriptorFactory.fromResource(R.drawable.amap_through);
       
    }

    /**
     * 获取两点间距离
     *
     * @param start
     * @param end
     * @return
     */
    public static int calculateDistance(LatLng start, LatLng end) {
        double x1 = start.longitude;
        double y1 = start.latitude;
        double x2 = end.longitude;
        double y2 = end.latitude;
        return calculateDistance(x1, y1, x2, y2);
    }

    public static int calculateDistance(double x1, double y1, double x2, double y2) {
        final double NF_pi = 0.01745329251994329; // 弧度 PI/180
        x1 *= NF_pi;
        y1 *= NF_pi;
        x2 *= NF_pi;
        y2 *= NF_pi;
        double sinx1 = Math.sin(x1);
        double siny1 = Math.sin(y1);
        double cosx1 = Math.cos(x1);
        double cosy1 = Math.cos(y1);
        double sinx2 = Math.sin(x2);
        double siny2 = Math.sin(y2);
        double cosx2 = Math.cos(x2);
        double cosy2 = Math.cos(y2);
        double[] v1 = new double[3];
        v1[0] = cosy1 * cosx1 - cosy2 * cosx2;
        v1[1] = cosy1 * sinx1 - cosy2 * sinx2;
        v1[2] = siny1 - siny2;
        double dist = Math.sqrt(v1[0] * v1[0] + v1[1] * v1[1] + v1[2] * v1[2]);

        return (int) (Math.asin(dist / 2) * 12742001.5798544);
    }


    //获取指定两点之间固定距离点
    public static LatLng getPointForDis(LatLng sPt, LatLng ePt, double dis) {
        double lSegLength = calculateDistance(sPt, ePt);
        double preResult = dis / lSegLength;
        return new LatLng((ePt.latitude - sPt.latitude) * preResult + sPt.latitude, (ePt.longitude - sPt.longitude) * preResult + sPt.longitude);
    }
    /**
     * 去掉DriveLineOverlay上的线段和标记。
     */
    @Override
    public void removeFromMap() {
        try {
            super.removeFromMap();
            if (this.throughPointMarkerList != null
                    && this.throughPointMarkerList.size() > 0) {
                for (int i = 0; i < this.throughPointMarkerList.size(); i++) {
                    this.throughPointMarkerList.get(i).remove();
                }
                this.throughPointMarkerList.clear();
            }
        } catch (Throwable e) {
            e.printStackTrace();
        }
    }
}
```

搜索结果中进行图层绘制：
```Java 
/**
    * 驾车规划路径结果
    *
    * @param driveRouteResult 结果
    * @param code            结果码
    */
@Override
public void onDriveRouteSearched(DriveRouteResult driveRouteResult, int code) {
    aMap.clear();// 清理地图上的所有覆盖物
    if (code == AMapException.CODE_AMAP_SUCCESS) {
        if (driveRouteResult != null && driveRouteResult.getPaths() != null) {
            if (driveRouteResult.getPaths().size() > 0) {
                final DrivePath drivePath = driveRouteResult.getPaths()
                        .get(0);
                if(drivePath == null) {
                    return;
                }
                DrivingRouteOverlay drivingRouteOverlay = new DrivingRouteOverlay(
                        this, aMap, drivePath,
                        driveRouteResult.getStartPos(),
                        driveRouteResult.getTargetPos(), null);
                drivingRouteOverlay.removeFromMap();
                drivingRouteOverlay.addToMap();
                drivingRouteOverlay.zoomToSpan();

                int dis = (int) drivePath.getDistance();
                int dur = (int) drivePath.getDuration();
                String des = MapUtil.getFriendlyTime(dur)+"("+MapUtil.getFriendlyLength(dis)+")";
                Log.d(TAG, des);
            } else if (driveRouteResult.getPaths() == null) {
                showMsg("对不起，没有搜索到相关数据！");
            }
        } else {
            showMsg("对不起，没有搜索到相关数据！");
        }
    } else {
        showMsg("错误码；" + code);
    }
}
```

### 9.4 公交路线规划

接口请求：
```Java
    case 3://公交
        //构建驾车路线搜索对象 第三个参数表示公交查询城市区号，第四个参数表示是否计算夜班车，0表示不计算,1表示计算
        RouteSearch.BusRouteQuery busQuery = new RouteSearch.BusRouteQuery(fromAndTo, RouteSearch.BusLeaseWalk, "0755",0);
        //公交规划路径计算
        routeSearch.calculateBusRouteAsyn(busQuery);
        break;
```

图层工具添加：
```Java
import android.content.Context;

import com.amap.api.maps.AMap;
import com.amap.api.maps.model.LatLng;
import com.amap.api.maps.model.MarkerOptions;
import com.amap.api.maps.model.PolylineOptions;
import com.amap.api.services.busline.BusStationItem;
import com.amap.api.services.core.LatLonPoint;
import com.amap.api.services.route.BusPath;
import com.amap.api.services.route.BusStep;
import com.amap.api.services.route.Doorway;
import com.amap.api.services.route.RailwayStationItem;
import com.amap.api.services.route.RouteBusLineItem;
import com.amap.api.services.route.RouteBusWalkItem;
import com.amap.api.services.route.RouteRailwayItem;
import com.amap.api.services.route.TaxiItem;
import com.amap.api.services.route.WalkStep;
import com.llw.mapdemo.util.MapUtil;

import java.util.ArrayList;
import java.util.List;

/**
 * 公交路线图层类。在高德地图API里，如果需要显示公交路线，可以用此类来创建公交路线图层。如不满足需求，也可以自己创建自定义的公交路线图层。
 * @since V2.1.0
 */
public class BusRouteOverlay extends RouteOverlay {

	private BusPath busPath;
	private LatLng latLng;

	/**
	 * 通过此构造函数创建公交路线图层。
	 * @param context 当前activity。
	 * @param amap 地图对象。
	 * @param path 公交路径规划的一个路段。详见搜索服务模块的路径查询包（com.amap.api.services.route）中的类<strong> <a href="../../../../../../Search/com/amap/api/services/route/BusPath.html" title="com.amap.api.services.route中的类">BusPath</a></strong>。
	 * @param start 起点坐标。详见搜索服务模块的核心基础包（com.amap.api.services.core）中的类 <strong><a href="../../../../../../Search/com/amap/api/services/core/LatLonPoint.html" title="com.amap.api.services.core中的类">LatLonPoint</a></strong>。
	 * @param end 终点坐标。详见搜索服务模块的核心基础包（com.amap.api.services.core）中的类 <strong><a href="../../../../../../Search/com/amap/api/services/core/LatLonPoint.html" title="com.amap.api.services.core中的类">LatLonPoint</a></strong>。
	 * @since V2.1.0
	 */
	public BusRouteOverlay(Context context, AMap amap, BusPath path,
                           LatLonPoint start, LatLonPoint end) {
		super(context);
		this.busPath = path;
		startPoint = MapUtil.convertToLatLng(start);
		endPoint = MapUtil.convertToLatLng(end);
		mAMap = amap;
	}

	/**
	 * 添加公交路线到地图上。
	 * @since V2.1.0
	 */

	public void addToMap() {
		/**
		 * 绘制节点和线<br>
		 * 细节情况较多<br>
		 * 两个step之间，用step和step1区分<br>
		 * 1.一个step内可能有步行和公交，然后有可能他们之间连接有断开<br>
		 * 2.step的公交和step1的步行，有可能连接有断开<br>
		 * 3.step和step1之间是公交换乘，且没有步行，需要把step的终点和step1的起点连起来<br>
		 * 4.公交最后一站和终点间有步行，加入步行线路，还会有一些步行marker<br>
		 * 5.公交最后一站和终点间无步行，之间连起来<br>
		 */
		try {
			List<BusStep> busSteps = busPath.getSteps();
			for (int i = 0; i < busSteps.size(); i++) {
				BusStep busStep = busSteps.get(i);
				if (i < busSteps.size() - 1) {
					BusStep busStep1 = busSteps.get(i + 1);// 取得当前下一个BusStep对象
					// 假如步行和公交之间连接有断开，就把步行最后一个经纬度点和公交第一个经纬度点连接起来，避免断线问题
					if (busStep.getWalk() != null
							&& busStep.getBusLine() != null) {
						checkWalkToBusline(busStep);
					}

					// 假如公交和步行之间连接有断开，就把上一公交经纬度点和下一步行第一个经纬度点连接起来，避免断线问题
					if (busStep.getBusLine() != null
							&& busStep1.getWalk() != null 
							&& busStep1.getWalk().getSteps().size() > 0) {
						checkBusLineToNextWalk(busStep, busStep1);
					}
					// 假如两个公交换乘中间没有步行，就把上一公交经纬度点和下一步公交第一个经纬度点连接起来，避免断线问题
					if (busStep.getBusLine() != null
							&& busStep1.getWalk() == null
							&& busStep1.getBusLine() != null) {
						checkBusEndToNextBusStart(busStep, busStep1);
					}
					// 和上面的很类似
					if (busStep.getBusLine() != null
							&& busStep1.getWalk() == null
							&& busStep1.getBusLine() != null) {
						checkBusToNextBusNoWalk(busStep, busStep1);
					}
					if (busStep.getBusLine() != null
							&& busStep1.getRailway() != null ) {
						checkBusLineToNextRailway(busStep, busStep1);
					}
					if (busStep1.getWalk() != null &&
							busStep1.getWalk().getSteps().size() > 0 &&
							busStep.getRailway() != null) {
						checkRailwayToNextWalk(busStep, busStep1);
					}
					
					if ( busStep1.getRailway() != null &&
							busStep.getRailway() != null) {
						checkRailwayToNextRailway(busStep, busStep1);
					}
					
					if (busStep.getRailway() != null && 
						busStep1.getTaxi() != null ){
						checkRailwayToNextTaxi(busStep, busStep1);
					}
					

				}

				if (busStep.getWalk() != null
						&& busStep.getWalk().getSteps().size() > 0) {
					addWalkSteps(busStep);
				} else {
					if (busStep.getBusLine() == null && busStep.getRailway() == null && busStep.getTaxi() == null) {
						addWalkPolyline(latLng, endPoint);
					}
				}
				if (busStep.getBusLine() != null) {
					RouteBusLineItem routeBusLineItem = busStep.getBusLine();
					addBusLineSteps(routeBusLineItem);
					addBusStationMarkers(routeBusLineItem);
					if (i == busSteps.size() - 1) {
						addWalkPolyline(MapUtil.convertToLatLng(getLastBuslinePoint(busStep)), endPoint);
					}
				}
				if (busStep.getRailway() != null) {
					addRailwayStep(busStep.getRailway());
					addRailwayMarkers(busStep.getRailway());
					if (i == busSteps.size() - 1) {
						addWalkPolyline(MapUtil.convertToLatLng(busStep.getRailway().getArrivalstop().getLocation()), endPoint);
					}
				}
				if (busStep.getTaxi() != null) {
					addTaxiStep(busStep.getTaxi());
					addTaxiMarkers(busStep.getTaxi());
				}
			}
			addStartAndEndMarker();

		} catch (Throwable e) {
			e.printStackTrace();
		}
	}



	private void checkRailwayToNextTaxi(BusStep busStep, BusStep busStep1) {
		LatLonPoint railwayLastPoint = busStep.getRailway().getArrivalstop().getLocation();
		LatLonPoint taxiFirstPoint = busStep1.getTaxi().getOrigin();
		if (!railwayLastPoint.equals(taxiFirstPoint)) {
			addWalkPolyLineByLatLonPoints(railwayLastPoint, taxiFirstPoint);
		}
	}

	private void checkRailwayToNextRailway(BusStep busStep, BusStep busStep1) {
		LatLonPoint railwayLastPoint = busStep.getRailway().getArrivalstop().getLocation();
		LatLonPoint railwayFirstPoint = busStep1.getRailway().getDeparturestop().getLocation();
		if (!railwayLastPoint.equals(railwayFirstPoint)) {
			addWalkPolyLineByLatLonPoints(railwayLastPoint, railwayFirstPoint);
		}
		
	}

	private void checkBusLineToNextRailway(BusStep busStep, BusStep busStep1) {
		LatLonPoint busLastPoint = getLastBuslinePoint(busStep);
		LatLonPoint railwayFirstPoint = busStep1.getRailway().getDeparturestop().getLocation();
		if (!busLastPoint.equals(railwayFirstPoint)) {
			addWalkPolyLineByLatLonPoints(busLastPoint, railwayFirstPoint);
		}
		
	}

	private void checkRailwayToNextWalk(BusStep busStep, BusStep busStep1) {
		LatLonPoint railwayLastPoint = busStep.getRailway().getArrivalstop().getLocation();
		LatLonPoint walkFirstPoint = getFirstWalkPoint(busStep1);
		if (!railwayLastPoint.equals(walkFirstPoint)) {
			addWalkPolyLineByLatLonPoints(railwayLastPoint, walkFirstPoint);
		}
		
	}

	private void addRailwayStep(RouteRailwayItem railway) {
		List<LatLng> railwaylistpoint = new ArrayList<LatLng>();
		List<RailwayStationItem> railwayStationItems = new ArrayList<RailwayStationItem>();
		railwayStationItems.add(railway.getDeparturestop());
		railwayStationItems.addAll(railway.getViastops());
		railwayStationItems.add(railway.getArrivalstop());
		for (int i = 0; i < railwayStationItems.size(); i++) {
			railwaylistpoint.add(MapUtil.convertToLatLng(railwayStationItems.get(i).getLocation()));
		}
		addRailwayPolyline(railwaylistpoint);
	}
	
	private void addTaxiStep(TaxiItem taxi){
		addPolyLine(new PolylineOptions().width(getRouteWidth())
				.color(getBusColor())
				.add(MapUtil.convertToLatLng(taxi.getOrigin()))
				.add(MapUtil.convertToLatLng(taxi.getDestination())));
	}

	/**
	 * @param busStep
	 */
	private void addWalkSteps(BusStep busStep) {
		RouteBusWalkItem routeBusWalkItem = busStep.getWalk();
		List<WalkStep> walkSteps = routeBusWalkItem.getSteps();
		for (int j = 0; j < walkSteps.size(); j++) {
			WalkStep walkStep = walkSteps.get(j);
			if (j == 0) {
				LatLng latLng = MapUtil.convertToLatLng(walkStep
						.getPolyline().get(0));
				String road = walkStep.getRoad();// 道路名字
				String instruction = getWalkSnippet(walkSteps);// 步行导航信息
				addWalkStationMarkers(latLng, road, instruction);
			}

			List<LatLng> listWalkPolyline = MapUtil
					.convertArrList(walkStep.getPolyline());
			this.latLng = listWalkPolyline.get(listWalkPolyline.size() - 1);

			addWalkPolyline(listWalkPolyline);

			// 假如步行前一段的终点和下的起点有断开，断画直线连接起来，避免断线问题
			if (j < walkSteps.size() - 1) {
				LatLng lastLatLng = listWalkPolyline.get(listWalkPolyline
						.size() - 1);
				LatLng firstlatLatLng = MapUtil
						.convertToLatLng(walkSteps.get(j + 1).getPolyline()
								.get(0));
				if (!(lastLatLng.equals(firstlatLatLng))) {
					addWalkPolyline(lastLatLng, firstlatLatLng);
				}
			}

		}
	}

	/**
	 * 添加一系列的bus PolyLine
	 *
	 * @param routeBusLineItem
	 */
	private void addBusLineSteps(RouteBusLineItem routeBusLineItem) {
		addBusLineSteps(routeBusLineItem.getPolyline());
	}

	private void addBusLineSteps(List<LatLonPoint> listPoints) {
		if (listPoints.size() < 1) {
			return;
		}
		addPolyLine(new PolylineOptions().width(getRouteWidth())
				.color(getBusColor())
				.addAll(MapUtil.convertArrList(listPoints)));
	}

	/**
	 * @param latLng
	 *            marker
	 * @param title
	 * @param snippet
	 */
	private void addWalkStationMarkers(LatLng latLng, String title,
                                       String snippet) {
		addStationMarker(new MarkerOptions().position(latLng).title(title)
				.snippet(snippet).anchor(0.5f, 0.5f).visible(nodeIconVisible)
				.icon(getWalkBitmapDescriptor()));
	}

	/**
	 * @param routeBusLineItem
	 */
	private void addBusStationMarkers(RouteBusLineItem routeBusLineItem) {
		BusStationItem startBusStation = routeBusLineItem
				.getDepartureBusStation();
		LatLng position = MapUtil.convertToLatLng(startBusStation
				.getLatLonPoint());
		String title = routeBusLineItem.getBusLineName();
		String snippet = getBusSnippet(routeBusLineItem);

		addStationMarker(new MarkerOptions().position(position).title(title)
				.snippet(snippet).anchor(0.5f, 0.5f).visible(nodeIconVisible)
				.icon(getBusBitmapDescriptor()));
	}
	
	private void addTaxiMarkers(TaxiItem taxiItem) {
		
		LatLng position = MapUtil.convertToLatLng(taxiItem
				.getOrigin());
		String title = taxiItem.getmSname()+"打车";
		String snippet = "到终点";

		addStationMarker(new MarkerOptions().position(position).title(title)
				.snippet(snippet).anchor(0.5f, 0.5f).visible(nodeIconVisible)
				.icon(getDriveBitmapDescriptor()));
	}

	private void addRailwayMarkers(RouteRailwayItem railway) {
		LatLng Departureposition = MapUtil.convertToLatLng(railway
				.getDeparturestop().getLocation());
		String Departuretitle = railway.getDeparturestop().getName()+"上车";
		String Departuresnippet = railway.getName();

		addStationMarker(new MarkerOptions().position(Departureposition).title(Departuretitle)
				.snippet(Departuresnippet).anchor(0.5f, 0.5f).visible(nodeIconVisible)
				.icon(getBusBitmapDescriptor()));
		
		
		LatLng Arrivalposition = MapUtil.convertToLatLng(railway
				.getArrivalstop().getLocation());
		String Arrivaltitle = railway.getArrivalstop().getName()+"下车";
		String Arrivalsnippet = railway.getName();

		addStationMarker(new MarkerOptions().position(Arrivalposition).title(Arrivaltitle)
				.snippet(Arrivalsnippet).anchor(0.5f, 0.5f).visible(nodeIconVisible)
				.icon(getBusBitmapDescriptor()));
	}
	/**
	 * 如果换乘没有步行 检查bus最后一点和下一个step的bus起点是否一致
	 *
	 * @param busStep
	 * @param busStep1
	 */
	private void checkBusToNextBusNoWalk(BusStep busStep, BusStep busStep1) {
		LatLng endbusLatLng = MapUtil
				.convertToLatLng(getLastBuslinePoint(busStep));
		LatLng startbusLatLng = MapUtil
				.convertToLatLng(getFirstBuslinePoint(busStep1));
		if (startbusLatLng.latitude - endbusLatLng.latitude > 0.0001
				|| startbusLatLng.longitude - endbusLatLng.longitude > 0.0001) {
			drawLineArrow(endbusLatLng, startbusLatLng);// 断线用带箭头的直线连?
		}
	}

	/**
	 *
	 * checkBusToNextBusNoWalk 和这个类似
	 *
	 * @param busStep
	 * @param busStep1
	 */
	private void checkBusEndToNextBusStart(BusStep busStep, BusStep busStep1) {
		LatLonPoint busLastPoint = getLastBuslinePoint(busStep);
		LatLng endbusLatLng = MapUtil.convertToLatLng(busLastPoint);
		LatLonPoint busFirstPoint = getFirstBuslinePoint(busStep1);
		LatLng startbusLatLng = MapUtil.convertToLatLng(busFirstPoint);
		if (!endbusLatLng.equals(startbusLatLng)) {
			drawLineArrow(endbusLatLng, startbusLatLng);//
		}
	}

	/**
	 * 检查bus最后一步和下一各step的步行起点是否一致
	 *
	 * @param busStep
	 * @param busStep1
	 */
	private void checkBusLineToNextWalk(BusStep busStep, BusStep busStep1) {
		LatLonPoint busLastPoint = getLastBuslinePoint(busStep);
		LatLonPoint walkFirstPoint = getFirstWalkPoint(busStep1);
		if (!busLastPoint.equals(walkFirstPoint)) {
			addWalkPolyLineByLatLonPoints(busLastPoint, walkFirstPoint);
		}
	}

	/**
	 * 检查 步行最后一点 和 bus的起点 是否一致
	 *
	 * @param busStep
	 */
	private void checkWalkToBusline(BusStep busStep) {
		LatLonPoint walkLastPoint = getLastWalkPoint(busStep);
		LatLonPoint buslineFirstPoint = getFirstBuslinePoint(busStep);

		if (!walkLastPoint.equals(buslineFirstPoint)) {
			addWalkPolyLineByLatLonPoints(walkLastPoint, buslineFirstPoint);
		}
	}

	/**
	 * @param busStep1
	 * @return
	 */
	private LatLonPoint getFirstWalkPoint(BusStep busStep1) {
		return busStep1.getWalk().getSteps().get(0).getPolyline().get(0);
	}

	/**
	 *
	 */
	private void addWalkPolyLineByLatLonPoints(LatLonPoint pointFrom,
                                               LatLonPoint pointTo) {
		LatLng latLngFrom = MapUtil.convertToLatLng(pointFrom);
		LatLng latLngTo = MapUtil.convertToLatLng(pointTo);

		addWalkPolyline(latLngFrom, latLngTo);
	}

	/**
	 * @param latLngFrom
	 * @param latLngTo
	 * @return
	 */
	private void addWalkPolyline(LatLng latLngFrom, LatLng latLngTo) {
		addPolyLine(new PolylineOptions().add(latLngFrom, latLngTo)
				.width(getRouteWidth()).color(getWalkColor()).setDottedLine(true));
	}

	/**
	 * @param listWalkPolyline
	 */
	private void addWalkPolyline(List<LatLng> listWalkPolyline) {

		addPolyLine(new PolylineOptions().addAll(listWalkPolyline)
				.color(getWalkColor()).width(getRouteWidth()).setDottedLine(true));
	}

	private void addRailwayPolyline(List<LatLng> listPolyline) {

		addPolyLine(new PolylineOptions().addAll(listPolyline)
				.color(getDriveColor()).width(getRouteWidth()));
	}
	
	
	private String getWalkSnippet(List<WalkStep> walkSteps) {
		float disNum = 0;
		for (WalkStep step : walkSteps) {
			disNum += step.getDistance();
		}
		return "\u6B65\u884C" + disNum + "\u7C73";
	}

	public void drawLineArrow(LatLng latLngFrom, LatLng latLngTo) {

		addPolyLine(new PolylineOptions().add(latLngFrom, latLngTo).width(3)
				.color(getBusColor()).width(getRouteWidth()));// 绘制直线
	}

	private String getBusSnippet(RouteBusLineItem routeBusLineItem) {
		return "("
				+ routeBusLineItem.getDepartureBusStation().getBusStationName()
				+ "-->"
				+ routeBusLineItem.getArrivalBusStation().getBusStationName()
				+ ") \u7ECF\u8FC7" + (routeBusLineItem.getPassStationNum() + 1)
				+ "\u7AD9";
	}

	/**
	 * @param busStep
	 * @return
	 */
	private LatLonPoint getLastWalkPoint(BusStep busStep) {

		List<WalkStep> walkSteps = busStep.getWalk().getSteps();
		WalkStep walkStep = walkSteps.get(walkSteps.size() - 1);
		List<LatLonPoint> lonPoints = walkStep.getPolyline();
		return lonPoints.get(lonPoints.size() - 1);
	}

	private LatLonPoint getExitPoint(BusStep busStep) {
		Doorway doorway = busStep.getExit();
		if (doorway == null) {
			return null;
		}
		return doorway.getLatLonPoint();
	}

	private LatLonPoint getLastBuslinePoint(BusStep busStep) {
		List<LatLonPoint> lonPoints = busStep.getBusLine().getPolyline();

		return lonPoints.get(lonPoints.size() - 1);
	}

	private LatLonPoint getEntrancePoint(BusStep busStep) {
		Doorway doorway = busStep.getEntrance();
		if (doorway == null) {
			return null;
		}
		return doorway.getLatLonPoint();
	}

	private LatLonPoint getFirstBuslinePoint(BusStep busStep) {
		return busStep.getBusLine().getPolyline().get(0);
	}
}
```

然后是结果使用图层绘制：
```Java
    /**
    * 公交规划路径结果
    *
    * @param busRouteResult 结果
    * @param code           结果码
    */
@Override
public void onBusRouteSearched(BusRouteResult busRouteResult, int code) {
    aMap.clear();// 清理地图上的所有覆盖物
    if (code == AMapException.CODE_AMAP_SUCCESS) {
        if (busRouteResult != null && busRouteResult.getPaths() != null) {
            if (busRouteResult.getPaths().size() > 0) {
                final BusPath busPath = busRouteResult.getPaths().get(0);
                if (busPath == null) {
                    return;
                }
                BusRouteOverlay busRouteOverlay = new BusRouteOverlay(
                        this, aMap, busPath,
                        busRouteResult.getStartPos(),
                        busRouteResult.getTargetPos());
                busRouteOverlay.removeFromMap();
                busRouteOverlay.addToMap();
                busRouteOverlay.zoomToSpan();

                int dis = (int) busPath.getDistance();
                int dur = (int) busPath.getDuration();
                String des = MapUtil.getFriendlyTime(dur) + "(" + MapUtil.getFriendlyLength(dis) + ")";
                Log.d(TAG, des);
            } else if (busRouteResult.getPaths() == null) {
                showMsg("对不起，没有搜索到相关数据！");
            }
        } else {
            showMsg("对不起，没有搜索到相关数据！");
        }
    } else {
        showMsg("错误码；" + code);
    }
}
```

## 10 总结

* 首先是配置依赖，可以远程或者本地引用，前提是配置key，注意配置SHA1安全码。
* 然后是地图使用，主要是在布局中声明Map，这个Map本质上是一个ViewGroup。
* 然后是设置地图的一些配置，比如marker，点击事件，拖拽事件等。
* 然后是定位的使用，逆地址编码和地址编码，注意动态申请权限。
* 然后是搜索结果的使用，这个调接口，配合地址编码，也是很容易。
* 然后是路径规划了，这个主要是调接口， RouteSearch很关键，然后是各种图层绘制，总体上还是比较简单的。