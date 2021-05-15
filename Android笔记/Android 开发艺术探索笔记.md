## Android 开发艺术探索笔记

### 1、Activity 生命周期和启动模式

#### 1.1、典型情况下生命周期分析

- 对一个特定的 Activity，第一次启动回调: onCreate - onStart - onResume
- 当用户打开新的 Activity 或切换到桌面时，回调: onPause - onStopo
  - 特殊情况：如果新 Activity 采用透明主题，当前 Activity 不会回调 onStop
- 当用户再次回到原 Activity 时，回调: onRestart - onStart - onResume
- 当用户按 back 键回退时，回调: onPause - onStop - onDestroy
- 当 Activity 被系统回收后再次打开，生命周期回调和 (1) 一样
- 从整个生命周期来说，onCreate 和 onDestroy 是配对的，分别标识着 Activity 的创建和销毁，且只可能有一次调用
- 从 Activity 是否可见来说，onStart 和 onStop 是配对的，随着用户的操作或设备屏幕的点亮和熄灭，这两个方法可能被调用多次
- 从 Activity 是否在前台来说，onResume 和 onPause 是配对的，随着用户操作或者设备屏幕的点亮和熄灭，这两个方法可能被调用多次

#### 1.2、异常情况下生命周期分析

- 资源相关的系统配置发生改变（如横屏竖屏切换）导致 Activity 被杀死并重新创建：

  - 当系统配置发生改变后，Activity 会被销毁，其 onPause、onStop、onDestroy 均会被调用，由于 Activity 在异常情况下终止，系统会调用 onSaveInstanceState 来保存当前 Activity 的状态

  - onSaveInstanceState  调用时机在 onStop 前，和 onPause 没有既定时序关系

  - Activity 被重新创建后，系统会调用 onRestoreInstanceState，把 Activity 销毁时 onSaveInstanceState 方法所保存的 Bundle 对象作为参数同时传递给 onRestoreInstanceState 和 onCreate 方法。可通过onRestoreInstanceState 和 onCreate 判断 Activity 是否被重建，如果被重建可以取出之前保存的数据并恢复，时序上 onRestoreInstanceState 调用时机在 onStart 后

    - onRestoreInstanceState 被调用时其参数 Bundle savedInstanceState 一定有值，不用额外判断空
    - onCreate 如果正常启动其参数 Bundle savedInstanceState 为 null，要额外判断

  - 在 onSaveInstanceState 和 onRestoreInstanceState 方法中，系统自动做了一定的恢复工作。当  Activity 在异常情况下需要重新创建时，系统默认保存当前 Activity 的视图结构，并在 Activity 重启后恢复，如文本框中用户输入的数据、ListView 滚动位置等，和 Activity一样，每个 View 都有 onSaveInstanceState 和 onRestoreInstanceState 两个方法

  - 保存和恢复 View 层次结构，系统工作流程：

    > Activity 被意外终止时，Activity 调用 onSaveInstanceState 保存数据，然后 Activity 委托 Window 保存数据，接着 Window 再委托上面的顶级容器保存数据。顶层容器是一个 ViewGroupr，一般是  DecorView，顶层容器再去一一通知其子元素保存数据

- 资源内存不足导致低优先级 Activity 被杀死

  - 数据存储和恢复过程和（1）完全一致

  - Activity 按优先级从高到低：

    - 前台 Activity：正在和用户交互的 Activity，优先级最高
    - 可见但非前台 Activity：如 Activity 中弹出一个对话框，导致 Activity 可见但位于后台无法和用户直接交互
    - 后台 Activity：已经被暂停的 Activity，如执行 onStop，优先级最低

    > 当系统内存不足时，按照上述优先级杀死目标 Activity 所在进程，在后续通过 onSaveInstanceState 和 onRestoreInstanceState 存储和恢复数据
    >
    > 如果一个进程中没有四大组件在执行，这个进程将很快被系统杀死，后台工作适合放入 Service 中保证进程有一定的优先级

- 配置改变 Activity 不要重新创建：

  - 添加 `android:configChanges=""` 属性，指定多个值可以 | 连接

  - 系统配置

    - 常用：locale、orientation、keyboardHidden

    |        项目        | 含义                                                         |
    | :----------------: | :----------------------------------------------------------- |
    |        mcc         | SIM卡唯一标识IMSI（国际移动用户识别码）中的国家代码，由三位数字组成，中国为460。此项标识 mcc 代码发生改变 |
    |        mnc         | SIM卡唯一标识IMSI中的运营商代码，由两位数字组成，中国移动TD系统为00，中国联通为01，中国电信为03。此项标识 mnc 发生改变 |
    |       locale       | 设备本地位置发生改变，一般指切换系统语言                     |
    |    touchscreen     | 触摸屏发生改变，正常情况无法发生，可以忽略                   |
    |      keyboard      | 键盘类型发生改变，如使用外插键盘                             |
    |   keyboardHidden   | 键盘的可访问性发生改变，如用户调出键盘                       |
    |     navigation     | 系统导航方式发生改变，如采用轨迹球导航，很难发生，可以忽略   |
    |    screenLayout    | 屏幕布局发生改变，可能是用户激活了另外一个显示设备           |
    |     fontScale      | 系统字体缩放比例发生改变，如用户选择了一个新字号             |
    |       uiMode       | 用户界面模式发生改变，如开启夜间模式（API 8添加）            |
    |    orientation     | 屏幕方向发生改变，最常用，如旋转手机屏幕                     |
    |     screenSize     | 屏幕尺寸信息发生改变，当旋转设备屏幕时屏幕尺寸会发生变化，和编译选项有关，当编译选项中的minSdkVersion和targetSdkVersion均低于13时不会导致Activity重启，否则会（API 13添加） |
    | smallestScreenSize | 设备物理屏幕尺寸发生改变，和屏幕方向没关系。仅表示实际物理屏幕的尺寸改变，如用户切换外部显示设备，和screenSize一样，当编译选项中的minSdkVersion和targetSdkVersion均低于13时不会导致Activity重启（APl 13添加） |
    |  layoutDirection   | 布局方向发生变化，用的比较少，正常情况无须修改布局的 layoutDirection属性（API 17添加) |

#### 1.3、Activity 启动模式

- standard：默认，每次启动都会创建新实例，实例在启动它的 Activity 所在的栈中（非 Activity 类型的 Context 启动没有任务栈，会报错，可以加 FLAG_ACTIVITY_NEW_TASK 标记位）

- singleTop：栈顶复用，如果新 Activity 位于任务栈栈顶，Activity 不会被重新创建，其 onNewIntent 方法会被回调，通过其参数可以取出当前请求的信息。Activity 的 onCreate、onStart 不会被系统调用，因为没有发生改变

- singleTask：栈内复用，单例模式，系统也会回调其 onNewIntent。当一个具有 singleTask 模式的 Activity 请求启动后，系统先寻找是否存在其想要的任务栈，不存在就创建，然后创建实例放到栈中；如果存在要看栈中是否有其实例存在，有就调到栈顶并调用其 onNewIntent 方法，默认有 clearTop 效果

- singleInstance：单例模式，除了具有 singleTask 所有特性，其只能单独地位于一个任务栈中，当 Activity 启动后，系统会为其创建一个新任务栈，其独自在该任务栈中，由于栈内复用，后续请求不会创建新 Activity，除非任务栈被系统销毁

  > 假设有 2 个任务栈，前台任务栈：AB，后台任务栈：CD，CD 启动模式均为 singleTask，现在启动 D，整个后台任务栈会被切换到前台，这时整个后退列表变成 ABCD。当用户按 back 键时，列表中的 Activity 一一出栈

