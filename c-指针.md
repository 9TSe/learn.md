---
title: 指针
date: 2023-10-21 20:58:53
tags: 
- 指针
categories: 
- C语言
cover: /pic/1.png
---



# 一、 指针指向字符常量

```c
char *pc1 = "9TSe";
char *pc2 = "9TSe";

//pc1和pc2指向的地址是相同的(相同字符串,系统为了节省空间就没有再开辟空间

//有些编译器在写以上代码时会报错
//以下是解决方式

const char *pc1 = "9TSe";
const char *pc2 = "9TSe";

//因为 指针指向的是字符常量,不可修改,编译器认为其不安全,故报错
//加上const修饰即可
```

---

# 二、 指针、数组

---

## 1. 指针数组
---

> 指针数组是一个数组 
> 数组里面存放着指针

```c
int a[3] = {1,2,3};
int b[3] = {4,5,6};
int c[3] = {7,8,9};

int* arr[3] = {a,b,c}; 
//数组名即为数组首元素地址
//arr就为有三个元素,每个元素是一个指针的数组
```
---
## 2. 数组指针
---

> 数组指针是一个指针 
> 指向数组的指针

```c
int arr[5] = {1,2,3,4,5};

int *parr = arr;//error arr是数组首元素的地址,而不是数组的地址
          //虽然数组的地址指向的也是数组首元素的地址,但二者本质却不相同
           
int *parr = &arr;     //error 虽然确实指向了数组arr的地址,但是类型不对

int *parr[5] = &arr;  //error parr先和[5]结合,表示的就是一个指针数组了

int (*parr)[5] = &arr;//bingo
```


```c
//写出 double* arr[5] 的指针

double * (*pd)[5] = &arr; //pd就是一个数组指针
```



---

## 3. 数组参数,指针参数传参
---
### ①一维数组传参
```c
void test1(int arr[]) //传过来的是数组,拿数组接收,可行

void test1(int arr[10]) //10是没有任何意义的,但也可行

void test1(int *arr) //即为地址,可行

void test2(int *arr[20]) //拿指针数组接收,可行

void test2(int **arr) // 即为地址,可行

int main()
{
  int arr1[10] = {0};
  int *arr2[20] = {0};
  test1(arr1); //传过去的是数组首元素地址
  test2(arr2); //传过去指针数组
}
```

----

### ②二维数组的传参

```c
void test(int arr[3][5]); // 二维数组接受,可行

void test(int arr[][]);   // 不能省略列

void test(int arr[][5]);  // 可行

//二维数组传参,函数形参的设计只能省略第一个[]中的数字
//因为对于一个二维数组,可以不知道有多少行,但必须知道一行有多少元素
//以方便运算

void test(int *arr);     //传过来数组的地址,不能拿单元素接收,不可行 

void test(int *arr[5]);  //这是指针数组,不可行 

void test(int (*arr)[5]);//接受二维数组的第一行,即二维数组首元素可行 

void test(int **arr);    //二级指针,不匹配,不可行

int main()
{
  int arr[3][5] = {0};
  test(arr);
} 
```

---

## 4. 一维数组,二维数组,指针大小判断例题
---
### ①整形数组
```c
int a[] = {1,2,3,4);

printf("%d\n",sizeof(a));   // 16 
printf("%d\n",sizeof(a+0)); // 4/8 a+0 是数组第一个元素的地址
printf("%d\n",sizeof(*a));  // 4   *a  是数组的第一个元素
printf("%d\n",sizeof(a+1)); // 4/8 a+1 是第二个元素的地址
printf("%d\n",sizeof(a[1]));// 4      第二个元素

printf("%d\n",sizeof(&a));     //数组的地址
printf("%d\n",sizeof(*&a));    //数组的大小
printf("%d\n",sizeof(&a+1));   //跳过整个数组后面的空间的地址
printf("%d\n",sizeof(&a[0]));  //第一个元素的地址
printf("%d\n",sizeof(&a[0]+1));//第二个元素的地址
```
---

### ②字符数组

```c
char arr[] = {'a','b','c','d','e','f'};

printf("%d\n",sizeof(arr));      //6    整个数组
printf("%d\n",sizeof(arr+0));    //4/8  首元素地址
printf("%d\n",sizeof(*arr));     //1    首元素
printf("%d\n",sizeof(arr[1]));   //1    第二个元素
printf("%d\n",sizeof(&arr));     //4/8  整个数组地址
printf("%d\n",sizeof(&arr+1));   //4/8  跳过一个数组后的地址
printf("%d\n",sizeof(&arr[0]+1));//4/8  第二个元素的地址

printf("%d\n",strlen(arr));       //随机值
printf("%d\n",strlen(arr+0));     //随机值
printf("%d\n",strlen(*arr));      //报错    首元素
printf("%d\n",strlen(arr[1]));    //报错    第二个元素
printf("%d\n",strlen(&arr));      //随机值
printf("%d\n",strlen(&arr+1));    //随机值-6
printf("%d\n",strlen(&arr[0]+1)); //随机值-1
```


