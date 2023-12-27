---
title: tips
date: 2023-10-21 21:02:43
tags:
- 动态内存分配
categories: 
- C++
cover: /pic/2.png
---

---


# 一、函数重载

---


> 在程序编译时分为四步:预编译 编译 汇编 链接
> 而C++的函数重载与C的区别就在于`链接`生成符号表时

> C :  0x000000 :
>  Add  C语言中, .o文件符号表中地址对应的函数名为 自己创建的函数名,所以不能创建同名函数

> C++:  0x000000 : _Z3Adddd 和 _Z3Addii
>   `_Z为标志符,3为函数字符数,Add为函数名,ii为两个参数类型`


由以上分析,我们可以知道函数重载的实现的重点为`参数`

 `数量or顺序or类型`不同时就能构成重载
 
 注意,如果构建两个函数名相同的函数,若只有返回值类型的不同无法构成重载
因为其与参数无任何关系

---

# 二、缺省函数
---
```cpp
int Func1(int a = 10, int b = 20,int c = 30) 
{
	return a + b;
}

int Func2(int a , int b = 20, int c = 30)
{
	return a + b;
}

int main()
{
	Func1();
	Func1(3);       //加入不完整的参数默认为从左到右,此时函数内a==3,b==20,c==30;
	Func1(3, 4);    //此时函数内a==3,b==4,c==30
	Func1(3, 4, 5); //此时函数内a==3,b==4,c==5
	return 0;
}
```




---
缺省函数有以下注意要点

> 1.填补缺省参数时,填补只能 `从右向左` 并且 `连续`才行
> 2.如果构建两个函数名称相同,参数相同,唯有参数的缺省值不同,此时并`不构成重载`,反而还会报错

---

# 三、引用 &
---
## 1.引用的基本定义



---
`引用必须在定义时就进行初始化`

```cpp
int& e;  //这样一写九成要爆炸
```

---
`关于const修饰的引用(权限的扩大和缩小)`

```cpp
const int f = 4; //f创建时的权限为只读
int& g = f;   //此时f的引用g,可读可写,权限被放大,直接爆炸
```

```cpp
const int f = 4;
const int& h = f; //此时权限没有放大,可行
```

> 在引用前一定要注意要引用的对象权限,只能缩小和不变,不能扩大权限

---

`临时变量具有常性`,`隐式类型转换`

```cpp
int k = 6;
double l = k;  //可行

float& m = k; //报错
double& n = k;//报错
//此时为什么两者都不行呢?

const float& o = k;  //可行
const double& p = k; //可行
//加上const不让权限放大即可

int q = 10;
const int& r = q;
q = 20; //无措且成功修改
cout << r << endl; //r变为20
r = 30; //报错
```

隐式类型转换发生在上述 double l = k; 时(隐式类型转换)

> 赋值前会先创建一个临时的double变量,k先将值赋值给这个临时的double变量,再将临时变量赋值给l

引用时报错的原因是(临时变量具有常性)

> 因为临时创建的临时变量具有 常性(即只读性),即访问权限放大了

---

## 2.引用作为函数参数和返回类型(浅提static,函数返回值的过程)
---
```cpp
int& Count1() 
{
	static int n = 0;  //static 在程序运行后只执行一次,因此程序只执行一次 static int n = 0;
	n++;
	return n; //返回时,引用一个临时名称代表n (int& tmp = n;)
}

int Count2()
{
	static int n = 0;
	n++;
	return n; //在返回时,n将值赋给一个临时变量 (int tmp = n;)
}

int main()
{
	int& ret1 = Count1();
	const int& ret2 = Count2(); //返回的临时变量具有常性,因此加上 const 才可以通过编译
	return 0;
}
```

关于引用作为函数返回值有最大的优点就是`提升效率`

> 由于不需要再创建额外变量,只要引用作为返回类型或者参数,提高程序运行的效率


---
`引用中的不慎会导致的错误`

```cpp
int& Add(int a, int b)
{
	int c = a + b;
	
	//解决方式
	//static int c = a + b;
	
	return c;
}
int main()
{
	int& ret = Add(1, 2);
	cout << ret << endl;
	cout << "Add(1,2) is : " << ret << endl; 
	
	return 0;
}


```
`为什么前面一有其他函数就会使得值变化呢?`

