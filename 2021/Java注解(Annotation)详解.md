## Java注解(Annotation)详解

### 1.Annotation的概念
 > An annotation is a form of metadata, that can be added to Java source code. Classes, methods, variables, parameters and packages may be annotated. Annotations have no direct effect on the operation of the code they annotate.
- 注解是一种可以添加到Java源代码的元数据.
- 类,方法,变量,参数,包都可以被注解.
- 注解对注解的代码并没有直接的影响.
- 注解仅仅是个标记.注解之所以起作用是对其解析后做了相应的处理
### 2.Annotation分类
- 标准Annotation
    - 标准Annotation是指Java内置的三个Annnotaion:
    - @Override：用于修饰此方法覆盖了父类的方法.
    - @Deprecated：用于修饰已经过时的方法.
    - @SuppressWarnnings:用于通知java编译器禁止特定的编译警告.
- 元Annotation(注解的注解)
    - 元Annotation是用来定义Annotation的Annotation
    - 元Annotation可以定义Annotation的作用范围,使用在什么元素上等
    - 元注解共有四种@Retention, @Target, @Inherited, @Documented
- 自定义Annotation

### 3.元Annotation
- @Retention:注在其他的注解A上,用来说明A的保留范围,可选值 SOURCE（源码时），CLASS（编译时），RUNTIME（运行时），默认为 CLASS
    - SOURCE:A只保留在源码中,A会被编译期忽略.**(源码可用)**
    - CLASS:A会通过编译保存在CLASS文件中,但会被JVM在运行时忽略,运行时不可见.**(源码+CLASS可用)**
    - RUNTIME:A会被JVM获取,并在运行时通过反射获取.**(源码+CLASS+运行时均可用)**
- @Target:注在其他的注解A上,用来限制A可用修饰那些程序元素.未标注Target表示无限制,可修饰所有元素.
    - ANNOTATION_TYPE:  A可以应用到其他注解上
    - CONSTRUCTOR:  A可以使用到构造器上
    - FIELD:    A可以使用到域或属性上
    - LOCAL_VARIABLE:   A可以使用到局部变量上。
    - METHOD:   A可以使用到方法上。
    - PACKAGE:  A可以使用到包声明上。
    - PARAMETER:    A可以使用到方法的参数上
    - **TYPE: A可以使用到类,接口(包括注解),或枚举的声明上**
- @Inherited:默认情况下,父类的注解不会被子类继承.
    - Inherited注在其他的注解A上.
    - 只有当A是注解在类Class上面,Inherited才会起作用,其他任何情况下无效果.
    - 当A注解在类C上面,则C的所有子孙类,都会继承应用A注解;
- @Documented:注在其他的注解A上,A将会作为Javadoc产生的文档中的内容。注解都默认不会成为成为文档中的内容。

### 4.自定义Annotation
1. 创建自定义Annotation流程
    - public @interface 自定义注解名称
        ```
        public @interface CustomAnnotation{***}
            
        ```
    - 设置自定义Annotation的保留范围和目标,Retention和Target是最重要的两个元Anitation.
        ```
        @Retention( RetentionPolicy.RUNTIME )
        @Target( ElementType.TYPE )
        public @interface CustomAnnotation{***}
        ```
    - 设置自定义Annotation的注解参数(注解成员)
        - 注解参数支持的数据类型
            - 所有基本数据类型（int,float,boolean,byte,double,char,long,short)
            - String类型
            - Class类型
            - enum类型
            - Annotation类型
            - 以上所有类型的一维数组
        - 注解参数声明方式
            ```
            @Retention( RetentionPolicy.RUNTIME )
            @Target( ElementType.TYPE )
            public @interface CustomAnnotation{
                //注解参数类型可以是1-6中任一种,包括枚举
                public enum Skill{JAVA,ANDROID,IOS}
                Skill mySkill() default Skill.ANDROID;
                String attr1();
                //可以使用default设置默认值
                int attr2() default 100;
                //修饰符只能用public
                public boolean attr3() default false;
            }
            @Retention( RetentionPolicy.RUNTIME )
            @Target( ElementType.TYPE )
            public @interface CustomAnnotation{
                //只有一个注解参数,使用value()
                String value();
            }
            ```
            - 自定义Annotation的参数类型必须满足上一条1到6中的范围.
            - 自定义Annotation的参数访问方法只能是public,或不写.
            - 自定义Annotation的参数可以加 default 设置默认值.
            - 自定义Annotation若只有1个参数,使用value().
