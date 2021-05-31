> 前两天写一个自定义控件，使用Drawable变色来展示EditText的不同状态，涉及到了DrawableCompat这个类，今天着重分析一下它。

####1:Drawable变色的通用代码
```
//1:通过图片资源文件生成Drawable实例
Drawable drawable = getResources().getDrawable(R.mipmap.ic_launcher).mutate();
//2:先调用DrawableCompat的wrap方法
drawable = DrawableCompat.wrap(drawable);
//3:再调用DrawableCompat的setTint方法，为Drawable实例进行着色
DrawableCompat.setTint(drawable, Color.RED);
```
这里涉及几个方法:
- Drawable.mutate()
- DrawableCompat.wrap(@NonNull Drawable drawable)
- DrawableCompat.setTint(@NonNull Drawable drawable, @ColorInt int tint)

下面看一下这几个方法的源码，Drawable.mutate()稍后分析，先看一下DrawableCompat中的wrap和setTint这两个方法。
####2:DrawableCompat.wrap(@NonNull Drawable drawable)
```
public static Drawable wrap(@NonNull Drawable drawable) {
  return IMPL.wrap(drawable);
}
```
从源码可见，wrap方法内部是return IMPL.wrap(drawable)，那这个IMPL是？
```
    static final DrawableImpl IMPL;
    /**
     * Interface for the full API.
     */
    interface DrawableImpl {
        void jumpToCurrentState(Drawable drawable);
        void setAutoMirrored(Drawable drawable, boolean mirrored);
        boolean isAutoMirrored(Drawable drawable);
        void setHotspot(Drawable drawable, float x, float y);
        void setHotspotBounds(Drawable drawable, int left, int top, int right, int bottom);
        void setTint(Drawable drawable, int tint);
        void setTintList(Drawable drawable, ColorStateList tint);
        void setTintMode(Drawable drawable, PorterDuff.Mode tintMode);
        Drawable wrap(Drawable drawable);
        boolean setLayoutDirection(Drawable drawable, int layoutDirection);
        int getLayoutDirection(Drawable drawable);
        int getAlpha(Drawable drawable);
        void applyTheme(Drawable drawable, Resources.Theme t);
        boolean canApplyTheme(Drawable drawable);
        ColorFilter getColorFilter(Drawable drawable);
        void clearColorFilter(Drawable drawable);
        void inflate(Drawable drawable, Resources res, XmlPullParser parser, AttributeSet attrs,
                     Resources.Theme t) throws IOException, XmlPullParserException;
    }
    static {
        final int version = android.os.Build.VERSION.SDK_INT;
        if (version >= 23) {
            IMPL = new MDrawableImpl();
        } else if (version >= 21) {
            IMPL = new LollipopDrawableImpl();
        } else if (version >= 19) {
            IMPL = new KitKatDrawableImpl();
        } else if (version >= 17) {
            IMPL = new JellybeanMr1DrawableImpl();
        } else if (version >= 11) {
            IMPL = new HoneycombDrawableImpl();
        } else {
            IMPL = new BaseDrawableImpl();
        }
    }
```
可见，在不同的SDK版本下，IMPL对应DrawableImpl的不同子类实例。下面分别看一下这几个实现类对wrap方法的实质执行代码。
- **MDrawableImpl**
        public Drawable wrap(Drawable drawable) {
            // No need to wrap on M+
            //未对Drawable实例做任何处理，直接返回
            return drawable;
        }
**可见在SDK版本>= 23(MDrawableImpl)情况下:DrawableCompat.wrap(@NonNull Drawable drawable)直接返回了原始的Drawable实例**
- **LollipopDrawableImpl**
        public Drawable wrap(Drawable drawable) {
            return DrawableCompatLollipop.wrapForTinting(drawable);
        }
继续跟踪DrawableCompatLollipop.wrapForTinting:
      public static Drawable wrapForTinting(final Drawable drawable) {
        if (!(drawable instanceof TintAwareDrawable)) {
            //当前传入的Drawable实例并不属于TintAwareDrawable
            return new DrawableWrapperLollipop(drawable);
        }
        return drawable;
      }
