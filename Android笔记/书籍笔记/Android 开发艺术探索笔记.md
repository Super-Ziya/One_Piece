## Android 开发艺术探索

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
            android:name="com.ryg.chapter.SecondActivity"
            android:configChanges="screenLayout"
            android:launchMode="singleTask"
            android:label="@string/app_name"/>
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
          android:path="string"
          android:pathPattern="string"
          android: pathPrefix="string"
          android:mimeType="string"/>
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
  - 通过 JNI 在 native 层 fork 一个新进程（特殊情况）

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
  User user = new User(0, "jake", true);
  Objectoutputstream out = new ObjectoutputStream(new FileOutputStream("cache.txt"));
  out.writeobject(user);
  out.close();
  //反序列化过程
  ObjectInputStream in = new ObjectInputStream(new FileInputStream("cache.txt"));
  User newUser = (User)in.readobject();
  in.close();
  ```

  - serialVersionUID 不必需，辅助序列化和反序列化过程，原则上序列化后的数据的 serialVersionUID 只有和当前类 serialVersionUID 相同才能正常被反序列化

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

  ![安卓开发艺术探索_Messager](..\Image.assets\安卓开发艺术探索_Messager.jpg)

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
      //beginBroadcast和finishBroadcast必须配对使用
      final int N = mListenerList.beginBroadcast();
      for (int i=0;i<N;i++){
          IOnNewBookArrivedListener l = mListenerList.getBroadcastItem(i);
          if (l != null){
              //TODO handle l
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
          if (check == PackageManager.PERMISSION_DENIED){
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
          return super.onTransact(code, data, reply, flags);
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
    <uses-permission android:name="android.permission.INTERNET"/>
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
      int delta = destX - scrollX;
      //1000ms内慢慢滑向destX
      mScroller.startScroll(scrollX, 0, delta, 0, 1000);
      invalidate();
  }
  
  @Override
  public void computescroll(){
      if(mScroller.computeScrolloffset()){
          scrollTo(mScroller.getCurrX(), mScroller.getCurrY());
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
    - 某个 View 决定拦截，这一事件序列只能由它其处理（事件序列能够传递给其的话)，且其 onInterceptTouchEvent 不被调用
    - 某个 View 开始处理事件，如果不消耗 ACTION_DOWN 事件（onTouchEvent 返回 false），同一事件序列其他事件都不会交给其处理，且事件重新交由其父元素处理，父元素的 onTouchEvent 被调用
    - 如果 View 不消耗除 ACTION_DOWN 以外其他事件，该点击事件会消失，父元素的 onTouchEvent  不被调用，且当前 View 可持续收到后续事件，消失的点击事件传给 Activity 处理
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

    - 点击事件达到顶级 View（一般是个 ViewGroup）后调用 ViewGroup 的 dispatchTouchEvent

    - 如果顶级 ViewGroup 拦截事件（onInterceptTouchEvent 返回 true），事件由 ViewGroup 处理

      > 如果 ViewGroup 的 mOnTouchListener 被设置，onTouch 被调用，否则 onTouchEvent 被调用，都提供时 onTouch 会屏蔽掉 onTouchEvent，onTouchEvent 如果设置 mOnClickListener，onClick 被调用

    - 如果顶级 ViewGroup 不拦截事件，事件传递给其点击事件链上的子 View，子 View 的 dispatchTouchEvent 被调用，如此循环，完成整个事件的分发

    - ViewGroup#dispatchTouchEvent

      > ViewGroup 在事件类型为 ACTION_DOWN 或 mFirstTouchTarget != null 时判断是否拦截
      >
      > 当事件由 ViewGroup 的子元素成功处理时，mFirstTouchTarget 会被赋值并指向子元素
      >
      > 当事件由 ViewGroup 拦截，mFirstTouchTarget != null 不成立，当 ACTION_MOVE、ACTION_UP 来时，判断条件为 false，ViewGroup 的 onInterceptTouchEvent 不再被调用，同一序列中其他事件默认交给其处理
      >
      > FLAG_DISALLOW_INTERCEPT 标记位通过 requestDisallowInterceptTouchEvent 方法设置，一般用于子 View，让 ViewGroup 不再拦截事件，前提是 ViewGroup 不拦截 ACTION_DOWN 事件
      >
      > ViewGroup 分发事件时，如果是 ACTION_DOWN 会重置 FLAG_DISALLOW_INTERCEPT 标记位，导致子 View 中该标记位无效

    ```java
    //检查是否被拦截
    final boolean intercepted;
    if (actionMasked == MotionEvent.ACTION_DOWN || mFirstTouchTarget != null){
        final boolean disallowIntercept = (mGroupFlags&FLAG_DISALLOwW_INTERCEPT) != 0;
        if(!disallowIntercept) {
            intercepted = onInterceptTouchEvent(ev);
            ev.setAction(action); //恢复操作以防更改
        }else {
            intercepted = false;
        }
    }else{
        //没有接触目标，此action不是初始down，此ViewGroup继续拦截触摸
        intercepted = true;
    }
    ```

    - ViewGroup#dispatchTouchEvent

      > 当 ViewGroup 决定拦截事件，后续点击事件默认交给其处理，不再调用其 onInterceptTouchEvent
      >
      > onInterceptTouchEvent 不是每次事件都被调用，想提前处理所有点击事件要选择 dispatchTouchEvent，只有其能每次都调用，前提是事件能传递到当前 ViewGroup

    ```java
    //处理初始down
    if (actionMasked == MotionEvent.ACTION_DOWN){
        //当开始一个新触摸手势时，扔掉所有旧状态，由于应用程序切换、ANR或其他一些状态更改，框架可能已放弃上个手势的up或cancel事件
        cancelAndclearTouchTargets(ev);
        resetTouchState();//重置FLAG_DISALLOW_INTERCEPT，子View调用requestDisallowInterceptTouchEvent并不影响ViewGroup对ACTION_DOWN事件的处理
    
    }
    ```

    - ViewGroup#dispatchTouchEvent

      > 不拦截时，事件向下分发交由其子 View 处理
      >
      > 遍历 ViewGroup 所有子元素，判断子元素是否能接收点击事件：是否在播动画、点击事件坐标是否落在子元素区域内
      >
      > dispatchTransformedTouchEvent 实际调用子元素的 dispatchTouchEvent 方法
      >
      > 如果子元素的 dispatchTouchEvent 返回 true，mFirstTouchTarget 被赋值（在 addTouchTarget 内部完成），跳出 for循环，返回 false 就分发给下一个元素（有的话）
      >
      > 如果遍历所有子元素后事件都没被处理：ViewGroup 无子元素、子元素处理点击事件但 dispatchTouchEvent 返回 false（一般是子元素在 onTouchEvent 中返回 false），此时 ViewGroup 自己处理点击事件

    ```java
    final View[] children = mChildren;
    for (int i = childrenCount-1;i >= 0;i--){
        final int childindex = customorder ? getChildDrawingorder(childrenCount,i) : i;
        final View child = (preorderedList == null) ? children[childIndex] : preorderedList.get(childIndex);
        if(!canViewReceivePointerEvents(child) || !isTransformedTouchPointInView(x, y, child, null)){
            continue;
        }
    
        newTouchTarget = getTouchTarget(child);
        if (newTouchTarget != nul1) {
            //孩子已在自己范围内受到触碰，除它正在处理的指针外，给它一个新指针
            newTouchTarget.pointerIdBits |= idBitsToAssign;
            break;
        }
    
        resetCancelNextUpFlag(child);
        if(dispatchTransformedTouchEvent (ev, false, child, idBitsToAssign)){
            //孩子想在自己范围内获得触碰
            mLastTouchDownTime = ev.getDownTime();
            if(preorderedList != null){
                //childIndex指向预先排序的列表，找到原始索引
                for (int j=0;j < childrenCount;j++) {
                    if(children[childIndex] == mchildren[j]){
                        mLastTouchDownIndex = j;
                        break;
                    }
                }
            }else {
                mLastTouchDownIndex = childIndex;
            }
            mLastTouchDownX = ev.getX();
            mLastTouchDownY = ev.getY();
            newTouchTarget = addTouchTarget(child, idBitsToAssign);
            alreadyDispatchedToNewTouchTarget = true;
            break;
        }
    }
    
    //分发触摸目标
    if (mEirstTouchTarget == null) {
        //无触摸目标，当作一个普通View，第三个参数child为null
        handled = dispatchTransformedTouchEvent(ev, canceled, null, TouchTarget.ALL_POINTER_IDS);
    }
    ```

    - dispatchTransformedTouchEvent

    ```java
    if(child == null){
        handled = super.dispatchTouchEvent(event);
    }else {
        handled = child.dispatchTouchEvent(event);
    }
    ```

    - addTouchTarget

      > mFirstTouchTarget 是单链表结构，其是否被赋值，直接影响到 ViewGroup 对事件的拦截策略，如为 null，ViewGroup 默认拦截接下来同一序列中所有点击事件

    ```java
    private TouchTarget addTouchTarget (View child, int pointerldBits){
        TouchTarget target = TouchTarget.obtain (child, pointerIdBits);
        target.next = mFirstTouchTarget;
        mFirstTouchTarget = target;
        return target;
    }
    ```

  - View 对点击事件的处理

    - View#dispatchTouchEvent

      > View（不包含 ViewGroup）是个单独元素，无法向下传递事件，只能自己处理
      >
      > 判断有无设置 OnTouchListener，如果 OnTouchListener 的 onTouch 返回 true，onTouchEvent 不被调用，OnTouchListener 优先级高于 onTouchEvent，方便在外界处理点击事件

    ```java
    public boolean dispatchTouchEvent (MotionEvent event){
        boolean result = false;
        if(onFilterTouchEventForSecurity(event)) {
            //无检验简化If语句
            ListenerInfo li = mListenerInfo;
            if(li != null && li.mOnTouchListener != null && (mviewFlags & ENABLED_MASK) == ENABLED && li.monTouchListener.onTouch(this,event)){
                result = true;
            }
            
            if (!result && onTouchEvent(event)) {
                result = true;
            }
        }
        //...
        return result;
    }
    ```

    - View#onTouchEvent

      > 不可用状态下 View 会消耗点击事件，只是不可用
      >
      > 如果 View 设置有代理，会执行 TouchDelegate 的 onTouchEvent 方法，工作机制和 OnTouchListener 类似
      >
      > 只要 View 的 CLICKABLE 和 LONG_CLICKABLE 有一个为 true，就会消耗该事件（onTouchEvent 返回 true），不管是不是 DISABLE 状态
      >
      > ACTION_UP 事件发生会触发 performClick 方法，如果 View 设置了 OnClickListener，performClick 内部会调用其 onClick
      >
      > View 的 LONG_CLICKABLE 默认为 false，CLICKABLE 和具体 View 有关，可点击 View 的 CLICKABLE 为 true（Button），不可点击为 false（TextView），setClickable、setLongClickable 可分别改变 View 的 CLICKABLE 和 LONG_CLICKABLE
      >
      > setOnClickListener 会自动将 View 的 CLICKABLE 设为 true，setOnLongClickListener 会自动将 View 的 LONG_CLICKABLE 设为 true

    ```java
    if((viewFlags & ENABLED_MASK) == DISABLED){
        if (event.getAction() == MotionEvent.ACTION_UP && (mPrivateFlags & PFLAG_PRESSED) != 0){
            setPressed (false);
        }
        //可单击的disable View仍然会使用触摸事件，只是不响应其
        return (((viewFlags & CLICKABLE) == CLICKABLE || (viewFlags & LONG_CLICKABLE) == LONG_CLICKABLE));
    }
    
    if(mTouchDelegate != null){
        if(mTouchDelegate.onTouchEvent(event)) {
            return true;
        }
    }
    
    if(((viewFlags & CLICKABLE) == CLICKABLE || (viewFlags & LONG_CLICKABLE) == LONG_CLICKABLE)){
        switch (event.getAction()){
            case MotionEvent.ACTION_UP:
                boolean prepressed = (mPrivateFlags & PFLAG_PREPRESSED) != 0;
                if((mPrivateElags & PFLAG_PRESSED) != 0 || prepressed){
                    //...
                    if (!mHasPerformedLongPress) {
                        //这是一个轻击，移除长按检查
                        removeLongPressCallback():
                        //仅当处于按下状态时才执行单击操作
                        if(!focusTaken){
                            //使用Runnable并post，而不直接调用performClick。这使view其他可视状态可在单击操作开始前更新
                            if (mPerformclick == null){
                                mPerformClick = new PerformClick();
                            }
                            if(!post(mPerformClick)){
                                performClick();
                            }
                        }
                    }
                    //...
                }break;
        }
        //...
        return true;
    }
    ```

    - performClick

    ```java
    public boolean performClick() {
        final boolean result;
        final ListenerInfo li = mListenerInfo;
        if (li != null && li.mOnClickListener != null) {
            playsoundEffect(SoundEffectConstants.CLICK);
            li.monclickListener.onClick (this);
            result = true;
        }else {
            result = false;
        }
        sendAccessibilityEvent (AccessibilityEvent.TYPE_VIEW_CLICKED);
        return result;
    }
    ```

    - setOnClickListener

    ```java
    public void setonclickListener (OnClickListener l){
        if(!isClickable()){
            setClickable (true);
        }
        getListenerInfo().monclickListener = l;
    }
    ```

    - setOnLongClickListener

    ```java
    public void setonLongClickListener(OnLongclickListener l){
        if(!isLongClickable()) {
            setLongClickable (true);
        }
        getListenerInfo().mOnLongClickListener = l;
    }
    ```

#### 3.5、View 滑动冲突

> 界面中只要内外两层同时可滑动，这时会产生滑动冲突

- 滑动冲突场景
  - 外部滑动方向和内部滑动方向不一致
    - 主要是 ViewPager 和 Fragment 配合使用，可通过左右滑动切换页面，每个页面内部是一个 ListView，ViewPager 内部处理了滑动冲突，采用 ScrollView 等必须手动处理滑动冲突，否则内外两层只能有一层能滑动
  - 外部滑动方向和内部滑动方向一致
    - 存在逻辑问题，当手指开始滑动时，系统不知想哪一层滑动，要么只有一层能滑动，要么内外两层都滑动很卡顿
  - 上面两种情况的嵌套
    - 几个单一滑动冲突的叠加，只需分别处理内层和中层、中层和外层间的滑动冲突，处理方法以上两者相同

![安卓开发艺术探索_滑动冲突场景](Image.assets\安卓开发艺术探索_滑动冲突场景.png)

- 处理规则

  - 场景1：左右滑动时，让外部 View 拦截点击事件，上下滑动时，让内部 View 拦截点击事件，根据滑动过程中两点间坐标可得水平还是竖直滑动，得到滑动方向：可依据滑动路径和水平方向所成夹角，可依据水平方向和竖直方向距离差、速度差来判断
  - 场景2：无法根据滑动角度、距离差、速度差判断，一般在业务上突破，如当处于某种状态时需外部 View 响应滑动，处于另一种状态时则需内部 View 响应滑动
  - 场景3：和场景 2 一样

- 解决方式

  - 外部拦截法：点击事件都先经过父容器拦截，如果父容器需此事件就拦截，不需要就不拦截，这样比较符合点击事件分发机制
    - 需重写父容器的 onInterceptTouchEvent，在内部做相应拦截
    - ACTION_DOWN 事件父容器必须返回 false（不拦截），一旦父容器拦截 ACTION_DOWN，后续 ACTION_MOVE、ACTION_UP 事件会直接交由父容器处理，没法再传给子元素
    - ACTION_MOVE 事件根据需要决定拦截
    - ACTION_UP 事件必须返回 false，假设事件交由子元素处理，如果父容器在 ACTION_UP 时返回 true，会导致子元素无法接收到 ACTION_UP 事件，这时子元素 onClick 事件无法触发，但父容器一旦它开始拦截任何一个事件，后续事件都会交给其处理，ACTION_UP 作为最后一个事件也可传给父容器，即便其 onInterceptTouchEvent 在 ACTION_UP 时返回 false

  ```java
  public boolean onInterceptTouchEvent (MotionEvent event) {
      boolean intercepted = false;
      int x = (int) event.getX();
      int y = (int) event.getY();
      switch (event.getAction()) {
          case MotionEvent.ACTION_DOWN:{
              intercepted = false;
              break;
          }
          case MotionEvent.ACTION_MOVE:{
              if(父容器需要当前点击事件){
                  intercepted = true;
              }else {
                  intercepted = false;
              }break;
          }
          case MotionEvent.ACTION_UP:{
              intercepted = false;
              break;
          }
          default:break;
      }
      mLastXIntercept = x;
      mLastYIntercept = y;
      return intercepted;
  }
  ```

  - 内部拦截法：父容器不拦截事件，所有事件传给子元素，如果子元素需此事件直接消耗掉，否则交由父容器处理，和事件分发机制不一致
    - 需配合 requestDisallowInterceptTouchEvent，较外部拦截稍复杂
    - 除子元素需处理外，父元素要默认拦截除 ACTION_DOWN 外其他事件，当子元素调用 parent.requestDisal-lowInterceptTouchEvent(false) 时，父元素才能继续拦截所需事件
    - 父容器不能拦截 ACTION_DOWN 事件，ACTION_DOWN 事件不受 FLAG_DISALLOW_INTERCEPT 标记位控制，一旦父容器拦截 ACTION_DOWN 事件，所有事件都无法传到子元素中

  ````java
  //子元素
  public boolean dispatchTouchEvent (MotionEvent event){
      int x = (int) event.getX():
      int y = (int) event.getY();
      
      switch (event.getAction()){
          case MotionEvent.ACTION_DOWN:{
              parent.requestDisallowInterceptTouchEvent (true);
              break;
          }
          case MotionEvent.ACTION_MOVE: {
              int deltaX = x - mLastX;
              int deltaY = Y - mLastY;
              if(父容器需要此类点击事件){
                  parent.requestDisallowInterceptTouchEvent (false);
              }break;
          }
          case MotionEvent.ACTION_UP:{
              break;
          }
          default:break;
      }
      mLastX = X;
      mLastY = y;
      return super.dispatchTouchEvent (event);
  }
  //父元素
  public boolean oninterceptTouchEvent (MotionEvent event){
      int action = event.getAction();
      if(action == MotionEvent.ACTION_DOWN){
          return false;
      }else{
          return true;
      }
  }
  ````

### 4、View 工作原理

#### 4.1、ViewRoot & DecorView

- ViewRoot：对应 ViewRootImpl 类，是连接 WindowManager 和 DecorView 的纽带，View 的 measure、layout、draw 流程通过其完成

  - ActivityThread 中当 Activity 对象被创建完毕，将 DecorView 添加到 Window 中，同时创建 ViewRootImpl 对象，将 ViewRootImpl 对象和 DecorView 建立关联，源码：

    ```java
    root = new ViewRootImpl(view.getContext(), display);
    root.setView (view, wparams, panelParentView);
    ```

  - View 绘制流程从 ViewRoot 的 performTraversals 开始，经过 measure、layout、draw 绘制，measure 测量 View 的宽高，layout 确定 View 在父容器中的放置位置，draw 负责绘制 View，工作流程
    - performTraversals 依次调用 performMeasure、performLayout、performDraw，其分别完成顶级 View 的 measure、layout、draw
    - performMeasure 调用 measure，measure 中调用 onMeasure，onMeasure 中对所有子元素进行 measure，这时 measure 流程从父容器传到子元素，完成一次 measure 过程
    - 子元素重复父容器的 measure 过程，完成整个 View 树遍历，performLayout、performDraw 的传递流程类似
    - measure 过程决定 View 的宽高，完成后可通过 getMeasuredWidth、getMeasuredHeight 获取 View 测量后的宽高，几乎所有情况都等同 View 最终宽/高
    - Layout 过程决定 View 四个顶点坐标和实际 View 的宽/高，完成后可通过 getTop、getBottom、getLeft、getRight 拿到 View 四个顶点位置，通过 getWidth、getHeight 拿到 View 最终宽高
    - Draw 过程决定 View 的显示，完成后 View 内容才能呈现屏幕上

  ![安卓开发艺术探索_performTraversales工作流程](Image.assets\安卓开发艺术探索_performTraversales工作流程.png)

- DecorView：顶级 View，一般情况包含一个竖直方向的 LinearLayout，其里面有两部分（和 Android 版本及主题有关），上面是标题栏，下面是内容栏

  - Activity 通过 setContentView 设置的布局文件被加到内容栏中，内容栏的 id 是 content，得到 content：`ViewGroup content = findViewById(R.android.id.content)`，得到设置的 View：`content.getChildAt(0)`
  - DecorView 是一个 FrameLayout，View 层事件先经过 DecorView，后传递给 View

  ![安卓开发艺术探索_DecorView结构](Image.assets\安卓开发艺术探索_DecorView结构.png)

#### 4.2、MeasureSpec

> 测量中，系统将 View 的 LayoutParams 根据父容器施加的规则转换成对应 MeasureSpec，根据其测量出  View 的宽高

- MeasureSpec：一个 32 位 int 值，高 2 位代表 SpecMode（测量模式），低 30 位代表 SpecSize（规格大小），源码：

  - SpecMode
    - UNSPECIFIED：父容器不对 View 有任何限制，一般用于系统内部
    - EXACTLY：父容器已检测出 View 所需精确大小，这时 View 最终大小是 SpecSize 指定值。对应于 LayoutParams 中 match_parent 和具体数值
    - AT_MOST：父容器指定一个可用大小 SpecSize，View 大小不能大于该值，对应于 LayoutParams 中 wrap_content

  ```java
  private static final int MODE_SHIFT = 30;
  private static final int MODE_MASK = 0x3 << MODE_SHIFT;
  public static final int UNSPECIFIED = 0 << MODE_SHIFT;
  public static final int EXACTLY = 1 << MODE_SHIFT;
  public static final int AT_MOST = 2 << MODE_SHIFT;
  public static int makeMeasureSpec(int size, int mode){
      if (sUseBrokenMakeMeasureSpec){
          return size + mode;
      }else{
          return (size & ~MODE_MASK) | (mode & MODE_MASK);
      }
  }
  public static int getMode (int measureSpec){
      return (measureSpec & MODE_MASK);
  }
  public static int getsize(int measureSpec) {
      return (measureSpec & ~MODE_MASK);
  }
  ```

- MeasureSpec 和 LayoutParams 对应关系

  - 系统内部通过 MeasureSpec 进行 View 测量，可给 View 设置 LayoutParams，View 测量时系统将 LayoutParams 在父容器的约束下转成对应 MeasureSpec，根据其确定 View 测量后的宽高，MeasureSpec 不是唯一由 LayoutParams 决定，LayoutParams 需和父容器一起才能决定 View 的 MeasureSpec

  - DecorView 的 MeasureSpec 由窗口尺寸和自身 LayoutParams 共同确定

    - LayoutParams.MATCH_PARENT：精确模式，大小是窗口大小

    - LayoutParams.WRAP_CONTENT：最大模式，大小不定，但不能超过窗口大小

    - 固定大小（如 100dp ）：精确模式，大小为 LayoutParams 中指定大小

    - ViewRootImpl#measureHierarchy

      > desiredWindowWidth、desiredWindowHeight 是屏幕尺寸

    ```java
    //DecorView的MeasureSpec创建过程
    childwidthMeasureSpec = getRootMeasureSpec(desiredWindowWidth,lp.width);
    childHeightMeasureSpec = getRootMeasureSpec(desiredwindowHeight,lp.height);
    performMeasure(childwidthMeasureSpec, childHeightMeasureSpec);
    ```
    - getRootMeasureSpec

    ```java
    private static int getRootMeasureSpec(int windowSize,int rootDimension){
        int measureSpec;
        switch (rootDimension) {
            case viewGroup.LayoutParams.MATCH_PARENT:
                //窗口无法调整大小，强制根视图为窗口大小
                measureSpec = MeasureSpec.makeMeasureSpec(windowSize, MeasureSpec.EXACTLY);break;
            case ViewGroup.LayoutParams.WRAP_CONTENT:
                //窗口可以调整大小。设置根视图的最大大小
                measureSpec = MeasureSpec.makeMeasureSpec(windowSize, MeasureSpec.AT_MOST);break;
            default:
                //窗口需要精确大小。强制根视图为该大小
                measureSpec = MeasureSpec.makeMeasureSpec(rootDimension, MeasureSpec.EXACTLY);break;
        }
        return measureSpec;
    }
    ```

  - 普通 View 的 MeasureSpec 由父容器的 MeasureSpec 和自身 LayoutParams 共同决定

    - ViewGroup#measureChildWithMargins

      > 对子元素进行 measure，调用子元素的 measure 前会通过 getChildMeasureSpec 得到子元素的 MeasureSpec
      >
      > 子元素 MeasureSpec 的创建与父容器 MeasureSpec 和子元素本身的 LayoutParams 有关，还和 View 的 margin 及 padding 有关

    ```java
    protected void measureChildwithMargins(view child, int parentWidthMeasureSpec, int widthUsed, int parentHeightMeasureSpec, int heightUsed){
        final MarginLayoutParams lp = (MarginLayoutParams) child.getLayoutParams();
        final int childwidthMeasureSpec = getchildMeasureSpec(parentWidthMeasureSpec, mPaddingLeft + mPaddingRight + lp.leftMargin + lp.rightMargin + widthUsed, lp.width);
        final int childHeightMeasureSpec = getchildMeasureSpec(parentHeightMeasureSpec, mPaddingTop + mPaddingBottom + lp.topMargin + lp.bottomMargin + heightUsed, lp.height);
        child.measure(childWidthMeasureSpec, childHeightMeasureSpec);
    }
    ```

    - ViewGroup#getChildMeasureSpec

      > 根据父容器的 MeasureSpec，结合本身 LayoutParams 确定子元素的 MeasureSpec

    ```java
    public static int getChildMeasureSpec(int spec, int padding,int childDimension) {
        int specMode = MeasureSpec.getMode(spec);
        int specsize = MeasureSpec.getSize(spec);
        //padding指父容器中已占用的空间大小
        int size = Math.max(0, specSize - padding);
        int resultSize = 0;
        int resultMode = 0;
    
        switch (specMode){
                //父View给了一个确切尺寸 
            case MeasureSpec.EXACTLY:
                if (childDimension >= 0){
                    resultSize = childDimension;
                    resultMode = MeasureSpec.EXACTLY;
                }else if(childDimension == LayoutParams.MATCH_PARENT){
                    //子View想和我们一样大
                    resultSize = size;
                    resultMode = MeasureSpec.EXACTLY;
                }else if (childDimension == LayoutParams.WRAP_CONTENT){
                    //子View想决定自己的大小，不能比我们大
                    resultSize = size;
                    resultMode = MeasureSpec.AT_MOST;
                }break;
                //父View施加了最大的限制
            case MeasureSpec.AT_MOST:
                if(childDimension >= 0){
                    //子View想要一个特定的尺寸，给他吧
                    resultSize = childDimension;
                    resultMode= MeasureSpec.EXACTLY;
                }else if(childDimension == LayoutParams.MATCH_PARENT){
                    //子View想要我们的尺寸，但我们的尺寸不是固定的，约束孩子不要比我们大
                    resultSize = size;
                    resultMode = MeasureSpec.AT_MOST;
                }else if (childDimension == LayoutParams.WRAP_CONTENT){
                    //子View想决定自己的大小，不能比我们大
                    resultSize = size;
                    resultMode = MeasureSpec.AT_MOST;
                }break;
                //父View问我们要多大
            case MeasureSpec.UNSPECIFIED:
                if(childDimension >= 0) {
                    //子View想要一个特定的尺寸，给他吧
                    resultSize = childDimension;
                    resultMode = MeasureSpec.EXACTLY;
                }else if (childDimension == LayoutParams. MATCH_PARENT){
                    //子View想和我们一样大，看它应该有多大
                    resultSize = 0;
                    resultMode = MeasureSpec.UNSPECIFIED;
                }else if (childDimension == LayoutParams.WRAP_CONTENT){
                    //子View想决定自己的大小，看它应该有多大
                    resultSize = 0;
                    resultMode = MeasureSpec.UNSPECIFIED;
                }break;
        }
        return MeasureSpec.makeMeasureSpec(resultsize, resultMode);
    }
    ```

    - 规则

      > parentSize 指父容器中目前可用大小
      >
      > 当 View 采用固定宽高时，View 的 MeasureSpec 都是精确模式，且大小遵循 Layoutparams 大小
      >
      > 当 View 的宽高是 match_parent 时，如果父容器是精准模式，View 是精准模式，且大小是父容器的剩余空间；如果父容器是最大模式，View 是最大模式，且大小不会超过父容器的剩余空间
      >
      > 当 View 的宽高是 wrap_content 时，View 模式总是最大化，且大小不能超过父容器剩余空间。
      >
      > UNSPECIFIED 模式主要用于系统内部多次 Measure 的情形，一般不需关注

    | chlidLayoutParams\parcntSpecModc |      EXACTLY       |      AT_MOST       |    UNSPECIFIED    |
    | :------------------------------: | :----------------: | :----------------: | :---------------: |
    |              dp/px               | EXACTLY childsize  | EXACTLY childsize  | EXACTLY childsize |
    |          match _parent           | EXACTLY parentSize | AT_MOST parentSize |   UNSPECIFIED 0   |
    |          warap__content          | AT_MOST parentSize | AT_MOST parentSize |   UNSPECIFIED 0   |

#### 4.3、View 的工作流程

- measure：measure 完成后，getMeasuredWidth/Height 方法就可正确获取到 View 的测量宽高。在某些极端情况下，系统可能需要多次 measure 才能确定最终测量宽/高，此时 onMcasure 拿到的测量宽/高可能不准确，要在 onLayout 中获取

  - View 的 measure：由其 measure 方法完成，measure 是 final 类型，子类不能重写，该方法中会调用 View 的 onMeasure 方法

    - View#onMeasure

      > getDefaultSize 返回的大小是 measureSpec 中的 specSize，specSize 是 View 测量后的大小，View 最终大小在 layout 阶段确定，几乎所有情况 View 的测量大小和最终大小相等

    ```java
    protected void onMeasure(int widthMeasureSpec,int heightMeasureSpec){
        setMeasuredDimension(getDefaultSize(getSuggestedMinimumWidth(), widthMeasureSpec), getDefaultsize(getSuggestedMinimumHeight(),heightMeasureSpec));
    }
    ```

    - getSuggestedMinimumWidth & getSuggestedMinimumHeight

      > 如果 View 没有设背景，View 的宽度为 mMinWidth，对应 android:minWidth 属性指定值，如果该属性不指定，mMinWidth 默认为 0
      >
      > 如果指定背景，View 宽度为 android:minWidth 和背景最小宽度两者中最大值

    ```java
    protected int getSuggestedMinimumWidth(){
        return (mBackground == null) ? mMinWidth : max (mMinWidth, mBackground.getMinimumWidth());
    }
    
    protected int getSuggestedMinimumHeight(){
        return (mBackground == null) ? mMinHeight : max (mMinHeight, mBackground.getMinimumHeight());
    }
    ```

    - getMinimumWidth

      > 返回 Drawable 原始宽度，前提是 Drawable 有原始宽度，否则返回 0
      >
      > ShapeDrawable 无原始宽高，BitmapDrawable 有原始宽高（图片尺寸)

    ```java
    public int getMinimunwidth() {
        final int intrinsieWidth = getIntrinsicwidth();
        return intrinsicwidth > 0 ? intrinsicwidth : 0;
    }
    ```

    - 直接继承 View 的自定义控件需重写 onMeasure 并设置 wrap_content 时的自身大小，否则在布局中使用 wrap_content 就相当于 match_parent，如果 View 在布局中使用 wrap_content，其 specMode 是 AT_MOST 模式，此时其宽高等于 specSize，此时 specSize 是 parentSize，和在布局中 match_parent 一致。解决：给 View 指定一个默认的内部宽高（mWidth 和 mHeight）

    ```java
    protected void onMeasure(int widthMeasureSpec, int heightMeasureSpec){
        super.onMeasure(widthMeasureSpec, heightMeasureSpec);
        int widthSpecMode = MeasureSpec.getMode(widthMeasureSpec);
        int widthSpecsize = MeasureSpec.getSize(widthMeasureSpec);
        int heightSpecMode = MeasureSpec.getMode(heightMeasureSpec);
        int heightspecsize = Measurespec.getsize(heightMeasureSpec);
        if (widthSpecMode == MeasureSpec.AT_MOST && heightSpecMode == MeasureSpec.AT_MOST) {
            setMeasuredDimension(mWidth, mHeight);
        }else if(widthSpecMode == MeasureSpec.AT_MOST) {
            setMeasuredDimension(mWidth, heightSpecsize);
        }else if(heightSpecMode == MeasureSpec.AT_MOST) {
            setMeasuredDimension(widthSpecSize, mHeight);
        }
    }
    ```

  - ViewGroup 的 measure：除完成自己的测量外，还遍历调用所有子元素的 measure 方法，各子元素再递归执行，ViewGroup 是一个抽象类，没有重写 View 的 onMeasure，但提供 measureChildren

    - 取出子元素的 LayoutParams，通过 getChildMeasureSpec 创建子元素的 MeasureSpec，将 MeasureSpec 传给 View 的 measure 方法测量
    - ViewGroup 统一实现，如 LinearLayout 和 RelativeLayout 的布局特性不同

    ```java
    protected void measureChildren (int widthMeasureSpec, int heightMeasureSpec){
        final int size = mChildrenCount;
        final view[] children = mChildren;
        for (int i=0;i<size;++i){
            final View child = children[i];
            if((child.mViewFlags & VISIBILITY_MASK) != GONE){
                measureChild(child, widthMeasureSpec, heightMeasureSpec);
            }
        }
    }
    
    protected void measureChild (View child, int parenthidthMeasureSpec, int parentHeightMeasureSpec) {
        final LayoutParams lp = child.getLayoutParams();
        final int childwidthMeasureSpec = getChildMeasurespec (parentwidthMeasureSpec, mPaddingLeft + mPaddingRight, lp.width);
        final int childHeightMeasureSpec = getChildMeasureSpec (parentHeightMeasureSpec, mPaddingTop + mPaddingBottom, lp.height);
        child.measure(childWidthMeasureSpec, childHeightMeasureSpec);
    }
    ```

    - LinearLayout#onMeasure

    ```java
    protected void onMeasure(int widthMeasureSpec, int heightMeasureSpec){
        if (morientation == VERTICAL){
            //该方法遍历子元素并执行其measureChildBeforeLayout，调用子元素measure，通过mTotalLength变量存储LinearLayout在竖直方向的初步高度，每测量一个子元素就增加，增加部分包括子元素高度及子元素竖直方向上margin等，子元素测量后，LinearLayout会测量自己的大小
            measureVertical(widthMeasureSpec, heightMeasureSpec);
        }else {
            measureHorizontal(widthMeasureSpec, heightMeasureSpec);
        }
    }
    ```

  - 在 Activity 已启动时候获取某个 View 的宽高，在 onCreate、onStart、onResume 中均无法正确得到其宽高信息，因为 View 的 measure 过程和 Activity 生命周期不同步执行，解决：

    - Activity / View#onWindowFocusChanged：View 初始化完，宽高已准备好，这时候可获取宽高。当 Activity 继续和暂停执行时（onResume、onPause），onWindowFocusChanged 均被调用

    ```java
    public void onWindowFocusChanged (boolean hasFocus) {
        super.onWindowFocusChanged (hasFocus);
        if (hasFocus){
            int width = view.getMeasuredwidth();
            int height = view.getMeasuredHeight();
        }
    }
    ```

    - view.post(runnable)：将一个 runnable 投递到消息队列尾部，等待 Looper 调用时，View 已初始化好

    ```java
    protected void onStart(){
        super.onStart();
        view.post(new Runnable(){
            @override
            public void run(){
                int width = view.getMeasuredwidth();
                int height = view.getMeasuredHeight();
            }
        });
    }
    ```

    - ViewTreeObserver：其众多回调可实现，如 OnGlobalLayoutListener 接口，当 View 树状态改变或 View 树内 View 可见性改变时，onGlobalLayout 方法被回调，可获取 View 的宽高，伴随 View 树状态改变，onGlobalLayout 会被调用多次

    ```java
    protected void onStart(){
        super.onstart();
        ViewTree0bserver observer = view.getViewTree0bserver();
        observer.addonGlobalLayoutListener(new onGlobalLayoutListener(){
            @SuppressWarnings("deprecation")
            @Override
            public void onGlobalLayout(){
                view.getViewTree0bserver().removeGlobalOnLayoutListener (this);
                int width = view.getMeasuredwidth();
                int height = view.getMeasuredHeight();
            }
        });
    }
    ```

    - view.measure(int widthMeasureSpec, int heightMeasureSpec)：手动对 View 进行 measure 得到 View 的宽高。根据 View 的 LayoutParams 分情况：

      > match_parent：无法 measure 出具体宽高，构造 MeasureSpec 需知道 parentSize（父容器剩余空间），而这时不知道 parentSize 大小
      >
      > 具体数值（dp/px）：如 100px

      ```java
      int widthMeasureSpec = MeasureSpec.makeMeasureSpec(100, MeasureSpec.EXACTLY);
      int heightMeasureSpec = MeasureSpec.makeMeasureSpec(100, MeasureSpec.EXACTLY);
      view.measure(widthMeasureSpec, heightMeasureSpec);
      ```

      > wrap_content：

      ```java
      //MeasureSpec的实现：View尺寸使用30位二进制表示(1<<30)-1
      int widthMeasureSpec = MeasureSpec.makeMeasureSpec((1 << 30) - 1, MeasureSpec.AT_MOST);
      int heightMeasureSpec = MeasureSpec.makeMeasureSpec((1 << 30) - 1, MeasureSpec.AT_MoST);
      view.measure(widthMeasureSpec, heightMeasureSpee);
      ```

      > 错误用法：无法通过错误的 MeasureSpec 得到合法的 SpecMode，导致 measure 出错

      ```java
      //第一种
      int widthMeasureSpec = MeasureSpec.makeMeasureSpec(-1, MeasureSpec.UNSPECIFIED);
      int heightMeasureSpec = MeasureSpec.makeMeasureSpec(-1, MeasureSpec,UNSPECIFIED);
      view.measure(widthMeasureSpec, heightMeasureSpec);
      //第二种
      view.measure(LayoutParams.WRAP_CONTENT, LayoutParams.WRAP_CONTENT);
      ```

- layout：当 ViewGroup 位置被确定后，在 onLayout 中遍历所有子元素并调用其 layout 方法，依次循环，layout 确定 View 本身位置，onLayout 确定所有子元素位置

  - View#layout
  - 通过 setFrame 设定 View 四个顶点位置（初始化 mLeft、mRight、mTop、mBottom），这四个顶点一旦确定，View 在父容器中的位置也确定
    - 调用 onLayout，让父容器确定子元素位置，和 onMeasure 类似，其具体实现和具体布局有关，View 和 ViewGroup 均无实现 onLayout 方法
  
  ```java
  public void layout (int l, int t, int r, int b){
      if((mPrivateFlags3 & PELAG3_MEASURE_NEEDED_BEFORE_LAYoUT) != 0){
          onMeasure(moldWidthMeasureSpec, moldHeightMeasureSpec);
          mPrivateFlags3 &= ~PFLAG3_MEASURE_NEEDED_BEFORE_LAYOUT;
      }
      
      int oldL = mLeft;
      int oldT = mTop;
      int oldB = mBottom;
      int oldR = mRight;
      boolean changed = isLayoutModeOptical(mParent) ? setOpticalFrame(l,t,r,b) : setFrame(l,t,r,b);
      if (changed || (mPrivateFlags & PFLAG_LAYOUT_REQUIRED) == PFLAGLAYOUT_REQUIRED){
          onLayout(changed, l, t, r, b);
          mPrivateFlags &= ~PFLAG_LAYOUT_REQUIRED;
          
          ListenerInfo li = mListenerInfo;
          if(li != null && li.mOnLayoutChangeListeners != null){
              ArrayList<OnLayoutChangeListener> listenersCopy = (ArrayList<OnLayoutChangeListener>)li.mOnLayoutChangeListeners.clone();
              int numListeners = listenersCopy.size();
              for (int i=0;i<numListeners;++i){
                  listenersCopy.get(i).onLayoutChange(this, l, t, r, b, oldE, oldT, oldR, oldB);
              }
          }
      }
      mPrivateFlags &= ~PFLAG_FORCE_LAYOUT;
      mPrivateFlags3 |= PFLAG3_IS_LAID_OUT;
}
  ```
  
  - LinearLayout#onLayout方法
  
  ```java
  protected void onLayout(boolean changed, int l, int t, int r, int b){
      if (mOrientation == VERTICAL){
          //遍历所有子元素并调用setChildFrame为子元素指定对应位置，后面的子元素会被放在靠下的位置，setChildFrame仅是调用子元素的layout，递归完成了整个View树的layout过程
          layoutVertical(1, t, r, b);
      }else {
          layoutHorizontal(l, t, r, b);
      }
  }
  ```
  
  - View 的 getMeasuredWidth 和 getWidth 的区别
    - View 默认实现中，其测量宽高和最终宽高相等，不过测量宽高形成于 View 的 measure 过程，最终宽高形成于 View 的 layout 过程，两者赋值时机不同
  
- draw

  - 绘制步骤
    - 绘制背景 background.draw(canvas)
    - 绘制自己 onDraw
    - 绘制 children dispatchDraw
    - 绘制装饰 onDrawScrollBars
  - draw

  ```java
  public void draw(Canvas canvas){
      final int privateFlags = mPrivateFlags;
      final boolean dirty0paque = (privateFlags & PFLAG_DIRTY_MASK) == PFLAG_DIRTY_OPAQUE && (mAttachInfo == null || !mAttachInfo.mIgnoreDirtystate);
      mPrivateFlags = (privateFlags & ~PFLAG_DIRTY_MASK) | PFLAG_DRAWN;
      //画背景，如果需要
      int saveCount;
      
      if(!dirtyopaque){
          drawBackground (canvas);
      }
      //如果可能，跳过步骤2(保存画布的层以备褪色)和5(画出褪色的边缘并恢复层) （常见情况）
      final int viewFlags = mviewFlags;
      boolean horizontalEdges = (viewFlags & FADING_EDGE HORIZONTAL) != 0;
      boolean verticalEdges = (viewFlags & FADING_EDGE_VERTICAL) != 0;
      if(!verticalEdges && !horizontalEdges){
          //绘制视图内容
          if (!dirtyopaque) onDraw(canvas);
          //画孩子
          dispatchDraw(canvas);
          //绘制装饰（如scrollbar）
          if (mOverlay != null && !moverlay.isEmpty()) {
              mOverlay.getOverlayView().dispatchDraw(canvas);
          }
          //we're done...
          return;
      }
      //...
  }
  ```

  - View#setWillNotDraw
    - 如果 View 不需绘制，设置该标记位为 true，系统会进行相应优化。默认 View 没启用该标记位，但 ViewGroup 会默认启用
    - 作用：当自定义控件继承 ViewGroup 且本身不具备绘制功能时，可开启该标记位优化，反之需显式关闭 WILL_NOT_DRAW 这个标记位

  ```java
  public void setwillNotDraw (boolean willNotDraw){
      setFlags (willNotDraw ? WILL_NOT_DRAW : 0, DRAW_MASK);
  }
  ```

#### 4.4、自定义View

- 分类

  - 继承 View 重写 onDraw：用于实现一些不规则效果（不方便通过布局的组合方式达到，需要通过绘制方式实现，需自己支持 wrap_content 和 padding

    - 处理 padding

      ```java
      protected void onDraw (Canvas canivas) {
          super.onDraw (canvas);
          final int paddingLeft = getPaddingLeft();
          final int paddingRight = getPaddingLeft();
          final int paddingTop = getPaddingLeft();
          final int paddingBottom = getPaddingLeft();
          int width = getwidth() - paddingLeft - paddingRight; 
          int height = getHeight() - paddingTop - paddingBottom;
          int radius = Math.min(width, height) / 2;
          canvas.drawCircle (paddingLeft + width/2, paddingTop + height/2, radius, mPaint);
      }
      ```

  - 继承 ViewGroup 派生特殊的 Layout：用于实现自定义的布局（除 LinearLayout、RelativeLayout  外的新布局），适合几种 View 组合，需处理 ViewGroup 和子元素的测量和布局

  - 继承特定 View（如 TextView）：用于扩展某种已有 View 功能，不需自己支持wrap_content、padding

  - 继承特定的 ViewGroup（如 LinearLayout）：适合几种 View 组合，不需自己处理 ViewGroup 的测量和布局

- 注意

  - 让 View 支持 wrap_content：直接继承 View 或 ViewGroup，如果不在 onMeasure 中对 wrap_content 
    处理，当外界在布局中使用 wrap_content 时无法达到预期效果
  - 让 View 支持 padding：直接继承 View，如果不在 draw 中处理 padding，padding 属性不起作用，直接继承 ViewGroup 需在 onMeasure、onLayout 考虑 padding 和子元素的 margin 对其影响
  - 尽量别在 View 使用 Handler：View 内部提供 post 系列方法，可替代 Handler
  - View 中如果有线程或动画，需及时停止：如有线程或动画需停止时，onDetachedFromWindow 
    是很好时机
    - 当包含 View 的 Activity 退出或当前 View 被 remove 时，View 的
      onDetachedFromWindow 被调用
    - 当包含 View 的 Activity 启动时，View 的 onAttachedToWindow 被调用
    - 当 View 不可见时也需停止，不及时处理可能造成内存泄漏
  - View 带有滑动嵌套情形时，处理好滑动冲突

- 自定义属性

  - 在 values 目录创建自定义属性 XML，如 attrs.xml，也可以 atrs\_ 开头的文件名

    - 声明一个自定义属性集合 CircleView，集合里可以有很多自定义属性，这里定义一个格式为 color 的属性 circle_color，color 指颜色，reference 指资源 id，dimension 指尺寸，string、integer、boolean 指基本数据类型

    ```xml
    <?xml version="1.0" encoding="utf-8"?>
    <resources>
        <declare-styleable name="CircleView">
            <attr name="circle_color" format="color" />
        </declare-styleable>
    </resources>
    ```

  - 在 View 构造方法中解析自定义属性值并做处理

    - 加载自定义属性集合 CircleView
    - 解析 CircleView 属性集合中 circle_ color 属性，id 为 R.styleable.CircleView_circle_color，如
      果没指定 circle_color 属性，选择红色作为默认颜色值
    - 解析后通过 recycle 实现资源

    ```java
    public CircleView (Context context, AttributeSet attrs, int defStyleAttr) {
        super(context, attrs, defStyleAttr);
        TypedArray a = context.obtainStyledAttributes(attrs, R.styleable.CircleView);
        mColor = a.getColor(styleable.CircleView_circle_color, Color.RED);
        a.recycle();
        init();
    }
    ```

  - 在布局文件中使用自定义属性

    - 在布局文件中添加 schemas 声明 `xmIns:app=http://schemas.android.com/apk/res-auto`，app 是自定义属性前缀，可换名，CircleView 中的自定义属性前缀必须和这里一致
    - 在自定义 View 中使用自定义属性，如：app:circle_color="@color/light_green"
    - 也可声明 schemas：`xmlns:app=http://schemas.android.com/apk/res/com.hzy.chapter`，在 apk/res/ 后附加应用包名，两者无本质区别

