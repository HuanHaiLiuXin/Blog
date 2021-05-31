## 说明
这篇文章是将很久以来看过的文章,包括自己写的一些测试代码的总结.属于笔记的性质,没有面面俱到,一些自己相对熟悉的点可能会略过.<br>
最开始看到的性能优化的文章,就是胡凯的优化典范系列,后来又陆续看过一些人写的,个人觉得anly_jun和胡凯的质量最好.<br>
文章大的框架也是先把优化典范过一遍,记录个人认为重要的点,然后是anly_jun的系列,将之前未覆盖的补充进去,也包括HenCoder的一些课程相关内容.<br>
当然除了上面几位,还有很多其他大神的文章,时间久了也记不太清,在此一并谢过.

## 笔记内容引用来源
1. [胡凯](http://hukai.me/android-performance-patterns/)
2. [anly_jun](https://juejin.im/post/6844903449742540814)
3. [HenCoder](https://plus.hencoder.com/)

## 1.Android性能优化之渲染篇
#### 1.VSYNC
1. 帧率:GPU在1秒内绘制操作的帧数.如60fps.
    - 我们通常都会提到60fps与16ms,这是因为人眼与大脑之间的协作无法感知超过60fps的画面更新.
    - 开发app的性能目标就是保持60fps,这意味着每一帧只有16ms=1000/60的时间来处理所有的任务
2. 刷新率:屏幕在1秒内刷新屏幕的次数.如60Hz,每16ms刷新1次屏幕.
3. GPU获取图形数据进行渲染,然后屏幕将渲染后的内容展示在屏幕上.
4. 大多数手机屏幕的刷新率是60Hz,如果GPU渲染1帧的时间低于1000/60=16ms,那么在屏幕刷新时候都有最新帧可显示.如果GPU渲染某1帧 f 的时间超过16ms,在屏幕刷新时候,f并没有被GPU渲染完成则无法展示,屏幕只能继续展示f的上1帧的内容.这就是掉帧,造成了UI界面的卡顿.
<br>
下面展示了帧率正常和帧率低于刷新率(掉帧)的情形

![](https://user-gold-cdn.xitu.io/2018/9/13/165d0b8f11f18a4f?w=485&h=214&f=png&s=57482)

![](https://user-gold-cdn.xitu.io/2018/9/13/165d0b97070dc3fc?w=482&h=341&f=png&s=59978)

#### 2.GPU渲染:GPU渲染依赖2个组件:CPU和GPU

![](https://user-gold-cdn.xitu.io/2018/9/13/165d0e1008dc510e?w=395&h=262&f=png&s=78502)

![](https://user-gold-cdn.xitu.io/2018/9/13/165d0e172470697a?w=378&h=432&f=png&s=89053)

1. CPU负责Measure,Layout,Record,Execute操作.
2. GPU负责Rasterization(栅格化)操作.
    - Resterization栅格化是绘制那些Button，Shape，Path，String，Bitmap等组件最基础的操作.它把组件拆分到不同的像素上进行显示.这是一个很费时的操作.
    - CPU负责把UI组件计算成Polygons(多边形),Texture(纹理),然后交给GPU进行栅格化渲染.
3. 为了App流畅,我们需要确保在16ms内完成所有CPU和GPU的工作.

#### 3.过度绘制
> Overdraw过度绘制是指屏幕上的某个像素在同一帧的时间内被绘制了多次.过度绘制会大量浪费CPU及GPU资源/占用CPU和GPU的处理时间
- 过度绘制的原因
    1. UI布局存在大量重叠
    2. 非必须的背景重叠.
        - 如Activity有背景,Layout又有背景,子View又有背景.仅仅移除非必要背景就可以显著提升性能.
    3. 子View在onDraw中存在重叠部分绘制的情况,比如Bitmap重叠绘制

#### 4.如何提升渲染性能
1. 移除XML布局文件中非必要的Background
2. 保持布局扁平化,尽量避免布局嵌套
3. 在任何时候都避免调用requestLayout(),调用requestLayout会导致该layout的所有父节点都发生重新layout的操作
4. 在自定义View的onDraw中避免过度绘制.
<br>代码实例:
```java
public class OverdrawView extends View {
    public OverdrawView(Context context) {
        super(context);
        init();
    }

    public OverdrawView(Context context, @Nullable AttributeSet attrs) {
        super(context, attrs);
        init();
    }

    public OverdrawView(Context context, @Nullable AttributeSet attrs, int defStyleAttr) {
        super(context, attrs, defStyleAttr);
        init();
    }

    private Paint paint = new Paint(Paint.ANTI_ALIAS_FLAG);
    private Bitmap bitmap1,bitmap2,bitmap3;
    private void init(){
        paint.setStyle(Paint.Style.FILL);
        bitmap1 = BitmapFactory.decodeResource(getResources(),R.mipmap.png1);
        bitmap2 = BitmapFactory.decodeResource(getResources(),R.mipmap.png2);
        bitmap3 = BitmapFactory.decodeResource(getResources(),R.mipmap.png3);
    }

    int w,h;
    @Override
    protected void onMeasure(int widthMeasureSpec, int heightMeasureSpec) {
        super.onMeasure(widthMeasureSpec, heightMeasureSpec);
        w = getMeasuredWidth();
        h = getMeasuredHeight();
    }

    private boolean Overdraw = true;
    @Override
    protected void onDraw(Canvas canvas) {
        if(Overdraw){
            //默认会出现过度绘制
            canvas.drawBitmap(bitmap1,0,0,paint);
            canvas.drawBitmap(bitmap2,w/3,0,paint);
            canvas.drawBitmap(bitmap3,w*2/3,0,paint);
        }else{
            //使用Canvas.clipRect避免过度绘制
            canvas.save();
            canvas.clipRect(0,0,w/3,h);
            canvas.drawBitmap(bitmap1,0,0,paint);
            canvas.restore();
            canvas.save();
            canvas.clipRect(w/3,0,w*2/3,h);
            canvas.drawBitmap(bitmap2,w/3,0,paint);
            canvas.restore();
            canvas.save();
            canvas.clipRect(w*2/3,0,w,h);
            canvas.drawBitmap(bitmap3,w*2/3,0,paint);
            canvas.restore();
        }
    }
    //切换是否避免过度绘制
    public void toggleOverdraw(){
        Overdraw = !Overdraw;
        invalidate();
    }
}
```
<br>效果图:

![过度绘制](https://user-gold-cdn.xitu.io/2018/9/13/165d1a2e0a6240a8?w=540&h=250&f=png&s=47788)

![避免过度绘制](https://user-gold-cdn.xitu.io/2018/9/13/165d1a31eb7a87e4?w=539&h=250&f=png&s=46659)

## 2.Android性能优化之内存篇
#### 1.Android虚拟机的 分代堆内存/Generational Heap Memory模型

![](https://user-gold-cdn.xitu.io/2018/9/13/165d215aa0bf00ea?w=424&h=318&f=png&s=73573)

![](https://user-gold-cdn.xitu.io/2018/9/13/165d215ca8791693?w=426&h=261&f=png&s=52348)

1. 和JVM不同:Android的堆内存多了1个永久代/Permanent Generation.
2. 和JVM类似:
    1. 新创建的对象存储在新生代/Young Generation
    2. GC所占用的时间和它是哪一个Generation有关,Young Generation的每次GC操作时间是最短的,Old Generation其次,Permanent Generation最长
    3. 无论哪一代,触发GC后,所有非垃圾回收线程暂停,GC结束后所有线程恢复执行
3. 如果短时间内进行过多GC,多次暂停线程进行垃圾回收的累积时间就会增大.占用过多的帧间隔时间/16ms,导致CPU和GPU用于计算渲染的时间不足,导致卡顿/掉帧.

#### 2.内存泄漏和内存溢出
内存泄漏就是无用对象占据的内存空间没有及时释放,导致内存空间浪费的情况.memory leak.
<br>
内存溢出是App为1个对象申请内存空间,内存空间不足的情况.out of memory.
<br>
内存泄漏数量足够大,就会引起内存溢出.或者说内存泄漏是内存溢出的原因之一.


## 3.Android性能优化典范-第2季
#### 1.提升动画性能
1. Bitmap的缩放,旋转,裁剪比较耗性能.例如在一个圆形的钟表图上,我们把时钟的指针抠出来当做单独的图片进行旋转会比旋转一张完整的圆形图性能好.
![](https://user-gold-cdn.xitu.io/2018/9/14/165d76d0bedff35f?w=373&h=173&f=jpeg&s=15468)
2. 尽量减少每次重绘的元素可以极大提升性能.可以把复杂的View拆分会更小的View进行组合,在需要刷新界面时候仅对指定View进行重绘.
    - 假如钟表界面上有很多组件,可以把这些组件做拆分,背景图片单独拎出来设置为一个独立的View,通过setLayerType()方法使得这个View强制用Hardware来进行渲染.至于界面上哪些元素需要做拆分,他们各自的更新频率是多少,需要有针对性的单独讨论
#### 2.对象池
1. 短时间内大量对象被创建然后很快被销毁,会多次触发Android虚拟机在Young generation进行GC,使用AS查看内存曲线,会看到内存曲线剧烈起伏,称为"内存抖动".
2. GC会暂停其他线程,短时间多次GC/内存抖动会引起CPU和GPU在16ms内无法完成当前帧的渲染,引起界面卡顿.
3. 避免内存抖动,可以使用对象池
    - 对象池的作用:减少频繁创建和销毁对象带来的成本,实现对象的缓存和复用
    - [1](https://droidyue.com/blog/2016/12/12/dive-into-object-pool/)
[2](https://www.jianshu.com/p/b981bb758d43)
[3](https://blog.csdn.net/self_study/article/details/51477002) [4](https://blog.csdn.net/zuochunsheng/article/details/54980997)
4. 实例
    ```java
    
    public class User {
        public String id;
        public String name;
        //对象池实例
        private static final SynchronizedPool<User> sPool = new SynchronizedPool<User>(10);
    
        public static User obtain() {
            User instance = sPool.acquire();
            return (instance != null) ? instance : new User();
        }
        public void recycle() {
            sPool.release(this);
        }
    }
    ```

#### 3.for index,for simple,iterator三种遍历性能比较
```java
public class ForTest {
    public static void main(String[] args) {
        Vector<Integer> v = new Vector<>();
        ArrayList<Integer> a = new ArrayList<>();
        LinkedList<Integer> l = new LinkedList<>();
        int time = 1000000;
        for(int i = 0; i< time; i++){
            Integer item = new Random().nextInt(time);
            v.add(item);
            a.add(item);
            l.add(item);
        }
        //测试3种遍历性能
        long start = System.currentTimeMillis();
        for(int i = 0;i<v.size();i++){
            Integer item = v.get(i);
        }
        long end = System.currentTimeMillis();
        System.out.println("for index Vector耗时:"+(end-start)+"ms");
        start = System.currentTimeMillis();
        for(int i = 0;i<a.size();i++){
            Integer item = a.get(i);
        }
        end = System.currentTimeMillis();
        System.out.println("for index ArrayList耗时:"+(end-start)+"ms");
        start = System.currentTimeMillis();
        for(int i = 0;i<l.size();i++){
            Integer item = l.get(i);
        }
        end = System.currentTimeMillis();
        System.out.println("for index LinkedList耗时:"+(end-start)+"ms");
        start = System.currentTimeMillis();
        for(Integer item:v){
            Integer i = item;
        }
        end = System.currentTimeMillis();
        System.out.println("for simple Vector耗时:"+(end-start)+"ms");
        start = System.currentTimeMillis();
        for(Integer item:a){
            Integer i = item;
        }
        end = System.currentTimeMillis();
        System.out.println("for simple ArrayList耗时:"+(end-start)+"ms");
        start = System.currentTimeMillis();
        for(Integer item:l){
            Integer i = item;
        }
        end = System.currentTimeMillis();
        System.out.println("for simple LinkedList耗时:"+(end-start)+"ms");
        start = System.currentTimeMillis();
        for(Iterator i = v.iterator();i.hasNext();){
            Integer item = (Integer) i.next();
        }
        end = System.currentTimeMillis();
        System.out.println("for Iterator Vector耗时:"+(end-start)+"ms");
        start = System.currentTimeMillis();
        for(Iterator i = a.iterator();i.hasNext();){
            Integer item = (Integer) i.next();
        }
        end = System.currentTimeMillis();
        System.out.println("for Iterator ArrayList耗时:"+(end-start)+"ms");
        start = System.currentTimeMillis();
        for(Iterator i = l.iterator();i.hasNext();){
            Integer item = (Integer) i.next();
        }
        end = System.currentTimeMillis();
        System.out.println("for Iterator LinkedList耗时:"+(end-start)+"ms");
    }
}

打印结果:
for index Vector耗时:28ms
for index ArrayList耗时:14ms
LinkedList就不能用for index方式进行遍历.
for simple Vector耗时:68ms
for simple ArrayList耗时:11ms
for simple LinkedList耗时:34ms
for Iterator Vector耗时:49ms
for Iterator ArrayList耗时:12ms
for Iterator LinkedList耗时:0ms
```
1. 不要用for index去遍历链表,因为LinkedList在get任何一个位置的数据的时候,都会把前面的数据走一遍.应该使用Iterator去遍历
    1. get(0),直接拿到0位的Node0的地址,拿到Node0里面的数据
    2. get(1),直接拿到0位的Node0的地址,从0位的Node0中找到下一个1位的Node1的地址,找到Node1,拿到Node1里面的数据
    3. get(2),直接拿到0位的Node0的地址,从0位的Node0中找到下一个1位的Node1的地址,找到Node1,从1位的Node1中找到下一个2位的Node2的地址,找到Node2,拿到Node2里面的数据
2. Vector和ArrayList,使用for index遍历效率较高

#### 4.Merge:通过Merge减少1个View层级
1. 可以将merge当做1个ViewGroup v,如果v的类型和v的父控件的类型一致,那么v其实没必要存在,因为白白增加了布局的深度.所以merge使用时必须保证merge中子控件所应该在的ViewGroup类型和merge所在的父控件类型一致.
2. Merge的使用场景有2个:
    1. Activity的布局文件的根布局是FrameLayout,则将FrameLayout替换为merge
        - 因为setContentView本质就是将布局文件inflate后加载到了id为android.id.content的FrameLayout上.
    2. merge作为根布局的布局文件通过include标签被引入其他布局文件中.这时候include所在的父控件,必须和merge所在的布局文件"原本根布局"一致.
3. 代码示例<br>
merge作为根布局的布局文件,用于Activity的setContentView:
    ```xml
    activity_merge.xml
    
    <?xml version="1.0" encoding="utf-8"?>
    <merge xmlns:android="http://schemas.android.com/apk/res/android"
        android:layout_width="match_parent"
        android:layout_height="match_parent">
        <Button
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:layout_marginTop="0dp"
            android:text="111111"
            />
        <Button
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:layout_marginTop="100dp"
            android:layout_marginLeft="40dp"
            android:text="222222"
            />
    </merge>
    ```
    <br>
    merge作为根布局的布局文件,被include标签引入其他布局文件中:
    
    ```xml
    activity_merge_include.xml
    
    <?xml version="1.0" encoding="utf-8"?>
    <LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
        android:orientation="vertical" android:layout_width="match_parent"
        android:layout_height="match_parent">
        <TextView
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:text="merge被include引用"
            />
        <include
            layout="@layout/activity_merge"
            />
    </LinearLayout>
    ```

#### 5.使用.9.png作为背景
- 典型场景是1个ImageView需要添加1个背景图作为边框.这样边框所在矩形的中间部分和实际显示的图片就好重叠发生Overdraw.
- 可以将背景图制作成.9.png.和前景图重叠部分设置为透明.Android的2D渲染器会优化.9.png的透明区域.

#### 6.减少透明区域对性能的影响
- 不透明的View,显示它只需要渲染一次;如果View设置了alpha值,会至少需要渲染两次,性能不好
    - 设置透明度setAlpha的时候,会把当前view绘制到offscreen buffer中,然后再显示出来.offscreen buffer是 一个临时缓冲区,把View放进来并做透明度的转化,然后显示到屏幕上,这个过程性能差,所以应该尽量避免这个过程
- 如何避免使用offscreen buffer
    1. 对于不存在过度绘制的View,如没有背景的TextView,就可以直接设置文字颜色;ImageView设置图片透明度setImageAlpha;自定义View设置绘制时的paint的透明度
    2. 如果是自定义View,确定不存在过度绘制,可以重写hasOverlappingRendering返回false即可.这样设置alpha时android会自动优化,避免使用offscreen buffer.
        ```java
        @Override
        public boolean hasOverlappingRendering() {
            return false;
        }
        ```
    3. 如果不是1,2两种情况,要设置View的透明度,则需要让GPU来渲染指定View,然后再设置透明度.
        ```java
        View v = findViewById(R.id.root);
        //通过setLayerType的方法来指定View应该如何进行渲染
        //开启硬件加速
        v.setLayerType(View.LAYER_TYPE_HARDWARE,null);
        v.setAlpha(0.60F);
        //透明度设置完毕后关闭硬件加速
        v.setLayerType(View.LAYER_TYPE_NONE,null);
        ```

## 4.Android性能优化典范-第3季
#### 1.避免使用枚举,用注解进行替代
1. 枚举的问题
    1. 每个枚举值都是1个对象,相比较Integer和String常量,枚举的内存开销至少是其2倍.
    2. 过多枚举会增加dex大小及其中的方法数量,增加App占用的空间及引发65536几率
2. 如何替代枚举:使用注解
    1. android.support.annotation中的@IntDef,@StringDef来包装Integer和String常量.
    2. 3个步骤
        1. 首先定义常量
        2. 然后自定义注解,设置取值范围就是刚刚定义的常量,并设置自定义注解的保留范围为源码时/SOURCE
        3. 位指定的属性及方法添加自定义注解.
    3. 代码实例
        ```java
        public class MainActivity extends Activity {
            //1:首先定义常量
            public static final int MALE = 0;
            public static final int FEMALE = 1;
            
            @Override
            protected void onCreate(Bundle savedInstanceState) {
                super.onCreate(savedInstanceState);
                setContentView(R.layout.main_activity);
                Person person = new Person();
                person.setSex(MALE);
                ((Button) findViewById(R.id.test)).setText(person.getSexDes());
            }
            class Person {
                //3.为指定的属性及方法添加自定义注解
                @SEX
                private int sex;
                //3.为指定的属性及方法添加自定义注解
                public void setSex(@SEX int sex) {
                    this.sex = sex;
                }
                //3.为指定的属性及方法添加自定义注解
                @SEX
                public int getSex() {
                    return sex;
                }
                public String getSexDes() {
                    if (sex == MALE) {
                        return "男";
                    } else {
                        return "女";
                    }
                }
            }
            //2:然后创建自定义注解,设置取值范围就是刚刚定义的常量,并设置自定义注解的保留范围是源码时
            @IntDef({MALE, FEMALE})
            @Retention(RetentionPolicy.SOURCE)
            public @interface SEX {
            }
        }
        ```

## 5.Android内存优化之OOM
#### 如何避免OOM:
1. 减小对象的内存占用
2. 内存对象复用防止重建
3. 避免内存泄漏
4. 内存使用策略优化
#### 1.减小对象的内存占用
1. 避免使用枚举,用注解替代
2. 减小创建的Bitmap的内存,使用合适的缩放比例及解码格式
    1. inSampleSize:缩放比例
    2. decode format:解码格式
3. 现在很多图片资源的URL都可以添加图片尺寸作为参数.在通过网络获取图片时选择合适的尺寸,减小网络流量消耗,并减小生成的Bitmap的大小.
#### 2.内存对象的重复利用
1. 对象池技术:减少频繁创建和销毁对象带来的成本,实现对象的缓存和复用
2. 尽量使用Android系统内置资源,可降低APK大小,在一定程度降低内存开销
3. ConvertView的复用
4. LRU的机制实现Bitmap的缓存(图片加载框架的必备机制)
5. 在for循环中,用StringBuilder代替String实现字符串拼接
#### 3.避免内存泄漏
1. 在App中使用leakcanary检测内存泄漏:[leakcanary](https://github.com/square/leakcanary)
2. Activity的内存泄漏
    1. Handler引起Activity内存泄漏
        1. 原因:Handler作为Activity的1个非静态内部类实例,持有Activity实例的引用.若Activity退出后Handler依然有待接收的Message,这时候发生GC,Message-Handler-Activity的引用链导致Activity无法被回收.
        2. 2种解决方法
            1. 在onDestroy调用Handler.removeCallbacksAndMessages(null)移除该Handler关联的所有Message及Runnable.再发生GC,Message已经不存在,就可以顺利的回收Handler及Activity
                ```java
                @Override
                protected void onDestroy() {
                    super.onDestroy();
                    m.removeCallbacksAndMessages(null);
                }
                ```
            2. 自定义静态内部类继承Handler,静态内部类实例不持有外部Activity的引用.在自定义Handler中定义外部Activity的弱引用,只有弱引用关联的外部Activity实例未被回收的情况下才继续执行handleMessage.自定义Handler持有外部Activity的弱引用,发生GC时不耽误Activity被回收.
                ```java
                    static class M extends Handler{
                        WeakReference<Activity> mWeakReference;
                        public M(Activity activity)
                        {
                            mWeakReference=new WeakReference<Activity>(activity);
                        }
                        @Override
                        public void handleMessage(Message msg) {
                            if(mWeakReference != null){
                                Activity activity=mWeakReference.get();
                                if(activity != null){
                                    if(msg.what == 15){
                                        Toast.makeText(activity,"M:15",Toast.LENGTH_SHORT).show();
                                    }
                                    if(msg.what == 5){
                                        Toast.makeText(activity,"M:5",Toast.LENGTH_SHORT).show();
                                    }
                                }
                            }
                        }
                    }
                    private M m;
                    @Override
                    protected void onResume() {
                        super.onResume();
                        m = new M(this);
                        m.sendMessageDelayed(m.obtainMessage(15),15000);
                        m.sendMessageDelayed(m.obtainMessage(5),5000);
                    }
                ```
            
            3. 在避免内存泄漏的前提下,如果要求Activity退出就不执行后续动作,用方法1.如果要求后续动作在GC发生前继续执行,使用方法2
3. Context:尽量使用Application Context而不是Activity Context,避免不经意的内存泄漏
4. 资源对象要及时关闭
#### 4.内存使用策略优化
1. 图片选择合适的文件夹进行存放
    - hdpi/xhdpi/xxhdpi等等不同dpi的文件夹下的图片在不同的设备上会经过scale的处理。例如我们只在hdpi的目录下放置了一张100100的图片，那么根据换算关系，xxhdpi的手机去引用那张图片就会被拉伸到200200。需要注意到在这种情况下，内存占用是会显著提高的。对于不希望被拉伸的图片，需要放到assets或者nodpi的目录下
2. 谨慎使用依赖注入框架.依赖注入框架会扫描代码,需要大量的内存空间映射代码.
3. 混淆可以减少不必要的代码,类,方法等.降低映射代码所需的内存空间
4. onLowMemory()与onTrimMemory():没想到应该怎么用
    1. onLowMemory
        - 当所有的background应用都被kill掉的时候,forground应用会收到onLowMemory()的回调.在这种情况下,需要尽快释放当前应用的非必须的内存资源,从而确保系统能够继续稳定运行
    2. onTrimMemory(int level)
        - 当系统内存达到某些条件的时候,所有正在运行的应用都会收到这个回调,同时在这个回调里面会传递以下的参数,代表不同的内存使用情况,收到onTrimMemory()回调的时候,需要根据传递的参数类型进行判断,合理的选择释放自身的一些内存占用,一方面可以提高系统的整体运行流畅度,另外也可以避免自己被系统判断为优先需要杀掉的应用

## 6.Android开发最佳实践
#### 1.注意对隐式Intent的运行时检查保护
1. 类似打开相机等隐式Intent,不一定能够在所有的Android设备上都正常运行.
    - 例如系统相机应用被关闭或者不存在相机应用,或者某些权限被关闭都可能导致抛出ActivityNotFoundException的异常.
    - 预防这个问题的最佳解决方案是在发出这个隐式Intent之前调用resolveActivity做检查
2. 代码实例
    ```java
    public class IntentCheckActivity extends AppCompatActivity {
        @Override
        protected void onCreate(Bundle savedInstanceState) {
            super.onCreate(savedInstanceState);
            setContentView(R.layout.activity_intent_check);
        }
        public void openSBTest(View view) {
            // 跳转到"傻逼"软件
            Intent sbIntent = new Intent("android.media.action.IMAGE_GO_SB");
            if (sbIntent.resolveActivity(getPackageManager()) != null) {
                startActivity(sbIntent);
            } else {
                //会弹出这个提示
                Toast.makeText(this,"设备木有傻逼!",Toast.LENGTH_SHORT).show();
            }
        }
        public void openCameraTest(View view) {
            // 跳转到系统照相机
            Intent cameraIntent = new Intent(MediaStore.ACTION_IMAGE_CAPTURE);
            if (cameraIntent.resolveActivity(getPackageManager()) != null) {
                startActivity(cameraIntent);
                //正常设备会进入相机并弹出提示
                Toast.makeText(this,"设备有相机!",Toast.LENGTH_LONG).show();
            } else {
                Toast.makeText(this,"设备木有相机!",Toast.LENGTH_SHORT).show();
            }
        }
    }
    ```
#### 2.Android 6.0的权限
#### 3.MD新控件的使用:Toolbar替代ActionBar,AppBarLayout,Navigation Drawer, DrawerLayout, NavigationView等

## 7.Android性能优化典范-第4季
#### 1.网络数据的缓存.okHttp,Picasso都支持网络缓存
[okHttp](https://blog.csdn.net/qq_33463102/article/details/60869879) [Picasso](https://www.cnblogs.com/tonycheng93/p/6381757.html)
<br>
[MVP架构实现的Github客户端(4-加入网络缓存)](https://www.jianshu.com/p/faa46bbe8a2e)
#### 2.代码混淆
**2.1.AS中生成keystore.jks应用于APK打包**
<br>
- 1:生成keystore.jks
![](https://user-gold-cdn.xitu.io/2018/9/27/166199a7d90e5065?w=1196&h=589&f=png&s=107815)
<br>

- 2:查看.jks文件的SHA1安全码
<br>
在AS的Terminal中输入:
<br>
keytool -list -v -keystore C:\Users\Administrator\Desktop\key.jks
<br>
keytool -list -v -keystore .jks文件详细路径
<br>
回车后,输入密钥库口令/就是.jks的密码,输入过程不可见,输入完毕回车即可!
<br>

![](https://user-gold-cdn.xitu.io/2018/9/27/16619a624f1ad42a?w=1000&h=853&f=png&s=246896)


**2.2.proguard-rules关键字及部分通配符含义**
<table align="center">
    <tr>
        <td>关键字</td>
        <td>描述</td>
    </tr>
    <tr>
        <td>keep</td>
        <td>保留类和类中的成员，防止它们被混淆或移除</td>
    </tr>
    <tr>
        <td>keepnames</td>
        <td>保留类和类中的成员，防止它们被混淆，但当成员没有被引用时会被移除</td>
    </tr>
    <tr>
        <td>keepclasseswithmembers</td>
        <td>保留类和类中的成员，防止它们被混淆或移除，前提是指名的类中的成员必须存在，如果不存在则还是会混淆</td>
    </tr>
    <tr>
        <td>keepclasseswithmembernames</td>
        <td>保留类和类中的成员，防止它们被混淆，但当成员没有被引用时会被移除，前提是指名的类中的成员必须存在，如果不存在则还是会混淆</td>
    </tr>
    <tr>
        <td>keepclassmembers</td>
        <td>只保留类中的成员，防止它们被混淆或移除</td>
    </tr>
    <tr>
        <td>keepclassmembernames</td>
        <td>只保留类中的成员，防止它们被混淆，但当成员没有被引用时会被移除</td>
    </tr>
</table>
<br>
<table align="center">
    <tr>
        <td>通配符</td>
        <td>描述</td>
    </tr>
    <tr>
        <td>< field ></td>
        <td>匹配类中的所有字段</td>
    </tr>
    <tr>
        <td>< method ></td>
        <td>匹配类中的所有方法</td>
    </tr>
    <tr>
        <td>< init ></td>
        <td>匹配类中的所有构造函数</td>
    </tr>
    <tr>
        <td>*</td>
        <td>
            1.*和字符串联合使用,*代表任意长度的不包含包名分隔符(.)的字符串:<br>
            a.b.c.MainActivity: a.*.*.MainActivity可以匹配;a.*就匹配不上;<br>
            2.*单独使用,就可以匹配所有东西
        </td>
    </tr>
    <tr>
        <td>**</td>
        <td>
            匹配任意长度字符串,包含包名分隔符(.)<br>
            a.b.**可以匹配a.b包下所有内容,包括子包
        </td>
    </tr>
    <tr>
        <td>***</td>
        <td>
            匹配任意参数类型.比如:<br>
            void set*(***)就能匹配任意传入的参数类型;<br>
            *** get*()就能匹配任意返回值的类型
        </td>
    </tr>
    <tr>
        <td>…</td>
        <td>
            匹配任意长度的任意类型参数.比如:<br>
            void test(…)就能匹配任意void test(String a)或者是void test(int a, String b)
        </td>
    </tr>
</table>
<br>

- keep 完整类名{*;}, 可以对指定类进行完全保留,不混淆类名,变量名,方法名.<br>
    ```
    -keep public class a.b.c.TestItem{
        *;
    }
    ```
- 在App中,我们会定义很多实体bean.往往涉及到bean实例和json字符串间互相转换.部分json库会通过反射调用bean的set和get方法.因而实体bean的set,get方法不能被混淆,或者说我们自己写的方法,如果会被第三方库或其他地方通过反射调用,则指定方法要keep避免混淆.
    ```
    -keep public class a.**.Bean.**{
        public void set*(***);
        public *** get*();
        # 对应获取boolean类型属性的方法
        public *** is*();
    }
    ```
- 我们自己写的使用了反射功能的类,必须keep
    ```
    #保留单个包含反射代码的类
    -keep public class a.b.c.ReflectUtils{
        *;
    }
    #保留所有包含反射代码的类,比如所有涉及反射代码的类都在a.b.reflectpackage包及其子包下
    -keep class a.b.reflectpackage.**{
        *;
    } 
    ```
- 如果我们要保留继承了指定类的子类,或者实现了指定接口的类
    ```
    -keep class * extends a.b.c.Parent{*;}
    -keep class * implements a.b.c.OneInterface{*;}
    ```

**2.3.proguard-rules.pro通用模板**

```
#####################基本指令##############################################
# If your project uses WebView with JS, uncomment the following
# and specify the fully qualified class name to the JavaScript interface
# class:
-keepclassmembers class fqcn.of.javascript.interface.for.webview {
   public *;
}

# Uncomment this to preserve the line number information for
# debugging stack traces.
#-keepattributes SourceFile,LineNumberTable

# If you keep the line number information, uncomment this to
# hide the original source file name.
-renamesourcefileattribute SourceFile
#代码混淆压缩比，在0～7之间，默认为5，一般不需要改
-optimizationpasses 5
#混淆时不使用大小写混合，混淆后的类名为小写
-dontusemixedcaseclassnames
#指定不去忽略非公共的库的类
-dontskipnonpubliclibraryclasses
#指定不去忽略非公共的库的类的成员
-dontskipnonpubliclibraryclassmembers
#不做预校验，preverify是proguard的4个步骤之一
#Android不需要preverify，去掉这一步可加快混淆速度
-dontpreverify
#有了verbose这句话，混淆后就会生成映射文件
#包含有类名->混淆后类名的映射关系
-verbose
#然后使用printmapping指定映射文件的名称
-printmapping mapping.txt
#指定混淆时采用的算法，后面的参数是一个过滤器，这个过滤器是谷歌推荐的算法，一般不改变
-optimizations !code/simplification/arithmetic,!field/*,!class/merging/*
#保护代码中的Annotation不被混淆，这在JSON实体映射时非常重要(保留注解参数)
-keepattributes *Annotation*
#避免混淆泛型，这在JSON实体映射时非常重要
-keepattributes Signature
#抛出异常时保留代码行号
-keepattributes SourceFile,LineNumberTable

#忽略所有警告
-ignorewarnings

###################需要保留的东西########################################

#保留反射的方法和类不被混淆================================================
#手动启用support keep注解
#http://tools.android.com/tech-docs/support-annotations

-keep class android.support.annotation.Keep
-keep @android.support.annotation.Keep class * {*;}
-keepclasseswithmembers class * {
    @android.support.annotation.Keep <methods>;
}
-keepclasseswithmembers class * {
    @android.support.annotation.Keep <fields>;
}
-keepclasseswithmembers class * {
    @android.support.annotation.Keep <init>(...);
}

#==========================================================================================

#保留所有的本地native方法不被混淆
-keepclasseswithmembernames class * {
    native <methods>;
}
#保留了继承自Activity、Application这些类的子类
-keep public class * extends android.app.Activity
-keep public class * extends android.app.Fragment
-keep public class * extends android.support.v4.app.Fragment
-keep public class * extends android.app.Application
-keep public class * extends android.app.Service
-keep public class * extends android.content.BroadcastReceiver
-keep public class * extends android.content.ContentProvider
-keep public class * extends android.app.backup.BackupAgentHelper
-keep public class * extends android.preference.Preference
-keep public class * extends android.view.View
-keep public class * extends com.android.vending.licensing.ILicensingService
-keep class android.support.** {*;}

#保留在Activity中的方法参数是view的方法，从而我们在layout里面便携onClick就不会受影响
-keepclassmembers class * extends android.app.Activity{
    public void *(android.view.View);
}
#枚举类不能被混淆
-keepclassmembers enum *{
    public static **[] values();
    public static ** valueOf(java.lang.String);
}
#保留自定义控件不被混淆
-keep public class * extends android.view.View {
    public <init>(android.content.Context);
    public <init>(android.content.Context, android.util.AttributeSet);
    public <init>(android.content.Context, android.util.AttributeSet, int);
    void set*(***);
    *** get*();
}
#保留Parcelable序列化的类不被混淆
-keep class * implements android.os.Parcelable{
    public static final android.os.Parcelable$Creator *;
}
#保留Serializable序列化的类的如下成员不被混淆
-keepclassmembers class * implements java.io.Serializable {
    static final long serialVersionUID;
    private static final java.io.ObjectStreamField[] serialPersistentFields;
    !static !transient <fields>;
    !private <fields>;
    !private <methods>;
    private void writeObject(java.io.ObjectOutputStream);
    private void readObject(java.io.ObjectInputStream);
    java.lang.Object writeReplace();
    java.lang.Object readResolve();
}
#对于R（资源）下的所有类及其方法，都不能被混淆
-keep class **.R$*{
    *;
}
#R文件中的所有记录资源id的静态字段
-keepclassmembers class **.R$* {
    public static <fields>;
}
#对于带有回调函数onXXEvent的，不能被混淆
-keepclassmembers class * {
    void *(**On*Event);
}

#============================针对app的量身定制=============================================

# webView处理，项目中没有使用到webView忽略即可
-keepclassmembers class * extends android.webkit.webViewClient {
    public void *(android.webkit.WebView, java.lang.String, android.graphics.Bitmap);
    public boolean *(android.webkit.WebView, java.lang.String);
}
-keepclassmembers class * extends android.webkit.webViewClient {
    public void *(android.webkit.webView, java.lang.String);
}
```

**2.4.混淆jar包**
<br>
[郭霖大神博客有介绍,自己没试过](https://blog.csdn.net/guolin_blog/article/details/50451259)

**2.5.几条实用的Proguard rules**
<br>
在上面提供的通用模板上继续添加下面几行:
```
-repackageclasses com
-obfuscationdictionary dict.txt
-classobfuscationdictionary dict.txt
-packageobfuscationdictionary dict.txt

-assumenosideeffects class android.util.Log {
    public static boolean isLoggable(java.lang.String, int);
    public static int v(...);
    public static int i(...);
    public static int w(...);
    public static int d(...);
    public static int e(...);
}
```
1. repackageclasses:除了keep的类,会把我们自己写的所有类以及所使用到的各种第三方库代码统统移动到我们指定的单个包下.
    - 比如一些比较敏感的被keep的类在包a.b.min下,我们可以使用 -repackageclasses a.b.min,这样就有成千上万的被混淆的类和未被混淆的敏感的类在a.b.min下面,正常人根本就找不到关键类.尤其是keep的类也只是保留关键方法,名字也被混淆过.
2. -obfuscationdictionary,-classobfuscationdictionary和-packageobfuscationdictionary分别指定变量/方法名,类名,包名混淆后的字符串集.
    - 默认我们的代码命名会被混淆成字母组合,使用这些配置可以用乱码或中文内容进行命名.中文命名可以破坏部分反编译软件的正常工作,乱码则极大加大了查看代码的难度.
    - dict.txt:需要放到和app模块的proguard-rules.pro同级目录.dict.txt具体内容可以自己写,参考开源项目:[一种生成阅读极其困难的proguard字典的算法](https://github.com/hqzxzwb/ProguardDictionaryGenerator)
3. -assumenosideeffects class android.util.Log是在编译成 APK 之前把日志代码全部删掉.
    - [Androidstudio 混淆去掉日志 assumenosideeffects 不起作用](https://www.baidu.com/link?url=UjsDSyf17oY1f42EKK-OfVu8NnK85UHBKm2eevpmnPCiN0VxDp1LnHnTx_mFasO0fyDdTIHAVSsiLQ2EKs0uHOB1b1FEywl6QuWuffg_Lxa&wd=&eqid=d2ba652d00014ea4000000045ba9d047)
        - 因为默认情况下,使用的是proguard-android.txt的混淆规则.proguard-android.txt中包含-dontoptimize.-dontoptimize导致日志语句不会被优化掉.所以我们提供的模板不包含-dontoptimize这一句.

**2.6.字符串硬编码**
1. 对于反编译者来说,最简单的入手点就是字符串搜索.硬编码留在代码里的字符串值都会在反编译过程中被原样恢复,不要使用硬编码.
2. 如果一定要使用硬编码
    1. 新建1个存储硬编码的常量类,静态存放字符串常量,即使找到了常量类,反编译者很难搜索到哪里用了这些字符串.
    2. 常量类中的静态常量字符串,用名称作为真正内容,而值用难以理解的编码表示.
        ```java
        //1:新建常量类,用于存放字符串常量
        public class HardStrings {
            //2:名称是真正内容,值是难以理解的编码.
            //这样即使是必须保存的Log,被反编译者看到的也只是难以理解的值,搞不清意义
            public static final String MaxMemory = "001";
            public static final String M = "002";
            public static final String MemoryClass = "003";
            public static final String LargeMemoryClass = "004";
            public static final String 系统总内存 = "005";
            public static final String 系统剩余内存 = "006";
            public static final String 系统是否处于低内存运行 = "007";
            public static final String 系统剩余内存低于 = "008";
            public static final String M时为低内存运行 = "009";
        }
        ```

**2.7.res资源混淆及多渠道打包**
<br>
简单讲,使用腾讯的2个gradle插件来实现res资源混淆及多渠道打包.
<br>
res资源混淆:[AndResGuard](https://github.com/shwenzhang/AndResGuard)
<br>
多渠道打包:[VasDolly](https://github.com/Tencent/VasDolly)
<br>
[多渠道打包原理+VasDolly和其他多渠道打包方案对比](https://cloud.tencent.com/developer/article/1004884)
<br><br>
具体流程:
<br>
AndResGuard使用了[chaychan](https://www.jianshu.com/p/3ba16f34bba9)的方法,单独创建gradle文件
<br>
1. 项目根目录下build.gradle中，添加插件的依赖,具体如下
    ```
    buildscript {
        repositories {
            google()
            jcenter()
        }
        dependencies {
            classpath 'com.android.tools.build:gradle:3.1.4'
    
            // NOTE: Do not place your application dependencies here; they belong
            // in the individual module build.gradle files
            //添加AndResGuard
            classpath 'com.tencent.mm:AndResGuard-gradle-plugin:1.2.12'
            //添加VasDolly
            classpath 'com.leon.channel:plugin:2.0.1'
        }
    }
    ```
2. 在app目录下单独创建gradle文件and_res_guard.gradle.内容如下
    ```
    apply plugin: 'AndResGuard'
    
    andResGuard {
        mappingFile = null
        use7zip = true
        useSign = true
        keepRoot = false
        compressFilePattern = [
                "*.png",
                "*.jpg",
                "*.jpeg",
                "*.gif",
                "resources.arsc"
        ]
        whiteList = [
    //            // your icon
    //            "R.drawable.icon",
    //            // for fabric
    //            "R.string.com.crashlytics.*",
    //            // for umeng update
    //            "R.string.tb_*",
    //            "R.layout.tb_*",
    //            "R.drawable.tb_*",
    //            "R.drawable.u1*",
    //            "R.drawable.u2*",
    //            "R.color.tb_*",
    //            // umeng share for sina
    //            "R.drawable.sina*",
    //            // for google-services.json
    //            "R.string.google_app_id",
    //            "R.string.gcm_defaultSenderId",
    //            "R.string.default_web_client_id",
    //            "R.string.ga_trackingId",
    //            "R.string.firebase_database_url",
    //            "R.string.google_api_key",
    //            "R.string.google_crash_reporting_api_key",
    //
    //            //友盟
    //            "R.string.umeng*",
    //            "R.string.UM*",
    //            "R.layout.umeng*",
    //            "R.drawable.umeng*",
    //            "R.id.umeng*",
    //            "R.anim.umeng*",
    //            "R.color.umeng*",
    //            "R.style.*UM*",
    //            "R.style.umeng*",
    //
    //            //融云
    //            "R.drawable.u*",
    //            "R.drawable.rc_*",
    //            "R.string.rc_*",
    //            "R.layout.rc_*",
    //            "R.color.rc_*",
    //            "R.id.rc_*",
    //            "R.style.rc_*",
    //            "R.dimen.rc_*",
    //            "R.array.rc_*"
        ]
    
        sevenzip {
            artifact = 'com.tencent.mm:SevenZip:1.2.12'
            //path = "/usr/local/bin/7za"
        }
    }
    ```
3. 模块app下的build.gradle文件添加依赖,具体如下
    ```
    apply plugin: 'com.android.application'
    //引入刚刚创建的and_res_guard.gradle
    apply from: 'and_res_guard.gradle'
    //依赖VasDolly
    apply plugin: 'channel'
    
    channel{
        //指定渠道文件
        channelFile = file("channel.txt")
        //多渠道包的输出目录，默认为new File(project.buildDir,"channel")
        baseOutputDir = new File(project.buildDir,"channel")
        //多渠道包的命名规则，默认为：${appName}-${versionName}-${versionCode}-${flavorName}-${buildType}
        apkNameFormat ='${appName}-${versionName}-${versionCode}-${flavorName}-${buildType}'
        //快速模式：生成渠道包时不进行校验（速度可以提升10倍以上，默认为false）
        isFastMode = true
        //buildTime的时间格式，默认格式：yyyyMMdd-HHmmss
        buildTimeDateFormat = 'yyyyMMdd-HH:mm:ss'
        //低内存模式（仅针对V2签名，默认为false）：只把签名块、中央目录和EOCD读取到内存，不把最大头的内容块读取到内存，在手机上合成APK时，可以使用该模式
        lowMemory = false
    }
    rebuildChannel {
        //指定渠道文件
        channelFile = file("channel.txt")
    //    baseDebugApk = new File(project.projectDir, "app-release_7zip_aligned_signed.apk")
        baseReleaseApk = new File(project.projectDir, "app-release_7zip_aligned_signed.apk")
        //默认为new File(project.buildDir, "rebuildChannel/debug")
    //    debugOutputDir = new File(project.buildDir, "rebuildChannel/debug")
        //默认为new File(project.buildDir, "rebuildChannel/release")
        releaseOutputDir = new File(project.buildDir, "rebuildChannel/release")
        //快速模式：生成渠道包时不进行校验（速度可以提升10倍以上，默认为false）
        isFastMode = false
        //低内存模式（仅针对V2签名，默认为false）：只把签名块、中央目录和EOCD读取到内存，不把最大头的内容块读取到内存，在手机上合成APK时，可以使用该模式
        lowMemory = false
    }
    
    android {
        signingConfigs {
            tcl {
                keyAlias 'qinghailongxin'
                keyPassword 'huanhailiuxin'
                storeFile file('C:/Users/Administrator/Desktop/key.jks')
                storePassword 'huanhailiuxin'
            }
        }
        compileSdkVersion 28
        defaultConfig {
            applicationId "com.example.administrator.proguardapp"
            minSdkVersion 15
            targetSdkVersion 28
            versionCode 1
            versionName "1.0"
            testInstrumentationRunner "android.support.test.runner.AndroidJUnitRunner"
            vectorDrawables.useSupportLibrary = true
        }
        buildTypes {
            release {
                minifyEnabled true
                shrinkResources true
                zipAlignEnabled true
                pseudoLocalesEnabled true
                proguardFiles 'proguard-rules.pro'
                signingConfig signingConfigs.tcl
            }
            debug {
                signingConfig signingConfigs.tcl
                minifyEnabled true
                pseudoLocalesEnabled true
                zipAlignEnabled true
            }
        }
    }
    
    dependencies {
        implementation fileTree(include: ['*.jar'], dir: 'libs')
        implementation 'com.android.support:appcompat-v7:28.0.0'
        implementation 'com.android.support.constraint:constraint-layout:1.1.3'
        implementation 'com.android.support:design:28.0.0'
        implementation 'com.android.support:support-vector-drawable:28.0.0'
        testImplementation 'junit:junit:4.12'
        androidTestImplementation 'com.android.support.test:runner:1.0.2'
        androidTestImplementation 'com.android.support.test.espresso:espresso-core:3.0.2'
        implementation files('libs/litepal-2.0.0.jar')
        //依赖VasDolly
        api 'com.leon.channel:helper:2.0.1'
    }
    ```
4. 首先使用AndResGuard实现资源混淆,再使用VasDolly实现多渠道打包
    1. 在Gradle界面中,找到app模块下andresguard的task.
        - 如果想打debug包,则执行resguardDebug指令;
        - 如果想打release包，则执行resguardRelease指令.
        - 此处我们双击执行resguardRelease指令,在app目录下的/build/output/apk/release/AndResGuard_{apk_name}/ 文件夹中找到混淆后的Apk,其中app-release_aligned_signed.apk为进行混淆并签名过的apk.
        - 我们查看app-release_aligned_signed.apk,res文件夹更名为r,里面的目录名称以及xml文件已经被混淆.
![](https://user-gold-cdn.xitu.io/2018/9/27/1661a5b6d810579b?w=999&h=556&f=png&s=156142)

    2. 将app-release_aligned_signed.apk放到app模块下,在Gradle界面中,找到app模块下channel的task,执行reBuildChannel指令.
        - 双击执行reBuildChannel指令,几秒钟就生成了20个通过app-release_aligned_signed.apk的多渠道apk.
![](https://user-gold-cdn.xitu.io/2018/9/27/1661a62ca123380f?w=485&h=566&f=png&s=10478)
        - 通过helper类库中的ChannelReaderUtil类读取渠道信息
            ```java
            String channel = ChannelReaderUtil.getChannel(getApplicationContext());
            ```

#### 3.APK瘦身
#### 4.更高效的数据序列化:只是看看从没用过,Protocal Buffers,Nano-Proto-Buffers,FlatBuffers
#### 5.数据呈现的顺序以及结构会对序列化之后的空间产生不小的影响

![改变数据结构对内存及gzip的影响](https://user-gold-cdn.xitu.io/2018/9/20/165f4996192feb00?w=700&h=2216&f=png&s=912454)

1. gzip
    1. gzip概念:HTTP协议上的GZIP编码是一种用来改进WEB应用程序性能的技术
        - 一般对纯文本内容可压缩到原大小的40%
        - 减少文件大小有两个明显的好处，一是可以减少存储空间，二是通过网络传输文件时，可以减少传输的时间
    2. okHttp对gzip的支持
        1. okHttp支持gzip自动解压缩,不需要设置Accept-Encoding为gzip
            1. 开发者手动设置Accept-Encoding,okHttp不负责解压缩
            2. 开发者没有设置Accept-Encoding时,则自动添加Accept-Encoding: gzip,自动添加的request,response支持自动解压
                1. 自动解压时移除Content-Length，所以上层Java代码想要contentLength时为-1
                2. 自动解压时移除 Content-Encoding
                3. 自动解压时，如果是分块传输编码，Transfer-Encoding: chunked不受影响
        2. 我们在向服务器提交大量数据的时候,希望对post的数据进行gzip压缩,需要使用[自定义拦截器](okhttp/samples/guide/src/main/java/okhttp3/recipes/RequestBodyCompression.java )
            ```java
            import okhttp3.Interceptor;
            import okhttp3.MediaType;
            import okhttp3.Request;
            import okhttp3.RequestBody;
            import okhttp3.Response;
            import okio.BufferedSink;
            import okio.GzipSink;
            import okio.Okio;
            
            public class GzipRequestInterceptor implements Interceptor {
                @Override
                public Response intercept(Chain chain) throws IOException {
                    Request originalRequest = chain.request();
                    if (originalRequest.body() == null || originalRequest.header("Content-Encoding") != null) {
                        return chain.proceed(originalRequest);
                    }
            
                    Request compressedRequest = originalRequest.newBuilder()
                            .header("Content-Encoding", "gzip")
                            .method(originalRequest.method(), gzip(originalRequest.body()))
                            .build();
                    return chain.proceed(compressedRequest);
                }
            
                private RequestBody gzip(final RequestBody body) {
                    return new RequestBody() {
                        @Override
                        public MediaType contentType() {
                            return body.contentType();
                        }
            
                        @Override
                        public long contentLength() {
                            return -1; // 无法提前知道压缩后的数据大小
                        }
            
                        @Override
                        public void writeTo(BufferedSink sink) throws IOException {
                            BufferedSink gzipSink = Okio.buffer(new GzipSink(sink));
                            body.writeTo(gzipSink);
                            gzipSink.close();
                        }
                    };
                }
            }
            
            然后构建OkhttpClient的时候，添加拦截器：
            OkHttpClient okHttpClient = new OkHttpClient.Builder() 
                .addInterceptor(new GzipRequestInterceptor())//开启Gzip压缩
                ...
                .build();
            ```
2. 改变数据结构,就是将原始集合按照集合中对象的属性进行拆分,变成多个属性集合的形式.
    1. 改变数据结构后JSON字符串长度有明显降低
    2. 使用GZIP对JSON字符串进行压缩,在原始集合中元素数据重复率逐渐变大的情况下,GZIP压缩后的原始JSON字符串长度/GZIP压缩后的改变数据结构的JSON字符串会明显大于1.
    3. 代码及测试结果如下,结果仅供参考(ZipUtils是网上荡的,且没有使用网上用过的Base64Decoder,Base64Encoder)
        ```java
        public final class ZipUtils {

            /**
             * Gzip 压缩数据
             *
             * @param unGzipStr
             * @return
             */
            public static String compressForGzip(String unGzipStr) {

        //        if (TextUtils.isEmpty(unGzipStr)) {
        //            return null;
        //        }
                try {
                    ByteArrayOutputStream baos = new ByteArrayOutputStream();
                    GZIPOutputStream gzip = new GZIPOutputStream(baos);
                    gzip.write(unGzipStr.getBytes());
                    gzip.close();
                    byte[] encode = baos.toByteArray();
                    baos.flush();
                    baos.close();
        //            return Base64Encoder.encode(encode);
                    return new String(encode);
                } catch (UnsupportedEncodingException e) {
                    e.printStackTrace();
                } catch (IOException e) {
                    e.printStackTrace();
                }

                return null;
            }

            /**
             * Gzip解压数据
             *
             * @param gzipStr
             * @return
             */
            public static String decompressForGzip(String gzipStr) {
        //        if (TextUtils.isEmpty(gzipStr)) {
        //            return null;
        //        }
        //        byte[] t = Base64Decoder.decodeToBytes(gzipStr);
                byte[] t = gzipStr.getBytes();
                try {
                    ByteArrayOutputStream out = new ByteArrayOutputStream();
                    ByteArrayInputStream in = new ByteArrayInputStream(t);
                    GZIPInputStream gzip = new GZIPInputStream(in);
                    byte[] buffer = new byte[1024];
                    int n = 0;
                    while ((n = gzip.read(buffer, 0, buffer.length)) > 0) {
                        out.write(buffer, 0, n);
                    }
                    gzip.close();
                    in.close();
                    out.close();
                    return out.toString();
                } catch (IOException e) {
                    e.printStackTrace();
                }
                return null;
            }
            /**
             * Zip 压缩数据
             *
             * @param unZipStr
             * @return
             */
            public static String compressForZip(String unZipStr) {

        //        if (TextUtils.isEmpty(unZipStr)) {
        //            return null;
        //        }
                try {
                    ByteArrayOutputStream baos = new ByteArrayOutputStream();
                    ZipOutputStream zip = new ZipOutputStream(baos);
                    zip.putNextEntry(new ZipEntry("0"));
                    zip.write(unZipStr.getBytes());
                    zip.closeEntry();
                    zip.close();
                    byte[] encode = baos.toByteArray();
                    baos.flush();
                    baos.close();
        //            return Base64Encoder.encode(encode);
                    return new String(encode);
                } catch (UnsupportedEncodingException e) {
                    e.printStackTrace();
                } catch (IOException e) {
                    e.printStackTrace();
                }

                return null;
            }

            /**
             * Zip解压数据
             *
             * @param zipStr
             * @return
             */
            public static String decompressForZip(String zipStr) {

        //        if (TextUtils.isEmpty(zipStr)) {
        //            return null;
        //        }
        //        byte[] t = Base64Decoder.decodeToBytes(zipStr);
                byte[] t = zipStr.getBytes();
                try {
                    ByteArrayOutputStream out = new ByteArrayOutputStream();
                    ByteArrayInputStream in = new ByteArrayInputStream(t);
                    ZipInputStream zip = new ZipInputStream(in);
                    zip.getNextEntry();
                    byte[] buffer = new byte[1024];
                    int n = 0;
                    while ((n = zip.read(buffer, 0, buffer.length)) > 0) {
                        out.write(buffer, 0, n);
                    }
                    zip.close();
                    in.close();
                    out.close();
                    return out.toString("UTF-8");
                } catch (IOException e) {
                    e.printStackTrace();
                }
                return null;
            }
        }

        public class GzipZipTest {
            public static void main(String[] args) {
                GzipZipTest t = new GzipZipTest();
                t.t();
            }
            private void t(){
                /*List<Person> l = new ArrayList<Person>();
                for(int i=0;i<1;i++){
                    for(int j=0;j<6000;j++){
                        Person p = new Person();
                        p.age = j;
                        p.gender = "gender"+j;
                        p.name = "name"+j;
                        l.add(p);
                    }
                }
                Gson gson = new Gson();
                List<String> names = new ArrayList<String>();
                List<String> genders = new ArrayList<String>();
                List<Integer> ages = new ArrayList<Integer>();
                for(Person p:l){
                    names.add(p.name);
                    genders.add(p.gender);
                    ages.add(p.age);
                }
                PersonItemList itemList = new PersonItemList();
                itemList.items = l;
                String jsonDataOri = gson.toJson(itemList);
                System.out.println("原始数据结构 压缩前json数据长度 ---->" + jsonDataOri.length());
                PersonAttrList attrList = new PersonAttrList();
                attrList.names = names;
                attrList.genders = genders;
                attrList.ages = ages;
                String jsonDataVariety = gson.toJson(attrList);
                System.out.println("变种数据结构 压缩前json数据长度 ---->" + jsonDataVariety.length());
                System.out.println("===================================================");

                for(int i=0;i<100;i++){
                    //1:原始数据结构
                    //Gzip压缩
                    long start = System.currentTimeMillis();
                    String gzipStr = ZipUtils.compressForGzip(jsonDataOri);
                    long end = System.currentTimeMillis();
                    System.out.println("原始数据结构 Gzip压缩耗时 cost time---->" + (end - start));
                    System.out.println("原始数据结构 Gzip压缩后json数据长度 ---->" + gzipStr.length());
                    //Zip压缩
        //        start = System.currentTimeMillis();
        //        String zipStr = ZipUtils.compressForZip(jsonDataOri);
        //        end = System.currentTimeMillis();
        //        System.out.println("原始数据结构 Zip压缩耗时 cost time---->" + (end - start));
        //        System.out.println("原始数据结构 Zip压缩后json数据长度 ---->" + zipStr.length());

                    //2:变种数据结构
                    //Gzip压缩
                    start = System.currentTimeMillis();
                    String gzipStrVariety = ZipUtils.compressForGzip(jsonDataVariety);
                    end = System.currentTimeMillis();
                    System.out.println("变种数据结构 Gzip压缩耗时 cost time---->" + (end - start));
                    System.out.println("变种数据结构 Gzip压缩后json数据长度 ---->" + gzipStrVariety.length());
                    //Zip压缩
        //        start = System.currentTimeMillis();
        //        String zipStrVariety = ZipUtils.compressForZip(jsonDataVariety);
        //        end = System.currentTimeMillis();
        //        System.out.println("变种数据结构 Zip压缩耗时 cost time---->" + (end - start));
        //        System.out.println("变种数据结构 Zip压缩后json数据长度 ---->" + zipStrVariety.length());
                    System.out.println("压缩后 原始结构长度:变种数据结构="+((float)gzipStr.length())/(float)gzipStrVariety.length());
                    System.out.println("===================================================");
                }*/

                float repetitionRatio = 0.00F;
                List<Person> l = new ArrayList<Person>();
                for(repetitionRatio = 0.000F; repetitionRatio < 0.500F; repetitionRatio+=0.005F){
                    int reportIndex = (int) (6000 * (1-repetitionRatio));
                    for(int i = 0;i<reportIndex;i++){
                        Person p = new Person();
                        p.age = i;
                        p.gender = "gender"+i;
                        p.name = "name"+i;
                        l.add(p);
                    }
                    if(repetitionRatio > 0.00F){
                        int reportCount = (int) (6000 * repetitionRatio);
                        for(int i = 0;i<reportCount;i++){
                            Person p = new Person();
                            p.age = i;
                            p.gender = "gender"+i;
                            p.name = "name"+i;
                            l.add(p);
                        }
                    }
                    Gson gson = new Gson();
                    List<String> names = new ArrayList<String>();
                    List<String> genders = new ArrayList<String>();
                    List<Integer> ages = new ArrayList<Integer>();
                    for(Person p:l){
                        names.add(p.name);
                        genders.add(p.gender);
                        ages.add(p.age);
                    }
                    PersonItemList itemList = new PersonItemList();
                    itemList.items = l;
                    String jsonDataOri = gson.toJson(itemList);
                    System.out.println("===================================================");
                    System.out.println("原始数据结构 压缩前json数据长度 ---->" + jsonDataOri.length());
                    PersonAttrList attrList = new PersonAttrList();
                    attrList.names = names;
                    attrList.genders = genders;
                    attrList.ages = ages;
                    String jsonDataVariety = gson.toJson(attrList);
                    System.out.println("变种数据结构 压缩前json数据长度 ---->" + jsonDataVariety.length());
                    //1:原始数据结构
                    //Gzip压缩
                    long start = System.currentTimeMillis();
                    String gzipStr = ZipUtils.compressForGzip(jsonDataOri);
                    long end = System.currentTimeMillis();
                    System.out.println("原始数据结构 Gzip压缩后json数据长度 ---->" + gzipStr.length());

                    //2:变种数据结构
                    //Gzip压缩
                    start = System.currentTimeMillis();
                    String gzipStrVariety = ZipUtils.compressForGzip(jsonDataVariety);
                    end = System.currentTimeMillis();
                    System.out.println("变种数据结构 Gzip压缩后json数据长度 ---->" + gzipStrVariety.length());
                    System.out.println("重复率为 "+repetitionRatio/(1-repetitionRatio)+" 压缩后:原始结构长度:变种数据结构="+((float)gzipStr.length())/(float)gzipStrVariety.length());
                }
            }
            public class Person implements Serializable{
                public String name;
                public String gender;
                public int age;

                public String getName() {
                    return name;
                }

                public void setName(String name) {
                    this.name = name;
                }

                public String getGender() {
                    return gender;
                }

                public void setGender(String gender) {
                    this.gender = gender;
                }

                public int getAge() {
                    return age;
                }

                public void setAge(int age) {
                    this.age = age;
                }
            }
            public class PersonItemList implements Serializable{
                public List<Person> items;

                public List<Person> getItems() {
                    return items;
                }

                public void setItems(List<Person> items) {
                    this.items = items;
                }
            }
            public class PersonAttrList implements Serializable{
                public List<String> names;
                public List<String> genders;
                public List<Integer> ages;

                public List<String> getNames() {
                    return names;
                }

                public void setNames(List<String> names) {
                    this.names = names;
                }

                public List<String> getGenders() {
                    return genders;
                }

                public void setGenders(List<String> genders) {
                    this.genders = genders;
                }

                public List<Integer> getAges() {
                    return ages;
                }

                public void setAges(List<Integer> ages) {
                    this.ages = ages;
                }
            }
        }

        首先看当单个对象属性重复率超过100%的情况下打印结果:

        List<Person> l = new ArrayList<Person>();
                for(int i=0;i<1;i++){
                    for(int j=0;j<6000;j++){
                        Person p = new Person();
                        p.age = j;
                        p.gender = "gender"+j;
                        p.name = "name"+j;
                        l.add(p);
                    }
        }
        原始数据结构 压缩前json数据长度 ---->273011	//因为i和j变动,数据会略有变化
        变种数据结构 压缩前json数据长度 ---->129032	//因为i和j变动,数据会略有变化

        i=x;	j=y;

        //x=1,j=6000:代表数据没有任何重复的情形
        x=1;	j=6000;
        原始数据结构 Gzip压缩后json数据长度 ---->44215
        变种数据结构 Gzip压缩后json数据长度 ---->39561
        压缩后 原始结构长度:变种数据结构=1.1176411

        //x=2,j=3000:代表每个对象都存在另1个属性完全一致的对象.单个对象重复率100%
        x=2;	j=3000;
        原始数据结构 Gzip压缩后json数据长度 ---->44204
        变种数据结构 Gzip压缩后json数据长度 ---->27628
        压缩后 原始结构长度:变种数据结构=1.599971

        //余下的代表每单个对象重复率超过100%的情况
        x=3;	j=2000;
        原始数据结构 Gzip压缩后json数据长度 ---->43733
        变种数据结构 Gzip压缩后json数据长度 ---->17020
        压缩后 原始结构长度:变种数据结构=2.5695064

        x=4;	j=1500;
        原始数据结构 Gzip压缩后json数据长度 ---->43398
        变种数据结构 Gzip压缩后json数据长度 ---->13914
        压缩后 原始结构长度:变种数据结构=3.119017

        x=6;	j=1000;
        原始数据结构 Gzip压缩后json数据长度 ---->42166
        变种数据结构 Gzip压缩后json数据长度 ---->8016
        压缩后 原始结构长度:变种数据结构=5.2602296

        x=7;	j=857;
        原始数据结构 Gzip压缩后json数据长度 ---->41743
        变种数据结构 Gzip压缩后json数据长度 ---->7024
        压缩后 原始结构长度:变种数据结构=5.94291

        x=8;	j=750;
        原始数据结构 Gzip压缩后json数据长度 ---->41561
        变种数据结构 Gzip压缩后json数据长度 ---->6378
        压缩后 原始结构长度:变种数据结构=6.516306

        x=9;	j=667;
        原始数据结构 Gzip压缩后json数据长度 ---->41491
        变种数据结构 Gzip压缩后json数据长度 ---->5870
        压缩后 原始结构长度:变种数据结构=7.0683136

        x=10;	j=600;
        原始数据结构 Gzip压缩后json数据长度 ---->7552
        变种数据结构 Gzip压缩后json数据长度 ---->5503
        压缩后 原始结构长度:变种数据结构=1.3723423

        x=12;	j=500;
        原始数据结构 Gzip压缩后json数据长度 ---->6955
        变种数据结构 Gzip压缩后json数据长度 ---->4962
        压缩后 原始结构长度:变种数据结构=1.4016526

        x=15;	j=400;
        原始数据结构 Gzip压缩后json数据长度 ---->6207
        变种数据结构 Gzip压缩后json数据长度 ---->4179
        压缩后 原始结构长度:变种数据结构=1.4852836

        x=20;	j=300;
        原始数据结构 Gzip压缩后json数据长度 ---->5117
        变种数据结构 Gzip压缩后json数据长度 ---->3576
        压缩后 原始结构长度:变种数据结构=1.4309285

        x=30;	j=200;
        原始数据结构 Gzip压缩后json数据长度 ---->4511
        变种数据结构 Gzip压缩后json数据长度 ---->3156
        压缩后 原始结构长度:变种数据结构=1.429341

        x=40;	j=150;
        原始数据结构 Gzip压缩后json数据长度 ---->4359
        变种数据结构 Gzip压缩后json数据长度 ---->3035
        压缩后 原始结构长度:变种数据结构=1.4362438

        x=60;	j=100;
        原始数据结构 Gzip压缩后json数据长度 ---->2832
        变种数据结构 Gzip压缩后json数据长度 ---->1382
        压缩后 原始结构长度:变种数据结构=2.049204

        x=80;	j=75;
        原始数据结构 Gzip压缩后json数据长度 ---->2581
        变种数据结构 Gzip压缩后json数据长度 ---->1217
        压缩后 原始结构长度:变种数据结构=2.1207888

        x=150;	j=40;
        原始数据结构 Gzip压缩后json数据长度 ---->1835
        变种数据结构 Gzip压缩后json数据长度 ---->890
        压缩后 原始结构长度:变种数据结构=2.0617979

        x=200;	j=30;
        原始数据结构 Gzip压缩后json数据长度 ---->1744
        变种数据结构 Gzip压缩后json数据长度 ---->797
        压缩后 原始结构长度:变种数据结构=2.1882057

        x=300;	j=20;
        原始数据结构 Gzip压缩后json数据长度 ---->1539
        变种数据结构 Gzip压缩后json数据长度 ---->739
        压缩后 原始结构长度:变种数据结构=2.082544

        x=316;	j=19;
        原始数据结构 Gzip压缩后json数据长度 ---->1269
        变种数据结构 Gzip压缩后json数据长度 ---->725
        压缩后 原始结构长度:变种数据结构=1.7503449

        x=400;	j=15;
        原始数据结构 Gzip压缩后json数据长度 ---->1488
        变种数据结构 Gzip压缩后json数据长度 ---->662
        压缩后 原始结构长度:变种数据结构=2.247734

        x=500;	j=12;
        原始数据结构 Gzip压缩后json数据长度 ---->1453
        变种数据结构 Gzip压缩后json数据长度 ---->563
        压缩后 原始结构长度:变种数据结构=2.580817

        x=600;	j=10;
        原始数据结构 Gzip压缩后json数据长度 ---->1044
        变种数据结构 Gzip压缩后json数据长度 ---->573
        压缩后 原始结构长度:变种数据结构=1.8219895

        x=667;	j=9;
        原始数据结构 Gzip压缩后json数据长度 ---->1291
        变种数据结构 Gzip压缩后json数据长度 ---->527
        压缩后 原始结构长度:变种数据结构=2.4497154

        x=750;	j=8;
        原始数据结构 Gzip压缩后json数据长度 ---->1155
        变种数据结构 Gzip压缩后json数据长度 ---->520
        压缩后 原始结构长度:变种数据结构=2.2211537

        x=1000;	j=6;
        原始数据结构 Gzip压缩后json数据长度 ---->1269
        变种数据结构 Gzip压缩后json数据长度 ---->429
        压缩后 原始结构长度:变种数据结构=2.958042

        x=1200;	j=5;
        原始数据结构 Gzip压缩后json数据长度 ---->1135
        变种数据结构 Gzip压缩后json数据长度 ---->478
        压缩后 原始结构长度:变种数据结构=2.374477

        x=3000;	j=2;
        原始数据结构 Gzip压缩后json数据长度 ---->990
        变种数据结构 Gzip压缩后json数据长度 ---->382
        压缩后 原始结构长度:变种数据结构=2.591623

        x=6000;	j=1;
        原始数据结构 Gzip压缩后json数据长度 ---->590
        变种数据结构 Gzip压缩后json数据长度 ---->311
        压缩后 原始结构长度:变种数据结构=1.897106

        当每个对象属性重复率低于100%的情况下打印结果:
        ===================================================
        原始数据结构 压缩前json数据长度 ---->314681
        变种数据结构 压缩前json数据长度 ---->170702
        原始数据结构 Gzip压缩后json数据长度 ---->44215
        变种数据结构 Gzip压缩后json数据长度 ---->39561
        重复率为 0.0 压缩后:原始结构长度:变种数据结构=1.1176411
        ===================================================
        原始数据结构 压缩前json数据长度 ---->629141
        变种数据结构 压缩前json数据长度 ---->341162
        原始数据结构 Gzip压缩后json数据长度 ---->88279
        变种数据结构 Gzip压缩后json数据长度 ---->66875
        重复率为 0.0050251256 压缩后:原始结构长度:变种数据结构=1.3200598
        ===================================================
        原始数据结构 压缩前json数据长度 ---->943421
        变种数据结构 压缩前json数据长度 ---->511442
        原始数据结构 Gzip压缩后json数据长度 ---->131892
        变种数据结构 Gzip压缩后json数据长度 ---->90806
        重复率为 0.01010101 压缩后:原始结构长度:变种数据结构=1.4524591
        ===================================================
        原始数据结构 压缩前json数据长度 ---->1257521
        变种数据结构 压缩前json数据长度 ---->681542
        原始数据结构 Gzip压缩后json数据长度 ---->175554
        变种数据结构 Gzip压缩后json数据长度 ---->116973
        重复率为 0.015228426 压缩后:原始结构长度:变种数据结构=1.5008079
        ===================================================
        原始数据结构 压缩前json数据长度 ---->1571501
        变种数据结构 压缩前json数据长度 ---->851522
        原始数据结构 Gzip压缩后json数据长度 ---->218945
        变种数据结构 Gzip压缩后json数据长度 ---->142129
        重复率为 0.020408163 压缩后:原始结构长度:变种数据结构=1.5404668
        ===================================================
        原始数据结构 压缩前json数据长度 ---->1885341
        变种数据结构 压缩前json数据长度 ---->1021386
        原始数据结构 Gzip压缩后json数据长度 ---->262306
        变种数据结构 Gzip压缩后json数据长度 ---->168725
        重复率为 0.025641024 压缩后:原始结构长度:变种数据结构=1.5546362
        ===================================================
        原始数据结构 压缩前json数据长度 ---->2199091
        变种数据结构 压缩前json数据长度 ---->1191160
        原始数据结构 Gzip压缩后json数据长度 ---->305678
        变种数据结构 Gzip压缩后json数据长度 ---->191222
        重复率为 0.030927831 压缩后:原始结构长度:变种数据结构=1.5985503
        ===================================================
        原始数据结构 压缩前json数据长度 ---->2512751
        变种数据结构 压缩前json数据长度 ---->1360844
        原始数据结构 Gzip压缩后json数据长度 ---->348774
        变种数据结构 Gzip压缩后json数据长度 ---->219050
        重复率为 0.036269426 压缩后:原始结构长度:变种数据结构=1.5922118
        ===================================================
        原始数据结构 压缩前json数据长度 ---->2826321
        变种数据结构 压缩前json数据长度 ---->1530438
        原始数据结构 Gzip压缩后json数据长度 ---->391506
        变种数据结构 Gzip压缩后json数据长度 ---->243066
        重复率为 0.041666664 压缩后:原始结构长度:变种数据结构=1.6106983
        ===================================================
        原始数据结构 压缩前json数据长度 ---->3139801
        变种数据结构 压缩前json数据长度 ---->1699942
        原始数据结构 Gzip压缩后json数据长度 ---->434274
        变种数据结构 Gzip压缩后json数据长度 ---->268432
        重复率为 0.047120415 压缩后:原始结构长度:变种数据结构=1.6178175
        ===================================================
        原始数据结构 压缩前json数据长度 ---->3453191
        变种数据结构 压缩前json数据长度 ---->1869356
        原始数据结构 Gzip压缩后json数据长度 ---->476356
        变种数据结构 Gzip压缩后json数据长度 ---->291550
        重复率为 0.052631572 压缩后:原始结构长度:变种数据结构=1.6338742
        ===================================================
        原始数据结构 压缩前json数据长度 ---->3766491
        变种数据结构 压缩前json数据长度 ---->2038680
        原始数据结构 Gzip压缩后json数据长度 ---->518371
        变种数据结构 Gzip压缩后json数据长度 ---->317122
        重复率为 0.058201052 压缩后:原始结构长度:变种数据结构=1.6346107
        ===================================================
        原始数据结构 压缩前json数据长度 ---->4079701
        变种数据结构 压缩前json数据长度 ---->2207914
        原始数据结构 Gzip压缩后json数据长度 ---->560526
        变种数据结构 Gzip压缩后json数据长度 ---->344023
        重复率为 0.06382978 压缩后:原始结构长度:变种数据结构=1.629327
        ===================================================
        原始数据结构 压缩前json数据长度 ---->4392821
        变种数据结构 压缩前json数据长度 ---->2377058
        原始数据结构 Gzip压缩后json数据长度 ---->602208
        变种数据结构 Gzip压缩后json数据长度 ---->365983
        重复率为 0.06951871 压缩后:原始结构长度:变种数据结构=1.6454535
        ===================================================
        原始数据结构 压缩前json数据长度 ---->4705851
        变种数据结构 压缩前json数据长度 ---->2546112
        原始数据结构 Gzip压缩后json数据长度 ---->643532
        变种数据结构 Gzip压缩后json数据长度 ---->391465
        重复率为 0.07526881 压缩后:原始结构长度:变种数据结构=1.6439068
        ===================================================
        原始数据结构 压缩前json数据长度 ---->5018791
        变种数据结构 压缩前json数据长度 ---->2715076
        原始数据结构 Gzip压缩后json数据长度 ---->684775
        变种数据结构 Gzip压缩后json数据长度 ---->415902
        重复率为 0.08108108 压缩后:原始结构长度:变种数据结构=1.6464816
        ===================================================
        原始数据结构 压缩前json数据长度 ---->5331691
        变种数据结构 压缩前json数据长度 ---->2883976
        原始数据结构 Gzip压缩后json数据长度 ---->725952
        变种数据结构 Gzip压缩后json数据长度 ---->438987
        重复率为 0.086956516 压缩后:原始结构长度:变种数据结构=1.6536982
        ===================================================
        原始数据结构 压缩前json数据长度 ---->5644501
        变种数据结构 压缩前json数据长度 ---->3052786
        原始数据结构 Gzip压缩后json数据长度 ---->767578
        变种数据结构 Gzip压缩后json数据长度 ---->464169
        重复率为 0.09289617 压缩后:原始结构长度:变种数据结构=1.6536607
        ===================================================
        原始数据结构 压缩前json数据长度 ---->5957221
        变种数据结构 压缩前json数据长度 ---->3221506
        原始数据结构 Gzip压缩后json数据长度 ---->808616
        变种数据结构 Gzip压缩后json数据长度 ---->488167
        重复率为 0.09890111 压缩后:原始结构长度:变种数据结构=1.6564331
        ===================================================
        原始数据结构 压缩前json数据长度 ---->6269851
        变种数据结构 压缩前json数据长度 ---->3390136
        原始数据结构 Gzip压缩后json数据长度 ---->848776
        变种数据结构 Gzip压缩后json数据长度 ---->511159
        重复率为 0.104972385 压缩后:原始结构长度:变种数据结构=1.6604931
        ===================================================
        原始数据结构 压缩前json数据长度 ---->6582391
        变种数据结构 压缩前json数据长度 ---->3558676
        原始数据结构 Gzip压缩后json数据长度 ---->889184
        变种数据结构 Gzip压缩后json数据长度 ---->536695
        重复率为 0.11111113 压缩后:原始结构长度:变种数据结构=1.6567771
        ===================================================
        原始数据结构 压缩前json数据长度 ---->6894841
        变种数据结构 压缩前json数据长度 ---->3727126
        原始数据结构 Gzip压缩后json数据长度 ---->928982
        变种数据结构 Gzip压缩后json数据长度 ---->557274
        重复率为 0.11731845 压缩后:原始结构长度:变种数据结构=1.6670111
        ===================================================
        原始数据结构 压缩前json数据长度 ---->7207201
        变种数据结构 压缩前json数据长度 ---->3895486
        原始数据结构 Gzip压缩后json数据长度 ---->968845
        变种数据结构 Gzip压缩后json数据长度 ---->583064
        重复率为 0.12359552 压缩后:原始结构长度:变种数据结构=1.6616443
        ===================================================
        原始数据结构 压缩前json数据长度 ---->7519471
        变种数据结构 压缩前json数据长度 ---->4063756
        原始数据结构 Gzip压缩后json数据长度 ---->1013093
        变种数据结构 Gzip压缩后json数据长度 ---->606056
        重复率为 0.12994352 压缩后:原始结构长度:变种数据结构=1.6716162
        ===================================================
        原始数据结构 压缩前json数据长度 ---->7831651
        变种数据结构 压缩前json数据长度 ---->4231936
        原始数据结构 Gzip压缩后json数据长度 ---->1057283
        变种数据结构 Gzip压缩后json数据长度 ---->626963
        重复率为 0.13636366 压缩后:原始结构长度:变种数据结构=1.6863563
        ===================================================
        原始数据结构 压缩前json数据长度 ---->8143741
        变种数据结构 压缩前json数据长度 ---->4400026
        原始数据结构 Gzip压缩后json数据长度 ---->1101480
        变种数据结构 Gzip压缩后json数据长度 ---->650165
        重复率为 0.14285716 压缩后:原始结构长度:变种数据结构=1.6941546
        ===================================================
        原始数据结构 压缩前json数据长度 ---->8455741
        变种数据结构 压缩前json数据长度 ---->4568026
        原始数据结构 Gzip压缩后json数据长度 ---->1145324
        变种数据结构 Gzip压缩后json数据长度 ---->675800
        重复率为 0.1494253 压缩后:原始结构长度:变种数据结构=1.6947677
        ===================================================
        原始数据结构 压缩前json数据长度 ---->8767651
        变种数据结构 压缩前json数据长度 ---->4735936
        原始数据结构 Gzip压缩后json数据长度 ---->1189441
        变种数据结构 Gzip压缩后json数据长度 ---->696474
        重复率为 0.15606937 压缩后:原始结构长度:变种数据结构=1.7078038
        ===================================================
        原始数据结构 压缩前json数据长度 ---->9079471
        变种数据结构 压缩前json数据长度 ---->4903756
        原始数据结构 Gzip压缩后json数据长度 ---->1233352
        变种数据结构 Gzip压缩后json数据长度 ---->720694
        重复率为 0.1627907 压缩后:原始结构长度:变种数据结构=1.7113394
        ===================================================
        原始数据结构 压缩前json数据长度 ---->9391201
        变种数据结构 压缩前json数据长度 ---->5071486
        原始数据结构 Gzip压缩后json数据长度 ---->1277550
        变种数据结构 Gzip压缩后json数据长度 ---->741108
        重复率为 0.16959064 压缩后:原始结构长度:变种数据结构=1.7238379
        ===================================================
        原始数据结构 压缩前json数据长度 ---->9702791
        变种数据结构 压缩前json数据长度 ---->5239100
        原始数据结构 Gzip压缩后json数据长度 ---->1321359
        变种数据结构 Gzip压缩后json数据长度 ---->763320
        重复率为 0.17647058 压缩后:原始结构长度:变种数据结构=1.7310683
        ===================================================
        原始数据结构 压缩前json数据长度 ---->10014291
        变种数据结构 压缩前json数据长度 ---->5406624
        原始数据结构 Gzip压缩后json数据长度 ---->1365756
        变种数据结构 Gzip压缩后json数据长度 ---->782468
        重复率为 0.18343192 压缩后:原始结构长度:变种数据结构=1.7454464
        ===================================================
        原始数据结构 压缩前json数据长度 ---->10325701
        变种数据结构 压缩前json数据长度 ---->5574058
        原始数据结构 Gzip压缩后json数据长度 ---->1409791
        变种数据结构 Gzip压缩后json数据长度 ---->809521
        重复率为 0.19047616 压缩后:原始结构长度:变种数据结构=1.7415125
        ===================================================
        原始数据结构 压缩前json数据长度 ---->10637021
        变种数据结构 压缩前json数据长度 ---->5741402
        原始数据结构 Gzip压缩后json数据长度 ---->1453682
        变种数据结构 Gzip压缩后json数据长度 ---->828981
        重复率为 0.19760476 压缩后:原始结构长度:变种数据结构=1.753577
        ===================================================
        原始数据结构 压缩前json数据长度 ---->10948308
        变种数据结构 压缩前json数据长度 ---->5908713
        原始数据结构 Gzip压缩后json数据长度 ---->1497843
        变种数据结构 Gzip压缩后json数据长度 ---->852966
        重复率为 0.20481923 压缩后:原始结构长度:变种数据结构=1.7560407
        ===================================================
        原始数据结构 压缩前json数据长度 ---->11259595
        变种数据结构 压缩前json数据长度 ---->6076024
        原始数据结构 Gzip压缩后json数据长度 ---->1542039
        变种数据结构 Gzip压缩后json数据长度 ---->872647
        重复率为 0.21212116 压缩后:原始结构长度:变种数据结构=1.7670822
        ===================================================
        原始数据结构 压缩前json数据长度 ---->11570882
        变种数据结构 压缩前json数据长度 ---->6243335
        原始数据结构 Gzip压缩后json数据长度 ---->1585781
        变种数据结构 Gzip压缩后json数据长度 ---->891023
        重复率为 0.21951213 压缩后:原始结构长度:变种数据结构=1.7797307
        ===================================================
        原始数据结构 压缩前json数据长度 ---->11882169
        变种数据结构 压缩前json数据长度 ---->6410646
        原始数据结构 Gzip压缩后json数据长度 ---->1629443
        变种数据结构 Gzip压缩后json数据长度 ---->915561
        重复率为 0.2269938 压缩后:原始结构长度:变种数据结构=1.7797209
        ===================================================
        原始数据结构 压缩前json数据长度 ---->12193456
        变种数据结构 压缩前json数据长度 ---->6577957
        原始数据结构 Gzip压缩后json数据长度 ---->1673135
        变种数据结构 Gzip压缩后json数据长度 ---->937219
        重复率为 0.23456782 压缩后:原始结构长度:变种数据结构=1.7852124
        ===================================================
        原始数据结构 压缩前json数据长度 ---->12504743
        变种数据结构 压缩前json数据长度 ---->6745268
        原始数据结构 Gzip压缩后json数据长度 ---->1717525
        变种数据结构 Gzip压缩后json数据长度 ---->956429
        重复率为 0.24223594 压缩后:原始结构长度:变种数据结构=1.7957684
        ===================================================
        原始数据结构 压缩前json数据长度 ---->12816030
        变种数据结构 压缩前json数据长度 ---->6912579
        原始数据结构 Gzip压缩后json数据长度 ---->1761849
        变种数据结构 Gzip压缩后json数据长度 ---->976092
        重复率为 0.24999991 压缩后:原始结构长度:变种数据结构=1.805003
        ===================================================
        原始数据结构 压缩前json数据长度 ---->13127317
        变种数据结构 压缩前json数据长度 ---->7079890
        原始数据结构 Gzip压缩后json数据长度 ---->1806001
        变种数据结构 Gzip压缩后json数据长度 ---->995442
        重复率为 0.25786152 压缩后:原始结构长度:变种数据结构=1.8142705
        ===================================================
        原始数据结构 压缩前json数据长度 ---->13438604
        变种数据结构 压缩前json数据长度 ---->7247201
        原始数据结构 Gzip压缩后json数据长度 ---->1850241
        变种数据结构 Gzip压缩后json数据长度 ---->1014463
        重复率为 0.26582268 压缩后:原始结构长度:变种数据结构=1.8238624
        ===================================================
        原始数据结构 压缩前json数据长度 ---->13749891
        变种数据结构 压缩前json数据长度 ---->7414512
        原始数据结构 Gzip压缩后json数据长度 ---->1893946
        变种数据结构 Gzip压缩后json数据长度 ---->1038690
        重复率为 0.27388522 压缩后:原始结构长度:变种数据结构=1.8233987
        ===================================================
        原始数据结构 压缩前json数据长度 ---->14061178
        变种数据结构 压缩前json数据长度 ---->7581823
        原始数据结构 Gzip压缩后json数据长度 ---->1938584
        变种数据结构 Gzip压缩后json数据长度 ---->1064229
        重复率为 0.28205115 压缩后:原始结构长度:变种数据结构=1.8215854
        ===================================================
        原始数据结构 压缩前json数据长度 ---->14372465
        变种数据结构 压缩前json数据长度 ---->7749134
        原始数据结构 Gzip压缩后json数据长度 ---->1982416
        变种数据结构 Gzip压缩后json数据长度 ---->1079948
        重复率为 0.29032245 压缩后:原始结构长度:变种数据结构=1.8356588
        ===================================================
        原始数据结构 压缩前json数据长度 ---->14683752
        变种数据结构 压缩前json数据长度 ---->7916445
        原始数据结构 Gzip压缩后json数据长度 ---->2026663
        变种数据结构 Gzip压缩后json数据长度 ---->1102001
        重复率为 0.29870114 压缩后:原始结构长度:变种数据结构=1.8390754
        ===================================================
        原始数据结构 压缩前json数据长度 ---->14995039
        变种数据结构 压缩前json数据长度 ---->8083756
        原始数据结构 Gzip压缩后json数据长度 ---->2070714
        变种数据结构 Gzip压缩后json数据长度 ---->1125712
        重复率为 0.30718938 压缩后:原始结构长度:变种数据结构=1.8394705
        ===================================================
        原始数据结构 压缩前json数据长度 ---->15306326
        变种数据结构 压缩前json数据长度 ---->8251067
        原始数据结构 Gzip压缩后json数据长度 ---->2114297
        变种数据结构 Gzip压缩后json数据长度 ---->1145723
        重复率为 0.3157893 压缩后:原始结构长度:变种数据结构=1.8453823
        ===================================================
        原始数据结构 压缩前json数据长度 ---->15617613
        变种数据结构 压缩前json数据长度 ---->8418378
        原始数据结构 Gzip压缩后json数据长度 ---->2158166
        变种数据结构 Gzip压缩后json数据长度 ---->1164141
        重复率为 0.32450312 压缩后:原始结构长度:变种数据结构=1.8538699
        ===================================================
        原始数据结构 压缩前json数据长度 ---->15928900
        变种数据结构 压缩前json数据长度 ---->8585689
        原始数据结构 Gzip压缩后json数据长度 ---->2201712
        变种数据结构 Gzip压缩后json数据长度 ---->1189557
        重复率为 0.33333313 压缩后:原始结构长度:变种数据结构=1.8508672
        ===================================================
        原始数据结构 压缩前json数据长度 ---->16240187
        变种数据结构 压缩前json数据长度 ---->8753000
        原始数据结构 Gzip压缩后json数据长度 ---->2245653
        变种数据结构 Gzip压缩后json数据长度 ---->1207825
        重复率为 0.3422817 压缩后:原始结构长度:变种数据结构=1.8592536
        ===================================================
        原始数据结构 压缩前json数据长度 ---->16551474
        变种数据结构 压缩前json数据长度 ---->8920311
        原始数据结构 Gzip压缩后json数据长度 ---->2289778
        变种数据结构 Gzip压缩后json数据长度 ---->1228716
        重复率为 0.35135114 压缩后:原始结构长度:变种数据结构=1.8635535
        ===================================================
        原始数据结构 压缩前json数据长度 ---->16862761
        变种数据结构 压缩前json数据长度 ---->9087622
        原始数据结构 Gzip压缩后json数据长度 ---->2333883
        变种数据结构 Gzip压缩后json数据长度 ---->1248197
        重复率为 0.36054403 压缩后:原始结构长度:变种数据结构=1.8698034
        ===================================================
        原始数据结构 压缩前json数据长度 ---->17174048
        变种数据结构 压缩前json数据长度 ---->9254933
        原始数据结构 Gzip压缩后json数据长度 ---->2377734
        变种数据结构 Gzip压缩后json数据长度 ---->1263293
        重复率为 0.3698628 压缩后:原始结构长度:变种数据结构=1.8821714
        ===================================================
        原始数据结构 压缩前json数据长度 ---->17485335
        变种数据结构 压缩前json数据长度 ---->9422244
        原始数据结构 Gzip压缩后json数据长度 ---->2421204
        变种数据结构 Gzip压缩后json数据长度 ---->1286647
        重复率为 0.3793101 压缩后:原始结构长度:变种数据结构=1.8817935
        ===================================================
        原始数据结构 压缩前json数据长度 ---->17796622
        变种数据结构 压缩前json数据长度 ---->9589555
        原始数据结构 Gzip压缩后json数据长度 ---->2464871
        变种数据结构 Gzip压缩后json数据长度 ---->1307479
        重复率为 0.38888866 压缩后:原始结构长度:变种数据结构=1.8852088
        ===================================================
        原始数据结构 压缩前json数据长度 ---->18107909
        变种数据结构 压缩前json数据长度 ---->9756866
        原始数据结构 Gzip压缩后json数据长度 ---->2508873
        变种数据结构 Gzip压缩后json数据长度 ---->1327997
        重复率为 0.39860114 压缩后:原始结构长度:变种数据结构=1.8892158
        ===================================================
        原始数据结构 压缩前json数据长度 ---->18419196
        变种数据结构 压缩前json数据长度 ---->9924177
        原始数据结构 Gzip压缩后json数据长度 ---->2552954
        变种数据结构 Gzip压缩后json数据长度 ---->1342020
        重复率为 0.40845042 压缩后:原始结构长度:变种数据结构=1.9023218
        ===================================================
        原始数据结构 压缩前json数据长度 ---->18730483
        变种数据结构 压缩前json数据长度 ---->10091488
        原始数据结构 Gzip压缩后json数据长度 ---->2596616
        变种数据结构 Gzip压缩后json数据长度 ---->1369092
        重复率为 0.41843942 压缩后:原始结构长度:变种数据结构=1.8965971
        ===================================================
        原始数据结构 压缩前json数据长度 ---->19041770
        变种数据结构 压缩前json数据长度 ---->10258799
        原始数据结构 Gzip压缩后json数据长度 ---->2640984
        变种数据结构 Gzip压缩后json数据长度 ---->1383626
        重复率为 0.42857113 压缩后:原始结构长度:变种数据结构=1.9087412
        ===================================================
        原始数据结构 压缩前json数据长度 ---->19353057
        变种数据结构 压缩前json数据长度 ---->10426110
        原始数据结构 Gzip压缩后json数据长度 ---->2685199
        变种数据结构 Gzip压缩后json数据长度 ---->1402782
        重复率为 0.4388486 压缩后:原始结构长度:变种数据结构=1.9141955
        ===================================================
        原始数据结构 压缩前json数据长度 ---->19664344
        变种数据结构 压缩前json数据长度 ---->10593421
        原始数据结构 Gzip压缩后json数据长度 ---->2729710
        变种数据结构 Gzip压缩后json数据长度 ---->1418750
        重复率为 0.44927505 压缩后:原始结构长度:变种数据结构=1.9240247
        ===================================================
        原始数据结构 压缩前json数据长度 ---->19975631
        变种数据结构 压缩前json数据长度 ---->10760732
        原始数据结构 Gzip压缩后json数据长度 ---->2773735
        变种数据结构 Gzip压缩后json数据长度 ---->1435122
        重复率为 0.45985368 压缩后:原始结构长度:变种数据结构=1.932752
        ===================================================
        原始数据结构 压缩前json数据长度 ---->20286918
        变种数据结构 压缩前json数据长度 ---->10928043
        原始数据结构 Gzip压缩后json数据长度 ---->2818175
        变种数据结构 Gzip压缩后json数据长度 ---->1458645
        重复率为 0.47058788 压缩后:原始结构长度:变种数据结构=1.93205
        ===================================================
        原始数据结构 压缩前json数据长度 ---->20598205
        变种数据结构 压缩前json数据长度 ---->11095354
        原始数据结构 Gzip压缩后json数据长度 ---->2862715
        变种数据结构 Gzip压缩后json数据长度 ---->1473688
        重复率为 0.4814811 压缩后:原始结构长度:变种数据结构=1.9425516
        ===================================================
        原始数据结构 压缩前json数据长度 ---->20909492
        变种数据结构 压缩前json数据长度 ---->11262665
        原始数据结构 Gzip压缩后json数据长度 ---->2906140
        变种数据结构 Gzip压缩后json数据长度 ---->1497577
        重复率为 0.49253693 压缩后:原始结构长度:变种数据结构=1.9405613
        ===================================================
        原始数据结构 压缩前json数据长度 ---->21220779
        变种数据结构 压缩前json数据长度 ---->11429976
        原始数据结构 Gzip压缩后json数据长度 ---->2951053
        变种数据结构 Gzip压缩后json数据长度 ---->1513485
        重复率为 0.50375897 压缩后:原始结构长度:变种数据结构=1.9498396
        ===================================================
        原始数据结构 压缩前json数据长度 ---->21532066
        变种数据结构 压缩前json数据长度 ---->11597287
        原始数据结构 Gzip压缩后json数据长度 ---->2995263
        变种数据结构 Gzip压缩后json数据长度 ---->1528176
        重复率为 0.5151511 压缩后:原始结构长度:变种数据结构=1.9600248
        ===================================================
        原始数据结构 压缩前json数据长度 ---->21843353
        变种数据结构 压缩前json数据长度 ---->11764598
        原始数据结构 Gzip压缩后json数据长度 ---->3039623
        变种数据结构 Gzip压缩后json数据长度 ---->1546990
        重复率为 0.5267171 压缩后:原始结构长度:变种数据结构=1.9648627
        ===================================================
        原始数据结构 压缩前json数据长度 ---->22154640
        变种数据结构 压缩前json数据长度 ---->11931909
        原始数据结构 Gzip压缩后json数据长度 ---->3083971
        变种数据结构 Gzip压缩后json数据长度 ---->1563906
        重复率为 0.5384611 压缩后:原始结构长度:变种数据结构=1.971967
        ===================================================
        原始数据结构 压缩前json数据长度 ---->22465927
        变种数据结构 压缩前json数据长度 ---->12099220
        原始数据结构 Gzip压缩后json数据长度 ---->3128112
        变种数据结构 Gzip压缩后json数据长度 ---->1580792
        重复率为 0.55038714 压缩后:原始结构长度:变种数据结构=1.9788258
        ===================================================
        原始数据结构 压缩前json数据长度 ---->22777214
        变种数据结构 压缩前json数据长度 ---->12266531
        原始数据结构 Gzip压缩后json数据长度 ---->3171693
        变种数据结构 Gzip压缩后json数据长度 ---->1600344
        重复率为 0.5624995 压缩后:原始结构长度:变种数据结构=1.981882
        ===================================================
        原始数据结构 压缩前json数据长度 ---->23088501
        变种数据结构 压缩前json数据长度 ---->12433842
        原始数据结构 Gzip压缩后json数据长度 ---->3215617
        变种数据结构 Gzip压缩后json数据长度 ---->1618740
        重复率为 0.57480264 压缩后:原始结构长度:变种数据结构=1.9864938
        ===================================================
        原始数据结构 压缩前json数据长度 ---->23399788
        变种数据结构 压缩前json数据长度 ---->12601153
        原始数据结构 Gzip压缩后json数据长度 ---->3259832
        变种数据结构 Gzip压缩后json数据长度 ---->1637726
        重复率为 0.5873011 压缩后:原始结构长度:变种数据结构=1.9904624
        ===================================================
        原始数据结构 压缩前json数据长度 ---->23711075
        变种数据结构 压缩前json数据长度 ---->12768464
        原始数据结构 Gzip压缩后json数据长度 ---->3304008
        变种数据结构 Gzip压缩后json数据长度 ---->1652686
        重复率为 0.5999994 压缩后:原始结构长度:变种数据结构=1.9991747
        ===================================================
        原始数据结构 压缩前json数据长度 ---->24022362
        变种数据结构 压缩前json数据长度 ---->12935775
        原始数据结构 Gzip压缩后json数据长度 ---->3347657
        变种数据结构 Gzip压缩后json数据长度 ---->1670445
        重复率为 0.61290264 压缩后:原始结构长度:变种数据结构=2.004051
        ===================================================
        原始数据结构 压缩前json数据长度 ---->24333649
        变种数据结构 压缩前json数据长度 ---->13103086
        原始数据结构 Gzip压缩后json数据长度 ---->3391716
        变种数据结构 Gzip压缩后json数据长度 ---->1683890
        重复率为 0.62601566 压缩后:原始结构长度:变种数据结构=2.0142148
        ===================================================
        原始数据结构 压缩前json数据长度 ---->24644936
        变种数据结构 压缩前json数据长度 ---->13270397
        原始数据结构 Gzip压缩后json数据长度 ---->3436086
        变种数据结构 Gzip压缩后json数据长度 ---->1704452
        重复率为 0.6393436 压缩后:原始结构长度:变种数据结构=2.0159476
        ===================================================
        原始数据结构 压缩前json数据长度 ---->24956223
        变种数据结构 压缩前json数据长度 ---->13437708
        原始数据结构 Gzip压缩后json数据长度 ---->3480064
        变种数据结构 Gzip压缩后json数据长度 ---->1719727
        重复率为 0.65289193 压缩后:原始结构长度:变种数据结构=2.0236142
        ===================================================
        原始数据结构 压缩前json数据长度 ---->25267510
        变种数据结构 压缩前json数据长度 ---->13605019
        原始数据结构 Gzip压缩后json数据长度 ---->3524494
        变种数据结构 Gzip压缩后json数据长度 ---->1735590
        重复率为 0.666666 压缩后:原始结构长度:变种数据结构=2.030718
        ===================================================
        原始数据结构 压缩前json数据长度 ---->25578797
        变种数据结构 压缩前json数据长度 ---->13772330
        原始数据结构 Gzip压缩后json数据长度 ---->3569109
        变种数据结构 Gzip压缩后json数据长度 ---->1757409
        重复率为 0.6806716 压缩后:原始结构长度:变种数据结构=2.0308926
        ===================================================
        原始数据结构 压缩前json数据长度 ---->25890084
        变种数据结构 压缩前json数据长度 ---->13939641
        原始数据结构 Gzip压缩后json数据长度 ---->3613919
        变种数据结构 Gzip压缩后json数据长度 ---->1770126
        重复率为 0.6949145 压缩后:原始结构长度:变种数据结构=2.041617
        ===================================================
        原始数据结构 压缩前json数据长度 ---->26201371
        变种数据结构 压缩前json数据长度 ---->14106952
        原始数据结构 Gzip压缩后json数据长度 ---->3658034
        变种数据结构 Gzip压缩后json数据长度 ---->1787002
        重复率为 0.70940095 压缩后:原始结构长度:变种数据结构=2.0470228
        ===================================================
        原始数据结构 压缩前json数据长度 ---->26512658
        变种数据结构 压缩前json数据长度 ---->14274263
        原始数据结构 Gzip压缩后json数据长度 ---->3702835
        变种数据结构 Gzip压缩后json数据长度 ---->1799515
        重复率为 0.7241371 压缩后:原始结构长度:变种数据结构=2.057685
        ===================================================
        原始数据结构 压缩前json数据长度 ---->26823945
        变种数据结构 压缩前json数据长度 ---->14441574
        原始数据结构 Gzip压缩后json数据长度 ---->3746980
        变种数据结构 Gzip压缩后json数据长度 ---->1818417
        重复率为 0.7391296 压缩后:原始结构长度:变种数据结构=2.0605724
        ===================================================
        原始数据结构 压缩前json数据长度 ---->27135232
        变种数据结构 压缩前json数据长度 ---->14608885
        原始数据结构 Gzip压缩后json数据长度 ---->3790555
        变种数据结构 Gzip压缩后json数据长度 ---->1836003
        重复率为 0.7543851 压缩后:原始结构长度:变种数据结构=2.064569
        ===================================================
        原始数据结构 压缩前json数据长度 ---->27446519
        变种数据结构 压缩前json数据长度 ---->14776196
        原始数据结构 Gzip压缩后json数据长度 ---->3834464
        变种数据结构 Gzip压缩后json数据长度 ---->1851563
        重复率为 0.76991063 压缩后:原始结构长度:变种数据结构=2.0709336
        ===================================================
        原始数据结构 压缩前json数据长度 ---->27757806
        变种数据结构 压缩前json数据长度 ---->14943507
        原始数据结构 Gzip压缩后json数据长度 ---->3879072
        变种数据结构 Gzip压缩后json数据长度 ---->1873192
        重复率为 0.7857134 压缩后:原始结构长度:变种数据结构=2.0708354
        ===================================================
        原始数据结构 压缩前json数据长度 ---->28069093
        变种数据结构 压缩前json数据长度 ---->15110818
        原始数据结构 Gzip压缩后json数据长度 ---->3923316
        变种数据结构 Gzip压缩后json数据长度 ---->1894024
        重复率为 0.80180085 压缩后:原始结构长度:变种数据结构=2.0714183
        ===================================================
        原始数据结构 压缩前json数据长度 ---->28380380
        变种数据结构 压缩前json数据长度 ---->15278129
        原始数据结构 Gzip压缩后json数据长度 ---->3967482
        变种数据结构 Gzip压缩后json数据长度 ---->1916387
        重复率为 0.81818086 压缩后:原始结构长度:变种数据结构=2.0702927
        ===================================================
        原始数据结构 压缩前json数据长度 ---->28691667
        变种数据结构 压缩前json数据长度 ---->15445440
        原始数据结构 Gzip压缩后json数据长度 ---->4011094
        变种数据结构 Gzip压缩后json数据长度 ---->1933486
        重复率为 0.8348614 压缩后:原始结构长度:变种数据结构=2.07454
        ===================================================
        原始数据结构 压缩前json数据长度 ---->29002954
        变种数据结构 压缩前json数据长度 ---->15612751
        原始数据结构 Gzip压缩后json数据长度 ---->4055289
        变种数据结构 Gzip压缩后json数据长度 ---->1953997
        重复率为 0.8518508 压缩后:原始结构长度:变种数据结构=2.0753813
        ===================================================
        原始数据结构 压缩前json数据长度 ---->29314241
        变种数据结构 压缩前json数据长度 ---->15780062
        原始数据结构 Gzip压缩后json数据长度 ---->4099592
        变种数据结构 Gzip压缩后json数据长度 ---->1974066
        重复率为 0.8691578 压缩后:原始结构长度:变种数据结构=2.076725
        ===================================================
        原始数据结构 压缩前json数据长度 ---->29625528
        变种数据结构 压缩前json数据长度 ---->15947373
        原始数据结构 Gzip压缩后json数据长度 ---->4143573
        变种数据结构 Gzip压缩后json数据长度 ---->1987771
        重复率为 0.88679135 压缩后:原始结构长度:变种数据结构=2.0845323
        ===================================================
        原始数据结构 压缩前json数据长度 ---->29936815
        变种数据结构 压缩前json数据长度 ---->16114684
        原始数据结构 Gzip压缩后json数据长度 ---->4187707
        变种数据结构 Gzip压缩后json数据长度 ---->2014350
        重复率为 0.9047608 压缩后:原始结构长度:变种数据结构=2.078937
        ===================================================
        原始数据结构 压缩前json数据长度 ---->30248102
        变种数据结构 压缩前json数据长度 ---->16281995
        原始数据结构 Gzip压缩后json数据长度 ---->4232504
        变种数据结构 Gzip压缩后json数据长度 ---->2034384
        重复率为 0.92307574 压缩后:原始结构长度:变种数据结构=2.0804844
        ===================================================
        原始数据结构 压缩前json数据长度 ---->30559389
        变种数据结构 压缩前json数据长度 ---->16449306
        原始数据结构 Gzip压缩后json数据长度 ---->4277046
        变种数据结构 Gzip压缩后json数据长度 ---->2053854
        重复率为 0.94174635 压缩后:原始结构长度:变种数据结构=2.082449
        ===================================================
        原始数据结构 压缩前json数据长度 ---->30870676
        变种数据结构 压缩前json数据长度 ---->16616617
        原始数据结构 Gzip压缩后json数据长度 ---->4321134
        变种数据结构 Gzip压缩后json数据长度 ---->2072485
        重复率为 0.960783 压缩后:原始结构长度:变种数据结构=2.0850012
        ===================================================
        原始数据结构 压缩前json数据长度 ---->31181963
        变种数据结构 压缩前json数据长度 ---->16783928
        原始数据结构 Gzip压缩后json数据长度 ---->4365924
        变种数据结构 Gzip压缩后json数据长度 ---->2087159
        重复率为 0.9801967 压缩后:原始结构长度:变种数据结构=2.0918024
        ===================================================
        原始数据结构 压缩前json数据长度 ---->31493250
        变种数据结构 压缩前json数据长度 ---->16951239
        原始数据结构 Gzip压缩后json数据长度 ---->4409476
        变种数据结构 Gzip压缩后json数据长度 ---->2100664
        重复率为 0.9999986 压缩后:原始结构长度:变种数据结构=2.0990868

        ```

## 8.Android性能优化典范-第5季
[多线程大部分内容源自凯哥的课程,个人觉得比优化典范写得清晰得多](https://plus.hencoder.com/)
#### 1.线程
1. 线程就是代码线性执行,执行完毕就结束的一条线.UI线程不会结束是因为其初始化完毕后会执行死循环,所以永远不会执行完毕.
2. 如何简单创建新线程:
    ```java
    //1:直接创建Thread,执行其start方法
    Thread t1 = new Thread(){
        @Override
        public void run() {
            System.out.println("Thread:run");
        }
    };
    t1.start();
    //2:使用Runnable实例作为参数创建Thread,执行start
    Runnable runnable = new Runnable() {
        @Override
        public void run() {
            System.out.println("Runnable:run");
        }
    };
    Thread t2 = new Thread(runnable);
    t2.start();
    ```
    - 两种方式创建新线程性能无差别,使用Runnable实例适用于希望Runnable复用的情形
    - 常用的创建线程池2种方式
        1. Executors.newCachedThreadPool():一般情况下使用newCachedThreadPool即可.
        2. Executors.newFixedThreadPool(int number):短时批量处理/比如要并行处理多张图片,可以直接创建包含图片精确数量的线程的线程池并行处理.
            ```java
            Runnable runnable = new Runnable() {
                @Override
                public void run() {
                    System.out.println("runnable:run()");
                }
            };
            ExecutorService executorService = Executors.newCachedThreadPool();
            executorService.execute(runnable);
            executorService.execute(runnable);
            executorService.execute(runnable);
            executorService.shutdown();
            //比如有40张图片要同时处理
            //创建包含40个线程的线程池,每个线程处理一张图片,处理完毕后shutdown
            ExecutorService service = Executors.newFixedThreadPool(40);
            for(Bitmap item:bitmaps){
                //比如runnable就是处理单张图片的
                service.execute(runnable);
            }
            service.shutdown();
            ```
        3. 《阿里巴巴Java开发手册》规定:
            - 线程池不允许使用 Executors 去创建,而是通过 ThreadPoolExecutor 的方式.这样
的处理方式让写的同学更加明确线程池的运行规则，规避资源耗尽的风险
            - 看Android中Executors源码.Executors.newCachedThreadPool/newScheduledThreadPool允许的创建线程数量为 Integer.MAX_VALUE,可能会创建大量的线程,从而导致 OOM.而newFixedThreadPool,newSingleThreadExecutor不会存在这种风险.
        4. [如何正确创建ThreadPoolExecutor:有点麻烦,晚点详述]()
        5. ExecutorService的shutdown和shutdownNow
            1. shutdown:在调用shutdown之前ExecutorService中已经启动的线程,在调用shutdown后,线程如果执行未结束会继续执行完毕并结束,但不会再启动新的线程执行新任务.
            2. shutdownNow:首先停止启动新的线程执行新任务;并尝试结束所有正在执行的线程,正在执行的线程可能被终止也可能会继续执行完成.
3. 如何正确创建ThreadPoolExecutor<br>
3.1:ThreadPoolExecutor构造参数
    ```java
    public ThreadPoolExecutor(int corePoolSize,
                              int maximumPoolSize,
                              long keepAliveTime,
                              TimeUnit unit,
                              BlockingQueue<Runnable> workQueue,
                              ThreadFactory threadFactory,
                              RejectedExecutionHandler handler)
    ```
    1. int corePoolSize:该线程池中核心线程最大数量.默认情况下,即使核心线程处于空闲状态也不会被销毁.除非通过allowCoreThreadTimeOut(true),则核心线程在空闲时间达到keepAliveTime时会被销毁<br>
    2. int maximumPoolSize:该线程池中线程最大数量<br>
    3. long keepAliveTime:该线程池中非核心线程被销毁前最大空闲时间,时间单位由unit决定.默认情况下核心线程即使空闲也不会被销毁,在调用allowCoreThreadTimeOut(true)后,该销毁时间设置也适用于核心线程<br>
    4. TimeUnit unit:keepAliveTime/被销毁前最大空闲时间的单位<br>
    5. BlockingQueue<Runnable> workQueue:该线程池中的任务队列.维护着等待被执行的Runnable对象.BlockingQueue有几种类型,下面会详述<br>
    6. ThreadFactory threadFactory:创建新线程的工厂.一般情况使用Executors.defaultThreadFactory()即可.当然也可以自定义.<br>
    7. RejectedExecutionHandler handler:拒绝策略.当需要创建的线程数量达到maximumPoolSize并且等待执行的Runnable数量超过了任务队列的容量,该如何处理.<br>
    
    3.2:当1个任务被放进线程池,ThreadPoolExecutor具体执行策略如下:<br>
    1. 如果线程数量没有达到corePoolSize,有核心线程空闲则核心线程直接执行,没有空闲则直接新建核心线程执行任务;
    2. 如果线程数量已经达到corePoolSize,且核心线程无空闲,则将任务添加到等待队列;
    3. 如果等待队列已满,则新建非核心线程执行该任务;
    4. 如果等待队列已满且总线程数量已达到maximumPoolSize,则会交由RejectedExecutionHandler handler处理.<br>
    
    3.3:阻塞队列/BlockingQueue<Runnable> workQueue<br>
    1. BlockingQueue有如下几种:SynchronousQueue/LinkedBlockingQueue/LinkedTransferQueue/ArrayBlockingQueue/PriorityBlockingQueue/DelayQueue.
    2. SynchronousQueue:SynchronousQueue的容量是0,不存储任何Runnable实例.新任务到来会直接尝试交给线程执行,如所有线程都在忙就创建新线程执行该任务.
    3. LinkedBlockingQueue:默认情况下没有容量限制的队列.
    4. ArrayBlockingQueue:一个有容量限制的队列.
    5. DelayQueue:一个没有容量限制的队列.队列中的元素必须实现了Delayed接口.元素在队列中的排序按照当前时间的延迟值,延迟最小/最早要被执行的任务排在队列头部,依次排序.延迟时间到达后执行指定任务.
    6. PriorityBlockingQueue:一个没有容量限制的队列.队列中元素必须实现了Comparable接口.队列中元素排序依赖元素的自然排序/compareTo的比较结果.
    7. 各种BlockingQueue的问题<br>
        1.SynchronousQueue缺点:因为不具备存储元素的能力,因而当任务很频繁时候,为了防止线程数量超标,我们往往设置maximumPoolSize是Integer.MAX_VALUE,创建过多线程会导致OOM.《阿里巴巴Java开发手册》中强调不能使用Executors直接创建线程池,就是对应Android源码中newCachedThreadPool和newScheduledThreadPool,本质上就是创建了maximumPoolSize为Integer.MAX_VALUE的ThreadPoolExecutor.<br>
        2.LinkedBlockingQueue因为没有容量限制,所以我们使用LinkedBlockingQueue创建ThreadPoolExecutor,设置maximumPoolSize是无意义的,如果线程数量已经达到corePoolSize,且核心线程都在忙,那么新来的任务会一直被添加到队列中.只要核心线程无空闲则一直得不到被执行机会.<br>
        3.DelayQueue和PriorityBlockingQueue也具有同样的问题.所以corePoolSize必须设置合理,否则会导致超出核心线程数量的任务一直得不到机会被执行.这两类队列分别适用于定时及优先级明确的任务.<br>
        
    3.4:RejectedExecutionHandler handler/拒绝策略有4种<br>
        1.hreadPoolExecutor.AbortPolicy:丢弃任务,并抛出RejectedExecutionException异常.ThreadPoolExecutor默认就是使用AbortPolicy.<br>
        2.ThreadPoolExecutor.DiscardPolicy:丢弃任务,但不会抛出异常.<br>
        3.ThreadPoolExecutor.DiscardOldestPolicy:丢弃排在队列头部的任务,不抛出异常,并尝试重新执行任务.<br>
        4.ThreadPoolExecutor.CallerRunsPolicy:丢弃任务,但不抛出异常,并将该任务交给调用此ThreadPoolExecutor的线程执行.
    
4. synchronized 的本质
    1. 保证synchronized方法或者代码块内部资源/数据的互斥访问
        - 即同一时间,由同一个Monitor监视的代码,最多只有1个线程在访问
    2. 保证线程之间对监视资源的数据同步.
        - 任何线程在获取Monitor后,会第一时间将共享内存中的数据复制到自己的缓存中;
        - 任何线程在释放Monitor后,会第一时间将缓存中的数据复制到共享内存中
5. volatile
    1. 保证被volatile修饰的成员的操作具有原子性和同步性.相当于简化版的synchronized
        - 原子性就是线程间互斥访问
        - 同步性就是线程之间对监视资源的数据同步
    2. volatile生效范围:基本类型的直接复制赋值 + 引用类型的直接赋值
        ```java
        //引用类型的直接赋值操作有效
        private volatile User u = U1;
        //修改引用类型的属性,则不是原子性的,volatile无效
        U1.name = "吊炸天"
        //对引用类型的直接赋值是原子性的
        u = U2;
        
        private volatile int a = 0;
        private int b = 100;
        //volatile无法实现++/--的原子性
        a++;
        ```
        1. volatile型变量自增操作的隐患
            - volatile类型变量每次在读取的时候，会越过线程的工作内存，直接从主存中读取，也就不会产生脏读
            - ++自增操作,在Java对应的汇编指令有三条 
                1. 从主存读取变量值到cpu寄存器 
                2. 寄存器里的值+1 
                3. 寄存器的值写回主存
            - 如果N个线程同时执行到了第1步,那么最终变量会损失(N-1).第二步第三步只有一个线程是执行成功.
        2. 对变量的写操作不依赖于当前值,才能用volatile修饰.
6. 针对num++这类复合类的操作,可以使用java并发包中的原子操作类原子操作类:AtomicInteger AtomicBoolean等来保证其原子性.
    ```java
    public static AtomicInteger num = new AtomicInteger(0);
    num.incrementAndGet();//原子性的num++,通过循环CAS方式
    ```
#### 2.线程间交互
1. 一个线程终结另一个线程
    1. Thread.stop不要用:
        - 因为线程在运行过程中随时有可能会被暂停切换到其他线程,stop的效果相当于切换到其他线程继续执行且以后再也不会切换回来.我们执行A.stop的时候,完全无法预知A的run方法已经执行了多少,执行百分比完全不可控.
        ```java
        下面的代码,每次执行最后打印的结果都不同,即我们完全不可预知调用stop时候当前线程执行了百分之多少.
        
        private static void t2(){
            Thread t = new Thread(){
                @Override
                public void run() {
                    for(int i=0;i<1000000;i++){
                        System.out.println(""+i);
                    }
                }
            };
            t.start();
            try {
                Thread.sleep(300);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            t.stop();
        }
        ```
    2. Thread.interrupt:仅仅设置当前线程为被中断状态.在运行的线程依然会继续运行.
        1. Thread.isInterrupted:获取当前线程是否被中断
        2. Thread.interrupted():如果线程A调用了Thread.interrupted()
            1. 如果A之前已经被中断,调用Thread.interrupted()返回false,A已经不是被中断状态
            2. 如果A之前不是被中断状态,调用Thread.interrupted()返回true,A变成被中断状态.
        3. 单纯调用A.interrupt是无效果的,interrupt需要和isInterrupted联合使用
            - 用于我们希望线程处于被中断状态时结束运行的场景.
            - interrupt和stop比较的优点:stop后,线程直接结束,我们完全无法控制当前执行到哪里;<br>
            interrupt后线程默认会继续执行,我们通过isInterrupted来获取被中断状态,只有被中断且满足我们指定条件才return,可以精确控制线程的执行百分比.
            ```java
            private static void t2(){
                Thread t = new Thread(){
                    @Override
                    public void run() {
                        for(int i=0;i<1000000;i++){
                            //检查线程是否处于中断状态,且检查是否满足指定条件
                            //如果不满足指定条件,即使处于中断状态也继续执行.
                            if(isInterrupted()&&i>800000){
                                //先做收尾工作
                                //return 结束
                                return;
                            }
                            System.out.println(""+i);
                        }
                    }
                };
                t.start();
                try {
                    Thread.sleep(100);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
                //调用了interrupt后,在run中监查是否已经被打断,如果已经被打断,且满足指定条件,
                //就return,线程就执行完了
                t.interrupt();
            }
            
            ......
            799999
            800000
            Process finished with exit code 0
            ```
        4. InterruptedException:
            1. 如果线程A在sleep过程中被其他线程调用A.interrupt(),会触发InterruptedException.
            2. 如果调用A.interrupt()时候,A并不在sleep状态,后面再调用A.sleep,也会立即抛出InterruptedException.
            ```java
            private static void t3(){
                Thread thread = new Thread(){
                    @Override
                    public void run() {
                        long t1 = System.currentTimeMillis();
                        try {
                            Thread.sleep(3000);
                        } catch (InterruptedException e) {
                            long t2 = System.currentTimeMillis();
                            System.out.println("老子被叫醒了:睡了"+(t2-t1)+"ms");
                            //用于做线程收尾工作,然后return
                            return;
                        }
                        System.out.println("AAAAAAAA");
                    }
                };
                thread.start();
                try {
                    Thread.sleep(500);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
                thread.interrupt();
            }
            
            老子被叫醒了:睡了493ms
            Process finished with exit code 0
            ```
2. 线程等待:wait,notifyAll,notify
    1. wait,notifyAll,notify是属于Object的方法.用于线程等待的场景,需用Monitor进行调用
    2. wait:
        - 当1个线程A持有Monitor M.
        - 此时调用M.wait,A会释放M并处于等待状态.并记录A在当前代码执行的位置Position.
    3. notify:
        - 当调用M.notify(),就会唤醒1个因为调用M.wait()而处于等待状态的线程
        - 如果有A,B,C--多个线程都是因为调用M.wait()而处于等待状态,不一定哪个会被唤醒并尝试获取M
    4. notifyAll:
        - 当调用M.notifyAll(),所有因为调用M.wait()而处于等待状态的线程都被唤醒,一起竞争尝试获取M
    5. 调用notify/notifyAll被唤醒并获取到M的线程A,会接着之前的代码执行位置Position继续执行下去
    ```java
    private String str = null;
    private synchronized void setStr(String str){
        System.out.println("setStr时间:"+System.currentTimeMillis());
        this.str = str;
        notifyAll();
    }
    private synchronized void printStr(){
        while (str==null){
            try {
                wait();
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }
        System.out.println("线程:"+Thread.currentThread().getName()+
        " printStr时间:"+System.currentTimeMillis());
        System.out.println("str:"+str);
    }
    private void t4(){
        (new Thread(){
            @Override
            public void run() {
                try {
                    Thread.sleep(2000);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
                setStr("老子设置一下");
            }
        }).start();
        (new Thread(){
            @Override
            public void run() {
                try {
                    Thread.sleep(1000);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
                System.out.println("线程:"+Thread.currentThread().getName()+
                " 尝试printStr时间:"+System.currentTimeMillis());
                printStr();
            }
        }).start();
        (new Thread(){
            @Override
            public void run() {
                try {
                    Thread.sleep(200);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
                System.out.println("线程:"+Thread.currentThread().getName()+
                " 尝试printStr时间:"+System.currentTimeMillis());
                printStr();
            }
        }).start();
    }
    
    线程:Thread-2 尝试printStr时间:1539247468146
    线程:Thread-1 尝试printStr时间:1539247468944
    setStr时间:1539247469944
    线程:Thread-1 printStr时间:1539247469944
    str:老子设置一下
    线程:Thread-2 printStr时间:1539247469944
    str:老子设置一下
    ```

#### 3.Executor、 AsyncTask、 HandlerThead、 IntentService 如何选择
1. HandlerThead就不要用,HandlerThead设计目的就是为了主界面死循环刷新界面,无其他应用场景.
2. 能用线程池就用线程池,因为最简单.
3. 涉及后台线程推送任务到UI线程,可以使用Handler或AsyncTask
4. Service:就是为了做后台任务,不要UI界面,需要持续存活.有复杂的需要长期存活/等待的场景使用Service.
5. IntentService:属于Service.当我们需要使用Service,且需要后台代码执行完毕后该Service自动被销毁,使用IntentService.

#### 4.AsyncTask的内存泄漏
0. GC Roots:由堆外指向堆内的引用,包括:
    1. Java方法栈帧中的局部变量
    2. 已加载类的静态变量
    3. native代码的引用
    4. 运行中的Java线程
1. AsyncTask内存泄漏本质:正在运行的线程/AsyncTask 在虚拟机中属于GC ROOTS,AsyncTask持有外部Activity的引用.被GC ROOTS引用的对象不能被回收.
2. 所以AsyncTask和其他线程工具一样,只要是使用线程,都有可能发生内存泄漏,都要及时关闭,AsyncTask并不比其他工具更差.
3. 如何避免AsyncTask内存泄漏:使用弱引用解决AsyncTask在Activity销毁后依然持有Activity引用的问题
    - [Android实现弱引用AsyncTask，将内存泄漏置之度外](https://blog.csdn.net/u013718120/article/details/53032986?utm_source=itdadao&utm_medium=referral)
![](https://user-gold-cdn.xitu.io/2018/10/14/16671cd7f2086873?w=1093&h=2296&f=png&s=578745)

#### 5.RxJava.
讲的太多了这里推荐1个专题[RxJava2.x](https://www.jianshu.com/c/299d0a51fdd4)<br>
下面记录一下自己不太熟的几点<br>
1. RxJava整体结构:
    1. 链的最上游:生产者Observable
    2. 链的最下游:观察者Observer
    3. 链的中间多个节点:双重角色.即是上一节点的观察者Observer,也是下一节点的生产者Observable.
2. Scheduler切换线程的原理:源码跟踪下去,实质是通过Excutor实现了线程切换.

#### 6.Android M对Profile GPU Rendering工具的更新
![](https://user-gold-cdn.xitu.io/2018/10/14/16671e449dcbcd00?w=407&h=566&f=jpeg&s=44982)
1. Swap Buffers:CPU等待GPU处理的时间
2. Command Issur:OpenGL渲染Display List所需要的时间
3. Sync&Upload:通常表示的是准备当前界面上有待绘制的图片所耗费的时间,为了减少该段区域的执行时间,我们可以减少屏幕上的图片数量或者是缩小图片本身的大小
4. Draw:测量绘制Display List的时间
5. Measure & Layout:这里表示的是布局的onMeasure与onLayout所花费的时间.一旦时间过长,就需要仔细检查自己的布局是不是存在严重的性能问题
6. Animation:表示的是计算执行动画所需要花费的时间.包含的动画有ObjectAnimator,ViewPropertyAnimator,Transition等等.一旦这里的执行时间过长,就需要检查是不是使用了非官方的动画工具或者是检查动画执行的过程中是不是触发了读写操作等等
7. Input Handling:表示的是系统处理输入事件所耗费的时间,粗略等于对于的事件处理方法所执行的时间.一旦执行时间过长,意味着在处理用户的输入事件的地方执行了复杂的操作
8. Misc/Vsync Delay:如果稍加注意,我们可以在开发应用的Log日志里面看到这样一行提示：I/Choreographer(691): Skipped XXX frames! The application may be doing too much work on its main thread。这意味着我们在主线程执行了太多的任务，导致UI渲染跟不上vSync的信号而出现掉帧的情况


## 9.Android性能优化典范-第6季
#### 1.启动闪屏
1. 当点击桌面图标启动APP的时候,App会出现短暂的白屏,一直到第一个Activity的页面的渲染加载完毕
2. 为了消除白屏,我们可以为App入口Activity单独设置theme.
    1. 在单独设置的theme中设置android:background属性为App的品牌宣传图片背景.
    2. 在代码执行到入口Activity的onCreate的时候设置为程序正常的主题.
    ```xml
    styles.xml
    <!-- Base application theme. -->
    //Activity默认主题
    <style name="AppTheme" parent="Theme.AppCompat.Light.DarkActionBar">
        <!-- Customize your theme here. -->
        //默认主题窗口背景设置为白色
        <item name="android:background">@android:color/white</item>
        <item name="colorPrimary">@color/colorPrimary</item>
        <item name="colorPrimaryDark">@color/colorPrimaryDark</item>
        <item name="colorAccent">@color/colorAccent</item>
        
        <item name="android:windowNoTitle">true</item>
        <item name="android:windowFullscreen">true</item>
        <item name="windowActionBar">false</item>
        <item name="windowNoTitle">true</item>
    </style>
    //入口Activity的theme单独设置
    <style name="ThemeSplash" parent="Theme.AppCompat.Light.NoActionBar">
        //入口Activity初始窗口背景设置为品牌宣传图片
        <item name="android:background">@mipmap/startbg</item>
        <item name="android:windowNoTitle">true</item>
        <item name="android:windowFullscreen">true</item>
        <item name="windowActionBar">false</item>
        <item name="windowNoTitle">true</item>
    </style>
    
    AndroidManifest.xml
    <application
        android:allowBackup="true"
        android:icon="@mipmap/ic_launcher"
        android:label="@string/app_name"
        android:roundIcon="@mipmap/ic_launcher_round"
        android:supportsRtl="true"
        android:theme="@style/AppTheme">
        <activity android:name=".MainActivity"
            android:theme="@style/ThemeSplash">//为入口Activity单独指定theme
            <intent-filter>
                <action android:name="android.intent.action.MAIN" />
                <category android:name="android.intent.category.LAUNCHER" />
            </intent-filter>
        </activity>
    </manifest>
    
    public class MainActivity extends AppCompatActivity {
        @Override
        protected void onCreate(Bundle savedInstanceState) {
            //在代码执行到入口Activity时候设置入口Activity为默认主题
            setTheme(R.style.AppTheme);
            super.onCreate(savedInstanceState);
            setContentView(R.layout.activity_main);
            tv_all = findViewById(R.id.tv_all);
            tv_local = findViewById(R.id.tv_local);
            //注册全局广播
            registerReceiver(globalReceiver,new IntentFilter("global"));
            //注册本地广播
            LocalBroadcastManager.getInstance(this).registerReceiver(localBroadReceiver,new IntentFilter("localBroadCast"));
        }
    }
    ```
#### 2.为App提供对应分辨率下的图片,系统会自动匹配最合适分辨率的图片执行拉伸或压缩的处理.
1. 如果是只有1张图片,放在mipmap-nodpi,或mipmap-xxxhdpi下
2. 所有的大背景图片,统一放在mipmap-nodpi目录,用一套1080P素材可以解决大部分手机适配问题,不用每个资源目录下放一套素材
3. 经过试验,不论ImageView宽高是否是wrap_content,只要图片所在文件夹和当前设备分辨率不匹配,都会涉及到放大或压缩,占用的内存都会相应的变化.尤其对于大图,放在低分辨率文件夹下直接OOM.
<br>具体原因:<br>
[郭霖:Android drawable微技巧，你所不知道的drawable的那些细节](https://blog.csdn.net/guolin_blog/article/details/50727753)
> 当我们使用资源id来去引用一张图片时，Android会使用一些规则来去帮我们匹配最适合的图片。什么叫最适合的图片？比如我的手机屏幕密度是xxhdpi，那么drawable-xxhdpi文件夹下的图片就是最适合的图片。因此，当我引用android_logo这张图时，如果drawable-xxhdpi文件夹下有这张图就会优先被使用，在这种情况下，图片是不会被缩放的。但是，如果drawable-xxhdpi文件夹下没有这张图时， 系统就会自动去其它文件夹下找这张图了，优先会去更高密度的文件夹下找这张图片，我们当前的场景就是drawable-xxxhdpi文件夹，然后发现这里也没有android_logo这张图，接下来会尝试再找更高密度的文件夹，发现没有更高密度的了，这个时候会去drawable-nodpi文件夹找这张图，发现也没有，那么就会去更低密度的文件夹下面找，依次是drawable-xhdpi -> drawable-hdpi -> drawable-mdpi -> drawable-ldpi。 
总体匹配规则就是这样，那么比如说现在终于在drawable-mdpi文件夹下面找到android_logo这张图了，但是系统会认为你这张图是专门为低密度的设备所设计的，如果直接将这张图在当前的高密度设备上使用就有可能会出现像素过低的情况，于是系统自动帮我们做了这样一个放大操作。
那么同样的道理，如果系统是在drawable-xxxhdpi文件夹下面找到这张图的话，它会认为这张图是为更高密度的设备所设计的，如果直接将这张图在当前设备上使用就有可能会出现像素过高的情况，于是会自动帮我们做一个缩小的操作

#### 3.尽量复用已经存在的图片.<br>
比如一张图片O已经存在,如果有View的背景就是O旋转过后的样子,可以直接用O创建RotateDrawable.然后将设置给View使用.<br>
注意:RotateDrawable已经重写了其onLevelChange方法,所以一定要设置level才会生效<br>
```java
@Override
protected boolean onLevelChange(int level) {
    super.onLevelChange(level);

    final float value = level / (float) MAX_LEVEL;
    final float degrees = MathUtils.lerp(mState.mFromDegrees, mState.mToDegrees, value);
    mState.mCurrentDegrees = degrees;

    invalidateSelf();
    return true;
}
```
实例:
```xml
1.首先创建xml文件
<?xml version="1.0" encoding="utf-8"?>
<rotate xmlns:android="http://schemas.android.com/apk/res/android"
    android:drawable="@mipmap/close10"
    android:fromDegrees="90"
    android:toDegrees="120"
    android:pivotX="50%"
    android:pivotY="50%"
    >
</rotate>

2.在Java代码中获取该xml对应的Drawable实例,并设置level为10000
Drawable drawable = getResources().getDrawable(R.drawable.rotate_close);
drawable.setLevel(10000);
3.将Drawable设置为View的背景
findViewById(R.id.v).setBackgroundDrawable(drawable);
```

#### 4.开启混淆和资源压缩:在app模块下的的build.gradle中
```
    buildTypes {
        release {
            minifyEnabled true
            shrinkResources true
            proguardFiles getDefaultProguardFile('proguard-android.txt'), 'proguard-rules.pro'
        }
    }
```

#### 5.对于简单/规则纹理的图片,使用VectorDrawable来替代多个分辨率图片.VectorDrawable 有很多注意事项,后面单独一篇文章总结

## 10.网络优化
网络优化主要有几个方面:降低网络请求数量,降低单次请求响应的数据量,在弱网环境下将非必要网络请求延缓至网络环境好的时候.
#### 1.降低网络请求数量:获取同样的数据,多次网络请求会增加电量消耗,且多次请求总体上将消耗服务端更多的时间及资源<br>
1. 接口Api设计要合理.可以将多个接口合并,多次请求显示1个界面,改造后1个接口即可提供完整数据.
2. 根据具体场景实时性需求,在App中加入网络缓存,在实时性有效区间避免重复请求:主要包括网络框架和图片加载框架的缓存.
#### 2.降低单次请求的数据量
1. 网络接口Api在设计时候,去除多余的请求参数及响应数据.
2. 网络请求及响应数据的传输开启GZIP压缩,降低传输数据量.
    - okHttp对gzip的支持前面已记录
3. Protocal Buffers,Nano-Proto-Buffers,FlatBuffers代替GSON执行序列化.
    - Protocal Buffers网上有使用的方法,相对GSON有点繁琐.如果对网络传输量很敏感,可以考虑使用.其他几种方案的文章不多.
4. 网络请求图片,添加图片宽高参数,避免下载过大图片增加流量消耗.
#### 3.弱网环境优化这块没有经验,直接看anly_jun的文章
[App优化之网络优化](http://blog.lmj.wiki/2016/10/06/app-opti/app_opt_network/)<br>
文章中提到:用户点赞操作, 可以直接给出界面的点赞成功的反馈, 使用JobScheduler在网络情况较好的时候打包请求.

## 11.电量优化
[anly_jun大神的:App优化之电池省着用](http://blog.lmj.wiki/2016/09/25/app-opti/app_opt_battery/)
<br>
## 12.JobScheduler,AlarmManager和WakeLock
JobScheduler在网络优化中出现过,WakeLock涉及电量优化,AlarmManager和WakeLock有相似,但侧重点不同.
1. WakeLock:比如一段关键逻辑T已经在执行,执行未完成Android系统就进入休眠,会导致T执行中断.WakeLock目的就在于阻止Android系统进入休眠状态,保证T得以继续执行.
    - 休眠过程中自定义的Timer、Handler、Thread、Service等都会暂停
2. AlarmManager:Android系统自带的定时器,可以将处于休眠状态的Android系统唤醒
    - 保证Android系统在休眠状态下被及时唤醒,执行 定时/延时/轮询任务
3. JobScheduler:JobScheduler目的在于将当下不紧急的任务延迟到后面更合适的某个时间来执行.我们可以控制这些任务在什么条件下被执行.
    - JobScheduler可以节约Android设备当下网络,电量,CPU等资源.在指定资源充裕情况下再执行"不紧要"的任务.


JobScheduler:<br>
[Android Jobscheduler使用](https://www.jianshu.com/p/9fb882cae239)<br>
[
Android开发笔记（一百四十三）任务调度JobScheduler](https://blog.csdn.net/aqi00/article/details/71638721)<br>
WakeLock:<br>
[Android WakeLock详解](https://blog.csdn.net/wzy_1988/article/details/46875343)<br>
[Android PowerManager.WakeLock使用小结](https://blog.csdn.net/zhoumushui/article/details/50389553)<br>
[Android的PowerManager和PowerManager.WakeLock用法简析](https://blog.csdn.net/qq_24531461/article/details/71155961)<br>
AlarmManager和WakeLock使用:<br>
[后台任务 - 保持设备唤醒状态](https://www.jianshu.com/p/5db15ce7de1e)


## 13.性能检测工具
#### 1.Android Studio 3.2之后,Android Device Monitor已经被移除.Android Device Monitor原先包含的工具由新的方案替代.[Android Device Monitor](https://developer.android.google.cn/studio/profile/monitor)
![](https://user-gold-cdn.xitu.io/2018/10/22/1669a29ca8b73e5e?w=1061&h=685&f=png&s=65296)
1. DDMS:由Android Profiler代替.可以进行CPU,内存,网络分析.
2. TraceView:可以通过Debug类在代码中调用Debug.startMethodTracing(String tracePath)和Debug.stopMethodTracing()来记录两者之间所有线程及线程中方法的耗时,生成.trace文件.通过abd命令可以将trace文件导出到电脑,通过CPU profiler分析.
3. Systrace:可以通过命令行生成html文件,通过Chrome浏览器进行分析.
    - 生成html文件已实现.但文件怎么分析暂未掌握,看了网上一些文章说实话还是没搞懂
4. Hierarchy Viewer:由Layout Inspector代替.但当前版本的Layout Inspector不能查看每个View具体的onMeasure,onLayout,onDraw耗时,功能是不足的.我们可使用系统提供的Window.OnFrameMetricsAvailableListener来计算指定View的onLayout及onDraw耗时.
5. Network Traffic tool:由Network Profiler代替.
#### 2.其中Android Profiler如何使用,直接看官网即可.[Profile your app performance](https://developer.android.google.cn/studio/profile/).
#### 3.TraceView
1. TraceView可以直接通过CPU profiler中点击Record按钮后,任意时间后点击Stop按钮.即可生成trace文件.并可将.trace文件导出.
![](https://user-gold-cdn.xitu.io/2018/10/22/1669a535d27b9ef2?w=960&h=515&f=png&s=135692)
![](https://user-gold-cdn.xitu.io/2018/10/22/1669a539a7137a18?w=1393&h=909&f=png&s=65791)
2. TraceView也可以通过Debug类在代码中精确控制要统计哪个区间代码/线程的CPU耗时.
*<br>这种用法是  anly_jun大神文章里学到的*
    ```java
    public class SampleApplication extends Application {
        @Override
        public void onCreate() {
            Debug.startMethodTracing("JetApp");
            super.onCreate();
            LeakCanary.install(this);
            // init logger.
            AppLog.init();
            // init crash helper
            CrashHelper.init(this);
            // init Push
            PushPlatform.init(this);
            // init Feedback
            FeedbackPlatform.init(this);
            Debug.stopMethodTracing();
        }
    ```
    代码执行完毕,会在Android设备中生成JetApp.trace文件.通过Device File Explorer,找到sdcard/Android/data/app包名/files/JetApp.trace<br>
    在JetApp.trace上点击右键->Copy Path,将trace文件路径复制下来.<br>
    Windows下cmd打开命令行,执行 adb pull 路径,即可trace文件导出到电脑.
    ![](https://user-gold-cdn.xitu.io/2018/10/22/1669a642569e89db?w=1048&h=61&f=png&s=5668)
3. trace文件分析很简单.我们可以看到每个线程及线程中每个方法调用消耗的时间.
#### 4.Layout Inspector很简单,在App运行后,点击Tools->Layout Inspector即可.
**下面只看Window.OnFrameMetricsAvailableListener怎么用.**
> 从Android 7.0 (API level 24)开始,Android引入Window.OnFrameMetricsAvailableList接口用于提供每一帧绘制各阶段的耗时,数据源与GPU Profile相同.
```java
public interface OnFrameMetricsAvailableListener {
    void onFrameMetricsAvailable(Window window, FrameMetrics frameMetrics,int dropCountSinceLastInvocation);
}
/**
 * 包含1帧的周期内,渲染系统各个方法的耗时数据.
 */
public final class FrameMetrics {
    ****
    //通过getMetric获取layout/measure耗时所用的id
    public static final int LAYOUT_MEASURE_DURATION = 3;
    public static final int DRAW_DURATION = 4;
    /**
    * 获取当前帧指定id代表的方法/过程的耗时,单位是纳秒:1纳秒(ns)=10的负6次方毫秒(ms)
    */
    public long getMetric(@Metric int id) {
        ****
    }
}
```
1. 在Activity中使用OnFrameMetricsAvailableListener:
    - 通过调用this.getWindow().addOnFrameMetricsAvailableListener(@NonNull OnFrameMetricsAvailableListener listener,Handler handler)来添加监听.
    - 通过调用this.getWindow().removeOnFrameMetricsAvailableListener(OnFrameMetricsAvailableListener listener)取消监听.
    - [解析ConstraintLayout的性能优势](https://mp.weixin.qq.com/s/gGR2itbY7hh9fo61SxaMQQ?)中引用了Google使用OnFrameMetricsAvailableListener的例子[android-constraint-layout-performance](https://github.com/googlesamples/android-constraint-layout-performance).其中Activity是Kotlin写的,尝试将代码转为java,解决掉报错后运行.
        ```java
        package p1.com.p1;
        
        import android.os.AsyncTask;
        import android.os.Bundle;
        import android.os.Handler;
        import android.support.annotation.RequiresApi;
        import android.support.v7.app.AppCompatActivity;
        import android.util.Log;
        import android.view.FrameMetrics;
        import android.view.View;
        import android.view.View.MeasureSpec;
        import android.view.View.OnClickListener;
        import android.view.ViewGroup;
        import android.view.Window;
        import android.view.Window.OnFrameMetricsAvailableListener;
        import android.widget.Button;
        import android.widget.TextView;
        import org.jetbrains.annotations.NotNull;
        import org.jetbrains.annotations.Nullable;
        import java.lang.ref.WeakReference;
        import java.util.Arrays;
        import kotlin.TypeCastException;
        import kotlin.jvm.internal.Intrinsics;
        
        public final class KtMainActivity extends AppCompatActivity {
            private final Handler frameMetricsHandler = new Handler();
            @RequiresApi(24)
            private final OnFrameMetricsAvailableListener frameMetricsAvailableListener = new OnFrameMetricsAvailableListener() {
                @Override
                public void onFrameMetricsAvailable(Window window, FrameMetrics frameMetrics, int dropCountSinceLastInvocation) {
                    long costDuration = frameMetrics.getMetric(FrameMetrics.LAYOUT_MEASURE_DURATION);
                    Log.d("Jet", "layoutMeasureDurationNs: " + costDuration);
                }
            };
            private static final String TAG = "KtMainActivity";
            private static final int TOTAL = 100;
            private static final int WIDTH = 1920;
            private static final int HEIGHT = 1080;
        
            @RequiresApi(3)
            protected void onCreate(@Nullable Bundle savedInstanceState) {
                super.onCreate(savedInstanceState);
                this.setContentView(R.layout.activity_for_test);
                final Button traditionalCalcButton = (Button) this.findViewById(R.id.button_start_calc_traditional);
                final Button constraintCalcButton = (Button) this.findViewById(R.id.button_start_calc_constraint);
                final TextView textViewFinish = (TextView) this.findViewById(R.id.textview_finish);
                traditionalCalcButton.setOnClickListener((OnClickListener) (new OnClickListener() {
                    public final void onClick(View it) {
                        Button var10000 = constraintCalcButton;
                        Intrinsics.checkExpressionValueIsNotNull(constraintCalcButton, "constraintCalcButton");
                        var10000.setVisibility(View.INVISIBLE);
                        View var4 = KtMainActivity.this.getLayoutInflater().inflate(R.layout.activity_traditional, (ViewGroup) null);
                        if (var4 == null) {
                            throw new TypeCastException("null cannot be cast to non-null type android.view.ViewGroup");
                        } else {
                            ViewGroup container = (ViewGroup) var4;
                            String var10002 = KtMainActivity.this.getString(R.string.executing_nth_iteration);
                            Intrinsics.checkExpressionValueIsNotNull(var10002, "getString(R.string.executing_nth_iteration)");
                            KtMainActivity.MeasureLayoutAsyncTask asyncTask = new KtMainActivity.MeasureLayoutAsyncTask(var10002, new WeakReference(traditionalCalcButton), new WeakReference(textViewFinish), new WeakReference(container));
                            asyncTask.execute(new Void[0]);
                        }
                    }
                }));
                constraintCalcButton.setOnClickListener((OnClickListener) (new OnClickListener() {
                    public final void onClick(View it) {
                        Button var10000 = traditionalCalcButton;
                        Intrinsics.checkExpressionValueIsNotNull(traditionalCalcButton, "traditionalCalcButton");
                        var10000.setVisibility(View.INVISIBLE);
                        View var4 = KtMainActivity.this.getLayoutInflater().inflate(R.layout.activity_constraintlayout, (ViewGroup) null);
                        if (var4 == null) {
                            throw new TypeCastException("null cannot be cast to non-null type android.view.ViewGroup");
                        } else {
                            ViewGroup container = (ViewGroup) var4;
                            String var10002 = KtMainActivity.this.getString(R.string.executing_nth_iteration);
                            Intrinsics.checkExpressionValueIsNotNull(var10002, "getString(R.string.executing_nth_iteration)");
                            KtMainActivity.MeasureLayoutAsyncTask asyncTask = new KtMainActivity.MeasureLayoutAsyncTask(var10002, new WeakReference(constraintCalcButton), new WeakReference(textViewFinish), new WeakReference(container));
                            asyncTask.execute(new Void[0]);
                        }
                    }
                }));
            }
            @RequiresApi(24)
            protected void onResume() {
                super.onResume();
                this.getWindow().addOnFrameMetricsAvailableListener(this.frameMetricsAvailableListener, this.frameMetricsHandler);
            }
            @RequiresApi(24)
            protected void onPause() {
                super.onPause();
                this.getWindow().removeOnFrameMetricsAvailableListener(this.frameMetricsAvailableListener);
            }
            @RequiresApi(3)
            private static final class MeasureLayoutAsyncTask extends AsyncTask {
                @NotNull
                private final String executingNthIteration;
                @NotNull
                private final WeakReference startButtonRef;
                @NotNull
                private final WeakReference finishTextViewRef;
                @NotNull
                private final WeakReference containerRef;
        
                @Nullable
                protected Void doInBackground(@NotNull Void... voids) {
                    Intrinsics.checkParameterIsNotNull(voids, "voids");
                    int i = 0;
        
                    for (int var3 = KtMainActivity.TOTAL; i < var3; ++i) {
                        this.publishProgress(new Integer[]{i});
        
                        try {
                            Thread.sleep(100L);
                        } catch (InterruptedException var5) {
                            ;
                        }
                    }
                    return null;
                }
                // $FF: synthetic method
                // $FF: bridge method
                public Object doInBackground(Object[] var1) {
                    return this.doInBackground((Void[]) var1);
                }
                protected void onProgressUpdate(@NotNull Integer... values) {
                    Intrinsics.checkParameterIsNotNull(values, "values");
                    Button var10000 = (Button) this.startButtonRef.get();
                    if (var10000 != null) {
                        Button startButton = var10000;
                        Intrinsics.checkExpressionValueIsNotNull(startButton, "startButton");
        //                StringCompanionObject var3 = StringCompanionObject.INSTANCE;
                        String var4 = this.executingNthIteration;
                        Object[] var5 = new Object[]{values[0], KtMainActivity.TOTAL};
                        String var9 = String.format(var4, Arrays.copyOf(var5, var5.length));
                        Intrinsics.checkExpressionValueIsNotNull(var9, "java.lang.String.format(format, *args)");
                        String var7 = var9;
                        startButton.setText((CharSequence) var7);
                        ViewGroup var10 = (ViewGroup) this.containerRef.get();
                        if (var10 != null) {
                            ViewGroup container = var10;
                            Intrinsics.checkExpressionValueIsNotNull(container, "container");
                            this.measureAndLayoutExactLength(container);
                            this.measureAndLayoutWrapLength(container);
                        }
                    }
                }
                // $FF: synthetic method
                // $FF: bridge method
                public void onProgressUpdate(Object[] var1) {
                    this.onProgressUpdate((Integer[]) var1);
                }
                protected void onPostExecute(@Nullable Void aVoid) {
                    TextView var10000 = (TextView) this.finishTextViewRef.get();
                    if (var10000 != null) {
                        TextView finishTextView = var10000;
                        Intrinsics.checkExpressionValueIsNotNull(finishTextView, "finishTextView");
                        finishTextView.setVisibility(View.VISIBLE);
                        Button var4 = (Button) this.startButtonRef.get();
                        if (var4 != null) {
                            Button startButton = var4;
                            Intrinsics.checkExpressionValueIsNotNull(startButton, "startButton");
                            startButton.setVisibility(View.GONE);
                        }
                    }
                }
                // $FF: synthetic method
                // $FF: bridge method
                public void onPostExecute(Object var1) {
                    this.onPostExecute((Void) var1);
                }
                private final void measureAndLayoutWrapLength(ViewGroup container) {
                    int widthMeasureSpec = MeasureSpec.makeMeasureSpec(KtMainActivity.WIDTH, View.MeasureSpec.AT_MOST);
                    int heightMeasureSpec = MeasureSpec.makeMeasureSpec(KtMainActivity.HEIGHT, View.MeasureSpec.AT_MOST);
                    container.measure(widthMeasureSpec, heightMeasureSpec);
                    container.layout(0, 0, container.getMeasuredWidth(), container.getMeasuredHeight());
                }
        
                private final void measureAndLayoutExactLength(ViewGroup container) {
                    int widthMeasureSpec = MeasureSpec.makeMeasureSpec(KtMainActivity.WIDTH, View.MeasureSpec.EXACTLY);
                    int heightMeasureSpec = MeasureSpec.makeMeasureSpec(KtMainActivity.HEIGHT, View.MeasureSpec.EXACTLY);
                    container.measure(widthMeasureSpec, heightMeasureSpec);
                    container.layout(0, 0, container.getMeasuredWidth(), container.getMeasuredHeight());
                }
        
                @NotNull
                public final String getExecutingNthIteration() {
                    return this.executingNthIteration;
                }
        
                @NotNull
                public final WeakReference getStartButtonRef() {
                    return this.startButtonRef;
                }
        
                @NotNull
                public final WeakReference getFinishTextViewRef() {
                    return this.finishTextViewRef;
                }
        
                @NotNull
                public final WeakReference getContainerRef() {
                    return this.containerRef;
                }
        
                public MeasureLayoutAsyncTask(@NotNull String executingNthIteration, @NotNull WeakReference startButtonRef, @NotNull WeakReference finishTextViewRef, @NotNull WeakReference containerRef) {
                    super();
                    Intrinsics.checkParameterIsNotNull(executingNthIteration, "executingNthIteration");
                    Intrinsics.checkParameterIsNotNull(startButtonRef, "startButtonRef");
                    Intrinsics.checkParameterIsNotNull(finishTextViewRef, "finishTextViewRef");
                    Intrinsics.checkParameterIsNotNull(containerRef, "containerRef");
                    this.executingNthIteration = executingNthIteration;
                    this.startButtonRef = startButtonRef;
                    this.finishTextViewRef = finishTextViewRef;
                    this.containerRef = containerRef;
                }
            }
        }
        
        D/Jet: layoutMeasureDurationNs: 267344
        D/Jet: layoutMeasureDurationNs: 47708
        D/Jet: layoutMeasureDurationNs: 647240
        D/Jet: layoutMeasureDurationNs: 59636
        D/Jet: layoutMeasureDurationNs: 50052
        D/Jet: layoutMeasureDurationNs: 49739
        D/Jet: layoutMeasureDurationNs: 75990
        D/Jet: layoutMeasureDurationNs: 296198
        D/Jet: layoutMeasureDurationNs: 0
        D/Jet: layoutMeasureDurationNs: 0
        D/Jet: layoutMeasureDurationNs: 0
        D/Jet: layoutMeasureDurationNs: 0
        D/Jet: layoutMeasureDurationNs: 0
        D/Jet: layoutMeasureDurationNs: 0
        D/Jet: layoutMeasureDurationNs: 0
        D/Jet: layoutMeasureDurationNs: 0
        D/Jet: layoutMeasureDurationNs: 0
        D/Jet: layoutMeasureDurationNs: 894375
        D/Jet: layoutMeasureDurationNs: 0
        D/Jet: layoutMeasureDurationNs: 0
        D/Jet: layoutMeasureDurationNs: 0
        D/Jet: layoutMeasureDurationNs: 0
        D/Jet: layoutMeasureDurationNs: 0
        D/Jet: layoutMeasureDurationNs: 0
        D/Jet: layoutMeasureDurationNs: 1248021
        D/Jet: layoutMeasureDurationNs: 0
        D/Jet: layoutMeasureDurationNs: 0
        D/Jet: layoutMeasureDurationNs: 0
        D/Jet: layoutMeasureDurationNs: 0
        D/Jet: layoutMeasureDurationNs: 0
        D/Jet: layoutMeasureDurationNs: 0
        D/Jet: layoutMeasureDurationNs: 0
        D/Jet: layoutMeasureDurationNs: 0
        D/Jet: layoutMeasureDurationNs: 0
        D/Jet: layoutMeasureDurationNs: 0
        D/Jet: layoutMeasureDurationNs: 0
        D/Jet: layoutMeasureDurationNs: 0
        D/Jet: layoutMeasureDurationNs: 0
        D/Jet: layoutMeasureDurationNs: 0
        D/Jet: layoutMeasureDurationNs: 0
        D/Jet: layoutMeasureDurationNs: 0
        D/Jet: layoutMeasureDurationNs: 0
        D/Jet: layoutMeasureDurationNs: 0
        D/Jet: layoutMeasureDurationNs: 0
        D/Jet: layoutMeasureDurationNs: 0
        D/Jet: layoutMeasureDurationNs: 0
        D/Jet: layoutMeasureDurationNs: 0
        D/Jet: layoutMeasureDurationNs: 0
        D/Jet: layoutMeasureDurationNs: 0
        D/Jet: layoutMeasureDurationNs: 0
        D/Jet: layoutMeasureDurationNs: 1290677
        D/Jet: layoutMeasureDurationNs: 2936563
        D/Jet: layoutMeasureDurationNs: 1387188
        D/Jet: layoutMeasureDurationNs: 2325521
        D/Jet: layoutMeasureDurationNs: 1940052
        D/Jet: layoutMeasureDurationNs: 1539271
        D/Jet: layoutMeasureDurationNs: 803750
        D/Jet: layoutMeasureDurationNs: 1405000
        D/Jet: layoutMeasureDurationNs: 1188437
        D/Jet: layoutMeasureDurationNs: 1748802
        D/Jet: layoutMeasureDurationNs: 3422240
        D/Jet: layoutMeasureDurationNs: 1400677
        D/Jet: layoutMeasureDurationNs: 2416094
        D/Jet: layoutMeasureDurationNs: 1532864
        D/Jet: layoutMeasureDurationNs: 1684063
        D/Jet: layoutMeasureDurationNs: 1092865
        D/Jet: layoutMeasureDurationNs: 1363177
        D/Jet: layoutMeasureDurationNs: 1067188
        D/Jet: layoutMeasureDurationNs: 1358333
        D/Jet: layoutMeasureDurationNs: 2999895
        D/Jet: layoutMeasureDurationNs: 2113021
        D/Jet: layoutMeasureDurationNs: 1957395
        D/Jet: layoutMeasureDurationNs: 1319740
        D/Jet: layoutMeasureDurationNs: 2207239
        D/Jet: layoutMeasureDurationNs: 1514167
        D/Jet: layoutMeasureDurationNs: 949114
        D/Jet: layoutMeasureDurationNs: 1691250
        D/Jet: layoutMeasureDurationNs: 1387448
        D/Jet: layoutMeasureDurationNs: 932552
        D/Jet: layoutMeasureDurationNs: 1223802
        D/Jet: layoutMeasureDurationNs: 2024740
        D/Jet: layoutMeasureDurationNs: 1242292
        D/Jet: layoutMeasureDurationNs: 2228230
        D/Jet: layoutMeasureDurationNs: 1382083
        D/Jet: layoutMeasureDurationNs: 2233282
        D/Jet: layoutMeasureDurationNs: 1907187
        D/Jet: layoutMeasureDurationNs: 2287552
        D/Jet: layoutMeasureDurationNs: 776354
        D/Jet: layoutMeasureDurationNs: 1225000
        D/Jet: layoutMeasureDurationNs: 875417
        D/Jet: layoutMeasureDurationNs: 1271302
        D/Jet: layoutMeasureDurationNs: 1211614
        D/Jet: layoutMeasureDurationNs: 1346459
        D/Jet: layoutMeasureDurationNs: 1978854
        D/Jet: layoutMeasureDurationNs: 2915677
        D/Jet: layoutMeasureDurationNs: 1330573
        D/Jet: layoutMeasureDurationNs: 2195364
        D/Jet: layoutMeasureDurationNs: 775208
        D/Jet: layoutMeasureDurationNs: 2492292
        D/Jet: layoutMeasureDurationNs: 400104
        D/Jet: layoutMeasureDurationNs: 2844375
        D/Jet: layoutMeasureDurationNs: 1563750
        D/Jet: layoutMeasureDurationNs: 3689531
        D/Jet: layoutMeasureDurationNs: 2019323
        D/Jet: layoutMeasureDurationNs: 1663906
        D/Jet: layoutMeasureDurationNs: 1004531
        D/Jet: layoutMeasureDurationNs: 738125
        D/Jet: layoutMeasureDurationNs: 1299166
        D/Jet: layoutMeasureDurationNs: 1223854
        D/Jet: layoutMeasureDurationNs: 1942240
        D/Jet: layoutMeasureDurationNs: 1392396
        D/Jet: layoutMeasureDurationNs: 1906458
        D/Jet: layoutMeasureDurationNs: 691198
        D/Jet: layoutMeasureDurationNs: 2620468
        D/Jet: layoutMeasureDurationNs: 1953229
        D/Jet: layoutMeasureDurationNs: 1120365
        D/Jet: layoutMeasureDurationNs: 3165417
        D/Jet: layoutMeasureDurationNs: 537709
        D/Jet: layoutMeasureDurationNs: 3019531
        D/Jet: layoutMeasureDurationNs: 706250
        D/Jet: layoutMeasureDurationNs: 1129115
        D/Jet: layoutMeasureDurationNs: 539427
        D/Jet: layoutMeasureDurationNs: 1633438
        D/Jet: layoutMeasureDurationNs: 1784479
        D/Jet: layoutMeasureDurationNs: 743229
        D/Jet: layoutMeasureDurationNs: 1851615
        D/Jet: layoutMeasureDurationNs: 851927
        D/Jet: layoutMeasureDurationNs: 1847916
        D/Jet: layoutMeasureDurationNs: 836718
        D/Jet: layoutMeasureDurationNs: 2892552
        D/Jet: layoutMeasureDurationNs: 1230573
        D/Jet: layoutMeasureDurationNs: 3886563
        D/Jet: layoutMeasureDurationNs: 2138281
        D/Jet: layoutMeasureDurationNs: 2198021
        D/Jet: layoutMeasureDurationNs: 1805885
        D/Jet: layoutMeasureDurationNs: 2316927
        D/Jet: layoutMeasureDurationNs: 1990937
        D/Jet: layoutMeasureDurationNs: 2261041
        D/Jet: layoutMeasureDurationNs: 2159010
        D/Jet: layoutMeasureDurationNs: 666562
        D/Jet: layoutMeasureDurationNs: 2332031
        D/Jet: layoutMeasureDurationNs: 1061875
        D/Jet: layoutMeasureDurationNs: 1879062
        D/Jet: layoutMeasureDurationNs: 1411459
        D/Jet: layoutMeasureDurationNs: 154635
        ```
2. 在Application中使用OnFrameMetricsAvailableListener,则可以统一设置,不需要每个Activity单独设置,推荐使用开源项目[ActivityFrameMetrics](https://github.com/frogermcs/ActivityFrameMetrics)
    1. 在Application的onCreate中设置单帧渲染总时间超过W毫秒和E毫秒,会在Logcat中打印警告和错误的Log信息.
        ```java
        public class SampleApplication extends Application {
            @Override
            public void onCreate() {
                registerActivityLifecycleCallbacks(new ActivityFrameMetrics.Builder()
                        .warningLevelMs(10)     //default: 17ms
                        .errorLevelMs(10)       //default: 34ms
                        .showWarnings(true)     //default: true
                        .showErrors(true)       //default: true
                        .build());
            }
        }
        ```
    2. Application设置完成运行App,出现单帧渲染总耗时超过指定时间,即可看到Logcat中的信息.
        ```
        E/FrameMetrics: Janky frame detected on KtMainActivity with total duration: 16.91ms
        Layout/measure: 1.66ms, draw:2.51ms, gpuCommand:3.13ms others:9.61ms
        Janky frames: 72/107(67.28972%)
        E/FrameMetrics: Janky frame detected on KtMainActivity with total duration: 15.47ms
        Layout/measure: 1.00ms, draw:2.05ms, gpuCommand:3.44ms others:8.98ms
        Janky frames: 73/108(67.59259%)
        E/FrameMetrics: Janky frame detected on KtMainActivity with total duration: 15.09ms
        Layout/measure: 1.30ms, draw:1.44ms, gpuCommand:2.91ms others:9.44ms
        Janky frames: 74/110(67.27273%)
        ****
        ```

#### 5.Systrace:通过命令行生成html文件,通过Chrome浏览器进行分析
1. 首先电脑要安装python,这里有几个坑:<br>
    1. Python要安装2.7x版本,不能安装最新的3.x.
        - 比如自己电脑中systrace文件夹路径是:C:\Users\你的用户名\AppData\Local\Android\Sdk\platform-tools\systrace,如果我们安装的是3.x版本,在这个路径下执行python systrace.py *** 命令会报错,提示你应该安装2.7
    2. Python安装时候,要记得勾选"Add python.exe to Path".
    3. 这时候直接执行python systrace.py ***命令还是会报错:ImportError: No module named win32com
        - [下载win32com安装](https://blog.csdn.net/netwalk/article/details/41243535)
2. [生成html及如何分析](https://blog.csdn.net/kitty_landon/article/details/79192377)
