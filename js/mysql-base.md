## 常用功能

mysql是一款很流行的开源数据库，由于其占用资源少，安装便利而广受欢迎。SQL类的语言都是相近的，基本上学习一种SQL，主流的数据库都可以使用。

**（1）**SQL语言是数据库通用操作语言，所以需要一个SQL解析器，将SQL命令解析为对应的ISAM操作。

**（2）联表（join）**是指数据库的两张表通过"外键"，建立连接关系。你需要对这种操作进行优化。

**（3）事务（transaction）**是指批量进行一系列数据库操作，只要有一步不成功，整个操作都不成功。所以需要有一个"操作日志"，以便失败时对操作进行回滚。

**（4）备份机制**：保存数据库的副本。

**（5）远程操作**：远程登录，操作数据库。

## 创建库表

以下操作均在linux环境下

（1）登录mysql

```
mysql -u mysql -p
Enter password: 在此输入密码
```

（2）查看mysql数据库列表

```
show databases;
```

（3）查询某一数据库信息（包括字符集）

```
show create database XX(数据库名);
```

（4）创建一个名为school的数据库

```
create database school; // 默认字符集是latin

汉字属于utf8，所以一般我们使用如下
create database XX(数据库名) charset = (字符编码，例如utf8);

修改字符集
alter database XX(数据库名) character set utf8;
```

（4）选择school数据库

```
use school;
```

（5）删除scholl数据库

```
mysql>drop database school;
```

（6）查询当前库中所有表

```
show tables;
```

（7）查看表结构

```
describe(或desc) XX(表名);
```



（7）创建表

```
CREATE TABLE 表名称 ( 列名称1 数据类型, 列名称2 数据类型, ....... )

举例：
CREATE TABLE dept   (deptno DECIMAL(2),   dname VARCHAR(14),   loc VARCHAR(13) );
```

（8）删除表

```
DROP TABLE table_name;

例如
root@host# mysql -u root -p
Enter password:*******
mysql> use RUNOOB;
Database changed
mysql> DROP TABLE runoob_tbl
Query OK, 0 rows affected (0.8 sec)
mysql>
```



## 增删改查

对表进行增删改查，操作指令

（1）**增**insert：插入数据

```
INSERT INTO table_name ( field1, field2,...fieldN )   value ( value1, value2,...valueN );
```

（2）**删**delete：删除数据

```
delete from dept where deptno='40'； //使用where限定条件进行删除
```

（3）**改**update：更新（修改）数据

举例：update dept set loc='BeiJing' where deptno='40';修改列deptno为40的所在地为BeiJing

（4）**查**select：查询数据

```
select * from dept;   //查询表中所有内容

select * from dept where deptno='10';   //使用where进行条件查询

select deptno from  dept;   //查询指定类
```

select字段：`*` 查询全部；`name, age`查询指定字段；`sum(age), avg(age), max(age), min(age), count(name)`聚合函数。

where条件：还可以使用比较运算、算数运算、逻辑运算、正则表达式。

模糊查询：Like用来匹配一部分的，% 任何字符出现任何位置区分大小写。

`SELECT * FROM movie WHERE created_date like '%04%';`

![like](/imgs/like.png)

结果处理：distinct去重，limit限定行数、order by结果排序(默认asc升序，des降序c)，group by分组，having设置条件，

关联查询：内联、左联、右联。

嵌套查询：查询里面套查询，一个查询结果是另一个的查询条件。



[搭建简易数据库](http://www.ruanyifeng.com/blog/2014/07/database_implementation.html)

[mysql操作](https://www.runoob.com/mysql/mysql-tutorial.html)