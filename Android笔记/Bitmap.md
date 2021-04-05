### Bitmap

#### 1、问题

- 移动设备内存资源很稀缺，每个应用分配的内存空间有限

- Bitmaps 本身非常占用资源

- 针对 ListView，GridView，ViewPager 控件有显示很多图片的需求

- BitmapFactory 提供很多通过各种渠道解码的方法：`decodeByteArray()`，`decodeFile()`，`decodeResource()` 等。这些方法容易引起 OOM，都可通过 BitmapFactory.Options 类指定解码选项。inJustDecodeBounds 设成 true 然后解码，这时不会分配内存，而是返回空的 bitmap，同时返回 outWidth，outHeight，outMimeType，然后可按照需要对图片压缩

  - 压缩注意

    - 预估加载整个图片需要内存
    - 目标 UI 控件大小
    - 设备屏幕分辨率和尺寸

  - 把一个 1024x768 图片全部加载显示在一个 128x96 像素的 ImageView 上，根据目标控件传入宽高计算合适 inSampleSize 值

    ```java
    public static int calculateInSampleSize(BitmapFactory.Options options,int reqWidth,int reqHeight) {
        //图像原始高宽
        final int height = options.outHeight;
        final int width = options.outWidth;
        int inSampleSize = 1;
        if (height > reqHeight || width > reqWidth) {
            final int halfHeight = height / 2;
            final int halfWidth = width / 2;
            //计算最大的inSampleSize值，该值是2的幂，使高宽都大于请求的高宽
            while((halfHeight / inSampleSize) > reqHeight && (halfWidth / inSampleSize) > reqWidth){
                inSampleSize *= 2;
            }
        }
        return inSampleSize;
    }
    
    public static Bitmap decodeSampledBitmapFromResource(Resources res, int resId,int reqWidth, int reqHeight){
        //第一次解码设置inJustDecodeBounds为true
        final BitmapFactory.Options options = new BitmapFactory.Options();
        options.inJustDecodeBounds = true;
        BitmapFactory.decodeResource(res, resId, options);
        //计算样本大小
        options.inSampleSize = calculateInSampleSize(options, reqWidth, reqHeight);
        //使用inSampleSize集合解码位图
        options.inJustDecodeBounds = false;
        return BitmapFactory.decodeResource(res, resId, options);
    }
    //使用
    mImageView.setImageBitmap(decodeSampledBitmapFromResource(getResources(),R.id.myimage,100,100));
    ```

#### （2）避免在 UI 线程上处理图片

- AsyncTask

  ```java
  class BitmapWorkerTask extends AsyncTask<Integer, Void, Bitmap> {
      private final WeakReference<ImageView> imageViewReference;
      private int data = 0;
      public BitmapWorkerTask(ImageView imageView) {
          //使用WeakReference确保ImageView可以被垃圾收集，避免内存泄露
          imageViewReference = new WeakReference<ImageView>(imageView);
      }
      //在背景中解码图像
      @Override
      protected Bitmap doInBackground(Integer... params) {
          data = params[0];
          return decodeSampledBitmapFromResource(getResources(), data, 100, 100));
      }
  
      //完成后查看ImageView是否仍然存在并设置位图
      @Override
      protected void onPostExecute(Bitmap bitmap) {
          if (imageViewReference != null && bitmap != null) {
              final ImageView imageView = imageViewReference.get();
              if (imageView != null) {
                  imageView.setImageBitmap(bitmap);
              }
          }
      }
  }
  
  //使用
  public void loadBitmap(int resId, ImageView imageView){
      BitmapWorkerTask task = new BitmapWorkerTask(imageView);
      task.execute(resId);
  }
  
  //保证并发，保证务完成时该子控件是否已被重用，任务开始和完成的顺序
  //创建一个专门Drawable子类储存载入图片的任务引用
  static class AsyncDrawable extends BitmapDrawable {
      private final WeakReference<BitmapWorkerTask> bitmapWorkerTaskReference;
      public AsyncDrawable(Resources res,Bitmap bitmap,BitmapWorkerTask bitmapWorkerTask){
          super(res,bitmap);
          bitmapWorkerTaskReference = new WeakReference<BitmapWorkerTask>(bitmapWorkerTask);
      }
      public BitmapWorkerTask getBitmapWorkerTask() {
          return bitmapWorkerTaskReference.get();
      }
  }
  
  //创建AsyncDrawable且绑定到目标ImageView中
  public void loadBitmap(int resId, ImageView imageView) {
      if (cancelPotentialWork(resId, imageView)) {
          final BitmapWorkerTask task = new BitmapWorkerTask(imageView);
          final AsyncDrawable asyncDrawable = new AsyncDrawable(getResources(), mPlaceHolderBitmap, task);
          imageView.setImageDrawable(asyncDrawable);
          task.execute(resId);
      }
  }
  ```

https://blog.csdn.net/caoxiao90/article/details/51545560

