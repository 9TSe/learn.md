---
title: 文件
date: 2023-10-21 21:00:37
tags:
- 文件IO
categories: 
- C语言
cover: /pic/1.png
---


# 一、什么是文件
---
> 磁盘上的文件就是文件
> 但在程序设计中,我们一般谈的文件有两种:程序文件、数据文件(功能角度分类)

---

## 1.程序文件

> 包括源文件(后缀为.c),目标文件(windows环境后缀为.obj),可执行程序(windows环境后缀为.exe)

---

## 2.数据文件

> 文件的内容不一定是程序,而是程序	运行时读写的数据,比如程序运行需要从中读取数据的文件
> 或者输出内容的文件


`本文主要以数据文件为中心进行简介`

---
## 3.文件名

文件名包含三个部分
> 文件路径+文件主干+文件后缀
> eg: C:\code\test.txt
> 路径: C:\code\
> 文件名主干: test
> 文件后缀: .txt

---

# 二、文件的打开和关闭
---
## 1.文件指针

> 每个被使用的文件都在内存开辟了一个相应的文件信息区
> 用来存放文件的相关信息(文件名,文件状态,文件当前位置等).
> 这些信息是保存在一个结构体变量中的.
> 该结构体系统声明取名为 FILE
>即 FILE 是一个结构体类型


具体内部实现如下(不同环境下,结构成员会有所差异))

```c
struct _iobuf
{
  //文件相关信息
  //
  //
};

typedef struct _iobuf FILE;
```

---

![](/img/6.1.png)

---

比如创建一个FILE*的指针变量

```c
FILE * pf; //pf即文件指针变量
```

---

## 2.文件的打开和关闭
---
### ①fopen


```c
#include<stdio.h>
.
FILE*fopen(const char* filename,const char* mode);
```

> 以mode的形式打开filename文件

其中mode主要分为以下几种
|文件使用方式|含义  | 如果指定文件不存在|
|--|--|--|
| "r"(只读) | 为了输入数据,打开一个已经存在的文本文件 | 出错 |
|  "w"(只写)| 为了输出数据,打开一个文本文件 |建立一个新的文件  |
|  "a"(追加)|  向文本文件尾添加数据|  建立一个新的文件|
| "rb" (只读)|为了输入数据,打开一个二进制文件  | 出错 |
| "wb"(只写) | 为了输出数据,打开一个二进制文件 | 建立一个新的文件 |
| "ab" (追加)| 向一个二进制文件尾添加数据 |出错  |
|  "r+"(读写)| 为了读和写,打开一个文本文件 | 出错 |
|  "w+"(读写)| 为了读和写,建立一个新文件 | 建立一个新的文件 |
|  "a+"(读写)| 打开一个文件,在文件尾进行读写 | 建立一个新的文件 |
|  "rb+"(读写)|  为了读和写,打开一个二进制文件| 出错 |
| "wb+"(读写) | 为了读和写,建立一个二进制文件 | 建立一个新的文件 |
|"ab+"(读写)  |打开一个二进制文件,在文件尾进行读和写  | 建立一个新的文件 |


---

以一段代码介绍fopen实际使用

```c
FILE * pf = fopen("test.dat","r");//test.dat 位于该项目同一文件内可以被打开,或创建于同一文件
                                  //但是并不代表只能打开同项目文件
FILE * pf = fopen("D:\\2023_code\\class\\test.dat","r");
                                  //注意是\\而不是\
                                  //给定一块指定的空间也可以打开或创建
                                  
if(pf == NULL)
{
  perror("fopen");  //如果打开失败返回错误信息
  return 1;
}
//处理文件

//关闭文件
fclose(pf);
pf = NULL;
```
---
### ②fclose

```c
#include<stdio.h>
.
int fclose(FILE* stream);
```
---

# 三、文件的顺序读写
---
## 1.读写函数表
| 功能 |函数名  |适用于 |
|--|--|--|
|  字符输入函数|fgetc  |所有输入流 |
| 字符输出函数 | fputc | 所有输出流|
| 文本行输入函数 | fgets |所有输入流 |
|文本行输出函数  | fputs |所有输出流 |
|  格式化输入函数| fscanf |所有输入流 |
| 格式化输出函数 | fprintf | 所有输出流|
|  二进制输入| fread |文件 |
| 二进制输出 |  fwrite| 文件|

