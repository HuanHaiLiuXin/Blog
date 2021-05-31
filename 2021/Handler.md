### 零碎的点
1. 获取Message实例,尽量用 Message.obtain(Handler h) ,而不是 new Message() ,可以减少内存占用.
```java
public static Message obtain(Handler h) {
    Message m = obtain();
    m.target = h;
    return m;
}
-->
public static Message obtain() {
    synchronized (sPoolSync) {
    	//sPool 不为null 时候,可以复用,不必新创建Message实例
        if (sPool != null) {
            Message m = sPool;
            sPool = m.next;
            m.next = null;
            m.flags = 0; // clear in-use flag
            sPoolSize--;
            return m;
        }
    }
    return new Message();
}
```
2. Handler.sendMessage(@NonNull Message msg) , Handler.post(@NonNull Runnable r) , Message.sendToTarget() ,最终都会调用 **Handler.sendMessageAtTime**
	- **设置Message实例的target是当前Handler实例**
    - 执行MessageQueue实例的enqueueMessage方法
```java
public boolean sendMessageAtTime(@NonNull Message msg, long uptimeMillis) {
    MessageQueue queue = mQueue;
    if (queue == null) {
        RuntimeException e = new RuntimeException(
                this + " sendMessageAtTime() called with no mQueue");
        Log.w("Looper", e.getMessage(), e);
        return false;
    }
    return enqueueMessage(queue, msg, uptimeMillis);
}
-->enqueueMessage
private boolean enqueueMessage(@NonNull MessageQueue queue, @NonNull Message msg,
        long uptimeMillis) {
	//Message实例的target属性为当前Handler
    msg.target = this;
    msg.workSourceUid = ThreadLocalWorkSource.getUid();
    if (mAsynchronous) {
        msg.setAsynchronous(true);
    }
    return queue.enqueueMessage(msg, uptimeMillis);
}
-->return queue.enqueueMessage(msg, uptimeMillis);
```
3. MessageQueue
```java
/**
 * Low-level class holding the list of messages to be dispatched by a
 * {@link Looper}.  Messages are not added directly to a MessageQueue,
 * but rather through {@link Handler} objects associated with the Looper.
 *
 * <p>You can retrieve the MessageQueue for the current thread with
 * {@link Looper#myQueue() Looper.myQueue()}.
 */
public final class MessageQueue {
```
- MessageQueue存储了1个Message单链表.
- Message实例是通过Looper对象传来的.
- Message实例不是直接通过MessageQeueu加入,而是通过关联了指定Looper的Handler对象传入.
- 通过Looper.myQueue获得当前线程的Looper实例
- enqueueMessage方法
	- 就是插入单链表的逻辑,根据Message的'执行时间'决定其插入位置
	- 当单链表的头部不存在,或者头部的'执行时间'晚于当前加入的Message,则将当前加入的Message作为头部
	- 当单链表的头部存在,且其'执行时间'早于当前加入的Message,则遍历该单链表,和其中每个Message的'执行时间'进行比较,找到合适的位置插入.若找不到,则插到单链表的尾部
  ```java
  boolean enqueueMessage(Message msg, long when) {
      ***
      synchronized (this) {
          ***
          msg.markInUse();
          msg.when = when;
          //mMessages:单链表的头部
          Message p = mMessages;
          boolean needWake;
          if (p == null || when == 0 || when < p.when) {
              // New head, wake up the event queue if blocked.
              //当单链表的头部不存在,或者头部的'执行时间'晚于当前加入的Message,则将当前加入的Message作为头部
              msg.next = p;
              mMessages = msg;
              needWake = mBlocked;
          } else {
              //当单链表的头部存在,且其'执行时间'早于当前加入的Message,则遍历该单链表,和其中每个
              //Message的'执行时间'进行比较,找到合适的位置插入.
              //若找不到,则插到单链表的尾部
              needWake = mBlocked && p.target == null && msg.isAsynchronous();
              Message prev;
              for (;;) {
                  prev = p;
                  p = p.next;
                  if (p == null || when < p.when) {
                      break;
                  }
                  if (needWake && p.isAsynchronous()) {
                      needWake = false;
                  }
              }
              msg.next = p; // invariant: p == prev.next
              prev.next = msg;
          }

          // We can assume mPtr != 0 because mQuitting is false.
          if (needWake) {
              nativeWake(mPtr);
          }
      }
      return true;
  }
  ```
