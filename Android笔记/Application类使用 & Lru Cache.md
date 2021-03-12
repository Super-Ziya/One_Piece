### Application类使用

---

<img src="C:\Users\13085\Desktop\git_work\Android\Android笔记\Image\Application解析.png" alt="Application解析" style="zoom:80%;" />

#### 1、定义

- 继承关系

  > java.lang.Object
  >
  > android.content.Context
  >
  > android.content.ContextWrapper
  >
  > android.app.Application

#### 2、特点

- 单例模式：每个 Android App 运行时，先自动创建 Application 类并实例化 Application 对象，且只有一个，可以通过继承 Application 类自定义 Application 类和实例
- 全局实例：Activity、Service 都可获得 Application 对象且是同一对象
- 生命周期：等于 Android App 生命周期

#### 3、方法

![Application方法](C:\Users\13085\Desktop\git_work\Android\Android笔记\Image\Application方法.png)



- `onCreate()` ：Application 实例创建时调用，是 Android 系统的入口，默认空实现
  - 作用：
    - 初始化应用程序级别资源，如全局对象、环境变量、图片资源初始化、推送服务注册，不要进行耗时操作
    - 数据共享、数据缓存：设置全局共享数据、变量、方法

- `registerComponentCallbacks()` & `unregisterComponentCallbacks()` ：注册、注销 ComponentCallbacks2 回调接口

  ```java
  registerComponentCallbacks(new ComponentCallbacks2(){
      @Override
      public void onTrimMemory(int level) {
          super.onTrimMemory(level);
          //清除缓存
          if(level >= ComponentCallbacks2.TRIM_MEMORY_MODERATE){
              mPendingRequest.clear();
              mBitmapHolderCache.evictAll();
              mBitmapCache.evictAll();
          }
      }
  
      @Override
      public void onLowMemory() {}
  
      @Override
      public void onConfigurationChanged(Configuration newConfig) {}
  });
  ```

  - `onTrimMemory()` ：通知应用程序当前内存使用情况（以内存级别进行识别）

    > 系统在内存不足时会按照 LRU Cache 从低到高杀死进程；优先杀死占用内存较高的应用
    >
    > 应用占用内存较小 = 被杀死几率降低，从而快速启动（即热启动 = 启动速度快）
    >
    > 可回收的资源：缓存（如文件、图片）、动态生成 & 添加的 View

    ![Application内存使用级别](C:\Users\13085\Desktop\git_work\Android\Android笔记\Image\Application内存使用级别.png)

    - 可回调对象

      ```java
      Application.onTrimMemory()
      Activity.onTrimMemory()
      Fragment.OnTrimMemory()
      Service.onTrimMemory()
      ContentProvider.OnTrimMemory()
      ```

    - OnTrimMemory() 的 TRIM_MEMORY_UI_HIDDEN 回调时刻：应用程序中所有 UI 组件全部不可见，Activity 的 onStop() 回调时刻是当一个 Activity 完全不可见，TRIM_MEMORY_UI_HIDDEN 等级在 onStop() 前调用

  - `onLowMemory()` ：监听 Android 系统整体内存较低时刻

    > OnTrimMemory() 是  OnLowMemory() Android 4.0 后的替代 API
    >
    > OnLowMemory() =  OnTrimMemory() 中的 TRIM_MEMORY_COMPLETE 级别

  - `onConfigurationChanged()` ：监听应用程序配置信息的改变，如屏幕旋转

    > 配置信息指 ：Manifest.xml 文件下的 Activity 标签属性 android:configChanges 的值

    ```xml
    <activity android:name=".MainActivity"
          android:configChanges="keyboardHidden|orientation|screenSize">
    // 设置该配置属性会使 Activity 在配置改变时不重启，只执行 onConfigurationChanged()
    </activity>
    ```

