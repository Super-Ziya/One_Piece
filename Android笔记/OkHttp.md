### OkHttp

#### 1、基本流程

##### （1）初始化 OkHttpClient

- 设计模式
  - Builder 模式

```java
//1、默认构造方式
public OkHttpClient() {
    this(new Builder());
}
//默认是调用该构造函数
OkHttpClient(Builder builder) {...}
//2、Builder 构造模式
public OkHttpClient build() {
    return new OkHttpClient(this);
}
```

##### （2）Request

- 设计模式
  - Builder 模式

```java
Request(Builder builder) {
    this.url = builder.url;
    this.method = builder.method;
    this.headers = builder.headers.build();
    this.body = builder.body;
    this.tag = builder.tag != null ? builder.tag : this;
}

public Builder newBuilder() {
    return new Builder(this);
}
////Builder是Request的内部类
public Builder() {
    this.method = "GET";
    this.headers = new Headers.Builder();
}

Builder(Request request) {
    this.url = request.url;
    this.method = request.method;
    this.body = request.body;
    this.tag = request.tag;
    this.headers = request.headers.newBuilder();
}

public Request build() {
    if (url == null) throw new IllegalStateException("url == null");
    return new Request(this);
}
```

##### （3）异步请求

- 构建 Call，一般 `Call call = mOkHttpClient.newCall(request);` 

  - 设计模式
    - 工厂模式：newCall 方法、eventListener 对象

  ```java
  public class OkHttpClient implements Cloneable, Call.Factory, WebSocket.Factory {
      //...
      //准备{@code request}在将来执行
      @Override 
      public Call newCall(Request request) {
          //工厂模式：构建细节交给具体实现，顶层只需拿到Call对象
          return RealCall.newRealCall(this, request, false /* for web socket */);
      }
      //...
  }
  
  //OkhttpClient实现了Call.Factory接口
  interface Factory {
      Call newCall(Request request);
  }
  
  //newRealCall
  final class RealCall implements Call {
      //...
      static RealCall newRealCall(OkHttpClient client, Request originalRequest, boolean forWebSocket){
          //安全地将调用实例发布到EventListener
          RealCall call = new RealCall(client, originalRequest, forWebSocket);
          call.eventListener = client.eventListenerFactory().create(call);
          return call;
      }
      
      private RealCall(OkHttpClient client,Request originalRequest,boolean forWebSocket)	  {
          this.client = client;
          this.originalRequest = originalRequest;
          this.forWebSocket = forWebSocket;
          //默认创建一个retryAndFollowUpInterceptor过滤器
          this.retryAndFollowUpInterceptor = new RetryAndFollowUpInterceptor(client, forWebSocket);
      }
      //...
  }
  ```

