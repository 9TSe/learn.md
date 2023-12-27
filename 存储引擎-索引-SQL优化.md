---
title: 存储引擎,索引,SQL优化
date: 2023-10-22 09:55:55
tags:
categories:
- MySQL
cover: /pic/8.png
---


# 1. 存储引擎

## 1.1 MySQL体系结构
![在这里插入图片描述](/img/c.6.png)

>- 连接层
最上层是一些客户端和链接服务，主要完成一些类似于连接处理、授权认证、及相关的安全方案。服务器也会为安全接入的每个客户
端验证它所具有的操作权限。
>- 服务层
第二层架构主要完成大多数的核心服务功能，如SQL接口，并完成缓存的查询，SQL的分析和优化，部分内置函数的执行。所有跨存储引擎的功能也在这一层实现，如过程、函数等。
>- 引擎层
存储引擎真正的负责了MSQL中数据的存储和提取，服务器通过API和存储引擎进行通信。不同的存储引擎具有不同的功能，这样我
们可以根据自己的需要，采选取合适的存储引擎。
>- 存储层
主要是将数据存储在文件系统之上，并完成与存储引的交互


---

## 1.2 存储引擎简述

>存储引擎就是存储数据、建立索引、更新/查询数据等技术的实现方式。存储引擎是基于表的，而不是基于库的，所以存储引擎也可被称为表类型。

```sql
show create table account; #查询建表语句,默认为innoDB
show engines; #查询当前数据库支持的搜索引擎

#创建表时,指定存储引擎
create table 表名(
	字段1,字段1类型[comment 注释]
	) engine = innodb [comment 注释];
```

---

## 1.3 存储引擎的特点

### 1.3.1 innoDB


- 介绍
nnoDB是一种兼顾高可靠性和高性能的通用存储引擎，在MySQL5.5之后，InnoDB是默认的MySOL存储引擎。
- 特点
DML操作遵循ACID模型，支持`事务`；
`行级锁`，提高并发访问性能
支持`外键`FOREIGNKEY约束，保证数据的完整性和正确性
- 文件
xxx.ibd：xxx代表的是表名，innoDB引擎的每张表都会对应这样一个表空间文件，存储该表的表结构（frm、sdi）、数据和索引。参数：innodb_file_per_table


![在这里插入图片描述](/img/c.7.png)

---

### 1.3.2 MyISAM


- 介绍
 MyISAM是MySQL早期的默认存储引擎。
 - 特点
 不支持事务，不支持外键
 支持表锁，不支持行锁
 访问速度快
 - 文件
xxx.sdi：存储表结构信息
XXX.MYD：存储数据
XXX.MYI：存储索引



---

### 1.3.3 Memory


- 介绍
Memory引擎的表数据时存储在内存中的，由于受到硬件问题、或断电问题的影响，只能将这些表作为临时表或缓存使用。
- 特点
内存存放
hash索引（默认）
- 文件
xxx.sdi：存储表结构信息



---

### 1.3.4 存储引擎的选择

- `InnoDB`：是Mysl的默认存储引擎，支持事务、外键。如果应用对事务的完整性有比较高的要求，在并发条件下要求数据的一致性，数据操作除了插入和查询之外，还包含很多的更新、删除操作，那么InnoDB存储引擎是比较合适的选择。
- MyISAM：如果应用是以读操作和插入操作为主，只有很少的更新和删除操作，并且对事务的完整性、并发性要求不是很高，那么选择这个存储引擎是非常合适的
- MEMORY：将所有数据保存在内存中，访问速度快，通常用于临时表及缓存。MEMORY的缺陷就是对表的大小有限制，太大的表无法缓存在内存中，而且无法保障数据的安全性

| 特点           | InnoDB              | MyISAM     | Memory    |
|----------------|---------------------|------------|-----------|
| 存储限制       | 64TB                | 有         | 有        |
| 事务安全       | 支持                | -     | -    |
| 锁机制         | 行锁                | 表锁       | 表锁      |
| B+tree索引     | 支持                | 支持       | 支持      |
| Hash索引       | -                | -     | 支持      |
| 全文索引       | 支持（5.6版本之后） | 不支持     | -    |
| 空间使用       | 高                  | 低         | N/A        |
| 内存使用       | 高                  | 低         | 中等     |
| 批量插入速度 | 低                  | 高         | 高       |
| 支持外键       | 支持                | -     | -    |




