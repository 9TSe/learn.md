---
title: tips
date: 2023-10-22 09:55:13
tags:
categories:
- MySQL
cover: /pic/8.png
---



# 1. MySQL基本使用

---
## 1.1 MySQL的启动和登录

---
### 1.1.1 MySQL的启动
**1. 管理员身份打开cmd**

```sql
net start mysql80 ##启动MySQL
net stop mysql80 ##关闭MySQL
```

---

### 1.1.2 MySQL的客户端连接

**1. 管理员身份打开cmd**
```sql
mysql [-h 127.0.0.1] [-P 3306] -u root -p 
```

`使用这种方法需要配置环境变量,MySQL的bin目录`


**2. 打开mysql命令行客户端,输入密码以打开**

---

## 1.2 数据模型

> 关系型数据库 (RDBMS)
> - 概念 : 建立在关系模型基础上,由多张相互连接的二维表组成的数据库
> - 特点 : 
> 	1. 使用表存储数据,格式同意,便于维护
> 	2. 使用SQL语言操作,标准统一,使用方便

---
# 2. SQL

>SQL通用语法
>	1. SQL语句可以单行或多行书写,`以分号结尾`
>	2. SQL依据可以使用空格/TAB增强可读
>	3. MySQL数据库的SQL语句`不区分大小写`,`关键字建议大写`
>	4. 注释:
>		- 单行注释 : --注释内容 / # 注释内容(MySQL特有)
>		- 多行注释: /\*注释内容*/





| 分类 | 全称 | 说明 |
|------|------|------|
| DDL  | Data Definition Language | 数据定义语言，用来定义数据库对象（数据库，表，字段） |
| DML  | Data Manipulation Language | 数据操作语言，用来对数据库表中的数据进行增删改 |
| DQL  | Data Query Language | 数据查询语言，用来查询数据库中表的记录 |
| DCL  | Data Control Language | 数据控制语言，用来创建数据库用户、控制数据库的访问权限 |

如果您需要其他格式或有其他问题，请随时告诉我。

## 2.1 SQL类型

### 2.1.1 数值类型

| 类型         | 大小     | 有符号范围                | 无符号范围               | 描述                  |
|--------------|----------|---------------------------|--------------------------|-----------------------|
| tinyint      | 1 byte   | (-128，127)              | (0, 255)                 | 小整数值             |
| smallint     | 2 bytes  | (-32768，32767)          | (0, 65535)               | 大整数值             |
| mediumint    | 3 bytes  | (-8388608，8388607)      | (0, 16777215)            | 大整数值             |
| int/integer  | 4 bytes  | (-2147483648，2147483647)| (0, 4294967295)          | 大整数值             |
| bigint       | 8 bytes  | (-2^63，2^63-1)           | (0, 2^64-1)              | 极大整数值          |
| float        | 4 bytes  | (-3.402823466 E+38，3.402823466351 E+38) | (0, 3.402823466 E+38)  | 单精度浮点数值  |
| double       | 8 bytes  | (-1.7976931348623157 E+308，1.7976931348623157 E+308) | (0, 1.7976931348623157 E+308) | 双精度浮点数值 |
| decimal      | 依赖于M和D的值 | 依赖于M和D的值     | 依赖于M和D的值            | 小数值(精确定点数)  |

ps : 
- decimal : 123.45 精度(M)为5,标度(D)为2
- 无符号类型写法: tinyint unsigned

```sql
double(4,1)
```

4代表整体长度,1代表小数点后的位数

---
### 2.1.2 字符串类型


| 类型         | 大小               | 描述                      |
|--------------|--------------------|---------------------------|
| char         | 0-255 bytes        | 定长字符串               |
| varchar      | 0-65535 bytes      | 变长字符串               |
| tinyblob     | 0-255 bytes        | 不超过255个字符的二进制数据 |
| tinytext     | 0-255 bytes        | 短文本字符串               |
| blob         | 0-65,535 bytes     | 二进制形式的长文本数据     |
| text         | 0-65,535 bytes     | 长文本数据               |
| mediumblob   | 0-16,777,215 bytes | 二进制形式的中等长度文本数据 |
| mediumtext   | 0-16,777,215 bytes | 中等长度文本数据           |
| longblob     | 0-4,294,967,295 bytes | 二进制形式的极大文本数据 |
| longtext     | 0-4,294,967,295 bytes | 极大文本数据               |


ps :
- char性能优与varchar
变长是指,在不超过最大容量时,1就是1,2就是2
定长是指,哪怕是1,也要占用最大容量

---
### 2.1.3 日期类型
| 类型          | 大小  | 范围                                       | 格式        |
|---------------|-------|------------------------------------------|-------------|
|date  | 3     | 1000-01-01 至 9999-12-31                | YYYY-MM-DD  |
| time      | 3     | -838:59:59 至 838:59:59                 | HH:MM:SS    |
|year    | 1     | 1901 至 2155                            | YYYY       |
| datetime      | 8     | 1000-01-01 00:00:00 至 9999-12-31 23:59:59 | YYYY-MM-DD HH:MM:SS |
| timestamp     | 4     | 1970-01-01 00:00:01 至 2038-01-19 03:14:07 | YYYY-MM-DD HH:MM:SS |


---
## 2.2 DDL

### 2.2.1 数据库操作

```sql
#查询
show databases; # 查询所有数据库
select database(); #查询当前数据库


#创建
create database [if not exists] 数据库名 [default charset 字符集] [collate 排序规则];

#eg:
creat datebase if not exists emp default charset utf8mb4;


#删除
drop database [if exists] 数据库名;


#使用
use 数据库名
```


