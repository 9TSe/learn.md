---
title: 多态
date: 2023-10-21 21:05:26
tags:
- C++特性
categories: 
- C++
cover: /pic/2.png

---


---
# 一、虚函数
---
## 1.重载,重写(覆盖),重定义(隐藏)的对比

>重载:        两个函数在一个作用域,函数名相同,参数不同
>.
>重写(覆盖):  两个函数分别在基类和派生类的作用域
>函数名,返回类型,参数都必须相同(协变例外),两个函数必须是虚函数
>.
>重定义(隐藏): 两个函数分别在基类和派生类的作用域
>函数名相同,两个基类和派生类的同名函数不构成重写(覆盖)就是重定义

---
## 2.虚函数的重写
---

```cpp
class person
{
public:
	void BuyTicket()      //输出全为all
	//virtual void BuyTicket()  //正确的输出
	{
		cout << "all" << endl;
	}
};

class student : public person
{
public:
	void BuyTicket()
	//virtual void BuyTicket()
	{
		cout << "half" << endl;
	}
};

void Fun1(person& p) //注意引用  有子类赋值给父类(切割,切片)
{
	p.BuyTicket();
}

void Fun2(person* p) //指针也可以
{
	p->BuyTicket();
}

int main()
{
	student s;
	person p;
	Fun1(s);
	Fun1(p);

	Fun2(&s);
	Fun2(&p);
	return 0;
}

//virtual 关键字: 可以修饰成员函数,为了完成虚函数的重写,满足多态的条件之一
//             : 可以在零菱形继承中,去完成虚继承,解决数据冗余和二义性
//虽然关键字一样,但他们之间没有一点关联

//多态的两个条件:
//1.虚函数的重写
//2.父类对象的指针或引用去调用虚函数

//满足多态:跟指向对象有关,指向哪个对象调用的就是他的虚函数
//不满足多态:跟对象调用的类型有关,类型是什么调用的就是谁
```



---
### ①协变


> 基类与派生类虚函数的返回值类型不同
> 却仍然可以构成重写

```cpp
//基类虚函数返回类型为基类对象的指针或引用,派生类虚拟函数返回类型为派生类对象的指针或引用,称为协变
class A{};
class B : public A{};

class person
{
public:	
	virtual person* BuyTicket()  
	//virtual A* BuyTicket()    //而且不限于本父子类
	//virtual B* BuyTicket()	//但必须对应父子类,否则会出错
	{
		cout << "all" << endl;
		return nullptr;
	}
};

class student : public person
{
public:
	virtual student* BuyTicket() //如果虚函数返回类型是基类和派生类的指针或引用时,就会构成协变,此时无错误
	//virtual B* BuyTicket()        
	//virtual A* BuyTicket()         
	{
		cout << "half" << endl;
		return nullptr;
	}
};

void Fun1(person& p)
{
	p.BuyTicket();
}
int main()
{
	student s;
	person p;
	Fun1(s);
	Fun1(p);
	return 0;
}
```
---

### ②析构函数的重写

> 基类与派生类的函数名不同
> 但仍然可以构成重写

```cpp
class person
{
public:
	//virtual ~person()
	~person() //析构函数名被编译器处理为同一名称destruct
	{
		cout << "~person" << endl;
	}
};

class student : public person
{
public:
	//virtual ~student()
	~student()
	{
		cout << "~student" << endl;
	}
};

int main()
{
	person* p1 = new person;
	person* p2 = new student; //不加virtual会使得只调用person的析构
	                          //调用不了student的析构函数,会内存泄漏
	//virtual: 		`person  `student  `person  , 感觉到了继承,根据情况调用不同的"同名"函数
	//加上virtual,构成多态,调用的指针指向谁就调用谁的析构函数
	
	//没有virtual:  `person  `person ,            函数只调一个,知道有继承,但就是只能调一个
	//不加上virtual,不构成多态,调用的指针类型是谁,调用的就是谁的析构函数
	
	delete p1;
	delete p2;
	return 0;
}
```

---
### ③虚函数的一些点

>虚函数有一个指针的大小
>虚函数重写生效的只是内容,而不是参数

```cpp
class A
{
public:
	virtual void func(int val = 1)
	{
		cout << "A->" << val << endl;
	}
	virtual void test() //传入的是对象B指针,所以调用B中的func
	{
		func();
	}
};