- Call 的调度

  ```java
  //请求加入调度(RealCall)
  call.enqueue(new Callback(){
      @Override
      public void onFailure(Request request, IOException e)
      {}
  
      @Override
      public void onResponse(final Response response) throws IOException
      {}
  });
  
  //RealCall的enqueue方法
  @Override 
  public void enqueue(Callback responseCallback) {
      //加入对象锁，防止多线程同时调用，executed判断当前call是否被执行了
      synchronized (this) {
          if (executed) throw new IllegalStateException("Already Executed");
          executed = true;
      }
      captureCallStackTrace();
      //回调eventListener的callStart方法
      eventListener.callStart(this);
      client.dispatcher().enqueue(new AsyncCall(responseCallback));
  }
  
  //captureCallStackTrace方法
  private void captureCallStackTrace() {
      Object callStackTrace = Platform.get().getStackTraceForCloseable("response.body().close()");
  	//为retryAndFollowUpInterceptor加入一个用于追踪堆栈信息的callStackTrace
      retryAndFollowUpInterceptor.setCallStackTrace(callStackTrace);
  }
  
  //OKHTTPClient的dispatcher方法
  public Dispatcher dispatcher() {
      return dispatcher;
  }
  
  //Dispatcher
  public final class Dispatcher {
      private int maxRequests = 64;
      private int maxRequestsPerHost = 5;
      private @Nullable Runnable idleCallback;
  
      //执行calls，懒汉式
      private @Nullable ExecutorService executorService;
      //按运行顺序的异步等待队列
      private final Deque<AsyncCall> readyAsyncCalls = new ArrayDeque<>();
      //异步运行队列，包括未结束但已取消的Calls
      private final Deque<AsyncCall> runningAsyncCalls = new ArrayDeque<>();
      //同步运行队列，包括未结束但已取消的Calls
      private final Deque<RealCall> runningSyncCalls = new ArrayDeque<>();
      //...
  }
  
  //enqueue方法
  synchronized void enqueue(AsyncCall call) {
      //正在执行的异步队列个数小于maxRequest(64)且请求同一主机的个数小于maxRequestsPerHost(5)时
      if (runningAsyncCalls.size() < maxRequests && runningCallsForHost(call) < maxRequestsPerHost) {
          //将请求加入异步运行队列runningAsyncCall
          runningAsyncCalls.add(call);
          //用线程池执行该call
          executorService().execute(call);
      } else {
          //否则加入异步等待队列
          readyAsyncCalls.add(call);
      }
  }
  
  //runningAsyncCalls方法
  //返回与{@code call}共享主机的正在运行的调用数
  private int runningCallsForHost(AsyncCall call) {
      int result = 0;
      //遍历runningAsyncCalls，记录同一Host的个数
      for (AsyncCall c : runningAsyncCalls) {
          if (c.host().equals(call.host())) result++;
      }
      return result;
  }
  
  //AsyncCall类，继承NameRunnable类
  final class AsyncCall extends NamedRunnable {
      //...
      @Override
      protected void execute() {
          boolean signalledCallback = false;
          try {
              //异步和同步走同样方式
              Response response = getResponseWithInterceptorChain();
              if (retryAndFollowUpInterceptor.isCanceled()) {
                  signalledCallback = true;
                  responseCallback.onFailure(RealCall.this,new IOException("Canceled"));
              } else {
                  signalledCallback = true;
                  responseCallback.onResponse(RealCall.this, response);
              }
          } catch (IOException e) {
              if (signalledCallback) {
                  //不要向回调发出两次信号
                  Platform.get().log(INFO,"Callback failure for "+toLoggableString(),e);
              } else {
                  eventListener.callFailed(RealCall.this, e);
                  responseCallback.onFailure(RealCall.this, e);
              }
          } finally {
              client.dispatcher().finished(this);
          }
      }
      //...
  }
  
  //NameRunnable类，继承Runnable接口
  public abstract class NamedRunnable implements Runnable {
      protected final String name;
      public NamedRunnable(String format, Object... args) {
          this.name = Util.format(format, args);
      }
      @Override
      public final void run() {
          String oldName = Thread.currentThread().getName();
      	//将当前执行的线程名字设为在构造方法中传入的名字
          Thread.currentThread().setName(name);
          try {
              execute();
          } finally {
              //名字再设置回来
              Thread.currentThread().setName(oldName);
          }
      }
      protected abstract void execute();
  }
  
  //getResponseWithInterceptorChain方法
  Response getResponseWithInterceptorChain() throws IOException {
      //建立一个完整的拦截器栈
      List<Interceptor> interceptors = new ArrayList<>();
      interceptors.addAll(client.interceptors());
      //失败和重定向过滤器
      interceptors.add(retryAndFollowUpInterceptor);
      //封装request和response过滤器
      interceptors.add(new BridgeInterceptor(client.cookieJar()));
      //缓存相关的过滤器，负责读取缓存直接返回、更新缓存
      interceptors.add(new CacheInterceptor(client.internalCache()));
      //负责和服务器建立连接
      interceptors.add(new ConnectInterceptor(client));
      if (!forWebSocket) {
          //配置OkHttpClient时设置的networkInterceptors
          interceptors.addAll(client.networkInterceptors());
      }
      //负责向服务器发送请求数据、从服务器读取响应数据(实际网络请求)
      interceptors.add(new CallServerInterceptor(forWebSocket));
      Interceptor.Chain chain = new RealInterceptorChain(interceptors, null, null, null, 0, originalRequest, this, eventListener, client.connectTimeoutMillis(), client.readTimeoutMillis(), client.writeTimeoutMillis());
      return chain.proceed(originalRequest);
  }
  ```

- 过滤器（高内聚，低耦合，可拓展）

  - retryAndFollowUpInterceptor：失败和重定向过滤器
  - BridgeInterceptor：封装 request 和 response 过滤器
  - CacheInterceptor：缓存相关的过滤器，负责读取缓存直接返回、更新缓存
  - ConnectInterceptor：负责和服务器建立连接，连接池等
  - networkInterceptors：配置 OkHttpClient 时设置的 networkInterceptors
  - CallServerInterceptor：负责向服务器发送请求数据、从服务器读取响应数据（实际网络请求）

  ```java
  public Response proceed(Request request, StreamAllocation streamAllocation, HttpCodec httpCodec, RealConnection connection) throws IOException {
      if (index >= interceptors.size()) throw new AssertionError();
      calls++;
      //如果已有一个流，确认传入的请求将使用它
      if(this.httpCodec != null && !this.connection.supportsUrl(request.url())) {
          throw new IllegalStateException("network interceptor " + interceptors.get(index - 1) + " must retain the same host and port");
      }
      //如果已有一个流，确认这是chain.proceed()唯一的call
      if (this.httpCodec != null && calls > 1) {
          throw new IllegalStateException("network interceptor " + interceptors.get(index - 1) + " must call proceed() exactly once");
      }
      //调用链中下一个拦截器，index=0，使用递归遍历过滤器链
      RealInterceptorChain next = new RealInterceptorChain(interceptors, streamAllocation, httpCodec, connection, index + 1, request, call, eventListener, connectTimeout, readTimeout,  writeTimeout);
      Interceptor interceptor = interceptors.get(index);
      Response response = interceptor.intercept(next);
      //确认下一个拦截器给chain.proceed()其需求的call
      if (httpCodec != null && index + 1 < interceptors.size() && next.calls != 1) {
          throw new IllegalStateException("network interceptor " + interceptor + " must call proceed() exactly once");
      }
      //确认截获的响应不是空的
      if (response == null) {
          throw new NullPointerException("interceptor " + interceptor + " returned null");
      }
      if (response.body() == null) {
          throw new IllegalStateException("interceptor " + interceptor + " returned a response with no body");
      }
      return response;
  }
  ```

  - ConnectInterceptor 过滤器

    - 递归循环
    - 责任链模式

    ```java
    public final class ConnectInterceptor implements Interceptor {
        public final OkHttpClient client;
        public ConnectInterceptor(OkHttpClient client) {
            this.client = client;
        }
        @Override
        public Response intercept(Chain chain) throws IOException {
            RealInterceptorChain realChain = (RealInterceptorChain) chain;
            //...
            return realChain.proceed(request,streamAllocation,httpCodec,connection);
        }
    }
    ```

