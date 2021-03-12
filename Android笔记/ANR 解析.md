### ANR 解析

> 若应用程序有一段时间响应不够灵敏，系统会向用户显示一个对话框——应用程序无响应**（ANR：Application Not Responding）**，用户可以强制退出 APP，避免卡机无响应，这是 Android 系统的一种自我保护机制

- 产生原因：根本原因是 APP 阻塞了 UI 线程。android 系统中每个 App 只有一个 UI 线程，在 App 创建时默认生成，UI 线程默认初始化一个消息循环来处理 UI 消息，ANR 往往就是处理 UI 消息超时

- UI 消息来源：

  - AMS 的回调消息：AMS 负责管理应用程序四大组件生命周期，当 AMS 对应用程序组件的生命周期进行回调超过 AMS 定义的响应时间时报 ANR。一般是因为在这些组件的回调函数里进行了耗时操作（如网络操作、SD 卡文件操作、数据库操作、大量计算等），AMS 对组件常见的回调函数及超时时间如下：

    > Activity/View：KeyDispatch Timeout，View 按钮事件或者触摸事件在 **5s** 内无法响应
    >
    > Service：Service Timeout，Service 各个生命周期函数在 **20s** 内无法完成处理
    >
    > BroadcastReceiver：Broadcast Timeout，BroadcastReceiver 的 `onReceiver()` 运行在主线程中，在 **10s** 内无法完成处理，后台广播 **20s** 

  - App 自己发出的消息：

    > AsyncTask：onPreExecute()，onProgressUpdate()，onPostExecute()，onCancel() 等，超时 5s
    >
    > Mainthread handler：handleMessage()，post*(runnable r) 等，超时 5s

- 避免 ANR
  - UI 线程尽量只做跟 UI 相关的工作，一些复杂的 UI 操作需要技巧处理，如让一个 Button 去 setText 一个 10M 的文本，UI 肯定崩掉了，分段加载貌似是最好的方法
  -  耗时工作（如数据库操作，I/O，连接网络或者别的有可能阻碍 UI 线程的操作）放入单独的线程处理
  -  尽量用 Handler 来处理 UIthread 和别的 thread 之间的交互

### WMS（WindowManagerService）

> AMS、WMS 都属于 Android 的系统服务

- Activity 与 Window

  - Activity 只负责生命周期和事件处理
  - Window 只控制视图
  - 一个 Activity 包含一个 Window，没有 Window 就相当于 Service

- AMS 与 WMS

  - AMS 统一调度所有应用程序的 Activity
  - WMS 控制所有 Window 的显示与隐藏以及要显示的位置

- WMS 作用

  - 为所有窗口分配 Surface，客户端向 WMS 添加一个窗口的过程就是 WMS 为其分配一块 Surface 的过程，一块块 Surface 在 WMS 的管理下有序排布在屏幕上
  - 管理 Surface 显示顺序、尺寸、位置
  - 管理窗口动画
  - 输入系统相关：WMS 派发系统按键和触摸消息，当接收一个触摸事件时寻找最合适的窗口来处理消息

- 设计模式——桥接模式

  ![WMS模式](C:\Users\13085\Desktop\git_work\Android\Android笔记\Image\WMS模式.png)

### AMS

- 作用

  - 统一调度所有应用程序的 Activity 的生命周期
  - 启动或杀死应用程序的进程
  - 启动并调度 Service 的生命周期
  - 注册 BroadcastReceiver，并接收和分发 Broadcast
  - 启动并发布 ContentProvider
  - 调度 task
  - 处理应用程序的 Crash
  - 查询系统当前运行状态

- 设计模式——代理模式

  ![AMS模式](C:\Users\13085\Desktop\git_work\Android\Android笔记\Image\AMS模式.png)