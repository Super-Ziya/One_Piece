### SQL

> Structured Query Language：结构化查询语言，管理数据库管理系统
>
> 一个数据库通常包含一个或多个表，每个表有一个名字标识，表包含带有数据的记录（行）
>
> SQL 对大小写不敏感，用单引号 '' 环绕文本值，命令结束要加分号 ;

#### 1、数据库表

- `use RUNOOB;` ：选择数据库

- `set names utf8;` ：设置使用的字符集

- `SELECT * FROM Websites;` ：读取数据表的信息，* 表示选中所有记录

- 基本操作：

  - `SELECT` ：从数据库中提取数据

    - 语法：`SELECT column_name,column_name FORM table_name;` ：选择部分列

    - 语法：`SELECT * FORM table_name;` ：选择全部列

    - 返回不同值：`DISTINCT` 

      > `SELECT DISTINCT column_name form table_name;` ：返回列中不重复的值

  - `UPDATE` ：更新数据库中的数据

    - 语法：`UPDATE table_name SET column1=value1,column2=value2,... WHERE some_column=some_value;` ：忽略 WHERE 会改动所有数据

  - `DELETE` ：从数据库中删除数据

    - 语法：`DELETE FROM table_name WHERE some_column=some_value;`：忽略 WHERE 将删除所有记录

  - `INSERT INTO` ：向数据库中插入新数据

    - 语法：`INSERT INTO table_name VALUES (value1,value2,...)`：插入所有列
    - 语法：`INSERT INTO table_name (column1,column2) VALUES (value1,value2,...)`：插入指定列

  - `CREATE DATABASE` ：创建新数据库

  - `ALTER DATABASE` ：修改数据库

  - `CREATE TABLE` ：创建新表

  - `ALTER TABLE` ：变更（改变）数据库表

  - `DROP TABLE` ：删除表

  - `CREATE INDEX` ：创建索引（搜索键）

  - `DROP INDEX` ：删除索引

#### 2、WHERE 子句

> 用于过滤，提取满足指定条件的记录
>
> 语法：`SELECT column_name,colum_name FROM table_name WHERE column_name operator value;`
>
> 如：`SELECT * FROM table1 WHERE id = 1;`

- 可用运算符

| 运算符  | 描述                                 |
| :------ | :----------------------------------- |
| =       | 等于                                 |
| <>      | 不等于，在 SQL 一些版本中可被写成 != |
| >       | 大于                                 |
| <       | 小于                                 |
| >=      | 大于等于                             |
| <=      | 小于等于                             |
| BETWEEN | 在某个范围内                         |
| LIKE    | 搜索某种模式                         |
| IN      | 指定针对某个列的多个可能值           |

- AND & OR 运算符，如：

  - `SELECT * FROM table1 WHERE country = 'CN' AND alexa > 50;` 
  - `SELECT *FROM table1 WHERE alexa > 50 AND (country = 'USE' OR country = 'CN');` 

- ORDER BY 关键字

  > 对结果集进行排序，ASC 升序，DESC 降序
  >
  > 语法：`SELECT column_name FROM table_name ORDER BY column_name ASC|DESC;` ：按column_name 排序

- like 操作符：搜索列中的指定模式

  - 语法：`SELECT column_name(s) FROM table_name WHERE column_name LIKE pattern;`
  - 如：`SELECT * FROM table1 WHERE name LIKE 'G%';`
    - %：通配符，G% 以 G 开头，%G 以 G 结尾，%G% 包含 G
    - NOT LIKE：不包含模式

- 通配符

  - 如：`SELECT * FROM table1 WHERE name LIKE '_O';`：以某一个字符开头，然后是“O”
  - 如：`SELECT * FROM table1 WHERE name REGEXP '^[DFs]';`：以  D、F、s 开头
    - REGEXP 和 NOT REGEXP（或 LIKE、NOT LIKE）：操作正则表达式
  - 如：`SELECT * FROM table1 WHERE name REGEXP '^[A-H]';`：以 A 到 H 开头
  - 如：`SELECT * FROM table1 WHERE name REGEXP '^[^A-H]';`：以不是 A 到 H 开头

| 通配符                         | 描述                       |
| :----------------------------- | :------------------------- |
| %                              | 替代 0 个或多个字符        |
| _                              | 替代一个字符               |
| [*charlist*]                   | 字符列中的任何单一字符     |
| [^*charlist*] 或 [!*charlist*] | 不在字符列中的任何单一字符 |

- IN 操作符：规定多个值
  - 语法：`SELECT column_name(s) FROM table_name WHERE column_name IN (value1,value2,...)` 
- BETWEEN 操作符：选取介于两个值之间的数据范围内的值（数值、文本、日期）
  - 语法：`SELECT column_name(s) FROM table_name WHERE column_name BETWEEN value1 AND value2;`
  - NOT BETWEEN：不在范围内

#### 3、SELECT TOP，LIMIT，ROWNUM 子句

- SELECT TOP：规定返回记录的数目
  - 语法：`SELECT TOP number|percent column_name(s) FROM table_name;` 
  - MySQL 语法：`SELECT column_name(s) FROM table_name LIMIT number;`
  - Oracle 语法：`SELECT column_name(s) FROM table_name WHERE ROWNUM <= number;` 

#### 4、别名

