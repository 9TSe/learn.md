---
title: 常见函数和模拟实现
date: 2023-10-21 20:59:15
tags:
- 模拟实现
categories: 
- C语言
cover: /pic/1.png
---

`
    前言: 学习C语言的过程中模拟实现各种库函数可以让我们了解更加底层的原理,在这里我整理了部分函数的模拟实现和注意事项.`


# 一、strlen

## 1.函数介绍

strlen的原型为

```c
#include<string.h>
.
int strlen(char *str);
```
.

其作用和原理是
>   传入要统计字符串的地址
>   统计该字符串'\0'(字符串结束标志)前的字符个数

---
## 2.模拟实现
###  ①  指针计数器方式

```c
int my_strlen(const char *str)
{
  assert(str); //判定不为空指针再运行
  int count = 0; //count作为计数器
  while(*str != '\0') //若str解引用不为字符串中的'\0' 则停止
  {
    count++;
    str++;
  }
  return count;
}
```

---

### ② 函数递归方式

```c
int my_strlen(const char *str)
{
  assert(str);
  if(*str != '\0')
  {
    return (1 + my_strlen(str + 1) );
    //注意 这里由一种易错的写法,如下"
    
  //return (1 + my_strlen(++str) );
  
    //此时会改变递归进行中str的值,导致计算中途出现错误
    
  }
  else
  {
    return 0;
  }
}
```

---

### ③ 指针减指针方法

我们要知道

> 同类型的指针减指针得到的结果为两个指针间的元素个数


所以我们可以通过指针减指针的方法来模拟实现strlen

具体实现方法如下

```c
int my_strlen(const char * str)
{
  assert(str);
  char * mem = str; //通过 mem 记录 str 的起始位置
  while( *str != '\0')
  {
    str++;
  }
  return str - mem; //指针减指针得到中间元素的个数
}
```

---


# 二、strcpy

## 1.函数介绍
strcpy的原型为

```c
#include<string.h>
.
char *strcpy(char* dest, const char *src);
```

.
strcpy的作用和原理是 
> 先后传入空间的目标空间的起始地址和源空间的起始地址 
> 将源空间的内容拷贝至目标空间


该函数的脾气

> 如果目标空间不够,会越界访问,导致报错 
> 所以目标空间必须足够大,确保能放进去;
> 源空间的'\0'也会被拷贝过去;
> 目标空间必须可以改变(const,指针指向字符串常量字符串就不行)

---
## 2.模拟实现
具体实现方法如下:

```c
void my_strcpy(char *destination,const char * source)
{
  assert( destination && source); //断言一下两者都不为空指针
  
  while( *source != '\0')
  {
    *destination = *source;
     destination++;
     source++;
  }
  *destination = *source;
}
```

略微观察一下这个代码,我们可以写出更简洁的版本,如下

```c
void my_strcpy(char *destination,const char * source)
{
  assert( destination && source);
  while( *destination++ = *source) //当碰到'\0'(即为0,判定为假)
  {
     ;
  }
}
```

我一开始有疑问,当source字符串里面有0这个函数功能是不是就失效了呢?

原来,判断的根据是ASCII码值为0 ,我所想的'0' ASCII值为48
而ASCII值为0的就是 '\0' (空字符)

以上

---

# 三、qsort
## 1.函数介绍
qsort函数的原型

```c
#include<stdlib.h>

void qsort(
           void*base,   //要被排序的第一个元素的地址
           size_t num,  //排序数据元素的个数
           size_t size, //排序数据中一个元素的大小
           int(*compar)(const void* e1, const void* e2)
                        //用来比较待排序数据中的两个元素 的函数
          );
```

---
我们以整形排序实例来看一下qsort的使用

```c
int Cmp_int(const void*e1,const void*e2) //比较两个元素大小的函数
{
  return *(int*)e1 - *(int*)e2; 
}
int main()
{
  int arr[] = {9,8,7,6,5,4,3,2,1};
  int sz = sizeof(arr) / sizeof(arr[0]);

  qsort(arr,sz,sizeof(int),Cmp_int);

  //打印出的结果为
  // 1,2,3,4,5,6,7,8,9
}
```

