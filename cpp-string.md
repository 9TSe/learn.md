---
title: string
date: 2023-10-21 21:03:03
tags:
- 数据结构
- 模拟实现
categories: 
- C++
cover: /pic/2.png
---


# 一、关于string常用的接口
---
## 1.string类对象的创建
---
```cpp
	//关于string类对象的创建
	string s1;
	string s2("hello");
	string s3(s2);  //这三个是常用且重点的,以下是了解一下

	string s4("hello", 2);         //输出前两个字符
	cout << s4 << endl;

	string s5(s2, 2, 1);           //输出s2内,第2个开始输出,输出1个字符
	cout << s5 << endl;

	string s6(s2, 2);              //输出s2内,第2个开始输出,输出至结束
	cout << s6 << endl;

	string s7(s2, 2,string::npos); //输出s2内,第2个开始输出,输出至结束
	cout << s7 << endl;

	string s8(10, 'a');            //10个字符a
	cout << s8 << endl;
	
```

---
## 2.reserve和resize
---
```cpp
	 //reverse
	s1.reserve(100);  //改变capacity
	cout << s1.capacity() << endl; 
		 //capacity有对齐,所以不一定为自己所指定改变的capacity
		 //capacity比实际输出多1,为了放 '\0'

	//resize
	s1.resize(3);         //改变size大小
	cout << s1 << endl;   //当size小于内容时,  输出  hel
	s1.resize(20);        //size变大
	cout << s1 << endl;
	s1.resize(20, '0');   //此时不需要初始化,因为没有扩大
	cout << s1 << endl;
	s2.resize(40, '0');   //size大于内容时,扩大了,扩大的空间初始化'0';
	cout << s2 << endl;
```

---
## 3.insert和erase
---

```cpp
	//insert
	s1.insert(s1.begin(), '0'); //在begin位置添加字符'0'
	s1.insert(2, "3");         //在下标为2的位置添加字符"3"
	cout << s1 << endl;


	//erase
	s1.erase(2, 3); //下标位置2的数据开始删除三个数据
	cout << s1 << endl;
	s2.erase(4); //下标位置4的数据开始删除所有数据
	cout << s2 << endl;
	s2.erase(1, 100); //如果超过了不会报错,会删除所有数据
	cout << s2 << endl;
```

---

## 4.c_str
---
```cpp
	//c_str  ,获取字符数组首地址,返回const char*指针
	//通过 c_str() 实现遍历
	string s1("hello");
	const char* str = s1.c_str();
	while (*str)
	{
		cout << *str << " ";
		++str;
	}
	cout << endl;


	//输出数组数据内容的方式
	cout << s1 << endl;          //调用string的重载 operator<<
	cout << s1.c_str() << endl;  //直接输出const char*

	//那么以上两者有什么区别呢 ??
	s1 += '\0';
	s1 += " world"; //s1 == hello(/0)world

	cout << s1 << endl;        //这种方式将整个数组内所有内容全部输出
	cout << s1.c_str() << endl;//这种方式遍历,遇见'\0'就停止
```
---
## 5.find(非算法中的find)
---

```cpp
	//由find可以分解URL : 协议  域名  资源名称
	//也可以将一下部分整理为函数接口
	string url("https://www.icourse163.org/home.htm?userId=1531618954#/home/course");
	size_t i1 = url.find(':');
	if (i1 != string::npos)
	{
		cout << url.substr(0, i1) << endl;
	}
    
	size_t i2 = url.find('/',i1 + 3); //从i1+3坐标开始寻找'/',之所以+3,跳过"://"
	if (i2 != string::npos)
	{
		cout << url.substr(i1 + 3, i2 - (i1 + 3)) << endl;
	}

	cout << url.substr(i2 + 1) << endl;
```
---
## 6.geline
---
```cpp
	//求最后一个英文单词长度
	string s;
	//cin >> s; //当想在一串字符串输入空格,自动结束
	getline(cin, s); //通过getline可以输入空格

	size_t pos = s.rfind(' ');
	if (pos != string::npos)
	{
		cout << s.size() - (pos + 1) << endl;
	}
```