> 查询中涉及超过一个表
>
> 查询中使用函数
>
> 列名很长或可读性差
>
> 需把两列或多列结合在一起

- 列的语法：`SELECT column_name AS alias_name FROM table_name;`
- 表的语法：`SELECT column_name(s) FROM table_name AS alias_name;` 
- 若列名包含空格，要加双引号 "" 或方括号 [] 
- 如：`SELECT name, CONCAT(url, ', ', alexa, ', ', country) AS site_info
  FROM table1;`：把三个列结合起来创建别名

#### 5、JOIN（连接）

> SQL JOIN 子句用于把来自两个或多个表的行结合起来，基于这些表间的共同字段

- 不同的 SQL JOIN 类型：
  - INNER JOIN：如果表中有至少一个匹配，返回行
  - LEFT JOIN：即使右表中没有匹配，从左表返回所有行
  - RIGHT JOIN：即使左表中没有匹配，从右表返回所有行
  - FULL JOIN：只要其中一个表中存在匹配，返回行

<img src="https://www.runoob.com/wp-content/uploads/2019/01/sql-join.png" alt="img" style="zoom: 67%;" />

- 例：Websites 表中的 id 列指向 access_log 表中字段 site_id，两个表通过 site_id 列联系

  - Websites 表

  |  id  | name          | url                       | alexa | country |
  | :--: | :------------ | ------------------------- | :---: | :-----: |
  |  1   | Google        | https://www.google.cm/    |   1   |   USA   |
  |  2   | 淘宝          | https://www.taobao.com/   |  13   |   CN    |
  |  3   | 菜鸟教程      | http://www.runoob.com/    | 4689  |   CN    |
  |  4   | 微博          | http://weibo.com/         |  20   |   CN    |
  |  5   | Facebook      | https://www.facebook.com/ |   3   |   USA   |
  |  7   | stackoverflow | http://stackoverflow.com/ |   0   |   IND   |

  - access_log 表

  | aid  | site_id | count |    date    |
  | :--: | :-----: | :---: | :--------: |
  |  1   |    1    |  45   | 2016-05-10 |
  |  2   |    3    |  100  | 2016-05-13 |
  |  3   |    1    |  230  | 2016-05-14 |
  |  4   |    2    |  10   | 2016-05-14 |
  |  5   |    5    |  205  | 2016-05-14 |
  |  6   |    4    |  13   | 2016-05-15 |
  |  7   |    3    |  220  | 2016-05-15 |
  |  8   |    5    |  545  | 2016-05-16 |
  |  9   |    3    |  201  | 2016-05-17 |

  - SQL

  ```sql
  SELECT Websites.id, Websites.name, access_log.count, access_log.date
  FROM Websites
  INNER JOIN access_log
  ON Websites.id=access_log.site_id;
  ```

  - 输出

  |  id  | name     | count |    date    |
  | :--: | -------- | :---: | :--------: |
  |  1   | Google   |  45   | 2016-05-10 |
  |  1   | Google   |  230  | 2016-05-14 |
  |  2   | 淘宝     |  10   | 2016-05-14 |
  |  3   | 菜鸟教程 |  100  | 2016-05-13 |
  |  3   | 菜鸟教程 |  220  | 2016-05-15 |
  |  3   | 菜鸟教程 |  201  | 2016-05-17 |
  |  4   | 微博     |  13   | 2016-05-15 |
  |  5   | Facebook |  205  | 2016-05-14 |
  |  5   | Facebook |  545  | 2016-05-16 |

#### 6、UNION 操作符

> 合并两个或多个 SELECT 语句结果集
>
> UNION 内部每个 SELECT 语句必须拥有相同数量的列，列也必须拥有相似的数据类型，每个 SELECT 语句中列的顺序必须相同
>
> UNION 结果集中列名总等于 UNION 中第一个 SELECT 语句中的列名

- UNION 语法：默认选取不同的值

  ````sql
  SELECT column_name(s) FROM table1
  UNION
  SELECT column_name(s) FROM table2;
  ````

- UNION ALL 语法：允许重复的值

  ```sql
  SELECT column_name(s) FROM table1
  UNION ALL
  SELECT column_name(s) FROM table2;
  ```

- 例

  - Websites 表

  |  id  | name          | url                       | alexa | country |
  | :--: | :------------ | ------------------------- | :---: | :-----: |
  |  1   | Google        | https://www.google.cm/    |   1   |   USA   |
  |  2   | 淘宝          | https://www.taobao.com/   |  13   |   CN    |
  |  3   | 菜鸟教程      | http://www.runoob.com/    | 4689  |   CN    |
  |  4   | 微博          | http://weibo.com/         |  20   |   CN    |
  |  5   | Facebook      | https://www.facebook.com/ |   3   |   USA   |
  |  7   | stackoverflow | http://stackoverflow.com/ |   0   |   IND   |

  - apps 表

  |  id  | app_name | url                     | country |
  | :--: | -------- | :---------------------- | :-----: |
  |  1   | QQ APP   | http://im.qq.com/       |   CN    |
  |  2   | 微博 APP | http://weibo.com/       |   CN    |
  |  3   | 淘宝 APP | https://www.taobao.com/ |   CN    |

  - UNION

    - SQL

    ```sql
    SELECT country FROM Websites
    UNION
    SELECT country FROM apps
    ORDER BY country;
    ```

    - 输出

    | country |
    | :-----: |
    |   CN    |
    |   IND   |
    |   USA   |

  - UNION ALL

    - SQL

    ```sql
    SELECT country FROM Websites
    UNION ALL
    SELECT country FROM apps
    ORDER BY country;
    ```

    - 输出

    | counrty |
    | :-----: |
    |   CN    |
    |   CN    |
    |   CN    |
    |   CN    |
    |   CN    |
    |   IND   |
    |   USA   |
    |   USA   |
    |   USA   |

  - 带有 WHERE 的 UNION ALL，

    - SQL

    ```sql
    SELECT country, name FROM Websites
    WHERE country='CN'
    UNION ALL
    SELECT country, app_name FROM apps
    WHERE country='CN'
    ORDER BY country;
    ```

    - 输出

    | country |   name   |
    | :-----: | :------: |
    |   CN    |   淘宝   |
    |   CN    |  QQ APP  |
    |   CN    | 菜鸟教程 |
    |   CN    | 微博 APP |
    |   CN    |   微博   |
    |   CN    | 淘宝 APP |