##### （4）同步请求

- 同步异步核心都是运行 `getResponseWithInterceptorChain()`

```java
@Override
public Response execute() throws IOException {
    //检查这个call是否运行过
    synchronized (this) {
        if (executed) throw new IllegalStateException("Already Executed");
        executed = true;
    }
    captureCallStackTrace();
    //回调
    eventListener.callStart(this);
    try {
        //将请求加入到同步队列中
        client.dispatcher().executed(this);
        //创建过滤器责任链，得到response
        Response result = getResponseWithInterceptorChain();
        if (result == null) throw new IOException("Canceled");
        return result;
    } catch (IOException e) {
        eventListener.callFailed(this, e);
        throw e;
    } finally {
        client.dispatcher().finished(this);
    }
}
```

![OkHttp_请求流程](Image.assets\OkHttp_请求流程.png)

#### 2、RetryAndFollowUpInterceptor 过滤器

> 重试和重定向

- 需要重定向时
  - 关闭响应流
  - 增加重定向次数，保证小于最大重定向次数 MAX_FOLLOW_UPS（20）
  - 不能是 UnrepeatableRequestBody 类型（用于标记那些只能请求一次的请求）
  - 判断是否相同，不相同则需重新创建一个 streamConnection
  - 重新赋值，结束当前循环，继续 while 循环（执行重定向请求）
- 拒绝重试的条件
  - 配置 OkHttpClient 中 retryOnConnectionFailure 为 false，表明拒绝失败重连
  - 如果请求已发送，且请求体是个 UnrepeatableRequestBody 类型，则不能重试
  - 如果是些严重问题（协议，安全等），拒绝重试
  - 没有更多可使用路由，则不重试

