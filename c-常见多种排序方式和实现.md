---
title: 常见多种排序方式和实现
date: 2023-10-21 21:02:10
tags:
- 排序算法
categories: 
- C语言
cover: /pic/1.png
---



# 头文件(Sort.h)

```c
#pragma once
#define  _CRT_SECURE_NO_WARNINGS
#include<stdio.h>
#include<stdlib.h>
#include<assert.h>
#include<time.h>
#include<string.h>


void PrintArray(int* arr,int n); //打印数组

void Swap(int* a, int* b); //交换数据

//默认全部实现升序排列
//插入排序
void InsertSort(int* arr,int n); //插入排序 : 稳定 最坏O( N^2 )  最好O( N )

void ShellSort(int* arr, int n); //希尔排序 : 不稳定 最坏O( N^2 )  最好O( N^1.3)


//选择排序
void SelectSort(int* arr, int n); //选择排序 : 不稳定 O( N^2 )  

void HeapSort(int* arr, int n); //堆排序 : 稳定 O( N*logN )
void AdjustDown(int* arr, int n,int root); //堆排序所需要的向下调整接口


//交换排序
void BubbleSort(int* arr, int n); //冒泡排序 : 稳定 O(N^2)

void QuickSort(int* arr,int left,int right); //快排 : 不稳定 三数取中、区间非快排优化后O( N*LogN )  空间复杂度 O(logN)


//归并排序
void MergeSort(int* arr, int n);  //归并排序 : 稳定 O( N*logN ) 空间复杂度 O(N) 

void MergeSortFile(const char* file); //对文件进行排序,适用于大量数据


//计数排序
void CountSort(int* arr, int n); 

//----------------------------------------------------------------------------------
//以下为非递归快排所需要的栈头文件

typedef int StackData;
typedef struct Stack
{
	StackData* _a;  //存放数据
	int _top;       //顶标
	int _capacity;  //存放数量
}Sta;


void StackInit(Sta* pst);     //初始化堆栈

void StackDestroy(Sta* pst);  //销毁堆栈

void StackPush(Sta* pst, StackData x);  //插入数据

void StackPop(Sta* pst);  //删除数据

int StackSize(Sta* pst);  //堆栈数据个数
// int StackSize(Sta st);  不传入指针也可以,但为了接口函数的一致性传入指针也可以

int StackEmpty(Sta* pst);  //判断堆栈是否为空

StackData StackTop(Sta* pst);  //取出堆栈头的数据
```

---

# 函数接口部分(Sort.c)

