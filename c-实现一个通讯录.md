---
title: 通讯录
date: 2023-10-21 21:01:11
tags: 
- 小项目
categories: 
- C语言
cover: /pic/1.png
---


# 一、项目头文件(project.h)

```c
#define _CRT_SECURE_NO_WARNINGS
#include<stdio.h>
#include<stdlib.h>
#include<malloc.h>
#include<string.h>
#include<assert.h>




enum inputnum //输入数字
{
	Exit,
	Add,
	Del,
	Query,
	Modify,
	Sort,
	Display,
	Reset
};

enum MAX            //字符串的最大大小
{
	Name_MAX = 20,
	Sex_MAX = 5,
	Adress_MAX = 30,
	Tel_MAX = 20,
};


typedef struct Peopleinfo   //一个人的信息
{
	char name[Name_MAX];
	char sex[Sex_MAX];
	int age;
	char tel[Tel_MAX];
	char adress[Adress_MAX];
}Ppinfo;

typedef struct Connect      //信息组
{
	Ppinfo* data;
	int size;
	int capacity;
}Con;

void menu();               //菜单  end

void Pp_Init(Con* con);    //初始化  end

void Pp_Add(Con* con);     //增加信息  end


void Pp_Del(Con* con);     //删除信息
void Pp_Delask(Con* con, int jud, int(*(*Cmp)[5])(Con*, const char*, int));   //删除信息中的函数分支，用于判断多个重复查找对象的删除


void Pp_Query(Con* con);   //查找
void print_goal(Con* con, int jud, int(*(*Cmp)[5])(Con*, const char*, int));  //通过选择方式打印出需要查找的内容函数


void Pp_Modify(Con* con);  //更改
void Pp_Modifyask(Con* con, int jud, int(*(*Cmp)[5])(Con*, const char*, int)); //更改选择函数分支


void Pp_Sort(Con* con);    //排列

void Pp_Display(Con* con); //展现出通讯录

void Pp_Reset(Con* con);   //销毁通讯录

void check_expand(Con* con);     //增加通信录容量  end

void  Pp_Save(Con* con);   //保存通讯录至硬盘      

void free_alloc(Con* con);      //释放动态内存 
```


---

# 二、项目主干部分(test.c)

```c
#include "project.h"

int main()
{
	int input = 0;
	Con con = { 0 };

	Pp_Init(&con);    //初始化结构体

	do
	{
		menu();//打印菜单
		scanf("%d", &input);
		switch (input)
		{
		case Add:
			Pp_Add(&con);  
			break;

		case Del:
			Pp_Del(&con); 
			break;

		case Query:
			Pp_Query(&con); 
			break;

		case Modify:
			Pp_Modify(&con); 
			break;

		case Sort:
			Pp_Sort(&con); 
			break;

		case Reset:
			Pp_Reset(&con); 
			break;

		case Display:
			Pp_Display(&con);
			break;

		case Exit:
			printf("正在存储信息中...\n");
			Pp_Save(&con); 
			printf("存储成功\n");
			break;
		default:
			printf("输入格式错误，请再次输入\n");
			break;
		}
	} while (input);

	free_alloc(&con);
	printf("退出成功\n");
	return 0;
}
```

---

# 三、函数实现部分(contact.c)

