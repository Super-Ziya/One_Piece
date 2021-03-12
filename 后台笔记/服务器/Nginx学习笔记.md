##Nginx学习笔记

### 一、Nginx安装

#### 1、环境

- Linux内核版本要2.6以上，只有2.6之后才支持epool ，在此之前使用 select 或 pool 多路复用的 IO 模型，无法解决高并发压力的问题。通过命令 `uname -a` 可查看版本
- Nginx需要GCC编译器，用来编译C语言环境，Nginx不会提供二进制可执行程序，只能下载源码进行编译
- PCRE 库专门用来做正则表达式，Nginx 里面很多地方都用到了正则表达式
- zlib 库用于对 HTTP 包的内容做 gzip 格式的压缩，如果在 nginx.conf 里配置了 `gzip on` ，并指定对某些类型（content-type）的 HTTP 响应使用 gzip 来进行压缩以减少网络传输量
- OpenSSL 开发库，若服务器不仅支持 HTTP，还需在更安全的 SSL 协议上传输 HTTP ，需要 OpenSSL。另外，使用 MD5、SHA1 等散列函数也需要安装它

#### 2、解压安装

> 统一安装：

 `yum -y install make zlib zlib-devel gcc-c++ libtool openssl openssl-devel pcre pcre-devel`

> 在对应目录解压：

 `tar -zxvf nginx-1.16.0.tar.gz`

>  进入 nginx 文件夹，执行命令

 `./configure` 、 `make` 、 `make install` 

> 完成后 nginx 运行文件被安装在 /usr/local/nginx 下

> 安装目录有四个文件夹：

- conf 配置目录、html 静态资源目录、logs 日志、sbin 一些执行命令

> 启动方式：

```shell
#默认方式启动：
./sbin/nginx

#指定配置文件启动：
./sbing/nginx -c /tmp/nginx.conf

#指定nginx程序目录启动：
./sbin/nginx -p /usr/local/nginx/
```

> 停止方式：

```shell
#快速停止
./sbin/nginx -s stop

#优雅停止
./sbin/nginx -s quit
```

> 其他命令：

```shell
#热装载配置文件，不用停止可以刷新配置（一定要熟练，这是用的最多的命令）
./sbin/nginx -s reload

#重新打开日志文件（下面单说）
./sbin/nginx -s reopen

#检测当前使用的是哪个配置文件，配置是否正确（这个命令也经常使用）
./sbin/nginx -t
```

> nginx 默认使用的端口是 80
>
> 修改默认端口：默认的配置在安装文件夹下的 conf 文件夹下的 ngixn.conf 文件中，目录为/usr/local/nginx/conf ，修改端口：

