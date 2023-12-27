---
title: 数据结构常见类型的搭建
date: 2023-10-21 21:01:40
tags:
- 数据结构
categories: 
- C语言
cover: /pic/1.png
---


# 一、栈

----

## 头文件(Stack.h)

```c
#pragma once
#define  _CRT_SECURE_NO_WARNINGS
#include<stdio.h>
#include<stdlib.h>
#include<assert.h>

typedef int StackData;
typedef struct Stack
{
	StackData* _a;  //存放数据
	int _top;       //顶标
	int _capacity;  //存放数量
}Sta;


void StackInit(Sta* pst);     //初始化堆栈

void StackDestroy(Sta* pst);  //销毁堆栈

void StackPush(Sta* pst,StackData x);  //插入数据

void StackPop(Sta* pst);  //删除数据

int StackSize(Sta* pst);  //堆栈数据个数
// int StackSize(Sta st);  不传入指针也可以,但为了接口函数的一致性传入指针也可以

int StackEmpty(Sta* pst);  //判断堆栈是否为空

StackData StackTop(Sta* pst);  //取出堆栈头的数据

```
---

## 函数接口部分(Stack.c)

```c
#include "Stack.h"
void StackInit(Sta* pst)
{
    assert(pst);

    /*pst->_a = NULL;
    pst->_capacity = 0;
    pst->_top = 0;*/

    //初始化之后还需再分配,可写为  以简洁

    pst->_a = NULL;
    pst->_top = 0;
    pst->_capacity = 0;
}


void StackDestroy(Sta* pst)
{
    assert(pst);
    free(pst->_a);
    pst->_a = NULL;
    pst->_top = 0;
    pst->_capacity = 0;
}


void StackPush(Sta* pst,StackData x)
{
    assert(pst);
    //判断空间是否足够,增容
    if (pst->_top == pst->_capacity)
    {
        int capacity = pst->_capacity == 0 ? 4 : pst->_capacity * 2;
        StackData* tmp = (StackData*)realloc(pst->_a,sizeof(StackData)*capacity);
        if (tmp != NULL)
        {
            pst->_a = tmp;
        }
        else
        {
            printf("realloc error");
            exit(-1);
        }
        pst->_capacity = capacity;
    }
    
    pst->_a[pst->_top] = x;
    pst->_top += 1;        //top标内没有数据
}


void StackPop(Sta* pst)
{
    assert(pst);
    assert(pst->_top > 0); //当堆栈满时不再进行删除
    pst->_top--;
}


int StackSize(Sta* pst)
{
    assert(pst);
    return pst->_top;
}


int StackEmpty(Sta* pst)  //为空返回1,否则返回0
{
    assert(pst);
    return pst->_top > 0 ? 0 : 1;
}


StackData StackTop(Sta* pst)
{
    assert(pst);
    assert(pst->_top > 0);         //堆栈内不能为空
    return pst->_a[pst->_top -1];  //减1的原因看 push
}
```

---

## 测试用例(test.c)

```c
#include "Stack.h"

void test1()
{
	Sta stack;
	StackInit(&stack);
	StackPush(&stack, 1);
	StackPush(&stack, 2);
	StackPush(&stack, 3);
	StackPush(&stack, 4);
	StackPush(&stack, 5);

	StackPop(&stack);
	StackPop(&stack);

	while (StackEmpty(&stack) != 1)
	{
		printf("%d ", StackTop(&stack));
		StackPop(&stack);
	}

	StackDestroy(&stack);
}

int main()
{
	test1();
	return 0;
}
```

---

# 二、队列

---

## 头文件(Queue.h)

```c
#pragma once
#define  _CRT_SECURE_NO_WARNINGS
#include<stdio.h>
#include<stdlib.h>
#include<assert.h>
#include<stdbool.h>  //布尔值,当然也可以用int代替,但布尔值较为方便,具体实现看函数

typedef int QueDataType;

typedef struct QueueNode
{
	struct QueueNode* Next;
	QueDataType data;
}QueNode;

typedef struct Queue
{
	QueNode* _head;
	QueNode* _tail;
}Que;

void QueueInit(Que* pq);   //初始化队列

void QueueDestroy(Que* pq); //销毁队列

void QueuePush(Que* pq, QueDataType x);  //插入数据

void QueuePop(Que* pq);  //删除数据

QueDataType QueueFront(Que* pq);  //取头数据

QueDataType QueueBack(Que* pq);   //取尾数据

int QueueSize(Que* pq);  //得到队列数据个数

bool QueueEmpty(Que* qp);  //判断队列是否为空
```

---

## 函数接口部分(Queue.c)

