---
title: 学生管理系统
date: 2023-11-03 10:12:41
tags:
- C++线程
- CMake
- socket
- 套接字通信
- 小项目
- 线程异步
categories:
- Linux
cover: /pic/3.png
---

# MySQL建表(my_sql.h)

```SQL
create database if not exists 9tse default charset utf8mb4;

use 9tse;

create table stu_msg(
  id int primary key AUTO_INCREMENT COMMENT '主键',
  num varchar(25) NOT NULL unique COMMENT '学号',
  name varchar(10) NOT NULL COMMENT '姓名',
  gender char(1) NOT NULL COMMENT '性别',
  profession varchar(50) NOT NULL COMMENT '专业',
  class int NOT NULL COMMENT '班级',
  score float NOT NULL COMMENT '成绩'
)COMMENT '学生信息表';

insert into stu_msg(num,name,gender,profession,class,score) VALUES('xxxxxx','xxx','男','计算机','1','4.1');

insert into stu_msg(id,num,name,gender,profession,class,score) VALUES(1,'xxxxxxxx','name','男','计算机',1,4.4);

alter table stu_msg MODIFY score varchar(10) comment'成绩';
alter table stu_msg MODIFY class varchar(10) comment'成绩';

update stu_msg set id = 1 where id = 4;

delete from stu_msg where id = 1;

select * from stu_msg where id = 1;

select * from stu_msg;
```

---

# 客户端

## 头文件

### TcpSocket.h

```C++
#include<string>
#include<arpa/inet.h>
#include<sys/socket.h>
#include<unistd.h>
#include<iostream>
#include<stdio.h>
#include<string.h>

class TcpSocket
{
public:
	TcpSocket();
	TcpSocket(int socket);
	~TcpSocket();
	int Connect_Host(std::string ip, unsigned short port);
	int Send_Msg(std::string msg);
	std::string Recv_Msg();

private:
	int readn(char* msg, int size);
	int written(const char* msg, int size);

private:
	int m_fd; 
};
```

### client_ready.h

