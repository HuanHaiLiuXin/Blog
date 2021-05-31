## Google官方
1. [Android 11 中的隐私权](https://developer.android.google.cn/preview/privacy)
2. [行为变更：所有应用](https://developer.android.google.cn/preview/behavior-changes-all)
3. [行为变更：以 Android 11 为目标平台的应用](https://developer.android.google.cn/preview/behavior-changes-11)
## 其他文章
1. [拖不得了，Android11真的要来了，最全适配实践指南奉上](https://juejin.im/post/6860370635664261128)
2. [抢先看！Android11超详细适配攻略！](https://juejin.im/post/6880922939349270541)


## 连接wifi
1. [Android Q 要来了，给你一份很"全面"的适配指南！](https://juejin.cn/post/6844903891113345032)
2. [适用于对等连接的 WLAN 网络请求 API](https://developer.android.google.cn/guide/topics/connectivity/wifi-bootstrap)
    ```java
    final NetworkSpecifier specifier =
      new WifiNetworkSpecifier.Builder()
      .setSsidPattern(new PatternMatcher("test", PatternMatcher.PATTERN_PREFIX))
      .setBssidPattern(MacAddress.fromString("10:03:23:00:00:00"), MacAddress.fromString("ff:ff:ff:00:00:00"))
      .build();

    final NetworkRequest request =
      new NetworkRequest.Builder()
      .addTransportType(NetworkCapabilities.TRANSPORT_WIFI)
      .removeCapability(NetworkCapabilities.NET_CAPABILITY_INTERNET)
      .setNetworkSpecifier(specifier)
      .build();

    final ConnectivityManager connectivityManager = (ConnectivityManager)
      context.getSystemService(Context.CONNECTIVITY_SERVICE);

    final NetworkCallback networkCallback = new NetworkCallback() {
      ...
      @Override
      void onAvailable(...) {
          // do success processing here..
      }

      @Override
      void onUnavailable(...) {
          // do failure processing here..
      }
      ...
    };
    connectivityManager.requestNetwork(request, networkCallback);
    ...
    // Release the request when done.
    connectivityManager.unregisterNetworkCallback(networkCallback);
    ```


## 分区存储
### 文章
- https://developer.android.google.cn/training/data-storage/shared/media#query-collection
- https://zhuanlan.zhihu.com/p/267971545
- https://juejin.im/post/6886618088826273800
- https://www.jianshu.com/p/af9903069ebe
- [Android 10(Q)/11(R) 分区存储适配](https://juejin.im/post/6862633674089693197)
- [Android 外部存储与内部存储详解](https://juejin.im/post/6885696367126118414)
- [Android 11 开发者常见问题: 存储 | FAQ・第二期](https://juejin.im/post/6886663558244139015)

### 自行试验
##### 1. 在内置SD卡上创建文件.
- 在Android R设置上可成功创建
- 在Android Q设备上可成功创建
1. AndroidManifest.xml
    ```xml
    <application
        android:requestLegacyExternalStorage="true"
        >
    ```
2. build.gradle
	- **targetSdkVersion 28 及 targetSdkVersion 29 都可以**
    ```groovy
    android {
        compileSdkVersion 30
        buildToolsVersion "30.0.1"

        defaultConfig {
            minSdkVersion 21
            targetSdkVersion 28
            ***
        }
    ```
3. Code.java
    ```java
    //文件拷贝/文件创建
    String destFilePath = Environment.getExternalStorageDirectory().getAbsolutePath() + File.separator + fileName;
    FileUtils.copyFile(srcFilePath,destFilePath);
    ```
##### 2.发个计算
