# ANR - Application Not Responding

## ANR 日志文件 /data/anr/traces.txt

#### 1. 什么是 ANR ？
    Application Not Responding
    主线程中做了耗时操作，会弹出对话框
    默认情况下是等待 5秒，在 BroadcastReceiver 的 onReceive() 方法中等待时间是 10秒

#### 2. 造成 ANR 的主要原因 - 主线程存在耗时操作
    1> Android 中哪些操作是在主线程呢？
      1) Activity 的所有生命周期回调都是执行在主线程的
      2）Service 默认是执行在主线程的
      3）BroadcastReceiver 的 onReceive 回调是执行在主线程的
      4）没有使用子线程的 Looper 的 Handler 的 handleMessage，post(Runnable) 是执行在主线程的
      5) AsyncTask 的回调中除了 doInBackground，其他都是执行在主线程的

#### 3. 如何解决 ANR
    1. 异步处理耗时操作 - AsyncTask，Handler
    2. 使用 Thread 或者 HandlerThread 提高优先级
    3. Activity 的 onCreate 和 onResume 回调中尽量避免耗时的代码