2. **自定义Annotation的注解参数的默认值**
    > **注解元素必须有确定的值，要么在定义注解的默认值中指定，要么在使用注解时指定，非基本类型的注解元素的值不可为null**。因此, 使用空字符串或0作为默认值是一种常用的做法。这个约束使得处理器很难表现一个元素的存在或缺失的状态，因为每个注解的声明中，所有元素都存在，并且都具有相应的值，为了绕开这个约束，我们只能定义一些特殊的值，例如空字符串或者负数，一次表示某个元素不存在，在定义注解时，这已经成为一个习惯用法。
    
    ```
    示例:
    @Target(ElementType.FIELD)
    @Retention(RetentionPolicy.RUNTIME)
    public @interface AnotherAnnotation{
        String author() default "";
        int age() default -1;
    }
    ```
3. 使用刚刚创建的自定义注解
    ```
    @CustomAnnotation(attr1 = "属性1", attr2 = 90, attr3 = true)
    public class AnnotationTestClass{
        ***
    }
    ```
### 5.Annotation解析
- 运行时 Annotation 解析
    > 运行时 Annotation 指 @Retention 为 RUNTIME 的 Annotation
    - Class,Method,Field中都有以下3个方法可以调用
        - public <T extends Annotation> T getAnnotation(Class<T> annotationClass) *按照传入的参数获取指定类型的注解。返回null说明当前元素不带有此注解。*
        - public final boolean isAnnotationPresent(Class<? extends Annotation> annotationType) *检查传入的注解是否存在于当前元素。*
        - public Annotation[] getAnnotations()  *返回该元素的所有注解，包括没有显式定义该元素上的注解。*
    - 运行时 Annotation 解析示例
        ```
        public void testCustomAnnotation() {
            try {
                Class cls = Class.forName("com.jet.annotation.AnnotationTestClass");
                CustomAnnotation customAnnotation = (CustomAnnotation)cls.getAnnotation(CustomAnnotation.class);
                System.out.println("customAnnotation mySkill:" + cus.mySkill());
                System.out.println("customAnnotation attr1:" + cus.attr1());
                System.out.println("customAnnotation attr2:" + cus.attr2());
            } catch (ClassNotFoundException e) {
                e.printStackTrace();
            }
        }
        ```
    
    
- 编译时 Annotation 解析
    > 编译时 Annotation 指 @Retention 为 CLASS 的 Annotation，甴编译器自动解析

----------

### 6.编译时Annotation解析
*编译时Annotation解析 相对复杂,下面单独进行分析*

首先申明：下面内容仅仅讨论 编译时Annotation的解析
1. 编译时Annotation的解析,是由Annotation Processor完成
2. Annotation Processor(注解处理器)
    - 注解处理器是一个在javac中的,用来在编译时扫描和处理注解的工具
    - 我们可以为特定的注解,注册自定义的注解处理器
    - 在编译期间,JVM会自动运行注册过的注解处理器
    - 一个注解的Annotation Processor,以Java代码(或者编译过的class)为输入,生成.java文件作为输出.**这意味着我们可以生成新的Java代码!这些生成的Java代码是在生成的.java文件中,新生成的.java文件会和普通的手动编写的Java源代码一样被javac编译**
3. 每一个注解处理器都是继承于AbstractProcessor,需要关注的有以下4个方法

```
public abstract class AbstractProcessor implements Processor {

    //对一些工具进行初始化
    public synchronized void init(ProcessingEnvironment processingEnv)
    
    //你在这里定义你的注解处理器注册到哪些注解上,必须指定;
    //它的返回值是一个字符串的集合，包含本处理器想要处理的注解类型的合法全称
    public Set<String> getSupportedAnnotationTypes()
    
    //指定该注解处理器使用的JAVA版本,通常返回SourceVersion.latestSupported()
    public SourceVersion getSupportedSourceVersion()
    
    //真正生成java代码的地方
    //annotations:请求处理的注解类型集合
    //roundEnv:可以让你查询出包含特定注解的被注解元素，相当于“有关全局源码的上下文环境”
    //如果返回 true，则这些注解已声明并且不要求后续 Processor 处理它们；
    //如果返回 false，则这些注解未声明并且可能要求后续 Processor 处理它们
    public abstract boolean process(Set<? extends TypeElement> annotations,RoundEnvironment roundEnv)
    
}
```
4. 自定义注解处理器,就是继承AbstractProcessor并重写上述4个方法

