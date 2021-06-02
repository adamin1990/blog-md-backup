# flutter 接入高德地图,打包安卓apk，自定义签名

#### 引入依赖

在pubspec.yaml引入 `amap_flutter_location: ^2.0.0` 依赖，然后执行 `pub get`

#### Android

###### 生成签名文件

在项目`android/app` 目录下打开终端输入

~~~
keytool -genkey -v -keystore xipingflutter.jks -keyalg RSA -keysize 2048 -validity 10000 -alias xiping
~~~

然后根据依次设置密钥，姓名，单位，组织，地区，别名密钥等：

~~~
输入密钥库口令:
再次输入新口令:
您的名字与姓氏是什么?
  [Unknown]:  adam
您的组织单位名称是什么?
  [Unknown]:  xiping
您的组织名称是什么?
  [Unknown]:  xiping
您所在的城市或区域名称是什么?
  [Unknown]:  cn
您所在的省/市/自治区名称是什么?
  [Unknown]:  cn
该单位的双字母国家/地区代码是什么?
  [Unknown]:  CN
CN=adam, OU=xiping, O=xiping, L=cn, ST=cn, C=CN是否正确?
  [否]:  是

正在为以下对象生成 2,048 位RSA密钥对和自签名证书 (SHA256withRSA) (有效期为 10,000 天):
         CN=adam, OU=xiping, O=xiping, L=cn, ST=cn, C=CN
输入 <xiping> 的密钥口令
        (如果和密钥库口令相同, 按回车):
[正在存储xipingflutter.jks]
~~~

之后会在`android/app`目录下生成xipingflutter.jks签名文件，此文件用户安卓打包的签名文件

###### Android Studio配置密钥

在 `android\app`目录下创建  `keystore.properties`文件，写入如下内容配置,保存

 ```
 storePassword=xiping2021
 keyPassword=xiping2021
 keyAlias=xiping
 storeFile=xipingflutter.jks 
 ```

修改 `andorid\app\build.gradle` 文件，增加如下配置

~~~
// 定义签名的信息
def keystoreProperties = new Properties()
def keystoreFile = rootProject.file("keysore.properties")
if (keystoreFile.exists()) {
    keystoreFile.withReader('UTF-8') { reader ->
        keystoreProperties.load(reader)
    }
}
def storePassword = keystoreProperties.getProperty("storePassword","xiping2021")
def keyPassword = keystoreProperties.getProperty("keyPassword","xiping2021")
def keyAlias = keystoreProperties.getProperty("keyAlias","xiping")
def storeFile = keystoreProperties.getProperty("storeFile","xipingflutter.jks")

//签名信息end

    signingConfigs {
        release {
            keyAlias keyAlias
            keyPassword keyPassword
            storeFile file(storeFile)
            storePassword storePassword
        }
    }
    
    buildTypes {
        release {
            signingConfig signingConfigs.release
        }
        debug {
            signingConfig signingConfigs.release
        }
    }
~~~

经过以上配置，我们调试安卓app就使用自定义的keystore调试了，发布上线可以用打包命令打包`flutter build apk --no-shrink`，自动使用自定义签名

###### 添加安卓应用

在高德lbs后台应用列表添加安卓应用，包名写我们的包名，安全码获取方式这里重点说一下，进入我们自定义的签名文件路径然后打开终端，我们这里的目录是android\app目录，输入下面指令获取安全码

  ```
  keytool -v -list -keystore xipingflutter.jks
  输入密钥库口令:
  
  密钥库类型: JKS
  密钥库提供方: SUN
  
  您的密钥库包含 1 个条目
  
  别名: xiping
  创建日期: 2021-6-2
  条目类型: PrivateKeyEntry
  证书链长度: 1
  证书[1]:
  所有者: CN=adam, OU=xiping, O=xiping, L=cn, ST=cn, C=CN
  发布者: CN=adam, OU=xiping, O=xiping, L=cn, ST=cn, C=CN
  序列号: 5c11ef65
  有效期开始日期: Wed Jun 02 16:42:24 CST 2021, 截止日期: Sun Oct 18 16:42:24 CST 2048
  证书指纹:
           MD5: 68:BB:E4:EE:E4:38:F6:0C:F7:BD:82:C5:3C:C9:2F:23
           SHA1: CB:E3:F9:8B:15:48:EE:E9:3D:4E:BE:F4:C4:D1:96:C7:56:DE:C0:AF
           SHA256: FB:47:E4:9E:80:93:50:CF:C1:9E:4F:64:E7:CF:C3:AE:88:F5:48:A9:2C:58:25:07:42:9A:CF:05:9C:C2:D7:5C
           签名算法名称: SHA256withRSA
           版本: 3
  
  ```

