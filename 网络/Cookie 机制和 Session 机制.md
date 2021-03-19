### Cookie 机制和 Session 机制

> Cookie 通过在客户端记录信息确定用户身份，Session 通过在服务器端记录信息确定用户身份

#### （1）Cookie 机制

> 理论上一个用户的所有请求操作都应属同一个会话
>
> Web 应用程序使用 HTTP 协议传输数据，HTTP 协议是无状态的协议，数据交换完毕客户端与服务器端的连接就关闭，再次交换数据需建立新连接
>
> Cookie 是一小段文本信息。客户端请求服务器，如果服务器需要记录该用户状态，就使用 response 向客户端浏览器颁发一个 Cookie。客户端浏览器把 Cookie 保存起来。当浏览器再请求该网站时，浏览器把请求的网址连同该 Cookie 一同提交给服务器。服务器检查该 Cookie 以辨认用户状态。服务器还可根据需要修改 Cookie 内容
>
> Cookie 需要浏览器的支持，不同浏览器保存 Cookie 不同

- 记录用户访问次数
  - Java 把 Cookie 封装成 javax.servlet.http.Cookie 类。每个 Cookie 都是该类的对象。服务器通过操作 Cookie 类对象对客户端 Cookie 进行操作。通过 `request.getCookie()` 获取客户端提交的所有 Cookie（以 Cookie[] 数组形式返回），通过 `response.addCookie(Cookiecookie)` 向客户端设置 Cookie
  - Cookie 对象使用 key-value 属性对形式保存用户状态，一个 Cookie 对象保存一个属性对，一个 request 或 response 同时使用多个 Cookie
- Cookie 不可跨域名性
  - 浏览器访问 Google 只会携带 Google 的 Cookie，不会携带 Baidu 的，Google 只能操作 Google 的 Cookie，不能操作 Baidu 的
  - Cookie 在客户端由浏览器管理。浏览器保证 Google 只会操作 Google 的 Cookie 而不会操作 Baidu 的，从而保证用户隐私安全。浏览器判断一个网站是否能操作另一个网站 Cookie 的依据是域名
  - 虽然网站 images.google.com 与 www.google.com 同属 Google，但域名不一样，二者同样不能互相操作彼此的 Cookie，但登录网站 www.google.com 发现访问 images.google.com 时登录信息仍有效，普通 Cookie 做不到，而 Google 做了特殊处理
- Fast Open

![Fast_Open](图片.assets\Fast_Open.png)

- Unicode 编码

https://www.cnblogs.com/l199616j/p/11195667.html



