### SharedPreferences

- SharedPreferences 一经加载，内部存储的数据会以 Map 的形式一直保存在内存中，应该要分清常用和不常用的数据，各自保存在一起
- 获取 SharedPreferences 如果是初次加载，需要从文件中加载数据，是耗时操作，加载数据过程在子线程中完成，但当对 SharedPreferences 操作而 SharedPreferences 加载数据还没结束时会导致线程等待，直到加载完再开始操作。要用到 SharedPreferences 应该提前获取，确保使用时已经将数据加载完毕（针对首次获取，如果加载过一次，之后直接从内存中获取）
  - `getSharedPreference(String name,int Mode)` 找到对应name的文件，加载对应文件到内存中SharedPreference，一个xml文件对应一个SharedPreferences单例（ArrayMap），首次加载会阻塞
- 保存数据时，`apply()` 先将数据保存到内存中，然后在子线程中将数据保存到磁盘，`commit()` 先将数据保存到内存中，之后立即将数据同步到磁盘，操作在当前线程中完成，会阻塞线程。android 建议使用 `apply()` ，如果保存数据时 `apply()` 没有结束，但这时退出 activity，activity 会先确保数据是否已经全部同步到磁盘，如果没有会有两种情况，一是保存的过程正在子线中执行，这时等待就好；二是还没分发给子线程，就直接切换主线程执行，使用 SharedPreferences.Editor 提交数据时，尽量在所有数据都提交后再调用 `apply()` 方法
- SharedPreferences 基于单个 xml 文件实现，且所有持久化数据是一次性加载到内存，如果数据过大，不合适采用 SharedPreferences 存放。而适用场景是单进程，Android 原生的文件访问不支持多进程互斥，所以 SharePreferences 也不支持，如果多个进程更新同一个 xml 文件，可能存在同不互斥问题

#### 1、源码分析

##### 1.1、持久化数据的加载

- SharedPreferences 的获取

  > context 可能是 Application \ Activity 的

```java
SharedPreferences currentSP = context.getSharedPreferences(spName, Context.MODE_PRIVATE);
SharedPreferences.Editor editor = mSharedPreferences.edit();
editor.putString(key, value);
editor.apply();
```

- ContextImpl 的 getSharedPreferences()

  > 在 api 19 之前，如果 name 为 null，默认将文件取名为 null.xml，之后会报 NullPointerException
  >
  > ArrayMap 是 Android 中提供的，作用类似 HashMap，在内存方面更有效率，android 推荐使用 ArrayMap。这里用于根据名字保存文件，文件是保存内容的地方，接下来调用 getSharedPreferences(file, mode)，如果不想使用默认保存文件的地方可以使用这个方法

```java
@Override
public SharedPreferences getSharedPreferences(String name, int mode) {
    SharedPreferencesImpl sp;
    synchronized(ContextImpl.class){
        if(sSharedPrefs == null){
            sSharedPrefs = new ArrayMap<String,ArrayMap<String,SharedPreferencesImpl>>();
        }

        final String packageName = getPackageName();
        ArrayMap<String,SharedPreferencesImpl> packagePrefs = sSharedPrefs.get(packageName);
        if(packagePrefs == null){
            packagePrefs = new ArrayMap<String,SharedPreferencesImpl>();
            sSharedPrefs.put(packageName, packagePrefs);
        }
        sp = packagePrefs.get(name);
        if (sp == null) {
            //读取文件
            File prefsFile = getSharedPrefsFile(name);
            sp = new SharedPreferencesImpl(prefsFile, mode);
            //缓存sp对象
            packagePrefs.put(name, sp);
            return sp;
        }
    }
    //跨进程同步问题
    if ((mode & Context.MODE_MULTI_PROCESS) != 0 || getApplicationInfo().targetSdkVersion<android.os.Build.VERSION_CODES.HONEYCOMB){
        sp.startReloadIfChangedUnexpectedly();
    }
    return sp;
}
```

- 先去内存中查询与 xml 对应的 SharePreferences 是否已经被创建加载，没有就创建加载，加载后将所有 key-value 保存到内存中，首次访问还需要创建 xml 文件

- SharePreferences 对应 xml 文件位置：一般在 /data/data/包名/shared_prefs 目录下，后缀是 .xml

- 数据存储样式

  ```xml
  <?xml version='1.0' encoding='utf-8' standalone='yes' ?>
  <map>
      <string name="name608">2608</string>
      <string name="name721">2721</string>
      <string name="name963">2963</string>
      <string name="name135">2135</string>
      <string name="name275">2275</string>
      <string name="name697">2697</string>
  </map>
  ```

- SharedPreferencesImpl

  - 数据加载从 new SharedPreferencesImpl 开始

  ```java
  SharedPreferencesImpl(File file, int mode) {
      mFile = file;
      mBackupFile = makeBackupFile(file);
      mMode = mode;
      mLoaded = false;
      mMap = null;
      startLoadFromDisk();
  }
  ```

