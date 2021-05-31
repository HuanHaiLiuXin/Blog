## 1.参考文章
- [Android中PopupWindow显示在指定位置](https://blog.csdn.net/jdsjlzx/article/details/20698455)

## 2.知识点
##### 1.常用方法
1. showAsDropDown
2. showAtLocation
	- 在屏幕居中显示PopWindow: popwindow.showAtLocation(1个View,Gravity.CENTER,0,0);
##### 2.源码
1. **PopWindow默认位置就是在锚点View的下方,'左边'对齐锚点View**,如果PopWindow的尺寸过大,则会进行偏移
```java
android/widget/PopupWindow.java

//默认对齐方式是 PopWindow在锚点View的下方,'左对齐/start对齐'
private static final int DEFAULT_ANCHORED_GRAVITY = Gravity.TOP | Gravity.START;

public void showAsDropDown(View anchor) {
    showAsDropDown(anchor, 0, 0);
}
public void showAsDropDown(View anchor, int xoff, int yoff) {
    showAsDropDown(anchor, xoff, yoff, DEFAULT_ANCHORED_GRAVITY);
}
```

## 3.本地验证
