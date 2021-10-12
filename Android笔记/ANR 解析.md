### ANR 解析

> 若应用程序有一段时间响应不够灵敏，系统会向用户显示一个对话框——应用程序无响应**（ANR：Application Not Responding）**，用户可以强制退出 APP，避免卡机无响应，这是 Android 系统的一种自我保护机制

- 产生原因：根本原因是 APP 阻塞了 UI 线程。android 系统中每个 App 只有一个 UI 线程，在 App 创建时默认生成，UI 线程默认初始化一个消息循环来处理 UI 消息，ANR 往往就是处理 UI 消息超时

- 类型：

  > Service与Bradcast只会打印trace信息，不会提示用户ANR弹窗，大部分可感知的ANR都是由于InputEvent

  - KeyDispatchTimeout(5 seconds) –主要类型：按键或触摸事件在特定时间内无响应
  - BroadcastTimeout(10 seconds)：BroadcastReceiver在特定时间内无法处理完成
  - ServiceTimeout(20 seconds) –小概率类型：Service在特定的时间内无法处理完成

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
  
- 常用场景

  - UI线程等待其它线程释放某个锁，导致UI线程无法处理用户输入
  - 游戏中每帧动画都进行了比较耗时的大量计算，导致CPU忙不过来
  - Web应用中网络状态不稳定，而界面在等待网络数据
  - UI线程中进行了一些磁盘IO（包括数据库、SD卡等等）的操作，在个别设备上因为硬件损坏等原因阻塞住了
  - 手机被其他App占用着CPU，自己获取不到足够的CPU 时间片，纯属误伤

- 通过ANR 日志定位问题：当ANR发生时，通过Logcat和traces文件（目录/data/anr/）的相关信息输出去定位问题，主要包含：

  - 基本信息，包括进程名、进程号、包名、系统build号、ANR 类型等
  - CPU使用信息，包括活跃进程的CPU 平均占用率、IO情况等
  - 线程堆栈信息，所属进程包括发生ANR的进程、其父进程、最近有活动的3个进程等等

- 导出ANR文件：

  ```
  // ANR发生后导出ANR文件：
  adb pull data/anr/traces.txt 存储路径
  
  //生成
  adb bugreport
  //导出
  adb pull 文件路径 存储路径
  ```

- 检测工具

  - FileObserver：监听ANR目录的变化
  - ANR-WatchDod：监听并打印ANR堆栈信息