继续跟踪DrawableWrapperLollipop:
      class DrawableWrapperLollipop extends DrawableWrapperKitKat {
        DrawableWrapperLollipop(Drawable drawable) {
          super(drawable);
        }
        ***
      }
继续跟踪DrawableWrapperKitKat:
      class DrawableWrapperKitKat extends DrawableWrapperHoneycomb {
        DrawableWrapperKitKat(Drawable drawable) {
          super(drawable);
        }
        ***
      }
继续跟踪DrawableWrapperHoneycomb:
      class DrawableWrapperHoneycomb extends DrawableWrapperGingerbread {
        DrawableWrapperHoneycomb(Drawable drawable) {
          super(drawable);
        }
        ***
      }
继续跟踪DrawableWrapperGingerbread:
```
    static final PorterDuff.Mode DEFAULT_TINT_MODE = PorterDuff.Mode.SRC_IN;
    private int mCurrentColor;
    private PorterDuff.Mode mCurrentMode;
    private boolean mColorFilterSet;
    DrawableWrapperState mState;  //mState默认是null
    private boolean mMutated;
    Drawable mDrawable;

    DrawableWrapperGingerbread(@Nullable Drawable dr) {
        mState = mutateConstantState();
        // Now set the drawable...
        setWrappedDrawable(dr);
    }
```
  - mState = mutateConstantState();
```
mutateConstantState()一路追踪到底：
    DrawableWrapperState mutateConstantState() {
        //返回一个DrawableWrapperStateBase实例
        //mState默认是null
        return new DrawableWrapperStateBase(mState, null);
    }
    private static class DrawableWrapperStateBase extends DrawableWrapperState {
        //调用父类 DrawableWrapperState 的构造函数
        //orig就是DrawableWrapperGingerbread中的mState，默认是null
        DrawableWrapperStateBase(
                @Nullable DrawableWrapperState orig, @Nullable Resources res) {
            super(orig, res);
        }
        @Override
        public Drawable newDrawable(@Nullable Resources res) {
            return new DrawableWrapperGingerbread(this, res);
        }
    }
    protected static abstract class DrawableWrapperState extends Drawable.ConstantState {
        int mChangingConfigurations;
        Drawable.ConstantState mDrawableState;
        ColorStateList mTint = null;
        PorterDuff.Mode mTintMode = DEFAULT_TINT_MODE;
        //orig就是DrawableWrapperGingerbread中的mState，默认是null
        DrawableWrapperState(@Nullable DrawableWrapperState orig, @Nullable Resources res) {
            //因为orig是null,所以mChangingConfigurations，mDrawableState，
            //mTint，mTintMode都是DrawableWrapperState中的默认值
            if (orig != null) {
                mChangingConfigurations = orig.mChangingConfigurations;
                mDrawableState = orig.mDrawableState;
                mTint = orig.mTint;
                mTintMode = orig.mTintMode;
            }
        }
}
```
代码一路跟下来可见:**mState = mutateConstantState(),mState被赋值为一个新的DrawableWrapperState实例**，
**其中：mState（DrawableWrapperState）中，下面成员变量的值都是默认值:**
*int mChangingConfigurations;
Drawable.ConstantState mDrawableState;
ColorStateList mTint = null;
PorterDuff.Mode mTintMode = DEFAULT_TINT_MODE;*
  - setWrappedDrawable(dr);
