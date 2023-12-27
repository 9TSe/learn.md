---
title: tips
date: 2023-10-21 21:08:43
tags:
categories:
- Linux
cover: /pic/3.png
---

---
# 一、Linux目录结构
---

![在这里插入图片描述](/img/5.5.png)

>Linux中 根目录和子目录结构是相对固定的,不同的目录功能也是固定的

|目录名称|功能  |
|--|--|
| bin | 二进制文件目录, 存储了可执行程序, 命令对应的可执行程序都在这个目录中 |
| sbin |super binary, root用户使用的一些二进制可执行程序  |
| etc | 配置文件目录, 系统的或者用户自己安装的应用程序的配置文件都存储在这个目录中 |
| lib | library, 存储了一些动态库和静态库，给系统或者安装的软件使用 |
| media | 挂载目录, 挂载外部设备，比如: 光驱, 扫描仪 |
| mnt | 临时挂载目录, 比如我们可以将U盘临时挂载到这个目录下 |
| proc |  内存使用的一个映射目录, 给操作系统使用的 |
| tmp |临时目录, 存放临时数据, 重启电脑数据就被自动删除了  |
| boot | 存储了开机相关的设置 |
|home  | 存储了普通用户的家目录，家目录名和用户名相同 |
| root | root用户的家目录 |
| dev | device , 设备目录, Linux中一切皆文件, 所有的硬件会抽象成文件存储起来，比如：键盘， 鼠标 |
| lost+found | 一般时候是空的, 电脑异常关闭/崩溃时用来存储这些无家可归的文件, 用于用户系统恢复 |
|opt  | 第三方软件的安装目录 |
| var | 存储了系统使用的一些经常会发生变化的文件， 比如：日志文件 |
| usr | unix system resource, 系统的资源目录 |
|/usr/bin | 可执行的二进制应用程序|
|/usr/games  | 游戏目录  |
| /usr/include |  包含的标准头文件目录  |
| /usr/local | 和opt目录作用相同, 安装第三方软件  |

---

# 二、命令解析器

---

## 1.命令提示行
![在这里插入图片描述](/img/5.6.png)

root : 表示当前登录用户的用户名

@ : 在(相当于一个分隔符)

sewerperson : 即自定义的主机名