- TaskAffinity：任务相关性，标识一个 Activity 所需任务栈名字，默认所有 Activity 所需任务栈名字为应用包名

  - 主要和 singleTask 或 allowTaskReparenting 属性配对使用，其他情况无意义
    - 和 singleTask 配对使用，具有 Activity 目前任务栈名字，待启动 Activity 会运行在名字和 TaskAffinity 相同的任务栈中
    - 和 allowTaskReparenting 结合，当应用 A 启动应用 B 某个 Activity，如果其 allowTaskReparenting 属性为 true，当应用 B 被启动后，其会直接从应用 A 的任务栈转移到应用 B 的任务栈中

- 指定启动模式

  - AndroidMenifest

  ```xml
  <activity
            android: name="com.ryg.chapter_1.SecondActivity"
            android:configChanges="screenLayout"
            android: launchMode="singleTask"
            android: label="@string/app_name"/>
  ```

  - Intent

  ```java
  Intent intent = new Intent();
  intent.setClass(MainActivity.this,SecondActivity.class);
  intent.addFlags(Intent.FLAG_ACTIVITY_NEW_TASK);
  startActivity(intent);
  ```

  - 区别
    - 第二种优先级高于第一种，当两种同时存在时以第二种为准
    - 第一种无法直接为 Activity 设定 FLAG_ACTIVITY_CLEAR_TOP 标识，第二种无法为 Activity 指定 singleInstance 模式

#### 1.4、Activity Flags

- FLAG_ACTIVITY_NEW_TASK：为 Activity 指定 singleTask 启动模式
- FLAG_ACTIVITY_SINGLE_TOP：为 Activity 指定 singleTop 启动模式
- FLAG_ACTIVITY_CLEAR_TOP：启动时在同一任务栈中所有位于其上面的 Activity 都出栈，一般要和  FLAG_ACTIVITY_NEW_TASK 配合使用，这时实例如果存在，系统就会调用其 onNewIntent，如果采用 standard 模式，出栈后系统会创建新实例放入栈顶
- FLAG_ACTIVITY_EXCLUDE_FROM_RECENTS：不会出现在历史 Activity 列表中，等同于在 AndroidMenifest中指定 `android:excludeFromRecents="true”` 

#### 1.5、IntentFilter 匹配规则

> 一个 Intent 同时匹配 action 类别、category 类别、data 类别才算完全匹配，只有完全匹配才能成功启动目标 Activity
>
> 一个过滤列表中的 action、category 和 data 可以有多个
>
> 一个 Activity 中可以有多个 intent-filter，一个 Intent 只要能匹配任何一组 intent-filter 即可成功启动

- action

  - action：字符串，系统预定义一些 action，可以自定义
  - 匹配规则：Intent 中的 action 必须能够和过滤规则中的 action 匹配（字符串值完全一样）。一个过滤规则中可以有多个 action，只要 Intent 中的 action 能和过滤规则中的任一 action 相同即可，action 区分大小写

- category

  - category：字符串，系统预定义一些 category，可以自定义
  - 匹配规则：Intent 如果含有 category，那么所有 category 都必须和过滤规则中其中一个 category 相同（Intent 所有 category 必须是过滤规则中已经定义的 category）。Intent 可以没有 category，仍可匹配成功。系统在调用 startActivity 或 startActivityForResult 时候会默认为 Intent 加上 `android.intent.category.DEFAULT` category，为使 activity 能接收隐式调用，必须在 intent-filter 中指定 `android.intent.category.DEFAULT` category

