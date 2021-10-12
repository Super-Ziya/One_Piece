### JUC

- while(true) 循环底层优化，很难从主存中读取数据，仅使用工作内存的，加 synchronize 后可使其被迫从主存中读取数据

#### 1、volatile

> 当多线程操作共享数据时，可保证内存中数据可见。相较于 synchronized 是一种轻量级同步策略

- 不具备“互斥性”
- 不能保证变量"原子性“

- i++

  ```java
  int temp = i;	//temp临时变量
  i = i + 1;
  i = temp;
  ```

#### 2、原子变量

> jdk1.5 后 java.util.concurrent.atomic 包下提供常用的原子变量

- volatile 保证内存可见性

- CAS（Compare-And-Swap）算法保证数据的原子性

  - 硬件对并发操作共享数据的支持

  - 包含三个操作数

    > 内存值 V
    >
    > 预估值 A
    >
    > 更新值 B

  - 当且仅当 v = A 时，V = B，否则不做任何操作，不会阻塞，可继续尝试

#### 3、ConcurrentHashMap

- 线程安全的哈希表，介于 HashMap 与 Hashtable 间。内部采用“锁分段”机制替代 Hashtable 的独占锁，提高性能
- java.util.concurrent.atomic 包提供设计用于多线程上下文中 collection 实现：ConcurrentHashMap、ConcurrentSkipListMap、ConcurrentSkipListSet、CopyOnWriteArrayList、CopyOnWriteArraySet
  - 期望许多线程访问一个给定 collection 时，ConcurrentHashMap 优于同步的 HashMap
  - ConcurrentSkipListMap 通常优于同步的 TreeMap
  - 期望读数和遍历远大于列表更新数时，CopyOnWriteArrayList 优于同步的 ArrayList
- 锁分段：Segment
  - 默认分 16 段，每段默认 16 个数组位置

#### 4、CopyOnWriteArrayList

> 写入复制，只在写操作才去复制一份新数据，否则都是访问同一资源

- 添加操作多时效率低，每次添加时都会复制，开销大
- 适合并发迭代操作多

#### 5、CountDownLatch

> 同步辅助类，完成一组正在其他线程中执行的操作前，允许一个或多个线程一直等待

- 可延迟线程进度直到终止状态，可用来确保某些活动直到其他活动都完成才继续执行
  - 确保某计算在其需要的所有资源都被初始化后才继续执行
  - 确保某服务在其依赖的所有其他服务都已启动后才启动
  - 等待直到某操作所有参与者都准备就绪再继续执行

```java
public class TestCountDownLatch{
    public static void main(String[] args){
        final countDownLatch latch = new countDownLatch(5);
        LatchDemo ld = new LatchDemo(latch);
        long start = System.currentTimeMillis();
        for (int i=0;i<5;i++){
            new Thread(ld).start();
        }
        try{
            latch.await();
        }catch (InterruptedException e){}
        long end = System.currentTimeMillis();
        system.out.print1n("耗费时间: " + (end - start));
    }
}

class LatchDemo implements Runnable{
    private countDownLatch latch;
    public LatchDemo(countDownLatch latch){
        this.latch = latch;
    }
    @override
    public void run(){
        synchronized(this){
            try{
                for(int i=0;i<50000;i++){
                    if(i % 2 == e){
                        system.out.print1n(i)；
                    }
                }
            }finally{
                latch.countDown();
            }
        }
    }
}
```

#### 6、创建线程

- 实现 Runnable 接口、继承 Thread 类、实现 Callable 接口
- Callable 接口：带泛型，泛型是返回值类型，可抛出异常
  - 需 FutureTask 实现类的支持，用于接收运算结果，FutureTask 是 Future 接口的实现类
  - 可用于闭锁

