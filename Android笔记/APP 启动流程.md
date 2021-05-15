### APP 启动流程

<img src="Image.assets\APP 启动流程_流程图.jpg" alt="APP 启动流程_流程图" style="zoom:80%;" />

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

<img src="Image.assets\APP 启动流程_客到服.jpg" alt="APP 启动流程_客到服" style="zoom:80%;" />

- 服务端 —> 客户端

  - 都实现相同接口 IApplicationThread

    ```java
    private class ApplicationThread extends ApplicationThreadNative {} 
    public abstract class ApplicationThreadNative extends Binder implements IApplicationThread{} 
    class ApplicationThreadProxy implements IApplicationThread {}
    ```

<img src="Image.assets\APP 启动流程_服到客.jpg" alt="APP 启动流程_服到客" style="zoom:80%;" />

#### （6）启动流程

- 创建进程

  - 从 Launcher 的 startActivity() 方法，通过 Binder 通信，调用 ActivityManagerService 的 startActivity 方法
  - 调用 `startProcessLocked()` 方法创建新进程
  - 该方法通过 socket 通道传递参数给 Zygote 进程，Zygote 孵化自身，调用 `ZygoteInit.main()` 方法实例化 ActivityThread 对象最终返回新进程 pid
  - 调用 `ActivityThread.main()` 方法，随后依次调用 `Looper.prepareLoop()` 和 `Looper.loop()` 开启消息循环

  ![APP 启动流程_创建进程](Image.assets\APP 启动流程_创建进程.jpg)

  - 直白流程解释：
    - App 发起进程：从桌面启动应用，发起进程是 Launcher 所在进程；从某 App 内启动远程进程，发送进程便是该 App 所在进程。发起进程先通过 binder 发送消息给 system_server 进程
    - system_server 进程：调用 `Process.start()` 方法，通过 socket 向 zygote 进程发送创建新进程请求
    - zygote 进程：执行 `ZygoteInit.main()` 后进入 `runSelectLoop()` 循环体内，有客户端连接时执行 ZygoteConnection.runOnce() 方法，经层层调用后 fork 出新应用进程
    - 新进程：执行 handleChildProc 方法，最后调用 `ActivityThread.main()` 方法

  <img src="Image.assets\APP 启动流程_创建进程直白版.jpg" alt="APP 启动流程_创建进程直白版" style="zoom: 67%;" />

- 绑定 Application
  - 创建进程后执行 `ActivityThread.main()` 方法，随后调用 attach() 方法
  - 将进程和指定的 Application 绑定起来，通过 ActivityThread 对象中调用 `bindApplication()` 方法完成,该方法发送一个 BIND_APPLICATION 的消息到消息队列中, 通过 `handleBindApplication()` 方法处理该消息，然后调用 `makeApplication()` 方法加载 App 的 classes 到内存中

  ![APP 启动流程_绑定 Application](Image.assets\APP 启动流程_绑定 Application.jpg)
  - 直白流程解释：

  <img src="Image.assets\APP 启动流程_绑定 Application直白版.jpg" alt="APP 启动流程_绑定 Application直白版" style="zoom:67%;" />

- 显示 Activity 界面
  - 系统已拥有该 application 的进程。后面的调用顺序是普通的从一个已存在进程中启动一个新进程的 activity
  - 实际调用方法是 `realStartActivity()`，它会调用 application 线程对象中的 `scheduleLaunchActivity()` 发送一个 LAUNCH_ACTIVITY 消息到消息队列中，通过 `handleLaunchActivity()` 处理。`handleLaunchActivity()` 通过 `performLaunchActiivty()` 回调 Activity 的 `onCreate()` 方法和 `onStart()` 方法，通过 `handleResumeActivity()` 方法，回调 Activity 的 `onResume()` 方法，最终显示 Activity 界面

  ![APP 启动流程_显示 activity](Image.assets\APP 启动流程_显示 activity.jpg)
  - 直白流程解释：

  <img src="Image.assets\APP 启动流程_显示 activity直白版.jpg" alt="APP 启动流程_显示 activity直白版" style="zoom:67%;" />

#### （7）Binder 通信