```c
#include "Queue.h"

void QueueInit(Que* pq)
{
	assert(pq);
	pq->_tail = pq->_head = NULL;
}


void QueueDestroy(Que* pq)
{
	assert(pq);
	QueNode* cur = pq->_head;
	while (cur != NULL)
	{
		QueNode* next = cur->Next;
		free(cur);
		cur = next;
	}
	pq->_head = pq->_tail = NULL;
}


void QueuePush(Que* pq, QueDataType x)  //数据放在tail部
{
	assert(pq);
	QueNode* newNode = (QueNode*)malloc(sizeof(QueNode));
	if (newNode != NULL)
	{
		newNode->data = x;
		newNode->Next = NULL;   //没有这一步,后面的 pop 和 Empty 就难以实现
	}
	else
	{
		printf("malloc error");
		exit(-1);
	}

	//当队列为空时
	if (QueueEmpty(pq))
	{
		pq->_tail = pq->_head = newNode;
	}
	else
	{
		//QueNode* tail = pq->_tail;
		pq->_tail->Next = newNode;
		pq->_tail = newNode;
	}
}


void QueuePop(Que* pq)
{
	assert(pq);
	//判断队列不为空
	assert(!(QueueEmpty(pq)));
	//assert(pq->_head != pq->_tail);

	QueNode* headnext = pq->_head->Next;
	free(pq->_head);
	pq->_head = headnext;
	if (pq->_head == NULL)  //防止把tail释放后却不为空指针仍然可以调用
	{
		pq->_tail = NULL;
	}
}


bool QueueEmpty(Que* pq)
{
	assert(pq);
	return pq->_head == NULL;  //为真返回ture(非0),为假返回false(0)
}


QueDataType QueueBack(Que* pq)
{
	assert(pq);
	assert(!(QueueEmpty(pq)));
	return pq->_tail->data;
}


QueDataType QueueFront(Que* pq)
{
	assert(pq);
	assert(!(QueueEmpty(pq)));
	return pq->_head->data;
}


int QueueSize(Que* pq)
{
	assert(pq);

	int count = 0;
	QueNode* cur = pq->_head;
	while (cur != NULL)
	{
		cur = cur->Next;
		count++;
	}
	return count;
}
```

---

## 测试用例(test.c)

```c
#include "Queue.h"

void test1()
{
	Que queue;
	QueueInit(&queue);
	QueuePush(&queue, 1);
	QueuePush(&queue, 2);
	QueuePush(&queue, 3);

	QueuePop(&queue);

	
	
	while (!QueueEmpty(&queue))
	{
		printf("%d ", QueueFront(&queue));
		QueuePop(&queue);
	}
	printf("\n");
	QueueDestroy(&queue);
}

int main()
{
	test1();
	return 0;
}
```

---

# 三、无头单向非循环链表

---
## 头文件(SL.h)

```c
#pragma once
#define  _CRT_SECURE_NO_WARNINGS
#include<stdio.h>
#include<stdlib.h>
#include<assert.h>

typedef int typeSL;
typedef struct SListNode
{
	typeSL data;
	struct SListNode* next;
}SLNode;

void SListpushfront(SLNode ** pphead,typeSL x); //头插  实现过程中要改变指针的内容,所以传二级指针  end

void SListpopfront(SLNode** pphead); //头删  end

void SListpushback(SLNode** pphead,typeSL x); //尾插  end

void SListpopback(SLNode** pphead); //尾删  end

void SListInsertAfter(SLNode* pos,typeSL x); //后插  不前插,因为查找后返回的地址想要得到上一个链书写量过大 end

void SListEraseAfter(SLNode* phead); //后删

void SListInsert(SLNode**pphead,SLNode*pos,typeSL x);    //前插;

void SListErase(SLNode**pphead,SLNode*pos);     //删除pos数据

SLNode* SListFind(SLNode* phead , typeSL x);//查找  end

void SListprint(SLNode* phead); //打印  end

SLNode* BuySListNode(typeSL x); //创建新节点  end

void SListDestory(SLNode** pphead); //销毁链表
```

---
## 函数接口部分(SL.c)

