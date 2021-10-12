### MyNews App学习

- 开发常用缓存路径
  ```java
  Environment.getDataDirectory() 					/data
  Environment.getDownloadCacheDirectory()			/cache
  Environment.getExternalStorageDirectory()		/mnt/sdcard
  Environment.getExternalStoragePublicDirectory(“test”)		/mnt/sdcard/test
  Environment.getRootDirectory()				/system
  context.getPackageCodePath()				/data/app/com.my.app-1.apk
  context.getPackageResourcePath()			/data/app/com.my.app-1.apk
  context.getCacheDir()						/data/data/com.my.app/cache
  context.getDatabasePath(“test”)				/data/data/com.my.app/databases/test
  context.getDir(“test”, Context.MODE_PRIVATE)	/data/data/com.my.app/app_test
  context.getExternalCacheDir()				/mnt/sdcard/Android/data/com.my.app/cache
  context.getExternalFilesDir(“test”)		/mnt/sdcard/Android/data/com.my.app/files/test
  context.getExternalFilesDir(null)		/mnt/sdcard/Android/data/com.my.app/files
  context.getFilesDir()					/data/data/com.my.app/files
  ```

  

  