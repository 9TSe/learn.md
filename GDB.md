---
title: GDB
date: 2023-10-22 09:51:47
tags:
- GDB调试
categories:
- Linux
cover: /pic/3.png
---


---

> gdb 是由 GNU 软件系统社区提供的调试器，同 gcc 配套 组成了一套完整的开发环境，可移植性好，支持非常多的体系结构并被移植到各种系统中（包括各种类 Unix 系统与 Windows 系统里的 MinGW 和 Cygwin 
> 此外，除了 C ，gcc/gdb 还支持包括 C++、Objective-C、Ada 和 Pascal 等语言后端的编译和调试。 gcc/gdb 是 Linux 和许多类 Unix 系统中的标准开发环境，Linux 内核也是专门针对 gcc 进行编码的。

GDB 是一套字符界面的程序集，可以使用命令 gdb 加载要调试的程序。

---
#  一、调试准备

> 项目程序如果是为调试而编译时， 必须要打开调试选项(`-g`)。
> -g选项的作用是在可执行文件中加入源代码信息，比如可执行文件中第几条机器指令对应源代码的第几行，但并不是把整个源文件嵌入到可执行文件中，所以在调试时必须保证gdb能找到源文件。
习惯上如果是c程序就使用gcc编译,  c++程序使用g++编译, 编译命令中添加上边提到的参数即可。
> 另外还有一些可选项，eg: 在尽量不影响程序行为的情况下关掉编译器的优化选项(`-O0`)，`-Wall`选项打开所有 warning，可以发现许多问题，避免一些不必要的 bug。


假设有一个文件 args.c, 要对其进行gdb调试，编译的时候必须要添加参数 -g，加入了源代码信息的可执行文件比之前要大。

```bash
gcc -g args.c -o app
#不加 -g 参数就无法进行gdb调试
```

---

# 二、启动和退出gdb

---

## 1.启动gdb

> gdb是一个用于应用程序调试的进程, 需要先将其打开, 一定要注意 gdb进程启动之后, 需要被调试的应用程序是没有执行的。
> 打开Linux终端，切换到要调试的可执行程序所在路径，执行如下命令就可以启动 gdb了。

```bash
gdb 可执行程序的名字
```

---

## 2.命令行传参

> 有些程序在启动时需传递命令行参数，如果要调试这类程序，
> 这些命令行参数必须要在应用程序启动之前通过调试程序的gdb进程传递进去。

```bash
# 1.编译出可执行程序
gcc args.c -o app -g

# 2.启动gdb进程, 指定需要gdb调试的应用程序名称
gdb app

# 3.在启动app之前设置命令行参数。
#gdb中设置参数的命令叫做set args ...，查看设置的命令行参数命令是 show args。 
#语法格式如下：
# 设置参数
set args 参数1 参数2 .... ...
#查看设置的命令行参数
show args
```


eg:

```bash
#非gdb调试命令行传参
#argv[0] == ./app， argv[1] == "1"  ...  argv[5] == "5"
./app 1 2 3 4 5	 #将数据传送给main函数


#使用 gdb 调试
set args 1 2 3 4 5
#查看设置的命令行参数
show args
```

---

## 3.gdb中启动程序

> 在gdb中启动要调试的应用程序有两种方式, 一种是使用`run`命令,
> 另一种是使用`start`命令启动。在整个 gdb 调试过程中, 启动应用程序的命令`只能使用一次`。

 - run: 可以缩写为 `r`, 如果程序中设置了断点会停在第一个断点的位置, 如果没设置断点, 程序就执行完了
 - start: 启动程序, 最终会阻塞在main函数的第一行，等待输入后续其它 gdb 指令
 如想让程序start后继续运行, 或在断点处继续运行，可使用 `continue`命令, 可以简写为 `c`

```bash
# 1.
r

# 2.
run  

# 3.
start
continue
c
```



---

## 4.退出gdb

> 退出gdb调试, 就是终止 gdb 进程, 需要使用 quit命令, 可以缩写为 q

```bash
# 1.
quit

# 2.
q
```

---

# 三、查看代码

> 因为gdb调试没有IDE那样的完善的可视化窗口界面，给调试的程序打断点又是调试之前必做的工作。
> 因此gdb提供了查看代码的命令，这样就可以定位要调试的代码行的位置了。
查看代码的命令叫做`list`可以缩写为 `l`, 通过这个命令我们可以查看项目中任意一个文件中的内容，并且还可以通过文件行号，函数名等方式查看。

---

## 1.当前文件

> 一个项目中一般有很多源文件, 默认通过list查看到代码信息位于程序入口函数main对应的的文件中。
> 因此如果不进行文件切换, main函数所在的文件就是当前文件, 如果进行了文件切换, 切换到哪个文件哪个文件就是当前文件。