> 问题就在Add函数在栈区每调用完这个栈就会销毁(下次使用这块栈区可能会直接覆盖),但是c的值还在
> 当有函数在此栈区运行时会使得c的别名变为随机值

所以要static修饰将其转移至静态区即可解决

---
**引用和指针的区别**

> 1.引用在初始化时引用一个实体后就不能再引用其他实体,而指针在没有const修饰时可以指向任何一个实体
> 2.没有空引用,但有空指针(NULL) 
> 3.有多级指针,没有多级引用 
> 4.访问实体时,指针需要解引用,引用编译器自己处理
> 5.引用相对于指针更安全
> 6.实际上在转换为汇编中指针和引用没有过大区别,效率几乎一样

---

# 四、内联函数
---
`inline`

```cpp
inline void Swap(int& a, int& b) 
{                               
	int tmp = a;                
	a = b;                       
	b = tmp;                    
}

int main()
{
	auto a = 3, b = 4;
	Swap(a, b);
	Swap(a, b);
	Swap(a, b);
	Swap(a, b);
	Swap(a, b);
	return 0;
}
```

内联函数的主要作用是`代替宏`
*汇编中直接将 main中的 call _Z4swapii 取代为swap函数内的算法*

但是内联函数有以下要点

> 1.频繁调用Swap是有消耗的
> 2.inline只是对编译器进行建议,如果函数内行数过多(多于20行左右),那么内联函数就不会使用

---
# 五、类和对象
---

## 1.类的大小如何计算?
---
```cpp
class Stack1//类中有成员函数和成员变量
{
public:       
	void Push(int x);
	void Pop();
	bool Empty(); 

private:
	int* _a;
	int _size;
	int _capacity; 
};

class Stack2 //类中只有成员函数
{
public:
	void Push(int x);
	void Pop();
	bool Empty();
};

class Stack3 //类中只有成员变量
{
	int* _a;
	int _size;
	int _capacity;
};

class Stack4 //类中什么也没有--空类
{};

int main()
{
	cout << sizeof(Stack1) << endl; //16
	cout << sizeof(Stack2) << endl; //1
	cout << sizeof(Stack3) << endl; //16
	cout << sizeof(Stack4) << endl; //1
	return 0;
}
```
根据输出,我们有以下问题

> **为什么对象中只存储成员变量,不存储成员函数?**
>  一个类实例化出多个对象,他们的成员变量可以存储不同的值 
>   但是他们调用的成员函数是同一个,若每个对象都存储成员函数,那么将会浪费空间

> **对于空类和只有成员函数的类,大小是1?**
> 开1字节不是为了存数据,而是占位,以表示该对象存在

而且要注意
`计算成员变量的和,并且要考虑对齐`

---

> const Date* p1;    const修饰指针指向的对象
Date const * p2;   const修饰指针指向的对象
Date* const p3;    const修饰指针本身

---

## 2.初始化列表
---
```cpp
class Date
{
public:
	Date(int year = 0, int month = 1, int day = 1)
		:_year(year)
		,_month(month)
		,_day(day)
		//初始化列表
	{
		//函数体内赋值
		/*_year = year;  
		_month = month;
		_day = day;*/
	}

private:
	int _year;
	int _month;
	int _day;
};
```

那么初始化列表有什么用呢?


初始化列表的初始化是分先后的,而不是一起同时初始化的,但其初始化顺序是
`定义成员变量的顺序`

```cpp
class A
{
public:
	A(int a = 10) //是构造函数,但不是无参数默认构造函数
		:_a(a)
	{}
private:
	int _a;
};

class B
{
public:
	B(int a, int ref)
		:_aobj()
		,_ref(ref)
		,_n(10)
	{}

private:
	//成员变量的声明
	A _aobj;     //没有默认构造函数
	int& _ref;   //引用
	const int _n; //const
};
```
可以理解为初始化列表是对象的成员变量定义的地方

