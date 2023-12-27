---
title: Makefile
date: 2023-10-22 09:51:39
tags:
- Makefile
categories:
- Linux
cover: /pic/3.png
---


---

> gcc命令进行单个文件的编译是比较方便的,但是如果文件数目众多就会显得冗余
> 此时就可以借助make这个命令工具
> make就好比建筑工人,makefile就是蓝图,make照着makefile进行工程
> makefile文件的命名有`Makefile`和`makefile`

---
# 一、规则


规则的基本语法格式

```bash
target1,target2,...:depend1,depend2,...
		command
		.......
		.......
```

规则主要有三部分组成:目标(target),依赖(depend),命令(command)

 - 目标(target)
 	通过执行规则中的命令，可以生成一个和目标同名的文件
 	规则可以有多个命令, 通过这多条命令来生成多个目标, 所有目标也可有很多个
	通过执行命令，可以只执行一个动作，不生成任何文件，这样的目标被称为`伪目标`
	
 - 依赖(depend)
 	如果规则的命令中不需要任何依赖，那么规则的依赖可以为空
	当前规则中的依赖可以是其他规则中的某个目标，这样就形成了规则之间的嵌套
	依赖可以根据要执行的命令的实际需求, 指定很多个
	
 - 命令(command):一般来说是一个shell命令
 	动作可是多个，每个命令前必须有一个`Tab缩进并且独占占一行`。
 	


eg:

```bash
# eg1: 有源文件 a.c b.c c.c, 生成可执行程序 app
app:a.c b.c c.c
	gcc a.c b.c c.c -o app

# eg2:有多个目标, 多个依赖, 多个命令
app,app1:a.c b.c c.c d.c
	gcc a.c b.c -o app
	gcc c.c d.c -o app1

# eg3:规则之间的嵌套
app:a.o b.o c.o
	gcc a.o b.o c.o -o app
# a.o 是第一条规则中的依赖
a.o:a.c
	gcc -c a.c
# b.o 是第一条规则中的依赖
b.o:b.c
	gcc -c b.c
# c.o 是第一条规则中的依赖
c.o:c.c
	gcc -c c.c
```

---

# 二、工作原理

---

## 1.规则的执行
在调用 make 命令编译程序的时候，make 会首先找到 Makefile 文件中的第 1 个规则，分析并执行相关的动作。
ps:很多时候要执行的动作（命令）中的依赖是不存在的，如果使用的依赖不存在，这个动作也就不会被执行。

解决方案是先将需要的依赖生成出来，我们可以在makefile中添加新的规则，将不存在的依赖作为这个新的规则中的目标，当这条新规则对应的命令执行完毕，对应的目标就被生成了，同时另一条规则中需要的依赖也就存在了。

这样，makefile中的某一条规则在需要的时候，就会被其他的规则调用，直到makefile中的第一条规则中的所有的依赖全部被生成，第一条规则中的命令就可以基于这些依赖生成对应的目标，make 的任务也就完成了。

以上即使嵌套的原理

```bash
# 规则1
app:a.o b.o c.o
	gcc a.o b.o c.o -o app
# 规则2
a.o:a.c
	gcc -c a.c
# 规则3
b.o:b.c
	gcc -c b.c
# 规则4
c.o:c.c
	gcc -c c.c
```

> 在这个例子中，如果执行 make 命令就会根据这个 makefile 中的4条规则编译这三个源文件。在解析第一条规则的时候发现里边的三个依赖都是不存在的，因此规则1对应的命令也就不能被执行。
当依赖不存在的时候，make就是查找其他的规则，看哪一条规则是用来生成需要的这个依赖的，找到之后就会执行这条规则中的命令。因此规则2， 规则3， 规则4里的命令会相继被执行，当规则1中依赖全部被生成之后对应的命令也就被执行了，因此规则1的目标被生成，make工作结束。

 - 如果想要执行 makefile 中非第一条规则对应的命令, 那么就不能直接 make, 需要将那条规则的目标也写到 make的后边, 比如只需要执行规则3中的命令, 就需要: make b.o。

---

## 2.文件的时间戳

> make 命令执行的时候会根据文件的时间戳判定是否执行makefile文件中相关规则中的命令。
> 即make的执行要根据时间来判断

主要有三种情况

 1. 如果规则中的目标对应的文件根本就不存在， 那么规则中的命令肯定会被执行。
 2.  当依赖文件被更新了, 文件时间戳也会随之被更新, 这时候 `目标时间戳 < 某些依赖的时间戳`, 在这种情况下目标文件会通过规则中的命令被重新生成。