- `registerActivityLifecycleCallbacks()` & `unregisterActivityLifecycleCallbacks()`

  - 作用：注册 / 注销应用程序内所有 Activity 的生命周期监听
  - 调用时刻：当应用程序内 Activity 生命周期发生变化时

  ```java
  //实际上需要重写的是 ActivityLifecycleCallbacks 接口里的方法
  registerActivityLifecycleCallbacks(new ActivityLifecycleCallbacks() {
      @Override
      public void onActivityCreated(Activity activity,Bundle savedInstanceState) {}
  
      @Override
      public void onActivityStarted(Activity activity){}
  
      @Override
      public void onActivityResumed(Activity activity){}
  
      @Override
      public void onActivityPaused(Activity activity){}
  
      @Override
      public void onActivityStopped(Activity activity){}
  
      @Override
      public void onActivitySaveInstanceState(Activity activity,Bundle outState){}
  
      @Override
      public void onActivityDestroyed(Activity activity){}
  });
  ```

- `onTerminate()` ：应用程序结束时调用，只用于 Android 仿真机测试

#### 4、应用场景

- 初始化应用程序级别的资源，如全局对象、环境配置变量等
- 数据共享、数据缓存，如设置全局共享变量、方法等
- 获取应用程序当前的内存使用情况，及时释放资源，从而避免被系统杀死
- 监听应用程序配置信息的改变，如屏幕旋转等
- 监听应用程序内所有 Activity 的生命周期

#### 5、自定义 Application 类

- 继承 Application 子类

- 配置自定义 Application 子类

  ```xml
  <application
          android:name=".MyApplication">
          //此处自定义 Application 子类名字
  </application>
  ```

- 使用自定义 Application 类实例

  ```java
  private MyApplicaiton app;
  //只需要调用 Activity.getApplication() 或 Context.getApplicationContext() 就可获得一个 Application 对象
  app = (MyApplication)getApplication();
  // 然后再得到相应的成员变量或方法即可
  app.exitApp();
  ```


#### 6、Lru Cache（Least Recently Used，最近最少使用）

> LruCache 是一个泛型类，内部采用 LinkedHashMap 以强引用方式存储外界的缓存对象，提供了 get 和 put 方法完成缓存的获取和添加操作，缓存满时，LruCache 会移除较早使用的缓存对象，然后再添加新缓存对象
>
> LinkedHashMap 特性：HashMap 与双向链表的组合，HashMap 是无序的，而加入双向链表后保证了元素的顺序
>
> 首先设置内部 LinkedHashMap 构造参数 accessOrder=true， 实现数据排序按照访问顺序
>
> LruCache 类调用 get(K key) 方法时，会调用 LinkedHashMap.get(Object key) 
>
> 设置 accessOrder=true 后调用 LinkedHashMap.get(Object key) 会通过 LinkedHashMap 的 afterNodeAccess() 方法将数据移到队尾
>
> 最新访问的数据在尾部，在 put 和 trimToSize 的方法执行下，若发生数据移除，优先移除头部数据

- LinkedHashMap 参数
  - initialCapacity：初始化 LinkedHashMap 大小
  - loadFactor（负载因子）：LinkedHashMap 父类 HashMap 里的构造参数，涉及扩容问题，如 HashMap 最大容量 100，设置 0.75f 到 75 时会扩容
  - accessOrder：排序模式，true 表示访问时进行排序（LruCache 核心工作原理），false 表示在插入时才排序
- LruCache.put(K key, V value)
  - 把值放入 LinkedHashMap，不管超不超过设定的缓存容量
  - 根据 safeSizeOf 方法计算此次添加数据的容量，并加到 size 里 
  - 通过 trimToSize() 方法判断 size 是否大于 maxSize
- trimToSize(int maxSize)：不断删除 LinkedHashMap 中队首元素，直到缓存小于最大值
- LruCache.get(K key)：调用 LruCache 的 get() 方法获取集合中的缓存对象时，代表访问了一次该元素，会更新队列，保持队列按照访问顺序排序，更新过程在 LinkedHashMap 的 get() 方法中完成