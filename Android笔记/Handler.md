### Handler

![handler_总览](Image.assets\handler_总览.png)

#### 1、Handler 使用

- 步骤

  - 调用 `Looper.prepare()`
  - 创建 `Handler` 对象
  - 调用 `Looper.loop()`

- 主线程不需要手动调用 `Looper.prepare()` ，启动 App 时系统创建，子线程需要手动调用

  ```java
  //ActivityThread.main 方法
  public static void main(String[] args) {
      ......
      //调用Looper.prepareMainLooper，调用Looper.loop，初始化Looper、MessageQueue等（当前线程）
      Looper.prepareMainLooper();
      //创建 ActivityThread 同时，初始化成员变量 Handler mH
      ActivityThread thread = new ActivityThread();
      thread.attach(false);
      if (sMainThreadHandler == null) {
          // 把创建的 Handler mH 赋值给 sMainThreadHandler
          sMainThreadHandler = thread.getHandler();
      }
      if (false) {
          Looper.myLooper().setMessageLogging(new LogPrinter(Log.DEBUG, "ActivityThread"));
      }
      //调用 Looper.loop() 方法开启死循环，从 MessageQueue 中不断取出 Message 处理
      Looper.loop();
      throw new RuntimeException("Main thread loop unexpectedly exited");
  }
  ```

#### 2、Handler 机制

- 涉及类

  - Message：消息
  - Hanlder：消息发起者
  - Looper：消息遍历者
  - MessageQueue：消息队列

- Looper.prepare()

  - 创建 Looper 对象
  - 创建 MessageQueue 对象，让 Looper 对象持有
  - 让 Looper 对象持有当前线程

  ```java
  public static void prepare() {
      prepare(true);
  }
  
  private static void prepare(boolean quitAllowed) {
      // 一个线程只有一个 Looper，一个线程只能调用一次 Looper.prepare()
      if (sThreadLocal.get() != null) {
          throw new RuntimeException("Only one Looper may be created per thread");
      }
      // 如果当前线程没有Looper，就创建一个存到 sThreadLocal 中
      sThreadLocal.set(new Looper(quitAllowed));
  }
  
  private Looper(boolean quitAllowed) {
      // 创建MessageQueue，并供Looper持有，一个线程只有一个消息队列
      mQueue = new MessageQueue(quitAllowed);
      // 让Looper持有当前线程对象
      mThread = Thread.currentThread();
  }
  ```

- new Handler()

  - 创建 Handler 对象
  - 得到当前线程的 Looper 对象，判断是否为空
  - 创建的 Handler 对象持有 Looper、MessageQueu、Callback 的引用

  ```java
  public Handler() {
      this(null, false);
  }
  
  public Handler(Callback callback, boolean async) {
      ......
      //得到当前线程的Looper，即调用sThreadLocal.get
      mLooper = Looper.myLooper();
      //当前线程没有Looper报运行时异常
      if (mLooper == null) {
          throw new RuntimeException("Can't create handler inside thread that has not called Looper.prepare()");
      }
      // 把得到的Looper的MessagQueue让Handler持有
      mQueue = mLooper.mQueue;
      // 初始化Handler的Callback
      mCallback = callback;
      mAsynchronous = async;
  }
  ```

- Looper.loop()

  - 从当前线程的 MessageQueue 不断取出 Message，调用其相关回调方法

  ```java
  public static void loop() {
      // 得到当前线程的Looper对象
      final Looper me = myLooper();
      if (me == null) {
          throw new RuntimeException("No Looper; Looper.prepare() wasn't called on this thread.");
      }
      // 得到当前线程的MessageQueue对象
      final MessageQueue queue = me.mQueue;
  
      ......
  
      // 死循环
      for (;;) {
          // 不断从当前线程MessageQueue取出Message，没有元素时阻塞
          Message msg = queue.next();
          if (msg == null) {
              // 没有消息表示消息队列正在退出
              return;
          }
          // Message.target是发送消息的Handler，这里调用它的dispatchMessage方法
          msg.target.dispatchMessage(msg);
          // 回收Message
          msg.recycleUnchecked();
      }
  }
  
  public void dispatchMessage(Message msg) {
      // 如果msg.callback不是null，调用handleCallback
      if (msg.callback != null) {
          handleCallback(msg);
      } else {
          // 如果 mCallback不为空，调用mCallback.handleMessage方法
          if (mCallback != null) {
              if (mCallback.handleMessage(msg)) {
                  return;
              }
          }
          // 调用Handler自身的handleMessage，这就是我们常常重写的那个方法
          handleMessage(msg);
      }
  }
  ```

  ![handler_Message分发](Image.assets\handler_Message分发.png)