```java
public class Testcallable{
    public static void main(String[] args){
        ThreadDemo td = new ThreadDemo();
        //执行Callable方式，需FutureTask实现类支持，用于接收运算结果
        FutureTask<Integer> result = new FutureTask<>(td);
        new Thread(result).start();
        //接收线程运算后结果
        try{
            Integer sum = result.get();	//FutureTask可用于闭锁
            System.out.println(sum);
            System.out.print1n("------");
        }catch (InterruptedException | ExecutionException e){
            e.printStackTrace();
        }
    }
}

class ThreadDemo implements Callable<Integer>{
    @override
    public Integer call() throws Exception{
        int sum = 0;
        for (int i=0;i<=100;i++){
            sum +=i;
        }
        return sum;
    }
}
```

#### 7、解决多线程安全方式

- synchronized 隐式锁：同步代码块、同步方法
- jdk 1.5 后：同步锁 Lock，显示锁，需通过 `lock()` 上锁，`unlock()` 释放锁（finally 块中）
- Condition 接口：描述可能会与锁有关的条件变量。该变量在用法上与 Object.wait 的隐式监视器类似，但提供更强大功能
  - 单个 Lock 可与多个 Condition 对象关联，为避免兼容性问题，Condition 方法名与对应的 Object 版本不同
  - Condition 对象中，与 wait、notify、notifyAll 对应的是 await、signal、signalAll
  - Condition 实例实质上被绑定到一个锁上，为 Lock 实例获得 Condition 实例，使用其 `newCondition()` 

#### 8、生产者消费者案例

- 虚假唤醒：当一个条件满足时，很多线程都被唤醒，但只有其中部分是有用的，其它唤醒是无用功
  - 例：如果商品本没有货物，突然进了一件商品，所有线程都被唤醒来买，但只能一个线程买，其他线程是假唤醒，获取不到对象锁

```java
//店员，synchronized
class Cierk{
    private int product = 0;
    //进货
    public synchronized void get(){
        while(product >= 10){//为避免虚假唤醒问题，应在循环中使用 wait
            System.out.print1n("产品已满!");
            try {
                this.wait();
            }catch (InterruptedException e){}
        }
        system.out.println(Thread.currentThread().getName() + ":" + --product);
        this.notifyAll();
    }
    //卖货     
    public synchronized void sale(){
        while(product <= e){
            System.out.println("缺货!");
            try {
                this.wait();
            }catch (InterruptedException e){}
        }
        system.out.println(Thread.currentThread().getName() + ":" + --product);
        this.notifyAll();
    }
}

//店员，Lock
class Cierk{
    private int product = 0;
    private Lock lock = new ReentrantLock();
    private Condition condition = lock.newCondition();
    //进货
    public void get(){
        lock.lock();
        try{
            while(product >= 10){//为避免虚假唤醒问题，应在循环中使用 wait
                System.out.print1n("产品已满!");
                try {
                    condition.await();
                }catch (InterruptedException e){}
            }
            system.out.println(Thread.currentThread().getName() + ":" + --product);
            condition.signalAll();
        }finally{
            lock.unlock();
        }
    }
    //卖货     
    public void sale(){
        lock.lock();
        try{
            while(product <= e){
                System.out.println("缺货!");
                try {
                    condition.await();
                }catch (InterruptedException e){}
            }
            system.out.println(Thread.currentThread().getName() + ":" + --product);
            condition.signalAll();
        }finally{
            lock.unlock();
        }
    }
}

//生产者
class Productor implements Runnable{
    private Clerk clerk;
    public Productor(Clerk clerk) {
        this.clerk = clerk;
    }
    @override
    public void run(){
        for (int i=0;i<20;i++){
            clerk.get();
        }
    }
}

//消费者
class Consumer implements Runnable{
    private Clerk clerk;
    public consumer(Clerk clerk){
        this.clerk = clerk;
    }
    @override
    public void run() {
        for (int i=0;i<20;i++){
            clerk.sale();
        }
    }
}
```

#### 9、线程按序打印

> 3 个线程，TD 分别为 A、B、C，每个线程将 TD 打印 10 遍，输出结果顺序显示，如：ABCABCABC.....