#### 7、SELECT INTO

> 从一个表复制数据，把数据插到另一个新表中
>
> MySQL 不支持 SELECT ... INTO 语句，支持 INSERT INTO ... SELECT

- 语法

  - 复制所有列插入到新表中：

    ```sql
    SELECT *
    INTO newtable [IN externaldb]
    FROM table1;
    ```

  - 只复制希望的列插入到新表中：

    ```sql
    SELECT column_name(s)
    INTO newtable [IN externaldb]
    FROM table1;
    ```

  - 新表将使用 SELECT 语句中定义的列名和类型进行创建，可用 AS 子句应用新名称

    ```sql
    CREATE TABLE 新表
    AS
    SELECT * FROM 旧表 
    ```

#### 8、INSERT INTO SELECT

> 从一个表复制信息到另一个表（已存在），目标表中任何已存在行不会受影响

- 语法
  - 从一个表中复制所有列插到另一个已存在的表中：

    ```sql
    INSERT INTO table2
    SELECT * FROM table1;
    ```

  - 只复制希望列插到另一个已存在的表中：

    ```sql
    INSERT INTO table2
    (column_name(s))
    SELECT column_name(s)
    FROM table1;
    ```

#### 9、CREATE DATABASE

> 创建数据库

- 语法：`CREATE DATABASE dbname;` 

#### 10、CREATE TABLE

> 创建数据库中的表
>
> 表由行和列组成，每个表都必须有表名

- 语法

  - column_name：规定表中列名
  - data_type：规定列的数据类型（如 varchar、integer、decimal、date 等）
  - size：规定表中列的最大长度

  ```sql
  CREATE TABLE table_name(
      column_name1 data_type(size),
      column_name2 data_type(size),
      column_name3 data_type(size),
      ....
  );
  ```

#### 11、约束 Constraints

> 规定表中数据规则
>
> 如果存在违反约束的数据行为，会被约束终止
>
> 约束可在创建表时规定（通过 CREATE TABLE），或在表创建后规定（通过 ALTER TABLE）

- 语法

  ```sql
  CREATE TABLE table_name(
      column_name1 data_type(size) constraint_name,
      column_name2 data_type(size) constraint_name,
      column_name3 data_type(size) constraint_name,
      ....
  );
  ```