- data

  - data 语法

    ```xml
    <data android:scheme="string"
          android:host="string"
          android:port="string"
          android:path-"string"
          android:pathPattern="string"
          android: pathPrefix="string"
          android:mimeType-"string"/>
    ```

  - data 组成：

    - mimeType：媒体类型，如 image/jpeg、audio/mpeg4-generic 和 video/* 等，表示图片、文本、视频等不同媒体格式

    - URI：结构 <scheme>://<host>:<port>/[<path>|<pathPrefix>|<pathPattern>]

      > Scheme：URI 模式，如 http、file、content 等，如果没有指定，URI 无效
      >
      > Host：URI 主机名，如 www.baidu.com，如果未指定，URI 无效
      >
      > Port：URI 端口号，如80，仅当 URI 中指定了 scheme 和 host 时 port 才有意义
      >
      > Path、pathPattern、pathPrefix：表述路径信息，path 表示完整路径信息；pathPattern 也表示完整路径信息，但可以包含通配符 “\*”，表示 0 或多个任意字符，由于正则表达式规范，想表示真实的字符串，“\*” 写成 “\\\\*”，“\” 写成 “\\\\\\\\”；pathPrefix 表示路径前缀信息

  - 匹配规则：

    - ```xml
      <intent-filter>
      	<data android:mimeType="image/*"
                .../>
      </intent-filter>
      ```

      指定媒体类型为所有类型的图片，Intent 中的 mimeType 属性必须为 image/* 才能匹配，这时 URI 的默认值为 content 和 file，URI 部分的 schema 必须为 content 或 file 才能匹配，如果要为 Intent 指定完整 data，必须调用 setDataAndType 方法，不能先调用 setData 再调用 setType，因为这两个方法彼此会清除对方的值

    - ```xml
      <intent-filter>
          <data android:mimeType="ideo/mpeg" 
                android:scheme="http"
                .../>
          <data android:mimeType="audio/mpeg" 
                android:scheme="http"
                .../>
      </intent-filter>
      ```

      指定两组 data 规则，且每个 data 都指定完整属性值，既有 URI 又有 mimeType。为匹配可以:

      ```java
      intent.setDataAndType(Uri.parse("http://abe"), "video/mpeg");
      //或者
      intent.setDataAndType(Uri.parse("http://abc"), "audio/mpeg");
      ```

    - ```xml
      <intent-filter ...>
          <data android:scheme="file" 
                android:host="waww.baidu.com"/>
          ...
      </intent-filter>
      <!--两者写法一样-->
      <intent-filter ...>
          <data android:scheme="file"/>
          <data android:host="www.baidu.com"/>
          ...
      </intent-filter>
      ```

- 判断有无 Activity 匹配隐式 Intent

  - PackageManager 的 resolveActivity 方法和 Intent 的 resolveActivity 方法，找不到匹配 Activity 返回 null

  - PackageManager 还提供 queryIntentActivities 方法，和 resolveActivity 方法不同是其不是返回最佳匹配的 Activity 信息而是返回所有成功匹配的 Activity 信息

    ```java
    public abstract List<ResolveInfo> queryIntentActivities(Intent intent,intflags);
    public abstract ResolveInfo resolveActivity(Intent intent,int flags);
    ```

    - 第二个参数要使用 MATCHDEFAULT_ONLY 标记位，含义是仅匹配在 intent-filter 中声明了 `<category android:name="android.intent.category.DEFAULT">` category 的 Activity
    - 使用这个标记位只要上述两个方法不返回 null，startActivity 一定成功。因为不含有 DEFAULT category 的 Activity 无法接收隐式 Intent 的

  ```xml
  <action android:name="android.intent.action.MAIN"/>
  <category android:name="android.intent.category.LAUNCHER"/>
  ```

  - 作用：标明入口 Activity 且会出现在系统的应用列表中，少任何一个都没意义也无法出现在系统应用列表中

### 2、IPC 机制

#### 2.1、Android IPC

> IPC（Inter-Process Communication），进程间通信或跨进程通信

- Windows 可通过剪贴板、管道和邮槽等进行进程间通信
- Linux 可通过命名管道、共享内容、信号量等进行进程间通信
- Android 进程间通信方式：Binder、Socket
- 多进程使用场景：
  - 一个应用因为某些原因自身需要采用多进程模式来实现，如有些模块由于特殊原因需要运行在单独的进程中，或者为了加大一个应用可使用内存需要通过多进程来获取多份内存空间，Android 对单个应用所使用的最大内存做了限制，早期一些版本是16MB，不同设备不同大小
  - 当前应用需向其他应用获取数据，必须采用跨进程方式获取，ContentProvider 查询数据时也是一种进程间通信

#### 2.2、Android 多进程模式

- 开启多进程
  - 给四大组件在 AndroidMenifest 中指定 `android:process` 属性
    - 进程名以 “:” 开头的进程属于当前应用的私有进程，其他应用组件不可以和其跑在同一进程
    - 进程名不以 “:” 开头的进程属于全局进程，其他应用通过 ShareUID 方式可以和它跑在同一进程
    - Android 系统为每个应用分配一个唯一 UID，具有相同 UID 的应用才能共享数据。但两应用通过 ShareUID 跑在同一个进程需要其有相同 ShareUID 且签名相同。此时其可互相访问私有数据，如  data 目录、组件信息等，不管是否跑在同一个进程中，如果在同一个进程中，还可以共享内存数据
  - 通过 JNI 在native 层 fork 一个新进程（特殊情况）

- 一般使用多进程会造成：
  - 静态成员和单例模式完全失效
    - Android 为每个进程分配一个独立虚拟机，不同虚拟机在内存分配上有不同的地址空间，导致在不同虚拟机中访问同一个类对象会产生多份副本，运行在不同进程中的四大组件通过内存共享数据都失败
  - 线程同步机制完全失效：与上述类似
  - SharedPreferences 可靠性下降
    - SharedPreferences 不支持两个进程同时执行写操作，会导致一定几率的数据丢失，其底层通过读写 XML 文件实现
  - Application 多次创建
    - 当一个组件跑在一个新进程中时，由于系统在创建新进程时分配独立虚拟机，相当于启动一个应用的过程，会创建新的 Application，运行在同一进程中的组件属于同一虚拟机和同一 Application

#### 2.3、IPC 基础概念

- Serializable 接口：Java 提供的一个序列化接口，空接口，为对象提供标准的序列化和反序列化操作，只需在类声明中指定一个标识即可自动实现默认的序列化过程

  ```java
  private static final long serialVersionUID = 8711368828010083044L;
  
  //序列化过程
  User user = new User(0,"jake", true);
  Objectoutputstream out = new ObjectoutputStream(new FileOutputStream("cache.txt"));
  out.writeobject(user);
  out.close():
  //反序列化过程
  ObjectInputStream in = new ObjectInputStream(new FileInputStream("cache.txt"));
  User newUser = (User)in.readobject();
  in.close():
  ```

  - 实际上 serialVersionUID 不是必需的，不声明会对反序列化过程产生影响，serialVersionUID 用来辅助序列化和反序列化过程，原则上序列化后的数据的 serialVersionUID 只有和当前类 serialVersionUID 相同才能正常被反序列化

  - 工作机制：序列化时系统把当前类的 serialVersionUID 写入序列化文件，反序列化时系统会去检测文件中的 serialVersionUID，看它是否和当前类的 serialVersionUID 一致，一致说明序列化的类版本和当前类版本相同，这时可以成功反序列化，否则说明当前类和序列化类相比发生某些变换，比如成员变量数量、类型发生改变，这时无法正常反序列化

  - 重写系统默认序列化和反序列化过程

    ```java
    private void writeobject(java.io.objectoutputstream out) throws IOException{
        // write 'this' to 'out...
    }
    
    private void read0bject(java.io.0bjectInputStream in) throws IOException, ClassNotFoundException {
        // populate the fields of 'this' from the data in 'in'...
    }
    ```

- Parcelable 接口

  - Parcel 内部包装了可序列化的数据，可在 Binder 中自由传输。在序列化过程中需要实现序列化、反序列化、内容描述

    - 序列化功能由 writeToParcel 方法完成，最终通过 Parcel 中一系列 write 方法完成
    - 反序列化功能由 CREATOR 完成，其内部标明如何创建序列化对象和数组，并通过 Parcel 一系列 read 方法完成
    - 内容描述功能由 describeContents 方法完成，仅当当前对象中存在文件描述符时返回 1

  - 方法说明

    | 方法                                | 功能                                                         | 标记位                        |
    | ----------------------------------- | ------------------------------------------------------------ | ----------------------------- |
    | createFromParcel(Parcel in)         | 从序列化后的对象中创建原始对象                               |                               |
    | newArray(int size)                  | 创建指定长度的原始对象数组                                   |                               |
    | User(Parcel in)                     | 从序列化后的对象中创建原始对象                               |                               |
    | writeToParcel(Parcel out,int flags) | 将当前对象写入序列化结构中，flags 为1时标识当前对象需作为返回值返回，不能立即释放资源 | PARCELABLE_WRITE_RETURN_VALUE |
    | describeContents                    | 返回当前对象的内容描述，如果含有文件插述符返回1，否则返回0   | CONTENTS_FILE_DESCRIPTOR      |

  - 比较

    - Serializable 使用简单但开销很大，序列化和反序列化过程需要大量 I/O 操作
    - Parcelable 更适合用在 Android 平台上，缺点是使用起来稍微麻烦，但效率很高，Parcelable 主要用在内存序列化上
    - 通过 Parcelable 将对象序列化到存储设备中或将对象序列化后通过网络传输会稍显复杂，建议使用 Serializable

- Binder：Android 的一个类，实现 IBinder 接口

  > Android 中一种跨进程通信方式
  >
  > 一种虚拟物理设备，设备驱动是 /dev/binder，该通信方式在 Linux 中没有
  >
  > ServiceManager 连接各种 Manager (ActivityManager、WindowManager 等）和相应 ManagerService 的桥梁
  >
  > 客户端和服务端通信媒介，bindService 时服务端会返回一个包含服务端业务调用的 Binder 对象，通过该 Binder 对象，客户端可以获取服务端提供的服务或数据，包括普通服务和基于 AIDL 的服务

  - 主要用在 Service 中，包括 AIDL 和 Messenger，Messenger 底层是 AIDL

  - AIDL 为例：

    > 尽管 Book 类和 IBookManager 位于相同包下，IBookManager 还要导入 Book 类

    - Book.java，实现 Parcelable 接口

    - Book.aidl，Book 类在 AIDL 中的声明

    - IBookManager.aidl，定义接口，Book 相关方法

    - IBookManager.java，系统生成，继承 IInterface 接口，本身也是接口，所有可在 Binder 中传输的接口都需继承 IInterface 接口

      > 首先声明了在 IBookManager.aidl 所声明的方法，同时声明整型 id 分别标识该方法，用于标识在 transact 过程中客户端所请求的方法
      >
      > 接着声明内部类 Stub，是个 Binder 类，当客户端和服务端位于同一进程时，方法调用不会走跨进程的 transact 过程，当两者位于不同进程时方法调用要走 transact 过程，逻辑由 Stub 内部代理类 Proxy 完成

  - 方法说明

    - DESCRIPTOR：Binder 唯一标识，一般用当前 Binder 类名表示

    - asInterface(android.os.IBinder obj)：用于将服务端 Binder 对象转换成客户端所需的 AIDL 接口类型对象，转换过程区分进程，如果客户端和服务端位于同一进程，此方法返回服务端 Stub 对象本身，否则返回系统封装后的 Stub.proxy 对象

    - asBinder：用于返回当前 Binder 对象

    - onTransact：运行在服务端中的 Binder 线程池中，当客户端发起跨进程请求时，远程请求通过系统底层封装后交由此方法来处理

      >方法原型 `public Boolean onTransact(intcode, android.os.Parcel data, android.os.Parcel reply, int flags)`
      >
      >服务端通过 code 可确定客户端所请求的目标方法
      >
      >从 data 中取出目标方法所需参数（如果有的话)
      >
      >执行目标方法，执行完毕后向 reply 写入返回值（如果有的话)
      >
      >如果此方法返回 false，客户端请求失败，可以利用这个特性做权限验证

    - Proxy#getBookList（Book 相关方法）：运行在客户端，当客户端远程调用此方法时，内部实现：

      > 创建该方法所需输入型 Parcel 对象 \_data、输出型 Parcel 对象 \_reply、返回值对象 List
      >
      > 把该方法参数信息写入 \_data 中（如果有的话)
      >
      > 调用 transact 方法发起 RPC（远程过程调用）请求，同时当前线程挂起
      >
      > 服务端的 onTransact 方法会被调用，直到 RPC 过程返回后当前线程继续执行，并从 \_reply 中取出 RPC 过程的返回结果
      >
      > 返回 \_reply 数据

  - 死亡代理：Binder 运行在服务端进程，如果服务端进程由于某种原因异常终止，这时候到服务端的 Binder 连接断裂（Binder 死亡)，会导致远程调用失败。如果不知道 Binder 连接已经断裂，客户端的功能就会受到影响

    - linkToDeath、unlinkToDeath：设置死亡代理，当Binder死亡时会收到通知

      > 第二个参数是标记位，直接设 0 即可
      >
      > 通过 Binder 的 isBinderAlive 方法也可判断 Binder 是否死亡

      ```java
      private IBinder.DeathRecipient mDeathRecipient = new IBinder.DeathRecipient(){
          @override
          public void binderDied(){
              if(mBookManager == null)
                  return;
              mBookManager.asBinder().unlinkToDeath(mDeathRecipient,0);
              mBookManager = null;
              //重新绑定远程 Service
          }
      };
      
      //在客户端绑定远程服务成功后，给binder设置死亡代理
      mService = IMessageBoxManager.Stub.asInterface(binder);
      binder.linkToDeath(mDeathRecipient,0);
      ```

![安卓开发艺术探索_Binder](Image.assets\安卓开发艺术探索_Binder.png)

#### 2.4、Android IPC方式

- Bundle：四大组件中三大组件（Activity、Service、Receiver）都支持在 Intent 中传递 Bundle 数据的，Bundle 实现 Parcelable 接口，可以方便地在不同进程间传输，传输的数据必须能够被序列化，如基本类型、实现 Serializable、Parcellable 接口的对象、一些 Android 支持的特殊对象

  - 如 A 进程正进行一个计算，计算完成后要启动 B 进程一个组件并把结果传给 B 进程，可是计算结果不支持放入 Bundle，无法通过 Intent 传输，可通过 Intent 启动进程 B 的一个 Service 组件（如 IntentService），让 Service 在后台计算，计算后再启动 B 进程中真正要启动的目标组件，由于Service也运行在 B 进程中，所以目标组件就可直接获取计算结果，避免进程间通信问题

- 文件共享：两个进程通过读/写同个文件来交换数据，如 A 进程把数据写入文件，B 进程通过读取文件来获取数据

  - 在 Windows 上，一个文件如果被加了排斥锁会导致其他线程无法对其进行访问，包括读写

  - Android 系统基于 Linux，其并发读/写文件可以没有限制地进行，甚至两个线程同时对同一文件进行写操作都是允许的，尽管这可能出问题

  - 通过文件共享对文件格式没有具体要求，如文本文件、XML 文件，只要读/写双方约定数据格式即可

  - 局限性：如并发读/写问题，读出的内容可能不是最新的，文件共享方式适合在对数据同步要求不高的进程之间进行通信，且要处理并发读/写问题

  - SharedPreferences：Android 提供的轻量级存储方案，通过键值对方式存储数据，底层实现采用 XML 文件存储键值对，每个应用的 SharedPreferences 文件都可在当前包所在的 data 目录下查到

    > 一般其目录位于/data/data/package name/shared_prefs 目录下，package name 表示当前应用包名
    >
    > 本质上属于文件的一种，但由于系统对其读/写有一定缓存策略（在内存中有一份 SharedPreferences 文件缓存），在多进程下系统对它的读/写变得不可靠，有很大几率丢失数据

- Messager：可在不同进程中传递 Message 对象，在 Message 中放入需要传递的数据。一种轻量级 IPC 方案，底层实现是 AIDL，对 AIDL 做了封装，一次处理一个请求， 在服务端不用考虑线程同步问题（不存在并发执行情形）

  ```java
  //构造方法
  public Messenger(Handler target){
      mTarget = target.getIMessenger();
  }
  public Messenger(IBinder target){
      mTarget = IMessenger.Stub.asInterface(target);
  }
  ```

  - 服务端进程：	
    - 在服务端创建一个 Service 来处理客户端的连接请求
    - 创建一个 Handler 并通过其创建一个 Messenger 对象
    - 在 Service 的 onBind 中返回该 Messenger对象底层的 Binder 即可
  - 客户端进程
    - 绑定服务端的 Service
    - 用服务端返回的 IBinder 对象创建一个 Messenger，通过该 Messenger 可向服务端发送消息，消息类型为 Message 对象
    - 如果服务端能够回应客户端，还需要创建一个 Handler 并创建一个新的 Messenger，并把其通过 Messager 的 replyTo 参数传递给服务端，服务端通过 replyTo 参数可以回应客户端

  ![安卓开发艺术探索_Messager](Image.assets\安卓开发艺术探索_Messager.jpg)

- AIDL：

  - 服务端

    - 创建一个 Service 用来监听客户端的连接请求
    - 创建一个 AIDL 文件，将暴露给客户端的接口在该文件中声明
    - 在 Service 中实现该 AIDL 接口

  - 客户端

    - 绑定服务端的 Service
    - 将服务端返回的 Binder 对象转成 AIDL 接口所属类型，就可调用 AIDL 中的方法了

  - AIDL 支持数据类型

    > 自定义 Parcelable 对象和 AIDL 对象都要显示 import
    >
    > 如果 AIDL 文件中用到自定义 Parcelable 对象，必须新建一个和它同名的 AIDL 文件，并在其中声明它为 Parcelable 类型
    >
    > AIDL 中除基本数据类型，其他类型参数必须标上方向：in（输入型参数）、out（输出型参数）、inout（输入输出型参数），不能一概使用 inout，因为在底层实现有开销
    >
    > AIDL 接口只支持方法，不支持声明静态常量，区别于传统接口

    - 基本数据类型（int、long、char、boolean、double 等）
    - String、CharSequence
    - List：只支持 ArrayList，每个元素都必须被 AIDL 支持（在服务端可以使用 CopyOnWriteArrayList（继承自 List），Binder 中会按照 List 接口规范访问数据并生成 ArrayList 传给客户端）
    - Map：只支持 HashMap，每个元素都必须被 AIDL 支持，包括 key 和 value
    - Parcelable：所有实现 Parcelable 接口的对象
    - AIDL：所有 AIDL 接口本身也可在 AIDL 文件中使用

  - RemoteCallbackList：系统提供用于删除跨进程 listener 的接口，一个泛型，支持管理任意 AIDL 接口，因为所有 AIDL 接口都继承自 IInterface 接口

    - Binder 会把客户端传递过来的对象重新转化并生成一个新对象，对象跨进程传输本质上是反序列化过程，无法传递 listener 对象解注册，但底层的 Binder 对象是同一个

    - `public class RemoteCallbackList<E extends IInterface>` 工作原理：

      > 内部有一个 Map 结构专门用来保存所有 AIDL 回调，Map 的 key 是 IBinder 类型，value 是 Callback 类型，`ArrayMap<IBinder, callback>`
      >
      > Callback 中封装了真正的远程 listener，当客户端注册 listener 时会把这个 listener 信息存入 mCallbacks 中，key 和 value 的获得：
      >
      > `IBinder key = listener.asBinder () ` 
      >
      > `Callback value = new Callback(listener, cookie)` 

    - RemoteCallbackList 当客户端进程终止能自动移除客户端所注册的 listener

    - RemoteCallbackList 内部自动实现线程同步

    - 使用：

      ```java
      //创建一个 RemoteCallbackList 对象替代 CopyOnWriteArrayList
      private RemoteCallbackList<IOnNewBookArrivedListener> mListenerList = new
      RemoteCallbackList<IOnNewBookArrivedListener>();
      //修改registerListener和unregisterListener两接口实现
      @Override
      public void registerListener(IOnNewBookArrivedListener listener) throws RemoteException{
          mListenerList.register(listener);
      }
      @Override
      public void unregisterListener (IOnNewBookArrivedListener listener) throws RemoteException {
          mListenerList.unregister(listener);
      }
      ```

    - 遍历

      ```java
      //1 beginBroadcast和finishBroadcast必须配对使用
      final int N = mListenerList.beginBroadcast();
      for (int i=0;i<N;i++){
          IOnNewBookArrivedListener l = mListenerList.getBroadcastItem(i);
          if (1 != null){
              //TODO handle 1
          }
      }
      mListenerList.finishBroadcast();
      ```

  - 权限验证：

    - 在 onBind 中进行验证，验证不通过直接返回 null，验证失败的客户端直接无法绑定服务，验证方式有多种，如使用 permission 验证，要先在 AndroidMenifest 中声明所需的权限，如

      ```xml
      <permission
                  android:name="com.ryg.hzy.permission.ACCESS_BOOK_SERVICE"
                  android:protectionLevel="normal"/>
      ```

      然后在 BookManagerService 的 onBind 方法中做权限验证，如

      ```java
      public IBinder onBind(Intent intent){
          int check = checkCallingOrSelfPermission("com.ryg.hzy.permission.ACCESS_BOOK_SERVICE");
          if (check == PackageManager.PERMISSION DENIED){
              return null;
          }
          return mBinder;
      }
      ```

      这种方法同样适用于 Messenger 中

    - 在服务端的 onTransact 方法中进行权限验证，验证失则直接返回 false，这样服务端不会终止执行 AIDL 中的方法从而达到保护服务端的效果，验证方式很多，如 permission 验证、Uid、Pid 验证，通过 getCallingUid 和 getCallingPid 可拿到客户端所属应用 Uid 和 Pid，然后可以验证包名等

      ```java
      public boolean onTransact(int code,Parcel data,Parcel reply,int flags) throws RemoteException{
          int check = checkCallingOrSelfPermission("com.ryg.hzy.permission.ACCESSBOOK_SERVICE");
          if(check == PackageManager.PERMISSION_DENIED){
              return false;
          }
          String packageName = null ;
          String[] packages = getPackageManager().getPackagesForUid(getCallingUid());
          if(packages != null && packages.length > 0){
              packageName = packages[0];
          }
          if(!packageName.startswith("com.ryg")){
              return false;
          }
          return super.onTransact(code,data,reply,flags);
      }
      ```

      > 一个应用如果想远程调用服务中的方法，首先要使用自定义权限 `com.ryg.hzy.permission.ACCESS_BOOK_SERVICE`，其次包名必须以 com.ryg 开始，否则调用服务端方法会失败

- ContentProvider：Android 提供的专门用于不同应用间数据共享方式，底层实现是 Binder，使用过程比 AIDL 简单，因为系统做了封装，系统预置许多 ContentProvider，如通讯录信息、日程表信息等

  - 创建自定义 ContentProvider：继承 ContentProvider 类并实现六个抽象方法：

    - onCreate：ContentProvider 的创建，做些初始化工作

    - getType：返回一个 Uri 请求对应的 MIME 类型（媒体类型），如图片、视频等，如果不关注该选项，可直接返回 null 或 “\*/\*”

    - query、update、insert、delete：对应 CRUD 操作，和 query 不同，update、insert、delete 会引起数据源改变，这时要通过 ContentResolver 的 notifyChange 方法通知外界当前 ContentProvider 数据发生改变。要观察一个 ContentProvider 的数据改变情况，可通过 ContentResolver 的 registerContentObserver 方法注册观察者，通过 unregisterContentObserver 方法解除观察者。四个方法存在多线程并发访问，内部要做好线程同步（SQLite 内部已处理同步）

    - 六个方法均运行在 ContentProvider 进程中，除 onCreate 由系统回调并运行在主线程，其他五个方法均由外界回调并运行在 Binder 线程池中

    - 注册

      > android:authorities：ContentProvider 唯一标识，通过该属性外部应用可访问 BookProvider，必须唯一，建议命名时加上包名前缀
      >
      > 添加权限，外界应用想访问 BookProvider 必须声明权限，权限可细分为读、写权限，分别对应 android:readPermission 和 android:writePermission 属性

      ```xml
      <provider
                android:name=".provider.BookProvider"
                android:authorities="com.ryg.hzy.provider"
                android:permission="com.ryg.PROVIDER"
                android:process=":provider">
      </provider>
      ```

    - ContentProvider 通过 Uri 区分外界访问的数据集合，要为不同的表定义单独的 Uri 和 Uri_Code，并将 Uri 和对应 Uri_Code 相关联，可使用 UriMatcher 的 addURI 方法

  - 主要以表格形式组织数据，且可包含多个表，每个表都具有行列的层次性，行对应一条记录，列对应一条记录中的一个字段，和数据库类似

    - ContentProvider 还支持文件数据，如图片、视频等，和表格数据结构不同，处理时可在 ContentProvider 中返回文件句柄给外界，让文件访问 ContentProvider 的文件信息，Android 系统提供的 MediaStore 功能是文件类型的 ContentProvider

  - ContentProvider 对底层数据存储方式没有要求，可使用 SQLite 数据库、普通文件、内存中对象