```java
class AlternateDemo{
    private int number = 1;//当前正在执行线程的标记
    private Lock lock = new ReentrantLock();
    private Condition condition1 = lock.newCondition();
    private Condition condition2 = lock.newCondition();
    private Condition condition3 = lock.newCondition();
    
    public void loopA(){
        lock.lock();
        try {
            //判断
            if(number != 1){
                condition1.await();
            }
            //打印
            for (int i=1;i<=1;i++){
                system.out.print1n(Thread.currentThread().getName() + "\t" + i + "\t" + totalLoop);
            }
            //唤醒
            number = 2;
            condition2.signal();
        }catch (Exception e) {
            e.printstackTrace();
        }finally{
            lock.unlock();
        }
    }
    
    public void loopB(){
        lock.lock();
        try {
            //判断
            if(number != 2){
                condition2.await();
            }
            //打印
            for (int i=1;i<=1;i++){
                system.out.print1n(Thread.currentThread().getName() + "\t" + i + "\t" + totalLoop);
            }
            //唤醒
            number = 3;
            condition3.signal();
        }catch (Exception e) {
            e.printstackTrace();
        }finally{
            lock.unlock();
        }
    }
    
    public void loopC(){
        lock.lock();
        try {
            //判断
            if(number != 3){
                condition3.await();
            }
            //打印
            for (int i=1;i<=1;i++){
                system.out.print1n(Thread.currentThread().getName() + "\t" + i + "\t" + totalLoop);
            }
            //唤醒
            number = 1;
            condition1.signal();
        }catch (Exception e) {
            e.printstackTrace();
        }finally{
            lock.unlock();
        }
    }
}

public class TestABCAlternate {
    public static void main(String[] args) {
        AlternateDemo ad = new AlternateDemo();
        
        new Thread(new Runnable() {
            @override
            public void run() {
                for (int i=1;i<20;i++) {
                    ad.loopA(i);
                }
            }
        },"a").start();
        
        new Thread(new Runnable() {
            @override
            public void run() {
                for (int i=1;i<20;i++) {
                    ad.loopB(i);
                }
            }
        },"b").start();
        
        new Thread(new Runnable() {
            @override
            public void run() {
                for (int i=1;i<20;i++) {
                    ad.loopC(i);
                }
            }
        },"c").start();
    }
}

```

#### 10、读写锁 ReadWriteLock

- 写写/读写要互斥
- 读读不需互斥

````java
class ReadwriteLockDemo{
    private int number = 0;
    private ReadwriteLock lock = new ReentrantReadwriteLock();
    //读
    public void get(){
        lock.readLock().1ock();	//上锁
        try{
            system.out.println(Thread.currentThread().getName() + ":" + number);
        }finally{
            lock.readLock().unlock(); //释放锁
        }
    }
    //写
    public void set(int number){
        lock.writeLock().1ock();
        try{
            System.out.print1n(Thread.currentThread().getName());
            this.number = number;
        }finally{
            lock.writeLock().un1ock();
        }
    }
}

public class TestReadwriteLock{
    public static void main(String[] args){
        ReadlwriteLockDemo rw = new ReadwriteLockDemo();
        
        new Thread(new Runnable() {
            @override
            public void run(){
                rw.set((int)(Math.random() * 101));
            }
        },"write:").start();
        
        for (int i=0;i<100;i++){
            new Thread(new Runnable){
                @override
                public void run() {
                    rw.get();
                }
            }).start();
        }
    }
}
````

#### 11、线程八锁：

- 判断打印的 one or two，one 延迟 1s
  - 两个普通同步方法，两个线程标准打印，一个 Number 对象，结果 one two
  - 新增 `Thread.sleep()` 给 `getone()`，结果 one two
  - 新增普通方法 `getThree()` ，三个线程标准打印，结果 three one two
  - 两个普通同步方法，两个 Number 对象，结果 two one
  - 修改 `getone()` 为静态同步方法，一个Number 对象，结果 two one
  - 修改两个方法均为静态同步方法，结果 one two
  - 一个静态同步方法，一个非静态同步方法，两个 Number 对象，结果 two one
  - 两个静态同步方法，两个 Number 对象，结果 one two