- intercept(Chain chain)

  - StreamAllocation：realChain.proceed 参数之一，处理 Connections、Streams、Calls 三者的关系

  ```java
  @Override
  public Response intercept(Chain chain) throws IOException {
      Request request = chain.request();
      RealInterceptorChain realChain = (RealInterceptorChain) chain;
      Call call = realChain.call();
      EventListener eventListener = realChain.eventListener();
      //streamAllocation的创建位置
      streamAllocation = new StreamAllocation(client.connectionPool(), createAddress(request.url()), call, eventListener, callStackTrace);
      int followUpCount = 0;
      Response priorResponse = null;
      while (true) {
          //取消
          if (canceled) {
              streamAllocation.release();
              throw new IOException("Canceled");
          }
          Response response;
          boolean releaseConnection = true;
          try {
              //递归主要方法
              response = realChain.proceed(request, streamAllocation, null, null);
              releaseConnection = false;
          } catch (RouteException e) {
              //尝试连接一个路由失败，请求还没被发送
              if (!recover(e.getLastConnectException(), false, request)) {
                  throw e.getLastConnectException();
              }
              releaseConnection = false;
              //重试，重新执行while循环体
              continue;
          } catch (IOException e) {
              //尝试与服务器通信失败，请求可能已发送
              //先判断当前请求是否已发送
              boolean requestSendStarted = !(e instanceof ConnectionShutdownException);
              //同样的重试判断
              if (!recover(e, requestSendStarted, request)) throw e;
              releaseConnection = false;
              //重试，重新执行while循环体
              continue;
          } finally {
              //执行时未检测到异常（releaseConnection为初始值true），释放部分资源
              if (releaseConnection) {
                  streamAllocation.streamFailed(null);
                  streamAllocation.release();
              }
          }
          //如果存在，附上先前的response，这样的response没有body
          //priorResponse用来保存前一个Resposne，这里将前一个Response和当前的Resposne结合，对应场景是当获得Resposne后发现需要重定向，则将当前Resposne设置给priorResponse，再执行一遍流程直到不需重定向，将priorResponse和Resposne结合起来
          if (priorResponse != null) {
              response = response.newBuilder()
                  .priorResponse(priorResponse.newBuilder()
                                 .body(null)
                                 .build()).build();
          }
          //判断是否需重定向，需重定向则返回一个重定向的Request，没有则为null
          Request followUp = followUpRequest(response);
          if (followUp == null) {
              //不需要重定向
              if (!forWebSocket) {
                  //是WebSocket，释放
                  streamAllocation.release();
              }
              //返回response
              return response;
          }
          //需重定向，关闭响应流
          closeQuietly(response.body());
          //重定向次数加1，且小于最大重定向次数MAX_FOLLOW_UPS(20)
          if (++followUpCount > MAX_FOLLOW_UPS) {
              streamAllocation.release();
              throw new ProtocolException("Too many follow-up requests: " + followUpCount);
          }
          //是UnrepeatableRequestBody(流类型)，没被缓存，不能重定向
          if (followUp.body() instanceof UnrepeatableRequestBody) {
              streamAllocation.release();
              throw new HttpRetryException("Cannot retry streamed HTTP body", response.code());
          }
          //判断是否相同，不然重新创建一个streamConnection
          if (!sameConnection(response, followUp.url())) {
              streamAllocation.release();
              streamAllocation = new StreamAllocation(client.connectionPool(), createAddress(followUp.url()), call, eventListener, callStackTrace);
          } else if (streamAllocation.codec() != null) {
              throw new IllegalStateException("Closing the body of " + response + " didn't close its backing stream. Bad interceptor?");
          }
          //赋值再来
          request = followUp;
          priorResponse = response;
      }
  }
  
  //recover方法
  //requestSendStarted：表明请求是否被发送
  private boolean recover(IOException e, boolean requestSendStarted, Request userRequest) {
      streamAllocation.streamFailed(e);
      //应用层禁止重试，如果OkHttpClient直接配置拒绝失败重连(retryOnConnectionFailure为false，默认方式创建OkHttpClient该属性是true)，return false
      if (!client.retryOnConnectionFailure()) return false;
      //如果请求已发送，且请求体是一个UnrepeatableRequestBody类型，则不能重试
      //StreamedRequestBody实现UnrepeatableRequestBody接口，是流类型，不会被缓存，只能执行一次
      if (requestSendStarted && userRequest.body() instanceof UnrepeatableRequestBody) return false;
      //此异常是致命的，一些严重的问题就不要重试
      if (!isRecoverable(e, requestSendStarted)) return false;
      //没有更多的路由就不要重试了
      if (!streamAllocation.hasMoreRoutes()) return false;
      //对于故障恢复，对新连接使用相同路由选择器
      return true;
  }
  
  //UnrepeatableRequestBody类
  //空接口，作用是标记不能被重复请求的请求体，目前只有StreamedRequestBody实现该接口
  public interface UnrepeatableRequestBody {}
  
  //isRecoverable方法
  //归纳了一些严重的错误：协议错误、超时问题可重传、安全问题
  private boolean isRecoverable(IOException e, boolean requestSendStarted) {
      //如果是协议问题，不要在重试了
      if (e instanceof ProtocolException) {
          return false;
      }
      //如果有中断则不恢复，如果连接某个路由时超时，尝试下一个路由(如果有)
      if (e instanceof InterruptedIOException) {
          //超时且请求还没被发送可以重试，其他不要重试
          return e instanceof SocketTimeoutException && !requestSendStarted;
      }
      //查找已知的客户端错误或协商错误，不太可能通过使用不同路由重试修复
      if (e instanceof SSLHandshakeException) {
          //如果问题来自X509TrustManager的CertificateException，不要重试（安全原因）
          if (e.getCause() instanceof CertificateException) {
              return false;
          }
      }
      if (e instanceof SSLPeerUnverifiedException) {
          //如证书固定错误（安全原因）
          return false;
      }
      return true;
  }
  
  //StreamAllocation的hasMoreRoutes方法
  //routeSelection用List保存
  public boolean hasMoreRoutes() {
      return route != null || (routeSelection != null && routeSelection.hasNext()) || routeSelector.hasNext();
  }
  
  //followUpRequest方法
  //当返回码满足某些条件时就重新构造一个Request，不满足就返回null
  private Request followUpRequest(Response userResponse) throws IOException {
      if (userResponse == null) throw new IllegalStateException();
      Connection connection = streamAllocation.connection();
      Route route = connection != null ? connection.route() : null;
      int responseCode = userResponse.code();
      final String method = userResponse.request().method();
      switch (responseCode) {
          case HTTP_PROXY_AUTH:
              Proxy selectedProxy = route != null ? route.proxy() : client.proxy();
              if (selectedProxy.type() != Proxy.Type.HTTP) {
                  throw new ProtocolException("Received HTTP_PROXY_AUTH (407) code while not using proxy");
              }
              return client.proxyAuthenticator().authenticate(route, userResponse);
          case HTTP_UNAUTHORIZED:
              return client.authenticator().authenticate(route, userResponse);
          case HTTP_PERM_REDIRECT:
          case HTTP_TEMP_REDIRECT:
              //如果响应GET或HEAD以外的请求，而接收到307或308状态码，用户代理不得自动重定向该请求
              if (!method.equals("GET") && !method.equals("HEAD")) {
                  return null;
              }
              //落空
          case HTTP_MULT_CHOICE:
          case HTTP_MOVED_PERM:
          case HTTP_MOVED_TEMP:
          case HTTP_SEE_OTHER:
              //客户端是否允许重定向
              if (!client.followRedirects()) return null;
              String location = userResponse.header("Location");
              if (location == null) return null;
              HttpUrl url = userResponse.request().url().resolve(location);
              //不遵循重定向到不支持的协议
              if (url == null) return null;
              //如果已配置，则不遵循SSL和非SSL之间的重定向
              boolean sameScheme = url.scheme().equals(userResponse.request().url().scheme());
              if (!sameScheme && !client.followSslRedirects()) return null;
              //大多数重定向不包括请求主体
              Request.Builder requestBuilder = userResponse.request().newBuilder();
              if (HttpMethod.permitsRequestBody(method)) {
                  final boolean maintainBody = HttpMethod.redirectsWithBody(method);
                  if (HttpMethod.redirectsToGet(method)) {
                      requestBuilder.method("GET", null);
                  } else {
                      RequestBody requestBody = maintainBody ? userResponse.request().body() : null;
                      requestBuilder.method(method, requestBody);
                  }
                  if (!maintainBody) {
                      requestBuilder.removeHeader("Transfer-Encoding");
                      requestBuilder.removeHeader("Content-Length");
                      requestBuilder.removeHeader("Content-Type");
                  }
              }
              //跨主机重定向时，删除所有身份验证标头
              if (!sameConnection(userResponse, url)) {
                  requestBuilder.removeHeader("Authorization");
              }
              //重新构造一个Request
              return requestBuilder.url(url).build();
          case HTTP_CLIENT_TIMEOUT:
              //408很少见，但有些服务器（如HAProxy）使用此响应代码。说明书说可不加修改地重复该请求，现代浏览器也会重复该请求（即使是非幂等的）
              if (!client.retryOnConnectionFailure()) {
                  //应用层已指示不要重试请求
                  return null;
              }
              if (userResponse.request().body() instanceof UnrepeatableRequestBody) {
                  return null;
              }
              if (userResponse.priorResponse() != null && userResponse.priorResponse().code() == HTTP_CLIENT_TIMEOUT) {
                  //试图重试，但超时，放弃
                  return null;
              }
              return userResponse.request();
          default:
              return null;
      }
  }
  
  //sameConnection方法
  private boolean sameConnection(Response response, HttpUrl followUp) {
      HttpUrl url = response.request().url();
      return url.host().equals(followUp.host()) && url.port() == followUp.port() && url.scheme().equals(followUp.scheme());
  }
  ```

