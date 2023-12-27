---
title: 哈希
date: 2023-10-21 21:07:04
tags:
- 哈希
- 数据结构
- 模拟实现
categories: 
- C++
cover: /pic/2.png
---


---
# 一、哈希的基本介绍

---
性能上
>查找数据，哈希时间复杂度为O(1)
>大量随机数据插入，map和set性能快
>大量随机数据删除，map和set性能快

哈希映射方法
>1.直接定址法:每个值都有自己唯一的位置
>优点:无哈希冲突,简单且均匀
>缺点:如果空间分散会有很大的空间浪费
>适应于范围小且连续的情况

>2,除留余数法
>优点:空间合适且不浪费过多的空间
>缺点:可能存在有相同的映射地址(哈希冲突)

哈希大部分都围绕着解决哈希冲突
>1.闭散列:冲突后占用下一个空位置
>缺点：查找效率变低，互相冲突导致插入逻辑复杂
>线性探测:线性找下一个空位置,++
>二次探测:平方步调找下一个位置,^2

>2.开散列(哈希桶)(拉链法):冲突的位置下加上链表(哈希桶),由此STL中并没有双向迭代器
>缺点:链表太长有效率问题
>优点:冲突不会相互影响

---

# 二、闭散列的哈希表实现

---
实际中开散列应用情况优于闭散列,点一下
```cpp
#include<iostream>
#include<vector>
using namespace std;

template<class K>
struct KeyToCmp //得到可以进行比较的数据
{
	size_t operator()(const K& k)
	{
		return (size_t)k;
	}
};

template<>
struct KeyToCmp<string> //特化一个string的仿函数,使得字符也能有其映射的地址
{
	size_t operator()(const string& k)
	{
		size_t hash = 0;
		for (auto& e : k)
		{
			hash += e;
		}
		return hash;
	}
};


enum State
{
	EMPTY,
	EXIST,
	DELETE
};
	
template<class K,class V>
struct HashData
{
	pair<K,V> _kv;
	State _state = EMPTY; //通过加入状态成员来方便进行增删查改
};
	
template<class K,class V,class Hash = KeyToCmp<int>>
class HashTable
{
public:
	typedef HashData<K,V> Data;

	HashTable() //构造
	{
		_tables.resize(10);
	}


	//负载因子 = 表中数据/表的大小  衡量哈希表满的程度
	//负载因子越大,插入数据越容易发生哈希冲突,冲突越多,效率越低
	//所以哈希表并不是满了才增容,一般负载因子在0.7左右就开始增容
	//控制的太小,会使得空间浪费
	bool Insert(const pair<K,V>& kv)
	{
		if (Find(kv.first) != nullptr) //已经有了
			return false;
		
		if (_num * 10 / _tables.size() >= 7) //负载因子大了,开始增容
		{
			HashTable<K, V, Hash> tmp;
			tmp._tables.resize(2 * _tables.size()); //新创建一个哈希表,将其数组空间扩大至旧哈希表的两倍
			
			for (auto& e : _tables)
			{
				if (e._state == EXIST) //如果有数据
					tmp.Insert(e._kv); //插入,扩容已经在上面完成了,递归时判断增容条件进不去,不会造成死循环
			}
			_tables.swap(tmp._tables);
		}

		Hash get;
		size_t index = get(kv.first) % _tables.size();

		while (_tables[index]._state == EXIST)
		{
			index++;
			index %= _tables.size();
		}

		_tables[index]._kv = kv;
		_tables[index]._state = EXIST;
		_num++;
		
		return true;
	}


	Data* Find(const K& k)
	{
		
		Hash get;
		size_t index = get(k) % _tables.size();

		while (_tables[index]._state != EMPTY)
		{
			if (_tables[index]._state == EXIST && _tables[index]._kv.first == k)
				return &_tables[index];
			index++;
			index %= _tables.size();
		}
		return nullptr; //哈希冲突是连续的,如果第一个后面为空,说明后面也没有冲突的情况,直接返回
	}


	bool Erase(const pair<K, V>& kv)
	{
		Data* ret = Find(kv.first);
		if (ret)
		{
			ret->_state = DELETE;
			--_num;
			return true;
		}
		else
		{
			return false;
		}
	}
	
private:
	vector<Data> _tables;
	size_t _num = 0; //存储当前存储有效数据的个数
};

```

---

# 三、开散列模拟实现unordered_xxx

---

unordered_xxx底层都是由开散列实现的