### 5、RemoteViews

> RemoteViews 表示一个 View 结构，可在其他进程显示，由于它在其他进程中显示，提供了一组基础操作用于跨进程更新界面

#### 5.1、RemoteViews 的应用

- 使用场景：二者界面都运行在其他进程（系统 SystemServer）进程

  - 通知栏：通过 NotificationManager 的 notify 实现

    - 使用系统默认样式弹出通知

      ```java
      Notification notification = new Notification();
      notification.icon = R.drawable.ic_launcher;
      notification.tickerText = "hello world";
      notification.when = system.currentTimeMillis();
      notification.flags = Notification.FLAG_AUTO_CANCEL;
      Intent intent = new Intent(this, DemoActivity.class);
      PendingIntent pendingIntent = PendingIntent.getActivity(this, 0, intent, PendingIntent.FLAG_UPDATE_CURRENT);
      notification.setLatestEventInfo(this,"chapter_5","this is notification", pendingIntent);
      NotificationManager manager = (NotificationManager) getSystemService(Context.NOTIFICATION_SERVICE);
      manager.notify(1, notification);
      ```

    - 自定义通知：提供一个布局文件，通过 RemoteViews 加载布局文件即可改变通知样式

      > RemoteViews
      >
      > 创建：提供当前应用包名、布局文件资源 id 可创建一个 RemoteViews 对象
      >
      > 更新：无法直接访问里面的 View，通过 RemoteViews 提供的方法更新。如设置 TextView 文本：`remoteViews.setTextViewText(R.id.msg, "chapter")`，两个参数分别为 TextView
      > id 和要设置的文本
      >
      > 给控件添加单击事件：使用 PendingIntent 并通过 setOnClickPendingIntent 实现，如
      > `remoteViews.setOnClickPendingIntent(R.id.open_activity, openActivity2PendingIntent)` 会给 id 为 open_activity 的 View 加上单击事件
      >
      > PendingIntent：一种待定 Intent，包含的意图必须由用户触发

      ```java
      Notification notification = new Notification();
      notification.icon = R.drawable.ic_launcher;
      notification.tickerText = "hello world";
      notification.when = System.currentTimeMillis();
      notification.flags = Notification.FLAG_AUTO_CANCEL;
      Intent intent = new Intent(this, DemoActivity.class);
      PendingIntent pendingIntent = PendingIntent.getActivity(this, 0, intent, PendingIntent.FLAG_UPDATE_CURRENT);
      RemoteViews remoteViews = new RemoteViews(getPackageName(), R.layout.layout_notification);
      remoteViews.setTextViewText(R.id.msg, "chapter");
      remoteviews.setImageViewResource(R.id.icon, R.drawable.icon1);
      PendingIntent openActivity2PendingIntent = PendingIntent.getActivity(this, 0, new Intent(this, DemoActivity.class), PendingIntent.FLAG_UPDATE_CURRENT);
      remoteViews.setOnClickPendingIntent(R.id.open_activity2, openActivity2PendingIntent);
      notification.contentView = remoteviews;
      notification.contentIntent = pendingIntent;
      NotificationManager manager = (NotificationManager) getSystemService(Context.NOTIFICATION_SERVICE);
      manager.notify(2, notification);
      ```

  - 桌面小部件：通过 AppWidgetProvider 实现，本质是广播

    - 定义小部件界面：在 res/layout/ 下新建一个 XML 文件，名称、内容可自定义

    - 定义小部件配置信息：在 res/xml/ 下新建 appwidget_provider_info.xml，名称随意，添加：

      > initialLayout：指小工具使用的初始化布局
      >
      > minHeight、minWidth：小工具最小尺寸
      >
      > updatePeriodMillis：小工具自动更新周期，单位亳秒

      ```xml
      <?xml version="1.0" encoding="utf-8"?>
      <appwidget-provider xmlns:androld="http://schemas.android.com/apk/res/android"
                          android:initialLayout="@layout/widget"
                          android:minHeight="84dp"
                          android:minWidth="84dp"
                          android:updatePeriodMillis="86400000">
      </appwidget-provider>
      ```

    - 定义小部件的实现类：该类需继承 AppWidgetProvider

      > 小部件上显示一张图片，单击后图片旋转一周
      >
      > 小部件被添加到桌面后通过 RemoteViews 加载布局文件，当小部件被单击后旋转效果通过不断更新 RemoteViews 实现
      >
      > 桌面小部件初始化界面、后续更新界面都必须使用 RemoteViews 完成

      ```java
      public class MyAppWidgetProvider extends AppwidgetProvider{
          public static final String TAG = "MyAppWidgetProvider";
          public static final String CLICK_ACTION = "com.hzy.chapter.action.CLICK";
          public MyAppWidgetProvider(){
              super();
          }
          
          @Override 
          public void onReceive(final Context context, Intent intent){
              super.onReceive(context, intent);
              //判断是自己的action,做自己的事情，如小部件被单击后干什么，这里是动画效果
              if (intent.getAction().equals(CLICK_ACTION)){
                  Toast.makeText(context,"clicked it",Toast.LENGTH_SHORT).show();
                  new Thread (new Runnable(){
                      @Override 
                      public void run(){
                          Bitmap srcbBitmap = BitmapFactory.decodeResource (context.getResources(), R.drawable.icon);
                          AppWidgetManager appWidgetManager = AppWidgetManager.getInstance(context);
                          for(int i=0;i<37;i++){
                              float degree = (1 * 10) % 360;
                              RemoteViews remoteViews = new RemoteViews (context.getPackageName(), R.layout.widget);
                              remoteViews.setImageViewBitmap(R.id.imageView1, rotateBitmap (context, srcbBitmap, degree));
                              Intent intentClick = new Intent();
                              intentClick.setAction(CLICK_ACTION);
                              PendingIntent pendingIntent = PendingIntent.getBroadcast (context, 0, intentClick, 0);
                              remoteViews.setOnClickPendingIntent(R.id.imageView1, pendingIntent);
                              appWidgetManager.updateAppWidget(new ComponentName (context, MyAppWidgetProvider.class), remoteViews);
                              SystemClock.sleep(30);
                          }
                      }
                  }).start();
              }
          }
          //桌面小部件每次更新时都调用一次该方法
          @Override
          public void onUpdate(Context context, AppWidgetManager appWidgetManager, int[] appWidgetIds){
              super.onUpdate(context, appWidgetManager, appWidgetIds);
              final int counter = appWidgetIds.length;
              for(int i=0;i<counter;i++){
                  int appwidgetId = appWidgetIds[i];
                  onWidgetUpdate(context, appwidgetManager, appWidgetId);
              }
          }
                  
          //桌面小部件更新
          private void onWidgetUpdate(Context context, AppWidgetManager appWidgeManger, int appWidgetId){
              RemoteViews remoteViews = new RemoteViews(context.getPackageName(), R.layout.widget);
              //桌面小部件单击事件发送的Intent广播
              Intent intentClick = new Intent();
              intentClick.setAction(CLICK_ACTION);
              PendingIntent pendingIntent = PendingIntent.getBroadcast(context, 0, intentClick, 0);
              remoteViews.setOnClickPendingIntent(R.id.imageViewl, pendingIntent);
              appWidgeManger.updateAppWidget(appWidgetId, remoteViews);
          }
          
          private Bitmap rotateBitmap(Context context, Bitmap srcbBitmap, float degree){
              Matrix matrix = new Matrix();
              matrix.reset();
              matrix.setRotate(degree);
              Bitmap tmpBitmap = B1 tmap.createBitmap(srcbBitmap, 0, 0, srcbBitmap.getWidth(), srcbBitmap.getHeight(), matrix, true);
              return tmpBitmap;
          }
      }
      ```
    
  - 在 AndroidManifest.xml 中声明小部件：本质是广播组件，必须注册
  
    > 第一个 Action 识别小部件单击行为，第二个作为小部件标识，如果不加，该 receiver 就不是一个桌面小部件且也无法出现在手机小部件列表里
  
      ```xml
      <receiver
                android:name=".MyAppwidgetProvider">
          <meta-data
                     android:name="android.appwidget.provider"
                     android:resource="@xml/appwidget_provider_info">
          </meta-data>
          <intent-filter>
              <action android:name="com.hzy.chapter.action.CLICK"/>
              <action android:name="android.appwidget.action.APPWIDGET_UPDATE"/>		</intent-filter>
      </receiver>
      ```
  
- AppWidgetProvider 方法会自动被 onReceive 在合适时间调用，当广播到来后，自动根据广播 Action 通过 onReceive 自动分发广播（调用下面方法）
  
    - onEnable：窗口小部件第一次添加到桌面时调用，可添加多次但只在第一次调用
    - onUpdate：小部件被添加时或每次小部件更新时都会调用，更新时机由 updatePeriodMillis 指定
    - onDeleted：每次删除桌面小部件调用
    - onDisabled：当最后一个该类型的桌面小部件被删除时调用
  - onReceive：广播内置方法，用于分发具体事件给其他方法
  
    ```java
    public void onReceive (Context context, Intent intent){
        //防止恶意更新广播（不是真正安全问题，只是过滤bad广播，这样子类不太可能崩溃）
        String action = intent.getAction();
        if (AppWidgetManager.ACTTON_APPWIDGET_UPDATE.equals(action)){
            Bundle extras = intent.getExtras();
            if(extras != nul1){
                int[] appWidgetIds = extras.getIntArray (AppWidgetManager.EXTRA_APPWIDGET_IDS);
                if (appWidgetIds != null && appwidgetIds.length > 0){
                    this.onUpdate(context, AppwidgetManager.getInstance(context), appWidgetIds);
                }
            }
        }else if(AppwidgetManager.ACTION_APPWIDGET_DELETED.equals(action)){
            Bundle extras = intent.getExtrasO;
            if (extras != null && extras.containsKey (AppWidgetManager.EXTRAAPPNIDGET_ID)){
                final int appWidgetId = extras.getInt (AppWidgetManager.EXTRAAPPwIDGET_ID);
                this.onDeleted(context, new int[]{appWidgetId});
            }
        }else if(AppWidgetManager.ACTION_APPWIDGET_OPTIONS_CHANGED.equals(action)){
            Bundle extras = intent.getExtras();
            if (extras != null && extras.containsKey(AppWidgetManager.EXTRAAPPWIDGET_ID)  && extras.containsKey(AppwidgetManager.EXTRA_APPWIDGET_OPTIONS)) {
                int appwidgetId = extras.getInt(AppWidgetManager.EXTRAJAPPWIDGET_ID);
                Bundle widgetExtras = extras.getBundle(AppwidgetManager.EXTRAAPPWIDGET_OPTIONS);
                this.onAppwidgetoptionsChanged(context, AppWidgetManager.getinstance(context), appWidgetId, widgetExtras);
            }
        }else if(ApphidgetManager.ACTION_APPWIDGET_ENABLED.equals(action)){
            this.onEnabled(context);
        }else if(AppWidgetManager.ACTION_APPWIDGET_DISABLED.equals(action)){
            this.onDisabled(context);
        }else if(AppwidgetManager.ACTION_APPWIDGET_RESTORED.equals(action)){
            Bundle extras = intent.getExtras();
            if (extras != null){
                int[] oldIds = extras.getIntArray(AppWidgetManager.EXTRA_APPWIDGET_OLD_IDS);
                int[] newIds = extras.getIntArray(AppwidgetManager.EXTRAAPPWIDGET_IDS);
                if (oldIds != null && oldIds.length > 0){
                    this.onRestored(context, oldIds, newIds);
                    this.onUpdate(context, AppWidgetManager.getInstance(context), newIds);
                }
            }
        }
    }
    ```

#### 5.2、PendingIntent

- PendingIntent：处于 pending 状态的意图，pending 状态是待定、等待、即将发生

  - 和 Intent 的区别：PendingIntent 在将来某时发生，Intent 立刻发生

  - 使用场景：给 RemoteViews 添加单击事件，RemoteViews 运行在远程进程中，无法直接向 View 那样通过 setOnClickListener 设置单击事件，PendingIntent 通过 send、cancel 发送、取消待定 Intent

  - 支持三种待定意图：启动 Activity、Service、发送广播，对应三个接口方法：

    - 第二个参数 requestCode：PendingIntent 发送方请求码，多数情况设 0，会影响 flags 效果
    - 第四个参数 flags：常见类型：FLAG_ONE_SHOT、FLAG_NO_CREATE、FLAG_CANCEL_CURRENT、FLAG_UPDATE_CURRENT
    - PendingIntent 匹配规则：如果两 PendingIntent 内部 Intent 相同且 requestCode 相同，其相同
    - Intent 匹配规则：如果两 Intent 的 ComponentName 和 intent-filter 相同，其相同，Extras 不参与其匹配过程

    | static PendingIntent | getActivity(Context context, int requestCode, Intent intent, int flags)，获得一个 PendingIntent，该意图发生时相当Contcxt.startActivity(Intent) |
    | -------------------- | ------------------------------------------------------------ |
    | static PendingIntent | getService(Context context, int requestCode, Intent intent, int flags)，获得一个 PendingIntent，该意图发生时相当 Context.startService(Intent) |
    | static PendingIntent | getBroadcast(Context context, int requestCode, Intent intent, int flags)，获得一个 PendingIntent，该意图发生时相当 Context.sendBroadcast(Intent) |

  - flags

    > `manager.notify(id,notification);`：如果 id 每次不同，当 PendingIntent 不匹配时，不管采用何种标记位，通知间互不干扰；如果处于匹配状态时：
    >
    > 如果采用 FLAG_ONE_SHOT 标记位，后续通知的 PendingIntent 会和第一条通知保持完全一致，包括 Extras，单击任何一条通知后，剩下通知均无法打开，当所有通知都被清除后再次重复该过程
    >
    > 如果采用 FLAG_CANCEL_CURRENT 标记位，只有最新通知可打开，之前弹出的所有通知均无法打开
    >
    > 如果采用 FLAG_UPDATE_CURRENT 标记位，之前弹出的通知的 PendingIntent 会被更新，最终和最新的通知保持完全一致，包括 Extras，且这通知都可打开

    - FLAG_ONE_SHOT：PendingIntent 只能被使用一次，然后被自动 cancel，如果后续还有相同 PendingIntent，其 send 调用失败，对通知栏消息来说，同类通知只能使用一次，后续通知单击后无法打开
    - FLAG_NO_CREATE：PendingIntent 不主动创建，如果之前不存在，getActivity、getService、getBroadcast 直接返回 null，即获取 PendingIntent 失败，无法单独使用
    - FLAG_CANCEL_CURRENT：PendingIntent 如果已存在会被 cancel，然后系统创建一个新 PendingIntent，对通知栏消息来说，被 cancel 的消息单击后无法打开
    - FLAG_UPDATE_CURRENT：PendingIntent 如果已存在会被更新，即其 Intent 的 Extras 会被替换成最新的

#### 5.3、RemoteViews 内部机制

- 构造方法：`public RemoteViews(String packageName, int layoutId)`，第一个参数是当前应用包名，第二个参数 待加载布局文件，RemoteViews 不能支持所有 View 类型，支持类型：

  - Layout：FrameLayout、LinearLayout、RelativeLayout、GridLayout
  - View：AnalogClock、Button、Chronometer、ImageButton、ImageView、ProgressBar、TextView、ViewFlipper、ListView、GridView、StackView、AdapterViewFlipper、ViewStub
  - 如果在通知栏 RemoteViews 使用系统 EditText，通知栏消息无法弹出且抛出异常

- RemoteViews 无法直接访问 View 元素，必须通其提供的一系列 set 方法完成，因为 RemoteViews 在远程进程中显示，无法直接 findViewByld，部分 set 方法：

  | 方法名                                                       | 作用                                          |
  | ------------------------------------------------------------ | --------------------------------------------- |
  | setTextViewText(int viewId, CharSequence text)               | 设置TextView文本                              |
  | setTextViewTextSize(int viewld, int units, float size)       | 设置TextView字体大小                          |
  | setTextColor(int viewld, int color)                          | 设置TextView字体颜色                          |
  | setImageViewResource(int viewId, int srcld)                  | 设置ImageView图片资源                         |
  | setImageViewResource                                         | 设置ImageView图片                             |
  | setInt(int viewId, String methodiName, int value)            | 反射调用view对象参数类型为int的方法           |
  | setLong(int viewld, String methodName, long value)           | 反射调用view对象参数类型为long的方法          |
  | setBoolcan(int viewld, String methodName, boolean value)     | 反射调用View对象参数类型为boolean的方法       |
  | setOnClickPendingIntent(int viewld, PendingIntent pendingIntent) | 为View添加单击事件，事件类型只为PendingIntent |

- 内部机制

  - NotificationManager、AppWidgetManager 通过 Binder 分别和 SystemServer 进程的 NotificationManagerService、AppWidgetService 通信，通知栏、桌面小部件中的布局文件在 NotificationManagerService、AppWidgetService 中被加载，其运行在系统 SystemServer 中，和应用进程构成了跨进程通信场景

  - RemoteViews 通过 Binder 传到 SystemServer 进程，RemoteViews 实现 Parcelable 接口，可跨进程传输，系统根据 RemoteViews 包名等信息得到该应用资源，通过 LayoutInflater 加载 RemoteViews 布局文件，SystemServer 进程加载后的布局文件是个普通 View，不过相应用进程是个 RemoteViews

  - 系统对 View 执行一系列界面更新任务，通过 set 方法提交，其对 View 的更新不立刻执行，在 RemoteViews 内部记录所有更新操作，等 RemoteViews 被加载后才执行，RemoteViews 就可在 SystemServer 进程中显示，当需更新 RemoteViews 时要调用一系列 set 方法并通过 NotificationManager、AppWidgetManager 提交更新任务，具体操作在 SystemServer 进程中完成

  - 如果系统通过 Binder 支持所有 View 和 View 操作的话代价太大，因为 View 方法太多，大量 IPC 操作影响效率，解决：

    - 系统提供 Action，代表一个 View 操作，实现 Parcelable 接口

    - 系统将 View 操作封装到 Action 对象并将其跨进程传到远程进程，在远程进程中执行 Action 对象的具体操作，在应用中每调用一次 set 方法，RemnoteViews 中就添加一个对应 Action 对象，通过 NotifcationManager、AppWidgetManager 提交更新时，Action 对象就传到远程进程并依次执行

    - 远程进程通过 RemoteViews 的 apply 更新 View，其会遍历所有 Action 对象并调用其 apply，具体 View 更新操作由 Action 对象的 apply 完成

      > 好处：不需定义大量 Binder 接口，通过在远程进程中批量执行 RemoteViews 的修改操作避免大量 IPC 操作，提高程序性能

    ![安卓开发艺术探索_RemoteView内部实现](Image.assets\安卓开发艺术探索_RemoteView内部实现.jpg)

  - RemoteView#setTextViewText

  ```java
  public void setTextViewText (int viewId, CharSequence text){
      //参数：View的id，方法名，设置的文本
      setCharSequence (viewId, "setText", text);
  }
  
  public void setCharSequence(int viewId, String methodName, CharSequence value){
      //添加一个ReflectionAction对象，是个反射类型动作
      addAction(new ReflectionAction(viewId, methodName, ReflectionAction.CHAR_SEQUENCE, value));
  }
  
  private void addAction (Action a) {
      if(mActions == nu11){
          mActions = new ArrayList<Action>();
      }
      mActions.add(a);
      //更新内存使用统计信息
      a.updateMemoryUsageEstimate(mMemoryUsageCounter);
  }
  ```

  - RemoteView#apply
    - 通过 Layoutlnflater 加载 RemoteViews 中的布局文件，布局文件可通过 getLayoutld 获得
    - 加载后通过 performApply执行一些更新操作

  ```java
  public View apply (Context context, ViewGroup parent, OnClickHandler handler){
      RemoteViews rvToApply = getRemoteViewsToApply(context);
      View result;
      //...
      LayoutInflater inflater = (LayoutInflater) context.getSystemService(Context.LAYOUT_INFLATER_SERVICE);
      //克隆inflater，以便从正确context加载资源，且不向getSystemService返回的static版本添加filter
      inflater = inflater.cloneInContext(inflationContext);
      inflater.setFilter(this);
      result = inflater.inflate(rvToApply.getLayoutId(), parent, false);
      rvToApply.performApply(result, parent, handler);
      return result;
  }
  ```

  - RemoteView#performApply
    - 遍历 mActions 列表并执行每个 Action 对象的 apply

  ```java
  private void performApply (View v, ViewGroup parent, OnClickHandler handler) {
      if (mActions != null){
          handler = handler == null ? DEFAULT_ON_CLICK_HANDLER : handler;
          final int count = mActions.gize();
          for(int i=0;i<count;i++){
              Action a = mActions.get(i);
              a.apply(v, parent, handler);
          }
      }
  }
  ```

  - 调用 RemoteViews 的 set 方法时并不立刻更新界面，而通过 NotificationManager 的 notify 及 AppWidgetManager 的 updateAppWidget 才能更新，内部实现通过 RemoteViews 的 apply 及 reapply 加载或更新，其区别：
    - apply 会加载布局并更新界面，而 reApply 只更新界面
    - 通知栏、桌面小插件初始化界面时调用 apply，后续更新时调用 reapply
  - BaseStatusBar#updateNotificationViews
    - 表通知栏界面需更新时，会通过 RemoteViews 的 reapply 更新界面

  ```java
  private void updateNotificationViews (NotificationData.Entry entry, StatusBarNotification notification, boolean isHeadsUp){
      final Remoteviews contentview = notification.getNotification().contentView;
      final RemoteViews bigContentView = isHeadsUp ? notification.getNotification().headsUpContentView : notification.getNotification().bigContentView;
      final Notification publicVersion = notification.getNotification().publ1cvers1on;
      final RemoteViews publicContentView = publicVersion != null ? publicVersion.contentView : nu1ll;
      //重新应用RemoteView
      contentView.reapply(mContext, entry.expanded, mOnC1ickHandler);
  	//...
  }
  ```

  - AppWidgetHostView#updateAppWidget
    - 桌面小部件在更新界面时通过 RemoteViews 的 reapply 实现

  ```java
  mRemoteContext = getRemoteContext();
  int layoutId = remoteViews.getLayoutId();
  //如果旧view已准备好匹配active，且新布局匹配，尝试回收
  if (content == null && layoutId == mLayoutId) {
      try {
          remoteViews.reapply(mContext, mView, mOnClickHandler);
          content = mView;
          recycled = true;
          if(LOGD) Log.d(TAG, "was able to recycled existing layout");
      } catch (RuntimeException e){
          exception = e;
      }
  }
  //试试普通RemoteView inflation
  if (content == null) {
      try {
          content = remoteViews.apply(mContext, this, mOnClickHandler);
          if(LOGD) Log.d(TAG, "had to inflate new layout");
          } catch (RuntimeException e) {
              exception = e;
      }
  }
  ```

  - ReflectionAction
    - 对 View 的操作以反射方式调用
    - getMethod：根据方法名得到反射所需 Method 对象
    - 使用 ReflectionAction 的 set 方法有: setTextViewText、setBoolean、setLong、setDouble等
    - 除 ReflectionAction 还有其他 Action，如 TextViewSizeAction、ViewPaddingAction、SetOnClickPendingIntent 等

  ```java
  private final class ReflectionAction extends Action {
      ReflectionAction(int viewId, String methodName, int type, object value) {
          this.viewId = viewId;
          this.methodName = methodName;
          this.type = type;
          this.value = value;
      }
      //...
      @Override
      public void apply(View root, ViewGroup rootParent, OnClickHandler handler){
          final View view = root.findViewById(viewId);
          if (view == null) return;
          Class<?> param = getParameterType();
          if (param = nul1) {
              throw new ActionException("bad type:"+ this.type);
          }
          try {
              getMethod(view,this.methodName,param).invoke(view,wrapArg(this.value));
          } catch (ActionException e) {
              throw e;
          } catch (Exception ex) {
              throw new ActionException(ex);
          }
      }
  }
  ```

  - TextViewSizeAction
    - 不用反射，因为 setTextSize 有 2 个参数无法复用 ReflectionAction，ReflectionAction 反射调用只
      有一个参数

  ```java
  private class TextViewSizeAction extends Action{
      public TextViewSizeAction(int viewId, int units, float size){
          this.viewId = viewId;
          this.units = units;
          this.size = size;
      }
      //...
      @Override
      public void apply(View root, ViewGroup rootParent, OnC1ickHandler handler){
          final TextView target = (TextView) root.findViewById(viewId);
          if (target == null) return;
          target.setTextSize(units, size);
      }
      
      public String getActionName (){
          return "TextViewSizeAction";
      }
      int units;
      float size;
      public final static int TAG = 13;
  }
  ```

  - RemoteViews 只支持发起 PendingIntent，不支持 onClickListener 模式
  - setOnClickPendingIntent、setPendingIntentTemplate、setOnClickFillInIntent 的区别：
    - setOnClickPendingIntent 用于给普通 View 设置单击事件，不能给集合（ListView、StackView）中的 View 设置，因为开销较大
    - 如果要给 ListView、StackView 的 item 添加单击事件，须将 setPendingIntentTemplate 和 setOnClickilInIntent 组合使用

### 6、Drawable

> Drawable：一种可在 Canvas 上绘制的抽象概念，种类很多，常见颜色和图片都可是一个 Drawable
>
> 在 Android 设计中，Drawable 是抽象类，是所有 Drawable 对象的基类，每个具体 Drawable 都是其子类，如 ShapeDrawable、BitmapDrawable 等
>
> getIntrinsicWidth、getIntrinsicHeight 可获取 Drawable 内部宽高，但并不所有 Drawable 都有内部宽高

![安卓开发艺术探索_Drawable结构](Image.assets\安卓开发艺术探索_Drawable结构.jpg)

- 优点：使用简单，比自定义 View 成本低，非图片类型的 Drawable 占空间较小，减小 apk 大小

#### 6.1、Drawable 分类