---

通过qsort排序结构体

```c
struct stu
{
  char name[20];
  int age;
};


int sortby_age(const void*e1,const void*e2)
{
  return (struct stu *)e1 -> age - (struct stu *)e2 -> age;
}


int sortby_name(const void*e1,const void*e2)
{
  return strcmp((struct stu *)e1 -> name, (struct stu *)e2 -> name);
}

//若想改变排序的升序或降序,改变一下 e1 和 e2 的位置即可

int main()
{
  struct stu s[3] = {{"zhang",30},{"lisi",34},{"wang",20}};
  int sz = sizeof(s) / sizeof(s[0]);
  //年龄排序
  qsort(s,sz,sizeof(s[0]),sortby_age);
  //结果为  20,30,34 方式排列出结构体

  //名字排序
  qsort(s,sz,sizeof(s[0],sortby_name);
  //结果为   lisi,wang,zhang 方式排列出结构体
```

---
.
## 2.模拟实现


```c
void swap(char* buf1, char * buf2 , int width)
{
  int i = 0;
  for(i = 0 ; i < width ; i++)
  {
    char tem = *buf1;
    *buf1 = *buf2;
    *buf2 = tem;
    buf1++;
    buf2++;
  }
} 

void my_qsort(
                 void*base,   
                 size_t sz,  
                 size_t width, 
                 int(*cmp)(const void* e1, const void* e2)
               )

{ 
  
  int i = 0;
  //趟数
  for(i = 0; i < sz-1 ; i++)
   {
     //每一趟的排序
     int j = 0;
     for(j = 0 ; j < sz - 1 - i ; j++)
     {
      //开始比较两个元素
      if( cmp( (char*)base + j*width , (char*)base + (j+1)*width)>0)
         //一个字节一个字节进行比较和交换
       {
         //交换
         swap( (char*)base + j*width , (char*)base + (j+1)*width)
       }
      }
    }
}
```

---

# 四、strcat
## 1.函数介绍
strcat原型

```c
#include<string.h>
.
char *strcat(char *des, const char *src);
```

strcat的作用

> 将源空间中的内容追加到目标空间后
> 目标空间中的 '\0' 被替换为源空间中第一个元素

`注意:strcat不能自己追加自己,会使'\0'被覆盖,导致无法终止`

---
## 2.模拟实现

```c
char* my_strcat(char * dest, const char * src)
{
  assert(dest && src);
  char * ret = dest;
  //寻找目标空间中的'\0'
  while(*dest)
  {
    dest++;
  }
  //开始追加
  while( *dest++ = *src++);
  
  return ret;
}
```
---


# 五、strcmp
## 1.函数介绍
函数原型为

```c
#include<string.h>
.
int strcmp(const char* str1, const char* str1);
```

函数的作用

> 我们知道,两个字符串无法直接用大于小于号来进行比较 
> 因此就存在了strcmp这个函数
> .
> 对字符串逐一比较字符ASCII大小
> 一旦有大于或小于便停止,返回结果,若都相等则返回0
> 若前面都相等,则较长的win

---
## 2.模拟实现

```c
int my_strcmp(const char* s1,const char* s2)
{
  assert(s1 && s2);
  while(*s1 == *s2)
  {
    if(*s1 == '\0')
    {
      return 0;
    }
    s1++;
    s2++;
  }
  return *s1 - *s2;
}
```
---
# 六、strncpy
## 1.函数介绍
该函数原型为

```c
#include<string.h>
.
char* strncpy(char* dest, const char* src, size_t num);
```
该函数作用和原理为

> 选择一定数量的字符拷贝至目标空间
> .
> 若传递数量超过源空间含有的数量,并且空间足够
> 那么就会先将源空间内所有字符传进去后,按照规定数量补'\0'

---
## 2.模拟实现

```c
char* my_strncpy(char * dest,const char * src,int num)
{
  char* ret = dest;
  while(num && (*dest++ = *src++) != '\0')
  {
    num--;
  }
  if(num)
  {
    while(--num)
    {
      *dest++ = '\0';
    }
  }
  return ret;
}
```