```cpp
#pragma once
#include<iostream>
#include<vector>
using namespace std;

namespace BsyOpen
{
	template<class K>
	struct GetWay
	{
		const K& operator()(const K& k)
		{
			return k;
		}
	};

	template<> //特化处理字符串,使得其返回值能够被模
	struct GetWay<string>
	{
		size_t operator()(const string& k)
		{
			size_t way = 0;
			for (size_t i = 0; i < k.size(); ++i)
			{
				way *= 131; //BKDR Hash,使得哈希冲突降低
				way += k[i];
			}
			return way;
		}
	};


	template<class T> //一个哈希表节点
	struct HashNode 
	{
		T _data;
		HashNode<T>* _next;
		//我们自己实现的哈希输出,并非输入的顺序,如果要实现按输入顺序输出就要多指针
		//HashNode<T>* _linknext; //以按顺序输出
		//HashNode<T>* _linkprev; //以删除

		HashNode(const T& data)
			:_data(data)
			,_next(nullptr)
		{}
	};


	// 前置声明一下, 否则在迭代器类中, 无法识别出HashTable是什么
	template<class K, class T, class TheKey, class GetWayy> //哈希表
	class HashTable; 

	template<class K,class T,class TheKey,class GetWayy> //哈希表迭代器
	struct __HashIterator
	{
		typedef __HashIterator< K, T, TheKey, GetWayy> Self;
		typedef HashTable<K, T, TheKey, GetWayy> HT;
		typedef HashNode<T> Node;
		Node* _node;
		HT* _pht; //以便在后续operator++中起到作用

		__HashIterator(Node* node,HT* pht)
			:_node(node)
			,_pht(pht)
		{}

		T& operator*()
		{
			return _node->_data;
		}

		T* operator->()
		{
			return &_node->_data;
		}

		Self operator++()
		{
			if (_node->_next)
			{
				_node = _node->_next; //如果桶没走完接着往下走
			}
			else
			{
				TheKey thekey;
				GetWayy getway;
				//计算数值应该位于哪个头节点
				size_t i = getway(thekey(_node->_data)) % _pht->_tables.size();
			  //size_t i = GetWayy()(TheKey()(_node->_data)) % _pht->_tables.size();
				i++; //现在i就是下一个需要验证是否为空的数组下标
				for (; i < _pht->_tables.size(); ++i)
				{
					Node* cur = _pht->_tables[i];
					if (cur)
					{
						_node = cur;
						return *this;
					}
				}
				//此时说明没有下一个节点了
				_node = nullptr;
			}
			return *this;
		}

		bool operator!=(const Self& s)
		{
			return _node != s._node;
		}
	};


	template<class K,class T,class TheKey,class GetWayy> //哈希表
	class HashTable
	{
	public:
		friend struct __HashIterator< K, T, TheKey, GetWayy>;
		typedef HashNode<T> Node;
		typedef __HashIterator< K, T, TheKey,GetWayy> iterator;

		iterator begin()
		{
			for (size_t i = 0; i < _tables.size(); ++i)
			{
				if (_tables[i])
				{
					return iterator(_tables[i],this);
				}
			}
			return end();
		}

		iterator end()
		{
			return iterator(nullptr,this);
		}

		~HashTable() //销毁完桶之后,,析构自动销毁this,即vector会随着析构函数销毁
		{
			Clear();
		}

		void Clear()
		{
			for (size_t i = 0; i < _tables.size(); ++i)
			{
				Node* cur = _tables[i];
				while (cur)
				{
					Node* next = cur->_next;
					delete cur;
					cur = next;
				}
				_tables[i] = nullptr; //使得无法访问每个头节点
			}
		}

		size_t GetNextPrime(size_t num) //这是一种算法,使得可以减小哈希冲突
		{
			const int primecount = 28;
			const size_t primelist[primecount] = //二倍关系质数表
			{
				53,97,193,389,769,
				1543,3079,6151,12289,24593,
				49157,98317,1996613,393241,786433,
				1572869,3145739,6291469,12582917,25165843,
				50331653,100663319,201326611,402653189,805306457,
				1610612741,3221225473,4294967291
			};

			for (size_t i = 0; i < primecount; ++i)
			{
				if (primelist[i] > num)
				{
					return primelist[i];
				}
			}
			return primelist[primecount - 1]; //如果还大,就一直返回最后一个
		}

		pair<iterator,bool> Insert(const T& data)
		{
			TheKey thekey;
			GetWayy getway;

			if (_num == _tables.size()) //此时负载因子为1,避免以后造成大量冲突,增容
			{
				vector<Node*> newtable;
				size_t newsize = GetNextPrime(_num);
				newtable.resize(newsize);
				for (size_t i = 0; i < _tables.size(); ++i)
				{
					Node* cur = _tables[i]; //取每个头
					while (cur)
					{
						//旧表取下重新计算新表中的位置
						Node* next = cur->_next;
						size_t index = getway(thekey(cur->_data)) % newtable.size(); //新表中的位置
						cur->_next = newtable[index]; //头插进去
						newtable[index] = cur;
						cur = next;
					}
					_tables[i] = nullptr; 
				}
				_tables.swap(newtable);
			}


			size_t index = getway(thekey(data)) % _tables.size();

			Node* cur = _tables[index];
			while (cur)
			{
				if (thekey(cur->_data) == thekey(data)) //重复了,不插入,结束
					return make_pair(iterator(cur,this),false);
				else
					cur = cur->_next;
			}

			//检验完后走到这里说明可以插入了
			//选择头插,因为有头节点的位置,不用找尾
			Node* newnode = new Node(data);
			newnode->_next = _tables[index];
			_tables[index] = newnode;

			++_num;
			return make_pair(iterator(newnode, this), true);
		}

		Node* Find(const K& k)
		{
			TheKey thekey;
			GetWayy getway;
			size_t index = getway(k) % _tables.size();
			Node* cur = _tables[index];
			while (cur)
			{
				if (thekey(cur->_data) == k)
					return cur;
				else
					cur = cur->_next;
			}
			return nullptr;
		}


		bool Erase(const K& k)
		{
			TheKey thekey;
			GetWayy getway;
			size_t index = getway(k) % _tables.size();
			Node* prev = nullptr;
			Node* cur = _tables[index]; //找到需要找到数据的头
			while (cur)
			{
				if (thekey(cur->_data) == k)//找到了
				{

					if (prev == nullptr) //说明要删除的对象就是第一个节点
						_tables[index] = cur->_next;
					else
						prev->_next = cur->_next;
					
					delete cur;
					return true;
				}
				else
				{
					prev = cur;
					cur = cur->_next;
				}
			}
			return false;
		}

	private:
		vector<Node*> _tables;
		size_t _num = 0;
	};
}
```

