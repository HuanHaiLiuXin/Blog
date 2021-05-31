[Android Drawable完全解析（一）：Drawable源码分析（上）](http://www.jianshu.com/p/384a70897ba6)
<br>
[Android Drawable完全解析（一）：Drawable源码分析（中）](http://www.jianshu.com/p/2213c62e4738)
<br>
[Android Drawable完全解析（一）：Drawable源码分析（下）](http://www.jianshu.com/p/c56b762210f2)
<br>

昨天下班前，分析了View实例将Drawable作为背景绘制到屏幕上面的流程，今天继续分析Drawable在ImageView中的绘制流程！

#### 3:Drawable绘制流程
#### 3.3:Drawable在ImageView中的绘制流程
ImageView使用Drawable的方式大体以下几种：
- 在xml中直接设置android:background="@mipmap/voice"
        <ImageView
            android:layout_width="200dp"
            android:layout_height="100dp"
            android:background="@mipmap/voice"
            />
- 在xml中直接设置android:src="@mipmap/voice"
        <ImageView
            android:layout_width="200dp"
            android:layout_height="100dp"
            android:src="@mipmap/voice"
            />
- Java代码中调用 setImageResource(@DrawableRes int resId)
- Java代码中调用 setImageDrawable(@Nullable Drawable drawable)
- Java代码中调用 setBackgroundDrawable，实质是调用View.setBackgroundDrawable，上篇文章已分析。

下面就这几种方式逐一分析：
首先上原图：
![voice.png](http://upload-images.jianshu.io/upload_images/3501388-0f4d8f75ec497a8b.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
#### 3.3.1：android:background="@mipmap/voice"
```
        <ImageView
            android:layout_width="200dp"
            android:layout_height="100dp"
            android:background="@mipmap/voice"
            />
```
实际效果：
![background.png](http://upload-images.jianshu.io/upload_images/3501388-078eeff90fd12c85.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/300)

可见直接使用android:background，图片作为背景完全铺满ImageView尺寸，会根据ImageView的范围缩放。
既然在xml中布局ImageView，那么肯定是调用**ImageView(Context context, @Nullable AttributeSet attrs)**，看一下关键代码：
```java
public class ImageView extends View {
    public ImageView(Context context, @Nullable AttributeSet attrs) {
        this(context, attrs, 0);
    }
****
    public ImageView(Context context, @Nullable AttributeSet attrs, int defStyleAttr,
            int defStyleRes) {
        //调用View的构造函数
        super(context, attrs, defStyleAttr, defStyleRes);
        ****
    }
}
一路追踪下去：
public class View implements Drawable.Callback, KeyEvent.Callback,AccessibilityEventSource {
    public View(Context context, @Nullable AttributeSet attrs, int defStyleAttr, int defStyleRes) {
        this(context);
        final TypedArray a = context.obtainStyledAttributes(
                attrs, com.android.internal.R.styleable.View, defStyleAttr, defStyleRes);
        ****
        Drawable background = null;
        ****
        final int N = a.getIndexCount();
        for (int i = 0; i < N; i++) {
            int attr = a.getIndex(i);
            switch (attr) {
                //获取在xml中设置的android:background="@mipmap/voice"
                case com.android.internal.R.styleable.View_background:
                    background = a.getDrawable(attr);
                    break; 
                *****
            }
        }
        ****
        if (background != null) {
            setBackground(background);
        }
        ****
    }
    public void setBackground(Drawable background) {
        //[Android Drawable完全解析（一）：Drawable源码分析（中）](http://www.jianshu.com/p/2213c62e4738)
        setBackgroundDrawable(background);
    }
}
```
可见：
**在xml中直接设置android:background="@mipmap/voice"实质是通过调用View.setBackgroundDrawable(Drawable background)将图片绘制到屏幕上！**

View.setBackgroundDrawable(Drawable background)在上一篇文章：[Android Drawable完全解析（一）：Drawable源码分析（中）](http://www.jianshu.com/p/2213c62e4738)有过分析！
为什么背景图会铺满整个ImageView，是因为在View绘制过程中，将背景Drawable的绘制范围设置为和View的尺寸一致：
```
    void setBackgroundBounds() {
        if (mBackgroundSizeChanged && mBackground != null) {
            mBackground.setBounds(0, 0, mRight - mLeft, mBottom - mTop);
            mBackgroundSizeChanged = false;
            rebuildOutline();
        }
    }
```

#### 3.3.2：android:src="@mipmap/voice"
```
        <ImageView
            android:layout_width="200dp"
            android:layout_height="100dp"
            android:src="@mipmap/voice"
            />
```
实际效果：
![src.png](http://upload-images.jianshu.io/upload_images/3501388-2edc05741123da04.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/300)

可见直接使用android:src，默认情况下图片会根据ImageView的尺寸在保留自身宽高比例下进行缩放，最后在ImageView的中心显示。
既然在xml中布局ImageView，那么肯定是调用**ImageView(Context context, @Nullable AttributeSet attrs)**，同样看一下关键代码：
```java
public class ImageView extends View {
    public ImageView(Context context, @Nullable AttributeSet attrs, int defStyleAttr,
            int defStyleRes) {
        //super上面分析过了，绘制的是android:background="@mipmap/voice"
        super(context, attrs, defStyleAttr, defStyleRes);
        //主要设置了 ImageView实例中图像边界 与 ImageView边界间的缩放关系
        initImageView();
        final TypedArray a = context.obtainStyledAttributes(
                attrs, R.styleable.ImageView, defStyleAttr, defStyleRes);
        //将xml中使用android:src="@mipmap/voice"设置的图片生成Drawable实例
        final Drawable d = a.getDrawable(R.styleable.ImageView_src);
        if (d != null) {
            //将src生成的Drawable实例设置为ImageView的内容
            setImageDrawable(d);
        }
        ****
        //在我们的例子中，没有设置scaleType属性，则index = -1；
        final int index = a.getInt(R.styleable.ImageView_scaleType, -1);
        if (index >= 0) {
            //在我们例子中，index = -1，下面代码不执行
            setScaleType(sScaleTypeArray[index]);
        }
        //解析在xml中设置的tint和tintMode属性值
        if (a.hasValue(R.styleable.ImageView_tint)) {
            mDrawableTintList = a.getColorStateList(R.styleable.ImageView_tint);
            mHasDrawableTint = true;
            // Prior to L, this attribute would always set a color filter with
            // blending mode SRC_ATOP. Preserve that default behavior.
            mDrawableTintMode = PorterDuff.Mode.SRC_ATOP;
            mHasDrawableTintMode = true;
        }
        if (a.hasValue(R.styleable.ImageView_tintMode)) {
            mDrawableTintMode = Drawable.parseTintMode(a.getInt(
                    R.styleable.ImageView_tintMode, -1), mDrawableTintMode);
            mHasDrawableTintMode = true;
        }
        //根据当前ImageView的ColorStateList对 通过src生成的Drawable实例进行着色
        applyImageTint();
        final int alpha = a.getInt(R.styleable.ImageView_drawableAlpha, 255);
        //设置透明度
        if (alpha != 255) {
            setImageAlpha(alpha);
        }
        mCropToPadding = a.getBoolean(
                R.styleable.ImageView_cropToPadding, false);
        a.recycle();
        //need inflate syntax/reader for matrix
    }
    private void initImageView() {
        ****
        //设置mScaleType = ScaleType.FIT_CENTER;可见ImageView中
        //mScaleType默认就是ScaleType.FIT_CENTER
        mScaleType = ScaleType.FIT_CENTER;
        ****
    }
    public void setImageDrawable(@Nullable Drawable drawable) {
        if (mDrawable != drawable) {
            ****
            //对src生成的Drawable实例设置一系列属性
            updateDrawable(drawable);
            ****
            //最后调用invalidate()触发draw
            invalidate();
        }
    }
    private void updateDrawable(Drawable d) {
        ****
        //将ImageView实例之前关联的Drawable实例的动画监听移除，
        //并停止其已经在执行的动画，解除其所有事件
        if (mDrawable != null) {
            mDrawable.setCallback(null);
            unscheduleDrawable(mDrawable);
            if (isAttachedToWindow()) {
                mDrawable.setVisible(false, false);
            }
        }
        //将mDrawable赋值为通过src属性生成的Drawable实例
        mDrawable = d;
        if (d != null) {
            //为通过src生成的Drawable实例设置动画监听为ImageView实例自身；
            //并设置其布局方向，状态数组，Drawable动画是否开启，Drawable的level值。
            d.setCallback(this);
            d.setLayoutDirection(getLayoutDirection());
            if (d.isStateful()) {
                d.setState(getDrawableState());
            }
            if (isAttachedToWindow()) {
                d.setVisible(getWindowVisibility() == VISIBLE && isShown(), true);
            }
            d.setLevel(mLevel);
            mDrawableWidth = d.getIntrinsicWidth();
            mDrawableHeight = d.getIntrinsicHeight();
            //根据当前ImageView实例的ColorStateList对其进行着色
            applyImageTint();
            //未执行实质代码
            applyColorMod();
            //设置Drawable实例的绘制范围不变，并根据ImageView实例内容区域和
            //Drawable实例原始绘制范围，确定Drawable实例在实际绘制时候的缩放。
            configureBounds();
        } else {
            mDrawableWidth = mDrawableHeight = -1;
        }
    }
    private void applyImageTint() {
        ****
                //根据当前ImageView的ColorStateList对 通过src生成的Drawable实例进行着色
                mDrawable.setTintList(mDrawableTintList);
        ****
    }
    private void applyColorMod() {
        //对应通过 src生成的Drawble实例来说，ImageView并未调用setColorFilter
        //mColorMod也为默认的false值，所以下面代码实质未执行
        if (mDrawable != null && mColorMod) {
            mDrawable = mDrawable.mutate();
            //如果当前ImageView实例调用过setColorFilter，
            //则对 通过src生成的Drawable实例设置相同的ColorFilter
            if (mHasColorFilter) {
                mDrawable.setColorFilter(mColorFilter);
            }
            mDrawable.setXfermode(mXfermode);
            mDrawable.setAlpha(mAlpha * mViewAlphaScale >> 8);
        }
    }
    private void configureBounds() {
        //通过src生成的Drawable实例 原始宽高
        final int dwidth = mDrawableWidth;
        final int dheight = mDrawableHeight;
        //ImageView实例的内容区域宽高(去除了padding值)
        final int vwidth = getWidth() - mPaddingLeft - mPaddingRight;
        final int vheight = getHeight() - mPaddingTop - mPaddingBottom;
        ****
        if (dwidth <= 0 || dheight <= 0 || ScaleType.FIT_XY == mScaleType) {
            //当ImageView设置过android:scaleType="fitXY" 或setScaleType(ScaleType.FIT_XY)，
            //则将此Drawable实例的绘制范围设定为ImageView实例的内容区域
            mDrawable.setBounds(0, 0, vwidth, vheight);
            mDrawMatrix = null;
        } else {
            //对应我们例子中，未设置android:scaleType情况下，
            //通过src生成的Drawable实例的绘制范围就是其原始范围
            mDrawable.setBounds(0, 0, dwidth, dheight);
            //下面代码设置了mDrawMatrix的属性
            //在initImageView()方法中已知：
            //ImageView中mScaleType默认就是ScaleType.FIT_CENTER
            if (ScaleType.MATRIX == mScaleType) {
                ****
            } else if (fits) {
                ****
            } else if (ScaleType.CENTER == mScaleType) {
                ****
            } else if (ScaleType.CENTER_CROP == mScaleType) {
                ****
            } else if (ScaleType.CENTER_INSIDE == mScaleType) {
                ****
            } else {
                //ImageView中mScaleType默认就是ScaleType.FIT_CENTER
                //则根据ImageView实例内容区域的范围和Drawable实例实际宽高来设置mDrawMatrix
                mTempSrc.set(0, 0, dwidth, dheight);
                mTempDst.set(0, 0, vwidth, vheight);
                mDrawMatrix = mMatrix;
                mDrawMatrix.setRectToRect(mTempSrc, mTempDst, scaleTypeToScaleToFit(mScaleType));
            }
        }
    }
}

ImageView实例生成后，肯定还是执行onDraw方法将自身绘制到屏幕上，继续追踪代码：

public class ImageView extends View {
    @Override
    protected void onDraw(Canvas canvas) {
        super.onDraw(canvas);
        ****
        //在上面分析过 mDrawMatrix不为null
        //mDrawMatrix的属性根据ImageView实例内容区域的范围和Drawable实例实际宽高来配置
        if (mDrawMatrix == null && mPaddingTop == 0 && mPaddingLeft == 0) {
            //如果矩阵mDrawMatrix为空，且ImageView的上下padding值都为0
            //则直接将Drawable实例绘制到画布上
            mDrawable.draw(canvas);
        } else {
            ****
            //我们例子中，矩阵mDrawMatrix不为空，则将其设置到ImageView的画布上
            if (mDrawMatrix != null) {
                canvas.concat(mDrawMatrix);
            }
            //然后在画布上面绘制Drawable实例
            mDrawable.draw(canvas);
            canvas.restoreToCount(saveCount);
        }
    }
}
```
至此，
**android:src="@mipmap/voice"整个流程就分析完了，流程总结如下：**

###### *在ImageView构造函数中：*
**1：设置缩放类型默认为 ScaleType.FIT_CENTER（图像居中等比例缩放）**
**2：在ImageView构造函数中，解析xml中android:src属性获取Drawable实例；**
**3：为生成的Drawable实例设置一系列属性：**
- 设置动画监听为ImageView实例自身：d.setCallback(this);
- 设置布局方向和ImageView实例一致：d.setLayoutDirection(getLayoutDirection())；
- 设置状态数组和ImageView实例一致：d.setState(getDrawableState())；
- 设置动画是否可见和 ImageView可见性一致：d.setVisible(getWindowVisibility() == VISIBLE && isShown(), true);
- 设置动画当前Level值和ImageView的mLevel值一致：d.setLevel(mLevel);
- 根据当前ImageView实例的ColorStateList对其进行着色：applyImageTint();
- 设置绘制范围为原始绘制范围setBounds 且 根据ImageView、Drawable实例的范围 和 缩放类型 来设置Matrix mDrawMatrix（用于onDraw）：configureBounds();

**4：如果我们在xml中还设置了缩放类型，着色，着色模式，透明度，
则为mScaleType重新赋值，并为生成的Drawable实例逐一设置着色，着色模式，透明度**

###### *在ImageView的onDraw方法中：*
**1：如果矩阵mDrawMatrix为空，且ImageView的上下padding值都为0，则直接将Drawable实例绘制到画布上**
**2：其余情况下：**
- **如果矩阵mDrawMatrix不为空，则将其设置到ImageView的画布上；**
- **然后在画布上面绘制Drawable实例**

**本质上还是执行了Drawable.draw(@NonNull Canvas canvas)将src生成的Drawable实例绘制到ImageView实例所在的画布**

#### 3.3.3：setImageDrawable(@Nullable Drawable drawable)
setImageDrawable在上面分析过程中出现过
```
    public void setImageDrawable(@Nullable Drawable drawable) {
        if (mDrawable != drawable) {
            mResource = 0;
            mUri = null;
            final int oldWidth = mDrawableWidth;
            final int oldHeight = mDrawableHeight;
            updateDrawable(drawable);
            if (oldWidth != mDrawableWidth || oldHeight != mDrawableHeight) {
                requestLayout();
            }
            //invalidate会引发重绘，调用onDraw方法，直接看上面onDraw的流程分析即可
            invalidate();
        }
    }
```
#### 3.3.4：setImageResource(@DrawableRes int resId)
```
    public void setImageResource(@DrawableRes int resId) {
        ****
        //在updateDrawable(Drawable d)中：mDrawable = d;
        //此处将mDrawable重置为null
        updateDrawable(null);
        //为mResource赋值为传入的资源ID，mUri重置为null
        mResource = resId;
        mUri = null;
        resolveUri();
        ****
        //引发重绘
        invalidate();
    }
    private void resolveUri() {
        //updateDrawable(null)已经将mDrawable重置为null
        if (mDrawable != null) {
            return;
        }
        if (getResources() == null) {
            return;
        }
        Drawable d = null;
        //在中setImageResource(@DrawableRes int resId)已知：mResource = resId;
        if (mResource != 0) {
            //通过setImageResource传入的resId通常不为0，执行如下：
            try {
                //通过传入的图片资源ID获取Drawable实例
                d = mContext.getDrawable(mResource);
            } catch (Exception e) {
                Log.w(LOG_TAG, "Unable to find resource: " + mResource, e);
                // Don't try again.
                mUri = null;
            }
        } else if (mUri != null) {
            d = getDrawableFromUri(mUri);
            if (d == null) {
                Log.w(LOG_TAG, "resolveUri failed on bad bitmap uri: " + mUri);
                // Don't try again.
                mUri = null;
            }
        } else {
            return;
        }
        //将通过传入的图片资源ID生成的Drawable实例作为参数，
        //调用updateDrawable，上面已经分析过此方法
        updateDrawable(d);
    }
```
**setImageResource(@DrawableRes int resId)整个流程就分析完了，流程总结如下：**
- **1：首先执行updateDrawable(null)，将已经绘制完毕的mDrawable动画停止，移除所有事件及动画监听，并重置为null**
- **2：用Resource实例通过resId获取Drawable实例，作为参数执行updateDrawable**
- **3：ImageView实例执行重绘，详见之前onDraw的分析**

***
*至此，Drawable在ImageView中的绘制流程就分析完毕了！Drawable源码分析也告一段落，如有错误或者翻译问题请各位大神留言！*



**'Android Drawable完全解析' 系列未完待续...**
