---
title: list
date: 2023-10-21 21:03:27
tags:
- 迭代器失效
- 数据结构
- 模拟实现
categories: 
- C++
cover: /pic/2.png
---


---
# 一、list要注意事项

---

## 1.迭代器失效
---

> list没有增容销毁空间这一说,因此insert等插入不会导致失效
>erase也是返回下一个位置的迭代器

```cpp
		if (*it % 2 == 0)
			it = l.erase(it); //erase 会导致迭代器失效
			                  //空间被erase后不能再进行操作,即++,*;
		else
			++it;
```

---
## 2.splic(连接两个链表)
---

```cpp
	list<int> l;
	l.push_back(1);
	l.push_back(2);
	l.push_back(3);
	l.push_back(4);

	list<int> lt;
	lt.push_back(10);
	lt.push_back(20);
	lt.push_back(30);
	lt.push_back(40);
	list<int>::iterator pos = l.begin();
	++pos;

	l.splice(pos, lt); //在 l 链表中的pos位置插入lt链表
	Print_list(l); //1 10 20 30 40 2 3 4
	Print_list(lt); //lt插入过去就没了,成了l的一部分
```
---

# 二、模拟实现list
---

```cpp
namespace 9TSe
{
	template<class T>  //一个节点的类
	struct __list_node
	{
		T _data;
		__list_node* _next;
		__list_node* _prev;

		__list_node(const T& x = T())
			:_data(x)
			,_next(nullptr)
			,_prev(nullptr)
		{}
	};


	template<class T, class Ref, class Ptr> //迭代器的类
	struct __list_iterator
	{
		typedef __list_node<T> Node;
		typedef __list_iterator<T, Ref, Ptr> Self;

		Node* _node; //创建一个新节点

		__list_iterator(Node* node) //构造节点
			:_node(node)
		{}

		Ref operator*()  //应该还有一个const类型的此函数,但要再写一个就要再创建一个const_iterator类其他都赋值,这两个加个const
		{	             //会导致代码冗余  T&
			return _node->_data;
		}

		Ptr operator->()  //同上,套用模板可以简写  T*
		{
			return &(_node->_data); //&取其地址以->,所以实际上使用应为 dit->->_year,只不过被编译器特殊处理优化
		}


		Self& operator++() //前置++,做到能像vector一样++就访问下一个数据
		{
			_node = _node->_next;
			return *this;
		}

		Self& operator++(int) //后置++
		{
			//__list_iterator tmp(*this);  
			Self tmp = *this;

			//_node = _node->_next;
			++(*this);
			return tmp;
		}

		Self& operator--() //前置--
		{
			_node = _node->_prev;
			return *this;
		}

		Self& operator--(int) //后置--
		{
			Self tmp(*this);
			//_node = _node->_prev;
			--(*this);
			return *this;
		}

		bool operator!=(const Self& it) //迭代器循环时需要!=  记得加上const
		{
			return _node != it._node; //迭代器(两个指针)不相同
		}

		bool operator==(const Self& it)
		{
			return _node == it._node;
		}

	};


	template<class T> //链表的类
	class list
	{
	public:
		typedef __list_node<T> Node;
		typedef __list_iterator<T,T&,T*> iterator;
		typedef __list_iterator<T,const T&, const T*> const_iterator;

		iterator begin()
		{
			return iterator(_head->_next); //哨兵的下一位才是数据的第一位
		}

		iterator end()
		{
			return iterator(_head);  //哨兵就是最后一位
		}

		const_iterator begin()const
		{
			return const_iterator(_head->_next); 
		}

		const_iterator end()const
		{
			return const_iterator(_head);
		}

		list()
		{
			_head = new Node;
			_head->_next = _head;
			_head->_prev = _head;
		}

		list(const list<T>& lt)
		{
			_head = new Node;
			_head->_next = _head;
			_head->_prev = _head;

			/*const_iterator it = lt.begin();
			while (it != end())
			{
				push_back(*it);
				++it;
			}*/

			for (auto e : lt)
			{
				push_back(e);
			}
		}

		/*list<T>& operator=(const list<T>& lt)
		{
			if (*this != &lt)
			{
				clear();
				for (auto& e : lt)
				{
					push_back(e);
				}
			}
			return *this;
		}*/

		list<T>& operator=(list<T> lt)
		{
			swap(_head, lt._head); //交换指针指向的方向,即哨兵
			return *this;
		}


		~list()
		{
			clear(); //清楚除哨兵外的数据
			delete _head;
			_head = nullptr;
		}

		void clear()
		{
			iterator it = begin();
			while (it != end())
			{
				erase(it++); //迭代器失效是因为,it被销毁后访问++或*,先调用重载++,后erase
			}
		}

		void push_back(const T& x)
		{
			//Node* tail = _head->_prev;
			//Node* newnode = new Node(x); //括号这里讲过
			//
			//tail->_next = newnode;
			//newnode->_prev = tail;
			//newnode->_next = _head;
			//_head->_prev = newnode;
			insert(end(), x); //插在哨兵前,哨兵前就是尾

		}

		void pop_back()
		{
			//erase(iterator(_head->_prev));
			erase(--end()); //哨兵前就是最后一个数据
		}

		void push_front(const T& x)
		{
			insert(begin(), x);
		}

		void pop_front()
		{
			erase(begin());
		}

		void insert(iterator pos, const T& x) //pos位置前插入
		{
			Node* newnode = new Node(x); //这里一定要初始值x
			Node* cur = pos._node;
			Node* prev = cur->_prev;
			cur->_prev = newnode;
			newnode->_next = cur; //1 2 3 3 4
			newnode->_prev = prev;
			prev->_next = newnode;
		}

		void erase(iterator pos)
		{
			assert(pos != end()); //不能删哨兵  1 2 3 3 4
			Node* cur = pos._node;
			Node* next = cur->_next;
			Node* prev = cur->_prev;
			delete cur; //可以不置空,出作用域自动销毁
			
			next->_prev = prev;
			prev->_next = next;
		}

	private:
		Node* _head;
	};






	void test_list1() //基本的遍历
	{
		list<int> lt;
		lt.push_back(1);
		lt.push_back(2);
		lt.push_back(3);
		lt.push_back(4);

		list<int>::iterator lit = lt.begin();
		while (lit != lt.end())
		{
			cout << *lit << " ";
			++lit;
		}
		cout << endl;
	}



	struct Date
	{
		int _year = 0;
		int _month = 1;
		int _day = 1;
	};
	void test_list2() //cout类的数据
	{
		list<Date> dlt;
		dlt.push_back(Date());
		dlt.push_back(Date());

		list<Date>::iterator dit = dlt.begin();
		while (dit != dlt.end())
		{
			//cout << *dit << " "; //此时无法输出,一个类解引用怎么得不出想要的结果
			cout << dit->_year << "-" << dit->_month << "-" << dit->_day << endl; //这里->需要重载
			      //  dit->  返回 Date*
			      //所以实际上应写为 dit->->_year 为了可读性,编译器将其优化了,这样写反而会出错


			cout <<(*dit)._year << "-" << (*dit)._month << "-" << (*dit)._day << endl; //这样写也可以
			      //但是推荐->写法

			dit++;
		}
		cout << endl;
	}


	void print_list(const list<int>& lt)
	{
		list<int>::const_iterator lit = lt.begin();
		while (lit != lt.end())
		{
			//*lit = 1; //在修饰了lt为const,这一步却不报错且可以执行,不符合我们的要求
			            //此时加入const_iterator begin,end,重载*和&的修改
			cout << *lit << " ";
			++lit;
		}
		cout << endl;
	}

	void test_list3() //测试部分操作接口
	{
		list<int> lt;
		lt.push_back(1);
		lt.push_back(2);
		lt.push_back(3);
		lt.push_back(4);

		print_list(lt);

		lt.pop_back();
		lt.pop_front();
		print_list(lt);

		lt.clear();
		print_list(lt);
	}

	void test_list4() //测试默认函数
	{
		list<int> lt1;
		lt1.push_back(1);
		lt1.push_back(2);
		lt1.push_back(3);
		lt1.push_back(4);

		list<int> lt2(lt1); //之所以在重载++地方报错是因为,clear时调用++,浅拷贝释放后无法++

		list<int> lt3;
		lt3 = lt2;

		print_list(lt2);
		print_list(lt3);
	}

	//Node* cur  和  iterator it   的区别
	//他们都指向同一个节点,在物理内存中他们都是存的这个节点的地址
	//但是他们类型不一样,他们的意义就不一样了
	//*cur表示的是指针的解引用       ,取到的值是节点
	//*it 表示去调用这个迭代器重载的*,返回的是节点中的值
}
```
