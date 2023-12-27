---
title: vector
date: 2023-10-21 21:03:17
tags: 
- 迭代器失效
- 模拟实现
- 数据结构
categories: 
- C++
cover: /pic/2.png
---



---
# 一、vector的注意事项

---

## 1. [] 和 at() 的越界访问

---

```cpp
	vector<int> v1{ 1,2,3,4 };

	//v1[4] = 5; //越界直接结束,assert判断

	try
	{
		v1.at(4) = 5; //抛异常
	}
	catch (...)
	{
		cout << "beyond" << endl;  //输出beyond
	}
```

---
## 2.迭代器失效问题

---

> 简单来说就是在迭代器创建后
> 使用了reverse,resize,insert,push_back等导致的增容
> 导致迭代器仍然指向已经被释放的旧空间

`一个应对迭代器失效的过程`

```cpp
    //删除容器中所有的偶数
	vector<int>::iterator it = v.begin();
	while (it != v.end())
	{
		if (*it % 2 == 0)
		{
			v.erase(it); 
		}
		//然后该怎么++it?
		
		++it;  //此时会报错,为什么?
		        //erase删除一个数据后,其他数据向前移动,移动后其实it已经指向我们想要遍历的下一个元素
		        //此时再++it就会导致错过部分数据

		else
		{
			++it;  //那么这样处理可以么?
			       //Linux环境下这样可以达到目的,但是vs编译环境自动检查且较为严格,仍然报错
		}
	}
	
	
    //正确做法
	vector<int>::iterator it2 = v.begin();
	while (it2 != v.end())
	{
		if (*it2 % 2 == 0)
			it2 = v.erase(it2); //正确做法 erase会返回删除的it的下一个位置的迭代器
		else
			++it2;  
	}
```

---

# 二、vector的模拟实现
---
有以下要点: 

> reserve中浅拷贝memmove问题
> 新型拷贝构造和重载赋值符写法
> insert时的迭代器失效


```cpp
namespace 9TSe
{
	template<class T>
	class vector
	{
	public:
		typedef T* iterator; 
		typedef const T* const_iterator;
		vector()
			:_start(nullptr)
			,_finish(nullptr)
			,_endofstorage(nullptr)
		{}

		//v2(v1)
		//vector(const vector<T>& v)
		//{
		//	_start = new T[v.capacity()];     //开辟新空间
		//	_finish = _start;
		//	_endofstorage = _start + capacity();

		//	for (size_t i = 0; i < v.size(); ++i)
		//	{
		//		*_finish = v[i];
		//		++_finish;
		//	}
		//}

		//拷贝构造的另一种方法
		vector(const vector<T>& v)
			:_start(nullptr)
			,_finish(nullptr)
			,_endofstorage(nullptr)
		{
			for (auto& e : v)
			{
				reserve(v.capacity());
				push_back(e);
			}
		}

		//vector<T>& operator=(const vector<T>& v)
		//{
		//	if (*this != &v) //防止自己赋值给自己
		//	{
		//		delete[] _start; //s1 = s2 s1有自己的空间,先释放
		//		_start = new T[v.capacity()];

		//		memcpy(_start, v._start, sizeof(T) * v.size());
		//	}
		//	return *this;
		//}

		//重载赋值运算符简便写法
		void swap(vector<T>& v)
		{
			::swap(_start, v._start); 
			::swap(_finish, v._finish);
			::swap(_endofstorage, v._endofstorage);
		}

		vector<T>& operator=(vector<T> v)
		{
			if (*this != &v)
				swap(v);		//这里不用库内自带的swap是因为,其会生成三个深拷贝,有很大的代价
			return *this;
		}

		~vector()
		{
			delete[] _start;
			_start = _finish = _endofstorage = nullptr;
		}

		void reserve(size_t n)
		{
			if (n > capacity()) //增大容量时
			{
				size_t sz = size();
				T* tmp = new T[n];
				if (_start) //这里的用意是防止_start为空指针时扩容,会导致memcpy出错(memcpy内不能存在空指针)
				{
					//memcpy(tmp, _start, sizeof(T) * size()); //这个memcpy是浅拷贝,当T为string时,会导致指针指向同一位置,释放两次同一空间
					for (size_t i = 0; i < sz; ++i)
					{
						tmp[i] = _start[i];
					}
					delete[] _start; 
				}
				_start = tmp;
				_finish = tmp + sz; //如果这里赋予 tmp + size() 会出错 
								    //sz的计算方式是_finish-_start
									//_start已经指向新空间了,finish还指向旧空间
				_endofstorage = tmp + n;//这里如果赋予tmp + capacity() 也会出错,与上同理
			}
			
		}

		void resize(size_t n, const T& v = T()) //这里的T()为T类型的默认缺省值,有int() double() 等都有
									   //int() double()等内置类型,只是专门创建了此形式的东西
									   //string ,vector 是调用构造函数
		{
			if (n < size()) //缩容
			{
				_finish = _start + n;
			}
			else
			{
				if (n > capacity())
				{
					reserve(n);
				}

				while (_finish < _start + n)
				{
					*_finish = v;
					++_finish;
				}
			}
		}

		void push_back(const T& x)
		{
			if (_finish == _endofstorage) //扩容
			{
				size_t newcapacity = capacity() == 0 ? 2 : capacity() * 2;
				reserve(newcapacity);
			}
			*_finish = x;
			++_finish;
		}


		void insert(iterator pos, const T& v) //这里会有迭代器失效问题
		{
			assert(pos <= _finish);
			if (_finish == _endofstorage)//pos会扩容后失效
			{
				size_t n = pos - _start; //因此要记录之间的距离,以明确新pos指针指向的位置

				size_t newcapacity = capacity() == 0 ? 2 : capacity() * 2;
				reserve(newcapacity);

				pos = _start + n; //设置新pos
			}

			iterator end = _finish - 1;
			while (end >= pos)   //移动数据
			{
				*(end + 1) = *end;
				end--;
			}
			*pos = v;
			++_finish;
		}

		iterator erase(iterator pos)
		{
			assert(pos < _finish&& _start != _finish);
			iterator it = pos;
			while (it < _finish)
			{
				*it = *(it + 1);
				++it;
			}
			_finish--;
			return pos;
		}


		size_t size()const
		{
			return _finish - _start;
		}

		size_t capacity()const
		{
			return _endofstorage - _start;
		}

		iterator end()
		{
			return _finish;
		}

		iterator begin()
		{
			return _start;
		}

		const_iterator end()const
		{
			return _finish;
		}

		const_iterator begin()const
		{
			return _start;
		}

		T& operator[](size_t i)
		{
			assert(i < size());
			return *(_start + i);
		}

		const T& operator[](size_t i)const
		{
			assert(i < size());
			return *(_start + i);
		}

		void pop_back()
		{
			assert(_start < _finish);
			_finish--;
		}

	private:
		iterator _start;
		iterator _finish; //这个指针指向的是最后一个数据的下一个数据的位置
		iterator _endofstorage;
	};
}
```