```C++
#include"TcpSocket.h"
#include <fstream>
#include <unordered_map>
#include <vector>

void Connect(TcpSocket& client) 
{
    int ret = client.Connect_Host("127.0.0.1", 10000);
    if (ret == -1)
        perror("client connect to host");
    return;
}


class Login {
public:
    std::string m_name;
    Login(const std::string& filename)
        : filename_(filename)
    {
        LoadDataFromFile();
    }

    bool Authenticate()
    {
        std::string username;
        std::cout << "please input your name: ";
        std::cin >> username;
        m_name = username;
        std::string password;
        std::cout << "please input your password: ";
        std::cin >> password;

        if (accounts_.find(username) != accounts_.end() && accounts_[username] == password)
        {
            std::cout << "Authentication successful for " << username << std::endl;
            return true;
        }
        else
        {
            std::cout << "Authentication failed for " << username << std::endl;
            return false;
        }
    }

    void AddAccount(const std::string& username, const std::string& password)
    {
        accounts_[username] = password;
        SaveDataToFile();
        std::cout << "Account added: " << username << std::endl;
    }

    bool DeleteAccount(const std::string& username)
    {
        auto it = accounts_.find(username);
        if (it != accounts_.end())
        {
            accounts_.erase(it);
            SaveDataToFile();
            std::cout << "Account deleted: " << username << std::endl;
            return true;
        }
        else
        {
            std::cout << "Account not found: " << username << std::endl;
            return false;
        }
    }

    bool ModifyPassword(const std::string& username, const std::string& newPassword)
    {
        auto it = accounts_.find(username);
        if (it != accounts_.end())
        {
            it->second = newPassword;
            SaveDataToFile();
            std::cout << "Password for account " << username << " modified." << std::endl;
            return true;
        }
        else
        {
            std::cout << "Account not found: " << username << std::endl;
            return false;
        }
    }

private:
    std::string filename_;
    std::unordered_map<std::string, std::string> accounts_;

    void LoadDataFromFile()  //��ȡ�ļ�����
    {
        std::ifstream file(filename_, std::ios::binary); //�����ƶ�ȡ
        if (file.is_open())
        {
            while (true)
            {
                std::string username, password;
                if (ReadStringFromBinaryFile(file, username) && ReadStringFromBinaryFile(file, password))
                {
                    accounts_[username] = password;
                }
                else
                {
                    break;
                }
            }
            file.close();
        }
    }

    void SaveDataToFile() {
        std::ofstream file(filename_, std::ios::binary);
        if (file.is_open())
        {
            for (const auto& entry : accounts_)
            {
                WriteStringToBinaryFile(file, entry.first);
                WriteStringToBinaryFile(file, entry.second);
            }
            file.close();
        }
    }

    bool ReadStringFromBinaryFile(std::ifstream& file, std::string& str)
    {
        uint32_t strLength = 0;
        if (file.read(reinterpret_cast<char*>(&strLength), sizeof(uint32_t)))
        {
            str.resize(strLength);
            if (file.read(&str[0], strLength))
            {
                return true;
            }
        }
        return false;
    }

    void WriteStringToBinaryFile(std::ofstream& file, const std::string& str)
    {
        uint32_t strLength = static_cast<uint32_t>(str.size());
        file.write(reinterpret_cast<const char*>(&strLength), sizeof(uint32_t));
        file.write(str.c_str(), strLength);
    }
};



enum class chose
{
    add,
    del,
    select,
    modify
};
int Chose_Function() 
{
    std::cout << "******************************************************" << std::endl;
    std::cout << "************    input num to use    ******************" << std::endl;
    std::cout << "*****    0.add    ************    1.delete    ********" << std::endl;
    std::cout << "*****    2.select ************    3.modify    ********" << std::endl;
    std::cout << "******************************************************" << std::endl;
    int input;
    std::cin >> input;
    return input;
}

void Add_Action(TcpSocket& client, Login& user) 
{
    if (user.m_name != "542213430101")
    {
        std::cout << "Apologize, you do not have the authority" << std::endl;
        return;
    }
    std::string sql = "INSERT INTO stu_msg (num, name, gender, profession, class, score) VALUES (";
    std::string input;
    std::string num;
    std::vector<std::string> inputValues;
    std::cout << "Enter the following information in order (num, name, gender, profession, class, score):" << std::endl;

    for (int i = 0; i < 6; i++)
    {
        std::cin >> input;
        inputValues.push_back(input);
        if (i == 0)
            num = input;
    }
    for (const std::string& value : inputValues)
        sql += "'" + value + "',";

    // Remove the trailing ", " 
    sql = sql.substr(0, sql.length() - 1);
    sql += ")";

    client.Send_Msg(sql);

    user.AddAccount(num, num);

}


void Del_Action(TcpSocket& client, Login& user) 
{
    if (user.m_name != "542213430101")
    {
        std::cout << "Apologize, you do not have the authority" << std::endl;
        return;
    }

    std::cout << "input the student number to delete: ";
    std::string num;
falg:
    std::cin >> num;
    if (num == "542213430101")
    {
        std::cout << "you can't delete yourself, try again" << std::endl;
        goto falg;
    }
    std::string sql("delete from stu_msg where num = ");
    client.Send_Msg(sql + num);

    user.DeleteAccount(num);
}

void Sel_Action(TcpSocket& client, Login& user)
{
    flag:
    std::cout << "select your find way" << std::endl
        << "1. select all" << std::endl
        << "2. select by more ways" << std::endl;
    int input;
    std::cin >> input;
    if(input != 1 && input != 2)
    {
        std::cout << "input error,try again" <<std::endl;
        goto flag;
    }

    std::string sql("select * from stu_msg ");
    if (input == 1)
    {
        client.Send_Msg(sql);
    }
    else
    {
        std::cout<<"chose your select way by ** write ** word"<<std::endl;
        std::string selectway;
        std::cout<<"num   name  gender  profession  class  score"<<std::endl;
        std::cin >> selectway;

        std::string message;
        std::cout << "input select message"<<std::endl;
        std::cin >> message;

        std::string sqlsend = sql + "where " + selectway + "= '" + message + "'";
        std::cout << sqlsend<<std::endl;
        client.Send_Msg(sqlsend);
    }
    //sleep(1);
    std::cout << client.Recv_Msg() << std::endl;
    return;
}


void Mod_Action(TcpSocket& client, Login& user)
{
    flag:
    std::cout << "which you want to modify :" << std::endl;
    std::cout << "account/table msg (1/0) :" << std::endl;
    int chose;
    std::cin >> chose;
    if(chose != 1 && chose != 2)
    {
        std::cout << "input error,try again" <<std::endl;
        goto flag;
    }

    if (chose) //�޸��˺���Ϣ
    {
        std::string newpassword;
        std::cout << "input your new password" << std::endl;
        std::cin >> newpassword;
        user.ModifyPassword(user.m_name, newpassword);
    }
    else //�޸ı����е���Ϣ
    {
        std::string rootchosenum;
        if (user.m_name == "542213430101")
        {
            std::cout << "chose num to change" << std::endl;
            std::cin >> rootchosenum;
        }
        std::cout << "input word to change message" << std::endl;
        std::cout << "num  name  gender  profession  class  score" << std::endl;
        std::string input;
        std::cin >> input;
        std::string sql("update stu_msg set ");
        input += "='";
        sql += input;
        std::cout << "input message to change" << std::endl;
        std::cin >> input;
        input += "' where num = ";
        if (user.m_name == "542213430101")
            input += rootchosenum;
        else
            input += user.m_name;
        sql += input;
        client.Send_Msg(sql);
    }
}
```