- Socket：

  - 声明权限

    ```xml
    <uses-permission android:name ="android.permission.INTERNET"/>
    <uses-permission android:name="android.permission.ACCESS_NETWORK_STATE"/>
    ```
  
- IPC 方式及适用场景

  | 名称            | 优点                                                         | 缺点                                                         | 适用场景                                                   |
  | --------------- | ------------------------------------------------------------ | ------------------------------------------------------------ | ---------------------------------------------------------- |
  | Bundle          | 简单易用                                                     | 只能传输Bundle支持的数据类型                                 | 四大组件间的进程间通信                                     |
  | 文件共享        | 简单易用                                                     | 不适合高并发场景，无法做到进程间即时通信                     | 无并发访问，交换简单数据，实时性不高                       |
  | AIDL            | 功能强大，支持一对多并发通信、实时通信                       | 使用稍复杂，需处理线程同步                                   | 一对多通信，有RPC需求                                      |
  | Messenger       | 功能一般，支持一对多串行通信、实时通信                       | 不能很好处理高并发情形，不支持 RPC，数据通过Message进行传输，只能传输Bundle支持的数据类型 | 低并发的一对多即时通信，无RPC需求，或无须返回结果的RPC需求 |
  | ContentProvider | 在数据源访问方面功能强大，支持一对多并发数据共享，可通过Call方法扩展其他操作 | 像受约束AIDL，主要提供数据源的CRUD操作                       | 一对多的进程间数据共享                                     |
  | Socket          | 功能强大，可通过网络传输字节流，支持一对多并发实时通信       | 实现稍烦琐，不支持直接RPC                                    | 网络数据交换                                               |