- 约束

  - NOT NULL：指示某列不能存储 NULL 值，默认接受 NULL 值

    - 添加：

      ```sql
      ALTER TABLE Persons
      MODIFY Age int NOT NULL;
      ```

    - 删除

      ```sql
      ALTER TABLE Persons
      MODIFY Age int NULL;
      ```

  - UNIQUE：保证某列每行必须有唯一的值

    - 添加：CREATE TABLE

      ```sql
      --MySQL
      CREATE TABLE Persons(
          P_Id int NOT NULL,
          LastName varchar(255) NOT NULL,
          FirstName varchar(255),
          Address varchar(255),
          City varchar(255),
          UNIQUE (P_Id)
      )
      --Oracle
      CREATE TABLE Persons(
          P_Id int NOT NULL UNIQUE,
          LastName varchar(255) NOT NULL,
          FirstName varchar(255),
          Address varchar(255),
          City varchar(255)
      )
      --需命名UNIQUE约束，并定义多列UNIQUE约束
      CREATE TABLE Persons(
          P_Id int NOT NULL,
          LastName varchar(255) NOT NULL,
          FirstName varchar(255),
          Address varchar(255),
          City varchar(255),
          CONSTRAINT uc_PersonID UNIQUE (P_Id,LastName)
      )
      ```

    - 添加：ALTER TABLE

      ```sql
      --当表已被创建
      ALTER TABLE Persons
      ADD UNIQUE (P_Id)
      --需命名UNIQUE约束，并定义多列UNIQUE约束
      ALTER TABLE Persons
      ADD CONSTRAINT uc_PersonID UNIQUE (P_Id,LastName)
      ```

    - 撤销

      ```sql
      --MySQL
      ALTER TABLE Persons
      DROP INDEX uc_PersonID
      --Oracle
      ALTER TABLE Persons
      DROP CONSTRAINT uc_PersonID
      ```

  - PRIMARY KEY：NOT NULL 和 UNIQUE 的结合，确保某列（或多列结合）有唯一标识，更容易更快找到表中特定记录，每个表可有多个 UNIQUE 约束，但每个表只能有一个 PRIMARY KEY 约束（主键）

    - 添加：CREATE TABLE

      ```sql
      --MySQL
      CREATE TABLE Persons(
          P_Id int NOT NULL,
          LastName varchar(255) NOT NULL,
          FirstName varchar(255),
          Address varchar(255),
          City varchar(255),
          PRIMARY KEY (P_Id)
      )
      --Oracle
      CREATE TABLE Persons(
          P_Id int NOT NULL PRIMARY KEY,
          LastName varchar(255) NOT NULL,
          FirstName varchar(255),
          Address varchar(255),
          City varchar(255)
      )
      --需命名PRIMARY KEY约束，并定义多列PRIMARY KEY约束
      CREATE TABLE Persons(
          P_Id int NOT NULL,
          LastName varchar(255) NOT NULL,
          FirstName varchar(255),
          Address varchar(255),
          City varchar(255),
          CONSTRAINT pk_PersonID PRIMARY KEY (P_Id,LastName)
      )
      ```

    - 添加：ALTER TABLE

      ```sql
      --当表已被创建
      ALTER TABLE Persons
      ADD PRIMARY KEY (P_Id)
      --需命名PRIMARY KEY约束，并定义多列PRIMARY KEY约束
      ALTER TABLE Persons
      ADD CONSTRAINT pk_PersonID PRIMARY KEY (P_Id,LastName)
      ```

    - 撤销

      ```sql
      --MySQL
      ALTER TABLE Persons
      DROP PRIMARY KEY
      --Oracle
      ALTER TABLE Persons
      DROP CONSTRAINT pk_PersonID
      ```

    - PRIMARY KEY：NOT NULL 和 UNIQUE

  - FOREIGN KEY：指向另一个表的 UNIQUE KEY（唯一约束键），用于预防破坏表间连接行为，防止非法数据插入外键列，因为它必须是它指向的那个表中的值之一

    - 如

      > Person 表

      | P_Id | LastName  | FirstName |   Address    |   City    |
      | :--: | :-------: | :-------: | :----------: | :-------: |
      |  1   |  Hansen   |    Ola    | Timoteivn 10 |  Sandnes  |
      |  2   | Svendson  |   Tove    |  Borgvn 23   |  Sandnes  |
      |  3   | Pettersen |   Kari    |  Storgt 20   | Stavanger |

      > Orders 表

      | O_Id | OrderNo | P_Id |
      | :--: | :-----: | :--: |
      |  1   |  77895  |  3   |
      |  2   |  44678  |  3   |
      |  3   |  22456  |  2   |
      |  4   |  24562  |  1   |

      > Orders 表中的 P_Id 列指向 Persons 表中的 P_Id 列
      >
      > Persons 表中的 P_Id 列是 Persons 表中的 PRIMARY KEY
      >
      > Orders 表中的 P_Id 列是 Orders 表中的 FOREIGN KEY

    - 添加：CREATE TABLE

      ```sql
      --MySQL
      CREATE TABLE Orders(
          O_Id int NOT NULL,
          OrderNo int NOT NULL,
          P_Id int,
          PRIMARY KEY (O_Id),
          FOREIGN KEY (P_Id) REFERENCES Persons(P_Id)
      )
      --Oracle
      CREATE TABLE Orders(
          O_Id int NOT NULL PRIMARY KEY,
          OrderNo int NOT NULL,
          P_Id int FOREIGN KEY REFERENCES Persons(P_Id)
      )
      --需命名FOREIGN KEY约束，并定义多列FOREIGN KEY约束
      CREATE TABLE Orders(
          O_Id int NOT NULL,
          OrderNo int NOT NULL,
          P_Id int,
          PRIMARY KEY (O_Id),
          CONSTRAINT fk_PerOrders FOREIGN KEY (P_Id)
          REFERENCES Persons(P_Id)
      )
      ```

    - 添加：ALTER TABLE

      ```sql
      --当表已被创建
      ALTER TABLE Orders
      ADD FOREIGN KEY (P_Id)
      REFERENCES Persons(P_Id)
      --需命名FOREIGN KEY约束，并定义多列FOREIGN KEY约束
      ALTER TABLE Orders
      ADD CONSTRAINT fk_PerOrders
      FOREIGN KEY (P_Id)
      REFERENCES Persons(P_Id)
      ```

    - 撤销

      ```sql
      --MySQL
      ALTER TABLE Orders
      DROP FOREIGN KEY fk_PerOrders
      --Oracle
      ALTER TABLE Orders
      DROP CONSTRAINT fk_PerOrders
      ```

  - CHECK：限制列中值范围，如果对单个列定义 CHECK 约束，该列只允许特定值；如果对一个表定义 CHECK 约束，会基于行中其他列的值在特定列中对值限制

    - 添加：CREATE TABLE，CHECK 约束 P_Id 列只包含大于 0 的整数

      ```sql
      --MySQL
      CREATE TABLE Persons(
          P_Id int NOT NULL,
          LastName varchar(255) NOT NULL,
          FirstName varchar(255),
          Address varchar(255),
          City varchar(255),
          CHECK (P_Id>0)
      )
      --Oracle
      CREATE TABLE Persons(
          P_Id int NOT NULL CHECK (P_Id>0),
          LastName varchar(255) NOT NULL,
          FirstName varchar(255),
          Address varchar(255),
          City varchar(255)
      )
      --需命名CHECK约束，并定义多列CHECK约束
      CREATE TABLE Persons(
          P_Id int NOT NULL,
          LastName varchar(255) NOT NULL,
          FirstName varchar(255),
          Address varchar(255),
          City varchar(255),
          CONSTRAINT chk_Person CHECK (P_Id>0 AND City='Sandnes')
      )
      ```

    - 添加：ALTER TABLE

      ```sql
      --当表已被创建
      ALTER TABLE Persons
      ADD CHECK (P_Id>0)
      --需命名CHECK约束，并定义多列CHECK约束
      ALTER TABLE Persons
      ADD CONSTRAINT chk_Person CHECK (P_Id>0 AND City='Sandnes')
      ```

    - 撤销

      ```sql
      --MySQL
      ALTER TABLE Persons
      DROP CHECK chk_Person
      --Oracle
      ALTER TABLE Persons
      DROP CONSTRAINT chk_Person
      ```

  - DEFAULT：向列中插入默认值，如无规定其他值，默认值添加到所有新记录

    - 添加：CREATE TABLE

      ```sql
      --MySQL、Oracle
      CREATE TABLE Persons(
          P_Id int NOT NULL,
          LastName varchar(255) NOT NULL,
          FirstName varchar(255),
          Address varchar(255),
          City varchar(255) DEFAULT 'Sandnes'
      )
      --用类似GETDATE()的函数
      CREATE TABLE Orders(
          O_Id int NOT NULL,
          OrderNo int NOT NULL,
          P_Id int,
          OrderDate date DEFAULT GETDATE()
      )
      ```

    - 添加：ALTER TABLE

      ```sql
      --MySQL
      ALTER TABLE Persons
      ALTER City SET DEFAULT 'SANDNES'
      --Oracle
      ALTER TABLE Persons
      MODIFY City DEFAULT 'SANDNES'
      ```

    - 撤销

      ```sql
      --MySQL
      ALTER TABLE Persons
      ALTER City DROP DEFAULT
      --Oracle
      ALTER TABLE Persons
      ALTER COLUMN City DROP DEFAULT
      ```

