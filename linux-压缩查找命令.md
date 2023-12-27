---
title: 压缩查找命令
date: 2023-10-21 21:09:04
tags:
categories:
- Linux
cover: /pic/3.png
---


---
# 一、压缩命令

---

## 1.tar

---

> Linux系统中自带两个原始压缩工具 : gzip , bzip2
> 但他们都有**不能打包压缩文件**和**压缩后不会保留原有文件**的问题
> 同时Linux中有自带的打包工具 : tar , 将他们联合起来可以各司其职发挥作用


关于tar的参数作用:
```bash
c : 创建压缩文件
x : 释放压缩文件内容
z : 使用gzip方式进行文件压缩
j : 使用bzip2方式进行文件压缩
v : 压缩过程中显示压缩信息, 可省略
f : 指定压缩包的名字
```

---
### ①压缩


关于tar的压缩语法:

```bash
tar 参数 生成的压缩包的名字 要压缩的文件(文件或者目录) (要压缩的文件)

压缩包的名字后缀最好使用标准后缀
gzip方式压缩 后缀为 : .tar.gz (.tgz)
bzip2 压缩 后缀为  : .tar.bz2
```

eg : 使用gzip进行压缩:

![在这里插入图片描述](/img/8.1.png)

eg : 使用bzip2进行压缩
![在这里插入图片描述](/img/8.2.png)



---
### ② 解压缩
解压的语法:

```bash
#1. 解压到当前目录
tar 参数 压缩包名

#2. 解压到指定目录
tar 参数 压缩包名 -C 解压目录
```

eg : 使用gzip方式进行解压

![在这里插入图片描述](/img/8.3.png)

eg : 使用bzip2方式进行解压
![在这里插入图片描述](/img/8.4.png)

---

## 2.zip

---

> zip并不是Linux自带的,需要安装才能使用

```bash
#ubuntu
sudo apt install zip
sudo apt install unzip

#centos
sudo yum install zip
sudo yum install unzip
```

### ①压缩
压缩的语法

```bash
zip [-r] 压缩包名 要压缩的文件 # 加入 -r 参数才能将要压缩的文件中的子目录一起压缩

#压缩包名不用指定后缀 .zip  会自行添加
```

eg:
![在这里插入图片描述](/img/8.5.png)



---
### ②解压缩
解压缩的语法

```bash
#压缩到当前目录
unzip 压缩包名

#压缩到指定目录
unzip 压缩包名 -d 解压目录
```

eg:

![在这里插入图片描述](/img/8.6.png)


---

## 3.rar

---

> rar这种压缩格式在Linux中并不常用,而是在windows中常用的格式
> 如果在Linux中压缩解压这种格式的文件需要额外安装

```bash
#Ubuntu
sudo apt install rar
```

```bash
#centos 等各种Linux版本
先在  https://www.rarlab.com/download.htm 找到Linux版本下载

然后使用 Xftp 等工具传输给 Linux 中

再对压缩包进行解压缩
tar xzvf rarlinux-x64-623.tar.gz 

再移动至opt
mv rar /opt

添加软链接方便命令解析器找到该命令
ln -s /opt/rar/rar /usr/local/bin/rar
ln -s /opt/rar/unrar /usr/local/bin/unrar
```

### ①压缩

> rar 同 zip 类似,如果压缩目录就加入参数 -r
> 且rar也会自动添加后缀

```bash
#压缩语法
rar [-r] a 压缩包名 要压缩的文件 #参数a(archive) 压缩归档
```

eg:
![在这里插入图片描述](/img/8.7.png)

### ②解压缩

```bash
#解压缩语法
#解压到当前目录
rar/unrar x 压缩包名字

#解压到指定目录
rar/unrar x 压缩包名字 解压目录
```

eg:
![在这里插入图片描述](/img/8.8.png)

---
## 4.xz

---

> .xz格式的压缩解压缩都较为繁琐,需要借助tar进行打包

### ①压缩

```bash
#压缩语法
#一:
tar cvf xxx.tar 要压缩的文件
#二:
xz -z xxx.tar
```

eg:
![在这里插入图片描述](/img/8.9.png)

---

### ②解压缩

```bash
#解压缩语法
#一:
xz -d xxx.tar.xz
#二:
tar xvf xxx.tar
```

eg: 由于释放到原本的目录会覆盖展现不出效果,因此移动到另一文件观察效果
![在这里插入图片描述](/img/8.10.png)

---


# 二、查找命令

---

> 当查找的需求比较简单时可以使用 locate which whereis
> 复杂时可以使用find和grep

`对应要搜索的文件内容, 建议放到引号中, 因为关键字中可能有特殊字符, 或者有空格, 从而导致解析错误。
关于引号， 单双都可以`
## 1.find

> find 的功能非常强大,可根据文件属性进行查找

### ①文件名(-name)

```bash
#根据文件名搜索的语法
find 搜索路径 -name 要搜索的文件名
##可以使用模糊搜索(*,?)这些模糊搜索关键字
```