#### 2.5、Binder 连接池

- 多个模块需要 AIDL：每个业务模块创建自己的 AIDL 接口并实现此接口，不同模块间不能耦合，向服务端提供唯一标识和对应 Binder 对象，服务端只需一个 Service，提供一个 queryBinder 接口，根据模块特征返回相
  应 Binder 对象，不同模块拿到所需 Binder 对象后就可进行远程方法调用
- Binder 连接池作用：将每个模块的 Binder 请求统一转发到远程 Service 中执行，从而避免重复创建 Service

![安卓开发艺术探索_Binder连接池](Image.assets\安卓开发艺术探索_Binder连接池.png)

### 3、View 事件体系

#### 3.1、View 基础

> View：Android 所有控件的基类，界面层控件一种抽象，代表一个控件
>
> ViewGroup：控件组，继承自 View

- View 位置参数
  
  - View 的位置由其四个顶点决定，对应四个属性：top（左上角纵坐标）、left（左上角横坐标）、right（右下角横坐标）、bottom（右下角纵坐标），View 提供了对应 get/set 方法（getLeft() ）
  
  ![安卓开发艺术探索_View位置](Image.assets\安卓开发艺术探索_View位置.jpg)
  
  - Android 3.0 View 增加 x、y、translationX、translationY，x、y 是 View 左上角坐标，translationX、translationY 是 View 左上角相对于父容器偏移量。都相对于父容器坐标，translationX、translationY 默认值 0，View 也提供了 get/set 方法，换算关系：
    - x = left + translationX
    - y = top + translationY
  - View 平移时，top 和 left 表示原始左上角位置信息，其值不变，改变的 x、y、translationX、translationY
  
