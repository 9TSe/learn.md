---
title: 动态内存分配
date: 2023-10-21 21:00:13
tags:
- 动态内存分配
categories: 
- C语言
cover: /pic/1.png
---


# 一、栈区,堆区,静态区(数据段)

---

![](/img/5.1.png)

---

# 二、malloc
---

该函数原型为

```c
#include<stdlib.h>
#include<malloc.h>
.
void * malloc(size_t size);
```

该函数作用为

> 在堆区开辟size个字节的空间
> 开辟成功返回开辟的地址
> 开辟失败返回空指针

---
实际应用如下

```c
int arr[10];
int *p = (int*)malloc(10*sizeof(int));
//开辟十个整形空间,强制转换位int* 类型方便接收和处理

//如果开辟失败
if(p == NULL)
{
  perror("main");
  return 0;
}

//使用
int i = 0;
for(i = 0; i < 10; i++)
{
  printf("%d",p[i]); //p[i] == *(p+i)
}

//回收空间
free(p);  //若不回收,会导致内存泄漏
p = NULL; //手动把p指针置为空指针

return 0;
```

---
# 三、free
---
该函数原型为

```c
#include<stdlib.h>
.
void free(void * ptr)
```

> free函数用来释放动态开辟的内存
> 如果参数ptr指向的空间不是动态开辟的,free函数的行为是未定义的
> 如果参数ptr指向的是NULL指针,则函数什么也不做


比如当ptr指向的空间不是动态开辟的

```c
int a = 10;
int * p = &a;
free(p); //error
```

`注意:动态开辟一块空间必须释放,但是同一块空间不能释放两次,否则会报错.`

**动态开辟的空间只有两种回收方式**

> 手动free释放
> 程序结束时释放

---
# 四、realloc
---

该函数原型

```c
#include<stdlib.h>
#include<malloc.h>
.
void * realloc(void * memblock,size_t size);
```

该函数的作用是

> 在memblock地点重新开辟size字节的空间
> 但开辟有三种情况
> 所以返回的地址不一定是原本memblock的地址

以一张图来解释一下

![](/img/5.2.png)

---

看一下realloc实际使用情况

```c
int arr[10];
int *p = (int*)malloc(10*sizeof(int));
//开辟十个整形空间,强制转换位int* 类型方便接收和处理

//如果开辟失败
if(p == NULL)
{
  perror("main");
  return 0;
}

//重新开辟为20字节的空间
int * ptr = realloc(p,20*sizeof(int)); //拿新指针接收

if(ptr != NULL)
{
  p = ptr       //如果开辟成功则将ptr的地址交给p.使p指向的空间更大
}

//回收空间
free(p);  //若不回收,会导致内存泄漏
p = NULL; //手动把p指针置为空指针

return 0;
```
---
与malloc相似
当出现以下情况时

```c
int * p = (int*)realloc(NULL,40); 
//功能类似于malloc,直接在堆区开辟40字节
```

---

# 五、calloc
---

```c
#include<stdlib.h>
#include<malloc.h>
.
void * calloc(size_t num , size_t size);
```

该函数的功能是

> 为num个大小为size的元素开辟一块空间,并把空间的每个字节初始化为0.
> 与malloc的区别在于calloc会在返回地址之前把申请的空间的每个字节初始化为0


上一段代码看看和malloc的区别

```c
int * pm = (int*)malloc(10*sizeof(int));//malloc不会初始化每个字节
                                        //即存储随机值

int * pc = (int*)calloc(10,sizeof(int));
                              //calloc会顺便初始化每个字节为0
```



---

# 六、柔性数组
---

> 结构中的最后一个元素允许是未知大小的数组
> 这就叫做柔性数组的成员




```c
struct A
{
  int n;
  int arr[];  //大小未知
};

struct B
{
  int n;
  int arr[0];  //大小未知
};
```

柔性数组的特点

> 1.结构中柔性数组成员前必须至少有一个其他成员
> 2.sizeof 返回这种结构大小,不包括柔性数组
>3. 包含柔性数组的结构用malloc进行内存的动态分配,分配的空间应大于结构的大小,以适应柔性数组的预期大小


柔性数组的应用

```c
struct S
{
  int n;
  int arr[0];
};

int main()
{
  //期望arr大小为十个整形
  struct S *ps = (struct*)malloc(sizeof(struct S) + 10*sizeof(int));
                                            //为柔性数组预留十个整形
  ps->n = 10;
  int i = 0;
  for(i = 0; i < 10; i++)
  {
    ps->arr[i] = i;
  }
  
 //增加
 struct S *p=(struct S*)realloc(ps,sizeof(struct S)+20*sizeof(int));
 if(p != NULL)
 {
   ps = p;
 }

 //使用

 //释放
 free(ps);
 ps = NULL;
 
}
```

---
指针也可以
创建可改变的空间

```c
struct S
{
  int n;
  int *arr;
};

int main()
{
  //为结构开辟空间
  struct S *ps = (struct*)malloc(sizeof(struct S));
  if(ps == NULL)
  {
    return 1;
  }
  
  //为arr指向的空间开辟空间
  ps->arr = (int*)malloc(10*sizeof(int));
  if(ps->arr == NULL)
  {
    return 1;
  }

  //使用
  int i = 0;
  for(i = 0; i < 10 ; i++)
  {
    ps->arr[i] = i;
  }
  
  //增加
  int * ptr = (int *)realloc(ps->arr,20*sizeof(int));
  if(ptr != NULL)
  {
    ps->arr = ptr;
  }

  //使用

  //释放
  free(ps->arr);
  ps->arr = NULL; //必须先释放ps->arr , 否则在释放完ps后找不到 ps->arr
  free(ps);
  ps = NULL;
```
---
**指针的方法用了两次malloc,两次free**

**多次使用malloc会使内存碎片变多,内存的利用率会下降**

此时柔性数组的优势就会体现出来



---