#### 3、BridgeInterceptor 过滤器

- 对 Request 和 Resposne 的封装
- 设置内容长度、内容编码
- 设置 gzip header 压缩，接收到内容后解压，省去应用层处理数据解压的麻烦
- 添加 cookie
- 设置其他报头，如 User-Agent、Host、Keep-alive（多路复用）

#### 4、CacheInterceptor 过滤器

- 负责 cache 缓存管理
- 网络请求有符合要求的 cache 时直接返回
- 服务器返回内容有改变时更新当前 cache
- 如果当前 cache 失效，删除
- 过程细节

```java
@Override
public Response intercept(Chain chain) throws IOException {
    //1、尝试通过该Request拿缓存
    Response cacheCandidate = cache != null ? cache.get(chain.request()) : null;
    long now = System.currentTimeMillis();
    //根据response,time,request创建一个缓存策略，用于判断怎样使用缓存
    CacheStrategy strategy = new CacheStrategy.Factory(now, chain.request(), cacheCandidate).get();
    Request networkRequest = strategy.networkRequest;
    Response cacheResponse = strategy.cacheResponse;
    if (cache != null) {
        cache.trackResponse(strategy);
    }
    if (cacheCandidate != null && cacheResponse == null) {
        closeQuietly(cacheCandidate.body()); //缓存候选对象不适用，关闭
    }
    //2、如果缓存策略中禁止使用网络，且缓存为空，构建一个Resposne直接返回，返回码504
    if (networkRequest == null && cacheResponse == null) {
        return new Response.Builder()
            .request(chain.request())
            .protocol(Protocol.HTTP_1_1)
            .code(504)
            .message("Unsatisfiable Request (only-if-cached)")
            .body(Util.EMPTY_RESPONSE)
            .sentRequestAtMillis(-1L)
            .receivedResponseAtMillis(System.currentTimeMillis())
            .build();
    }
    //3、不使用网络，但有缓存，直接返回缓存
    if (networkRequest == null) {
        return cacheResponse.newBuilder()
            .cacheResponse(stripBody(cacheResponse))
            .build();
    }
    Response networkResponse = null;
    try {
        //4、链式调用下一个过滤器
        networkResponse = chain.proceed(networkRequest);
    } finally {
        //如果在I/O或其他方面崩溃，不要泄漏缓存
        if (networkResponse == null && cacheCandidate != null) {
            closeQuietly(cacheCandidate.body());
        }
    }
    //5、当缓存响应和网络响应同时存在时选择用哪个
    if (cacheResponse != null) {
        if (networkResponse.code() == HTTP_NOT_MODIFIED) {
            //如果返回码是304，客户端有缓冲文档并发出一个条件性请求（一般是提供If-Modified-Since头表示客户只想比指定日期更新的文档），服务器告诉客户原来缓冲的文档还可继续使用，则使用缓存的响应
            Response response = cacheResponse.newBuilder()
                .headers(combine(cacheResponse.headers(), networkResponse.headers()))
                .sentRequestAtMillis(networkResponse.sentRequestAtMillis())
                .receivedResponseAtMillis(networkResponse.receivedResponseAtMillis())
                .cacheResponse(stripBody(cacheResponse))
                .networkResponse(stripBody(networkResponse))
                .build();
            networkResponse.body().close();
            //在合并头后、剥离内容编码头前更新缓存（initContentStream()执行）
            cache.trackConditionalCacheHit();
            cache.update(cacheResponse, response);
            return response;
        } else {
            closeQuietly(cacheResponse.body());
        }
    }
    //6、使用网络请求得到的Resposne，且将其缓存起来（前提当然是能缓存）
    Response response = networkResponse.newBuilder()
        .cacheResponse(stripBody(cacheResponse))
        .networkResponse(stripBody(networkResponse))
        .build();
    //默认创建的OkHttpClient没有缓存
    if (cache != null) {
        //将响应缓存
        if (HttpHeaders.hasBody(response) && CacheStrategy.isCacheable(response, networkRequest)) {
            //缓存Resposne的Header信息
            CacheRequest cacheRequest = cache.put(response);
            //缓存body
            return cacheWritingResponse(cacheRequest, response);
        }
        //只能缓存GET，不然移除request
        if (HttpMethod.invalidatesCache(networkRequest.method())) {
            try {
                cache.remove(networkRequest);
            } catch (IOException ignored) {
                //无法写入缓存
            }
        }
    }
    return response;
}

//CacheInterceptor类
public final class CacheInterceptor implements Interceptor {
    final InternalCache cache;
    public CacheInterceptor(InternalCache cache) {
        this.cache = cache;
    }
}

//RealCall构造该过滤器也传入OkHttpClient设置的interanlCache，默认是null
interceptors.add(new CacheInterceptor(client.internalCache()));

//InternalCache接口
public interface InternalCache {
    Response get(Request request) throws IOException;
    CacheRequest put(Response response) throws IOException;
    void remove(Request request) throws IOException;
    void update(Response cached, Response network);
    void trackConditionalCacheHit();
    void trackResponse(CacheStrategy cacheStrategy);
}

//Cache类实现了InternalCache接口
public final class Cache implements Closeable, Flushable {
    final InternalCache internalCache = new InternalCache() {
        @Override
        public Response get(Request request) throws IOException {
            return Cache.this.get(request);
        }
    };
    //1.通过执行DiskLruCache.get方法拿到snapshot信息
    //2.通过拿到的snapshot信息，取cleanFiles[0]中保存的头信息，构建头相关信息的Entry
    //3.通过snapshot的cleanFiles[1]构建body信息，最终构建成缓存中保存的Response
    //4.返回缓存中保存的Resposne
    @Nullable
    Response get(Request request) {
        //缓存的Key和request的url直接相关
        String key = key(request.url());
        //DiskLruCache类型
        DiskLruCache.Snapshot snapshot;
        Entry entry;
        try {
            snapshot = cache.get(key);
            if (snapshot == null) {
                //没拿到，返回null
                return null;
            }
        } catch (IOException e) {
            //无法读取缓存，放弃
            return null;
        }
        try {
            //创建一个Entry（Cache内部类），这里传入CleanFiles数组第一个（ENTRY_METADATA=0）得到头信息（key.0）
            entry = new Entry(snapshot.getSource(ENTRY_METADATA));
        } catch (IOException e) {
            Util.closeQuietly(snapshot);
            return null;
        }
        //得到缓存构建得到的response
        Response response = entry.response(snapshot);
        if (!entry.matches(request, response)) {
            Util.closeQuietly(response.body());
            return null;
        }
        return response;
    }
}

//DiskLruCache.get方法
//1.初始化日志文件和lruEntries
//2.检查保证key正确后获取缓存中保存的Entry
//3.操作计数器+1
//4.往日志文件中写入这次的READ操作
//5.根据redundantOpCount判断是否需要清理日志信息
//6.需要则开启线程清理
//7.不需要则返回缓存
public synchronized Snapshot get(String key) throws IOException {
    //对journalFile文件的操作，有则删除无用冗余信息，构建新文件，没有则new个新的
    initialize();
    //判断是否关闭，如果缓存损坏会被关闭
    checkNotClosed();
    //检查key是否满足格式要求，正则表达式
    validateKey(key);
    //获取key对应的entry
    Entry entry = lruEntries.get(key);
    if (entry == null || !entry.readable) return null;
    //获取entry里snapshot的值
    Snapshot snapshot = entry.snapshot();
    if (snapshot == null) return null;
    //有则计数器+1
    redundantOpCount++;
    //把这个内容写入文档中
    journalWriter.writeUtf8(READ).writeByte(' ').writeUtf8(key).writeByte('\n');
    //判断是否达清理条件
    if (journalRebuildRequired()) {
        //开启线程清理
        executor.execute(cleanupRunnable);
    }
    return snapshot;
}

//DiskLruCache.initialize方法
//1.线程安全
//2.如果初始化了，则什么都不干，只初始化一遍
//3.如果有journalFile日志文件，对journalFile文件和lruEntries进行初始化，主要是删除冗余信息和DIRTY信息
//4.没有则构建一个journalFile文件
public synchronized void initialize() throws IOException {
    //断言，当持有自己锁时继续执行，没有持有锁直接抛异常
    assert Thread.holdsLock(this);
    //如果初始化过，直接跳出
    if (initialized) {
        return;
    }
    // .
    //如果有journalFileBackup这个文件（日志备份文件）
    if (fileSystem.exists(journalFileBackup)) {
        //如果有journalFile这个文件（日志记录文件，对缓存一系列操作的记录，不影响缓存执行流程）
        if (fileSystem.exists(journalFile)) {
            //删除journalFileBackup这个文件（日志备份文件）
            fileSystem.delete(journalFileBackup);
        } else {
            //没有journalFile这个文件，且有journalFileBackup这个文件，将journalFileBackup改名为journalFile
            fileSystem.rename(journalFileBackup, journalFile);
        }
    }
    //最后结果只有：什么都没有、有journalFile文件
    if (fileSystem.exists(journalFile)) {
        //如果有journalFile文件
        try {
            readJournal();
            processJournal();
            //标记初始化完成，无论有没有journal文件都标记为true
            initialized = true;
            return;
        } catch (IOException journalIsCorrupt) {
            Platform.get().log(WARN, "DiskLruCache " + directory + " is corrupt: " + journalIsCorrupt.getMessage() + ", removing", journalIsCorrupt);
        }
        //缓存已损坏，请尝试删除目录内容。这可能会抛出，因为可能存在严重的文件系统问题
        try {
            //有缓存损坏导致异常，则删除缓存目录下所有文件
            delete();
        } finally {
            closed = false;
        }
    }
    //如果没有journaFile,则重新创建一个
    rebuildJournal();
    //标记初始化完成,无论有没有journal文件，initialized都会标记为true，只执行一遍
    initialized = true;
}

//DiskLruCache.readJournal方法
private void readJournal() throws IOException {
    //利用Okio读取journalFile文件
    //Okio库内部对输入输出流进行很多优化，分帧读取写入、帧、池概念
    BufferedSource source = Okio.buffer(fileSystem.source(journalFile));
    try {
        String magic = source.readUtf8LineStrict();
        String version = source.readUtf8LineStrict();
        String appVersionString = source.readUtf8LineStrict();
        String valueCountString = source.readUtf8LineStrict();
        String blank = source.readUtf8LineStrict();
        //保证和默认值相同
        if (!MAGIC.equals(magic) || !VERSION_1.equals(version) || !Integer.toString(appVersion).equals(appVersionString) || !Integer.toString(valueCount).equals(valueCountString) || !"".equals(blank)) {
            throw new IOException("unexpected journal header: [" + magic + ", " + version + ", " + valueCountString + ", " + blank + "]");
        }
        int lineCount = 0;
        while (true) {
            try {
                //逐行读取，根据每行开头，不同状态执行不同操作，主要是往lruEntries里add或remove
                readJournalLine(source.readUtf8LineStrict());
                lineCount++;//记录行数
            } catch (EOFException endOfJournal) {
                break;
            }
        }
        //日志操作记录数=总行数-lruEntries中实际add的行数
        redundantOpCount = lineCount - lruEntries.size();
        //source.exhausted()表示是否还多余字节，如果没有多余字节，返回true，有多余字节返回false
        if (!source.exhausted()) {
            //有多余的字节，则重新构建下journal文件
            rebuildJournal();
        } else {
            //没有多余的字节，获取这个文件的Sink,以便Writer
            journalWriter = newJournalWriter();
        }
    } finally {
        Util.closeQuietly(source);
    }
}

//DiskLruCache.readJournalLine方法
//journalFile每行保存格式：REMOVE sdkjlg 2341 1234
//第一个空格前代表日志操作内容，后面的第一个保存的是key，后面两个内容根据前面的操作存入缓存内容对应的length
private void readJournalLine(String line) throws IOException {
    //记录第一个空串位置
    int firstSpace = line.indexOf(' ');
    if (firstSpace == -1) {
        throw new IOException("unexpected journal line: " + line);
    }
    int keyBegin = firstSpace + 1;
    //记录第二个空串的位置
    int secondSpace = line.indexOf(' ', keyBegin);
    final String key;
    if (secondSpace == -1) {
        //如果中间没有空串，直接截取得到key（REMOVE skjdglajslkgjl格式）
        key = line.substring(keyBegin);
        //如果解析出来是"REMOVE skjdglajslkgjl"这样以REMOVE开头
        if (firstSpace == REMOVE.length() && line.startsWith(REMOVE)) {
            //移除这个key，lruEntries是LinkedHashMap
            lruEntries.remove(key);
            return;
        }
    } else {
        //解析两个空格间的字符串为key
        key = line.substring(keyBegin, secondSpace);
    }
    //取出Entry对象
    Entry entry = lruEntries.get(key);
    //如果Enty对象为null
    if (entry == null) {
        //new一个Entry，put进去
        entry = new Entry(key);
        lruEntries.put(key, entry);
    }
    //如果是“CLEAN 1 2”这样的以CLAEN开头
    if (secondSpace != -1 && firstSpace == CLEAN.length() && line.startsWith(CLEAN)) {
        //取第二个空格后面的字符串，parts变成[1,2]
        String[] parts = line.substring(secondSpace + 1).split(" ");
        //可读
        entry.readable = true;
        //不被编辑
        entry.currentEditor = null;
        //设置长度
        entry.setLengths(parts);
    } else if (secondSpace == -1 && firstSpace == DIRTY.length() && line.startsWith(DIRTY)) {
        //如果是“DIRTY lskdjfkl”这样以DIRTY开头，新建一个Editor
        entry.currentEditor = new Editor(entry);
    } else if (secondSpace==-1 && firstSpace==READ.length() && line.startsWith(READ)) {
        //如果是“READ slkjl”这样以READ开头，不需要做什么事
    } else {
        throw new IOException("unexpected journal line: " + line);
    }
}

//DiskLruCache内部类Entry
private final class Entry {
    final String key;
    //entry文件的长度
    final long[] lengths;
    //用于保存持久数据，作用是读取 最后的格式：key.0
    final File[] cleanFiles;
    //用于保存编辑的临时数据，作用是写，写操作后赋值给cleanFiles，最后的格式：key.0.tmp
    final File[] dirtyFiles;
}

//DiskLruCache.rebuildJournal方法
//将lruEntries中保存的内容逐行写成一个journalFileTmp，将新构建的journalFileTmp替换当前包含冗余信息的journalFile文件，达到重新构建的效果
//作用：读取journalFile，根据日志文件的日志信息过滤无用冗余的信息，有冗余则重新构建，保证journalFile日志文件没有冗余信息
synchronized void rebuildJournal() throws IOException {
    if (journalWriter != null) {
        journalWriter.close();
    }
    BufferedSink writer = Okio.buffer(fileSystem.sink(journalFileTmp));
    try {
        //写入校验信息
        writer.writeUtf8(MAGIC).writeByte('\n');
        writer.writeUtf8(VERSION_1).writeByte('\n');
        writer.writeDecimalLong(appVersion).writeByte('\n');
        writer.writeDecimalLong(valueCount).writeByte('\n');
        writer.writeByte('\n');
        //利用刚才逐行读的内容按照格式重新构建
        for (Entry entry : lruEntries.values()) {
            if (entry.currentEditor != null) {
                writer.writeUtf8(DIRTY).writeByte(' ');
                writer.writeUtf8(entry.key);
                writer.writeByte('\n');
            } else {
                writer.writeUtf8(CLEAN).writeByte(' ');
                writer.writeUtf8(entry.key);
                entry.writeLengths(writer);
                writer.writeByte('\n');
            }
        }
    } finally {
        writer.close();
    }
    //用新构建的journalFileTmp替换当前的journalFile文件
    if (fileSystem.exists(journalFile)) {
        fileSystem.rename(journalFile, journalFileBackup);
    }
    fileSystem.rename(journalFileTmp, journalFile);
    fileSystem.delete(journalFileBackup);
    journalWriter = newJournalWriter();
    hasJournalErrors = false;
    mostRecentRebuildFailed = false;
}

//DiskLruCache.processJournal方法
//只保留CLEAN持久性数据，删除DIRTY编辑的数据
private void processJournal() throws IOException {
    //删除journalFileTmp文件
    fileSystem.delete(journalFileTmp);
    for (Iterator<Entry> i = lruEntries.values().iterator(); i.hasNext(); ) {
        Entry entry = i.next();
        if (entry.currentEditor == null) {
            //表明数据是CLEAN,循环记录SIZE
            for (int t = 0; t < valueCount; t++) {
                size += entry.lengths[t];
            }
        } else {
            //表明数据是DIRTY，删除
            entry.currentEditor = null;
            for (int t = 0; t < valueCount; t++) {
                fileSystem.delete(entry.cleanFiles[t]);
                fileSystem.delete(entry.dirtyFiles[t]);
            }
            //移除Entry
            i.remove();
        }
    }
}

//DiskLruCache.journalRebuildRequired方法
//判断是否需要清理缓存
boolean journalRebuildRequired() {
    final int redundantOpCompactThreshold = 2000;
    //清理的条件：当前redundantOpCount大于2000，且redundantOpCount大于linkedList的size
    return redundantOpCount >= redundantOpCompactThreshold && redundantOpCount >= lruEntries.size();
}

//DiskLruCache.cleanupRunnable线程
//1.如果还没初始化或缓存关闭，则不清理
//2.执行清理操作
//3.如果清理完判断后还需清理，只能重新构建日志文件，且日志记录器记0
private final Runnable cleanupRunnable = new Runnable() {
    public void run() {
        synchronized (DiskLruCache.this) {
            //如果没有初始化或已关闭，则不需要清理，注意|和||的区别，|会两个条件都检查
            if (!initialized | closed) {
                return;
            }
            try {
                //清理
                trimToSize();
            } catch (IOException ignored) {
                mostRecentTrimFailed = true;
            }
            try {
                if (journalRebuildRequired()) {
                    //如果还要清理，重新构建
                    rebuildJournal();
                    //计数器置0
                    redundantOpCount = 0;
                }
            } catch (IOException e) {
                //如果抛异常了，设置最近的一次构建失败
                mostRecentRebuildFailed = true;
                journalWriter = Okio.buffer(Okio.blackhole());
            }
        }
    }
};

//DiskLruCache.trimToSize方法
void trimToSize() throws IOException {
    //遍历直到满足大小，maxSize可设置
    while (size > maxSize) {
        Entry toEvict = lruEntries.values().iterator().next();
        removeEntry(toEvict);
    }
    mostRecentTrimFailed = false;
}

//DiskLruCache.removeEntry方法
//1.停止编辑操作
//2.清除用于保存的cleanFiles
//3.增加一条清除日志记录，计数器+1
//4.移除对应key的entry
//5.由于增加一条日志，判断是否需清理，不然可能会越清越多
boolean removeEntry(Entry entry) throws IOException {
    if (entry.currentEditor != null) {
        //结束editor
        entry.currentEditor.detach(); //阻止编辑正常完成
    }
    for (int i = 0; i < valueCount; i++) {
        //清除用于保存文件的cleanFiles
        fileSystem.delete(entry.cleanFiles[i]);
        size -= entry.lengths[i];
        entry.lengths[i] = 0;
    }
    //计数器加1
    redundantOpCount++;
    //增加一条删除日志
    journalWriter.writeUtf8(REMOVE).writeByte(' ').writeUtf8(entry.key).writeByte('\n');
    //移除entry
    lruEntries.remove(entry.key);
    //如果需要重新清理一下，边界情况
    if (journalRebuildRequired()) {
        //清理
        executor.execute(cleanupRunnable);
    }
    return true;
}

//DiskLruCache.Snapshot类
public final class Snapshot implements Closeable {
    private final String key;
    private final long sequenceNumber;
    private final Source[] sources;
    private final long[] lengths;
    Snapshot(String key, long sequenceNumber, Source[] sources, long[] lengths) {
        this.key = key;
        this.sequenceNumber = sequenceNumber;
        this.sources = sources;
        this.lengths = lengths;
    }
    public Source getSource(int index) {
        return sources[index];
    }
}
```

#### 5、ConnectInterceptor 拦截器

- 为当前请求找到合适连接，可能复用已有连接或重新创建连接，返回的连接由连接池负责

#### 6、CallServerInterceptor 拦截器

- 向服务器发起真正请求，接收响应时返回

