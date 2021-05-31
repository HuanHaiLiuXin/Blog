### 待看文章
1. [【译】一文带你了解Android中23个关于Canvas绘制的方法](https://www.jianshu.com/p/af99ef0618d5)
2. [Android 多点触控及应用（画板控件 DrawView）](https://www.jianshu.com/p/c8cc619f69db)
3. [android双缓冲绘图技术分析](https://www.jianshu.com/p/efc0bebfd22e)
4. [Android拆轮子-唯美细腻的夕阳海浪动画](https://mp.weixin.qq.com/s?__biz=MzAxMTI4MTkwNQ==&mid=2650821393&idx=1&sn=4106295c865daa0aadaebd2936a61a37&chksm=80b7878fb7c00e9972d50b6e47c125725b5ff6792eedb610bc777aeb895ae844247404cda8be&scene=4#wechat_redirect)
5. [Learn_Depth](https://github.com/ImmortalZ/Learn_Depth)

### 绘制
##### 1.Camera.setLocation(x, y, z) 设置虚拟相机的位置
1. x,y,z的单位是英寸/inch
2. inch和px的换算是固定的: 1英寸=72像素
3. Camera默认设置是 0, 0, -8
    ```java
    /**
     * Sets the location of the camera. The default location is set at
     * 0, 0, -8.
     * 
     * @param x The x location of the camera
     * @param y The y location of the camera
     * @param z The z location of the camera
     */
    public native void setLocation(float x, float y, float z);
    ```
4. 如何保证不同分辨率的手机上,Camera投影/3维旋转的效果接近:2种思路
	1. 计算View的宽度/高度的px值,camera垂直高度为宽高的指定倍数,得到其像素值,进而推算出z = viewSize * fraction / 72 .
    	- 这样操作,即使是同一设备,不同大小的View,Camera的高度都不同,可以保证三维投影的缩放比例一致
    2. 不同设备上,dp和pix的比例是不同的.假设View的宽高使用dp作为单位,保证所有设备上Camera都是'同样高度': float z = -16 * getResources().getDisplayMetrics().density
        ```java
        float z = -16 * getResources().getDisplayMetrics().density;
        camera.setLocation(0,0,z);
        ```
##### 2.paint.setXfermode 及 硬件加速
- [扔物线 硬件加速](https://hencoder.com/ui-1-8/)
1. 在透明的地方才能做paint.setXfermode.
2. 如何创建1块透明的'地方':使用离屏缓冲.
3. ObjectAnimatorActivity.java

##### 3.Bitmap及Drawable,自定义Drawable
- BitmapDrawableMaterialEditTextActivity.java
1. Drawable转Bitmap
	![](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/861cbbcd21434fb3a698b4d8bda015bb~tplv-k3u1fbpfcp-watermark.image)

##### 4.


### 非事件分发
##### 1. 父View对子View的尺寸限制: 父View将**开发者对子View的要求**进行处理计算后所得到的的更精确的要求.[HenCoder Android UI 2-2 全新定制 View 的尺寸](https://www.youtube.com/watch?v=aOb4Hvqbeu4&feature=youtu.be)
##### 2. 开发者对子View的要求: xml文件中子View中以 **layout开头的属性**.
##### 3. View在XML布局文件中的以**layout开头的属性**,不是给View自己看的,是给该View的父View看的.
    ```
    <LinearLayout
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:orientation="vertical">
        <androidx.appcompat.widget.AppCompatImageView
            android:layout_width="100dp"
            android:layout_height="100dp"
            android:layout_marginTop="100dp"
            android:layout_marginBottom="100dp"
            android:src="@drawable/head"
            />
    ```
    AppCompatImageView的layout_width, layout_height, layout_marginTop, layout_marginBottom 等以 layout开头的属性,是给LinearLayout看的.LinearLayout在执行自己的onMeasure时候,会将以上属性生成 int widthMeasureSpec, int heightMeasureSpec 两个值,作为 **父View对AppCompatImageView尺寸的限制** ,在调用AppCompatImageView的 **measure** 时候作为参数传入.最终作为AppCompatImageView的 **onMeasure** 的属性传入.
##### 4. 全新自定义View尺寸:
1. 在onMeasure中计算尺寸
2. 对计算得到的 width 和 height, 调用 resolveSize , 以使计算得到的尺寸满足 父View的限制.
3. setMeasuredDimension
    ```
    @Override
    protected void onMeasure(int widthMeasureSpec, int heightMeasureSpec) {
        //1:selfExpectWidth , selfExceptHeight :自定义View计算自身期望的尺寸
        int selfExpectWidth = 200;
        int selfExceptHeight = 200;
        //2:调用 resolveSize , 对自定义View自身期望的尺寸 进行调整,以符合 '父 View 的限制'
        selfExpectWidth = resolveSize(selfExpectWidth, widthMeasureSpec);
        selfExceptHeight = resolveSize(selfExpectWidth, heightMeasureSpec);
        //3:调用setMeasuredDimension保存尺寸
        setMeasuredDimension(selfExpectWidth, selfExceptHeight);
    }
    ```
4. 代码实例
    ```
    public class Layout2View extends View {
        ***
        @Override
        protected void onMeasure(int widthMeasureSpec, int heightMeasureSpec) {
            //1:selfExpectWidth , selfExceptHeight :自定义View计算自身期望的尺寸
            int selfExpectWidth = 200;
            int selfExceptHeight = 200;
            //2:调用 resolveSize , 对自定义View自身期望的尺寸 进行调整,以符合 '父 View 的限制'
            selfExpectWidth = resolveSize(selfExpectWidth, widthMeasureSpec);
            selfExceptHeight = resolveSize(selfExceptHeight, heightMeasureSpec);
            //3:调用setMeasuredDimension保存尺寸
            setMeasuredDimension(selfExpectWidth, selfExceptHeight);

            //log打印mode，size
            int specMode = MeasureSpec.getMode(widthMeasureSpec);
            int specSize = MeasureSpec.getSize(widthMeasureSpec);
            switch (specMode) {
                case MeasureSpec.AT_MOST:
                    L.d("Layout2View onMeasure: MeasureSpec.AT_MOST .specSize:"+specSize);
                    break;
                case MeasureSpec.EXACTLY:
                    L.d("Layout2View onMeasure: MeasureSpec.EXACTLY .specSize:"+specSize);
                    break;
                case MeasureSpec.UNSPECIFIED:
                    L.d("Layout2View onMeasure: MeasureSpec.UNSPECIFIED .specSize:"+specSize);
                    break;
                default:
                    L.d("Layout2View onMeasure: default .specSize:"+specSize);
                    break;
            }
        }
        @Override
        protected void onDraw(Canvas canvas) {
            int width = getMeasuredWidth();
            int height = getMeasuredHeight();
            float radius = Math.min(width, height) / 2;
            paint.setColor(Color.RED);
            canvas.drawCircle(width / 2, height / 2, radius, paint);
            //将View实际尺寸显示出来
            paint.setColor(Color.BLACK);
            paint.setTextSize(30);
            Paint.FontMetrics fontMetrics = paint.getFontMetrics();
            float baseline = (height - (fontMetrics.bottom - fontMetrics.top)) / 2 - fontMetrics.top;
            paint.setTextAlign(Paint.Align.CENTER);
            canvas.drawText("View实际尺寸: " + width + "*" + height, width / 2, baseline, paint);
        }
    }

    <LinearLayout
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:orientation="vertical">
        <Layout2View
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:layout_marginBottom="20dp" />
        <Layout2View
            android:layout_width="400px"
            android:layout_height="500px"
            android:layout_marginBottom="20dp" />
        <Layout2View
            android:layout_width="match_parent"
            android:layout_height="match_parent"
            android:layout_marginBottom="20dp" />
    </LinearLayout>

    android:layout_width="wrap_content" :
    Jet2020: Layout2View onMeasure: MeasureSpec.AT_MOST .specSize:1080

    android:layout_width="400px" :
    Jet2020: Layout2View onMeasure: MeasureSpec.EXACTLY .specSize:400

    android:layout_width="match_parent" :
    Jet2020: Layout2View onMeasure: MeasureSpec.EXACTLY .specSize:1080
    ```
5. 由上面log打印可见:
    - 当View的宽/高为 wrap_content: 其 widthMeasureSpec/heightMeasureSpec 对应的 mode 是 MeasureSpec.AT_MOST; size是父View的宽/高.
    - match_parent : mode 是 MeasureSpec.EXACTLY; size是父View的宽/高.
    - 精确尺寸在,例如400px : mode 是 MeasureSpec.EXACTLY; size是定义的精确尺寸.
		![](//p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/d9b55afe7dc64d5ca6213b052498205c~tplv-k3u1fbpfcp-zoom-1.image)

##### 5. View.resolveSize
```
public static int resolveSize(int size, int measureSpec) {
    return resolveSizeAndState(size, measureSpec, 0) & MEASURED_SIZE_MASK;
}
public static int resolveSizeAndState(int size, int measureSpec, int childMeasuredState) {
    final int specMode = MeasureSpec.getMode(measureSpec);
    final int specSize = MeasureSpec.getSize(measureSpec);
    final int result;
    switch (specMode) {
        //wrap_content 对应着 MeasureSpec.AT_MOST
        //size对应着父View的宽/高
        case MeasureSpec.AT_MOST:
            if (specSize < size) {
                result = specSize | MEASURED_STATE_TOO_SMALL;
            } else {
                result = size;
            }
            break;
        //match_parent 和 精确尺寸/400px 对应着 MeasureSpec.EXACTLY
        //match_parent 的size对应着父View的宽/高
        //精确尺寸, size就是定义的精确值
        case MeasureSpec.EXACTLY:
            result = specSize;
            break;
        case MeasureSpec.UNSPECIFIED:
        default:
            result = size;
    }
    return result | (childMeasuredState & MEASURED_STATE_MASK);
}
```
1. resolveSize 和 resolveSizeAndState 的区别
	- public static int resolveSizeAndState(int size, int measureSpec, int childMeasuredState)中,childMeasuredState默认填0即可.
	- 当View X调用resolveSizeAndState返回结果,width和height,设置自身测量尺寸:setMeasuredDimension(width,height)调用后,父View P调用:
      ```
      int xMeasuredState = X.getMeasuredState();
      boolean xIsSmalledByMe = (xMeasuredState & View.MEASURED_STATE_TOO_SMALL) != 0;
      ```
		xIsSmalledByMe为true: 即代表X尺寸被P给压缩了.
		- X尺寸被P给压缩了: X的尺寸虽然听从P的限制,但是X当前尺寸不是X自身想要的,是因为P给的尺寸限制太小了,X被强行压缩了.
        - 父View判断子View是不是被强行压缩了,可以用于二次测量等,但Android原生控件对 View.MEASURED_STATE_TOO_SMALL 的支持并不好.
        	- 比如TextView即使被父View压缩了,父View使用 View.MEASURED_STATE_TOO_SMALL 判断也看不出来. 
            - 对于自定义ViewGroup,每次调用 measureChildWithMargins 测量子View,widthUsed要先用0,测量出子View正常宽度,判断正常宽度下需不需要换行.
	- resolveSizeAndState 精品文章:
		- https://www.cnblogs.com/aademeng/articles/11038547.html
		- https://vimsky.com/zh-tw/examples/detail/java-method-android.support.v4.view.ViewCompat.getMeasuredWidthAndState.html
	- 自己写自定义View,使用resolveSize 和 resolveSizeAndState区别不大.一般不用resolveSizeAndState.

##### 6. View/ViewGroup的布局过程 [HenCoder Android 自定义 View 2-1 布局基础](https://hencoder.com/ui-2-1/)
1. View/ViewGroup的布局过程,就是根据开发者的要求,计算出正确的尺寸和位置,并将其按照计算出的尺寸和位置进行'摆放',包括测量阶段 和 布局阶段.
2. 测量阶段
    - View/ViewGroup的measure被其父View调用,measure是1个调度方法,在onMeasure中执行实际的自我测量.
        1. 普通View: 在onMeasure中计算自身尺寸,并调用setMeasuredDimension保存计算出的宽高.
        2. ViewGroup: 根据每个子View的期望尺寸和ViewGroup自身可用尺寸,执行makeMeasureSpec得到子View的宽高属性值,将其作为参数调用子View的measure对其进行测量.
        <br>
        并根据子View测量后的尺寸,得到每个子View的实际尺寸和位置,进行保存.
        <br>
        最后根据所有子View的尺寸和位置,计算出自身的尺寸,调用setMeasuredDimension进行保存.
    - 不是所有ViewGroup都需要保存子View的位置,例如LinearLayout根据垂直或水平布局方向,在onLayout方法内,就可以通过获取每个子View的尺寸,在水平/垂直方向累加,将子View放到合适位置.
    - 有些情况,ViewGroup需要对子View进行2次或多次测量才能得到其正确的尺寸及位置.如图:
    ![](//p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/f99727fa2a8243059ccc3f97ffdff420~tplv-k3u1fbpfcp-zoom-1.image)
3. 布局阶段
    - View/ViewGroup的layout被其父View调用,layout方法参数确定其被摆放的具体位置.
        ```
        /**
         * Assign a size and position to a view and all of its
         * descendants
         * @param l Left position, relative to parent
         * @param t Top position, relative to parent
         * @param r Right position, relative to parent
         * @param b Bottom position, relative to parent
         */
        @SuppressWarnings({"unchecked"})
        public void layout(int l, int t, int r, int b) {
        ```
        layout会调用onLayout,执行实际的内部布局.内部布局,指的是将所有子View摆放到正确位置.onLayout对View和ViewGroup不一样.
        1. 普通View:
        没有子View,其onLayout是空方法.
        2. ViewGroup:
        会调用所有子View的layout方法,将之前保存的每个子View的位置(int l, int t, int r, int b)传入,摆放到正确位置上.
4. 布局过程的自定义有3类
    1. extends TextView/ImageView,对原有组件进行扩展:
        - 重写onMeasure,首先调用super.onMeasure,触发原始自我测量逻辑;
        - 执行 getMeasuredWidth(),getMeasuredHeight()获取原始测量宽高,并对原始测量宽高进行调整;
        - 调用setMeasuredDimension进行保存;
    2. extends View.创建全新的自定义View:
        - 流程见 '4. 全新自定义View尺寸'
    3. extends ViewGroup.创建全新的自定义ViewGroup: [HenCoder Android 自定义 View 2-3 定制 Layout 的内部布局](https://hencoder.com/ui-2-3/)
        - 计算子View尺寸:
            1. 根据开发者的要求(xml中子View以layout开头的属性),和自己的可用空间,获得调用子View的measure方法的2个参数. 通用的获取方式如下:
        - 通用的自定义ViewGroup的测量及布局逻辑.
        ```java
        public class CustomLayout extends ViewGroup{
            List<Rect> childLayoutParams = new ArrayList<Rect>();

            //重写onMeasure
            @Override
            protected void onMeasure(int widthMeasureSpec, int heightMeasureSpec) {
                //1:获取ViewGroup宽高的mode及size
                int selfWidthSpecMode = MeasureSpec.getMode(widthMeasureSpec);
                int selfWidthSpecSize = MeasureSpec.getSize(widthMeasureSpec);
                int selfHeightSpecMode = MeasureSpec.getMode(heightMeasureSpec);
                int selfHeightSpecSize = MeasureSpec.getSize(heightMeasureSpec);
                //2:对子View进行遍历,根据开发者对每个子View的尺寸要求,及ViewGroup自身的可用空间,获取子View的宽高属性,调用其measure进行测量,并保存子View的尺寸及位置
                //将要计算的子View宽高属性
                int childWidthMeasureSpec;
                int childHeightMeasureSpec;
                //子View测量后的宽高
                int childMeasuredWidth;
                int childMeasuredHeight;
                for(int i=0;i<getChildCount();i++){
                    //获取子View
                    View child = getChildAt(i);
                    //获取开发者对子View的尺寸要求/布局要求.就是ViewGroup.LayoutParams实例.
                    LayoutParams lp = child.getLayoutParams();
                    //lp.width有3种情况: 
                    //ViewGroup.LayoutParams.MATCH_PARENT , 
                    //ViewGroup.LayoutParams.WRAP_CONTENT , 具体像素值.
                    //源码:Information about how wide the view wants to be. Can be one of the constants FILL_PARENT (replaced by MATCH_PARENT in API Level 8) or WRAP_CONTENT, or an exact size
                    int width = lp.width;
                    int height = lp.height;
                    switch(width){
                        //子View的layout_width是match_parent
                        case ViewGroup.LayoutParams.MATCH_PARENT:
                            if(selfWidthSpecMode == MeasureSpec.EXACTLY || selfWidthSpecMode == MeasureSpec.AT_MOST){
                                //当前ViewGroup的layout_width是match_parent,具体像素值, 或 wrap_content
                                //match_parent : 则selfWidthSpecSize对应着当前ViewGroup的父View的具体宽度
                                //具体像素值 : 则selfWidthSpecSize对应着当前ViewGroup具体像素值
                                //wrap_content : 则selfWidthSpecSize对应着当前ViewGroup的父View的具体宽度
                                //ViewGroup自身可用空间,就是MeasureSpec.getSize得到的值.

                                childWidthMeasureSpec = MeasureSpec.makeMeasureSpec(selfWidthSpecSize - usedWidth, MeasureSpec.EXACTLY);
                            }else{
                                //selfWidthSpecMode == MeasureSpec.UNSPECIFIED
                                //代表当前ViewGroup宽度无上限
                                //当前ViewGroup宽度无上限,而子View又要填满ViewGroup,传入参数0,UNSPECIFIED即可.
                                childWidthMeasureSpec = MeasureSpec.makeMeasureSpec(0, MeasureSpec.UNSPECIFIED);
                            }
                            break;
                        //子View的layout_width是wrap_content
                        //wrap_content有一层隐藏含义:该子View的尺寸最多和父View一样大,不能超出父View的范围
                        case ViewGroup.LayoutParams.WRAP_CONTENT:
                            if(selfWidthSpecMode == MeasureSpec.EXACTLY || selfWidthSpecMode == MeasureSpec.AT_MOST){
                                //MeasureSpec.AT_MOST : 子View的宽度不要超过ViewGroup的可用宽度
                                childWidthMeasureSpec = MeasureSpec.makeMeasureSpec(selfWidthSpecSize - usedWidth, MeasureSpec.AT_MOST);
                            }else{
                                childWidthMeasureSpec = MeasureSpec.makeMeasureSpec(0, MeasureSpec.UNSPECIFIED);
                            }
                            break;
                        //子View的layout_width是 具体像素值
                        //这种情况下,应该直接返回该精确像素值.不需要再计算.
                        default:
                            childWidthMeasureSpec = MeasureSpec.makeMeasureSpec(width, MeasureSpec.EXACTLY);
                            break;
                    }
                    //childHeightMeasureSpec 的计算逻辑和 childWidthMeasureSpec 保持一致.
                    switch(height){
                        ****
                    }
                    //获取到调用该子View的measure方法所需的 childWidthMeasureSpec 和 childHeightMeasureSpec 后,执行.
                    child.measure(childWidthMeasureSpec, childHeightMeasureSpec);
                    //然后获取该子View测量后的宽高
                    childMeasuredWidth = child.getMeasuredWidth();
                    childMeasuredHeight = child.getMeasuredHeight();
                    //将该子View的尺寸及位置保存下来
                    Rect rect = new Rect(currLeft, currTop, currLeft + childMeasuredWidth, currTop + childMeasuredHeight);
                    childLayoutParams.add(rect);
                }
                //3:根据所有子View的尺寸及位置,计算自身的尺寸,并调用setMeasuredDimension保存
                int correctWidthMeasureSpec = **;
                int correctHeightMeasureSpec = **;
                setMeasuredDimension(correctWidthMeasureSpec, correctHeightMeasureSpec);
            }

            //重写onLayout
            @Override
            protected void onLayout(boolean changed, int left, int top, int right, int bottom) {
                //对于ViewGroup而言,onLayout目的就是将每个子View摆放到之前计算到的合适位置
                for(int i=0;i<childLayoutParams.size();i++){
                    //遍历每个子View,及之前计算出来的对应位置参数
                    Rect position = childLayoutParams.get(i);
                    View child = getChildAt(i);
                    //调用子View的layout方法,将其摆放到对应位置
                    child.layout(position.left, position.top, position.right, position.bottom);
                }
            }
        }
        ```
5. MeasureSpec.UNSPECIFIED
	- 什么时候会用到 MeasureSpec.UNSPECIFIED ? ScrollView, RecyclerView 在测量子View/Item高度的时候会用到,因为可滚动,不限制子View高度.
    - [南尘 每日一问：详细说一下 MeasureSpec.UNSPECIFIED](https://www.cnblogs.com/liushilin/p/11055741.html)
	- ScrollView源码
      ```
      android/widget/ScrollView.java
      @Override
      protected void measureChildWithMargins(View child, int parentWidthMeasureSpec, int widthUsed,
              int parentHeightMeasureSpec, int heightUsed) {
          final MarginLayoutParams lp = (MarginLayoutParams) child.getLayoutParams();
          //宽度需要限制
          final int childWidthMeasureSpec = getChildMeasureSpec(parentWidthMeasureSpec,
                  mPaddingLeft + mPaddingRight + lp.leftMargin + lp.rightMargin
                          + widthUsed, lp.width);
          final int usedTotal = mPaddingTop + mPaddingBottom + lp.topMargin + lp.bottomMargin +
                  heightUsed;
          //高度使用了 MeasureSpec.UNSPECIFIED ,不限制子View的高度
          final int childHeightMeasureSpec = MeasureSpec.makeSafeMeasureSpec(
                  Math.max(0, MeasureSpec.getSize(parentHeightMeasureSpec) - usedTotal),
                  MeasureSpec.UNSPECIFIED);

          child.measure(childWidthMeasureSpec, childHeightMeasureSpec);
      }
      ```

##### 7. onSizeChanged
1. 在layout方法之中调用.layout中发现View的尺寸发生改变才会调用.如果尺寸没有变化,即使layout调用多次也不会重复调用onSizeChanged.
    ```
    This is called during layout when the size of this view has changed. 
    protected void onSizeChanged(int w, int h, int oldw, int oldh) {
    }
    ```
##### 8. PathDashPathEffect 可用于绘制类似仪表盘刻度.
1. Path是有方向性的
	![](//p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/8159fbd70b864311bb545ff853cf6ad5~tplv-k3u1fbpfcp-zoom-1.image)
##### 9. 对于自定义View 实现固定尺寸
- 在onMeasure中执行setMeasuredDimension,告知父View,X自身测量的宽高
  ```java
  @Override
  protected void onMeasure(int widthMeasureSpec, int heightMeasureSpec) {
      setMeasuredDimension(200, 200);
  }
  ```
- 在layout中,可以对4个参数进行调整,自己决定自身摆放在父View的具体位置.
    - 一般不会这么写,只是说明子View可以通过这种方式决定自身尺寸及位置
  ```java
  @Override
  public void layout(int l, int t, int r, int b) {
      l = **;
      t = **;
      r = **;
      b = **;
      super.layout(l, t, r, b);
  }
  ```
- 示例:无论xml怎么写,最终都占用固定大小的View
  ```java
  public class SpecificSizeView extends View {
      ***
      @Override
      protected void onMeasure(int widthMeasureSpec, int heightMeasureSpec) {
          //测量阶段,告知父View自身尺寸是固定值
          setMeasuredDimension(200, 200);
      }
      @Override
      public void layout(int l, int t, int r, int b) {
          //布局阶段,如果父View实际给到自身的尺寸不是自身计算的值,可以调整4个参数,达到自身预期尺寸及位置.
          super.layout(300, 500, 300 + 200, 500 + 200);
      }
  }
  ```
##### 10. [Android应用坐标系统全面详解 大神的文章](https://blog.csdn.net/yanbober/article/details/50419117/)



### 事件分发
[自定义 View3-1 触摸反馈，以及 HenCoder Plus](https://hencoder.com/ui-3-1/)
<br>
##### 1. 几个关键方法:
- dispatchTouchEvent
- onInterceptTouchEvent
- onTouchEvent
- requestDisallowInterceptTouchEvent
##### 2. onTouchEvent
- 重写 onTouchEvent()，在里面写上你的触摸反馈算法，并返回 true（关键是 ACTION_DOWN 事件时返回 true）。
##### 3. onInterceptTouchEvent
- 如果是会发生触摸冲突的 ViewGroup，还需要重写 onInterceptTouchEvent()，在事件流开始时返回 false，并在确认接管事件流时返回一次 true，以实现对事件的拦截。
##### 4. requestDisallowInterceptTouchEvent
- 当子 View 临时需要阻止父 View 拦截事件流时，可以调用父 View 的 requestDisallowInterceptTouchEvent() ，通知父 View 在当前事件流中不再尝试通过 onInterceptTouchEvent() 来拦截。
##### 5. TouchTarget 非常重要
1. 和多点触控密切相关
2. Describes a touched view and the ids of the pointers that it has captured.
	- 在ViewGroup中,TouchTarget用于记录 每一个要消费事件的子View children,记录了其中每1个子View是被哪些手指触摸.
3. 
	- [ViewGroup事件分发总结-TouchTarget](https://juejin.im/post/6844904065613201421)
	- [ViewGroup事件分发总结-多点触摸事件拆分](https://juejin.im/post/6844904065617362952)
##### 6.


##### 20. 几个点的代码验证
1. 手指触摸一个子View,不断滑动,滑出子View的范围继续滑动,最后抬起手指.子View最终会收到什么事件?
    - MotionEvent.ACTION_UP
    - 即使手指滑动出子View范围,会继续收到MOVE,直至手指抬起,收到UP.
    ```
    public class MotionEventLayout extends LinearLayout {
        @Override
        public boolean onInterceptTouchEvent(MotionEvent ev) {
            boolean result = super.onInterceptTouchEvent(ev);
            switch (ev.getAction()) {
                case MotionEvent.ACTION_DOWN:
                    L.d("MotionEventLayout onInterceptTouchEvent ACTION_DOWN . result:" + result);
                    break;
                case MotionEvent.ACTION_MOVE:
                    L.d("MotionEventLayout onInterceptTouchEvent ACTION_MOVE . result:" + result);
                    break;
                case MotionEvent.ACTION_UP:
                    L.d("MotionEventLayout onInterceptTouchEvent ACTION_UP . result:" + result);
                    break;
                case MotionEvent.ACTION_CANCEL:
                    L.d("MotionEventLayout onInterceptTouchEvent ACTION_CANCEL . result:" + result);
                    break;
                case MotionEvent.ACTION_OUTSIDE:
                    L.d("MotionEventLayout onInterceptTouchEvent ACTION_OUTSIDE . result:" + result);
                    break;
                default:
                    L.d("MotionEventLayout onInterceptTouchEvent default . result:" + result);
                    break;
            }
            return result;
        }
        @Override
        public boolean onTouchEvent(MotionEvent event) {
            boolean result = super.onTouchEvent(event);
            switch (event.getAction()) {
                case MotionEvent.ACTION_DOWN:
                    L.d("MotionEventLayout onTouchEvent ACTION_DOWN . result:" + result);
                    break;
                case MotionEvent.ACTION_MOVE:
                    L.d("MotionEventLayout onTouchEvent ACTION_MOVE . result:" + result);
                    break;
                case MotionEvent.ACTION_UP:
                    L.d("MotionEventLayout onTouchEvent ACTION_UP . result:" + result);
                    break;
                case MotionEvent.ACTION_CANCEL:
                    L.d("MotionEventLayout onTouchEvent ACTION_CANCEL . result:" + result);
                    break;
                case MotionEvent.ACTION_OUTSIDE:
                    L.d("MotionEventLayout onTouchEvent ACTION_OUTSIDE . result:" + result);
                    break;
                default:
                    L.d("MotionEventLayout onTouchEvent default . result:" + result);
                    break;
            }
            return result;
        }
    }

    public class MotionEvent1View extends View {
        @Override
        public boolean onTouchEvent(MotionEvent event) {
            switch (event.getAction()){
                case MotionEvent.ACTION_DOWN:
                    L.d("MotionEvent1View onTouchEvent ACTION_DOWN");
                    break;
                case MotionEvent.ACTION_MOVE:
                    L.d("MotionEvent1View onTouchEvent ACTION_MOVE");
                    break;
                case MotionEvent.ACTION_UP:
                    L.d("MotionEvent1View onTouchEvent ACTION_UP");
                    break;
                case MotionEvent.ACTION_CANCEL:
                    L.d("MotionEvent1View onTouchEvent ACTION_CANCEL");
                    break;
                case MotionEvent.ACTION_OUTSIDE:
                    L.d("MotionEvent1View onTouchEvent ACTION_OUTSIDE");
                    break;
                default:
                    L.d("MotionEvent1View onTouchEvent default");
                    break;
            }
            return true;
        }
    }

    <MotionEventLayout
        android:layout_width="match_parent"
        android:layout_height="300dp"
        android:background="@color/colorAccent"
        android:gravity="center"
        android:orientation="vertical"
        android:padding="30dp">
        <MotionEvent1View
            android:layout_width="200dp"
            android:layout_height="100dp"
            android:background="@drawable/head" />
    </MotionEventLayout>

    日志:
    //最开始 ACTION_DOWN
    D/Jet2020: MotionEventLayout onInterceptTouchEvent ACTION_DOWN . result:false
    D/Jet2020: MotionEvent1View onTouchEvent ACTION_DOWN
    //变成 ACTION_MOVE
    D/Jet2020: MotionEventLayout onInterceptTouchEvent ACTION_MOVE . result:false
    D/Jet2020: MotionEvent1View onTouchEvent ACTION_MOVE
    ***
    //超出子View范围继续滑动,子View依然收到 ACTION_MOVE
    D/Jet2020: MotionEventLayout onInterceptTouchEvent ACTION_MOVE . result:false
    D/Jet2020: MotionEvent1View onTouchEvent ACTION_MOVE
    ***
    //手指抬起,最终子View收到 ACTION_UP
    D/Jet2020: MotionEventLayout onInterceptTouchEvent ACTION_UP . result:false
    D/Jet2020: MotionEvent1View onTouchEvent ACTION_UP
    ```
2. 子View在onTouchEvent方法中,只有MotionEvent.ACTION_DOWN返回true,其余都返回false,会不会失去对事件流的处理?
    - 不会.
    - 只要MotionEvent.ACTION_DOWN返回true,即可获取该组事件流的处理权.
    ```
    public class MotionEvent2View extends View {
        @Override
        public boolean onTouchEvent(MotionEvent event) {
            switch (event.getAction()){
                case MotionEvent.ACTION_DOWN:
                    L.d("MotionEvent2View onTouchEvent ACTION_DOWN");
                    //只在 MotionEvent.ACTION_DOWN 返回true
                    return true;
                case MotionEvent.ACTION_MOVE:
                    L.d("MotionEvent2View onTouchEvent ACTION_MOVE");
                    break;
                case MotionEvent.ACTION_UP:
                    L.d("MotionEvent2View onTouchEvent ACTION_UP");
                    break;
                case MotionEvent.ACTION_CANCEL:
                    L.d("MotionEvent2View onTouchEvent ACTION_CANCEL");
                    break;
                case MotionEvent.ACTION_OUTSIDE:
                    L.d("MotionEvent2View onTouchEvent ACTION_OUTSIDE");
                    break;
                default:
                    L.d("MotionEvent2View onTouchEvent default");
                    break;
            }
            //其余情况都返回false
            return false;
        }
    }

    <MotionEventLayout
        android:layout_width="match_parent"
        android:layout_height="300dp"
        android:background="@color/colorPrimary"
        android:gravity="center"
        android:orientation="vertical"
        android:padding="30dp">
        <MotionEvent2View
            android:layout_width="200dp"
            android:layout_height="100dp"
            android:background="@drawable/head" />
    </MotionEventLayout>

    日志:
    //最开始 ACTION_DOWN , 子View在 onTouchEvent 返回true
    D/Jet2020: MotionEventLayout onInterceptTouchEvent ACTION_DOWN . result:false
    D/Jet2020: MotionEvent2View onTouchEvent ACTION_DOWN
    //后续所有事件, 子View在 onTouchEvent 返回false
    //依然可以收到后续事件流.
    D/Jet2020: MotionEventLayout onInterceptTouchEvent ACTION_MOVE . result:false
    D/Jet2020: MotionEvent2View onTouchEvent ACTION_MOVE
    ***
    //最终手指抬起,收到 ACTION_UP
    D/Jet2020: MotionEventLayout onInterceptTouchEvent ACTION_UP . result:false
    D/Jet2020: MotionEvent2View onTouchEvent ACTION_UP
    ```
3. [ACTION_CANCEL](https://blog.csdn.net/cufelsd/article/details/89471402)
    ```
    如果某一个子View处理了Down事件，那么随之而来的Move和Up事件也会交给它处理。
    但是交给它处理之前，父View还是可以拦截事件的，如果拦截了事件，
    那么子View就会收到一个Cancel事件，并且不会收到后续的Move和Up事件
    ```
##### 21. 待试验
1. View处于GONE/不可见状态,调用其invalidate,无法触发onDraw
2. 调用invalidate,自定义View无法触发onDraw,原因是什么/onDraw执行的前提是什么?
	- [view.draw()以及调用invalidate()没有触发onDraw()](https://blog.csdn.net/yizunda/article/details/51423610)