- MotionEvent

  - ACTION_DOWN：手指刚接触屏幕
  - ACTION_MOVE：手指在屏幕上移动
  - ACTION_UP：手机从屏幕上松开的一瞬间
  - 正常一次手指触摸屏幕行为会触发一系列点击事件
    - 点击屏幕后离开，事件序列 DOWN - UP
    - 点击屏幕滑动一会再松开，事件序列 DOWN - MOVE -  ... - MOVE - UP
  - 通过 MotionEvent 对象可得到点击事件发生的 x、y 坐标，系统提供两组方法 getX/getY、getRawX/getRawY，区别
    - getX/getY：返回相对当前 View 左上角的 x、y 坐标
    - getRawX/getRawY：返回相对于手机屏幕左上角的 x、y 坐标

- TouchSlop：系统认为滑动的最小距离，当手指在屏幕上滑动时，如果两次滑动间距离小于该常量，系统不认为在滑动操作，常量和设备有关，获取 `ViewConfiguration.get(getContext()).getScaledTouchSlop()` 

  - 作用：处理滑动时，可利用其做过滤，当两次滑动事件的滑动距离小于该值，可认为未达到滑动距离临界值，不是滑动，源码定义在 frameworks/base/core/res/res/values/config.xml 文件中，对应 config_viewConfigurationTouchSlop

- VelocityTracker：速度追踪，追踪手指滑动速度，包括水平、竖直方向的速度，使用：

  - 在 View 的 onTouchEvent 方法中追踪当前单击事件的速度：

    ```java
    velocityTracker velocityTracker = VelocityTracker.obtain();
    velocityTracker.addMovement(event);
    ```

  - 先知道当前滑动速度时，获得当前速度：

    - 获取速度前先计算速度，即 getXVelocity、getYVelocity 前要调用 computeCurrentVelocity，参数表示一个时间单元或时间间隔
    - 速度：一段时间内手指所滑过的像素数，如时间间隔设为 1000ms，1s 内手指在水平方向从左向右滑过 100 像素，水平速度是 100，可为负
    - 速度 = (终点位置 - 起点位置) / 时间段

    ```java
    velocityTracker.computeCurrentVelocity(1000);
    int xVelocity = (int)velocityTracker.getxvelocity();
    int yvelocity = (int)velocityTracker.getYvelocity();
    ```

  - 回收：调用 clear 方法重置并回收内存

    ```java
    velocityTracker.clear();
    velocityTracker.recycle();
    ```

- GestureDetector：手势检测，辅助检测用户单击、滑动、长按、双击等行为，使用：

  - 创建一个 GestureDetector 对象并实现 OnGestureListener 接口，还可实现 OnDoubleTapListener 监听双击行为

    ```java
    //解决长按屏幕后无法拖动的现象
    GestureDetector mGestureDetector = new GestureDetector(this);
    mGestureDetector.setIsLongpressEnabled (false);
    ```

  - 接管目标 View 的 onTouchEvent 方法，在待监听 View 的 onTouchEvent 方法中添加：

    ```java
    boolean consume = mGestureDetector.onTouchEvent(event);
    return consume;
    ```

  - 接口方法

    - OnGestureListener

    | 方法名        | 描述                                                         |
    | ------------- | ------------------------------------------------------------ |
    | onDown        | 手指轻触屏幕一瞬间，由1个ACTION_DOWN触发                     |
    | onShowPress   | 手指轻触屏幕，未松开或拖动，由1个ACTION_DOWN触发             |
    | onSingleTapUp | 手指(轻触屏幕后)松开，伴随1个ACTION_UP触发，单击行为         |
    | onScroll      | 手指按下屏幕并拖动，由1个ACTION_DOWN、多个ACTION_MOVE触发，拖动行为 |
    | onLongPress   | 用户长久按着屏幕不放，长按                                   |
    | onFling       | 用户按下触摸屏、快速滑动后松开，由1个ACTION_DOWN、多个ACTION_MOVE、1个ACTION_UP触发，快速滑动行为 |

    - OnDoubleTapListener


    | 方法名               | 描述                                                         |
    | -------------------- | ------------------------------------------------------------ |
    | onDoubleTap          | 双击，由2次连续单击组成，不能和onSingleTapConfirmed共存      |
    | onSingleTapConfirmed | 严格单击行为，如果触发onSingleTapConfirmed，后面不能再跟着另一个单击行为，即只能是单击，不能是双击中的一次单击 |
    | onDoubleTapEvent     | 发生双击行为，双击期间，ACTION_DOWN、ACTION_MOVE、ACTION_UP都会触发此回调 |

- Scroller：弹性滑动对象，用于实现 View 的弹性滑动，使用 View 的 scrollTo、scrollBy 方法进行滑动时是瞬间完成的，没有过渡效果。Scroller 实现有过渡效果，其本身无法让 View 弹性滑动，需要和 View 的 computeScroll 方法配合使用，使用：

  ```java
  scroller scroller = new Scroller(mContext);
  //缓慢滚动到指定位置
  private void smoothScrollTo(int destx,int destY){
      int scrollX = getScrollX();
      int delta = destX - scro1lX;
      //1000ms内慢慢滑向destX
      mScroller.startScroll(scrollx, 0, delta, 0, 1000);
      invalidate();
  }
  
  @Override
  public void computescroll(){
      if(mScroller.computeScrolloffset()){
          scrollTo(mScroller.getCurrx(), mScroller.getCurrY());
          postInvalidate();
      }
  }
  ```

