---
title: 智能指针
date: 2023-10-21 21:08:22
tags:
- 智能指针
- 模拟实现
categories: 
- C++
cover: /pic/2.png
---

---
# 1. 智能指针介绍

## 1.1 为什么使用智能指针
---
根据所需构建一个智能指针

> RAII (Resource Acquisition Is Initialization) 是一种利用对象生命周期来控制程序资源
智能指针就是依靠RAII实现的

```cpp
template<typename T>
class Smart_Ptr
{
public:
	Smart_Ptr(T* ptr)
		:_ptr(ptr)
	{}


	~Smart_Ptr()
	{
		if (_ptr)
		{
			cout << "deleted" << endl;
			delete _ptr;
			_ptr = nullptr;
		}
	}

	T& operator*()
	{
		return *_ptr;
	}

	T* operator->()
	{
		return _ptr;
	}

private:
	T* _ptr;
};
```
```cpp
int Fun2()
{
	int m, n;
	cin >> m >> n;
	if (n == 0)
		throw invalid_argument("/0 error");
	return m / n;
}

void Fun1()
{
	int* p = new int;

	//此时有内存泄漏的风险
	cout << Fun2() << endl;
	delete p; //以下是解决方案
	
	//1.重新抛异常解决
	try
	{
		cout << Fun2() << endl;
	}
	catch (...) //一般接受类型为...,否则其他类型可能接受不到
	{
		delete p;
		throw;
	}
	delete p;
	//但是如果: new了很多对象,new失败了抛异常,就很难处理

	//2.智能指针解决
	Smart_Ptr<int> sp(p); //直接将指针交给智能指针,任其生命周期结束调用析构函数从而达到释放空间的目的
	cout << Fun2() << endl;

	//出了这个作用域sp生命周期结束进行析构
}

void main()
{
	try
	{
		Fun1();
	}
	catch (exception& e)
	{
		cout << e.what() << endl;
	}
}
```

---

## 1.2 智能指针遇到的问题
---





```cpp
void main()
{
	Smart_Ptr<int> sp1(new int);
	*sp1 = 20;

	Smart_Ptr<pair<int, int>> sp2(new pair<int, int>);
	sp2->first = 30;
	sp2->second = 40;

	//但是不仅仅这么简单
	Smart_Ptr<int> sp3(new int);
	Smart_Ptr<int> sp4(new int);
	sp3 = sp4; 
	//如果赋值后两个指针指向同一空间
	//析构时重复释放一块空间两次
}
```
---
有三种解决方法

> C++98 : 管理权转移
> C++11 : 防拷贝 , 计数

---

# 2. 解决方案
## 2.1 管理权转移(auto_ptr)
---

```cpp
namespace 9TSe
{
	template<typename T>
	class auto_ptr
	{
	public:
		auto_ptr(T* ptr)
			:_ptr(ptr)
		{}

		//p1(p2)
		auto_ptr(auto_ptr<T>& ap)
			:_ptr(ap._ptr)
		{
			ap._ptr = nullptr; //直接将p2置空,将管理权全部转移给p1
		}

		auto_ptr<T>& operator=(const auto_ptr<T>& ap)
		{
			if (this != &ap)
			{
				if (_ptr)
					delete _ptr; //释放原本指向的空间

				_ptr = ap._ptr;  //指向新空间
				ap._ptr = nullptr;  //管理权全部转移给_ptr
			}
			return *this;
		}

		~auto_ptr()
		{
			if (_ptr)
			{
				cout << "deleted" << endl;
				delete _ptr;
				_ptr = nullptr;
			}
		}

		T& operator*()
		{
			return *_ptr;
		}

		T* operator->()
		{
			return _ptr;
		}

	private:
		T* _ptr;
	};
}
```

> 当同时指向一块空间时
> 将一个指针赋空,权力全部转移给另一个指针

>但同时又有问题,赋空的指针无法进行访问,会导致空指针操作
>当代码量大时难以发现



