#### 1. 文章
1. Dialog,系统下拉栏 对Activity的生命周期的影响 : 无影响
- [dialog 弹框时activity生命周期调用过程](https://blog.csdn.net/cdkd123/article/details/93078140)
- [Android 下拉通知栏时Activity的生命周期——重新理解onPause()](https://www.jianshu.com/p/781bc86f8042)
- [Android中下拉通知栏，Activity会走哪些生命周期？](https://blog.csdn.net/ming_147/article/details/105634325)
2. Dialog的创建,只能用Activity作为Context?
- [Dialog为何只能用Activity的Context](https://blog.csdn.net/weitangzhu_2008/article/details/86706353)
- [创建Dialog所需的上下文为什么必须是Activity？](https://www.jianshu.com/p/413ec659500a)
3. onPause
- [对onPause调用时机的误解](https://www.jianshu.com/p/f755c2c2ca24)
- [Activity什么时候调用onPause()后不调用onStop()](https://www.jianshu.com/p/f51be7f6b9f8)
- [在activity的主题里加上这条属性，被该activity遮挡的Activity不会调用onStop()](https://blog.csdn.net/qq_26916671/article/details/73201614)
4. 生命周期
- [关于Activity生命周期相关总结](https://my.oschina.net/tanghaoo/blog/1929759)
- [Android-Activity生命周期较详细理解实践](https://zhuanlan.zhihu.com/p/50647518?from_voters_page=true)
- [Activity生命周期里的“隐秘”](https://zhuanlan.zhihu.com/p/23947267)

#### 2.零散的点
1. Activity中弹出Dialog,不会触发Activity任何生命周期函数.
2. 启动1个Activity,然后弹出Dialog,再进入第2个Activity.
  ```
  //启动1个Activity
  TagActivityLifeCycleActivityLife: onCreate
  TagActivityLifeCycleActivityLife: onStart
  TagActivityLifeCycleActivityLife: onResume

  //弹出Dialog
  //不会改变生命周期

  //进入下一个Activity
  TagActivityLifeCycleActivityLife: onPause

  //下一个Activity
  TagActivityLifeCycleActivity2Life: onCreate
  TagActivityLifeCycleActivity2Life: onStart
  TagActivityLifeCycleActivity2Life: onResume

  TagActivityLifeCycleActivityLife: onStop
  TagActivityLifeCycleActivityLife: onSaveInstanceState
  ```
3. Activity的onPause
	- 当前Activity不再可交互,但是在屏幕上依然可见,会触发onPause.
    - 当前Activity的onPause必须执行结束,下一个Activity的onCreate才会执行.所以onPause中不能执行耗时逻辑. 
    - onPause被调用,一定是另一个Activity参与进来了.
    	- Toast,Dialog不会对当前Activity的生命周期产生影响,原因是它们只是使用WindowManager.addView(),并没有产生新的Activity.
		- 下拉通知栏挡住当前Activity,对当前Activity的生命周期也没有影响.
        
    ```
    android/app/Activity.java
        /**
        当前Activity不再可交互,但是在屏幕上依然可见,会触发onPause.
         * Called as part of the activity lifecycle when the user no longer actively interacts with the
         * activity, but it is still visible on screen. The counterpart to {@link #onResume}.
         *
        当前Activity的onPause必须执行结束,下一个Activity的onCreate才会执行.所以onPause中不能执行耗时逻辑. 
         */
        @CallSuper
        protected void onPause() {
    ```
4. Activity生命周期
	- **必须要有另一个Activity的参与,才会引起当前Activity生命周期的变化.**Dialog,Toast,下拉通知栏都不涉及新的Activity,因而不会触发当前Activity生命周期变化.

5. Activity.onWindowFocusChanged
	- 弹出Dialog,下拉栏,系统弹框等 '覆盖当前Activity' 的行为,会触发onWindowFocusChanged(boolean hasFocus)
    - hasFocus: tru-当前Activity获取到焦点 . false-当前Activity失去焦点.
4. 创建Dialog只能用Activity的Context
```
     Caused by: android.view.WindowManager$BadTokenException: Unable to add window -- token null is not valid; is your activity running?
        at android.view.ViewRootImpl.setView(ViewRootImpl.java:965)
        at android.view.WindowManagerGlobal.addView(WindowManagerGlobal.java:387)
        at android.view.WindowManagerImpl.addView(WindowManagerImpl.java:96)
        at android.app.Dialog.show(Dialog.java:344)
        at com.huanhailiuxin.jet2020.dialog.DialogActivity.showApplicationDialog(DialogActivity.java:24)
        ****
```
