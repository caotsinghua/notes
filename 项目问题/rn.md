### react-native

#### 产出

- 安装包 apk/ipa

  新的rn项目中本身有ios/android的项目,打包出来是一个安装包。

- 集成到现有app

  需要复制原先的android目录到空目录，再添加`RCTRootView`

#### 定位/地图

rn原生的geolocation无法在国内安卓使用（需要谷歌框架支持）。需要第三方库。

- react-native-baidu-map（最近更新）
- react-native-amap-geolocation（高德定位模块）
- react-native-amap3d
- 安卓引入sdk-配合rn

#### 推送

- 极光推送 (jpush-react-native,jcore-react-native,)
  - 应用杀死进程后无法推送( https://segmentfault.com/q/1010000017030182 )

- 其他推送

### ui

- [react-native-elements]( https://react-native-elements.github.io/react-native-elements/docs/header.html )
- [nativebase]( https://docs.nativebase.io/Components.html#Components )

### 持久化本地存储

- AsyncStorage（只存字符串键值对）
- realm

### 可能用到的库

 极光推送：https://github.com/jpush/jpush-react-native
emoji表情：https://github.com/omnidan/node-emoji
高德定位：https://github.com/qiuxiang/react-native-amap-geolocation
录音：https://github.com/jsierles/react-native-audio
模糊视图：https://github.com/react-native-community/react-native-blur
相机：https://github.com/react-native-community/react-native-camera
热更新：https://github.com/Microsoft/react-native-code-push
通讯录：https://github.com/rt2zz/react-native-contacts
elements：https://github.com/react-native-training/react-native-elements
文件传输：https://github.com/wkh237/react-native-fetch-blob
iap：https://github.com/dooboolab/react-native-iap
图片选择器，这里有两个，各有特点：
1.https://github.com/react-community/react-native-image-picker
2.https://github.com/ivpusic/react-native-image-crop-picker
图片缓存：https://github.com/wcandillon/react-native-img-cache
进度条：https://github.com/oblador/react-native-progress
二维码生成：https://github.com/cssivision/react-native-qrcode
二维码扫描：https://github.com/moaazsidat/react-native-qrcode-scanner
滚动tab：https://github.com/happypancake/react-native-scrollable-tab-view
音频播放：https://github.com/zmxv/react-native-sound
加载动画：https://github.com/maxs15/react-native-spinkit
启动白屏解决：https://github.com/crazycodeboy/react-native-splash-screen
icon：https://github.com/oblador/react-native-vector-icons
视频播放：https://github.com/react-native-community/react-native-video
本地视频压缩（目前好像只支持IOS，安卓有报错）：https://github.com/shahen94/react-native-video-processing
微信SDK：https://github.com/yorkie/react-native-wechat
很不错的UI库：https://github.com/rilyu/teaset
聊天UI：https://github.com/FaridSafi/react-native-gifted-chat 



### 关于地图需要用到sha1值和bundle identifier

1. 获取开发版

    https://lbs.amap.com/faq/android/map-sdk/create-project/43112 

   ```js
   cd .android
   keytool -v -list -keystore debug.keystore
   ```

   - 找不到密钥库文件

       在.android目录下 keytool -genkey -v -keystore debug.keystore -alias androiddebugkey -keyalg RSA -validity 10000 生成一个

     

2. 获取发布版

   android studio打开项目，terminal下执行keytool -v -list -keystore  ./app/debug.keystore

3. 获取ios itentifier

    https://lbs.amap.com/api/ios-location-sdk/guide/create-project/get-key 

### 关于native-base图标显示[x]的问题

在build.gradle添加

>  apply from: "../../node_modules/react-native-vector-icons/fonts.gradle" 

### 动态路由

- react-navigation不支持（有解决方案，增加复杂度）

