#### 服务器笔记

> - 本地域名：localhost
> - Tomcat端口号：8088

#####1、MVC架构

> M：数据层
>
> V（view）：视图层
>
> C（controller）：控制层，一般分为Controller和Server两层

> - mybatis包：建立和数据库的连接，session（会话\连接）
> - junit包：测试用的
> - mysql包：管理数据库
> - xampp：数据库

> - Cookie：浏览器端的一块存储区域，存储容量小
> - Session：服务器端的一块存储区域，存储容量受内存大小限制，提供用户唯一标识SessionID，Cookie和Session缺一不可

> - 前端分发器 `"/*"`：根据请求地址，分发用户请求
> - Java反射机制：根据类的信息，动态创建对象，并调用相关方法
> - Class对象：记录类的信息，例如类的名称，方法，属性

#####2、Java serve项目

>WebContent：部署
>
>Java Resources/src：java程序
>
>`doget` get请求方法

##### 3、Filter过滤器

#####4、Maven Project

> mapper：映射
>
> Java Resources
>
> > src/main/java
> >
> > > controller（控制层）、mapper、po、service（服务层）、util（配置层）、web
> >
> > src/mian/resources
> >
> > > mapper
>
> target
>
> > pom.xml

##### 5、数据库

> 数据库：管理数据
>
> Mysql：数据库管理工具（记得加 `;`）
>
> > `mysql -u账户 -p` 回车后输入密码登录
> >
> > `create database 数据库名称 default charset utf8;` 以utf-8格式创建数据库
> >
> > `show databases;` 展示所有数据库
> >
> > `show create database 数据库名称;` 查看某一数据库信息
> >
> > `show tables;` 展示数据表名称
> >
> > `desc 表名;` 展示表结构
> >
> > `use 数据库名称;` 选择数据库
> >
> > 创建数据表：
> >
> > ```mysql
> > creat table 表名 /* 一般以tb_开头 */(
> > 
> > 	名称 类型 primary key /* 主键，非空且为1 */ ;
> > 
> > )DEFAULT CHARSET=utf8;
> > 
> > ```
> >
> > 插入数据：
> >
> > ```mysql
> > insert into 表名(
> > 	列名，列名，列名
> > ) values (
> > 	一一对应值
> > )
> > ```
> >
> > 
> >
> > `select 列名 form 表名` 获取数据表中某一数据
> >
> > `select * from 表名` 获取数据表中所有数据

##### 6、json（jackson包）

- 一种数据格式（字典dict）
- 表示对象：`{name:value}` 
- 表示数组：`[]` 

##### 7、Elispe

- `Shift+Alt+S` 快速添加 `set` `get` 方法