"~" :表示当前的家目录 (普通用户:/home/用户名 --- root用户 : /root

![在这里插入图片描述](/img/5.7.png)

shcode : 表示当前用户所在的目录

"#" : 表示当前用户为root用户

"$" : 表示当前用户为普通用户

---

## 2.工作原理

命令解析器在Linux操作系统中就是一个进程(运行的应用程序)

> 有bash和shell
> unix版本时的命令解析器为shell
> Linux版本时有人(Bourne)进行了更改 取名为bash(Bourne Again SHell)即shell的更新版 

在Linux操作系统中默认使用的命令解析器是 bash, 当然也同样支持使用sh。
当我们打开窗口,输入指令,按下回车键,此时命令解析器就开始了工作


```cpp
Linux中有 PATH 环境变量,存储了一些系统目录 (window : path)
命令解析器依次搜索PATH路径中的目录,检查是否有对应指令
```

![在这里插入图片描述](/img/5.8.png)

---

## 3.命令行快捷键
|快捷键| 功能 | 备注|
|--|--|--|
| Tab |命令自动补全  | /|
| Ctrl+p |显示输入的上一个历史命令  | 也可以使用 ↑键 |
| Ctrl+n |显示输入的下一个历史命令  |  也可以使用 ↓键|
| Ctrl+a | 光标移动命命令行首 | 也可以使用 Home键 |
| Ctrl+e | 光标移动命命令行尾 |  也可以使用 End键|
| Ctrl+u | 删除光标前的部分字符串 |/  |
| Ctrl+k | 删除光标后的部分字符串 | / |

---
# 三、文件管理命令

---
## 1.cd
对于切换为根目录的三种方法

```bash
cd
cd ~
cd /home/用户名
```

对于在两个较深且复杂的目录下一直切换

```bash
cd /目录1
cd /目录2
cd -  #可以一直在两个目录之间进行切换
```

---

## 2.ls

```bash
ls [args]        #查看当前目录
ls [args] 目录名  #查看指定目录
ls [args] 文件名  #查看文件信息
```


ls -l :(list)显示文件详细信息

![在这里插入图片描述](/img/5.9.png)

查询文件详细信息有简单写法 : ll

> 有的版本ll等价于 ls -l
> 有的则是 ls -laF

---
ls -a : (all) 显示所有文件,包括隐藏文件
默认情况下,隐藏的文件不会被显现出来
文件名前有一个 . 就说明文件有隐藏属性

![在这里插入图片描述](/img/5.10.png)

---

ls -h :(human) 人性化的将ls -l中显示文件大小的数据显示出来
原本的默认大小单位是字节(byte)
![在这里插入图片描述](/img/5.11.png)

---

ls -F : 将文件类型的前面加上/ 
![在这里插入图片描述](/img/5.12.png)

---

## 3.文件详细信息
![在这里插入图片描述](/img/5.13.png)
![在这里插入图片描述](/img/5.14.png)
`如果文件名所表示的是一个目录,那么其大小只是表示目录的大小,而并非其子目录/文件的和`

文件类型又分为7种
|-| 普通的文件, 在Linux终端中没有执行权限的为白色, 压缩包为红色, 可执行程序为绿色字体 |
|--|--|
| d | 目录(directory), 在Linux终端中为蓝色字体, 如果目录的所有权限都是开放的, 有绿色的背景色 |
|  l| 软链接文件(link), 相当于windows中的快捷方式, 在Linux终端中为淡蓝色(青色)字体 |
| c |  字符设备(char)(键盘..), 在Linux终端中为黄色字体|
|  b|块设备(block)(u盘,磁盘...), 在Linux终端中为黄色字体  |
|  p| 管道文件(pipe), 在Linux终端中为棕黄色字体 |
|  s|  本地套接字文件(socket), 在Linux终端中为粉色字体 |

![在这里插入图片描述](/img/5.15.png)


## 4.目录的创建和删除


```bash
mkdir 目录名     #单层目录的创建
mkdir -p 目录名  #多层目录的创建
```


```bash
rmdir 目录名     #只能删除空目录(如果目录种有子文件/目录就无法删除)
rm 文件          #删除文件
rm -r 目录名     #(递归)删除目录
rm -i           #删除时给提示
rm -f           #强制删除文件,没有提示且不能恢复
```

---
## 5.cp

```bash
cp 要拷贝的文件 得到的文件
cp 文件A 文件B 
如果文件B不存在就创建文件B
如果文件B存在就被文件A覆盖
```

```bash
cp -r 目录A 目录B
1:目录B不存在就创建且拷贝
2:目录B存在,A目录就会将自己全部拷贝到B的子目录中
```
---
## 6.mv

```bash
#文件/目录的移动
mv 目录/文件 目录
mv A B		#A既可以是文件也可以是目录,B必须是目录且必须存在
```

```bash
#文件/目录改名
mv 文件名/目录名 文件名/目录名
mv A B		#A可以是文件也可以是目录,B是必须不存在的
```

```bash
#文件覆盖
mv 存在的文件 存在的文件
mv A B		#AB必须都是文件且存在,A的内容覆盖B,A被删除
```

---

## 7.查看文件内容

> 终端是有缓存的,因此显示的字节也会受限,需要用合适的命令

```bash
cat 文件名 		#直接显示所有

more 文件名		#可用翻屏查看
#more快捷键 回车:下一行 空格:向下一屏 b:向上一屏 q:退出
less 文件名		#同more类似,多出来可以用上下箭头滚动

head 文件名		#默认显示文件前十行
head -行数 文件名 #显示文件前n行
tail 文件名		#默认显示文件尾十行
tail -行数 文件名 #显示文件后n行
```

---

## 8.链接的创建

```bash
#软链接的创建
ln -s 源文件的路径 软链接的路径(名字)
#注意:源文件的路径最好是绝对路径,否则可能会:软链接的移动使得软链接无法使用


#硬链接的创建
ln 源文件 硬链接的路径(名字)
#硬链接和软链接不同, 它是通话文件名直接找对应的硬盘地址, 而不是基于路径
#因此 源文件使用相对路径即可,无需为其制定绝对路径
#目录是不允许创建硬链接的
```

---

## 9.更改文件权限

```bash
#文字设定法
chmod who [+|-|=] mod 文件名
	- who:
		u: user  -> 文件所有者
		g: group -> 文件所属组用户
		o: other -> 其他
		a: all, 以上是三类人 u+g+o
	- 对权限的操作:
		+: 添加权限
		-: 去除权限
		=: 权限的覆盖
	- mod: 权限
		r: read, 读
		w: write, 写
		x: execute, 执行
		-: 没有权限


#数字设定法
chmod [+|-|=] mod 文件名
	权限操作中=可以不写
	- mod:
		4: read, r
		2: write, w
		1: execute , x
		0: 没有权限
```

---
## 10.修改文件所有者/组

```bash
#修改文件所有者
sudo chown 新的所有者 文件名			#只修改文件所有者
sudo chown 新的所有者:新的组名 文件名  #修改文件所有者和所属组

#修改文件所属组
sudo chgrp 新的组 文件名
```

对于普通用户无法使用sudo的解决方式

![在这里插入图片描述](/img/5.16.png)
![在这里插入图片描述](/img/5.17.png)


---
## 11.其他命令

```bash
tree [目录名] -L n  #树状显示(目录)n层

touch 文件名 #如果文件已经存在则只会更新文件的修改日期

which 命令 #查看要执行命令的实际路径
#该命令只能查看非内建的shell指令所在的实际路径, 有些命令是直接写到内核中的, 无法查看

>  #将文件内容写入到指定文件中, 如果文件中已有数据, 则会使用新数据覆盖原数据
>> #将输出的内容追加到指定的文件尾部

id 用户名 #显示其id,组id
```

---

# 四、用户管理命令

---
## 1.用户的切换

```bash
su 用户名 	#此时切换工作目录并不会发生变化
su - 用户名  #此时切换会切换为当前用户的家目录

如果A->B用户,此时又想切回A
exit
```

---
## 2.添加删除用户

```bash
#添加用户
sudo adduser 用户名 #centos和Ubuntu通用
sudo useradd 用户名 #centos
sudo useradd -m -s /bin/bash  用户名 #Ubuntu
```
检测是否真的添加成功

> 1.在home下观察是否有新用户名目录
> 2.在etc/passwd文件中观察(vim)

![在这里插入图片描述](/img/5.18.png)
`用户名:加密后的密码:用户id:所属组id:用户家目录:用户默认使用的命令解析器`

---

```bash
#删除用户
sudo userdel 用户名 -r #删除用户的同时删除其家目录 (centos && Ubuntu 支持)

sudo deluser 用户名    #不能添加参数 -r,并且删除后家目录依然存在(Ubuntu特有)
#若要删除家目录
sudo rm /home/用户名 -r 
```

---

## 3.添加删除用户组

```bash
sudo groupadd 组名 #添加组
sudo groupdel 组名 #删除组
```

> 可通过/etc/group文件检验(vim)

![在这里插入图片描述](/img/5.19.png)

**最后的数字是用户组的id**

```bash
在Ubuntu中可以使用 addgroup/groupadd 和 delgroup/groupdel
在CentOS中只能使用 groupadd 和 groupdel
可通过 which 命令查看该Linux版本是否支持使用该命令了。
```

---
## 4.更改用户所属组

```bash
#增加用户时直接指定组
useradd -g 组 用户名 #如果想加入多个组,组后跟","

#更改用户所属组
usermod -g 组 用户名 #更改
```

---
## 5.修改密码

```bash
passwd #修改自己的用户密码
sudo passwd 用户名 #修改其他用户的密码
```