#### 3.2、View 的滑动

- 使用 scrollTo、scrollBy

  - scrollBy 实际也是调用 scrollTo 方法，基于当前位置的相对滑动，scrollITo 基于所传递参数的绝对滑动
  - mScrollX、mScrollY 可通过 getScrollX、getScrollY 方法得到
  - 滑动中，mScrollX 的值总等于 View 左边缘和 View 内容左边缘在水平方向的距离，mScrollY 的值总等于 View 上边缘和 View 内容上边缘在竖直方向的距离，View 边缘指 View 位置，由四个顶点组成，View 内容边缘指 View 中内容边缘
  - scrollTo、scrollBy 只能改变 View 内容位置，不能改变 View 在布局中的位置
  - mScrollX、mScrollY 单位为像素
  - 当 View 左边缘在 View 内容左边缘的右边时，mScrollX 为正值，反之为负值；当 View 上边缘在 View 内容上边缘的下边时，mScrollY 为正值，反之为负值（从左向右滑动，mScrollX 为负值，反之为正值；从上往下滑动，mScrollY 为负值，反之为正值）

  ```java
  public void scro1To(int x，int y) {
      if(mScrollX != x || mScrolly != y){
          int oldX = mScrollX;
          int oldY = mScrollY;
          mScrollX = X;
          mScrolly = y;
          invalidateParentCaches();
          onScrollChanged(mScrol1X, mScrollY, oldx, oldY);
          if(!awakenScrollBars()){
              postInvalidateOnAnimation();
          }
      }
  }
  public void scrollBy(int x, int y) {
      scrollTo(mScrollX + x， mScrollY + y);
  }
  ```

  ![安卓开发艺术探索_Scroll](Image.assets\安卓开发艺术探索_Scroll.jpg)

- 动画：主要操作 View 的 translationX、translationY 属性，可采用 View 动画、属性动画，属性动画为了兼容 3.0 以下版本，需采用开源动画库 nineoldandroids

  - View 动画：100ms 内将一个 View 从原始位置向右下角移动 100 像素

    ```xml
    <?xml version="1.0" encoding="utf-8"?>
    <set xm1ns: android="http://schemas.android.com/apk/res/android"
         android:fillAfter="true"
         android:zAdjustment="normal">
        <translate
                   android:duration="100"
                   android:fromXDelta="0"
                   android:fromYDelta="0"
                   android:interpolator="@android:anim/linear_interpolator"
                   android:toXDelta="100"
                   android:toYDelta="100"/>
    </set>
    ```

  - 属性动画：100ms 内将一个 View 从原始位置向右平移 100 像素

    ```xml
    ObjectAnimator.ofFloat(targetView,"translationX",0,100).setDuration(100).start();
    ```

  - View 动画对 View 影像操作，不能改变 View 的位置参数，如宽、高，如果希望动画后状态得以保留，要将 fillAfter 属性设为 true，否则动画完成后动画结果消失。如把 View 向右移动 100 像素，如果 fillAfter 为 false，在动画完成时 View 瞬间恢复到动画前状态
  - 属性动画不存在上述问题，但 Android 3.0 以下通过 nineoldandroids 来兼容的属性动画本质上是 View 动画
  - View 动画后新位置无法响应点击事件
    - 属性动画解决，但 Android 3.0 以下无法使用属性动画
    - 在新位置创建一个和目标 Button 一样的 Button，外观、onClick 事件一样，当目标 Button 完成平移动画后把目标 Button 隐藏，把创建的 Button 显示出来

- 改变布局参数：改变 LayoutParams

  - 如把一个控件右移 100px，将该控件的 LayoutParams 的 marginLeft 参数值增加 100px 即可

  - 可在控件左边放置一个空 View，默认宽度为 0，向右移动控件时，重新设置空 View 宽度即可，空 View 宽度增大时，控件自动被挤向右边

  - 重新设置一个 View 的 LayoutParams：

    ```java
    MarginLayoutParams params = (MarginLayoutParams)mButton.getLayoutParams();
    params.width += 100;
    params.leftMargin += 100;
    mButton.requestLayout();
    //或mButtonl.setLayoutParams(params);
    ```

- 三种滑动比较
  - scrollTo、scrollBy：View 提供原生方法，适合 View 内容滑动
    - 优点：实现方便，不影响内部元素的单击事件
    - 缺点：只能滑动 View 内容，不能滑动 View 本身
  - 动画：Android 3.0 以上采用属性动画，没明显缺点；View 动画或在 Android 3.0 以下均不能改变 View 本身属性。如果动画元素不需响应用户的交互，使用动画合适
    - 优点：一些复杂效果必须通过动画才能实现
  - 改变布局：
    - 缺点：使用麻烦
    - 适用对象：具有交互性 View

#### 3.3、弹性滑动

- Scroller：

  - Scrollr 本身不能实现 View 滑动，需配合 View 的 computeScroll 方法，不断让 View 重绘，每次重绘距滑动起始时间有个时间间隔，通过该间隔 Scroller 就可得出 View 当前滑动位置，就可通过 scolTo 方法完
    成 View 滑动，View 每次重绘都导致 View 小幅度滑动，多次小幅度滑动组成弹性滑动

  ```java
  Scroller scroller = new Scroller(mContext);
  //缓慢滚动到指定位置
  private void smoothScrollTo(int destX, int destY){
      int scrol1X = getScrollX();
      int deltaX = destX - scrol1X;
      //1000ms内慢慢滑向destX
      mScroller.startScroll(scrol1X, 0, deltaX, 0, 1000);
      invalidate();//导致View重绘，View的draw方法调用computeScroll（View中空实现）
  }
  
  @Override
  public void computeScroll(){
      if(mScroller.computeScrolloffset()){
          scrollTo(mScroller.getCurrx(), mScroller.getCurrY());
          postInvalidate();//二次重绘，导致反复调用computeScroll
      }
  }
  //构造一个Scroller对象并调用startScroll方法时，内部只是保存传递的几个参数
  public void startScroll(int startX, int startY, int dx, int dy, int duration){
      mMode = SCROLL_MODE;
      mFinished = false;
      mDuration = duration;
      mStartTime = AnimationUtils.currentAnimationTimeMillis();
      mStartX = startX;//滑动起点
      mStartY = startY;//滑动起点
      mFinalX = startX + dx;
      mFinalY = startY + dy;
      mDeltaX = dx;
      mDeltaY = dy;
      mDurationReciprocal = 1.0f / (float)mDuration;
  }
  //根据时间流逝百分比计算出当前scrollX、scrollY改变的百分比并计算出当前的值，类似动画中的插值器
  //返回值true表示滑动还未结束，false表示滑动结束
  public boolean computeScrolloffset() {
      int timePassed = (int)(AnimationUtils.currentAnimationTimeMillis() - mStartTime);
      if (timePassed < mDuration) {
          switch(mMode){
              case SCROLL_MODE:
                  final float x = mInterpolator.getInterpolation(timePassed * mDurationReciprocal);
                  mCurrX = mStartX + Math.round(x * mDeltaX);
                  mCurrY = mStartY + Math.round(x * mDeltaY);
                  break;
                  //...
          }
      }
      return true;
  }
  ```

