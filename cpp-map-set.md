---
title: map,set
date: 2023-10-21 21:06:56
tags:
- 数据结构
- 模拟实现
- 树
categories: 
- C++
cover: /pic/2.png
---


---

# 一、set
---
## 1.set简介和基本用法
---

> set底层的实现就是红黑树

相较于算法中的  find O(N)
set用AVL树实现的find可以达到 O(logN)
用红黑树实现的find可以达到O(2logN)

将数据放入set中后会自动去重排序

---

## 2.multiset

> 相较于set少了去重的部分,可以让数据重复

find,erase等成员删除的都是中序遍历中第一个成员

---
# 二、map

---
## 1.pair(键值对)

```cpp
template<class T1, class T2>
struct pair
{
	typedef T1 first_type;
	typedef T2 second_type;

	T1 first;
	T2 second;

	pair()
		:first(T1()) //传入类型默认值构造
		,second(T2())
	{}

	pair(const T1& a,const T2& b)
		:first(a)
		,second(b)
	{}
};
```
## 2.插入

```cpp
	map<int, int> m;
	m.insert(pair<int, int>(1, 1)); //相当于传入pair临时构造的匿名对象
									//pair构造函数,构造一个匿名对象
	m.insert(make_pair(2, 2));      //函数 模板 构造一个pair对象
									//实际上更推荐用make_pair,不用声明模板参数,自动推断
```

map的插入返回值为

```cpp
pair<iterator, bool> Insert(const T& data)
//当插入的数在map中没有时,则插入成功,返回插入val的迭代器(引用),和true
//当插入的数在map中已经有了,则插入失败,返回已经和val相同值的迭代器(引用),和false
```
作为其operator[]的底层

---

## 3.operator[]

---

map用来计数
```cpp
	map<string, int> countmap;
	string strs[] = { "iphone","iphone","huawei","honor","huawei","summim" };
	for (auto& e : strs)
	{
		map<string, int>::iterator ret = countmap.find(e);
		if (ret != countmap.end())
			//(*ret).second++;
			ret->second++;
		else
			countmap.insert(make_pair(e, 1));
	}

	for (auto& e : countmap)
	{
		cout << e.first << ":" << e.second << endl;
	}
```

也可以用operator[]更加便捷

```cpp
	map<string, int> countmap;
	string strs[] = { "iphone","iphone","huawei","honor","huawei","summim" };
	for (auto& e : strs)
	{
		//如果不在map中,则operator[]会插入pair<str,0> (0 == int() 即默认类型值)  返回映射对象(次数)的引用并且++
		//如果在map中	,则operator[]返回key对应的映射对象(次数)的应用,对其++
		countmap[e]++;
	}
```
operator的底层可以看作为

```cpp
mapped_type& operator[](const key_type& k)
		{
			return (*((this->insert (make_pair(k, mapped_type()) )).first)).second;
			//this->insert (make_pair(k, mapped_type()) 返回值为 pair<map<string,int>::iterator,bool>
			//.first                                    取 map<string,int>::iterator
			//*											取 pair<key_type,mapped_type>
			//.second									取 mapped_type 并返回其引用
			//返回引用的作用是,方便对其进行以后的操作
		}
```



> 所以operator有三种用途
> 1.插入
> 2.查找k对应的映射对象
> 3.修改k对应的映射对象的值

```cpp
	map<string, int> countmap;
	countmap["vivo"];     //插入
	countmap["vivo"] = 1; //修改
	cout << countmap["vivo"] << endl; //查找
	countmap["oppo"] = 2; //插入+修改

	map<string, string> dict;
	dict.insert(make_pair("string", "字符串"));
	dict["string"] = "字字符串";
	dict["sort"] = "排序";
```

---
## 4.multimap

> multimap也是去掉了查重的功能,但没有 operator[]
> 当有多个相同的键值时,不知道返回哪一个对应的val


---

# 三、模拟实现set,map
---

底层都是由红黑树实现的,为了方便了解逻辑,删除了红黑树部分接口

