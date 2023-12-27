---
title: 搜索二叉树
date: 2023-10-21 21:06:23
tags:
- 数据结构
- 模拟实现
- 树
categories: 
- C++
cover: /pic/2.png
---


---
# 一、key模型搜索二叉树的模拟实现

---

## 1.非递归实现

```cpp
#pragma once
#include<iostream>
using namespace std;

template<class K>
struct BSTNode //binary search tree
{
	BSTNode* _left;
	BSTNode* _right;
	K _key;

	BSTNode(const K& key)
		:_left(nullptr)
		,_right(nullptr)
		,_key(key)
	{}
};

template<class K>
class BSTree
{
	typedef BSTNode<K> Node;
public:

	bool Insert(const K& key) 
	{
		if (_root == NULL)
		{
			_root = new Node(key);
			return true;
		}

		Node* parent = nullptr;
		Node* cur = _root;
		while (cur)
		{
			if (cur->_key > key)
			{
				parent = cur;
				cur = cur->_left;
			}
			else if (cur->_key < key)
			{
				parent = cur;
				cur = cur->_right;
			}
			else
			{
				return false;
			}
		}

		cur = new Node(key);          //找到位置后进行链接
		if (cur->_key > parent->_key) //判定插在左或者右
			parent->_right = cur;
		else
			parent->_left = cur;
		return true;
	}

	bool Find(const K& key)
	{
		Node* cur = _root;
		while (cur)
		{
			if (cur->_key > key)
				cur = cur->_left;
			else if (cur->_key < key)
				cur = cur->_right;
			else
				return true;
		}
		return false;
	}

	bool Erase(const K& key)
	{
		Node* cur = _root;
		Node* parent = nullptr;
		//由于要用双指针,所以不能直接用Find来执行这一部分
		while (cur!=nullptr)
		{
			if (cur->_key > key)
			{
				parent = cur;
				cur = cur->_left;
			}
			else if (cur->_key < key)
			{
				parent = cur;
				cur = cur->_right;
			}
			else //找到了
			{
				//1.无左无右
				//2.都有
				if(cur->_left == nullptr) //无左,和无左无右
				{
					if (cur == _root) //如果要删除的就是根
					{
						_root = cur -> _right;
					}
					else
					{
						if (parent->_right == cur)
							parent->_right = cur->_right;
						else
							parent->_left = cur->_right;
					}
					delete cur;
					return true;
				}
				else if (cur->_right == nullptr) //无右
				{
					if (cur == _root)
					{
						_root = cur->_left;
					}
					else
					{
						if (parent->_right == cur)
							parent->_right = cur->_left;
						else
							parent->_left = cur->_left;
					}
					delete cur;
					return true;
				}
				else//找到左树最右叶,或者右树最左叶,替换cur
				{
					Node* rightmin = cur->_right;
					Node* rightminparent = cur;
					while (rightmin->_left != nullptr)
					{
						rightminparent = rightmin;
						rightmin = rightmin->_left;
					}
					cur->_key = rightmin->_key;

					//转换为删除rightmin(rightmin左为空,删除他的右)
					if(rightmin == rightminparent->_left) //如果进入了循环
					rightminparent->_left = rightmin->_right;
					else
					rightminparent->_right = rightmin->_right;
					delete rightmin;
					return true;
				}
			}
		}
		return false;
	}

	void _InOrder(Node* root) //中序打印数据
	{
		if (root == nullptr)
			return;

		_InOrder(root->_left);
		cout << root->_key << " ";
		_InOrder(root->_right);
	}

	void InOrder() //使中序打印函数使用更方便
	{
		_InOrder(_root);
		cout << endl;
	}
private:
	Node* _root = nullptr;
};
```

---

## 2.递归实现

---