---

# 2. Linux下的MySQL

```bash
#先将MySQL导入至yum
rpm -ivh https://dev.mysql.com/get/mysql80-community-release-el7-10.noarch.rpm

#MySQL下载至yum
yum info mysql-community-server

#通过yum下载MySQL
yum -y install mysql-community-server

#启动MySQL
systemctl start mysqld

#找到默认MySQL密码
grep 'temporary password' /var/log/mysqld.log

#登录MySQL
mysql -u root -p

#临时更改严格型密码
alter user 'root'@'localhost' identified by '符合Linux检查的密码';

#改变Linux密码校验文件
set global validate_password.policy = 0; #设置检查格式为0(默认1)
set global validate_password.length = 4; #设置密码最短4个

#设置平常使用的密码
alter user 'root'@'localhost' identified by '9tse';

#创建全域用户
create user 'root'@'%' identified with mysql_native_password by '9tse';

#授予该用户所有权限
grant all on *.* to 'root'@'%';

#再登录即可
```

如果数据库连接不到Linux中,则只需开放Linux3306端口

```bash
#开放端口
sudo firewall-cmd --zone=public --add-port=3306/tcp --permanent

#刷新
sudo firewall-cmd --reload

#检查是否生效
sudo firewall-cmd --list-all
```
---

# 3. 索引

## 3.1 索引概述

> 索引（index）是帮助MySQL高效获取数据的数据结构（有序）。在数据之外，数据库系统还维护着满足特定查找算法的数据结构，这些数据结构以某种方式引用（指向）数据，  这样就可以在这些数据结构上实现高级查找算法，这种数据结构就是案引


                   
| 优势                                                   | 劣势                                                       |
|--------------------------------------------------------|------------------------------------------------------------|
| 提高数据检索的效率，降低数据库的IO成本                   | 索引列也是要占用空间的                                      |
| 通过索引列对数据进行排序，降低数据排序的成本，降低CPU的消耗 | 索引大大提高了查询效率，同时却也降低更新表的速度，如对表进行INSERT、UPDATE、DELETE时，效率降低。   |

---

## 3.2 索引结构
>MySQL的索引是在存储引擎层实现的,不同的存储引擎有不同的结构

| 索引结构          | 描述                                                                           |
|--------------------|--------------------------------------------------------------------------------|
| B+Tree索引         | 最常见的索引类型，大部分引擎都支持B+树索引。                                  |
| Hash索引           | 数据结构是用哈希表实现的，只有精确匹配索引列的查询才有效，不支持范围查询。    |
| R-tree（空间索引） | 空间索引是MyISAM引擎的一个特殊索引类型，主要用于地理空间数据类型，通常使用较少。|
| Full-text（全文索引） | 是一种通过建立倒排索引快速匹配文档的方式。类似于Lucene、Solr、Elasticsearch。  |






| 索引           | InnoDB          | MyISAM         | Memory         |
|--------------|-----------------|----------------|----------------|
| B+tree索引    | 支持             | 支持             | 支持            |
| Hash索引      | 不支持           | 不支持           | 支持            |
| R-tree索引    | 不支持           | 支持             | 不支持          |
| Full-text    | 5.6版本之后支持 | 支持             | 不支持          |

`我们平常所说的索引,如果没有特别指明,都是指的B+树结构组织的索引`




MySQL索引数据结构对经典的B+Tree进行了优化。在原B+Tree的基础上，增加一个指向相邻叶子节点的链表指针，就形成了带有顺序指针的B+Tree，提高区间访问的性能
![在这里插入图片描述](/img/c.8.png)


Hash
  - Hash索引特点
    1. Hash索引只能用于对等比较（=，in），不支持范围查询（between，>，<，...)
     2. 无法利用素引完成排序操作
	3.  查询效率高，通常只需要一次检索就可以了，效率通常要高于B+tree索引
   - 存储引擎支持
在MySQL中，支持hash素引的是Memory引擎，而InnoDB中具有自适应hash功能，hash索引是存储引擎根据B+Tree索引在指定条件下自动构建的。


