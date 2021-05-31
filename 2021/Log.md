#### 1. 使用Log打印函数调用栈
```java
Log.d(Tag.CATCH_EXCEPTION, "func. detail:", new Throwable());
Log.d(Tag.CATCH_EXCEPTION, Log.getStackTraceString(new Throwable()));
```
#### 2. 使用Log打印异常堆栈
```java
private void func() {
    try {
        ***
    } catch (Exception e) {
        Log.e(Tag.CATCH_EXCEPTION, "func. err:", e);
        Log.e(Tag.CATCH_EXCEPTION, Log.getStackTraceString(e));
        e.printStackTrace();
    }
}
```
