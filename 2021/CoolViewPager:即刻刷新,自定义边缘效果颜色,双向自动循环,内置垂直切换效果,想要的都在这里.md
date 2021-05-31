![](https://user-gold-cdn.xitu.io/2018/6/6/163d2eccf66b830b?w=1400&h=157&f=png&s=15999)

这两天在GitHub上传了一个自定义ViewPager:[**CoolViewPager**](https://github.com/HuanHaiLiuXin/CoolViewPager),具有以下功能特征:
1. 支持水平及垂直方向循环滚动
2. 支持自动滚动
3. 支持自动滚动方向、滚动时间、间隔时间的设置
4. **支持调用notifyDataSetChanged实时刷新界面**
5. **支持边缘效果颜色的设置**
6. **为垂直滚动提供了适宜的界面切换效果**

![录屏GIF](https://user-gold-cdn.xitu.io/2018/6/6/163d307e7f6d00d7?w=450&h=647&f=gif&s=4812613)

## 为什么写这个库
我们平时使用support包中的ViewPager,当adapter中数据变更后,调用notifyDataSetChanged并不能刷新界面,需要重新调用ViewPager.setAdapter方法;网上所有的自定义ViewPager,几乎都没有提供垂直方向的切换效果;很多时候,我们需要变更ViewPager滑动到边缘的渐变色以配合App特定场景.CoolViewPager可以很方便的解决上述问题.

## 使用步骤
在你的build.gradle中添加依赖
```
dependencies {
    implementation 'com.huanhailiuxin.view:coolviewpager:1.0.0'
}
```
在你的布局文件中引入CoolViewPager
```
<com.huanhailiuxin.coolviewpager.CoolViewPager
    android:id="@+id/vp"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    />
```
在Java代码中获取CoolViewPager,设置各种属性,为其设置Adapter
```java
public class ActivityEdgeEffectColor extends BaseActivity {
    private CoolViewPager vp;
    
    ****
    CoolViewPager vp = findViewById(R.id.vp);
    vp.setScrollMode(CoolViewPager.ScrollMode.HORIZONTAL);
    vp.setAdapter(adapter);
    ****
}
```

## 属性:
```
<?xml version="1.0" encoding="utf-8"?>

<resources>
    <declare-styleable name="CoolViewPager">
        <attr name="cvp_scrollmode" format="enum">
            <enum name="horizontal" value="0" />
            <enum name="vertical" value="1" />
        </attr>
        <attr name="cvp_autoscroll" format="boolean" />
        <attr name="cvp_intervalinmillis" format="integer"/>
        <attr name="cvp_autoscrolldirection" format="enum">
            <enum name="forward" value="0" />
            <enum name="backward" value="1" />
        </attr>
        <attr name="cvp_infiniteloop" format="boolean" />
        <attr name="cvp_scrollduration" format="integer"/>
        <attr name="cvp_drawedgeeffect" format="boolean"/>
        <attr name="cvp_edgeeffectcolor" format="color"/>
    </declare-styleable>
</resources>
```
我们可以通过xml或Java代码的方式设置CoolViewPager实例的属性.

| attribute name | description |
|:---|:---|
| cvp_scrollmode | 滚动方向 |
| cvp_autoscroll | 是否开启自动滚动 |
| cvp_intervalinmillis | 自动滚动时间间隔 |
| cvp_autoscrolldirection | 自动滚动方向 |
| cvp_infiniteloop | 是否循环滚动 |
| cvp_scrollduration | 自动滚动耗时 |
| cvp_drawedgeeffect | 是否绘制边缘效果 |
| cvp_edgeeffectcolor | 绘制的边缘效果颜色 |

#### 通过XML布局文件
```
<com.huanhailiuxin.coolviewpager.CoolViewPager
    android:id="@+id/vp"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    app:cvp_scrollmode="vertical"
    app:cvp_autoscroll="true"
    app:cvp_intervalinmillis="1000"
    app:cvp_autoscrolldirection="backward"
    app:cvp_infiniteloop="true"
    app:cvp_scrollduration="600"
    app:cvp_drawedgeeffect="true"
    app:cvp_edgeeffectcolor="@color/colorPrimary"
    />
```
#### 通过Java代码
```java
public class ActivityEdgeEffectColor extends BaseActivity {
    private CoolViewPager vp;
    
    private void initViewPager(){
        vp = findViewById(R.id.vp);
        vp.setScrollMode(CoolViewPager.ScrollMode.VERTICAL);
        vp.setAutoScroll(true,1000);
        vp.setAutoScrollDirection(CoolViewPager.AutoScrollDirection.BACKWARD);
        vp.setInfiniteLoop(true);
        vp.setScrollDuration(true,600);
        vp.setDrawEdgeEffect(true);
        vp.setEdgeEffectColor(getResources().getColor(R.color.colorPrimary));
    }
}

```