---

## 3.3 索引分类
| 分类       | 含义                           | 特点              | 关键字  |
|------------|--------------------------------|-------------------|---------|
| 主键索引  | 针对于表中主键创建的索引        | 默认自动创建，只能有一个 | primary |
| 唯一索引  | 避免同一个表中某数据列中的值重复 | 可以有多个                | unique |
| 常规索引  | 快速定位特定数据                | 可以有多个                | -       |
| 全文索引  | 全文索引查找的是文本中的关键词   | 可以有多个                | fulltext |


InnoDB存储引擎中,根据索引的存储形式,又可以分为以下两种

| 分类               | 含义                                                          | 特点                                                |
|--------------------|---------------------------------------------------------------|-----------------------------------------------------|
| 聚集索引         | 将数据存储与索引放到了一块，索引结构的叶子节点保存了行数据     | 必须有，而且只有一个                               |
| 二级索引           | 将数据与索引分开存储，索引结构的叶子节点关联的是对应的主键     | 可以存在多个                                       |

聚集索引选取规则
   - 如果存在主键，主键引就是聚集索引
   - 如果不存在主键，将使用第一个唯一（unique）引作为聚集索引。
   - 如果表没有主键，或没有合适的唯一索引，则InnoDB会自动生成一个rowid作为隐藏的聚集索引。


>思考,innoDB主键索引的B+tree高度为多高呢？


假设：
    一行数据大小为1k，一页中可以存储16行这样的数据。InnoDB的指针占用6个字节的空间，主键即使为bigint，占用字节数为8
高度为2：
     n * 8+（n+1）* 6 = 16 * 1024，算出n约为1170
     1171*16=18736
高度为3
       1171 * 1171 * 16 = 21939856


---

## 3.4 索引语法

```sql
#创建索引
create [unique | fulltext] index index_name on table_name(index_col_name,...);

#查看索引
show index from table_name;

#删除索引
drop index index_name on table name;
```

>eg

```sql
#1.  name字段为姓名字段，该字段的值可能会重复，为该字段创建索引。
create index idx_user_name on tb_user(name);

#2.  phone手机号字段的值，是非空，且唯一的，为该字段创建唯一索引
create unique index idx_user_phone on tb_user(phone);

#3.  profession、age、status创建联合索引
create index idx_user_pro_age_sta on tb_user(profession,age,status);

#4.  为email建立合适的引来提升查询效率。
create index idx_user_email on tb_user(email);
```



---

## 3.5 索引性能分析

**SQL执行频率**
>MySQL客户端连接成功后，通过`show[session | global] status`命令可以提供服务器状态信息。通过如下指令，可以查看当前数据库的INSERT、UPDATE、DELETE、 SELECT的访问频次：

```sql
show global status like 'Com_______'; #七个_
```



---

  **慢查询日志**
>慢查询日志记录了所有执行时间超过指定参数（long_query_time，单位：秒，默认10秒）的所有SOL语句的日志
>MySQL的慢查询日志默认没有开启，需要在MySQL的配置文件（`/etc/my.cnf`）中配置如下信息：

```bash
#开启MySQL慢日志查询开关
slow_query_log=1
#设置慢日志的时间为2秒，SQL语句执行时间超过2秒，就会视为慢查询，记录慢查询日志
long_query_time=2
```

 
配置完毕之后，重新启动MySQL服务器进行测试，查看慢日志文件中记录的信息`/var/lib/mysql/localhost-slow.log`


```sql
#查看慢查询是否开启
show variables like 'slow_query_log';
```

```bash
#实时跟踪慢查询日志文件记录的信息
tail -f localhost-slow.log 
```


---
   **profile详情**
>`show profiles` 能够在做SQL优化时帮助我们了解时间都耗费到哪里去了。
>通过 `have_profiling` 参数，能够看到当前MySQL是否支持profile操作：

```sql
#是否支持profile
select @@have_profiling;

#profiling是否开启
select @@profiling;
```

>默认profiling是关闭的，可以通过`set`语句在session/global级别开启profiling

```sql
set profiling=1;
```


执行一系列的业务SOL的操作，然后通过如下指令查看指令的执行耗时

