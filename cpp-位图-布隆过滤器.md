---
title: 位图,布隆过滤器
date: 2023-10-21 21:07:22
tags:
- 哈希
- 数据结构
- 模拟实现
categories: 
- C++
cover: /pic/2.png
---

---
# 一、位图
---

> 位图就是利用好每一个比特位进行利用
> 适用于海量数据,数据无重复的场景,通常用来判断数据是否存在

位图的作用有
>排序,去重,集合的交并集

模拟实现

```cpp
#pragma once
#include<vector>
#include<iostream>
using namespace std;

namespace 9TSe
{
	class bitset
	{
	public:
		bitset(size_t N)
		{
			_bits.resize(N/32+1,0); //开空间和初始化
			_num = 0;
		}

		void set(size_t x) //增
		{
			size_t index = x / 32; //算出映射的位置在第几个整形
			size_t pos = x % 32; //算出x在整形的第几位

			_bits[index] |= (1 << pos); //找到位置置1
		}

		void reset(size_t x)//删
		{
			size_t index = x / 32;
			size_t pos = x % 32;
			_bits[index] &= ~(1 << pos); //注意:左移的并不是位置,而是低位向高位移动,同理右移
										 //找到位置置零
		}

		bool test(size_t x) //查,找到为1,没有为0
		{
			size_t index = x / 32;
			size_t pos = x % 32;
			return _bits[index] & (1 << pos); //查1
		}

	private:
		vector<int> _bits;
		size_t _num;
	};


	void test_bitset()
	{
		bitset bs(100); //存储100个,开100/32+1个空间
		bs.set(99);
		bs.set(98);
		bs.set(97);
		bs.set(5);

		for (size_t i = 0; i < 100; ++i)
		{
			printf("[%d]:%d\n", i, bs.test(i));
		}
		//如果要创建出最大的空间
		//bitset bss(-1); //-1转换为size_t

		//库内的使用
		bitset<100> bs;
		bs.set(); 	 //全1
		bs.set(3,0); //3位置0  并非从左往右,大小端不同,位置不同
		bs.set(3);   //3位置1
	}
}
```

---

# 二、布隆过滤器
---

> 假如有100亿个ip地址在文件中，如何判断一个ip地址是否在这个文件中？
1.哈希表浪费空间
2.位图一般只能处理整形，如果内容编号是字符串，就无法处理。
3.将哈希与位图结合，即布隆过滤器

所以布隆过滤器的底层就是bitset

```cpp
#include"bitset.hpp"

namespace 9TSe
{
	//BKDR
	struct Hashstr1
	{
		size_t operator()(const std::string& s)
		{
			size_t theway = 0;
			for (size_t i = 0; i < s.size(); ++i)
			{
				theway *= 131;
				theway += s[i];
			}
			return theway;
		}
	};

	//RSHash
	struct Hashstr2
	{
		size_t operator()(const std::string& s)
		{
			size_t theway = 0;
			size_t magic = 63689;
			for (size_t i = 0; i < s.size(); ++i)
			{
				theway *= magic;
				theway += s[i];
				magic *= 378551;
			}
			return theway;
		}
	};

	//SDBMHash
	struct Hashstr3
	{
		size_t operator()(const std::string& s)
		{
			size_t theway = 0;
			for (size_t i = 0; i < s.size(); ++i)
			{
				theway *= 65599;
				theway += s[i];
			}
			return theway;
		}
	};

	//哈希函数太多效率会变低,哈希函数太少误报率会变高
	template<class K = std::string,class Hash1 = Hashstr1,class Hash2 = Hashstr2,class Hash3 = Hashstr3>
	class bloomfilter
	{
	public:

		bloomfilter(size_t num)
			:_bs(5*num) //算法,算出开空间并不为3*num,布隆过滤器越长其误报概率越小
			, _N(5*num) //空间小误报率高,空间大可能有空间浪费,4.3即为最优
		{}

		void set(const K& k)
		{
			//Hash1 hash1; ....
			size_t index1 = Hash1()(k) % _N; //加括号构成匿名对象
			size_t index2 = Hash2()(k) % _N;
			size_t index3 = Hash3()(k) % _N;

			//cout << index1 << endl;
			//cout << index2 << endl;
			//cout << index3 << endl<<endl; //观测一下插入位置

			_bs.set(index1);
			_bs.set(index2);
			_bs.set(index3);
		}

		bool test(const K& k) //都找到了,说明已经有了
		{
			size_t index1 = Hash1()(k) % _N;
			if (_bs.test(index1) == false)
				return false;

			size_t index2 = Hash2()(k) % _N;
			if (_bs.test(index2) == false)
				return false;

			size_t index3 = Hash3()(k) % _N;
			if (_bs.test(index3) == false)
				return false;
			return true;
		}

		void reset(const K& k)
		{
			//映射位置不能直接置零
			//不支持删除,可能会存在误删,一般布隆过滤器不支持删除
		}

	private:
		bitset _bs; //位图
		size_t _N;
	};

	void test_bloomfilter()
	{
		bloomfilter<std::string> bf(100);
		bf.set("abcd");
		bf.set("aadd");
		bf.set("bcad");

		cout << bf.test("abcd") << endl; //能找到
		cout << bf.test("aadd") << endl;
		cout << bf.test("bcad") << endl;
		cout << bf.test("cdbc") << endl;//返回0找不到

	}
}
```

---

>其原理大概就是一个数据通过多个哈希函数(优质算法)映射到不同的位置
>两个不同的数据映射位置完全相同的概率非常低(有可能不同数据映射相同位置)
>


优点
>节省空间,高效,可以标记存储任何类型,
>增查效率有O(K),k==哈希函数个数

缺点
>有误判(可以创建白名单:将可能出现误判的数据存入)
>无法获取数据本身
>一般情况无法删除数据
>通过计数删除可能有计数环绕问题(8->0)

计数删除:比特位扩展一个计数器(还要考虑要占几个bit位),便可以支持删除,不过哈希冲突仍然存在