`MySQL里实现的utf8最长使用3个字节,utf8mb4 是 utf8 的超集并完全兼容utf8，能够用四个字节存储更多的字符。`

比如:最常见的就算现在手机端常用的表情字符 emoji和一些不常用的汉字，如 “墅” ，这些需要四个字节才能编码出来。

---

### 2.2.2 表操作 - 查询



```sql
show tables; # 查询当前数据库的所有表

desc 表名; # 查询表结构

show create table 表名; # 查询指定表的建表语句
```

---
### 2.2.3 表操作 - 创建表
语法:

```sql
create table 表名(
	字段1 字段1的类型 [comment '注释内容'], #逗号
	...
	字段2 字段2的类型 [comment '注释内容']
	)
```

>eg:创建一个如下表
>![在这里插入图片描述](/img/c.1.png)

```sql
create table tb_user(
	id int comment '编号',
	name varchar(50) comment '名字',
	age int comment '年龄',
	gender char(1) comment '性别',
	)comment '用户表';
```

---


>eg: 根据需求创建表
>员工信息表,要求:
>	1. 编号(纯数字)
>	2. 员工工号(字符串类型,长度不超过10位)
>	3. 员工姓名(字符串类型,长度不超过10位)
>	4. 性别
>	5. 年龄
>	6. 身份证号
>	7. 入职时间(取值年月日即可)

```sql
create table emp(
	id int comment'编号',
	worknum varchar(10),
	name varchar(10),
	gender char(1),
	age tinyint unsigned,
	idcard char(18),
	entrydate date
	)
```


---

### 2.2.4 表操作 - 修改

```sql
#添加字段
alter table 表名 add 字段名 类型(长度) [comment 注释] [约束]; #添加字段

#eg : 为emp表增加一个新的字段 "昵称" 为nickname,类型为varchar(20)
alter table emp add nickname varchar(20) comment '昵称';


#修改字段
alter table 表名 modify 字段名 新数据类型(长度) # 修改数据类型
alter table 表名 change 旧字段名 新字段名 类型(长度) [comment 注释] [约束]; #修改字段名和字段类型

#eg : 将emp表的nickname字段修改为username,类型为varchar(30)
alter table emp change nickname username varchar(30) comment'昵称';


#删除字段
alter table 表名 drop 字段名;

#eg : 将emp表的字段username删除
alter table emp drop username;


#修改表名
alter table 表名 rename to 新表名;

#eg : 将emp表的表名修改为employee
alter table emp rename to employee;


#删除表
drop table [if exists] 表名;
truncate table 表名; ##删除指定表,并重新创建该表
```


---

## 2.3 DML

### 2.3.1 添加数据

```sql
insert into 表名(字段名1,字段名2,...) values(值1,值2,...); #给指定字段添加数据
#eg:
insert into emp(id,worknum,name,gender,age) values(1,'1','9tse','男',19);

insert into 表名 values(值1,值2,...); #给全部字段添加数据
#eg:
insert into emp values(2,'2','sewerperson','男',20);

insert into 表名(字段名1,字段名2,...) values(值1,值2,...),(值1,值2,...)...;
insert into 表名 values(值1,值2,...),(值1,值2,...)...;
#eg:
insert into emp values(1,'1','9tse','男',19),(2,'2','sewerperson','男',20);
```

ps: 
1. 插入数据时,指定的字段顺序需要与值的顺序对应
2. 字符串和日期数据应该包含在引号中
3. 插入数据大小,应在字段的规定范围内



---
### 2.3.2 修改数据

```sql
update 表名 set 字段名1 = 值1,字段名2 = 值2,... [where 条件];

#eg: 修改id为1的数据,将name修改为9tse
update emp set name = '9tse' where id = 1;
# 修改id为1的数据,将name修改为 sewer,gender修改为 女
update emp set name = 'sewer',gender = '女' where id = 1;
# 将所有员工的入职日期修改为2008-01-01
update emp set entrydate = '2008-01-01';
```

ps: 
- 修改语句的条件如果没有,默认修改整张表的所有数据


---
### 2.3.3 删除数据

```sql
delete from 表名 [where 条件];
#eg:删除gender为女的员工
delete from emp where gender = '女';
# 删除所有员工
delete from emp;
```
ps:
1. delete语句的条件如果没有,默认作用于整张表的所有数据
2. delete语句不能删除某一个字段的值(可以使用update)



---

## 2.4 DQL
语法:

```sql
select
	字段列表
from
	表名列表
where
	条件列表
group by
	分组字段列表
having
	分组后条件判断
order by
	排序字段列表
limit
	分页参数
```

- 基本查询
- 条件查询(where)
- 聚合函数(count,max,min,avg,sum)
- 分组查询(group by)
- 排序查询(order by)
- 分页查询(limit)



### 2.4.1 基本查询


```sql
#查询多个字段
select 字段1,字段2,字段3,... from 表名;
select * from 表名; #效率较低

#设置别名
select 字段1[as] 别名1,字段2[as] 别名2 ... from 表名;

#去除重复记录
select distinct 字段列表 from 表名;
```


eg:

```sql
#查询指定字段返回
select name,worknum,age from emp; 

#查询所有字段返回
select id,worknum,name,gender,age;
select * from emp; #尽量不写*,影响效率

#查询所有员工工作地址,起别名
select workaddress as '工作地址' from emp;
select workaddress '工作地址' from emp;

#查询工作员工的上班地址(不要重复)
select distinct workaddress '工作地址' from emp;
```


---

### 2.4.2 条件查询
语法:

```sql
select 字段列表 from 表名 where 条件列表;
```