## 源文件
### TcpSocket.cpp

```C++
#include "TcpSocket.h"

TcpSocket::TcpSocket() //一般用于客户端,并通过这个文件描述符进行和服务器的连接
{
	m_fd = socket(AF_INET,SOCK_STREAM,0);
}

TcpSocket::TcpSocket(int socket) //一般用于服务器,直接用这个套接字通信
{
	m_fd = socket;
}

TcpSocket::~TcpSocket()
{
	if(m_fd > 0)
		close(m_fd);
}

int TcpSocket::Connect_Host(std::string ip, unsigned short port)
{
	sockaddr_in saddr;
	saddr.sin_family = AF_INET;
	saddr.sin_port = htons(port); //端口转换为网络字节序
	inet_pton(AF_INET, ip.data(), &saddr.sin_addr.s_addr); //ip地址转换为大端(网络字节序)
	int ret = connect(m_fd, (sockaddr*)&saddr, sizeof(saddr)); //连接,m_fd,ip port存放在saddr

	if (ret == -1)
	{
		perror("connect");
		return -1;
	}
	std::cout << "connect with server sucessfully" << std::endl;
	return ret; //返回文件描述符
}

int TcpSocket::written(const char* msg, int size) //作为发射端,前四个字节初始化后就不用管了
{
	int readplace = 0; //开始读取的位置
	int remain = size; //剩余读取的大小
	const char* buf = msg; //临时存放流,以便输出
	while(remain > 0) //只有还有剩余的大小就继续读
	{
		if((readplace = write(m_fd, buf, remain)) > 0) //将buf中内容输入m_fd,write返回值为写入的字节
		{
			remain -= readplace;
			buf += readplace;
		}
		else if(readplace == -1) //如果write写入错误就会返回-1
		{
			return -1;
		}
	}
	return 0;
}
int TcpSocket::Send_Msg(std::string msg)
{
	char* buf = new char[msg.size() + 4];
	int flagsize = htonl(msg.size());
	memcpy(buf,&flagsize,4); //前四位设置字符串大小
	memcpy(buf+4,msg.data(), msg.size()); //后面照常装字符串 erro没有发完

	int ret = written(buf, msg.size() + 4); //ret返回值为一个字符串的大小
	delete[] buf;
	return ret;
}



std::string TcpSocket::Recv_Msg()
{
	int len = 0;
	readn((char*)&len, 4); //读取传来的前四个字节数据,得到数据内容大小
	len = ntohl(len);
	std::cout << "thesize of recive msg is: " << len << std::endl;

	char* buf = new char[len + 1]; //+1存放换行
	int ret = readn(buf, len); //读取数据,放置buf内
	buf[len] = '\0';//手动添加结束符
	std::string returnstr(buf); //返回值
	delete[] buf;
	return returnstr;
}

int TcpSocket::readn(char* msg, int size)
{
	int readplace = 0;
	int remain = size; 
	char* buf = msg;
	while (remain > 0)
	{
		if ((readplace = read(m_fd, buf, size)) > 0)
		{
			remain -= readplace;
			buf += readplace;
		}
		else if (readplace == -1)
		{
			return -1;
		}
	}
	return 0;
}





```

