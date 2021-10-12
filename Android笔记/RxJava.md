### RxJava

> Rx（Reactivex，响应式）

- 响应式：根据上一层的响应，响应当前层
- 修改需求简单

```java
//起点，第三步
Observable.just(PATH)	//内部分发PATH String
    //需求，第四步
    .map(new Function<String, Bitmap>(){
        @Override
        public Bitmap apply(String s) throws Exception{
            URL url = new URL(PATH);
            HttpURLConnecion conn = (HttpURLConnecion)url.openConnection();
            conn.setConnectTimeout(5000);
            int responseCode = conn.getResponseCode();
            if(responseCode == HttpURLConnection.HTTP_OK){
                InputStream is = conn.getInputStream();
                Bitmap bitmap = BitmapFactory.decodeStream(is);
                return bitmap;
            }
            return null;
        }
    })
    //给上面分配异步线程
    .subscribeOn(Schedulers.io())
    //给下面分配Android主线程
    .observeOn(AndroidSchedulers.mainThread())
    //起点和终点关联起来，订阅,第一步
    .subcribe(
    new Observer<Bitmap>(){
        //第二步
        @Override
        public void onSubcribe(Disposable d){
            //加载框
            progressDialog = new ProgressDialog(MainActivity.this);
            progressDialog.setTitle("RXJava");
            progressDialog.show();
        }
        //第五步
        @Override
        public void onNext(Bitmap bitmap){
            image.setImageBitmap(bitmap);
        }
        
        @Override
        public void onError(Throwable e){}
		//第六步，链条结束
        @Override
        public void onComplete(){
            if(progressDialog != null){
                progressDialog.dismiss();
            }
        }
    }
)
```

