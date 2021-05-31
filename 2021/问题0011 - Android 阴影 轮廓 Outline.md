## 1.阴影
### 相关文章
1. [Android materialDesign 风格阴影改变阴影颜色](https://blog.csdn.net/mg2flyingff/article/details/105877114)
2. setOutlineAmbientShadowColor 及 setOutlineSpotShadowColor
3. [承香墨影 聊聊 Material Design里，阴影的那些事儿！](https://juejin.im/post/6844903507749765134)
### 知识点
1. Api Level>=21情况下:
	- 直接使用 elevation , translationZ 即可实现阴影.
    - 使用其中1个就行,同时使用,则View在Z轴的高度是 elevation + translationZ . 两者效果是累加的.
2. elevation,translationZ要生效,对应的View一定要有背景:
	- android:background 或 View.setBackground()
3. Api Level<21: 无法实现.
	- 可是使用.9图模拟阴影效果,且颜色,阴影范围可控制,但是会占用View的尺寸,见 承香墨影
4. 在 sdk>=21 情况下,android:elevation 和 android:translationZ才有用,不占用View的大小,但是无法控制阴影颜色.
5. 

### 本地验证
1. Android未提供现成的工具,实现自定义颜色的阴影.
2. 开源库使用的是Paint.setShadowLayer方法:
	- 在原始View外部套了1个ViewGroup,增加了View的层级;横向和纵向占用尺寸都会增大;
    - Paint.setShadowLayer试验下来,颜色的透明度变化太快,只有很窄的范围能看到颜色渐变;
3. 使用Shader在特定场景下,实现阴影/颜色渐变效果,更容易控制.
    - 更进一步,使用Paint.setXfermode,仅仅截取出'阴影部分',可以解决上层内容区域是半透明情况下,底部的'阴影'冗余显示问题.还未试验.
    ```java
    public class ShaderView extends View {
        private Paint paint;
        private Shader shader;
        int[] colors;
        float[] stops;

        public ShaderView(Context context) {
            this(context, null, 0);
        }
        public ShaderView(Context context, @Nullable AttributeSet attrs) {
            this(context, attrs, 0);
        }
        public ShaderView(Context context, @Nullable AttributeSet attrs, int defStyleAttr) {
            super(context, attrs, defStyleAttr);
            init();
        }
        private void init() {
            setLayerType(LAYER_TYPE_SOFTWARE, null);
            paint = new Paint(Paint.ANTI_ALIAS_FLAG);
            paint.setStyle(Paint.Style.FILL);
            colors = new int[]{
                    Color.parseColor("#000C51CA"),
                    Color.parseColor("#A00C51CA"),
                    Color.parseColor("#500C51CA"),
                    Color.parseColor("#000C51CA")
            };
            stops = new float[]{
                    0.75F,
                    0.80F,
                    0.90F,
                    1.00F
            };
        }

        @Override
        protected void onSizeChanged(int w, int h, int oldw, int oldh) {
            super.onSizeChanged(w, h, oldw, oldh);
            shader = new RadialGradient(getWidth() / 2.0F, getHeight() / 2.0F, 300.0f, colors, stops, Shader.TileMode.CLAMP);
        }

        @Override
        protected void onDraw(Canvas canvas) {
            super.onDraw(canvas);
            paint.setShader(shader);
            //经试验,同时使用Shader和setShadowLayer,setShadowLayer无效果
            //paint.setShadowLayer(40,0,0,Color.parseColor("#040C51CA"));
            canvas.drawCircle(getWidth() / 2, getHeight() / 2, 300.0F, paint);
            paint.setShader(null);
            paint.setColor(Color.parseColor("#FF0C51CA"));
            canvas.drawCircle(getWidth() / 2, getHeight() / 2 - 300.0F * 0.20F, 300.0F, paint);
        }
    }
    ```
    
	![](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/8052db5a341a4116b28ce681f4c5e14c~tplv-k3u1fbpfcp-watermark.image)
    
	- 直接继承ImageView,添加部分属性,实现'上,下,左,右'四个位置绘制阴影,可以调整阴影初始透明度及阴影宽度与实际内容区域的比例.也可以添加'角度'属性,可以围绕实际内容区域任意角度绘制阴影.
    ```java
    public class RoundShadowImageView extends AppCompatImageView {
        private Paint paint;
        private Shader shader;
        int[] colors;
        float[] stops;
        private float shadowRatio = 0.20F;
        private float shadowRadius = 0.0F;
        private boolean shadowRatioViaHeight = true;
        private boolean shadowOnEnd = true;
        private boolean shadowOnBottom = true;
        private @IntRange(from = 1, to = 255)
        int shadowStartAlpha = 255;

        ***
    }
    ```

### RoundShadowImageView
1. 自定义阴影宽度相对于内容区域半径的比例
2. 自定义阴影中心相对于内容区域中心的角度,以内容区域垂直向下为0度/起始角度
3. 自定义阴影颜色
4. 自定义阴影颜色初始透明度
5. 自定义阴影位置
6. attr属性:
    ```xml
    <?xml version="1.0" encoding="utf-8"?>
    <resources>
        <declare-styleable name="RoundShadowImageView">
            <!--阴影宽度相对于内容区域半径的比例-->
            <attr name="shadowRatio" format="float" />
            <!--阴影中心相对于内容区域中心的角度,以内容区域垂直向下为0度/起始角度-->
            <attr name="shadowCircleAngle" format="float" />
            <!--阴影颜色-->
            <attr name="shadowColor" format="color|reference" />
            <!--阴影颜色初始透明度-->
            <attr name="shadowStartAlpha" format="float" />
            <!--阴影位置-->
            <attr name="shadowPosition" format="enum">
                <enum name="start" value="1" />
                <enum name="top" value="2" />
                <enum name="end" value="3" />
                <enum name="bottom" value="4" />
            </attr>
        </declare-styleable>
    </resources>
    ```
7. Java源码
    ```java
    import static com.huanhailiuxin.jet2020.othertest.shadow.ShadowPosition.BOTTOM;
    import static com.huanhailiuxin.jet2020.othertest.shadow.ShadowPosition.END;
    import static com.huanhailiuxin.jet2020.othertest.shadow.ShadowPosition.START;
    import static com.huanhailiuxin.jet2020.othertest.shadow.ShadowPosition.TOP;
    @IntDef({
            START,
            TOP,
            END,
            BOTTOM
    })
    @Retention(RetentionPolicy.SOURCE)
    @Target({ElementType.FIELD, ElementType.PARAMETER})
    @interface ShadowPosition {
        int START = 1;
        int TOP = 2;
        int END = 3;
        int BOTTOM = 4;
    }

    /**
     * @author HuanHaiLiuXin
     * @github https://github.com/HuanHaiLiuXin
     * @date 2020/11/23
     */
    public class RoundShadowImageView extends AppCompatImageView {
        private Paint paint;
        private Shader shader;
        int[] colors;
        float[] stops;
        private float contentSize;
        @FloatRange(from = 0.0F, to = 1.0F)
        private float shadowRatio = 0.30F;
        private float shadowRadius = 0.0F;
        private float shadowCenterX, shadowCenterY;
        @ShadowPosition
        private int shadowPosition = ShadowPosition.BOTTOM;
        private float shadowCircleAngle = 0F;
        private boolean useShadowCircleAngle = false;
        private int red, green, blue;
        private int shadowColor = Color.RED;
        private @FloatRange(from = 0F, to = 1F)
        float shadowStartAlpha = 0.5F;
        private boolean isLtr = true;

        public RoundShadowImageView(Context context) {
            this(context, null, 0);
        }

        public RoundShadowImageView(Context context, @Nullable AttributeSet attrs) {
            this(context, attrs, 0);
        }

        public RoundShadowImageView(Context context, @Nullable AttributeSet attrs, int defStyleAttr) {
            super(context, attrs, defStyleAttr);
            initAttrs(context, attrs);
        }

        private void initAttrs(Context context, @Nullable AttributeSet attrs) {
            setLayerType(LAYER_TYPE_SOFTWARE, null);
            paint = new Paint(Paint.ANTI_ALIAS_FLAG);
            paint.setStyle(Paint.Style.FILL);
            isLtr = getResources().getConfiguration().getLayoutDirection() == View.LAYOUT_DIRECTION_LTR;
            if (attrs != null) {
                TypedArray typedArray = context.obtainStyledAttributes(attrs, R.styleable.RoundShadowImageView);
                shadowRatio = typedArray.getFloat(R.styleable.RoundShadowImageView_shadowRatio, shadowRatio);
                shadowCircleAngle = typedArray.getFloat(R.styleable.RoundShadowImageView_shadowCircleAngle, shadowCircleAngle);
                if (shadowCircleAngle > 0F) {
                    useShadowCircleAngle = true;
                }
                if (!useShadowCircleAngle) {
                    shadowPosition = typedArray.getInt(R.styleable.RoundShadowImageView_shadowPosition, shadowPosition);
                }
                shadowColor = typedArray.getColor(R.styleable.RoundShadowImageView_shadowColor, shadowColor);
                gainRGB();
                shadowStartAlpha = typedArray.getFloat(R.styleable.RoundShadowImageView_shadowStartAlpha, shadowStartAlpha);
                typedArray.recycle();
            }
        }

        private void gainRGB() {
            red = Color.red(shadowColor);
            green = Color.green(shadowColor);
            blue = Color.blue(shadowColor);
        }

        private void gainShadowCenterAndShader() {
            gainShadowCenter();
            gainShader();
        }

        private void gainShadowCenter() {
            shadowRadius = contentSize / 2F;
            if (useShadowCircleAngle) {
                double radians = Math.toRadians(shadowCircleAngle + 90);
                shadowCenterX = (float) (getWidth() / 2 + Math.cos(radians) * shadowRadius * shadowRatio);
                shadowCenterY = (float) (getHeight() / 2 + Math.sin(radians) * shadowRadius * shadowRatio);
            } else {
                switch (shadowPosition) {
                    case ShadowPosition.START:
                        if (isLtr) {
                            shadowCenterX = getWidth() / 2 - shadowRadius * shadowRatio;
                        } else {
                            shadowCenterX = getWidth() / 2 + shadowRadius * shadowRatio;
                        }
                        shadowCenterY = getHeight() / 2;
                        break;
                    case ShadowPosition.TOP:
                        shadowCenterY = getHeight() / 2 - shadowRadius * shadowRatio;
                        shadowCenterX = getWidth() / 2;
                        break;
                    case ShadowPosition.END:
                        if (isLtr) {
                            shadowCenterX = getWidth() / 2 + shadowRadius * shadowRatio;
                        } else {
                            shadowCenterX = getWidth() / 2 - shadowRadius * shadowRatio;
                        }
                        shadowCenterY = getHeight() / 2;
                        break;
                    case ShadowPosition.BOTTOM:
                        shadowCenterY = getHeight() / 2 + shadowRadius * shadowRatio;
                        shadowCenterX = getWidth() / 2;
                        break;
                    default:
                        shadowCenterY = getHeight() / 2 + shadowRadius * shadowRatio;
                        shadowCenterX = getWidth() / 2;
                        break;
                }
            }
        }

        private void gainShader() {
            colors = new int[]{
                    Color.TRANSPARENT,
                    Color.argb((int) (shadowStartAlpha * 255), red, green, blue),
                    Color.argb((int) (shadowStartAlpha * 255 / 2), red, green, blue),
                    Color.argb(0, red, green, blue)
            };
            stops = new float[]{
                    (1F - shadowRatio) * 0.95F,
                    1F - shadowRatio,
                    1F - shadowRatio * 0.50F,
                    1F
            };
            shader = new RadialGradient(shadowCenterX, shadowCenterY, shadowRadius, colors, stops, Shader.TileMode.CLAMP);
        }

        private void contentSizeChanged() {
            contentSize = Math.min(getWidth(), getHeight()) / (1 + this.shadowRatio);
            setPadding((int) (getWidth() - contentSize) / 2, (int) (getHeight() - contentSize) / 2, (int) (getWidth() - contentSize) / 2, (int) (getHeight() - contentSize) / 2);
            gainShadowCenterAndShader();
        }

        @Override
        protected void onSizeChanged(int w, int h, int oldw, int oldh) {
            super.onSizeChanged(w, h, oldw, oldh);
            contentSizeChanged();
        }

        public void setShadowRatio(@FloatRange(from = 0.0F, to = 1.0F) float shadowRatio) {
            shadowRatio = shadowRatio % 1F;
            if (shadowRatio != this.shadowRatio) {
                this.shadowRatio = shadowRatio;
                contentSizeChanged();
                invalidate();
            }
        }

        public void setShadowColor(@ColorInt int shadowColor) {
            if (shadowColor != this.shadowColor) {
                this.shadowColor = shadowColor;
                gainRGB();
                gainShader();
                invalidate();
            }
        }

        public void setShadowStartAlpha(@FloatRange(from = 0F, to = 1F) float shadowStartAlpha) {
            shadowStartAlpha = shadowStartAlpha % 1F;
            if (shadowStartAlpha != this.shadowStartAlpha) {
                this.shadowStartAlpha = shadowStartAlpha;
                gainShader();
                invalidate();
            }
        }

        public void setShadowCircleAngle(float shadowCircleAngle) {
            shadowCircleAngle = Math.abs(shadowCircleAngle) % 360.0F;
            if (shadowCircleAngle != this.shadowCircleAngle) {
                this.shadowCircleAngle = shadowCircleAngle;
                if (this.shadowCircleAngle > 0F) {
                    useShadowCircleAngle = true;
                }
                gainShadowCenterAndShader();
                invalidate();
            }
        }

        public void setShadowPosition(@ShadowPosition int shadowPosition){
            if(useShadowCircleAngle || shadowPosition != this.shadowPosition){
                useShadowCircleAngle = false;
                this.shadowPosition = shadowPosition;
                gainShadowCenterAndShader();
                invalidate();
            }
        }

        public float getShadowRatio() {
            return shadowRatio;
        }

        public float getShadowCircleAngle() {
            return shadowCircleAngle;
        }

        public int getShadowColor() {
            return shadowColor;
        }

        public float getShadowStartAlpha() {
            return shadowStartAlpha;
        }

        public int getShadowPosition() {
            return shadowPosition;
        }

        @Override
        protected void onDraw(Canvas canvas) {
            paint.setShader(shader);
            canvas.drawCircle(shadowCenterX, shadowCenterY, shadowRadius, paint);
            paint.setShader(null);
            super.onDraw(canvas);
        }

        @Override
        protected void onConfigurationChanged(Configuration newConfig) {
            super.onConfigurationChanged(newConfig);
            boolean newLtr = getResources().getConfiguration().getLayoutDirection() == View.LAYOUT_DIRECTION_LTR;
            if (newLtr != isLtr) {
                this.isLtr = newLtr;
                gainShadowCenterAndShader();
                invalidate();
            }
        }
    }
    ```
8. 使用
    ```xml
    <?xml version="1.0" encoding="utf-8"?>
    <ScrollView xmlns:android="http://schemas.android.com/apk/res/android"
        xmlns:app="http://schemas.android.com/apk/res-auto"
        xmlns:tools="http://schemas.android.com/tools"
        android:layout_width="match_parent"
        android:layout_height="match_parent"
        tools:context=".othertest.shadow.ShadowActivity"
        android:fillViewport="true"
        >
        <LinearLayout
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:orientation="vertical"
            android:padding="30dp"
            android:background="@android:color/white"
            >
            <com.huanhailiuxin.jet2020.othertest.shadow.RoundShadowImageView
                android:id="@+id/shadowImageView"
                android:layout_width="700px"
                android:layout_height="800px"
                android:src="@drawable/ic_smile"
                app:shadowColor="#FFCD00"
                app:shadowStartAlpha="0.7"
                />
            <androidx.appcompat.widget.AppCompatButton
                android:layout_width="match_parent"
                android:layout_height="wrap_content"
                android:layout_marginTop="20dp"
                android:text="调整阴影位置"
                android:onClick="modifyShadowPosition"
                />
            <androidx.appcompat.widget.AppCompatButton
                android:layout_width="match_parent"
                android:layout_height="wrap_content"
                android:layout_marginTop="20dp"
                android:text="旋转阴影"
                android:onClick="rotateTheShadow"
                />
            <androidx.appcompat.widget.AppCompatButton
                android:layout_width="match_parent"
                android:layout_height="wrap_content"
                android:layout_marginTop="20dp"
                android:text="调整阴影尺寸比例"
                android:onClick="modifyShadowRatio"
                />
            <androidx.appcompat.widget.AppCompatButton
                android:layout_width="match_parent"
                android:layout_height="wrap_content"
                android:layout_marginTop="20dp"
                android:text="调整阴影起始透明度"
                android:onClick="modifyShadowStartAlpha"
                />
        </LinearLayout>
    </ScrollView>
    ```
    ```java
    public class ShadowActivity extends BaseActivity {
        private RoundShadowImageView shadowImageView;
        ObjectAnimator animator;
        private float shadowCircleAngle,shadowRatio,shadowStartAlpha;
        private @ShadowPosition int shadowPosition;
        @Override
        protected void onCreate(Bundle savedInstanceState) {
            super.onCreate(savedInstanceState);
            setContentView(R.layout.activity_shadow);
            setTitle("试验阴影");
            shadowImageView = findViewById(R.id.shadowImageView);
            shadowCircleAngle = shadowImageView.getShadowCircleAngle();
            shadowRatio = shadowImageView.getShadowRatio();
            shadowStartAlpha = shadowImageView.getShadowStartAlpha();
            shadowPosition = shadowImageView.getShadowPosition();
        }
        private void reset(){
            shadowImageView.setShadowPosition(shadowPosition);
            shadowImageView.setShadowCircleAngle(shadowCircleAngle);
            shadowImageView.setShadowRatio(shadowRatio);
            shadowImageView.setShadowStartAlpha(shadowStartAlpha);
        }
        private void initAnimator(){
            animator.setRepeatCount(ValueAnimator.INFINITE);
            animator.setRepeatMode(ObjectAnimator.RESTART);
            animator.setDuration(4000);
            animator.setInterpolator(new LinearInterpolator());
        }
        private void initAngleAnimator(){
            if(animator != null){
               animator.cancel();
            }
            animator = ObjectAnimator.ofFloat(shadowImageView,"shadowCircleAngle",0F,360F);
            initAnimator();
        }
        public void rotateTheShadow(View view) {
            reset();
            initAngleAnimator();
            animator.start();
        }
        private void initShadowRatioAnimator(){
            if(animator != null){
                animator.cancel();
            }
            animator = ObjectAnimator.ofFloat(shadowImageView,"shadowRatio",0.1F,0.5F);
            initAnimator();
        }
        public void modifyShadowRatio(View view) {
            reset();
            initShadowRatioAnimator();
            animator.start();
        }
        private void initShadowStartAlphaAnimator(){
            if(animator != null){
                animator.cancel();
            }
            animator = ObjectAnimator.ofFloat(shadowImageView,"shadowStartAlpha",0.1F,0.9F);
            initAnimator();
        }
        public void modifyShadowStartAlpha(View view) {
            reset();
            initShadowStartAlphaAnimator();
            animator.start();
        }

        private int shadowPositionIndex = 0;
        private @ShadowPosition int[] shadowPositions = new int[]{
                ShadowPosition.START,
                ShadowPosition.TOP,
                ShadowPosition.END,
                ShadowPosition.BOTTOM
        };
        public void modifyShadowPosition(View view) {
            if(animator != null){
                animator.cancel();
            }
            reset();
            int index = shadowPositionIndex ++ % shadowPositions.length;
            shadowImageView.setShadowPosition(shadowPositions[index]);
        }
    }
    ```

	![](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/657e03485659486a94b57497c6627c0c~tplv-k3u1fbpfcp-watermark.image)


## 2.轮廓
### 相关文章
1. [ViewOutlineProvider轮廓裁剪(5.0以上特性)](https://blog.csdn.net/K_Hello/article/details/87370304)
2. [自定义 View：用贝塞尔曲线绘制酷炫轮廓背景](https://mp.weixin.qq.com/s/SzZuiRMz8QWzNqCjq2gI_A)
3. [Android使用ViewOutlineProvider实现圆角](https://www.jianshu.com/p/686cf24580c2)

### 知识点
1. 源码解析:
    - 要想通过ViewOutlineProvider生成的Outline绘制阴影,需要调用Outline.setRoundRect, Outline.setOval, Outline.setConvexPath , set(@NonNull Outline src) ,改变 mMode 的默认值(MODE_EMPTY)
    - 要想通过ViewOutlineProvider生成的Outline实现View的轮廓裁剪, Outline需要最终调用 setRoundRect.
    - View的默认ViewOutlineProvider生成的的Outline可以实现轮廓裁剪.
        ```java
        View.java
        //View默认的ViewOutlineProvider是ViewOutlineProvider.BACKGROUND
        ViewOutlineProvider mOutlineProvider = ViewOutlineProvider.BACKGROUND;
        /**
         * Sets the {@link ViewOutlineProvider} of the view, which generates the Outline that defines
         * the shape of the shadow it casts, and enables outline clipping.
         * <p>
         * The default ViewOutlineProvider, {@link ViewOutlineProvider#BACKGROUND}, queries the Outline
         * from the View's background drawable, via {@link Drawable#getOutline(Outline)}. Changing the
         * outline provider with this method allows this behavior to be overridden.
         * <p>
         * If the ViewOutlineProvider is null, if querying it for an outline returns false,
         * or if the produced Outline is {@link Outline#isEmpty()}, shadows will not be cast.
         * <p>
         * Only outlines that return true from {@link Outline#canClip()} may be used for clipping.
         *
         * @see #setClipToOutline(boolean)
         * @see #getClipToOutline()
         * @see #getOutlineProvider()
         */
        //ViewOutlineProvider用于View的轮廓裁切,并生成定义View阴影形状的Outline实例.
        //当生成的Outline实例的isEmpty为true,则不能为View绘制阴影.
        //当生成的Outline实例的canClip为false,则不能为View进行轮廓裁切.
        public void setOutlineProvider(ViewOutlineProvider provider) {
            mOutlineProvider = provider;
            invalidateOutline();
        }
        /**
         * Sets whether the View's Outline should be used to clip the contents of the View.
         * <p>
         * Only a single non-rectangular clip can be applied on a View at any time.
         * Circular clips from a {@link ViewAnimationUtils#createCircularReveal(View, int, int, float, float)
         * circular reveal} animation take priority over Outline clipping, and
         * child Outline clipping takes priority over Outline clipping done by a
         * parent.
         * <p>
         * Note that this flag will only be respected if the View's Outline returns true from
         * {@link Outline#canClip()}.
         *
         * @see #setOutlineProvider(ViewOutlineProvider)
         * @see #getClipToOutline()
         */
        //设置是否使用View关联的Outline来进行轮廓裁剪.
        //即使clipToOutline为true,也需要Outline.canClip()fanhuitrue,该方法才会生效.
        //即生效条件: clipToOutline==true 且 Outline最终调用过setRoundRect.
        public void setClipToOutline(boolean clipToOutline) {
            damageInParent();
            if (getClipToOutline() != clipToOutline) {
                mRenderNode.setClipToOutline(clipToOutline);
            }
        }

        ViewOutlineProvider.java
        //View默认的ViewOutlineProvider,最总调用了setRoundRect
        1:
        Drawable background.getOutline(outline);
        -->
        Drawable.java
        public void getOutline(@NonNull Outline outline) {
            outline.setRect(getBounds());
            outline.setAlpha(0);
        }
        -->
        Outline.java
        public void setRect(@NonNull Rect rect) {
            setRect(rect.left, rect.top, rect.right, rect.bottom);
        }
        public void setRect(int left, int top, int right, int bottom) {
            //最终调用了setRoundRect
            setRoundRect(left, top, right, bottom, 0.0f);
        }
        2:
        outline.setRect(0, 0, view.getWidth(), view.getHeight());
        -->
        setRoundRect
        public static final ViewOutlineProvider BACKGROUND = new ViewOutlineProvider() {
            @Override
            public void getOutline(View view, Outline outline) {
                Drawable background = view.getBackground();
                if (background != null) {
                    background.getOutline(outline);
                } else {
                    outline.setRect(0, 0, view.getWidth(), view.getHeight());
                    outline.setAlpha(0.0f);
                }
            }
        };

        Outline.java
        @Mode
        //mMode 默认就是 MODE_EMPTY
        public int mMode = MODE_EMPTY;
        /**
         * Returns whether the Outline is empty.
         * <p>
         * Outlines are empty when constructed, or if {@link #setEmpty()} is called,
         * until a setter method is called
         *
         * @see #setEmpty()
         */
        //当 mMode 是 MODE_EMPTY.返回true.
        public boolean isEmpty() {
            return mMode == MODE_EMPTY;
        }
        /**
         * Returns whether the outline can be used to clip a View.
         * <p>
         * Currently, only Outlines that can be represented as a rectangle, circle,
         * or round rect support clipping.
         *
         * @see android.view.View#setClipToOutline(boolean)
         */
        //当 mMode 不是 MODE_CONVEX_PATH.返回true.
        public boolean canClip() {
            return mMode != MODE_CONVEX_PATH;
        }
        //看 mMode 如何变化:
        setEmpty:
        -> mMode = MODE_EMPTY;

        set(@NonNull Outline src):
        -> mMode = src.mMode;

        setRoundRect(int left, int top, int right, int bottom, float radius):
        -> mMode = MODE_ROUND_RECT;

        setOval(int left, int top, int right, int bottom):
        -> mMode = MODE_CONVEX_PATH;

        setConvexPath(@NonNull Path convexPath):
        -> mMode = MODE_CONVEX_PATH;
        ```

### 本地验证