- 发送消息：本质把 Message 加入 Handler 的 MessageQueue 队列中

  - sendMessage(Message msg)

  ```java
  public final boolean sendMessage(Message msg){
      return sendMessageDelayed(msg, 0);
  }
  
  public final boolean sendMessageDelayed(Message msg, long delayMillis){
      if (delayMillis < 0) {
          delayMillis = 0;
      }
      return sendMessageAtTime(msg, SystemClock.uptimeMillis() + delayMillis);
  }
  
  public boolean sendMessageAtTime(Message msg, long uptimeMillis) {
      // 这里的MessageQueue是创建时的MessageQueue，默认情况是当前线程的Looper对象的MessageQueue
      MessageQueue queue = mQueue;
      if (queue == null) {
          RuntimeException e = new RuntimeException(this + " sendMessageAtTime() called with no mQueue");
          Log.w("Looper", e.getMessage(), e);
          return false;
      }
      // 调用enqueueMessage，把消息加入到MessageQueue中
      return enqueueMessage(queue, msg, uptimeMillis);
  }
  
  private boolean enqueueMessage(MessageQueue queue, Message msg, long uptimeMillis){
      // 把当前Handler对象，也就是发起消息的handler作为Message的target属性
      msg.target = this;
      if (mAsynchronous) {
          msg.setAsynchronous(true);
      }
      // 调用MessageQueue中的enqueueMessage方法
      return queue.enqueueMessage(msg, uptimeMillis);
  }
  
  boolean enqueueMessage(Message msg, long when) {
      if (msg.target == null) {
          throw new IllegalArgumentException("Message must have a target.");
      }
      // 一个Message，只能发送一次
      if (msg.isInUse()) {
          throw new IllegalStateException(msg + " This message is already in use.");
      }
      synchronized (this) {
          if (mQuitting) {
              IllegalStateException e = new IllegalStateException(msg.target + " sending message to a Handler on a dead thread");
              Log.w("MessageQueue", e.getMessage(), e);
              msg.recycle();
              return false;
          }
          // 标记Message已经使用了
          msg.markInUse();
          msg.when = when;
          // 得到当前消息队列的头部
          Message p = mMessages;
          boolean needWake;
          // when（延迟时间）为0表示立即处理的消息
          if (p == null || when == 0 || when < p.when) {
              // 把消息插入到消息队列的头部
              msg.next = p;
              mMessages = msg;
              needWake = mBlocked;
          } else {
              // 根据需要把消息插入到消息队列的合适位置，通常调用xxxDelay方法，延时发送消息
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
              // 把消息插入到合适位置
              msg.next = p;
              prev.next = msg;
          }
          // 如果队列阻塞则唤醒
          if (needWake) {
              nativeWake(mPtr);
          }
      }
      return true;
  }
  ```

  - post(Runnable r)

  ```java
  public final boolean post(Runnable r){
      return sendMessageDelayed(getPostMessage(r), 0);
  }
  
  private static Message getPostMessage(Runnable r) {
      // 构造一个Message，让其callback执行传来的Runnable
      Message m = Message.obtain();
      m.callback = r;
      return m;
  }
  ```

#### 3、IdleHandler

