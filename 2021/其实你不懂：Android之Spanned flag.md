>年前的这几天，一直在看代码，发现自己之前对很多技术点的理解都不全面，甚至根本就是想当然。所以有这样一个想法，要把之前理解很粗浅的东西慢慢整理一遍，补一下技术上的短板。

>这个过程会比较长，准备**把涉及到的不同技术点写一个系列文章，系列名字**嘛，就直白点：**其实你不懂**，是不是很直白O(∩_∩)O~

[**其实你不懂：Android之Spanned flag**](http://www.jianshu.com/p/1956e15c9a27)就作为**'其实你不懂'系列的开篇！**

### 1：Spanned flag都有哪些可选值
- SPAN_POINT_MARK_MASK
- SPAN_MARK_MARK
- SPAN_MARK_POINT
- SPAN_POINT_MARK
- SPAN_POINT_POINT
- SPAN_PARAGRAPH
- SPAN_PARAGRAPH
- **SPAN_INCLUSIVE_EXCLUSIVE（常用）**
- **SPAN_INCLUSIVE_INCLUSIVE（常用）**
- **SPAN_EXCLUSIVE_EXCLUSIVE（常用）**
- **SPAN_EXCLUSIVE_INCLUSIVE（常用）**
- SPAN_COMPOSING
- SPAN_INTERMEDIATE
- SPAN_USER_SHIFT
- SPAN_USER
- SPAN_PRIORITY_SHIFT
- SPAN_PRIORITY

### 2：Spanned flag常用值的效果
- SPAN_INCLUSIVE_INCLUSIVE
  - 前后都包括，在指定范围前后插入新字符，都会应用新样式
- SPAN_EXCLUSIVE_EXCLUSIVE
  - 前后都不包括，在指定范围前后插入新字符，样式无变化
- SPAN_INCLUSIVE_EXCLUSIVE
  - 前面包括，后面不包括
- SPAN_EXCLUSIVE_INCLUSIVE
  - 后面包括，前面不包括

### 3：不同的值flag，当EditText文本内容发生改变，表现有何区别？（不包含SPAN_POINT_MARK_MASK，SPAN_PARAGRAPH）
实验初始设置：不同的EditText初始的文本都是"一二三四五六七八九十"构造的SpannableString，都使用相同的AbsoluteSizeSpan，ForegroundColorSpan实例设置文本样式，区别只在于flag的值。
```
//设置初始内容
String str = "一二三四五六七八九十";
//1:构造SpannableString实例
SpannableString spannableStringEE = new SpannableString(str);
SpannableString spannableStringII = new SpannableString(str);
SpannableString spannableStringEI = new SpannableString(str);
SpannableString spannableStringIE = new SpannableString(str);
//2:构造指定类型的Span,这里使用AbsoluteSizeSpan,ForegroundColorSpan
AbsoluteSizeSpan sizeSpan = new AbsoluteSizeSpan(24,true);
ForegroundColorSpan colorSpan = new ForegroundColorSpan(Color.RED);
//3:将构造的AbsoluteSizeSpan实例,ForegroundColorSpan实例应用于SpannableString实例，只有 flags 有区别
```
##### 不同的EditText实例，初始内容如下：
![原始1.png](http://upload-images.jianshu.io/upload_images/3501388-aed39983b4dae297.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/310)
![原始2.png](http://upload-images.jianshu.io/upload_images/3501388-88df23ad5ae7c3cd.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/310)
##### 3.1：向EditText中继续插入文字：
![插入文字1.png](http://upload-images.jianshu.io/upload_images/3501388-fa0254f291748ff8.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/310)
![插入文字2.png](http://upload-images.jianshu.io/upload_images/3501388-9caa17e04cf4b769.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/310)
###### 通过图片可以得到结论，单纯插入文字：
SPAN_POINT_MARK 作用等同于 SPAN_INCLUSIVE_INCLUSIVE；
SPAN_POINT_MARK 作用等同于 SPAN_EXCLUSIVE_EXCLUSIVE；
SPAN_POINT_POINT 作用等同于 SPAN_EXCLUSIVE_INCLUSIVE；
SPAN_COMPOSING 关联的EditText中所有内容都失去新样式；
其余flag值作用等同于 SPAN_INCLUSIVE_EXCLUSIVE；
##### 3.2：将EditText中内容完全删除后重新输入：
![完全删除后重新输入.png](http://upload-images.jianshu.io/upload_images/3501388-1de7abf316742905.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/310)
![完全删除后重新输入.gif](http://upload-images.jianshu.io/upload_images/3501388-065244148ffae0fa.gif?imageMogr2/auto-orient/strip)
看GIF动图可能表述的更清晰。
###### 通过图片可以得到结论，内容完全删除后重新输入：
SPAN_POINT_MARK  和 SPAN_INCLUSIVE_INCLUSIVE 关联的EditText中所有内容都会保持新样式；
其余所有的flag值，都会失去新样式；
##### 3.3：按照EditText中新样式的起止值，进行文字替换：
![按照原始的起止位置进行文字不等长替换.png](http://upload-images.jianshu.io/upload_images/3501388-f66ff54f65b92c7e.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/310)
![按照原始的起止位置进行文字不等长替换.gif](http://upload-images.jianshu.io/upload_images/3501388-496a0449ea06fd2d.gif?imageMogr2/auto-orient/strip)
###### 通过图片可以得到结论，按照EditText中新样式的起止值，进行文字替换：
SPAN_COMPOSING 关联的EditText中所有内容都失去新样式；
其余任意flag关联的EditText,替换来的文字都保持了新样式；
##### 3.4：新替换文字的起止值范围大于EditText中新样式的起止值，进行文字替换：
![起止位置超过原始起止位置替换.gif](http://upload-images.jianshu.io/upload_images/3501388-d8251df5dc81b246.gif?imageMogr2/auto-orient/strip)
###### 通过图片可以得到结论，新替换文字的起止值范围大于EditText中新样式的起止值，进行文字替换：
SPAN_POINT_MARK  和 SPAN_INCLUSIVE_INCLUSIVE 关联的EditText，新替换文字会保持新样式；
其余所有的flag值，都会失去新样式；

### 4：SPAN_POINT_MARK_MASK，SPAN_PARAGRAPH，当EditText文本内容发生改变，表现有何区别？
##### 4.1：为什么这两个flag值的表现要单独分析？
我在实验过程中，出现过2个异常，描述如下：
> PARAGRAPH span must start at paragraph boundary
> PARAGRAPH span must end at paragraph boundary

后来看了SDK中这两个flag值的描述，在java代码中进行实验，理解了这两个flag值使用的特殊条件：
- start值必须为 0（SpannableString的开头）或 \n(换行符)所在的索引值+1；
- end必须要为 \n(换行符)所在的索引值+1 或 SpannableString的结尾字符索引值+1(SpannableString的长度值)
- start值有误  :报异常   PARAGRAPH span must start at paragraph boundary
- end值有误    :报异常   PARAGRAPH span must end at paragraph boundary
- 同样，要是start<0或者end>=length,也会报错。

怎么理解呢？就是这两个flag值，对于新样式的设定，是针对于“**段落**”的，何为“段落”？就是EditText中**1:所有的文本**或者**2:完整的1行或者多行**。如果你在调用SpannableString的setSpan方法时候，start和end的值所指定的文本不属于**1:所有的文本**或者**2:完整的1行或者多行**，那么就会报上面提到的异常。
> setSpan(Object what, int start, int end, int flags)）]

**此处的‘行’，指的是\n(换行符)引起的换行；**

看了下面的实际演示，应该更容易理解。
##### 4.2：EditText中的文本，完全删除后重新输入
![P-完全删除后重新输入.gif](http://upload-images.jianshu.io/upload_images/3501388-57ae091bab5938db.gif?imageMogr2/auto-orient/strip)
###### 通过图片可以得到结论，EditText中的文本，完全删除后重新输入：
新输入的文本未使用新样式；
##### 4.3：在EditText实例新样式的左侧，进行不完全的替换
![P-起始小于原始起始终止大于原始起始小于原始终止 替换.gif](http://upload-images.jianshu.io/upload_images/3501388-0109f3dedac1e797.gif?imageMogr2/auto-orient/strip)
###### 通过图片可以得到结论，在EditText实例新样式的左侧，进行不完全的替换：
替换的文字所涉及到的完整的的新样式丢失；
替换的文字没有涉及到的完整行，新样式依然保留；
##### 4.4：在EditText实例新样式的右侧，进行不完全的替换
![P-起始大于原始起始小于原始终止终止大于原始终止 替换.gif](http://upload-images.jianshu.io/upload_images/3501388-d1c53063dfd94bdc.gif?imageMogr2/auto-orient/strip)
###### 通过图片可以得到结论，在EditText实例新样式的右侧，进行不完全的替换：
替换文字起始位置所在的完整行，新样式依然保留，
且
新样式一直到延伸到替换文字终止位置所在的完整行；
### 5：涉及到的代码
**java文件：**
```
public class SpannedActivity extends AppCompatActivity implements View.OnClickListener{
    private ScrollView activitySpanned;
//    private EditText tvSpanExclusiveExclusive;
//    private EditText tvSpanInclusiveInclusive;
//    private EditText tvSpanExclusiveInclusive;
//    private EditText tvSpanInclusiveExclusive;
    private EditText tv_span_point_mark_mask;
//    private EditText tv_span_mark_mark;
//    private EditText tv_span_mark_point;
//    private EditText tv_span_point_mark;
//    private EditText tv_span_point_point;
    private EditText tv_span_paragraph;
//    private EditText tv_span_composing;
//    private EditText tv_span_intermediate;
//    private EditText tv_span_user_shift;
//    private EditText tv_span_user;
//    private EditText tv_span_priority_shift;
//    private EditText tv_span_priority;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        requestWindowFeature(Window.FEATURE_NO_TITLE);
        getWindow().setFlags(WindowManager.LayoutParams.FLAG_FULLSCREEN,WindowManager.LayoutParams.FLAG_FULLSCREEN);
        setContentView(R.layout.activity_spanned);
        initViews();
    }

    private void initViews() {
        activitySpanned = (ScrollView) findViewById(R.id.activity_spanned);
//        tvSpanExclusiveExclusive = (EditText)findViewById(R.id.tv_span_exclusive_exclusive);
//        tvSpanInclusiveInclusive = (EditText) findViewById(R.id.tv_span_inclusive_inclusive);
//        tvSpanExclusiveInclusive = (EditText) findViewById(R.id.tv_span_exclusive_inclusive);
//        tvSpanInclusiveExclusive = (EditText) findViewById(R.id.tv_span_inclusive_exclusive);
        tv_span_point_mark_mask = (EditText) findViewById(R.id.tv_span_point_mark_mask);
//        tv_span_mark_mark = (EditText) findViewById(R.id.tv_span_mark_mark);
//        tv_span_mark_point = (EditText) findViewById(R.id.tv_span_mark_point);
//        tv_span_point_mark = (EditText) findViewById(R.id.tv_span_point_mark);
//        tv_span_point_point = (EditText) findViewById(R.id.tv_span_point_point);
        tv_span_paragraph = (EditText) findViewById(R.id.tv_span_paragraph);
//        tv_span_composing = (EditText) findViewById(R.id.tv_span_composing);
//        tv_span_intermediate = (EditText) findViewById(R.id.tv_span_intermediate);
//        tv_span_user_shift = (EditText) findViewById(R.id.tv_span_user_shift);
//        tv_span_user = (EditText) findViewById(R.id.tv_span_user);
//        tv_span_priority_shift = (EditText) findViewById(R.id.tv_span_priority_shift);
//        tv_span_priority = (EditText) findViewById(R.id.tv_span_priority);

        //设置初始内容
        String str = "一二三四五六七八九十";
        //1:构造SpannableString实例
        SpannableString spannableStringEE = new SpannableString(str);
        SpannableString spannableStringII = new SpannableString(str);
        SpannableString spannableStringEI = new SpannableString(str);
        SpannableString spannableStringIE = new SpannableString(str);
        //2:构造指定类型的Span,这里使用 AbsoluteSizeSpan,ForegroundColorSpan
        AbsoluteSizeSpan sizeSpan = new AbsoluteSizeSpan(24,true);
        ForegroundColorSpan colorSpan = new ForegroundColorSpan(Color.RED);
        //3:将构造的AbsoluteSizeSpan实例,ForegroundColorSpan实例应用于SpannableString实例，只有 flags 有区别
        /**
        {@link SpannableString#setSpan(Object, int, int, int)}
         */
        spannableStringEE.setSpan(sizeSpan,2,4, Spanned.SPAN_EXCLUSIVE_EXCLUSIVE);
        spannableStringII.setSpan(sizeSpan,2,4, Spanned.SPAN_INCLUSIVE_INCLUSIVE);
        spannableStringEI.setSpan(sizeSpan,2,4, Spanned.SPAN_EXCLUSIVE_INCLUSIVE);
        spannableStringIE.setSpan(sizeSpan,2,4, Spanned.SPAN_INCLUSIVE_EXCLUSIVE);
        spannableStringEE.setSpan(colorSpan,2,4, Spanned.SPAN_EXCLUSIVE_EXCLUSIVE);
        spannableStringII.setSpan(colorSpan,2,4, Spanned.SPAN_INCLUSIVE_INCLUSIVE);
        spannableStringEI.setSpan(colorSpan,2,4, Spanned.SPAN_EXCLUSIVE_INCLUSIVE);
        spannableStringIE.setSpan(colorSpan,2,4, Spanned.SPAN_INCLUSIVE_EXCLUSIVE);
        //4:将SpannableString实例应用于对应的EditText实例
//        tvSpanExclusiveExclusive.setText(spannableStringEE);
//        tvSpanInclusiveInclusive.setText(spannableStringII);
//        tvSpanExclusiveInclusive.setText(spannableStringEI);
//        tvSpanInclusiveExclusive.setText(spannableStringIE);
        SpannableString spannableStringPMM = new SpannableString("一二三四五六七八九十\n一二三四五六七八九十\n一二三四五六七八九十\n一二三四五六七八九十\n一二三四五六七八九十");
        spannableStringPMM.setSpan(sizeSpan,11,33, Spanned.SPAN_POINT_MARK_MASK);
        spannableStringPMM.setSpan(colorSpan,11,33, Spanned.SPAN_POINT_MARK_MASK);
        tv_span_point_mark_mask.setText(spannableStringPMM);
        SpannableString spannableStringMM = new SpannableString(str);
        spannableStringMM.setSpan(sizeSpan,2,4, Spanned.SPAN_MARK_MARK);
        spannableStringMM.setSpan(colorSpan,2,4, Spanned.SPAN_MARK_MARK);
//        tv_span_mark_mark.setText(spannableStringMM);
        SpannableString spannableStringMP = new SpannableString(str);
        spannableStringMP.setSpan(sizeSpan,2,4, Spanned.SPAN_MARK_POINT);
        spannableStringMP.setSpan(colorSpan,2,4, Spanned.SPAN_MARK_POINT);
//        tv_span_mark_point.setText(spannableStringMP);
        SpannableString spannableStringPM = new SpannableString(str);
        spannableStringPM.setSpan(sizeSpan,2,4, Spanned.SPAN_POINT_MARK);
        spannableStringPM.setSpan(colorSpan,2,4, Spanned.SPAN_POINT_MARK);
//        tv_span_point_mark.setText(spannableStringPM);
        SpannableString spannableStringPP = new SpannableString(str);
        spannableStringPP.setSpan(sizeSpan,2,4, Spanned.SPAN_POINT_POINT);
        spannableStringPP.setSpan(colorSpan,2,4, Spanned.SPAN_POINT_POINT);
//        tv_span_point_point.setText(spannableStringPP);
        SpannableString spannableStringParagraph = new SpannableString("一二三四五六七八九十\n一二三四五六七八九十\n一二三四五六七八九十\n一二三四五六七八九十\n一二三四五六七八九十");
        spannableStringParagraph.setSpan(sizeSpan,11,33, Spanned.SPAN_PARAGRAPH);
        spannableStringParagraph.setSpan(colorSpan,11,33, Spanned.SPAN_PARAGRAPH);
        tv_span_paragraph.setText(spannableStringParagraph);
        SpannableString spannableStringComposing = new SpannableString(str);
        spannableStringComposing.setSpan(sizeSpan,2,4, Spanned.SPAN_COMPOSING);
        spannableStringComposing.setSpan(colorSpan,2,4, Spanned.SPAN_COMPOSING);
//        tv_span_composing.setText(spannableStringComposing);
        SpannableString spannableStringIntermediate = new SpannableString(str);
        spannableStringIntermediate.setSpan(sizeSpan,2,4, Spanned.SPAN_INTERMEDIATE);
        spannableStringIntermediate.setSpan(colorSpan,2,4, Spanned.SPAN_INTERMEDIATE);
//        tv_span_intermediate.setText(spannableStringIntermediate);
        SpannableString spannableStringUserShift = new SpannableString(str);
        spannableStringUserShift.setSpan(sizeSpan,2,4, Spanned.SPAN_USER_SHIFT);
        spannableStringUserShift.setSpan(colorSpan,2,4, Spanned.SPAN_USER_SHIFT);
//        tv_span_user_shift.setText(spannableStringUserShift);
        SpannableString spannableStringUser = new SpannableString(str);
        spannableStringUser.setSpan(sizeSpan,2,4, Spanned.SPAN_USER);
        spannableStringUser.setSpan(colorSpan,2,4, Spanned.SPAN_USER);
//        tv_span_user.setText(spannableStringUser);
        SpannableString spannableStringPriorityShift = new SpannableString(str);
        spannableStringPriorityShift.setSpan(sizeSpan,2,4, Spanned.SPAN_PRIORITY_SHIFT);
        spannableStringPriorityShift.setSpan(colorSpan,2,4, Spanned.SPAN_PRIORITY_SHIFT);
//        tv_span_priority_shift.setText(spannableStringPriorityShift);
        SpannableString spannableStringPriority = new SpannableString(str);
        spannableStringPriority.setSpan(sizeSpan,2,4, Spanned.SPAN_PRIORITY);
        spannableStringPriority.setSpan(colorSpan,2,4, Spanned.SPAN_PRIORITY);
//        tv_span_priority.setText(spannableStringPriority);
    }

    @Override
    public void onClick(View view) {
        switch (view.getId()) {
            default:
                break;
        }
    }
}
```
**xml文件：**
```
<?xml version="1.0" encoding="utf-8"?>
<ScrollView xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:tools="http://schemas.android.com/tools"
    android:id="@+id/activity_spanned"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:fillViewport="true"
    android:paddingBottom="@dimen/activity_vertical_margin"
    android:paddingLeft="@dimen/activity_horizontal_margin"
    android:paddingRight="@dimen/activity_horizontal_margin"
    android:paddingTop="@dimen/activity_vertical_margin"
    tools:context="com.jet.coordbehsnack.SpannedActivity">
    <LinearLayout
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:orientation="vertical">
        <!--
        <TextView android:textColor="@android:color/black"
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:text="SPAN_EXCLUSIVE_EXCLUSIVE" />
        <EditText
            android:id="@+id/tv_span_exclusive_exclusive"
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:hint="SPAN_EXCLUSIVE_EXCLUSIVE" />
        <TextView android:textColor="@android:color/black"
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:text="SPAN_INCLUSIVE_INCLUSIVE" />
        <EditText
            android:id="@+id/tv_span_inclusive_inclusive"
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:hint="SPAN_INCLUSIVE_INCLUSIVE" />
        <TextView android:textColor="@android:color/black"
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:text="SPAN_EXCLUSIVE_INCLUSIVE" />

        <EditText
            android:id="@+id/tv_span_exclusive_inclusive"
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:hint="SPAN_EXCLUSIVE_INCLUSIVE" />
        <TextView android:textColor="@android:color/black"
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:text="SPAN_INCLUSIVE_EXCLUSIVE" />
        <EditText
            android:id="@+id/tv_span_inclusive_exclusive"
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:hint="SPAN_INCLUSIVE_EXCLUSIVE" />
        -->
        <TextView android:textColor="@android:color/black"
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:text="SPAN_POINT_MARK_MASK" />
        <EditText
            android:id="@+id/tv_span_point_mark_mask"
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:hint="SPAN_POINT_MARK_MASK" />
        <!--
        <TextView android:textColor="@android:color/black"
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:text="SPAN_MARK_MARK" />
        <EditText
            android:id="@+id/tv_span_mark_mark"
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:hint="SPAN_MARK_MARK" />
        <TextView android:textColor="@android:color/black"
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:text="SPAN_MARK_POINT" />
        <EditText
            android:id="@+id/tv_span_mark_point"
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:hint="SPAN_MARK_POINT" />
        <TextView android:textColor="@android:color/black"
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:text="SPAN_POINT_MARK" />
        <EditText
            android:id="@+id/tv_span_point_mark"
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:hint="SPAN_POINT_MARK" />
        <TextView android:textColor="@android:color/black"
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:text="SPAN_POINT_POINT" />
        <EditText
            android:id="@+id/tv_span_point_point"
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:hint="SPAN_POINT_POINT" />
        -->
        <TextView android:textColor="@android:color/black"
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:text="SPAN_PARAGRAPH" />
        <EditText
            android:id="@+id/tv_span_paragraph"
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:hint="SPAN_PARAGRAPH" />
        <!--
        <TextView android:textColor="@android:color/black"
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:text="SPAN_COMPOSING" />
        <EditText
            android:id="@+id/tv_span_composing"
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:hint="SPAN_COMPOSING" />
        <TextView android:textColor="@android:color/black"
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:text="SPAN_INTERMEDIATE" />
        <EditText
            android:id="@+id/tv_span_intermediate"
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:hint="SPAN_INTERMEDIATE" />
        <TextView android:textColor="@android:color/black"
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:text="SPAN_USER_SHIFT" />
        <EditText
            android:id="@+id/tv_span_user_shift"
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:hint="SPAN_USER_SHIFT" />
        <TextView android:textColor="@android:color/black"
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:text="SPAN_USER" />
        <EditText
            android:id="@+id/tv_span_user"
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:hint="SPAN_USER" />
        <TextView android:textColor="@android:color/black"
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:text="SPAN_PRIORITY_SHIFT" />
        <EditText
            android:id="@+id/tv_span_priority_shift"
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:hint="SPAN_PRIORITY_SHIFT" />
        <TextView android:textColor="@android:color/black"
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:text="SPAN_PRIORITY" />
        <EditText
            android:id="@+id/tv_span_priority"
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:hint="SPAN_PRIORITY" />
        -->
    </LinearLayout>
</ScrollView>
```
以上就是个人实验和分析的一点结果，若有错误，请各位同学留言告知！

希望能给大家一点点的收获!也希望‘其实你不懂’系列文章真的可以写完并得到大家的指教和喜欢！

**That's all !**