- next() 获取下一个待执行的Message实例
	- 获取到Message单链表头结点,然后死等,直到时间到了将其返回,并在返回前更新 单链表头结点的指针.
  ```java
  Message next() {
      ***
      int nextPollTimeoutMillis = 0;
      //死循环
      for (;;) {
          if (nextPollTimeoutMillis != 0) {
              Binder.flushPendingCommands();
          }
          //阻塞当前调用线程 nextPollTimeoutMillis 毫秒值
          //每次循环都阻塞计算得到的毫秒值
          nativePollOnce(ptr, nextPollTimeoutMillis);
          synchronized (this) {
              final long now = SystemClock.uptimeMillis();
              Message prevMsg = null;
              //当下待执行Message
              Message msg = mMessages;
              ***
              if (msg != null) {
                  if (now < msg.when) {
                      //时间不到,得到需要等待多少毫秒
                      nextPollTimeoutMillis = (int) Math.min(msg.when - now, Integer.MAX_VALUE);
                  } else {
                      //时间到了,则将'当下待执行Message'返回
                      mBlocked = false;
                      if (prevMsg != null) {
                          prevMsg.next = msg.next;
                      } else {
                          //并将 mMessages 指针移动到 '当下待执行Message' 的下一个
                          mMessages = msg.next;
                      }
                      //情况next属性后返回
                      msg.next = null;
                      msg.markInUse();
                      return msg;
                  }
              } else {
                  // No more messages.
                  nextPollTimeoutMillis = -1;
              }
              ***
          }
          ***
      }
  }
  ```
4. Looper
- Looper用于未1个线程执行消息循环.
- Thread默认并不具备关联的Looper实例.
- Thread通过调用Looper.prepare()来创建其关联的Looper实例,并调用Looper.loop()来开启消息循环.
  ```
  Class used to run a message loop for a thread.  Threads by default do 
  not have a message loop associated with them; to create one, call 
  {@link #prepare} in the thread that is to run the loop, and then
  {@link #loop} to have it process messages until the loop is stopped.

  Most interaction with a message loop is through the {@link Handler} class.
  *  class LooperThread extends Thread {
  *      public Handler mHandler;
  *      public void run() {
  *          Looper.prepare();
  *          mHandler = new Handler() {
  *              public void handleMessage(Message msg) {
  *                  // process incoming messages here
  *              }
  *          };
  *          Looper.loop();
  *      }
  *  }
  ```
- prepare
	- prepare为指定线程创建了1个Looper实例,并将其存储到ThreadLocal中.
	- 创建Looper实例,会创建对应的MessageQueue实例,并获取调用方法的线程
  ```java
  public static void prepare() {
      prepare(true);
  }
  private static void prepare(boolean quitAllowed) {
      //prepare不能重复执行,否则触发异常
      if (sThreadLocal.get() != null) {
          throw new RuntimeException("Only one Looper may be created per thread");
      }
      //将创建的Looper实例保存到ThreadLocal中
      sThreadLocal.set(new Looper(quitAllowed));
  }
  private Looper(boolean quitAllowed) {
      //构造方法就是创建了MessageQueue实例,并获取调用方法的线程
      mQueue = new MessageQueue(quitAllowed);
      mThread = Thread.currentThread();
  }
  ```
- prepareMainLooper
	- 创建主线程关联的Lopper实例,内部实际调用 prepare.
	- 主线程的Looper实例是由android系统创建,不应该有开发者调用prepareMainLooper.
  ```
  /**
   * Initialize the current thread as a looper, marking it as an
   * application's main looper. The main looper for your application
   * is created by the Android environment, so you should never need
   * to call this function yourself.  See also: {@link #prepare()}
   */
  public static void prepareMainLooper() {
      prepare(false);
      synchronized (Looper.class) {
          if (sMainLooper != null) {
              throw new IllegalStateException("The main Looper has already been prepared.");
          }
          sMainLooper = myLooper();
      }
  }
  ```