3. 目标是通过依赖生成的，因此正常情况下：`目标时间戳 > 所有依赖的时间戳`, 如果执行 make 命令的时候检测到规则中的目标和依赖满足这个条件, 那么规则中的命令就不会被执行。


```bash
# 规则1
app:a.o b.o c.o
	gcc a.o b.o c.o -o app
# 规则2
a.o:a.c
	gcc -c a.c
# 规则3
b.o:b.c
	gcc -c b.c
# 规则4
c.o:c.c
	gcc -c c.c
```

修改例子中的 a.c, 再次通过make编译这几个源文件
那么这个时候先执行规则2更新目标文件a.o， 然后再执行规则1更新目标文件app
其余的规则是不会被执行的。

---

## 3.自动推导
> 有时漏写一些构建规则,程序还是会执行成功
> make 有自动推导的能力，不会完全依赖 makefile。

假设目录中只有.c文件,没有 .o 文件,
```bash
calc:add.o  div.o  main.o  mult.o  sub.o
        gcc  add.o  div.o  main.o  mult.o  sub.o -o calc
```

依赖中所有的 .o文件在本地项目目录中是不存在的, 并且没有规则用来生成这些依赖文件
这时 make 会使用内部默认的构造规则(`cc -c`)先将这些依赖文件生成出来, 然后在执行规则中的命令, 最后生成目标文件 calc。

---

# 三、变量

> makefile中的变量分为三种：自定义变量，预定义变量和自动变量。


---
## 1.自定义变量

> makefile 中的变量是没有类型的，直接创建变量然后给其赋值就可以了。
> 其操作类似于shell

```bash
#定义变量并赋值
obj=add.o  div.o  main.o  mult.o  sub.o

#取变量值
$(obj)
```


通过自定义变量我们可以有新的写法

```bash
obj=add.o  div.o  main.o  mult.o  sub.o
target=calc
$(target):$(obj)
        gcc  $(obj) -o $(target)
```

---

## 2.预定义变量

> Makefile 中有一些已经定义的变量，用户可以直接使用这些变量，不用进行定义。
> 预定义变量的名字一般都是大写的

| 变量名      | 含义                     | 默认值            |
|------------|--------------------------|------------------|
| AR         | 生成静态库库文件的程序名称    | ar               |
| AS         | 汇编编译器的名称            | as               |
| CC         | C 语言编译器的名称          | cc               |
| CPP        | C 语言预编译器的名称        | $(CC) -E         |
| CXX        | C++ 语言编译器的名称        | g++              |
| FC         | FORTRAN 语言编译器的名称    | f77              |
| RM         | 删除文件程序的名称          | rm -f            |
| ARFLAGS    | 生成静态库库文件程序的选项    | 无默认值          |
| ASFLAGS    | 汇编语言编译器的编译选项      | 无默认值          |
| CFLAGS     | C 语言编译器的编译选项      | 无默认值          |
| CPPFLAGS   | C 语言预编译的编译选项      | 无默认值          |
| CXXFLAGS   | C++ 语言编译器的编译选项    | 无默认值          |
| FFLAGS     | FORTRAN 语言编译器的编译选项 | 无默认值          |


由此我们可以进一步优化上述代码

```bash
obj=add.o  div.o  main.o  mult.o  sub.o
target=calc
CFLAGS=-O3 #编译器代码优化等级(最高)
$(target):$(obj)
        $(CC)  $(obj) -o $(target) $(CFLAGS)
```

---

## 3.自动变量

> 自动变量用来代表这些规则中的目标文件和依赖文件，并且它们`只能在规则的命令中使用`。

| 变量 | 含义                                             |
|------|--------------------------------------------------|
| $*   | 表示目标文件的名称，不包含目标文件的扩展名             |
| $+   | 表示所有的依赖文件，这些依赖文件之间以空格分开，按照出现的先后顺序，可能包含重复的依赖文件 |
| $<   | 表示依赖项中第一个依赖文件的名称                     |
| $?   | 依赖项中，所有比目标文件时间戳晚的依赖文件，依赖文件之间以空格分开 |
| $@   | 表示目标文件的名称，包含文件扩展名                    |
| $^   | 依赖项中，所有不重复的依赖文件，这些文件之间以空格分开  |

较为常用的由 \$^ ,\$<,\$@

由以上也可进行优化

```bash
calc:add.o  div.o  main.o  mult.o  sub.o
		gcc $^ -o $@ 	
```

---
# 四、模式匹配