---
## 1.unordered_set

```cpp
#pragma once
#include"Open_Hash.hpp"

namespace Bsy
{
	template<class K, class GetWayy = BsyOpen::GetWay<K>>
	class unordered_set
	{
		struct TheKeyOfSet
		{
			const K& operator()(const K& k)
			{
				return k;
			}
		};


	public:
		typedef typename BsyOpen::HashTable<K, K, TheKeyOfSet, GetWayy>::iterator iterator;

		iterator begin()
		{
			return _ht.begin();
		}

		iterator end()
		{
			return _ht.end();
		}

		pair<iterator,bool> insert(const K& k)
		{
			return _ht.Insert(k);
		}

	private:
		BsyOpen::HashTable<K, K, TheKeyOfSet,GetWayy> _ht;
	};

	void test_unordered_set1()
	{
		unordered_set<int> s;
		s.insert(1);
		s.insert(5);
		s.insert(6);
		s.insert(2);

		unordered_set<int>::iterator it = s.begin();
		while (it != s.end())
		{
			cout << *it << " ";
			++it;
		}
		cout << endl;
	}

}
```

---

## 2.unordered_map

```cpp
#pragma once
#include"Open_Hash.hpp"

namespace Bsy
{
	template<class K,class V, class GetWayy = BsyOpen::GetWay<K>>
	class unordered_map
	{
		struct TheKeyOfMap
		{
			const K& operator()(const pair<K, V>& kv)
			{
				return kv.first;
			}
		};

	public:
		typedef typename BsyOpen::HashTable<K, pair<K,V>, TheKeyOfMap, GetWayy>::iterator iterator;

		iterator begin()
		{
			return _ht.begin();
		}

		iterator end()
		{
			return _ht.end();
		}

		pair<iterator,bool> insert(const pair<K, V>& kv)
		{
			return _ht.Insert(kv);
		}

		V& operator[](const K& k)
		{
			//pair<iterator, bool> ret = _ht.Insert(make_pair(k, V()));
			//return ret.first->second;
			return _ht.Insert(make_pair(k, V())).first->second;
		}

	private:
		BsyOpen::HashTable<K, pair<K, V>, TheKeyOfMap,GetWayy> _ht;
	};


	void test_unordered_map1()
	{
		unordered_map<string, string> dict;
		dict.insert(make_pair("sort", "paixu"));
		dict.insert(make_pair("left", "zuo"));
		dict.insert(make_pair("long", "chang"));
		dict["left"] = "you";
		dict["hash"] = "haxi";

		unordered_map<string, string>::iterator it = dict.begin();
		while (it != dict.end())
		{
			cout << it->first <<":"<<it->second<< endl;
			++it;
		}
		cout << endl;
	}
}
```