```c
#include "SL.h"

SLNode* BuySListNode(typeSL x) //创建结点并赋值
{
	SLNode* newNode = (SLNode*)malloc(sizeof(SLNode));
	if (newNode == NULL)
	{
		perror("Buy malloc");
		exit(-1);
	}
	newNode->data = x;
	newNode->next = NULL; //至空
	return newNode;
}


void SListpushback(SLNode** pphead,typeSL x) //尾插
{
	assert(pphead);  //即使*pphead为NULL , pphead不可能为空,assert可以方便我们更快的找出错误
	                 //并且当pphead为NULL时,会使*pphead访问冲突

	SLNode* newNode = BuySListNode(x); //创建节点
	if (*pphead == NULL) //如果没有结点,直接赋
	{
		*pphead = newNode;
	}
	else
	{
		//找尾
		SLNode* tail = *pphead;
		while (tail->next)
		{
			tail = tail->next;
		}
		tail->next = newNode;  //对地址上的数据进行改变
	}
}


void SListpopback(SLNode** pphead) //尾删
{
	assert(pphead);
	if (*pphead == NULL) //1.没有结点  assert(*pphead);
	{
		return;
	}
	else if ((*pphead)->next == NULL) //2.只有一个节点  注意括号的作用
	{
		free(*pphead);
		*pphead = NULL;
	}
	else                              //3.一个以上的节点
	{
		//找尾
		SLNode* prev = NULL;
		SLNode* tail = *pphead;
		while (tail->next)
		{
			prev = tail;
			tail = tail->next;
		}
		free(tail);
		prev->next = NULL;
	}
}


void SListprint(SLNode* phead) //打印链表
{
	SLNode* cur = phead;
	while (cur) //处理当前位置,不用cur->next
	{
		printf("%d->",cur->data );
		cur = cur->next;
	}
	printf("NULL\n");
}


void SListpushfront(SLNode** pphead, typeSL x) //头插
{
	assert(pphead);
	SLNode* newNode = BuySListNode(x);
	newNode->next = *pphead;
	*pphead = newNode; //head是头,这一步使头变成插入的数据
}


void SListpopfront(SLNode** pphead) //头删
{
	assert(pphead);
	//1.没有结点
	//2.有一个或一个以上的结点
	
	if (*pphead == NULL) //没有结点时   assert(*pphead);
	{
		return;
	}
	else                 //有一个或一个以上,实现方法一样,故而写在一起
	{
		SLNode* Next = (*pphead)->next;
		free(*pphead);
		*pphead = Next;
	}
}

SLNode* SListFind(SLNode* phead, typeSL x) //查找目标数值,返回地址
{
	SLNode* pos = phead;
	while (pos)
	{
		if(pos->data == x)
		{
			return pos;
		}
		pos = pos->next;
	}
	return NULL;
}


void SListInsertAfter( SLNode* pos, typeSL x)  //根据返回地址之后进行插入
{
	assert(pos);

	SLNode* newNode = BuySListNode(x);
	newNode->next = pos->next;
	pos->next = newNode;
}


void SListEraseAfter(SLNode* pos) //后删
{
	assert(pos);
	//assert(pos->next);  可以加上,使得后删时pos后面必须有数字
	if (pos->next)  //如果pos后面有元素
	{
		SLNode* posnext = pos->next;
		SLNode* posnextnext = posnext->next; //pos->next = posnext->next;
		pos->next = posnextnext;             //可以代替这两行
		free(posnext);
		//posnext->next = NULL;  由于函数运行后会自动释放内存,所以置空指针可以省去
	}
}


void SListDestory(SLNode** pphead)   //销毁链表
{
	assert(pphead);

	SLNode* cur = *pphead;
	while (cur)
	{
		SLNode* next = cur->next;   //每次额外创建一个存储cur的下一个
		free(cur);                  //释放指针指向的内存后
		cur = next;                 //cur在变为额外创建的指针
	}
	*pphead = NULL;
}

void SListInsert(SLNode** pphead,SLNode*pos,typeSL x)  //前插
{
	assert(pphead);
	assert(pos);

	SLNode* newNode = BuySListNode(x);
	if (*pphead == pos)    //当pphead就是所要寻找数字的位置
	{
		newNode->next = pos;
		*pphead = newNode;
	}
	else
	{
		SLNode* pospre = *pphead;   //找到pos前的位置
		while (pospre->next != pos)
		{
			pospre = pospre->next;
		}
		pospre->next = newNode;
		newNode->next = pos;
	}
}


void SListErase(SLNode** pphead, SLNode* pos)   //删除pos数据
{
	assert(pphead);
	assert(pos);

	if(pos == *pphead)
	{
		//直接调用头删函数
		SListpopfront(pphead);
	}
	else
	{
		//找到pos前的位置
		SLNode* pospre = *pphead;   //找到pos前的位置
		while (pospre->next != pos)
		{
			pospre = pospre->next;
		}
		pospre->next = pos->next;
		free(pos);
		//pos = NULL;
	}
	
}
```

## 测试用例(test.c)

```c
#include "SL.h"

void test1()
{
	SLNode* phead =NULL; //初始化
	SListpushfront(&phead, 1);
	SListpushfront(&phead, 2);
	SListpushfront(&phead, 3);
	SListpushfront(&phead, 4);
	SListprint(phead);  //4321N

	SListpopfront(&phead);
	SListpopfront(&phead);
	SListpopfront(&phead);
	SListprint(phead);  //1N

	SListpushback(&phead, 2);
	SListpushback(&phead, 3);
	SListpushback(&phead, 4);
	SListpushback(&phead, 5);
	SListprint(phead); //12345N

	SListpopback(&phead);
	SListpopback(&phead);
	SListpopback(&phead);
	SListprint(phead); //12N

	SListInsertAfter(SListFind(phead, 1), 3);
	SListprint(phead);    //132N
	SListEraseAfter(SListFind(phead, 3));
	SListprint(phead);    //13N 

	SListDestory(&phead);
	SListprint(phead); //N
}


void test2()
{
	SLNode* phead = NULL; //初始化
	SListpushfront(&phead, 1);
	SListpushfront(&phead, 2);
	SListpushfront(&phead, 3);
	SListpushfront(&phead, 4);
	SListprint(phead);  //4321N

	SListInsert(phead, SListFind(phead, 3), 1);
	SListprint(phead);  //41321N
}
int main()
{
	test1();
	//test2();
	return 0;
}
```