先观察以下makefile文件
```bash
calc:add.o  div.o  main.o  mult.o  sub.o
        gcc  add.o  div.o  main.o  mult.o  sub.o -o calc
# 语法格式重复的规则, 将 .c -> .o, 使用的命令都是一样的 gcc *.c -c
add.o:add.c
        gcc add.c -c

div.o:div.c
        gcc div.c -c

main.o:main.c
        gcc main.c -c

sub.o:sub.c
        gcc sub.c -c

mult.o:mult.c
        gcc mult.c -c
```

发现规则2-规则6几乎是重复做同样的事
解决冗余可以将规则像以下这样写,这种操作就是模式匹配


```bash
%.o:%.c            # %是通配符,类似于*
		gcc $< -c
```

---

# 五、函数

> makefile中有函数并且所有的函数都是有返回值的。makefile中函数的格式和C/C++中函数也不同，其写法是这样的
>  `$(函数名 参数1, 参数2, 参数3, ...)`
>  主要目的是让我们能够快速方便的得到函数的返回值。
> 主要介绍两个 makefile 中使用频率比较高的函数：`wildcard`和`patsubst`。

---
## 1.wildcard

> 主要作用是获取指定目录下指定类型的文件名，其返回值是以空格分割的、指定目录下的所有符合条件的文件名列表。

```bash
# 函数原型为:
# 该函数的参数只有一个, 但是这个参数可以分成若干个部分, 通过空格间隔
$(wildcard PATTERN...)
	参数:指定某个目录, 搜索这个路径下指定类型的文件，比如： *.c
```

 - 参数功能
  	1.PATTERN 指的是某个或多个目录下的对应的某种类型的文件
  	比如`当前目录下的.c文件`可以写成 `*.c`
  	2.可以指定多个目录，每个路径之间使用`空格间隔`
 - 返回值
	得到的若干个文件的文件列表， 文件名之间使用空格间隔

eg:

```bash
# 分别搜索三个不同目录下的 .c 格式的源文件
src = $(wildcard /home/robin/a/*.c /home/robin/b/*.c *.c)  # *.c == ./*.c
# 返回值: 得到一个大的字符串, 里边有若干个满足条件的文件名, 文件名之间使用空格间隔
/home/robin/a/a.c /home/robin/a/b.c /home/robin/b/c.c /home/robin/b/d.c e.c f.c
```

---

## 2.patsubst

> 按照指定的模式替换指定的文件名的后缀

```bash
# 函数原型为
# 有三个参数, 参数之间使用 逗号间隔
$(patsubst <pattern>,<replacement>,<text>)
```

参数功能

 - pattern: 这是一个模式字符串, 需要指定出要被替换的文件名中的后缀是什么
 
 - [ ] 文件名和路径不需要关心, 因此使用 % 表示即可 [通配符]
 - [ ] 在通配符后边指定出要被替换的后缀, 比如: %.c, 意味着 .c的后缀要被替换掉
 
 - replacement: 这是一个模式字符串, 指定参数pattern中的后缀最终要被替换为什么
 
 - [ ] 还是使用 % 来表示参数pattern 中文件的路径和名字
 - [ ] 在通配符 % 后边指定出新的后缀名, 比如: %.o 这表示原来的后缀被替换为 .o
 - text
 该参数中存储这要被替换的原始数据
 
 - 返回值
 函数返回被替换过后的字符串。




```bash
# eg:把变量 src 中的所有文件名的后缀从 .cpp 替换为 .o
src = a.cpp b.cpp c.cpp e.cpp
obj = $(patsubst %.cpp, %.o, $(src)) 
# obj 的值为: a.o b.o c.o e.o
```


---

# 六、makefile的编写

> 基于一个简单的项目, 演示一下编写一个makefile从不标准到标准的过程。

```bash
# 项目目录结构
.
├── add.c
├── div.c
├── head.h
├── main.c
├── mult.c
└── sub.c
# 需要编写makefile对该项目进行自动化编译
```

---

## 版本1

```bash
calc:add.c  div.c  main.c  mult.c  sub.c
        gcc add.c  div.c  main.c  mult.c  sub.c -o calc
```

> 优点:书写简单 
> 缺点：只要依赖中的某一个源文件被修改，`所有`的源文件都需要被重新编译,效率低
> 改进方式：提高效率，修改哪一个源文件, 哪个源文件被重新编译, 不修改就不重新编译

---
## 版本2

```bash
# 默认所有的依赖都不存在, 需要使用其他规则生成这些依赖
# 因为 add.o 被更新, 需要使用最新的依赖, 生成最新的目标
calc:add.o  div.o  main.o  mult.o  sub.o
        gcc  add.o  div.o  main.o  mult.o  sub.o -o calc

# 如果修改了add.c, add.o 被重新生成
add.o:add.c
        gcc add.c -c

div.o:div.c
        gcc div.c -c

main.o:main.c
        gcc main.c -c

sub.o:sub.c
        gcc sub.c -c

mult.o:mult.c
        gcc mult.c -c
```