---
# 二、string的模拟实现
---
## 1.模拟实现
```cpp
namespace 9TSe
{
	class string
	{
		
		friend istream& operator>>(istream& in, string& s);
	public:
		string(const char* str = "")
			:_str(new char[strlen(str) + 1])
		{
			strcpy(_str, str);
			_size = strlen(str);
			_capacity = _size;		
		}

		string(const string& s)
			:_str(new char[strlen(s._str) + 1])
		{
			strcpy(_str, s._str);
			_capacity = s._capacity;
			_size = s._size;
		}

		~string()
		{
			delete[] _str;
			_str = nullptr;
			_size = _capacity = 0;
		}


		typedef char* iterator; //迭代器的仿实现

		iterator begin()
		{
			return _str;
		}

		iterator end()
		{
			return _str + _size;
		}


		size_t size()const
		{
			return _size;
		}

		size_t capacity()const
		{
			return _capacity;
		}

		char& operator[](size_t i)
		{
			assert(i < _size);
			return _str[i];
		}

		const char& operator[](size_t i)const //两种取下标值的方法,以方便使用
		{
			assert(i < _size);
			return _str[i];
		}
		
		const char* c_str()
		{
			return _str;
		}



		//增:
		void reserve(size_t i)
		{
			size_t newcapacity = i;
			char* tmp = new char[newcapacity + 1]; //+1 为\0开辟空间
			strcpy(tmp, _str);
			delete[] _str;
			_str = tmp;
			_capacity = newcapacity;
		}

		void push_back(char ch) //增加字符
		{
			////增容
			//if (_size == _capacity)
			//{
			//	size_t newcapacity = _capacity == 0 ? 4 : _capacity * 2;
			//	reserve(newcapacity);
			//}
			//_str[_size] = ch;
			//_size++;
			//_str[_size] = '\0';

			insert(_size,ch); //可以调用insert减少步骤
		}

		void append(const char* str) //增加字符串
		{
			//size_t len = strlen(str);
			//if (len + _size > _capacity) //增容
			//{
			//	size_t newcapacity = len + _size;
			//	reserve(newcapacity);
			//}
			//strcpy(_str + _size, str);
			//_size += len;

			insert(_size,str);
		}

		string& operator+=(char ch)
		{
			push_back(ch);
			return *this;
		}

		string& operator+=(const char* str)
		{
			append(str);
			return *this;
		}

		string& insert(size_t pos, char ch)
		{
			assert(pos <= _size);
			if (_size == _capacity)
			{
				size_t newcapacity = _capacity == 0 ? 4 : _capacity * 2;
				reserve(newcapacity);
			}

			int end = _size;//end 类型不能给size_t 否则在pos为0时,end在0--后将变为npos的值,将无限循环
			while (end >= (int)pos) //类型不同形式比大小,隐式类型转换int -> size_t,强转pos为int可进行比较,可以阻止隐式类型转换
			{                       //int -1 >= size_t 0 ??  ==>> size_t -1 >= size_t 0;
				_str[end + 1] = _str[end];
				end--;
			}
			_str[pos] = ch;
			++_size;

			return *this;
		}

		string& insert(size_t pos, const char* str)
		{
			assert(pos <= _size);
			size_t len = strlen(str);
			if (len + _size > _capacity)
			{
				reserve(len + _size);
			}

			int end = _size; 
			while (end >= (int)pos)
			{
				_str[end + len] = _str[end];
				end--;
			}
			strncpy(_str+pos, str, len);
			_size += len;

			return *this;
		}
		

		//删
		void resize(size_t n,char ch)    //1 2 3 4 5
		{	
			if (n > _size)
			{
				if (n > _capacity)
				{
					reserve(n);
				}

				for (size_t i = _size; i < n; ++i)
				{
					_str[i] = ch;
				}
			}
			_str[n] = '\0';
			_size = n;
		}

		void erase(size_t pos, size_t len = npos)  //1 2 3 4 5
		{
			assert(pos < _size);
			if (len + pos >= _size)
			{
				_str[pos] = '\0';
				_size = pos;
			}
			else
			{
				size_t end = pos;
				while (end <= pos + len)
				{
					_str[end] = _str[end + len];
					end++;
				}
				_size -= len;
			}
		}

		//查
		size_t find(char ch, size_t pos = 0) //wdnmd
		{
			for (size_t i = pos; i < _size; ++i)
			{
				if (_str[i] == ch)
				{
					return i;
				}
			}
			return npos;
		}

		size_t find(const char* str, size_t pos = npos)
		{
			char* ret = strstr(_str + pos, str);
			if (ret)
			{
				return ret - _str;
			}
			else
			{
				return npos;
			}
		}


		//重载
		string& operator=(const string& s)
		{
			if (this != &s) //防止同对象自己赋值给自己
			{
				char* tmp = new char[strlen(s._str) + 1];
				strcpy(tmp, s._str);
				delete[] _str;
				_str = tmp;
				return *this;
			}
		}

		bool operator<(const string& s1)
		{
			size_t ret = strcmp(_str, s1._str);
			return ret < 0;
		}
		bool operator>(const string& s1)
		{
			return !(*this < s1);
		}
		bool operator==(const string& s1)
		{
			size_t ret = strcmp(_str, s1._str);
			return ret == 0;
		}
		bool operator>=(const string& s1)
		{
			return *this == s1 || *this > s1;
		}
		bool operator<=(const string& s1)
		{
			return !(*this > s1);
		}
		bool operator!=(const string& s1)
		{
			return !(*this == s1);
		}

		istream& getline(istream& in, string& s) //和operator>>二者本质没有区别,只是判断条件少了一个' '
		{
			while (1)
			{
				char ch;
				//in >> ch; //这样写会导致当输入' ' 或 '\n' 时被解读为开始读取下一个字符,直接被跳过
						  //导致无法读取' ' 和'\n'
				ch = in.get(); //使用这种方法可以读取任意字符,包括回车和空格
				if (ch == '\n')
				{
					break;
				}
				else
				{
					s += ch;
				}
			}
			return in;
		}

	private:
		char* _str;
		size_t _size;
		size_t _capacity;   //\0不是有效字符,不算在capacity和size内部
		                   //但在实际创建空间时需要多创建一个空间,存放\0
		static size_t npos;
	};

	size_t string::npos = -1;

	
	istream& operator>>(istream& in, string& s)
	{
		while (1)
		{
			char ch;
			//in >> ch; //这样写会导致当输入' ' 或 '\n' 时被解读为开始读取下一个字符,直接被跳过
					  //导致无法读取' ' 和'\n'
			ch = in.get(); //使用这种方法可以读取任意字符,包括回车和空格
			if (ch==' '||ch == '\n')
			{
				break;
			}
			else
			{
				s += ch;
			}
		}
		return in;
	}
	ostream& operator<<(ostream& out, const string& s) //由于运算符参数数量问题,和使用方便,重载写在外面
	{
		for (size_t i = 0; i < s.size(); ++i)
		{
			cout << s[i] << " ";
		}
		cout << endl;
		return out;
	}
```
---
## 2.string中构造,拷贝构造,赋值重载的简洁写法
---