```cpp
#pragma once
#include<iostream>
#include<string>
using namespace std;

template<class K>
struct BSTreeNode
{
	BSTreeNode(const K& key)
		:_key(key)
		, _left(nullptr)
		, _right(nullptr)
	{}
	BSTreeNode<K>* _left;
	BSTreeNode<K>* _right;
	K _key;
};

template<class K>
class BSTree
{
	typedef BSTreeNode<K> Node;
public:
	BSTree()
		:_root(nullptr)
	{}



	Node* Copy(Node* root)
	{
		if (root == nullptr)
			return nullptr;

		Node* newroot = new Node(root->_key);
		newroot->_left = Copy(root->_left);
		newroot->_right = Copy(root->_right);
		return newroot;
	}

	BSTree(const BSTree<K>& t)
	{
		_root = Copy(t._root);
	}
	


	BSTree<K>& operator=(BSTree<K> t)
	{
		swap(_root, t._root);
		return *this;
	}



	void Destory(Node* root)
	{
		if (root == nullptr)
			return;

		Destory(root->_left);
		Destory(root->_right);
		delete root;
	}

	~BSTree()
	{
		Destory(_root);
		_root = nullptr;
	}



	void _Print(Node* root)
	{
		if (root == nullptr)
			return;
		_Print(root->_left);
		cout << root->_key << " ";
		_Print(root->_right);
	}

	void Print()
	{
		_Print(_root);
		cout << endl;
	}



	bool _InsertR(Node* root, const K& k, Node*& parent)
	{
		Node* cur = root;
		if (cur != nullptr)
		{
			if (cur->_key > k)
			{
				parent = cur;
				_InsertR(cur->_left, k, parent);
			}
			else if (cur->_key < k)
			{
				parent = cur;
				_InsertR(cur->_right, k, parent);
			}
		}
		else
		{
			cur = new Node(k);
			if (parent->_key > k)
				parent->_left = cur;
			else
				parent->_right = cur;
			return true;
		}
		if (root->_key == k)
			return false;
	}

	bool Insert(const K& k)
	{
		if (_root == nullptr)
		{
			_root = new Node(k);
			return true;
		}
		Node* parent = nullptr;
		return _InsertR(_root, k, parent);
	}



	//进阶思路----引用的使用
	bool _InsertR(Node*& root, const K& k)
	{
		if (root == nullptr)
		{
			root = new Node(k); //这里要修改指针指向的内容,所以要传指针的指针类型的参数(*& || **)
			return true;
		}
		if (root->_key < k)
			return _InsertR(root->_right, k);
		else if (root->_key > k)
			return _InsertR(root->_left, k);
		else
			return false;
	}

	bool Insert(const K& k)
	{
		return _InsertR(_root, k);
	}



	bool _Find(Node* root, const K& k)
	{
		if (root == nullptr)
			return false;
		if (root->_key > k)
			_Find(root->_left, k);
		else if (root->_key < k)
			_Find(root->_left, k);
		else
			return true;
	}

	bool Find(const K& k)
	{
		return _Find(_root, k);
	}



	bool _Erase(Node*& root, const K& k)
	{
		if (root == nullptr)
			return false;

		if (root->_key > k)
			_Erase(root->_left, k);
		else if (root->_key < k)
			_Erase(root->_right, k);
		else
		{
			Node* del = root; //root的key对应k
			if (root->_left == nullptr)
				root = root->_right;
			else if (root->_right == nullptr)
				root = root->_left;
			else
			{
				Node* minRight = root->_right;
				while (minRight->_left)
				{
					minRight = minRight->_left;
				}
				swap(root->_key, minRight->_key);
				return _Erase(root->_right, k);
			}
			delete del;
			return true;
		}
	}

	bool Erase(const K& k)
	{
		return _Erase(_root, k);
	}

private:
	Node* _root = nullptr;
};
```

---

# 二、Key/Value 模型搜索二叉树的模拟实现

---