> 优点：相较于版本1效率提升
缺点：规则比较冗余, 需要精简
改进方式：在 makefile 中使用变量 和 模式匹配

---

## 版本3

```bash
obj=add.o  div.o  main.o  mult.o  sub.o
target=calc

$(target):$(obj)
        gcc $(obj)  -o $(target)

%.o:%.c
        gcc $< -c
```

> 优点：简洁
缺点：变量 obj 的值需要手动的写出来, 当需要编译的项目文件很多，用手写不现实
改进方式：在makefile中使用函数

---

## 版本4

```bash
# 使用函数搜索当前目录下的源文件 .c
src=$(wildcard *.c)
# 将源文件的后缀替换为 .o
obj=$(patsubst %.c, %.o, $(src))
target=calc

$(target):$(obj)
        gcc $(obj)  -o $(target)

%.o:%.c
        gcc $< -c
```

> 优点：解决了自动加载项目文件的问题，解放了双手
缺点：没有文件删除的功能，不能删除项目编译过程中生成的目标文件（*.o）和可执行程序
改进方式: 在makefile文件中添加新的规则用于删除生成的目标文件（*.o）和可执行程序

---

## 版本5

```bash
src=$(wildcard *.c)
obj=$(patsubst %.c, %.o, $(src))
target=calc
$(target):$(obj)
        gcc $(obj)  -o $(target)

%.o:%.c
        gcc $< -c

# 添加规则, 删除生成文件 *.o 可执行程序
# 这个规则比较特殊, clean根本不会生成, 这是一个伪目标
clean:
        rm $(obj) $(target)
```

> 优点: 添加了新的规则（16行）用于文件的删除, 直接 make clean 就可以执行规则中的删除命令了
缺点: goto flag;
改进方式: 在makefile文件中声明 clean是一个伪目标，让 make 放弃对它的时间戳检测。

flag:

正常情况下这个版本的makefile是可以正常工作的，如果在这个项目目录中添加一个叫做clean的文件（和规则中的目标名称相同），再进行 make clean发现这个规则就不能正常工作了。

> 这个问题的关键在于 clean是一个伪目标, 不对应任何实体文件
>  在前边关于文件时间戳更新问题的时说过，如果目标不存在的命令肯定被执行， 如果目标文件存在了就需要比较规则中目标文件和依赖文件的时间戳，满足条件才执行规则的命令，否则不执行。
解决这个问题需要在 makefile 中声明 clean是一个伪目标，这样 make 就不会对文件的时间戳进行检测，规则中的命令也就每次都会被执行了。
在 makefile 中声明一个伪目标需要使用 `.PHONY `关键字, 声明方式为: `.PHONY:伪文件名称`

---
## 版本6

```bash
src=$(wildcard *.c)
obj=$(patsubst %.c, %.o, $(src))
target=calc

$(target):$(obj)
        gcc $(obj)  -o $(target)

%.o:%.c
        gcc $< -c
# 声明clean为伪文件
.PHONY:clean
clean:
        # shell命令前的 - 表示强制这个指令执行, 如果执行失败也不会终止
        -rm $(obj) $(target) 
        echo "test, test"
```
说一下` - ` :
如果第一条命令执行失败,且没有加`-`,那么第二条命令就不会执行
如果加上`-`,不论执行失败或者成功都会继续将命令执行下去

---

# 七、一道例题

```bash
# 目录结构
.
├── include
│   └── head.h	==> 头文件, 声明了加减乘除四个函数
├── main.c		==> 测试程序, 调用了head.h中的函数
└── src
    ├── add.c	==> 加法运算
    ├── div.c	==> 除法运算
    ├── mult.c  ==> 乘法运算
    └── sub.c   ==> 减法运算
```

```bash
# 最终的目标名 app
target = app
# 搜索当前项目目录下的源文件
src=$(wildcard *.c ./src/*.c)
# 将文件的后缀替换掉 .c -> .o
obj=$(patsubst %.c, %.o, $(src))
# 头文件目录
include=./include

# 第一条规则
# 依赖中都是 xx.o yy.o zz.o
# gcc命令执行的是链接操作
$(target):$(obj)
        gcc $^ -o $@

# 模式匹配规则
# 执行汇编操作, 前两步: 预处理, 编译是自动完成
%.o:%.c
        gcc $< -c -I $(include) -o $@

# 添加一个清除文件的规则
.PHONY:clean
clean:
        -rm $(obj) $(target) -f
```

---