```c
#include "Sort.h"
void Swap(int* a, int* b)
{
	int tmp = *a;
	*a = *b;
	*b = tmp;
}


void PrintArray(int* arr,int n)  //打印数组
{
	for(int i = 0; i < n; i++)
	{
		printf("%d ", arr[i]);
	}
	printf("\n");
}


void InsertSort(int* arr, int n) //插入排序
{                                
	assert(arr);                 
                                 
	for (int i = 0; i < n - 1; i++)
	{
		//以下为一趟,即只排序一个数字			
		int end = i;
		int tmp = arr[end + 1];  //竖线后的值赋给tmp
		while (end >= 0)
		{
			if (arr[end] > tmp)
			{
				arr[end + 1] = arr[end];  //使大于该排序的数整体后移
				--end;
			}
			else
			{
				break;
			}
		}
		arr[end + 1] = tmp;    //最后将值赋给后移部分数组的头
	}
	// 3 6 7 | 2 
	// 3 6 7 | 7 
	// 3 6 6 | 7 
	// 3 3 6 | 7 
	// 2 3 6 | 7 
}


void ShellSort(int* arr, int n) //插入排序的优化 希尔排序
{
	int gap = n;
	while (gap > 1)
	{
		gap = gap / 3 + 1; //+1确保最后一次循环为gap为1的循环,即插入排序
		for (int i = 0; i < n - gap; i++)
		{
			int end = i;
			int tmp = arr[end + gap];
			while (end >= 0)
			{
				if (tmp < arr[end])
				{
					arr[end + gap] = arr[end];
					end -= gap;
				}
				else
				{
					break;
				}
			}
			arr[end + gap] = tmp;
		}
	}
	//间隔gap之间进行插入排序
	//9 1 3 6 7 2 5 4 10 8
}


void SelectSort(int* arr, int n) //选择排序  即选择最小的数排在前面,为了提高效率,同时选择最大和最小
{
	assert(arr);

	int left = 0;
	int	right = n - 1;
	while (left < right)
	{
		int maxi = right;
		int mini = left;
		int tmp = left;
		while (tmp <= right)        //9 1 3 6 7 2 5 4 10 8
		{
			if (arr[tmp] < arr[mini])  //当找到更小的,记录坐标
			{
				//Swap(&arr[tmp], &arr[mini]);  //swap使用太频繁,效率略低
				mini = tmp;
			}
			if (arr[tmp] > arr[maxi])
			{
				//Swap(&arr[tmp], &arr[maxi]);
				maxi = tmp;
			}
			tmp++;
		}
		Swap(&arr[left], &arr[mini]);
		if (left== maxi) //如果left上就是最大值,那么第一次交换会使得第二次交换失效
		{
			maxi = mini;
		}
		Swap(&arr[right], &arr[maxi]);
		left++;
		right--;
	}	
}

void AdjustDown(int* arr, int n, int root) //向下调整算法  使最大位于顶
{
	int parent = root;
	int child = root * 2 + 1;
	
	while (child < n)
	{
		if (arr[child + 1] > arr[child] && child + 1 < n) //是child为最大的孩子,并且不越数组
		{
			child++;
		}
		if (arr[child] > arr[parent])
		{
			Swap(&arr[child], &arr[parent]);
			parent = child;
			child = child * 2 + 1;
		}
		else
		{
			return;
		}
	}
}
void HeapSort(int* arr, int n) //堆排序
{
	//要排升序,建立大堆
	for (int i = (n-1-1)/2; i >= 0; i--)
	{
		AdjustDown(arr, n, i);  //n为结束的标志,虽然开始数据很少,但是i初始赋值很大
	}

	int end = n - 1;
	while (end > 0)
	{
		Swap(&arr[end], &arr[0]);
		AdjustDown(arr, end, 0);
		end--;
	}
}


void BubbleSort(int* arr, int n) //冒泡排序法   9 1 3 6 7 2 5 4 10 8
{
	
	int end = n;
	while (end > 0)
	{
		int exchange = 0;  //当一趟冒泡排序中没有发生交换,则已经有序,不需要再排
		for (int i = 0; i < end; i++) //一趟
		{
			if (arr[i-1] > arr[i])
			{
				Swap(&arr[i-1], &arr[i]);
				exchange = 1;
			}
		}
		end--;
		if (exchange == 0)
		{
			return;
		}
	}
	
}


void _Find_Mid(int* arr,int left,int right) //以防每次快排都是最差的情况(选择出最大或最小的数字作为key)
{
	int mid = (left + right) / 2;
	if (arr[left] < arr[mid])
	{
		if (arr[mid] < arr[right])  // 3 5 7
		{
			Swap(&arr[mid], &arr[right]);
		}
		else if (arr[right] < arr[left])
		{
			Swap(&arr[left], &arr[right]);
		}
		else
		{
			return;
		}
	}
	else if (arr[left] > arr[mid])  //  5  3  1
	{
		if (arr[mid] > arr[right])
		{
			Swap(&arr[mid], &arr[right]);
		}
		else if (arr[right] > arr[left])
		{
			Swap(&arr[left], &arr[right]);
		}
		else
		{
			return;
		}
	}
}

int _Left_right_point_Part_Sort(int* arr,int left,int right)  //左右指针法
{
	_Find_Mid(arr,left,right); //优化

	int keyindex = right;
	int key = arr[right]; //将key默认设置为最后一个数
	//左大右小 进行交换  9 1 3 6 7 2 5 4 10 8

	while (left < right)
	{
		while (arr[left] <= key && left < right)  //这里第二次判断left > right 画图测试
		{
			left++;
		}
		while (arr[right] >= key && left < right)  
		{
			right--;
		}
		Swap(&arr[left], &arr[right]);
	}
	Swap(&arr[left], &arr[keyindex]);
	return left;  //返回key值所在下标
}


int _Dig_Blank_Part_Sort(int* arr, int left, int right)  //挖坑法 本质上与左右指针法类似
{
	_Find_Mid(arr,left,right);
	// 9 1 3 6 7 2 5 4 10 8
	//最右挖坑 从左开始  同样两个指针一起填坑  替换坑位  左大换,右小换
	int key = arr[right];
	
	while (left < right)
	{
		while (left < right && arr[left] < key) //左边小于右边时,向后继续搜索直到找到大的
		{
			left++;
		}
		arr[right] = arr[left];
		while (left < right && arr[right] > key)
		{
			right--;
		}
		arr[left] = arr[right];
	}
	arr[left] = key;
}


int _Prev_Back_point_Part_Sort(int* arr, int left, int right) //前后指针法  类似于将大数翻在前面,小数留在后面
{
	_Find_Mid(arr, left, right);
	int prev = left;
	int back = left - 1;
	while (prev < right)  // 9 1 3 6 7 2 5 4 10 8  前指针小于key(代码中right)就交换前后指针数值
	{                                           
		if( arr[prev] < arr[right] && ++back != prev)//前指针大于key,if条件中(由于&&)不执行++back
		{                                            //前指针小于key,prev不动,先让back动
			Swap(&arr[prev], &arr[back]);            //有时在开始时都小于,back++后,prev == back
		}                                            //交换不交换一样,所以直接跳出if提升效率
		prev++;
	}
	Swap(&arr[++back], &arr[right]); //此时在back前面一定是大于key的,所以++,使key为div
	return back;
}


//递归改非递归一般两种方法 1.简单的递归可以使用普通的循环处理 2.较为困难的需要使用栈
//非递归的方法具有两个缺点
// 1.提高效率,递归需要建立栈帧(每个未运行完的函数都有一个栈帧,栈帧保存了函数的返回地址和局部变量)
//   会有消耗,但对于现代计算机,消耗可以忽略不计
// 2.最大的缺陷是,如果栈帧太深,会导致栈溢出,栈的空间不大,只有10M左右

void _Without_Recursion_Part_Sort(int* arr, int left, int right)
{
	assert(arr);         // 9 1 3 6 7 2 5 4 10 8 将数据坐标入栈
	Sta stack;
	StackInit(&stack);
	StackPush(&stack, right);   //输入坐标
	StackPush(&stack, left);

	while (!StackEmpty(&stack)) //当栈内不为空时
	{
		int begin = StackTop(&stack);
		StackPop(&stack);   //使用完删除
		int end = StackTop(&stack);
		StackPop(&stack);

		int div = _Prev_Back_point_Part_Sort(arr, begin, end);
		if (div + 1 < end)
		{
			StackPush(&stack, end);
			StackPush(&stack, div + 1);
		}
		if (begin < div - 1)
		{
			StackPush(&stack,div - 1);
			StackPush(&stack, begin);
		}
	}
	StackDestroy(&stack);
}


void QuickSort(int* arr, int left, int right)  //快速排序  本质使用递归
{
	assert(arr);
	if (left >= right)
	{
		return;
	}

	//int div = _Left_right_point_Part_Sort(arr, left, right); 
	//int div = _Dig_Blank_Part_Sort(arr, left, right);
	int div = _Prev_Back_point_Part_Sort(arr, left, right);
	//_Without_Recursion_Part_Sort(arr, left, right);  //单独一个函数就可完成排序
	
	//对key值左右进行处理
	//当每个递归后剩余的数字小于15时可以不使用递归,增高效率,此时排序最优解为插入排序

	if (right - left > 15)
	{
		QuickSort(arr, left, div - 1);
		QuickSort(arr, div + 1,right);
	}
	else
	{
		InsertSort(arr, right - left + 1);
	}

	/*QuickSort(arr, left, div - 1);
	QuickSort(arr, div + 1, right);*/
}


void Merge_Sort_Arr_Part( int* arr, int* tmp,int begin1,int end1,int begin2,int end2) //合并的过程
{
	int left = begin1, right = end2;  //为后面赋值arr做准备
	int index = begin1;
	while (begin1 <= end1 && begin2 <= end2)
	{
		if (arr[begin1] <= arr[begin2])
		{
			tmp[index++] = arr[begin1++];
		}
		else
		{
			tmp[index++] = arr[begin2++];
		}
	}

	while (begin1 <= end1)
	{
		tmp[index++] = arr[begin1++];
	}
	while (begin2 <= end2)
	{
		tmp[index++] = arr[begin2++];
	}

	//将tmp内的值赋给arr
	for (int i = left; i <= right; i++)
	{
		arr[i] = tmp[i];
	}
}
void Merge_Sort_Part(int* arr, int* tmp,int left, int right)  //归并排序,将两段有序的数组归并为一段有序的数组
{
	if (left >= right) //分割停止标志
	{
		return;
	}

	int mid = (left + right) / 2;     // 9 1 3 6 7 2 5 4 10 8
	//当两边无序,即左右两段数组数据个数大于1
	//开始分割
	Merge_Sort_Part(arr, tmp, left, mid);
	Merge_Sort_Part(arr, tmp, mid+1, right);


	//当两边有序,左右两端数组数据个数为1
	//开始合并
	Merge_Sort_Arr_Part(arr,tmp,left,mid,mid+1,right);
}


void Merge_No_Recursion_Sort_Part(int* arr, int* tmp, int n) //使用非递归的方式归并
{
	//逐渐增长的gap之间归并
	int gap = 1;
	while (gap < n)
	{
		for (int i = 0; i < n; i+=2*gap)
		{
			int begin1 = i, end1 = i + gap - 1;
			int begin2 = i+gap, end2 = i + 2*gap - 1;

			if (begin2 >= n) //此时被分割的第二组已经超过整个数组,即不存在
			{
				break;
			}
			if (end2 >= n) //此时第二组不能被整分,需要调整end2
			{
				end2 = n - 1;
			}

			//调整之后开始归并
			Merge_Sort_Arr_Part(arr, tmp, begin1, end1, begin2, end2);
		}
		gap *= 2;
	}
}


void MergeSort(int* arr, int n) //归并排序法
{
	assert(arr);
	int* tmp = (int*)malloc(sizeof(int) * n);

	//Merge_Sort_Part(arr,tmp,0,n-1);  //递归法
	Merge_No_Recursion_Sort_Part(arr, tmp,n);  //非递归法

	free(tmp);
}


void MergeFile(const char* file1,const char* file2,const char* mfile)
{
	FILE* fout1 = fopen(file1, "r");
	if (fout1==NULL)
	{
		printf("fout1 error");
		exit(-1);
	}

	FILE* fout2 = fopen(file2, "r");
	if (fout2 == NULL)
	{
		printf("fout2 error");
		exit(-1);
	}

	FILE* fin = fopen(mfile, "w");
	if (fin == NULL)
	{
		printf("fin error");
		exit(-1);
	}

	int num1, num2;
	int ret1 = fscanf(fout1, "%d\n", &num1); //文件内数据赋予num
	int ret2 = fscanf(fout2, "%d\n", &num2);
	while (ret1 != EOF && ret2 != EOF)
	{
		if (num1 < num2)
		{
			fprintf(fin, "%d\n", num1);
			ret1 = fscanf(fout1, "%d\n", &num1);
		}
		else
		{
			fprintf(fin, "%d\n", num2);
			ret2 = fscanf(fout2, "%d\n", &num2);
		}
	}

	while (ret1 != EOF)
	{
		fprintf(fin, "%d\n", num1);
		ret1 = fscanf(fout1, "%d\n", &num1);
	}
	while (ret2 != EOF)
	{
		fprintf(fin, "%d\n", num2);
		ret2 = fscanf(fout2, "%d\n", &num2);
	}
	fclose(fout1);
	fclose(fout2);
	fclose(fin);
}


void MergeSortFile(const char* file) //对文件进行排序
{
	FILE* fout = fopen(file, "r"); //打开待排序文件
	if (fout == NULL)
	{
		printf("fout error");
		exit(-1);
	}

	//将文件内数据分割为小文件,对小文件内可放置入内存的数据进行快排
	//最后将小文件归并

	int n = 10; //一个小文件内存放的数据个数
	int a[10];  //文件内即将快排的数据放入此数组
	int i = 0;
	int num = 0;
	char subfile[20];//文件名字符串
	int filei = 1;//储存第filei组内排序后的数据的文件的文件名

	memset(a, 0, sizeof(int) * n);
	while (fscanf(fout, "%d\n", &num) != EOF) //将fout内数据读入num
	{
		if (i < n - 1)//读取前n-1个数据
		{
			a[i++] = num;
		}
		else
		{
			a[i] = num;//读取第n个数据
			QuickSort(a, 0, n - 1);
			sprintf(subfile, "%d", filei++);//将储存第filei组内排序后的数据的文件的文件名命名为filei
			FILE* fin = fopen(subfile, "w");//创建一个以字符串subfile[20]为名字的文件并打开
			if (fin == NULL)
			{
				printf("first fin error");
				exit(-1);
			}
			for (int i = 0; i < n; i++)//将内排序排好的数据写入到subfile文件中
			{
				fprintf(fin, "%d\n", a[i]);
			}
			fclose(fin);

			i = 0;
			memset(a, 0, sizeof(int) * n);
		}
	}

    //相互排序归并至文件,实现整体有序
	char mfile[100] = "12";
	char file1[100] = "1";
	char file2[100] = "2";
	for (int i = 2; i <= n; i++)
	{
		MergeFile(file1, file2, mfile);//将file1文件和file2文件中的数据归并到mfile文件中

		strcpy(file1, mfile);//下一次待归并的第一个文件就是上一次归并好的文件
		sprintf(file2, "%d", i + 1);//上一次待归并的第二个文件的文件名加一，就是下一次待归并的第二个文件的文件名
		sprintf(mfile, "%s%d", mfile, i + 1);//下一次归并后文件的文件名
	}
	fclose(fout);
}


void CountSort(int* arr, int n) //数据范围内创建数组,遍历后遇见数组下标有关的数据,数组内容++
{
	assert(arr);

	int min = arr[0];
	int max = arr[0];
	//创建数组,范围确定
	for (int i = 0; i < n; i++)
	{
		if (arr[i] > max)
		{
			max = arr[i];
		}
		if (arr[i] < min)
		{
			min = arr[i];
		}
	}
	int range = max - min + 1;
	int* tmp = (int*)malloc(sizeof(int) * range); //创建数组
	memset(tmp, 0, sizeof(int) * range); //数组初始化

	for (int i = 0; i < n; i++)
	{
		tmp[arr[i] - min]++; //每遇见下标对应的数据,和数组数据相同时是,下标内容++
	}

	int index = 0;
	for (int i = 0; i < range; i++)
	{
		while (tmp[i]--) //下标数据为0时停止赋值,进入下一个下标
		{
			arr[index++] = i + min; //i可看作tmp的下标,进行赋值
		}
	}
	free(tmp);
}
```
---
# 非递归方式实现快排的栈接口函数(Stack.c)