```cpp
#pragma once
#include<iostream>
#include<string>
using namespace std;

template<class K,class V>
struct BSTNode //binary search tree
{
	BSTNode* _left;
	BSTNode* _right;
	K _key;
	V _value;

	BSTNode(const K& key,const V& value)
		: _left(nullptr)
		, _right(nullptr)
		, _key(key)
		, _value(value)
	{}
};

template<class K,class V>
class BSTree
{
	typedef BSTNode<K,V> Node;
public:

	bool Insert(const K& key,const V& value) //插入数据
	{
		if (_root == NULL)
		{
			_root = new Node(key,value); //new 自动调用构造函数
			return true;
		}

		Node* parent = nullptr;
		Node* cur = _root;
		while (cur) //cur为空时结束
		{
			if (cur->_key > key)
			{
				parent = cur;
				cur = cur->_left;
			}
			else if (cur->_key < key)
			{
				parent = cur;
				cur = cur->_right;
			}
			else
			{
				return false;
			}
		}

		cur = new Node(key,value);          //找到位置后进行链接
		if (cur->_key > parent->_key) //判定插在左或者右
		{
			parent->_right = cur;
		}
		else
		{
			parent->_left = cur;
		}
		return true;
	}

	Node* Find(const K& key)
	{
		Node* cur = _root;

		while (cur)
		{
			if (cur->_key > key)
				cur = cur->_left;
			else if (cur->_key < key)
				cur = cur->_right;
			else
				return cur;
		}
		return nullptr;
	}

	bool Erase(const K& key)
	{
		Node* cur = _root;
		Node* parent = nullptr;
		//由于要用双指针,所以不能直接用Find来执行这一部分
		while (cur != nullptr)
		{
			if (cur->_key > key)
			{
				parent = cur;
				cur = cur->_left;
			}
			else if (cur->_key < key)
			{
				parent = cur;
				cur = cur->_right;
			}
			else //找到了
			{
				//1.无左无右
				//2.都有
				if (cur->_left == nullptr) //无左,和无左无右
				{
					if (cur == _root) //如果要删除的就是根
					{
						_root = cur->_right;
					}
					else
					{
						if (parent->_right == cur)
							parent->_right = cur->_right;
						else
							parent->_left = cur->_right;
					}
					delete cur;
					return true;
				}
				else if (cur->_right == nullptr) //无右
				{
					if (cur == _root)
					{
						_root = cur->_left;
					}
					else
					{
						if (parent->_right == cur)
							parent->_right = cur->_left;
						else
							parent->_left = cur->_left;
					}
					delete cur;
					return true;
				}
				else//找到左树最右叶,或者右树最左叶,替换cur
				{
					Node* rightmin = cur->_right;
					Node* rightminparent = cur;
					while (rightmin->_left != nullptr) 
					{
						rightminparent = rightmin;
						rightmin = rightmin->_left;
					}

					cur->_key = rightmin->_key;

					//转换为删除rightmin(rightmin左为空,删除他的右)
					if (rightmin == rightminparent->_left) //如果进入while循环
						rightminparent->_left = rightmin->_right;
					else
						rightminparent->_right = rightmin->_right;
					delete rightmin;
					return true;
				}
			}
		}
		return false;
	}

	void _InOrder(Node* root) //中序打印数据
	{
		if (root == nullptr)
			return;

		_InOrder(root->_left);
		cout << root->_key << ":" << root->_value << endl;
		_InOrder(root->_right);
	}

	void InOrder() //使中序打印函数使用更方便
	{
		_InOrder(_root);
		cout << endl;
	}

private:
	Node* _root = nullptr;
};


void test_BST3() //key/value模型可以做简单的英译汉
{
	BSTree<string, string> dict;
	dict.Insert("sort", "排序");
	dict.Insert("int", "整形");
	dict.Insert("computer", "计算机");
	dict.Insert("mouse", "老鼠");

	string eng;
	while (cin >> eng)
	{
		BSTNode<string, string>* ret = dict.Find(eng);
		if (ret)
			cout << ret->_value << endl;
		else
			cout << "Not Found" << endl;
	}
}

void test_BST4()//也可以做到统计数据出现次数
{
	string Arr[] = { "hehe","haha", "haha", "hehe", "hehe"};
	BSTree<string, int> CountV;
	for (auto str : Arr)
	{
		BSTNode<string, int>* ret = CountV.Find(str);
		if (ret)
			ret->_value++;
		else
			CountV.Insert(str, 1);
	}
	CountV.InOrder();
}
```
