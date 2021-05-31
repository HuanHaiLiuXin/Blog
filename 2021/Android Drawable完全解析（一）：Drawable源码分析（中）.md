[Android Drawable完全解析（一）：Drawable源码分析（上）](http://www.jianshu.com/p/384a70897ba6)
<br>
[Android Drawable完全解析（一）：Drawable源码分析（中）](http://www.jianshu.com/p/2213c62e4738)
<br>
[Android Drawable完全解析（一）：Drawable源码分析（下）](http://www.jianshu.com/p/c56b762210f2)

呃...我不是故意要凑篇幅写个什么上下篇，实在是因为Drawable源码有点长，一篇写不下啦O(∩_∩)O~

鉴于源码一般较长，以后所有源码分析的部分，英文注释非必要情况都不再保留！
#### 2:Drawable源码分析/翻译
**继续上Drawable源码:**
```
package android.graphics.drawable;

public abstract class Drawable {
    ****
    略
    ****

    /**
    这个方法很重要，故保留英文注释！
    调用mutate()，使当前Drawable实例mutable，这个操作不可逆。
    一个mutable的Drawable实例不会和其他Drawable实例共享它的状态。
    当你需要修改一个从资源文件加载的Drawable实例时，mutate()方法尤其有用。
    默认情况下，所有加载同一资源文件生成的Drawable实例都共享一个通用的状态，
    如果你修改了其中一个Drawable实例，所有的相关Drawable实例都会发生同样的变化。

    这个方法在[其实你不懂：Drawable着色(tint)的兼容方案 源码解析]
    这篇文章里有过介绍，就是为了限定Drawable实例的编辑生效范围仅限于自身。
     * Make this drawable mutable. This operation cannot be reversed. A mutable
     * drawable is guaranteed to not share its state with any other drawable.
     * This is especially useful when you need to modify properties of drawables
     * loaded from resources. By default, all drawables instances loaded from
     * the same resource share a common state; if you modify the state of one
     * instance, all the other instances will receive the same modification.
     *
     * Calling this method on a mutable Drawable will have no effect.
     *
     * @return This drawable.
     * @see ConstantState
     * @see #getConstantState()
     */
    public @NonNull Drawable mutate() {
        return this;
    }

    /**
    被隐匿
     * @hide
     */
    public void clearMutated() {
        // Default implementation is no-op.
    }

    //下面几个方法介绍了通过不同的方式创建Drawable实例：
    //流、XML、文件地址
    public static Drawable createFromStream(InputStream is, String srcName) {
        Trace.traceBegin(Trace.TRACE_TAG_RESOURCES, srcName != null ? srcName : "Unknown drawable");
        try {
            return createFromResourceStream(null, null, is, srcName);
        } finally {
            Trace.traceEnd(Trace.TRACE_TAG_RESOURCES);
        }
    }
    public static Drawable createFromResourceStream(Resources res, TypedValue value,
            InputStream is, String srcName) {
        Trace.traceBegin(Trace.TRACE_TAG_RESOURCES, srcName != null ? srcName : "Unknown drawable");
        try {
            return createFromResourceStream(res, value, is, srcName, null);
        } finally {
            Trace.traceEnd(Trace.TRACE_TAG_RESOURCES);
        }
    }
    public static Drawable createFromResourceStream(Resources res, TypedValue value,
            InputStream is, String srcName, BitmapFactory.Options opts) {
        if (is == null) {
            return null;
        }
        Rect pad = new Rect();
        if (opts == null) opts = new BitmapFactory.Options();
        opts.inScreenDensity = Drawable.resolveDensity(res, 0);
        Bitmap  bm = BitmapFactory.decodeResourceStream(res, value, is, pad, opts);
        if (bm != null) {
            byte[] np = bm.getNinePatchChunk();
            if (np == null || !NinePatch.isNinePatchChunk(np)) {
                np = null;
                pad = null;
            }
            final Rect opticalInsets = new Rect();
            bm.getOpticalInsets(opticalInsets);
            return drawableFromBitmap(res, bm, np, pad, opticalInsets, srcName);
        }
        return null;
    }
    public static Drawable createFromXml(Resources r, XmlPullParser parser)
            throws XmlPullParserException, IOException {
        return createFromXml(r, parser, null);
    }
    public static Drawable createFromXml(Resources r, XmlPullParser parser, Theme theme)
            throws XmlPullParserException, IOException {
        AttributeSet attrs = Xml.asAttributeSet(parser);
        int type;
        //noinspection StatementWithEmptyBody
        while ((type=parser.next()) != XmlPullParser.START_TAG
                && type != XmlPullParser.END_DOCUMENT) {
            // Empty loop.
        }
        if (type != XmlPullParser.START_TAG) {
            throw new XmlPullParserException("No start tag found");
        }
        Drawable drawable = createFromXmlInner(r, parser, attrs, theme);
        if (drawable == null) {
            throw new RuntimeException("Unknown initial tag: " + parser.getName());
        }
        return drawable;
    }
    public static Drawable createFromXmlInner(Resources r, XmlPullParser parser, AttributeSet attrs)
            throws XmlPullParserException, IOException {
        return createFromXmlInner(r, parser, attrs, null);
    }
    public static Drawable createFromXmlInner(Resources r, XmlPullParser parser, AttributeSet attrs,
            Theme theme) throws XmlPullParserException, IOException {
        return r.getDrawableInflater().inflateFromXml(parser.getName(), parser, attrs, theme);
    }
    public static Drawable createFromPath(String pathName) {
        if (pathName == null) {
            return null;
        }
        Trace.traceBegin(Trace.TRACE_TAG_RESOURCES, pathName);
        try {
            Bitmap bm = BitmapFactory.decodeFile(pathName);
            if (bm != null) {
                return drawableFromBitmap(null, bm, null, null, null, pathName);
            }
        } finally {
            Trace.traceEnd(Trace.TRACE_TAG_RESOURCES);
        }
        return null;
    }
    public void inflate(@NonNull Resources r, @NonNull XmlPullParser parser,
            @NonNull AttributeSet attrs) throws XmlPullParserException, IOException {
        inflate(r, parser, attrs, null);
    }

    /**
    从XML文件中加载Drawable实例，Drawable实例接受主题设置的风格
     */
    public void inflate(@NonNull Resources r, @NonNull XmlPullParser parser,
            @NonNull AttributeSet attrs, @Nullable Theme theme)
            throws XmlPullParserException, IOException {
        final TypedArray a = obtainAttributes(r, theme, attrs, R.styleable.Drawable);
        mVisible = a.getBoolean(R.styleable.Drawable_visible, mVisible);
        a.recycle();
    }
    /**
    从XML文件中加载Drawable实例
     */
    void inflateWithAttributes(@NonNull @SuppressWarnings("unused") Resources r,
            @NonNull @SuppressWarnings("unused") XmlPullParser parser, @NonNull TypedArray attrs,
            @AttrRes int visibleAttr) throws XmlPullParserException, IOException {
        mVisible = attrs.getBoolean(visibleAttr, mVisible);
    }

    /**
    这段注释很重要，故保留英文注释！

    ConstantState这个抽象类被用于存储 多个Drawable实例间 共享的 常量状态值及数据。
    如从同一个图片资源创建的多个BitmapDrawable实例，它们将共享
    同一个存储在它们的ConstantState中的Bitmap。
     * This abstract class is used by {@link Drawable}s to store shared constant state and data
     * between Drawables. {@link BitmapDrawable}s created from the same resource will for instance
     * share a unique bitmap stored in their ConstantState.
     *
    newDrawable可以运用ConstantState创建一个新的Drawable实例
     * <p>
     * {@link #newDrawable(Resources)} can be used as a factory to create new Drawable instances
     * from this ConstantState.
     * </p>
     *
    Drawable#getConstantState可以获取一个Drawable关联的ConstantState。
    调用Drawable#mutate()，则将为新创建的Drawable实例单独关联一个ConstantState。
     * Use {@link Drawable#getConstantState()} to retrieve the ConstantState of a Drawable. Calling
     * {@link Drawable#mutate()} on a Drawable should typically create a new ConstantState for that
     * Drawable.
     */
    public static abstract class ConstantState {
        /**
        运用ConstantState创建一个新的Drawable实例
         */
        public abstract @NonNull Drawable newDrawable();
        /**
         运用ConstantState创建一个新的Drawable实例
         */
        public @NonNull Drawable newDrawable(@Nullable Resources res) {
            return newDrawable();
        }
        /**
        运用ConstantState创建一个新的Drawable实例
         */
        public @NonNull Drawable newDrawable(@Nullable Resources res,
                @Nullable @SuppressWarnings("unused") Theme theme) {
            return newDrawable(res);
        }
        /**
        返回会影响Drawable实例的一个bit掩码变化设置
         */
        public abstract @Config int getChangingConfigurations();
        /**
        返回所有的像素数
        public int addAtlasableBitmaps(@NonNull Collection<Bitmap> atlasList) {
            return 0;
        }
        /** @hide */
        protected final boolean isAtlasable(@Nullable Bitmap bitmap) {
            return bitmap != null && bitmap.getConfig() == Bitmap.Config.ARGB_8888;
        }
        /**
        返回当前共享状态是否可以设置主题
         */
        public boolean canApplyTheme() {
            return false;
        }
    }

    /**
    返回当前Drawable的用于存储共享状态值的ConstantState实例
     */
    public @Nullable ConstantState getConstantState() {
        return null;
    }
    //通过Bitmap实例创建Drawable实例
    private static Drawable drawableFromBitmap(Resources res, Bitmap bm, byte[] np,
            Rect pad, Rect layoutBounds, String srcName) {
        if (np != null) {
            return new NinePatchDrawable(res, bm, np, pad, layoutBounds, srcName);
        }
        return new BitmapDrawable(res, bm);
    }

    /**
    确保色彩过滤器和当前色彩与色彩模式一致
     */
    @Nullable PorterDuffColorFilter updateTintFilter(@Nullable PorterDuffColorFilter tintFilter,
            @Nullable ColorStateList tint, @Nullable PorterDuff.Mode tintMode) {
        if (tint == null || tintMode == null) {
            return null;
        }
        final int color = tint.getColorForState(getState(), Color.TRANSPARENT);
        if (tintFilter == null) {
            return new PorterDuffColorFilter(color, tintMode);
        }
        tintFilter.setColor(color);
        tintFilter.setMode(tintMode);
        return tintFilter;
    }

    /**
    如果主题有效，则从中获取样式属性，
    如果主题无效，则返回没有样式的资源。
     */
    static @NonNull TypedArray obtainAttributes(@NonNull Resources res, @Nullable Theme theme,
            @NonNull AttributeSet set, @NonNull int[] attrs) {
        if (theme == null) {
            return res.obtainAttributes(set, attrs);
        }
        return theme.obtainStyledAttributes(set, attrs, 0, 0);
    }

    /**
    根据 原始像素值，资源单位密度和目标设备单位密度 获得一个float像素值
     */
    static float scaleFromDensity(float pixels, int sourceDensity, int targetDensity) {
        return pixels * targetDensity / sourceDensity;
    }
    static int scaleFromDensity(
            int pixels, int sourceDensity, int targetDensity, boolean isSize) {
        if (pixels == 0 || sourceDensity == targetDensity) {
            return pixels;
        }
        final float result = pixels * targetDensity / (float) sourceDensity;
        if (!isSize) {
            return (int) result;
        }
        final int rounded = Math.round(result);
        if (rounded != 0) {
            return rounded;
        } else if (pixels > 0) {
            return 1;
        } else {
            return -1;
        }
    }

    //获取单位密度
    static int resolveDensity(@Nullable Resources r, int parentDensity) {
        final int densityDpi = r == null ? parentDensity : r.getDisplayMetrics().densityDpi;
        return densityDpi == 0 ? DisplayMetrics.DENSITY_DEFAULT : densityDpi;
    }
    static void rethrowAsRuntimeException(@NonNull Exception cause) throws RuntimeException {
        final RuntimeException e = new RuntimeException(cause);
        e.setStackTrace(new StackTraceElement[0]);
        throw e;
    }

    /**
   通过解析tintMode属性枚举值获得一个PorterDuff.Mode
   
   被隐匿
     * @hide
     */
    public static PorterDuff.Mode parseTintMode(int value, Mode defaultMode) {
        switch (value) {
            case 3: return Mode.SRC_OVER;
            case 5: return Mode.SRC_IN;
            case 9: return Mode.SRC_ATOP;
            case 14: return Mode.MULTIPLY;
            case 15: return Mode.SCREEN;
            case 16: return Mode.ADD;
            default: return defaultMode;
        }
    }
}
```
Drawable类本身源码先写到这儿，接着往下看。

#### 3:Drawable绘制流程
看过Drawable源码，其实我们还是不清楚:
**Drawable实例到底是如何被绘制到屏幕上面？
Drawable源码中的那些方法又是什么时候被谁调用的？**

我们回想一下，使用Drawable最通常的步骤：
**通过Resource获取Drawable实例**
**将获取的Drawable实例当做背景设置给View或者作为ImageView的src进行显示:**

下面就逐步分析理解Drawable的绘制流程。
###### 3.1:通过Resource获取Drawable实例
最常用写法：getResources().getDrawable(int id)，看下关键代码：
```
public class Resources {
    ****
    public Drawable getDrawable(@DrawableRes int id) throws NotFoundException {
        final Drawable d = getDrawable(id, null);
        *****
        return d;
    }
    public Drawable getDrawable(@DrawableRes int id, @Nullable Theme theme)
            throws NotFoundException {
        final TypedValue value = obtainTempTypedValue();
        try {
            final ResourcesImpl impl = mResourcesImpl;
            //
            impl.getValue(id, value, true);
            //将获取到的Drawable实例返回
            return impl.loadDrawable(this, value, id, theme, true);
        } ****
    }
}
一路追踪下去：
public class ResourcesImpl {
    //Resource实例，TypedValue，资源ID，Theme实例，true
    @Nullable
    Drawable loadDrawable(Resources wrapper, TypedValue value, int id, Resources.Theme theme,
            boolean useCache) throws NotFoundException {
        try {
            ********
            //是否属于ColorDrawable
            final boolean isColorDrawable;
            //Drawable缓存
            final DrawableCache caches;
            final long key;
            //判断资源是否属于颜色资源
            if (value.type >= TypedValue.TYPE_FIRST_COLOR_INT
                    && value.type <= TypedValue.TYPE_LAST_COLOR_INT) {
                isColorDrawable = true;
                caches = mColorDrawableCache;
                key = value.data;
            } else {
                //如果是加载一张普通的图片，不属于颜色资源
                isColorDrawable = false;
                caches = mDrawableCache;
                key = (((long) value.assetCookie) << 32) | value.data;
            }
            if (!mPreloading && useCache) {
                final Drawable cachedDrawable = caches.getInstance(key, wrapper, theme);
                if (cachedDrawable != null) {
                    return cachedDrawable;
                }
            }
            //如果在Drawable缓存里面未找到资源ID对应的Drawable实例，继续
            final Drawable.ConstantState cs;
            if (isColorDrawable) {
                cs = sPreloadedColorDrawables.get(key);
            } else {
                //如果不属于颜色资源，则从sPreloadedDrawables中查询
                //sPreloadedDrawables只有在执行cacheDrawable方法时
                //才会进行数据添加：而第一次加载图片时候还未执行cacheDrawable
                //所以此时cs = null.
                cs = sPreloadedDrawables[mConfiguration.getLayoutDirection()].get(key);
            }
            Drawable dr;
            if (cs != null) {
                dr = cs.newDrawable(wrapper);
            } else if (isColorDrawable) {
                dr = new ColorDrawable(value.data);
            } else {
                //当第一次加载图片资源时候，cs=null且不属于颜色资源，
                //实际是通过loadDrawableForCookie来获取Drawable实例
                dr = loadDrawableForCookie(wrapper, value, id, null);
            }
            *********
    }
    private Drawable loadDrawableForCookie(Resources wrapper, TypedValue value, int id,
            Resources.Theme theme) {
        ****
        final String file = value.string.toString();
        ****
        final Drawable dr;
        Trace.traceBegin(Trace.TRACE_TAG_RESOURCES, file);
        try {
            if (file.endsWith(".xml")) {
                //如果是从xml文件加载Drawable
                final XmlResourceParser rp = loadXmlResourceParser(
                        file, id, value.assetCookie, "drawable");
                dr = Drawable.createFromXml(wrapper, rp, theme);
                rp.close();
            } else {
                //从图片资源加载Drawable，执行Drawable.createFromResourceStream
                //获取Drawable实例
                final InputStream is = mAssets.openNonAsset(
                        value.assetCookie, file, AssetManager.ACCESS_STREAMING);
                dr = Drawable.createFromResourceStream(wrapper, value, is, file, null);
                is.close();
            }
        } catch (Exception e) {
            ****
        }
        ****
        return dr;
    }
}
一路追踪下去：
public abstract class Drawable {
    public static Drawable createFromResourceStream(Resources res, TypedValue value,
            InputStream is, String srcName, BitmapFactory.Options opts) {
        ****
            return drawableFromBitmap(res, bm, np, pad, opticalInsets, srcName);
        ****
    }
    private static Drawable drawableFromBitmap(Resources res, Bitmap bm, byte[] np,
            Rect pad, Rect layoutBounds, String srcName) {
        if (np != null) {
            //如果加载的图片资源是.9 PNG，返回NinePatchDrawable实例
            return new NinePatchDrawable(res, bm, np, pad, layoutBounds, srcName);
        }
        //对于普通图片资源，返回BitmapDrawable
        return new BitmapDrawable(res, bm);
    }
}
```
**由此可见，通过Resource实例加载一张资源图片：
.9图返回1个NinePatchDrawable实例；
普通图片返回1个BitmapDrawable实例。**
###### 3.2:将获取的Drawable实例当做背景设置给View
最常用写法：targetView.setBackgroundDrawable(Drawable bg),
同样看一下关键代码
```
public class View implements Drawable.Callback, KeyEvent.Callback,
        AccessibilityEventSource {
    ****
    public void setBackgroundDrawable(Drawable background) {
        ****
        if (background == mBackground) {
            //如果当前背景和background相同，直接return
            return;
        }
        boolean requestLayout = false;
        mBackgroundResource = 0;
        if (mBackground != null) {
            if (isAttachedToWindow()) {
                //如果当前View实例已经被绘制到屏幕上，则首先取消
                //该View实例原始背景Drawable的动画
                mBackground.setVisible(false, false);
            }
            //移除该View实例原始背景Drawable的动画监听接口
            mBackground.setCallback(null);
            //取消该View实例原始背景Drawable的所有事件
            unscheduleDrawable(mBackground);
        }
        if (background != null) {
            ****
            //设置background的布局方向和View实例一致，
            //Drawable.setLayoutDirection见上一篇文章
            background.setLayoutDirection(getLayoutDirection());
            if (background.getPadding(padding)) {
                //如果Drawable实例background有padding
                resetResolvedPaddingInternal();
                switch (background.getLayoutDirection()) {
                    case LAYOUT_DIRECTION_RTL:
                        //布局方向从右至左
                        mUserPaddingLeftInitial = padding.right;
                        mUserPaddingRightInitial = padding.left;
                        internalSetPadding(padding.right, padding.top, padding.left, padding.bottom);
                        break;
                    case LAYOUT_DIRECTION_LTR:
                    default:
                        //布局方向从左至右
                        mUserPaddingLeftInitial = padding.left;
                        mUserPaddingRightInitial = padding.right;
                        //internalSetPadding会将四个参数值和View实例的padding进行比对，若不同则会重新布局+重建View的外部轮廓
                        internalSetPadding(padding.left, padding.top, padding.right, padding.bottom);
                }
                mLeftPaddingDefined = false;
                mRightPaddingDefined = false;
            }
            if (mBackground == null
                    || mBackground.getMinimumHeight() != background.getMinimumHeight()
                    || mBackground.getMinimumWidth() != background.getMinimumWidth()) {
                requestLayout = true;
            }
            //设置当前View实例的背景为传入的Drawable实例 background
            mBackground = background;
            if (background.isStateful()) {
                //如果background会根据状态值变更外观，则设置其状态为
                //当前View实例的state
                background.setState(getDrawableState());
            }
            if (isAttachedToWindow()) {
                //如果当前View实例已经被绘制到屏幕上
                //且实例和实例的父控件及递归获得的根布局都处于可见状态，
                //则设置background开启动画效果
                background.setVisible(getWindowVisibility() == VISIBLE && isShown(), false);
            }
            applyBackgroundTint();
            //设置background动画接口监听为View实例本身（View实现了 Drawable.Callback）：
            //public class View implements Drawable.Callback
            background.setCallback(this);
            if ((mPrivateFlags & PFLAG_SKIP_DRAW) != 0) {
                mPrivateFlags &= ~PFLAG_SKIP_DRAW;
                //需要重新布局
                requestLayout = true;
            }
        } else {
            mBackground = null;
            if ((mViewFlags & WILL_NOT_DRAW) != 0
                    && (mForegroundInfo == null || mForegroundInfo.mDrawable == null)) {
                mPrivateFlags |= PFLAG_SKIP_DRAW;
            }
            requestLayout = true;
        }
        computeOpaqueFlags();
        if (requestLayout) {
            //重新布局
            requestLayout();
        }
        mBackgroundSizeChanged = true;
        //重绘View实例
        invalidate(true);
        //重建View实例的外部轮廓
        invalidateOutline();
    }
}
```
由此可见，**setBackgroundDrawable方法，调用了Drawable实例的一系列方法，最终引发了View实例的重新布局（requestLayout()）重绘（invalidate(true)）及重建View实例的外部轮廓（invalidateOutline()）。**
invalidate会触发draw方法，我们继续看View.draw方法的关键代码：
```
    public void draw(Canvas canvas) {
        ****
        /*
        翻译可能不甚准确，欢迎英语好的同学留言指正O(∩_∩)O~

        Draw方法会执行以下几个步骤，且必须按顺序执行：
        1：绘制View实例的背景
        2：如有必要，保存画布图层以备褪色
        3：绘制View实例的内容
        4：绘制View实例的中的子控件
        5：如有必要，绘制边缘并恢复图层
        6：绘制滚动条
         *      1. Draw the background
         *      2. If necessary, save the canvas' layers to prepare for fading
         *      3. Draw view's content
         *      4. Draw children
         *      5. If necessary, draw the fading edges and restore layers
         *      6. Draw decorations (scrollbars for instance)
         */
        //从步骤顺序上看，和Drawable相关的就是第1步，只看第1步代码
        // Step 1, draw the background, if needed
        int saveCount;
        if (!dirtyOpaque) {
            //绘制背景
            drawBackground(canvas);
        }
        ****
    }

    一路追踪下去：

    private void drawBackground(Canvas canvas) {
        //mBackground就是之前setBackgroundDrawable传入的Drawable实例
        final Drawable background = mBackground;
        if (background == null) {
            return;
        }
        //设置background绘制范围为View实例的所在范围
        setBackgroundBounds();
        ****
        final int scrollX = mScrollX;
        final int scrollY = mScrollY;
        if ((scrollX | scrollY) == 0) {
            //最终调用Drawable.draw(Canvas canvas)将Drawable实例
            //绘制到屏幕上
            background.draw(canvas);
        } else {
            canvas.translate(scrollX, scrollY);
            //最终调用Drawable.draw(Canvas canvas)将Drawable实例
            //绘制到屏幕上
            background.draw(canvas);
            canvas.translate(-scrollX, -scrollY);
        }
    }
```
由此可见，**View实例的背景Drawable实例最终还是调用自身的Drawable.draw(@NonNull Canvas canvas)方法绘制到屏幕上。**

继续查看Drawable.draw方法：
```
public abstract class Drawable {
    //Drawable中的draw是一个抽象方法，应该是为了众多的
    //子类Drawable拥有自定义的绘制逻辑进行重写
    public abstract void draw(@NonNull Canvas canvas);
}

在分析Resource.getDrawable时候已经知道，
对于普通的图片资源，获取到的是一个BitmapDrawable实例，
我们就来看看BitmapDrawable的draw具体的绘制逻辑：

public class BitmapDrawable extends Drawable {
    @Override
    public void draw(Canvas canvas) {
        ****
        if (shader == null) {
            ****
            //最终调用了Canvas.drawBitmap方法，将Drawable实例中的bitmap绘制到View实例关联的画布上
            canvas.drawBitmap(bitmap, null, mDstRect, paint);
            if (needMirroring) {
                canvas.restore();
            }
        } ****
    }
}
```
**至此，将获取的Drawable实例当做背景设置给View，和Drawable相关的一系列逻辑就分析完了，大致如下：**
- **1：setBackgroundDrawable方法，调用了Drawable的一系列方法，设置了Drawable实例一系列属性值，最终引发了View实例的重新布局（requestLayout()），重绘（invalidate(true)）及重建View实例的外部轮廓（invalidateOutline()）**
- **2：在View实例重绘过程的第一步，将得到的Drawable实例（View实例的背景）绘制到屏幕上，实质是调用了Drawable.draw(@NonNull Canvas canvas)**
- **3：Drawable.draw本身是个抽象方法，绘制具体逻辑由其子类实现。
我们以之前获得的BitmapDrawable为例进行分析：
最终调用了Canvas.drawBitmap方法，将Drawable实例中的bitmap绘制到View实例关联的画布上**

Drawable绘制流程今天先写到这儿，现在是2017/03/07 20:41，加班码字到现在有点累了，明后天继续把ImageView和Drawable关联的部分写完吧！

未完待续...

以上就是个人分析的一点结果，若有错误，请各位同学留言告知！

**That's all !**