```
setWrappedDrawable(Drawable dr)一路追踪到底:
    public final void setWrappedDrawable(Drawable dr) {
        if (mDrawable != null) {
            mDrawable.setCallback(null);
        }
        mDrawable = dr;
        if (dr != null) {
            dr.setCallback(this);
            // Only call setters for data that's stored in the base Drawable.
            setVisible(dr.isVisible(), true);
            setState(dr.getState());
            setLevel(dr.getLevel());
            setBounds(dr.getBounds());
            //mState不为null:为一个新的DrawableWrapperState实例
            if (mState != null) {
                //为mState的mDrawableState赋值为Drawable原始实例
                //关联的ConstantState
                mState.mDrawableState = dr.getConstantState();
            }
        }
        invalidateSelf();
    }
```
这里涉及到DrawableWrapperGingerbread中的几个方法:
**setVisible(boolean visible, boolean restart)**
        @Override
        public boolean setVisible(boolean visible, boolean restart) {
          //Drawable中的setVisible，用于控制Drawable实例是否执行动画,对于AnimationDrawable实例会产生效果，控制是否执行动画
          return super.setVisible(visible, restart) || mDrawable.setVisible(visible, restart);
        }
**setState(final int[] stateSet)**
```
    @Override
    public boolean setState(final int[] stateSet) {
        boolean handled = mDrawable.setState(stateSet);
        handled = updateTint(stateSet) || handled;
        return handled;
    }
    private boolean updateTint(int[] state) {
        //isCompatTintEnabled()这里直接返回了true
        if (!isCompatTintEnabled()) {
            // If compat tinting is not enabled, fail fast
            return false;
        }
        //mState.mTint是默认值:null
        final ColorStateList tintList = mState.mTint;
        //mState.mTintMode是默认值:DEFAULT_TINT_MODE = PorterDuff.Mode.SRC_IN
        final PorterDuff.Mode tintMode = mState.mTintMode;
        if (tintList != null && tintMode != null) {
            //tintList为null,所以不会执行下面代码
            final int color = tintList.getColorForState(state, tintList.getDefaultColor());
            if (!mColorFilterSet || color != mCurrentColor || tintMode != mCurrentMode) {
                setColorFilter(color, tintMode);
                mCurrentColor = color;
                mCurrentMode = tintMode;
                mColorFilterSet = true;
                return true;
            }
        } else {
            //tintList为null
            mColorFilterSet = false;
            //执行的是其父类Drawable的clearColorFilter()
            clearColorFilter();
        }
        return false;
    }
    protected boolean isCompatTintEnabled() {
        // It's enabled by default on Gingerbread
        //这里直接返回了true
        return true;
    }
Drawable的clearColorFilter方法：移除了当前Drawable实例关联的ColorFilter
    public void clearColorFilter() {
        setColorFilter(null);
    }
```
可见：setState(dr.getState())这一步直接移除了Drawable实例关联的ColorFilter.
**setLevel直接使用的是其父类Drawable中的方法setLevel(@IntRange(from=0,to=10000) int level) **
```
Drawable的setLevel方法：
    public final boolean setLevel(@IntRange(from=0,to=10000) int level) {
        //在这里，因为mLevel就是之前DrawableWrapperGingerbread构造函数中的Drawable dr的level值，
        //而level=dr.getLevel()返回的也是Drawable dr的level值，mLevel == level，
        //所以下面的代码并不会执行
        if (mLevel != level) {
            mLevel = level;
            return onLevelChange(level);
        }
        return false;
    }
```
可见：setLevel(dr.getLevel())这一步并未产生实质影响，未执行处理逻辑。
**setBounds直接使用的是其父类Drawable中的方法setBounds(@NonNull Rect bounds)**
```
    /**
     * Specify a bounding rectangle for the Drawable. This is where the drawable
     * will draw when its draw() method is called.
     */
    public void setBounds(@NonNull Rect bounds) {
        setBounds(bounds.left, bounds.top, bounds.right, bounds.bottom);
    }
```
可见：setBounds(dr.getBounds())这一步为新产生的DrawableWrapperGingerbread实例设置其绘制范围与原始Drawable实例一致。

**可见在SDK版本>= 21(LollipopDrawableImpl)情况下:DrawableCompat.wrap(@NonNull Drawable drawable)返回了Drawable的子类DrawableWrapperGingerbread的一个新实例。
	且在updateTint方法中移除了该新实例关联过的ColorFilter,设置了该新实例的绘制范围和原始Drawable实例相同**
