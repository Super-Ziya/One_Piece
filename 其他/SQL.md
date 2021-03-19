### SQL

> Structured Query Language：结构化查询语言，管理数据库管理系统
>
> 一个数据库通常包含一个或多个表，每个表有一个名字标识，表包含带有数据的记录（行）
>
> SQL 对大小写不敏感，用单引号 '' 环绕文本值，命令结束要加分号 ;

#### 1、数据库表

- `use RUNOOB`; ：选择数据库

- `set names utf8;` ：设置使用的字符集

- `SELECT * FROM Websites;` ：读取数据表的信息，* 表示选中所有记录

- 基本操作：

  - `SELECT` ：从数据库中提取数据

    - 语法：`SELECT column_name,column_name FORM table_name;` ：选择部分列

    - 语法：`SELECT * FORM table_name;` ：选择全部列

    - 返回不同的值：`DISTINCT` 

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

https://www.runoob.com/sql/sql-join.html