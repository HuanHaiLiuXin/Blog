源码及所在DEMO已上传至<b>GitHub:</b>[<b>SweetTips</b>](https://github.com/HuanHaiLiuXin/SweetTips),欢迎大家提Bug,喜欢的话记得Star或Fork下哈!

##### 1.为什么要写这个库?
上面的问题也可以这样问:有哪些常见的需求,Android原生Toast及Design包中的Snackbar实现起来相对繁琐?
<b>Toast:</b>
1. 原生Toast无法/不方便自定义显示时间;
2. 原生Toast,需要等待队列中前面的Toast实例显示完毕之后才可以显示,实时性差;
3. 原生Toast,想在正在显示的Toast实例上显示新的内容并设置新内容的显示时间,实现较繁琐;
4. 原生Toast,无法/不方便自定义动画;
5. Android系统版本过多,不同的厂商对系统的定制也很不同,同一段代码在不同的机器上,Toast的样式差异很大,不利于App的一致性体验;

<b>Snackbar:</b>
- Design包中的Snackbar,无法自定义动画;

##### 2.SweetTips有什么用?
很显然,可以解决上面列举的那些很常见的小问题;
<b>截图:</b>
![SweetToast及SweetSnackbar效果录屏.gif](http://upload-images.jianshu.io/upload_images/3501388-0442aeca00619aba.gif?imageMogr2/auto-orient/strip)

##### 3.SweetTips的结构?
自定义Toast:SweetToast
<b>+</b>
自定义Snackbar:SweetSnackbar
<b>+</b>
SnackbarUtils:SweetSnackbar的工具类

##### 4.SweetTips的实现思路
<b>SweetToast:</b>
- 在SweetToastManager中,利用队列实现对SweetToast实例的管理,直接调用SweetToast的show()方法,可以实现和原生Toast几乎一致的体验;
- 在SweetToastManager中,通过对队列的清空,实现即时显示当前SweetToast实例的内容;
- 在SweetToast中,通过设置WindowManager.LayoutParams.windowAnimations,实现SweetToast实例自定义的出入场动画;
- SweetToast支持链式调用,调用尽可能的快捷;

<b>SweetSnackbar:</b>
- 几乎完全拷贝了Design包中的Snackbar,只是添加了一个设置自定义出入场动画的方法:setAnimations
- 参照之前写过的一个工具类<b>GitHub:</b>[<b>SnackbarUtils</b>](https://github.com/HuanHaiLiuXin/SnackbarUtils),为SweetSnackbar也写了一个工具类,同样支持练市调用,实现'一行代码设置多重属性';

<b>SweetTips.java</b>
- 这个工具类待完善,是为了通过SweetToast或SweetSnackbar,封装一些比较常用且精美的效果,通过静态方法直接调用,提升开发者一些效率.

另外,为了这个提示库,也花了不少时间收集了一些常用的颜色,保存在Constant.java中,可作为一个通用的工具类适用于不同项目,喜欢的同学尽管拿走.

##### 5.SweetTips的使用限制
SweetToast是通过WindowManager向屏幕添加View来展示提示信息:
>params.type = WindowManager.LayoutParams.TYPE_TOAST;

在Manifest.xml中已经声明过权限:
><uses-permission android:name="android.permission.SYSTEM_ALERT_WINDOW"/>

在SDK>=23(Android 6)的系统中,用户需要手动允许当前App使用这个权限,才可以正常显示!

##### 6.SweetTips部分代码
```
/**
 * 自定义Toast
 *
 * 作者:幻海流心
 * GitHub:https://github.com/HuanHaiLiuXin
 * 邮箱:wall0920@163.com
 * 2016/12/13
 */

public final class SweetToast {
    public static final int LENGTH_SHORT = 0;
    public static final int LENGTH_LONG = 1;
    public static final long SHORT_DELAY = 2000; // 2 seconds
    public static final long LONG_DELAY = 3500; // 3.5 seconds
    //SweetToast默认背景色
    private static int mBackgroundColor = 0XE8484848;
    //
    private View mContentView = null;   //内容区域View
    private SweetToastConfiguration mConfiguration = null;
    private WindowManager mWindowManager = null;
    private boolean showing = false;    //是否在展示中
    private boolean showEnabled = true; //是否允许展示
    private boolean hideEnabled = true; //是否允许移除
    private boolean stateChangeEnabled = true;  //是否允许改变展示状态

    public static SweetToast makeText(Context context, CharSequence text){
        return makeText(context, text, LENGTH_SHORT);
    }
    public static SweetToast makeText(View mContentView){
        return makeText(mContentView, LENGTH_SHORT);
    }
    public static SweetToast makeText(Context context, CharSequence text, int duration) {
        try {
            LayoutInflater inflate = (LayoutInflater)context.getSystemService(Context.LAYOUT_INFLATER_SERVICE);
            View v = inflate.inflate(R.layout.transient_notification, null);
            TextView tv = (TextView)v.findViewById(R.id.message);
            tv.setText(text);
            SweetToast sweetToast = new SweetToast();
            sweetToast.mContentView = v;
            sweetToast.mContentView.setBackgroundDrawable(getBackgroundDrawable(sweetToast, mBackgroundColor));
            initConfiguration(sweetToast,duration);
            return sweetToast;
        }catch (Exception e){
            Log.e("幻海流心","e:"+e.getLocalizedMessage()+":69");
        }
        return null;
    }
    public static SweetToast makeText(View mContentView, int duration){
        SweetToast sweetToast = new SweetToast();
        sweetToast.mContentView = mContentView;
        initConfiguration(sweetToast,duration);
        return sweetToast;
    }
    private static void initConfiguration(SweetToast sweetToast,int duration){
        try {
            if(duration < 0){
                throw new RuntimeException("显示时长必须>=0!");
            }
            //1:初始化mWindowManager
            sweetToast.mWindowManager = (WindowManager) sweetToast.getContentView().getContext().getApplicationContext().getSystemService(Context.WINDOW_SERVICE);
            //2:初始化mConfiguration
            SweetToastConfiguration mConfiguration = new SweetToastConfiguration();
            //2.1:设置显示时间
            mConfiguration.setDuration(duration);
            //2.2:设置WindowManager.LayoutParams属性
            WindowManager.LayoutParams params = new WindowManager.LayoutParams();
            final Configuration config = sweetToast.getContentView().getContext().getResources().getConfiguration();
            final int gravity = Gravity.CENTER_HORIZONTAL | Gravity.BOTTOM;
            params.gravity = gravity;
            if ((gravity & Gravity.HORIZONTAL_GRAVITY_MASK) == Gravity.FILL_HORIZONTAL) {
                params.horizontalWeight = 1.0f;
            }
            if ((gravity & Gravity.VERTICAL_GRAVITY_MASK) == Gravity.FILL_VERTICAL) {
                params.verticalWeight = 1.0f;
            }
            params.x = 0;
            params.y = sweetToast.getContentView().getContext().getResources().getDimensionPixelSize(R.dimen.toast_y_offset);
            params.verticalMargin = 0.0f;
            params.horizontalMargin = 0.0f;
            params.height = WindowManager.LayoutParams.WRAP_CONTENT;
            params.width = WindowManager.LayoutParams.WRAP_CONTENT;
            params.format = PixelFormat.TRANSLUCENT;
            params.windowAnimations = R.style.Anim_SweetToast;
            //在小米5S上实验,前两种type均会报错
            params.type = WindowManager.LayoutParams.TYPE_TOAST;
//            params.type = WindowManager.LayoutParams.TYPE_SYSTEM_ALERT;
//            params.type = WindowManager.LayoutParams.TYPE_PHONE;
            params.setTitle("Toast");
            params.flags = WindowManager.LayoutParams.FLAG_KEEP_SCREEN_ON
                    | WindowManager.LayoutParams.FLAG_NOT_FOCUSABLE
                    | WindowManager.LayoutParams.FLAG_NOT_TOUCHABLE;
            mConfiguration.setParams(params);
            sweetToast.setConfiguration(mConfiguration);
        }catch (Exception e){
            Log.e("幻海流心","e:"+e.getLocalizedMessage()+":120");
        }
    }
    /**
     * 根据指定的背景色,获得mToastView的背景drawable实例
     * @param backgroundColor
     * @return
     */
    private static ShapeDrawable getBackgroundDrawable(SweetToast sweetToast, @ColorInt int backgroundColor){
        try {
            ShapeDrawable shapeDrawable = new ShapeDrawable();
            DrawableCompat.setTint(shapeDrawable,backgroundColor);
            //获取当前设备的屏幕尺寸
            //实验发现不同的设备上面,Toast内容区域的padding值并不相同,根据屏幕的宽度分别进行处理,尽量接近设备原生Toast的体验
            int widthPixels = sweetToast.getContentView().getResources().getDisplayMetrics().widthPixels;
            int heightPixels = sweetToast.getContentView().getResources().getDisplayMetrics().heightPixels;
            float density = sweetToast.getContentView().getResources().getDisplayMetrics().density;
            if(widthPixels >= 1070){
                //例如小米5S:1920 x 1080
                shapeDrawable.setPadding((int)(density*13),(int)(density*12),(int)(density*13),(int)(density*12));
            }else {
                //例如红米2:1280x720
                shapeDrawable.setPadding((int)(density*14),(int)(density*13),(int)(density*14),(int)(density*13));
            }
            float radius = density*8;
            float[] outerRadii = new float[]{radius,radius,radius,radius,radius,radius,radius,radius};
            int width = sweetToast.getContentView().getWidth();
            int height = sweetToast.getContentView().getHeight();
            RectF rectF = new RectF(1,1,width-1,height-1);
            RoundRectShape roundRectShape = new RoundRectShape(outerRadii,rectF,null);
            shapeDrawable.setShape(roundRectShape);
            DrawableCompat.setTint(shapeDrawable,backgroundColor);
            return shapeDrawable;
        }catch (Exception e){
            Log.e("幻海流心","e:"+e.getLocalizedMessage()+":154");
        }
        return null;
    }
    /**
     * 自定义SweetToast实例的入场出场动画
     * @param windowAnimations
     * @return
     */
    public SweetToast setWindowAnimations(@StyleRes int windowAnimations){
        mConfiguration.getParams().windowAnimations = windowAnimations;
        return this;
    }
    public SweetToast setGravity(int gravity, int xOffset, int yOffset) {
        mConfiguration.getParams().gravity = gravity;
        mConfiguration.getParams().x = xOffset;
        mConfiguration.getParams().y = yOffset;
        return this;
    }
    public SweetToast setMargin(float horizontalMargin, float verticalMargin) {
        mConfiguration.getParams().horizontalMargin = horizontalMargin;
        mConfiguration.getParams().verticalMargin = verticalMargin;
        return this;
    }
    /**
     * 向mContentView中添加View
     *
     * @param view
     * @param index
     * @return
     */
    public SweetToast addView(View view, int index) {
        if(mContentView != null && mContentView instanceof ViewGroup){
            ((ViewGroup)mContentView).addView(view,index);
        }
        return this;
    }
    /**
     * 设置SweetToast实例中TextView的文字颜色
     *
     * @param messageColor
     * @return
     */
    public SweetToast messageColor(@ColorInt int messageColor){
        if(mContentView !=null && mContentView.findViewById(R.id.message) != null && mContentView.findViewById(R.id.message) instanceof TextView){
            TextView textView = ((TextView) mContentView.findViewById(R.id.message));
            textView.setTextColor(messageColor);
        }
        return this;
    }
    /**
     * 设置SweetToast实例的背景颜色
     *
     * @param backgroundColor
     * @return
     */
    public SweetToast backgroundColor(@ColorInt int backgroundColor){
        if(mContentView!=null){
            mContentView.setBackgroundDrawable(getBackgroundDrawable(this, backgroundColor));
        }
        return this;
    }
    /**
     * 设置SweetToast实例的背景资源
     *
     * @param background
     * @return
     */
    public SweetToast backgroundResource(@DrawableRes int background){
        if(mContentView!=null){
            mContentView.setBackgroundResource(background);
        }
        return this;
    }
    /**
     * 设置SweetToast实例的文字颜色及背景颜色
     *
     * @param messageColor
     * @param backgroundColor
     * @return
     */
    public SweetToast colors(@ColorInt int messageColor, @ColorInt int backgroundColor) {
        messageColor(messageColor);
        backgroundColor(backgroundColor);
        return this;
    }
    /**
     * 设置SweetToast实例的文字颜色及背景资源
     *
     * @param messageColor
     * @param background
     * @return
     */
    public SweetToast textColorAndBackground(@ColorInt int messageColor, @DrawableRes int background) {
        messageColor(messageColor);
        backgroundResource(background);
        return this;
    }

    /**
     * 设置SweetToast实例的宽高
     *  很有用的功能,参考了简书上的文章:http://www.jianshu.com/p/491b17281c0a
     * @param width     SweetToast实例的宽度,单位是pix
     * @param height    SweetToast实例的高度,单位是pix
     * @return
     */
    public SweetToast size(int width, int height){
        if(mContentView!=null && mContentView instanceof LinearLayout){
            mContentView.setMinimumWidth(width);
            mContentView.setMinimumHeight(height);
            ((LinearLayout)mContentView).setGravity(Gravity.CENTER);
            try {
                TextView textView = ((TextView) mContentView.findViewById(R.id.message));
                LinearLayout.LayoutParams params = (LinearLayout.LayoutParams) textView.getLayoutParams();
                params.width = LinearLayout.LayoutParams.MATCH_PARENT;
                params.height = LinearLayout.LayoutParams.MATCH_PARENT;
                textView.setLayoutParams(params);
                textView.setGravity(Gravity.CENTER);
            }catch (Exception e){
                Log.e("幻海流心","e:"+e.getLocalizedMessage());
            }
        }
        return this;
    }

    /**
     * 设置SweetToast实例的显示位置:左上
     * @return
     */
    public SweetToast leftTop(){
        return setGravity(Gravity.LEFT|Gravity.TOP,0,0);
    }
    /**
     * 设置SweetToast实例的显示位置:右上
     * @return
     */
    public SweetToast rightTop(){
        return setGravity(Gravity.RIGHT|Gravity.TOP,0,0);
    }
    /**
     * 设置SweetToast实例的显示位置:左下
     * @return
     */
    public SweetToast leftBottom(){
        return setGravity(Gravity.LEFT|Gravity.BOTTOM,0,0);
    }
    /**
     * 设置SweetToast实例的显示位置:右下
     * @return
     */
    public SweetToast rightBottom(){
        return setGravity(Gravity.RIGHT|Gravity.BOTTOM,0,0);
    }
    /**
     * 设置SweetToast实例的显示位置:上中
     * @return
     */
    public SweetToast topCenter(){
        return setGravity(Gravity.TOP|Gravity.CENTER_HORIZONTAL,0,0);
    }
    /**
     * 设置SweetToast实例的显示位置:下中
     * @return
     */
    public SweetToast bottomCenter(){
        return setGravity(Gravity.BOTTOM|Gravity.CENTER_HORIZONTAL,0,0);
    }
    /**
     * 设置SweetToast实例的显示位置:左中
     * @return
     */
    public SweetToast leftCenter(){
        return setGravity(Gravity.LEFT|Gravity.CENTER_VERTICAL,0,0);
    }
    /**
     * 设置SweetToast实例的显示位置:右中
     * @return
     */
    public SweetToast rightCenter(){
        return setGravity(Gravity.RIGHT|Gravity.CENTER_VERTICAL,0,0);
    }
    /**
     * 设置SweetToast实例的显示位置:正中
     * @return
     */
    public SweetToast center(){
        return setGravity(Gravity.CENTER,0,0);
    }
    /**
     * 将SweetToast实例显示在指定View的顶部
     * @param targetView    指定View
     * @param statusHeight  状态栏显示情况下,状态栏的高度
     * @return
     */
    public SweetToast layoutAbove(View targetView, int statusHeight){
        if(mContentView!=null){
            int[] locations = new int[2];
            targetView.getLocationOnScreen(locations);
            //必须保证指定View的顶部可见
            int screenHeight = ScreenUtil.getScreenHeight(mContentView.getContext());
            if(locations[1] > statusHeight&&locations[1]<screenHeight){
                setGravity(Gravity.BOTTOM|Gravity.CENTER_HORIZONTAL,0,screenHeight - locations[1]);
            }
        }
        return this;
    }
    /**
     * 将SweetToast实例显示在指定View的底部
     * @param targetView
     * @param statusHeight
     * @return
     */
    public SweetToast layoutBellow(View targetView, int statusHeight){
        if(mContentView!=null){
            int[] locations = new int[2];
            targetView.getLocationOnScreen(locations);
            //必须保证指定View的底部可见
            int screenHeight = ScreenUtil.getScreenHeight(mContentView.getContext());
            if(locations[1]+targetView.getHeight() > statusHeight&&locations[1]+targetView.getHeight()<screenHeight){
                setGravity(Gravity.TOP|Gravity.CENTER_HORIZONTAL,0,locations[1]+targetView.getHeight()-statusHeight);
            }
        }
        return this;
    }


    /**********************************************  SweetToast显示及移除  **********************************************/
    Handler mHandler = new Handler();
    Runnable mHide = new Runnable() {
        @Override
        public void run() {
            handleHide();
        }
    };
    protected void handleHide() {
        if(this != null && mContentView != null){
            if(stateChangeEnabled){
                if(hideEnabled){
                    if(showing){
                        mWindowManager.removeView(mContentView);
                    }
                    showing = false;
                    mContentView = null;
                }else{
                }
            }
        }
    }
    protected void handleShow() {
        if(mContentView != null){
            if(stateChangeEnabled){
                if(showEnabled){
                    try {
                        mWindowManager.addView(mContentView,mConfiguration.getParams());
                        long delay = (mConfiguration.getDuration() == LENGTH_LONG || mConfiguration.getDuration() == Toast.LENGTH_LONG) ? LONG_DELAY : ((mConfiguration.getDuration() == LENGTH_SHORT || mConfiguration.getDuration() == Toast.LENGTH_SHORT)? SHORT_DELAY : mConfiguration.getDuration());
                        mHandler.postDelayed(mHide,delay);
                        showing = true;
                    }catch (Exception e){
                        Log.e("幻海流心","e:"+e.getLocalizedMessage()+":213");
                    }
                }
            }
        }
    }

    /**
     * 保持当前实例的显示状态:不允许向Window中添加或者移除View
     */
    protected void removeCallbacks(){
        stateChangeEnabled = false;
    }

    /**
     * 设置是否允许展示当前实例
     * @param showEnabled
     */
    public void setShowEnabled(boolean showEnabled) {
        this.showEnabled = showEnabled;
    }

    /**
     * 设置是否允许移除当前实例中的View
     * @param hideEnabled
     */
    public void setHideEnabled(boolean hideEnabled) {
        this.hideEnabled = hideEnabled;
    }

    /**
     * 设置是否允许改变当前实例的展示状态
     * @param stateChangeEnabled
     */
    public void setStateChangeEnabled(boolean stateChangeEnabled) {
        this.stateChangeEnabled = stateChangeEnabled;
    }

    /**
     * 将当前实例添加到队列{@link SweetToastManager#queue}中,若队列为空,则加入队列后直接进行展示
     */
    public void show(){
        try {
            if (Build.VERSION.SDK_INT >= 23) {
                //Android6.0以上，需要动态声明权限
                if(mContentView!=null && !Settings.canDrawOverlays(mContentView.getContext().getApplicationContext())) {
                    //用户还未允许该权限
                    Intent intent = new Intent(Settings.ACTION_MANAGE_OVERLAY_PERMISSION);
                    mContentView.getContext().startActivity(intent);
                    return;
                } else if(mContentView!=null) {
                    //用户已经允许该权限
                    SweetToastManager.show(this);
                }
            } else {
                //Android6.0以下，不用动态声明权限
                if (mContentView!=null) {
                    SweetToastManager.show(this);
                }
            }
//            SweetToastManager.show(this);
        }catch (Exception e){
            Log.e("幻海流心","e:"+e.getLocalizedMessage()+":232");
        }
    }
    /**
     * 利用队列{@link SweetToastManager#queue}中正在展示的SweetToast实例,继续展示当前实例的内容
     */
    public void showByPrevious(){
        try {
            if (Build.VERSION.SDK_INT >= 23) {
                //Android6.0以上，需要动态声明权限
                if(mContentView!=null && !Settings.canDrawOverlays(mContentView.getContext().getApplicationContext())) {
                    //用户还未允许该权限
                    Intent intent = new Intent(Settings.ACTION_MANAGE_OVERLAY_PERMISSION);
                    mContentView.getContext().startActivity(intent);
                    return;
                } else if(mContentView!=null) {
                    //用户已经允许该权限
                    SweetToastManager.showByPrevious(this);
                }
            } else {
                //Android6.0以下，不用动态声明权限
                if (mContentView!=null) {
                    SweetToastManager.showByPrevious(this);
                }
            }
//            SweetToastManager.showByPrevious(this);
        }catch (Exception e){
            Log.e("幻海流心","e:"+e.getLocalizedMessage()+":290");
        }
    }
    /**
     * 清空队列{@link SweetToastManager#queue}中已经存在的SweetToast实例,直接展示当前实例的内容
     */
    public void showImmediate(){
        try {
            if (Build.VERSION.SDK_INT >= 23) {
                //Android6.0以上，需要动态声明权限
                if(mContentView!=null && !Settings.canDrawOverlays(mContentView.getContext().getApplicationContext())) {
                    //用户还未允许该权限
                    Intent intent = new Intent(Settings.ACTION_MANAGE_OVERLAY_PERMISSION);
                    mContentView.getContext().startActivity(intent);
                    return;
                } else if(mContentView!=null) {
                    //用户已经允许该权限
                    SweetToastManager.showImmediate(this);
                }
            } else {
                //Android6.0以下，不用动态声明权限
                if (mContentView!=null) {
                    SweetToastManager.showImmediate(this);
                }
            }
//            SweetToastManager.showImmediate(this);
        }catch (Exception e){
            Log.e("幻海流心","e:"+e.getLocalizedMessage()+":252");
        }
    }
    /**
     * 移除当前SweetToast并将mContentView置空
     */
    public void hide() {
        mHandler.post(mHide);
    }
    /**********************************************  SweetToast显示及移除  **********************************************/

    //Setter&Getter
    public View getContentView() {
        return mContentView;
    }
    public void setContentView(View mContentView) {
        this.mContentView = mContentView;
    }
    public SweetToastConfiguration getConfiguration() {
        return mConfiguration;
    }
    public void setConfiguration(SweetToastConfiguration mConfiguration) {
        this.mConfiguration = mConfiguration;
    }
    public WindowManager getWindowManager() {
        return mWindowManager;
    }
    public void setWindowManager(WindowManager mWindowManager) {
        this.mWindowManager = mWindowManager;
    }
    public boolean isShowing() {
        return showing;
    }
    public void setShowing(boolean showing) {
        this.showing = showing;
    }
}
```

源码及所在DEMO已上传至<b>GitHub:</b>[<b>SweetTips</b>](https://github.com/HuanHaiLiuXin/SweetTips),欢迎大家提Bug,喜欢的话记得Star或Fork下哈!

<b>That's all !</b>