---
# 七、strncat
## 1.函数介绍
函数的原型为

```c
#include<string.h>
.
char *strncat(char *dest, const char *src, size_t num);
```

函数作用为

> 选择固定数量的字符从源空间中追加到目标空间内

---
## 2.模拟实现

```c
char* strncat(char *dest, const char *src, size_t num)
{
  char *ret = dest;
  while(*dest++)
  {
    dest--;
  }
  while(num--)
  {
    if((*dest++ == *src++)==0) //发现'\0'直接结束,不再继续进行循环
    {
      return ret;
    }
  }
  *dest='\0';   //提前结束
  return ret;
}
```
---
# 八、strncmp
## 1.函数介绍
strncmp的原型为

```c
#include<string.h>
.
int strncmp(const char* str1, const char* str2, size_t num);
```
strncmp的作用为

> 固定一个数量
> 使两个字符串的前几个数量的字符进行比较

---

## 2.模拟实现

```c
int My_strncmp(const char* str1, const char* str2, size_t n)
{
	if (n==0)
    {
	    return 0;
	}
	while (n-- && *str1 != '\0' && *str1 == *str2)
	{
		str1++;
		str2++;
	}
	return *str1 - *str2;
}
```

---

# 九、strstr
## 1.函数介绍
strstr函数原型为

```c
#include<string,h>
.
char*strstr(const char* str1,const char* str2);
```

该函数作用为

> 在str1中寻找str2
> 返回str1中出现str2首元素的地址
>若找不到则返回空指针

---
## 2.模拟实现

```c
char * my_strstr ( const char * str1, const char * str2)
{
  assert(str1 && str2);
  const char * s1 = NULL;
  const char * s2 = NULL;
  const char * cp = str1;
  
  if(*str2 == '\0')
  {
    return (char*)str1;
  while(*cp)
  {
    s1 = cp;
    s2 = str2;
    while(*s1 && *s2 &&(*s1 == *s2) )
    {
      s1++;
      s2++;
    }
    if(*S2 == '\0')
    {
      return (char*)cp;
    }
    cp++;
  }
  return NULL;
}
```

---

# 十、strtok
## 1.函数介绍
该函数原型为

```c
#include<string.h>
.
char* strtok(char * str, const char * sep);
```

该函数的介绍

> 在str中  以sep中的字符分割str
> sep中可以有一个或多个分隔符标记
> .
> strtok函数找到str中的下一个标记,并将其用'\0'结尾,返回一个指向这个标记的指针
> 所以该函数会改变被操作的字符串,所以在使用时一般面向对象都是临时拷贝的内容
> .
> strtok函数第一个参数不为空指针,函数将找到str中第一个标记,strtok会保存它在字符串中的位置
> strtok函数第一个参数为空指针,函数将在同一字符串中被保存的位置开始,查找下一个标记
> 如果字符串中不存在标记则会返回空指针


---
我们以一个实例来加深理解

```c
char arr[30] = "9TSe@qq.com";
char *p = "@.";

char tmp[30] = {0}; //创建临时拷贝份
strcpy(tmp,arr);

//9TSe@qq.com\0
strtok(tmp,p); //返回9TSe   9TSe\0qq.com\0
strtok(NULL,p);//返回qq     9TSe\0qq\0com\0
strtok(NULL,p);//返回com    9TSe\0qq\0com\0
```

---
那么实际上使用该函数的用法为

```c
char arr[30] = "9TSe@qq.com";
char *p = "@.";

char tmp[30] = {0}; 
strcpy(tmp,arr);

char * ret = NULL;
for(ret = strtok(tmp,p); ret != NULL ; ret = strtok(NULL,p) )
{
  printf("%s\n",ret);
}
```
---
**该函数由于使用情况不多,暂且不做模拟实现**

---

# 十一、strerror
## 1.函数介绍

该函数原型为

```c
#include<string.h>
#include<errno.h>
.
char * strerror(int errnum);
```


该函数的作用为

> 返回错误码所对应的错误信息

---

使用情况一般如下

```c
printf("%s",strerror(errno)); //errno是库函数里本有的全局变量,加入头文件就不用再创建
```
---
**模拟暂不实现**