- 一个接口

  ```java
  //回调接口，用于发现线程何时阻塞去等待消息
  public static interface IdleHandler {
      //当消息队列已用完消息且现在在等待更多消息时调用
      //返回 true 保留消息，等到下次空闲时再次执行
      //返回 false 删除
      //如果队列仍有挂起的消息，但这些消息都计划在当前时间后调度，则可调用此函数
      boolean queueIdle();
  }
  ```

> 消息队列空闲时执行其 `queueIdle()`，返回一个 Boolean 值，false 表示执行完后移除这条消息，true 则

- MessageQueue.next

  - 处理 IdleHandler 后会将 nextPollTimeoutMillis 设为 0（不阻塞消息队列）， 注意这里执行的代码不能太耗时（同步执行，太耗会影响后面的 message 执行）
  - 本质是趁消息队列空闲时干点事情提高处理性能，

  - 使用 IdleHandler 只需调用 `MessageQueue#addIdleHandler(IdleHandler handler)` 方法
  - 流程
    - 如果本次循环拿到的 Message 为空或者 Message 是个延时消息且还没到指定触发时间，认定当前队列为空闲时间
    - 接着遍历 mPendingIdleHandlers 数组（数组里的元素每次都到 mIdleHandlers 中拿），调用每个 IdleHandler 实例的 queueIdle 方法
    - 如果返回 false，实例从 mIdleHandlers 中移除

  ```java
  Message next() {
      //......
      for (;;) {
          //......
          synchronized (this) {
              // 此处为正常消息队列的处理
              //......
              if (mQuitting) {
                  dispose();
                  return null;
              }
              //IdleHandler数组mPendingIdleHandlers，放的IdleHandler实例都是临时的，即每次使用完(调用queueIdle方法)后都会置空(mPendingIdleHandlers[i]=null)
              if (pendingIdleHandlerCount < 0 && (mMessages == null || now < mMessages.when)) {
                  //mIdleHandlers，存放IdleHandler的ArrayList
                  pendingIdleHandlerCount = mIdleHandlers.size();
              }
              if (pendingIdleHandlerCount <= 0) {
                  //没有空闲的处理程序可运行,循环再等等
                  mBlocked = true;
                  continue;
              }
              if (mPendingIdleHandlers == null) {
                  mPendingIdleHandlers = new IdleHandler[Math.max(pendingIdleHandlerCount, 4)];
              }
              mPendingIdleHandlers = mIdleHandlers.toArray(mPendingIdleHandlers);
          }
          for (int i = 0; i < pendingIdleHandlerCount; i++) {
              final IdleHandler idler = mPendingIdleHandlers[i];
              mPendingIdleHandlers[i] = null; //释放对处理程序的引用
              boolean keep = false;
              try {
                  keep = idler.queueIdle();
              } catch (Throwable t) {
                  Log.wtf(TAG, "IdleHandler threw exception", t);
              }
              if (!keep) {
                  synchronized (this) {
                      mIdleHandlers.remove(idler);
                  }
              }
          }
          pendingIdleHandlerCount = 0;
          nextPollTimeoutMillis = 0;
      }
  }
  ```

- 应用
  - Activity 启动优化：onCreate，onStart，onResume 耗时较短但非必要的代码可放到 IdleHandler 中执行，减少启动时间
  - ActivityThread 中有一个 GcIdler 内部类，实现 IdleHandler 接口，在 queueIdle 方法被回调时会强行 GC（调用 BinderInternal 的 faceGc 方法），前提是与上一次强行 GC 至少相隔 5 秒以上，当 ActivityThread 的 mH（Handler）收到 GC_WHEN_IDLE 消息后调用，收到 GC_WHEN_IDLE 消息：
    - doLowMemReportIfNeededLocked：内存不够时调用
    - activityIdle：当 ActivityThread 的 handleResumeActivity 方法被调用时（Activity 的 onResume 方法在这里回调）调用
  - 想要一个 View 绘制完成后添加其他依赖于这个 View 的 View，可以 `View#post()` 实现，但用IdleHandler 会在消息队列空闲时执行
  - 一些第三方库中使用，如 LeakCanary，Glide