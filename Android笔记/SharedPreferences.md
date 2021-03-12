### SharedPreferences

- SharedPreferences 一经加载，内部存储的数据会以 Map 的形式一直保存在内存中，应该要分清常用和不常用的数据，各自保存在一起
- 获取 SharedPreferences 如果是初次加载，需要从文件中加载数据，是耗时操作，加载数据过程在子线程中完成，但当对 SharedPreferences 操作而 SharedPreferences 加载数据还没结束时会导致线程等待，直到加载完再开始操作。要用到 SharedPreferences 应该提前获取，确保使用时已经将数据加载完毕（针对首次获取，如果加载过一次，之后直接从内存中获取）
- 保存数据时，`apply()` 先将数据保存到内存中，然后在子线程中将数据保存到磁盘，`commit()` 先将数据保存到内存中，之后立即将数据同步到磁盘，操作在当前线程中完成，会阻塞线程。android 建议使用 `apply()` ，如果保存数据时 `apply()` 没有结束，但这时退出 activity，activity 会先确保数据是否已经全部同步到磁盘，如果没有会有两种情况，一是保存的过程正在子线中执行，这时等待就好；二是还没分发给子线程，就直接切换主线程执行，使用 SharedPreferences.Editor 提交数据时，尽量在所有数据都提交后再调用 `apply()` 方法

#### 1、源码分析

- SharedPreferences 的获取

  > context 可能是 Application \ Activity 的

```java
SharedPreferences currentSP = context.getSharedPreferences(spName, Context.MODE_PRIVATE);
```

- ContextImpl 的 getSharedPreferences()

  > 在 api 19 之前，如果 name 为 null，默认将文件取名为 null.xml，之后会报 NullPointerException
  >
  > ArrayMap 是 Android 中提供的，作用类似 HashMap，但在内存方面更有效率，在 android 中推荐使用 ArrayMap。这里用于根据名字保存文件，文件是保存内容的地方，接下来调用 getSharedPreferences(file, mode)，如果不想使用默认保存文件的地方可以使用这个方法

```java
public SharedPreferences getSharedPreferences(String name, int mode) {
    //api 19之前
    if (mPackageInfo.getApplicationInfo().targetSdkVersion < Build.VERSION_CODES.KITKAT) {
        if (name == null) {
            name = "null";
        }
    }

    File file;
    synchronized (ContextImpl.class) {
        if (mSharedPrefsPaths == null) {
            mSharedPrefsPaths = new ArrayMap<>();
        }
        file = mSharedPrefsPaths.get(name);
        if (file == null) {
            file = getSharedPreferencesPath(name);
            mSharedPrefsPaths.put(name, file);
        }
    }
    return getSharedPreferences(file, mode);
}
```

- getSharedPreferences(file, mode)

  > getSharedPreferences(file, mode) 主要逻辑是先获取缓存，根据文件查看缓存中是否存在，存在就直接返回，不存在就新建一个 SharedPreferences，保存到缓存中再返回，获取缓存用 getSharedPreferencesCacheLocked()

```java
public SharedPreferences getSharedPreferences(File file, int mode) {
    SharedPreferencesImpl sp;
    synchronized (ContextImpl.class) {
        final ArrayMap<File, SharedPreferencesImpl> cache = getSharedPreferencesCacheLocked();	//获取缓存
        sp = cache.get(file);
        if (sp == null) {
            checkMode(mode);
            //SdkVersion >= 27
            if (getApplicationInfo().targetSdkVersion >= android.os.Build.VERSION_CODES.O) {
                if (isCredentialProtectedStorage() && !getSystemService(UserManager.class).isUserUnlockingOrUnlocked(UserHandle.myUserId())) {
                    throw new IllegalStateException("SharedPreferences in credential encrypted " + "storage are not available until after user is unlocked");
                }
            }
            sp = new SharedPreferencesImpl(file, mode);
            cache.put(file, sp);
            return sp;
        }
    }
    if ((mode & Context.MODE_MULTI_PROCESS) != 0 || getApplicationInfo().targetSdkVersion < android.os.Build.VERSION_CODES.HONEYCOMB) {
        //如果其他进程在我们背后更改了 prefs 文件，将重新加载它。这是历史（如果没有记录的话）行为
        sp.startReloadIfChangedUnexpectedly();
    }
    return sp;
}
```

- getSharedPreferencesCacheLocked()

  > getSharedPreferencesCacheLocked() 用 static 的 ArrayMap，在第一次加载后就一直在内存中获取，初次加载是创建 sp = new SharedPreferencesImpl(file, mode) 对象返回

```java
private static ArrayMap<String, ArrayMap<File, SharedPreferencesImpl>> sSharedPrefsCache;
private ArrayMap<File, SharedPreferencesImpl> getSharedPreferencesCacheLocked() {
    if (sSharedPrefsCache == null) {
        sSharedPrefsCache = new ArrayMap<>();
    }
    final String packageName = getPackageName();
    ArrayMap<File, SharedPreferencesImpl> packagePrefs = sSharedPrefsCache.get(packageName);
    if (packagePrefs == null) {
        packagePrefs = new ArrayMap<>();
        sSharedPrefsCache.put(packageName, packagePrefs);
    }
    return packagePrefs;
}
```

- SharedPreferencesImpl(file, mode)

```java
SharedPreferencesImpl(File file, int mode) {
    mFile = file;
    mBackupFile = makeBackupFile(file);
    mMode = mode;
    mLoaded = false;
    mMap = null;
    startLoadFromDisk();	//去磁盘中加载数据，创建实例时才会加载数据，在第一次获取的时候才会去磁盘中加载数据
}
```

- 