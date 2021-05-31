### 1. Android常见输入inputType类型
```xml
android:inputType="none"//输入普通字符
android:inputType="text"//输入普通字符
android:inputType="textCapCharacters"//输入普通字符
android:inputType="textCapWords"//单词首字母大小
android:inputType="textCapSentences"//仅第一个字母大小
android:inputType="textAutoCorrect"//前两个自动完成
android:inputType="textAutoComplete"//前两个自动完成
android:inputType="textMultiLine"//多行输入
android:inputType="textImeMultiLine"//输入法多行（不一定支持）
android:inputType="textNoSuggestions"//不提示
android:inputType="textUri"//URI格式
android:inputType="textEmailAddress"//电子邮件地址格式
android:inputType="textEmailSubject"//邮件主题格式
android:inputType="textShortMessage"//短消息格式
android:inputType="textLongMessage"//长消息格式
android:inputType="textPersonName"//人名格式
android:inputType="textPostalAddress"//邮政格式
android:inputType="textPassword"//密码格式
android:inputType="textVisiblePassword"//密码可见格式
android:inputType="textWebEditText"//作为网页表单的文本格式
android:inputType="textFilter"//文本筛选格式
android:inputType="textPhonetic"//拼音输入格式


//数值类型
android:inputType="number"//数字格式
android:inputType="numberSigned"//有符号数字格式
android:inputType="numberDecimal"//可以带小数点的浮点格式
android:inputType="phone"//拨号键盘
android:inputType="datetime"//日期+时间格式
android:inputType="date"//日期键盘
android:inputType="time"//时间键盘
```
1. 输入可见密码:
```java
editText.setInputType(InputType.TYPE_CLASS_TEXT|InputType.TYPE_TEXT_VARIATION_VISIBLE_PASSWORD);

//对应xml中属性:
android:inputType="text"//输入普通字符
android:inputType="textVisiblePassword"//密码可见格式
```

### 2. EditText设置只能输入 ASCII 类型字符
```java
//xml
imeOptions="flagForceAscii"

//Java
editText.setImeOptions(EditorInfo.IME_FLAG_FORCE_ASCII);
```

### 3. imeOptions
```
IME_ACTION_UNSPECIFIED. 编辑器决定Action按钮的行为
IME_ACTION_GO Action按钮将作为 “开始” 按钮。点击后跳转到输入字符的意图页面
IME_ACTION_SEARCH 执行“搜索”按钮。点击后跳转到输入字符的搜索结果页面
IME_ACTION_SEND. 执行 “发送”按钮。点击后将输入字符发送给它的目标
IME_ACTION_NEXT. Action按钮将作为next(下一个)按钮。点击后将进行下一个输入框的输入
IME_ACTION_DONE. Action按钮将作为done(完成)按钮。点击后IME输入法将会关闭
IME_ACTION_PREVIOUS. 作为”上一个”按钮。点击后将进行上一个输入框的输入
IME_FLAG_NO_FULLSCREEN. 请求IME输入法永远不要进入全屏模式
IME_FLAG_NAVIGATE_PREVIOUS. 类似IME_FLAG_NAVIGATE_NEXT， 表明这里有后退导航可以关注的兴趣点
IME_FLAG_NAVIGATE_NEXT. 表明这里有前进导航可以关注的兴趣点，类似IME_ACTION_NEXT，不过允许IME输入多行且提供前进导航。
IME_FLAG_NO_EXTRACT_UI. 请求IME输入法不要显示额外的文本UI
IME_FLAG_NO_ACCESSORY_ACTION. 和一个Action结合使用表明在全屏输入法中不作为可访问性按钮
IME_FLAG_NO_ENTER_ACTION. 多行文本将自动设置了该标志位，执行Action时为换行效果，如果未设置，IME输入法将把Enter按钮自动替换为Action按钮
IME_FLAG_FORCE_ASCII. 请求IME输入法接受ASCII字符的输入
```
[Android软键盘输入imeOptions](https://blog.csdn.net/honjane/article/details/78699002)

### 4. editText设置最大长度
```
//xml
android:maxLength = "30"

//java
InputFilter[] filters = {new InputFilter.LengthFilter(30)};
editText.setFilters(filters);
```