### main_client.cpp

```C++
#include "client_ready.h"

int main()
{

	Login user("account.txt");
	//user.AddAccount("542213430101", "9tse");

	while (!user.Authenticate());


	TcpSocket client;
	Connect(client);	

	while (1)
	{
		switch (Chose_Function())
		{
		case (int)chose::add:
			Add_Action(client, user);
			break;
		case (int)chose::del:
			Del_Action(client, user);
			break;
		case (int)chose::select:
			Sel_Action(client, user);
			break;
		case (int)chose::modify:
			Mod_Action(client, user);
			break;
		default:
			std::cout<<"chose error"<<std::endl;
		}
	}


	return 0;
}
```

## CMakeList.txt

```CMake
cmake_minimum_required(VERSION 3.0)
project(test)
set(CMAKE_CXX_STANDARD 11)

include_directories(${CMAKE_CURRENT_SOURCE_DIR}/include) #头文件
set(EXECUTABLE_OUTPUT_PATH ${CMAKE_CURRENT_SOURCE_DIR}) #生成路径
file(GLOB SRC_LIST ${CMAKE_CURRENT_SOURCE_DIR}/src/*.cpp) #源文件
add_executable(client_app  ${SRC_LIST}) #编译
target_link_libraries(client_app) 
```

---

# 服务器

## 头文件

### TcpServer.h

```C++
#include "TcpSocket.h"

class TcpServer
{
public:
	TcpServer();
	~TcpServer();
	int Set_Listen(unsigned short port);
	TcpSocket* Accept_Connect(sockaddr_in* addr = nullptr);

private:
	int m_fd;
};
```

### TcpSocket.h

```C++
#include<string>
#include<arpa/inet.h>
#include<sys/socket.h>
#include<unistd.h>
#include<iostream>
#include<stdio.h>
#include<string.h>

class TcpSocket
{
public:
	TcpSocket();
	TcpSocket(int socket);
	~TcpSocket();
	int Connect_Host(std::string ip, unsigned short port);
	int Send_Msg(std::string msg);
	std::string Recv_Msg();

private:
	int readn(char* msg, int size);
	int written(const char* msg, int size);

private:
	int m_fd; //通信的套接字
};
```

### my_sql.h

```C++
#include <string>
#include <mysql.h>
#include <iostream>
#include <iomanip> // 添加头文件以使用 setw
#include <sstream> // 添加头文件以使用 std::ostringstream

MYSQL* Connect_MySQL()
{
    MYSQL* mysql = mysql_init(nullptr); //启动MySQL服务,初始化
    if (mysql == nullptr)
    {
        perror("mysql_init");
        return nullptr;
    }

    //连接MySQL
    mysql = mysql_real_connect(mysql, "192.168.200.131", "root", "9tse",
        "9tse", 0, NULL, 0);
    if (mysql == nullptr)
    {
        perror("mysql connect");
        return nullptr;
    }

    mysql_set_character_set(mysql, "utf8"); //设置为utf8编码

    std::cout << "Mysql connect successfully" << std::endl;
    return mysql;
}


void Act_Sql(MYSQL* mysql, const std::string sql, std::ostringstream& result)
{
    const char* char_sql = sql.c_str();
    int ret = mysql_query(mysql, char_sql); // 执行SQL
    if (ret != 0) {
        perror("action mysql");
        return;
    }

    MYSQL_RES* res = mysql_store_result(mysql); // 取出结果集
    if (res == nullptr) {
        perror("mysql_store_result");
        return;
    }

    int num = mysql_num_fields(res); // 结果集列数
    MYSQL_FIELD* fields = mysql_fetch_fields(res); // 所有列名字

    // 打印列名并设置字段宽度和左对齐
    result << std::left;
    for (int i = 0; i < num; ++i) {
        result << std::setw(12) << fields[i].name << "\t";
    }
    result << "\n";

    MYSQL_ROW row;
    while ((row = mysql_fetch_row(res)) != NULL) // 遍历行
    {
        // 打印每一列并设置字段宽度和左对齐
        for (int i = 0; i < num; ++i) {
            result << std::setw(12) << row[i] << "\t";
        }
        result << "\n";
    }

    mysql_free_result(res); // 释放资源 - 结果集
}
```