```cpp
int main()
{
	bsy::auto_ptr<int> ap1(new int);
	bsy::auto_ptr<int> ap2 = ap1;    //直接将ap1置空,管理权全部转移给ap2

	//但是缺陷很大,有时会没有注意
	//*ap1 = 3; //空指针访问,导致出错
	return 0;
}
```

---

## 2.2 防拷贝(unique_ptr)

```cpp
namespace 9TSe
{
	template<typename T>
	class unique_ptr
	{
	public:
		unique_ptr(T* ptr)
			:_ptr(ptr)
		{}

		//p1(p2)
		unique_ptr(unique_ptr<T>& up) = delete;

		unique_ptr<T>& operator=(const unique_ptr<T>& up) = delete;

		~unique_ptr()
		{
			if (_ptr)
			{
				cout << "deleted" << endl;
				delete _ptr;
				_ptr = nullptr;
			}
		}

		T& operator*()
		{
			return *_ptr;
		}

		T* operator->()
		{
			return _ptr;
		}

	private:
		T* _ptr;
	};
}
```
---

> 根本上解决了问题
> 但是如果有需要拷贝的场景,就没法使用

```cpp
int main()
{
	bsy::unique_ptr<int> up1(new int);
	bsy::unique_ptr<int> up2 = (new int);
	//bsy::unique_ptr<int> up2 = up1; 
	//up1 = up2;					//直接报错,不让拷贝构造
	return 0;
}
```

---

## 2.3 计数(shared_ptr)

### 2.3.1 基本模拟实现

```cpp
namespace 9TSe
{
	template<typename T>
	class shared_ptr
	{
	public:
		shared_ptr(T* ptr)
			:_ptr(ptr)
			,_pcount (new int(1))
		{}

		shared_ptr(shared_ptr<T>& sp)
			:_ptr(sp._ptr)
			,_pcount(sp._pcount)
		{
			++(*_pcount);
		}

		~shared_ptr()
		{
			if (--(*_pcount) == 0 && _ptr)
			{
				cout << "delete : " << _ptr << endl;
				delete _ptr;
				_ptr = nullptr;

				delete _pcount;
				_pcount = nullptr;
			}
		}

		//sp1 = sp2
		shared_ptr<T>& operator=(shared_ptr<T>& sp)
		{
			if (this != &sp)
			{
				if (--(*_pcount) == 0) 
				//当sp1原本管的空间在sp1走后便没人管理时,释放原本空间
				{
					delete _pcount;
					delete _ptr;
				}

				_ptr = sp._ptr;
				_pcount = sp._pcount;
				++(*_pcount);
			}
			return *this;
		}

		T& operator*()
		{
			return *_ptr;
		}

		T* operator->()
		{
			return _ptr;
		}
	private:
		T* _ptr;
		int* _pcount;
	};
}
```

> 通过计数来判断是否需要析构,是较好的处理方法

```cpp
int main()
{
	bsy::shared_ptr<int> sp1(new int);
	bsy::shared_ptr<int> sp2(sp1);

	bsy::shared_ptr<int> sp3(new int);
	bsy::shared_ptr<int> sp4(sp3);
	bsy::shared_ptr<int> sp5(sp3);

	sp1 = sp3;
	return 0;
}
```
---

### 2.3.2 线程问题

> 由于智能指针涉及到公共资源，难免会在多线程下使用，那么此时就会出现线程安全问题。

```cpp
void test_share_ptr()
{
	shared_ptr<int> sp1(new int(1));

	thread t1([&]()
		{
			for (int i = 0; i < 10000; i++)
			{
				shared_ptr<int> sp2(sp1);
			}
		});

	thread t2([&]()
		{
			for (int i = 0; i < 10000; i++)
			{
				shared_ptr<int> sp3(sp1);
			}
		});

	t1.join();
	t2.join();

	cout << sp1.use_count() << endl;
}
```

由于线程问题导致得到的结果并不正确

---

### 2.3.3 线程问题的解决