```cpp
int& _b;
A _a;
const int _c;
//这些类型都需要初始化
//所以这些成员的初始化必须使用初始化列表
```
---

## 3.计算一个类产生了多少个对象(小实例)
---
```cpp
//static成员
//设计出一个类,计算这个类总共产生了多少对象
//如果仅仅只有一个全局变量来统计,那么将没有封装性,因此我们引入static成员
class A
{
public:
	A()
	{
		n++;
	}
	A(const A& a)
	{
		n++;
	}

	static int GetN() //没有this指针,函数中不能访问非静态的成员
	{
		return n;
	}
private:
	int _a;
	static int n; //声明 不是属于某个对象,是属于类的所有对象,属于这个类
};

int A::n = 0; //进行定义

A& f1(A& a)
//A f1(A& a)
//A f1(A a)  这三种因为引用的原因,所进行的调用次数不一样,可以影响n,会有不一样的结果
{
	return a;
}

int main()
{
	A a1;
	A a2;
	f1(a1);

	cout << a1.GetN() << endl;
	A a3;
	A a4;
	f1(a2);
	cout << a1.GetN() << endl;
	return 0;
}
```
### ①static的理解
上述代码有一关键点:突破类域
有两种方式

```cpp
A a1;
//突破类域的两种方式
a1.f4;
A::f4;
```

---
以下是static的函数调用理解

```cpp
void f1()
	{
		f2(); //this->f2();
	}
	static void f2()
	{}
	void f3()
	{}
	static void f4() //static 成员函数内部没有this指针,因此无法调用
	{
		f3();
	}
```
---

在C++11中也有一点关于static

```cpp
private:
	//非静态成员变量可以声明时给缺省值
	int _year = 0;
	int _month = 1;
	int _day = 1;
	int* p = (int*)malloc(4); //指针也可以在这里开辟空间
	A aa = 10;
	//static int n = 0;  static类型并不行
```

> C++11中可以在声明时给`非静态`变量缺省值

---


## 4.匿名对象
---
```cpp
class Date
{
public:
	Date()
	{
		cout << "Date" << endl;
	}
	void Print()
	{
		cout << "Print" << endl;
	}
	~Date()
	{
		cout << "~Date" << endl;
	}
private:
	int _year;
};
int main()
{
	Date d1; //声明周期持续到main结束
	d1.Print();

	//Date(); //匿名对象 生命周期只有一行
	Date().Print(); //只有这一行使用这个创建的对象,别人不需要使用,这一行开始自动调用构造,结束调用析构
	return 0;
}
```

> 匿名对象的生命周期只有一行,结束调用析构函数

---
## 5.全局变量,静态成员变量,调用构造和析构的顺序
---
```cpp
class A
{public:
	A(){cout << "A" << endl;}
	~A(){cout << "~A" << endl;}
};

class B
{public:
	B(){cout << "B" << endl;}
	~B(){cout << "~B" << endl;}
};

class C
{public:
	C(){cout << "C" << endl;}
	~C(){cout << "~C" << endl;}
};

class D
{public:
	D(){cout << "D" << endl;}
	~D(){cout << "~D" << endl;}
};

A a;
int main()
{
	B b;
	static D d;
	C c;
	

	// A  B  D  C
	//~C ~B ~D ~A
	return 0;
}
```

> 先构造全局,静态构造无优先级,和局部对象无区别
> 先照常析构局部对象,再释放静态,最后释放全局