```bash
#eg:搜索 root 家目录下文件后缀为txt的文件
find /root -name "*.txt"  
```

### ②文件类型(-type)

```bash
#语法格式
find 搜索路径 -type 文件类型
```

|文件类型| 类型的字符描述 |
|--|--|
|普通文件类型  |f  |
| 目录类型 |  d|
| 软连接类型 |  l|
|字符设备类型  |  c|
| 块设备类型 |  b|
| 管道类型 |  p|
| 本地套接字类型 |  s|


```bash
#eg:root目录下软链接类型的文件
find /root -type l
```

---

### ③文件大小(-size)

```bash
#语法格式
find 搜索路径 -size [ +|- ] 文件大小
			#文件大小需要加单位
			#-k(小写)
			#-M
			#-G
```

> 文件大小区间非常重要
> 1.-size 4k : 表示的区间为(3k,4k]
> 2.-size -4k : 表示的区间为(0k,3k]
> 3.-size -4k : 表示的区间为(4k,无穷)


```bash
#eg:搜索当前目录下大于1M且小于等于3M的文件
find ./ -size +1M -size -3M
```

---

### ④目录层级

> 由于Linux目录是树形,所以目录可能有很多层

```bash
#语法格式
find 搜索路径 -maxdepth n 搜索属性 属性参数 #搜索最多n层
find 搜索路径 -mindepth n 搜索属性 属性参数 #搜索最少n层
```

> 这两个参数不能单独使用,必须和其他属性一起使用

```bash
#eg:从根目录开始搜索,最多5层,文件后缀为.txt
sudo find / -maxdepth 5 -name "*.txt"
```

---

### ⑤同时执行多个操作

#### 5.1: -exec

> -exec 是find的参数, 可以在exec参数后添加其他需要被执行的shell命令。

```bash
#语法格式
find 路径 参数 参数值 -exec shell命令 {} \;
#结尾要加上{} \;
且{} \之间有空格
```

```bash
#eg: 搜索最多两层目录,以 .txt结尾的文件,并查看文件信息
find ./ -maxdepth 2 -name "*.txt" -exec ls -l {} \;
```
---

#### 5.2: -ok

> 加入 -ok 执行shell时会向用户发起询问，比如在删除搜索结果的时候

```bash
#语法格式
find 路径 参数 参数值 -ok shell命令 {} \;
```


```bash
#eg:
find ./ -maxdepth 1  -name "*.txt" -ok ls -l {} \; 
##之后同意显示文件信息,同理删除时询问是否删除
```


---
#### 5.3: xargs

> xargs 参数不同于 -exec和-ok 需要在结尾加符号,有着更直观简便的写法
> 并且在处理数据时 xargs更高效
> -exec:  将find查询的结果逐条传递给后边的shell命令
	xargs: 将find查询的结果一次性传递给后边的shell命令

```bash
#语法格式
find 路径 参数 参数值 | xargs shell命令 
```

```bash
#eg:查找文件并显示信息
find ./ -maxdepth 1  -name "*.cpp" | xargs ls -l
```

---
## 2.grep

```bash
#语法格式
grep "搜索内容" 搜索的路径/文件 参数
#	参数: -r :搜索目录中的文件内容时,必须加上-r
#   	  -i :忽略大小写
#		  -n :显示符合搜索结果那一行之前显示行号
```

```bash
#eg:搜索指定目录中哪些文件中包含字符串 include 并且显示关键字所在的行号
grep "include" ./ -rn 
```


---
## 3.locate

> locate可看作是一个简化版的find,  但是locate的效率比find要高很多。
> 原因在于它不搜索具体目录，而是搜索一个本地的数据库文件，这个数据库中含有本地所有文件信息。
> Linux系统自动创建这个数据库，并且每天自动更新一次，所以使用locate命令查不到最新变动的文件。
> 为了避免这种情况，可以在使用locate之前，先使用updatedb命令，手动更新数据库。

```bash
#先进行数据库更新
sudo update

#语法格式
locate xxx 					 #搜索所有目录下以 xxx 开头的文件
locate /home/sewerperson/xxx #指定搜索目录下以 xxx 开头的文件
locate XxX -i				 #-i 忽略搜索文件名的大小写(以xxx开头的文件也能搜索出来)
locate xxx -n 5  			 #搜索前缀为xxx的文件,并且只显示前5个匹配到的
locate -r "\.cpp$"			 #-r 表示可以使用正则表达式 (以.cpp结尾的文件)
```

关于正则表达式

```c
在正则表达式中 .可以匹配任意一个 非 \n的单字符
上边的命令中使用转译字符\对特殊字符.转译, 就得到了普通的字符.
在正则表达式中 $放到字符尾部, 表示字符串必须以这个字符结尾, 上边的命令中修饰的是字符p
正则表达式中的 字符c和后边的字符p需要进行字节匹配, 没有特殊含义
通过上面的解释 \.cpp$ 说的就是以 .cpp结尾的字符串
```

---