class B : public A
{
public:
	void func(int val = 0) //重写生效的只是内容,而不是参数,所以缺省参数中 int val = 0 无效,所以 val == 1
	{
		cout << "B->" << val << endl;
	}
};

int main()
{
	A* p = new B;
	p->test();
	return 0;
}
```

---
### ④抽象类(纯虚函数)

```cpp
//在虚函数后面加上 =0 ,则这个函数就是纯虚函数
class car  //有了纯虚函数,这个类就被成为抽象类
{
public:
	virtual void drive() = 0;//纯虚函数,不需要实现,实现了不报错,但也没用
	//virtual void drive() = 0 
	//{
	//	cout << "aa" << endl;
	//}
};

class benz : public car
{
public:
	virtual void drive()
	{
		cout << "benz" << endl;
	}
};

int main()
{
	//car autocar; //抽象类实例化不出对象
	benz autoccar; //抽象类的子类,也实例化不出对象,继承了纯虚函数(没有重写)
	               //重写后即可以实例化出对象
	return 0;
}
```
纯虚函数的作用是

> 强迫重写
>表示抽象的类型(现实中没有对应实体的)

---

# 二、两个关键字
---
## 1.final

> 修饰虚函数:可以阻止函数被重写,final无法对类中非虚函数进行修饰
> 修饰类: 可以阻止类被继承

```cpp
class person
{
public:
	virtual void BuyTicket() final //此后,该函数不能再被重写
	{							   
		cout << "all" << endl;
	}
};

class student : public person
{
public:
	virtual void BuyTicket() //此时会报错,无法继承
	{
		cout << "half" << endl;
	}
};

class A final{};  //final后A不能被继承
class B : public A{};
```

---
## 2.override

> 可以检查出虚函数是否被成功重写

```cpp
class A
{
public:
	void out()
	//virtual void out()
	{
		cout << "111" << endl;
	}
};

class B : public A
{
public:
	virtual void out() override //如果检查出错则无法编译通过
	{
		cout << "222" << endl;
	}
};
```

---

# 三、多态的原理

---
## 1.原理
```cpp
class base
{
public:
	virtual void fun1()
	{
		cout << "base:fun1()" << endl;
	}
	virtual void fun2()
	{
		cout << "base:fun2()" << endl;
	}
private:
	int _b = 1;
};

class derive : public base
{
public:
	virtual void fun1()
	{
		cout << "derive:fun1()" << endl;
	}
private:
	int _d = 1;
};

void function1(base& b) //运行时,到指向对象的虚表中,查找对应的虚函数的地址
{
	b.fun1();
}

//多态如何实现的指向谁就调用谁的虚函数?
//多态是在 运行时 到指向对象的虚表中,查找要调用虚函数的地址来调用

void function2(base b) //编译时,直接确定通过p的类型确定要调用函数的地址
{
	b.fun1();
}

int main()
{
	base b1;
	cout << sizeof(base) << endl; //16   8+4+对齐 == 16

	//监视窗口中的_vfptr就是 虚函数表指针:简称虚表指针
	//其本质是 指针数组(虚函数指针)

	derive d1;
	function1(b1);
	function1(d1);

	return 0;
}
```

> 类里面有一个虚表,虚表里面存的指针指向虚函数
> 重写了,子指针改变指向
> 没重写,父指哪,子指哪

![在这里插入图片描述](/img/3.2.png)


```c
监视窗口中的_vfptr就是 虚函数表指针:简称虚表指针
其本质是 指针数组(虚函数指针)
一般来说对象前八个字节存的虚表指针
虚表指针数组最后一个指针为0x0000000000000000 表示这个虚函数数组结束了
```


![在这里插入图片描述](/img/3.3.png)


---
## 2.验证虚函数和虚表的位置都是在代码段

---

```cpp
class base
{
public:
	virtual void fun1()
	{
		cout << "base::fun1()" << endl;
	}
	virtual void fun2()
	{
		cout << "base::fun2()" << endl;
	}
	virtual void fun3()
	{
		cout << "base::fun3()" << endl;
	}
private:
	int _b = 1;
};