---
## 6. Cpp/C 的内存分布
---
```cpp

int globalval = 1;  
static int staticglobalval = 1;//以上两种位于数据段(静态区),main函数之前就初始化,在哪里都能用,作用域是全局的
                               //区别,前者所有文件可用,后者只有当前"8process.cpp"可以用
int main()
{
	static int staticval = 1;  //位于数据段,运行到这里再初始化,作用域在main中,只能在main中使用
	int val = 1;               //局部变量,位于栈内

	int num[10] = { 1,2,3,4 }; //栈内
	char char2[] = "abcd";     //栈内
	const char* pchar3 = "abcd"; //指针位于栈内,指针指向的内容位于代码段(常量区)

	int* ptr1 = (int*)malloc(sizeof(int) * 4); //指针位于栈内,指向的内容位于堆
	int* ptr2 = (int*)calloc(4,sizeof(int));
	int* ptr3 = (int*)realloc(ptr2,sizeof(int) * 8);
	free(ptr1);
	free(ptr2);

	//数据段(静态区)存储全局数据,静态数据
	//代码段(常量区)存储可执行代码/只读常量

	return 0;
}
```
---
# 六、new和delete
---
## 1. malloc 、free 和 new 、delete
---
```cpp
class A
{
public:
	A()
	{
		cout << "A" << endl;
	}
	~A()
	{
		cout << "~A" << endl;
	}
};
int main()
{
	//C 函数
	int* p1 = (int*)malloc(sizeof(int));   //申请一个int4个字节的空间
	int* p2 = (int*)malloc(sizeof(int)*10);//为指向数组的指针开辟内存

	free(p1);
	free(p2);

	//C++  操作符
	int* p3 = new int;     //申请一个int4个字节的空间
	int* p4 = new int[10]; //为指向数组的指针开辟内存
	int* p5 = new int(10); //申请一个int4个字节的空间,初始化为10

	delete p3;
	delete[] p4;
	delete p5;

	//这样看来功能几乎类似,那么new和delete存在的意义是什么?
	A* p6 = (A*)malloc(sizeof(A));
	A* p7 = new A; //申请空间+构造函数初始化

	free(p6);
	delete p7;  //析构函数+销毁空间
	return 0;
}

```

> 对于内置类型和malloc,free作用相同
>对于自定义类型就有其他作用
>new申请空间后对于自定义类型还会调用构造函数初始化
>delete在销毁空间前还会调用自定义类型的析构函数

---

## 2.operator new 和 operator delete
---
`在使用方法上`

```cpp
A* p1 = (A*)malloc(sizeof(A));
A* p2 = (A*)operator new(sizeof(A));
```
**operator new和malloc使用方法一样**

---
`和new的区别`

```cpp
class A
{
	A()
	{}
private:
	int a;
};
int main()
{
	A* p1 = (A*)malloc(sizeof(A));
	A* p2 = (A*)operator new(sizeof(A)); //operator new 的用法和malloc一样

	//malloc申请出错时
	size_t size = 3;
	void* p4 = malloc(size * 1024 * 1024 * 1024); //申请3G内存,32位平台下申请绝对失败,观察失败后运行结果
	cout << p4 << endl; //输出p4地址的结果是000000,失败返回0

	//operator new 申请出错时
	//这部分语法暂时了解
	try
	{
		void* p5 = operator new(size * 1024 * 1024 * 1024);
		cout << p5 << endl;//失败抛异常
		operator delete(p5); //operator delete 和  free 没有区别,只是为了对应operator new
	}
	catch(exception& e)
	{
		cout << e.what() << endl; //输出bad allocation
	}
	return 0;
}
```

> 在观察operator new 和 operator delete内部,发现内部是用malloc和free实现的,在operator new内部多了一个申请出错输出bad allocation

总结

```cpp
malloc
operator new -> malloc + 失败抛异常
new          ->operator new + 构造函数

free
operator delete -> 和free没有区别
delete          -> free + 析构函数
```

---

## 3.定位new和delete
---
```cpp
int main()
{
	List* p1 = new List;
	delete p1;
	
	//模拟上面的行为,即,创建对象后调用构造,销毁对象后调用析构
	//显式调用构造函数和析构函数
	
	List* p2 = (List*)malloc(sizeof(List));
	new(p2)List(1);//调用构造函数,new(空间指针)类型(参数)

	p2->~List(); //调用析构函数
	operator delete(p2);

	return 0;
}
```

> 调用构造函数为 new(空间指针)类型(参数)
> 调用析构函数为 空间指针->析构函数名()

---

## 4.总结new,malloc...

>使用效果上: new会调用构造函数,失败了抛异常,malloc失败返回0
>概念性质上: new是一个操作符,malloc是一个函数
>使用方法上: new后面跟申请对象的类型,返回的是类型的指针;malloc参数传字节数,返回值为void*


---


