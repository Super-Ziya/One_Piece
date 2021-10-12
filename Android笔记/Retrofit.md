### Retrofit

- 封装OkHttp

- OkHttp总结

  ![image-20210704154839209](C:\Users\13085\AppData\Roaming\Typora\typora-user-images\image-20210704154839209.png)

  - 优点
    - 高内聚：功能独立（只做网络请求）
  - 缺点
    - 用户网络请求的接口配置繁琐，尤其是需要配置复杂请求body，请求头，参数的时候
    - 数据解析过程需要用户手动拿到 responsbody 进行解析，不能复用
    - 无法适配自动进行线程的切换
    - 万一存在嵌套网络请求就会陷入“回调陷阱”

- Retrofit：一个RESTful的HTTP网络请求框架的封装，网络请求本质上是OkHttp 完成，Retrofit 仅负责网络请求接口的封装

  - App应用程序通过 Retrofit请求网络，实际上是使用Retrofit 接口层封装请求参数、Header、Url等信息，之后由OkHttp 完成后续的请求操作。
  - 在服务端返回数据之后，OkHttp 将原始的结果交给Retrofit，Retrofit根据用户的需求对结果进行解析

- 封装的点

  - okhttp创建的是OkhttpClient，retrofit创建的是Retrofit实例
  - 构建蓝色的Requet的方案，retrofit是通过注解来进行的适配
  - 配置Call的过程中, retrofit是利用Adapter适配的Okhttp 的Call，为call的适配提供了多样性
  - 相对okhttp，retrofit会对responseBody进行自动的Gson解析，提供了可复用，易拓展的数据解析方案
  - 相对okhttp, retrofit会自动的完成线程的切换

  ![image-20210704160405775](C:\Users\13085\AppData\Roaming\Typora\typora-user-images\image-20210704160405775.png)

  ![image-20210704160444074](C:\Users\13085\AppData\Roaming\Typora\typora-user-images\image-20210704160444074.png)

- 创建过程

  - 门面模式（外观模式），像manager，唯一访问入口
  - build：构建者设计模式（参数多，存在可选参数）
  - create：动态代理，拦截请求，反射获取注解（用缓存提升性能），生成具体URL

  ![image-20210704161036909](C:\Users\13085\AppData\Roaming\Typora\typora-user-images\image-20210704161036909.png)

  ![image-20210704162305760](C:\Users\13085\AppData\Roaming\Typora\typora-user-images\image-20210704162305760.png)

  - ServiceMethod实例创建

    ![image-20210704165619103](C:\Users\13085\AppData\Roaming\Typora\typora-user-images\image-20210704165619103.png)

    ![image-20210704165736295](C:\Users\13085\AppData\Roaming\Typora\typora-user-images\image-20210704165736295.png)

- RxJavaCallAdapterFactory：解决嵌套问题（回调陷阱）

  - 链式调度
  - 抽象工厂设计模式：ResponseCallAdapter、ResultCallAdapter、SimpleCallAdapter
  - 适配器设计模式

  ![image-20210704172313481](C:\Users\13085\AppData\Roaming\Typora\typora-user-images\image-20210704172313481.png)

- 设计模式

  - Retrofit 实例使用建造者模式通过Builder类构建。
    - 当构造函数的参数大于4个，且存在可选参数的时候既可以使用建造者设计模式
  - Retrofit创建时的callactory，使用工厂方法设计模式，但是似乎并不打算支持其他的工厂
  - 整个retrofit 采用的时外观模式。统一的调用创建网络请求接口实例和网络请求参数配置的方法
  - retrofit里面使用了动态代理来创建网络请求接口实例，这个是retroft对用户使用来说最大的复用,其它的代码都是为了支撑这个动态代理给用户带来便捷性的
  - 使用了策略模式对serviceMethod对象进行网络请求参数配置，即通过解析网络请求接口方法的参数、返回值和注解类型，从Retrofit对象中获取对应的网络的url地址、网络请求执行器、网络请求适配器和数据转换器
  - ExecutorCallbackCall使用装饰者模式来封装callbackExecutor，用于完成线程的切换
  - ExecutorCallbackCall使用静态代理(委托)代理了Call进行网络请求，真正的网络请求由okhttpCall执行，然而okHttpCall不是自己执行，它是okhttp提供call给外界(retrofit)使用的唯一门户，其实这个地方就是门面模式
  - ExecutorCallbackCall的被初始化是在ExecutorCallAdapterFactory里面通过适配器模式被创建的。CalIAdapter采用了适配器模式为创建访问Call接口提供服务。默认不添加Rxjava则使用默认的
    ExecutorCallAdapterFactory 将okhttp3.call转变成为retroift中的call，如果有Rxiava则将okhttp3.call转化为abservable

  ![image-20210704173736464](C:\Users\13085\AppData\Roaming\Typora\typora-user-images\image-20210704173736464.png)