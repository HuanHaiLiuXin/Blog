### 0. 资料
- [Google 通知概览](https://developer.android.google.cn/guide/topics/ui/notifiers/notifications)
- [Google 创建自定义通知布局](https://developer.android.google.cn/training/notify-user/custom-notification)
- [Android通知栏介绍与适配总结（上篇）](https://sq.163yun.com/blog/article/192705627588341760)
- [Android通知栏介绍与适配总结（下篇）](https://sq.163yun.com/blog/article/192710644351221760)
- [Google通知栏完整DEMO](https://github.com/googlearchive/android-Notifications)

### 1. 问题集锦
1. [android自定义通知栏被压缩解决](https://www.jianshu.com/p/2db4ed955d83)
```
测试系统：andriod 6.0
问题：通过builder.setContent(remoteViews);给通知添加自定义界面，结果由于自定义界面布局过高，底部界面被压缩显示

解决办法：
notification = builder.build();
notification.bigContentView = remoteViews;//通知栏通知
```
2. [Android通知栏详解 解决自定义通知栏显示不全问题](https://www.jianshu.com/p/3aac6df7c2cf)
3. [Android通知栏踩坑记](https://www.jianshu.com/p/5c8f8510e8d5)
4. [android Notification的自定义和实现通知栏的展开和收起](https://blog.csdn.net/gongzhiyao3739124/article/details/52082634)
5. [setContent()方法获得的Notification是定高的 如何实现wrap_content](https://www.cnblogs.com/dongweiq/p/5407815.html)
6. [安卓仿网易云音乐通知栏控制音乐，默认显示Notification bigView 实验下来实际无效果](https://www.jianshu.com/p/77da5abd3870)
7. [Android 7.0消息通知超过3条合并引发的问题](https://blog.csdn.net/qq1073273116/article/details/83087120)
8. [[Android]展开/收起通知栏 Android Studio会报错](https://www.jianshu.com/p/f974fdc34b9e)

### 1. NotificationChannel
### 2. NotificationManager.IMPORTANCE_**
- IMPORTANCE_NONE 关闭通知
- IMPORTANCE_MIN 开启通知，不会弹出，但没有提示音，状态栏中无显示
- IMPORTANCE_LOW 开启通知，不会弹出，不发出提示音，状态栏中显示
- IMPORTANCE_DEFAULT 开启通知，不会弹出，发出提示音，状态栏中显示
- IMPORTANCE_HIGH 开启通知，**会弹出**，发出提示音，状态栏中显示

![](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/6cbb41a490a548d5a3e5a2efa1084657~tplv-k3u1fbpfcp-zoom-1.image)

### 3. RemoteViews支持的控件类型
- RemoteViews只支持4种根布局:
  - FrameLayout
  - LinearLayout
  - RelativeLayout
  - GridLayout
- 根布局下只支持如下控件:
  - AnalogClock
  - Button
  - Chronometer
  - ImageButton
  - ImageView
  - ProgressBar
  - TextView
  - ViewFlipper
  - ListView
  - GridView
  - StackView
  - AdapterViewFlipper
### 4. RemoteViews中的 setInt, setLong, setFloat, set** 方法 : 
- 调用RemoteView中指定ID的控件的指定参数类型的方法
  ```
  //设置RemoteViews中指定ID的TextView的文字颜色
  removeView.setInt(R.id.textView,"setTextColor",specificColor);
  ```
### 5. Android 8.0 及更高版本的设备中:
- 使用 NotificationChannel.setImportance()，而非 NotificationCompat.Builder.setPriority()

### [6. 从通知启动 Activity 应用内常规Activity 及 专门针对通知的Activity 重要](https://developer.android.google.cn/training/notify-user/navigation)

### 7. 自定义通知栏布局高度限制
- 因为抽屉式通知栏中的空间非常有限。自定义通知布局的可用高度取决于通知视图。通常情况下，收起后的视图布局的 **高度上限为 64 dp**，展开后的视图布局的 **高度上限为 256 dp**


