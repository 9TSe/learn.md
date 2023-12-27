---
title: gcc/g++
date: 2023-10-22 09:51:14
tags:
- gcc/g++
categories:
- Linux
cover: /pic/3.png
---



---
# 一、安装gcc

```bash
#Ubuntu
sudo apt update   #更新本地的软件下载列表, 得到最新的下载地址
sudo apt install gcc g++

#centos
sudo yum update 
sudo yum install gcc g++
```
>之所以更新下载列表,是因为这样可以下载最新的gcc g++ 以支持c++11

```bash
#两种方法查看gcc g++版本
gcc -v
gcc --version

g++ -v
g++ --version
```

`最低4.8.5的版本才支持C++11`

---
# 二、gcc工作流程


---

>gcc工作分为四步
>1.预处理: 在这个阶段主要做了三件事: 展开头文件 、宏替换 、去掉注释行
这个阶段需要GCC调用预处理器来完成, 最终得到的还是源文件, 文本格式
>2.编译: 这个阶段需要GCC调用编译器对文件进行编译, 最终得到一个汇编文件
>3.汇编: 这个阶段需要GCC调用汇编器对文件进行汇编, 最终得到一个二进制文件
>4.链接: 这个阶段需要GCC调用链接器对程序需要调用的库进行链接, 最终得到一个可执行的二进制文件

|文件名后缀|说明  | gcc参数|
|--|--|--|
| .c |源文件  | 无 |
| .i | 预处理后的 C 文件 |  -E|
|  .s| 编译之后得到的汇编语言的源文件 |  -S|	
| .o |  汇编后得到的二进制文件|  -c|


>如果要编译单个文件非常简单,只需要
>gcc c源文件
>会自动生成文件名为a.out的可执行文件(若已经存在会覆盖)
>也可以通过 -o 参数指定生成的文件名

以下是4个步骤单独执行的方式

```bash
#对源文件进行预处理,gcc使用-E参数
#1. 预处理,-o指定生成的文件名
gcc -E test.c -o test.i

#2. 编译,得到汇编文件
gcc -S test.i -o test.s

#3.汇编
gcc -c test.s -o test.o

#4.链接
gcc test.o -o test
```

>如果直接执行汇编或者链接,前面几步会自动执行
```bash
# 参数 -c 是进行文件的汇编, 汇编之前的两步会自动执行
gcc test.c -c -o app.o

# 该命令是直接进行链接生成可执行程序, 链接之前的三步会自动执行
gcc test.c -o app    
```
---
# 三、gcc常用参数

---


`参数在gcc命令中并没有位置要求,指定即可`
| gcc编译选项      | 选项的意义                                    |
|----------------|--------------------------------------------|
| -E             | 预处理指定的源文件，不进行编译                      |
| -S             | 编译指定的源文件，但是不进行汇编                    |
| -c             | 编译、汇编指定的源文件，但是不进行链接                |
| -o [file1] [file2] / [file2] -o [file1] | 将文件 file2 编译成文件 file1 |
| -I directory   | 指定 include 包含文件的搜索目录                   |
| -g             | 在编译的时候，生成调试信息，该程序可以被调试器调试    |
| -D             | 在程序编译的时候，指定一个宏                        |
| -w             | 不生成任何警告信息, 不建议使用, 有些时候警告就是错误 |
| -Wall          | 生成所有警告信息                                 |
| -On            | n的取值范围：0~3。编译器的优化选项的4个级别        |
| -l             | 在程序编译的时候，指定使用的库                     |
| -L             | 指定编译的时候，搜索的库的路径。                   |
| -fPIC/fpic     | 生成与位置无关的代码                              |
| -shared        | 生成共享目标文件。通常用在建立共享库时             |
| -std           | 指定C方言，如:-std=c99，gcc默认的方言是GNU C    |

---


## 1.指定生成的文件名(-o)

```bash
# 参数 -o的用法 , 原材料 test.c 最终生成的文件名为 app
# test.c 写在 -o 之前
$ gcc test.c -o app

# test.c 写在 -o 之后
gcc -o app test.c
```
---
## 2.搜索头文件(-I)

> 如果在程序中包含了一些头文件, 但是包含的一些头文件在程序预处理的时候因为找不到无法被展开，导致程序编译失败，这时候我们可以在gcc命令中添加 -I参数重新指定要引用的头文件路径, 保证编译顺利完成。


```bash
gcc *.c -o calc -I ./include #编译当前目录内所有的c程序,命名为calc 
#头文件在include文件中,指明头文件所在地
#如果头文件就在当前目录中不加-I也可以执行
```

---
## 3.指定一个宏(-D)


> 如果不想在程序中定义某个宏， 但是又想让它存在，通过gcc的参数 -D就可以实现，编译器会认为参数后边指定的宏在程序中是存在的。

```bash
gcc test.c -o app -D DEBUG #编译中定义DEBUG这个宏
```

> -D 参数的应用场景:
在发布程序的时候, 一般都会要求将程序中所有的log(日志)输出去掉,
如果不去掉会影响程序的执行效率，删除这些打印log的源代码是很麻烦的
解决方案是这样的：
将所有的打印log的代码都写到一个宏判定中, 可以模仿上边的例子
在编译程序的时候指定 -D 就会有log输出
在编译程序的时候不指定 -D, log就不会输出


---

# 四、多文件编译

gcc是可以多文件一起编译的

```bash
#生成可执行程序test
gcc -o test string.c main.c

#执行test
./test
```

```bash
#先将源文件编成目标文件，然后进行链接得到可执行程序
# 汇编生成二进制目标文件, 指定了 -c 参数之后, 源文件会自动生成 string.o 和 main.o
$ gcc –c string.c main.c

# 链接目标文件, 生成可执行程序 test
$ gcc –o test string.o main.o

# 运行可执行程序
$ ./test
```

---

# 五、gcc与g++

---
以下是gcc和g++的区别

 1. 在代码编译阶段（第二个阶段）:
后缀为 .c 的，gcc 把它当作是C程序，而 g++ 当作是 C++ 程序
后缀为.cpp的，两者都会认为是 C++ 程序，C++ 的语法规则更加严谨一些
g++会调用gcc，对于C++代码，两者是等价的, 也就是说 gcc 和 g++ 都可以编译 C/C++代码
 2. 在链接阶段（最后一个阶段）:
gcc 和 g++ 都可以自动链接到标准C库
g++ 可以自动链接到标准C++库, gcc如果要链接到标准C++库需要加参数 -lstdc++
 3. 关于 __cplusplus宏的定义
g++ 会自动定义__cplusplus宏，但是这个不影响它去编译C程序
gcc 需要根据文件后缀判断是否需要定义 __cplusplus 宏 （规则参考第一条）

> 综上所述：
不管是 gcc 还是 g++ 都可以编译 C 程序，编译程序的规则和参数都相同
g++可以直接编译C++程序， **gcc 编译 C++程序需要添加额外参数 -lstdc++**
不管是 gcc 还是 g++ 都可以定义 __cplusplus宏

```bash
#编译 c 程序
gcc test.c -o test	# 使用gcc
g++ test.c -o test	# 使用g++

#编译 c++ 程序
gcc test.cpp -lstdc++ -o test     # 使用gcc
g++ test.cpp -o test              # 使用g++
```


---