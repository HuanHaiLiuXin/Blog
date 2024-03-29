## 参考文章
- [Android中自定义属性（attrs.xml，TypedArray的使用）](https://www.cnblogs.com/zhangs1986/p/3243040.html)

## Android中自定义属性的格式详解
```xml

1. reference：参考某一资源ID。
    （1）属性定义：
            <declare-styleable name = "名称">
                   <attr name = "background" format = "reference" />
            </declare-styleable>
    （2）属性使用：
             <ImageView
                     android:layout_width = "42dip"
                     android:layout_height = "42dip"
                     android:background = "@drawable/图片ID"
                     />
2. color：颜色值。
    （1）属性定义：
            <declare-styleable name = "名称">
                   <attr name = "textColor" format = "color" />
            </declare-styleable>
    （2）属性使用：
            <TextView
                     android:layout_width = "42dip"
                     android:layout_height = "42dip"
                     android:textColor = "#00FF00"
                     />
3. boolean：布尔值。
    （1）属性定义：
            <declare-styleable name = "名称">
                   <attr name = "focusable" format = "boolean" />
            </declare-styleable>
    （2）属性使用：
            <Button
                    android:layout_width = "42dip"
                    android:layout_height = "42dip"
                    android:focusable = "true"
                    />
4. dimension：尺寸值。
    （1）属性定义：
            <declare-styleable name = "名称">
                   <attr name = "layout_width" format = "dimension" />
            </declare-styleable>
    （2）属性使用：
            <Button
                    android:layout_width = "42dip"
                    android:layout_height = "42dip"
                    />
5. float：浮点值。
    （1）属性定义：
            <declare-styleable name = "AlphaAnimation">
                   <attr name = "fromAlpha" format = "float" />
                   <attr name = "toAlpha" format = "float" />
            </declare-styleable>
    （2）属性使用：
            <alpha
                   android:fromAlpha = "1.0"
                   android:toAlpha = "0.7"
                   />
6. integer：整型值。
    （1）属性定义：
            <declare-styleable name = "AnimatedRotateDrawable">
                   <attr name = "visible" />
                   <attr name = "frameDuration" format="integer" />
                   <attr name = "framesCount" format="integer" />
                   <attr name = "pivotX" />
                   <attr name = "pivotY" />
                   <attr name = "drawable" />
            </declare-styleable>
    （2）属性使用：
            <animated-rotate
                   xmlns:android = "http://schemas.android.com/apk/res/android" 
                   android:drawable = "@drawable/图片ID" 
                   android:pivotX = "50%" 
                   android:pivotY = "50%" 
                   android:framesCount = "12" 
                   android:frameDuration = "100"
                   />
7. string：字符串。
    （1）属性定义：
            <declare-styleable name = "MapView">
                   <attr name = "apiKey" format = "string" />
            </declare-styleable>
    （2）属性使用：
            <com.google.android.maps.MapView
                    android:layout_width = "fill_parent"
                    android:layout_height = "fill_parent"
                    android:apiKey = "0jOkQ80oD1JL9C6HAja99uGXCRiS2CGjKO_bc_g"
                    />
8. fraction：百分数。
    （1）属性定义：
            <declare-styleable name="RotateDrawable">
                   <attr name = "visible" />
                   <attr name = "fromDegrees" format = "float" />
                   <attr name = "toDegrees" format = "float" />
                   <attr name = "pivotX" format = "fraction" />
                   <attr name = "pivotY" format = "fraction" />
                   <attr name = "drawable" />
            </declare-styleable>
    （2）属性使用：
            <rotate  xmlns:android = "http://schemas.android.com/apk/res/android"
　　             android:interpolator = "@anim/动画ID"
                 android:fromDegrees = "0"
　　             android:toDegrees = "360"
                 android:pivotX = "200%"
                 android:pivotY = "300%"
　　             android:duration = "5000"
                 android:repeatMode = "restart"
                 android:repeatCount = "infinite"
                   />
9. enum：枚举值。
    （1）属性定义：
            <declare-styleable name="名称">
                   <attr name="orientation">
                          <enum name="horizontal" value="0" />
                          <enum name="vertical" value="1" />
                   </attr>           
            </declare-styleable>
    （2）属性使用：
            <LinearLayout
                    xmlns:android = "http://schemas.android.com/apk/res/android"
                    android:orientation = "vertical"
                    android:layout_width = "fill_parent"
                    android:layout_height = "fill_parent"
                    >
            </LinearLayout>
10. flag：位或运算。
     （1）属性定义：
             <declare-styleable name="名称">
                    <attr name="windowSoftInputMode">
                            <flag name = "stateUnspecified" value = "0" />
                            <flag name = "stateUnchanged" value = "1" />
                            <flag name = "stateHidden" value = "2" />
                            <flag name = "stateAlwaysHidden" value = "3" />
                            <flag name = "stateVisible" value = "4" />
                            <flag name = "stateAlwaysVisible" value = "5" />
                            <flag name = "adjustUnspecified" value = "0x00" />
                            <flag name = "adjustResize" value = "0x10" />
                            <flag name = "adjustPan" value = "0x20" />
                            <flag name = "adjustNothing" value = "0x30" />
                     </attr>        
             </declare-styleable>
     （2）属性使用：
            <activity
                   android:name = ".StyleAndThemeActivity"
                   android:label = "@string/app_name"
                   android:windowSoftInputMode = "stateUnspecified | stateUnchanged　|　stateHidden">
                   <intent-filter>
                          <action android:name = "android.intent.action.MAIN" />
                          <category android:name = "android.intent.category.LAUNCHER" />
                   </intent-filter>

             </activity>
     注意：
     属性定义时可以指定多种类型值。
    （1）属性定义：
            <declare-styleable name = "名称">
                   <attr name = "background" format = "reference|color" />
            </declare-styleable>
    （2）属性使用：
             <ImageView
                     android:layout_width = "42dip"
                     android:layout_height = "42dip"
                     android:background = "@drawable/图片ID|#00FF00"
                     />
```