#### 12、CREATE INDEX

> 在表中创建索引
>
> 不读取整个表的情况下，索引使数据库应用程序可更快查找数据
>
> 用户无法看到索引，只被用来加速搜索/查询
>
> 更新一个包含索引的表要比没有索引的表花费更多时间，由于索引本身也要更新，理想做法是仅在常被搜索的列（表）上创建索引

- 语法

  - CREATE INDEX：创建一个简单索引，允许使用重复值

    ```sql
    CREATE INDEX index_name
    ON table_name (column_name)
    ```

  - CREATE UNIQUE INDEX：创建一个唯一索引，不允许使用重复值

    ```sql
    CREATE UNIQUE INDEX index_name
    ON table_name (column_name)
    ```

#### 13、撤销 DROP

> 撤销索引、表、数据库

- 语法

  - DROP INDEX：删除表中索引

    ```sql
    --MySQL
    ALTER TABLE table_name DROP INDEX index_name
    --Oracle
    DROP INDEX index_name
    ```

  - DROP TABLE：删除表：`DROP TABLE table_name` 

  - DROP DATABASE：删除数据库：`DROP DATABASE database_name` 

  - TRUNCATE TABLE：仅需删除表内数据，并不删除表本身：`TRUNCATE TABLE table_name` 

    请使用 

#### 14、ALTER TABLE

> 在已有表中添加、删除、修改列

- 语法

  - 在表中添加列

    ```sql
    ALTER TABLE table_name
    ADD column_name datatype
    ```

  - 删除表中列（某些数据库系统不允许在数据库表中删除列的方式）

    ```sql
    ALTER TABLE table_name
    DROP COLUMN column_name
    ```

  - 改变表中列的数据类型

    ```sql
    --MySQL
    ALTER TABLE table_name
    MODIFY COLUMN column_name datatype
    --Oracle
    ALTER TABLE table_name
    MODIFY column_name datatype;
    ```

#### 15、AUTO_INCREMENT 

> 在新记录插入表时生成一个唯一数字

- 语法

  - MySQL：把 Persons 表的 ID 列定义为 auto-increment 主键字段，默认开始值是 1，每条新记录递增 1

    ```sql
    CREATE TABLE Persons(
        ID int NOT NULL AUTO_INCREMENT,
        LastName varchar(255) NOT NULL,
        FirstName varchar(255),
        Address varchar(255),
        City varchar(255),
        PRIMARY KEY (ID)
    )
    --让AUTO_INCREMENT序列以其他值起始
    ALTER TABLE Persons AUTO_INCREMENT=100
    --在表中插入新记录，不必为ID列规定值（会自动添加一个唯一值）
    INSERT INTO Persons (FirstName,LastName)
    VALUES ('Lars','Monsen')
    ```

  - Oracle：通过 sequence 对象（生成数字序列）创建 auto-increment 字段，创建 seq_person 的 sequence 对象，以 1 起始，以 1 递增，该对象缓存 10 个值以提高性能，cache 选项规定为提高访问速度存储多少个序列值

    ```sql
    CREATE SEQUENCE seq_person
    MINVALUE 1
    START WITH 1
    INCREMENT BY 1
    CACHE 10
    --在表中插入新记录要使用nextval函数（该函数从seq_person序列中取回下一个值）
    INSERT INTO Persons (ID,FirstName,LastName)
    VALUES (seq_person.nextval,'Lars','Monsen')
    ```