上面对应的SHA1就是我们需要在高德控制台填入的安全码`CB:E3:F9:8B:15:48:EE:E9:3D:4E:BE:F4:C4:D1:96:C7:56:DE:C0:AF`,

具体可以参考[高德地图官方文档]([常见问题 | 高德地图API (amap.com)](https://lbs.amap.com/faq/android/map-sdk/create-project/43112))

#### iOS

配置参考 [集成高德地图Flutter插件-地图Flutter插件-开发指南-Flutter插件 | 高德地图API (amap.com)](https://lbs.amap.com/api/flutter/guide/map-flutter-plug-in/map-flutter-info/)

#### Flutter

基本工具类`AmapLocationUtil`：

~~~
import 'dart:async';

import 'package:amap_flutter_location/amap_flutter_location.dart';
import 'package:amap_flutter_location/amap_location_option.dart';
import 'package:permission_handler/permission_handler.dart';
import 'package:xiping/model/location/location_result.dart';

///
/// @author adam
/// @site  https://www.lixiaopeng.top
/// @create 2021/6/2 18:22
///高德地图定位
typedef OnLocationChanged = void  Function(LocationResult result);
typedef OnPermissionDenied =void Function();
class AmapLocationUtil{

  StreamSubscription<Map<String, Object>>? _locationListener;

  AMapFlutterLocation _locationPlugin = new AMapFlutterLocation();

  OnPermissionDenied? onPermissionDenied;
  OnLocationChanged? onLocationChanged;

  AmapLocationUtil({
    this.onLocationChanged,
    this.onPermissionDenied
}){
    initAMapLocationKey();
  }

  /// 动态申请定位权限
  void requestPermissionAndStartLocation() async {
    bool hasLocationPermission = await _requestLocationPermission();
    if (hasLocationPermission) {
      _setLocationOption();
      startLocation();
    } else {
      onPermissionDenied!=null?onPermissionDenied!():print("location permission denied");
    }
  }

  Future<bool> _requestLocationPermission() async {
    var status = await Permission.location.status;
    if (status == PermissionStatus.granted) {
      //已经授权
      return true;
    } else {
      //未授权则发起一次申请
      status = await Permission.location.request();
      if (status == PermissionStatus.granted) {
        return true;
      } else {
        return false;
      }
    }
  }

  void startLocation() => _locationPlugin.startLocation();

  void stopLocation() => _locationPlugin.stopLocation();

  void dispose() {
    if (_locationListener != null) _locationListener!.cancel();
    if (_locationPlugin != null) _locationPlugin.destroy();
  }

  /// 设置 key
  void initAMapLocationKey() {
    AMapFlutterLocation.setApiKey("19edef297ba779471b291bfba9896298", "19edef297ba779471b291bfba9896298");
    _locationListener = _locationPlugin.onLocationChanged().listen(
          (Map<String, Object> result) {
            result.forEach((key, value) {
              print("---location $key -----$value");
            });
        LocationResult location = LocationResult.fromJson(result);
        onLocationChanged!=null?onLocationChanged!(location):print("no location callback");
        if (location.province != null) {
          stopLocation();
        }
      },
    );
  }




  /// 申请定位权限
  /// 授予定位权限返回true， 否则返回false
  Future<bool> requestLocationPermission() async {
    //获取当前的权限
    var status = await Permission.location.status;
    if (status == PermissionStatus.granted) {
      //已经授权
      return true;
    } else {
      //未授权则发起一次申请
      status = await Permission.location.request();
      if (status == PermissionStatus.granted) {
        return true;
      } else {
        return false;
      }
    }
  }



  ///设置定位参数
  void _setLocationOption() {
    AMapLocationOption locationOption = new AMapLocationOption();
    ///是否单次定位
    locationOption.onceLocation = false;

    ///是否需要返回逆地理信息
    locationOption.needAddress = true;

    ///逆地理信息的语言类型
    locationOption.geoLanguage = GeoLanguage.DEFAULT;

    locationOption.desiredLocationAccuracyAuthorizationMode = AMapLocationAccuracyAuthorizationMode.ReduceAccuracy;

    locationOption.fullAccuracyPurposeKey = "AMapLocationScene";

    ///设置Android端连续定位的定位间隔
    locationOption.locationInterval = 2000;

    ///设置Android端的定位模式<br>
    ///可选值：<br>
    ///<li>[AMapLocationMode.Battery_Saving]</li>
    ///<li>[AMapLocationMode.Device_Sensors]</li>
    ///<li>[AMapLocationMode.Hight_Accuracy]</li>
    locationOption.locationMode = AMapLocationMode.Hight_Accuracy;

    ///设置iOS端的定位最小更新距离<br>
    locationOption.distanceFilter = -1;

    ///设置iOS端期望的定位精度
    /// 可选值：<br>
    /// <li>[DesiredAccuracy.Best] 最高精度</li>
    /// <li>[DesiredAccuracy.BestForNavigation] 适用于导航场景的高精度 </li>
    /// <li>[DesiredAccuracy.NearestTenMeters] 10米 </li>
    /// <li>[DesiredAccuracy.Kilometer] 1000米</li>
    /// <li>[DesiredAccuracy.ThreeKilometers] 3000米</li>
    locationOption.desiredAccuracy = DesiredAccuracy.Best;

    ///设置iOS端是否允许系统暂停定位
    locationOption.pausesLocationUpdatesAutomatically = false;

    ///将定位参数设置给定位插件
    _locationPlugin.setLocationOption(locationOption);
  }



}
 
~~~

定位返回结果实体类`LocationResult`:

  ```
  import 'package:json_annotation/json_annotation.dart';
  
  ///
  /// @author adam
  /// @site  https://www.lixiaopeng.top
  /// @create 2021/6/2 19:55
  /// 定位返回数据
  part 'location_result.g.dart';
   @JsonSerializable()
   class LocationResult{
     String? callbackTime;
     String? locationTime;
     int? locationType;
     var latitude;
     var longitude;
     double? accuracy;
     double? altitude;
     double? bearing;
     double? speed;
     String? country;
     String? province;
     String? city;
     String? district;
     String? street;
     String? streetNumber;
     String? cityCode;
     String? adCode;
     String? address;
     String? description;
  
     LocationResult({
       this.callbackTime,
       this.locationTime,
       this.locationType,
       this.latitude,
       this.longitude,
       this.accuracy,
       this.altitude,
       this.bearing,
       this.speed,
       this.country,
       this.province,
       this.city,
       this.district,
       this.street,
       this.streetNumber,
       this.cityCode,
       this.adCode,
       this.address,
       this.description,
     });
  
     factory LocationResult.fromJson(Map<String, dynamic> json) =>
         _$LocationResultFromJson(json);
  
     Map<String, dynamic> toJson() => _$LocationResultToJson(this);
   }
  ```

#### 定位使用

在StateFullWidget中声明工具类：

~~~
  AmapLocationUtil? amapLocationUtil;
~~~

在initState方法中调用定位：

  ```
    @override
    void initState() {
      super.initState();
      amapLocationUtil=AmapLocationUtil(onLocationChanged: (result){
        print("----定位成功----");
        print(result.longitude);
        amapLocationUtil!.stopLocation();
  
      },onPermissionDenied: (){
        print("location permission denied");
      });
      amapLocationUtil!.requestPermissionAndStartLocation();
    }
  ```

在dispose中释放资源：

 ```
   @override
   void dispose() {
     super.dispose();
     if(amapLocationUtil!=null){
       amapLocationUtil!.dispose();
     }
   }
 ```