|比较运算符| 功能 |
|--|--|
|>  | 大于 |
| >= | 大于等于 |
|  <|  小于|
|  <=|  小于等于|
| = | 等于 |
| <> 或 != | 不等于 |
|between...and...  | 在某范围内(含最小,最大) |
|in(...)  | 在in之后的列表中的值,多选一 |
| like 占位符 | 模糊匹配(_匹配单字符,%匹配任意个字符) |
| is null | 是NULL |

| 逻辑运算符 | 功能                |
|------------|---------------------|
| AND 或 &&  | 并且(多个条件同时成立) |
| OR 或 \|\\|    | 或者(多个条件任意一个成立) |
| NOT 或 !   | 非不是               |


eg:

```sql
#查询年龄等于88的员工
select * from emp where age = 88;

#小于20
select * from emp where age < 20;

#查询没有身份证号的员工
select * from emp where idcard is null;

#有身份证号的员工
select * from emp where idcard is not null;

#年龄在15到20(包含)的员工
select * from emp where age >= 15 && age <= 20;
select * from emp where age between 15 and 20;

# 性别女且小于25
select * from emp where gender = '女' and age < 25;

#18 20 40 的
select * from emp where age = 18 || age = 20 || age = 40;
select * from emp where age in(18,20,40);

#名字是两个字的
select * from emp where name like '__';

#身份证号最后一位为X的
select from emp where idcard like '%X';
```

---

### 2.4.3 聚合函数

>介绍: 将一列数据作为一个整体,进行纵向计算

| 函数  | 功能     |
|-------|----------|
| count | 统计数量  |
| max   | 最大值    |
| min   | 最小值    |
| avg  | 平均值    |
| sum   | 求和      |

语法

```sql
select 聚合函数(字段列表) from 表名;
```

**ps : null值不参与聚合函数的运算**

```sql
#统计员工数量
select count(*) from emp;
select count(idcard) from emp;

#统计平均年龄
select avg(age) from emp;

#最大年龄
select max(age) from emp;

#最小
select min(age) from emp;

#西安地区员工年龄之和
select sum(age) from emp where workaddress = '西安';
```


---

### 2.4.4 分组查询
语法:

```sql
select 字段列表 from 表名 [where 条件] group by 分组字段名 [having 分组后过滤条件];
```
>`where`和`having`的区别
>	- 执行实际不同:
>	where是分组之前进行过滤,不满足where条件,不参与分组;
>having是分组后对结果进行过滤
>- 判断条件不同
>where不能对聚合函数进行判断
>having可以
>
>执行顺序 : where > 聚合函数 > having
>分组后,查询的字段一般为聚合函数和分组字段,查询其他字段无意义

```sql
#根据性别分组,统计男性员工和女性员工的数量
select gender,count(*) from emp group by gender;

#根据性别分组,统计男女平均年龄
select gender,avg(age) from emp group by gender;

#查询年龄小于45,根据工作地址分组,获取员工数量大于3的工作地址
select workaddress,count(*) address_count from emp where age < 45 group by workaddress having address_count >= 3;
```
---
### 2.4.5 排序查询



>语法

```sql
select 字段列表 from 表名 group by 字段1 排序方式1, 字段2 排序方式2;
```
排序方式:
1. asc : 升序(默认值)
2. desc : 降序

**ps: 如果是多字段排序,当第一个字段值相同时,才会根据第二个字段进行排序;**

>eg:

```sql
#根据年龄升序
select * from emp order by age asc;
select * from emp order by age;

#入职时间降序
select * from emp order by entrydate desc;

#年龄升序,相同则入职时间降序
select * from emp order by age asc , entrydate desc;
```

---


### 2.4.6 分页查询


>语法

```sql
select 字段列表 from 表名 limit 起始索引,查询记录数;
```
ps:
1. 起始索引从0开始,`起始索引 = (查询页码-1) * 每页显示记录数`
2. 分页查询时数据库方言,不同数据库由不同的实现,MySQL时limit
3. 如果查询的是第一页数据,起始索引可以省略,直接简写为 limit 10

>eg:

```sql
#查询第一页员工数据,每页10条
select * from emp limit 0,10;
select * from emp limit 10;

#查询第二页,每页10
select * from emp limit 10,10;

#查询年龄为20,21,22,23岁的女性员工信息。
select * from emp where gender ='女' and age in(20,21,22,23);

#查询性别为 男 ，并且年龄在 20-40 岁(含)以内的姓名为三个字的员工。
select * from emp where gender = '男' and ( age between 20 and 40 ) and name like '___';

#统计员工表中，年龄小于60岁的 ，男性员工和女性员工的人数。
select gender，count(*) from emp where age < 60 group by gender;

#查询所有年龄小于等于35岁员工的姓名和年龄，并对查询结果按年龄升序排序，如果年龄相同按入职时间降序排序
select name , age from emp where age <= 35 order by age asc , entrydate desc;

#查询男，20-40 岁(含)以内的前5个员工信息 按年龄升序，相同按入职时间升序排序。
select * from emp where gender = '男' and age between 20 and 40 order by age asc , entrydate asc limit 5 ;
```


### 2.4.7 执行顺序

```sql
#查询年龄大于15的员工的姓名、年龄，并根据年龄进行升序排序
#通过设置别名和使用别名的方法进行验证
select e.name ename , e.age eage from emp e where e.age > 15 order by age asc;
```
```sql
#编写顺序
select
	字段列表
from
	表名列表
where
	条件列表
group by
	分组字段列表
having
	分组后条件判断
order by
	排序字段列表
limit
	分页参数
```
```sql
#执行顺序,selec跑到having后
from
	表名列表
where
	条件列表
group by
	分组字段列表
having
	分组后条件判断
select
	字段列表
order by
	排序字段列表
limit
	分页参数
```