#### 16、视图 Views

> 基于 SQL 语句的结果集的可视化表
>
> 视图包含行列，每个字段是来自一个或多个数据库中真实表中的字段
>
> 可向视图添加 SQL 函数、WHERE、JOIN 语句，可呈现数据，就如其来自于某个单一表
>
> 视图总是显示最新数据，每当用户查询视图时，数据库引擎通过使用视图的 SQL 语句重建数据

- 语法

  - CREATE VIEW：创建视图

  ```sql
  CREATE VIEW view_name AS
  SELECT column_name(s)
  FROM table_name
  WHERE condition
  ```
  - CREATE OR REPLACE VIEW：更新视图

  ```sql
  CREATE OR REPLACE VIEW view_name AS
  SELECT column_name(s)
  FROM table_name
  WHERE condition
  ```

  - DROP VIEW：撤销视图：`DROP VIEW view_name` 

#### 17、日期 Date 函数

- MySQL 日期函数

| 函数          | 描述                                |
| ------------- | ----------------------------------- |
| NOW()         | 返回当前日期和时间                  |
| CURDATE()     | 返回当前日期                        |
| CURTIME()     | 返回当前时间                        |
| DATE()        | 提取日期或日期/时间表达式的日期部分 |
| EXTRACT()     | 返回日期/时间的单独部分             |
| DATE_ADD()    | 向日期添加指定时间间隔              |
| DATE_SUB()    | 从日期减去指定时间间隔              |
| DATEDIFF()    | 返回两个日期间的天数                |
| DATE_FORMAT() | 用不同的格式显示日期/时间           |

- 数据类型（MySQL）：如果没有时间部分，默认时间为 00:00:00
  - DATE 格式：YYYY-MM-DD
  - DATETIME 格式：YYYY-MM-DD HH:MM:SS
  - TIMESTAMP 格式：YYYY-MM-DD HH:MM:SS
  - YEAR 格式：YYYY 或 YY

#### 18、NULL

- NULL 值：遗漏的未知数据，默认表的列可存放 NULL 值
  - NULL 用作未知或不适用的值的占位符
  - 无法比较 NULL 和 0，不等价
- 处理
  - 无法使用比较运算符测试 NULL 值，如 =、<、>
  - 使用 IS NULL 和 IS NOT NULL 操作符

- 如果值是 NULL 返回 0：

  ```sql
  --MySQL：IFNULL()函数
  SELECT ProductName,UnitPrice*(IFNULL(UnitsOnOrder,0))
  FROM Products
  --MySQL：COALESCE()函数
  SELECT ProductName,UnitPrice*(COALESCE(UnitsOnOrder,0))
  FROM Products
  --Oracle：NVL()函数
  SELECT ProductName,UnitPrice*(NVL(UnitsOnOrder,0))
  FROM Products
  ```

#### 19、数据类型

- 数据类型通用名称

| 数据类型          | Access                  | SQLServer                                            | Oracle           | MySQL       | PostgreSQL       |
| :---------------- | :---------------------- | :--------------------------------------------------- | :--------------- | :---------- | :--------------- |
| boolean           | Yes/No                  | Bit                                                  | Byte             | N/A         | Boolean          |
| integer           | Number (integer)        | Int                                                  | Number           | Int Integer | Int Integer      |
| float             | Number (single)         | Float Real                                           | Number           | Float       | Numeric          |
| currency          | Currency                | Money                                                | N/A              | N/A         | Money            |
| string (fixed)    | N/A                     | Char                                                 | Char             | Char        | Char             |
| string (variable) | Text (<256) Memo (65k+) | Varchar                                              | Varchar Varchar2 | Varchar     | Varchar          |
| binary object     | OLE Object Memo         | Binary (fixed up to 8K) Varbinary (<8K) Image (<2GB) | Long Raw         | Blob Text   | Binary Varbinary |

