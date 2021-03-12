### APP 启动流程

<img src="C:\Users\13085\Desktop\git_work\Android\Android笔记\Image\APP 启动流程_流程图.jpg" alt="APP 启动流程_流程图" style="zoom:80%;" />

- 点击桌面 App 图标，Launcher 进程采用 Binder IPC 向 system_server 进程发起 startActivity 请求
- system_server 进程接收到请求后，向 zygote 进程发送创建进程的请求
- Zygote 进程 fork 出新子进程，即 App 进程
- App 进程通过 Binder IPC 向 sytem_server 进程发起 attachApplication 请求
- system_server 进程收到请求后，进行一系列准备工作后，通过 binder IPC 向 App 进程发送 scheduleLaunchActivity 请求
- App 进程的 binder 线程（ApplicationThread）收到请求后，通过 handler 向主线程发送 LAUNCH_ACTIVITY 消息
- 主线程收到 Message 后，通过发射机制创建目标 Activity，回调 Activity.onCreate() 等方法
- App 正式启动，进入 Activity 生命周期，执行完 onCreate/onStart/onResume 方法，UI 渲染结束后看到 App 主界面

#### （1）zygote

- Linux 中所有进程由 init 进程直接或间接 fork 出来，zygote 进程也是，init 进程在 Linux 内核加载完后启动
- 每个 APP 都有一个单独的虚拟机，都是一个的单独的进程

#### （2）system_server

- SystemServer 进程由 zygote 进程 fork 出来，系统重要服务在该进程里开启，如 ActivityManagerService、PackageManagerService、WindowManagerService 等
- App 与 AMS 通过 Binder 进行 IPC 通信，AMS（SystemServer 进程）与 zygote 通过 Socket 进行 IPC 通信（**因为 zygote 比 service manager 先启动，没有 service manager 可以注册，无法使用 binder，socket 的所有者是 root，只有系统权限的用户才能读写，多了一个安全保障；zygote 必须保证单线程（fork，可能造成死锁），binder 是多线程的**）
- AMS
  - 打开一个 App 需要 AMS 通知 zygote 进程，所有 Activity 的开启、暂停、关闭都需要 AMS 控制，AMS 负责系统中所有 Activity 的生命周期
  - 任一个 Activity 的启动都是由 AMS 和应用程序进程（主要是 ActivityThread）相互配合完成。AMS 服务统一调度系统所有进程的 Activity 启动，每个 Activity 的启动过程由其所属进程具体来完成

#### （3）Launcher

- Launcher 本质也是一个应用程序，和 App 一样继承自 Activity

  ```java
  packages/apps/Launcher2/src/com/android/launcher2/Launcher.java
  
  public final class Launcher extends Activity implements View.OnClickListener, OnLongClickListener, LauncherModel.Callbacks, View.OnTouchListener { }
  ```

  > Launcher 实现点击、长按等回调接口来接收用户输入。点击图标时，捕捉图标点击事件， startActivity() 发送对应的 Intent 请求

#### （4）Instrumentation 和 ActivityThread

- 每个 Activity 都持有 Instrumentation 对象的一个引用，但整个进程只会存在一个 Instrumentation 对象，Instrumentation 类是完成对 Application 和 Activity 初始化和生命周期的工具类（大管家）
- ActivityThread 依赖于 UI 线程，App 和 AMS 通过 Binder 传递信息，ActivityThread 专门与 AMS 的传递消息

#### （5）ApplicationThread

- 客户端 —> 服务端
  - 由于继承同样公共接口类，ActivityManagerProxy 提供与 ActivityManagerService 一样的函数原型，用户感觉不出 Server 运行在本地还是远端，可更方便调用重要系统服务

<img src="C:\Users\13085\Desktop\git_work\Android\Android笔记\Image\APP 启动流程_客到服.jpg" alt="APP 启动流程_客到服" style="zoom:80%;" />

- 服务端 —> 客户端

  - 都实现相同接口 IApplicationThread

    ```java
    private class ApplicationThread extends ApplicationThreadNative {} 
    public abstract class ApplicationThreadNative extends Binder implements IApplicationThread{} 
    class ApplicationThreadProxy implements IApplicationThread {}
    ```

<img src="C:\Users\13085\Desktop\git_work\Android\Android笔记\Image\APP 启动流程_服到客.jpg" alt="APP 启动流程_服到客" style="zoom:80%;" />

#### （6）启动流程