```sql
#查看每一条SQL的耗时基本情况
show profiles;

#查看指定queny_id的SQL语句各个阶段的耗时情况
show profile for query query_id;

#查看指定queryid的SQL语句CPU的使用情况
show profile cpu for query query_id;
```



---

 **explain执行计划**
 
>`EXPLAIN或者DESC`命令获取MySQL如何执行SELECT语句的信息，包括在SELECT语句执行过程中表如何连接和连接的顺序。语法：

```sql
#直接在select语句之前加上关键字explain/des
explain select 字段列表 from 表名 where 条件;
```

EXPLAIN执行计划各字段含义：
1. id
    select查询的序列号，表示查询中执行select子句或者是操作表的顺序（`id相同，执行顺序从上到下；id不同，值越大，越先执行`）。
2. select_type
    表示SELECT的类型，常见的取值有SIMPLE（简单表，即不使用表连接或者子查询）、PRIMARY（主查询，即外层的查询）、UNION （UNION中的第二个或者后面的查询语句）、SUBQUE（SELECT/WHERE之后包含了子查询）等
3. type
   表示连接类型，性能由好到差的连接类型为`NULL`、system、`const`、eq_ref、ref、range、index、`all` 
 4. possible_key
     显示可能应用在这张表上的索引，一个或多个。
 5. Key
     实际使用的索引，如果为NULL，则没有使用索引
 6. Key_Len
    表示索引中使用的字节数，该值为索引字段最大可能长度，`并非实际使用长度`，在不损失精确性的前提下，`长度越短越好`。
 7. rows
    MySQL`认为`必须要执行查询的行数，在innodb引l擎的表中，是一个`估计值`，可能并不总是准确的
  8. filtered
    表示返回结果的行数占需读取行数的百分比，fitered的值`越大越好`。



---
## 3.6 索引的使用

  验证索引效率,略知索引的重要性

```sql
#在未建立索引之前，执行如下SOL语句，查看SOL的耗时。
select * from tb_sku where sn = '100000003745001';

#针对字段创建索引
create index idx_sku_sn on tb_sku(sn);

#然后再次执行相同的SQL语句，再次查看SQL的耗时。
select * from tb_sku where sn = '100000003745001';
```

---


### 3.6.1 索引失效


  **最左前法则**
>如果索引了多列（联合索引），要遵守最左前缀法则。最左前缀法则指的是查询从索引的最左列开始，并且不跳过索引中的列。如果跳跃某一列，`索引将部分失效（后面的字段索引失效）`。

```sql
explain select * from tb_user where profession = '软件工程' and age=3 and status='0';
explain select * from tb_user where profession = '软件工程' and age=3;
explain select * from tb_user where profession = '软件工程'; #以上索引都没有失效
explain select * from tb_user where age=3 and status = '0'; #profession不在,由于最左前缀法,索引失效
explain select * from tb_user where status = '0'; #profession,age的索引失效
```

   **范围查询**
联合索引中，出现范围查询（>,<），`范围查询右侧的列索引失效`
而 >= <=就不会

```sql
##age索引失效
explain select * from tb_user where profession='软件工程' and age > 30 and status='0';

##不失效
explain select * from tb_user where profassion='软件工程' and age >= 30 and status='0';
```


  **索引列运算**
不要在索引列上进行运算操作，索引将失效

```sql
explain select * from tb_user where substring(phone,10,2) = '15';
```

  **字符串不加引号**
字符串类型字段使用时，不加引号，索引将失效

```sql
 explain select * from tb_user where profession = '软件工程' and age= 3 and status= 0;
 explain select * from tb_user where phone = 17799990015;
```

   **模糊查询**
如果仅仅是尾部模糊匹配，索引不会失效。如果是`头部模糊匹配，索引失效`。

```sql
explain select * from tb_user where profssion like '软件%'; #不失效
explain select * from tb_user where profession like '%工程'; ##失效
explain select * from tb_user where profession like '%工%'; ##失效
```

  **or连接的条件**
用or分割开的条件，如果or前的条件中的列有索引，而后面的列中没有索引，那么涉及的索引都不会被用到。

