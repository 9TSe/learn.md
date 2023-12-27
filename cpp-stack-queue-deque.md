---
title: stack,queue,deque
date: 2023-10-21 21:03:45
tags:
- 适配器
- 数据结构
- 模拟实现
categories: 
- C++
cover: /pic/2.png
---

---
# 一、适配器和迭代器
---

> 迭代器模式 是STL中的迭代器在封装后不暴露内部细节，使得上层使用时达到一种统一的方法访问
>适配器模式 是通过原内容封装转换出形式

stack 和 queue 通过容器适配转换出来的,不是原生实现的

---

# 二、stack
---
`stack没有迭代器`

```cpp
namespace bsy
{
	template<class T , class Container> //stack类模拟实现
	class stack
	{
	public:
		void push(const T& x)
		{
			_con.push_back(x);
		}

		void pop()
		{
			_con.pop_back();
		}

		size_t size()
		{
			return _con.size();
		}

		bool empty()
		{
			return _con.empty();
		}

		T& top()
		{
			return _con.back();
		}

	private:
		Container _con;
	};

	void test_stack1()
	{
		//stack底层可以是链表也可以是vector
		//stack<int, vector<int> > st;
		stack<int, list<int> > st;
		st.push(1); //stack没有push_back
		st.push(2);
		st.push(3);
		st.push(4);
		while (!st.empty())
		{
			cout << st.top() << " ";
			st.pop();
		}
		cout << endl;
	}
}

```

---
# 三、queue

---
`queue没有迭代器`
```cpp
namespace 9TSe
{
	template<class T, class Container = deque<T>>  //vector无法实现,vector没有前插前删操作,效率太低,一般是list作为底层
	class queue
	{
	public:
		void push(const T& x)
		{
			_con.push_back(x);
		}

		void pop()
		{
			_con.pop_front();  
		}

		size_t size()
		{
			return _con.size();
		}

		bool empty()
		{
			return _con.empty();
		}

		T& front()
		{
			return _con.front();
		}

		T& back()
		{
			return _con.back();
		}
		
	private:
		Container _con;
	};
	
	void test_queue1()
	{
		queue<int, list<int> > que;
		que.push(1);
		que.push(2);
		que.push(3);
		que.push(4);
		while (!que.empty())
		{
			cout << que.front() << " ";
			que.pop();
		}
		cout << endl;
	}
}
```
---

# 四、deque
---

> vector不支持前删前插,list不支持随机访问,但是有deque(双端队列)是其优点的集合
> deque有迭代器

```cpp
void test_deque1() //基本使用方法
{
	deque<int> d;
	d.push_back(1);
	d.push_back(2);
	d.push_back(3);
	d.push_back(4);
	d.push_front(0);
	d.push_front(-1);
	d.pop_front();
	for (size_t i = 0; i < d.size(); ++i)
	{
		cout << d[i] << " ";
	}
	cout << endl;
	//deque既可以支持头插头删,也可以支持下标随机访问
}

//那为什么集大成者却没有完全替代上述两种容器呢?
//效率上还是不如vector 和 list

void test_deque2() //测试效率差距
{
	deque<int> d;
	vector<int> v;
	const int n = 1000000;
	int* a1 = new int[n];
	int* a2 = new int[n];

	srand(time(0));
	for (size_t i = 0; i < n; ++i)
	{
		int x = rand();
		d.push_back(x);
		v.push_back(x);
	}

	size_t begin1 = clock();
	sort(d.begin(), d.end());
	size_t end1 = clock();

	size_t begin2 = clock();
	sort(v.begin(), v.end());
	size_t end2 = clock();

	cout <<"deque : " << end1 - begin1 << endl; //250
	cout <<"vector : " << end2 - begin2 << endl; //50

	//约有五倍的效率差
	//随机访问的效率无法替代vector(sort底层用下标访问)
	//头尾插入删除的效率还可以,stack和queue没有使用到它的随机访问
}
```

> deque的原理是一小段数组一小段数组组成的,形状可以理解为list< vector > 但不是,list不支持随机访问
>由中控管理的指针数组,存储每一个小数组的地址,且该指针数组会增容,所以数据量大的话,效率就低了
>迭代器由四个部分组成 cur(当前位置),first(当前小数组的头位),last(当前小数组的末尾),node(中控管理的指针数组的当前访问数组的指针)
>总之,一般来说deque不行
	
---