---


## 2."输入","输出"是什么

![](/img/6.2.png)

---

## 3.流

> 流是一个高度抽象的概念

同样的我们以图的形式展现流

![](/img/6.3.png)


具体理解可能略微生硬
在fputc中我们可以在逐渐深入了解

---

## 4.fputc
---

```c
#include<stdio.h>
.
int fputc(int c,FILE*stream);
```


> 将一个字符c,输出到 stream(输出流) 中

---
举一个实例

![](/img/6.4.png)

函数运行后观察该项目文件夹

![](/img/6.n.png)

**文件夹内创建了我们输入的文件名,并向文件内输出了我们想要的结果**

---
**当然输出流可以改变,比如stdout(标准输出流--屏幕)**

![](/img/6.5.png)

---

##  5.fgetc
---


```c
#include<stdio.h>
.
int fgetc(FILE * stream);
```



> 从流中读取单字符
> 每使用一次向后偏移一次
> 若读取失败则返回EOF( -1 )

---
举个例子
![](/img/6.6.png)
先将要读取的文件加入字符

运行程序后

![](/img/6.7.png)

---

**同样的,输入流也可以不仅仅是文件,也可以是stdin(标准输入流---键盘)**

![](/img/6.8.png)
***输入后再打印***

---

## 6.fputs
---
```c
#include<stdio.h>
.
int fputs(const char * string, FILE * stream);
```


![](/img/6.9.png)



---

## 7.fgets
---
```c
#include<stdio.h>
.
char * fgets(char * string , int n ,FILE * stream);
```


> 从流中读取最多n个字符到string
> 返回string的地址

---
举个例子

![](/img/6.10.png)

在文件中提前放入字符串

然后读取打印

![](/img/6.11.png)

可以看到打印成功

但是之所以只打印了三个字符

`是因为需要预留一字节空间存储'\0'`

---

## 8.fprintf
---
```c
#include<stdio.h>
.
int fprintf(FILE * stream , const char * format [,argument]....);
```



```c
struct S
{
  char arr[10];
  int num;
  float sc;
};

int main()
{
  struct S s = {"abcdef",10,5.5f};

  //对格式化数据进行写文件
  FILE*pf = fopen("test.dat","w");
  if(pf == NULL)
  {
    perror("fopen");
    return 1;
  }
  
  //写文件
  fprintf(pf,"%s %d %f",s.arr,s.num,s.sc); //可以理解为fprintf为输出函数
                                           //在pf文件内输出这几组数据
  
  //关闭文件
  fclose(pf);
  pf = NULL;
  return 0;
}
```

fprintf也可以适用于其他输出流

---

## 9.fscanf
---

```c
#include<stdio.h>
.
int fscanf(FILE*stream, const char * format[,argument]....);
```

继上一段代码将结构中的数据存储于文件中
我们也可以读取文件中的结构

```c
struct S
{
  char arr[10];
  int num;
  float sc;
};

struct A
{
  char arr[10];
  int num;
  float sc;
};

int main()
{
 
   
  //对格式化数据进行读文件
  FILE*pf = fopen("test.dat","r");
  if(pf == NULL)
  {
    perror("fopen");
    return 1;
  }
  
  //读文件
  struct S s = {0};
  fscanf(pf,"%s %d %f",s.arr,s.num,s.sc);  //从pf中读取结构,再放到已之前输出的结构中
  fscanf(pf,"%s %d %f",a.arr,a.num,a.sc);  //同类型的结构不能放入,报错
  
  //打印
  printf("%s %d %f",s.arr,s.num,s.sc);

  //关闭文件
  fclose(pf);
  pf = NULL;
  return 0;
}
```

也可使用于其他流


---

## 10.fwrite
---

```c
#include<stdio.h>
.
size_t fwrite(const void*buffer,size_t isze,size_t count,FILE* stream);
```

该函数作用是

> 再stream流中写入大小为size,个数为count的二进制buffer数据

---

