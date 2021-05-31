### 1. 通过包名获取apk路径:<br>
adb shell pm path com.huanhailiuxin.jet2020

### 2. 获取当前显示的Activity:<br>
adb shell dumpsys activity activities | findstr mCurrentFocus
```
D:\***>adb shell dumpsys activity activities | findstr mCurrentFocus
  mCurrentFocus=Window{687e9a6 u0 android/com.android.internal.app.ResolverActivity}

android是包名
com.android.internal.app.ResolverActivity是Activity的具体路径
```

adb shell dumpsys window | findstr mCurrentFocus
  ```
  C:\Users\***>adb shell dumpsys window | findstr mCurrentFocus
    mCurrentFocus=Window{c392414 u0 Application Error: com.huanhailiuxin.jet2020}
    mCurrentFocus=Window{aec97 u0 com.android.settings/com.**.***.TargetActivity}
  ```
### 3. 修改手机分辨率
    - adb shell dumpsys window displays
    - adb shell dumpsys window visible-apps
    - adb shell wm size 1080x1920
    - adb shell wm density 480
    - //wm size reset
    - //wm density reset
![](//p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/dfb04d41c4e64ffd99577f348c6096d7~tplv-k3u1fbpfcp-zoom-1.image)
### 4. 进程
	- 查看指定进程
      ```
      adb shell
      ps -ef | grep com.huanhailiuxin.jet2020
      ```
	- 杀死指定进程
      ```
      kill 进程ID
      ```
	- 查看指定进程oom_adj值
      ```
      cat /proc/进程id/oom_adj
      ```
      
### 5.logcat
1. 查看指定TAG的log
> 只指定TAG输出 : 只看TAG为HelloTag
```
adb shell
logcat -s HelloTag
```
2. 过滤包含指定内容的log
> 只看包含 HelloAdb 的log
```
adb shell
logcat | grep "HelloAdb"
```
3. 将log保存到指定文件
> adb logcat *** > 文件路径
> 注:不要执行 adb shell, 否则提示: /system/bin/sh: can't create ***testlog.txt: Read-only file system
```
adb logcat -s KeyguardStatusView > C:\Users\***\Desktop\testlog.txt
```
4. 只看级别 >=X 的日志
> adb logcat *:E 仅仅看ERROR及ERROR以上级别的日志
```
adb logcat *:E
```
- V — 明细 verbose(最低优先级)
- D — 调试 debug
- I — 信息 info
- W — 警告 warn
- E — 错误 error
- F — 严重错误 fatal
- S — 无记载 silent

### 6.install
- adb install -r 替换已存在的应用程序，也就是说强制安装  
- adb install -l 锁定该应用程序  
- adb install -t 允许测试包  
- adb install -s 把应用程序安装到sd卡上  
- adb install -d 允许进行将见状，也就是安装的比手机上带的版本低  
- adb install -g 为应用程序授予所有运行时的权限