---

# 四、有头双向循环链表

---
## 头文件(ListNode.h)

```c
#pragma once
#define  _CRT_SECURE_NO_WARNINGS
#include<stdio.h>
#include<stdlib.h>
#include<assert.h>

typedef int ListNode;

typedef struct ListNode
{
	ListNode data;
	ListNode* next;
	ListNode* prev;
}LNode;

LNode* ListNodeInit(); //初始化双链表

void ListNodepushback(LNode* phead,ListNode x); //尾插

void ListNodepopback(LNode* phead);  //尾删

void ListNodepushfront(LNode* phead,ListNode x);  //头插

void ListNodepopfront(LNode* phead); //头删

void ListNodeDestroy(LNode* phead); //销毁链表

LNode* ListNodeFind(LNode* phead,ListNode x);   //寻找

void ListNodepop(LNode* pos);    //位删

void ListNodepush(LNode* pos,ListNode x);   //位前插

LNode* BuyListNode(ListNode x);   //补充数据

void ListNodePrint(LNode* phead);  //打印链表
```

---

## 函数接口部分(ListNode.c)

```c
#include "ListNode.h"
LNode* BuyListNode(ListNode x)    //创建新链表
{
	LNode* newNode = (LNode*)malloc(sizeof(LNode));
	if (newNode != NULL)
	{
		newNode->data = x;
	}
	else
	{
		printf("newNode创建失败");
		exit(-1);
	}
	return newNode;
}


LNode* ListNodeInit() //也可以创建二级指针,使返回类型为void   初始化
{
	LNode*phead = (LNode*)malloc(sizeof(LNode));
	if (phead != NULL)
	{
		phead->next = phead;
		phead->prev = phead;
		return phead;
	}
	else
	{
		printf("init error");
		exit(-1);
	}
}


void ListNodePrint(LNode* phead)        //打印链表
{
	assert(phead);
	LNode* cur = phead->next;
	while (cur != phead)
	{
		printf("%d ", cur->data);
		cur = cur->next;
	}
}


void ListNodepushback(LNode* phead, ListNode x)  //尾插
{
	assert(phead);
	/*LNode* newNode = BuyListNode(x);
	LNode* tail = phead->prev;

	newNode->prev = tail;
	newNode->next = phead;
	tail->next = newNode;
	phead->prev = newNode;    */

	//相当于在phead位置前插
    //所以断言之后可以写为
	// ListNodepush(phead,x);
	ListNodepush(phead, x);
}




void ListNodepopback(LNode* phead)   //尾删
{
	assert(phead);
	LNode* tail = phead->prev;
	assert(tail != phead);   //当只有哨兵时不进入


	//phead->prev = tail->prev;
	//tail = tail->prev;
	//free(tail->next);   
	//tail->next = phead;

	//ListNodepop(tail);
	ListNodepop(tail);
}


void ListNodepushfront(LNode* phead, ListNode x)  //头插
{
	/*assert(phead);
	LNode* newNode = BuyListNode(x);
	LNode* Next = phead->next;
	phead->next = newNode;
	newNode->next = Next;
	newNode->prev = phead;
	Next->prev = newNode;*/

	//同尾插几乎一样
	//可以写为
	//ListNodepush(phead->next,x);
	ListNodepush(phead->next, x);
}


void ListNodepopfront(LNode* phead)  //头删
{
	assert(phead);
	assert(phead->next != phead);
	/*LNode* Next = phead->next;
	LNode* Nextnext = Next->next;
	phead->next = Nextnext;
	Nextnext->prev = phead;
	free(Next);*/

	//ListNodepop(phead->next);
	ListNodepop(phead->next);
}


void ListNodeDestroy(LNode* phead) //销毁链表
{
	assert(phead);
	LNode* cur = phead->next;
	while (cur != phead)
	{
		LNode* curnext = cur->next;
		free(cur);
		cur = curnext;
	}
	free(phead);
	phead = NULL;   //函数结束自动释放内存 可加可不加
}


LNode* ListNodeFind(LNode* phead, ListNode x)  //寻找数据
{
	LNode* cur = phead->next;
	while (cur != phead)
	{
		if (cur->data == x)
		{
			return cur;
		}
		cur = cur->next;
	}
	return NULL;
}


void ListNodepop(LNode* pos)   //位删,phead被删了怎么办   可以加入哨兵位参数实现防止删除哨兵位
{
	assert(pos);
	LNode* posnext = pos->next;
	LNode* posprev = pos->prev;
	
	posnext->prev = posprev;
	posprev->next = posnext;
	free(pos);
	pos = NULL;
}


void ListNodepush(LNode* pos, ListNode x)  //位前插
{
	assert(pos);
	LNode* newNode = BuyListNode(x);
	LNode* posprev = pos->prev;

	newNode->prev = posprev;
	newNode->next = pos;
	posprev->next = newNode;
	pos->prev = newNode;
}

```