- system_server 进程中调用 startProcessLocked 方法，该方法最终通过 socket 方式将需要创建新进程的消息告知 Zygote 进程并阻塞等待 Socket 返回新创建进程的 pid
- Zygote 进程接收到 system_server 发送过来的消息，通过 fork 方法，将自身进程复制生成新进程，并将 ActivityThread 相关资源加载到新进程 app process，这个进程用于承载 activity 等组件
- app process 向 servicemanager 查询 system_server 进程中 binder 服务端 AMS，获取对应 Client 端 AMP，有了这对 binder c/s 对，app process 可通过 binder 跨进程向 system_server 发送请求，`attachApplication()` 
- system_server 进程接收到相应 binder 操作后经多次调用，利用 ATP 向 app process 发送 binder 请求，即  `bindApplication()`，system_server 拥有 ATP/AMS，每个新创建进程都有一个相应的 AT/AMP，从而可跨进程互相通信

<img src="Image.assets\APP 启动流程_Binder通信.jpg" alt="APP 启动流程_Binder通信" style="zoom:67%;" />

#### （8）Android开机启动流程

![img](https://upload-images.jianshu.io/upload_images/2929448-29279bde1a08d782.png)

- Boot Rom：引导芯片从固化在 ROM 的预设代码开始执行，然后加载引导程序到 RAM

- BootLoader（引导程序）：在操作系统运行前运行的一段程序（运行的第一个程序）。检查 RAM、初始化硬件参数等功能，最终目的是拉起操作系统

  > 文件路径： /bootable/bootloader/legacy/

  - 主要作用：把操作系统映像文件拷贝到 RAM 中，跳转到其入口处执行，称为启动加载模式，该过程没有用户介入

    - 第一步

      > 1、硬件设备初始化。为第二步的执行及随后内核的执行准备好基本的硬件环境
      >
      > 2、为第二步准备 ram 空间。为获得更好的执行速度，通常把第二步加载到 ram 中执行
      >
      > 3、复制第二步的代码到 ram 中
      >
      > 4、设置好堆栈
      >
      > 5、跳转到第二步的 c 程序入口

    - 第二步

      >1、初始化本阶段要使用的硬件设备
      >
      >2、检测系统内存映射
      >
      >3、将内核映像和根文件系统映像从 flash 读到 ram 中
      >
      >4、为内核设置启动参数
      >
      >5、调用内核

- 初始化 Kernel：C 语言编写，入口函数是 start_kernel 函数，该函数完成内核的大部分初始化工作。可将其看做内核的main函数。其最后调用了 reset_init 函数进行后续的初始化。该函数主要任务是启动内核线程 kernel_init，kernel_init 函数完成设备驱动程序的初始化，并调用 init_post 函数启动用户空间的 init 进程。init_post 函数结束，内核初始化基本完成

  > 文件路径：/kernel_imx/init/main.c

- init 进程（祖先进程）：Linux 中所有进程都由 init 进程直接或间接 fork 出来的。init 进程负责创建系统中最关键的几个子进程，尤其是 zygote。还提供 property service（属性服务），类似 windows 系统的注册表服务

  - Android 系统中有个 init.rc 脚本。init 进程启动会读取并解析该脚本文件，把其中的元素整理成自己的数据结构（链表）

  > 文件路径：
  >  /system/core/init/init.c
  >  /system/core/rootdir/init.rc
  >  /system/core/init/readme.txt

- Zygote 进程：init 进程创建后 fork 出来的，该进程是所有 Java 进程的父进程，会 fork 出一个 Zygote Java 进程用来 fork 出其他进程。zygote 开启时会调用 `ZygoteInit.main()` 进行初始化

```java
public static void main(String argv[]) {
    //...
    // 加载zygote的时候，会传入参数，startSystemServer变为true
    boolean startSystemServer = false;
    for (int i=1;i<argv.length;i++) {
        if ("start-system-server".equals(argv[i])) {
            startSystemServer = true;
        }else if (argv[i].startsWith(ABI_LIST_ARG)) {
            abiList = argv[i].substring(ABI_LIST_ARG.length());
        }else if (argv[i].startsWith(SOCKET_NAME_ARG)) {
            socketName = argv[i].substring(SOCKET_NAME_ARG.length());
        }else {
            throw new RuntimeException("Unknown command line argument: " + argv[i]);
        }
    }
    //...
    // 启动的SystemServer进程
    if (startSystemServer) {
        startSystemServer(abiList, socketName);
    }
    //...
}
```

> 文件路径：/frameworks/base/core/java/com/android/internal/os/ZygoteInit.java

- SystemServer 进程：ZygoteInit.java 通过 `startSystemServer()` fork 出 SystemServer 进程，和 Zygote 进程是 Android Framework 层的两大重要进程。系统重要的服务都在该进程开启，如 AMS、WindowsManager、PackageManagerService 等，SystemServer 进程开启时会初始化 ActivityManagerService、加载本地系统的服务库，调用 `createSystemContext()` 创建系统上下文，创建 ActivityThread 及开启各种服务等

```java
public final class SystemServer {
    // The main entry point from zygote.
    public static void main(String[] args) {
        new SystemServer().run();
    }
    
    public SystemServer() {
        // Check for factory test mode.
        mFactoryTestMode = FactoryTest.getMode();
    }
    
    private void run() {
        //...
        // 初始化原生服务库
        System.loadLibrary("android_servers");
        nativeInit();
        // 初始化系统上下文
        createSystemContext();
        // 创建SystemServiceManager对象
        mSystemServiceManager = new SystemServiceManager(mSystemContext);
        // 开启服务
        try {
            startBootstrapServices();
            startCoreServices();
            startOtherServices();
        } catch (Throwable ex) {
            Slog.e("System", "******************************************");
            Slog.e("System", "************ Failure starting system services", ex);
            throw ex;
        }
        //...
        // Loop forever.
        Looper.loop();
        throw new RuntimeException("Main thread loop unexpectedly exited");
    }
    
    //初始化系统上下文对象mSystemContext，并设置默认的主题。
    private void createSystemContext() {
        ActivityThread activityThread = ActivityThread.systemMain();
        mSystemContext = activityThread.getSystemContext();
        mSystemContext.setTheme(android.R.style.Theme_DeviceDefault_Light_DarkActionBar);
    }

    //在这里开启了几个核心的服务，因为这些服务之间相互依赖，所以都放在了这个方法里面。
    private void startBootstrapServices() {
        //...
        //初始化ActivityManagerService
        mActivityManagerService = mSystemServiceManager
            .startService(ActivityManagerService.Lifecycle.class).getService();
        mActivityManagerService.setSystemServiceManager(mSystemServiceManager);
        //初始化PowerManagerService，因为其他服务需要依赖这个Service，因此需要尽快的初始化
        mPowerManagerService = mSystemServiceManager.startService(PowerManagerService.class);
        // 现在电源管理已经开启，ActivityManagerService负责电源管理功能
        mActivityManagerService.initPowerManagement();
        // 开启DisplayManagerService
        mDisplayManagerService = mSystemServiceManager.startService(DisplayManagerService.class);
        // 开启PackageManagerService
        mPackageManagerService = PackageManagerService.main(mSystemContext,mInstaller, mFactoryTestMode != FactoryTest.FACTORY_TEST_OFF, mOnlyCore);
        //...
    }
    private void startCoreServices() {...}// 启动一些基本服务。
    private void startOtherServices() {...}// 启动其他服务。
}
```

> 文件路径：/frameworks/base/services/java/com/android/server/SystemServer.java

- Home Activity：ActivityManagerService 开启后会调用 `finishBooting()` ，完成引导过程。同时发送开机广播 ACTION_BOOT_COMPLETED，之后开启系统主程序（Launcher 程序），完成系统界面的加载与显示

```java
final void finishBooting() {
    //...
    final int userId = mStartedUsers.keyAt(i);
    Intent intent = new Intent(Intent.ACTION_BOOT_COMPLETED, null);
    intent.putExtra(Intent.EXTRA_USER_HANDLE, userId);
    intent.addFlags(Intent.FLAG_RECEIVER_NO_ABORT);
    broadcastIntentLocked(...);
    //...
}
```

> 文件路径：/frameworks/base/services/java/com/android/server/am/ActivityManagerService.java