```bash
# list==l
# 从第一行开始显示
list 

# 列值这行号对应的上下文代码, 默认情况下只显示10行内容
list 行号

# 显示这个函数的上下文内容, 默认显示10行
list 函数名
```

若想要向下继续查看可以按 `Enter` (再次执行上一次执行的gdb命令)

---

## 2.切换文件

> 在查看文件内容的时，需要进行文件切换
> 我们只需要在list命令后将要查看的文件名指定出来就可以了，切换命令执行完毕之后，这个文件就变成了当前文件。

```bash
# 切换到指定的文件，并列出这行号对应的上下文代码, 默认显示10行
l 文件名:行号

# 切换到指定的文件，并显示这个函数的上下文内容, 默认显示10行
l 文件名:函数名
```

---

## 3.设置显示行数

> 默认list只能一次查看10行代码, 如果想显示更多, 可通过`set listsize`设置
>  同样如果想查看当前显示行数可通过 `show listsize`查看
>  这里的listsize可以简写为 `list`。

```bash
#listsize == list
set listsize 行数

show listsize
```

---

# 四、断点操作

> 想要通过gdb调试某一行或者得到某个变量在运行状态下的实际值，就需要在在这一行设置断点
> 程序指定到断点的位置就会阻塞，我们就可通过gdb的调试命令得到我们想要的信息了。
设置断点的命令叫做`break`可以缩写为`b`。

---

## 1.设置断点

> 断点的设置有两种方式
> `常规断点`，程序只要运行到这个位置就会被阻塞
> `条件断点`，只有指定的条件被满足了程序才会在断点处阻塞。
调试程序的断点可以设置到某个具体的行, 也可以设置到某个函数上

 - 设置普通断点到当前文件

```bash
b 行号
b 函数名   #停在函数的第一行
```

 - 设置普通断点到非当前文件上
 ```bash
b 文件名:行号
b 文件名:函数名
```

- 设置条件断点

```bash
#通常情况下, 在循环中条件断点用的比较多
b 行数 if 变量名==某个值
```

---

## 2.查看断点

> 可以通过 `info break`命令查看设置的断点信息，其中info可以缩写为`i`
> 即 `i b`

```bash
i b
Num     Type           Disp Enb Address            What
1       breakpoint     keep y   0x0000000000400cb5 in main() at test.cpp:12
2       breakpoint     keep y   0x0000000000400cbd in main() at test.cpp:13
3       breakpoint     keep y   0x0000000000400cec in main() at test.cpp:18
4       breakpoint     keep n   0x00000000004009a5 in insertionSort(int*, int) 
                                                   at insert.cpp:8
5       breakpoint     keep y   0x0000000000400cdd in main() at test.cpp:16
6       breakpoint     keep y   0x00000000004009e5 in insertionSort(int*, int) 
                                                   at insert.cpp:16
```

 - `Num`: 断点的编号, 删除或设置断点状态时候需要使用
 - `Enb`: 断点的状态, y表示断点可用, n表示断点不可用
 - `What`: 描述断点被设置在了哪个文件的哪一行或者哪个函数上



---

## 3.删除断点

> 如果确定设置的某个断点不再使用, 可将其删除
>  删除命令是 `delete 断点编号`, 这个delete可以简写为`del`也可以再简写为`d`。
删除断点的方式有两种: 删除(一个或者多个)指定断点或者删除一个连续的断点区间，

```bash
d 1          # 删除第1个断点
d 2 4 6      # 删除第2,4,6个断点
d 1-5		 # 删除第1至第5个断点
```

---

## 4.设置断点状态


> 如果某断点只是临时不需要，可将其设置为不可用状态
> 设置命令为`disable 断点编号`,可简写为`dis`
> 
> 当需要的时候再将其设置回可用状态
> 设置命令为 `enable 断点编号`,可简写为`ena`

```bash
dis 2 4		# 设置第2, 第4 个断点无效
dis 5-8		# 设置 第5,6,7,8个 断点无效
ena 2 4		# 设置第2, 第4个断点有效
ena 5-7		# 设置第5,6,7个断点有效
```
用法与 del 类似


---

# 五、调试命令

---

## 1.继续运行gdb

> 如果调试的程序被断点阻塞,又想程序继续执行
> 这时候就可以使用`continue`命令。程序会继续运行, 直到遇到下一个有效的断点。
> continue可以缩写为 `c`。

```bash
continue
```

---

## 2.手动打印信息

> 当程序被某个断点阻塞之后, 可通过一些命令打印变量的名字或者变量的类型
> 并且还可以跟踪打印某个变量的值。

---
### ①打印变量的值