---

## 测试用例(test.c)

```c
#include "ListNode.h"
void test1()
{
	
	LNode* phead = ListNodeInit();
	ListNodepushback(phead, 1);
	ListNodepushback(phead, 2);
	ListNodepushback(phead, 3);
	ListNodepopback(phead);
	ListNodepopback(phead);
	ListNodepopback(phead);
	ListNodepushfront(phead, 1);
	ListNodepushfront(phead, 2);
	ListNodepushfront(phead, 3);
	ListNodepopfront(phead);
	ListNodepush(ListNodeFind(phead, 2), 4);
	ListNodepush(ListNodeFind(phead, 2), 3);
	ListNodepush(ListNodeFind(phead, 2), 3);
	ListNodepop(ListNodeFind(phead, 4));
	ListNodepop(ListNodeFind(phead, 1));
	ListNodepop(ListNodeFind(phead, 3));
	
	ListNodePrint(phead);

	ListNodeDestroy(phead);
	phead = NULL;  //手动置空
}

void test2()
{
	LNode* phead = ListNodeInit();
	ListNodepushback(phead, 1);
	ListNodepushback(phead, 2);
	ListNodepushback(phead, 3);
	ListNodepopback(phead);
	ListNodepopback(phead);
	ListNodepopback(phead);
	ListNodepushfront(phead, 1);
	ListNodepushfront(phead, 2);
	ListNodepushfront(phead, 3);
	ListNodepopfront(phead);
	ListNodepopfront(phead);
	ListNodepopfront(phead);
	ListNodepopfront(phead);
	ListNodePrint(phead);
}
int main()
{
	//test1();
	test2();
	return 0;
}
```

---


# 五、堆
---

## 头文件(Heap.h)

```c

//只对根两边都为小堆情况下使用
//当然可以自行将其处理为根两边都是小堆
//小堆即父都小于子


//对AdjustDown进行处理也可以将顺序表变为大堆,父子大小对比符号改变即可
#pragma once
#define  _CRT_SECURE_NO_WARNINGS
#include<stdio.h>
#include<assert.h>
#include<stdlib.h>
#include<string.h>

typedef int Hpdatatype;

typedef struct Heap
{
	Hpdatatype* _a;
	int _size;
	int _capacity;
}Heap;

void HeapInit(Heap* php, Hpdatatype* x, int n);  //初始化堆
void AdjustDown(Hpdatatype* a, int n, int root); //向下调整算法
void swap(Hpdatatype* a, Hpdatatype* b);   //交换两数据函数

void HeapDestroy(Heap* php);  //销毁堆

void HeapPush(Heap* php, Hpdatatype x);   //添加数据

void HeapPop(Heap*php);   //删除一个数据

Hpdatatype Heaptop(Heap* php);  //取root
```

---

## 函数接口的实现(Heap.c)

```c
#include "Heap.h"

void swap(Hpdatatype * a,Hpdatatype *b)  //AjustDown 内部的交换函数,接下的函数也会经常调用, 因此创建
{
	Hpdatatype* tmp = *a;
	*a = *b;
	*b = tmp;
}


void AdjustDown(Hpdatatype *a,int n ,int root)  //原顺序表,数量,根
{
	int parant = root;
	int child = parant * 2 + 1;  //创建坐标  默认较小的孩子为左孩子

	while (child < n)  //当孩子坐标超过创建数组大小,停止向下调整算法
	{
		if (child + 1 < n && a[child + 1] < a[child]) //判断child+1 < n 防止越界, 如果 >n 说明只有一个孩子
		{
			child = child + 1;    //通过计算将较小孩子的坐标变为child
		}

		if (a[parant] > a[child]) //当父亲大于子,开始交换
		{
			swap(&a[parant], &a[child]);

			//交换后改变坐标 继续下次循环
			parant = child;
			child = child * 2 + 1;
		}

		else  //当父亲本来就比子小
		{
			break;   //因为默认两边为小堆,所以直接break跳出
		}
	}
}
void HeapInit(Heap* php, Hpdatatype* x, int n)  //n为总结点
{
	php->_a = (Hpdatatype*)malloc(sizeof(Heap) * n); //先创建空间供给于 php
	if (php->_a != NULL)
	{
		memcpy(php->_a, x, sizeof(Hpdatatype) * n);//将函数外部创建的数组复制到 php内,以完成初始化
	}
	else
	{
		printf("malloc error");
		exit(-1);
	}
	    

	php->_size = n;
	php->_capacity = n;

	//以上只是构建出了一个数组
	//以下为构建堆

	//从坐标最后的 叶的父,开始使用向下调整算法,使其变为小堆
	//最后一个数据坐标为 n-1  , 那么推出其父坐标为 (坐标 -1) /2
	//所以得(n-1-1)
	for (int i = (n - 1 - 1) / 2; i >= 0; i--)
	{
		AdjustDown(php->_a,php->_size,i);
	}
}


void HeapDestroy(Heap* php)
{
	assert(php);
	free(php->_a);
	php->_a = NULL;
	php->_size = 0;
	php->_capacity = 0;
}


void AdjustUp(Hpdatatype* a, int n, int child) //数组,数量,最后一个孩子
{
	int parent = (child - 1) / 2;

	//while (parent >= 0)  parent的判断条件不允许其为负数  -1 / 2 = 0;
	while (child > 0)
	{
		if (a[child] >= a[parent])
		{
			break;
		}
		else
		{
			swap(&a[child], &a[parent]);
			child = parent;
			parent = (parent - 1) / 2;
		}
	}
}
void HeapPush(Heap* php, Hpdatatype x)  //添加数据后 考虑扩容  关系的改变
{
	assert(php);
	if (php->_size == php->_capacity)
	{
		php->_capacity *= 2;
		Hpdatatype* tmp = (Hpdatatype*)realloc(php->_a, sizeof(Hpdatatype) * php->_capacity); //扩容
		if (tmp != NULL)
		{
			php->_a = tmp;
		}
		else
		{
			printf("realloc error");
			exit(-1);
		}
	}
	php->_a[php->_size] = x;
	++php->_size;

	//添加完开始处理关系
	//观察可发现并不是需要调整整个树,而是一条线的关系,即向上调整算法

	AdjustUp(php->_a, php->_size, php->_size - 1);
}


void HeapPop(Heap* php)  //删除堆顶数据  普通删除方式删除后还需要移动整个顺序表,这里采用头尾交换后除尾
{
	assert(php);
	assert(php->_size > 0);

	swap(&php->_a[0], &php->_a[php->_size - 1]);
	--php->_size;

    //交换后对除去尾的顺序表再Adjust 使头变为次小数 
	//调整树之间的关系
	AdjustDown(php->_a,php->_size,0);
}


Hpdatatype Heaptop(Heap* php)  //取根数据 , 即当前树 最小/最大 的数据
{
	assert(php);
	assert(php->_size > 0);

	return php->_a[0];
}
```