```c
#include "project.h"
int judgee = 0;

void menu() //菜单  end
{
	printf("     ---------------------------------------------- \n");
	printf("    |       请根据您所需要的功能选择数字           |\n");
	printf("     ---------------------------------------------- \n");
	printf("    |      0:Exit   |    1:Add     |  2:Delete     |\n");
	printf("     ---------------------------------------------- \n");
	printf("    |      3:Query  |    4:Modify  |  5:Sort       |\n");
	printf("     ---------------------------------------------- \n");
	printf("    |           6:Display  |   7:Reset             |\n");
	printf("     ---------------------------------------------- \n");
}

void check_expand(Con* con)   //扩展空间为原来的二倍   
{
	if (con->size == con->capacity)
	{
		Ppinfo* str = (Ppinfo*)realloc(con->data, (con->capacity * 2) * (sizeof(Ppinfo)));
		if (str == NULL)
		{
			perror("expand");
			exit(-1);
		}
		con->data = str;
		con->capacity *= 2;
		printf("扩容成功\n");
	}

}


void Pp_Add(Con* con) //增加个人信息   end
{
	//判断内存是否够用
	check_expand(con);

	printf("请按,姓名,性别,年龄,电话,地址,的顺序输入信息\n");
	scanf("%s %s %d %s %s", &con->data[con->size].name, &con->data[con->size].sex, &con->data[con->size].age, &con->data[con->size].tel, &con->data[con->size].adress);
	con->size++;
	printf("输入成功...\n");
}

void File_load(Con* con)  //初始化时加载之前保存文件至当前通讯录  end
{
	FILE* pf = fopen("contact.dat", "r");
	if (pf == NULL)
	{
		perror("Load fopen");
		exit(-1);
	}
	Ppinfo tmp = { 0 };
	while (fread(&tmp, sizeof(Ppinfo), 1, pf))
	{
		check_expand(con);
		con->data[con->size] = tmp;
		con->size++;
	}
	fclose(pf);
	pf = NULL;
}

void Pp_Init(Con* con)  //初始化  end
{
	assert(con);
	con->size = 0;
	con->capacity = 8;
	con->data = (Ppinfo*)malloc(con->capacity * (sizeof(Ppinfo)));

	File_load(con);
}

void free_alloc(Con* con)  //释放动态内存分配过的空间  end
{
	free(con->data);
	con->data = NULL;
}

typedef int (*Cmp)(Con*, const char*, int); //声明一个函数指针类型 int为这个函数指针所指向函数的返回类型

int Cmp_name(Con* con, const char* str, int i)
{
	return strcmp(str, con->data[i].name);
}
int Cmp_sex(Con* con, const char* str, int i)
{
	return strcmp(str, con->data[i].sex);
}
int Cmp_age(Con* con, const char* str, int i)
{
	return atoi(str) - con->data[i].age;
}
int Cmp_tel(Con* con, const char* str, int i)
{
	return strcmp(str, con->data[i].tel);
}
int Cmp_adress(Con* con, const char* str, int i)
{
	return strcmp(str, con->data[i].adress);
}

//

void Pp_Delask(Con* con, int jud, Cmp* ptr)
{

	char str[20] = { 0 };
	printf("请输入要删除联系人的信息\n");
	scanf("%s", &str);
	int i = 0;



	for (i = 0; i < con->size; i++)
	{
		if (ptr[jud - 1](con, str, i) == 0)   //函数指针数组的调用
		{
			printf("%s %s %d %s %s\n", con->data[i].name, con->data[i].sex, con->data[i].age, con->data[i].tel, con->data[i].adress);
			printf("该联系人是否为您要删除的对象(Y/N)\n");
			int judge = 0;
		flag:
			while (getchar() != '\n');


			judge = getchar();
			switch (judge)
			{
			case 'Y':
				for (; i < con->size - 1; i++)
				{
					con->data[i] = con->data[i + 1];
				}
				con->size--;
				printf("删除成功\n");
				return;
			case 'N':
				break;
			default:
				printf("请输入Y或N分别表示是或不是\n");
				goto flag;

			}

		}
	}
	printf("查找结束，您所要删除的对象不存在\n");
}



//void Pp_Del(Con* con)  //指定性删除信息   end
//{
//	judgee = 0;
//	int(*Cmp[5])(Con*, const char*, int) = { Cmp_name,Cmp_sex ,Cmp_age,Cmp_tel,Cmp_adress }; //创建函数指针数组
//
//	printf("请输入数字选择查找要删除联系人的方式\n");
//	printf("1：姓名 2：性别 3：年龄 4：电话 5：地址\n");
//flag:
//	scanf("%d", &judgee);
//	if (judgee == 1 || judgee == 2 || judgee == 3 || judgee == 4 || judgee == 5)
//	{
//		Pp_Delask(con, judgee, &Cmp); //第三个参数为函数指针数组的地址
//	}
//	else
//	{
//		printf("输入格式错误请重新输入:\n");
//		goto flag;
//	}
//
//}

void Pp_Del(Con* con)  //指定性删除信息   end
{
	judgee = 0;
	Cmp ptr[5] = {Cmp_name,Cmp_sex ,Cmp_age,Cmp_tel,Cmp_adress}; //创建函数指针数组

	printf("请输入数字选择查找要删除联系人的方式\n");
	printf("1：姓名 2：性别 3：年龄 4：电话 5：地址\n");
flag:
	scanf("%d", &judgee);
	if (judgee == 1 || judgee == 2 || judgee == 3 || judgee == 4 || judgee == 5)
	{
		Pp_Delask(con, judgee, &ptr); //第三个参数为函数指针数组的地址
	}
	else
	{
		printf("输入格式错误请重新输入:\n");
		goto flag;
	}

}



void print_goal(Con* con, int jud, int(*(*Cmp)[5])(Con*, const char*, int))  //查找选择后开始选择打印
{
	char str[20] = { 0 };
	printf("请输入要查找联系人的信息\n");
	scanf("%s", &str);
	int i = 0;

	int exist = 0;

	for (i = 0; i < con->size; i++)
	{
		if (Cmp[0][jud - 1](con, str, i) == 0)   //替换函数
		{
			exist = 1;
			printf("%s %s %d %s %s\n", con->data[i].name, con->data[i].sex, con->data[i].age, con->data[i].tel, con->data[i].adress);
		}
	}
	if (exist)
	{
		printf("以上是为您查找到的数据\n");
	}
	else
	{
		printf("您所要查找的对象不存在\n");
	}
}

void Pp_Query(Con* con)    //指定行查找信息  wait
{
	judgee = 0;
	printf("请输入数字选择查找联系人的方式\n");
	printf("1：姓名 2：性别 3：年龄 4：电话 5：地址\n");
	int(*Cmp[5])(Con*, const char*, int) = { Cmp_name,Cmp_sex ,Cmp_age,Cmp_tel,Cmp_adress }; //创建函数指针数组
flag:
	scanf("%d", &judgee);
	if (judgee == 1 || judgee == 2 || judgee == 3 || judgee == 4 || judgee == 5)
	{
		print_goal(con, judgee, &Cmp);   //
	}
	else
	{
		printf("输入格式错误请重新输入:\n");
		goto flag;
	}
}

void Pp_Display(Con* con)  //展现通讯录  end
{
	int i = 0;
	printf(" -------------------------------------------------------------------------------\n");
	printf("%-15s\t%-5s\t%-5s\t%-12s\t%-30s\n", "姓名", "性别", "年龄", "电话", "地址");
	for (i = 0; i < con->size; i++)
	{
		printf("%-15s\t%-5s\t%-5d\t%-12s\t%-30s\n", con->data[i].name, con->data[i].sex, con->data[i].age, con->data[i].tel, con->data[i].adress);
	}
	printf(" -------------------------------------------------------------------------------\n");
}


void Pp_Modifyask(Con* con, int jud, int(*(*Cmp)[5])(Con*, const char*, int))
{
	char str[20] = { 0 };
	printf("请输入要查找联系人的信息\n");
	scanf("%s", &str);
	int i = 0;
	


	for (i = 0; i < con->size; i++)
	{
		if (Cmp[0][jud - 1](con, str, i) == 0)   //函数指针数组
		{
			printf("%s %s %d %s %s\n", con->data[i].name, con->data[i].sex, con->data[i].age, con->data[i].tel, con->data[i].adress);
			printf("该联系人是否为您要修改的对象(Y/N)\n");
			int judge = 0;
		flag2:
			while (getchar() != '\n');
			
			judge = getchar();
			switch (judge)
			{
			case 'Y':
				printf("选择你要修改的信息所对应的数字\n");
				printf("1:姓名  2：性别  3：年龄  4：电话  5：地址 6：全部\n");
				int jjud = 0;
			flag1:
				while (getchar() != '\n');
				jjud = getchar();
				switch (jjud)
				{
				case '1':
					printf("请输入修改后的姓名");
					scanf("%s", con->data[i].name);
					printf("修改姓名成功\n");
					return;
				case '2':
					printf("请输入修改后的性别");
					scanf("%s", con->data[i].sex);
					printf("修改性别成功\n");
					return;
				case '3':
					printf("请输入修改后的年龄");
					scanf("%d", &con->data[i].age);
					printf("修改年龄成功\n");
					return;
				case '4':
					printf("请输入修改后的电话");
					scanf("%s", con->data[i].tel);
					printf("修改电话成功\n");
					return;
				case '5':
					printf("请输入修改后的地址");
					scanf("%s", con->data[i].adress);
					printf("修改地址成功\n");
					return;
				case '6':
					printf("请输入修改后的信息");
					scanf("%s", con->data[i].name);
					scanf("%s", con->data[i].sex);
					scanf("%d", &con->data[i].age);
					scanf("%s", con->data[i].tel);
					scanf("%s", con->data[i].adress);
					printf("修改信息成功\n");
					return;
				default:
					printf("输入格式错误，请再次输入");
					goto flag1;
				}
			case 'N':
				break;
			default:
				printf("请输入Y或N分别表示是或不是\n");
				goto flag2;

			}

		}
	}
	printf("查找结束，您所要修改的对象不存在\n");
}

void Pp_Modify(Con* con) //更改通讯录内的内容
{
	judgee = 0;
	int(*Cmp[5])(Con*, const char*, int) = { Cmp_name,Cmp_sex ,Cmp_age,Cmp_tel,Cmp_adress }; //创建函数指针数组

	printf("请输入数字选择查找要更改联系人的方式\n");
	printf("1：姓名 2：性别 3：年龄 4：电话 5：地址\n");
flag:
	scanf("%d", &judgee);
	if (judgee == 1 || judgee == 2 || judgee == 3 || judgee == 4 || judgee == 5)
	{
		Pp_Modifyask(con, judgee, &Cmp);
	}
	else
	{
		printf("输入格式错误请重新输入:\n");
		goto flag;
	}

}


int Strcmp_byname(const void* e1, const void* e2)
{
	return strcmp(((Ppinfo*)e1)->name, ((Ppinfo*)e2)->name);
}
int Strcmp_bysex(const void* e1, const void* e2)
{
	return strcmp(((Ppinfo*)e1)->sex, ((Ppinfo*)e2)->sex);
}
int Strcmp_byage(const void* e1, const void* e2)
{
	return ((Ppinfo*)e1)->age - ((Ppinfo*)e2)->age;
}
int Strcmp_bytel(const void* e1, const void* e2)
{
	return strcmp(((Ppinfo*)e1)->tel, ((Ppinfo*)e2)->tel);
}
int Strcmp_byadress(const void* e1, const void* e2)
{
	return strcmp(((Ppinfo*)e1)->adress, ((Ppinfo*)e2)->adress);
}

void Pp_Sort(Con* con)  //排列通讯录内的内容
{
	int jud = 0;
	printf("请输入数字选择排列方式\n");
	printf("1：姓名 2：性别 3：年龄 4：电话 5：地址\n");
flag:
	scanf("%d", &jud);
	switch (jud)
	{
	case 1:
		qsort(con->data, con->size, sizeof(con->data[0]), Strcmp_byname);
		
		break;
	case 2:
		qsort(con->data, con->size, sizeof(con->data[0]), Strcmp_bysex);
		break;
	case 3:
		qsort(con->data, con->size, sizeof(con->data[0]), Strcmp_byage);
		break;
	case 4:
		qsort(con->data, con->size, sizeof(con->data[0]), Strcmp_bytel);
		break;
	case 5:
		qsort(con->data, con->size, sizeof(con->data[0]), Strcmp_byadress);
		break;
	default:
		printf("选择格式错误，请重新输入");
		goto flag;
	}
	printf("已排序成功\n");
}


void Pp_Reset(Con* con)  //初始化通讯录所有内容
{
	printf("确定要初始化所有内容么？(Y/N)");
	int judge = 0;
flag:
	getchar();
	judge = getchar();
	switch (judge)
	{
	case 'Y':
		con->size = 0;
		Pp_Save(con);
		printf("初始化成功\n");
		return;
	case 'N':
		break;
	default:
		printf("选择格式错误，请重新输入");
		goto flag;
	}
	printf("已取消初始化\n");
	return;
}


void  Pp_Save(Con* con)  //保存当前通讯录至文件内
{
	FILE* pf = fopen("contact.dat", "w");
	if (pf == NULL)
	{
		perror("Save fopen");
		exit(-1);
	}
	int i = 0;
	for (i = 0; i < con->size; i++)
	{
		fwrite(con->data + i, sizeof(Ppinfo), 1, pf);
	}
	fclose(pf);
	pf = NULL;

}
```

---