- myLooper
	- 返回ThreadLocal中保存的Looper实例.
	- 在调用 prepare 前,sThreadLocal是空的.
  ```java
  static final ThreadLocal<Looper> sThreadLocal = new ThreadLocal<Looper>();
  public static @Nullable Looper myLooper() {
      return sThreadLocal.get();
  }
  ```
	- 为什么在子线程创建Handler实例,需要先执行Looper.prepare()? --> 会触发异常
  ```java
  public Handler() {
      this(null, false);
  }
  public Handler(@Nullable Callback callback, boolean async) {
      ***
      //这里会获取创建Handler的线程关联的Looper实例
      //如果该线程之前未执行prepare(),则其关联的Looper实例是null
      //会触发RuntimeException
      mLooper = Looper.myLooper();
      if (mLooper == null) {
          throw new RuntimeException(
              "Can't create handler inside thread " + Thread.currentThread()
                      + " that has not called Looper.prepare()");
      }
      mQueue = mLooper.mQueue;
      mCallback = callback;
      mAsynchronous = async;
  } 
  ```
	- 为什么在主线程,创建Handler不需要执行Looper.prepare: 因为主线程创建之初系统自动为其创建了Looper并开启消息循环.
		- 主线程是谁创建的? ActivityThread
		- ActivityThread的main方法
		- 由源码可见,在主线程创建之初,系统就为其创建了关联的Looper实例,并开启了消息循环.
      ```java
      public static void main(String[] args) {
          ***
          //这里执行Looper.prepareMainLooper
          //上面可知这里为主线程创建了对应的Looper实例
          Looper.prepareMainLooper();
          ***
          ActivityThread thread = new ActivityThread();
          thread.attach(false, startSeq);
          ***
          //开启主线程关联的Looper的消息循环
          Looper.loop();
      }
      ```
- loop
	- loop就是不断调用Looper实例内部的MessageQueue的next对象,获取当下可以执行的Message实例
	- 获取到Message实例,执行其 msg.target.dispatchMessage(msg); 即 Handler的dispatchMessage.
	- **msg.target就是之前执行sendMessage时调用的Handler实例**
```java
public static void loop() {
	//校验当前线程是否已创建过Looper实例
    final Looper me = myLooper();
    if (me == null) {
        throw new RuntimeException("No Looper; Looper.prepare() wasn't called on this thread.");
    }
    //获取Looper实例关联的消息队列
    final MessageQueue queue = me.mQueue;
	***
    //死循环
    for (;;) {
    	//上面介绍过,这里会死等,直到拿到可以执行的 Message 实例
        Message msg = queue.next(); // might block
        if (msg == null) {
            // No message indicates that the message queue is quitting.
            return;
        }
		***
        try {
        	//获取到 Message,执行msg.target.dispatchMessage(msg);
            //继续看Handler.dispatchMessage
            msg.target.dispatchMessage(msg);
            ***
        }***
        msg.recycleUnchecked();
    }
}
```
5. Handler
- dispatchMessage
  ```java
  public void dispatchMessage(@NonNull Message msg) {
      if (msg.callback != null) {
          //1:Message实例包含Runnable实例,执行handleCallback
          handleCallback(msg);
      } else {
          if (mCallback != null) {
              //2:mCallback不为空,则先由mCallback处理,处理未完成,继续执行handleMessage
              if (mCallback.handleMessage(msg)) {
                  return;
              }
          }
          //3:执行 handleMessage .
          handleMessage(msg);
      }
  }
  //1:实际就是执行Message实例中Runnable的run方法.即实际执行的是Handler.post(Runnable)中传入的Runnable实例
  private static void handleCallback(Message message) {
      message.callback.run();
  }
  //1.1:message.callback赋值位置:
  Handler:
  public final boolean post(@NonNull Runnable r) {
  	return  sendMessageDelayed(getPostMessage(r), 0);
  }
  private static Message getPostMessage(Runnable r) {
      Message m = Message.obtain();
      m.callback = r;
      return m;
  }
  Message:
  public Message setCallback(Runnable r) {
    callback = r;
    return this;
  }
  //2:mCallback可以由创建Handler实例时候传入
  public interface Callback {
      boolean handleMessage(@NonNull Message msg);
  }
  ```
