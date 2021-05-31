### 开源音视频播放器
- [Android开源在线音乐播放器——波尼音乐 非常好](https://github.com/wangchenyan/ponymusic)
- [remusic 非常好](https://github.com/aa112901/remusic)
- [ListenerMusicPlayer 非常牛逼](https://github.com/hefuyicoder/ListenerMusicPlayer)
- [TimberX 非常牛逼](https://github.com/naman14/TimberX)
- [Musicoco 重点看PlayNotifyManager](https://github.com/DuanJiaNing/Musicoco)
- [MusicDNA](https://github.com/harjot-oberai/MusicDNA)
- [Music-Player](https://github.com/andremion/Music-Player)
- [Orin](https://github.com/aliumujib/Orin)
### 非音视频相关知识点
- [动画特辑(二) 音乐律动 非常有意思!!!](https://juejin.im/post/6854573220285284365)
### 音视频相关知识点
1. [Android 音频系统播放延迟时间获取(latency)](https://blog.csdn.net/King1425/article/details/104901314)
    <br>
    getLatency得到的ms值,就是AudioTrack.write写入的byte总量对应的'播放时长',和人耳实际听到的'当前播放时间'的差值.
    ```
    public static int getLatencyByReflect() {
        //set default:80ms
        int latencys = 80;
        if(mAudioTrack != null ) {
            try {
                Class<?> classAudioTrack = mAudioTrack.getClass();
                Method getLatencyMethod = classAudioTrack.getDeclaredMethod("getLatency");
                getLatencyMethod.setAccessible(true);
                latencys = (int) getLatencyMethod.invoke(mAudioTrack);
                latencys = Math.max(latencys, 80);
            } catch (Exception ex) {
                ex.printStackTrace();
            }
        }
        return latencys;
    }
    ```
    ![](//p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/9903cf7b3f86416bbb5382a8eb492741~tplv-k3u1fbpfcp-zoom-1.image)
2. [Android多媒体分析-通过MediaStore获取Audio信息](https://blog.csdn.net/sl1990129/article/details/79932720)
    ```
    //歌曲名  
    String title = cursor.getString(cursor  
            .getColumnIndex(MediaStore.Audio.Media.TITLE));  
      
    //歌手  
    String singer = cursor.getString(cursor  
            .getColumnIndex(MediaStore.Audio.Media.ARTIST));  
      
    //专辑  
    String album = cursor.getString(cursor  
            .getColumnIndex(MediaStore.Audio.Media.ALBUM));  
      
    //长度  
    long size = cursor.getLong(cursor  
            .getColumnIndex(MediaStore.Audio.Media.SIZE));  
      
    //时长  
    int duration = cursor.getInt(cursor  
            .getColumnIndex(MediaStore.Audio.Media.DURATION));  
      
    //路径  
    String url = cursor.getString(cursor  
            .getColumnIndex(MediaStore.Audio.Media.DATA));  
      
    //文件名
    String _display_name = cursor.getString(cursor  
            .getColumnIndex(MediaStore.Audio.Media.DISPLAY_NAME));
    ```
3. [Audio Audio：AudioTrack()中write()函数梳理过程](https://blog.csdn.net/qq_43443900/article/details/103959485)
4. [Android Audio:AudioTrack构造函数分析](https://blog.csdn.net/qq_43443900/article/details/103933776)
5. [AudioRecord的getMinBufferSize函数的分析](https://blog.csdn.net/ameyume/article/details/7677687)
    ![](//p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/0063cbcfa5634cf282da4cce02dab715~tplv-k3u1fbpfcp-zoom-1.image)
6. [DamonRen 有一些音频文章很好](https://juejin.im/user/3491704658998312/posts)
7. [Android 解读开源项目UniversalMusicPlayer（数据管理）](https://juejin.im/post/6844903586392981518)
8. AnliaLee
	- [Android 媒体播放框架MediaSession分析与实践](https://juejin.im/post/6844903575814930439)
    - [Android 解读开源项目UniversalMusicPlayer（播放控制层）](https://juejin.im/post/6844903583557615629)
    - [Android 解读开源项目UniversalMusicPlayer（数据管理）](https://juejin.im/post/6844903586392981518)
    - [UniversalMusicPlayer](https://github.com/android/uamp)
9. 多声道/立体声
	- [Android立体声pcm的数据结构，左右声道拆分、左右声道反转](https://blog.csdn.net/hi_ugly/article/details/80977850)
    - [Android 分离PCM中每个Channel的数据](https://blog.csdn.net/zchy198799/article/details/98882894)
    - [使用Java分离音频左右声道](https://www.jianshu.com/p/cc9e0ee2661d)
    - [视音频数据处理入门：PCM音频采样数据处理](https://blog.csdn.net/leixiaohua1020/article/details/50534316)