- 动画

  - 动画本质上没有作用于任何对象上，只在 1000ms 内完成整个动画过程，可在动画每一帧到来时获取动画完成比例，再根据该比例计算出当前 View 要滑动的距离，针对 View 的内容而非 View 本身，思想和 Scroller 类似，都通过改变一个百分比配合 scrollTo 方法完成 View 滑动

  ```java
  final int startx = 0;
  final int deltax = 100;
  ValueAnimator animator = ValueAnimator.ofInt(0, 1).setDuration(1000);
  animator.addUpdateListener(new AnimatorUpdateListener(){
      @Override
      public void onAnimationUpdate(ValueAnimator animator){
          float fraction = animator.getAnimatedFraction();
          mButton.scrollTo(startx + (int)(deltaX * fraction), 0);
      }
  });
  animator.start();
  ```

- 延时策略：思想是通过发送一系列延时消息达到渐近式效果，可使用 Handler、View 的 postDelayed 方法、sleep 方法，postDelayed 可以延时发送一个消息，在消息中进行 View 滑动，不断发送延时消息就可实现弹性滑动效果，sleep 在 while 循环中不断滑动 View 和 sleep 就可实现弹性滑动效果

#### 3.4、View 事件分发机制

- 点击事件传递规则

  - `public boolean dispatchTouchEvent(MotionEvent ev)`：进行事件分发，如果事件能够传递给当前 View，此方法会被调用，返回结果受当前 View 的 onTouchEvent 和下级 View 的 dispatchTouchEvent 方法的影响，表示是否消耗当前事件

  - `public boolean onInterceptTouchEvent(MotionEvent event)`：在 dispatchTouchEvent 方法内部调用，判断是否拦截某事件，如当前 View 拦截某事件，在同一事件序列中此方法不会被再次调用，返回结果表示是否拦截当前事件

  - `public boolean onTouchEvent(MotionEvent event)`：在 dispatchTouchEvent 方法中调用，处理点击事件，返回结果表示是否消耗当前事件，如不消耗，在同一事件序列中当前 View 无法再次接收到事件

  - 三方法关系伪代码：

    ```java
    public boolean dispatchTouchEvent(MotionEvent ev){
        boolean consume = false;
        if(onInterceptTouchEvent (ev)){
            consume = onTouchEvent(ev);
        }else{
            consume = child.dispatchTouchEvent(ev);
        }
        return consume;
    }
    ```

  - 点击事件传递规则：

    - 对一个根 ViewGroup，点击事件产生后先会传给自己，自己的 dispatchTouchEvent 被调用，如果其 onInterceptTouchEvent 返回 true，拦截当前事件，事件交给其处理，其 onTouchEvent 被调用；如果其 onInterceptTouchEvent 返回 false，不拦截当前事件，当前事件会继续传递给其子元素，子元素的 dispatchTouchEvent 被调用，反复直到事件被最终处理
    - 当一个 View 需处理事件时，如果设置了 OnTouchListener，OnTouchListener 的 onTouch 会被回调，onTouch 返回 false，当前 View 的 onTouchEvent 被调用；返回 true，onTouchEvent 不被调用，OnTouchListener 优先级比 onTouchEvent 高，onTouchEvent 方法中如果当前设有 OnClickListener，其 onClick 会被调用，OnClickListener 优先级最低
    - 点击事件传递顺序：Activity - Window - View，顶级 View 收到事件后按事件分发机制分发事件，如果一个 View 的 onTouchEvent 返回 false，其父容器的 onTouchEvent 将被调用，如果所有元素都不处理该事件，事件会传给 Activity 处理，Activity 的 onTouchEvent 方法被调用

  - 事件传递结论：

    - 同一事件序列：手指接触屏幕那一刻起，到离开屏幕那一刻结束，这过程所产生的一系列事件，以 down 事件开始，中间含数量不定 move 事件，以 up 事件结束
    - 正常一个事件序列只能被一个 View 拦截消耗，一个元素拦截某事件，同一事件序列所有事件会直接交给其处理，通过特殊手段，如一个 View 将本该自已处理的事件通过 onTouchEvent 强行传递给其他 View 处理可实现同一事件序列事件由两个 View 同时处理
    - 某个 View 决定拦截，这一事件序列只能由它其处理（事件序列能够传递给其的话)，且其 onterceptTouchEvent 不被调用
    - 某个 View 开始处理事件，如果不消耗 ACTION _DOWN 事件（onTouchEvent 返回 false），同一事件序列其他事件都不会交给其处理，且事件重新交由其父元素处理，父元素的 onTouchEvent 被调用
    - 如果 View 不消耗除 ACTION _DOWN 以外其他事件，该点击事件会消失，父元素的 onTouchEvent  不被调用，且当前 View 可持续收到后续事件，消失的点击事件传给 Activity 处理
    - ViewGroup 默认不拦截任何事件，Android 源码中 ViewGroup 的 onInterceptTouchEvent 默认返回 false
    - View 没有 onInterceptTouchEvent 方法，有点击事件传给它，其 onTouchEvent 被调用
    - View 的 onTouchEvent 默认消耗事件（返回 true），除非其不可点击（clickable 和 longClickable 同为 false），View 的 longClickable 属性默认为 false，Button 的 clickable 属性默认为 true，TextView 的 clickable 属性默认为 false
    - View 的 enable 属性不影响 onTouchEvent 默认返回值，哪怕其为 disable 状态，只要其 clickable 或 longClickable 有一个为 true，其 onTouchEvent 返回 true
    - onClick 发生前提：当前 View 可点击且其收到 down 和 up 事件
    - 事件传递由外向内的，即事件先传给父元素，再由父元素分发给子 View，通过 requestDisallowInterceptTouchEvent 方法可在子元素中干预父元素事件分发过程，但ACTION_DOWN 事件除外

- 事件分发源码解析

  - Activity 对点击事件分发过程

    - 事件最先传给当前 Activity，由 Activity 的 dispatchTouchEvent 进行事件派发，具体工作由 Activity  内部 Window 完成
    - Window 将事件传给 decor view，decor view 一般是当前界面的底层容器（setContentView 设置的 View 的父容器)，通过 `Activity.getWindow.getDecorView()` 可获得
    - 返回 true，整个事件循环结束，返回 false 意味事件没人处理，所有 View 的 onTouchEvent 都返回 false，Activity 的 onTouchEvent 会被调用
    - Window：抽象类，其 superDispatchTouchEvent 也是抽象方法
    - Window 唯一实现是 PhoneWindow
    -  Activity#dispatchTouchEvent

    ```java
    public boolean dispatchTouchEvent(MotionEvent ev){
        if (ev.getAction() == MotionEvent.ACTION_DOWN) {
            onUserInteraction();
        }
        if (getwindow().superDispatchTouchEvent(ev)) {
            return true;
        }
        return onTouchEvent(ev);
    }
    ```

    - PhoneWindow#superDispatchTouchEvent

    ```java
    //PhoneWindow将事件直接传给DecorView
    public boolean superDispatchTouchEvent (MotionEvent event) {
        return mDecor.superDispatchTouchEvent(event);
    }
    ```

    - DecorView

    ```java
    private final class DecorView extends FrameLayout implements RootViewSurfaceTaker{
        private DecorView mDecor;
        @Override
        public final View getDecorView(){
            if(mDecor == null){
                installDecor();
            }
            return mDecor;
        }
    }
    ```

    - 通过 `((ViewGroup)getWindow().getDecorView().findViewById(android.R.id.content).getChildAt(0)` 可获取 Activity 设置的 View，mDecor 是 `getWindow().getDecorView())` 返回的 View，通过 setContentView 设置的 View 是其一个子 View
    - DecorView 继承自 FrameLayout 且是父 View，事件会传给 View（顶级 View，Activity 中通过
      setContentView 设置的 View）

  - 顶级 View 对点击事件分发过程

    - 