```c
char arr[] = "abcdef";           //{'a','b','c','d','e','f','\0'};

printf("%d\n",sizeof(arr));      //7    数组内所有元素
printf("%d\n",sizeof(arr+0));    //4/8  首元素地址
printf("%d\n",sizeof(*arr));     //1    第一个元素
printf("%d\n",sizeof(arr[1]));   //1    第二个元素
printf("%d\n",sizeof(&arr));     //4/8  数组的地址
printf("%d\n",sizeof(&arr+1));   //4/8  跳过整个数组后的地址
printf("%d\n",sizeof(&arr[0]+1));//4/8  第二个元素的地址

printf("%d\n",strlen(arr));       //6
printf("%d\n",strlen(arr+0));     //6
printf("%d\n",strlen(*arr));      //报错
printf("%d\n",strlen(arr[1]));    //报错
printf("%d\n",strlen(&arr));      //6
printf("%d\n",strlen(&arr+1));    //随机值  数组后的'\0'不知道在哪里
printf("%d\n",strlen(&arr[0]+1)); //5
```

---

### ③指针指向字符串

```c
char *p = "abcdef";

printf("%d\n",sizeof(p));       //4/8   一个指针
printf("%d\n",sizeof(p+1));     //4/8   b的地址
printf("%d\n",sizeof(*p));      //1     即第一个元素
printf("%d\n",sizeof(p[0]));    //1     p[0] == *(p+0)
printf("%d\n",sizeof(&p));      //4/8   二级指针
printf("%d\n",sizeof(&p+1));    //4/8   二级指针+1
printf("%d\n",sizeof(&p[0]+1)); //4/8   b的地址


printf("%d\n",strlen(p));       //6
printf("%d\n",strlen(p+1));     //5
printf("%d\n",strlen(*p));      //报错
printf("%d\n",strlen(p[0]));    //报错
printf("%d\n",strlen(&p));      //随机值
printf("%d\n",strlen(&p+1));    //另一个随机值
printf("%d\n",strlen(&p[0]+1)); //5
```
---
### ④二维数组

```c
int a[3][4] = {0};

printf("%d\n",sizeof(a));            //48	
printf("%d\n",sizeof(a[0][0]));      //4
printf("%d\n",sizeof(a[0]));         //16  可以理解为第一行数组的数组名

printf("%d\n",sizeof(a[0]+1));       //4/8  a[0]作为数组名并没有单独放在sizeof内部
                                     //     也没有取地址,即a[0]就是第一行第一个元素的地址
                                     //     a[0]就是第一行第二个元素的地址
                                     
printf("%d\n",sizeof(*(a[0]+1)) );   //4    第一行第二个元素

printf("%d\n",sizeof(a+1));          //4/8  a是数组名.并没有单独存放在sizeof内部
                                     //     也没有取地址,即a就是第一行的地址
                                     //     所以a+1为第二行的地址
                                     
printf("%d\n",sizeof(*(a+1)));       //16   == a[1] 即第二行所有元素
printf("%d\n",sizeof(&a[0]+1));      //4/8  第二行的地址
printf("%d\n",sizeof(*(&a[0]+1)));   //16   第二行所有元素
printf("%d\n",sizeof(*a));           //16   == *(a+0) == a[0] 即第一行元素
printf("%d\n",sizeof(a[3]));         //16   计算的是第四行的数组名,计算的是类型大小,即 int[4] 类型
```
---
# 三、函数指针


## 1.函数指针的介绍
---
函数指针即

> 指向函数的指针
>  存放函数地址的指针

前面我们知道,在数组当中
`&(数组名) !=  数组名`

但是在函数指针中

`&(函数名) == 函数名`

---

**函数指针的创建和使用方式:**

```c
int Add(int x,int y)
{
  return x+y;
}

int main()
{
  int a = 1;
  int b = 2;
  

  int (*pf)(int, int) = &Add; 
  int (*pf)(int, int) = Add; //以上两种方式均可
  //pf即为函数指针
  //int 代表Add的返回类型
  //(*pf)代表声明pf为一个指针
  //(int,int)即为Add 的参数类型

  int ret = (*pf)(a,b);  //*并无意义,这样写只是更方便理解
  int ret = (******pf)(a,b);//这种方式计算出的结果也并无差错
  int ret = pf(a,b);
  int ret = Add(a,b);
  
  int ret = * pf(a,b);//这样写时就会出错,相当于 *(pf(a,b)) == *3

  return 0;
}
```

---
## 2.函数指针数组

我们以一段代码来展示以下函数指针数组的创建和使用

```c
int Add(int a,int b)
{
 return a+b;
}
int Sub(int a,int b)
{ 
 return a-b;
}
int main()
{
  int (*pA)(int ,int) = Add;
  int (*pS)(int ,int) = Sub;
  
  int (*parr[2])(int ,int) = {Add,Sub}; 
  //parr即为函数指针数组
```

---

# 四、回调函数
---

> 回调函数就是
> 一个通过函数指针调用的函数

举一个简单的例子

```c
int Add(int a,int b)
{
 return a+b;
}
int Sub(int a,int b)
{ 
 return a-b;
}
int all( int(*pf)(int,int) )
{
  int x = 0;
  int y = 0;
  scanf("%d %d",&x,&y);
  return pf(x,y);
}
```

---