6. 主线程Looper一直执行死循环,为什么不会触发ANR
- [**看完这篇还不明白Handler你砍我 写的非常好**](https://juejin.im/post/6866015512192876557)
<br>
截取文章中部分内容:
![](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/b3bbe9c1ec8248d1996803878fb6e4c7~tplv-k3u1fbpfcp-zoom-1.image)

7. Handler要及时释放
- 使用Handler延时执行Activity的finish.此时如果有其他Handler已经有延时消息在等待执行,可能导致当下Handler中的finish不能被执行.具体原因不清楚.
  ```java
  //此时,若有其他Handler正在处理Message,或之前其他Handler实例执行了mHandler.sendEmptyMessageDelayed
  //则finish可能不会执行
  new Handler().postDelayed(new Runnable() {
      @Override
      public void run() {
          //finish可能不会执行
          finish();
      }
  }, 200);
  ```
- Activity执行finish后,若有Handler在等待延时消息,或Handler正在处理消息,则必须等延时消息到来,或当前消息处理结束,onDestroy才会执行.
- 及时释放Handler
  ```java
  private void releaseHandler() {
      if (mHandler != null) {
          try {
              mHandler.removeCallbacksAndMessages(null);
          } catch (Exception e) {
              e.printStackTrace();
          }
          mHandler = null;
      }
  }
  ```


### 待看
- [**"一篇就够"系列: Handler消息机制完全解析**](https://juejin.cn/post/6924084444609544199)
- [**"一篇就够"系列: Handler扩展篇**](https://juejin.cn/post/6932608660354891790)
- [Handler之同步屏障机制(sync barrier)](https://blog.csdn.net/asdgbc/article/details/79148180)
- [每日问答 Handler应该是大家再熟悉不过的类了，那么其中有个同步屏障机制，你了解多少呢？ 2020](https://www.wanandroid.com/wenda/show/8710)
- [Android 源码分析 - Handler的同步屏障机制
](https://www.jianshu.com/p/2535f24d291c)
- [Android筑基——可视化方式理解 Handler 的同步屏障机制
](https://blog.csdn.net/willway_wang/article/details/108328820)
- [面试官：“看你简历上写熟悉 Handler 机制，那聊聊 IdleHandler 吧？”](https://www.jianshu.com/p/afa97cb01f29)
- [关于Handler 的这 15 个问题，你都清楚吗？](https://mp.weixin.qq.com/s/vCnftbD3z07X79gHj30Kiw)
- [面试官："Handler的runWithScissors()了解吗？为什么Google不让开发者用？"](https://mp.weixin.qq.com/s/ZhB_DUA3juybE7eeFd6Lkw)
- [Android | 面试必问的 Handler，你确定不看看？](https://www.jianshu.com/p/70d5785ee4c3)
- [Handler全家桶之 —— Handler 源码解析](https://mp.weixin.qq.com/s/xb6eQx9iaUAWur6K9Xv9yg)
- [换个姿势，带着问题看Handler](https://juejin.im/post/6844904150140977165)
- [Android源码剖析：基于 Handler、Looper 实现拦截全局崩溃、监控ANR等](https://www.jianshu.com/p/312d9f9cab67)
- **HandlerThread**
	- [Android HandlerThread 完全解析](https://blog.csdn.net/lmj623565791/article/details/47079737)
	- [Android多线程开发之HandlerThread的使用](https://blog.csdn.net/isee361820238/article/details/52589731)
	- [从源码角度理解HandlerThread和IntentService](https://blog.csdn.net/xlh1191860939/article/details/107225817)
	- [Android多线程：这是一份详细的HandlerThread源码分析攻略](https://www.jianshu.com/p/4a8dc2f50ae6)
	- [Android多线程：手把手教你使用HandlerThread](https://www.jianshu.com/p/9c10beaa1c95)