---
## 2.5 DCL

### 2.5.1 管理用户

>语法
```sql
#查询用户
use mysql;
select * from user;

#创建用户
create user '用户名'@'主机名' identified by '密码';

#修改用户密码
alter user '用户名'@'主机名' identified with mysal_native_password by '新密码';

#删除用户
drop user '用户名'@'主机名'
```
ps : 
- 主机名可以使用 % 通配
 使用这类SQL开发人员操作的比较少，主要是DBA (Database Administrator数据库管理员)

>eg:

```sql
#创建用户 9tse ，只能够在当前主机Localhost访间，密码123456;
create user '9tse'@'localhost' identified by '123456':

#创建用户 sewer ，可以在任意主机访问该数据库，密码123456 ;
create user 'sewer'@'%' identified by '123456';

#修改用户 9tse 的访问密码为 1234 ;
alter user '9tse'@'%' identified with mysql_native_password by '1234';

#删除9tse@localhost用户
drop user'9tse'@'localhost'
```

---
### 2.5.2 权限控制
常用权限如下

| 权限                 | 说明                 |
|----------------------|----------------------|
| ALL, ALL PRIVILEGES | 所有权限             |
| SELECT               | 查询数据              |
| INSERT               | 插入数据              |
| UPDATE               | 修改数据              |
| DELETE               | 删除数据              |
| ALTER                | 修改表                |
| DROP                 | 删除数据库/表/视图    |
| CREATE               | 创建数据库/表         |




>语法

```sql
#查询权限
show grants for '用户名'@'主机名';

#授予权限
grant 权限列表 on 数据库名.表名 to '用户名'@'主机名';

#撤销权限
revoke 权限列表 on 数据库名.表名 from '用户名'@'主机名';
```
ps:
1. 多个授权之间逗号隔开
2. 授权时,数据库名和表名可以用通配符*表示所有;

>eg:

```sql
-- 查询权限
show grants for '9tse'@'%';

-- 授予权限
grant all on sewerperson.* to '9tse'@'%';

-- 撤销权限
revoke all on sewerperson.* from '9tse'@'%';
```


---

# 3. 函数

## 3.1 字符串函数

| 函数                     | 功能                                               |
|--------------------------|----------------------------------------------------|
| concat(s1, s2, ... sn)   | 字符串拼接，将s1，s2，... sn拼接成一个字符串        |
| lower(str)               | 将字符串str全部转为小写                           |
| upper(str)               | 将字符串str全部转为大写                           |
| lpad(str, n, pad)        | 左填充，用字符串pad对str的左边进行填充，达到n个字符串长度 |
| rpad(str, n, pad)        | 右填充，用字符串pad对str的右边进行填充，达到n个字符串长度 |
| trim(str)                | 去掉字符串头部和尾部的空格                         |
| substring(str, start, len) | 返回从字符串str从start位置起的len个长度的字符串    |


>字符串函数的基本使用
```sql
-- concat
select concat('Hello''MysQL');

-- Tower
select lower('Hello'):

-- upper
select upper('Hello');

-- lpad
select Lpad('01'，5,'-'); # ---01

-- rpad
select rpad('01'，5,'-');

-- trim
select trim(' HelloMysQL');

-- substring
select substring('Hello MySL',1,5); # Hello
```

>案例

```sql
-- 业务需求变更，员工的工号统一5位数，不足5位数的全部在前面补0。比如: 1号员工的工号应该为00001.
update emp set workno = lpad(workno,5,'0');
```


---

## 3.2 数值函数
| 函数          | 功能                       |
|---------------|----------------------------|
| ceil(x)       | 向上取整                   |
| floor(x)      | 向下取整                   |
| mod(x; y)     | 返回 x/y 的模               |
| rand()        | 返回 0~1 内的随机数         |
| round(x; y)   | 求参数 x 的四舍五入的值，保留 y 位小数 |

>数值函数的基本利用
```sql
-- ceil
select ceil(1.1):

-- floor
select floor(1.9);

-- mod
select mod(7,4);

-- rand
select rand();

- round
select round(2.344,2);
```

>案例

```sql
-- 案例：通过数据库的函数，生成一个六位数的随机验证码。select
lpad(round(rand()*1000000, 0), 6, '0'); 
```


---

## 3.3 日期函数
| 函数              | 功能                                 |
|-------------------|--------------------------------------|
| curdate()         | 返回当前日期                         |
| curtime()         | 返回当前时间                         |
| now()             | 返回当前日期和时间                   |
| year(date)        | 获取指定 date 的年份                  |
| month(date)       | 获取指定 date 的月份                  |
| day(date)         | 获取指定 date 的日期                  |
| date_add(date, interval expr type) | 返回一个日期/时间值加上一个时间间隔 expr 后的时间值 |
| datediff(date1, date2) | 返回起始时间 date1 和结束时间 date2 之间的天数 |

>日期函数的基本使用

```sql
-- curtime()
select curtime();

 -- now()
select now();

-- YEAR,MONTH,DAY 
select YEAR(now());
select MONTH(now()); 
select DAY(now()); 

-- date_add
select date_add(now(), interval 70 year); 

-- datediff
select datediff('2021-10-01', '2021-12-01'); 
```

>案例

```sql
-- 案例：查询所有员工的入职天数，并根据入职天数倒序排序。
select name, datediff(curdate(), entrydate) as 'entrydays' from emp order by entrydays desc; 
```

---