```sql
explain select * from tb_user where id = 10 or age=23;
explain select * from tb_user where phone= '7799990017' or age= 23;
```

由于age没有索引，所以即使id、phone有索引，索引也会失效。所以需要针对于age也要建立索引。


  **数据分布影响**
如果MySOL评估使用索引比全表更慢，则不使用索引。

```sql
select * from tb_user where phone >= '17799990005'; #手机号码都>05 使用全表
select * from tb_user where phone >= '17799990015';
```

---

### 3.6.2 索引使用
  **SQL提示**
SQL提示，是优化数据库的一个重要手段，简单来说，就是在SQL语句中加入一些人为的提示来达到优化操作的目的。

```sql
#use index
explain select * from tb_user use index(idx_user_pro）where profession='软件工程';
#ignore index
explain select * from tb_user ignore index(idx_user_pro）where profession='软件工程';
#force index
explain select * from tb_user force index(idx_user_pro）where profession='软件工程';
```


**覆盖索引**
`尽量使用覆盖引`（查询使用了索引，并且需要返回的列，在该引中已经全部能够找到），减少select*

```sql
explain select id,profession from tb_user where profession='软件工程' and age = 30 and status='0';
explain select id,profession,age,status from tb_user where profession='软件工程' and age = 30 and status='0';
explain select id,profession,age,status,name from tb_user where profession='软件工程' and age = 30 and status='0';
explain select * from tb_user where profession='软件工程' and age = 30 and status='0';
```

using index condition 查找使用了素引，但是需要`回表查询`数据
using where;using index查找使用了引，但是`需要的数据都在引列中能找到`，所以不需要回表查询数据

`虽然是二级索引中寻找,但并不需要去聚集索引中再寻找`


  **前缀索引**
当字段类型为字符串（varchar，text等）时，有时候需要索引很长的字符串，这会让索引变得很大，查询时，浪费大量的磁盘IO， 影响查询效率。此时可以只将字符串的`一部分前缀，建立索引`，这样可以大大节约索引空间，从而提高索引效率，

- 语法
```sql
create index idx_xxxx on tale name(column(n));

#eg
create index idx_email_5 on tb_user(email(5));
```
 - 前缀长度
    可以根据索引的选择性来决定，而选择性是指不重复的索引值（基数）和数据表的记录总数的比值，索引选择性越高则查询效率越高
     唯一索引的选择性是1，这是最好的索引选择性，性能也是最好的
   

```sql
select count(distinct email)/count(*) from tb_user;
select count(distinct substring(email,1,5))/count(*) from tb_user;
```


  **单列索引与联合索引**
单列索引：即一个索引只包含单个列。
联合案引：即一个引包含了多个列
`如果存在多个查询条件,考虑针对于查询字段建立索引时,建议建立联合索引`



**素引设计原则**
1. 针对于数据量较大，且查询比较频繁的表建立索引。
2. 针对于常作为查询条件（where）、排序（order by）、分组（group by）操作的字段建立索引
3. 尽量选择区分度高的列作为索引，尽量建立唯一索引，区分度越高，使用索引的效率越高。
4. 如果是字符串类型的字段，字段的长度较长，可以针对于字段的特点，建立前缀索引。
5.   尽量使用联合索引，减少单列索引，查询时，联合索引很多时候可以覆盖索引，节省存储空间，避免回表，提高查询效率，
6.   要控制索引的数量，索引并不是多多益善，索引越多，维护索引结构的代价也就越大，会影响增删改的效率
7. 如果索引列不能存储NULL值，请在创建表时使用NOT NULL约束它。
当优化器知道每列是否包含NULL值时，它可以更好地确定哪个索引最有效地用于查询。



---
# 4. SQL优化

## 4.1 插入数据

 **insert优化**

```sql
#批量插入
insert into tb_test values (1,'Tom'),(2,'Cat'),(3,'lery');

#手动提交事务
stat transaction; #begin
insert into tb_test values(1,'Tom'),(2,'Cat'),(3,'Jerry');
insert into tb_test values(4,'rom'),(5,'sat'),(6,'perry');
...
commit;
```

>当大批量插入数据,使用insert语句插入性能较低,此时可以使用MySQL数据库提供的`load`插入

