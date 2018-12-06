## Handler
Handler 机制

## 1. handler 四大组成部分：Handler，Message, MessageQueue, Looper

#### 1.1 Handler: handler.sendMessage(Message msg) / handler.handleMessage(Message msg);
#### 1.2 Message: 数据载体，还持有一个 handler 的引用 target，即是谁发送的该 message;
#### 1.3 MessageQueue: message 队列，Looper.prepare() / Looper.prepareMainLooper() 时会创建 messagequeue;
#### 1.4 Looper: Looper.loop() 调用该方法启动死循环，通过 MessageQueue.next() 不断从 messagequeue 中取 message;

###### 1.4.1 messagequeue 中没有 message 时: 阻塞状态;
###### 1.4.2 messagequeue 中有 message 时: 取出 message 后，调用 message.target（即 handler）的 dispatchMessage() 方法，在该方法中会调用 handler 的 handleMessage() 方法，来根据 message 中的数据，进行 UI 操作;

## 2. Handler 内存泄漏问题: 在Java中，非静态的内部类和匿名内部类都会隐式地持有其外部类的引用，而静态的内部类不会持有外部类的引用。

#### 2.1 错误写法：

    private MyHandler handler = new MyHandler();

    class MyHandler extends Handler {
      @Override
      public void handleMessage(Message message) {
        super.handleMessage(message);
        // TODO
      }
    }

#### 由于内部类 MyHandler 持有对外部类 (MainActivity) 的引用，下面情况会发生内存泄漏：当 messagequeue 中还有消息没有执行完，handler 会按顺序处理这些 message，但是如果此时 MainActivity 已经被销毁，当 handler 处理 message 时用到了 MainActivity，此时会发生内存泄漏;

#### 2.2 正确做法：

###### 2.2.1 MyHandler 放到单独的类文件中;
###### 2.2.1 在 Activity 的 onDestroy() 方法中添加如下代码:
        handler.removeCallbacksAndMessages(null);
###### 2.2.2 静态内部类：

    private MyHandler handler = new MyHandler(this);

    static class MyHandler extends Handler {
      private WeakReference<MainActivity> mOuter;

      public MyHandler(MainActivity activity) {
        mOuter = new WeakReference<>(activity);
      }

      @Override
      public void handleMessage(Message message) {
        super.handleMessage(message);

        MainActivity activity = mOuter.get();
        if (null != activity) {
          // TODO
        }
      }
    }