```c
struct S
{
  char arr[10];
  int num;
  float sc;
};

int main()
{
 
   
  //对格式化数据进行写文件
  FILE*pf = fopen("test.dat","w");
  if(pf == NULL)
  {
    perror("fopen");
    return 1;
  }
  
  //写文件
  struct S s = { "abcde",10,5.5f };
  fwrite(&s,sizeof(struct S),1,pf);
  

  //关闭文件
  fclose(pf);
  pf = NULL;
  return 0;
}
```


![](/img/6.12.png)
二进制数据
引出fread函数

---

## 11.fread
---

```c
#include<stdio.h>
.
size_t fread(void * buffer,size_t size,size_t count,FILE*stream);
```
该函数作用为

> 从stream流中读取count个大小为size的数据放入buffer内

---
我们以上述代码带入的结果测试一下

```c
struct S
{
  char arr[10];
  int num;
  float sc;
};

int main()
{
 
   
  //对格式化数据进行读文件
  FILE*pf = fopen("test.dat","r");
  if(pf == NULL)
  {
    perror("fopen");
    return 1;
  }
  
  //读文件
  struct S s = {0};
  fread(&s,sizeof(struct S),1,pf);

  //打印观察结果
  printf("%s %d %f",s.arr,s.num,s.sc);
  

  //关闭文件
  fclose(pf);
  pf = NULL;
  return 0;
}
```

![](/img/6.13.png)

二进制数据可以被读取为我们可以读懂的数据


---

## 12.sscanf、sprintf
---
```c
#include<stdio.h>
.
int sscanf(const char * buffer,const char*format[,argument]...);
```



> 从一段被格式化转换为字符串的数据
> 还原出原本的格式化数据

---



```c
#include<stdio,h>
.
int sprintf(char*buffer,const char*format[,argumen]...);
```


> 把一个格式化的数据(包含各种类型)
> 转化为字符串存储至buffer



```c
struct S
{
  char arr[10];
  int age;
  float f;
};

int main()
{
  struct S s = {"hello",20,5.5f};
  char buf[100] = {0};
  
  sprintf(buf,"%s %d %f",s.arr,s.age,s.f);
  //将s的格式化数据全部转换为字符串存入到buf内
  printf("%s\n",buf);
  
  return 0;
}
```

![](/img/6.14.png)

打印结果如上

我们还可以将这段字符串通过fscanf还原出结构中的数据

```c
struct S
{
  char arr[10];
  int age;
  float f;
};

int main()
{
  struct S s = {"hello",20,5.5f};
  struct S tmp = {0};
  
  char buf[100] = {0};
  
  sprintf(buf,"%s %d %f",s.arr,s.age,s.f);
  //将s的格式化数据全部转换为字符串存入到buf内
  printf("%s\n",buf);
  
  //从buf数据中还原出一个结构体数据
  sscanf(buf, "%s %d %f",tmp.arr,&(tmp.age),&(tmp.f));
  printf("%s %d %f\n",tmp.arr,tmp.age,tmp.f);
  
  return 0;
}
```

![](/img/6.15.png)



---

# 四、文件的随机读写
---

在文件顺序读写中,我们发现以fgetc为例

总是从第一个字符开始,并向后逐一递增

怎样可以自定义读写方式呢

这里就要用到文件随机读写的知识

---

## 1.fseek

---
```c
#include<stdio.h>
.
int fseek(FILE * stream,long int offset,int origin);
```


> 将指针定位至与起始位置 origin 偏移 offset 的位置
> origin分为三种情况
> SEEK_CUR  --- 当前文件指针的位置
> SEEK_END --- 文件末尾
> SEEK_SET --- 文件的起始位置


---


测试一下实际情况

![](/img/6.16.png)


```c
int main()
{
  FILE* pf = fopen("test.dat","r");
  if(pf == NULL)
  {
    perror("fopen");
    return 1;
  }

  //读取文件
  int ch = fgetc(pf);
  printf("%c\n",ch); //a
  
  //调整文件指针
  fseek(pf,1,SEEK_CUR);  //将指针向回偏移1,从a到c


  ch = fgetc(pf);
  printf("%c\n",ch); //c
  ch = fgetc(pf);
  printf("%c\n",ch);//d


  //关闭文件
  fclose(pf);
  pf=NULL;

  return 0;
}
```

![](/img/6.17.png)

---

## 2.ftell
---

