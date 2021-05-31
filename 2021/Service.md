### [Google Service](https://developer.android.google.cn/guide/components/services#Foreground)
### 1. 在前台运行服务
- 前台服务是用户主动意识到的一种服务，因此在内存不足时，系统也不会考虑将其终止
- 前台服务必须为状态栏提供通知，将其放在运行中的标题下方
- 对于前台服务,除非将服务停止或从前台移除，否则不能清除该通知
- 台服务必须显示优先级为 **PRIORITY_LOW 或更高** 的状态栏通知
### 2. 服务的生命周期
![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/a42ad88e753445f683317da21000618f~tplv-k3u1fbpfcp-zoom-1.image)
![](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/f3bc5fd45b7e439a821f49e5006e69d4~tplv-k3u1fbpfcp-zoom-1.image)
- Service.stopSelf()
	- 同Context.stopService
    ```
    /**
     * Stop the service, if it was previously started.  This is the same as
     * calling {@link android.content.Context#stopService} for this particular service.
     *  
     * @see #stopSelfResult(int)
     */
    public final void stopSelf() {
        stopSelf(-1);
    }
    ```
- Context.stopService()
	- stopService注释可印证上面图片.startService创建的Service,后续又执行bindService,直接执行stopService,不能直接销毁Service.要先unbind,再stopService.
```
/**
 * 如果stopService之前通过bindService + BIND_AUTO_CREATE,已经生成了 ServiceConnection ,
 * 则所有调用过bindService的客户端都执行unbind之前,该Service不会被销毁.
 *
 * Note that if a stopped service still has {@link ServiceConnection}
 * objects bound to it with the {@link #BIND_AUTO_CREATE} set, it will
 * not be destroyed until all of these bindings are removed.
 */
public abstract boolean stopService(Intent service);
```
### [3. 绑定服务概览/重点是其生命周期](https://developer.android.google.cn/guide/components/bound-services#Lifecycle)

### 4. 各种异常
**1. java.lang.IllegalStateException: Not allowed to start service Intent**
- 异常
  ```
  AndroidRuntime: java.lang.IllegalStateException: Not allowed to start service Intent { cmp=**/.**Service
  (has extras) }: app is in background uid UidRecord{***}
  AndroidRuntime: 	at android.app.ContextImpl.startServiceCommon(ContextImpl.java:1831)
  AndroidRuntime: 	at android.app.ContextImpl.startService(ContextImpl.java:1771)
  AndroidRuntime: 	at android.content.ContextWrapper.startService(ContextWrapper.java:720)
  AndroidRuntime: 	at android.content.ContextWrapper.startService(ContextWrapper.java:720)
  AndroidRuntime: 	at com.**.**Activity$9.run(SourceFile:1657)
  10-12 11:22:55.018  7403  9203 E AndroidRuntime: 	at java.lang.Thread.run(Thread.java:923)
  ```
- 源码中触发位置
  ```
  frameworks/base/core/java/android/app/ContextImpl.java

  private ComponentName startServiceCommon(Intent service, boolean requireForeground,
          UserHandle user) {
      try {
          validateServiceIntent(service);
          service.prepareToLeaveProcess(this);
          ComponentName cn = ActivityManager.getService().startService(
                  mMainThread.getApplicationThread(), service,
                  service.resolveTypeIfNeeded(getContentResolver()), requireForeground,
                  getOpPackageName(), getAttributionTag(), user.getIdentifier());
          if (cn != null) {
              if (cn.getPackageName().equals("!")) {
                  throw new SecurityException(
                          "Not allowed to start service " + service
                          + " without permission " + cn.getClassName());
              } else if (cn.getPackageName().equals("!!")) {
                  throw new SecurityException(
                          "Unable to start service " + service
                          + ": " + cn.getClassName());
              } else if (cn.getPackageName().equals("?")) {
                  //是在这儿抛出了异常
                  throw new IllegalStateException(
                          "Not allowed to start service " + service + ": " + cn.getClassName());
              }
          }
          return cn;
      } catch (RemoteException e) {
          throw e.rethrowFromSystemServer();
      }
  }
  ```
- 具体原因:[解决java.lang.IllegalStateException: Not allowed to start service Intent xxxx app is in background u](https://blog.csdn.net/ouzhuangzhuang/article/details/83540808)
- 解决方案
<br>
Android 8.0 的应用尝试在不允许其创建后台服务的情况下使用 startService() 函数，则该函数将引发一个 IllegalStateException。
<br>
新的 Context.startForegroundService() 函数将启动一个前台服务。现在，即使应用在后台运行，系统也允许其调用 Context.startForegroundService()。
<br>
不过，应用必须在创建服务后的五秒内调用该服务的 startForeground() 函数。
  ```java
  1:startService调用端添加版本判断,使用不同API
  if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.O) {
      context.startForegroundService(new Intent(context, CustomService.class));
  } else {
      context.startService(new Intent(context, CustomService.class));
  }
  2:CustomService必须在onCreate添加对应判断,>=0情况下展示通知栏/前台服务
  @Override
  public void onCreate() {
    super.onCreate();
    if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.O) {
      Notification noti = gainNotificationViaIntent(getIntent());
      startForeground(1,noti); 
    }
  }
  ```
  
### 5. Binder
[上次没砍我的,这次我又来了。看完这篇还不明白Binder你砍我](https://juejin.im/post/6867139592739356686)

### 6. aidl
1. **aidl文件中不要加中文注释,AS会报错.**
2. **在新版AS里面创建aidl文件,会报错**
	- 关闭AS,再重新打开,发现aidl中的文件是乱码,修改一下就好了.
    - 关闭前,aidl文件的内容看着正常,build就会报错,日了.

### 7. Service自定义权限
1. [Android 自定义权限 （ 跨进程调用 Android Service/Broadcast 权限控制）](https://blog.csdn.net/yxz30/article/details/43163339)
2. [Android自定义权限](https://www.cnblogs.com/aimqqroad-13/p/8927179.html)