- MySQL 数据类型

  - Text（文本）类型

  | 数据类型         | 描述                                                         |
  | :--------------- | :----------------------------------------------------------- |
  | CHAR(size)       | 保存固定长度的字符串（可包含字母、数字及特殊字符），括号中指定字符串长度，最多 255 个字符 |
  | VARCHAR(size)    | 保存可变长度的字符串（可包含字母、数字及特殊字符），括号中指定字符串最大长度，最多 255 个字符，如果大于 255，则被转为 TEXT 类型 |
  | TINYTEXT         | 存放最大长度 255 个字符的字符串                              |
  | TEXT             | 存放最大长度 65,535 个字符的字符串                           |
  | BLOB             | 用于 BLOBs（Binary Large OBjects），存放最多 65,535 字节数据 |
  | MEDIUMTEXT       | 存放最大长度 16,777,215 个字符的字符串                       |
  | MEDIUMBLOB       | 用于 BLOBs（Binary Large OBjects），存放最多 16,777,215 字节数据 |
  | LONGTEXT         | 存放最大长度 4,294,967,295 个字符的字符串                    |
  | LONGBLOB         | 用于 BLOBs (Binary Large OBjects)，存放最多 4,294,967,295 字节数据 |
  | ENUM(x,y,z,etc.) | 允许输入可能值的列表。可在 ENUM 列表中列出最大 65535 个值。如果列表中不存在插入的值，则插入空值，该值按输入顺序排序 |
  | SET              | 与 ENUM 类似，最多只包含 64 个列表项且 SET 可存储一个以上的选择 |

  - Number（数字）类型
    - size 代表的并不是存储在数据库中的具体长度，如 int(4) 并不只能存储 4 个长度的数字，int(size) 占多少存储空间无关，int(3)、int(4)、在磁盘上都占 4 btyes 存储空间，就是在显示给用户的方式有点不同

  | 数据类型        | 描述                                                         |
  | :-------------- | :----------------------------------------------------------- |
  | TINYINT(size)   | 带符号 -128 到 127 ，无符号 0 到 255                         |
  | SMALLINT(size)  | 带符号 -32768 到 32767，无符号 0 到 65535，size 默认 6       |
  | MEDIUMINT(size) | 带符号 -8388608 到 8388607，无符号 0 到 16777215，size 默认 9 |
  | INT(size)       | 带符号 -2147483648 到 2147483647，无符号 0 到 4294967295，size 默认 11 |
  | BIGINT(size)    | 带符号 -9223372036854775808 到 9223372036854775807，无符号 0 到 18446744073709551615，size 默认 20 |
  | FLOAT(size,d)   | 带浮动小数点的小数字，size 参数规定显示最大位数，d 参数规定小数点右侧最大位数 |
  | DOUBLE(size,d)  | 带浮动小数点的大数字，size 参数规定显示最大位数，d 参数规定小数点右侧最大位数 |
  | DECIMAL(size,d) | 字符串存储的 DOUBLE 类型，允许固定小数点，size 参数规定显示最大位数，d 参数规定小数点右侧最大位数 |

  - Date/Time（日期/时间）类型
    - 即便 DATETIME、TIMESTAMP 返回相同格式，其工作方式不同，在 INSERT、UPDATE 查询中，TIMESTAMP 自动把自身设为当前日期时间，其接受不同格式，如 YYYYMMDDHHMMSS、YYMMDDHHMMSS、YYYYMMDD、YYMMDD

  | 数据类型    | 描述                                                         |
  | :---------- | :----------------------------------------------------------- |
  | DATE()      | 日期，格式：YYYY-MM-DD，支持 1000-01-01 到 9999-12-31        |
  | DATETIME()  | 日期和时间组合，格式：YYYY-MM-DD HH:MM:SS，支持 1000-01-01 00:00:00 到 9999-12-31 23:59:59 |
  | TIMESTAMP() | 时间戳，使用 Unix 纪元(1970-01-01 00:00:00 UTC) 至今的秒数存储，格式：YYYY-MM-DD HH:MM:SS，支持 1970-01-01 00:00:01 UTC 到 2038-01-09 03:14:07 UTC |
  | TIME()      | 时间，格式：HH:MM:SS，支持 -838:59:59 到 838:59:59           |
  | YEAR()      | 2 或 4 位格式的年，4 位：1901 到 2155，2 位：70 到 69，表示从 1970 到 2069 |

#### 20、函数

- Aggregate 函数：计算从列中取得的值，返回一个单一值

  - AVG()：返回平均值

    - 语法：`SELECT AVG(column_name) FROM table_name` 

  - COUNT()：返回行数

    - 语法

      ```sql
      --COUNT(column_name)：返回指定列的值的数目（NULL不计入）
      SELECT COUNT(column_name) FROM table_name;
      --COUNT(*)：返回表中记录数
      SELECT COUNT(*) FROM table_name;
      --COUNT(DISTINCT column_name)：返回指定列的不同值数目
      SELECT COUNT(DISTINCT column_name) FROM table_name;
      ```

  - FIRST()：返回第一个记录的值，只有 MS Access 支持

    - 语法

      ```sql
      SELECT FIRST(column_name) FROM table_name;
      --MySQL
      SELECT column_name FROM table_name
      ORDER BY column_name ASC
      LIMIT 1;
      --Oracle
      SELECT column_name FROM table_name
      ORDER BY column_name ASC
      WHERE ROWNUM <=1;
      ```

  - LAST()：返回最后一个记录的值，只有 MS Access 支持

    - 语法

      ```sql
      SELECT LAST(column_name) FROM table_name;
      --MySQL
      SELECT column_name FROM table_name
      ORDER BY column_name DESC
      LIMIT 1;
      --Oracle
      SELECT column_name FROM table_name
      ORDER BY column_name DESC
      WHERE ROWNUM <=1;
      ```

  - MAX()：返回最大值

    - 语法：`SELECT MAX(column_name) FROM table_name;` 

  - MIN()：返回最小值

    - 语法：`SELECT MIN(column_name) FROM table_name;` 

  - SUM：返回总和

    - 语法：`SELECT SUM(column_name) FROM table_name;` 