- startLoadFromDisk

  - 读取 xml 配置，如果其他线程想要在读取前使用会被阻塞，一直 wait 等待，直到数据读取完成
  - 直接使用 xml 解析工具 XmlUtils，直接在当前线程读取 xml 文件，如果 xml 文件稍大尽量不在主线程读取，读取完成后，xml 中的配置项都会被加载到内存，再次访问时，其实访问的是内存缓存

  ```java
  private void loadFromDiskLocked() {
      //...
      Map map = null;
      StructStat stat = null;
      try {
          stat = Os.stat(mFile.getPath());
          if (mFile.canRead()) {
              BufferedInputStream str = null;
              try {
                  //读取xml中配置
                  str = new BufferedInputStream(new FileInputStream(mFile),16*1024);
                  map = XmlUtils.readMapXml(str);
              }
              //...
              mLoaded = true;
              //...
              //唤起其他等待线程
              notifyAll();
          }
      }
  }
  ```

##### 1.2、持久化数据的更新

```java
SharedPreferences.Editor editor = mSharedPreferences.edit();
editor.putString(key1, value1);
editor.putString(key2, value2);
editor.apply();//或者commit
```

- Editor 是一个接口，这里的实现是一个 EditorImpl 对象，先批量预处理更新操作，再提交更新，提交事务时候有 apply、commit

- 区别：将数据持久化到 xml 文件，前者异步，后者同步。Google 推荐使用前一种，单进程只要保证内存缓存正确就能保证运行时数据的正确性，持久化不必太及时

- 两者都先调用 commitToMemory 将更改提交到内存，之后调用 enqueueDiskWrite 进行数据持久化

- commit：正在执行的写入磁盘操作的数量为1时直接在当前线程执行，否则异步到QueuedWork中执行

- QueuedWork

  - 当apply()方式提交的时，默认消息会延迟发送100毫秒，避免频繁的磁盘写入操作
  - 当commit()方式，调用QueuedWork的queue()时，会立即向handler()发送Message
  
- 在Activiy的 onPause()、BroadcastReceiver的onReceive()以及Service的onStartCommand()方法之前都会调用waitToFinish()，执行文件写入操作，但超时会ANR

  ```java
  public void apply() {
      //添加到内存
      final MemoryCommitResult mcr = commitToMemory();
      final Runnable awaitCommit = new Runnable() {
          public void run() {
              try {
                  mcr.writtenToDiskLatch.await();
              } catch (InterruptedException ignored) {}
          }
      };
  
      QueuedWork.add(awaitCommit);
      Runnable postWriteRunnable = new Runnable() {
          public void run() {
              awaitCommit.run();
              QueuedWork.remove(awaitCommit);
          }
      };
      //延迟写入到xml文件
      SharedPreferencesImpl.this.enqueueDiskWrite(mcr, postWriteRunnable);
      //通知数据变化
      notifyListeners(mcr);
  }
  
  public boolean commit() {
      MemoryCommitResult mcr = commitToMemory();
      SharedPreferencesImpl.this.enqueueDiskWrite(mcr, null/*sync write on this thread okay*/);
      try {
          mcr.writtenToDiskLatch.await();
      } catch (InterruptedException e) {
          return false;
      }
      notifyListeners(mcr);
      return mcr.writeToDiskResult;
  }
  
  private void enqueueDiskWrite(final MemoryCommitResult mcr,final Runnable postWriteRunnable){
      final Runnable writeToDiskRunnable = new Runnable() {
          public void run() {
              synchronized (mWritingToDiskLock) {
                  writeToFile(mcr);
              }
              synchronized (SharedPreferencesImpl.this) {
                  mDiskWritesInFlight--;
              }
              if (postWriteRunnable != null) {
                  postWriteRunnable.run();
              }
          }
      };
      final boolean isFromSyncCommit = (postWriteRunnable == null);
      if (isFromSyncCommit) {
          boolean wasEmpty = false;
          synchronized (SharedPreferencesImpl.this) {
              wasEmpty = mDiskWritesInFlight == 1;
          }
          //如果没有其他线程在写文件，直接在当前线程执行
          if (wasEmpty) {
              writeToDiskRunnable.run();
              return;
          }
      }
      QueuedWork.singleThreadExecutor().execute(writeToDiskRunnable);
  }
  ```

##### 1.3、多进程使用

- SharePreferences 的新建有个 mode 参数，可以指定加载模式，MODE_MULTI_PROCESS 是 Google 提供的一个多进程模式，并不支持多进程同步更新等，只在 getSharedPreferences 时重新从 xml 重加载，如果在一个进程中更新 xml 但没有通知另一个进程，另一个进程的 SharePreferences 不会自动更新

##### 1.4、使用

- 低频：尽量保证多次edit一个apply
- 异步：能用apply()方法提交的就用apply()方法提交，有延迟（100ms）
- 小量：尽量维持Sharepreferences的体量小些，方便磁盘快速写入
- 合规：如果存JSON数据，就不用Sharepreferences，因为SharedPerences本质是xml文件格式存储的，存储JSON文件需要转义，效率很低