> 加锁
2.不过，对于我们而言，放在类的对象中的锁这个做法是行不通的，因为这样构造出来的指针明明指向的是同一个结构，但是所谓的锁不是同一把锁，那么就算是加锁这个操作也是没有意义的
3.基于2的问题，我们需要的是不同对象拥有同一把锁，那么做法其实加入计数器的做法一样，都传入的是其指针

```cpp
shared_ptr(T* ptr)
	:_ptr(ptr)
	,_pcount(new int(1))
	,_pmtx(new mutex)
{}
 
void release()
{
	bool flag = false;
	_pmtx->lock();
	if (--(*_pcount) == 0)
	{
		delete _ptr;
		delete _pcount;
		flag = true;
	}
	_pmtx->unlock();
	if (flag) //如果是最后一个指针已经销毁了,那么销毁锁
	{
		delete _pmtx;
	}
}
 
shared_ptr(const shared_ptr<T>& sp)
	:_ptr(sp._ptr)
	, _pcount(sp._pcount)
	, _pmtx(sp._pmtx)
{
	_pmtx->lock();
	++(*_pcount);
	_pmtx->unlock();
}
 
shared_ptr<T>& operator=(const shared_ptr<T>& sp)
{
			
	if(sp._ptr!=_ptr)
	{
		release();
		_ptr = sp._ptr;
		_pcount=(sp._pcount);
		
		_pmtx->lock();
		++(*_pcount);
		_pmtx->unlock();
	}
	return *this;
}
```

```c
private:
		T* _ptr;
		int* _pcount;
		mutex* _pmtx;
```

---

### 2.3.4 循环引用问题

```cpp
struct ListNode
{
	~ListNode()
	{
		cout << "~ListNode()" << endl;
	}
	shared_ptr<ListNode> _next;
	shared_ptr<ListNode> _prev;
};
 
void test_share_ptr2()
{
	std::shared_ptr<ListNode> n1(new ListNode);
	std::shared_ptr<ListNode> n2(new ListNode);
 
	n1->_next = n2;
	n2->_prev = n1;
    cout << n1.use_count() << " " << n2.use_count() << endl;
}
```

---

## 2.4 循环引用解决方案(weak_ptr)

> weak_ptr 本质上是不计数的shared_ptr

```cpp
template<class T>
class weak_ptr
{
public:
	weak_ptr()
		:_ptr(nullptr)
	{}
 
	weak_ptr(const shared_ptr<T>& sp)
		:_ptr(sp.get())
	{}
 
	weak_ptr<T>& operator=(const shared_ptr<T>& sp)
	{
		_ptr = sp.get();
		return *this;
	}
 
 	//像指针一样
	T& operator*()
	{
		return *_ptr;
	}
 
	T* operator->()
	{
		return _ptr;
	}
 
public:
	T* _ptr;
};
```


---

# 3. 定制删除器

> 默认的删除其实只是针对指针，如果我们构造的智能指针指向一个特定的结构体，就无法删除

```cpp
template<class T>
struct DeleteArray
{
	void operator()(const T* ptr)
	{
		delete[] ptr;
	}
};

int main()
{
	//MY::test_share_ptr1();
	//MY::test_share_ptr2();
	std::shared_ptr<int> sp1(new int[10], DeleteArray<int>());
	std::shared_ptr<string> sp2(new string[10], DeleteArray<string>());
	std::shared_ptr<string> sp3(new string[10], [](string* ptr) {delete[] ptr; });
	std::shared_ptr<FILE> sp4(fopen("Test.cpp", "r"), [](FILE* ptr) {fclose(ptr); });
	return 0;
}

```

> 定制删除器，在库中的智能指针在构造时会传入删除器，随后传入内部自动删除指定结构体的内存，防止内存泄漏。

```cpp
template<class T>
struct Fclose
{
	void operator()(const T* ptr)
	{
		fclose(ptr);
	}
};
BSY::shared_ptr < FILE, Fclose<FILE>> n2(fopen("Text.cpp", "r"));
```

> 一般使用仿函数来定制删除器

---