---

## 测试用例(test.c)

```c
#include "Heap.h"

void HeapSort(int* a, int n)  //a为数组,n为数组元素个数  这里测试降序排序  降序创建小堆  升序创建大堆
{
	for (int i = (n - 1 - 1) / 2; i >= 0; --i)  //先将数组变为小堆
	{
		AdjustDown(a,n,i);
	}

	//开始通过交换首尾的方式进行排序
	//首在小堆中绝对是最小的数,将它放在最后
	//再暂时排除最后一个数,进行Adjust
	//首即为次小的数
	//以此类推便可得到降序排列的数据 
	int end = n - 1;  //end 为最后一个数据的坐标
	while (end > 0)
	{
		swap(&a[0], &a[end]);
		AdjustDown(a, end, 0);
		--end;
	}
}
void test2()
{
	int arr[] = { 27,15,19,18,28,34,65,49,25,37 };
	HeapSort(arr, sizeof(arr) / sizeof(Hpdatatype));
}



void test1()
{
	int arr[] = { 27,15,19,18,28,34,65,49,25,37 };
	Heap hp;
	HeapInit(&hp, arr, sizeof(arr) / sizeof(Hpdatatype)); //arr整个大小除一个Hpdata大小得出arr内数据个数
	HeapPush(&hp, 3);
	HeapPush(&hp, 66);
}


int main()
{
	test1();  //接口测试

	//test2(); //通过堆排序
	return 0;
}
```

---

# 六、完全二叉树

---
## 头文件(BinaryTree.h)

```c
//树本质上也是递归出来的树
#pragma once
#define  _CRT_SECURE_NO_WARNINGS
#include<stdio.h>
#include<stdlib.h>
#include<assert.h>
#include<string.h>
#include<stdbool.h> //布尔值的头文件包含

typedef char BTDataType;
typedef struct BinaryTreeNode
{
	BTDataType _data;
	struct BinaryTreeNode* _left;
	struct BinaryTreeNode* _right;
}BTNode;

BTNode* CreatBTNode(BTDataType x);  //创建一个树结点

int BTSize(BTNode* root);  //求出树的节点个数

int BTLeafSize(BTNode* root); //求出树叶的个数

int BTDeepSize(BTNode* root); //求出树的深度

void PrintPrevOrder(BTNode* root); //按前序打印树

void PrintInOrder(BTNode* root); //按中序打印树

void PrintBackOrder(BTNode* root); //按后序打印树

int BTLevelKSize(BTNode* root,int k); //树的第K层结点个数

BTNode* BTFindX(BTNode* root, BTDataType x); //寻找结点数据为x的结点

void BTDestroy(BTNode* root);  //销毁树

void PrintLevelOrder(BTNode* root); //层序打印树  此时需要使用队列方便打印

bool BTComplete(BTNode* root);  //判断树是否为完全二叉树




//------------------------------------------------------------------------------------------
//以下为队列的声明
//此时队列内部数据应为 树
//以调用树的孩子 来进行算法的实现
//放入队列第一个根的左右孩子,同时将队列第一个根移除

typedef BTNode* QueDataType;  //队列类型为树,以便接口实现
typedef struct QueueNode
{
	struct QueueNode* Next;
	QueDataType data;
}QueNode;

typedef struct Queue
{
	QueNode* _head;
	QueNode* _tail;
}Que;

void QueueInit(Que* pq);   //初始化队列

void QueueDestroy(Que* pq); //销毁队列

void QueuePush(Que* pq, QueDataType x);  //插入数据

void QueuePop(Que* pq);  //删除数据

QueDataType QueueFront(Que* pq);  //取头数据

QueDataType QueueBack(Que* pq);   //取尾数据

int QueueSize(Que* pq);  //得到队列数据个数

bool QueueEmpty(Que* qp);  //判断队列是否为空
```