class derive : public base
{
public:
	virtual void fun1()
	{
		cout << "derive::fun1()" << endl;
	}
private:
	int _d = 2;
};

void test()
{
	base b1;
	printf("_vftptr:%p\n", *(double*)&b1);
	printf("_vftptr:%p\n", *(long long*)&b1);
	//printf("_vftptr:%p\n", (long long)b1);  直接强制转换语法不接受 取对象前八个字节

	int i = 0;
	int* p1 = &i;
	int* p2 = new int;
	const char* p3 = "hello";
	printf("栈变量:%p\n", p1);
	printf("堆变量:%p\n", p2);
	printf("代码段常量:%p\n", p3);
	printf("代码段函数地址:%p\n", &base::fun3);
	printf("代码段函数地址:%p\n", test);
}

int main()
{
	base b1;
	derive d1;

	test();
	return 0;
}
//vs2022没有显示不出fun3()的情况
//有虚函数的对象,前八个字节是虚表指针的地址
```
![在这里插入图片描述](/img/3.4.png)

---

# 四、动静态绑定

---

> 静态的绑定:编译时确定函数地址
> 动态的绑定:运行时到虚表中找虚函数的地址

> 编译时 是指: 高级语言转换为汇编语言时(初次检查错误)
> 运行时 是指: 程序开始运行,开始占用cpu和内存

```cpp
class base
{
public:
	virtual void fun1()
	{
		cout << "base::fun1()" << endl;
	}
private:
	int _b = 1;
};

class derive : public base
{
public:
	virtual void fun1()
	{
		cout << "derive::fun1()" << endl;
	}
private:
	int _d = 2;
};

void f1(int i) {}
void f2(double d) {}
int main()
{
	//静态的绑定 静态的多态 (静态:编译时确定函数地址)
	int i = 0;
	double d = 1.1;
	f1(i);
	f2(d);
	
	//动态绑定 动态的多态  (动态:运行时到虚表中找虚函数的地址)
	base* p = new base;
	p->fun1();
	p = new derive;
	p->fun1();
	return 0;
}

```

---
# 五、设计函数展现_vftptr所有函数

---

## 1. 单继承展现虚表

---
![在这里插入图片描述](/img/3.5.png)

```cpp
class base
{
public:
	virtual void fun1()
	{
		cout << "base::fun1()" << endl;
	}
	virtual void fun2()
	{
		cout << "base::fun2()" << endl;
	}
private:
	int b;
};

class derive : public base
{
public:
	virtual void fun1()
	{
		cout << "derive::fun1()" << endl;
	}
	virtual void fun3()
	{
		cout << "derive::fun3()" << endl;
	}
	virtual void fun4()
	{
		cout << "derive::fun4()" << endl;
	}
private:
	int d;
};

//void(*p)() 定义一个函数指针变量
typedef void(*VF_PTR)(); //函数指针类型

//打印虚表
//void PrintVFTable(VF_PTR* pTable)
void PrintVFTable(VF_PTR pTable[])
{
	for (size_t i = 0; pTable[i] != 0; ++i) //!=nullptr也可以
	{
		printf("vftabel[%d]:%p->", i, pTable[i]);
		VF_PTR f = pTable[i];
		f();
	}
	cout << endl;
}

int main()
{
	base b;
	derive d;

	//base的虚表
	PrintVFTable((VF_PTR*)(*(long long*)&b));
	PrintVFTable((VF_PTR*)(*(void**)&b));

	//derive的虚表
	//而监视窗口只有继承下来的fun1和fun2
	PrintVFTable((VF_PTR*)(*(long long*)&d));
	PrintVFTable((VF_PTR*)(*(void**)&d));
	return 0;
}
```
![在这里插入图片描述](/img/3.6.png)

---
## 2.多继承展现虚表

---

```cpp
//多继承
class base1
{
public:
	virtual void fun1()
	{
		cout << "base::fun1()" << endl;
	}
	virtual void fun2()
	{
		cout << "base::fun2()" << endl;
	}
private:
	int b1;
};

