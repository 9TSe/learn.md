---
title: priority_queue
date: 2023-10-21 21:04:05
tags:
- 数据结构
- 模拟实现
categories: 
- C++
cover: /pic/2.png
---
---
# 一、priority_queue简介

---

> priority_queue也是适配出来的
>其与queue不同点在于能够排序,因为底层是由heap实现的,并且
```cpp
#include<queue>
#include<functionnal> //默认仿函数 greater / less

priority_queue<int> pq; //此时输出默认顺序是降序的,大堆
priority_queue<int, vector<int>, greater<int>> pq;
```

---

# 二、模拟实现
---

> 仿函数的使用,向上向下调整算法


```cpp
namespace 9TSe
{
	template<class T,class Container = vector<T>,class Compare = less<T>> 
	class priority_queue
	{
	public:

		void AdjustUp(int child)
		{
			Compare com;
			int parent = (child - 1) / 2;
			while (child > 0)
			{
				//if (_con[child] > _con[parent])
				if(com(_con[parent],_con[child]))
				{
					swap(_con[child], _con[parent]);
					child = parent;
					parent = (child - 1) / 2;
				}
				else
					break;
			}
		}

		void AdjustDown(int root)
		{
			Compare com;
			int parent = root;
			int child = parent * 2 + 1;
			while (child < _con.size())
			{
				//选出较大的孩子
				//if ( child + 1 < _con.size() && _con[child + 1] > _con[child] )
				if ( child + 1 < _con.size() && com(_con[child], _con[child + 1]))
				{
					child++;
				}

				//if (_con[child] > _con[parent])
				if (com(_con[parent],_con[child]))
				{
					swap(_con[child], _con[parent]);
					parent = child;
					child = parent * 2 + 1;
				}
				else
				{
					break;
				}

			}
		}

		void push(const T& x)
		{
			_con.push_back(x);
			AdjustUp(_con.size()-1);
		}

		void pop()
		{
			swap(_con[0], _con[_con.size() - 1]);
			_con.pop_back();

			AdjustDown(0);
		}

		T& top()
		{
			return _con[0];
		}

		size_t size()
		{
			return _con.size();
		}

		bool empty()
		{
			return _con.empty();
		}
	private:
		Container _con;
	};
	
	//仿函数(函数对象)
	template <class T>
	struct less
	{
		bool operator()(const T& x1, const T& x2)
		{
			return x1 < x2;
		}
	};

	template <class T>
	struct greater
	{
		bool operator()(const T& x1, const T& x2)
		{
			return x1 > x2;
		}
	};
}
```

---
# 三、仿函数,lambda表达式,sort,priority_queue

---
可以通过仿函数和lambda表达式实现
sort的升序降序

```cpp
sort(t.begin(),t.end()) //升序
sort(t.begin(),t.end(), [](auto a, auto b){return a > b; }); //变为降序
sort(t.begin(),t.end(), std::greater<int>()); 				 //降序
```

那么priority_queue

```cpp
priority_queue<int> pq; //降序
priority_queue<int, vector<int>, std::greater<int>> pq; //升序

priority_queue<int,vector<int>,decltype([](auto x,auto y){return x > y;})>pq;
 //这段代码只有C++20才支持,C++20之前不支持在priority内传入lambda表达式的类
```

值得注意的是,sort和priority_queue的第三个参数并不相同
sort传的是对象(函数,仿函数中的结构体对象,注意有括号)
priority_queue传入的是类,仿函数后面没括号,是一个类型名

---