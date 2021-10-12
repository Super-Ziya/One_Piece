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
  - MessageQueue：消息队列，底层实现采用的是单链表，这是因为链表在插入和删除方面的性能好，mMessages保存链表的第一个元素，next保存下一个
    - Java层MessageQueue的创建：创建一个native层的MessageQueue，将引用地址返回给Java层保存在 mPtr 变量，将Java层对象与Native层对象关联在一起
    - Native层MessageQueue的创建：创建一个Looper，与线程绑定，与 Java 层的 Looper 没有多大关系，用来处理 Native 层事件，Java层的 Looper 调用 MessageQueue 的 next 方法获取下一个消息
    - Java 层 `nativePollOnce(ptr, nextPollTimeoutMillis)` 是 native 方法，通过 Native 层 MessageQueue 阻塞 nextPollTimeoutMillis 毫秒的时间
    - 插入异步消息（msg.target 为 null）：`postSyncBarrier(long when)`，返回一个 int 类型的 token，移除屏障用 `removeSyncBarrier(int token)`，设置异步消息 `setAsynchronous(boolean async)`，ViewRootImpl 的 `void scheduleTraversals()` 在绘图前会插入一个消息屏障

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
  - 创建的 Handler 对象持有 Looper、MessageQueue、Callback 的引用

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
      //msg.target就是发送此消息的Handler
      if (msg.target == null) {
          throw new IllegalArgumentException("Message must have a target.");
      }
      // 一个Message，只能发送一次
      if (msg.isInUse()) {
          throw new IllegalStateException(msg + " This message is already in use.");
      }
      synchronized (this) {
          // 此消息队列已经被放弃了
          if (mQuitting) {
              IllegalStateException e = new IllegalStateException(msg.target + " sending message to a Handler on a dead thread");
              Log.w("MessageQueue", e.getMessage(), e);
              msg.recycle();
              return false;
          }
          // 标记Message已经使用了
          msg.markInUse();
          // 将延迟时间封装到msg内部
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
              // 延时消息，将其添加到队列中，原理是链表添加新元素，按照when 延迟时间来插入的，延迟的时间越长越靠后，插入延时消息不需要唤醒Looper线程
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
              //通过ptr获取NativeMessageQueue对象的指针，再调用NativeMessageQueue::wake -- Looper::wake，write方法写入一个1，epoll就能监听到，也被唤醒
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

  - 发送延时消息 sendMessageDelayed

    - 获取当前时间用 `SystemClock.uptimeMillis()` 而不用 `SystemClock.currentTimeMillis()`：

      > `System.currentTimeMillis()`：产生一个标准的自1970年1月1号0时0分0秒所差的毫秒数。可通过调用`setCurrentTimeMillis(long)` 手动设置，也可通过网络自动获取。如果在执行时间间隔的值期间用户更改了手机系统的时间，得到的结果是不可预料的。不适合用在需要时间间隔的地方，如 `Thread.sleep()`、`Object.wait()` 等，因为其值可能被改变
      >
      > `SystemClock.uptimeMillis()`：用来计算自开机启动到目前的毫秒数。如果系统进入了深度睡眠状态（CPU停止运行、显示器息屏、等待外部输入设备）该时钟会停止计时，但是该方法不会受时钟刻度、时钟闲置时间或其它节能机制的影响，是计算间隔的基本依据，如 `Thread.sleep()`、`Object.wait()`、`System.nanoTime()`、Handler 等。保证单调性，适用于计算不跨越设备的时间间隔

  ```java
  public final boolean sendMessageDelayed(Message msg, long delayMillis){
      if (delayMillis < 0) {
          delayMillis = 0;
      }
      //消息被处理的时间=当前时间+延迟的时间
      return sendMessageAtTime(msg, SystemClock.uptimeMillis() + delayMillis);
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
  - 本质是趁消息队列空闲时干点事情提高处理性能

  - 使用 IdleHandler 只需调用 `MessageQueue#addIdleHandler(IdleHandler handler)` 方法
  - 流程
    - 如果本次循环拿到的 Message 为空或者 Message 是个延时消息且还没到指定触发时间，认定当前队列为空闲时间
    - 接着遍历 mPendingIdleHandlers 数组（数组里的元素每次都到 mIdleHandlers 中拿），调用每个 IdleHandler 实例的 queueIdle 方法
    - 如果返回 false，实例从 mIdleHandlers 中移除

  ```java
  Message next() {
      final long ptr = mPtr;
      if (ptr == 0) {
          //只有looper被放弃时（调用quit）才返回null，mPtr是MessageQueue的一个long型成员变量，关联一个在C++层的MessageQueue，阻塞操作就是通过底层的这个MessageQueue来操作的；当队列被放弃的时候其变为0
          return null;
      }
      for (;;) {
          if (nextPollTimeoutMillis != 0) {
              Binder.flushPendingCommands();
          }
          //阻塞方法，主要是通过native层的epoll监听文件描述符的写入事件实现
          //如果nextPollTimeoutMillis=-1，一直阻塞不会超时
          //如果nextPollTimeoutMillis=0，不会阻塞，立即返回
          //如果nextPollTimeoutMillis>0，最长阻塞nextPollTimeoutMillis毫秒(超时)，如果期间有程序唤醒会立即返回
          //通过ptr获取NativeMessageQueue对象的指针，调用NativeMessageQueue::pollOnce() -- Looper::pollOnce() -- Looper::pollInner，根据Native Message信息计算此次需要等待的时间，再调用epoll_wait等待事件发生
          nativePollOnce(ptr, nextPollTimeoutMillis);
          synchronized (this) {
              final long now = SystemClock.uptimeMillis();
              Message prevMsg = null;
              Message msg = mMessages;
              if (msg != null && msg.target == null) {
                  //msg.target == null表示此消息为消息屏障（通过postSyncBarrier发送）
                  //如果发现一个消息屏障，会循环找出第一个异步消息（如果有的话），所有同步消息都将忽略（平常发送的一般都是同步消息）
                  do {
                      prevMsg = msg;
                      msg = msg.next;
                  } while (msg != null && !msg.isAsynchronous());
              }
              if (msg != null) {
                  if (now < msg.when) {
                      //如果消息此刻还没有到时间，设置一下阻塞时间nextPollTimeoutMillis，进入下次循环会调用nativePollOnce(ptr, nextPollTimeoutMillis)阻塞；
                      nextPollTimeoutMillis = (int) Math.min(msg.when - now, Integer.MAX_VALUE);
                  } else {
                      //正常取出消息
                      //设置mBlocked = false代表目前没有阻塞
                      mBlocked = false;
                      if (prevMsg != null) {
                          prevMsg.next = msg.next;
                      } else {
                          mMessages = msg.next;
                      }
                      msg.next = null;
                      msg.markInUse();
                      return msg;
                  }
              } else {
                  //没有消息，会一直阻塞，直到被唤醒
                  nextPollTimeoutMillis = -1;
              }
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

- 不会阻塞主线程
  - 管道epoll模型 ：当没有消息的时候会epoll.wait，等待句柄写的时候再唤醒，这个时候其实是阻塞的
    - 在主线程的MessageQueue没有消息时，便阻塞在 `loop的queue.next()` 中的 `nativePollOnce()` 方法里，此时主线程会释放CPU资源进入休眠状态，直到下个消息到达或者有事务发生，通过往pipe管道写端写入数据来唤醒主线程工作
    - epoll机制是一种IO多路复用机制，可以同时监控多个描述符，当某个描述符就绪(读或写就绪)，立刻通知相应程序进行读或写操作，本质是同步I/O，即读写阻塞
    - 所以，主线程大多数时候处于休眠状态，并不会消耗大量CPU资源
  - ActivityThread的内部类H继承于Handler，通过handler消息机制处理消息，所有的ui操作都通过handler来发消息操作：比如屏幕刷新16ms一个消息，你的各种点击事件，所以就会有句柄写操作，唤醒上文的wait操作，所以不会被卡死了
  - 主线程被唤醒只是为了读取消息，当消息读取完毕，再次睡眠。因此loop的循环并不会对CPU性能有过多的消耗