## 3.4 流程控制函数
| 函数                   | 功能                                       |
|------------------------|--------------------------------------------|
| if(value , t, f)       | 如果 value 为 true，则返回 t，否则返回 f   |
| ifnull(value1, value2) | 如果 value1 不为空，返回 value1，否则返回 value2 |
| case when [val1] then [res1] ... else [default] end | 如果 val1 为 true，则返回 res1，... 否则返回 default 默认值 |
| case [expr] when [val1] then [res1] ... else [default] end | 如果 expr 的值等于 val1，则返回 res1，... 否则返回 default 默认值 |


>流程控制函数的基本使用

```sql
--if
select if(false, '0k', 'Error'); 

-- ifnull
select ifnull('0k','Default');
select ifnull('','Default'); 
select ifnull(null,'Default'); 
```

>案例

```sql
-- case when then elserend

-- 需求：查询emp表的员工姓名和工作地址（北京／上海————＞一线城市，其他————＞二线城市）
select
	name,
	(case workaddress when ＇北京＇ then ＇一线城市＇ when＇上海＇ then ＇一线城市＇ else ＇二线城市＇end)as＇工作地址＇ 
from emp;
```


```sql
--案例: 统计班级各个学员的成绩，展示的规则如下:
-->= 85,展示松秀
-->= 60，展示及格
--否则，展示不及格

select
	id,
	name,
	(case when math >= 85 then '优秀' when math >=60 then '及格' else '不及格' end)'数学',
	(case when english >= 85 then '优秀' when english >=60 then '及格' else '不及格' end)'英语',
	(case when chinese >= 85 then '优秀' when chinese >=60 then '及格' else '不及格' end)'语文',
from score;
```
---

# 4. 约束
## 4.1 概念

>概念 : 约束时作用于表中字段上的规则,用于限制存储在表中的数据
>目的 : 保证数据库中数据的正确,有效性和完整性
>`注意` : 约束时作用于表中的字段上的,可以在创建表/修改表的时候添加约束. 

| 约束       | 描述                                               | 关键字        |
|------------|----------------------------------------------------|----------------|
| 非空约束   | 限制该字段的数据不能为 null                      | not null       |
| 唯一约束   | 保证该字段的所有数据都是唯一、不重复的             | unique         |
| 主键约束   | 主键是一行数据的唯一标识，要求非空且唯一           | primary key    |
| 默认约束   | 保存数据时，如果未指定该字段的值，则采用默认值     | default        |
| 检查约束   | 检查约束 (8.0.16 版本之后) 保证字段值满足某一个条件 | check          |
| 外键约束   | 用来让两张表的数据之间建立连接，保证数据的一致性和完整性 | foreign key    |


---
## 4.2 案例

建表要求:

| 字段名 | 字段含义 | 字段类型  | 约束条件                          | 约束关键字                |
|--------|----------|-----------|-----------------------------------|--------------------------|
| id     | ID唯一标识 | int       | 主键，并且自动增长               | primary key auto increment |
| name   | 姓名     | varchar(10) | 不为空，并且唯一                  | not null, unique          |
| age    | 年龄     | int       | 大于0，并且小于等于120            | check (age > 0 and age <= 120) |
| status | 状态     | char(1)   | 如果没有指定该值，默认为1         | default '1'               |
| gender | 性别     | char(1)   | 无                                |                          |



```sql
create table user(
	id int primary key auto_increment comment '主键',
	name varchar(10) not null unique comment '姓名' ,
	age int check ( age >  && age <= 120 ) comment'年龄'
	status char(1) default '1' comment'状态',
	gender char(1) comment '性别'
)comment'用户表';
```

```sql
insert into user(name,age,status,gender) values ('Tom1',19,'1','男'),('Tom2',25,'0','男');
insert into user(name,age,status,gender) values ('Tom3',19,1'男');

insert into user(name,age,status,gender) values (null,19,'1'，'男');
insert into user(name,age,status,gender) values ('Tom3',19,'1'，'男');

insert into user(name,age,status,gender) values ('Tom4',80,1'，男');
insert into user(name,age,status,gender) values ('Tom5',-1,1'，男');
insert into user(name,age,status,gender) values ('Tom5',121,'1'，'男');
insert into user(name,age,gender) values ('Tom5',120，'男');
```
`当不满足建表约束时就会报错 : 无法插入数据`

---

## 4.3 外键约束
>语法
```sql
#添加外键
create table 表名(
	字段名 数据类型,
	...
	[constraint] [外键名称] foreign key(外键字段名) references 主表(主表列名)
);

alter table 表名 add constraint 外键名称 foreign key(外键字段名) references 主表(主表列名);


#删除外键
alter table 表名 drop foreign key 外键名称;
```

>案例

```sql
-- 添加外键
alter table emp add constraint fk_emp_dept_id foreign key (dept_id) references dept(id);
-- 删除外键
alter table emp drop foreign key fk_emp_dept_id;
```


`实际中并不常用外键,会导致耦合度较高`
>删除更新行为的函数

| 行为        | 说明                                                                                               |
|------------|----------------------------------------------------------------------------------------------------|
| no action  | 当在父表中删除/更新对应记录时，首先检查该记录是否有对应外键，如果有则不允许删除/更新。(与restrict一致) |
| restrict   | 当在父表中删除/更新对应记录时，首先检查该记录是否有对应外键，如果有则不允许删除/更新。(与no action 一致) |
| cascade    | 当在父表中删除/更新对应记录时，首先检查该记录是否有对应外键，如果有，则也删除/更新外键在子表中的记录。    |
| set null   | 当在父表中删除对应记录时，首先检查该记录是否有对应外键，如果有则设置子表中该外键值为null (这就要求该外键允许取null)。 |
| set default| 父表有变更时，子表将外键列设置成一个默认的值(Innodb不支持)。                                      |

