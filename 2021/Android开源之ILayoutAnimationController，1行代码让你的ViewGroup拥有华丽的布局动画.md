*直接上动图：*
![ILayoutAnimationController录屏.gif](http://upload-images.jianshu.io/upload_images/3501388-ce815cd39cf99b67.gif?imageMogr2/auto-orient/strip)
源码及DEMO已上传至**GitHub:**[**ILayoutAnimationController**](https://github.com/HuanHaiLiuXin/ILayoutAnimationController),欢迎大家提Bug,喜欢的话记得Star或Fork下哈!

#### 1：ILayoutAnimationController是什么？其实现思路是？
- LayoutAnimationController大家应该都了解，应用于ViewGroup实例的布局动画，但Android原生布局动画，仅支持顺序、倒序、随机3种动画执行顺序！
- ILayoutAnimationController是一个自定义LayoutAnimationController，通过重写其**getTransformedIndex**方法，**1行代码即可任意定制布局动画的执行顺序**，实现不同展示效果！

#### 2：使用方法：
###### 方法一：1行代码直接搞定,以下两种方法任选其一

- **ILayoutAnimationController.setLayoutAnimation**(@NonNull ViewGroup viewGroup, **@NonNull Animation animation**, float delay, @Nullable final IndexAlgorithm indexAlgorithm)

- **ILayoutAnimationController.setLayoutAnimation**(@NonNull ViewGroup viewGroup,**@AnimRes int animResId**, float delay,@Nullable final IndexAlgorithm indexAlgorithm)

###### 方法二：首先创建ILayoutAnimationController实例，然后将此实例作为参数为ViewGroup设置布局动画

- **ILayoutAnimationController.generateController**(@NonNull Animation animation, float delay, @Nullable final IndexAlgorithm indexAlgorithm)

- **ViewGroup.setLayoutAnimation**(LayoutAnimationController controller)

#### 3：示例代码：
```java
LinearLayout ll = (LinearLayout) findViewById(R.id.ll);
//两行代码设置布局动画：
ILayoutAnimationController controller = 
  ILayoutAnimationController.generateController(
    AnimationUtils.loadAnimation(this,R.anim.activity_open_enter),
    0.8f,
    ILayoutAnimationController.IndexAlgorithm.INDEXSIMPLEPENDULUM);
ll.setLayoutAnimation(controller);

//一行代码直接搞定：
ILayoutAnimationController.setLayoutAnimation(
    ll,
    R.anim.activity_open_enter,
    0.8f,
    ILayoutAnimationController.IndexAlgorithm.INDEXSIMPLEPENDULUM);
```
#### 4：ILayoutAnimationController部分代码：
```
public class ILayoutAnimationController extends LayoutAnimationController {
    private Callback onIndexListener;
    public void setOnIndexListener(Callback onIndexListener) {
        this.onIndexListener = onIndexListener;
    }
    public ILayoutAnimationController(Context context, AttributeSet attrs) {
        super(context, attrs);
    }
    public ILayoutAnimationController(Animation animation) {
        super(animation);
    }
    public ILayoutAnimationController(Animation animation, float delay) {
        super(animation, delay);
    }
    @Override
    protected int getTransformedIndex(AnimationParameters params) {
        if (onIndexListener!=null){
            return onIndexListener.getTransformedIndex(
              this,
              params.count,
              params.index);
        }else{
            return super.getTransformedIndex(params);
        }
    }
    /**
     * callback for get play animation order
     */
    public interface Callback{
        public int getTransformedIndex(
          ILayoutAnimationController controller, 
          int count, 
          int index);
    }
    /**
     * 根据当前枚举类型的值，确定在setOnIndexListener方法中的
     * CustomLayoutAnimationController.Callback
     * 实例的getTransformedIndex方法调用GetTransformedIndexUtils中的那种方法
     */
    public enum IndexAlgorithm{
        INDEX1325476,
        INDEX135246,
        INDEX246135,
        INDEXSIMPLEPENDULUM,
        MIDDLETOEDGE,
        INDEX15263748,
        INDEX1325476REVERSE,
        INDEX135246REVERSE,
        INDEX246135REVERSE,
        INDEXSIMPLEPENDULUMREVERSE,
        INDEXMIDDLETOEDGEREVERSE,
        INDEX15263748REVERSE
    }
    public static ILayoutAnimationController generateController(
        Animation animation, float delay){
        return generateController(animation,delay,null);
    }
    /**
     * 根据指定的动画、单个子View动画延时、
     * 子View动画执行顺序算法枚举值，
     * 创建一个新的CustomLayoutAnimationController实例
     * @param animation the animation to use on each child of the view group
     * @param delay the delay by which each child's animation must be offset
     * @param indexAlgorithm 子View动画执行顺序算法枚举值
     * @return
     */
    public static ILayoutAnimationController generateController(
        @NonNull Animation animation, 
        float delay, 
        @Nullable final IndexAlgorithm indexAlgorithm){
        ILayoutAnimationController controller = 
          new ILayoutAnimationController(animation,delay);
        controller.setOnIndexListener(new Callback() {
            @Override
            public int getTransformedIndex(
              ILayoutAnimationController controller, int count, int index) {
                if(indexAlgorithm != null){
                    switch (indexAlgorithm){
                        case INDEX1325476:
                            return GetTransformedIndexUtils
                                    .getTransformedIndex1325476(count,index);
                        case INDEX135246:
                            return GetTransformedIndexUtils
                                    .getTransformedIndex135246(count,index);
                        case INDEX246135:
                            return GetTransformedIndexUtils
                                    .getTransformedIndex246135(count,index);
                        case INDEXSIMPLEPENDULUM:
                            return GetTransformedIndexUtils
                                    .getTransformedIndexSimplePendulum(
                                      count,index);
                        case MIDDLETOEDGE:
                            return GetTransformedIndexUtils
                                    .getTransformedIndexMiddleToEdge(
                                      count,index);
                        case INDEX15263748:
                            return GetTransformedIndexUtils
                                    .getTransformedIndex15263748(count,index);
                        case INDEX1325476REVERSE:
                            return GetTransformedIndexUtils
                                    .getTransformedIndex1325476REVERSE(
                                      count,index);
                        case INDEX135246REVERSE:
                            return GetTransformedIndexUtils
                                    .getTransformedIndex135246REVERSE(
                                      count,index);
                        case INDEX246135REVERSE:
                            return GetTransformedIndexUtils
                                    .getTransformedIndex246135REVERSE(
                                      count,index);
                        case INDEXSIMPLEPENDULUMREVERSE:
                            return GetTransformedIndexUtils
                                    .getTransformedIndexSimplePendulumREVERSE(
                                      count,index);
                        case INDEXMIDDLETOEDGEREVERSE:
                            return GetTransformedIndexUtils
                                    .getTransformedIndexMiddleToEdgeREVERSE(
                                      count,index);
                        case INDEX15263748REVERSE:
                            return GetTransformedIndexUtils
                                    .getTransformedIndex15263748REVERSE(
                                      count,index);
                        default:
                            break;
                    }
                }
                return index;
            }
        });
        return controller;
    }

    /**
     * 根据指定的动画、单个子View动画延时、
     * 子View动画执行顺序算法枚举值，
     * 创建一个新的CustomLayoutAnimationController实例，
     * 将此实例作为参数为viewGroup设置布局动画
     * @param viewGroup
     * @param animation
     * @param delay
     * @param indexAlgorithm
     */
    public static void setLayoutAnimation(
        @NonNull ViewGroup viewGroup, 
        @NonNull Animation animation, 
        float delay, 
        @Nullable final IndexAlgorithm indexAlgorithm){
        ILayoutAnimationController controller = 
          generateController(animation,delay,indexAlgorithm);
        viewGroup.setLayoutAnimation(controller);
    }

    /**
     * 根据传入的动画资源ID、单个子View动画延时、
     * 子View动画执行顺序算法枚举值，
     * 创建一个新的CustomLayoutAnimationController实例，
     * 将此实例作为参数为viewGroup设置布局动画
     * @param viewGroup
     * @param animResId
     * @param delay
     * @param indexAlgorithm
     */
    public static void setLayoutAnimation(
        @NonNull ViewGroup viewGroup,
        @AnimRes int animResId, 
        float delay,
        @Nullable final IndexAlgorithm indexAlgorithm){
        Animation animation = AnimationUtils.loadAnimation(
          viewGroup.getContext(),animResId);
        setLayoutAnimation(viewGroup,animation,delay,indexAlgorithm);
    }
}

public final class GetTransformedIndexUtils {
**略**
    /**
     * 先执行第1项 的布局动画,
     * 然后执行第3项 的布局动画,然后执行第2项 的布局动画,
     * 然后执行第5项 的布局动画,然后执行第4项 的布局动画,
     * 然后执行第7项 的布局动画,然后执行第6项 的布局动画---
     * @param count
     * @param index
     * @return
     */
    public static int getTransformedIndex1325476(int count, int index){
        if ((index + 1) % 2 != 0) {
            if (index == 0) {
                return index;
            } else {
                return index - 1;
            }
        } else {
            if (index == count - 1) {
                return index;
            } else {
                return index + 1;
            }
        }
    }

    /**
     * 先执行奇数项 的布局动画,再执行偶数项 的布局动画
     * @param count
     * @param index
     * @return
     */
    public static int getTransformedIndex135246(int count, int index){
        if (index%2==0){
            //1:奇数项
            return index/2;
        }else {
            //2:偶数项
            if(count%2==0){
                //2.1:当总项数是偶数
                return (count/2-1) + (index+1)/2;
            }else {
                //2.2:当总项数是奇数
                return (count/2) + (index+1)/2;
            }
        }
    }
**略**
}
```

#### 注意：
- *使用ILayoutAnimationController获取的ILayoutAnimationController实例，调用setOrder(int order)方法无效！*

#### LayoutAnimation基础知识：
就不再详述，不太了解的同学可以看看这篇博客,CSDN上随意搜就好多：
[Android 动画之LayoutAnimation](http://blog.csdn.net/IO_Field/article/details/53100803)
* * * 

源码及DEMO已上传至**GitHub:**[**ILayoutAnimationController**](https://github.com/HuanHaiLiuXin/ILayoutAnimationController),欢迎大家提Bug,喜欢的话记得Star或Fork下哈!

**That's all !**