> 在gdb调试的时候如果需要打印变量的值， 命令是 `print`, 可缩写为 `p`。
> 如果打印的变量是`整数`还可以指定输出的整数的格式, 格式化输出的整数对应的字符表如下：

| 格式化字符(/fmt) | 说明                                      |
|-----------------|-------------------------------------------|
| /x              | 以十六进制的形式打印出整数                  |
| /d              | 以有符号、十进制的形式打印出整数             |
| /u              | 以无符号、十进制的形式打印出整数             |
| /o              | 以八进制的形式打印出整数                   |
| /t              | 以二进制的形式打印出整数                   |
| /f              | 以浮点数的形式打印变量或表达式的值           |
| /c              | 以字符形式打印变量或表达式的值               |


p的语法格式

```bash
p 变量名

p/fmt 变量名
```

eg:

```bash
# 10进制
p i  

# 16进制
p/x i    

# 8进制
p/o i 
```

---

### ②打印变量类型

> 使用命令`ptype`

```bash
ptype 变量名
```

---

## 3.自动打印信息
---

### ①设置变量名自动显示

> 和 print 命令一样，display 命令也用于调试阶段查看某个变量或表达式的值
> 它们的区别是，
> 使用 `display` 命令查看变量或表达式的值，每当程序暂停执行（例如单步执行）时，GDB 调试器都会`自动打印`出来
> 而 `print` 命令则不会。
> 因此，想频繁查看某个变量或表达式的值从而观察它的变化情况时， display 命令可以一劳永逸。display 命令`没有缩写`形式

```bash
# 在变量的有效取值范围内, 自动打印变量的值(设置一次, 以后就会自动显示)
display 变量名

# 以指定的整形格式打印变量的值, 关于 fmt 的取值, 参考 print 命令
display/fmt 变量名
```

---

### ②查看自动显示列表

> 对于使用 display 命令查看的目标变量或表达式，都会被记录在一张列表（称为自动显示列表）中。通过执行`info dispaly`命令，可以打印出这张表

```bash
info display

Auto-display expressions now in effect:
Num Enb Expression
1:   y  i
2:   y  array[i]
3:   y  /x array[i]
```

 - `Num` : 变量或表达式的编号，GDB 调试器为每个变量或表达式都分配有唯一编号
- `Enb` : 表示当前变量（表达式）处于激活状态or禁用状态
如果处于激活状态（用 y 表示），则每次程序停止执行，该变量的值都会被打印出来；
如果处于禁用状态（用 n 表示），则该变量（表达式）的值不会被打印。
 - `Expression` ：被自动打印值的变量或表达式的名字。

---

### ③取消自动显示

> 对于不需要再打印值的变量或表达式，可以将其删除或者禁用。

```bash
undisplay num [num1 ...]
undisplay num1-numN

delete display num [num1 ...]
delete display num1-numN
```

 - 如果不想删除自动显示的变量, 也可以禁用自动显示列表中处于激活状态下的变量或表达式
 

```bash
disable display num [num1 ...]

disable display num1-numN
```

- 当需要启用自动显示列表中被禁用的变量或表达式时

```bash
enable  display num [num1 ...]
enable display num1-numN
```

---

## 3.单步调试

> 当程序阻塞到某个断点上之后, 可以通过以下命令对程序进行单步调试

---

### ①step

> `step`命令可以缩写为`s`, 命令执行一次代码向下执行一行
> 如果这一行是函数调用，那么程序会进入到函数体内部。

```bash
#step==s
step
```

---


### ②finish

> 如果通过 `s` 单步调试进入到函数内部, 想要跳出这个函数体， 可以执行`finish`命令
> 如果想要跳出函数体必须要保证函数体内不能有有效断点，否则无法跳出。

```bash
finish
```

---

### ③next

> `next`命令和`step`命令功能是相似的
只是在使用next调试程序的时候`不会进入到函数体内部`，next可以缩写为 `n`

```bash
next
```

---

### ④until

>  `until` 命令可直接跳出某个循环体，提高调试效率
>  如果想直接从循环体中跳出, 必须要满足以下的条件，否则命令不会生效：

 1. 要跳出的循环体内部不能有有效的断点
 2. 必须要在循环体的开始/结束行执行该命令

---

## 4.设置变量值

> 在调试程序的时候, 我们需要在某个变量等于某个特殊值的时候查看程序的运行状态
> 但通过程序运行让变量等于这个值又非常困难, 这种情况下就可以在 gdb 中直接对这个变量进行值的设置, 或在单步调试的时候通过设置循环因子的值直接跳出某个循环, 
值设置的命令格式为: `set var 变量名=值`

```bash
#可以在循环中使用, 直接设置循环因子的值
set var 变量名=值
```

---