---
## 函数接口的实现(BinaryTree.c)

```c
#include "BinaryTree.h"

void PrintPrevOrder(BTNode* root)  //以递归为主
{
	if (root == NULL)
	{
		printf("NULL ");
		return;
	}
	printf("%c ",root->_data);
	PrintPrevOrder(root->_left);
	PrintPrevOrder(root->_right);
}

void PrintInOrder(BTNode* root)
{
	if (root == NULL)
	{
		printf("NULL ");
		return;
	}
	PrintInOrder(root->_left);
	printf("%c ", root->_data);
	PrintInOrder(root->_right);
}

void PrintBackOrder(BTNode* root)
{
	if (root == NULL)
	{
		printf("NULL ");
		return;
	}
	PrintBackOrder(root->_left);
	PrintBackOrder(root->_right);
	printf("%c ", root->_data);
}


BTNode* CreatBTNode(BTDataType x)   //创建一个树结点
{
	BTNode* node = (BTNode*)malloc(sizeof(BTNode));
	if (node != NULL)
	{
		node->_left = node->_right = NULL;
		node->_data = x;
		return node;
	}
	else
	{
		printf("Creat malloc error");
		exit(-1);
	}
}


//求出节点个数,int*以防止数据重复或丢失,但并不为最优解
//int BTSize(BTNode* root,int* size) 
//{
//	if (root == NULL)
//	{
//		return 0;
//	}
//	else
//	{
//		(*size)++;
//	}
//	BTSize(root->_left,size);
//	BTSize(root->_right,size);
//	return *size;
//}


int BTSize(BTNode* root)  //求出树结点个数优解
{
	if (root == NULL)
	{
		return 0;
	}
	else
	{
		return 1 + BTSize(root->_left) + BTSize(root->_right);  //递归思想
	}
}


int BTLeafSize(BTNode* root)  //求出树叶个数
{
	if (root == NULL)
	{
		return 0;
	}
	if (root->_left == NULL && root->_right == NULL)
	{
		return 1;
	}
	else
	{
		return BTLeafSize(root->_left) + BTLeafSize(root->_right);  //递归思想
	}
}


int BTDeepSize(BTNode* root)
{
	if (root == NULL)
	{
		return 0;
	}

	//return BTDeepSize(root->_left) > BTDeepSize(root->_right) ? 1+BTDeepSize(root->_left) : 1+BTDeepSize(root->_right);
	//这样写虽然不错,但是效率太低,递归重复

	int leftdepth = BTDeepSize(root->_left);
	int rightdepth = BTDeepSize(root->_right);
	return leftdepth > rightdepth ? leftdepth + 1 : rightdepth + 1;
}


int BTLevelKSize(BTNode* root,int k)
{
	if (root == NULL)
	{
		return 0;
	}
    
	if (k == 1)  //第一层只有根,因此只有一个
	{
		return 1;
	}

	return BTLevelKSize(root->_left, k - 1) + BTLevelKSize(root->_right, k - 1);
	//每当进入左或右子树,此时所求的第K层变为相对于新根的K-1层
	//当到所求的K层时, k==1 成立, +1;
}


BTNode* BTFindX(BTNode* root, BTDataType x)
{
	if (root == NULL)  
	{
		return NULL;
	}

	if (root->_data == x)
	{
		return root;
	}

	BTNode* node = BTFindX(root->_left,x);  //以下递归可以直接求出结果
	if (node)
	{
		return node;
	}

	node = BTFindX(root->_left, x);
	if (node)
	{
		return node;
	}

	return NULL;
}


void BTDestroy(BTNode* root)
{
	if (root == NULL)
	{
		return;
	}

	BTDestroy(root->_left);
	BTDestroy(root->_right);
	free(root);
}


void PrintLevelOrder(BTNode* root) //调用队列
{
	Que que;
	QueueInit(&que);
	QueuePush(&que, root);  //传入树根

	while (!(QueueEmpty(&que)))
	{
		BTNode* front = QueueFront(&que);  //取出队列头结点,并删除
		QueuePop(&que);
		printf("%c ", front->_data);

		if (front->_left)   //当结点孩子不为空时,就放入其孩子于队列
		{
			QueuePush(&que, front->_left);
		}

		if (front->_right)
		{
			QueuePush(&que, front->_right);
		}
	}
	QueueDestroy(&que);  //销毁队列释放
}


bool BTComplete(BTNode* root)  //若为完全二叉树,则在队列中如果出现NULL,则后面全是NULL
{
	if (root == NULL)
	{
		return true;
	}

	Que que;
	QueueInit(&que);

	while (!QueueEmpty(&que))
	{
		BTNode* front = QueueFront(&que);
		QueuePop(&que);

		if (front == NULL)
		{
			break;
		}

		QueuePush(&que, front->_left); //放在if后,防止break前Push空指针的左右孩子
		QueuePush(&que, front->_right);
	}

	while (!QueueEmpty(&que))
	{
		BTNode* front = QueueFront(&que);
		if (front != NULL)
		{
			QueueDestroy(&que);
			return false;
		}
		QueuePop(&que);
	}

	return true;
	QueueDestroy(&que);
}
```