### thread_pool.h

```C++
#pragma once
#include<queue>
#include<thread>
#include<condition_variable>
#include<atomic>
#include<stdexcept>
#include<future>
#include<vector>
#include<functional>
#include<mutex>

namespace std
{
	#define THREADPOOL_MAX_NUM 16
	class threadpool
	{
		unsigned short _initsize; //初始线程池大小
		using Task = function<void()>; //任务函数
		vector<thread> _pool; //线程容器
		queue<Task> _tasks; //任务队列
		mutex _lock; //增加任务队列的锁
		mutex _lockGrow; //增加线程的锁
		condition_variable _task_cv; //
		atomic<bool> _run{true}; //程序是否运行
		atomic<int> _spa_trd_num{0}; //当前线程数量

	public:
		inline threadpool(unsigned short size = 4)
		{
			_initsize = size;
			Add_Thread(size);
		}
		inline ~threadpool()
		{
			_run = false;
			_task_cv.notify_all();
			for (thread& thread : _pool)
			{
				if (thread.joinable())
					thread.join();
			}
		}

		//把任务提交给线程池中的子线程
		template<typename F, typename... Args> //F为函数类型,args为参数类型
		auto commit(F&& f, Args&& ...args) -> future<decltype(f(args...))> //auto 推断为f函数调用后的返回值类型 
		{
			//future的作用就是存储任意类型的值
			if (!_run)
				throw runtime_error{"commit auto stop"};
			using RetType = decltype(f(args...)); //RetType就是当前函数返回值类型
			//创建一个名字为task,指向一个包装任务器类型的,以绑定器作为构造的智能指针
			//绑定器用于绑定函数地址和参数(占位符)
			auto task = make_shared<packaged_task<RetType()>>(bind(forward<F>(f), forward<Args>(args)...));
			future<RetType> future = task->get_future(); //取函数返回值
			{
				lock_guard<mutex> lock{_lock};
				//任务队列加入一个function<void()>的函数,这个函数是一个lambda表达式,解引用之后,调用包装器(即task函数)
				_tasks.emplace([task]() {(*task)(); }); 
			}
			if (_spa_trd_num < 1 && _pool.size() < THREADPOOL_MAX_NUM) //将任务加入任务队列后,如果当前没有子线程
				Add_Thread(1); //加个子线程
			_task_cv.notify_one(); //唤醒条件变量的堵塞
			return future; //返回当前函数的返回值
		}

		template<typename F>
		void commit2(F&& f) //无参数类型函数
		{
			if (!_run)
				return;
			{
				lock_guard<mutex> lock{_lock};
				_tasks.emplace(forward<F>(f)); //任务队列直接加入函数
			}
			if (_spa_trd_num < 1 && _pool.size() < THREADPOOL_MAX_NUM)
				Add_Thread(1);
			_task_cv.notify_one();
		}

		int idlCount() { return _spa_trd_num; }
		int thrCount() { return _pool.size(); }

	private:
		void Add_Thread(unsigned short size) //加线程
		{
			if (!_run)
				throw runtime_error{"Add_Thread stop"};
			unique_lock<mutex> lockgrow{_lockGrow}; //锁整个线程创建的函数
			for (; _pool.size() < THREADPOOL_MAX_NUM && size > 0; --size) //如果还可以加入线程
			{
				_pool.emplace_back([this] //线程容器里面加入lambda表达式,lambda表达式即子线程
					{
						while (true)
						{
							Task task; //每个线程不断取任务队列中的任务
							{
								unique_lock<mutex> lock{_lock};
								//wait 返回false阻塞(不跑了或者任务队列不为空,可以干活了就解除阻塞)
								_task_cv.wait(lock, [this] {return !_run || !_tasks.empty(); });
								//如果任务队列为空并且不跑了,那么这个线程就直接结束
								if (!_run && _tasks.empty()) 
									return;
								_spa_trd_num--; //当前线程数--(因为要执行任务了)
								task = move(_tasks.front()); //移动拷贝获取任务
								_tasks.pop(); //任务队列删除一个
							}
							task(); //执行任务
							//如果当前线程数大于初始构造值,线程也被杀死,即当前线程数不再加了
							if (_spa_trd_num > 0 && _pool.size() > _initsize) 
								return;
							{
								unique_lock<mutex> lock{_lock};
								_spa_trd_num++;//执行玩任务栏了,当前线程数++
							}
						}
					});
				{
					unique_lock<mutex> lock{_lock};
					_spa_trd_num++; //即将创建下一个子线程,提前线程数++
				}
			}
		}
	};
}
```