- BitmapDrawable：表示一张照片，可直接引用原始图片，也可通过 XML 描述

  - android:src：图片资源 id

  - android:antialias：是否开启图片抗锯齿功能，开启后图片变平滑，一定程度降低图片清晰度，但可忽略

  - android:dither：是否开启抖动效果，当图片像素配置和手机屏幕像素配置不一致时可让高质量图片在低质量屏幕上还能保持较好显示效果，如图片色彩模式为 ARGB8888，设备屏幕支持 RGB555，抖动选项可让图片显示不会过于失真，在 Android 中创建的 Bitmap 一般选用 ARGB8888（ARGB 四个通道各占 8 位，一个像素大小 4 个字节，像素位数总和越高图像越逼真）

  - android:filter：是否开启过滤效果，当图片尺寸被拉伸或压缩时可保持较好显示效果

  - android:gravity：当图片小于容器尺寸时可对图片定位，选项通过 | 组合使用

    | 可选项            | 含义                                     |
    | ----------------- | ---------------------------------------- |
    | top               | 图片放在容器顶部，不改变图片大小         |
    | bottom            | 图片放在容器底部，不改变图片大小         |
    | left              | 图片放在容器左部，不改变图片大小         |
    | right             | 图片放在容器右部，不改变图片大小         |
    | center_vertical   | 图片竖直居中，不改变图片大小             |
    | fill_vertical     | 图片竖直方向填充容器                     |
    | center_horizontal | 图片水平居中，不改变图片大小             |
    | fill_horizontal   | 图片水平方向填充容器                     |
    | center            | 图片在水平和竖直方向居中，不改变图片大小 |
    | fill              | 图片在水平和竖直方向填充容器，默认值     |
    | clip_vertical     | 附加选项，竖直方向的裁剪，较少使用       |
    | clip_horizontal   | 附加选项，水平方向的裁剪，较少使用       |

  - android:mipMap：纹理映射，图像处理技术，默认 false

  - android:tileMode：平铺模式，disable 表示关闭平铺模式，默认值，开启平铺模式后 gravity 属性被忽略

    - repeat、miror、clamp 区别：都表示平铺模式

      > repeat：简单的水平和竖直方向上的平铺效果
      >
      > mirror：一种在水平和竖直方向上的镜面投影效果
      >
      > clamp：图片四周像素扩展到周围区域

      ![安卓开发艺术探索_Drawable平铺模式](Image.assets\安卓开发艺术探索_Drawable平铺模式.jpg)

  ```xml
  <?xml version="1.0" encoding="utf-8"?>
  <bitmap
          xmlns:android="http://schemas.android.com/apk/res/android"
          android:src="@[package:]drawable/drawable_resource"
          android:antialias=["true"|"false"]
          android:dither=["true"|"false"]
          android:filter=["true"|"false"]
          android:gravity=["top" | "bottom" | "left" | "right" | "center_vertical" | "fill_vertical" | "center_horizontal" | "fill_horizontal" | "center" | "fill" | "clip_vertical" | "clip_horizontal"]
          android:mipMap=["true"|"false"]
          android:tileMode=["disabled"|"clamp"|"repeat"|"mirror"]/>
  ```

- NinePatchDrawable：一张 .9 格式的图片，可自动根据所需宽高进行缩放并保证不失真，可通过 XML 描述 .9 图，BitmapDrawable 可代表一个 .9 图

  ```xml
  <?xml version="1.0" encoding="utf-8"?>
  <nine-patch
              xmIns:android="http://schemas.android.com/apk/res/android"
              android:src="@[package:]drawable/drawable_resource"
              android:dither=["true"|"false"]/>
  ```