```sql
alter table 表名 add constraint 外键名称 foreign key (外键字段) references 主表名(主表字段名) on update cascade on delete cascade;
```

>案例

```sql
-- 外键的删除和更新行为
alter table emp add constraint fk_emp_dept_id foreign key (dept_id) references dept(id) on update cascade on delete cascade;
alter table emp add constraint fk_emp_dept_id foreign key (dept_id) references dept(id) on update set null on delete set null;
```



---

# 5. 多表查询

**先拿一段案例展示一下表之间的外键连接**

>多对多 中间以外键相连

```sql
create table student_course(
	id int auto increment comment '主键' primary key,
	studentid int not null comment '学生ID',
	courseid  int not null comment '课程ID',
	constraint fk_courseid foreign key (courseid) references counse (id),
	constraint fk_studentid foreign key (studentid) references student (id)
)comment '学生课程中间表';
insert into student_course values (null,1,1),(null,1,2),(null,1,3),(null,2,2),(null,2,3),(null,3,4);
```


>一对一 中间多创建一个表(tb_user_edu) 相连两个表

```sql
create table tb_user(
	id int auto_increment primary key comment comment '主键ID',
	name varchar(10) comment '姓名',
	age int comment '年龄',
	gender char(1) comment '1: 男 ，2: 女'，
	phone char(11) comment '手机号'
)comment '用户基本信息表';

create table tb_user_edu(
	id int auto_increment primary key comment '主键ID',
	degree varchar(20) comment '学历',
	major varchar(50) comment '专业',
	primaryschool varchar(50) comment'小学'
	middleschool varchar(50) comment'中学'
	university varchar(50) comment'大学',
	userid int unique comment'用户ID',
	constraint fk_userid foreign key (userid) references tb_user(id)
)comment'用户教育信息表';
```


**多表查询概述**
- 概述：指从多张表中查询数据
- 笛卡尔积：笛卡尔乘积是指在数学中，两个集合A集合和B集合的所有组合情况。（在多表查询时，需要消除无效的笛卡尔积）

>![在这里插入图片描述](/img/c.2.png)

`可以通过where来消除笛卡尔积`


```sql
select * from emp,dept where emp.dept_id = dept.id;
```

多表查询可以分为
- 连接查询
	- 内连接
	- 外连接
		- 左外连接
		- 右外连接
	- 自连接
- 子查询

具体含义在小章节会简述

---
## 5.1 连接查询
### 5.1.1 内连接
 `内连接查询的是两张表交集的部分`
 
```sql
#内连接查询语法
-- 隐式内连接
select 字段列表 from 表1,表2 where 条件...；
 
-- 显式内连接
select 字段列表 from 表1 [inner] join 表2 on 连接条件...；
```

>eg:


```sql
-- 1.查询每一个员工的姓名，及关联的部门的名称（隐式内连接实现）
-- 表结构：emp，dept
-- 连接条件：emp.dept_id=dept.id
select emp.name，dept.name from emp,dept where emp.dept_id = dept.id;
select e.name,d.name from emp e , dept d where e.dept_id = d.id;


-- 2.查询每一个员工的姓名，及关联的部门的名称（显式内连接实现）---- INNER JOIN..ON//
-- 表结构：emp，dept
-- 连接条件：emp.dept_id=dept.id
select e.name，d.name from emp e inner join dept done.dept_id =d.id;
select e.name，d.name from emp e join dept d on e.dept_id = d.id;
```

---

### 5.1.2 外连接

```sql
#外连接查询语法
-- 左外连接
select 字段列表 from 表1 left [outer] join 表2 on 条件...;
#相当于查询表1（左表）的所有数据包含表1和表2交集部分的数据


-- 右外连接
select 字段列表 from 表1 right [outer] join 表2 on 条件...;
#相当于查询表2（右表）的所有数据包含表1和表2交集部分的数据
```

>eg

```sql
-- 1 查询emp表的所有数据，和对应的部门信息（左外连接
-- 表结构：emp，dept
-- 连接条件：emp.dept_id=dept.id
select e.*，d.name from emp e left outer join dept d on e.dept_id = d.id；
select e.*，d.name from emp e left join dept d on e.dept_id =d.id;


-- 2 查询dept表的所有数据，和对应的员工信息（右外连接）
select d.* , e. from emp e right outer join dept d on e.dept_id = d.id;
select d.* , e. from dept d left outer join emp e on e.dept_id = d.id;
```

---

### 5.1.3 自连接

```sql
#自连接查询语法：
select 字段列表 from 表A 别名A join 表A 别名B on 条件...;
#自连接查询，可以是内连接查询，也可以是外连接查询
```

>eg:

```sql
-- 1.查询员工及其所属领导的名字
-- 表结构：emp
select a.name , b.name from emp a , emp b where a.managerid = b.id;


-- 2，查询所有员工emp及其领导的名字emp，如果员工没有领导，也需要查询出来
-- 表结构：emp a，emp b
select a.name  '员工' , b.name '领导' from emp a left join emp b on a.managerid = b.id；
```

---

## 5.2 联合查询

`对于union查询，就是把多次查询的结果合并起来，形成一个新的查询结果集`
```sql
select 字段列表 from 表a ...
union [all]
select 字段列表 from 表b ...;
```
ps : 
- 对于联合查询的多张表的列数必须保持一致，字段类型也需要保持一致。
- **unionall会将全部的数据直接合并在一起， union会对合并之后的数据去重。**


>eg : 


```sql
-- union all , union
-- 1.将薪资低于50的员工，和年龄大于50岁的员工全部查询出来。
select * from emp Where salary<500
union all
select * from emp where age>50;
# 查询出的结果有可能出现重复的行

select * from emp where salary< 5000
union
select * from emp where age > 50;
#去掉 all 就可以去重
```