- **KitKatDrawableImpl**
跟踪终点同LollipopDrawableImpl
```
在KitKatDrawableImpl情况下，wrap(Drawable drawable)一路跟踪到底:
@Override
public Drawable wrap(Drawable drawable) {
    return DrawableCompatKitKat.wrapForTinting(drawable);
}
**
继承关系跟踪到最后还是DrawableWrapperGingerbread，和LollipopDrawableImpl相同：
DrawableWrapperGingerbread(@Nullable Drawable dr) {
    mState = mutateConstantState();
    // Now set the drawable...
    setWrappedDrawable(dr);
}
```
- **JellybeanMr1DrawableImpl**
跟踪终点同LollipopDrawableImpl
- **HoneycombDrawableImpl**
跟踪终点同LollipopDrawableImpl
- **BaseDrawableImpl**
跟踪终点同LollipopDrawableImpl

**综上可见：
1：在SDK版本>= 23(MDrawableImpl)情况下:DrawableCompat.wrap(@NonNull Drawable drawable)直接返回了原始的Drawable实例；
2：其余情况下，DrawableCompat.wrap(@NonNull Drawable drawable)返回了Drawable的子类DrawableWrapperGingerbread的一个新实例，且在updateTint方法中移除了该新实例关联过的ColorFilter,设置了该新实例的绘制范围和原始Drawable实例相同；**
####3:DrawableCompat.setTint(@NonNull Drawable drawable, @ColorInt int tint)
```
    public static void setTint(@NonNull Drawable drawable, @ColorInt int tint) {
        IMPL.setTint(drawable, tint);
    }
```
之前分析wrap方法时候已经看到IMPL在不同SDK版本下有不同的实现，还是逐一查看：
- **MDrawableImpl**
```
setTint一路跟踪：
@Override
public void setTint(Drawable drawable, int tint) {
    DrawableCompatLollipop.setTint(drawable, tint);
}
public static void setTint(Drawable drawable, int tint) {
    //执行的是Drawable原生的setTint方法
    drawable.setTint(tint);
}
```
- **LollipopDrawableImpl**
```
setTint一路跟踪：
@Override
public void setTint(Drawable drawable, int tint) {
    //同MDrawableImpl
    DrawableCompatLollipop.setTint(drawable, tint);
}
```
- **KitKatDrawableImpl**
```
setTint一路跟踪：
@Override
public void setTint(Drawable drawable, int tint) {
    DrawableCompatBase.setTint(drawable, tint);
}
public static void setTint(Drawable drawable, int tint) {
    if (drawable instanceof TintAwareDrawable) {
        ((TintAwareDrawable) drawable).setTint(tint);
    }
}
public interface TintAwareDrawable {
    void setTint(@ColorInt int tint);
    void setTintList(ColorStateList tint);
    void setTintMode(PorterDuff.Mode tintMode);
}
```
在上面分析**DrawableCompat.wrap**方法时候，已知其返回结果为**DrawableWrapperGingerbread新实例**，看一下DrawableWrapperGingerbread类的声明：class DrawableWrapperGingerbread extends Drawable **implements** Drawable.Callback, DrawableWrapper, **TintAwareDrawable** 
由此可见，setTint实质执行的还是DrawableWrapperGingerbread的setTint方法，继续跟踪：
```
    setTint一路追踪：
    @Override
    public void setTint(int tint) {
        setTintList(ColorStateList.valueOf(tint));
    }
    @Override
    public void setTintList(ColorStateList tint) {
        //1:在上面分析DrawableCompat.wrap时候，mState的值如下：
        //mState（DrawableWrapperState）中，下面成员变量的值都是默认值:
        //int mChangingConfigurations;
        //Drawable.ConstantState mDrawableState;
        //ColorStateList mTint = null;
        //PorterDuff.Mode mTintMode = DEFAULT_TINT_MODE;

        //2:在DrawableCompat.setTint时候，mState.mTint不再为空值
        mState.mTint = tint;
        updateTint(getState());
    }

    //之前DrawableCompat.wrap已经执行过一次updateTint，
    //现在DrawableCompat.setTint第二次执行！！
    private boolean updateTint(int[] state) {
        //isCompatTintEnabled()返回true
        if (!isCompatTintEnabled()) {
            // If compat tinting is not enabled, fail fast
            return false;
        }
        //此时mState.mTint已经在setTintList中赋值不为null
        final ColorStateList tintList = mState.mTint;
        //mState.mTintMode依然为默认值不为null
        final PorterDuff.Mode tintMode = mState.mTintMode;
        if (tintList != null && tintMode != null) {
            //两者都不为空，因而执行if条件下代码

            //获取当前状态下对应的颜色
            final int color = tintList.getColorForState(state, tintList.getDefaultColor());
            //mColorFilterSet默认是false
            //color即为setTint时候传入的颜色
            //mCurrentColor默认值是0
            //tintMode是mState中的mTintMode=DEFAULT_TINT_MODE = PorterDuff.Mode.SRC_IN
            //mCurrentMode默认值是null
            if (!mColorFilterSet || color != mCurrentColor || tintMode != mCurrentMode) {
                //对Drawable实例产生着色的，本质上还是执行了Drawable中的setColorFilter方法。
                setColorFilter(color, tintMode);
                mCurrentColor = color;
                mCurrentMode = tintMode;
                mColorFilterSet = true;
                return true;
            }
        } else {
            mColorFilterSet = false;
            clearColorFilter();
        }
        return false;
    }
```
- **JellybeanMr1DrawableImpl**
跟踪终点同KitKatDrawableImpl
- **HoneycombDrawableImpl**
跟踪终点同KitKatDrawableImpl
- **BaseDrawableImpl**
跟踪终点同KitKatDrawableImpl