## 源文件

### TcpSocket.cpp

```C++
#include "TcpSocket.h"

TcpSocket::TcpSocket() //一般用于客户端,并通过这个文件描述符进行和服务器的连接
{
	m_fd = socket(AF_INET,SOCK_STREAM,0);
}

TcpSocket::TcpSocket(int socket) //一般用于服务器,直接用这个套接字通信
{
	m_fd = socket;
}

TcpSocket::~TcpSocket()
{
	if(m_fd > 0)
		close(m_fd);
}

int TcpSocket::Connect_Host(std::string ip, unsigned short port)
{
	sockaddr_in saddr;
	saddr.sin_family = AF_INET;
	saddr.sin_port = htons(port); //端口转换为网络字节序
	inet_pton(AF_INET, ip.data(), &saddr.sin_addr.s_addr); //ip地址转换为大端(网络字节序)
	int ret = connect(m_fd, (sockaddr*)&saddr, sizeof(saddr)); //连接,m_fd,ip port存放在saddr

	if (ret == -1)
	{
		perror("connect");
		return -1;
	}
	std::cout << "connect with server sucessfully" << std::endl;
	return ret; //返回文件描述符
}

int TcpSocket::written(const char* msg, int size) //作为发射端,前四个字节初始化后就不用管了
{
	int readplace = 0; //开始读取的位置
	int remain = size; //剩余读取的大小
	const char* buf = msg; //临时存放流,以便输出
	while(remain > 0) //只有还有剩余的大小就继续读
	{
		if((readplace = write(m_fd, buf, remain)) > 0) //将buf中内容输入m_fd,write返回值为写入的字节
		{
			remain -= readplace;
			buf += readplace;
		}
		else if(readplace == -1) //如果write写入错误就会返回-1
		{
			return -1;
		}
	}
	return 0;
}
int TcpSocket::Send_Msg(std::string msg)
{
	char* buf = new char[msg.size() + 4];
	int flagsize = htonl(msg.size());
	memcpy(buf,&flagsize,4); //前四位设置字符串大小
	memcpy(buf+4,msg.data(), msg.size()); //后面照常装字符串 erro没有发完

	int ret = written(buf, msg.size() + 4); //ret返回值为一个字符串的大小
	delete[] buf;
	return ret;
}



std::string TcpSocket::Recv_Msg()
{
	int len = 0;
	readn((char*)&len, 4); //读取传来的前四个字节数据,得到数据内容大小
	len = ntohl(len);
	std::cout << "thesize of recive msg is: " << len << std::endl;

	char* buf = new char[len + 1]; //+1存放换行
	int ret = readn(buf, len); //读取数据,放置buf内
	buf[len] = '\0';//手动添加结束符
	std::string returnstr(buf); //返回值
	delete[] buf;
	return returnstr;
}

int TcpSocket::readn(char* msg, int size)
{
	int readplace = 0;
	int remain = size; 
	char* buf = msg;
	while (remain > 0)
	{
		if ((readplace = read(m_fd, buf, size)) > 0)
		{
			remain -= readplace;
			buf += readplace;
		}
		else if (readplace == -1)
		{
			return -1;
		}
	}
	return 0;
}
```

### TcpServer.cpp

