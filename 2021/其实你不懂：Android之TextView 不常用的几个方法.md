这篇文章记录一下TextView中不常用的几个方法，直接上动图：
![TextView不常用方法效果.gif](http://upload-images.jianshu.io/upload_images/3501388-d01857e545c21acb.gif?imageMogr2/auto-orient/strip)

##### setTextIsSelectable(boolean selectable)：
> setTextIsSelectable(boolean selectable)对应xml中的android:textIsSelectable，用于声明TextView中的内容是否可被选中。

setTextIsSelectable截图：
![可选.png](http://upload-images.jianshu.io/upload_images/3501388-6f3547bccc90ed65.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/310)

##### setSelectAllOnFocus(boolean selectAllOnFocus)：
>setSelectAllOnFocus(boolean selectAllOnFocus)对应xml中的android:selectAllOnFocus，用于声明TextView/EditText实例在获取焦点后是否选中全部内容。

setSelectAllOnFocus截图：
![可选+全选.png](http://upload-images.jianshu.io/upload_images/3501388-57f6601b705adcc5.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/310)

##### setHighlightColor(@ColorInt int color)：
> setHighlightColor(@ColorInt int color)对应xml中的android:textColorHighlight，用于声明TextView中被选中内容的高亮背景色。

setHighlightColor截图：
![可选+选中高亮背景色.png](http://upload-images.jianshu.io/upload_images/3501388-f851cade89c1c98f.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/310)

##### 链接相关:
*setAutoLinkMask(int mask)*
> setAutoLinkMask(int mask)对应xml中的android:autoLink，用于声明TextView实例可识别的链接类型。

*setLinkTextColor(@ColorInt int color)*
> setLinkTextColor(@ColorInt int color)对应xml中的android:textColorLink，用于设置TextView实例中链接的颜色。

*setLinksClickable(boolean whether)*
> setLinksClickable(boolean whether)对应xml中的android:linksClickable，用于设置TextView实例中的链接是否可点击/点击是否执行对应动作。

##### setTextScaleX(float size)
> setTextScaleX(float size)对应xml中的android:textScaleX，用于设置TextView实例中文字横向拉伸倍数。

setTextScaleX截图（黄色高亮区域）：
![文字横向拉伸倍数高亮标记.png](http://upload-images.jianshu.io/upload_images/3501388-c422982632259998.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/310)

##### 设置TextView高度为行数相关：
*setMinLines(int minlines)*
> setMinLines(int minlines)对应xml中的android:minLines，用于设置TextView最小高度为指定行高度。

*setMaxLines(int maxlines)*
> setMaxLines(int maxlines)对应xml中的android:maxLines，用于设置TextView最大高度为指定行高度。

*setLines(int lines)*
> setLines(int lines)对应xml中的android:lines，用于设置TextView精确高度为指定行高度。

设置TextView高度为行数相关截图（黄色高亮区域）：
![行数高亮.png](http://upload-images.jianshu.io/upload_images/3501388-95e4fcfc43e8df86.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/310)

##### android:password
> android:password 用于设置TextView中内容是否为明文。

##### setShadowLayer(float radius, float dx, float dy, int color)
> setShadowLayer(float radius, float dx, float dy, int color)对应xml中的android:shadowRadius，android:shadowDx，android:shadowDy，android:shadowColor，用于设置文字阴影。

setShadowLayer截图（黄色高亮区域）：
![阴影高亮.png](http://upload-images.jianshu.io/upload_images/3501388-52b50e536f5bbd6c.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/310)

##### setEllipsize(TextUtils.TruncateAt where)
> setEllipsize(TextUtils.TruncateAt where)对应xml中的android:ellipsize，用于文字省略样式设置。

##### setCompoundDrawablePadding(int pad)
> setCompoundDrawablePadding(int pad)对应xml中的android:drawablePadding，用于设置TextView中文字与四周drawable的间距。

setCompoundDrawablePadding截图（黄色高亮区域）：
![drawablePadding高亮.png](http://upload-images.jianshu.io/upload_images/3501388-fa90b3678f1b997c.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/310)

##### setCompoundDrawableTintList(@Nullable ColorStateList tint)
> setCompoundDrawableTintList(@Nullable ColorStateList tint)对应xml中的android:drawableTint，用于设置TextView四周drawable的着色。

**设置TextView的四周图片的颜色，只有在SDK>=23情况下有效！
低版本兼容方法： {@link Drawable#setColorFilter(int, PorterDuff.Mode)}方法，在java代码中手动为四周图片进行着色
**
```
示例：

经试验在Android 4.22上，以下方法无效：
DrawableCompat.setTint(d1,Color.RED);
DrawableCompat.setTintMode(d1, PorterDuff.Mode.SRC_ATOP);

正确做法：
Drawable d1 = getResources().getDrawable(R.mipmap.i1).mutate();
d1.setBounds(0,0,64,64);
d1.setColorFilter(Color.RED, PorterDuff.Mode.SRC_ATOP);
Drawable d2 = getResources().getDrawable(R.mipmap.i2).mutate();
d2.setBounds(0,0,64,64);
d2.setColorFilter(Color.RED, PorterDuff.Mode.SRC_ATOP);
Drawable[] drawables = tv_drawableTint.getCompoundDrawables();
drawables[0] = d1;
drawables[2] = d2;
tv_drawableTint.setCompoundDrawables(drawables[0],drawables[1],drawables[2],drawables[3]);
```

##### setLineSpacing(float add, float mult)：
> setLineSpacing(float add, float mult)对应xml中的android:lineSpacingMultiplier和android:lineSpacingExtra，用于设置多行TextView中单行高度。

setLineSpacing截图（黄色高亮区域）：
![多行内容中单行高度黄色高亮.png](http://upload-images.jianshu.io/upload_images/3501388-82633c9f12c1766c.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/600)

##### setLetterSpacing(float letterSpacing)
> setLetterSpacing(float letterSpacing)对应xml中的android:letterSpacing，用于设置文本的字符间距。

setLetterSpacing截图（黄色高亮区域）：
![字符间距黄色高亮.png](http://upload-images.jianshu.io/upload_images/3501388-cfbb7bf1dfc4a936.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/600)

##### setAllCaps(boolean allCaps)
> setAllCaps(boolean allCaps)对应xml中的android:textAllCaps，用于设置TextView中的字符是否全部显示为大写形式。

##### setTransformationMethod(TransformationMethod method)
> setTransformationMethod(TransformationMethod method)用于设置TextView展示内容与实际内容之间的转换规则。

#####setTextSize(int unit, float size)
> setTextSize(int unit, float size)用于设置TextView实例的文字尺寸为指定单位unit的指定值size。

##### xml及Java代码:
```
<?xml version="1.0" encoding="utf-8"?>
<ScrollView
    xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:tools="http://schemas.android.com/tools"
    android:id="@+id/activity_article1"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    >
    <LinearLayout
        android:layout_width="match_parent"
        android:layout_height="match_parent"
        android:paddingBottom="@dimen/activity_vertical_margin"
        android:paddingLeft="@dimen/activity_horizontal_margin"
        android:paddingRight="@dimen/activity_horizontal_margin"
        android:paddingTop="@dimen/activity_vertical_margin"
        android:orientation="vertical"
        >
        <TextView
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:hint="textIsSelectable:文字是否可被选中"
            />
        <TextView
            android:id="@+id/tv_textIsSelectable"
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:layout_marginTop="8dp"
            android:layout_marginBottom="8dp"
            android:textColor="@android:color/black"
            android:text="测试文字是否可被选中\n测试文字是否可被选中"
            android:textIsSelectable="true"
            />
        <View
            android:layout_width="match_parent"
            android:layout_height="1px"
            android:background="@color/colorAccent"
            />
        <TextView
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:hint="selectAllOnFocus:文字获取焦点后是否被全选"
            />
        <EditText
            android:id="@+id/tv_selectAllOnFocus"
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:layout_marginTop="8dp"
            android:layout_marginBottom="8dp"
            android:textColor="@android:color/black"
            android:text="文字获取焦点后是否被全选"
            android:textIsSelectable="true"
            android:selectAllOnFocus="true"
            />
        <View
            android:layout_width="match_parent"
            android:layout_height="1px"
            android:background="@color/colorAccent"
            />
        <TextView
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:hint="textColorHighlight:文字被选中时的高亮背景色"
            />
        <TextView
            android:id="@+id/tv_textColorHighlight"
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:layout_marginTop="8dp"
            android:layout_marginBottom="8dp"
            android:textColor="@android:color/black"
            android:text="文字被选中时的高亮背景色\n文字被选中时的高亮背景色"
            android:textIsSelectable="true"
            android:textColorHighlight="#36B34A"
            />
        <View
            android:layout_width="match_parent"
            android:layout_height="1px"
            android:background="@color/colorAccent"
            />
        <TextView
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:hint="链接相关:autoLink+textColorLink+linksClickable"
            />
        <TextView
            android:id="@+id/tv_link"
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:layout_marginTop="8dp"
            android:layout_marginBottom="8dp"
            android:textColor="@android:color/black"
            android:text="http://baidu.com\n为啥耳机福克斯\n2346726\nhsfkd@55.com"
            android:autoLink="all"
            android:textColorLink="@color/colorPrimary"
            android:linksClickable="true"
            />
        <View
            android:layout_width="match_parent"
            android:layout_height="1px"
            android:background="@color/colorAccent"
            />
        <TextView
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:hint="textScaleX:文字横向拉伸倍数"
            />
        <TextView
            android:id="@+id/tv_textScaleX"
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:layout_marginTop="8dp"
            android:layout_marginBottom="8dp"
            android:textColor="@android:color/black"
            android:text="hgjkshgkhsdkfgjldfjhgkhshjg;ld"
            android:textScaleX="2.6"
            />
        <View
            android:layout_width="match_parent"
            android:layout_height="1px"
            android:background="@color/colorAccent"
            />
        <TextView
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:hint="minLines:设置TextView最小高度为指定行高度\n当前:minLines=3"
            />
        <TextView
            android:id="@+id/minLines"
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:layout_marginTop="8dp"
            android:layout_marginBottom="8dp"
            android:textColor="@android:color/black"
            android:text="文字只有一行"
            android:minLines="3"
            />
        <View
            android:layout_width="match_parent"
            android:layout_height="1px"
            android:background="@color/colorAccent"
            />
        <TextView
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:hint="maxLines:设置TextView最大高度为指定行高度\n当前:maxLines=2 文字有5行"
            />
        <TextView
            android:id="@+id/maxLines"
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:layout_marginTop="8dp"
            android:layout_marginBottom="8dp"
            android:textColor="@android:color/black"
            android:text="第一行文字\n第二行文字\n第三行文字\n第四行文字\n第五行文字"
            android:maxLines="2"
            />
        <View
            android:layout_width="match_parent"
            android:layout_height="1px"
            android:background="@color/colorAccent"
            />
        <TextView
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:hint="lines:设置TextView精确高度为指定行高度\n当前:lines=2 文字有5行"
            />
        <TextView
            android:id="@+id/lines"
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:layout_marginTop="8dp"
            android:layout_marginBottom="8dp"
            android:textColor="@android:color/black"
            android:text="第一行文字\n第二行文字\n第三行文字\n第四行文字\n第五行文字"
            android:lines="2"
            />
        <View
            android:layout_width="match_parent"
            android:layout_height="1px"
            android:background="@color/colorAccent"
            />
        <TextView
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:hint="password:设置内容是否为明文"
            />
        <TextView
            android:id="@+id/password"
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:layout_marginTop="8dp"
            android:layout_marginBottom="8dp"
            android:textColor="@android:color/black"
            android:text="测试文字测试文字测试文字测试文字测试文字"
            android:password="true"
            />
        <View
            android:layout_width="match_parent"
            android:layout_height="1px"
            android:background="@color/colorAccent"
            />
        <TextView
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:hint="shadow:阴影相关"
            />
        <TextView
            android:id="@+id/tv_shadow"
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:layout_marginTop="8dp"
            android:layout_marginBottom="8dp"
            android:textColor="@android:color/black"
            android:text="测试文字测试文字测试文字测试文字测试文字"
            android:shadowDx="12.0"
            android:shadowDy="16.0"
            android:shadowColor="#EA2000"
            android:shadowRadius="8.0"
            />
        <View
            android:layout_width="match_parent"
            android:layout_height="1px"
            android:background="@color/colorAccent"
            />
        <TextView
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:hint="ellipsize:文字省略样式设置"
            />
        <TextView
            android:id="@+id/tv_ellipsize"
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:layout_marginTop="8dp"
            android:layout_marginBottom="8dp"
            android:textColor="@android:color/black"
            android:text="测试文字测试文字测试文字测试文字测试文字测试文字测试文字测试文字测试文字测试文字测试文字测试文字测试文字测试文字测试文字测试文字测试文字测试文字测试文字测试文字"
            android:maxLines="1"
            />
        <View
            android:layout_width="match_parent"
            android:layout_height="1px"
            android:background="@color/colorAccent"
            />
        <TextView
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:hint="drawablePadding:设置文字与TextView四周drawable的间距"
            />
        <TextView
            android:id="@+id/tv_drawablePadding"
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:layout_marginTop="8dp"
            android:layout_marginBottom="8dp"
            android:textColor="@android:color/black"
            android:text="测试文字测试文字"
            android:drawableLeft="@mipmap/ic_launcher"
            android:drawablePadding="32dp"
            />
        <View
            android:layout_width="match_parent"
            android:layout_height="1px"
            android:background="@color/colorAccent"
            />
        <TextView
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:hint="drawableTint:设置TextView四周drawable的着色"
            />
        <TextView
            android:id="@+id/tv_drawableTint"
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:layout_marginTop="8dp"
            android:layout_marginBottom="8dp"
            android:textColor="@android:color/black"
            android:text="测试文字测试文字"
            android:gravity="center"
            android:drawableLeft="@mipmap/ic_launcher"
            android:drawableTop="@mipmap/ic_launcher"
            android:drawableRight="@mipmap/ic_launcher"
            android:drawableBottom="@mipmap/ic_launcher"
            android:drawableTint="#3A92FF"
            />
        <View
            android:layout_width="match_parent"
            android:layout_height="1px"
            android:background="@color/colorAccent"
            />
        <TextView
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:hint="lineSpacing:设置多行TextView中单行高度"
            />
        <TextView
            android:id="@+id/tv_lineSpacing_original"
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:textColor="@android:color/black"
            android:text="测试文字测试文字测试文字测试文字\n测试文字测试文字测试文字测试文字\n测试文字测试文字测试文字测试文字"
            android:lineSpacingMultiplier="1.0"
            android:lineSpacingExtra="0dp"
            />
        <View
            android:layout_width="match_parent"
            android:layout_height="1px"
            android:background="@android:color/darker_gray"
            />
        <TextView
            android:id="@+id/tv_lineSpacing_setting"
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:textColor="@android:color/black"
            android:text="测试文字测试文字测试文字测试文字\n测试文字测试文字测试文字测试文字\n测试文字测试文字测试文字测试文字"
            android:lineSpacingMultiplier="2.0"
            android:lineSpacingExtra="16dp"
            />
        <View
            android:layout_width="match_parent"
            android:layout_height="1px"
            android:background="@color/colorAccent"
            />
        <TextView
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:hint="letterSpacing:设置文本的字符间距"
            />
        <TextView
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:layout_marginTop="8dp"
            android:layout_marginBottom="8dp"
            android:textColor="@android:color/black"
            android:text="letterSpacing=2.0"
            android:letterSpacing="2.0"
            />
        <TextView
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:layout_marginTop="8dp"
            android:layout_marginBottom="8dp"
            android:textColor="@android:color/black"
            android:text="letterSpacing=-0.2"
            android:letterSpacing="-0.2"
            />
        <View
            android:layout_width="match_parent"
            android:layout_height="1px"
            android:background="@color/colorAccent"
            />
        <TextView
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:hint="textAllCaps:文字是否全部大写形式"
            />
        <TextView
            android:id="@+id/tv_textAllCaps"
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:layout_marginTop="8dp"
            android:layout_marginBottom="8dp"
            android:textColor="@android:color/black"
            android:text="ghkldjgkldjlkldffjgjfdslkfh;hkslkhfgkdj"
            android:textAllCaps="true"
            />
        <View
            android:layout_width="match_parent"
            android:layout_height="1px"
            android:background="@color/colorAccent"
            />
        <TextView
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:hint="setTransformationMethod:设置TextView展示内容与实际内容之间的转换规则"
            />
        <TextView
            android:id="@+id/tv_SetTransformationMethod"
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:layout_marginTop="8dp"
            android:layout_marginBottom="8dp"
            android:textColor="@android:color/black"
            android:text="abcdefghijklmn"
            />
        <View
            android:layout_width="match_parent"
            android:layout_height="1px"
            android:background="@color/colorAccent"
            />
        <TextView
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:hint="setTextSize(int unit, float size):设置文字尺寸为指定单位unit的指定值size"
            />
        <TextView
            android:id="@+id/tv_setTextSize"
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:layout_marginTop="8dp"
            android:layout_marginBottom="8dp"
            android:textColor="@android:color/black"
            android:text="setTextSize(int unit, float size)"
            />
        <View
            android:layout_width="match_parent"
            android:layout_height="1px"
            android:background="@color/colorAccent"
            />
    </LinearLayout>

</ScrollView>
```
```
public class Article1Activity extends AppCompatActivity {
    private TextView tv_ellipsize = null;
    int index = 0;
    private TextUtils.TruncateAt[] ellipsizes = new TextUtils.TruncateAt[]{
            TextUtils.TruncateAt.START,
            TextUtils.TruncateAt.MIDDLE,
            TextUtils.TruncateAt.END
    };
    Handler handlerEllipsize = new Handler(){
        @Override
        public void handleMessage(Message msg) {
            index = ++index%ellipsizes.length;
            tv_ellipsize.setEllipsize(ellipsizes[index]);
            handlerEllipsize.sendMessageDelayed(handlerEllipsize.obtainMessage(),2000);
        }
    };
    Handler handlerTransformationMethod = new Handler(){
        @Override
        public void handleMessage(Message msg) {
            int what = 0;
            if(msg.what == 0){
                //大写
                tv_SetTransformationMethod.setTransformationMethod(new MyTransformationMethod(true));
                what = 1;
            }else {
                //小写
                tv_SetTransformationMethod.setTransformationMethod(new MyTransformationMethod());
                what = 0;
            }
            handlerTransformationMethod.sendEmptyMessageDelayed(what,2000);
        }
    };
    private TextView tv_SetTransformationMethod = null;
    private int[] units = new int[]{
            TypedValue.COMPLEX_UNIT_PX,
            TypedValue.COMPLEX_UNIT_DIP,
            TypedValue.COMPLEX_UNIT_SP
    };
    Handler handlerSetTextSize = new Handler(){
        @Override
        public void handleMessage(Message msg) {
            index = ++index%units.length;
            tv_setTextSize.setTextSize(units[index],24.0f);
            handlerSetTextSize.sendMessageDelayed(handlerSetTextSize.obtainMessage(),2000);
        }
    };
    private TextView tv_setTextSize = null;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_article1);
        tv_ellipsize = (TextView) findViewById(R.id.tv_ellipsize);
        handlerEllipsize.sendMessage(handlerEllipsize.obtainMessage());
        tv_SetTransformationMethod = (TextView) findViewById(R.id.tv_SetTransformationMethod);
        handlerTransformationMethod.sendEmptyMessage(0);
        tv_setTextSize = (TextView) findViewById(R.id.tv_setTextSize);
        handlerSetTextSize.sendEmptyMessage(0);
    }
    class MyTransformationMethod implements TransformationMethod {
        private boolean upperCase = false;
        public MyTransformationMethod(boolean ... upperCase){
            this.upperCase = (upperCase==null||upperCase.length<=0)?false:upperCase[0];
        }
        @Override
        public CharSequence getTransformation(CharSequence source, View view) {
            if(this.upperCase){
                return TextUtils.isEmpty(source)?"":source.toString().toUpperCase(getResources().getConfiguration().locale)+"大写";
            }else{
                return TextUtils.isEmpty(source)?"":source.toString().toLowerCase(getResources().getConfiguration().locale)+"小写";
            }
        }
        @Override
        public void onFocusChanged(View view, CharSequence sourceText, boolean focused, int direction, Rect previouslyFocusedRect) {
        }
    }
}
```

以上就是个人实验和分析的一点结果，若有错误，请各位同学留言告知！

**That's all !**
