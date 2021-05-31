1. LOCAL_PRIVILEGED_MODULE
    - android.mk下加入一句LOCAL_PRIVILEGED_MODULE := true , APP会被预置到 system/priv-app 下.
    - [LOCAL_PRIVILEGED_MODULE 详解（1）](https://blog.csdn.net/zhanglianyu00/article/details/75099025)
    - [LOCAL_PRIVILEGED_MODULE 详解（2）](https://blog.csdn.net/zhanglianyu00/article/details/75214161)
    - [LOCAL_PRIVILEGED_MODULE 详解（3）](https://blog.csdn.net/zhanglianyu00/article/details/75420583)
    - [LOCAL_PRIVILEGED_MODULE 详解（4）](https://blog.csdn.net/zhanglianyu00/article/details/76086857)
    - [LOCAL_PRIVILEGED_MODULE 详解（5）](https://blog.csdn.net/zhanglianyu00/article/details/76436280)
    - 总结：
        - （1）使用LOCAL_PRIVILEGED_MODULE设置为true编译的app，即ROM中的system/priv-app/下的app，通过PackageManager拿到的ApplicationInfo，其privateFlags字段<<3标志位为1，即ApplicationInfo.PRIVATE_FLAG_PRIVILEGED。也就是说，LOCAL_PRIVILEGED_MODULE为true编译的app即所谓privileged app（特权app）。
        - （2）除此之外，ROM中的system/framework/framework-res.apk，也具有上述特征。
2. LOCAL_PRIVATE_PLATFORM_APIS
    - 对于使用系统@hide api的，我们默认可以设置 LOCAL_PRIVATE_PLATFORM_APIS 为true即可.
3. [Google Android.mk](https://developer.android.google.cn/ndk/guides/android_mk.html)

4. [Apk的几种预置方式](https://www.jianshu.com/p/6fb4e98b9b90)

5. [/system/app 和 /system/priv-app 有什么区别](https://juejin.cn/post/6870822164225622023)