**综上可见：
1：在SDK版本>= 21(MDrawableImpl和LollipopDrawableImpl)情况下:DrawableCompat.setTint(@NonNull Drawable drawable, @ColorInt int tint)执行的是Drawable原生的setTint方法；
2：其余情况下，DrawableCompat.setTint(@NonNull Drawable drawable, @ColorInt int tint)本质上还是执行了Drawable中的setColorFilter方法；**

####4:原生Drawable.setTint(@ColorInt int tintColor)
```
public void setTint(@ColorInt int tintColor) {
    setTintList(ColorStateList.valueOf(tintColor));
}
public void setTintList(@Nullable ColorStateList tint) {
    //你没有看错，竟然是个空方法！！！！
}
```
刚看到这儿时候也有些纳闷，后来一想肯定是我们在获取Drawable原始实例的时，获取的其实是Drawable的子类实例，在Drawable子类里对setTintList做了重写，有图有真相：
![setTintList重写.png](http://upload-images.jianshu.io/upload_images/3501388-fb81a15c80b0226c.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/480)
####5:Drawable.mutate()的作用
```
    /**
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
```
单纯看源码解释可能比较抽象，说的通俗一点，我们通过Resource获取mipmap文件夹下的一张资源图片，在获取Drawable初始实例时候如果不使用mutate()，那么我们对这个Drawable进行着色，不仅改变了当前Drawable实例的颜色，以后任何通过这个图片获取到的Drawable实例，都会具有之前设置的颜色。所以如果我们对一张资源图片的着色不是APP全局生效的，就需要使用mutate()。

**具体原因：**
Android为了优化系统性能，同一张资源图片生成的Drawable实例在内存中只存在一份，在不使用mutate的情况下，修改任意Drawable都会全局发生变化。
使用mutate，Android系统也没有把Drawable实例又单独拷贝一份，仅仅是单独存放了状态值，很小的一部分数据，Drawable实例在内存中仍然保持1份，因而并不会影响系统的性能。
具体变化可以通过2张图片说明：
**1:不使用mutate:**
![共享状态.png](http://upload-images.jianshu.io/upload_images/3501388-a8849472c631a770.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/480)
**2:使用mutate:**
![不共享状态.png](http://upload-images.jianshu.io/upload_images/3501388-e9897dd5b5bdb8e9.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/480)


以上就是个人分析的一点结果，若有错误，请各位同学留言告知！

**That's all !**