---
## 队列的引用接口实现部分(Queue.c)

```c
#include "BinaryTree.h"

void QueueInit(Que* pq)
{
	assert(pq);
	pq->_tail = pq->_head = NULL;
}


void QueueDestroy(Que* pq)
{
	assert(pq);
	QueNode* cur = pq->_head;
	while (cur != NULL)
	{
		QueNode* next = cur->Next;
		free(cur);
		cur = next;
	}
	pq->_head = pq->_tail = NULL;
}


void QueuePush(Que* pq, QueDataType x)  //数据放在tail部
{
	assert(pq);
	QueNode* newNode = (QueNode*)malloc(sizeof(QueNode));
	if (newNode != NULL)
	{
		newNode->data = x;
		newNode->Next = NULL;   //没有这一步,后面的 pop 和 Empty 就难以实现
	}
	else
	{
		printf("malloc error");
		exit(-1);
	}

	//当队列为空时
	if (QueueEmpty(pq))
	{
		pq->_tail = pq->_head = newNode;
	}
	else
	{
		//QueNode* tail = pq->_tail;
		pq->_tail->Next = newNode;
		pq->_tail = newNode;
	}
}


void QueuePop(Que* pq)
{
	assert(pq);
	//判断队列不为空
	assert(!(QueueEmpty(pq)));
	//assert(pq->_head != pq->_tail);

	QueNode* headnext = pq->_head->Next;
	free(pq->_head);
	pq->_head = headnext;
	if (pq->_head == NULL)  //防止把tail释放后却不为空指针仍然可以调用
	{
		pq->_tail = NULL;
	}
}


bool QueueEmpty(Que* pq)
{
	assert(pq);
	return pq->_head == NULL;  //为真返回ture(非0),为假返回false(0)
}


QueDataType QueueBack(Que* pq)
{
	assert(pq);
	assert(!(QueueEmpty(pq)));
	return pq->_tail->data;
}


QueDataType QueueFront(Que* pq)
{
	assert(pq);
	assert(!(QueueEmpty(pq)));
	return pq->_head->data;
}


int QueueSize(Que* pq)
{
	assert(pq);

	int count = 0;
	QueNode* cur = pq->_head;
	while (cur != NULL)
	{
		cur = cur->Next;
		count++;
	}
	return count;
}
```

---

## 测试用例(test.c)

```c
#include "BinaryTree.h"

int main()
{
	BTNode* A = CreatBTNode('A');
	BTNode* B = CreatBTNode('B');
	BTNode* C = CreatBTNode('C');
	BTNode* D = CreatBTNode('D');
	BTNode* E = CreatBTNode('E');
	//    A
	//  B   C
	// D E 

	A->_left = B;  
	A->_right = C; 
	B->_left = D;  
	B->_right = E; 

	PrintPrevOrder(A);  //前序打印
	printf("\n");

	PrintInOrder(A);    //中序打印
	printf("\n");

	PrintBackOrder(A);  //后序打印  以上三种为顺序
	printf("\n");

	PrintLevelOrder(A); //层序打印  层序
	printf("\n");

	//以下注释非最优解的处理方式
	/*int sizea = 0;
	sizea = BTSize(A);
	printf("ATree's Size is %d",sizea);

	int sizeb = 0;
	sizeb = BTSize(B);
	printf("BTree's Size is %d", sizeb);*/

	//    A
	//  B   C
	// D E 

	printf("ATree %d\n", BTSize(A)); //5
	printf("BTree %d\n", BTSize(B)); //3

	printf("ATree leaf %d\n", BTLeafSize(A)); //3
	printf("BTree leaf %d\n", BTLeafSize(B)); //2

	printf("ATree Deep %d\n", BTDeepSize(A)); //3
	printf("ATree Deep %d\n", BTDeepSize(A)); //3
	printf("BTree Deep %d\n", BTDeepSize(B)); //2

	if (BTComplete(A))
	{
		printf("complete");
	}
	else
	{
		printf("uncomplete");
	}
	return 0;
}
```

---