---

## 5.3 子查询


### 5.3.1 标量子查询

>标量子查询
子查询返回的结果是单个值（数字、字符串、日期等），最简单的形式，这种子查询称为`标量子查询`
常用的操作符：=  ,<> ,  > , < , >=  , <= 


>eg : 
```sql
-- 标量子查询
-- 1. 查询“销售部”的所有员工信息
-- a. 查询“销售部”部门ID
select id from dept where name  = '销售部';

-- b. 根据销售部部门ID， 查询员工信息
select * from emp where dept_id = (select id from dept where name = '销售部');



-- 2. 查询在“方东白”入职之后的员工信息
-- a. 查询方东白的入职日期
select entrydate from emp where name = '方东白';

-- b.查询指定入职日期之后入职的员工信息
select * from emp where entrydate >（select entrydate from emp where name = '方东白');
```


---

### 5.3.2 列子查询

>子查询返回的结果是一列（可以是多行），这种子查询称为`列子查询`
常用的操作符：IN、NOT IN、ANY、SOME、ALL


| 操作符    | 描述                                  |
|-----------|---------------------------------------|
| in        | 在指定的集合范围之内，多选一             |
| not in    | 不在指定的集合范围之内                   |
| any       | 子查询返回列表中，有任意一个满足即可     |
| some      | 与 any 等同，使用 some 的地方都可以使用 any |
| all       | 子查询返回列表的所有值都必须满足 in    |


>eg : 
```sql
-- 列子查询
-- 1. 查询“销售部”和“市场部”的所有员工信息
-- a. 查询“销售部”和“市场部”的部门ID
select id from dept where name = '销售部' or name = '市场部';
-- b. 根据部门ID，查询员工信息
select * from emp where dept_id in (select id from dept where name = '销售部' or name = '市场部');


-- 2. 查询比财务部所有工资都高的员工信息
-- a. 查询所有财务部人员工资
select id from dept where name = '财务部';
select salary from emp where dept_id =（select id from dept where name = '财务部');
-- b.比财务部所有人工资都高的员工信息
select * from emp where salary > all (select salary from emp where dept_id = (select id from dept where name = '财务部'));


-- 3. 查询比研发部其中任意一人工资高的员工信息
-- a. 查询研发部所有人工资
select salary from emp where dept_id = （select id from dept where name = '研发部');
-- b. 比研发部其中任意一人工资高的员工信息
select * from emp Where salary > some (select salary from emp where dept_id = (select id from dept where name '研发部'));
```


---

### 5.3.3 行子查询

>子查询返回的结果是一行（可以是多列），这种子查询称为`行子查询`
>常用的操作符：= , <> , in , not in


>eg : 

```sql
-- 1. 查询与“张无”的薪资及直属领导相同的员工信息
-- a. 查询“张无”的薪资及直属领导
select salary，managerid from emp where name = '张无'
-- b. 查询与“张无”的薪资及直属领导相同的员工信息
select from emp where (salary，managerid)=( select salary， managerid from emp where name = '张无');
```


---

### 5.3.4 表子查询

>子查询返回的结果是多行多列，这种子查询称为`表子查询`
>常用的操作符：IN


>eg : 
```sql
-- 表子查询
-- 1. 查询与“鹿客”“宋远桥”的职位和薪资相同的员工信息
-- a. 查询“鹿枝客”，“宋远桥”的职位和薪资
select job，salary from emp where name = '鹿客' or name = '宋远桥';
-- b. 查询与“鹿客”，“宋远桥”的职位和薪资相同的员工信息
select * from emp where (job,salary) in (select job，salary from emp where name = '鹿杖客' or name = '宋远桥');


-- 2. 查询入职日期是“2086-01-01之后的员工信息，及其部门信息
-- a. 入职日期是“206-1-01”之后的员工信息
select * from emp where entrydate > '2006-01-01';
-- b. 查询这部分员工，对应的部门信息
select e.*,d.* from (select from emp where entrydate > '2006-01-01'）e left join dept d on e.dept_id = d.id; 
```

---

## 5.4 多表查询案例