```C++
#include "TcpServer.h"

TcpServer::TcpServer()
{
	m_fd = socket(AF_INET, SOCK_STREAM, 0);
}

TcpServer::~TcpServer()
{
	close(m_fd);
}

int TcpServer::Set_Listen(unsigned short port)
{
	//绑定ip 和 port
	sockaddr_in saddr;
	saddr.sin_family = AF_INET;
	saddr.sin_port = htons(port); //大端
	saddr.sin_addr.s_addr = INADDR_ANY; //大端
	int ret = bind(m_fd, (sockaddr*)&saddr, sizeof(saddr));

	if (ret == -1)
	{
		perror("bind");
		return -1;
	}
	std::cout << "sucessfully bind to ip: " << inet_ntoa(saddr.sin_addr) << std::endl
		<< "port: " << port << std::endl;

	//设置监听
	ret = listen(m_fd, 128);
	{
		perror("listen");
		return -1;
	}
	std::cout << "set listen sucessfully" << std::endl;
	return 0;
}

TcpSocket* TcpServer::Accept_Connect(sockaddr_in* addr)
{
	if (addr == nullptr)
	{
		perror("accept(addr) is nullptr");
		return nullptr;
	}

	socklen_t addrlen = sizeof(sockaddr_in);
	int cfd = accept(m_fd, (sockaddr*)addr, &addrlen);
	if (cfd == -1)
	{
		perror("accept");
		return nullptr;
	}
	std::cout << "connect with client sucessfully" << std::endl;
	return new TcpSocket(cfd);
}
```

### main_server.cpp

```C++
#include "my_sql.h"
#include "TcpServer.h"
#include "thread_pool.h"

struct SocketPag {
	TcpSocket* msg_tcp;
	TcpServer* listen_tcp;
	sockaddr_in addr;
};

void Working(void* args, MYSQL* mysql) //线程池任务函数
{
	SocketPag* pkg = static_cast<SocketPag*>(args);
	//打印客户端基本信息
	char ip[32];
	std::cout << "client ip: " << inet_ntop(AF_INET, &pkg->addr.sin_addr.s_addr, ip, sizeof(ip)) << std::endl;
	std::cout << "client port: " << ntohs(pkg->addr.sin_port) << std::endl;

	//获取转换过来的Sql指令
	while (1)
	{
		std::ostringstream result;
		std::string sql = pkg->msg_tcp->Recv_Msg(); //先确定是不是一波一波的,不是
		if (!sql.empty())
		{
			Act_Sql(mysql, sql, result); //执行sql语句
			pkg->msg_tcp->Send_Msg(result.str()); //将sql执行结果发送给client
		}
		else
		{
			break;
		}
		result.clear();
	}

	delete pkg->listen_tcp;
	delete pkg;
	return;
}

int main()
{
	MYSQL* mysql = Connect_MySQL(); //连接MySQL

	TcpServer server;
	server.Set_Listen(10000); //绑定并且设置监听

	std::threadpool threadpool(32); //实例化线程池

	while (true)
	{
		SocketPag* pag = new SocketPag();
		//将建立连接后的信息传递给pag->addr,在以此参数构造Tcpsocket
		TcpSocket* Msgret = server.Accept_Connect(&pag->addr);
		if (Msgret == nullptr) //如果为空有两种情况
		{
			perror("main accept");
			continue;
		}
		pag->listen_tcp = &server; //将即将进入子线程的任务包完整一下
		pag->msg_tcp = Msgret; //而客户端传来的信息就在Msgret中

		std::future<void> work = threadpool.commit(Working, pag, mysql);
	}
}
```

## CMakeList.txt

```CMake
cmake_minimum_required(VERSION 3.0)
project(test)
set(CMAKE_CXX_STANDARD 11)

include_directories(${CMAKE_CURRENT_SOURCE_DIR}/include) #头文件
include_directories(/usr/include/mysql)
set(EXECUTABLE_OUTPUT_PATH ${CMAKE_CURRENT_SOURCE_DIR})

file(GLOB SRC_LIST ${CMAKE_CURRENT_SOURCE_DIR}/src/*.cpp)  #源文件
add_executable(server_app  ${SRC_LIST})  #编译
target_link_libraries(server_app mysqlclient pthread) 
```