```c
#include "Sort.h"
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


void StackPush(Sta* pst, StackData x)
{
    assert(pst);
    //判断空间是否足够,增容
    if (pst->_top == pst->_capacity)
    {
        int capacity = pst->_capacity == 0 ? 4 : pst->_capacity * 2;
        StackData* tmp = (StackData*)realloc(pst->_a, sizeof(StackData) * capacity);
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
    return pst->_a[pst->_top - 1];  //减1的原因看 push
}
```

---

# 测试用例(test.c)

```c
#include "Sort.h"

void TestInsertSort() //插入排序用例
{
	int arr[] = { 9,1,3,6,7,2,5,4,10,8 };
	int sz = sizeof(arr) / sizeof(arr[0]);

	PrintArray(arr, sz);
	InsertSort(arr, sz);
	PrintArray(arr, sz);
}


void TestShellSort() //希尔排序用例
{
	int arr[] = { 9,1,3,6,7,2,5,4,10,8 };
	int sz = sizeof(arr) / sizeof(arr[0]);

	PrintArray(arr, sz);
	ShellSort(arr, sz);
	PrintArray(arr, sz);
}


void TestSelectSort() //选择排序用例
{
	int arr[] = { 10,1,3,6,7,2,5,4,9,8 };
	//int arr[] = { 3,1,4,1,7,9,8,2,0,5};
	int sz = sizeof(arr) / sizeof(arr[0]);

	PrintArray(arr, sz);
	SelectSort(arr, sz);
	PrintArray(arr, sz);
}


void TestHeapSort() //堆排序用例
{
	int arr[] = { 9,1,3,6,7,2,5,4,10,8 };
	int sz = sizeof(arr) / sizeof(arr[0]);

	PrintArray(arr, sz);
	HeapSort(arr, sz);
	PrintArray(arr, sz);
}


void TestBubbleSort() //冒泡排序用例
{
	int arr[] = { 9,1,3,6,7,2,5,4,10,8 };
	int sz = sizeof(arr) / sizeof(arr[0]);

	PrintArray(arr, sz);
	BubbleSort(arr, sz);
	PrintArray(arr, sz);
}


void TestQuickSort()
{
	int arr[] = { 9,1,3,6,7,2,5,4,10,8 };
	int sz = sizeof(arr) / sizeof(arr[0]);

	PrintArray(arr, sz);
	QuickSort(arr,0,sz-1);
	PrintArray(arr, sz);
}


void TestMergeSort()
{
	int arr[] = { 9,1,3,6,7,2,5,4,10,8 };
	int sz = sizeof(arr) / sizeof(arr[0]);

	PrintArray(arr, sz);
	MergeSort(arr, sz);
	PrintArray(arr, sz);
}


TestMergeFileSort()
{
	MergeSortFile("Sort.txt");
}


TestCountSort()
{
	int arr[] = { 9,1,3,6,7,2,5,4,10,8 };
	int sz = sizeof(arr) / sizeof(arr[0]);

	PrintArray(arr, sz);
	CountSort(arr, sz);
	PrintArray(arr, sz);
}
void TestDiffernt()  //测试不同函数处理方式的效率
{
	srand(time(NULL));  //time中没有负数,仅用于测试
	const int N = 100000;
	int* a1 = (int*)malloc(sizeof(int*) * N);
	int* a2 = (int*)malloc(sizeof(int*) * N);
	int* a3 = (int*)malloc(sizeof(int*) * N);
	int* a4 = (int*)malloc(sizeof(int*) * N);
	int* a5 = (int*)malloc(sizeof(int*) * N);
	int* a6 = (int*)malloc(sizeof(int*) * N);
	int* a7 = (int*)malloc(sizeof(int*) * N);
	int* a8 = (int*)malloc(sizeof(int*) * N);
	for (int i = 0; i < N; i++)
	{
		a1[i] = rand();
		a2[i] = a1[i];
		a3[i] = a1[i];
		a4[i] = a1[i];
		a5[i] = a1[i];
		a6[i] = a1[i];
		a7[i] = a1[i];
		a8[i] = a1[i];
	}

	int begin1 = clock();
	InsertSort(a1, N);
	int end1 = clock();

	int begin2 = clock();
	ShellSort(a2, N);
	int end2 = clock();

	int begin3 = clock();
	SelectSort(a3, N);
	int end3 = clock();

	int begin4 = clock();
	HeapSort(a4, N);
	int end4 = clock();

	int begin5 = clock();
	BubbleSort(a5, N);
	int end5 = clock();

	int begin6 = clock();
	QuickSort(a6, 0,N);
	int end6 = clock();

	int begin7 = clock();
	MergeSort(a7, N);
	int end7 = clock();

	int begin8 = clock();
	CountSort(a8, N);
	int end8 = clock();

	printf("InsertSort time:%d\n", end1 - begin1); //5000
	printf("ShellSort time:%d\n", end2 - begin2);  //20
	printf("SelectSort time:%d\n", end3 - begin3); //5000
	printf("HeapSort time:%d\n", end4 - begin4);   //20
	printf("BubbleSort time:%d\n", end5 - begin5); //30000
	printf("QuickSort time:%d\n", end6 - begin6); //20
	printf("MergeSort time:%d\n", end7 - begin7);//20
	printf("CountSort time:%d\n", end8 - begin8);//1

	free(a1);
	free(a2);
	free(a3);
	free(a4);
	free(a5);
	free(a6);
	free(a7);
	free(a8);
}
int main()
{

	//TestInsertSort();
	//TestShellSort();

	//TestSelectSort();
	//TestHeapSort();

	//TestBubbleSort();
	//TestQuickSort();

	//TestMergeSort();
	//TestMergeFileSort();

	//TestCountSort();
	
	TestDiffernt();
	return 0;
}
```