```sql
-- 1. 查询员工的姓名、年龄、职位、部门信息 （隐式内连接）
-- 表：emp，dept
-- 连接条件：emp.dept_id = dept.id
select e.name , e.age , e.job , d.name from emp e , dept d where e.dept_id=d.id;

-- 2. 查询年龄小于3岁的员工的姓名、年龄、职位、部门信息（显式内连接）
-- 表：emp，dept
-- 连接条件：emp.dept_id=dept.id
select e.name , e.age , e.job , d.name from emp e inner join dept d on e.dept_id = d.id where e.age < 30;


-- 3.查询拥有员工的部门ID、部门名称
-- 表：emp，dept
-- 连接条件：emp.dept_id=dept.i
select distinct d.id , d.name from emp e , dept d where e.dept_id = d.id;


-- 4. 查询所有年龄大于4岁的员工，及其归属的部门名称；如果员工没有分配部门，也需要展示出来
-- 表：emp，dept
-- 连接条件：emp.dept_id=dept.id
-- 外连接
select e.* , d.name from emp e Left join dept d on e.dept_id = d.id where e.age > 40;


-- 5. 查询所有员工的工资等级
-- 表：emp，sagrade
-- 连接条件：emp.salary>= salgrade.losal and demp.salary <= sagrade.hisal
select e.* , s.grade , s.losal , s.hisal from emp e , salgrade s where e.salary >= s.losa and e.salary <= s.hisal;
select e.* , s.grade , s.losal , s.hisal from emp e , salgrade s where e.salary between s.losal and s.hisal;


-- 6. 查询“研发部”所有员工的信息及工资等级
-- 表：emp，salgrade，dept
-- 连接条件：emp.salary between salgrade.losal and salgrade.hisal , emp.dept_id = dept.id
-- 查询条件：dept.name= '研发部'
select e.* ,  s.grade from emp e , dept d , salgrade s where e.dept_id = d.id and (e.salary between s.losal and s.hisal)  and d.name = '研发部';


-- 7. 查询“研发部”员工的平均工资
-- 表：emp，dept
-- 连接条件： emp.dept_id = dept.id
select avg(e.salary) from emp e , dept d where e.dept_id = d.id and d.name = '研发部';


-- 8. 查询工资比“灭绝”高的员工信息
-- a. 查询“灭绝”的薪资
select salary from emp where name = '灭绝';
-- b. 查询比她工资高的员工数据
select * from emp where salary > (select salary from emp where name = '灭绝');


-- 9. 查询比平均薪资高的员工信息
-- a. 查询员工的平均薪资
select avg(salary) from emp;
-- b. 查询比平均薪资高的员工信息
select * from emp where salary > (select avg(salary) from emp);


-- 10. 查询低于本部门平均工资的员工信息
-- a. 查询指定部门平均薪资
select avg(e1.salary) from emp e1 Where e1.dept_id = 1;
select avg(e1.salary) from emp e1 where e1.dept_id = 2;
-- b. 查询低于本部门平均工资的员工信息
select * from emp e2 where e2.salary < (select avg(e1.salary) from emp e1 where e1.dept_id = e2.dept_id );


-- 11. 查询所有的部门信息，并统计部门的员工人数
select d.id , d.name , (select count(*) from emp e where e.dept_id = d.id) '人数' from dept d;
select count(*) from emp where dept_id = 1;


-- 12. 查询所有学生的选课情况，展示出学生名称，学号，课程名称
-- 表 : student , course , student_course
-- 连接条件：student.id = student_course.studentid , course.id = student_course.courseid
select s.name , s.no , c.name from student s , student_course sc , course c where s.id = sc.studentid and sc.courseid = c.id;
```

----

# 6. 事务

事务的四大特性(ACID)
- 原子性(`A`tomicity) : 事务是不可分割的最小操作单元,要么全部成功,要么全部失败.
- 一致性(`C`onsistency) : 事务完成时,必须使所有的数据都保持一致状态.
- 隔离性(`I`solation) : 数据库系统提供的隔离机制,保证事务在不受外部并发操作影响的独立环境下运行.
- 持久性(`D`urability) : 事务一旦提交或回滚,他对数据库中的数据的改变就是永久的.

## 6.1 事务的基本操作

```sql
#查看/设置事务提交方式
select @@autocommit;
set @@autoccommit = 0; --0是自动提交,1是手动提交,即使用commit

#提交事务
commit;

#回滚事务
rollback;
```



>eg : 方式一

```sql
select @@autocommit;

set @@autocommit = 0； --设置为手动提交

-- 转账操作（张三给李四转账1088）
-- 1.查询张三账户余额
select * from account where name = '张三';

-- 2.将张三账户余额-1080
update account set money = money - 1000 where name = '张三';

程执行报错... #手动报错

-- 3.将李四账户余额+1000
update account set money = money+1000 where name = '李四';

--提交事务
commit;

--回滚事务
rollback;
```

>eg : 方式二

```sql
#开启事务
start transaction 或 begin;

#提交事务
commit;

#回滚事务
rollback;
```


`已经设置为自动提交`

```sql
-- 方式二
-- 转账操作（张三给李四转账1000
start transaction;

-- 1. 查询张三账户余额
select * from account where name = '张三';

-- 2. 将张三账户余额-18
update account set money = money - 1000 where name = '张三';

手动执行报错...

-- 3.将季四账户余额+1000
update account set money = money + 1000 where name = '李四';

-- 提交事务
commit;

--回滚事务
rollback;
```


---
## 6.2 并发事务问题

| 问题         | 描述                                             |
|--------------|--------------------------------------------------|
| 脏读         | 一个事务读到另外一个事务还没有提交的数据           |
| 不可重复读    | 一个事务先后读取同一条记录，但两次读取的数据不同，称之为不可重复读。 |
| 幻读         | 一个事务按照条件查询数据时，没有对应的数据行，但是在插入数据时，又发现这行数据已经存在，好像出现了 "幻影"。   |


- 脏读
B事务读取到了A事务还没有提交的数据
>![在这里插入图片描述](/img/c.3.png)
- 不可重复读
事务A用同样的方法读到了不一样的数据
>![在这里插入图片描述](/img/c.4.png)
- 幻读
事务A插入不进去,也读不出来,可以理解为,脏写,hhh
>![在这里插入图片描述](/img/c.5.png)


---

## 6.3 事务的隔离级别
| 隔离级别          | 读    | 不可重复读 | 幻读   |
|-------------------|-------|------------|--------|
| read uncommitted  | false | false      | false  |
| read committed  (oracle 的默认)   | true  | false      | false  |
| repeatable read (默认)   | true  | true       | false  |
| serializable      | true  | true       | true   |


```sql
#查看事务隔离级别
select @@transaction_isolation;

#是指事务隔离级别
set [session|global] transaction isolation level [read uncommitted | read committed | repeatable read | serializable ];
```

```sql
-- 查看事务隔离级别
select @@atransaction_isoation

-- 设置事务隔离级别
set session transaction isolation level read uncommitted ;
set session transaction isolation Level repeatable read ;
```

需要注意的是 : `事务隔离级别越高,数据越安全,但是性能会越低`