```c
#include<stdio.h>
.
long int ftell(FILE * stream);
```



> 返回文件指针相对于起始位置的偏移量

---



```c
int main()
{
  FILE* pf = fopen("test.dat","r");
  if(pf == NULL)
  {
    perror("fopen");
    return 1;
  }

  //读取文件
  int ch = fgetc(pf);
  printf("%c\n",ch); //a
  
  //调整文件指针
  fseek(pf,1,SEEK_CUR);  //将指针向回偏移1,从a到c


  ch = fgetc(pf);
  printf("%c\n",ch); //c
  ch = fgetc(pf);
  printf("%c\n",ch);//d

  int ret = ftell(pf);
  printf("%d",ret); //打印结果为4

  //关闭文件
  fclose(pf);
  pf=NULL;

  return 0;
}
```


---

## 3.rewind
---

```c
#include<stdio.h>
.
void rewind(FILE*stream); 
```



> 让文件指针回到起始的位置

---

还以上述举例

```c
int main()
{
    FILE* pf = fopen("test.dat", "r");
    if (pf == NULL)
    {
        perror("fopen");
        return 1;
    }

   

    //调整文件指针
    fseek(pf, 3, SEEK_CUR);  //将指针向回偏移3


    int ch = fgetc(pf);
    printf("%c\n", ch); //d

    int ret = ftell(pf);
    printf("%d\n", ret); //打印结果为4

    rewind(pf);  //使文件指针回到起始位置

    ret = ftell(pf);
    printf("%d\n", ret); //观察偏移量

    ch = fgetc(pf);
    printf("%c\n", ch);//a 

    //关闭文件
    fclose(pf);
    pf = NULL;

    return 0;
}
```
![](/img/6.18.png)

---

# 五、文件读取结束的判定(feof)
---

feof的作用使

> 判断文件是读取失败结束的,还是遇到文件尾结束的

---
对于文本文件:

> 通过判定返回值来确定
> 判断返回值是否尾EOF(fgetc),或者NULL(fgets)

**fgetc函数在读取结束后返回的是EOF
正常读取时返回的是读取到的字符的ASCII值**


**fgets函数在读取结束时返回的是NULL
正常读取时返回的是,存放字符串空间的起始地址**

---

对于二进制文件:

> 判断返回值是否小于实际要读的个数

**fread函数在读取的时候,返回的是实际读取到的完整元素的个数
如果发现读取到完整的元素个数小于 指定元素个数,这就是最后一次读取**


---
举个例子

```c
//文件拷贝

int main()
{
  FILE * pread = fopen("test.txt","r");
  if(pread == NULL)
  {
    perror("pread");
    return 1;
  }
  
  FILE * pwrite = fopen("test.txt2","w");
  if(pwrite == NULL)
  {
    //如果失败直接返回,文件没有关闭,会出问题
    fclose(pread);
    pread = NULL;
    return 1;
  }

  //开始拷贝
  int ch = 0;
  while( ch = fgetc(fread) != EOF)
  {
    fputc(ch,fwrite);
  }


  //判断文件怎样结束
  if(feof(pread)) //如果读取成功返回非0
  {
    printf("遇到文件结束标志,文件正常结束");
  }
  else if(ferror(pread)
  {
    printf("文件读取失败");
  }

  //关闭文件
  fclose(pread);
  pread = NULL;
  fclose(pwrite);
  pwrite = NULL;
}
```

---
# 六、文件缓冲区

> 简而言之就是
> 从程序到文件、从文件到程序
> 必须塞够足够的数据或者刷新( fflush (pf) )(高版本无法使用刷新)
> 数据才会完全输送到目的地


---
# 七、文本文件和二进制文件

> 1.数据在内存中以二进制的形式存储,如果不加以转换输出到外存,就是二进制文件
> 2.如果要求在外存以ASCII的形式存储,则需要在存储前转换
> 以ASCII字符的形式存储的文件就是文本文件

一个数据在内存中是怎么存储的呢

> 字符一律以ASCII形式存储
> 数值即可以ASCII存储也可以二进制形式存储

---
假设整数10000
ASCII存储需要5个字符 --五字节
而二进制只需要占4字节 --转换为二进制再转换为16进制计算


---
