### JetPack学习

> 一系列辅助 android 开发实用工具的合称，一套库、工具和指南

![img](https://upload-images.jianshu.io/upload_images/20285170-11b52f3aa91c0be7.jpeg)

- Architecture（架构组件）
  - DataBinding：以声明方式将可观察数据绑定到界面元素，通常和 ViewModel 配合
  - Lifecycle：用于管理Activity和Fragment的生命周期，可生成更易于维护的轻量级代码
  - LiveData：在底层数据库更改时通知视图。它是一个可观察的数据持有者，与常规observable不同，LiveData是生命周期感知的
  - Navigation：处理应用内导航
  - Paging：可以一次加载和显示小块数据，按需加载部分数据，减少网络带宽和系统资源的使用
  - Room：友好、流畅的访问SQLite数据库。它在SQLite的基础上提供了一个抽象层，允许更强大的数据库访问。
  - ViewModel：以生命周期的方式管理界面相关的数据，通常和DataBinding配合使用，实现MVVM架构
  - WorkManager：管理Android的后台作业，即使应用程序退出或设备重新启动也可以运行可延迟的异步任务

  ![img](https://upload-images.jianshu.io/upload_images/20285170-f2cb2f70180beaf7.jpeg)

-  Foundationy（基础组件）

  > 提供横向功能，例如向后兼容性、测试、安全、Kotlin 语言支持
  - Android KTX：优化了供Kotlin使用的Jetpack和Android平台API
  - AppCompat：帮助较低版本的Android系统进行兼容
  - Benchmark：从AndroidStudio中快速检测基于Kotlin或Java的代码
  - Multidex：为具有多个Dex文件应用提供支持
  - Security：安全的读写加密文件和共享偏好设置
  - Test：用于单元和运行时界面测试的Android测试框架

- Behavior（行为组件）

  > 与标准Android服务（如通知、权限、分享）相集成
  - CameraX：相机应用开发。它提供一致且易于使用的界面，适用于大多数Android。设备，并可向后兼容至Android 5.0（API 21）
  - DownloadManager：处理长时间运行的HTTP下载的系统服务
  - Media：用于媒体播放和路由（包括Google Cast）的向后兼容API
  - Notification：提供向后兼容的通知API，支持Wear和Auto
  - Permission：用于检查和请求应用权限的兼容性API
  - Sharing：实现用户分享操作
  - Slices：切片是一种UI模板，创建可在应用外部显示应用数据的灵活界面元素

- UI（界面组件）
  - Animation & Transition：该框架包含用于常见效果的内置动画，允许开发者创建自定义动画和生命周期回调
  - Emoji Compatibility：即便用户没有更新Android系统也可以获取最新的表情符号
  - Fragment：组件化界面的基本单位
  - Layout：用XML中声明UI元素或者在代码中实例化UI元素
  - Palette：从调色板中提取出有用的信息

#### 1、LiveData

> LiveData是一个可以被观察的数据持有类，可以感知并遵循Activity、Fragment等组件的生命周期（防止内存泄露）。可以做到仅在组件处于生命周期的激活状态时才更新UI数据
>
> 保存、传递数据

![image-20210702200825562](C:\Users\13085\AppData\Roaming\Typora\typora-user-images\image-20210702200825562.png)

- 界面控制器（UI Controller）

  - 显示界面逻辑
  - 操作响应
  - 权限管理
  - 加载数据

  ```java
  nameText = findviewById(R.id.name_text);
  
  liveData = new MutableLiveData<String>();
  liveData.observe(this, new Observer<String>() {
      @Override
      public void onChanged(@Nullable String s){ //handLeMessage
          nameText.setText(s);
          Toast.makeText(MainActivity.this, "get a message from LiveData,String:" + s, Toast.LENGTH_SHORT).show();
          Log.d("MainActivity", "onchanged: LiveData");
      }
  });
  ```

- 与Handler不同

  - Handler主要用来进行事件分发，与AMS有关，像按钮点击、屏幕滑动响应

- 使用：LiveData 是一个抽象类，它的实现子类有 MutableLiveData ，MediatorLiveData，常结合 ViewModel 一起使用，MutableLiveData将postValue、setValue方法由protected修饰改为public

  - 继承 ViewModel

    ```java
    public class TestViewModel extends ViewModel {
        private MutableLiveData<String> mNameEvent = new MutableLiveData<>();
    
        public MutableLiveData<String> getNameEvent() {
            return mNameEvent;
        }
        
        public TestViewModel(String key) {
            mKey = key;
        }
    
        public static class Factory implements ViewModelProvider.Factory {
            private String mKey;
    
            public Factory(String key) {
                mKey = key;
            }
    
            @Override
            public <T extends ViewModel> T create(Class<T> modelClass) {
                return (T) new TestViewModel(mKey);
            }
        }
    
        public String getKey() {
            return mKey;
        }
    }
    ```

  - 在 Activity 中创建 ViewModel，监听 ViewModel 里面 mNameEvent 数据变化，当数据改变时，打印相应的 log，并设置给 textView

    - 如果携带参数，调用 `ViewModelProvider of(@NonNull Fragment fragment, @Nullable Factory factory)` 方法，多传递一个 factory 参数，Factory 是一个接口，只有一个 create 方法，实现 Factory 接口，重写 create 方法，在 create 方法里面调用相应的构造函数，返回相应的实例

    - ViewModelProvider 的 of 方法主要有四个

      >ViewModelProvider of(@NonNull Fragment fragment)
      >
      >ViewModelProvider of(@NonNull FragmentActivity activity)
      >
      >ViewModelProvider of(@NonNull Fragment fragment, @Nullable Factory factory)
      >
      >ViewModelProvider of(@NonNull FragmentActivity activity, @Nullable Factory factory)

    - 通过 ViewModel of 方法创建的 ViewModel 实例， 对于同一个 fragment 或者 fragmentActivity 实例，ViewModel 实例是相同的。在 Fragment 中创建 ViewModel 的时候，传入 Fragment 所依附的 Activity。因而他们的 ViewModel 实例是相同的，可以共享数据

    ```java
    mTestViewModel = ViewModelProviders.of(this).get(TestViewModel.class);
    MutableLiveData<String> nameEvent = mTestViewModel.getNameEvent();
    nameEvent.observe(this, new Observer<String>() {
        @Override
        public void onChanged(@Nullable String s) {
            Log.i(TAG, "onChanged: s = " + s);
            mTvName.setText(s);
        }
    });
    ```

  - 当数据源改变时，需要调用 livedata 的 setValue 或者 postvalue 方法。区别是 setValue 方法，Observer 的 onChanged 方法会在调用 serValue 方法的线程回调。而 postvalue 方法，Observer 的 onChanged 方法将会在主线程回调：`mTestViewModel.getNameEvent().setValue(name);` 

- 源码

  - `liveData.observe(this,new Observer<String>(){...})`
  - LiveData
    - observe()
    - LifecycleBoundObserver
    - mObservers:SafeIterableMap<Observer<? super T>, ObserverWrapper>，每次注册观察者都放进这里，当组件生命周期结束移除观察者
    - owner.getLifecycle().addObserver(wrapper)，wrapper是组件和观察者的封装类，给owner（activity）添加生命周期的观察者
  - `liveData.postValue()`
  - liveData
    - postValue()
    - postToMainThread(mPostValueRunnable)
    - setValue()
    - dispatchingValue()，迭代观察者容器
    - considerNotify()
    - observer.mObserver.onChanged(mData)，mObserver是观察者
  - setValue()只能在主线程执行
  - 在点击事件注册观察者会由于生命周期（onCreate--onStart--onResume）调用 onStateChanged--activeStateChanged--dispatchingValue遍历观察者容器，但只会回调一次
    - mLastVersion、mVersion初始值-1
    - 当observer.mLastVersion >= mVersion，return，否则将mVersion 赋值给 mLastVersion，执行回调
    - 当postValue时，mVersion++
  - `liveData = LiveDataBus.getInstance().with("key",String.class)`，多个liveData的容器，单例
  - LiveDataBus
    - with()，第一次调用new一个

- 自定义Livedata

  - 方法

    - observe
    - onActive：当回调该方法的时候，表示该 liveData 正在被使用，因此应该保持最新
    - onInactive：当该方法回调时，表示他所有的 obervers 没有一个状态处理 STARTED 或者 RESUMED，不代表没有 observers
    - observeForever：跟 observe 方法不太一样，在 Activity 处于 onPause ，onStop， onDestroy 的时候，都可以回调 obsever 的 onChange 方法，但是必须手动 remove obsever，否则会发生内存泄漏

  - 例：重写 onActive 方法和 onInactive 方法，在 onActive 方法中，注册监听网络变化的广播，在 onInactive 方法的时候注销广播

    ```java
    public class NetworkLiveData extends LiveData<NetworkInfo> {
        private final Context mContext;
        static NetworkLiveData mNetworkLiveData;
        private NetworkReceiver mNetworkReceiver;
        private final IntentFilter mIntentFilter;
    
        public NetworkLiveData(Context context) {
            mContext = context.getApplicationContext();
            mNetworkReceiver = new NetworkReceiver();
            mIntentFilter = new IntentFilter(ConnectivityManager.CONNECTIVITY_ACTION);
        }
    
        public static NetworkLiveData getInstance(Context context) {
            if (mNetworkLiveData == null) {
                mNetworkLiveData = new NetworkLiveData(context);
            }
            return mNetworkLiveData;
        }
    
        @Override
        protected void onActive() {
            super.onActive();
            mContext.registerReceiver(mNetworkReceiver, mIntentFilter);
        }
    
        @Override
        protected void onInactive() {
            super.onInactive();
            mContext.unregisterReceiver(mNetworkReceiver);
        }
    
        private static class NetworkReceiver extends BroadcastReceiver {
            @Override
            public void onReceive(Context context, Intent intent) {
                ConnectivityManager manager = (ConnectivityManager) context
                        .getSystemService(Context.CONNECTIVITY_SERVICE);
                NetworkInfo activeNetwork = manager.getActiveNetworkInfo();
                getInstance(context).setValue(activeNetwork);
            }
        }
    }
    ```

    - 监听网络变化时：

    ```java
    NetworkLiveData.getInstance(this).observe(this, new Observer<NetworkInfo>() {
        @Override
        public void onChanged(@Nullable NetworkInfo networkInfo) {
            Log.d(TAG, networkInfo);
        }
    });
    ```

- 原理

  - observe

    - 判断是否已经销毁，如果销毁，直接移除
    - 用 LifecycleBoundObserver 包装传递进来的 observer
    - 是否已经添加过，添加过，直接返回
    - 将包装后的 LifecycleBoundObserver 添加进去

    ```java
    @MainThread
    public void observe(@NonNull LifecycleOwner owner, @NonNull Observer<T> observer){
        // 判断是否已经销毁
        if (owner.getLifecycle().getCurrentState() == DESTROYED) {
            // ignore
            return;
        }
        LifecycleBoundObserver wrapper = new LifecycleBoundObserver(owner, observer);
        ObserverWrapper existing = mObservers.putIfAbsent(observer, wrapper);
        //observer已经添加过了，并且缓存的observer跟owner的observer不一致，状态异常，抛出异常
        if (existing != null && !existing.isAttachedTo(owner)) {
            throw new IllegalArgumentException("Cannot add the same observer"
                                               + " with different lifecycles");
        }
        // 已经添加过 Observer 了，返回
        if (existing != null) {
            return;
        }
        // 添加 observer
        owner.getLifecycle().addObserver(wrapper);
    }
    ```

    > 因此，当 owner（Activity 或者 fragment） 生命周期变化的时，回调 LifecycleBoundObserver 的 onStateChanged 方法，onStateChanged 方法又会回调 observer 的 onChange 方法

  - LifecycleBoundObserver，继承 ObserverWrapper，实现 GenericLifecycleObserver 接口，而 GenericLifecycleObserver 接口又实现了 LifecycleObserver 接口，包装了外部的 observer，类似代理模式

    ```java
    class LifecycleBoundObserver extends ObserverWrapper implements GenericLifecycleObserver{
        @NonNull final LifecycleOwner mOwner;
    
        LifecycleBoundObserver(@NonNull LifecycleOwner owner, Observer<T> observer) {
            super(observer);
            mOwner = owner;
        }
    
        @Override
        boolean shouldBeActive() {
            return mOwner.getLifecycle().getCurrentState().isAtLeast(STARTED);
        }
    
        @Override
        public void onStateChanged(LifecycleOwner source, Lifecycle.Event event) {
            if (mOwner.getLifecycle().getCurrentState() == DESTROYED) {
                removeObserver(mObserver);
                return;
            }
            activeStateChanged(shouldBeActive());
        }
    
        @Override
        boolean isAttachedTo(LifecycleOwner owner) {
            return mOwner == owner;
        }
    
        @Override
        void detachObserver() {
            mOwner.getLifecycle().removeObserver(this);
        }
    }
    ```

    - GenericLifecycleObserver#onStateChanged：Activity 回调周期变化时候回调，先判断 `mOwner.getLifecycle().getCurrentState()` 是否已经 destroy，如果已经 destroy，直接移除观察者。这也是不需要手动 remove observer 的原因；如果不是，调用 activeStateChanged 方法，携带的参数为 `shouldBeActive()` 返回的值，当 lifecycle 的 state 为 started 或者 resume 时，shouldBeActive 返回值为 true，表示激活

    - activeStateChanged：当 newActive 为 true 且不等于上一次的值，会增加 LiveData 的 mActiveCount 计数。接着，onActive 在 mActiveCount 为 1 时触发，onInactive 方法只会在 mActiveCount 为 0 时触发，即回调 onActive 方法的时候活跃的 observer 恰好为 1，回调 onInactive 方法的时候，没有一个 Observer 处于激活状态。当 mActive 为 true 时，会促发 dispatchingValue 方法

      ```java
      void activeStateChanged(boolean newActive) {
          if (newActive == mActive) {
              return;
          }
          //immediately set active state, so we'd never dispatch anything to inactive
          // owner
          mActive = newActive;
          boolean wasInactive = LiveData.this.mActiveCount == 0;
          LiveData.this.mActiveCount += mActive ? 1 : -1;
          if (wasInactive && mActive) {
              onActive();
          }
          if (LiveData.this.mActiveCount == 0 && !mActive) {
              onInactive();
          }
          if (mActive) {
              dispatchingValue(this);
          }
      }
      ```

    - dispatchingValue

      ```java
      private void dispatchingValue(@Nullable ObserverWrapper initiator) {
          // 如果正在处理，直接返回
          if (mDispatchingValue) {
              mDispatchInvalidated = true;
              return;
          }
          mDispatchingValue = true;
      
          do {
              mDispatchInvalidated = false;
              // initiator 不为 null，调用 considerNotify 方法
              if (initiator != null) {
                  considerNotify(initiator);
                  initiator = null;
              } else { // 为 null 的时候，遍历所有的 obsever，进行分发
                  for (Iterator<Map.Entry<Observer<T>, ObserverWrapper>> iterator =
                       mObservers.iteratorWithAdditions(); iterator.hasNext(); ) {
                      considerNotify(iterator.next().getValue());
                      if (mDispatchInvalidated) {
                          break;
                      }
                  }
              }
          } while (mDispatchInvalidated);
          // 分发完成，设置为 false
          mDispatchingValue = false;
      }
      ```

    - considerNotify

      > 如果状态不是在活跃中，直接返回，这也是当 Activity 处于 onPause，onStop，onDestroy 时不会回调 observer 的 onChange 方法的原因
      >
      > 判断数据是否是最新，如果最新返回，不处理；不是最新，回调 mObserver.onChanged 方法，并将 mData 传递过去

      ```java
      private void considerNotify(ObserverWrapper observer) {
          // 如果状态不是在活跃中，直接返回
          if (!observer.mActive) {
              return;
          }
          // Check latest state b4 dispatch. Maybe it changed state but we didn't get the event yet.
          // we still first check observer.active to keep it as the entrance for events. So even if the observer moved to an active state, if we've not received that event, we better not notify for a more predictable notification order.
          if (!observer.shouldBeActive()) {
              observer.activeStateChanged(false);
              return;
          }
      
          if (observer.mLastVersion >= mVersion) {
              // 数据已经是最新，返回
              return;
          }
          // 将上一次的版本号置为最新版本号
          observer.mLastVersion = mVersion;
          //noinspection unchecked
          // 调用外部的 mObserver 的 onChange 方法
          observer.mObserver.onChanged((T) mData);
      }
      ```

  - setValue：

    - 断言是主线程，接着 mVersion + 1，并将 value 赋值给 mData，调用 dispatchingValue 方法，传递 null，代表处理所有 observer
    - 这个时候如果依附的 activity 处于 onPause 或者 onStop 的时候，虽然在 dispatchingValue 方法中直接返回，不会调用 observer 的 onChange 方法。但是当所依附的 activity 重新回到前台的时会促发 LifecycleBoundObserver onStateChange 方法，onStateChange 又会调用 dispatchingValue 方法，因为 mLastVersion < mVersion。所以会回调 obsever 的 onChange 方法，这是 LiveData 设计得比较巧妙的一个地方。同理，当 activity 处于后台的时候，多次调用 livedata 的 setValue 方法，最终只会回调 livedata observer 的 onChange 方法一次

    ```java
    @MainThread
    protected void setValue(T value) {
        assertMainThread("setValue");
        mVersion++;
        mData = value;
        dispatchingValue(null);
    }
    ```

  - postValue

    - 采用同步机制，通过 postTask = mPendingData == NOT_SET 有没有任务在处理。 true，没有， false，有，直接返回
    - 调用 `AppToolkitTaskExecutor.getInstance().postToMainThread` 到主线程执行 mPostValueRunnable 任务

    ```java
    protected void postValue(T value) {
        boolean postTask;
        // 锁住
        synchronized (mDataLock) {
            // 当前没有人在处理 post 任务
            postTask = mPendingData == NOT_SET;
            mPendingData = value;
        }
        if (!postTask) {
            return;
        }
        AppToolkitTaskExecutor.getInstance().postToMainThread(mPostValueRunnable);
    }
    
    private final Runnable mPostValueRunnable = new Runnable() {
        @Override
        public void run() {
            Object newValue;
            synchronized (mDataLock) {
                newValue = mPendingData;
                // 处理完毕之后将 mPendingData 置为 NOT_SET
                mPendingData = NOT_SET;
            }
            //noinspection unchecked
            setValue((T) newValue);
        }
    };
    ```

  - AlwaysActiveObserver：没有实现 GenericLifecycleObserver 方法接口，所以在 Activity o生命周期变化的时候，不会回调 onStateChange 方法，也不会主动 remove 掉 observer。因为 obsever 被 remove 掉是依赖于 Activity 生命周期变化时候，回调 GenericLifecycleObserver 的 onStateChange 方法

    ```java
    @MainThread
    public void observeForever(@NonNull Observer<T> observer) {
        AlwaysActiveObserver wrapper = new AlwaysActiveObserver(observer);
        ObserverWrapper existing = mObservers.putIfAbsent(observer, wrapper);
        if (existing != null && existing instanceof LiveData.LifecycleBoundObserver) {
            throw new IllegalArgumentException("Cannot add the same observer"
                                               + " with different lifecycles");
        }
        if (existing != null) {
            return;
        }
        wrapper.activeStateChanged(true);
    }
    
    private class AlwaysActiveObserver extends ObserverWrapper {
        AlwaysActiveObserver(Observer<T> observer) {
            super(observer);
        }
    
        @Override
        boolean shouldBeActive() {
            return true;
        }
    }
    ```

  - 总结

    - liveData 当 addObserver 的时候，用 LifecycleBoundObserver 包装 observer，而 LifecycleBoundObserver 可以感应生命周期，当 activity 生命周期变化的时候，如果不是处于激活状态，判断是否需要 remove 生命周期，需要 remove，不需要直接返回
    - 当处于激活状态的时候，判断是不是 mVersion最新版本，不是需要将上一次缓存的数据通知相应的 observer，并将 mLastVsersion 置为最新
    - 调用 setValue 的时候，mVersion +1，如果处于激活状态，直接处理，如果不是处理激活状态，返回，等到下次处于激活状态的时候，在进行相应的处理
    - 如果想 livedata setValue 之后立即回调数据，而不是等到生命周期变化的时候才回调数据，可以使用 observeForever 方法，但是必须在 onDestroy 时 removeObserver。因为 AlwaysActiveObserver 没有实现 GenericLifecycleObserver 接口，不能感应生命周期

- 跨页面实现：单例 Map<String, LiveData>

  ```java
  public class LiveDataBus{
      private final Map<String, MutableLiveData<Object>> bus;
  
      private LiveDataBus(){
          bus = new HashMap<>();
      }
      
      private static class SingleHolder{
      	private static final LiveDataBus DATA_BUS = new LiveDataBus();
      }
      
      public static LiveDataBus getInstance(){
          return SingleHolder.DATA_BUS;
      }
      
      public <T> MutableLiveData<T> with(String key,Class<T> type){
          if(!bus.containsKey(key)){
              bus.put(key,new MutableLiveData<Object>());
          }
          return (MutableLiveData<t>)bus.get(key);
      }
      
      public MutableLiveData<Object> with(String target){
          return with(target,Object.class);
      }
      
      public void remove(String key){
          if(bus.containsKey(key)){
              bus.remove(key);
          }
      }
  }
  ```

  - 粘性事件：发送事件之后再订阅该事件也能收到该事件

    - 原因

    ![image-20210703220243872](C:\Users\13085\AppData\Roaming\Typora\typora-user-images\image-20210703220243872.png)

    ![image-20210703220332467](C:\Users\13085\AppData\Roaming\Typora\typora-user-images\image-20210703220332467.png)

    - 解决：没有注册观察者时不要传数据

- 偶现问题解决：看源码，理解原理

  - 如Fragment中getActivity为null，正确是在onAttach后、onDetach前执行，错误是在Fragment中定义Context存放activity（内存泄漏）

#### 2、LifeCycle

> LiveData、ViewModel依赖于 Lifecycle 框架

- 在处理Activity或Fragment组件的生命周期相关时，在onCreate初始化某些成员，在onStop中对这些成员进行对应处理，在onDestroy中释放这些资源，这样导致代码非常复杂，最终会有太多的类似调用且导致 onCreate和 onDestroy方法变的非常臃肿

- 使用：

  - support library 26.1.0 之后，继承 FragmentActivity，直接调用 `getLifecycle().addObserver` 方法即可，当 Activity 的生命周期变化的时候，回调 onStateChanged 的方法，状态分别是一一对应的

  ```java
  public class MainActivity extends AppCompatActivity {
      private static final String TAG = "MainActivity";
  
      @Override
      protected void onCreate(Bundle savedInstanceState) {
          super.onCreate(savedInstanceState);
          setContentView(R.layout.activity_main);
     
          getLifecycle().addObserver(new GenericLifecycleObserver() {
              @Override
              public void onStateChanged(LifecycleOwner source, Lifecycle.Event event) {
                  Log.d(TAG, "onStateChanged: event =" + event);
              }
          });
      }
  }
  ```

  - support library 26.1.0 之后，简单地继承 Actiivty
    - 实现 LifecycleOwner 接口，返回 mLifecycleRegistry（默认的LifeCycle实例）
    - 在 Activity 生命周期变化的时候，调用 `mLifecycleRegistry.markState()` 方法标记相应的状态
    - 如果想添加 observer，调用 addObserver 方法添加观察者，这样会在 activity 生命周期变化的时候，回调 observer 的 onChange 方法

  ```java
  public class SimpleLifecycleActivity extends Activity implements LifecycleOwner {
      private static final String TAG = "SimpleLifecycleActivity";
      private LifecycleRegistry mLifecycleRegistry;
  
      @Override
      protected void onCreate(Bundle savedInstanceState) {
          super.onCreate(savedInstanceState);
          setContentView(R.layout.activity_simple_lifecycle);
          mLifecycleRegistry = new LifecycleRegistry(this);
          mLifecycleRegistry.markState(Lifecycle.State.CREATED);
          getLifecycle().addObserver(new GenericLifecycleObserver() {
              @Override
              public void onStateChanged(LifecycleOwner source, Lifecycle.Event event){
                  Log.d(TAG, "onStateChanged: event =" + event);
              }
          });
      }
  
      @Override
      protected void onStart() {
          super.onStart();
          mLifecycleRegistry.markState(Lifecycle.State.STARTED);
      }
  
      @NonNull
      @Override
      public Lifecycle getLifecycle() {
          return mLifecycleRegistry;
      }
  }
  ```

- 原理

  ![img](https://upload-images.jianshu.io/upload_images/7293029-e8b3a15d2ed0a6ee.png)

  - LifecycleObserver接口（ Lifecycle观察者）：实现该接口的类，通过注解的方式，可以通过被LifecycleOwner类的 `addObserver(LifecycleObserver o)` 方法注册，被注册后，LifecycleObserver便可以观察到LifecycleOwner的生命周期事件

    - 如Fragment实现LifecycleOwner接口，意味着Fragment对象持有生命周期对象（Lifecycle），并可以通过 `Lifecycle getLifecycle()` 方法获取内部的Lifecycle对象
    - `getLifecycle()` 方法实际返回 LifecycleRegistry 对象，LifecycleRegistry 对象实际继承 Lifecycle

  - LifecycleOwner接口（Lifecycle持有者）：实现该接口的类持有生命周期(Lifecycle对象)，该接口的生命周期(Lifecycle对象)的改变会被其注册的观察者LifecycleObserver观察到并触发其对应的事件

  - Lifecycle(生命周期)：和LifecycleOwner不同的是，LifecycleOwner本身持有Lifecycle对象，LifecycleOwner通过其 `Lifecycle getLifecycle()` 接口获取内部Lifecycle对象

    ```java
    public abstract class Lifecycle {
        //注册LifecycleObserver （比如Presenter）
        public abstract void addObserver(@NonNull LifecycleObserver observer);
        //移除LifecycleObserver 
        public abstract void removeObserver(@NonNull LifecycleObserver observer);
        //获取当前状态
        public abstract State getCurrentState();
    
        public enum Event {
            ON_CREATE,
            ON_START,
            ON_RESUME,
            ON_PAUSE,
            ON_STOP,
            ON_DESTROY,
            ON_ANY
        }
    
        public enum State {
            DESTROYED,
            INITIALIZED,
            CREATED,
            STARTED,
            RESUMED;
    
            public boolean isAtLeast(@NonNull State state) {
                return compareTo(state) >= 0;
            }
        }
    }
    ```

    - LifecycleRegistry 同样也能通过addObserver方法注册LifecycleObserver ，当LifecycleRegistry 本身的生命周期改变后（内部有一个成员变量State记录当前的生命周期），LifecycleRegistry 就会逐个通知每一个注册的LifecycleObserver ，并执行对应生命周期的方法。
    - 在Fragment对应的生命周期内，发送对应的生命周期事件给内部的 LifecycleRegistry对象处理

    ```java
    public class Fragment implements xxx, LifecycleOwner {
        //...
        void performCreate(Bundle savedInstanceState) {
            onCreate(savedInstanceState);  //1.先执行生命周期方法
            //...省略代码
            //2.生命周期事件分发
            mLifecycleRegistry.handleLifecycleEvent(Lifecycle.Event.ON_CREATE);
        }
    }
    ```

    - Fragment中 `performCreate()、performStart()、performResume()` 会先调用自身的 `onXXX()` 方法，然后再调用LifecycleRegistry的 `handleLifecycleEvent()` 方法；而在`performPause()、performStop()、performDestroy()` 中先LifecycleRegistry的 `handleLifecycleEvent()` 方法 ，然后调用自身的 `onXXX()` 方法

  - State(当前生命周期所处状态)

    ```java
    public enum State {
        DESTROYED,
    
        INITIALIZED,
    
        CREATED,
    
        STARTED,
    
        RESUMED;
    
        public boolean isAtLeast(@NonNull State state) {
            return compareTo(state) >= 0;
        }
    }
    ```

  - Event(当前生命周期改变对应的事件)：当Lifecycle发生改变，如进入onCreate，会自动发出ON_CREATE事件

  - addObserve：

    - 判断当前 mState 是否是 DESTROYED，如果是将初始状态置为 DESTROYED，否则为 INITIALIZED
    - 用 ObserverWithState 包装 observer 和 初始化状态 initialState
    - 将 observer 作为 key，在缓存的 mObserverMap 中查找是否存在，如果存在，证明该 observer 已经添加过，直接返回，不必处理

    ```java
    public void addObserver(@NonNull LifecycleObserver observer) {
        // 判断是否是 DESTROYED，如果是将初始状态置为 DESTROYED，否则为 INITIALIZED
        State initialState = mState == DESTROYED ? DESTROYED : INITIALIZED;
        // ObserverWithState 包装
        ObserverWithState statefulObserver = new ObserverWithState(observer, initialState);
        //  将 observer 作为key，在缓存的 mObserverMap 中查找是否存在
        ObserverWithState previous = mObserverMap.putIfAbsent(observer, statefulObserver);
    
        // 存在，直接返回回去，证明该 observer 已经添加过了。否则，证明还没有添加过该 observer
        if (previous != null) {
            return;
        }
    
        LifecycleOwner lifecycleOwner = mLifecycleOwner.get();
        if (lifecycleOwner == null) {
            // it is null we should be destroyed. Fallback quickly
            return;
        }
    
        // 这里 mAddingObserverCounter 为 0 ，mHandlingEvent 为 false
        boolean isReentrance = mAddingObserverCounter != 0 || mHandlingEvent;
        State targetState = calculateTargetState(observer);
        mAddingObserverCounter++;
        while ((statefulObserver.mState.compareTo(targetState) < 0
                && mObserverMap.contains(observer))) {
            //pushParentState 只是将 statefulObserver 的状态添加到 mParentStates 集合中
            pushParentState(statefulObserver.mState);
            //upEvent 只是返回它的下一个 event
            statefulObserver.dispatchEvent(lifecycleOwner, upEvent(statefulObserver.mState));
            popParentState();
            // mState / subling may have been changed recalculate
            targetState = calculateTargetState(observer);
        }
    
        if (!isReentrance) {
            // we do sync only on the top level.
            sync();
        }
        mAddingObserverCounter--;
    }
    ```

  - calculateTargetState

    - 取出 mObserverMap 中上一个的 entry，该 LifecycleRegistry 实例如果是第一次调用 addObserver 实例的话是 null，否则是上一个 observer 的 entry
    - 根据 previous 是否为 null，设置 siblingState 的值
    - 判断 mParentStates 是否为 null，不则取 mParentStates 最后一次的状态
    - 取 mState, siblingState, parentState 最小的状态

    ```java
    private State calculateTargetState(LifecycleObserver observer) {
        // 取出 mObserverMap 的上一个 entry，previous
        Entry<LifecycleObserver, ObserverWithState> previous = mObserverMap.ceil(observer);
        // 如果不为空，获取它的状态
        State siblingState = previous != null ? previous.getValue().mState : null;
        // 判断 mParentStates 是否为 null，不为 null，去最后的一个状态，否则，为 null
        State parentState = !mParentStates.isEmpty() ? mParentStates.get(mParentStates.size() - 1) : null;
        // 取最小的状态
        return min(min(mState, siblingState), parentState);
    }
    ```

  - `getLifecycle()` -- SupportActivity 的 getLifecycle 方法

    - 在 SupportActivity 中，默认初始化 mLifecycleRegistry，作为成员变量。接着在onCreate 方法中调用了 `ReportFragment.injectIfNeededIn(this);` 
    - 在 injectIfNeededIn 方法中判断是否已经添加 ReportFragment，没有的话，添加进去，然后在 onCreat ，onStart， onResume， onPause， onStop， onDestroy 方法中分别调用 dispatch 方法进行分发生命周期
    - 在 dispatch 方法中先判断 activity 是不是实现了 LifecycleRegistryOwner ，如果是直接分发，不是，判断是否实现 LifecycleOwner，获取它的 lifecycle，调用它的 handleLifecycleEvent 进行分发

  - 没有在 onStart，onResume, onPause , onStop 和 onDestroy 方法中调用 mLifecycleRegistry.handleLifecycleEvent 方法，怎样促发 Observer 的 onStateChanged 方法

    - LifecycleDispatcher 在 init 方法中，通过 `context.getApplicationContext().registerActivityLifecycleCallbacks` 监听全局 activity 的创建，在 activity oncreate 的时候，调用 `ReportFragment.injectIfNeededIn(activity)` 添加 fragment，进而分发相应的事件
    - init 在 ProcessLifecycleOwnerInitializer 的 onCreate 方法中调用。ProcessLifecycleOwnerInitializer 是一个 ContentProvider，在 Manifest 里面注册了，而 ContentProvider 的 onCreate 方法优先于 Application 的 onCreate 执行，所以在 Application 之前就调用了 ProcessLifecycleOwnerInitializer init 方法，监听了 Activity 的创建，当 Actiivty 创建的时候，尝试为 Activity 添加 ReportFragment。而 ReportFragment 会在 Activity 生命周期变化的时候帮助我们分发生命周期

- 时序图

  ![img](https://upload-images.jianshu.io/upload_images/7293029-a125ace9440970e6.png)

- 设计

  - 利用 ProcessLifecycleOwnerInitializer contentProvider 隐式加载，如果不的话，对于普通的 Activity，旧版本等，使用 lifecycle 必须在基类中手动调用 ReportFragment.injectIfNeededIn(activity) 方法
  - 利用 fragment 来分发生命周期
    - 将逻辑从 Activity 中剥离出来，减少耦合，方便复用
    - 做到在 Activity onCreate 之后才回调 observer 的 CREATED Event 事件。如果通过 Application registerActivityLifecycleCallbacks 方法来分发生命周期，ActivityLifecycleCallbacks 的 onActivityCreated 是在 Activity oncreate 之前调用