```cpp
#pragma once
#include<iostream>
using namespace std;

template <class T, class Ref, class Ptr>
struct __BRTreeIterator
{
	typedef BRTreeNode <T> Node;
	Node* _node;
	typedef __BRTreeIterator<T, Ref, Ptr> Self;
	typedef __BRTreeIterator<T, T&, T*> iterator;
	
	__BRTreeIterator(Node* node)
		:_node(node)
	{}

	//普通迭代器时，是拷贝构造
	//const迭代器，指出迭代器构造const迭代器
	__BRTreeIterator(const iterator& s)
		:_node(s._node)
	{}
	
	Ref operator*()
	{
		return _node->_data;
	}

	Ptr operator->()
	{
		return &_node->_data;
	}

	Self operator++()
	{
		if (_node->_right)
		{
			Node* min = _node->_right;
			while (min->_left)
			{
				min = min->_left;
			}
			_node = min;
		}
		else
		{
			Node* cur = _node;
			Node* parent = cur->_parent;
			while (parent && cur == parent->_right)
			{
				cur = parent;
				parent = parent->_parent;
			}
			_node = parent;
		}
		return *this;
	}

	Self operator--()
	{
		if (_node->_left)
		{
			Node* max = _node->_left;
			while (max->_right)
			{
				max = max->_right;
			}
			_node = max;
		}
		else
		{
			Node* cur = _node;
			Node* parent = cur->_parent;
			while (parent && cur == parent->_left)
			{
				cur = parent;
				parent = parent->_parent;
			}
			_node = parent;
		}
		return *this;
	}

	bool operator!=(const Self& s) const
	{
		return _node != s._node;
	}

	bool operator==(const Self& s) const
	{
		return _node == s._node;
	}
};


template<class K, class T, class KeyOfT>
class BRTree
{
public:
	typedef BRTreeNode<T> Node;
	typedef __BRTreeIterator<T, T&, T*> iterator;
	typedef __BRTreeIterator<T, const T&, const T*> const_iterator;

	iterator begin()
	{
		Node* left = _root;
		while (left && left->_left)
		{
			left = left->_left;
		}
		return iterator(left);
	}

	iterator end()
	{
		return iterator(nullptr);
	}

	const_iterator begin() const
	{
		Node* left = _root;
		while (left && left->_left)
		{
			left = left->_left;
		}
		return const_iterator(left);
	}

	const_iterator end() const
	{
		return const_iterator(nullptr);
	}

	pair<iterator, bool> Insert(const T& data)
	{
		if (_root == nullptr)
		{
			_root = new Node(data);
			_root->_col = Black;
			return make_pair(iterator(_root), true);
		}
		KeyOfT kot;
		//父子节点确定插入的位置
		Node* parent = nullptr;
		Node* cur = _root;
		while (cur)
		{
			if (kot(cur->_data) > kot(data))
			{
				parent = cur;
				cur = cur->_left;
			}
			else if (kot(cur->_data) < kot(data))
			{
				parent = cur;
				cur = cur->_right;
			}
			else
				return make_pair(iterator(cur), false);
		}

		//走到这cur就是要插入的位置
		//cur要连接parent，parent也要连接cur---判断靠kv的大小
		cur = new Node(data);
		Node* newnode = cur;
		if (kot(parent->_data) > kot(cur->_data))
		{
			parent->_left = cur;
			cur->_parent = parent;
		}
		else
		{
			parent->_right = cur;
			cur->_parent = parent;
		}

		while (parent && parent->_col == Red)
		{
			Node* grandparent = parent->_parent;
			//parent分在grandparent左右
			if (grandparent->_left == parent)
			{
				//关键是看uncle节点不存在/红色/黑色的情况
				Node* uncle = grandparent->_right;
				if (uncle && uncle->_col == Red)
				{
					grandparent->_col = Red;
					parent->_col = uncle->_col = Black;
					.....
					
		return make_pair(iterator(newnode), true);
	}

private:
	Node* _root = nullptr;
};
```

---
## 1.set

```cpp
#include "RBTree.hpp"

namespace 9TSe
{
	template<class K>
	class Set
	{
	public:
		struct SetKeyOfT
		{
			const K& operator()(const K& k)
			{
				return k;
			}
		};
		typedef typename BRTree<K, K, SetKeyOfT>::const_iterator iterator;
		typedef typename BRTree<K, K, SetKeyOfT>::const_iterator const_iterator;

		pair<iterator, bool> Insert(const K& k)
		{
			return _t.Insert(k);
		}

		iterator begin()
		{
			return _t.begin();
		}

		iterator end()
		{
			return _t.end();
		}

		const_iterator begin() const
		{
			return _t.begin();
		}

		const_iterator end() const
		{
			return _t.end();
		}

	private:
		BRTree<K, K, SetKeyOfT> _t;
	};
```

> set内数据本身是不能修改的
> 否则会破坏树的结构
> 所以迭代器都是由红黑树的const_iterator构成的

---
## 2.map

```cpp
#include "RBTree.hpp"

namespace 9TSe
{
	template<class K, class V>
	class Map
	{
	public:
		struct MapKeyOfT
		{
			const K& operator()(const pair<K, V>& kv)
			{
				return kv.first;
			}
		};

		V& operator[](const K& k)
		{
			pair<iterator, bool> ret = Insert(make_pair(k, V()));
			return ret.first->second;
		}

		typedef typename BRTree<K, pair<const K, V>, MapKeyOfT>::iterator iterator;
		typedef typename BRTree<K, pair<const K, V>, MapKeyOfT>::const_iterator const_iterator;

		pair<iterator, bool> Insert(const pair<K, V>& kv)
		{
			return _t.Insert(kv);
		}

		const_iterator begin() const
		{
			return _t.begin();
		}

		const_iterator end() const
		{
			return _t.end();
		}

		iterator begin()
		{
			return _t.begin();
		}

		iterator end()
		{
			return _t.end();
		}

	private:
		BRTree<K, pair<const K, V>, MapKeyOfT> _t;
	};
```