![img](https://upload-images.jianshu.io/upload_images/3673891-e0cca8dc43c16d5b.png?imageMogr2/auto-orient/strip|imageView2/2/w/407/format/webp)

启动完后 nginx 有两个进程：master、worker。master 进程主要用来做热装载更新或者日志之类的。worker进程才是真正执行客户端连接的进程，为了提高性能，worker进程可以设置成多个

nginx把日志写入日志文件的时，不是根据文件路径找文件，而是根据日志文件的句柄。句柄中记录了使用哪个日志文件，不会因为文件名改变而改变。要写使用新建的日志文件，必须用 reopen 命令重新打开文件，相当于从新切换句柄中的日志文件引用：

`./sbin/nginx -s reopen` 

### 二、Nginx架构简介

#### 1、架构概览

![img](https://upload-images.jianshu.io/upload_images/3673891-ccbb9bfe5fc290db.png?imageMogr2/auto-orient/strip|imageView2/2/w/547/format/webp)

Nginx 有两种进程，一个master进程，一种是worker进程。nginx启动时，会生成两种类型的进程，一个是主进程（Master），一个（windows版本的目前只有一个）或者多个工作进程（Worker）。

主进程不处理网络请求，负责调度工作进程：加载配置、启动工作进程及非停升级。

服务器实际处理网络请求及响应的是工作进程（worker），在类unix系统上，nginx可以配置多个worker，每个worker进程都可以同时处理数以千计的网络请求。worker进程的数量不是越多越好，实际调优时，一般根据cpu 的数量而定。

worker 能支撑上万甚至上十万的并发的原因是： Nginx 设计时基于非阻塞式的方式，能做到非阻塞是因为它的线程模型基于 Linux 的 epoll/select 模型，类似 java 中 nio 的多路复用选择器模型。

#### 2、模块化设计

nginx 的 worker，包括核心和功能性模块，核心模块负责维持一个运行循环（run-loop），执行网络请求处理的不同阶段的模块功能，如网络读写、存储读写、内容传输、外出过滤，以及将请求发往上游服务器等。

<img src="https://upload-images.jianshu.io/upload_images/3673891-653686c520f571fc.png?imageMogr2/auto-orient/strip|imageView2/2/w/765/format/webp" alt="img" style="zoom: 67%;" />

#### 3、反向代理

> 正向代理：位于国外的某网站通过浏览器没法访问，此时可能会用一个操作 FQ 进行访问，FQ 的方式主要是找到一个可以访问国外网站的代理服务器，将请求发送给代理服务器，代理服务器去访问国外的网站，然后将访问到的数据传递回来
>
> 正向代理最大的特点是客户端非常明确要访问的服务器地址；服务器只清楚请求来自哪个代理服务器，而不清楚来自哪个具体的客户端；正向代理模式屏蔽或者隐藏了真实客户端信息。

> 反向代理：某网站每天同时连接的访问人数爆表，单个服务器远远不能满足需求。分布式部署：通过部署多台服务器来解决访问人数限制的问题

### 三、Nginx常用配置

> 进入 nginx 安装目录的 conf 文件夹下，备份一下默认的配置文件，防止修改错恢复不了

 `cp nginx.conf nginx.conf.bak`

> Nginx 也是分为一个个配置块，用大括号括起来。配置块的名称（events）和左大括号直接要有空格。

> 配置块里面的以分号结尾的一行（如 worker_connections 1024;）就是该配置块的一个属性。

> 各部分含义：

- worker_processes 表示 worker 进程的进程数

- events 表示对事件的配置

  - events 中 worker_connections 是最大链接数

- http 对http请求的配置

  - http 中 include 表示引入的意思，示例中表示nginx的配置文件引入了mime.types文件
  - default_type 表示请求响应的默认数据类型
  - sendfile 值为 on 表示启用sendfile，其作用是在底层拷贝数据时可跳过应用，直接从内核拷贝到网卡，加快速度
  - keepalive_timeout 表示长连接的一个保活时间，一个连接请求完并不是立马销毁，通过这个属性可以等一段时间，以便下次有请求继续用
  - server 配置块代表配置一个虚拟机，用来启动并响应
    - server 中 listen 表示监听的端口，不能和其它重复
    - server 中 server_name 用来配置域名，分发请求时会根据访问的域名和配置的域名的对应关系分发
    - server 中 ocation 表示其中一个请求的地址，后面可加斜杠、等于号等正则表达式方式匹配，其中的内容是对应的链接和页面，root表示页面所载目录

### 四、Nginx 静态缓存

#### 1、缓存配置

> 提高QPS（每秒查询次数），有两种方法：

-  采用Ajax 动态加载 价格、广告、库存等服务
- 采用key value 缓存详情页主体html

ajax异步可以大大提高页面的性能，但到一定程度会有瓶颈，可以采用类似redis缓存的方式，redis本身支持高并发，把99% 的信息缓存到redis中，还可以进一步提高性能。架构图如下：

![img](https://upload-images.jianshu.io/upload_images/3673891-db5904b3b749437b.png?imageMogr2/auto-orient/strip|imageView2/2/w/454/format/webp)

主题传输数据量可以达到几百KB，在500QPS的程度上接近千兆网络的极限，内网通信的数据大小也是关键。可以调整缓存的位置，不要放在后台redis服务上，而是放在Nginx上面。在一定程度上再次提高并发性能。

![img](https://upload-images.jianshu.io/upload_images/3673891-799abc0794222130.png?imageMogr2/auto-orient/strip|imageView2/2/w/547/format/webp)

nginx访问本地静态缓存文件，本地没有时，再访问 redis 或数据库，再一步提高并发。同样，后台修改了数据，可以去 NGINX 清除指定的缓存。

> nginx 缓存配置

- 在http元素下添加缓存区声明
  - \#proxy_cache_path 缓存路径
  - \#levels 缓存层级及目录位数
  - \#keys_zone 缓存区内存大小
  - \#inactive 有效期
  - \#max_size 硬盘大小

- 为指定 location 设定缓存策略。
  - \# 指定缓存区
  - proxy_cache cache_name;
  - \#以全路径md5值做做为Key
  - proxy_cache_key \$host\$uri\$is_args\$args;
  - \#对不同的HTTP状态码设置不同的缓存时间
  - proxy_cache_valid 200 304 12h;

> 在http中配置一个缓存区块

- 首先创建一个缓存目录：

 `mkdir  -p  /cache/nginx` 

- 然后配置nginx，位置在http下：
   `proxy_cache_path /cache/nginx levels=1:2 keys_zone=cache_nginx:500m inactive=20d max_size=1g;` 
  - 配置的参数名字是proxy_cache_path
  - 缓存的路径是/cache/nginx
  - levels表示缓存级别，1:2表示有两个级别，第一个级别存储一个字母，第二个级别存储两个字母，如果配置成1:2:2，那么就是三个级别，第一级是一个字母，第二级和第三级都是两个字母，级数越多，目录会越多，就会分撒一些，这样缓存不会集中在一个目录下面
  - keys_zone设置缓存区名字和大小，cache_nginx是名字，500m自然就是500兆，这样的话热点缓存会放在内存区块里面

  - inactive表示内存中缓存的有效期，20d表示20天
  - max_size表示缓存在硬盘中的最大存储

> 配置缓存的使用，在location下面配置

![img](https:////upload-images.jianshu.io/upload_images/3673891-1cdbd03a421b88b7.png?imageMogr2/auto-orient/strip|imageView2/2/w/367/format/webp)

- proxy_cache 表示引用上面缓存的配置
- proxy_cache_key 表示缓存的key，在缓存时会根据这个名称，进行一个md5，然后存储到目录中去，其中\$host表示域名，\$uri表示除了问号之外的域名信息，\$is_args就是问号，\$args就是参数。
- proxy_cache_valid 表示哪些情况会进行缓存，会对有意义的页面进行缓存，类似404,500之类的就不用了，缓存时间定义为12小

#### 2、缓存更新

数据更新时，需要清除nginx的缓存，该功能可采用第三方模块 ngx_cache_purge 实现。

> 下载模块：

 `wget http://labs.frickle.com/files/ngx_cache_purge-2.3.tar.gz` 

> 解压：

 `tar -zxvf ngx_cache_purge-2.3.tar.gz` 

> 增加一个第三方模块，需要基于参数的形式重新安装，在命令中加一个参数，路径根据解压路径调整

 `./configure --prefix=/usr/local/nginx --with-http_stub_status_module --with-http_ssl_module --with-debug --add-module=/packages/ngx_cache_purge-2.3` 

> 查看已安装模块

 `./sbin/nginx -V` 

> 配置清除缓存内容

![img](https://upload-images.jianshu.io/upload_images/3673891-306aa9a4deccca6a.png?imageMogr2/auto-orient/strip|imageView2/2/w/636/format/webp)

- 重新配置一个 location 路径为 clear 开头，deny all 表示拒绝所有访问，因为清除缓存肯定需要特殊权限
- allow表示允许特定的 ip 访问，只有这个 ip 有权限清除缓存
- proxy_cache_purge 表示配置要清除的缓存，后面跟缓存块名字和路径，uri需要换成1，因为清除缓存的链接本身需要剔除

### 五、Nginx性能参数调优

Nginx是一个多进程单线程的应用。

#### 1、调优参数一：worker_processes

 **worker_processes 表示 worker线程的数量，**每个worker进程都是单线程的进程，它们会调用各个模块实现多种多样的功能。如果这些模块确认不会出现阻塞式的调用，那么有多少CPU内核就应该配置多少个进程；反之需要配置多一些的 worker 进程。

#### 2、调优参数二：worker_connections

worker_connections 表示每个 worker 进程的最大连接数，默认1024个，使用worker_processes 和worker_connections 可以实际设置 nginx 的总体最大连接数，比如worker_processes设置为2，worker_connections设置为1024，那么nginx的总体最大连接数就是2048个。但是这些连接数，不仅包含了nginx和客户端的链接，还包含了nginx和被代理的服务端的链接，因此不代表nginx的并发能力是2048，具体设置多少，要根据压测结果确定。不过实际情况中，nginx往往不会成为性能的瓶颈，瓶颈一般都在被代理的服务上（如tomcat）。

#### 3、调优参数三：worker_cpu_affinity

**worker_cpu_affinity 表示绑定Nginx的**worker进程到指定的CPU内核，假定每一个worker进程都非常繁忙，如果多个worker进程都在抢同一个CPU，就会出现同步问题。反之每一个worker进程都独享一个CPU，就在内核的调度策略上实现了完全的并发。

#### 4、调优参数四：worker_priority

worker_priority 表示Nginx worker进程优先级设置，默认0，优先级由静态优先级和内核根据进程执行情况所做的动态调整（目前只有±5的调整）共同决定。nice值是进程的静态优先级，它的取值范围是–20～+19，–20是最高优先级，+19是最低优先级。如果希望Nginx占有更多的系统资源，那么可以把nice值配置小一些，但不建议比内核进程的nice值（通常为–5）还要小。

#### 5、**调优参数五：worker_rlimit_nofile**

worker_rlimit_nofile 表示Nginx worker进程可以打开的最大句柄描述符个数，更改worker进程的最大打开文件数限制。如果没设置的话，这个值为操作系统的限制。设置后你的操作系统和Nginx可以处理比 “ulimit -a” 更多的文件，把这个值设高nginx就不会有 “too many open files” 问题。

#### 6、调优参数六：accept_mutex

accept_mutex 表示是否打开accept锁，accept_mutex 是Nginx的负载均衡锁，当某一个worker进程建立的连接数量达到 worker_connections 配置的最大连接数的 7/8 时，会大大减小该 worker 进程试图建立新 TCP 连接的机会，accept锁默认是打开的，如果关闭，建立TCP连接的耗时会更短，但worker进程之间的负载会非常不均衡，因此不建议关闭它。

#### 7、**调优参数七：accept_mutex_delay**

accept_mutex_delay 表示使用 accept 锁后到真正建立连接之间的延迟时间，默认500ms，在使用 accept 锁后，同一时间只有一个 worker 进程能够取到 accept 锁。这个 accept 锁不是堵塞锁，如果取不到会立刻返回。如果只有一个 worker 进程试图取锁而没有取到，他至少要等待 accept_mutex_delay 定义的时间才能再次试图取锁。