- ShapeDrawable：通过颜色构造图形，可纯色，可渐变，<shape> 标签创建的 Drawable，实际是 GradientDrawable

  - android:shape：图形形状，rectangle（矩形，默认）、oval（椭圆）、line（横线）、ring（圆环）， line、ring 要通过 <stroke> 标签指定线宽和颜色等

    - ring

    | Value                    | Desciption                                                   |
    | ------------------------ | ------------------------------------------------------------ |
    | android:innerRadius      | 圆环内半径，android:innerRadiusRatio存在时以此为准           |
    | android:thickness        | 圆环厚度，外半径减内半径，android:thicknessRatio存在时以此为准 |
    | android:innerRadiusRatio | 内半径占整个Drawable宽度比例，默认9，内半径=宽度/9           |
    | android:thicknessRatio   | 厚度占整个Drawable宽度比例，默认3，厚度=宽度/3               |
    | android:useLevel         | 一般false，除非被当作LevelListDrawable使用                   |

  - <corners>：shape 四个圆角程度（px），只适用矩形

    - android:radius：为四个角同时设定相同圆角，优先级较低，会被其他属性覆盖
    - android:topLeftRadius：设定左上圆角
    - android:topRightRadius：设定右上圆角
    - android:bottomLeftRadius：设定左下圆角
    - android:bottomRightRadius：设定右下圆角

  - <gradient>：与 <solid> 排斥的，solid 纯色填充，gradient 渐变效果

    - android:angle：渐变角，默认0（从左到右），必须为45倍数
    - android:centerX：渐变中心点横坐标
    - android:centerY：渐变中心点纵坐标
    - android:startColor：渐变起始色
    - android:centerColor：渐变中间色
    - android:endColor：渐变结束色
    - android:gradientRadius：渐变半径，当 android:type= "radial" 时有效
    - android:useLevel：一般 false，当 Drawable 作为 StateListDrawable 时为 true
    - android:type：渐变类别，linear（线性渐变，默认)、radial(径向渐变)、sweep(扫描线渐变）

    ![安卓开发艺术探索_shape渐变](Image.assets\安卓开发艺术探索_shape渐变.png)

  - <solid>：纯色填充，通过 android:color 指定填充颜色
  - <stroke>：Shape 描边，如果 android:dashWidth、android:dashGap 有一个为 0，虚线效果不生效
    - android:width：描边宽度，越大 shape 边缘线越粗
    - android:color：描边颜色
    - android:dashWidth：虚线段宽度
    - android:dashGap：虚线段间隔
  - <padding>：包含 shape 的 View 的空白
  - <size>：shape 大小，但作为 View 背景时，shape 会被拉伸或缩小为 View 大小

  ```xml
  <?xml version="1.0"encoding="utf-8"?>
  <shape
         xmlns:android="http://schemas.android.com/apk/res/android"
         android:shape=["rectangle"|"oval"|"line"|"ring"]>
      <corners
               android:radius="integer"
               android:topLeftRadius="integer"
               android:topRightRadius="integer"
               android:bottomLeftRadius="integer"
               android:bottomRightRadius="integer"/>
      <gradient
                android:angle="integer"
                android:centerX="integer"
                android:centerY="integer"
                android:centerColor="integer"
                android:endcolor="color"
                android:gradientRadius="integer"
                android:startColor="color"
                android:type=["linear"|"radial"|"sweep"]
                android:useLevel=["true"|"false"]/>
      <padding
               android:left="integer"
               android:top="integer"
               android:right="integer"
               android:bottom="integer"/>
      <size
            android:width="integer"
            android:height="integer"/>
      <solid android:color="color"/>
      <stroke
              android:width="integer"
              android:color="color"
              android:dashwidth="integer"
              android:dashGap="integer"/>
  </shape>
  ```

- LayerDrawable：<layer-list>，表示 Drawable 集合，将不同 Drawable 放在不同层达到叠加效果

  - 可包含多个 item，每个 item 表示一个 Drawable，下 item 会覆盖上 item
  - android:top、android:bottom、android:left、android:right 分别表示 Drawable 相对 View 的上下左右偏移量，单位 px
  - 可通过 android:drawable 直接引用一个已有 Drawable 资源，也可在 item 自定义 Drawable
  - 默认 layer-list 所有 Drawable 会被缩放至 View 大小，bitmap 需使用 android:gravity 才能控制图片显示效果

  ```xml
  <?xml version="1.0" encoding="utf-8"?>
  <layer-list
              xmlns:android="http://schemas.android.com/apk/res/android">
      <item
            android:drawable="@[package:]drawable/drawable_resource"
            android:id="@[+][package:]id/resource_name"
            android:top="dimension"
            android:right="dimension"
            android:bottom="dimension"
            android:left="dimension"/>
  </layer-list>
  ```

- StateListDrawable：<selector>，表示 Drawable 集合，每个 Drawable 对应 View 一种状态，系统根据 View 状态选择 Drawable，主要用于设置可单击的 View（Button）

  - android:constantSize：StateListDrawable 固有大小不随其状态改变而改变，状态改变会导致 StateListDrawable 切换具体 Drawable，而不同 Drawable 不同固有大小，True 表示 StateListDrawable 固有大小不变，为内部所有 Drawable 固有大小最大值，默认 false 

  - android:dither：是否开启抖动效果，和 BitmapDrawable 类似，默认 true

  - android:variablePadding：是否随其状态改变而改变，true 表示会，false 表示 StateListDrawable 的  padding 是内部所有 Drawable 的 padding 最大值，默认

  - <item>：表示具体 Drawable，android:drawable 是 Drawable 资源id，剩下属性是 View 各种状态

    | 状态                   | 含义                                             |
    | ---------------------- | ------------------------------------------------ |
    | android:state_pressed  | 按下状态，如Button按下后仍没松开时的状态         |
    | android:state_focused  | View已获取焦点                                   |
    | android:state_selected | 用户选择View                                     |
    | android:state_checked  | 用户选中View，如CheckBox在选中和非选中状态间切换 |
    | android:state_enabled  | View处于可用状态                                 |

  ```xml
  <?xml version="1.0" encoding="utf-8"?>
  <selector xmlns:android="http://schemas.android.com/apk/res/android"
            android:constantSize=["true"|"false"]
            android:dither=["true"|"false"]
            android:variablePadding=["true"|"false"]>
      <item
            android:drawable="[package:]drawable/drawable_resource"
            android:state_pressed=["true"|"false"]
            android:state_focused=["true"|"false"]
            android:state_hovered=["true"|"false"]
            android:state_selected=["true"|"false"]
            android:state_cheekable=["true"|"false"]
            android:state_checked=["true"|"false"]
            android:state_enabled=["true"|"false"]
            android:state_activated=["txue"|"false"]
            android:state_window_focused=["true"|"false"]/>
  </selector>
  ```

- LevelListDrawable：<level-list>，表示 Drawable 集合，每个 Drawable 都有等级，不同等级切换对应 Drawable

  - 每个 item 表示一个 Drawable，且有对应等级范围，由 android:minLevel、android:maxLevel 指定，在其之间的等级会对应此 item 的 Drawable，可通过 Drawable 的 setLevel 设置等级切换具体 Drawable

  ```xml
  <?xml version="1.0" encoding="utf-8"?>
  <level-list
              xmins:android="http://schemas.android.com/apk/res/android">
      <item
            android:drawable="@drawable/drawable_resource"
            android:maxLevel="integer"
            android:minLevel="integer"/>
  </level-list>
  ```

- TransitionDrawable：<transition>，两个 Drawable 间淡入淡出效果

  ```xml
  <?xml version="1.0" encoding="utf-8"?>
  <transition xmlns:android="http://schemas.android.com/apk/res/android">
      <item
            android:drawable="@[package:]drawable/drawable_resource"
            android:id="@[+][package:]id/resource_name"
            android:top="dimension"
            android:right="dimension"
            android:bottom="dimension"
            android:left="dimension"/>
  </transition>
  ```

  - 将 view 背景设为 transition，通过 startTransition、reverseTransition 实现淡入淡出效果及逆过程

  ```java
  TextView textView = (TextView) findViewById (R.id.test_transition);
  TransitionDrawable drawable = (TransitionDrawable)textView.getBackground();
  drawable.startTransition(1000);
  ```

- InsetDrawable：<inset>，将其他 Drawable 内嵌到自己当中，可在四周留出一定间距

  - android:insetTop、android:insetBottom、 android:insetLeft、android:insetRight 分别表示顶部、底部、左边、右边内凹大小

  ```xml
  <?xml version="1.0" encoding="utf-8"?>
  <inset
         xmlns:android="http://schemas.android.com/apk/res/android"
         android:drawable="edrawable/drawable_resource"
         android:insetTop="dimension"
         android:insetRight="dimension"
         android:insetBottom="dimension"
         android:insetLeft="dimension"/>
  ```

- ScaleDrawable：<scale>，根据等级将指定 Drawable 缩放到一定比例

  - android:scaleGravity：等同 shape 的 android:gravity
  - android:scaleWidth、android:scaleHeight 分别表示对指定 Drawable 宽高的缩放比例，百分比形式
  - 等级 0 表示 ScaleDrawable 不可见，默认，可见等级不为 0，`ScaleDrawable.setLevel()` 设置等级

  ```xml
  <?xml version=1.0" encoding="utf-8"?>
  <scale
         xmlns:android="http://schemas.android.com/apk/res/android"
         android:drawable="@drawable/drawable_resource"
         android:scaleGravity= ["top" | "bottom" | "left" | "right" | "center_vertical" | "Eill_vertical" | "center_horizontal" | "fill_horizontal" | "center" | "fill" | "clip_vertical" | "clip_horizontal"]
         android:scaleHeight="percentage"
         android:scalewidth="pcrcentage"/>
  ```

  - ScaleDrawable#draw

  ```java
  public void draw (Canvas canvas){
      if(mScalestate.mDrawable.getLevel() != 0)
          mScalestate.mDrawable.draw(canvas);
  }
  ```

  - ScaleDrawable#onBoundsChange

    ```java
    protected void onBoundsChange (Rect bounds){
        final Rect r = mTmpRect;
        final boolean min = mScalestate.mUseIntrinsicSizeAsMin;
        int level = getLevel(0):
        int w = bounds.width();
        if (mscalestate.mScalewidth > 0){
            //mDrawable大小与等级及缩放比例关系，等级最大10000
            final int iw = min ? mScalestate.mDrawable.getIntrinsicWidth() : 0;
            w -= (int)((w - iw)*(10000 - level)*mScaleState.mScaleWidth / 10000);
            int h - bounds.height();
        }
        if (mScalestate.mScaleHeight > 0){
            final int ih = min ? mScalestate.mDrawable.getIntrinsicHeight() : 0;
            h -= (int)((h - ih)*(10000 - level)*mScalestate.mScaleHeight / 10000);
        }
        final int layoutDirection = getLayoutDirection();
        Gravity.apply(mscaleState.mGravity, w, h, bounds, r, layoutDirection);
        if(w > 0 && h > 0){
            mScalestate.mDrawable.setBounds (r.left, r.top, r.right, r.bottom);
        }
    }
    ```

- ClipDrawable：<clip>，根据当前等级裁剪另一个 Drawable，裁剪方向通过 android:clipOrientation、android:gravity 控制

  - clipOrientation：裁剪方向，有水平和竖直两方向

  - gravity 需和 clipOrientation 一起作用，各种选项可通过 | 组合

    | 选项              | 含义                                                         |
    | ----------------- | ------------------------------------------------------------ |
    | top               | 将内部Drawable放在容器顶部，不改变大小，如为竖直裁剪，从底部裁剪 |
    | bottom            | 将内部Drawable放在容器底部，不改变大小，如为竖直裁剪，从顶部裁剪 |
    | left，**默认**    | 将内部Drawable放在容器左边，不改变大小，如为水平裁剪，从右边裁剪 |
    | right             | 将内部Drawable放在容器右边，不改变大小，如为水平裁剪，从左边裁剪 |
    | center_vertical   | 使内部Drawable在容器竖直居中，不改变大小。如为竖直裁剪，从上下同时裁剪 |
    | fill_vertical     | 使内部Drawable在竖直方向填充容器。如为竖直裁剪，仅当ClipDrawable等级为0（完全裁剪，不可见）时，才能裁剪 |
    | center_horizontal | 使内部Drawable在容器水平居中，不改变大小。如为水平裁剪，从左右同时裁剪 |
    | fill_horizontal   | 使内部Drawable在水平方向填充容器。如为水平裁剪，仅当ClipDrawable等级为0时，才能裁剪 |
    | center            | 使内部Drawable在容器水平竖直居中，不改变大小。如为竖直裁剪，从上下同时裁剪；如为水平裁剪，从左右同时裁剪 |
    | fill              | 使内部Drawable在水平、竖直方向同时填充容器。仅当ClipDrawable等级为0时，才能裁剪 |
    | clip_vertical     | 附加选项，竖直方向的裁剪，较少使用                           |
    | clip_horizontal   | 附加选项，水平方向的裁剪，较少使用                           |

  - 等级范围 0-10000，8000 表示裁掉 20% 区域

  ```xml
  <?xml version="1.0" encoding="utf-8"?>
  <clip
        xmlns:android="http://schemas.android.com/apk/res/android"
        android:drawable="@drawable/drawable_resource"
        android:clipOrientation=["horizontal"|"vertical"]
        android:gravity=["top" | "bottom" | "left" | "right" | "center_vertical" | "fi11_vertical" | "center_horizontal" | "fill horizontal" | "center" | "fill" | "clip_vertical" | "clip horizontal"] />
  ```

- 自定义 Drawable：重写 draw，无法在 XML 中使用

### 7、Android 动画

#### 7.1、View 动画

- 种类：Animation 四个子类，可用 XML 定义，也可代码

  |    名称    |    标签     |        子类        | 效果           |
  | :--------: | :---------: | :----------------: | -------------- |
  |  平移动画  | <translate> | TranslateAnimation | 移动View       |
  |  缩放动画  |   <scale>   |   ScaleAnimation   | 放大、缩小View |
  |  旋转动画  |  <rotate>   |  RotateAnimation   | 旋转View       |
  | 透明度动画 |   <alpha>   |   AlphaAnimation   | 改变View透明度 |

- 创建动画 XML 文件，路径：res/anim/filename.xml，既可是单动画，也可由一系列动画组成

  - <set>：动画集合，对应 AnimationSet 类，可包含若干动画，内部可嵌套其他动画集合
    - android:interpolator：动画集合采用的插值器，影响动画速度，如非匀速动画，默认 @android:anim/accelerate_decelerate_interpolator（加速减速插值器）
    - android:shareInterpolator：集合动画是否和集合共享同一插值器，如集合不指定插值器，子动画需单独指定所需插值器或使用默认值
  - <translate>：平移动画，对应 TranslatcAnimation 类，使 View 在水平和竖直方向平移
    - android:fromXDelta：x 起始值，如 0
    - android:toXDelta：x 结束值，如 100
    - android:fromYDelta：y 起始值
    - android:toYDelta：y 结束值
  - <scale>：缩放动画，对应 ScaleAnimation，使 View 放大或缩小
    - android:fromXScale：水平方向缩放起始值，如 0.5
    - android:toXScale：水平方向缩放结束值，如 1.2
    - android:fromYScale：竖直方向缩放起始值
    - android:toYScale：竖直方向缩放结束值
    - android:pivotX：缩放轴点 x 坐标，影响缩放效果
    - android:pivotY：缩放轴点 y 坐标，影响缩放效果
  - <rotate>：旋转动画，对应 RotateAnimation，使 View 旋转
    - android:fromDegrees：旋转开始角度，如 0
    - android:toDegrees：旋转结束角度，如 180
    - android:pivotX：旋转轴点 x 坐标
    - android:pivotY：旋转轴点 y 坐标
  - <alpha>：透明度动画，对应 AlphaAnimation，改变 View 透明度
    - android:fromAlpha：透明度起始值，如 0.1
    - android:toAlpha：透明度结束值，如 1
  - android:duration：动画持续时间
  - android:fillAfter：动画结束后 View 是否停在结束位置，true 停留，false 不停留

  ```xml
  <?xml version="1.0" encoding-"utf-8"?>
  <set xmIns:android="http://schemas.android.com/apk/res/android"
       android:interpolator="@[package:]anim/interpolator_resource"
       android:shareInterpolator=["true"|"false"]>
      <alpha
             android:fromAlpha="float"
             android:toAlpha="float"/>
      <scale
             android:fromXScale="float"
             android:toXScale="float"
             android:fromYScale="float"
             android:toYScale="float"
             android:pivotX="float"
             android:pivotY="float"/>
      <translate
                 android:fromXDelta="float"
                 android:toXDelta="float"
                 android:fromYDelta="float"
                 android:toYDelta="float"/>
      <rotate
              android:fromDegrees="float"
              android:toDegrees="float"
              android:pivotX="float"
              android:pivotY="float"/>
      <set>
          <!--...-->
      </set>
  </set>
  ```
  - 代码创建

  ```java
  AlphaAnimation alphaAnimation = new AlphaAnimation(0, 1);
  alphaAnimation.setDuration(300);
  mButton.startAnimation(alphaAnimation);
  ```

- 监听 View 动画：

  ```java
  public static interface AnimationListener{
      void onAnimationStart(Animation animation);
      void onAnimationEnd(Animation animation);
      void onAnimationRepeat(Animation animation);
  }
  ```

- 自定义 View 动画：继承 Animation 抽象类，重写 initialize、applyTransformation，在 initialize 做些初始化工作，applyTransformation 进行矩阵变换，可用 Camera 简化矩阵变换

#### 7.2、帧动画

> 帧动画：顺序播放一组预先定义的图片。系统提供类 AnimationDrawable

- 使用，容易 OOM

  ```xml
  //res/drawable/frame_animation.xml
  <?xml version="1.0" encoding="utf-8"?>
  <animation-list xmlns:android="http://schemas.android.com/apk/res/android"
                  android:oneshot="false">
      <item android:drawable="@drawable/image1" android:duration="500"/>
      <item android:drawable="@drawable/image2" android:duration="500"/>
      <item android:drawable="@drawable/image3" android:duration="500"/>
  </animation-list>
  ```

  - 作为 view 背景

  ```java
  Button mButton = (Button)findViewById(R.id.button1);
  mButton.setBackgroundResource(R.drawable.frame animation);
  AnimationDrawable drawable = (AnimationDrawable) mButton.getBackground();
  drawable.start();
  ```

#### 7.3、view 动画特殊使用场景

- LayoutAnimation：作用于 ViewGroup，为其指定动画， 当其子元素出场时具有该动画效果，使用：

  - 定义 LayoutAnimation

    - android:delay：子元素开始动画的时间延迟，如入场动画时间周期 300ms，0.5 表示每个子元素都需延迟 150ms 才能播放入场动画
    - android:animationOrder：子元素动画顺序，normal（顺序）、reverse（逆向）、random（随机）
    - android:animation：为子元素指定具体入场动画

    ```xml
    //res/anim/anim_layout.xml
    <layoutAnimation
                     xmlns:android="http://schemas.android.com/apk/res/android"
                     android:delay="0.5"
                     android:animationOrder="normal"
                     android:animation="@anim/anim_item"/>
    ```

  - 为子元素指定具体入场动画

    ```xml
    //res/anim/anim_item.xml
    <?xml version="1.0" encoding="utf-8"?>
    <set xmlns:android="http://schemas.android.com/apk/res/android"
         android:duration="300"
         android:interpolator="eandroid:anim/accelerate_interpolator"
         android:shareInterpolator="true">
        <alpha
               android:fromAlpha="0.0"
               android:toAlpha="1.0"/>
        <translate
                   android:fromXDelta="500"
                   android:toXDelta="O"/>
    </set>
    ```

  - 为 ViewGroup 指定 android:layoutAnimation

    - xml 指定

    ```xml
    <ListView
              android:layoutAnimation="@anim/anim_layout"
              .../>
    ```

    - LayoutAnimationCotoller 实现

    ```java
    Listview listview = (ListView) layout.findViewById(R.id.list);
    Animation animation = AnimationUtils.loadAnimation(this, R.anim.anim_item);
    LayoutAnimationController controller = new LayoutAnimationController(animation);
    controller.setDelay(0.5f);
    controller.setOrder(LayoutAnimationController.ORDER_NORMAL);
    listView.setLayoutAnimation (controller);
    ```

- Activity 切换效果：有默认效果，自定义用 `overridePendingTransition(int enterAnim, int exitAnim)`，其需在 `startActivity(Intent)` 或 `finish()` 后被调用才能生效

  - 参数

    - enterAnim：Activity 被打开时所需动画资源 id
    - exitAnim：Activity 被暂停时所需动画资源 id

  - 启动 Activity 时添加自定义切换效果：

    ```java
    Intent intent = new Intent (this, TestActivity.class);
    startActivity(intent);
    //必须在startActivity后
    overridePendingTransition(R.anim.enter_anim, R.anim.exit_anim);
    ```

  - Activity 退出时指定切换效果

    ```java
    @Override
    public void finish(){
        super.finish();
        overridePendingTransition(R.anim.enter_anim, R.anim.exit_anim);
    }
    ```

  - Fragment 也可添加切换动画，Fragment 在 API11 引入，为兼容性需使用 support-v4 兼容包，通过
    FragmentTransaction 的 setCustomAnimations 添加切换动画，需 View 动画

#### 7.4、属性动画

> 属性动画 API11 加入，可对任何对象动画，甚至可无对象，动画默认时间间隔 300ms，默认帧率 10ms/帧
>
> 可用开源动画库 nineoldandroids 兼容老版本，内部通过代理 View 动画实现

- 使用

  - ValueAnimator、ObjectAnimator（继承自 ValueAnimator）、AnimatorSet（动画集合）

  - 改变对象 translationY 属性，沿 Y 轴向上平移：`objectAnimator.ofFloat(myobject, "translationY", -myObject.getHeight()).start()` 

  - 改变对象背景色，3 秒内从 0xFFFF8080 到 0xFF8080FF 渐变，动画无限循环且有反转效果

    ```java
    ValueAnimator colorAnim = objectAnimator.ofInt(this, "backgroundColor",
    /*Red*/0xFFFF8080, /*Blue*/0xFF8080FF);
    colorAnim.setDuration(3000);
    colorAnim.setEvaluator(new ArgbEvaluator());
    colorAnim.setRepeatCount(ValueAnimator.INFINITE);
    colorAnim.setRepeatMode(ValueAnimator.REVERSE);
    colorAnim.start();
    ```

  - 动画集合，5 秒内对 View 旋转、平移、缩放、透明度改变

    ```java
    AnimatorSet set = new AnimatorSet();
    set.playTogether(
        ObjectAnimator.ofFloat (myView, "rotationX", 0, 360),
        ObjectAnimator.ofFloat (myView, "rotationY", 0, 180),
        ObjectAnimator.ofFloat (myView, "rotation", 0, -90),
        ObjectAnimator.ofFloat (myView, "translationX", 0, 90),
        objectAnimator.ofFloat (myView, "translationY", 0, 90),
        objectAnimator.ofFloat (myView, "scalex", 1, 1.5f),
        ObjectAnimator.ofFloat (myView, "scaleY", 1, 0.5f),
        ObjectAnimator.ofFloat (myView, "alpha", 1, 0.25f, 1)
    );
    set.setDuration(5*1000).start();
    ```

  - XML 定义

    - 可以定义 ValueAnimator、ObjectAnimator、AnimatorSet

    - <set>：对应 AnimatorSet

      > android:ordering：together 表示动画集合中子动画同时播放，默认，sequntially 表示动画集合中子动画按前后顺序依次播放

    - <animator>：对应 ValueAnimator

    - <objectAnimator>：对应 ObjectAnimator

      > 比<animator>多一个android:propertyName
      >
      > android:propertyName：属性动画作用对象的属性名
      >
      > android:duration：动画时长
      >
      > android:valueFrom：属性起始值
      >
      > android:valueTo：属性结束值
      >
      > android:startOffset：动画开始后延迟时间播放
      >
      > android:repeatCount：动画重复次数，默认 0，-1 无限循环
      >
      > android:repeatMode：动画重复模式，repeat、reverse 分别表示连续重复、逆向重
      > 复（会倒着播放）
      >
      > android:valueType：android:propertyName 指定属性的类型，intType、floatType 分别表示整型、浮点型，如果 android:propertyName 指定属性是颜色，不需指定 android:valueType，系统自动处理

    ```xml
    <set
         android:ordering=["together"|"sequentially"]>
        <objectAnimator
                        android:propertyName="string"
                        android:duration="int"
                        android:valueFrom="float | int | color"
                        android:valueTo="float | int | color" 
                        android:startoffset="int"
                        android:repeatCount="int"
                        android:repeatMode=["repeat"|"reverse"]
                        android:valueType=["intType"|"floatType"]/>
        <animator
                  android:duration="int"
                  android:valueFrom="float | int | color"
                  android:valueTo="float | int | color"
                  android:startOffset="int"
                  android:repeatCount="int"
                  android:repeatMode=["repeat"|"reverse"]
                  android:valueType=["intType"|"floatType"]/>
        <set>
            <!--...-->
        </set>
    </set>
    ```

    - 代码使用

    ```java
    AnimatorSet set = (AnimatorSet) AnimatorInflater.loadAnimator(myContext, R.anim.property_animator);
    set.setTarget(mButton);
    set.start();
    ```

- 插值器和估值器
  - TimeInterpolator：时间插值器，根据时间流逝百分比计算当前属性值改变百分比，系统预置 LinearInterpolator（线性插值器：匀速动画）、AccelerateDecelerateInterpolator（加速减速插值器：两头慢中间快）、DecelerateInterpolator（减速插值器：动画越来越慢）

    - LinearInterpolator

    ```java
    public class LinearInterpolator implements Interpolator {
        public LinearInterpolator() {}
        public LinearInterpolator(Context context, AttributeSet attrs) {}
        public float getInterpolation(float input) {
            //返回值和输入一样
            return input;
        }
    }
    ```

  - TypeEvaluator：类型估值算法，根据当前属性改变百分比计算改变后的属性值，系统预置
    IntEvaluator（针对整型属性）、FloatEvaluator（针对浮点型属性）、ArgbEvaluator（针对 Color 属性）

    - IntEvaluator

    ```java
    public class IntEvaluator implements TypeEvaluator<Integer> {
        //参数：估值小数、开始值、结束值
        public Integer evaluate (float fraction, Integer startvalue, Integer endValue) {
            int startInt = startvalue;
            return (int) (startInt + fraction * (endValue - startInt));
        }
    }
    ```

  - 自定义插值器、估值算法，其都是一个接口，且内部只有一个方法，自定义插值器需实现 Interpolator 或 TimeInterpolator，自定义估值算法需实现 TypeEvaluator

- 监听器：监听动画播放过程，主要有两个接口：AnimatorUpdateListener、AnimatorListener

  - AnimatorListener：监听动画开始、结束、取消、重复播放

    - AnimatorListenerAdapter：AnimatorListener 的适配器，可有选择地实现

    ```java
    public static interface AnimatorListener {
        void onAnimationStart (Animator animation);
        void onAnimationEnd (Animator animation);
        void onAnimationCancel (Animator animation);
        void onAnimationRepeat (Animator animation);
    }
    ```

  - onAnimationUpdate：动画由许多帧组成，每播放一帧， 其被调用一次

    ```java
    public static interface AnimatorUpdateListener {
        void onAnimationUpdate (ValueAnimator animation);
    }
    ```

- 条件，如对 object 属性 abc 做动画，要满足：

  - object 提供 setAbe 方法，如果动画时没传递初始值，还要提供 getAbc 方法，系统需获取 abc 属性初始值，否则程序 Crash

  - object 的 setAbe 对属性 abe 所做的改变能够反映出来，否则动画无效果

  - 解决：

    - 给对象加上 get、set 方法

    - 用一个类包装原始对象，间接为其提供 get、set 方法

    - 采用 ValueAnimator 监听动画过程，自己实现属性改变

      > ValueAnimator 本身不作用于任何对象，可对一个值做动画，监听其动画过程，在动画过程中修改对象属性值

      ```java
      private void performAnimate (final View target, final int start, final int end) {
          ValueAnimator valueAnimator = ValueAnimator.ofInt(1, 100);
          valueAnimator.addUpdateListener (new AnimatorUpdateListener(){
              //持有IntEvaluator对象，方便估值
              private IntEvaluator mEvaluator = new IntEvaluator();
              @Override
              public void onAnimationUpdate (ValueAnimator animator){
                  //获得当前动画进度值，整型，1~100
                  int currentValue = (Integer) animator.getAnimatedValue();
                  //获得当前进度占整个动画比例，浮点型，0~1
                  float fraction = animator.getAnimatedFraction();
                  //调用整型估值器，通过比例计算宽度，设给Button
                  target.getLayoutParams().width = mEvaluator.evaluate(fraction, start, end);
                  target.requestLayout();
              }
          });
          valueAnimator.setDuration(5000).start();
      }
      
      @Override
      public void onClick (View v){
          if (v == mButton) {
              performAnimate(mButton, mButton.getwidth(), 500);
          }
      }
      ```

- 工作原理：对象提供属性 set 方法，根据属性初始值和最终值，以动画效果多次调用 set 方法，随着时间推移，传递的值越来越接近最终值。如果动画时没有传递初始值，还要提供 get 方法获取属性初始值

  - ObjectAnimator#start
    - 判断如果当前动画、等待的动画、延迟的动画中有和当前动画相同的动画，把相同动画取消
    - 调用父类 `super.start()`，ObjectAnimator 继承 ValueAnimator

  ```java
  public void start() {
      //查看是否需取消任何当前活动的/挂起的animators
      AnimationHandler handler = sAnimationHandler.get();
      if (handler != null) {
          int numAnims = handler.mAnimations.size();
          for(int i=numAnims-1;i>=0;i--){
              if (handler.mAnimations.get(i) instanceof ObjectAnimator) {
                  objectAnimator anim = (ObjectAnimator) handler.mAnimations.get(i);
                  if (anim.mAutoCancel && hasSameTargetAndProperties(anim)) {
                      anim.cancel();
                  }
              }
          }
          numAnims = handler.mPendingAnimations.size();
          for(int i=numAnims-1;i>=0;i--){
              if (handler.mPendingAnimations.get(i) instanceof ObjectAnimator) {
                  ObjectAnimator anim = (ObjectAnimator) handler.mPendingAnimations.get(i);
                  if (anim.mAutoCancel && hasSameTargetAndProperties(anim)) {
                      anim.cancel();
                  }
              }
          }
          numAnims = handler.mDelayedAnims.size();
          for(int i=numAnims-1;i>=0;i--){
              if (handler.mDelayedAnims.get(i) instanceof ObjectAnimator) {
                  ObjectAnimator anim = (objectAnimator) handler.mDelayedAnims.get(i);
                  if (anim.mAutoCancel && hasSameTargetAndProperties(anim)) {
                      anim.cancel();
                  }
              }
          }
      }
      if (DBG) {
          for(int i=0;i<mValues.length;++i) {
              PropertyValuesHolder pvh = mValues[i];
          }
      }
      super.start();
  }
  ```

  - ValueAnimator#start
    - 属性动画需运行在有 Looper 的线程中
    - 最终调用 AnimationHandler 的 start 方法，AnimationHandler 不是 Handler，是 Runnable，其很快调到 JNI 层，其 run 方法被调用，涉及和底层交互

  ```java
  private void start (boolean playBackwards) {
      if (Looper.myLooper() == null) {
          throw new AndroidRuntimeException("Animators may only be run on Looper threads");
      }
      mPlayingBackwards = playBackwards;
      mCurrentIteration = 0;
      mPlayingState = STOPPED;
      mStarted = true;
      mStartedDelay = false;
      mPaused = false;
      updateScaledDuration(); //使比例因子从创建时起发生变化
      AnimationHandler animationHandler = getOrCreateAnimationHandler();
      animationHandler.mPendingAnimations.add(this);
      if (mStartDelay == 0) {
          //在实际开始运行动画前设置动画初始值
          setCurrentPlayTime(0);
          mPlayingState = STOPPED;
          mRunning = true;
          notifyStartListeners();
      }
      animationHandler.start();
  }
  ```

  - ValueAnimator#doAnimationFrame

  ```java
  final boolean doAnimationFrame (long frameTime) {
      if (mPlayingState == STOPPED){
          mPlayingState = RUNNING;
          if (mSeekTime < 0) {
              mStartTime = frameTime;
          } else{
              mStartTime = frameTime - mSeekTime;
              //现在开始播放，重置mSeekTime
              mSeekTime = -1;
          }
      }
      if (mPaused){
          if (mPauseTime < 0) {
              mPauseTime = frameTime;
          }
          return false;
      } else if (mResumed) {
          mResumed = false;
          if (mPauseTime > 0){
              //由动画暂停的持续时间偏移
              mStartTime += (frameTime - mPauseTime);
          }
      }
      //动画第一帧的帧时间可能早于开始时间，当前时间必须在开始时间或之后，以避免以负时间间隔设置帧动画，实践中非常罕见，只发生在seek倒退
      final long currentTime = Math.max(frameTime, mStartTime);
      return animationFrame(currentTime);
  }
  ```

  - animationFrame

  ```java
  void animateValue (float fraction){
      fraction = mInterpolator.getInterpolation(fraction);
      mCurrentFraction = fraction;
      int numvalues = mValues.length;
      for (int i=0;i<numValues;++i) {
          //计算每帧动画对应属性值
          mValues[i].calculatevalue(fraction);
      }
      if(mUpdateListeners !=null) {
          int numListeners = mUpdateListeners.size();
          for(int i=0;i<numListeners;++i){
              mUpdateListeners.get(i).onAnimationUpdate(this);
          }
      }
  }
  ```
  
  - ValuesHolder#setupValue
    - get 方法通过反射调用
  
  ```java
  private void setupValue (Object target, Keyframe kf){
      if(mProperty != null) {
          Object value = convertBack(mProperty.get(target));
          kf.setValue (value);
      }
      try{
          if(mGetter == null){
              Class targetClass = target.getClass();
              setupGetter(targetClass);
              if (mGetter == null) {
                  //已经记录错误,返回避免NPE
                  return;
              }
          }
          object value = convertBack (mGetter.invoke(target));
          kf.setValue(value);
      }catch (InvocationTargetException e){
          Log.e("PropertyvaluesHolder", e.toString());
      }catch (IllegalAccessException e){
          Log.e("PropertyvaluesHolder", e.toString());
      }
  }
  ```
  
  - PropertyValuesHolder#setAnimatedValue
    - 动画下一帧到来时，setAnimatedValue 将新属性值设给对象，调用其 set 方法，通过反射调用
  
  ```java
  void setAnimatedvalue(Object target){
      if(mProperty != null){
          mProperty.set(target, getAnimatedvalue());
      }
      if (mSetter != null){
          try {
              mTmpvalueArray[0] = getAnimatedvalue();
              mSetter.invoke(target, mTmpValueArray);
          }catch (InvocationTargetException e){
              Log.e("PropertyvaluesHolder", e.toString());
          }catch(IllegalAccessException e){
              Log.e("PropertyvaluesHolder", e.toString());
          }
      }
  }
  ```

#### 7.5、使用注意

- OOM：主要在帧动画，当图片数量较多且图片较大时易出现
- 内存泄露：属性动画有一类无限循环动画，需在 Activity 退出时及时停止，否则导致 Activity 无法释放造成内存泄露，View 动画不存在此问题
- 兼容性问题：动画在 3.0 以下系统有兼容性问题，某些特殊场景可能无法工作
- View 动画问题：View 动画不真正改变 View 状态，有时动画完成后 View 无法隐藏（`setVisibility(View.GONE)` 失效），调用 `view.clearAnimation()` 清除 View 动画即可
- 不要使用 px：动画中尽量使用 dp，使用 px 在不同设备有不同效果
- 动画元素交互：view 移动后，在 3.0 以前系统，View 动画和属性动画的新位置均无法触发单击事件，老位置仍可触发单击事件
- 硬件加速：使用动画建议开启硬件加速，提高动画流畅性

### 8、Window & WindowManager

> Window：抽象类，具体实现是 PhoneWindow，创建通过 WindowManager 完成，具体实现位于 WindowManagerService 中
>
> WindowManager、WindowManagerService 交互是 IPC 过程
>
> Android 所有视图都通过 Window 呈现，Activity、Dialog、Toast 的视图实际上都附加在 Window 上

#### 8.1、Window & WindowManager

- WindowManager 添加 Window

  - 将 Button 添加到屏幕坐标为(100，300）位置

  ```java
  mFloatingButton = new Button(this);
  mFloatingButton.setText("button");
  mLayoutParams = new windowManager.LayoutParams(LayoutParams.WRAP_CONTENT, LayoutParams.WRAP_CONTENT, 0, 0, PixelFormat.TRANSPARENT);
  mLayoutParams.flags = LayoutParams.FLAG_NOT_TOUCH_MODAL | LayoutParams.FLAG_NOT_FOCUSABLE | LayoutParams.FLAG_SHOW_WHEN_LOCKED;
  mLayoutParams.gravity = Gravity.LEFT | Gravity.TOP;
  mLayoutParams.x = 100;
  mLayoutParams.y = 300;
  mWindowManager.addView(mFloatingButton, mLayoutParams);
  ```

  - WindowManager.LayoutParams 的 Flags 参数：Window 属性
    - FLAG_NOT_FOCUSABLE：Window 不需获取焦点、接收各种输入事件，会同时启用 FLAG_NOT_TOUCH_MODAL，最终事件直接传给下层有焦点的 Window
    - FLAG_NOT_TOUCH_MODAL：系统将当前 Window 区域外的单击事件传给底层 Window，区域内的自己处理，一般都需开启，否则其他 Window 无法收到单击事件
    - FLAG_SHOW_WHEN_LOCKED：让  Window 显示在锁屏界面上
  - Window 分层：每个 Window 都有对应 z-ordered，层级大会覆盖在层级小的 Window 上
    - 系统层级有很多值，如采用 TYPE_SYSTEM_ERROR，只需为 type 参数指定该层级即可：`mLayoutParams.type = LayoutParams.TYPE_SYSTEM_ERROR`，同时声明权限：`<uses-permission android:name="android.permission.SYSTEM_ALERT_WINDOW"/>`，因为系统 Window 需检查权限
  - WindowManager.LayoutParams 的 Type 参数：Window 类型
    - 应用 Window：对应一个 Activity，层级范围 1~99
    - 子 Window：不能单独存在，需附属在特定父 Window 中，如 Dialog，层级范围1000~1999
    - 系统 Window：需声明权限才能创建，如 Toast、系统状态栏，层级范围 2000~2999

- WindowManager 功能：添加 View、更新 View、删除 View，定义在 ViewManager 中，WindowManager 继承 ViewManager

  ```java
  public interface ViewManager{
      public void addView(View view,ViewGroup. LayoutParams params);
      public void updateViewLayout (View view, ViewGroup.LayoutParams params);
      public void removeview(View view);
  }
  ```

#### 8.2、Window 内部机制

> 每个 Window 对应一个 View 和 ViewRootImpl，Window 和 View 通过 ViewRootImpl 建立联系

- Window 的添加：通过 WindowManager 的 addView 实现，WindowManager 是接口，实现是 WindowManagerImpl 类

  - WindowManagerImpl
    - 没实现三大操作，交给 WindowManagerGlobal 处理（桥接模式），WindowManagerGlobal 以工厂形式向外提供自己的实例：`private final WindowManagerGlobalmGlobal = WindowManagerGlobal.getInstance()` 

  ```java
  @Override
  public void addview(View view, ViewGroup.LayoutParams params){
      mGlobal.addview(view, params, mDisplay, mParentWindow);
  }
  @Override
  public void updateViewLayout(View view, ViewGroup.LayoutParams params){
      mGlobal.updateviewLayout(view, params);
  }
  @Override
  public void removeView(View view){
      mGlobal.removeview(view, false);
  }
  ```

  - WindowManagerGlobal#addView

  ```java
  //1、检查参数是否合法，是则子Window还需调整些布局参数
  if (view == null){
      throw new IllegalArgumentException ("view must not be null");
  }
  if(display == null){
      throw new IllegalArgumentException ("display must not be null");
  }
  if (!(params instanceof WindowManager.LayoutParams)){
      throw new IllegalArgumentException("Params must be WindowManager.LayoutParams");
  }
  
  final windowManager.LayoutParams wparams = (windowManager.LayoutParams)params;
  if (parentwindow != null){
      parentwindow.adjustLayoutParamsForSubwindow(wparams);
  }
  
  //2、创建ViewRootImpl并将View添加到列表中，重要列表：
  //存储所有Window对应View
  private final ArrayList<View> mViews = new ArrayList<>();
  //存储所有Window对应ViewRootImpl
  private final ArrayList<ViewRootImpl> mRoots = new ArrayList<>();
  //存储所有Window对应布局参数
  private final ArrayList<WindowManager.LayoutParams> mParams = new ArrayList<>();
  //存储正被删除的View对象，已调用removeView但未完成的Window对象
  private final Arrayset<View> mDyingviews = new Arrayset<>();
  //addView将Window一系列对象添加到列表
  root = new ViewRootImpl(view.getContext(), display);
  view.setLayoutParams(wparams);
  mViews.add(view);
  mRoots.add(root);
  mParams.add(wparams);
  
  //3、通过ViewRootlmpl更新界面并完成Window添加过程
  //setView通过requestLayout完成异步刷新请求
  public void requestLayout() {
      if(!mHandlingLayoutInLayoutRequest){
          checkThread();
          mLayoutRequested = true;
          //View绘制入口
          scheduleTraversals();
      }
  }
  //通过WindowSession完成Window添加
  try{
      //mWindowSession类型是IWindowSession，Binder对象
      mOrigwindowType = mWindowAttributes.type;
      mAttachInfo.mRecomputeGlobalAttributes = true;
      collectViewAttributes();
      //真正实现类是Session，Window添加过程是IPC调用
      res = mWindowSession.addToDisplay(mWindow,mSeq,mWindowAttributes, getHostVisibility(),mDisplay.getDisplayId(),mAttachInfo.mContentInsets,mInputChannel);
  }catch (RemoteException e){
      mAdded = false;
      mView = null;
      mAttachInfo.mRootView = null;
      mInputChannel = null;
      mFallbackEventHandler.setView (null);
      unscheduleTraversals();
      setAccessibilityFocus(null,null);
      throw new RuntimeException ("Adding window failed",e);
  }
  //Session通过WindowManagerService实现Window添加，WindowManager内为每个应用保留一个单独Session
  public int addToDisplay(Iwindow window,int seq,WindowManager.LayoutParamsattrs,
  int viewVisibility,int displayld,Rect outContentInsets,InputChannel outInputChannel){
      return mService.addWindow(this, window, seg, attrs, viewVisibility, displayId,
  outContentInsets, outInputChannel);
  }
  ```

- Window 的删除：先通过 WindowManagerImpl 后，再通过 WindowManagerGlobal 实现

  - WindowManagerGlobal#removeView

    ```java
    public void removeView(View view, boolean immediate){
        if (view == null) {
            throw new IllegalArgumentException("view must not be null");
        }
        synchronized(mLock){
            //查找待删除View索引
            int index = findviewLocked(view, true);
            View curView = mRoots.get(index).getview();
            //进一步删除
            removeViewLocked(index, immediate);
            if (curView == view){
                return;
            }
            throw new IllegalStateException("Calling with view " + view + " but the ViewAncestor is attached to " + curView);
        }
    }
    ```

  - removeViewLocked

    - WindowManager 提供两种删除接口 removeView、removeViewImmediate，分别表示异步、同步删除，异步删除由 ViewRootImpl 的 die 方法完成，其发送一个请求删除消息立刻返回，这时 View 并无完成删除，所以会将其加到 mDyingViews 中（待删除 View 列表）

    ```java
    private void removeViewLocked (int index,boolean immediate){
        ViewRootImpl root = mRoots.get(index);
        view view = root.getview();
        if(view != null){
            InputMethodManager imm = InputMethodManager.getInstance();
            if (imm != null) {
                imm.windowDismissed(mViews.get(index).getwindowToken());
            }
        }
        boolean deferred = root.die(immediate);
        if(view != null){
            view.assignParent(null);
            if (deferred){
                mDyingviews.add (view);
            }
        }
    }
    ```

  - ViewRootImpl#die

    - 如果异步删除，发送 MSG_DIE 消息，ViewRootImpl 中 Handler 会处理并调用 doDie
    - 如果同步删除（立即删除），不发消息直接调用 doDie

    ```java
    boolean die (boolean immediate){
        //如果正处于遍历或damage过程中，确保立即执行
        //由DispatchedDetachedFromWindow执行将在返回时造成严重破坏
        if(immediate && !mIsInTraversal) {
            doDie();
            return false;
        }
        if(!mIsDrawing){
            destroyHardwareRenderer();
        }else {
            Log.e(TAG, "Attempting to destroy the window while drawing!\n" + " window=" + this +", title=" + mWindowAttributes.getTitle());
        }
        mHandler.sendEmptyMessage (MSG_DIE);
        return true;
    }
    ```

  - doDie 内部调用 dispatchDetachedFromWindow，真正删除逻辑在其实现：
    - 垃圾回收相关工作，如清除数据、消息、移除回调
    - 通过 Session 的 remove 删除 Window：`mWindowSession.remove(mWindow)`，IPC 过程，最终调用 WindowManagerService 的 removeWindow
    - 调用 View 的 dispatchDetachedFromWindow，内部调用 View 的 onDetachedFromWindow 及 onDetachedFromWindowInternal，onDetachedFromWindow 当 View 从 Window 中移除时被调用，可做些资源回收工作，如终止动画、停止线程等
    - 调用 WindowManagerGlobal 的 doRemoveView 刷新数据，包括 mRoots、mParams、mDyingViews，需将当前 Window 关联的这三类对象从列表中删除

- Window 的更新

  - WindowManagerGlobal#updateViewLayout

    - 更新 View 的 LayoutParams 替换老 LayoutParams
    - 更新 ViewRootImpl 的 LayoutParams，通过 ViewRootImpl 的 setLayoutParams 实现
    - ViewRootImpl 通过 scheduleTraversals 对 View 重新布局，包括测量、布局、重绘
    - ViewRootImpl 通过 WindowSession 更新 Window 视图，最终由 WindowManagerService 的relayoutWindow 实现，IPC 过程

    ```java
    public void updateviewLayout (View view, ViewGroup.LayoutParams params){
        if(view == null){
            throw new IllegalArgumentException("view must not be null");
        }
        if(!(params instanceof WindowManager.LayoutParams)){
            throw new TllegalArgumentException("Params must be WindowManager.LayoutParams");
        }
        final windowManager.LayoutParams wparams = (WindowManager.LayoutParams)params;
        view.setLayoutParams(wparams);
        synchronized(mLock){
            int index = findviewLocked(view, true);
            ViewRootImpl root = mRoots.get(index);
            mParams = remove(index);
            mParams.add(index, wparams);
            root.setLayoutParams(wparams, false);
        }
    }
    ```

#### 8.3、Window 创建过程

- Activity 的 window 创建过程

  - Activity 最终由 ActivityThread 的 performLaunchActivity 完成整个启动过程，其通过类加载器创建 Activity 实例对象，调用其 attach 为其关联运行过程中依赖的一系列上下文环境变量

  - Activity 的 attach 会创建 Activity 所属 Window 对象并为其设置回调接口，Window 对象的创建通过 PolicyManager 的 makeNewWindow 实现，Activity 实现 Window 的 Callback 接口，当 Window 接收到外界状态改变时会回调 Activity 方法，Callback 接口方法：

    - onAttachedToWindow、onDetachedFromWindow、 dispatchTouchEvent 等

  - Activity 的 Window 通过 PolicyManager 的工厂方法创建，其是策略类，实现的工厂方法在策略接口 IPolicy 中声明

    ```java
    public interface IPolicy{
        public window makeNewWindow(Context context);
        public LayoutInflater makeNewLayoutInflater(Context context);
        public windowManagerPolicy makeNewWindowManager();
        public FallbackEventHandler makeNewFallbackEventHandler(Contextcontext);
    }
    ```

  - PolicyManager 真正实现是 Policy 类，Window 具体实现是 PhoneWindowPolicy

  - Activity#setContentView

    - 具体实现交给 Window，Window 具体实现是 PhoneWindow

    ```java
    public void setContentView(int layoutResID) {
        getWindow ().setcontentview (layoutReSID);
        initWindowDecorActionBar();
    }
    ```

  - PhoneWindow#setContentView

    - 如果没有 DecorView，创建

      > 由 installDecor 完成，其通过 generateDecor 直接创建 DecorView，这时只是空白 FrameLayout

      ```java
      protected DecorView generateDecor(){
          return new DecorView(getContext(),-1);
      }
      ```

      > 初始化 DecorView 结构，PhoneWindow 通过 generateLayout 加载具体布局文件到 DecorView，具体布局文件和系统版本及主题有关

      ```java
      View in = mLayoutInflater.inflate(layoutResource, null);
      decor.addView(in, new ViewGroup.LayoutParams(MATCH_PARENT, MATCH_PARENT));
      mContentRoot = (ViewGroup)in;
      // ID_ANDROID_CONTENT对应ViewGroup是mContentParent
      ViewGroup contentParent = (ViewGroup) findViewById(ID_ANDROID_CONTENT);
      ```

    - 将 View 添加到 DecorView 的 mContentParent 中：`mLayoutInflater.inflate(layoutResID, mContentParent)`

    - 回调 Activity 的 onContentChanged 通知 Activity 视图已改变

      > onContentChanged 是空实现，可在子 Activity 中处理回调

      ```java
      final callback cb = getCallback();
      if (cb != null && !isDestroyed()){
          cb.onContentChanged();
      }
      ```

  - DecorView 被 WindowManager 添到 Window 中：ActivityThread 的 handleResumeActivity 先调用 Activity 的 onResume，再调用 Activity 的 makeVisible，其完成添加、显示两过程，Activity 视图才能被用户看到

    ```java
    void makeVisible(){
        if(!mwindowAdded){
            viewManager wm = getwindowManager();
            wm.addview(mDecor, getWindow().getAttributes());
            mwindowAdded = true;
        }
        mDecor.setVisibility(View.VISIBLE);
    }
    ```

- Dialog 的 window 创建过程

  - 创建 Window：通过 PolicyManager 的 makeNewWindow 完成，创建后对象是 PhoneWindow，和 Activity 的 Window 创建过程是一致

  - 初始化 DecorView 并将 Dialog 视图添到 DecorView 中：和 Activity 类似，通过 Window 添加指定布局文件

    ```java
    public void setContentview (int layoutResID){
        mwindow.setContentView(layoutResID);
    }
    ```

  - 将 DecorView 添到 Window 中并显示：

    - 在 Dialog 的 show 中通过 WindowManager 将 DecorView 添加到 Window 中

      ```java
      mWindowManager.addview(mDecor, l);
      mShowing = true;
      ```

  - 当 Dialog 被关闭时，通过 WindowManager 移除 DecorView：`mWindowManager.removeViewImmediate(mDecor)` 

  - Dialog 必须采用 Activity 的 Context，采用 Application 的报错，没有应用 token 导致，其一般只有 Activity 有，系统 Window 可不需 token

- Toast 的 window 创建过程

  - Toast 基于 Window 实现，Toast 定时取消功能采用 Handler，其有两类 IPC，一是 Toast 访问 NotificationManagerService（NMS），二是 NotificationManagerService 回调 Toast 的 TN 接口

  - Toast 属于系统 Window，内部视图由两种方式指定，一是系统默认样式，二是通过 setView 指定自定义 View，其都对应 Toast 的 View 类型内部成员 mNextView

  - Toast 提供 show、cancel 显示、隐藏 Toast，内部是 IPC 过程

    - 显示、隐藏 Toast 都通过 NMS 实现，其运行在系统进程中，只能通过远程调用
    - TN：Binder 类，Toast 和 NMS IPC 过程中，NMS 处理 Toast 请求时跨进程回调 TN 中方法，其运行在 Binder 线程池中，通过 Handler 将其切换到当前线程中（发送 Toast 请求所在线程），Toast 无法在无 Looper 线程中弹出
    - NMS 的 enqueueToast 方法第一个参数表示当前应用包名，第二表示远程回调，第三表示 Toast 时长，其先将 Toast 请求封装为 ToastRecord 对象并将其添到 mToastQueue 队列
    - mToastQueue：ArrayList，非系统应用最多同时存在 50 个 ToastRecord，防止 DOS (Denial of Service，拒绝服务)，否则如果连续弹出 Toast 会导致其他应用没机会弹出 Toast，对于其他应用系统拒绝服务

    ```java
    public void show(){
        if (mNextView == null){
            throw new RuntimeException("setview must have been called");
        }
        INotificationManager service = getservice();
        String pkg = mContext.getOpPackageName();
        TN tn = mTN;
        tn.mNextView = mNextView;
        try{
            //Toast显示过程
            service.enqueueToast(pkg, tn, mDuration);
        }catch (RemoteException e){
                // Empty
        }
    }
     
    public void cancel() {
        mTN.hide();
        try {
            getService().cancelToast(mContext.getPackageName(), mTN);
        } catch (RemoteException e) {
                // Empty
        }
    }
    ```

  - 当 ToastRecord 被添到 mToastQueue 中，NMS 通过 showNextToastLocked 显示 Toast

    - Toast 显示由 ToastRecord 的 callback 完成，其实际是 Toast 的 TN 对象的远程 Binder，最终被调用的 TN 中的方法运行在发起 Toast 请求的应用的 Binder 线程池中

    ```java
    void showNextToastLocked() {
        ToastRecord record = mToastQueue.get(0);
        while (record != null){
            if(DBG) Slog.d(TAG, "Show pkg=" + record.pkg + "callback=" + record.callback);
            try{
                record.callback.show();
                scheduleTimeoutLocked(record);
                return;
            }catch (RemoteException e){
                Slog.w(TAG, "Object died trying to show notification " + record.callback + " in package " + record.pkg);
                //从列表删除，让进程结束
                int index = mToastQueue.indexof(record);
                if(index >= 0) {
                    mToastQueue.remove(index);
                }
                keepProcessAliveLocked(record.pid);
                if(mToastQueue.size() > 0){
                    record = mToastQueue.get(0);
                }else{
                    record = null;
                }
            }
        }
    }
    ```

  - Toast 显示后，NMS 通过 scheduleTimeoutLocked 发送一个延时消息，延时取决于 Toast 时长，LONG_DELAY 3.5s，SHORT_DELAY 2s
  
    ```java
    private void scheduleTimeoutLocked(ToastRecord r){
        mHandler.removeCallbacksAndMessages(r);
        Message m = Message.obtain(mHandler, MESSAGE_TIMEOUT, r);
        long delay = r.duration == Toast.LENGTH_LONG ? LONG_DELAY : SHORT_DELAY;
        mHandler.sendMessageDelayed(m, delay);
    }
    ```
  
  - NMS 通过 canceToastLocked 隐藏 Toast 并将其从 mToastQueue 移除，如果 mToastQueue 有其他 Toast，NMS 继续显示其他 Toast，
    隐藏通过 ToastRecord 的 callback 完成，是 IPC 过程
  
  - TN#show、hide
  
    - mShow、mHide 是 Runnable，其分别调用 handleShow、handleHide
  
    ```java
    //将handleShow安排到正确的线程中
    @Override
    public void show() {
        if(localLOGV) Log.v(TAG, "SHON:" + this);
        mHandler.post (mShow);
    }
    //将handleHide安排到正确的线程中
    @Override
    public void hide(){
        if (localLOGV) Log.v(TAG, "HIDE:"+ this);
        mHandler.post (mHide);
    }
    //TN的handleShow将Toast视图添加到Window
    mWM =（WindowManager) context.getSystemService (Context.WINDOW_SERVICE);
    mwM.addview (mView, mParams);
    //NT的handleHide将Toast视图从Window移除
    if (mview.getParent() != null){
        if (localLOGV) Log.v(TAG, "REMOVE!" + mView + "in" + this);
        mWM.removeView(mView);
    }
    ```

### 9、四大组件工作过程

#### 9.1、Activity 工作过程

- 启动具体 Activity

  ```java
  Intent intent = new Intent (this, TestActivity.class);
  startActivity(intent);
  ```
  - startActivity 有好几种重载，最终都调用 startActivityForResult
    - mParent：ActivityGroup，最开始被用来在一个界面中嵌入多个子 Activity，在 API13 废弃，系统推荐 Fragment 代替其
    - `mMainThread.getApplicationThread()` 是 ApplicationThread，ActivityThread 的内部类

  ```java
  public void startActivityForResult(Intent intent, int requestCode, @Nullable Bundle options){
      if (mParent == null) {
          Instrumentation.ActivityResult ar =
  mInstrumentation.execStartActivity(this, mMainThread.getApplicationThread(), mToken, this, intent, requestCode, options);
          if(ar != null){
              mMainThread.sendActivityResult(mToken, mEmbeddedID, requestCode, ar.getResultcode(), ar.getResultData());
          }
          if(requestCode >= O){
              //如果这个开始请求一个结果，可避免在收到结果前使活动可见。在onCreate或onResume期间设置此代码将使活动在此期间隐藏，以避免闪烁。只能在请求结果时进行，保证在活动完成时获得信息，无论发生什么
              mStartedActivity = true;
          }
          final View decor = mWindow != null ? mWindow.peekDecorView() : null;
          if(decor != null) {
              decor.cancelPendingInputEvents();
          }
          //TODO考虑清除/刷新子窗口的其他事件源和事件
      }else{
          if (options != null){
              mParent.startActivityFromchild(this, intent, requestCode, options);
          }else{
              //要检查此方法，以与可能已重写其现有应用程序兼容
              mParent.startActivityFromChild(this, intent, requestCode);
          }
      }
      if (options != null && !isTopOfTask()){
          mActivityTransitionstate.startExitoutTransition (this, options);
      }
  }
  ```

  - Instrumentation#execStartActivity
    - ActivityManagerService（AMS）继承 ActivityManagerNative（AMN），AMN 继承 Binder 并实现 IActivityManager 这个 Binder 接口，AMS 是 Binder，是 IActivityManager 的具体实现
    - 启动 Activity 真正实现由 `ActivityManagerNative.getDefault()` 的 startActivity 完成，是 IActivityManager 类型的 Binder 对象，其具体实现是 AMS

  ```java
  public ActivityResult execstartActivity(Context who, IBinder contextThread, IBinder token, Activity target, Intent intent, int requestcode, Bundle options) {
      IApplicationThread whoThread = (IApplicationThread) contextThread;
      if (mAetivityMonitors != null) {
          synchronized (mSync){
              final int N = mActivityMonitors.size();
              for (int i=0;i<N;i++){
                  final ActivityMonitor am = mActivityMonitors.get(i);
                  if (am.match(who, null, intent)){
                      am.mHits++;
                      if(am.isBlocking()){
                          return requestCode >= 0 ? am.getResult() : null;
                      }
                      break;
                  }
              }
          }
      }
      try {
          intent.migrateExtrastreamToClipData();
          intent.prepareToLeaveProcess();
          int result = ActivityManagerNative.getDefault().startActivity(whoThread, who.getBasePackageName(), intent, intent.resolveTypelfNeeded(who.getContentResolver()), token, target != null ? target.mEmbeddedID : null, requestcode, 0, null, options);
          //检查启动Activity结果，无法正确启动Activity时抛出异常
          checkStartActivityResult(result, intent);
      }catch (RemoteException e){}
      return null;
  }
  ```

  - AMN 中的 AMS 对象采用单例模式对外提供，Singleton 是单例封装类，首次调用其 get 方法时会通过 create 方法初始化 AMS 对象，后续调用中直接返回之前创建的对象

  ```java
  static public IActivityManager getDefault(){
      return gDefault.get();
  }
  private static final Singleton<IActivityManager> gDefault = new Singleton<>(){
      protected IActivityManager create(){
          IBinder b = ServiceManager.getService("activity");
          if(false){
              Log.v("ActivityManager", "default service binder= " +b);
          }
          IActivityManager am = asInterface(b);
          if(false) {
              Log.v("ActivityManager", "default service=" + am);
          }
          return am;
      }
  };
  ```

  - AMS#startActivity
    - Activity 启动转到 ActivityStackSupervisor 的 startActivityMayWait，其调用 startActivityLocked -> startActivityUncheckedLocked -> ActivityStack 的 resumeTopActivitiesLocked，启动过程转到 ActivityStack

  ```java
  public final int startActivity(IApplicationThread caller, string callingPackage, Intent intent, String resolvedType, IBinder resultTo, String resultwho, int requestCode, int startFlags, ProfilerInfo profilerInfo, Bundle options){
      return startActivityAsUser(caller, callingPackage, intent, resolvedType, resultTo, resultwho, requestCode, startFlags, profilerInfo, options, UserHandle.getCallingUserId());
  }
  
  public final int startActivityAsUser(IApplicationThread caller, StringcallingPackage, Intent intent, String resolvedType, IBinder resultTo, Stringresultwho, int requestCode, int startFlags, ProfilerInfo profilerInfo, Bundle options, int userId){
      enforceNotIsolatedCaller("startActivity");
      userId = handleIncomingUser(Binder.getcallingPid(), Binder.getcallingUid(), userId, false, ALLOW_FULL_ONLY, "startActivity", null);
      //TODO:在此切换到用户应用栈
      return mStackSupervisor.startActivityMayWait(caller, -1, callingPackage, intent, resolvedType, null, null, resultTo, resultWho, requestCode, startFlags, profilerInfo, null, null, options, userId, null, null);
  }
  ```

  - ActivityStack#resumeTopActivitiesLocked
    - 调用 resumeTopActivityInnerLocked -> ActivityStackSupervisor 的 startSpecificActivityLocked

  ```java
  final boolean resumeTopActivityLocked (ActivityRecord prev, Bundle options){
      if (inResumeTopActivity){
          //别再重复
          return false;
      }
      boolean result = false;
      try{
          //防止递归
          inResumeTopActivity = true;
          result = resumeTopActivityInnerLocked(prev, options);
      }finally {
          inResumeTopActivity = false;
      }
      return result;
  }
  ```

  - ActivityStackSupervisor#startSpecificActivityLocked

    - 调用 realStartActivityLocked

    - Activity 启动过程在 ActivityStackSupervisor 和 ActivityStack 间传递顺序：

      ![安卓开发艺术探索_ActivityStackSupervisor和ActivityStack间传递顺序](Image.assets\安卓开发艺术探索_ActivityStackSupervisor和ActivityStack间传递顺序.png)

  ```java
  void startSpecificActivityLoeked (ActivityRecord r, boolean andResume, boolean checkConfig){
      ProcessRecord app = mService.getProcessRecordLocked (r.processName, r.info.applicationInfo.uid, true);
      r.task.stack.setLaunchTime(r);
      if (app != null && app.thread != null){
          try{
              if((r.info.flags & ActivityInfo.FLAGMULTIPROCESS) == 0 || !"android".equals (r.info.packageName)){
                  //如果是被标记为在多个进程中运行的平台组件，则不添加，因为这实际是框架的一部分，在进程中作为单独apk跟踪没有意义
                  app.addPackage(r.info.packageName, r.info.applicationInfo.versionCode, mService.mProcessStats);
              }
              realStartActivityLocked(r, app, andResume, checkConfig);
              return;
          }catch (RemoteException e) {
              Slog.w(TAG, "Exception when starting activity"
  + r.intent.getComponent().flattenToShortString(), e);
          }
          //如果抛出了一个死对象异常——无法重新启动应用程序
      }
      mService.startProcessLocked (r.processName, r.info.applicationInfo, true, 0,"activity", r.intent.getComponent(), false, false, true);
  }
  ```

  - ActivityStackSupervisor#realStartActivityLocked
    - app.thread 类型 IApplicationThread

  ```java
  app.thread.scheduleLaunchActivity(new Intent(r.intent), r.appToken, System.identityHashCode(r)， r.info, new Configuration(mService.mConfiguration), r.compat,r.task.voiceInteractor, app.repProcState, r.icicle, r·persistentstate, results, newIntents, !andResume, mService.isNextTransitionForward(), profilerInfo);
  ```

  - IApplicationThread
    - 继承 IInterface 接口，Binder类型的接口，包含大量启动、停止 Activity、服务的接口，IApplicationThread 的实现者是 ActivityThread 的内部类 ApplicationThread

  ```java
  public interface IApplicationThread extends IInterface{
      void schedulePauseActivity(IBinder token, boolean finished, boolean userLeaving, int configChanges, boolean dontReport) throws RemoteException;
      void scheduleStopActivity(IBinder token, boolean showwindow, int configChanges) throws RemoteException;
      void scheduleWindowVisibility(IBinder token，boolean showWindow) throws RemoteException;
      void scheduleSleeping(IBinder token, boolean sleeping) throws RemoteException;
      void scheduleResumeActivity (IBinder token, int procState, boolean isForward, Bundle resumeArgs) throws RemoteException;
      void scheduleSendResult(iBinder token, List<ResulEIafo> roaulta) throws RemoteException;
      void scheduleLaunchActivity(Intent intent, IBinder token，int ident, ActivityInfo info, Configuration curConfig, CompatibilityInfocompatInfo, IVoiceInteractor voiceInteractor, int procstate, Bundle state, PersistableBundle persistentstate, List<ResultInfo> pendingResults, List<Intent> pendingNewIntents, boolean notResumed, boolean isForward, ProfilerInfo profilerInfo) throws RemoteException;
      void scheduleRelaunchActivity(IBinder token, List<ResultInfo> pendingResults, List<Intent> pendingNewIntents, int configChanges, boolean notResumed, Configuration config) throws RemoteException;
      void scheduleNewIntent(List<Intent> intent, IBinder token) throws RemoteException;
      void scheduleDestroyActivity(IBinder token, boolean finished, int configChanges) throws RemoteException;
      void scheduleReceiver(Intent intent, ActivityInfo info, CompatibilityInfo compatInfo, int resultCode, String data, Bundle extras, boolean sync, int sendingUser, int processstate) throws RemoteException;
      
      static final int BACKUP_MODE_INCREMENTAL = 0;
      static final int BACKUP_MODE_FULL = 1;
      static final int BACKUP_MODE_RESTORE = 2;
      static final int BACKUP_MODE_RESTORE_FULL = 3;
      
      void scheduleCreateBackupAgent(Applicationinfo app, CompatibilityInfo compatInfo, int backupMode) throws RemoteException;
      void scheduleDestroyBackupAgent(ApplicationInfo app, CompatibilityInfo compatInfo) throws RemoteException;
      void scheduleCreateService(IBinder token, ServiceInfo info, CompatibilityInfo compatInfo, int processstate) throws RemoteException;
      void scheduleBindservice(IBinder token, Intent intent, boolean rebind, int processstate) throws RemoteException;
      void scheduleUnbindservice(IBinder token, Intent intent) throws RemoteException;
      void scheduleServiceArgs(IBinder token, boolean taskRemoved, intstartId, int flags, Intent args) throws RemoteException;
      void scheduleStopService (IBinder token) throws RemoteException;
  ```

  - ApplicationThread
    - 继承 ApplicationThreadNative（ATN）
    - ATN 继承 Binder 并实现 IApplicationThread 接口
    - ATN 内有 ApplicationThreadProxy（ATP）类，系统为 AIDL 文件自动生成的代理类，ATN 是 IApplicationThread 的实现者，ANT 是抽象类，所以 ApplicationThread 是 IApplicationThread 的实现者

  ```java
  private class ApplicationThread extends ApplicationThreadNative{}
  public abstract class ApplicationThreadNative extends Binder{}
  implements IApplicationThread{}
  
  class ApplicationThreadProxy implements IApplicationThread{
      private final IBinder mRemote;
      
      public ApplicationThreadProxy(IBinder remote){
          mRemote = remote;
      }
      
      public final IBinder asBinder(){
          return mRemote;
      }
      
      public final void schedulePauseActivity(IBinder token, boolean finished, boolean userLeaving, int configChanges, boolean dontReport) throws RemoteException{
          Parcel data = Parcel.obtain();
          data.writeInterfaceToken (ApplicationThread.descriptor);
          data.writeStrongBinder(token);
          data.writeInt(finished ? 1 : 0);
          data.writeInt(userLeaving ? 1 : 0);
          data.writeInt(configChanges);
          data.writeInt(dontReport ? 1 : 0);
          mRemote.transact (SCHEDULE_PAUSE_ACTIVITY_TRANSACTION, data, null, IBinder.FLAG ONEWAY);
          data.recycle();
      }
      
      public final void scheduleStopActivity(IBinder token, boolean showWindow, int configChanges) throws RemoteException{
          Parcel data = Parcel.obtain();
          data.writeInterfaceToken (IApplicationThread.descriptor);
          data.writeStrongBinder(token);
          data.writeInt(showwindow ? 1 : 0);
          data.writeInt(configChanges);
          mRemote.transact (SCHEDULE_STOP_ACTIVITY_TRANSACTION, data, null, IBinder.FLAG_ONEWAY);
          data.recycle();
      }
  	//...
  }
  ```

  - ApplicationThread#scheduleLaunchActivity
    - 发送一个启动 Activity 的消息由 Handler（H）处理

  ```java
  //使用token标识该活动，不必将Activity本身发回活Activity管理器(比ipc重要)
  public final void scheduleLaunchActivity(Intent intent, IBinder token, int ident, ActivityInfo info, Configuration curConfig, CompatibilityInfocompatInfo, IVoiceInteractor voiceInteractor, int proastate, Bundle state, PersistableBundle persistentstate, List<ResultInfo> pendingResults, List<Intent> pendingNewIntents, boolean notResumed, boolean isForward, ProfilerInfo profilerInfo){
      updateProcessState(procState, false);
      AetivityClientRecosd r = new ActivityclientRecord();
      r.token = token;
      r.ident = ident;
      r.intent = intent;
      r.voiceInteractor = voiceInteractor;
      r.activityInfo = info;
      r.compatInfo = compatInfo;
      r.state = state;
      r.persistentState = persistentState;
      r.pendingResults = pendingResults;
      r.pendingIntents = pendingNewIntents;
      r.startsNotResumed = notResumed;
      r.isForward = isForward;
      r.profilerInfo = profilerInfo;
      updatePendingconfiguration(curConfig);
      //发送消息给H处理
      sendMessage(H.LAUNCH_ACTIVITY, r);
  }
  
  private void sendMessage(int what, object obj, int arg1, int arg2, boolean async){
      if (DEBUG MESSAGES) Slog.v(TAG, "SCHEDULE" + what + mH.codeToString(what) + ":" + arg1 + "/" + obj);
      Message msg = Message.obtain();
      msg.what = what;
      msg.obj = obj;
      msg.arg1 = arg1;
      msg.arg2 = arg2;
      if(async){
          msg.setAsynchronous(true);
      }
      mH.sendMessage(msg);
  }    
  ```

  - Handler H

  ```java
  private class H extends Handler{
      public static final int LAUNCH_ACTIVITY = 100;
      public static final int PAUSE_ACTIVITY = 101;
      public static final int PAUSE_ACTIVITY_FINI_SHING = 102;
      public static final int STOP_ACTIVITY_SHOW = 103;
      public static final int STOP_ACTIVITY_HIDE = 104;
      public static final int SHOW_WINDOW = 105;
      public static final int HIDE_WINDOW = 106;
      public static final int RESUME_ACTIVITY = 107;
      //...
      public void handleMessage(Message msg) {
          if(DEBUG MESSAGES) Slog.v(TAG, ">>>handling:" + codeToString(msg.what));
          switch (msg.what) {
              case LAUNCH_ACTIVITY: {
                  Trace.traceBegin (Trace.TRACE_TAG_ACTIVITY_MANAGER, "activityStart");
                  final ActivityClientRecord r = (ActivityClientRecord) msg.obj;
                  r.packageInfo = getPackageInfoNoCheck (r.activityInfo.applicationInfo, r.compatInfo);
                  handleLaunchActivity(r, nu11);
                  Trace.traceEnd (Trace.TRACE_TAG_ACTIVITY_MANAGER); 
              }break;
              case RELAUNCH_ACTIVITY:{
                  Trace.traceBegin (Trace.TRACE_TAG_ACTIVITY_MANAGER, "activityRestart");
                  ActivityClientRecord r = (ActivityClientRecord) msg.obj;
                  handleRelaunchActivity(r);
                  Trace.traceEnd (Trace.TRACE_TAG_ACTIVITY_MANAGER);
              } break;
              case PAUSE_ACTIVITY:
                  Trace.traceBegin (Trace.TRACE_TAG_ACTIVITY_MANAGER, "activityPause");
                  handlePauseActivity((IBinder)msg.obj, false, (msg.arg1 & 1) != 0, msg.arg2, (msg.arg1 & 2) != 0);
                  maybeSnapshot();
                  Trace.traceEnd (Trace.TRACE_TAG_ACTIVITY_MANAGER);
                  break;
               //...
          }
          if(DEBUG MESSAGES) Slog.v(TAG, "<<<done:" + codeToString(msg.what));
      }
  }
  ```

  - ActivityThread#handleIaunchActivity
    - 完成 Activity 对象的创建和启动，ActivityThread 通过 handleResumeActivity 调用被启动 Activity 的 onResume

  ```java
  private void handleLaunchActivity(ActivityClientRecord r, Intent customIntent){
      if(localLOGV) Slog.v(TAG，"Handling launchof" + r);
      Activity a = performLaunchActivity(r, customIntent);
      if(a != null) {
          r.createdConfig = new Configuration(mConfiguration);
          Bundle oldState = r.state;
          handleResumeActivity(r.token, false, r.isForward, !r.activity.mFinished && !r.startsNotResumed);
          //...
      }
      //...
  }
  ```

  - performLaunchActivity

    - 从 ActivityClientRecord 中获取待启动 Activity 的组件信息

    ```java
    ActivityInfo aInfo = r.activityInfo;
    if (r.packageInfo == null) {
        r.packageInfo = getPackageInfo (aInfo.applicationInfo, r.compatInfo, Context.CONTEXT_INCLUDE_CODE);
    }
    ComponentName component = r.intent.getComponent();
    if (component == null){
        component = r.intent.resolveActivity (mInitialApplication.getPackageManager());
        r.intent.setComponent(component);
    }
    if (r.activityInfo.targetActivity != null) {
        component = new ComponentName (r.activityInfo.packageName, r.activityInfo.targetActivity)
    };
    ```

    - 通过 Instrumentation 的 newActivity 使用类加载器创建 Activity 对象

    ```java
    Activity activity = null;
    try {
        java.lang.ClassLoader cl = r.packageInfo.getClassLoader();
        activity = mInstrumentation.newActivity(cl, component.getClassName(), r.intent);
        StrictMode.incrementExpectedActivityCount (activity.getClass());
        r.intent.setExtrasClassLoader(cl);
        r.intent.prepareToEnterProcess();
        if (r.state != null) {
            r.state.setClassLoader(cl);
        }
    } catch (Exception e) {
        if (!mInstrumentation.onException(activity, e)){
            throw new Runt imeException ("Unable to instantiate activity" + component + e.toString(), e);
        }
    }
    //Instrumentation的newActivity通过类加载器创建Activity对象
    public Activity newActivity(ClassLoader c1, String className, Intent intent) throws InstantiationException, IllegalAccessException, ClassNotFoundException{
        return (Activity) c1.loadClass(className).newInstance();
    }
    ```

    - 通过 LoadedApk 的 makeApplication 尝试创建 Application 对象

      > 如果 Application 已被创建过，不会重复创建，一个应用只有一个 Application 对象，其创建通过 Instrumentation 完成，通过类加载器实现，创建完后系统通过 Instrumentation 的 callApplicationOnCreate 调用 Application 的 onCreate

    ```java
    public Application makeApplication (boolean forceDefaultAppClass, Instrumentation instrumentation){
        if (mApplication != null) {
            return mApplication;
        }
        Application app = null;
        String appClass = mApplicationInfo.className;
        if (forceDefaultAppClass || (appClass == null)) {
            appClass = "android.app.Application";
        }
        try {
            java.lang.ClassLoader cl = getClassLoader();
            if (!mPackageName.equals("android")) {
                initializeJavaContextClassLoader();
            }
            ContextImpl appContext = ContextImpl.createAppContext (mActivityThread, this);
            app = mActivityThread.mInstrumentation.newApplication(cl, appClass, appContext);
            appContext.setOuterContext(app);
        } catch (Exception e) {
            if (!mActivityThread.mInstrumentation.onException(app, e)) {
                throw new RuntimeException("Unable to instantiate application" + appClass + ": "+ e.tostring(), e);
            }
        }
        mActivityThread.mAllApplications.add(app);
        mApplication = app;
        if (instrumentation != null) {
            try {
                instrumentation.callApplicationOnCreate (app);
            } catch (Exception e){
                if (!instrumentation.onException(app, e)) {
                    throw new RuntimeException("Unable to create application" + app.getClass().getName () + ":" + e.toString(), e);
                }
            }
        }
        //...
        return app;
    }
    ```

    - 创建 Contextlmpl 对象并通过 Activity 的 attach 完成重要数据初始化

      > Contextlmpl 是 Context 的具体实现，其通过 Activity 的 attach 和 Activity 建立关联，attach 中 Activity 完成 Window 的创建并建立和 Window 的关联，当 Window 接收到外部输入事件后可将事件传递给 Activity

    ```java
    Context appContext = createBaseContextForActivity(r, activity);
    CharSequence title = r.activityInfo.loadLabel (appContext.getPackageManager());
    Configuration config = new Configuration (mCompatConfiguration);
    if (DEBUG_CONF_IGURATION) Slog.v(TAG, "Launching activity" + r.activityInfo.name + "with config" + config);
    activity.attach(appContext, this, getInstrumentation(), r.token, r.ident, app, r.intent, r.activityInfo, title, r.parent, r.embeddedID, r.las, tNonConfigurationInstances, config, r.voiceInteractor);
    ```

    - 调用 Activity 的 onCreate

      > `mInstrumentation.callActivityOnCreate(activity, r.state)`

#### 9.2、Service 工作过程

- 使用

```java
//Context的startService
Intent intentService = new Intent(this, MyService.class);
startService (intentService);
//Context的bindService
bindService(intentService, mServiceConnection, BIND_AUTO_CREATE);
```

- 启动过程

  - ContextWrapper#startService

  ```java
  public ComponentName startService (Intent service) {
      //mBase:ComtextImpl对象
      return mBase.startService (service);
  }
  ```

  - ComtextImpl#startService

  ```java
  public ComponentName startService (Intent service) {
      warnIfCallingFromSystemProcess();
      return startServiceCommon(service, mUser);
  }
  
  private ComponentName startServiceCommon (Intent service, UserHandle user) {
      try {
          validateServiceIntent (service);
          service.prepareToLeaveProcess();
          ComponentName en = ActivityManagerNative.getDefault().startService (mMainThread.getApplicationThread(), service, service.resolveTypeIfNeeded(getContentResolver()), user.getIdentifier());
          if (cn != null) {
              if (cn.getPackageName().equals("!")) {
                  throw new SecurityException("Not allowed to start service" + service + "without permission" + cn.getClassName());
              } else if (cn.getPackageName().equals("!!")) {
                  throw new SecurityException("Unable to start service" + service ":" + cn.getClassName());
              }
          }
          return cn;
      }catch (RemoteException e) {
          return null;
      }
  }
  ```

  - AMS#startService

  ```java
  public ComponentName startService (IApplicationThread caller, Intent service, String resolvedType, int userId) {
      enforceNotIsolatedCaller ("startService");
      //拒绝可能泄漏的文件描述符
      if (service != null && service.hasFileDescriptors() == true) {
          throw new IllegalArgumentException("File descriptors passed in Intent") ;
      }
      if (DEBUG_SERVICE) Slog.v(TAG, "startService:" + service + "type=" + resolvedType);
      synchronized(this) {
          final int callingPid = Binder.getCallingPid();
          final int callingUid = Binder.getCallingUid();
          final long origId = Binder.clearCallingIdentity();
          //mServices类型ActiveService，辅助AMS管理Service的类（启动、绑定、停止），该方法会调用startServicelnnerLocked
          ComponentName res = mServices.startServiceLocked(caller, service, resolvedType, callingPid, callingUid, userId);
          Binder.restoreCallingIdentity (origId);
          return res;
      }
  }
  ```

  - ActiveService#startServicelnnerLocked
    - ServiceRecord 描述一个 Service 记录，其贯穿整个 Service 启动过程
    - 把后续工作交给 bringUpServiceLocked 处理，其又调用realStartServiceLocked

  ```java
  ComponentName startServiceInnerLocked (ServiceMap smap, Intent service, ServiceRecord r, boolean callerFg, boolean addToStarting) {
      Processstats.ServiceState stracker = r.getTracker();
      if (stracker != null) {
          stracker.setStarted(true, mAm.mProcessStats.getMemFactorLocked(), r.lastActivity);
      }
      r.callStart = false;
      synchronized (r.stats.getBatteryStats()) {
          r.stats.startRunningLocked();
      }
      String error = bringUpServiceLocked(r, service.getFlags(), callerFg, false);
      if(error != null) {
          return new ComponentName("!!", error);
      }
      if (r.startRequested && addToStarting) {
          boolean first = smap.mStartingBackground.size() == 0;
          smap.mStartingBackground.add(r);
          r.startingBgTimeout = SystemClock.uptimeMillis() + BG_START_TIMEOUT;
          if (DEBUG_DELAYED_SERVICE){
              RuntimeException here = new RuntimeException("here");
              here.fillInStackTrace();
              Slog.v (TAG, "Starting background(first=" + first + "):" + r, here);
          }else if (DEBUG_DELAYED_STARTS) {
              Slog.v (TAG, "Starting background (first=" + first +"): "+r);
          }
          if (first) {
              smap.rescheduleDelayedstarts();
          }
      } else if (callerFg) {
          smap.ensureNotStartingBackground(r);
      }
      return r.name;
  }
  ```

  - realStartServiceLocked
    - 通过 app.thread 的 scheduleCreateService 创建 Service 对象并调用其 onCreate，通过 sendServiceArgsLocked 调用 Service 其他方法，如 onStartCommand，均是进程间通信
    - app.thread 是 IApplicationThread 类型，实际是 Binder, 具体实现是 ApplicationThread 和 ApplicationThreadNative，ApplicationThread 继承 ApplicationThreadNative

  ```java
  private final void realStartServiceLocked (ServiceRecord r， ProcessRecord app, boolean execInFg) throws RemoteException{
      //...
      boolean created = false;
  	try {
          String nameTerm;
          int lastPeriod = r.shortName.lastIndexOf('.');
          nameTerm = lastPeriod >= 0 ? r.shortName.substring(lastPeriod) : r.shortName;
          if (LOG_SERVICE_START_STOP) {
              EventLogTags.writeAmCreateService(r.userId, System.identityHashCode(r), nameTerm, r.app.uid, r.app.pid);
          }
          synchronized (r.stats.getBatteryStats()){
              r.stats.startLaunchedLocked();
          }
          mAm.ensurePackageDexOpt(r.serviceInfo.packageName); 
          app.forceProcessStateUpTo (ActivityManager.PROCESS_STATE_SERVICE);
          app.thread.scheduleCreateService(r, r.serviceInfo, mAm.compatibilityInfoForPackageLocked (r.serviceInfo.applicationInfo), app.repProcState);
          r.postNotification();
          created = true;
      }catch (DeadobjectException e){
          Slog.w(TAG, "Application dead when creating service" + r);
          mAm.appDiedLocked(app);
      }finally {
          if (!created){
              app.services.remove(r);
              r.app = null;
              scheduleServiceRestartLocked(r, false);
              return;
          }
      }
      requestServiceBindingsLocked(r, execInEg);
      updateServiceClientActivitiesLocked(app, null, true);
      //如果服务处于started状态，且无挂起参数，则伪造参数，以便调用其onStartCommand
  	if (r.startRequested && r.callStart && r.pendingStarts.size() == 0) {
          r.pendingStarts.add (new ServiceRecord.StartItem(r, false, r.makeNextStartId(), null, null));
      }
      sendServiceArgsLocked(r, execInFg, true);
  	//...
  }
  ```
  
- ApplicationThread#scheduleCreateService
  
    - 发消息给 Handler H 完成，其接收 CREATE_SERVICE 消息并通过 ActivityThread 的 handleCreateService 完成 Service 最终启动

```java
  public final void scheduleCreateService(IBinder token, ServiceInfo info, CompatibilityInfo compatInfor, int processState){
      updateProcessState(processState, false);
      CreateServiceData s = new CreateServiceData();
      s.token = token;
      s.info = info;
      s.compatInfo = compatInfo;
      sendMessage (H.CREATE_SERVICE, s);
  }
```

- ActivityThread#handleCreateService
    - 通过类加载器创建 Service 实例
    - 创建 Application 对象并调用其 onCreate，创建过程只有一次
    - 创建 ConTextImpl 对象并通过 Service 的 attach 建立二者间关系，和 Activity 类似，Service 和 Activity 都是一个 Context
    - 调用 Service 的 onCreate 并将 Service 对象存到 ActivityThread 一个列
      表中：`final ArrayMap<IBinder, Service> mServices = new ArrayMap<>()` 

```java
  private void handleCreateService (CreateServiceData data) {
      //如果进入后台后准备gc，又回到活动状态，跳过
      unscheduleGcIdler();
      LoadedApk packageInfo = getPackageInfoNoCheck (data.info.applicationInfo, data.compatInfo);
      Service service = null;
      try {
          java.lang.ClassLoader cl = packageInfo.getClassLoader();
          service = (Service) cl.loadClass (data.info.name).newInstance();
      } catch (Exception e) {
          if (!mInstrumentation.onException(service, e)){
              throw new RuntimeException ("Unable to instantiate service" + data.info.name + ":" + e.toString(), e);
          }
      }
      try {
          if(localLOGV) Slog.v(TAG, "Creating service" + data.info.name);
          ContextImpl context = ContextImpl.createAppContext (this, packageInfo);
          context.setOuterContext (service);
          Application app = packageInfo.makeApplication(false, mInstrumentation);
          service.attach(context, this, data.info.name, data.token, app, ActivityManagerNative.getDefault());
          service.onCreate();
          mServices.put (data.token, service);
          try {
              ActivityManagerNative.getDefault().serviceDoneExecuting(
  data.token, 0, 0, 0);
          }catch (RemoteException e) {}
      }catch (Exception e) {
          if (!mInstrumentation.onException(service, e)){
              throw new RuntimeException ("Unable to create service" + data.info.name + ":" + e.toString(), e);
          }
      }
  }
```

- ActivityThread 通过 handleServiceArgs 调用 Service 的 onStartCommand方法
  
```java
  private void handleServiceArgs (ServiceArgsData data) {
      Service S = mServices.get (data.token);
      if (s != null) {
          try {
              if (data.args != null) {
                  data.args.setExtrasClassLoader (s.getClassLoader());
                  data.args.prepareToEnterProcess();
              }
              int res;
              if (!data.taskRemoved) {
                  res = s.onStartCommand (data.args, data.flags, data.startId);
              } else{
                  s.onTaskRemoved(data.args);
                  res = Service.START_TASK_REMOVED_COMPLETE;
              }
              Queuedwork.waitToFinish();
              try{
                  ActivityManagerNative.getDefault().serviceDoneExecuting (data, token, 1, data.startId, res);
              } catch (RemoteException e){}
              ensureJitEnabled();
          } catch (Exception e) {
              if (!mInstrumentation.onException(s, e)) {
                  throw new RuntimeException("Unable to start service" + s + "with" + data.args + ":" + e.toString(), e);
              }
          }
      }
  }
```

- 绑定过程

  - bindService

  ```java
  public boolean bindService (Intent service, Servi ceConnection conn, int flags) {
      //mBase是ContextImpl类型对象，该方法最终调用bindServiceCommon
      return mBase.bindService (service, conn, flags);
  }
  ```

  - bindServiceCommon
    - 将客户端的 ServiceConnection 对象转为 ServiceDispatcher.InnerConnection 对象，因为服务的绑定可能跨进程的，须借助 Binder 让远程服务端回调自己的方法
    - ServiceDispatcher 连接 ServiceConnection 和 InnerConnection，由 LoadedApk 的 getServiceDispatcher 完成
  
```java
  private boolean bindserviceCommon (Intent service, ServiceConnection conn, int flags, UserHandle user) {
      IServiceConnection sd;
      if (conn == null) {
          throw new IllegalArgumentException("connection is null");
      }
      if (mPackageInfo != null) {
          sd = mPackageInfo.getServiceDispatcher(conn, getOuterContext(), mMainThread.getHandler()，flags);
      } else{
          throw new RuntimeException("Not supported in system context");
      }
      validateServiceIntent (service);
      try {
          IBinder token = getActivityToken();
          if (token == null && (flags & BIND_AUTO_CREATE) == 0 && mPackageInfo != null && mPackageInfo.getApplicationInfo().targetSdkVersion < android.os.Build.VERSION_CODES.ICE_CREAM_SANDWICH) {
              flags |= BIND_WAIVE_PRIORITY;
          }
          service.prepareToLeaveProcess;
          int res = ActivityManagerNative.getDefault().bindService (mMainThread.getApplicationThread(), getActivityToken(), service, service.resolveTypeIfNeeded(getContentResolver()), sd, flags, user.getIdentifier());
          if(res < 0){
              throw new SecurityException("Not allowed to bind to service" + service);
          }
          return res != 0;
      } catch (RemoteException e) {
          return false;
      }
  }
```

  - LoadedApk#getServiceDispatcher
  - mServices 是 ArrayMap，存储应用当前活动的 ServiceConnection 和 ServiceDispatcher 的映射关系：`private final ArrayMap<Context, ArrayMap<ServiceConnection, LoadedApk.ServiceDispatcher>> mServices new ArrayMap<>();`
    - 系统先查找是否存在相同 ServiceConnection，不存在就重新创建一个 ServiceDispatcher 对象并将其存在 mServices，key 是 ServiceConnection，value 是 ServiceDispatcher，value 内部保存 ServiceConnection 和 InnerConnection 对象
    - Service 和客户端建立连接后系统通过 InnerConnection 调用 ServiceConnection 的onServiceConnected，可能跨进程
    - ServiceDispatcher 创建好后，getServiceDispatcher 返回其保存的 InnerConection 对象
    - bindServiceCommon 通过 AMS 完成 Service 具体绑定，对应 AMS 的 bindService

  ```java
public final IServiceConnection getServiceDispatcher (ServiceConnection c, Context context, Handler handler, int flags){
      synchronized (mServices) {
          LoadedApk.ServiceDispatcher sd = null;
          ArrayMap<ServiceConnection, LoadedApk.ServiceDispatcher> map = mServices.get(context);
          if (map != null) {
              sd = map.get(c);
          }
          if (sd == null) {
              sd = new ServiceDispatcher(c, context, handler, flags);
              if (map == null) {
                  map = new ArrayMap<ServiceConnection, LoadedApk.ServiceDispatcher>();
                  mServices.put (context, map);
              }
              map.put(c, sd);
          } else {
              sd.validate(context, handler);
          }
          return sd.getIServiceConnection();
      }
  }
  ```

  - AMS#bindService
  - 调用 ActiveServices 的 bindServiceLocked -> bringUpServiceLocked -> realStartServiceLocked，最终通过 ApplicationThread 完成 Service 实例的创建并执行其 onCreate
    - 和启动 Service 不同，绑定过程会调用 app.thread 的 scheduleBindService，其实现在 ActiveServices 的 requestServiceBindingLocked

  ```java
public int bindService (IApplicationThread caller, IBinder token, Intent service, String resolvedType, IServiceConnection connection, int flags, int userId) {
      enforceNotIsolatedCaller ("bindService");
      //拒绝可能泄漏的文件描述符 
      if (service != null && service.hasFileDescriptors() == true) {
          throw new IllegalArgumentException("File descriptors passed in Intent");
      }
      synchronized(this) {
          return mServices.bindServiceLocked(caller, token, service, resolvedType， connection, flags, userId);
      }
  }
  ```

  - ActiveServices#requestServiceBindingLocked
  - app.thread 是 ApplicationThread，其一系列以 schedule 开头的方法内部都通过 Handler H 中转

  ```java
private final boolean requestServiceBindingLocked (ServiceRecord r,
  IntentBindRecord i, boolean execInFg, boolean rebind){
      if (r.app == null || r.app.thread == null){
          //如果服务未运行，无法绑定
          return false;
      }
      if ((!i.requested || rebind) && i.apps.size() > 0) {
          try {
              bumpServiceExecutingLocked(r,execInFg,"bind");
              r.app.forceProcessStateUpTo (ActivityManager.PROCESS_STATE_SERVICE);
              r.app.thread.scheduleBindService(r, i.intent.getIntent(), rebind, r.app.repProcstate);
              if (!rebind) {
                  i.requested = true;
              }
              i.hasBound = true;
              i.doRebind = false;
          } catch (RemoteException e) {
              if(DEBUG SERVICE) Slog.v(TAG, "Crashed while binding" + r);
              return false;
          }
      }
      return true;
  }
  ```

  - scheduleBindService

  ```java
public final void scheduleBindService (IBinder token, Intent intent, boolean rebind, int processState) {
      updateProcessState (processState, false);
      BindServiceData s = new BindServiceData ();
      s.token = token;
      s.intent = intent;
      s.rebind = rebind;
      if (DEBUG_SERVICE) Slog.v(TAG, "scheduleBindService token=" + token + "intent=" + intent + "uid=" + Binder.getCallingUid() + "pid=" + Binder.getCallingPid());
      sendMessage(H.BIND_SERVICE, s);
  }
  ```

  - H 收到 BIND_SERVICE 这类消息时交给 ActivityThread 的 handleBindService 处理，其先根据 Service 的 token 取出 Service 对象，调用 Service 的 onBind，返回 Binder 对象给客户端使用
- Service 的 onBind 被调用后，客户端不知道已成功连接 Service 了，须调用客户端的 ServiceConnection 的 onServiceConnected，由 `ActivityManagerNative.getDefault()` 的 publishService 完成，其是 AMS
  - handleBindService
    - 多次绑定同一 Service 时，其 onBind 只执行一次，除非 Service 被终止，执行后系统需告知客户端已成功连接 Service，由 AMS 的 publishService 实现
  
  ```java
private void handleBindService (BindServiceData data) {
      Service s = mServices.get (data. token);
      if (DEBUG SERVICE)
          Slog.v (TAG, "handleBindservice S=" + S + "rebind=" + data.rebind);
      if(s != null) {
          try {
              data.intent.setExtrasClassLoader (s.getClassLoader());
              data.intent.prepareToEnterProcess();
              try {
                  if (!data.rebind){
                      IBinder binder = s.onBind (data.intent);
                      ActivityManagerNative.getDefault().publishService(data.token, data.intent, binder);
                  }else{
                      s.onRebind (data.intent);
                      ActivityManagerNative.getDefault().serviceDoneExecuting(data.token, 0, 0, 0);
                  }
                  ensureJitEnabled();
              } catch (RemoteException ex) {}
          } catch (Exception e){
              if (!mInstrumentation.onException(s, e)) {
                  throw new RuntimeException ("Unable to bind to service" + s + "with" + data.intent + ":" + e.toString(), e);
              }
          }
      }
  }
  ```
  
  - AMS#publishService
  - 将具体工作交给 ActiveServices 类型的 mServices 对象处理
    - ActiveServices 的 publishServiceLocked 核心代码：`c.conn.connected(r.name, service)`， c 类型是 ConnectionRecord，c.conn 类型是 ServiceDispatcher.InnerConnection，service 是 Service 的 onBind 返回的 Binder 对象
  
  ```java
  public void publishService (IBinder token, Intent intent, IBinder service) {
      //拒绝可能泄漏的文件描述符
    if (intent != null && intent.hasFileDescriptors() == true) {
          throw new IllegalArgumentException ("File descriptors passed in Intent");
      }
  	synchronized (this) {
          if (!(token instanceof ServiceRecord)) {
              throw new IllegalArgumentException("Invalid service token");
          }
          mServices.publishServiceLocked((ServiceRecord)token, intent, service);
      }
  }
  ```
  
  - ServiceDispatcher.InnerConnection
  
  ```java
private static class InnerConnection extends IServiceConnection.Stub{
      final WeakReference<LoadedApk.ServiceDispatcher> mDispatcher;
    InnerConnection (LoadedApk.ServiceDispatcher sd) {
          mDispatcher = new WeakReference<LoadedApk.ServiceDispatcher>(sd);
      }
      
      public void connected (ComponentName name, IBinder service) throws RemoteException {
          LoadedApk.ServiceDispatcher sd = mDispatcher.get();
          if (sd != null){
              sd.connected (name, service);
          }
      }
  }
  ```
  
  - ServiceDispatcher#connected
    - ServiceDispatcher 的 mActivityThread 是 ActivityThread 的 H，mActivityThread 不为 null 时 RunConnection 可经 H 的 post 运行在主线程中，客户端 ServiceConnection 中的方法在主线程被回调
  
```java
  public void connected (ComponentName name, IBinder service){
      if (mActivityThread != null){
        mActivityThread.post (new RunConnection(name, service, 0));
      }else{
          doConnected (name, service);
      }
  }
```

  - ServiceDispatcher#RunConnection
    - ServiceDispatcher 内部保存客户端 ServiceConnection 对象，可方便地调用 ServiceConnection 的 onServiceConnected

  ```java
private final class RunConnection implements Runnable {
      RunConnection(ComponentName name, IBinder service, int command){
          mName = name;
        mService = service;
          mCommand = command;
      }
      
      public void run() {
          if (mCommand == 0){
              doConnected (mName, mService);
          } else if (mCommand == 1){
              doDeath (mName, mService);
          }
      }
      final ComponentName mName;
      final IBinder mService;
      final int mCommand;
  }
  ```

#### 9.3、BroadcastReceiver 工作过程

- 使用：

  - 定义广播接收者：继承 BroadcastReceiver 并重写 onReceive

  ```java
  public class MyReceiver extends BroadcastReceiver {
      @Override
      public void onReceive(Context context, Intent intent){
          //onReceive不能做耗时事情，参考值:10s以内
          Log.d("scott", "on receive action=" + intent.getAction());
          String action = intent.getAction();
          // do some works
      }
  }
  ```

  - 注册广播接收者：在 AndroidManifest 文件中静态注册、通过代码动态注册

    - 静态注册

    ```xml
    <receiver android:name=".MyReceiver">
        <intent-filter>
            <action android:name="com.ryg.receiver.LAUNCH"/>
        </intent-filter>
    </receiver>
    ```

    - 动态注册（需解注册）

    ```java
    IntentFilter filter = new IntentFilter();
    filter.addAction ("com.ryg.receiver.LAUNCH");
    registerReceiver (new MyReceiver(), filter);
    ```

  - 发送广播

  ```java
  Intent intent = new Intent ();
  intent.setAction ("com.ryg.receiver.LAUNCH");
  sendBroadcast (intent);
  ```

- 广播注册过程（动态注册）

  - 静态注册在应用安装时系统自动完成，由 PMS（PackageManagerService）完成，其他三大组件也在应用安装时由 PMS 解析注册
  - 动态注册从 ContextWrapper 的 registerReceiver 开始
    - Contextlmpl 的 registerReceiver 调用自己的 registerReceiverIntermal

  ```java
  public Intent registerReceiver(BroadcastReceiver receiver, IntentFilter filter) {
      return mBase.registerReceiver(receiver, filter);
  }
  ```

  - Contextlmpl#registerReceiverIntermal
    - 系统先从 mPackageInfo 获取 IIntentReceiver 对象
    - 采用跨进程方式向 AMS 发送广播注册请求，采用 IIntentReceiver 是因为 BroadcastReceiver 不能直接跨进程传递，IIntentReceiver 是 Binder 接口，具体实现是 LoadedApk.ReceiverDispatcher.InnerReceiver
    - ReceiverDispatcher 内部保存 BroadcastReceiver 和 InnerReceiver，当接收到广播时可以方便调用
      BroadcastReceiver 的 onReceive
    - `ActivityManagerNative.getDefault()` 是 AMS
  
```java
  private Intent registerReceiverInternal(BroadcastReceiver receiver, int userId, IntentFilter filter, String broadcastPermission, Handler scheduler, Context context){
      IIntentReceiver rd = null;
      if (receiver != null){
          if (mPackageInfo != null && context != nul1) {
              if (scheduler == null){
                  scheduler = mMainThread.getHandler();
              }
              rd = mPackageInfo.getReceiverDispatcher(receiver, context, scheduler, mMainThread.getInstrumentation(), true);
          } else{
              if (scheduler == null){ 
                  scheduler = mMainThread.getHandler();
              }
              rd = new LoadedApk.ReceiverDispatcher (receiver, context, scheduler, null, true).getIIntentReceiver();
          }
      }
      try {
          return ActivityManagerNative.getDefault().registerReceiver (mMainThread.getApplicationThread(), mBasePackageName, rd, filter, broadcastPermission, userId);
      } catch (RemoteException e) {
          return null;
      }
  }
```

- ReceiverDispatcher#getReceiverDispatcher
    - 重新创建一个 ReceiverDispatcher 对象并将其保存的 InnerReceiver 对象作为返回值

```java
  public IIntentReceiver getReceiverDispatcher (BroadcastReceiver r, Context context, Handler handler, Instrumentation instrumentation, boolean registered){
      synchronized (mReceivers){
          LoadedApk.ReceiverDispatcher rd = null;
          ArrayMap<BroadcastReceiver, LoadedApk.ReceiverDispatcher> map = null;
          if (registered){
              map = mReceivers.get (context);
              if (map != null) {
                  rd = map.get(r);
              }
          }
          if (rd == null) {
              rd = new ReceiverDispatcher(r, context, handler, instrumentation, registered);
              if (registered) {
                  if (map == null) {
                      map = new ArrayMap<BroadcastReceiver, LoadedApk.ReceiverDispatcher>();
                      mReceivers.put (context, map);
                  }
                  map.put(r, rd);
              }
          }else {
              rd.validate (context, handler);
          }
          rd.mForgotten = false;
          return rd.getIIntentReceiver();
      }
  }
```

- AMS#registerReceiver
    - 注册广播的真正实现
    - 最终把远程 InnerReceiver 对象及 IntentFilter 对象存储起来

```java
  public Intent registerReceiver(IApplicationThread caller, String callerPackage, IIntentReceiver receiver, IntentFilter filter, String permission, intuserId){
      //...
      mRegisteredReceivers.put(receiver.asBinder(), rl);
      BroadcastFilter bf = new BroadcastFilter(filter, rl, callerPackage, permission, callingUid, userId);
      rl.add(bf);
      if (!bf.debugCheck()) {
          Slog.W(TAG, "==> For Dynamic broadast");
      }
      mReceiverResolver.addFilter(bf);
      //...
  }
```

- 广播的发送和接收过程（普通广播）

  - 通过 send 发送广播时，AMS 会查找匹配广播接收者并将广播发给其处理
  - 广播发送类型：普通广播、有序广播、粘性广播
  - 广播的发送开始于 ContextWrapper 的 sendBroadcast，不是 Context 是因为其 sendBroadcast 是抽象方法
  - ContextImpl#sendBroadcast
    - 直接向 AMS 发起了一个异步请求用于发送广播

  ```java
  public void sendBroadcast (Intent intent) {
      warnIfCallingFromSystemProcess ();
      String resolvedType = intent.resolveTypeIfNeeded (getContentResolver()); 
      try {
          intent.prepareToLeaveProcess ();
          ActivityManagerNative.getDefault().broadcastIntent (mMainThread.getApplicationThread(), intent, resolvedType, null, Activity.RESULT_OK, null, null, null, AppOpsManager.OP_NONE, false, false, getUserId());
      } catch (RemoteException e) {}
  }
  ```

  - AMS#broadcastIntent

  ```java
  public final int broadcastIntent(IApplicationThread caller, Intent intent, String resolvedType, IIntentReceiver resultTo, int resultCode, String resultData, Bundle map, String requiredPermission, int appOp, boolean serialized, boolean sticky, int userId) {
      enforceNotIsolatedCaller ("broadcastIntent");
      synchronized(this) {
          intent = verifyBroadcastLocked (intent);
          final ProcessRecord callerApp = getRecordForAppLocked(caller);
          final int callingPid = Binder.getCallingPid();
          final int callingUid = Binder.getCallingUid();
          final long origId = Binder.clearCallingIdentity();
          int res = broadcastIntentLocked(callerApp, callerApp != null ? callerApp.info.packageName : null, intent, resolvedType, iresultTo, resultCode, resultData, map, requiredPermission, appOp, serialized, sticky, callingPid, callingUid, userId);
          Binder.restoreCallingIdentity(origId);
          return res;
      }
  }
  ```
  
  - broadcastIntentLocked 最开始一行：`intent.addFlags(Intent.FLAG_EXCLUDE_STOPPED_PACKAGES) `，
  表示在 Android 5.0 中，默认广播不会发给已停止的应用，从 Android 3.1 开始具有该特性，系统为 Intent 添加两个标记位控制广播是否对处于停止状态的应用起作用
    - FLAG_INCLUDE_STOPPED_PACKAGES：包含已停止应用（优先级高）
    - FLAG_EXCLUDE_STOPPED_PACKAGES：不包含已停止应用（默认）
    - 停止状态：应用安装后未运行、应用被手动或其他应用强停
  - broadcastIntentLocked 根据 intent-filter 查找匹配广播接收者并过滤，将满足条件的广播接收者添到 BroadcastQueue 中，接着 BroadcastQueue 将广播发给相应广播接收者
  
  ```java
if ((receivers != null && receivers.size() > 0) || resultTo != null){
      BroadcastQueue queue = broadcastQueueForIntent (intent);
      BroadcastRecord r = new BroadcastRecord (queue, intent, callerApp, callerPackage, callingPid, callingUid, resolvedType, requiredPermission, appop, receivers, resultTo, resultCode, resultData, map, ordered, sticky, false, userId);
      if (DEBUG_BROADCAST) S1og.v(TAG, "Enqueueing ordered broadcast" + r + ":prev had" + queue.mOrderedBroadcasts.size());
      if (DEBUG_BROADCAST) {
          int seq = r.intent.getIntExtra("seq", -1);
          Slog.i(TAG, "Enqueueing broadcast" + r.intent.getAction() + "seq=" + seq);
      }
      boolean replaced = replacePending && queue.replaceOrderedBroadcastLocked(r);
      if (!replaced) {
          queue.enqueueOrderedBroadcastLocked(r);
          queue.scheduleBroadcastsLocked();
      }
  }
  ```
  
  - BroadcastQueue#scheduleBroadcastsLocked
    - 发送 BROADCAST_INTENT_MSG 类型的消息，BroadcastQueue 收到消息后调用 processNextBroadcast
  
```java
  public void scheduleBroadcastsLocked(){
      if(DEBUG_BROADCAST) Slog.v(TAG, "Schedule broadcasts [" + mQueueName + "]:current=" + mBroadcastsScheduled);
    if(mBroadcastsscheduled){
          return;
      }
      mHandler.sendMessage (mHandler.obtainMessage(BROADCAST_INTENT_MSG,this));
      mBroadcastsScheduled = true;
  }
```

  - BroadcastQueue#processNextBroadcast 处理普通广播
    - 无序广播存储在 mParallelBroadcasts，系统遍历 mParallelBroadcasts 并将广播发给其所有接收者，通过 deliverToRegisteredReceiverLocked 实现，内部调用 performReceiveLocked 完成

```java
  //首先，立即发送任何非序列化广播
  while (mParallelBroadcasts.size() > 0){
    r = mParallelBroadcasto.remove(O);
      r.dispatchTime = SystemClock.uptimeMi1lis();
      r.dispatchclockTime = System.currentTimeMillis();
      final int N = r.receivers.size();
      if（DEBUG_BROADCAST_LIGHT) Slog.v(TAG,"Processing parallel broadcast [" + mQueueName + "]" + r);
      for(int i=0;i<N;i++){
          Object target = r.receivers.get(i);
          if (DEBUG_BROADCAST) Slog.v(TAG, "Delivering non-ordered on [ " + mQueueName + "] to registered" + target + ":" + r);
          deliverToRegisteredReceiverLocked(r, (BroadcastFilter)target, false);
      }
      addBroadcastToHistoryLocked(r);
      if(DEBUG_BROADCAST_LIGHT) Slog.v(TAG,"Done with parallel broadcast["
  + mQueueName + "]" + r);
  }
```

  - performReceiveLocked 
    - app.thread 指 ApplicationThread

```java
  private static void performReceiveLocked(ProcessRecord app, IIntentReceiver receiver, Intent intent, int resultCode, String data, Bundle extras, boolean ordered, boolean sticky, int sendingUser) throws RemoteException{
      //使用单向绑定器调用将intent异步发送给接收方
    if (app != null){
          if (app.thread != null){
              //如果有一个应用程序线程，则通过该线程调用，以便与其他单向调用一起正确排序
              app.thread.scheduleRegisteredReceiver(receiver, intent, resultCode, data, extras, ordered, sticky, sendingUser, app.repProcState);
          }else{
              //应用程序已死亡。接收器不存在
              throw new RermoteException ("app.thread must not be null");
          }
      }else {
          receiver.performReceive(intent, resultCode, data, extras, ordered, sticky, sendingUser);
      }
  }
```

  - ApplicationThread#scheduleRegisteredReceiver
    - 通过 InnerReceiver 实现广播接收，InnerReceiver 的 performReceive 调用 LoadedApk.ReceiverDispatcher 的 performReceive

```java
  public void scheduleRegisteredReceiver(IIntentReceiver receiver, Intent intent, int resultCode, String dataStr, Bundle extras, boolean ordered, boolean sticky, int sendingUser,int processState) throws RemoteException{
      updateProcessState(processState, false);
    receiver.performReceive(intent, resultCode, dataStr ,extras, ordered,
  sticky, sendingUser);
  }
```

  - LoadedApk.ReceiverDispatcher#performReceive
    - 创建 Args 对象并通过 mActivityThread 的 post 执行 Args 的逻辑，其实现 Runnable 接口，mActivityThread 是 Handler（ActivityThread 的 mH，ActivityThread 内部类 H）

```java
  public void performReceive (Intent intent, int resultcode, String data,
  Bundle extras, boolean ordered, boolean sticky, int sendingUser){
    if(ActivityThread. DEBUG_BROADCAST) {
          int seq = intent.getIntExtra("seg", -1);
          Slog.i(ActivityThread.TAG, "Enqueueing broadcast " + intent.getAction() + " seq=" + seq + " to" + mReceiver);
      }
      Args args = new Args(intent, resultCode, data, extras, ordered, sticky, sendingUser);
      if(!mActivityThread.post (args)){
          if(mRegistered && ordered){
              IActivityManager mgr = ActivityManagerNative.getDefault();
              if (ActivityThread.DEBUG_BROADCAST) Slog.i(ActivityThread.TAG,
  "Finishing sync broadcast to" + mReceiver);
              argssendFinished (mgr);
          }
      }
  }
```

  - Args#run
    - BroadcastReceiver 的 onReceive 被执行，应用已收到广播，同时 onReceive 是在广播接收者的主线程中被调用

```java
  final BroadcastReceiver receiver = mReceiver;
  receiver.setPendingResult (this);
receiver.onReceive (mContext, intent);
```

#### 9.4、ContentProvider 工作过程

- ContentProvider 通过 Binder 向其他组件、应用提供数据，当进程启动时，ContentProvider 会同时启动并被发布到 AMS 中，其 onCreate 先于 Application 的 onCreate 执行
- AMS 的 attachApplication 会调用 ApplicationThread 的 bindApplication，经过 ActivityThread 的 mH Handler 切换到 ActivityThread 的 handleBindApplication，其创建 Application 对象并加载 ContentProvider，ActivityThread 先加载 ContentProvider，再调用 Application 的 onCreate
- 通过增删改查 insert、delete、update、query 四个方法操作 ContentProvider 的数据源，其都通过 Binder 调用，只能通过 AMS 根据 Uri 获取对应 ContentProvider 的 Binder 接口 IConentProvider，通过其访问 ContentProvider 的数据源

![安卓开发艺术探索_ContentProvider启动过程](Image.assets\安卓开发艺术探索_ContentProvider启动过程.jpg)

- 一般 ContentProvider 是单例的，由其 android:multiprocess 属性决定，false（默认值）是单实例，true 为多实例，在每个调用者进程中都存在一个 ContentProvider 对象
- 访问 ContentProvider 通过 ContentResolver，是抽象类，通过 Context 的 getContentResolver 获取 ApplicationContentResolver 对象，ApplicationContentResolver 继承 ContentResolver 并实现其抽象方法
- 通过 ContentProvider 四个方法都可触发 ContentProvider 的启动过程，通过 acquireProvide 获取 ContentProvider
- ApplicationContentResolver#acquireProvider
  - 调用 ActivityThread 的 acquireProvider

```java
protected IContentProvider acquireProvider(Context context, String auth){
    return mMainThread.acquireProvider(context, ContentProvider.getAuthorityWithoutUserId(auth), resolveUserIdEromAuthority(auth), true);
}
```

- ActivityThread#acquireProvider
  - 先从 ActivityThread 查找是否存在目标 ContentProvider，存在直接返回
  - ActivityThread 通过 mProviderMap 存储已启动的 ContentProvider 对象，
  - mProviderMap：`final ArrayMap<ProviderKey, ProviderClientRecord> mProviderMap = new ArrayMap<>();` 
  - 不存在就发送一个进程间请求给 AMS 让其启动目标 ContentProvider，通过 installProvider 修改引用计数。AMS 先启动 ContentProvider 所在进程，由 AMS 的 startProcessLocked 完成，其通过 Process 的 start 完成新进程的启动，启动后入口方法为 ActivityThread 的 main 方法，再启动 ContentProvider

```java
public final IcontentProvider acquireProvider(Context c, String auth, int userId, boolean stable){
    final IContentProvider provider = acquireExistingProvider(c, auth,userId, stable);
    if(provider != null) {
        return provider;
    }
    //可能另一线程同时尝试获取同一提供程序,要确保第一个获得，不能在获取和安装提供程序时保持锁，因为可能需要很长时间才能运行，在提供程序处于同一进程时，可能是重新进入的
    IActivityManager.contentProviderHolder holder = null;
    try {
        holder = ActivityManagerNative.getDefault().getContentProvider (getApplicationThread(), auth, userId, stable);
    }catch(RemoteException ex){}
    if (holder == null) {
        Slog.e (TAG, "Eailed to find provider info for " + auth);
        return null;
    }
    //安装提供程序将增加引用计数，并打破比赛中的任何关系。
    holder = installProvider(c, holder, holder.info, true /*noisy*/, holder.noReleaseNeeded, stable);
    return holder.provider;
}
```

- ActivityThread#main
  - main 方法是静态方法，先创建 ActivityThread 实例并调用 attach 进行一系列初始化，接着开始进行消息循环，attach 将 ApplicationThread 对象通过 AMS 的 attachApplication 方法跨进程传给 AMS，调用 attachApplication -- attachApplicationLocked -- ApplicationThread 的 bindApplication，发送 BIND_APPLICATION 类型消息给 Handler mH，收到后调用 ActivityThread 的 handleBindApplication

```java
public static void main (String[] args){
    SamplingProfilerIntegration.start();
    //CloseGuard默认为true，在这里disable，稍后（通过StrictMode）在调试构建中有选择地启用，但是使用DropBox，而不是logs
    CloseGuard.setEnabled(false);
    Environment.initForCurrentUser();
    //在libcore中设置事件日志的报告程序
    EventLogger.setReporter (new EventLoggingReporter());
    Security.addProvider(new AndroidKeyStoreProvider());
    //确保TrustedCertificatestore在正确位置查找CA证书
    final File configDir = Environment.getUserConfigDirectory (UserHandle.myUserid());
    TrustedcertificateStore.setDefaultUserDirectory(configDir);
    Process.setArgvO("<pre-initialized>");
    Looper.prepareMainLooper();
    ActivityThread thread = new ActivityThread();
    thread.attach(false);
    if(sMainThreadHandler == nul1) {
        sMainThreadHandler = thread.getHandler();
    }
    AsyncTask.init();
    if(false) {
        Looper.myLooper().setMessageLogging (new LogPrinter(Log.DEBUG, "ActivityThread"));
    }
    Looper.loop();
    throw new RuntimeException("Main thread loop unexpectedly exited");
}
```

- ActivityThread 的 handleBindApplication 完成 Application 的创建及 ContentProvider 的创建

  - 创建 ContextImpl 和 Instrumentation

  ```java
  ContextImpl instrContext = ContextImpl.createAppContext(this, pi);
  try{
      java.lang.classLoader cl = instrContext.getclassLoader();
      mInstrumentation = (Instrumentation)cl.loadClass (data.instrumentationName.getClassName()).newInstance();
  }catch (Exception e){
      throw new RuntimeException("Unable to instantiate instrumentation " + data.instrumentationName + ":" + e.toString(), e);
  }
  mInstrumentation.init(this, instrContext, appContext, new ComponentName(ii.packageName, ii.name), data.instrumentationWatcher,
  data.instrumentationUiAutomationConnection);
  ```

  - 创建 Application 对象

  ```java
  Application app = data.info.makeApplication(data.restrictedBackupMode, null);
  mInitialApplication = app;
  ```

  - 启动当前进程的 ContentProvider 并调用其 onCreate

    - installContentProviders 完成 ContentProvider 的启动工作，先遍历当前进程的 ProviderInfo 列表并一一调用 installProvider 启动，将已启动 ContentProvider 发布到 AMS 中，AMS 把其存在 ProviderMap 中，外部调用者可直接从 AMS 获取

    ```java
    List<ProviderInfo> providers = data.providers;
    If(providers != null) {
        installcontentProviders(app, providers);
        //对于包含内容提供者的流程，希望确保“在某个时候”启用JIT
        mH.sendEmptyMessageDelayed(H.ENABLE_JIT, 10*100);
    }
    ```
    - installcontentProviders

    ```java
    private void instal1contentProviders(Context context, List<ProviderInfo> providers){
        final ArrayList<IActivityManager.ContentProviderHolder> results =
    new ArrayList<>();
        for (ProviderInfo opi : providers) {
            if (DEBUG_PROVIDER){
                StringBuilder buf = new StringBuilder(128);
                buf.append("Pub ");
                buf.append(epi.authority);
                buf.append(":");
                buf.append(cpi.name);
                Log.i(TAG, buf.toString());
            }
            IActivityManager.ContentProviderHolder cph = installProvider(context, null, cpi, false /*noisy*/, true /*不需释放*/, true /*stable*/);
            if (eph != null){
                cph.noReleaseNeeded = true;
                results.add(cph);
            }
        }
        try{
            ActivityManagerNative.getDefault().publishContentProviders (getApplicationThread(), results);
        }catch (RemoteException ex){}
    }
    ```

    - installProvider

      > 通过类加载器完成 ContentProvider 对象的创建
      >
      > 通过 ContentProvider 的 attachInfo 调用其 onCreate

    ```java
    final java.lang.classLoader cl = c.getClassLoader();
    localProvider = (contentProvider) ci.loadclass(info.name).newInstance();
    provider = localProvider.getIContentProvider();
    if (provider == null){
        Slog.e(TAG, "Failed to instantiate class " + info.name + " from sourceDir " + info.applicationInfo.sourceDir);
        return null;
    }
    if(DEBUG_PROVIDER) Slog.v(TAG, "Instantiating local provider " + info.name);
    //XXX需为此提供程序创建正确的上下文
    localProvider.attachInfo(c, info);
    ```

    - ContentProvider#attachInfo

    ```java
    private void attachInfo(Context context, ProviderInfo info, boolean testing){
        //...
        if (mcontext == nul1) {
            mContext = context;
            if (context != null){
                mTransport.mApp0psManager = (App0psManager) context.getSystemService(Context.APP_OPS_SERVICE);
            }
            mMyUid = Process.myUid();
            //...
            contentProvider.this.onCreate();
        }
    }
    ```

  - 调用 Application 的 onCreate

    - ContentProvider 所在进程完成启动，其他应用可通过 AMS 访问其，获得 ContentProvider 的 Binder 类型对象 IContentProvider，具体实现是 ContentProviderNative 和 ContentProvider.Transport，其继承 ContentProviderNative

    ```java
    try{
        mInstrumentation.callApplicationonCreate(app);
    }catch(Exception e) {
        if (!mInstrumentation.onException (app, e)) {
            throw new RuntimeException("Unable to create application " + app.getclass().getName() + ":" + e.toString(), e);
        }
    }
    ```

    - query 方法，其他应用通过 AMS 获取 IContentProvider，实现者是 ContentProvider.Transport，其他应用调用 IContentProvider 的 query 最终以进程间通信方式调用 ContentProvider.Transport 的 query

    > 调用 ContentProvider 的 query，其执行结果通过 Binder 返回给调用者，insert、delete、update 类似

    ```java
    public cursor query (string callingPkg, Uri uri, string[] projection, String selection, String[] selectionArgs, String sortorder, ICancellationSignal cancellationSignal) {
        validateIncominguri(uri);
        uri = getUriWithoutUserId(uri);
        if (enforceReadPermission(callingPkg, uri) != App0psManager.MODEALLOWED) {
            return rejectQuery (uri, projection, selection, selectionArgs, sortOrder, CancellationSignal.fromTransport(cancellationSignal));
        }
        final String original = setCallingPackage(callingPkg);
        try {
            return contentProvider.this.query(uri, projection, selection, selectionArgs, sortOrder, CancellationSignal.fromTransport (cancellationsignal));
        }finally{
            setCallingPackage(original);
        }
    }
    ```

### 10、Android 消息机制

#### 10.1、Handler 概述

- Handler
  - MessageQueue 采用单链表的数据结构存储消息列表
  - Looper 以无限循环形式查找是否有新消息，有就处理，否则一直等待。ThreadLocal 可在不同线程中互不干扰地存储并提供数据，通过 ThreadLocal 可轻松获取每个线程的 Looper，线程默认没有 Looper，使用 Handler 必须为线程创建 Looper
- Android 规定访问 UI 只能在主线程中进行，ViewRootlmpl 对 UI 操作做了验证，由 ViewRootImpl 的 checkThread 完成，因为 Android 的 UI 控件不是线程安全的，多线程并发访问可能导致 UI 控件处于不可预期状态，而锁机制：让 UI 访问逻辑变复杂、降低 UI 访问效率，因为锁机制会阻塞某些线程的执行

```java
void checkThread(){
    if (mThread != Thread.currentThread()){
        throw new CalledFromWrongThreadException("Only the original thread that created a view hierarchy can touch its views.");
    }
}
```

- 通过 Handler 的 post 将 Runnable 投到 Handler 内的 Looper 处理，也可通过 Handler 的 send 发送消息，post 最终通过 send 完成
- Handler 的 send 调用 MessageQueue 的 enqueueMessage 将该消息放入消息队列，Looper发现有新消息就会处理，消息的 Runnable 或 Handler 的 handleMessage 方法会被调用，Looper 运行在创建 Handler 所在线程，Handler 中的逻辑就被切换到创建 Handler 所在线程中执行

![安卓开发艺术探索_Handler工作流程](Image.assets\安卓开发艺术探索_Handler工作流程.png)

#### 10.2、ThreadLocal 工作原理

- ThreadLocal：线程内部的数据存储类，存储后只有在指定线程中可获取到存储的数据，其他线程无法获取。
- Looper. ActivityThread 及 AMS 都用到 ThreadLocal，如果不用 ThreadLocal，系统需提供全局哈希表供 Handler 查找指定线程的 Looper
- ThreadLocal 另一使用场景是复杂逻辑下的对象传递，如监听器的传递，有时一个线程任务过于复杂，需要监听器能贯穿整个线程的执行过程，ThreadLocal 可让监听器作为线程全局对象，线程内通过 get 就可获取。如果不用 ThreadLocal，将监听器通过参数形式在函数调用栈中传递（当函数调用栈很深时，程序设计很糟糕），或将监听器作为静态变量供线程访问（不具有可扩充性，如 10 个线程并发执行，要提供 10 静态变量）

- 内部实现

  - ThreadLocal 是泛型类，定义 public class ThreadLocal<T>
  - ThreadLocal#set
    - 通过 values 获取当前线程的 ThreadLocal，Thread 类内部有一个成员用于存储线程 ThreadLocal 数据：ThreadLocal.Values localValues
    - 如果 localValues 为 null，需对其初始化，再将 ThreadLocal 值存储，localValues 内有个数组：private Object[] table，存储 ThreadLocal 值

  ```java
  public void set(T value){
      Thread currentThread = Thread.currentThread ();
      Values values = values(currentThread);
      if(values == null){
          values = initializeValues(currentThread);
      }
      values.put(this, value);
  }
  ```

  - localValues#put
    - 存储规则：ThreadLocal 值在 table 数组的存储位置为 ThreadLocal 的 reference 字段所标识对象的下一位置，如 ThreadLocal 的 reference 对象在 table 数组索引为 index，ThreadLocal 值在 table 数组索引是 index+1，ThreadLocal 值被存在 table 数组中：table[index + 1] = value

  ```java
  void put (ThreadLocal<?> key, Object value) {
      cleanUp ();
      //追踪第一个Tombstone，这我们想回去的地方，如果需要添加一个条目
      int firstTombstone = -1;
      for (int index = key.hash&mask;;index = next(index)){
          Objectk = table[index];
          if(k == key.reference) {
              //替换现有条目
              table[index + 1] = value;
              return;
          }
          if(k == null){
              if(firstTombstone == -1){
                  //填充空槽
                  table[index] = key.reference;
                  table[index + 1] = value;
                  size++;
                  return;
              }
              //回去换第一个Tombstone
              table[firstTombstone] = key.reference;
              table[firstTombstone + 1] = value;
              tombstones--;
  			size++;
              return;
          }
          //记住第一个Tombstone
          if (firstTombstone == -1 && k == TOMBSTONE){
              first Tombstone = index;
          }
      }
  }
  ```

  - ThreadLocal#get
    - 取出当前线程的 localValucs 对象，如果为 null 返回初始值，由 ThreadLocal 的 initialValue 描述，默认为 null，也可重写该方法；如果不为 null，取出其 table 数组并找出 ThreadLocal 的 reference 对象在 table 数组的位置，其下一位置存储的数据是 ThreadLocal

  ```java
  public T get(){
      //为快速路径而优化。
      Thread currentThread = Thread.currentThread();
      Values values = values(currentThread);
      if (values != null){
          0bject[] table = values.table;
          int index = hash & values.mask;
          if (this.reference == table[index]){
              return (T)table[index + 1];
          }
      }else{
          values = initializeValues(currentThread);
      }
      return (T)values.getAfterMiss(this);
  }
  ```

#### 10.3、消息队列工作原理

- MessageQueue
  - 读取操作伴随着删除操作，插入、读取对应方法分别为 enqueueMessage、next，enqueueMessage 往消息队列中插入一条消息，next 从消息队列中取出一条消息并将其从消息队列中移除
  - 内部实现通过单链表的数据结构维护消息列表，在插入和删除上有优势
- MessageQueue#enqueueMessage

```java
boolean enqueueMessage (Message msg, long when){
    //...
    synchronized (this){
        //...
        msg.markInUse();
        msg.when = when;
        Message p = mMessages;
        boolean needwake;
        if (p == null || when == 0 || when < p.when){
            //新head，如果被阻止,唤醒事件队列
            msg.next = P;
            mMessages = msg;
            needwake = mBlocked;
        }else {
            //在队列中间插入，通常不需唤醒事件队列，除非在队头有barrier，且消息是队列中最早的异步消息
            needWake = mBlocked && p.target == null && msg.isAsynchronous();
            Message prev;
            for(;;){
                prev = p;
                p = p.next;
                if (p == null || when < p.when) {
                    break;
                }
                if(needwake && p.isAsynchronous()) {
                    needwake = false;
                }
            }
            msg.next = p;	//invariant: p == prev.nextprev.
            next = msg;
        }
        // we can assume mPtr != 0 because mQuitting is false.
        if (needWake) {
            nativewake (mPtr);
        }
    }
    return true;
}
```

- MessageQueue#next
  - 无限循环，如果消息队列中没有消息会一直阻塞。有新消息到来时返回这条消息并将其从单链表中移除

```java
Message next(){
    int pendingIdleHandlerCount = -1;	// -1仅在第一次迭代期间
    int nextPollTimeoutMillis = 0;
    for(;;){
        if (nextPol1TimeoutMillis != 0){
            Binder.flushPendingCommands();
        }
        nativePollOnce(ptr, nextPollTimeoutMillis);
        synchronized (this){
            //尝试检索下一条消息，如找到，返回
            final long now = SystemCloek.uptimeMillis();
            Message prevMsg = null;
            Message msg = mMessages;
            if (msg != null && msg.target == null) {
                //被barrier阻挡，在队列中查找下一个异步消息
                do{
                    prevMsg = msg;
                    msg = msg.next;
                } while (msg != null && !msg.isAsynchronous());
            }
            if (msg != null){
                if(now < msg.when){
                    //下一条消息尚未准备好，设置超时时间，在它准备好时唤醒
                    nextPollTimeoutMillis = (int) Math.min(msg.when - now, Integer.MAX_VALUE);
                }else{
                    // Got a message.
                    mBlocked = false;
                    if (prevMsg != null){
                        prevMsg.next = msg.next;
                    }else{
                        mMessages = msg.next;
                    }
                    msg.next = null;
                    if(false) Log.v("MessageQueue", "Returning message: " + msg);
                    return msg;
                }
            } else{
                // No more messages.
                nextPollTimeoutMillis = -1;
            }
            //...
        }
        //...
    }
}
```

#### 10.4、Looper 工作原理

- 构造方法

```java
private Looper (boolean quitAllowed){
    mQueue = new MessageQueue(quitAllowed);
    mThread = Thread.currentThread();
}
```

- 为线程创建 Looper 通过 Looper.prepare，接着通过 Looper.loop 开启消息循环

```java
new Thread("Thread#2"){
    @Override
    public void run(){
        Looper.prepare();
        Handler handler = new Handler();
        Looper.loop();
    }
}.start();
```

- prepareMainLooper：给主线程 ActivityThread 创建 Looper，本质通过 prepare 实现，Looper 提供 getMainLooper 方法，可在任何地方获取到主线程的 Looper
- Looper 也可退出，其提供 quit、quitSafely，区别
  - quit 直接退出 Looper，quitSafely 设定一个退出标记，把消息队列中已有消息处理完毕后安全地退出
  - Looper 退出后通过 Handler 发送消息会失败，Handler 的 send 返回 false
  - 子线程如果手动创建 Looper，在所有事情完成后应该调用 quit 终止消息循环，否则子线程一直处于等待状态，如果退出 Looper 后，线程立刻终止
- Looper#loop
  - 死循环，唯一跳出循环的方式是 MessageQueue 的 next 返回 null
  - 当 Looper 的 quit 被调用时，Looper 调用 MessageQueue 的 quit 或 quitSafely 通知消息队列退出，当消息队列被标记为退出状态时，其 next 返回 null
  - `msg.target.dispatchMessage(msg)` 的 msg.target 是发送这条消息的 Handler 对象，Handler 发送的消息最终交给其 dispatchMessage 处理，Handler 的 dispatchMessage 在创建 Handler 时的 Looper 中执行，将代码逻辑切换到指定线程中执行

```java
//在此线程中运行消息队列，一定要调用{@link#quit()}结束循环
public static void loop(){
    final Looper me = myLooper();
    if (me == null){
        throw new RuntimeException("No Looper; Looper.prepare() wasn'tcalled on this thread.");
    }
    final MessageQueue queue = me.mQueue;
    //确保此线程标识是本地进程的标识，并跟踪该标识令牌实际是什么
    Binder.clearCallingIdentity();
    final long ident = Binder.clearcallingIdentity();
    for (;;) {
        Message msg = queue.next();	//might block
        if(msg == null){
            //没有消息表示消息队列正在退出
            return;
        }
        //如果UI事件设置logger，则必须在局部变量中
        Printer logging = me.mLogging;
        i(logging != null){
            logging.println(">>>>> Dispatching to" + msg.target + "" + msg.calLback + ":" + msg.what);
        }
        msg.target.dispatchMessage(msg);
        if (logging != null){
            1ogging.println("<<<<< Finished to" + msg.target + "" + msg.callback);
        }
        //确保在调度过程中线程标识没有损坏
        final long newIdent = Binder.clearCallingIdentity();
        if(ident != newIdent){
            Log.wtf(TAG, "Thread identity changed from 0x" + Long.toHexString(ident) + "to 0x" + Long.toHexString(newIdent) + " while dispatching to" + msg.target.getclass().getName() + " " + msg.callback + " what=" + msg.what);
        }
        msg.recycleUnchecked();
    }
}
```

#### 10.5、Handler 工作原理

- 消息发送可通过 post 的一系列方法及 send 的一系列方法实现，post 一系列方法最终通过 send 一系列方法实现

```java
public final boolean sendMessage (Message msg){
    return sendMessageDelayed(msg, 0);
}

public final boolean sendMessageDelayed (Message msg, long delayMillis){
    if(delayMi1lis < 0) {
        delayMillis = 0;
    }
    return sendMessageAtTime(msg, SystemClock.uptimeMillis() + delayMillis);
}

public boolean sendMessageAtTime (Message msg, long uptimeMillis){
    MessageQueue queue = mQueue;
    if(queue == null) {
        RuntimeException e = new RuntimeException(this + " sendMessageAtTime() called with no mQueue");
        Log.w( "Looper", e.getMessage, e);
        return false;
    }
    return enqueueMessage(queue, msg, uptimeMillis);
}

private boolean enqueueMessage (MessageQueue queue, Message msg, long uptimeMillis){
    msg.target = this;
    if(mAsynchronous){
        msg.setAsynchronous(true);
    }
    return queue.enqueueMessage(msg, uptimeMillis);
}
```

- Handler 发送消息仅向消息队列插入消息，MessageQueue 的 next 会返回该消息给 Looper，Looper 收到后开始处理，最终交由 Handler 处理，Handler 的 dispatchMessage 被调用

```java
public void dispatchMessage (Message msg) {
    if (msg.callback != null){
        handlecallback (msg);
    }else {
        if (mcallback != null){
            if (mCallback.handleMessage (msg)){
                return;
            }
        }
        handleMessage (msg);
    }
}
```

- Handler 处理消息过程：

  - 检查 Message 的 callback 是否为 null，不就通过 handleCallback 处理消息，Message 的 callback 是 Runnable 对象，实际是 Handler 的 post 传递的 Runnable 参数

  ```java
  private static void handleCallback (Message message){
      message.callback.run ();
  }
  ```

  - 检查 mCallback 是否为 null，不就调用 mCallback 的 handleMessage 处理消息，Callback 是接口，定义：

  ```java
  public interface Callback {
      public boolean handleMessage (Message msg);
  }
  ```

  - 通过 Callback 可以创建 Handler 对象：`Handler handler = newHandler(callback)`，用来创建一个 Handler 的实例但并不需派生其子类，通常是派生一个 Handler 的子类并重写其 handleMessage 处理具体消息
  - 调用 Handler 的 handleMessage 处理消息

  ![安卓开发艺术探索_Handler消息处理](Image.assets\安卓开发艺术探索_Handler消息处理.png)

- 特殊构造方法
  
  - 通过特定 Looper 构造

```java
public Handler(Looper looper) {
    this(looper, null, false);
}
```

- 默认构造方法 `public Handler()`，会调用下面的构造方法，如果当前线程没有 Looper 会抛出异常

```java
public Handler (Callback callback, boolean async) {
    mLooper = Looper.myLooper();
    if(mLooper == nu1l) {
        throw new RuntimeException("Can't create handler inside thread that has not calledLooper.prepare()");
    }
    mQueue = mLooper.mQueue;
    mCallback = callback;
    mAsynchronous = async;
}
```

#### 10.6、主线程消息循环

- Android 主线程是 ActivityThread，主线程入口方法为 main，系统通过 `Looper.prepareMainLooper()` 创建主线程 Looper 及 MessageQueue，通过 `Looper.loop()` 开启主线程消息循环

```java
public static void main (String[] args) {
    Process.setArgV0("<pre-initialized>");
    Looper.prepareMainLooper();
    ActivityThread thread = new ActivityThread();
    thread.attach(false);
    if(sMainThreadHandler == null){
        sMainThreadHandler = thread.getHandler();
    }
    AsyncTask.init();
    if (false) {
        Looper.myLooper().setMessageLogging(new LogPrinter(Log.DEBUG, "ActivityThread"));
    }
    Looper.loop();
    throw new RuntimeException("Main thread loop unexpectedly exited");
}
```

- ActivityThread 需要 ActivityThread.H 和消息队列交互，其内部定义一组消息类型，包含四大组件启动和停止等过程

```java
private class H extends Handler {
    public static final int LAUNCH_ACTIVITY = 100;
    public static final int PAUSE_ACTIVITY = 101;
    public static final int PAUSE_ACTIVITY_FINISHING = 102;
    public static final int STOP_ACTIVITY_SHOW = 103;
    public static final int STOP_ACTIVITY_HIDE = 104;
    public static final int SHOW_WINDOW = 105;
    public static final int HIDE_WINDOW = 106;
    public static final int RESUME_ACTIVITY = 107;
    public static final int SEND_RESULT = 108;
    public static final int DESTROY_ACTIVITY = 109;
    public static final int BIND_APPLICATION = 110;
    public static final int EXIT_APPLICATION = 111;
    public static final int NEW_INTENT = 112;
    public static final int RECEIVER = 113;
    public static final int CREATE_SERVICE = 114;
    public static final int SERVICE_ARGS = 115;
    public static final int STOP_SERVICE = 116;
    //...
}
```

- ActivityThread 通过 ApplicationThread 和 AMS 进程间通信，AMS 完成请求后回调 ApplicationThread 的 Binder 方法，ApplicationThread 向 H 发送消息，H 收到消息后将 ApplicationThread 逻辑切换到 ActivityThread（主线程）执行

### 11、Android 线程和线程池

#### 11.1、AsyncTask

- 一种轻量级异步任务类，可在线程池中执行后台任务，把执行进度和最终结果传给主线程并在主线程更新 UI
- 封装 Thread、Handler，不适合进行特别耗时的后台任务，建议使用线程池
- 抽象泛型类，提供 Params（参数类型）、Progress（后台任务执行进度类型）、Result（后台任务返回结果类型） 泛型参数，可用 Void 代替，声明：`public abstract class AsyncTask<Params, Progress, Result>` 

- 核心方法
  - `onPreExecute()`：主线程执行，在异步任务执行前调用，一般用于做些准备工作
  - `doInBackground(Params...params)`：在线程池中执行，用于执行异步任务，params 参数表示异步任务的输入参数，可通过 publishProgress 更新任务进度，其调用 onProgressUpdate，需返回计算结果给 onPostExecute
  - `onProgressUpdate(Progress...values)`：主线程执行，后台任务执行进度改变时调用
  - `onPostExecute(Result result)`：主线程执行，异步任务执行后调用，result 参数是后台任务（doInBackground）返回值
  - `onCancelled`：主线程执行，异步任务被取消时调用，这时候 onPostExecute 不被调用

- 使用限制

  - AsyncTask 类必须在主线程加载，第一次访问 AsyncTask 必须发生在主线程，Android 4.1 及以上版本系统自动完成
  - AsyncTask 对象必须在主线程创建
  - execute 必须在 UI 线程调用
  - 不要在程序中直接调用 onPreExecute、onPostExecute、doInBackground、onProgressUpdate
  - 一个 AsyncTask 对象只能执行一次（只能调用一次 execute），否则报运行时异常
  - Android 1.6 前，AsyncTask 串行执行任务，Android 1.6 时 AsyncTask 采用线程池处理并行任务，Android 3.0 为避免 AsyncTask 的并发错误，采用一个线程串行执行任务，Android 3.0 以及后续版本可通过 AsyncTask 的 executeOnExecutor 并行执行任务

- 工作原理

  - AsyncTask#execute -- executeOnExecutor
    - sDefaultExecutor：串行线程池，进程中所有 AsyncTask 在其排队执行

  ```java
  public final AsyncTask<Params, Progress, Result> execute (Params... params){
      return executeonExecutor (sDefaultExecutor, params);
  }
  
  public final AsyncTask<Params, Progress,Result> executeOnExecutor (Executor exec, Params... params){
      if (mStatus != Status.PENDING){
          switch (mStatus) {
              case RUNNING:
                  throw new IllegalstateException("Cannot execute task:" + "the task is already running.");
              case FINISHED:
                  throw new IllegalStateException("cannot execute task:" + "the task has already been executed " + "(a task can be executed only once)");
          }
      }
      mStatus = Status.RUNNING;
      onPreExecute();
      mWorker.mParams = params;
      exec.execute(mFuture);
      return this;
  }
  ```

  - 线程池的执行过程

    - 系统把 AsyncTask 的 Params 参数封装为 FutureTask 对象，其是并发类，充当 Runnable 作用，交给 SerialExecutor 的 execute 处理，先把 FutureTask 对象插入任务队列 mTasks 中，如果这时没有正在活动的 AsyncTask，调用 SerialExecutor 的 scheduleNext 执行下一 AsyncTask，当 AsyncTask 执行后，会继续执行其他任务直到所有任务都被执行，默认 AsyncTask 串行执行

    - AsyncTask 两个线程池（SerialExecutor、THREAD_POOL_EXECUTOR），一个 Handler （InternalHandler）

      > SeriaIExecutor：用于任务排队
      >
      > THREAD_POOL_EXECUTOR：用于真正执行任务
      >
      > InternalHandler：用于将执行环境从线程池切到主线

  ```java
  public static final Executor SERIAL_EXECUTOR = new SerialExecutor();
  private static volatile Executor sDefaultExecutor = SERIAL_EXECUTOR;
  
  private static class SerialExecutor implements Executor{
      final ArrayDeque<Runnable> mTasks = new ArrayDeque<Runnable>();
      Runnable mActive;
      
      public synchronized void execute (final Runnable r){
          mTasks.offer(new Runnable(){
              public void run(){
                  try{
                      r.run();
                  }finally{
                      scheduleNext();
                  }
              }
          });
          if(mActive == null){
              scheduleNext();
          }
      }
      
      protected synchronized void scheduleNext(){
          if ((mActive = mTasks.poll()) != null){
              THREAD_POOL_EXECUTOR.execute (mActive);
          }
      }
  }
  ```

  - 构造方法
    - FutureTask 的 run 调用 mWorker 的 call，其最终在线程池中执行
    - 将 mTaskInvoked 设 true，表示当前任务已被调用，执行 AsyncTask 的 doInBackground，将其返回值传给 postResult

  ```java
  mworker = new workerRunnable<Params, Result>(){
      public Resultcall() throws Exception {
          mTaskInvoked.set(true);
          Process.setThreadPriority(Process.THREAD_PRIORITY_BACKGROUND);
          //未检查
          return postResult(doInBackground (mParams));
      }
  };
  ```

  - postResult
    - postResult 通过 sHandler 发送 MESSAGE_POST_RESULT 消息

  ```java
  private Result postResult (Result result) {
      @SupPressWarnings("unchecked")
      Message message = sHandler.obtainMessage(MESSAGE_POST_RESULT, new AsyneTaskResult<Result>(this, result));
      message.sendToTarget();
      return result;
  }
  ```

  - sHandler 定义
    - 静态 Handler 对象，在主线程创建（才能将运行环境切换到主线程）
    - AsyncTask 类必须在主线程中加载（静态成员在加载类时初始化），否则同一进程的 AsyncTask 都无法正常工作
    - sHandler 收到 MESSAGE_POST_RESULT 后调用 AsyncTask 的 finish

  ```java
  private static final InternalHandler sHandler = new InternalHandler();
  
  private static class InternalHandler extends Handler {
      @Suppresswarnings ({"unchecked", "RawUseofParameterizedType"})
      @verride
      public void handleMessage(Message msg){
          AsyncTaskResult result = (AsyncTaskResult) msg.obj;
          switch (msg.what){
              case MESSAGE_POST_RESULT:
                  //结果只有一个
                  resultmTask.finish(result.mData[0]);
                  break;
              case MESSAGE_POST_PROGRESS:
                  result.mTask.onProgressUpdate (result.mData);
                  break;
          }
      }
  }
  ```

  - AsyncTask#finish
    - 如果 AsyncTask 被取消执行，调用 onCancelled，否则调用 onPostExecute，doInBackground 返回结果传给 onPostExecute
    - Android 3.0 开始，默认 AsyncTask 串行执行

  ```java
  private void finish (Result result){
      if(isCancelled()) {
          onCancelled(result);
      }else{
          onPostExecute(result);
      }
      mStatus = Status.FINISHED;
  }
  ```

#### 11.2、HandlerThread

- HandlerThread 继承 Thread
- 实现：在 run 中通过 `Looper.prepare()` 创建消息队列，通过 `Looper.loop()` 开启消息循环

```java
public void run (){
    mTid = Process.myTid();
    Looper.prepare();
    synchronized (this) {
        mLooper = Looper.myLooper();
        notifyAll();
    }
    Process.setThreadPriority(mPriority);
    onLooperPrepared();
    Looper.1oop();
    mTid = -1;
}
```

- 和普通 Thread 不同：
  - 普通 Thread 主要用在 run 执行耗时任务，HandlerThread 在内部创建消息队列，外界通过 Handler 消息方式通知 HandlerThread 执行具体任务
- HandlerThread 在 Android 具体使用场景是 IntentService
- HandlerThread 的 run 无限循环，不用时通过其 quit、quitSafely 终止线程

#### 11.3、IntentService

- IntentService：继承 Service，抽象类，必须创建其子类才能使用，用于执行后台耗时任务，执行后自动停止，由于服务原因，其优先级比单纯线程高很多，适合执行高优先级后台任务
- IntentService 封装 HandlerThread 和 Handler，其 onCreate 当 IntentService 被第一次启动时调用
  - 创建 HandlerThread，使用其 Looper 构造 Handler 对象 mServiceHandler，通过 mServiceHandler 发送的消息最终在 HandlerThread 执行
  - 每次启动 IntentService, 其 onStartCommand 调用一次，其处理每个后台任务的 Intent，其调用 onStart

```java
public void onCreate (){
    // TODO:最好有一个在处理过程中保持部分wakelock的选项，并有一个启动服务并传递wakelock的静态startService(Context, Intent)方法
    super.onCreate();
    HandlerThread thread = new HandlerThread ("IntentService[" + mName + "]");
    thread.start();
    mServiceLooper = thread.getLooper ();
    mServiceHandler = new ServiceHandler (mServiceLooper);
}
```

- onStart
  - IntentService 通过 mServiceHandler 发送消息，在 HandlerThread 处理
  - mServiceHandler 收到消息后将 Intent 对象传给 onHandleIntent 处理，其内容和 `startService(intent)` 的一致， 通过其解析出外界启动 IntentService 时所传参数，可区分具体后台任务，在 onHandleIntent 可对不同后台任务处理
  - onHandleIntent 执行结束后，IntentService 通过 `stopSelf(int startId)` 尝试停止服务，不采用 `stopSelf()` 停止是因为其会立刻停止，这时可能有其他消息未处理，`stopSelf(int startId)` 会等待所有消息处理完后才终止
  - `stopSelf(int startld)` 尝试停止服务前判断最近启动服务次数是否和 startId 相等，相等就立刻停止，不相等则不停止

```java
public void onStart (Intent intent, int startId){
    Message msg = mServiceHandler.obtainMessage ();
    msg.arg1 = startId;
    msg.obj = intent;
    mServiceHandler.sendMessage (msg);
}
```

- ServiceHandler

```java
private final class ServiceHandler extends Handler {
    public ServiceHandler (Looper looper) {
        super (looper);
    }
    @Override
    public void handleMessage (Message msg){
        onHandleIntent ((Intent)msg.obj);
        stopSelf (msg.arg1);
    }
}
```

- IntentService 的 onHandleIntent 是抽象方法，需在子类中实现，作用是从 Intent 参数区分具体任务并执行,如果目前只存在一个任务，onHandleIntent 执行完后，`stopSelf(int startId)` 会直接停止；如果多个，当 onHandleIntent 执行完最后一个任务时，`stopSelf(int startld)` 才会直接停止
- 每执行一个后台任务启动一次 IntentService，IntentService 内部通过消息方式向 HandlerThread 请求执行任务，Handler 的 Looper 顺序处理消息，IntentService 也是

#### 11.4、Android 线程池

- 优点

  - 重用线程池的线程，避免线程的创建、销毁带来的性能开销
  - 有效控制线程池的最大并发数，避免大量线程间因互相抢占系统资源导致阻塞
  - 对线程进行简单管理，提供定时执行及指定间隔循环执行等功能

- Java 的 Executor：java 的线程池，接口

- ThreadPoolExecutor

  - 线程池真正实现，构造方法提供一系列参数配置线程池
  - 常用构造方法

  ```java
  public ThreadPoolExecutor(int corePoolSize, int maximumPoolSize, long keepAliveTime, TimeUnit unit, BlockingQueue<Runnable> workQueue, ThreadFactory threadFactory)
  ```

  - corePoolSize：核心线程数，默认核心线程在线程池中一直存活，如果 ThreadPoolExccutor 的 allowCoreThreadTimeOut 属性设置 true，闲置的核心线程在等待新任务时有超时策略，时间间隔由 keepAliveTime 指定，等待时间超过时，核心线程被终止
  - maximumPoolSize：线程池容纳最大线程数，当活动线程数达到该数值，后续新任务会被阻塞
  - keepAliveTime：非核心线程闲置超时时长，超过该时长非核心线程被回收，当 ThreadPoolExecutor 的 allowCoreThreadTimeOut 属性设为 true 时会作用于核心线程
  - unit：指定 keepAliveTime 参数时间单位，是枚举，有 TimeUnit.MILLISECONDS（毫秒）、TimeUnit.SECONDS（秒）、TimeUnit.MINUTES（分钟）等
  - workQueue：任务队列，通过线程池的 execute 提交的 Runnable 对象存储在其
  - threadFactory：线程工厂，为线程池提供创建新线程功能，接口，一个方法：`Thread newThread(Runnable r)`
  - RejectedExecutionHandler handler：当线程池无法执行新任务时，可能由于任务队列已满或无法成功执行，这时 ThreadPoolExecutor 调用 handler 的 rejectedExecution 通知调用者，默认其直接拋出一个 RejectedExecutionException
    - 可选值：CallerRunsPolicy、AbortPolicy（默认值，直接抛出 RejectedExecutionException）、 DiscardPolicy、DiscardOldestPolicy

- 遵循规则

  - 如果线程池线程数量未达到核心线程数量，直接启动一个核心线程执行任务
  - 如果线程池线程数量已达或超核心线程数量，任务被插到任务队列中排队等待
  - 如果步骤 2 无法将任务插到任务队列中，往往是任务队列已满，如果线程数量未达线程池规定最大值，立刻启动一个非核心线程执行任务
  - 如果步骤 3 线程数量已达线程池规定最大值，拒绝执行任务，ThreadPoolExecutor 调用 RejectedExecutionHandler 的 rejectedExecution 通知调用者

- 线程池分类

  - FixedThreadPool：通过 Executors 的 newFixedThreadPool 创建，线程数量固定

    - 当线程处于空闲状态时不会被回收，除非线程池关闭。当所有的线程都处于活动状态时，新任务都处于等待状态，直到有线程空闲
    - 只有核心线程且没有超时机制，任务队列也没有大小限制
    - newFixedThreadPool

    ```java
    public static ExecutorService newFixedThreadPool (int nThreads) {
        return new ThreadPoolExecutor(nThreads, nThreads, OL, TimeUnit.MILLISECONDS, new LinkedBlockingQueue<Runnable>());
    }
    ```

  - CachedThreadPool：通过 Executors 的 newCachedThreadPool 创建，线程数不定

    - 只有非核心线程，最大线程数为 Integer.MAX_VALUE，当线程池的线程都处于活动状态时，线程池会创建新线程处理新任务，否则利用空闲线程处理新任务
    - 线程池中的空闲线程都有超时机制，超时时长 60 秒
    - 和 FixedThreadPool 不同，其任务队列相当空集合，任何任务都会立即被执行，SynchronousQueue 无法插入任务，其是无法存储元素的队列
    - 适合执行大量耗时较少的任务
    - newCachedThreadPool

    ```java
    public static ExecutorService newCachedThreadPool() {
        return new ThreadPoolExecutor(0, Integer.MAX_ VALUE, 60L, TimeUnit.SECONDS, new SynchronousQueue<Runnable>());
    ```

  - ScheduledThreadPool：通过 Executors 的 newScheduledThreadPool 创建，核心线程数固定，非核心线程数没有限制，当非核心线程闲置时立即回收

    - 主要用于执行定时任务和具有固定周期的重复任务
    - newScheduledThreadPool

    ```java
    public static ScheduledExecutorService newScheduledThreadPool(int corePoolsize){
        return new ScheduledThreadPoolExecutor (corePoolSize);
    }
     
    public ScheduledThreadPoolExecutor (int corePoolSize) {
        super (corePoolSize, Integer.MAX_VALUE, 0, NANOSECONDS, new DelayedWorkQueue());
    }
    ```

  - SingleThreadExecutor：通过 Executors 的 newSingleThreadExecutor 创建，只有一个核心线程，确保所有任务都在同一线程按序执行

    - 统一所有任务到一个线程中，使得这些任务间不需处理线程同步问题
    - newSingleThreadExecutor

    ```java
    public static Executorservice newS ingleThreadExecutor() {
        return new FinalizableDelegatedExecutorService(new ThreadPoolExecutor(1, 1, 0L, TimeUnit.MILLISECONDS, new LinkedBlockingQueue<Runnable>()));
    }
    ```

### 12、Bitmap 加载与 Cache

#### 12.1、Bitmap 加载

- BitmapFactory 类提供四类方法 decodeFile、decodeResource、decodeStream、decodeByteArray，分别用于从文件系统、资源、输入流、字节数组中加载 Bitmap 对象，decodeFile、decodeResource 间接调用 decodeStream，都最终在 Android 底层实现，对应 BitmapFactory 类几个 native 方法

- 高效加载：采用 BitmapFactory.Options 加载所需尺寸图片，按一定采样率加载缩小后的图片，降低内存占用，一定程度避免 OOM，提高 Bitmap 加载性能

  - 四类方法都支持 BitmapFactory.Options 参数，通过 BitmapFactory.Options 缩放图片，主要用到其 inSampleSize 参数（采样率），为 1 时，采样后图片大小为原始大小；大于 1 时，采样后图片宽高均为原图的 1/n，像素数为原图的 1/n^2，占有内存也为原图的 1/n^2，其取值总为 2 的指数，如 1、2、4、8、16 等，如果不是，系统会向下取整并选择一个最接近的 2 的指数代替

  - 获取采样率

    > inJustDecodeBounds 为 true 时，BitmapFactory 只解析图片原始宽高，不真正加载图片，是轻量级操作

    - 将 BitmapFactory.Options 的 inJustDecodeBounds 参数设为 true 并加载图片
    - 从 BitmapFactory.Options 中取出图片原始宽高，对应outWidth、outHeight 参数
    - 根据采样率规则并结合目标 View 所需大小计算采样率 inSampleSize
    - 将 BitmapFactory.Options 的 inJustDecodeBounds 参数设为 false，重新加载图片

    ```java
    public static Bitmap decodeSampledBitmapFromResource(Resources res, int resId, int reqWidth, int reqHeight){
        //首先用inJustDecodeBounds=true解码以检查尺寸
        final BitmapFactory.Options options = new BitmapFactory.options();
        options.inJustDecodeBounds = true;
    	BitmapFactory.decodeResource(res, resId, options); 
        //计算采样值
        options.inSampleSize = calculateInSampleSize(options, reqWidth, reqHeight); 
        //使用inSampleSize集合解码位图
        options.inJustDecodeBounds = false;
        return BitmapFactory.decodeResource(res, resId, options);
    }
    
    public static int calculateInSampleSize(BitmapFactory.options options, int reqWidth, int reqHeight) {
        //图片的原始宽高
        final int height = options.outHeight;
        final int width = options.outwidth;
        int inSampleSize = 1;
        if (height > reqHeight || width > reqWidth) {
            final int halfHeight = height / 2;
            final int halfWidth = width / 2;
            //计算最大的inSampleSize值，是2的幂，并使高度和宽度都大于请求的高度和宽度
            while ((halfHeight / inSampleSize) >= reqHeight && (halfWidth / inSampleSize) >= reqWidth){
                inSampleSize *= 2;
            }
        }
        return inSampleSize;
    }
    ```

#### 12.2、Android 缓存策略

- LruCache：

  > Android 3.1 提供的缓存类，通过 support-v4 兼容包可兼容早期版本
  >
  > 泛型类，内部采用 LinkedHashMap 以强引用方式存储外界缓存对象，提供 get、put 方法完成缓存的获取和添加，缓存满时，LruCache 会移除较早使用的缓存对象，再添加新缓存对象
  >
  > 线程安全

  - 定义

  ```java
  public class LruCache<K, V> {
      private final LinkedHashMap<K, V> map;
      //...
  }
  ```

  - 引用
    - 强引用：直接的对象引用
    - 软引用：当一个对象只有软引用存在时，系统内存不足时此对象被 gc 回收
    - 弱引用：当一个对象只有弱引用存在时，此对象会随时被 gc 回收
  - 初始化
    - 只需提供缓存总容量大小并重写 sizeOf，作用是计算缓存对象大小，特殊情况还需重写 LruCache 的 entryRemoved，LruCache 移除旧缓存时调用 entryRemoved，可完成资源回收工作

  ```java
  int maxMemory = (int) (Runtime.getRuntime().maxMemory() / 1024);
  int cacheSize = maxMemory / 8;
  mMemoryCache = new LruCache<String, Bitmap>(cacheSize) {
      @Override
      protected int sizeOf(String key, Bitmap bitmap) {
          return bitmap.getRowBytes() * bitmap.getHeight() / 1024;
      }
  };
  ```

  - 缓存的获取：`mMemoryCache.get(key)` 
  - 缓存的添加：`mMemoryCache.put(key, bitmap)` 
  - 缓存的添加：通过 remove 可删除一个指定的缓存对象

- DiskLruCache

  > 用于实现存储设备缓存（磁盘缓存），通过将缓存对象写入文件系统实现缓存
  >
  > DiskLruCache 不属于 Android SDK

  - 创建

    - DiskLruCache 提供 open 用于创建自身

    ```java
    public static DiskLruCache open(File directory, int appVersion, int valueCount, long maxSize)
    ```

    - 第一个参数表示磁盘缓存在文件系统中的存储路径，可选择 SD 卡上的缓存目录，具体指 /sdcard/Android/data/package_name/cache 目录，package_name 表示当前应用包名，应用被卸载后，此目录被删除。也可选择 SD 卡上其他目录、data 下当前应用目录，如果应用卸载后希望删除缓存文件选择 SD 卡上的缓存目录，如果希望保留缓存数据选择 SD 卡上其他特定目录
    - 第二个参数表示应用版本号，一般设为 1，版本号改变时 DiskLruCache 会清空之前所有缓存文件
    - 第三个参数表示单节点对应数据个数，一般设为 1
    - 第四个参数表示缓存总大小，如 50MB，缓存大小超出设定值后，DiskLruCache 会清除一些缓存保证总大小不大于设定值

    ```java
    private static final long DISK_CACHE_SIZE = 1024 * 1024 * 50; //50MB
    File diskCacheDir = getDiskCacheDir (mContext, "bitmap");
    if (!diskCacheDir.exists()){
        diskCacheDir.mkdirs();
    }
    mDiskLruCache = DiskLruCache.open(diskCacheDir, 1, 1, DISK_CACHE_SIZE);
    ```

  - 缓存添加

    - 通过 Editor 完成，表示缓存对象的编辑对象，先获取图片 url 对应 key，根据 key 可通过 `edit()` 获取 Editor 对象
    - 如果缓存正被编辑，`edit()` 返回 null，DiskLruCache 不允许同时编辑一个缓存对象
    - 把 url 转成 key，因为图片的 url 可能有特殊字符，影响 url 直接使用，一般用 url 的 md5 值作为 key

    ```java
    private String hashKeyFormUrl (String url){
        String cacheKey;
        try {
            final MessageDigest mDigest = MessageDigest.getInstance("MD5");
            mDigest.update (url.getBytes());
            cacheKey = bytesToHexString (mDigest.digest());
        } catch (NoSuchAlgorithmException e){
            cacheKey = String.valueOf (url.hashCode());
        }
        return cacheKey;
    }
    
    private String bytesToHexString (byte[] bytes) {
        StringBuilder sb = new StringBuilder() ;
        for (int i=0; i<bytes.length; i++) {
            String hex = Integer.toHexString(0xFF & bytes[i]);
            if (hex.length() == 1) {
                sb.append('0');
            }
            sb.append(hex);
        }
        return sb.toString(); 
    }
    ```

    - 对 key，如果当前不存在其他 Editor 对象，`edit()` 返回一个新 Editor 对象，通过其可得到文件输出流
    - 当从网络下载图片时，图片可通过文件输出流写到文件系统上

    ```java
    public boolean downloadUrlToStream(String urlString, OutputStream outputStream){
        HttpURLConnection urlConnection = null;
        BufferedOutputStream out = null;
        BufferedInputStream in = null;
        try{ 
            final URL url = new URL(urlString);
            urlConnection = (HttpURLConnection) ur1.openConnection();
            in = new BufferedInputStream (urlConnection.getInputStream(),
    IO_BUFFER_SIZE);
            out = new Bufferedoutputstream (outputStream, IO_BUFFER_SIZE);
            int b;
            while ((b = in.read()) != -1) {
                out.write(b)
            }
            return true;
        } catch (IOException e) {
            Log.e (TAG, " downloadBitmap failed." + e);
        } finally{
            if (urlConnection != null) {
                urlConnection.disconnect();
            }
            MyUtils.close(out);
            MyUtils.close(in);
        }
        return false;
    }
    ```

    - 通过 Editor 的 `commit()` 提交写入操作，如果下载过程发生异常，可通过 Editor 的 `abort()` 回退整个操作

    ```java
    OutputStream outputStream = editor.newOutputStream(DISK_CACHE_INDEX);
    if (downloadUrlToStream(url, outputStream)){
        editor.commit();
    } else {
        editor.abort();
    }
    mDiskLruCache.flush();
    ```

  - 缓存查找

    - 需将 url 转为 key，通过 DiskLruCache 的 get 得到 Snapshot 对象，通过 Snapshot 对象得到缓存的文件输入流，得到 Bitmap 对象
    - 为避免加载过程导致 OOM，一般不建议直接加载原始图片，通过 BitmapFactory.Options 对象加载缩放后的图片，对 FileInputStream 缩放存在问题，FileInputStream 是种有序文件流，两次 decodeStream 调用影响文件流位置属性，导致第二次 decodeStream 得到 null，可通过文件流得到其对应文件描述符，通过 BitmapFactory.decodeFileDescriptor 加载缩放后图片
    
```java
    Bitmap bitmap = null;
    String key = hashKeyFormUrl(ur1);
    DiskLruCache.Snapshot snapShot = mDiskLruCache.get (key);
    if (snapShot != null){
        FileInputStream fileInputStream = (FileInputStream) snapShot.getInputStream(DISK_CACHE_INDEX);
        FileDescriptor fileDescriptor = fileInputStream.getFD();
        bitmap = mImageResizer.decodeSampledBitmapFromFileDescriptor(fileDescriptor, reqWidth, reqHeight);
        if (bitmap != null) {
            addBitmapToMemoryCache (key, bitmap);
        }
    }
```

#### 12.3、ImageLoader

- 功能
  - 图片同步、异步加载
  - 图片压缩
  - 内存、磁盘缓存
  - 网络拉取
- 处理问题——列表错位：如在 ListView 或 GridView，View 复用既是优点也是缺点，假设一个 item A 正从网络加载图片，其对应的 ImageView 为 A，这时用户快速下滑列表，可能 iterm B 复用 ImageView A，等一会之前的图片下载完，如果直接给 ImageView A 设置图片，由于这时 ImageView A 被 item B 复用，item B 显示 item A 的图片，但 item B 要显示的不是 item A 刚下载好的图片
- 图片压缩功能实现

```java
public class ImageResizer{
    private static final String TAG = "ImageResizer";
    
    public ImageResizer() {}
    
    public BitmapdecodeSampledBitmapFromResource(Resources res, int resId, int reqWidth, int reqHeight) {//与bitmap采样一致
    }
    
    public BitmapdecodeSampledBitmapFromFileDescriptor(FileDescriptor fd, int reqWidth, int reqHeight){
        //首先用inJustDecodeBounds=true解码以检查尺寸
        final BitmapFactory.Options options = new BitmapFactory.Options();
        options.inJustDecodeBounds = true;
        BitmapFactory.decodeFileDescriptor(fd, null, options);
		//计算采样值
        options.inSampleSize = calculateInSampleSize(options, reqWidth, reqHeight);
        //使用inSampleSize集合解码位图
        options.inJustDecodeBounds = false;
        return BitmapFactory.decodeFileDescriptor(fd, null, options);
    }
    
    public int calculateInSampleSize(BitmapFactory.Options options, int regWidth, int regHeight){//与bitmap采样一致
    }
}
```

- 内存和磁盘缓存实现

  - 选择 LruCache 和 DiskLruCache 分别完成内存缓存和磁盘缓存
  - 创建磁盘缓存时做判断，有可能磁盘剩余空间小于磁盘缓存所需大小，一般指用户手机空间不足，没办法创建磁盘缓存，这时磁盘缓存失效

  ```java
  private LruCache<String, Bitmap> mMemoryCache;
  private DiskLruCache mDiskLruCache;
  
  private ImageLoader (Context context){
      mContext = context.getApplicationContext();
      int maxMemory = (int) (Runtime.getRuntime().maxMemory() / 1024);
      int cacheSize = maxMemory / 8;
      mMemoryCache = new LruCache<String, Bitmap>(cacheSize){
          @Override
          protected int sizeOf (String key, Bitmap bitmap) {
              return bitmap.getRowBytes() * bitmap.getHeight() / 1024;
          }
      };
      File diskCacheDir = getDiskCacheDir (mContext, "bitmap");
      if (!diskCacheDir.exists()) {
          diskCacheDir.mkdirs();
      }
      if (getUsablespace(diskCacheDir) > DISK_CACHE_SIZE) {
          try {
              mDiskLruCache = DiskLruCache.open (diskCacheDir, 1, 1, DISK_CACHE_SIZE);
              mIsDiskLruCacheCreated = true;
          } catch (IOException e) {
              e.printStackTrace();
          }
      }
  }
  ```

  - 内存缓存的添加和获取

  ```java
  private void addBitmapToMemoryCache (String key, Bitmap bitmap) {
      if (getBitmapFromMemCache(key) == null) {
          mMemoryCache.put (key, bitmap);
      }
  }
  
  private Bitmap getBitmapFromMemCache(String key) {
      return mMemoryCache.get (key);
  }
  ```

  - 磁盘缓存的添加和获取
    - 磁盘缓存添加通过 Editor 完成，其提供 commit、abort 提交、撤销对文件系统的写操作
    - 磁盘缓存读取通过 Snapshot 完成，通过其得到磁盘缓存对象对应 FileInputStream，通过 FileDescriptor 加载压缩后的图片，将加载后的 Bitmap 添加到内存缓存

  ```java
  private BitmaploadBitmapFromHttp (String url, int reqWidth, int reqHeight)
  throws I0Exception{
      if (Looper.myLooper() == Looper.getMainLooper()) {
          throw new RuntimeException("can not visit network from UI Thread.");
      }
      if (mDiskLruCache == null) {
          return null;
      }
      String key = hashKeyFormUrl (url);
      DiskLruCache.Editor editor = mDiskLruCache.edit (key);
      1f (editor != null){
          OutputStream outputStream = editor.newOutputStream (DISK_CACHE_INDEX);
          if (downloadUrlToStream (url, outputStream)) {
              editor.commit();
          } else{
              editor.abort ();
          }
          mDiskLruCache.flush();
      }
      return loadBitmapFromDiskCache (url, reqWidth, reqHeight);
  }
  
  private BitmaploadBitmapFromDiskCache(String url, int reqwidth, int reqHeight) throws IOException {
      if (Looper.myLooper() == Looper.getMainLooper()){
          Log.w(TAG, "load bitmap from UI Thread, it's not recommended!");
      }
      if (mDiskLruCache == null) {
          return null;
      }
      Bitmap bitmap = null;
      String key = hashKeyFormUrl (url);
      DiskLruCache.Snapshot snapShot = mDiskLruCache.get (key);
      if (snapShot != null) {
          FileInputStream fileInputStream = (FileInputStream) snapShot.getInputStream (DISK_CACHE_INDEX); 
          FileDescriptor fileDescriptor = fileInputStream.getFD();
          bitmap = mImageResizer.decodeSampledBitmapFromFileDescriptor(fileDescriptor, reqWidth, reqHeight);
          if (bitmap != null) {
              addBitmapToMemoryCache (key, bitmap);
          }
      }
      return bitmap;
  }
  ```
  
- 同步和异步加载接口设计

  - 同步加载

    - 先尝试从内存缓存中读取图片，接着尝试从磁盘缓存中读取图片，最后从网络中拉取图片

    ```java
    //从内存缓存、磁盘缓存或网络加载位图
    public Bitmap loadBitmap (String uri, int reqWidth, int reqHeight) {
        Bitmap bitmap = loadBitmapFromMemCache (uri);
        if (bitmap != null) {
            Log .d (TAG, "load Bitmap From MemCache,url:" + uri);
            return bitmap;
        }
        try {
            bitmap = loadBitmapFromDiskCache (uri, reqWidth, reqHeight);
            if (bitmap != null) {
                Log.d (TAG, "load Bitmap From Disk,url:" + uri);
                return bitmap;
            }
            bitmap = loadBitmapFromHttp(uri, reqWidth, reqHeight);
            Log.d (TAG, "load Bitmap From Http,url:" + uri);
        }catch (IOException e) {
            e.printStackTrace();
        }
        if (bitmap == null && !mIsDiskLruCacheCreated) {
            Log.w (TAG，"encounter error, DiskLruCache is not created.");
            bitmap = downloadBitmapFromUrl (uri);
        }
        return bitmap;
    }
    ```

    - 不能在主线程中调用，否则就抛出异常，可能比较耗时，检查在 loadBitmapFromHttp 实现，检查当前线程的 Looper 是否为主线程 Looper 判断当前线程是否主线程，不是就直接抛出异常中止程序

    ```java
    if (Looper.myLooper() == Looper.getMainLooper()) {
        throw new RuntimeException("can not visit network from UI Thread.") ;
    }
    ```

  - 异步加载

    - 尝试从内存缓存读取图片，成功就直接返回结果，否则在线程池调用 loadBitmap，加载成功后将图片、图片地址、需绑定 imageView 封装成 LoaderResult 对象，通过 mMainHandler 向主线程发送消息，就可在主线程给 imageView 设置图片

    ```java
    public void bindBitmap(final String uri, final ImageView imageView, final int reqWidth, final int reqHeight) {
        imageView.setTag(TAG_KEY_URI, uri);
        Bitmap bitmap = loadBitmapFromMemCache (uri);
        if (bitmap != null){
            imageView.setImageBitmap (bitmap);
            return;
        }
        Runnable loadBitmapTask = new Runnable () {
            @Override
            public void run() {
                Bitmap bitmap = loadBitmap (uri, reqWidth, reqHeight);
                if (bitmap != null){
                    LoaderResult result = new LoaderResult (imageView, uri, bitmap);
                    mMainHandler.obtainMessage(MESSAGE_POST_RESULT, result).sendToTarget();
                }
            }
        };
        THREAD_POOL_EXECUTOR.execute (loadBitmapTask);
    }
    ```

    - 线程池 THREAD_POOL_EXECUTOR，核心线程数为当前设备 CPU 核心数加 1，最大容量为 CPU 核心数的 2 倍加 1，线程闲置超时时长 10 秒

      > 没有采用 AsyncTask，因为其在 3.0 以上版本无法实现并发

    ```java
    private static final int CORE_POOL_SIZE = CPU_COUNT + 1;
    private static final int MAXIMUM_POOL_SIZE = CPU_COUNT * 2 + 1;
    private static final long KEEP_ALIVE = 10L;
    private static final ThreadFactory sThreadFactory = new ThreadFactory (){
    	private final atomicInteger mCount = new AtomicInteger(1);
        public Thread newThread(Runnable r) {
            return new Thread(r, "ImageLoader#" + mCount.getAndIncrement ());
        }
    };
        
    public static final Executor THREAD_POOL_EXECUTOR = new ThreadPoolExecutor(CORE_PO0L_SIZE, MAXIMUM_ POOL_SIZE, KEEP_ALIVE, TimeUnit.SECONDS, new LinkedBlockingQueue<Runnable>(), sThreadFactory);
    ```

    - Handler

      >  为解决由于 View 复用导致的列表错位，给 ImageView 设置图片前检查其 url 有没有改变，改变就不再给其设置图片

    ```java
    private Handler mMainHandler = new Handler(Looper.getMainLooper()){
        @Override
        public void handleMessage (Message msg){
            LoaderResult result = (LoaderResult) msg.obj;
            ImageView imageView = result.imageView;
            imageView.setImageBitmap(result.bitmap);
            String uri =(String) imageView.getTag(TAG_KEY_URI);
            if(uri.equals(result.uri)){
                imageView.set ImageBitmap(result.bitmap);
            }else{
                Log.w(TAG, "set image bitmap, but url has changed, ignored!");
            }
        };
    };
    ```

- 优化卡顿

  - 不在 getView 中执行耗时操作，加载图片是耗时操作，必须通过异步方式处理
  - 控制异步任务执行频率，如果用户刻意频繁上下滑动会在一瞬间产生上百个异步任务，造成线程池拥堵并带来大量 UI 更新操作，UI 操作运行在主线程，造成一定程度卡顿，可考虑在列表滑动时停止加载图片，给 ListView 或 GridView 设置 setOnScrollListener，并在 OnScrollListener 的 onScrollStateChanged 判断列表是否处于滑动状态，如果是就停止加载图片

  ```java
  public void onScrollStateChanged (AbsListView view, int scrollState){
      if (scrollState == OnScrollListener.SCROLL_STATE_IDLE){
          mIsGridviewIdle = true;
          mImageAdapter.notifyDataSetChanged();
      }else {
          mIsGridViewIdle = false;
      }
  }
  ```

  - 在 getView 方法中，仅当列表静止时才能加载图片

  ```java
  if (mIsGridviewIdle && mCanGetBitmapFromNetwork){
      imageView.setTag(uri);
      mImageLoader.bindBitmap(uri, imageview, mImagewidth, mImageWidth);
  }
  ```

  - 
    还可开启硬件加速，设置 `android:hardwareAccelerated="true"` 可为 Activity 开启硬件加速

### 13、综合技术

#### 13.1、CrashHandler

- Android 应用不可避免会发生 crash，崩溃，可能Android 系统底层的 bug，可能不充分的机型适配或糟糕网络状况
- crash 时系统会 kill 掉正在执行的程序，现象是闪退或提示用户程序已停止运行，Android 提供处理的方法，Thread#setDefaultUncaughtExceptionHandler

```java
//设置默认的未捕获异常处理程序。如果任何线程由于未处理的异常而死亡，会调用此处理程序
public static void setDefaultUncaughtExceptionHandler(UncaughtExceptionHandler handler){
    Thread.defaultUncaughtHandler = handler;
}
```

* crash 发生时系统回调 UncaughtExceptionHandler 的 uncaughtException 方法，在 uncaughtException 中获取异常信息，可把异常信息存到 SD 卡，在合适时机通过网络将 crash 信息上传服务器

* 使用

  * 实现 UncaughtExceptionHandler 对象，在其 uncaughtException 中获取异常信息并将其存储在 SD 卡中或上传到服务器，调用 Thread 的 setDefaultUncaughtExceptionHandler 将其设为线程默认的异常处理器，默认异常处理器是 Thread 类的静态成员，其作用对象是当前进程的所有线程

  ```java
  public class CrashHandler implements UncaughtExceptionHandler {
      private static final String TAG = "CrashHandler";
      private static final boolean DEBUG = true;
      private static final string PATH = Environment.getExternalStorageDirectory().getPath() + "/crashTest/log/";
      private static final string FILE_NAME = "crash";
      private static final String FILE_NAME_SUFFIX = ".trace";
      private static CrashHandler sInstance = new CrashHandler();
      private UncaughtExceptionHandler mDefaultCrashHandler;
      private Context mContext;
      
      Private CrashHandlerO{}
      
      public static CrashHandler getInstance() {
          return sInstance;
      }
      
      public void init(Context context){
          mDefaultCrashHandler = Thread.getDefaultUncaughtExceptionHandler();
          Thread.setDefaultUncaughtExceptionHandler(this);
          mContext = context.getApplicationContext();
      }
      
      //最关键函数，当程序中有未被捕获的异常，系统自动调用#uncaughtException，thread为出现未捕获异常的线程，ex为未捕获异常，通过ex得到异常信息
      @Override
      public void uncaughtException(Thread thread, Throwable ex){
          try{
              //导出异常信息到sD卡中
              dumpExceptionToSDcard(ex);
              //这里可以上传异常信息到服务器，便于开发人员分析日志从而解决bug
              uploadExceptionToServer();
          }catch (IOException e){
              e.printstackTrace();
          }
          ex.printstackTrace();
          //如果系统提供默认的异常处理器，交给系统结束程序，否则由自己结束自己
          if (mDefaultCrashHandler != null) {
              mDefaultCrashHandler.uncaughtException(thread, ex);
          }else{
              Process.killProcess(Process.myPid());
          }
      }
      
      private void dumpExceptionTosDCard(Throwable ex) throws IOExceptiont{
          //如果sD卡不存在或无法使用，无法把异常信息写入SD卡
          if (!Environment.getExternalStorageState().equals(Environment.MEDIA_MOUNTED)){
              if(DEBUG){
                  Log.w(TAG, "sdcard unmounted, skip dump exception");
                  return;
              }
          }
          File dir = new File(PATH);
          if (!dir.exists()){
              dir.mkdirs();
          }
          long current = System.currentTimeMillis();
          string time = new SimpleDateFormat("yyyy-MM-dd HH:mm:ss").format (new Date(current));
  		File file = new File(PATH + FILE_NAME + time + FILE_NAME_SUFEIX);
          try{
              PrintWriter pw = new PrintWriter(new Bufferedwriter(newFilewriter(file)));
              pw.println(time);
              dumpPhoneInfo(pw);
              pw.print1n();
              ex.printStackTrace(pw);
              pw.close();
          }catch (Exception e){
              Log.e(TAG, "dump crash info failed");
          }
      }
      
      private void dumpPhoneinfo(PrintWriter pw) throws NameNotFoundException{
          PackageManager pm = mContext.getPackageManager();
          PackageInfo pi = pm.getPackageInfo(mContext.getPackageName(), PackageManager.GET_ACTIVITIES);
          pw.print("App version:");
          pw.print(pi.versionName);
          pw.print('_');
          pw.println(pi.versionCode);
          //Android版本号
          Pw.Print("OS version: ");
          pw.print(Build.VERSION.RELEASE);
  		pw.print("_");
          pw.printin(Build.VERSION.SDKINT);
          //手机制造商
          pw.print("vendor: ");
          pw.println(Build.MANUFACTURER);
          //手机型号
          pw.print("Model:");
          Pw.println(Build.MODEL);
          //CPU架构
  		pw.print("CPU ABI:");
          pw.println(Build.CPU_ABI);
      }
      
      private void uploadExceptionToServer(){
          //TODO 将异常消息上载到web服务器
      }
  }
  ```

  - 使用：在 Application 初始化时为线程设置 CrashHandler

  ```java
  public class TestApp extends Application {
      private static TestApp sInstance;
      @Override
      public void onCreate() {
          super.onCreate();
          sInstance = this;
          //在这里为应用设置异常处理，然后程序才能获取未处理的异常
          CrashHandler crashHandler = CrashHandler.getInstance();
          crashHandler.init(this);
      }
      
    public static TestApp getInstance(){
          returnsInstance;
      }
  }
  ```
  
  - 
    被 catch 的异常不会交给 CrashHandler 处理，其只能收到未被捕获的异常

#### 13.2、multidex

- Android 单个 dex 文件能够包含的最大方法数为 65536，包含 Android FrameWork、依赖的 jar 包及应用本身代码所有方法

- 有时方法数没达到 65536，且编译器正常完成，但应用在低版本手机安装时异常中止，dexopt 是程序，应用安装时，系统通过 dexopt 优化 dex 文件，其采用一个固定大小的缓冲区（LinearAlloc）存储应用所有方法信息，其在新版本系统中大小是 8MB 或 16MB，但在 Android 2.2、2.3 中只有 5MB，当待安装的 apk 方法数没有达到 65536，但其存储空间有可能超出 5MB，这时 dexopt 程序报错导致安装失败

- 解决：插件化机制动态加载部分 dex，通过将一个 dex 拆成两个或多个 dex，是一套重量级技术方案，兼容性问题较多

- Android 5.0 开始默认支持 multidex，可从  apk 加载多个 dex 文件，其主要针对 AndroidStudio 和 Gradle 编译环境

- 使用

  - 使用 Android SDK Build Tools 21.1 及以上版本
  - app目录的 build.gradle 文件，在 defaultConfig 添加 multiDexEnabled true 配置项

  ```groovy
  android {
      compilesdkversion 22
      buildToolsversion "22.0.1"
      defaultConfig{
          applicationId "com.ryg.multidextest"
          minsdkversion 8
          targetsdkversion 22
          versionCode 1
          versionName "1.0"
          //enable multidex support
          multiDexEnabled true
      }
      //...
  }
  ```

  - 在 dependencies 添加 multidex 依赖：`compile 'com.android.support:multidex:1.0.0'` 

  - 代码加入三种方案

    - 在 manifest 文件指定 Application 为 MultiDexApplication

    ```xml
    <application
                 android:name="android.support.multidex.MultiDexApplication"
                 android:allowBackup="true"
                 android:icon="@mipmap/ic_launcher"
                 android:label="@string/app_name"
                 android:theme="@style/AppTheme">
    </application>
    ```

    - 让应用 Application 继承 MultiDexApplication
    - 重写 Application 的 attachBaseContext，其比 Application 的 onCreate 先执行

    ```java
    public class TestApplication extends Application{
        @Override
        protected void attachBaseContext (Context base){
            super.attachBaseContext(base);
            MultiDex.install(this);
        }
    }
    ```

  ![安卓开发艺术探索_multidex](Image.assets\安卓开发艺术探索_multidex.png)
  - 通过 build.gradle 文件其他配置项定制 dex 文件的生成，通过 --main-dex-list 选项指定主 dex 文件包含的类
    - --multi-dex：当方法数越界时生成多个 dex 文件
    - -main-dex-list：指定要在主 dex 中打包类列表
    - --minimal-main-dex：表明只有 -main-dex-list 指定类才能打包到主 dex 中，输入是一个文件，如下配置中输入是 app 目录的 maindexlist.txt 文件，其指定一系列类，所有其中的类都被打包到主 dex 中

  ```groovy
  apply plugin: 'com.android.application'
  android {
      compilesdkVersion 22
      buildToolsversion "22.0.1"
      defaultConfig {
          applicationId "com.ryg.multidextest"
          minsdkversion 8
          targetSdkVersion 22
          versioncode 1
          versionName "1.0"
          //enable multidex support
          multiDexEnabled true
      }
      buildTypes{
          release{
              minifyEnabled false
              proguardFiles getDefaultProguardFile ('proguard-android.txt'),'proguard-rules.pro'
          }
      }
  }
  
  afterEvaluate {
      println "afterEvaluate"
      tasks.matching {
          it.name.startsWith('dex')
      }.each{ dx ->
          def listFile = project.rootDir.absolutePath + '/app/maindexlist.txt'
          println "root dir:" + project.rootDir.absolutePath
          println "dex task found: " + dx.name
          if(dx.additionalParameters == nul1){
              dx.additionalParameters = []
          }
          dx.additionalParameters += '--multi-dex'
          dx.additionalParameters += '--main-dex-list=' + listFile
          dx.additionalParameters += '--minimal-main-dex'
      }
  }
  
  dependencies{
      compile fileTree(dir: 'libs', include: ['*.jar'])
      compile 'com.android.support:appcompat-v7:22.1.1'
      compile 'com.android.support :multidex:1.0.0'
  }
  ```

  - multidex 的 jar 包中 9 个类必须打包到主 dex 中，否则程序运行会抛出异常，因为 Application 对象被创建后会在 attachBaseContext 中通过 `MultiDex.install(this)` 加载其他 dex 文件，如果 MultiDex 相关类不在主 dex 中这些类无法被加载，同时 Application 的成员和代码块先于 attachBaseContext 初始化，而其他 dex 文件还没被加载，不能在 Application 的成员及代码块中访问其他 dex 的类

- 问题

  - 应用启动速度降低，应用启动时加载额外的 dex 文件
  - 由于Dalvik linearAlloc 的 bug，可能导致 multidex 的应用无法在 Android 4.0 前的手机运行，可能出现应用在运行中采用 multidex 方案产生大量的内存消耗，导致应用崩溃

#### 13.3、Android 动态加载

- 插件化技术：当项目越来越大时，通过插件化减轻应用内存、CPU 占用，实现热插拔

- 热插拔：在不发布新版本时更新某些模块

- 宿主指普通 apk，插件一般是经处理的 dex 或 apk，插件 Activity 的启动大多借助一个代理 Activity 实现

- 解决三个问题

  - 资源访问

    - 插件中以 R 开头的资源不能访问，因为宿主程序没有插件资源

    - 将插件资源在宿主程序预置一份，弊端：

      > 需要宿主和插件同时持有一份相同资源，增加宿主 apk 的大小
      >
      > 每次发布一个插件都需将资源复制到宿主程序中，每发布一个插件都要更新宿主程序，和插件化思想相违背了

    - 将插件资源解压，通过文件流读取资源，实际操作难

      > 不同资源有不同文件流格式，如图片、XML 等
      >
      > 不同设备加载资源可能不一样

    - Activity 主要通过 ContextImpl 完成，Activity 的 mBase 成员变量类型是 ContextImpl，Context 有两个抽象方法，通过其获取资源，其真正实现在 ContextImpl 中

    ```java
    //返回应用程序包的 AssetManager 实例
    public abstract AssetManager getAssets();
    //返回应用程序包的资源实例
    public abstract Resources getResources();
    ```

    - 先加载 apk 资源

      > 加载资源方法是通过反射，通过调用 AssetManager 的 addAssetPath 将 apk 资源加载到 Resources 对象中
      >
      > addAssetPath 是隐藏 API 无法直接调用，只能通过反射

    ```java
    protected void loadResources() {
        try{
            AssetManager assetManager = AssetManager.class.newInstance();
            Method addAssetPath = assetManager.getClass().getMethod ("addAssetPath", String.class);
            addAssetPath.invoke(assetManager, mDexPath);
            mAssetManager = assetManager;
        }catch (Exception e){
            e.printStackTrace();
        }
        Resources superRes = super.getResources();
        mResources = new Resources (mAssetManager, superRes.getDisplayMetrics(), superRes.getConfiguration());
        mTheme = mResources.newTheme();
        mTheme.setTo(super.getTheme());
    }
    ```

    - addAssetPath 

      > 传递路径可以是 zip 文件、资源目录，apk 是一个 zip，直接将其路径传递，通过 AssetManager 创建一个新 Resources 对象，通过其访问插件 apk 资源

    ```java
    //向asset manager添加其他assets set，可以是目录、ZIP文件，不供应用程序使用，返回添加assets的 cookie，失败返回 0
    public final int addAssetPath(String path) {
        synchronized (this) {
            int res = addAssetPathNative(path);
            makeStringBlocks (mstringBlocks);
            return res;
        }
    }
    ```

    - 在代理 Activity 实现 `getAssets()`、`getResources()` 

    ```java
    @Override
    public AssetManager getAssets(){
        return mAssetManager == null ? super.getAssets() : mAssetManager;
    }
    
    @Override
    public Resources getResources() {
        return mResources == null ? super.getResources() : mResources;
    }
    ```

  - Activity 生命周期的管理

    - 反射方式、接口方式
    - 反射方式通过 java 反射获取 Activity 各种生命周期方法，如 onCreate、onStart、onResume 等，在代理 Activity 调用插件 Activity 对应生命周期方法

    ```java
    @Override
    protected void onResume(){
        super.onResume();
        Method onResume = mActivityLifecircleMethods.get ("onResume");
        if(onResume != null) {
            try{
                onResume.invoke (mRemoteActivity,new 0bject[]{});
            }catch(Exception e) {
                e.printstackTrace();
            }
        }
    }
    
    @Override
    protected void onPause () {
        Method onPause = mActivityLifecircleMethods.get ("onPause");
        if (onPause != null) {
            try {
                onPause.invoke(mRemoteActivity, new Object[]{});
            } catch (Exception e) {
                e.printStackTrace();
            }
        }
        super.onPause();
    }
    ```

    - 缺点：

      > 反射代码写起来比较复杂
      >
      > 过多使用反射有一定性能开销

    - 接口方式将 Activity 生命周期方法提取出来作为一个接口，通过代理 Activity 调用插件 Activity 的生命周期方法，完成插件 Activity 的生命周期管理，且没采用反射解决性能问题，同时接口声明比较简单

    ```java
    public interface DLPlugin{
        public void onStart ();
        public void onRestart();
        public void onActivityResult(int requestCode, int resultCode, Intcntdata);
        public void onResume ();
        public void onPause();
        public void onStop ();
        public void onDestroy();
        public void onCreate(Bundle savedInstanceState);
        public void setProxy(Activity proxyActivity,string dexPath);
        public void onSaveInstanceState (Bundle outState);
        public void onNewIntent (Intent intent);
        public void onRestoreInstancestate (Bundle savedInstancestate);
        public boolean onTouchEvent (MotionEvent event);
        public boolean onKeyUp(int keycode, KeyEvent event);
        public void onwindowAttributesChanged (LayoutParams params);
        public void onwindowFocusChanged (boolean hasFocus);
        public void onBackPressed();
        //...
    }
    ```

    - 在代理 Activity 中按如下方式调用插件 Activity 生命周期方法

    ```java
    @Override
    protected void onStart() {
        mRemoteActivity.onStart ();
        super.onStart();
    }
    
    @Override
    protected void onRestart(){
        mRemoteActivity.onRestart();
        super.onRestart();
    }
    
    @Override
    protected void onResume() {
        mRemoteActivity.onResume();
        super.onResume();
    }
    //...
    ```

  - ClassLoader 的管理

    - 为支持多插件，需合理管理各插件的 DexClassoader，同个插件可采用同个 ClassLoader 加载类，避免多个 ClassLoader 加载同个类所引发类型转换错误
    - 通过将不同插件的 ClassLoader 存储在一个 HashMap 中，保证不同插件的类互不干扰

    ```java
    public class DLClassLoader extends DexclassLoader{
        private static final string TAG = "DLClassLoader";
        private static final HashMap<String, DLClassLoader> mPluginClassLoaders = new HashMap<>();
        protected DEClassLoader (String dexPath, String optimizedDirectory, String libraryPath, ClassLoader parent){
            super(dexPath,optimizedDirectory, libraryPath, parent);
        }
        //返回属于不同apk的可用类加载器
        public static DLClassLoader getClassLoader(String dexPath, Contextcontext, ClassLoader parentLoader) {
            DLClassLoader dLClassLoader = mPluginclassLoaders.get (dexPath);
            if(dLClassLoader != null) return dLClassLoader;
            File dexoutputDir = context.getDir ("dex", Context.MODE_PRIVATE);
            final string dexoutputPath = dexOutputDir,getAbsolutePath();
            dLClassLoader = new DLClassLoader(dexPath, dexOutputPath, null, parentLoader);
            mPluginClassLoaders.put (dexPath,dLClassLoader);
            return dLClassLoader;
        }
    }
    ```

#### 13.4、反编译

- dex2jar、jd-gui 反编译

  - Dex2jar、jd-gui 适用很多操作系统
  - Dex2jar：命令行工具，将 dex 文件转为 jar 包的工具，dex 文件来源于 apk，将 apk 通过 zip 包方式解压缩可提出里面的 dex 文件，jar 包中都是 class 文件

  ```
  Linux (Ubuntu): ./dex2jar.sh classes.dex
  windows: dex2jar.bat classes.dex
  ```

  - jd-gui：图形化工具，将 jar 包转为 java 代码

- apktool 对 apk 二次打包

  - dex2jar、jd-gui 无法反编译出 apk 的二进制数据资源，apktool 可以，还能二次打包（山寨版应用）
  - 使用：Linux（Ubuntu），apk 名为 CrashTest
    - 解包：`./apktool d -f CrashTest.apk CrashTest` 
    - 二次打包：`./apktool b CrashTest CrashTest-fake.apk` 
    - 签名：`java -jar signapk.jar testkey.x509.pem testkey.pk8 CrashTest-fake.apk CrashTest-fake-signed.apk` 
  - 使用：Windows
    - 解包：`apktool.bat d -f CrashTest.apk CrashTest` 
    - 二次打包：`apktool.bat b CrashTest CrashTest-fake.apk` 
    - 签名：`java -jar signapk.jar testkey.x509.pem testkey.pk8 CrashTest-fake.apk CrashTest-fake-signed.apk` 
  - 命令
    - d：解包，CrashTest.apk 表示待解包的 apk，CrashTest 表示解包后文件存储路径
    - -f：如果 CrashTest 目录已存在，直接覆盖
    - b：打包，CrashTest 表示解包后文件存储路径，CrashTest-fake.apk 表示二次打包后文件名
    - apktool 解包后可查看 apk 资源及 smali 文件
    - smali 文件：dex 文件反编译（不同于 dex2jar 反编译）结果，其有自己的语法且可修改，修改后可被二次打包，通过其可修改 apk 执行逻辑
    - 二次打包后必须签名后才能安装
    - signapk.jar：签名，所用签名文件不是官方的，最终是二次打包的山寨版 apk

### 14、JNI 和 NDK 编程

- Java JNI（Java Native Interface，Java 本地接口）：Java 调用 C、C++ 等本地代码封装的接口
- NDK：Android 提供的工具集合，通过其可在 Android 通过 JNI 访问本地代码，提供交叉编译器，只需简单修改 mk 文件就可生成特定 CPU 平台动态库
  - 优点
    - 提高代码安全性，so 库反编译较困难
    - 可方便使用已有 C/C++ 开源库
    - 便于平台间移植，通过 C/C++ 实现的动态库可方便在其他平台使用
    - 提高程序在某些特定情形下的执行效率，但不能明显提升 Android 程序性能

#### 14.1、JNI 开发流程

- 在 Java 声明 native 方法
  - 声明两个 native 方法：get、set(String)，需在 JNI 中实现
  - jni-test 是 so 库的标识，完整名称为 libjni-test.so，是加载 so 库的规范

```java
import java.lang.System;

public class JniTest {
    static {
        System.loadLibrary("jni-test");
    }
    
    public static void main(String args[]){
        JniTest jniTest = new JniTest();
        System.out-println(jniTest.get());
        jniTest.set("hello world");
    }
    
    public native String get();
    public native void set(String str);
}
```

- 编译 Java 源文件得到 class 文件，通过 javah 命令导出 JNI 头文件

  ```
  javac com/ryg/JniTest.java
  javah com.ryg.JniTest
  ```

  - 当前目录下会产生 com_ryg_JniTest.h 的头文件，是 javah 命令自动生成的
    - 函数名格式：Java\_包名\_类名\_方法名
    - JNIEnv*：一个指向 JNI 环境的指针，可通过其访问 JNI 提供的接口方法
    - jobject：Java 对象的 this
    - JNIEXPORT、JNICALL：JNI 定义的宏，可在 jni.h 头文件中查到

  ```c
  /* DO NOT EDIT THIS FILE-it is machine generated*/
  #include <jni.h>
  /* Header for class com ryg_JniTest */
  #ifndef _Included_com_ryg_JniTest
  #define _Included_com_ryg_JniTest
  /*指定extern "C"内函数采用C语言命名风格编译*/
  #ifdef _cplusplus
  extern "C"{
  #endif
  /*
   * Class: com ryg_JniTest
   * Method: get
   * Signature: ()Ljava/lang/String;
   */
  JNIEXPORT jstring JNICALL Java_com_ryg_JniTest_get(JNIEnv *, jobject);
  /*
   * Class: com_ryg_JniTest
   * Method: set
   * Signature: (Ljava/lang/String;)V
   */
  JNIEXPORT void JNICALL Java_com_ryg_JniTest_set(JNIEnv*,jobject,jstring);
  #ifdef _cplusplus
  }
  #endif
  #endif
  ```

- 实现 JNI 方法

  - 在主目录创建子目录，通过 javah 生成的头文件复制到 jni 目录下，创建 test.cpp 和 test.c 文件

  ```c
  // test.cpp
  # include "com ryg JniTest.h"
  #include <stdio.h>
  
  JNIEXPORT jstring JNICALL Java_com_ryg_JniTest_get (JNIEnv *env, jobjectthiz) {
      printf ("invoke get inc++\n");
      return env->NewStringUTF ("Hello from JNI !");
  }
  
  JNIEXPORT void JNICALL Java_com_ryg_JniTest_set (UNIEnv *env, jobject thizjstring string) {
      printf ("invoke set from C++\n");
      char* str = (char*)env->GetStringUTFChars (string, NULL);
      printf("%s\n", str);
      env -> ReleaseStringUTFChars(string, str);
  }
  
  //test.c
  #include "com ryg JniTest.h"
  #include <stdio.h>
  JNIEXPORT jstring JNICALL Java_com_ryg_JniTest_get (UJNIEnv *env, jobject.thiz){
      printf("invoke get from cn");
      return (*env) -> NewStringUTF(env, "Hello from JNI !");
  }
  
  JNIEXPORT void JNICALL Java_com_ryg_JniTest_set (JNIEnv *env, jobject thizjstring string){
      printf("invoke set from C\n");
      char* str = (char*)(*env) -> GetStringUTrChars (env, string, NOLL);
      printf("%s\n", str);
      (*env) -> ReleasestringUTFChars (env, string, str);
  }
  ```

- 编译 so 库并在 Java 中调用

  - so 库编译采用 gcc，切换到 jni 目录
    - /usr/libljvm/java-7-openjdk-amd64：本地 jdk 安装路径
    - libjni-test.so：生成的 so 库名

  ```
  C++: gcc -shared -I /usr/lib/jvm/java-7-openjdk-amd64/include -fPIC test.cpp -o libjni-test.so
  c: gcc -shared -I /usr/lib/jvm/java-7-openjdk-amd64/include -fPIC test.c -o libjni-test.so
  ```

  - Java 加载：`System.loadLibrary("jni-test")`，so 库名中 “lib”、“.so” 不需明确指出
  - 切换到主目录，执行：`java -Djava.library.path=jnicom.ryg.JniTest`，-Djava.library.path=jni 指明 so 库路径

#### 14.2、NDK 开发流程

- 配置 NDK
  - 打开环境变量配置文件：`vim ~/ .bashrc` 
  - 在文件后面添加：`export PATH=-/Android/android-ndk-r10d:$PATH`，
    ~/Android/android-ndk-r10d 是本地 NDK 存放路径
  - 执行 `source ~/.bashrc` 立刻刷新刚设置环境的变量
- 创建 Android 项目，声明所需 native 方法

```java
import android,support.v7.app.ActionBarActivity;
import android.os.Bundle;
import android.util.Log;
import android.view.Menu;
import android.view.MenuItem;
import android.widget.TextView;

public class MainActivity extends ActionBarActivity {
    static {
        System.loadLibrary("jni-test");
    }
    @Override
    protected void onCreate (Bundle savedInstanceState){
        super.onCreate (savedInstanceState); 
        setContentView (R.layout.activity_main);
        TextView textView = (TextView) findViewById(R.id.msg);
        textView.setText (get());
        set("hello world from JniTestApp");
    }
    public native String get();
    public native void set (String str);
}
```

- 实现 Android 项目声明的 native 方法

  - 在外部创建名为 jni 的目录，在 jni 目录下创建 3 个文件：test.cpp、Android.mk、Application.mk
  - Android.mk 中，LOCAL_MODULE 表示模块名称，LOCAL_SRC_FILES 表示需参与编译的源文件
  - Application.mk 常用配置项是 APP_ABI，表示 CPU 架构平台类型，目前常见有 armeabi、x86、mips，移动设备占据主要地位是 armeabi，默认 NDK 编译产生各 CPU 平台的 so 库，通过 APPABI 选项可指定 so 库的 CPU 平台类型，all 表示编译所有 CPU 平台的 so 库

  ```c
  // test.cpp
  #include <jni.h>
  #include <stdio.h>
  #ifdef __cplusplus
  extern "C" {
  #endif
      
  jstring Java com_ryg_JniTestApp_MainActivity_get (JNIEnv *env, jobject thiz) {
      printf("invoke get in c++\n");
      return env -> NewStringUTF ("Hello from JNI in libjni-test.so !");
  }
      
  void Java_com_ryg_JniTestApp_MainActivity_set (JNIEnv *env, jobject thiz,
  jstring string){
      printf ("invoke set from C++\n");
      char* str = (char*)env -> GetStringUTFChars (string, NULL);
      printf("%s\n", str);
      env -> ReleaseStringUTFChars (string, str);
  }
  
  #ifdef __cplusplus
  }
  #endif
  
  // Android. mk
  LOCAL PATH := $(call my-dir)
  include $(CLEAR VARS)
  LOCAL_MODULE := jni-test
  LOCAL_SRC_FILES := test.cpp
  include $ (BUILD_SHARED_LIBRARY)
  
  // Application.mk
  APP_ABI := armeabi
  ```

- 切到 jni 目录的父目录，通过 ndk-build 命令编译产生 so 库

  - NDK 会创建一个和 jni 目录平级的目录 libs，存放 so 库目录
  - ndk- build 默认指定 jni 目录为本地源码目录，如果源码存放目录名不是 jni，ndk-build 无法成功编译
  - 在 app/src/main 中创建 jniLibs 目录，将生成的 so 库复制到 jniLibs 目录中，通过 AndroidStudio 编译运行即可
  - 如果使用其他目录，修改 App 的 build.gradle 文件，jniLibs.srcDir 选项指定新存放目录

  ```groovy
  android{
      //...
      sourceSets.main{
          jniLibs.srcDir 'src/main/jni_libs'
      }
  }
  ```

  - 也可通过 AndroidStudio 自动编译产生 so 库

    - 在 App 的 build.gradle 的 defaultConfig 区域添加 NDK 选项，moduleName 指定模块名称，指定打包后的 so 库文件名
    
```groovy
    android{
        //...
        defaultConfig {
            applicationId "com.ryg.JniTestApp"
            minSdkVersion 8
            targetsdkVersion 22
            versionCode 1
            versionName "1.0"
            ndk{
                moduleName "jni-test"
            }
        }
    }
```

- 将 JNI 代码放在 app/sr/main/jni 目录下，目录名必须为 jni，否则可通过如下指定 JNI 代码路径，jni.srcDirs 指定 JNI 代码路径
  
```groovy
    android {
        sourceSets.main {
            jni.srcDirs 'src/main/jni_src'
        }
    }
```

- 这时 AndroidStudio 会把所有 CPU 平台 so 库打包到 apk 中，一般只需 armeabi 平台的，可按如下修改 build.gradle 配置，在Build Variants 面板选择 armDebug 选项进行编辑
  
```groovy
    android {
        //...
        productFlavors {
            arm{
                ndk {
                    abiFilter "armeabi"
                }
            }
            x86 {
                ndk {
                    abiFilter "x86"
                }
            }
        }
    }
```

#### 14.3、JNI 数据类型与类型签名

- JNI 数据类型：基本类型、引用类型
- 基本类型：jboolean、jchar、 jint 等

| JNI 类型 | Java 类型 |      描述      |
| :------: | :-------: | :------------: |
| jboolean |  boolean  | 无符号8位整型  |
|  jbyte   |   byte    | 有符号8位整型  |
|  jchar   |   char    | 无符号16位整型 |
|  jshort  |   short   | 有符号16位整型 |
|   jint   |    int    |    32位整型    |
|  jlong   |   long    |    64位整型    |
|  jfloat  |   float   |   32位浮点型   |
| jdouble  |  double   |   64位浮点型   |
|   void   |   void    |     无类型     |

- 引用类型：类、对象、数组

|   JNI 类型    | Java 类型 |    描述     |
| :-----------: | :-------: | :---------: |
|    jobject    |  Object   | Object类型  |
|    jclass     |   Class   |  Class类型  |
|    jstring    |  String   |   字符串    |
| jobjectArray  | Object[]  |  对象数组   |
| jbooleanArray | boolean[] | boolean数组 |
|  jbyteArray   |  byte[]   |  byte数组   |
|  jshortArray  |  char[]   |  char数组   |
|  jshortArray  |  short[]  |  short数组  |
|   jintArray   |   int[]   |   int数组   |
|  jlongArray   |  long[]   |  long数组   |
|  jfloatArray  |  float[]  |  float数组  |
| jdoubleArray  | double[]  | double数组  |
|  jthrowable   | Throwable |  Throwable  |

- 类的签名：“L+包名+类名+;”，如 java.lang.String 的签名为 Ljava/lang/tring;
- 基本数据类型签名采用一系列大写字母表示，

| Java 类型 | 签名 | Java 类型 | 签名 |
| :-------: | :--: | :-------: | :--: |
|  boolean  |  Z   |   long    |  J   |
|   byte    |  B   |   float   |  F   |
|   char    |  C   |  double   |  D   |
|   short   |  S   |   void    |  V   |
|    int    |  I   |           |      |

- 对象的签名是对象所属类的签名，如 String 的签名为 Ljava/lang/String;
- 数组的签名为 [+类型签名，如 int 数组的签名是 [I
  - char []：[C
  - float[]：[F
  - double[]：[D
  - long[]：[J
  - String[]：[Ljava/lang/String;
  - object[]：[Ljava/lang/Object;
- 多维数组的签名为 n 个 [+类型签名，n 表示数组维度，如 int\[\]\[\] 的签名为 [[I
- 方法的签名为（参数类型签名）+ 返回值类型签名，如方法 `boolean fun(int a, double b, int[] c)` 的参数类型签名连在一起是 ID[I，返回值类型签名为 Z，所以方法签名是 (ID[)Z

#### 14.4、JNI 调用 Java 方法流程

- 通过类名找到类，根据方法名找到方法 id，就可调用该方法，如果调用 Java 非静态方法，需构造出类对象后才能调用
- java 定义静态方法供 jni 调用

```java
public static void methodCalledByJni (String msgFromJni) {
    Log.d (TAG, "methodCalledByJni,msg:" + msgFromJni);
}
```

- JNI 调用

```c++
void callJavaMethod(JNIEnv *env, jobject thiz) {
    jclass clazz = env -> FindClass("com/ryg/JniTestApp/MainActivity");
    if (clazz == NULL) {
        printf ("find class MainActivity error!");
        return;
    }
    jmethodID id = env -> GetStaticMethodID (clazz, "methodCalledByJni",
"(Ljava/lang/String;)V");
    if (id == NULL){
        printf ("find method methodCalledByJni error!");
    }
    jstring msg = env -> NewStringUTF("msg send by callJavaMethod in test.Cpp.");
    env -> CallStaticVoidMethod(clazz, id, msg);
}
```

### 15、Android 性能优化

#### 15.1、布局优化

- 尽量减少布局文件层级，先删除布局中无用控件和层级，有选择地使用性能较低的 ViewGroup, 如 RelativeLayout，如既可 LinearLayout 也可 RelativeLayout，用 LinearLayout，因为 RelativeLayout 功能较复杂，其布局过程需花费更多 CPU 时间，FrameLayout 和 LinearLayout 都是简单高效的 ViewGroup，而需嵌套时用 RelativeLayout 避免嵌套
- \<include\> 标签：将指定布局文件加载到当前布局文件中
  - 优点不用把指定布局文件内容再重复写一遍
  - 只支持以 android:layout\_ 开头的属性，如 android:layout_width、android:layout_height，其他属性不支持，如 android:background，android:id 是个特例，如果 <include> 指定 id 属性，被包含的布局文件根元素也指定 id 属性，以 <include> 指定的为准
  - 如果 <include> 标签指定 android:layout\_\* 属性，要求 android:layout_width、android:layout_height 必须存在，否则其他 android:layout\_\* 形式的属性无效

```xml
<LinearLayout xmIns: android= "http://schemas.android.com/apk/res/android"
              android:orientation="vertical"
              android:layout_width="match_parent"
              android:layout_height="match_parent"
              android:background="@color/app_bg"
              android:gravity="center horizontal">
    
    <include layout="@layout/titlebar"/>
    
    <TextView android:layout_width="match_parent"
              android:layout_height="wrap_content"
              android:text="@string/text"
              android:padding="5dp" />
</LinearLayout>
```

- <merge> 标签：一般和 <include> 标签一起使用减少布局层级
  - 如果当前布局是竖直方向的 LinearLayout，被包含的布局文件也采用竖直方向的 LinearLayout，被包含的布局文件中的 LinearLayout 是多余的，<merge> 标签可去掉多余一层 LinearLayout

```xml
<merge xmIns:android= "http://schemas.android.com/apk/res/android">
    <Button
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:text="@string/one"/>
    <Button
            android:layout_width="wrap content"
            android:layout_height="wrap_content"
            android:text="@string/two"/>
</merge>
```

- ViewStub：继承 View，非常轻量级且宽高都是 0，不参与布局和绘制，按需加载所需布局文件，很多布局文件在正常情况不会显示，如网络异常界面，ViewStub 可在使用时再加载，提高程序初始化时性能
  - stub_import 是 ViewStub 的 id
  - panel_import 是 layout/layout_network_error 布局的根元素 id
  - 需加载 ViewStub 布局时，`((ViewStub) findViewById (R.id.stub_import)).setVisibility (View.VISIBLE);` 或
    `View importPanel = ((ViewStub) findViewById (R.id.stub_import)).inflate();` 
  - 通过 setVisibility 或 inflate 加载后，ViewStub 被其内部布局替换，不再是整个布局一部分了
  - 不支持 <merge> 标签

```xml
<ViewStub
          android:id="@+id/stub_import"
          android:inflatedId="@+id/panel_import"
          android:layout="@layout/layout_network_error"
          android:layout_width="match_parent"
          android:layout_height="wrap_content"
          android:layout_gravity="bottom"/>
```

#### 15.2、绘制优化

- View 的 onDraw 避免执行大量操作
- onDraw 不要创建局部对象，因为 onDraw 可能被频繁调用，在一瞬间产生大量临时对象，占用过多内存且导致系统频繁 gc，降低程序执行效率
- onDraw 不要做耗时任务、成千上万次循环操作，抢占 CPU 时间片，造成 View 绘制不流畅，View 绘制帧率保证 60fps 最佳，每帧绘制时间不超过16ms

#### 15.3、内存泄露优化

- 在开发过程避免写出有内存泄露的代码

- 通过分析工具如 MAT 找出潜在内存泄露继而解决

- 使用场景

  - 静态变量导致内存泄露

    - Activity 无法正常销毁，静态变量 sContext 引用了其

    ```java
    public class MainActivity extends Activity{
        private static final String TAG = "MainActivity";
        private static ContextsContext;
        @Override
        protected void onCreate (Bundle savedInstanceState){
            super.onCreate(savedInstanceState);
            setContentView(R.layout.activity_main);
            sContext = this;
        }
    }
    ```

    - sView 是静态变量，内部持有当前 Activity，Activity 无法释放

    ```java
    public class MainActivity extends Activity{
        private static final String TAG = "MainActivity";
        private static View sView;
        @Override
        protected void onCreate (Bundle savedInstanceState) {
            super.onCreate (savedinstanceState);
            setContentView (R.layout.activity_main);
            sView = new View(this);
        }
    }
    ```

  - 单例模式导致内存泄露（容易忽视）

    - 提供单例模式 TestManager，其可接收外部注册并将外部监听器存储起来

    ```java
    public class TestManager {
        private List<OnDataArrivedListener> mOnDataArrivedListeners = new ArrayList<>();
        
        private static class SingletonHolder {
            public static final TestManager INSTANCE = new TestManager();
        }
         
        private TestManager() {}
        
        public static TestManager getInstance() {
            return SingletonHolder.INSTANCE;
        }
        
        public synchronized void registerListener (OnDataArrivedListenerlistener){
            if (!mOnDataArrivedListeners.contains(listener)){
                monDataArrivedListeners.add(listener);
            }
        }
        
        public synchronized void unregisterListener (OnDataArrivedListenerlistener){
            mOnDataArrivedListeners.remove (listener);
        }
        
        public interface OnDataArrivedListener {
            public void onDataArrived (object data);
        }
    }
    ```

    - 让 Activity 实现 OnDataArrivedListener 接口并向 TestManager 注册监听，缺少解注册操作会引起内存泄露，原因是 Activity 对象被单例模式 TestManager 持有，而单例模式生命周期和 Application 一致，Activity 对象无法被及时释放

    ```java
    protected void onCreate (Bundle savedInstanceState){
        super.onCreate(savedInstanceState);
        setcontentView(R.layout.activity_main);
        TestManager.getInstance().registerListener(this);
    }
    ```

    - 属性动画导致内存泄露

      > 属性动画有类无限循环动画，如果在 Activity 中播放此类动画且没有在 onDestroy停止，动画会一直播放下去，这时 Activity 的 View 被动画持有，View 又持有 Activity，Activity 无法释放，解决是在 Activity 的 onDestroy 调用 `animator.cancel()` 停止动画

    ```java
    protected void onCreate (Bundle savedInstanceState){
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
        mButton = findViewById(R.id.buttonl);
        0bjectAnimator animator = 0bjectAnimator.ofFloat (mButton, "rotation", 0, 360).setDuration(2000);
        animator.setRepeatCount(ValueAnimator.INFINITE);
        animator.start();
        // animator.cancel();
    }
    ```

#### 15.4、响应速度优化和ANR日志分析

- Android 规定 Activity 5 秒内无法响应屏幕触摸事件或键盘输入事件会出现 ANR，BroadcastReceiver 10 秒内未执行完操作会 ANR
- 当进程发生 ANR 后，系统在 /data/anr 目录下创建文件 traces.txt，分析其能定位出 ANR 原因
- 查看 traces.txt 文件：`adb pull /data/anr/traces.txt` 
- traces 分析
  - 在 Activity 的 onCreate 开启线程，线程执行 `testANR()`，其和 `initView()` 被加同个锁，让 `testANR()` 先获得锁，`initView()` 因为等待 `testANR()` 的锁被同步住，产生 ANR
  - 主线程在 initView 方法正等待锁 <Ox422a0120>，锁类型是 MainActivity 对象，且其被线程 id 为 11（tid=11）的线程持有，其是 “Thread-13248”，其正 sleep，原因是 MainActivity 的 57 行（testANR），这时 ANR 原因明确

```
----- pid 29395 at 2015-05-31 16:14:36 -----
Cmd line: com.ryg.chapter

DALVIK THREADS:
(mutexes: tll=0 tsl=0 tscl=0 ghl=0)

"main" prio=5 tid=1 MONITOR
  | group="main" scount=1 dsCount=0 obj=0x4185b700 self=0x4012d0b0
  | sysTid=32662 nice=0 sched=0/0 cgrp=apps handle=1073954608
  | schedstat=( 0 0 0 ) utm=0 stm=4 core=0
  at com.ryg.chapter.MainActivity.initView(MainActivity.java:~62)
- waiting to lock <0x422a0120> (a com.ryg.chapter.MainActivity) held by tid=11(Thread-13248)
  at com.ryg.chapter.MainActivity.oncreate(MainActivity.java:53)
  at android.app.Activity.performCreate(Activity.java:5086)
  at android.app.Instrumentation.callActivityoncreate(Instrumentation.java: 1079)
  at android.app.ActivityThread.performLaunchActivity(ActivityThread.java:2056)
  at android.app.ActivityThread.handleLaunchActivity(ActivityThread.java:2117)
  at android.app.ActivityThread.access$600(ActivityThread.java:140)
  at android.app.ActivityThreadH.handLeMessage(ActivityThread.java:1213)
  at android.os.Handler.dispatchMessage(Handler.java:99)
  at android.os.Looper.loop(Looper.java:137)
  at android.app.ActivityThread.main(ActivityThread.java:4914)
  at java.lang.reflect.Method.invokeNative(Native Method)
  at java.lang.reflect.Method.invoke(Method.java:511)
  at com.android.internal.os.ZygoteInit$MethodAndArgsCaller.run(ZygoteInit.java:808)
  at com.android.internal.os.ZygoteInit.main(ZygoteInit.java:575)
  at dalvik.system.NativeStart.main(Native Method)
  
"Thread-13248" prio=5 tid=11 TIMED_WAIT
  | group="main" sCount=1 dsCount=0 obj=0x422b0ed8 self=0×683d20c0
  | sysTid=32687 nice=0 sched=0/ 0 cgrp=apps handle=1751804288
  | schedstat=( 0 0 0 ) utm=0 stm-0 core=0
  at java.lang.VMThread.sleep(Native Method)
  at java.lang.Thread.sleep(Thread.java:1031)
  at java.lang.Thread.sleep (Thread.java:1013)
  at android.os.SystemClock.sleep(SystemClock.java:114)
  at com.ryg.chapter.MainActivity.testANR(MainActivity.java:57)
  at com.ryg.chapter.MainActivity.access$0(MainActivity.java:56)
  at com.ryg.chapter.MainActivity$1.run(MainActivity.java:49)
  at java.lang.Thread.run(Thread.java:856)
```

#### 15.5、ListView 和 Bitmap 优化

- ListView
  - 采用 ViewHolder 并避免在 getView 执行耗时操作
  - 根据列表滑动状态控制任务的执行频率，当列表快速滑动时不适合开启大量异步任务
  - 尝试开启硬件加速使 Listview 滑动更流畅
- Bitmap
  - 通过 BitmapFactory.Options 根据需要对图片进行采样，采样用到 BitmapFactory.Options 的 inSampleSize 参数

#### 15.6、线程优化

- 采用线程池，避免程序存在大量 Thread，避免线程的创建、销毁带来的性能开销，有效控制线程池最大并发数，避免大量线程因互相抢占系统资源导致阻塞现

#### 15.7、其他优化

- 不要过多使用枚举，枚举占用内存空间比整型大
- 常量用 static final 修饰
- 使用 Android 特有数据结构，如 SparseArray、Pair 等，有更好的性能
- 适当使用软引用、弱引用
- 内存缓存和磁盘缓存
- 尽量采用静态内部类，可避免潜在的由于内部类导致的内存泄露

#### 15.8、MAT 工具

- MAT（Eclipse Memory Analyzer）：强大的内存泄露分析工具
- 打开 DDMS 界面（AndroidStudio 位于 Monitor 中），选中要分析进程，使用待分析应用一些功能，这是将尽量多的内存泄露暴露出来，单击 Dump HPROF file 按钮，导出 hprof 后缀文件
- hprof 文件不能被 MAT 直接识别，通过 hprof-conv 命令转换，其是 Android SDK 提供的工具,位于 Android SDK 的 platform-tools 目录：`hprof-conv com.ryg.chapter.hprof com.ryg.chapter-conv.hprof`，Eclipse 插件版 MAT 可直接用 MAT 插件打开
- MAT 提供很多功能，常用有
  - Histogram：可直观看出内存中不同类型 buffer 数量和占用内存大小
  - Dominator Tree：把内存对象按从大到小顺序排序，且可分析对象间引用关系，内存泄露分析通过其完成
    - 按从大到小顺序排查，一般 Bitmap 泄露是由于程序某地方发生内存泄露引起，选中后右键 -- Path To GC Roots -- exclude wake/soft references（排除弱引用、软引用），可看到 sContext 引用 Bitmap 导致 Bitmap 无法释放，根本原因是 sContext 无法释放

#### 15.9、提高程序可维护性

- 命名规范，少用缩写，变量前缀可参考 Android 源码命名方式，如私有成员 m 开头，静态成员 s 开头，常量全部大写字母等
- 代码排版需留出合理的空白区分不同代码块，同类变量声明放一组，两类变量间留一行空白区分
- 仅为非常关键的代码加注释，对变量和方法命名风格提出很高要求