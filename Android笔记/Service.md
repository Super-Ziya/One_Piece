### Service

#### 1、分类

- 本地服务（LocalService）：本地服务依附在主进程上而不是独立进程，节约资源，在同一进程不需 IPC，不需要 AIDL，相应 bindService 会方便很多，主进程被 Kill 后服务终止
- 远程服务（RemoteService）：远程服务为独立进程，进程名格式为所在包名加上指定的 android:process 字符串，Activity 所在进程被 Kill 时候服务依然运行，不受其他进程影响，会占用一定资源，使用 AIDL 进行 IPC
- 使用方式
  - startService 启动的服务：主要用于启动一个服务执行后台任务，不进行通信，停止服务用 stopService
  - bindService 启动的服务：可进行通信，停止服务用 unbindService
  - startService、bindService 启动的服务：停止服务应同时使用 stepService 与 unbindService

#### 2、Service 与 Thread 的区别

- Thread：程序执行最小单元，分配 CPU 的基本单位，可执行一些异步操作
- Service：android 一种机制
  - Local Service 运行在主进程的 main 线程上，onCreate，onStart 函数被系统调用时在主进程的 main 线程上运行
  - RemoteService 运行在独立进程的 main 线程上

- 两者对比：
  - Thread 的运行独立于 Activity 的（Activity 被 finish 后，如果没有主动停止 Thread 或 Thread 的 run 方法没执行完时，Thread 会一直执行）
  - 没办法在不同 Activity 中对同一 Thread 进行控制，而任何 Activity 都可以控制同一 Service，系统只会创建一个对应 Service 的实例
  - Service 可在任何有 Context 的地方调用 Context.startService、Context.stopService、Context.bindService，Context.unbindService 来控制，也可在 Service 里注册 BroadcastReceiver，在其他地方通过发送 broadcast 来控制，Thread 做不到

#### 3、启动方式

- 通过 StartService 启动 Service

  > 启动后 service 会无限期运行下去，只有外部调用了 `stopService()` 或 `stopSelf()` 方法才会停止运行并销毁

  - 实现：继承 Service 类，重写方法（回调方法，在主线程中执行）

    - `onCreate()`

      > 如果 service 没被创建过，调用 `startService()` 后会执行 `onCreate()` 回调
      >
      > 如果 service 处于运行中，调用 `startService()` 不会执行 `onCreate()` 
      >
      > `onCreate()` 只在第一次创建 service 时调用，多次执行 `startService()` 不重复调用，适合完成一些初始化工作

    - `onStartCommand()`

      > 如果多次执行 Context 的 `startService()` 方法，Service 的 `onStartCommand()` 方法会相应多次调用。方法中根据传入的 Intent 参数进行实际的操作，如在此处创建一个线程用于下载数据或播放音乐等

    - `onBind()`

      > 抽象方法，Service 是抽象类，`onBind()` 必须重写

    - `onDestory()`

      > 销毁时执行

  - 每个 Service 必须在 AndroidManifest.xml 中注册

- 通过 bindService 启动 Service

  - 启动的服务和调用者之间是 client-server 模式，调用者是 client，可以有多个，服务是 server，只有一个
  - 可通过 IBinder 接口获取 Service 实例，实现在 client 端直接调用 Service 中的方法实现交互，这在通过 startService 启动无法实现
  - bindService 启动服务的生命周期与其绑定的 client 息息相关。当 client 销毁时会自动与 Service 解绑，client 也可明确调用 Context 的 `unbindService()` 与 Service 解绑，没有 client 与 Service 绑定时，Service 会自行销毁
  - 实现
    - Server 端：
    - 在 Service 的 `onBind()` 中返回 IBinder 类型实例
    - `onBInd()` 返回的 IBinder 实例要能够返回 Service 实例本身。最简单的方法是在 service 中创建 binder 内部类，加入类似 `getService()` 返回 Service，绑定的 client 可通过 `getService()` 获得 Service 实例
    - client 端：
    - 创建 ServiceConnection 类型实例，重写 `onServiceConnected()` 、`onServiceDisconnected()` 
    - 执行到 onServiceConnected 回调时，通过 IBinder 实例得到 Service 实例对象，可实现 client 与 Service 的连接
    - onServiceDisconnected 回调被执行时，表示 client 与 Service 断开连接，可以写一些断开连接后的处理

#### 4、保证 Service 不被杀死

##### （1）onStartCommand 返回值

> 调用 Context.startService 方式启动 Service 时，如果 Android 内存匮乏，可能会销毁当前运行的 Service，待内存充足时可重建。Service 被系统强制销毁并重建的行为依赖于 Service 的 `onStartCommand()` 返回值

- START_NOT_STICKY：表示 Service 运行的进程被系统强制杀掉后，不会重新创建该 Service
  - PS：Service 定时从服务器获取数据：通过一个定时器每隔 N 分钟启动 Service 获取。当执行到 Service 的 onStartCommand 时，在该方法内再规划 N 分钟后的定时器用于再次启动该 Service 并开辟一个新线程执行网络操作。假设 Service 从服务器获取时被系统强制杀掉，Service 不会重新创建，因为再过 N 分钟定时器会再次启动该 Service 并重新获取
- START_STICKY：表示 Service 运行的进程被系统强制杀掉后，系统会将该 Service 依然设为 started 状态（运行状态），但不再保存 onStartCommand 方法传入的 intent 对象，然后系统尝试再次重建该 Service，并执行 onStartCommand 回调方法，但 onStartCommand 的 Intent 参数为 null
  - PS：用来播放背景音乐功能的 Service
- START_REDELIVER_INTENT：表示 Service 运行的进程被系统强制杀掉后，与 START_STICKY 类似，系统会再次重建该 Service，并执行 onStartCommand，但系统会将 Service 被杀掉前最后一次传入 onStartCommand 中的 Intent 保留下来并再次传入重建后的 Service 的 onStartCommand

##### （2）提高 Service 优先级

> 在 AndroidManifest.xm l文件中对 intent-filter 可通过 android:priority = "1000" 属性设置优先级，1000 最高，越小优先级越低，适用于广播

##### （3）提升 Service 进程优先级

> 系统进程空间紧张时会照优先级自动进行进程回收

- 进程等级，按由高到低
  - 前台进程 foreground_app
  - 可视进程 visible_app
  - 次要服务进程 secondary_server
  - 后台进程 hiddena_app
  - 内容供应节点 content_provider
  - 空进程 empty_app
- 可使用 startForeground 将 service 放到前台状态降低杀死概率

##### （4）在 onDestroy 里重启 Service：

> 当 service 走到 `onDestroy()` 时，发送一个自定义广播，收到广播时重新启动 service

##### （5）系统广播监听 Service 状态

##### （6）APK 安装到 /system/app，变为系统级应用

#### 5、系统服务（SystemService）

![service_系统服务](Image.assets\service_系统服务.png)

- 常用系统服务	
  - 窗口管理服务 WindowManager
  - 电源管理服务 PowerManager
  - 通知管理服务 NotifacationManager
  - 振动管理服务 Vibrator
  - 电池管理服务 BatteryManager

https://blog.csdn.net/geyunfei_/article/details/78851024