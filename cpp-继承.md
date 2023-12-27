---
title: 继承
date: 2023-10-21 21:06:02
tags:
- C++特性
categories: 
- C++
cover: /pic/2.png
---


---
# 一、继承方式和访问方式
---
注意:`友元不能继承`,`静态成员只会在父类创造一次`
> 父类如果用private修饰的成员,子类无法访问
>protected修饰的子类可以访问,但除此之外不可访问

> 继承方式为private,struct为public ,一般继承方式都为public
> 基类的成员在子类的访问方式 = ==Min(成员在基类的访问限定符,继承方式)

---
# 二、赋值兼容规则
---

```cpp
int main()
{
	//1.子类对象可以赋值给父类对象/指针/引用
	person p;
	student s;
	p = s;//对象赋值
	s = p;//不行
	
	//子类赋值给父类,通过切割(切片)可以进行赋值
	person* ptr = &s; //指针
	person& ref = s;  //引用
	
	//student* sptr = &p;			//父类赋值给子类,不行
	
	student* sptr = (student*)&s; //ptr指向的本来就是子类,强制转换下可以赋值	
	
	sptr = (student*)&p; //强行转换可以运行,但容易导致越界访问,访问到子类独有的类
	
	//也可以通过C++11中的类型操作符来进行试探
	sptr = dynamic_cast<student*>(ptr); //注意此时父类必须有虚函数构成多态
	//如果ptr指向父类,返回nullptr
	
	//student& sref = p;//引用赋值,不行

	return 0;
}
```
---
# 三、默认函数的继承

---

```cpp
class person
{
public:
	person(const char* name = "peter")
		:_name(name)
	{
		cout << "person()" << endl;
	}

	person(const person& p)
		:_name(p._name)
	{
		cout << "person(const person& p)" << endl;
	}

	person& operator=(const person& p)
	{
		cout << "person& operator=(const person& p)" << endl;
		if (this != &p)
		{
			_name = p._name;
		}
		return *this;
	}

	~person()
	{
		cout << "~person()" << endl;
	}

protected:
	string _name;
};


class student : public person
{
public:
	student(const char* name, int num)
		://_name(name) 父类归父类管,所以这样初始化错误
		person(name) //直接调用父类的构造就行了
		, _num(num)
	{
		cout << "student()" << endl;
	}

	student(const student& s)
		://_name(s._name)
		person(s) //同上
		,_num(s._num)
	{
		cout << "student(const student& s)" << endl;
	}

	student& operator=(const student& s)
	{
		cout << "student& operator=(const student& s)" << endl;
		if (this != &s)
		{
			//operator=(s); 此时会无限递归,自己调用自己,要声明域才行
			person::operator=(s);
			_num = s._num;
		}
		return *this;
	}

	~student() //子类的析构函数和父类的析构函数构成 隐藏(重定义) ,他们的名字会被编译器同一处理成destructor 
	{
		//person::~person(); //派生类没必要再调用父类的析构函数,析构的顺序不对,不符合栈形式
		//结束时会自动调用父类的析构函数,因为这样才能保证先析构子类,再析构父类
		//并且如果再调用会导致重复释放同一块空间
		cout << "~student()" << endl;
	}

private:
	int _num;
};

int main()
{
	student s1("jack", 18);
	student s2(s1);
	student s3("rose", 19);
	s1 = s3;
	return 0;
}
```
---

# 四、不能被继承的类
---

> 1.将父类全部私有,使得子类无法调用父类的析构
> 2.利用final关键字

```cpp
class A final{public:A(){}};

class B :public A{}; //此时B就无法继承A
```

---
# 五、菱形继承
---

## 1.菱形继承的问题

```cpp
//单继承:一个子类只有一个直接定义父类时称为单继承
class person{};
class student : public person{};
class teacher : public student{};

//多继承:一个子类有两个以上或以上直接父类时称为多继承
class student{};
class teacher{};
class assistant : public student,public teacher{};

//菱形继承
                          class person {};

class student : public person {};    class teacher : public student {};

           class assistant : public student, public teacher {};

//菱形继承的问题是:数据冗余和二义性
```
>菱形继承时有两个问题
> 数据冗余:即派生类有多份相同类型的数据
> 二义性:   即因为有多相同类型的数据而导致对数据处理时,不知道要处理谁

```cpp
class person
{
public:
	string _name;
};

class student :virtual public person //中间人都加上virtual 即可解决
{
protected:
	int _num;
};

class teacher :virtual public person //中间人都加上virtual 即可解决
{
protected:
	int _id;
};

class assistant : public student, public teacher
{
protected:
	string _course;
};

void main()
{
	assistant a;
	a._name = "peter"; //此时二义性导致不知道要赋给哪个_name,加上关键词virtual即可解决二义性和冗余性

	//显示指定访问可以解决,但是冗余性还是没有解决(两个name)
	a.student::_name = "xxx";
	a.teacher::_name = "yyy";
}
```
---

## 2.菱形继承virtual解决的原理

---

```cpp
class A
{
public:
	int _a;
 //int _a[10000];
};

class B :virtual public A
{
public:
	int _b;
};

class C :virtual public A
{
public:
	int _c;
};

class D : public B, public C
{
public:
	int _d;
};

int main()
{
	D d;
	cout << sizeof(d) << endl;

	d.B::_a = 1;
	d.C::_a = 2;
	d._b = 3;
	d._c = 4;
	d._d = 5;
	d._a = 6;
	return 0;
}
//所以实际上,不到万不得已,不要把类的关系设计为菱形继承
//virtual解决了冗余性和二义性,但是会有略微的效率损失
//这个例子size变大了,但是如果父类成员size很大,那么在virtual继承后会减小很多
```
![在这里插入图片描述](/img/3.8.png)

---

# 六、继承和组合

---

```cpp
class A{};
class B : public A{};   //继承是白箱复用,父类对子类的是透明的,但是一定程度上破坏了父类的封装,继承的类是一种高耦合

class C{};
class D					//组合是黑箱封装,C对D不透明,C保持他的封装,组合的类耦合度更低
{C c;};

//都完成了类层次的复用
//综合来看,还是组合合适

//符合 is-a  使用继承  ,eg:奔驰是车
//符合 has-a 使用组合  ,eg:车有轮子
//如果都可以,优先使用组合
```
---