```sql
#客户端连接服务器时,加上参数 --loacl-infile
mysql --local-infile -u root -p  

#设置全局参数 local_infile 为 1,开启从本地加载文件导入数据的开关
set global local_infile = 1;

#执行load指令将准备好的数据加载到表结构中
load data local infile '/root/sql.log' into table 'tb_user' fields terminated by ',' lines terminated by '\n';
```

`主键顺序插入效率大于主键乱序`

---

## 4.2 主键优化

数据组织方式
在innoDB引擎中,表数据都是根据主键顺序组织存放的,这种存储方式的表成为`索引组织表`(index organized table) `IOT`


**页分裂**
![在这里插入图片描述](/img/c.9.png)

![在这里插入图片描述](/img/c.10.png)



**页合并**
![在这里插入图片描述](/img/c.11.png)

ps:
	`merge_threshold`: 合并页的阈值,可以自己设置,在创建表或者创建索引时指定


主键设计原则
 - 满足业务需求的情况下，尽量降低主键的长度
 - 插入数据时，尽量选择顺序插入，选择使用AUTO_INCREMENT自增主键。
 - 尽量不要使用UUID做主键或者是其他自然主键，如身份证号。(会导致非顺序)
  - 业务操作时，避免对主键的修改


---

## 4.3 order by 优化

1. `Using filesort`通过表的索引或全表扫描，读取满足条件的数据行，然后在排序缓冲区sort buffer中完成排序操作，所有不是通过索引直接返回排序结果的排序都叫FileSort排序
2. `Using index`：通过有序索引顺序扫描直接返回有序数据，这种情况即为using index，不需要额外排序，操作效率高。

```sql
#以age,phone建立索引
#不符合最左前缀法则
explain select id,age,phone from tb_user order by phone , age;

#需要额外的排序,using filesort
explain select id,age,phone from tb_user order by age asc,phone desc;

#可以通过建立以下索引以使用using index
create index idx_user_age_pho_ad on tb_user(age asc,phone desc);
```

原则
- 据排序字段建立合适的索引，多字段排序时，也遵循最左前法则
- 尽量使用覆盖引。
- 多字段排序，一个升序一个降序，此时需要注意联合索引在创建时的规则（ASC/DESC）
- 如果不可避免的出现filesort，大数据量排序时，可以适当增大排序缓冲区大小sort_buffer_size（默认256K）


---

## 4.4 limit优化

imit 2000000 ,10，此时需要MySQL排序前20000 10记录，仅仅返回2000000-2000010的记录，其他记录丢弃，查询排序的代价非常大。
优化思路：一般分页查询时，通过创建覆盖索引能够比较好地提高性能，可以通过`覆盖索引加子查询形式`进行优化。


---

## 4.5 count优化

```sql
explain select count(*) from tb_user;
```

 - MyISAM引擎把一个表的总行数存在了磁盘上，因此执行count（*）的时候会直接返回这个数，效率很高
 - InnoDB引擎就麻烦了，它执行count（*）的时候，需要把数据一行一行地从引擎里面读出来，然后累积计数


**count的几种用法**
count是一个聚合函数，对于返回的结果集，一行行地判断，如果count函数的参数不是NULL，累计值就加1，否则不加，最后返回累计值。

用法：count（*）、count（主键）、count（字段）、count（1）


- count（主键）
    InnoDB引擎会遍历整张表，把每一行的主键id值都取出来，返回给服务层。服务层拿到主键后，直接按行进行累加（主键不可能为null）
 -  count（字段）
    没有not null约束：InnoDB引擎会遍历整张表把每一行的字段值都取出来，返回给服务层，服务层判断是否为null，不为null，计数累加。
    有not null约束：InnoDB引擎会遍历整张表把每一行的字段值都取出来，返回给服务层，直接按行进行累加。
 -  count （1）
    InnoDB引擎遍历整张表，但`不取值`。服务层对于返回的每一行，放一个数字“1”进去，直接按行进行累加
  -  count （*）
    InnoDB引擎并不会把全部字段取出来，而是专门做了优化，`不取值`，服务层直接按行进行累加
    
`按照效率排序的话，count（字段）< count（主键id）< count（1）count（*），所以尽量使用count（*）`


---