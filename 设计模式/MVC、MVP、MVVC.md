### MVC、MVP、MVVC

#### 1、MVC

- MVC 组成及与 Android 对应
  - 视图（View）：用户界面。主要是 xml 布局文件，自定义 View 或 ViewGroup，负责将用户请求通知 Controller，并根据 model 更新界面
  - 控制器（Controller）：业务逻辑。是数据模型，类似 javabean，封装对数据库、网络等的操作，负责数据处理相关的逻辑，对应 Android 中的 datebase、SharePreference 等
  - 模型（Model）：数据保存。Activity 或 Fragment，接收用户请求并更新 model

- MVC 是一种框架模式，而不是设计模式，（《设计模式》四位作者，被称为四人组）GOF 把 MVC 看做 3 种设计模式：观察者模式（核心）、策略模式、组合模式
- 框架模式与设计模式的区别
  - 框架模式是对代码的重用；设计模式是对设计的重用
  - 框架面向于一系列相同行为代码的重用，设计面向一系列相同结构代码的重用，架构介于框架和设计之间

- 各部分间通信

  > 所有通信都是单向的

  - View 传送指令到 Controller
  - Controller 完成业务逻辑后，要求 Model 改变状态
  - Model 将新的数据发送到 View，用户得到反馈

> android 规定一个 Activity 的响应时间是 5 秒，超过可能被回收
>
> 在 Android UI 系统中，Activity 主要作用是解耦，将视图 View 和模型 Model 分离

#### 2、MVP

- MVP 组成及与 Android 对应

  - Presenter（交互中间人）：从 Model 层检索数据后返回给 View 层，使 View 和Model 解耦
  - View（用户界面）：通常指 Activity 、Fragment 或某个 View 控件，含有一个 Presenter 成员变量。通常 View 需实现一个逻辑接口，将 View 上的操作转交给 Presenter 实现，最后 Presenter 调用 View 逻辑接口将结果返回给 View 元素
  - Model（数据的存取）：Model 是封装了数据库 DAO（Data Access Object）或网络获取数据的角色，或两种数据获取方式的集合

- MVP 三者间是松耦合，Presenter 持有的 View、Model 引用都是抽象，当 UI 变化时，只需替换 View 即可；数据库引擎需替换时，只需重新构建一个实现 ArticleModel 接口的类实现相关的存取逻辑即可。使得 View 、Model 、Presenter 三者可独立变化，可扩展性、灵活性都很高

  - 各部分间通信是双向的
  - View 与 Model 不发生联系，都通过 Presenter 传递
  - View 很薄，不部署任何业务逻辑，称为"被动视图"，没有任何主动性，而 Presenter 非常厚，所有逻辑都部署在那里

  - MVP 模式解除 View 与Model 的耦合，良好可扩展性、可测试性，保证系统的整洁性、灵活性
  - MVP 模式可分离显示层和逻辑层，通过接口进行通信，降低藕合

> MVC 耦合性相对较高， View 可直接访问 Model，3 者间构成回路
>
> MVP 与 MVC 主要区别是 MVP 中 View 不能直接访问 Model，需通过 Presenter 发出请求， View 与 Model 不直接通信

![MVC_MVP结构](图片.assets\MVC_MVP结构.jpg)

#### 3、MVVC

- MVVM 与 MVP 非常相似，唯一区别是 View 和 Model 双向绑定，两者间有一方发生变化会反应到另一方上
- MVP 与 MVVM 主要区别，：
  - MVP 中 View 更新需通过 Presenter，MVVM 不需要，因为 View 与 Model 进行了双向绑定，数据的修改会直接反应到 View 上，View 的修改也会导致数据的变更
  - ViewModel 只负责业务逻辑的处理，以及修改 View 或 Model 的状态。MVVM 像 ListView 与 Adapter、数据集的关系，Adapter 是 ViewModel 角色，与 View 进行绑定，又与数据集进行绑定，当数据集合发生变化时，调用 Adapter 的 notifyDataSetChanged 后 View 直接更新，之间没有直接耦合，使 ListView  更灵活