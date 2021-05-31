> 在看TextView源码时候又看到了这两个接口：Spannable和Editable；

之前一直没有认真研究过两者的关系，现在看了源码记录下来。

#### 1：两者属于继承关系，Editable继承于Spannable
> 
Editable：
![Editable继承关系.png](http://upload-images.jianshu.io/upload_images/3501388-cd41a8895fc0d45d.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
Spannable:
![Spannable.png](http://upload-images.jianshu.io/upload_images/3501388-456d44fbfba15d66.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

相较于Spannable,Editable还继承了另2个接口：CharSequence，Appendable。
CharSequence大家应该比较熟，看一下Appendable：
> 
![Appendable.png](http://upload-images.jianshu.io/upload_images/3501388-5cee048b15eedd3e.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

由图可见，**Appendable**这个接口，主要用来**向CharSequence 添加/插入新的文本**，通过其定义的方法可以看出其作用：
- append(CharSequence csq)
- append(CharSequence csq, int start, int end)
- append(char c)

#### 2：Spannable中主要方法
- setSpan(Object what, int start, int end, int flags)
  - 这个方法我们经常用，用于向文本设置/添加新的样式
- removeSpan(Object what)
  - 移除指定的样式，作用和setSpan相反

由此可见，**Spannable**作用是**为CharSequence实例设置或者移除指定样式**。

#### 2：Editable中主要方法
> Editable:
![Editable源代码.png](http://upload-images.jianshu.io/upload_images/3501388-00e294d99afd18f0.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

This is the interface for text whose content and markup can be changed：
可见，**Editable接口关联的文本，不仅可以标记/设置样式，其内容也可以变化；**

#### 3：实际使用总结
- 如果一段文本，仅仅是**样式**发生变化，使用Spannable的子类**SpannableString**即可实现
- 如果一段文本，**样式和内容**都要发生变化，则必须使用Editable实例，我们最常用的应该就是**SpannableStringBuilder**.
- 调用TextView实例的setText方法时，type使用**TextView.BufferType.EDITABLE**，可以实现TextView中的文本不断的增加/更新（**比如一些场景是需要向TextView实例中不断插入从网络获取的最新数据**）
```
/**
     * Sets the text that this TextView is to display (see
     * {@link #setText(CharSequence)}) and also sets whether it is stored
     * in a styleable/spannable buffer and whether it is editable.
     *
     * @attr ref android.R.styleable#TextView_text
     * @attr ref android.R.styleable#TextView_bufferType
     */
    public void setText(CharSequence text, BufferType type) {
        setText(text, type, true, 0);

        if (mCharWrapper != null) {
            mCharWrapper.mChars = null;
        }
    }
```
示例代码：
```
    .................    
        tv_setText = (TextView) findViewById(R.id.tv_setText);
        bt_setText = (Button) findViewById(R.id.bt_setText);
        tv_setText.setText("", TextView.BufferType.EDITABLE);
        bt_setText.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {
                Editable content = (Editable) tv_setText.getText();
                content.append(":"+(insertIndex++));
            }
        });
    }
    int insertIndex = 0;
```
- 上面的效果，也可以直接使用**TextView的append**方法实现，今天刚刚看到（2017/01/23 17:19）,不需要设置TextView.BufferType.EDITABLE
```
        .................
        tv_setText = (TextView) findViewById(R.id.tv_setText);
        bt_setText = (Button) findViewById(R.id.bt_setText);
//        tv_setText.setText("", TextView.BufferType.EDITABLE);
        bt_setText.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {
                tv_setText.append(":"+(insertIndex++));
            }
        });
```
**原因：在调用append方法时候，append方法内部会自动设置为TextView.BufferType.EDITABLE**
> ![append方法.png](http://upload-images.jianshu.io/upload_images/3501388-5ffd907a2f6dc6ee.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

**That's all !**