---

# 十二、perror
## 1.函数介绍

该函数原型为

```c
#include<stdio.h>
.
void perror(const char * str);
```

该函数的作用为

> perror可以直接转换错误信息并且打印
> 比如
> perror("fopen") ;
> fopen: No such file or directory 

---
**该函数暂不模拟实现**

---


# 十三、字符分类函数和字符转换函数
## 1.函数介绍
他们公用原型

```c
#include<ctype.h>
.
int xxx(int c);
```

函数功能为

> 为真返回非0
> 为假返回0
---
|函数| 如果符合以下条件返回真 |
|--|--|
| iscntrl | 任何控制字符 |
|  isspace| 空白字符:空格' ' ,换页'\f',换行'\n',回车'\r',制表符'\t',垂直制表符'\v' |
|  isdigit|十进制数字 0 ~ 9  |
|  isxdigit| 十六进制数字,包括所有十进制数字和大小写字母 a~f 、A~F |
|  islower|  小写字母 a ~ z|
| isuper | 大写字母 A ~ Z |
|  isalpha|  字母 a ~ z 和 A ~ Z|
| isalnum |  字母或数字 a ~ z , A ~ Z , 0 ~ 9|
|  ispunct|  标点符号,任何不属于数字或字母的图形字符(可打印)|
| isgraph |  任何图形字符|
|  isprint|  任何可打印字符,包括图形字符和空白字符|

---

| 字符转换函数 | 功能 |
|--|--|
| tolower | 将字符大写转换为小写 |
|  toupper| 将字符小写转换为大写 |

---


#  十四、memcpy
## 1.函数介绍
该函数原型为

```c
#include<string.h>
.
void * memcpy(void * dest , const void * src , size_t num);
```

该函数的功能


> memcpy为内存拷贝,将src中的内容拷贝至dest
> num中存放的为拷贝的**字节大小**

---
## 2.模拟实现

```c
void * my_memcpy(void * dest , const void * src , size_t num)
{
  assert(dest && src);
  void * ret = dest;
  while(num--)
  {
    *(char*)dest = *(char*)src; //一个字节一个字节拷贝,所以强制转换为char*再解引用
    dest = (char*)dest + 1;
    src  = (char*)src + 1;

    //那能不能写如下的更加简便呢
    //*(char*)dest++ = *(char*)src++;
    
    //很遗憾并不可以 ++在生效时dest和src还是void*类型,不能++;
  }
  return ret;
}
```

---

`注意:memcpy不能拷贝重叠的内存`

```c
int arr[10] = {1,2,3,4,5,6,7,8,9,10};
memcpy(arr+2,arr,20);
//结果并不为 {1,2,1,2,3,4,5,8,9,10}
//而是为     {1,2,1,2,1,2,1,8,9,10}
```

但是memmove却可以做到

---

# 十五、memmove
## 1.函数介绍
该函数原型为

```c
#include<string.h>
.
void * memmove( void * dest, const void * src , size_t num);
```
---
## 2.模拟实现

```c
void * memmove( void * dest, const void * src , size_t num)
{
  assert(dest && src);
  void * ret = dest;

  if(dest<src)
  {
    //前->后
    while(num--)
    {
      *(char*)dest = *(char*)src;
      dest = (char*)dest + 1;
      src  = (char*)src + 1;
    }
  }
  else
  {
    //后->前
    while(num--)
    {
      *((char*)dest+num) = *((char*)src+num);
    }
  }
  return ret;
}
  
```

![](/img/3.1.png)
如图为分情况讨论理解图

---

# 十六、memcmp
## 1.函数介绍

该函数原型为

```c
#include<string.h>
.
int memcmp(const void * str1 , const void * str2 , size_t num);
```

该函数作用

> 对比str1和str2中前 num 字节的内容


该函数较strcmp较相似,不再模拟实现

---

# 十七、memset
## 1.函数介绍
该函数原型为

```c
#include<string.h>
.
void * memset(void * ptr , int value , size_t num);
```

该函数作用为

> 设置ptr内前num字节 全部初始化为 value

---
**该函数不进行模拟实现**

----