- 创建进程

  - 从 Launcher 的 startActivity() 方法，通过 Binder 通信，调用 ActivityManagerService 的 startActivity 方法
  - 调用 `startProcessLocked()` 方法创建新进程
  - 该方法通过 socket 通道传递参数给 Zygote 进程，Zygote 孵化自身，调用 `ZygoteInit.main()` 方法实例化 ActivityThread 对象最终返回新进程 pid
  - 调用 `ActivityThread.main()` 方法，随后依次调用 `Looper.prepareLoop()` 和 `Looper.loop()` 开启消息循环

  ![APP 启动流程_创建进程](C:\Users\13085\Desktop\git_work\Android\Android笔记\Image\APP 启动流程_创建进程.jpg)

  - 直白流程解释：
    - App 发起进程：从桌面启动应用，发起进程是 Launcher 所在进程；从某 App 内启动远程进程，发送进程便是该 App 所在进程。发起进程先通过 binder 发送消息给 system_server 进程
    - system_server 进程：调用 `Process.start()` 方法，通过 socket 向 zygote 进程发送创建新进程请求
    - zygote 进程：执行 `ZygoteInit.main()` 后进入 `runSelectLoop()` 循环体内，有客户端连接时执行 ZygoteConnection.runOnce() 方法，经层层调用后 fork 出新应用进程
    - 新进程：执行 handleChildProc 方法，最后调用 `ActivityThread.main()` 方法

  <img src="C:\Users\13085\Desktop\git_work\Android\Android笔记\Image\APP 启动流程_创建进程直白版.jpg" alt="APP 启动流程_创建进程直白版" style="zoom: 67%;" />

- 绑定 Application
  - 创建进程后执行 `ActivityThread.main()` 方法，随后调用 attach() 方法
  - 将进程和指定的 Application 绑定起来，通过 ActivityThread 对象中调用 `bindApplication()` 方法完成,该方法发送一个 BIND_APPLICATION 的消息到消息队列中, 通过 `handleBindApplication()` 方法处理该消息，然后调用 `makeApplication()` 方法加载 App 的 classes 到内存中

  ![APP 启动流程_绑定 Application](C:\Users\13085\Desktop\git_work\Android\Android笔记\Image\APP 启动流程_绑定 Application.jpg)
  - 直白流程解释：

  <img src="C:\Users\13085\Desktop\git_work\Android\Android笔记\Image\APP 启动流程_绑定 Application直白版.jpg" alt="APP 启动流程_绑定 Application直白版" style="zoom:67%;" />

- 显示 Activity 界面
  - 系统已拥有该 application 的进程。后面的调用顺序是普通的从一个已存在进程中启动一个新进程的 activity
  - 实际调用方法是 `realStartActivity()`，它会调用 application 线程对象中的 `scheduleLaunchActivity()` 发送一个 LAUNCH_ACTIVITY 消息到消息队列中，通过 `handleLaunchActivity()` 处理。`handleLaunchActivity()` 通过 `performLaunchActiivty()` 回调 Activity 的 `onCreate()` 方法和 `onStart()` 方法，通过 `handleResumeActivity()` 方法，回调 Activity 的 `onResume()` 方法，最终显示 Activity 界面

  ![APP 启动流程_显示 activity](C:\Users\13085\Desktop\git_work\Android\Android笔记\Image\APP 启动流程_显示 activity.jpg)
  - 直白流程解释：

  <img src="C:\Users\13085\Desktop\git_work\Android\Android笔记\Image\APP 启动流程_显示 activity直白版.jpg" alt="APP 启动流程_显示 activity直白版" style="zoom:67%;" />

#### （7）Binder 通信

- system_server 进程中调用 startProcessLocked 方法，该方法最终通过 socket 方式将需要创建新进程的消息告知 Zygote 进程并阻塞等待 Socket 返回新创建进程的 pid
- Zygote 进程接收到 system_server 发送过来的消息，通过 fork 方法，将自身进程复制生成新进程，并将 ActivityThread 相关资源加载到新进程 app process，这个进程用于承载 activity 等组件
- app process 向 servicemanager 查询 system_server 进程中 binder 服务端 AMS，获取对应 Client 端 AMP，有了这对 binder c/s 对，app process 可通过 binder 跨进程向 system_server 发送请求，`attachApplication()` 
- system_server 进程接收到相应 binder 操作后经多次调用，利用 ATP 向 app process 发送 binder 请求，即  `bindApplication()`，system_server 拥有 ATP/AMS，每个新创建进程都有一个相应的 AT/AMP，从而可跨进程互相通信

<img src="C:\Users\13085\Desktop\git_work\Android\Android笔记\Image\APP 启动流程_Binder通信.jpg" alt="APP 启动流程_Binder通信" style="zoom:67%;" />