关于编译时Annotation解析,这里推荐一篇**文章[【Android】注解框架（三）-- 编译时注解，手写ButterKnife](https://www.jianshu.com/p/57211e053d0c)**,按照文章上面流程敲一遍代码,相信可以对自定义注解的创建及解析有一个深入的了解!

### 7.注解对App的影响
1. **运行时注解对性能有影响,编译时注解对App的性能没有影响.**
2. 运行时注解的解析完全依赖于反射,反射的效率比直接调用慢,只有过多使用运行时注解时才对效率有一定影响
3. java文件编译成.class文件。再对class文件进行打包等一系列处理。生成apk。最终才运行到我们的手机上。而编译时注解,就是在java编译生成.class文件这一步进行的操作。根本和我们的apk运行无关,不存在影响性能的问题;
4. 编译时注解库,在app的引用一般如下:
```
dependencies {
    implementation fileTree(dir: 'libs', include: ['*.jar'])
    implementation 'com.android.support:appcompat-v7:27.1.1'
    //API模块:android library
    implementation project(':butterknife')
    //Annotation模块:java library
    implementation project(':butterknife-annotations')
    //Annotation解析模块:java library
    annotationProcessor project(':butterknife-compiler')
}
```
实际在打包生成APK的过程中,只有 API模块和Annotation模块 会被打包进APK,Annotation解析模块是提供给IDE使用的,在我们APK中并不存在.

### 8.注解解析流程
注解解析流程:
1. 创建注解所在的Java Module : lib_annotation
    - 所有需要的注解都写在这个模块里
  ```
  @Retention(RetentionPolicy.SOURCE)
  @Target(ElementType.FIELD)
  public @interface BindView {
      int value();
  }
  ```
2. 创建解析注解所需的Java Module : lib_processor
	- build.gradle
  ```
  dependencies {
      implementation fileTree(dir: 'libs', include: ['*.jar'])
      //引用需要解析是注解所在Module
      implementation project(':lib_annotation')
      //javapoet 用于生成java文件
      implementation 'com.squareup:javapoet:1.11.1'
  }
  ```
	- 在package下,创建注解解析类 CustomAnnotationProcess,继承AbstractProcessor
	- 在main文件夹下
      - 创建 resources/META-INF/services/文件夹
      - 在services文件夹下创建 javax.annotation.processing.Processor 文件
      - javax.annotation.processing.Processor 中指向创建的注解解析类 CustomAnnotationProcess
      ```
      com.huanhailiuxin.a28_lib_processor.CustomAnnotationProcess
      ```
		- 最重要的部分,CustomAnnotationProcess怎么写
3. 创建生成 lib_processor 中声明的'bindClass' 的调用Android Module : lib_bind
  ```
  public final class Binding {
      public static void bind(Activity oriActivity) {
          try {
              Class oriClass = Class.forName(oriActivity.getClass().getCanonicalName());
              Class bindClass = Class.forName(oriActivity.getClass().getCanonicalName() + "$Binding");
              Constructor constructor = bindClass.getDeclaredConstructor(oriClass);
              constructor.newInstance(oriActivity);
          } catch (ClassNotFoundException e) {
              e.printStackTrace();
          } catch (IllegalAccessException e) {
              e.printStackTrace();
          } catch (InstantiationException e) {
              e.printStackTrace();
          } catch (InvocationTargetException e) {
              e.printStackTrace();
          } catch (NoSuchMethodException e) {
              e.printStackTrace();
          }

      }
  }
  ```
4. 在app模块中,引用 lib_annotation , lib_processor , lib_bind
    - 在Activity中对指定UI控件添加注解
    - 执行bind操作
  ```
  @BindView(R.id.tv_compile_annotation)
  public TextView tv_compile_annotation;
  @BindView(R.id.parent)
  public ScrollView parent;

  protected void onCreate(Bundle savedInstanceState) {
      super.onCreate(savedInstanceState);
      setContentView(R.layout.activity_compile_annotations);
      ***
      //调用绑定方法
      Binding.bind(this);
      tv_compile_annotation.setText("测试编译时注解解析");
      parent.setBackgroundColor(getResources().getColor(R.color.colorAccent));
  }

  //实际打包成APK后, lib_processor 在APK中并不存在.
  //lib_processor 的使命就是单纯解析指定注解,生成对应的**Binding 类.
  implementation project(':lib_annotation')
  implementation project(':lib_bind')
  annotationProcessor project(':lib_processor')
  ```
   */
   
## Android已有详解使用
- [Android 中注解的使用](https://juejin.im/post/6844903496114765831)
    ```java
    @IntDef({RED, BLUE, YELLOW})
    @Retention(RetentionPolicy.SOURCE)
    public @interface LightColors{};
      public static final int RED = 1;
      public static final int BLUE = 2;
      public static final int YELLOW = 3;
      public void setColor(@LightColors int color){
      }
    }
    ```