- Scalar 函数：基于输入值，返回一个单一值

  - UCASE()：将某个字段转换为大写
    - 语法：`SELECT UCASE(column_name) FROM table_name;` 
  - LCASE()：将某个字段转换为小写
    - 语法：`SELECT LCASE(column_name) FROM table_name;` 
  - MID()：从某个文本字段提取字符，MySQL 使用
    - 语法：`SELECT MID(column_name,start[,length]) FROM table_name;` 
    - column_name：必需，要提取字符的字段
    - start：必需，规定开始位置（起始值 1）
    - length：可选，要返回的字符数，省略返回剩余文本
  - SubString(字段，1，end)：从某个文本字段提取字符
  - LEN()：返回某个文本字段的长度
    - 语法：`SELECT LEN(column_name) FROM table_name;` 
    - MySQL 中为 LENGTH()，`SELECT LENGTH(column_name) FROM table_name;` 
  - ROUND()：对某个数值字段进行指定小数位数的四舍五入
    - 语法：`SELECT ROUND(column_name,decimals) FROM table_name;` 
    - column_name：必需，要舍入的字段
    - decimals：必需，规定要返回的小数位数
  - NOW()：返回当前的系统日期和时间
    - 语法：`SELECT NOW() FROM table_name;` 
  - FORMAT()：格式化某个字段的显示方式
    - 语法：`SELECT FORMAT(column_name,format) FROM table_name;`
    -  column_name：必需，要格式化的字段
    - format：必需，规定格式

- GROUP BY：结合聚合函数，根据一或多列对结果集进行分组

  - 语法

    ```sql
    SELECT column_name, aggregate_function(column_name)
    FROM table_name
    WHERE column_name operator value
    GROUP BY column_name;
    ```

  - 例

    - Websites 表

    | id   | name          | url                       | alexa | country |
    | ---- | ------------- | ------------------------- | ----- | ------- |
    | 1    | Google        | https://www.google.cm/    | 1     | USA     |
    | 2    | 淘宝          | https://www.taobao.com/   | 13    | CN      |
    | 3    | 菜鸟教程      | http://www.runoob.com/    | 4689  | CN      |
    | 4    | 微博          | http://weibo.com/         | 20    | CN      |
    | 5    | Facebook      | https://www.facebook.com/ | 3     | USA     |
    | 7    | stackoverflow | http://stackoverflow.com/ | 0     | IND     |

    - access_log 表

    | aid  | site_id | count | date       |
    | ---- | ------- | ----- | ---------- |
    | 1    | 1       | 45    | 2016-05-10 |
    | 2    | 3       | 100   | 2016-05-13 |
    | 3    | 1       | 230   | 2016-05-14 |
    | 4    | 2       | 10    | 2016-05-14 |
    | 5    | 5       | 205   | 2016-05-14 |
    | 6    | 4       | 13    | 2016-05-15 |
    | 7    | 3       | 220   | 2016-05-15 |
    | 8    | 5       | 545   | 2016-05-16 |
    | 9    | 3       | 201   | 2016-05-17 |

    - 统计 access_log 各 site_id 访问量

      ```sql
      SELECT site_id, SUM(access_log.count) AS nums
      FROM access_log GROUP BY site_id;
      ```
      > 输出

      | site_id | nums |
      | ------- | ---- |
      | 1       | 275  |
      | 2       | 10   |
      | 3       | 521  |
      | 4       | 13   |
      | 5       | 750  |

- HAVING：WHERE 无法与聚合函数一起使用，HAVING 可筛选分组后的各组数据

  - 语法

    ```sql
    SELECT column_name, aggregate_function(column_name)
    FROM table_name
    WHERE column_name operator value
    GROUP BY column_name
    HAVING aggregate_function(column_name) operator value;
    ```

  - 例

    - 查找总访问量大于 200 的网站

      ```sql
      SELECT Websites.name, Websites.url, SUM(access_log.count) AS nums FROM (access_log
      INNER JOIN Websites
      ON access_log.site_id=Websites.id)
      GROUP BY Websites.name
      HAVING SUM(access_log.count) > 200;
      ```

      > 输出

      | name     | url                       | nums |
      | -------- | ------------------------- | ---- |
      | FaceBook | https://www.facebook.com/ | 750  |
      | Google   | https://www.google.cm/    | 275  |
      | 菜鸟教程 | http://www.runoob.com/    | 521  |

- EXISTS：判断查询子句是否有记录，如有一或多条记录存在返回 True，否则返回 False
  - 语法

    ```sql
    SELECT column_name(s)
    FROM table_name
    WHERE EXISTS
    (SELECT column_name FROM table_name WHERE condition);
    ```

#### 21、CASE

> 类似JAVA中的IF ELSE语句

- 格式
  - THEN后边的值与ELSE后边的值类型应一致，否则会报错

```sql
CASE WHEN condition THEN result
    [WHEN...THEN...]
    ELSE result
END

--score<60返回不及格，score>=60返回及格，score>=80返回优秀
SELECT
    STUDENT_NAME,
    (CASE WHEN score < 60 THEN '不及格'
     WHEN score >= 60 AND score < 80 THEN '及格'
     WHEN score >= 80 THEN '优秀'
     ELSE '异常' END) AS REMARK
FROM
    TABLE
```

#### 22、ALL

> 将单个值与子查询返回的单列值集进行比较
>
> 以比较运算符开头，例如：`>`，`>=`，`<`，`<=`，`<>`，`=`，后跟子查询

- 查找`column_name`列中的值大于子查询返回的最大值的行

  ```sql
  SELECT 
      *
  FROM
      table_name
  WHERE
      column_name > ALL (subquery);
  ```

  