- 关键
  - 非静态方法锁默认为 this，静态方法锁为对应 Class 实例
  - 某一个时刻只有一个线程持有锁，无论几个方法

#### 12、线程池

> 提供了一个线程队列，队列中保存所有等待状态的线程。避免创建与销毁额外开销，提高响应速度

- 体系结构:

  - java.util.concurrent.Executor：负责线程使用、调度的根接口

    - ExecutorService 子接口：线程池主要接口

      > ThreadPoolExecutor：线程池实现类
      > ScheduledExecutorService 子接口：负责线程调度
      >
      > - ScheduledThreadPoolExecutor：继承 ThreadPoolExecutor，实现  ScheduledExecutorService

- 工具类：Executors

  - `ExecutorService newFixedThreadPool()`：创建固定大小线程池
  - `ExecutorService newCachedThreadPool()`：缓存线程池，线程池数量不固定，可根据需求自动更改
  - `ExecutorService newSingleThreadExecutor()`：创建单个线程池，线程池中只有一个线程
  - `ScheduledExecutorService newScheduledThreadPool()`：创建固定大小线程池，可延迟、定时执行任务
  
- 参数

  - corePoolSize：线程池核心线程大小

    - 即使核心线程处于空闲状态也不会被销毁，除非设置 allowCoreThreadTimeOut

  - maximumPoolSize：线程池最大线程数量

    - 一个任务被提交到线程池后，先找有无空闲存活线程，有则直接将任务交给该空闲线程，没有则缓存到工作队列中，工作队列满了才创建一个新线程，然后从工作队列头部取出一个任务交由新线程处理，将刚提交的任务放入工作队列尾部
    - 线程池不会无限制创建新线程，有一个最大线程数量限制

  - keepAliveTime：空闲线程存活时间

    - 一个线程如果处于空闲状态，且当前线程数量大于 corePoolSize，在 keepAliveTime 后，该空闲线程会被销毁

  - unit：空闲线程存活时间单位：keepAliveTime 的计量单位

  - workQueue：工作队列

    - 新任务被提交后先进入此工作队列中，任务调度时再从队列中取出任务，jdk 提供四种工作队列

      > ArrayBlockingQueue：基于数组，有界，FIFO，可防止资源耗尽。当线程池线程数量达到 corePoolSize 后，有新任务进来会将任务放入该队列队尾，等待被调度。如果队列已满，则创建一个新线程，如果线程数量已达 maxPoolSize，执行拒绝策略
      >
      > LinkedBlockingQuene：基于链表，无界（最大容量 Interger.MAX），FIFO。由于近似无界，当线程池线程数量达到 corePoolSize 后，有新任务进来会一直存入该队列而不去创建新线程直到 maxPoolSize，参数 maxPoolSize 不起作用
      >
      > SynchronousQuene：不缓存任务，新任务进来时不缓存，直接被调度执行该任务，如果无可用线程，创建新线程，如果线程数达到 maxPoolSize，执行拒绝策略
      >
      > PriorityBlockingQueue：有优先级，无界，优先级通过参数 Comparator 实现

  - threadFactory：线程工厂，创建新线程使用的工厂，可设定线程名、是否为 daemon（守护）线程等

  - handler：拒绝策略

    - 当工作队列中任务已达最大限制，且线程池中线程数量达最大限制，如果有新任务提交，执行拒绝策略，jdk 提供 4 种拒绝策略

      > CallerRunsPolicy：在调用者线程中直接执行被拒绝任务的 run 方法，除非线程池已 shutdown，则抛弃任务
      >
      > AbortPolicy：丢弃任务，抛出 RejectedExecutionException 异常
      >
      > DiscardPolicy：丢弃任务，什么都不做
      >
      > DiscardOldestPolicy：抛弃进入队列最早的任务，尝试把拒绝的任务放入队列
      >
      > 该策略下，抛弃进入队列最早的那个任务，然后尝试把这次拒绝的任务放入队列
      >
      > 该策略下，直接丢弃任务，什么都不做。



