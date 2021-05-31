[Android Drawable完全解析（一）：Drawable源码分析（上）](http://www.jianshu.com/p/384a70897ba6)
<br>
[Android Drawable完全解析（一）：Drawable源码分析（中）](http://www.jianshu.com/p/2213c62e4738)
<br>
[Android Drawable完全解析（一）：Drawable源码分析（下）](http://www.jianshu.com/p/c56b762210f2)

呃...我不是故意要凑篇幅写个什么上下篇，实在是因为Drawable源码有点长，一篇写不下啦O(∩_∩)O~

> 上一篇文章 [**其实你不懂：Drawable着色(tint)的兼容方案 源码解析**](http://www.jianshu.com/p/658fc7f2d4f4) 描述了Drawable的着色原理，文章里有涉及到Drawable的一些方法，顺便看一下Drawabld的源码，发现Drawable涉及的面很广，尤其是竟然有那么多的继承类。想想自己平时也就用过ColorDrawable,StateListDrawable,BitmapDrawable很有限的几个子类，对于Drawable的应用还是太零散了，所以写这个Drawable系列文章，对其做一个相对完整的分析！

这是“Drawable完全解析”系列文章的第一篇，就从Drawable的源码分析开始！
#### 1:Drawable与其子类的继承关系
直接看图，暂不详细介绍具体使用方法。
![Drawable及其子类继承关系.png](http://upload-images.jianshu.io/upload_images/3501388-37773a8badb712e8.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/600)
可以看到Drawable竟然有这么多子类，真的需要花一些时间才能分别搞得清楚。
#### 2:Drawable源码分析/翻译
此处源码是SDK 24版本下的Drawable.java文件，不同版本下应该会有出入，敬请注意！
源码较长，英文较好的同学记得指出我翻译和理解的错误！
```
package android.graphics.drawable;

import com.android.internal.R;
*****略

/**
Drawable是一个用于处理各种可绘制资源的抽象类。我们使用Drawable最常见的情况就是将获取到的资源绘制到屏幕上；Drawable类提供了一些通用的API来处理以下具有多种表现形式的可视资源/视觉资源。
 * A Drawable is a general abstraction for "something that can be drawn."  Most
 * often you will deal with Drawable as the type of resource retrieved for
 * drawing things to the screen; the Drawable class provides a generic API for
 * dealing with an underlying visual resource that may take a variety of forms.
和View不同，Drawable实例不具备任何能力接收事件或与用户交互。
 * Unlike a {@link android.view.View}, a Drawable does not have any facility to
 * receive events or otherwise interact with the user.
 *
除了简单绘图，Drawable提供了一些通用的机制使客户端与当前正在绘制的内容进行交互。
 * <p>In addition to simple drawing, Drawable provides a number of generic
 * mechanisms for its client to interact with what is being drawn:
 *

setBounds方法必须被Drawable实例调用，用于声明Drawable实例绘制的位置和大小。所有的Drawable实例都会生成请求的尺寸，这一点通常可以通过缩放图像很容易就达到。对一些Drawable实例，客户端可以通过调用getIntrinsicHeight和getIntrinsicWidth方法得到其首选尺寸。
 * <ul>
 *     <li> The {@link #setBounds} method <var>must</var> be called to tell the
 *     Drawable where it is drawn and how large it should be.  All Drawables
 *     should respect the requested size, often simply by scaling their
 *     imagery.  A client can find the preferred size for some Drawables with
 *     the {@link #getIntrinsicHeight} and {@link #getIntrinsicWidth} methods.

getPadding会将Drawable实例与实例中内容的间隔信息存储在Rect实例中。
 *     <li> The {@link #getPadding} method can return from some Drawables
 *     information about how to frame content that is placed inside of them.
例如，一个Drawable实例作为一个Button的背景，Button控件实例需要返回padding值用来放置Button控件显示的内容。
 *     For example, a Drawable that is intended to be the frame for a button
 *     widget would need to return padding that correctly places the label
 *     inside of itself.
 *

setState方法允许客户端告知Drawable实例在什么状态下才进行绘制。
例如“焦点获取状态”，“选中状态”等等。某些Drawable可能会根据选定的状态值变更它们的外观。
 *     <li> The {@link #setState} method allows the client to tell the Drawable
 *     in which state it is to be drawn, such as "focused", "selected", etc.
 *     Some drawables may modify their imagery based on the selected state.
 *

setLevel方法允许客户端提供一个单一的连续控制器来编辑正在显示的Drawable实例，例如电量水平或者进度值。某些Drawable实例可以根据当前的level值变更它们的外观。
 *     <li> The {@link #setLevel} method allows the client to supply a single
 *     continuous controller that can modify the Drawable is displayed, such as
 *     a battery level or progress level.  Some drawables may modify their
 *     imagery based on the current level.
 *

通过Callback接口，一个Drawable实例可以回调其客户端来执行动画。为了动画可以被执行，所有的客户端都应该支持这个Callback接口。实现这一效果最简单的方法就是通过系统提供的机制，例如ImageView，View.setBackgoound方法。
 *     <li> A Drawable can perform animations by calling back to its client
 *     through the {@link Callback} interface.  All clients should support this
 *     interface (via {@link #setCallback}) so that animations will work.  A
 *     simple way to do this is through the system facilities such as
 *     {@link android.view.View#setBackground(Drawable)} and
 *     {@link android.widget.ImageView}.
 * </ul>
 *

尽管通常情况下对应用不可见，Drawable实例可能存在以下多种形式：
Bitmap:最简单的Drawable形式，PNG或者JPEG图片。
.9图：PNG的一个扩展，可以支持设置其如何填充内容，如何被拉伸。
Shape：包含简单的绘制指令，用于替代bitmap，某些情况下对大小调整有更好表现。
Layers：一个复合的Drawable，按照层级进行绘制，单个Drawable实例绘制于其下层Drawable实例集合之上。
States：一个复合的Drawable，根据它的state选择一个Drawable集合。
Levels：一个复合的Drawable，根据它的level选择一个Drawable集合。
Scale：一个复合的Drawable和单个Drawable实例构成，它的总体尺寸由它的当前level值决定。
 * Though usually not visible to the application, Drawables may take a variety
 * of forms:
 *
 * <ul>
 *     <li> <b>Bitmap</b>: the simplest Drawable, a PNG or JPEG image.
 *     <li> <b>Nine Patch</b>: an extension to the PNG format allows it to
 *     specify information about how to stretch it and place things inside of
 *     it.
 *     <li> <b>Shape</b>: contains simple drawing commands instead of a raw
 *     bitmap, allowing it to resize better in some cases.
 *     <li> <b>Layers</b>: a compound drawable, which draws multiple underlying
 *     drawables on top of each other.
 *     <li> <b>States</b>: a compound drawable that selects one of a set of
 *     drawables based on its state.
 *     <li> <b>Levels</b>: a compound drawable that selects one of a set of
 *     drawables based on its level.
 *     <li> <b>Scale</b>: a compound drawable with a single child drawable,
 *     whose overall size is modified based on the current level.
 * </ul>
 *

自定义Drawable
所有的Android版本都支持框架层提供的Drawable类被扩展/自定义和应用于运行时。从Android版本24开始，自定义Drawable可以在XML中直接使用。
 * <a name="Custom"></a>
 * <h3>Custom drawables</h3>
 *
 * <p>
 * All versions of Android allow the Drawable class to be extended and used at
 * run time in place of framework-provided drawable classes. Starting in
 * {@link android.os.Build.VERSION_CODES#N API 24}, custom drawables classes
 * may also be used in XML.
 * <p>
注：
自定义Drawable仅能应用于当前应用，其他应用无法加载它们。
自定义Drawable必须继承Drawable类，至少重写draw方法以绘制内容。
 * <strong>Note:</strong> Custom drawable classes are only accessible from
 * within your application package. Other applications will not be able to load
 * them.
 * <p>
 * At a minimum, custom drawable classes must implement the abstract methods on
 * Drawable and should override the {@link Drawable#draw(Canvas)} method to
 * draw content.
自定义Drawable用于XML中有多种方式：
1：直接引用自定义Drawable类名的全称，且该类必须为公共顶层类。
2：使用drawable作为XML的元素名称，指定该自定义Drawable类的全称。该自定义Drawable类可以是 公共顶层类或者公共静态内部类。
 * <p>
 * Custom drawables classes may be used in XML in multiple ways:
 * <ul>
 *     <li>
 *         Using the fully-qualified class name as the XML element name. For
 *         this method, the custom drawable class must be a public top-level
 *         class.
 * <pre>
 * <com.myapp.MyCustomDrawable xmlns:android="http://schemas.android.com/apk/res/android"
 *     android:color="#ffff0000" />
 * </pre>
 *     </li>
 *     <li>
 *         Using <em>drawable</em> as the XML element name and specifying the
 *         fully-qualified class name from the <em>class</em> attribute. This
 *         method may be used for both public top-level classes and public
 *         static inner classes.
 * <pre>
 * <drawable xmlns:android="http://schemas.android.com/apk/res/android"
 *     class="com.myapp.MyTopLevelClass$InnerCustomDrawable"
 *     android:color="#ffff0000" />
 * </pre>
 *     </li>
 * </ul>
 *
 略
 */
public abstract class Drawable {
    private static final Rect ZERO_BOUNDS_RECT = new Rect();

    static final PorterDuff.Mode DEFAULT_TINT_MODE = PorterDuff.Mode.SRC_IN;

    private int[] mStateSet = StateSet.WILD_CARD;
    private int mLevel = 0;
    private @Config int mChangingConfigurations = 0;
    private Rect mBounds = ZERO_BOUNDS_RECT;  // lazily becomes a new Rect()
    private WeakReference<Callback> mCallback = null;
    private boolean mVisible = true;

    private int mLayoutDirection;

    /**
    在通过setBounds设置的范围内进行绘制，通过调用setAlpha和setColorFilter
    等方法可以影响绘制的效果。
    canvas：当前Drawable实例要被绘制到canvas上。
     * Draw in its bounds (set via setBounds) respecting optional effects such
     * as alpha (set via setAlpha) and color filter (set via setColorFilter).
     *
     * @param canvas The canvas to draw into
     */
    public abstract void draw(@NonNull Canvas canvas);
    /**
    为当前Drawable实例设置一个矩形范围，在draw方法调用时候，
    Drawable实例将被绘制到这个矩形范围内。
     * Specify a bounding rectangle for the Drawable. This is where the drawable
     * will draw when its draw() method is called.
     */
    public void setBounds(int left, int top, int right, int bottom) {
        Rect oldBounds = mBounds;

        if (oldBounds == ZERO_BOUNDS_RECT) {
            oldBounds = mBounds = new Rect();
        }

        if (oldBounds.left != left || oldBounds.top != top ||
                oldBounds.right != right || oldBounds.bottom != bottom) {
            if (!oldBounds.isEmpty()) {
                // first invalidate the previous bounds
                invalidateSelf();
            }
            mBounds.set(left, top, right, bottom);
            onBoundsChange(mBounds);
        }
    }
    public void setBounds(@NonNull Rect bounds) {
        setBounds(bounds.left, bounds.top, bounds.right, bounds.bottom);
    }

    /**
    将当前Drawable实例通过setBounds设置的绘制范围拷贝到客户端提供的Rect实例中返回
     * Return a copy of the drawable's bounds in the specified Rect (allocated
     * by the caller). The bounds specify where this will draw when its draw()
     * method is called.
     *
     * @param bounds Rect to receive the drawable's bounds (allocated by the
     *               caller).
     */
    public final void copyBounds(@NonNull Rect bounds) {
        bounds.set(mBounds);
    }
    public final Rect copyBounds() {
        return new Rect(mBounds);
    }
    /**
    返回当前Drawable实例的矩形绘制范围。注：返回的矩形就是
    当前Drawable实际的绘制范围矩形，所以如果是需要一个拷贝的矩形范围，
    应该调用copyBounds来代替。

    调用getBounds，你不能修改返回的矩形，会影响Drawable实例。
     * Return the drawable's bounds Rect. Note: for efficiency, the returned
     * object may be the same object stored in the drawable (though this is not
     * guaranteed), so if a persistent copy of the bounds is needed, call
     * copyBounds(rect) instead.
     * You should also not change the object returned by this method as it may
     * be the same object stored in the drawable.
     *
     * @return The bounds of the drawable (which may change later, so caller
     *         beware). DO NOT ALTER the returned object as it may change the
     *         stored bounds of this drawable.
     *
     * @see #copyBounds()
     * @see #copyBounds(android.graphics.Rect)
     */
    @NonNull
    public final Rect getBounds() {
        if (mBounds == ZERO_BOUNDS_RECT) {
            mBounds = new Rect();
        }

        return mBounds;
    }
    /**
    返回当前Drawable实例的模糊绘制范围矩形。
    注：返回的矩形和当前Drawable绘制返回矩形是同一个对象。
     * Return the drawable's dirty bounds Rect. Note: for efficiency, the
     * returned object may be the same object stored in the drawable (though
     * this is not guaranteed).
     * <p>
     * By default, this returns the full drawable bounds. Custom drawables may
     * override this method to perform more precise invalidation.
     *
     * @return The dirty bounds of this drawable
     */
    @NonNull
    public Rect getDirtyBounds() {
        return getBounds();
    }

    /**
    这段我没怎么看懂，勉强翻译一下，哪位同学懂可以留言！
    为配置参数设置一个标记，当该配置参数变更时可能改变当前
    Drawable实例，要求当前Drawable实例重新创建。
     * Set a mask of the configuration parameters for which this drawable
     * may change, requiring that it be re-created.
     *
     * @param configs A mask of the changing configuration parameters, as
     * defined by {@link android.content.pm.ActivityInfo}.
     *
     * @see android.content.pm.ActivityInfo
     */
    public void setChangingConfigurations(@Config int configs) {
        mChangingConfigurations = configs;
    }
    public @Config int getChangingConfigurations() {
        return mChangingConfigurations;
    }
    /**
    当设置为true，该Drawable实例在绘制到一个低于8-bits每单位色值
    的设备上时候颜色将发生‘抖动’？
     * Set to true to have the drawable dither its colors when drawn to a
     * device with fewer than 8-bits per color component.
     *
     * @see android.graphics.Paint#setDither(boolean);
     * @deprecated This property is ignored.
     */
    @Deprecated
    public void setDither(boolean dither) {}
    /**
    当设置为true,则该Drawable实例在缩放或者旋转时候将
    对它关联的bitmap进行滤波过滤。可以提升旋转时的绘制效果。
    如果该Drawable实例未使用bitmap,这个方法无作用。
     * Set to true to have the drawable filter its bitmaps with bilinear
     * sampling when they are scaled or rotated.
     *
     * <p>This can improve appearance when bitmaps are rotated. If the drawable
     * does not use bitmaps, this call is ignored.</p>
     *
     * @see #isFilterBitmap()
     * @see android.graphics.Paint#setFilterBitmap(boolean);
     */
    public void setFilterBitmap(boolean filter) {}
    public boolean isFilterBitmap() {
        return false;
    }
    /**
    一个回调接口，用于调度和执行Drawable实例的动画。
    如果要实现自定义的动画Drawable,就需要实现这个接口。
     * Implement this interface if you want to create an animated drawable that
     * extends {@link android.graphics.drawable.Drawable Drawable}.
     * Upon retrieving a drawable, use
     * {@link Drawable#setCallback(android.graphics.drawable.Drawable.Callback)}
     * to supply your implementation of the interface to the drawable; it uses
     * this interface to schedule and execute animation changes.
     */
    public interface Callback {
        /**
        Drawable实例被重绘时候调用。在当前Drawable实例位置的View
        实例需要重绘，或者至少部分重绘。
         * Called when the drawable needs to be redrawn.  A view at this point
         * should invalidate itself (or at least the part of itself where the
         * drawable appears).
         *
         * @param who The drawable that is requesting the update.
         */
        void invalidateDrawable(@NonNull Drawable who);

        /**
        一个Drawable实例可以调用这个方法预先安排动画的下一帧。
        也可以通过Handler.postAtTime实现。
         * A Drawable can call this to schedule the next frame of its
         * animation.  An implementation can generally simply call
         * {@link android.os.Handler#postAtTime(Runnable, Object, long)} with
         * the parameters <var>(what, who, when)</var> to perform the
         * scheduling.
         *
         * @param who The drawable being scheduled.
         * @param what The action to execute.
         * @param when The time (in milliseconds) to run.  The timebase is
         *             {@link android.os.SystemClock#uptimeMillis}
         */
        void scheduleDrawable(@NonNull Drawable who, @NonNull Runnable what, long when);

        /**
        一个Drawable实例可以调用这个方法取消之前安排的某一帧。
        也可以通过Handler.removeCallbacks实现。
         * A Drawable can call this to unschedule an action previously
         * scheduled with {@link #scheduleDrawable}.  An implementation can
         * generally simply call
         * {@link android.os.Handler#removeCallbacks(Runnable, Object)} with
         * the parameters <var>(what, who)</var> to unschedule the drawable.
         *
         * @param who The drawable being unscheduled.
         * @param what The action being unscheduled.
         */
        void unscheduleDrawable(@NonNull Drawable who, @NonNull Runnable what);
    }
    /**
    如果客户端要求支持动画Drawable,将一个Callback实例绑定到当前
    Drawable实例上。
     * Bind a {@link Callback} object to this Drawable.  Required for clients
     * that want to support animated drawables.
     *
     * @param cb The client's Callback implementation.
     *
     * @see #getCallback()
     */
    public final void setCallback(@Nullable Callback cb) {
        mCallback = cb != null ? new WeakReference<>(cb) : null;
    }
    public Callback getCallback() {
        return mCallback != null ? mCallback.get() : null;
    }
    /**
    通过由调用setCallBack设置过的Callback实例执行
    invalidateDrawable。如果没有调用过setCallback，则无效果
     * Use the current {@link Callback} implementation to have this Drawable
     * redrawn.  Does nothing if there is no Callback attached to the
     * Drawable.
     *
     * @see Callback#invalidateDrawable
     * @see #getCallback()
     * @see #setCallback(android.graphics.drawable.Drawable.Callback)
     */
    public void invalidateSelf() {
        final Callback callback = getCallback();
        if (callback != null) {
            callback.invalidateDrawable(this);
        }
    }
    /**
    通过由调用setCallBack设置过的Callback实例执行
    scheduleDrawable。如果没有调用过setCallback，则无效果
     * Use the current {@link Callback} implementation to have this Drawable
     * scheduled.  Does nothing if there is no Callback attached to the
     * Drawable.
     *
     * @param what The action being scheduled.
     * @param when The time (in milliseconds) to run.
     *
     * @see Callback#scheduleDrawable
     */
    public void scheduleSelf(@NonNull Runnable what, long when) {
        final Callback callback = getCallback();
        if (callback != null) {
            callback.scheduleDrawable(this, what, when);
        }
    }
    /**
    通过由调用setCallBack设置过的Callback实例执行
    unscheduleDrawable。如果没有调用过setCallback，则无效果
     * Use the current {@link Callback} implementation to have this Drawable
     * unscheduled.  Does nothing if there is no Callback attached to the
     * Drawable.
     *
     * @param what The runnable that you no longer want called.
     *
     * @see Callback#unscheduleDrawable
     */
    public void unscheduleSelf(@NonNull Runnable what) {
        final Callback callback = getCallback();
        if (callback != null) {
            callback.unscheduleDrawable(this, what);
        }
    }
    /**
    获取当前Drawable实例的布局方向。
     * Returns the resolved layout direction for this Drawable.
     *
     * @return One of {@link android.view.View#LAYOUT_DIRECTION_LTR},
     *         {@link android.view.View#LAYOUT_DIRECTION_RTL}
     * @see #setLayoutDirection(int)
     */
    public @View.ResolvedLayoutDir int getLayoutDirection() {
        return mLayoutDirection;
    }
    /**
    设置当前Drawable实例的布局方向。
     * Set the layout direction for this drawable. Should be a resolved
     * layout direction, as the Drawable has no capacity to do the resolution on
     * its own.
     *
     * @param layoutDirection the resolved layout direction for the drawable,
     *                        either {@link android.view.View#LAYOUT_DIRECTION_LTR}
     *                        or {@link android.view.View#LAYOUT_DIRECTION_RTL}
     * @return {@code true} if the layout direction change has caused the
     *         appearance of the drawable to change such that it needs to be
     *         re-drawn, {@code false} otherwise
     * @see #getLayoutDirection()
     */
    public final boolean setLayoutDirection(@View.ResolvedLayoutDir int layoutDirection) {
        if (mLayoutDirection != layoutDirection) {
            //如果当前Drawable布局方向和layoutDirection不一致，
            //则修改布局方向为layoutDirection，然后执行onLayoutDirectionChanged
            mLayoutDirection = layoutDirection;
            return onLayoutDirectionChanged(layoutDirection);
        }
        return false;
    }
    /**
    当调用setLayoutDirection方法，Drawable布局方向发生变化后调用
     * Called when the drawable's resolved layout direction changes.
     *
     * @param layoutDirection the new resolved layout direction
     * @return {@code true} if the layout direction change has caused the
     *         appearance of the drawable to change such that it needs to be
     *         re-drawn, {@code false} otherwise
     * @see #setLayoutDirection(int)
     */
    public boolean onLayoutDirectionChanged(@View.ResolvedLayoutDir int layoutDirection) {
        return false;
    }
    /**
    设置Drawable实例的透明度。
    0：完全透明
    255:完全不透明
     * Specify an alpha value for the drawable. 0 means fully transparent, and
     * 255 means fully opaque.
     */
    public abstract void setAlpha(@IntRange(from=0,to=255) int alpha);
    @IntRange(from=0,to=255)
    public int getAlpha() {
        return 0xFF;
    }
    /**
    被隐匿
     * @hide
     *
     * Internal-only method for setting xfermode on certain supported drawables.
     *
     * Should not be made public since the layers and drawing area with which
     * Drawables draw is private implementation detail, and not something apps
     * should rely upon.
     */
    public void setXfermode(@Nullable Xfermode mode) {
        // Base implementation drops it on the floor for compatibility. Whee!
    }
    /**
    为当前Drawable实例设置颜色滤镜
     * Specify an optional color filter for the drawable.
     * <p>
     * If a Drawable has a ColorFilter, each output pixel of the Drawable's
     * drawing contents will be modified by the color filter before it is
     * blended onto the render target of a Canvas.
     * </p>
     * <p>
     * Pass {@code null} to remove any existing color filter.
     * </p>
     * <p class="note"><strong>Note:</strong> Setting a non-{@code null} color
     * filter disables {@link #setTintList(ColorStateList) tint}.
     * </p>
     *
     * @param colorFilter The color filter to apply, or {@code null} to remove the
     *            existing color filter
     */
    public abstract void setColorFilter(@Nullable ColorFilter colorFilter);
    /**
    为当前Drawable实例设置滤镜效果
     * Specify a color and Porter-Duff mode to be the color filter for this
     * drawable.
     * <p>
     * Convenience for {@link #setColorFilter(ColorFilter)} which constructs a
     * {@link PorterDuffColorFilter}.
     * </p>
     * <p class="note"><strong>Note:</strong> Setting a color filter disables
     * {@link #setTintList(ColorStateList) tint}.
     * </p>
     */
    public void setColorFilter(@ColorInt int color, @NonNull PorterDuff.Mode mode) {
        setColorFilter(new PorterDuffColorFilter(color, mode));
    }
    /**
    为当前Drawable实例着色
     * Specifies tint color for this drawable.
     * <p>
    当前Drawable实例的绘制内容在被绘制到屏幕上之前将被指定颜色着色
    当前方法和setColorFilter类似。
     * A Drawable's drawing content will be blended together with its tint
     * before it is drawn to the screen. This functions similarly to
     * {@link #setColorFilter(int, PorterDuff.Mode)}.
     * </p>
     * <p>
     * To clear the tint, pass {@code null} to
     * {@link #setTintList(ColorStateList)}.
     * </p>
     * <p class="note"><strong>Note:</strong> Setting a color filter via
     * {@link #setColorFilter(ColorFilter)} or
     * {@link #setColorFilter(int, PorterDuff.Mode)} overrides tint.
     * </p>
     *
     * @param tintColor Color to use for tinting this drawable
     * @see #setTintList(ColorStateList)
     * @see #setTintMode(PorterDuff.Mode)
     */
    public void setTint(@ColorInt int tintColor) {
        setTintList(ColorStateList.valueOf(tintColor));
    }
    /**
    根据ColorStateList对当前Drawable实例进行着色
    这个一个空方法！！！在上一篇文章中已经指出，Drawabld的子类
    实现了这个方法。
     * Specifies tint color for this drawable as a color state list.
     * <p>
     * A Drawable's drawing content will be blended together with its tint
     * before it is drawn to the screen. This functions similarly to
     * {@link #setColorFilter(int, PorterDuff.Mode)}.
     * </p>
     * <p class="note"><strong>Note:</strong> Setting a color filter via
     * {@link #setColorFilter(ColorFilter)} or
     * {@link #setColorFilter(int, PorterDuff.Mode)} overrides tint.
     * </p>
     *
     * @param tint Color state list to use for tinting this drawable, or
     *            {@code null} to clear the tint
     * @see #setTint(int)
     * @see #setTintMode(PorterDuff.Mode)
     */
    public void setTintList(@Nullable ColorStateList tint) {}
    /**
    设置当前Drawable实例着色的过滤模式
     * Specifies a tint blending mode for this drawable.
     * <p>
     * Defines how this drawable's tint color should be blended into the drawable
     * before it is drawn to screen. Default tint mode is {@link PorterDuff.Mode#SRC_IN}.
     * </p>
     * <p class="note"><strong>Note:</strong> Setting a color filter via
     * {@link #setColorFilter(ColorFilter)} or
     * {@link #setColorFilter(int, PorterDuff.Mode)} overrides tint.
     * </p>
     *
     * @param tintMode A Porter-Duff blending mode
     * @see #setTint(int)
     * @see #setTintList(ColorStateList)
     */
    public void setTintMode(@NonNull PorterDuff.Mode tintMode) {}
    public @Nullable ColorFilter getColorFilter() {
        return null;
    }
    /**
    取消当前Drawable实例的滤镜。
     * Removes the color filter for this drawable.
     */
    public void clearColorFilter() {
        setColorFilter(null);
    }
    /**
    设置当前Drawable实例热点区域的中心点坐标
     * Specifies the hotspot's location within the drawable.
     *
     * @param x The X coordinate of the center of the hotspot
     * @param y The Y coordinate of the center of the hotspot
     */
    public void setHotspot(float x, float y) {}
    /**
    设置当前Drawable实例的热点区域的边界
     * Sets the bounds to which the hotspot is constrained, if they should be
     * different from the drawable bounds.
     *
     * @param left position in pixels of the left bound
     * @param top position in pixels of the top bound
     * @param right position in pixels of the right bound
     * @param bottom position in pixels of the bottom bound
     * @see #getHotspotBounds(android.graphics.Rect)
     */
    public void setHotspotBounds(int left, int top, int right, int bottom) {}
    /**
     * Populates {@code outRect} with the hotspot bounds.
     *
     * @param outRect the rect to populate with the hotspot bounds
     * @see #setHotspotBounds(int, int, int, int)
     */
    public void getHotspotBounds(@NonNull Rect outRect) {
        outRect.set(getBounds());
    }
    /**
    被隐匿
     * Whether this drawable requests projection.
     *
     * @hide magic!
     */
    public boolean isProjected() {
        return false;
    }
    /**
    标示当前Drawable实例的外观是否要根据state进行变更。
    客户端可以用这个方法判断是否有必要计算state并调用setState。
     * Indicates whether this drawable will change its appearance based on
     * state. Clients can use this to determine whether it is necessary to
     * calculate their state and call setState.
     *
     * @return True if this drawable changes its appearance based on state,
     *         false otherwise.
     * @see #setState(int[])
     */
    public boolean isStateful() {
        return false;
    }
    /**
    为当前Drawable实例设置一个状态值集合。当现有状态和stateSet
    不同时候，触发onStateChange(stateSet)方法。
     * Specify a set of states for the drawable. These are use-case specific,
     * so see the relevant documentation. As an example, the background for
     * widgets like Button understand the following states:
     * [{@link android.R.attr#state_focused},
     *  {@link android.R.attr#state_pressed}].
     *
     * <p>If the new state you are supplying causes the appearance of the
     * Drawable to change, then it is responsible for calling
     * {@link #invalidateSelf} in order to have itself redrawn, <em>and</em>
     * true will be returned from this function.
     *
     * <p>Note: The Drawable holds a reference on to <var>stateSet</var>
     * until a new state array is given to it, so you must not modify this
     * array during that time.</p>
     *
     * @param stateSet The new set of states to be displayed.
     *
     * @return Returns true if this change in state has caused the appearance
     * of the Drawable to change (hence requiring an invalidate), otherwise
     * returns false.
     */
    public boolean setState(@NonNull final int[] stateSet) {
        if (!Arrays.equals(mStateSet, stateSet)) {
            mStateSet = stateSet;
            return onStateChange(stateSet);
        }
        return false;
    }
    /**
     * Describes the current state, as a union of primitve states, such as
     * {@link android.R.attr#state_focused},
     * {@link android.R.attr#state_selected}, etc.
     * Some drawables may modify their imagery based on the selected state.
     * @return An array of resource Ids describing the current state.
     */
    public @NonNull int[] getState() {
        return mStateSet;
    }
    /**
    如果当前Drawable实例在执行过渡动画，要求当前实例
    立即跳转到当前状态并跳过任何正在执行的动画。
     * If this Drawable does transition animations between states, ask that
     * it immediately jump to the current state and skip any active animations.
     */
    public void jumpToCurrentState() {
    }
    /**
    返回当前Drawable实例正在使用的Drawable实例，
    对于一般单个Drawable，返回值就是自身，对于像StateListDrawable
    这样的复合Drawable实例，则返回其持有的一个子Drawable实例。
     * @return The current drawable that will be used by this drawable. For simple drawables, this
     *         is just the drawable itself. For drawables that change state like
     *         {@link StateListDrawable} and {@link LevelListDrawable} this will be the child drawable
     *         currently in use.
     */
    public @NonNull Drawable getCurrent() {
        return this;
    }
    /**
    为当前Drawable实例设置图像级别，从0到10000。setLevel使得
    Drawable实例可以通过一个不断变化的控制器来变更它的图像，
    例如音量等级或者进度。
    
     * Specify the level for the drawable.  This allows a drawable to vary its
     * imagery based on a continuous controller, for example to show progress
     * or volume level.
     *
     * <p>If the new level you are supplying causes the appearance of the
     * Drawable to change, then it is responsible for calling
     * {@link #invalidateSelf} in order to have itself redrawn, <em>and</em>
     * true will be returned from this function.
     *
     * @param level The new level, from 0 (minimum) to 10000 (maximum).
     *
     * @return Returns true if this change in level has caused the appearance
     * of the Drawable to change (hence requiring an invalidate), otherwise
     * returns false.
     */
    public final boolean setLevel(@IntRange(from=0,to=10000) int level) {
        if (mLevel != level) {
            //修改图像等级为level，并调用onLevelChange
            mLevel = level;
            return onLevelChange(level);
        }
        return false;
    }
    /**
     * Retrieve the current level.
     *
     * @return int Current level, from 0 (minimum) to 10000 (maximum).
     */
    public final @IntRange(from=0,to=10000) int getLevel() {
        return mLevel;
    }

~~~~(>_<)~~~~ 
当前文章长度正在接近简书的限度，请考虑分篇书写吧
~~~~(>_<)~~~~ 

    /**
    设置当前Drawable实例是否可见，并不会影响Drawable实例的行为，
    但是可以被某些Drawable来控制是否执行动画。
    例如：AnimationDrawable可以通过这个方法启动或者停止动画，
    后续文章会有验证。
     * Set whether this Drawable is visible.  This generally does not impact
     * the Drawable's behavior, but is a hint that can be used by some
     * Drawables, for example, to decide whether run animations.
     *
     * @param visible Set to true if visible, false if not.
     * @param restart You can supply true here to force the drawable to behave
     *                as if it has just become visible, even if it had last
     *                been set visible.  Used for example to force animations
     *                to restart.
     *
     * @return boolean Returns true if the new visibility is different than
     *         its previous state.
     */
    public boolean setVisible(boolean visible, boolean restart) {
        boolean changed = mVisible != visible;
        if (changed) {
            mVisible = visible;
            invalidateSelf();
        }
        return changed;
    }
    public final boolean isVisible() {
        return mVisible;
    }
    /**
    设置当前Drawable实例是不是自动被“镜像”/左右对调
    当它的布局模式是从右到左
     * Set whether this Drawable is automatically mirrored when its layout direction is RTL
     * (right-to left). See {@link android.util.LayoutDirection}.
     *
     * @param mirrored Set to true if the Drawable should be mirrored, false if not.
     */
    public void setAutoMirrored(boolean mirrored) {
    }
    /**
     * Tells if this Drawable will be automatically mirrored  when its layout direction is RTL
     * right-to-left. See {@link android.util.LayoutDirection}.
     *
     * @return boolean Returns true if this Drawable will be automatically mirrored.
     */
    public boolean isAutoMirrored() {
        return false;
    }
    /**
    为当前Drawable实例和它的子实例应用指定的主题
     * Applies the specified theme to this Drawable and its children.
     *
     * @param t the theme to apply
     */
    public void applyTheme(@NonNull @SuppressWarnings("unused") Theme t) {
        //空方法，Drawable子类会做自定义实现
    }
    public boolean canApplyTheme() {
        return false;
    }
    /**
    返回当前Drawable实例的透明或者不透明。返回值是其中之一：
     {@link android.graphics.PixelFormat#UNKNOWN}-透明度未知
     {@link android.graphics.PixelFormat#TRANSLUCENT}-半透明
     {@link android.graphics.PixelFormat#TRANSPARENT}-完全透明
     {@link android.graphics.PixelFormat#OPAQUE}-完全不透明
    如果Drawable中的内容可见性不确定，最安全的方案
    是返回TRANSLUCENT/半透明
     
     * Return the opacity/transparency of this Drawable.  The returned value is
     * one of the abstract format constants in
     * {@link android.graphics.PixelFormat}:
     * {@link android.graphics.PixelFormat#UNKNOWN},
     * {@link android.graphics.PixelFormat#TRANSLUCENT},
     * {@link android.graphics.PixelFormat#TRANSPARENT}, or
     * {@link android.graphics.PixelFormat#OPAQUE}.
     *
     * <p>An OPAQUE drawable is one that draws all all content within its bounds, completely
     * covering anything behind the drawable. A TRANSPARENT drawable is one that draws nothing
     * within its bounds, allowing everything behind it to show through. A TRANSLUCENT drawable
     * is a drawable in any other state, where the drawable will draw some, but not all,
     * of the content within its bounds and at least some content behind the drawable will
     * be visible. If the visibility of the drawable's contents cannot be determined, the
     * safest/best return value is TRANSLUCENT.
     *
     * <p>Generally a Drawable should be as conservative as possible with the
     * value it returns.  For example, if it contains multiple child drawables
     * and only shows one of them at a time, if only one of the children is
     * TRANSLUCENT and the others are OPAQUE then TRANSLUCENT should be
     * returned.  You can use the method {@link #resolveOpacity} to perform a
     * standard reduction of two opacities to the appropriate single output.
     *
     * <p>Note that the returned value does not necessarily take into account a
     * custom alpha or color filter that has been applied by the client through
     * the {@link #setAlpha} or {@link #setColorFilter} methods. Some subclasses,
     * such as {@link BitmapDrawable}, {@link ColorDrawable}, and {@link GradientDrawable},
     * do account for the value of {@link #setAlpha}, but the general behavior is dependent
     * upon the implementation of the subclass.
     *
     * @return int The opacity class of the Drawable.
     *
     * @see android.graphics.PixelFormat
     */
    public abstract @PixelFormat.Opacity int getOpacity();
    /**
    根据两个不透明度值，返回合适的值。
    两个不透明度值若相等，直接返回；
    否则如果两个透明度值有至少一个是UNKNOWN，返回UNKNOWN；
    否则如果两个透明度值有至少一个是TRANSLUCENT，返回TRANSLUCENT；
    否则如果两个透明度值有至少一个是TRANSPARENT，返回TRANSPARENT；
    否则返回OPAQUE；
     * Return the appropriate opacity value for two source opacities.  If
     * either is UNKNOWN, that is returned; else, if either is TRANSLUCENT,
     * that is returned; else, if either is TRANSPARENT, that is returned;
     * else, OPAQUE is returned.
     *
     * <p>This is to help in implementing {@link #getOpacity}.
     *
     * @param op1 One opacity value.
     * @param op2 Another opacity value.
     *
     * @return int The combined opacity value.
     *
     * @see #getOpacity
     */
    public static @PixelFormat.Opacity int resolveOpacity(@PixelFormat.Opacity int op1,
            @PixelFormat.Opacity int op2) {
        if (op1 == op2) {
            return op1;
        }
        if (op1 == PixelFormat.UNKNOWN || op2 == PixelFormat.UNKNOWN) {
            return PixelFormat.UNKNOWN;
        }
        if (op1 == PixelFormat.TRANSLUCENT || op2 == PixelFormat.TRANSLUCENT) {
            return PixelFormat.TRANSLUCENT;
        }
        if (op1 == PixelFormat.TRANSPARENT || op2 == PixelFormat.TRANSPARENT) {
            return PixelFormat.TRANSPARENT;
        }
        return PixelFormat.OPAQUE;
    }
    /**
    返回在当前Drawable实例中完全透明的一个区域。
    这个区域可以用来影响绘制操作，定义当前Drawable实例的目标
    在渲染当前Drawable实例时候哪个区域不需要改变。
     * Returns a Region representing the part of the Drawable that is completely
     * transparent.  This can be used to perform drawing operations, identifying
     * which parts of the target will not change when rendering the Drawable.
     * The default implementation returns null, indicating no transparent
     * region; subclasses can optionally override this to return an actual
     * Region if they want to supply this optimization information, but it is
     * not required that they do so.
     *
     * @return Returns null if the Drawables has no transparent region to
     * report, else a Region holding the parts of the Drawable's bounds that
     * are transparent.
     */
    public @Nullable Region getTransparentRegion() {
        return null;
    }
    /**
    如果子类需要根据state来变更Drawable实例的外观，则需要重写该方法。
    如果state的变更引起了Drawable实例外观变化，则返回true,
    否则返回false;
     * Override this in your subclass to change appearance if you recognize the
     * specified state.
     *
     * @return Returns true if the state change has caused the appearance of
     * the Drawable to change (that is, it needs to be drawn), else false
     * if it looks the same and there is no need to redraw it since its
     * last state.
     */
    protected boolean onStateChange(int[] state) {
        return false;
    }
    /** 
    如果子类需要根据level来变更Drawable实例的外观，则需要重写该方法。
    如果level的变更引起了Drawable实例外观变化，则返回true,
    否则返回false;
    Override this in your subclass to change appearance if you vary based
     *  on level.
     * @return Returns true if the level change has caused the appearance of
     * the Drawable to change (that is, it needs to be drawn), else false
     * if it looks the same and there is no need to redraw it since its
     * last level.
     */
    protected boolean onLevelChange(int level) {
        return false;
    }
    /**
    如果子类实例需要在绘制范围发生变化后变更Drawable实例的外观，则需要重写该方法。
     * Override this in your subclass to change appearance if you vary based on
     * the bounds.
     */
    protected void onBoundsChange(Rect bounds) {
        // Stub method.
    }
    /**
    返回当前Drawable实例的实质宽度。
    实质宽度是Drawable实例占据的宽度，包含padding值。
    如果Drawable实例没有实际宽度，例如是一个颜色，则返回-1
     * Returns the drawable's intrinsic width.
     * <p>
     * Intrinsic width is the width at which the drawable would like to be laid
     * out, including any inherent padding. If the drawable has no intrinsic
     * width, such as a solid color, this method returns -1.
     *
     * @return the intrinsic width, or -1 if no intrinsic width
     */
    public int getIntrinsicWidth() {
        return -1;
    }
    /**
    返回当前Drawable实例的实质高度。
    实质宽度是Drawable实例占据的高度，包含padding值。
    如果Drawable实例没有实际高度，例如是一个颜色，则返回-1
     * Returns the drawable's intrinsic height.
     * <p>
     * Intrinsic height is the height at which the drawable would like to be
     * laid out, including any inherent padding. If the drawable has no
     * intrinsic height, such as a solid color, this method returns -1.
     *
     * @return the intrinsic height, or -1 if no intrinsic height
     */
    public int getIntrinsicHeight() {
        return -1;
    }
    /**
    返回当前Drawable建议的最小宽度。
    如果一个View实例用当前Drawable当做背景，那么建议该View实例
    宽度最小为这个值。
     * Returns the minimum width suggested by this Drawable. If a View uses this
     * Drawable as a background, it is suggested that the View use at least this
     * value for its width. (There will be some scenarios where this will not be
     * possible.) This value should INCLUDE any padding.
     *
     * @return The minimum width suggested by this Drawable. If this Drawable
     *         doesn't have a suggested minimum width, 0 is returned.
     */
    public int getMinimumWidth() {
        final int intrinsicWidth = getIntrinsicWidth();
        return intrinsicWidth > 0 ? intrinsicWidth : 0;
    }
    /**
    返回当前Drawable建议的最小高度。
    如果一个View实例用当前Drawable当做背景，那么建议该View实例
    高度最小为这个值。
     * Returns the minimum height suggested by this Drawable. If a View uses this
     * Drawable as a background, it is suggested that the View use at least this
     * value for its height. (There will be some scenarios where this will not be
     * possible.) This value should INCLUDE any padding.
     *
     * @return The minimum height suggested by this Drawable. If this Drawable
     *         doesn't have a suggested minimum height, 0 is returned.
     */
    public int getMinimumHeight() {
        final int intrinsicHeight = getIntrinsicHeight();
        return intrinsicHeight > 0 ? intrinsicHeight : 0;
    }
    /**
    将当前Drawable实例的padding值作为参数设置为Recti实例padding
    的边界值。
    如果当前实例有padding值，返回true,否则返回false；
    当返回false，则Recti实例padding的边界值都设置为0；
     * Return in padding the insets suggested by this Drawable for placing
     * content inside the drawable's bounds. Positive values move toward the
     * center of the Drawable (set Rect.inset).
     *
     * @return true if this drawable actually has a padding, else false. When false is returned,
     * the padding is always set to 0.
     */
    public boolean getPadding(@NonNull Rect padding) {
        padding.set(0, 0, 0, 0);
        return false;
    }
    /**
    被隐匿
     * Return in insets the layout insets suggested by this Drawable for use with alignment
     * operations during layout.
     * @hide
     */
    public @NonNull Insets getOpticalInsets() {
        return Insets.NONE;
    }

    /**
    调用此方法获取当前Drawable实例的绘制区域轮廓。
    这个方法默认被ViewOutlineProvider调用去定义View实例的轮廓。
    ViewOutlineProvider：后续文章会做介绍，是个蛮有趣的类。
     * Called to get the drawable to populate the Outline that defines its drawing area.
     * <p>
     * This method is called by the default {@link android.view.ViewOutlineProvider} to define
     * the outline of the View.
     * <p>
     * The default behavior defines the outline to be the bounding rectangle of 0 alpha.
     * Subclasses that wish to convey a different shape or alpha value must override this method.
     *
     * @see android.view.View#setOutlineProvider(android.view.ViewOutlineProvider)
     */
    public void getOutline(@NonNull Outline outline) {
        outline.setRect(getBounds());
        outline.setAlpha(0);
    }

未完待续...

```
未完待续...

以上就是个人分析的一点结果，若有错误，请各位同学留言告知！

**That's all !**