class base2
{
public:
	virtual void fun1()
	{
		cout << "base::fun1()" << endl;
	}
	virtual void fun2()
	{
		cout << "base::fun2()" << endl;
	}
private:
	int b2;
};

class derive : public base1,public base2
{
public:
	virtual void fun1()
	{
		cout << "derive::fun1()" << endl;
	}
	virtual void fun3()
	{
		cout << "derive::fun3()" << endl;
	}
private:
	int d;
};

//typedef void (*VF_PTR)();
using VF_PTR = void(*)(); 
void PrintVFTable(VF_PTR pTable[])
{
	for (size_t i = 0; pTable[i] != nullptr; ++i)
	{
		printf("vftabel[%d]:%p->", i, pTable[i]);
		VF_PTR f = pTable[i];
		f();
	}
	cout << endl;
}

int main()
{
	derive d;
	//base1 //监视窗口中只有fun1和fun2
	PrintVFTable(( VF_PTR*)(*(void**)&d));
	//我们发现derive的fun3存在于base1的虚表中

	//base2 //监视窗口也有fun1和fun2
	PrintVFTable((VF_PTR*)(*(void**)((char*)&d + sizeof(base1))));

	return 0;
}

```
![在这里插入图片描述](/img/3.7.png)

> 子类独有的函数，存在第一个虚表中

关于函数指针数组指针的用法

> 第一种 typedef void(*PFUNC)(); 
>第二种 using VF_PTR = void(*)();



留下了一个问题

> 在64位平台下base1 和 base2的函数内部只要操作不同,就会导致 base2无法依次访问时遇见nullptr停止,但是从虚表开头开始遍历却没有问题
> 32位平台下并没有这种问题

---
# 六、继承和多态

---
## 1.几个关于继承和多态的点
```cpp
1.inline函数不可以是虚函数,inline函数没有地址,无法把地址放到虚函数表中
2.静态成员不可以是虚函数,静态成员函数没有this指针,使用 类型::成员函数 的调用方式无法访问虚表,所以静态成员函数无法放进虚函数表
3.构造函数不可以是虚函数,对象中的虚函数表指针是在构造函数初始化列表阶段才初始化的
4.对象访问普通函数快还是虚函数快?
如果是普通对象一样快;  如果是指针对象或者是引用对象,普通函数快,构成多态,运行时调用虚函数需要需要到虚表中去查找;
```

---

## 2.继承多态的大小

```cpp
class A
{
public:
	A(int x = 1) 
	{
		cout << "aa" << endl;
	}
	virtual void Aa(){}
};

class B :virtual public A
{
public:
	B(int x = 0) 
	{
		cout << "bb" << endl;
	}
	virtual void Aa(){}
};
int main()
{
	B b;
	A a;
	cout << sizeof(B) << endl;
	printf("%p\n", &A::Aa);
	printf("%p\n", &B::Aa); //两个地址不一样,触发了重写
	return 0;
}
//虚继承算出来大小是24 : virtual两个指针 8+8, 虚表+8 = 24
//不用虚继承大小是  8 :  virtual算指针8字节
```
---

## 3.一道题

计算输出结果
```cpp
class A 
{
public:
    A(const char* s) 
    { 
        cout << s << endl; 
    }
};

class B :virtual public A
{
public:
    B(const char* s1, const char* s2)
        :A(s1) 
    { 
        cout << s2 << endl; 
    }
};

class C :virtual public A
{
public:
    C(const char* s1, const char* s2) 
        :A(s1) 
    { 
        cout << s2 << endl; 
    }
};

class D :public B, public C
{
public:
    D(const char* s1,const char* s2,const char* s3,const char* s4) 
        :B(s1, s2), C(s1, s3), A(s1)
    {
        cout << s4 << endl;
    }
};

int main() {
    D* p = new D("class A", "class B", "class C", "class D");
    delete p;
    return 0;
}
```

1.A只有一个，所以A只构造一次,优先A构造
2.初始化列表优先于函数体，D最后
3.构造按继承顺序，即使先C(s1，s3)，B(s1，s2) 也是B先输出，C再输出
所以结果为ABCD

---