```cpp
//实现一个有基本功能的string类:构造,析构,拷贝,operator=  ,,  深拷贝的现代写法 -- 简洁
namespace 9TSe
{
	class string
	{
	public:
		string(const char* str = "\0") //"\0" 或者""都可以
			:_str(new char[strlen(str)+1])
		{
			strcpy(_str,str);
		}

		////拷贝构造的传统写法
		//string(const string& s)
		//	:_str(new char[strlen(s._str) + 1])
		//{
		//	strcpy(_str, s._str);
		//}

		//拷贝构造的现代写法
		string(const string& s)
			:_str(nullptr)
		{
			string tmp(s._str);   //调用构造函数
			swap(_str, tmp._str); //如果不初始化_str为nullptr,tmp在析构随即地址时会崩溃
		}



		////operator=的传统写法
		//string& operator=(const string& s)
		//{
		//	if (this != &s)
		//	{
		//		char* tmp = new char[strlen(s._str) + 1];
		//		strcpy(tmp, s._str);
		//		delete[] _str;
		//		_str = tmp;
		//	}
		//	return *this;
		//}

		////operator=的现代写法
		//string& operator=(const string& s)
		//{
		//	if (this != &s)
		//	{
		//		string tmp(s);
		//		swap(_str, tmp._str);  //交换后,tmp还会再次调用析构,将_str交换过来的随机值释放空间
		//	}                          //可谓狸猫换太子
		//	return *this;
		//}

		//operator=的更简单的写法,传值操作
		string& operator=(string s) //相当于直接调用拷贝构造 // new: 口 
		{
			swap(_str, s._str);
		
	private:
		char* _str;
	};
}




namespace 9TSe //现代化方法,有时成员较多,库中的swap无法一次交换多个数据,以下为解决方式
{
	class string
	{
	public:
		string(const char* str = "")
		{
			_size = strlen(str);
			_capacity = _size;
			_str = new char[_capacity + 1];
			strcpy(_str, str);
		}

		string(const string& s)
			:_str(nullptr)
			,_size(0)
			,_capacity(0)
		{
			string tmp(s._str);
			//this->swap(tmp);
			swap(tmp);
		}

		void swap(string& s)
		{
			::swap(_str, s._str);  //由于参数相同,swap会优先调用自己,导致循环
			::swap(_size,s._size); //前面加入 :: 表示调用全局的swap
			::swap(_capacity, s._capacity);
		}

		string& operator=(string s) //这里不初始化_str是因为它交换于栈区,函数结束自动释放
		{
			_str = nullptr;
			//this->swap(s);
			swap(s);
			return *this;
		}

	private:
		char* _str;
		size_t _size;
		size_t _capacity;
	};
	
```



