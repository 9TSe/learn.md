---
title: tips
date: 2023-10-22 09:52:06
tags:
categories:
- 线程
cover: /pic/4.png
---

# 1. 线程介绍

线程是轻量级的进程（LWP：light weight process），在Linux环境下线程的本质仍是进程。
在计算机上运行的程序是一组指令及指令参数的组合，指令按照既定的逻辑控制计算机运行。
操作系统会以进程为单位，分配系统资源，可以这样理解
进程是`资源分配的最小单位，线程是操作系统调度执行的最小单位。`

从概念上来说线程和进程的区别:

 - 进程有自己独立的地址空间, 多个线程共用同一个地址空间
 
 	- 线程更节省系统资源, 效率不仅可以保持, 而且能更高
 	-  在一个地址空间中多个线程独享: 每个线程都有属于自己的栈区, 寄存器(内核中管理的)
 	-  在一个地址空间中多个线程共享: 代码段, 堆区, 全局数据区, 打开的文件(文件描述符表)
 
 - 线程是程序的最小执行单位, 进程是操作系统中最小的资源分配单位
 	-  每个进程对应一个虚拟地址空间，一个进程只能抢一个CPU时间片
 	-  一个地址空间中可以划分出多个线程, 能在有效的资源基础上, 抢更多的CPU时间片

![在这里插入图片描述](/img/8.16.png)

 - CPU的调度和切换: 线程的上下文切换比进程要快的多
上下文切换：进程/线程分时复用CPU时间片，在切换之前会将上一个任务的状态进行保存, 下次切换回这个任务的时候, 加载这个状态继续运行，`任务从保存到再次加载`这个过程就是一次上下文切换。
 - 线程更加廉价, 启动速度更快, 退出也快, 对系统资源的冲击小。

**在处理多任务程序的时候使用多线程比使用多进程要更有优势，但是线程并不是越多越好**

 1. 文件IO操作：文件IO对CPU是使用率不高, 因此可以分时复用CPU时间片
	线程的个数 = 2 * CPU核心数 (效率最高)
 2. 处理复杂的算法(主要是CPU进行运算, 压力大)
 线程的个数 = CPU的核心数 (效率最高)


---

# 2. 创建线程

## 2.1 线程函数

> 每一个线程都有一个唯一的线程ID，ID类型为`pthread_t`
> 这个ID是一个无符号长整形(`unsigned long`)

如果想要得到当前线程的线程ID，可以调用如下函数：

```c
pthread_t pthread_self(void);	// 返回当前线程的线程ID
```

在一个进程中调用线程创建函数，就可得到一个子线程
和进程不同，需要给每一个创建出的线程指定一个处理函数，否则这个线程无法工作。

```c
#include <pthread.h>
int pthread_create(pthread_t *thread,
 					const pthread_attr_t *attr,
                    void *(*start_routine) (void *), 
                    void *arg);
// Compile and link with -pthread
//线程库的名字叫pthread, 全名: libpthread.so libptread.a
```

介绍一下pthread_create函数的

 - 参数
`thread`: 传出参数,无符号长整形,线程创建成功, 会将线程ID写入到这个指针指向的内存中
`attr`: 线程的属性, 一般情况下使用默认属性即可, 写NULL
`start_routine`: 函数指针，创建出的子线程的处理动作，该函数在子线程中执行。
`arg`: 作为实参传递到 start_routine 指针指向的函数内部
 - 返回值：线程创建成功返回0，创建失败返回对应的错误号

---

## 2.2 创建线程

![在这里插入图片描述](/img/8.17.png)

```c
#include <stdio.h>
#include <stdlib.h>
#include <pthread.h>
#include <unistd.h>    

//// 子线程的处理代码
void* Work(void* args)
{
	printf("子线程id: %ld\n",pthread_self());
    for(int i = 0; i < 5 ; ++i)
    {
        printf("child == i : %d\n",i);
     }
    return NULL;
}
int main()
{
    //1.创建子线程
    pthread_t tid;
    pthread_create(&tid,NULL,Work,NULL);

    printf("子线程创建成功,线程id : %ld\n",tid);
    
    //2.子线程不会执行下面的代码,由主线程执行
    printf("主线程id : %ld\n", pthread_self());
    for(int i = 0; i < 3 ; ++i)
    {
        printf("i == %d\n",i);
    }
    //进行休息,否则可能导致子线程未执行时,主线程已经结束,直接结束了进程
    sleep(1);
    return 0;
}
```

`使用gcc编译时容易出现的错误`

```bash
gcc pthread_create.c 
```
错误原因是编译器链接不到线程库文件(动态库)，需在编译时通过参数指定出来
动态库名为 `libpthread.so`需要使用的参数为 `-l`
据规则掐头去尾最终形态应该写成：`-lpthread`（参数和参数值中间可以有空格）。

所以正确的写法是

```bash
 gcc pthread_create.c -lpthread
```

> 在打印的输出中为什么子线程处理函数没有执行呢（只看到了子线程的部分日志输出?
主线程一直在运行, 执行期间创建出了子线程，说明主线程有CPU时间片, 在这个时间片内将代码执行完毕了, 主线程就退出了。
子线程被创建出来之后需要抢cpu时间片, 抢不到就不能运行，如果主线程退出了, `虚拟地址空间就被释放了, 子线程一并销毁`。
但是如果某一个子线程退出了, 主线程仍在运行, 虚拟地址空间依旧存在。
结论：在没有人为干预的情况下，虚拟地址空间的生命周期和主线程是一样的，与子线程无关。
目前的解决方案: 让子线程执行完毕, 主线程再退出, 可以在主线程中添加挂起函数 `sleep();`


---

# 3. 线程退出

> 在编写多线程程序的时候，如果想要让线程退出，但不导致虚拟地址空间的释放（针对于主线程）
> 我们就可以调用线程库中的线程退出函数，只要调用该函数当前线程就马上退出了，并且不会影响到其他线程的正常运行，不管是在`子线程或者主线程中都可以使用`。

```c
#include <pthread.h>
void pthread_exit(void *retval);
```

 - 参数: 
 线程退出时携带的数据，当前子线程的主线程会得到该数据。如果不需要使用，指定为NULL


eg:

```c
#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>
#include <string.h>
#include <pthread.h>

// 子线程的处理代码
void* working(void* arg)
{
    sleep(1);
    printf("我是子线程, 线程ID: %ld\n", pthread_self());
    for(int i=0; i<9; ++i)
    {
        if(i==6)
        {
            pthread_exit(NULL);	// 直接退出子线程
        } 
        printf("child == i: = %d\n", i);
    }
    return NULL;
}

int main()
{
    // 1. 创建一个子线程
    pthread_t tid;
    pthread_create(&tid, NULL, working, NULL);

    printf("子线程创建成功, 线程ID: %ld\n", tid);
    // 2. 子线程不会执行下边的代码, 主线程执行
    printf("我是主线程, 线程ID: %ld\n", pthread_self());
    for(int i=0; i<3; ++i)
    {
        printf("i = %d\n", i);
    }

    // 主线程调用退出函数退出, 地址空间不会被释放
    pthread_exit(NULL);
    
    return 0;
}
```

---

# 4. 线程回收

## 4.1 线程函数

> 线程和进程一样，子线程退出的时候其内核资源主要由主线程回收
> 线程库中提供的线程回收函叫做`pthread_join()`
> 这个函数是一个阻塞函数
> 子线程在运行, 调用该函数就会阻塞
> 子线程退出, 函数解除阻塞进行资源的回收. 
函数被调用一次，只能回收一个子线程，如果有多个子线程则需要循环进行回收。
另外通过线程回收函数还可以获取到子线程退出时传递出来的数据，函数原型如下：

```c
#include <pthread.h>
// 这是一个阻塞函数, 子线程在运行这个函数就阻塞
// 子线程退出, 函数解除阻塞, 回收对应的子线程资源, 类似于回收进程使用的函数 wait()
int pthread_join(pthread_t thread, void **retval);
```

 - 参数:
	 -  thread: 要被回收的子线程的线程ID
	 -  retval: 二级指针, 指向一级指针的地址, 是一个传出参数, 这个地址中存储了pthread_exit() 传递出的数据，如果不需要这个参数，可以指定为NULL
 - 返回值：线程回收成功返回0，回收失败返回错误号。

---

## 4.2 回收子线程数据

> 在子线程退出的时候可以使用`pthread_exit()`的参数将数据传出
> 在回收这个子线程的时候可以通过`phread_join()`的第二个参数来接收子线程传递出的数据。接收数据有很多种处理方式，列举几种：

### 4.2.1 使用子线程栈

> 通过函数`pthread_exit(void *retval);`可以得知，子线程退出的时候，需要将数据记录到一块内存中，通过参数传出的是存储数据的内存的地址，而不是具体数据，因为参数是void*类型，所有这个万能指针可以指向任意类型的内存地址。先来看第一种方式，将子线程退出数据保存在子线程自己的栈区：

```c
// pthread_join.c
#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>
#include <string.h>
#include <pthread.h>

// 定义结构
struct Persion
{
    int id;
    char name[36];
    int age;
};

// 子线程的处理代码
void* working(void* arg)
{
    printf("我是子线程, 线程ID: %ld\n", pthread_self());
    for(int i=0; i<9; ++i)
    {
        printf("child == i: = %d\n", i);
        if(i == 6)
        {
            struct Persion p;
            p.age  =12;
            strcpy(p.name, "tom");
            p.id = 100;
            // 该函数的参数将这个地址传递给了主线程的pthread_join()
            pthread_exit(&p);
        }
    }
    return NULL;	// 代码执行不到这个位置就退出了
}

int main()
{
    // 1. 创建一个子线程
    pthread_t tid;
    pthread_create(&tid, NULL, working, NULL);

    printf("子线程创建成功, 线程ID: %ld\n", tid);
    // 2. 子线程不会执行下边的代码, 主线程执行
    printf("我是主线程, 线程ID: %ld\n", pthread_self());
    for(int i=0; i<3; ++i)
    {
        printf("i = %d\n", i);
    }

    // 阻塞等待子线程退出
    void* ptr = NULL;
    // ptr是一个传出参数, 在函数内部让这个指针指向一块有效内存
    // 这个内存地址就是pthread_exit() 参数指向的内存
    pthread_join(tid, &ptr);
    // 打印信息
    struct Persion* pp = (struct Persion*)ptr;
    printf("子线程返回数据: name: %s, age: %d, id: %d\n", pp->name, pp->age, pp->id);
    printf("子线程资源被成功回收...\n");
    
    return 0;
}
```


但是当我们编译时打印出的信息并没有得到子线程的数据

> 具体原因是这样的：
如果多个线程共用同一个虚拟地址空间，每个线程在栈区都有一块属于自己的内存，相当于栈区被这几个线程平分了，当线程退出，线程在栈区的内存也就被回收了，因此随着子线程的退出，写入到栈区的数据也就被释放了。


---

### 4.2.2 使用全局变量

> 位于同一虚拟地址空间中的线程，虽然不能共享栈区数据，但可以共享全局`数据区和堆区`数据，因此在子线程退出的时候可以将传出数据存储到全局变量、静态变量或者堆内存中。

```c
#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>
#include <string.h>
#include <pthread.h>

// 定义结构
struct Persion
{
    int id;
    char name[36];
    int age;
};

struct Persion p;	// 定义全局变量

// 子线程的处理代码
void* working(void* arg)
{
    printf("我是子线程, 线程ID: %ld\n", pthread_self());
    for(int i=0; i<9; ++i)
    {
        printf("child == i: = %d\n", i);
        if(i == 6)
        {
            // 使用全局变量
            p.age  =12;
            strcpy(p.name, "tom");
            p.id = 100;
            // 该函数的参数将这个地址传递给了主线程的pthread_join()
            pthread_exit(&p);
        }
    }
    return NULL;
}

int main()
{
    // 1. 创建一个子线程
    pthread_t tid;
    pthread_create(&tid, NULL, working, NULL);

    printf("子线程创建成功, 线程ID: %ld\n", tid);
    // 2. 子线程不会执行下边的代码, 主线程执行
    printf("我是主线程, 线程ID: %ld\n", pthread_self());
    for(int i=0; i<3; ++i)
    {
        printf("i = %d\n", i);
    }

    // 阻塞等待子线程退出
    void* ptr = NULL;
    // ptr是一个传出参数, 在函数内部让这个指针指向一块有效内存
    // 这个内存地址就是pthread_exit() 参数指向的内存
    pthread_join(tid, &ptr);
    // 打印信息
    struct Persion* pp = (struct Persion*)ptr;
    printf("name: %s, age: %d, id: %d\n", pp->name, pp->age, pp->id);
    printf("子线程资源被成功回收...\n");
    
    return 0;
}
```

---

### 4.2.3 使用主线程栈

> 虽然每个线程都有属于自己的栈区空间，但是位于`同一个地址空间的多个线程是可以相互访问对方的栈空间上的数据的。`
> 由于很多情况下还需要在主线程中回收子线程资源，所以主线程一般都是最后退出，基于这个原因在下面程序中将子线程返回的数据保存到了主线程的栈区内存中：

```c
#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>
#include <string.h>
#include <pthread.h>

// 定义结构
struct Persion
{
    int id;
    char name[36];
    int age;
};


// 子线程的处理代码
void* working(void* arg)
{
    struct Persion* p = (struct Persion*)arg;
    printf("我是子线程, 线程ID: %ld\n", pthread_self());
    for(int i=0; i<9; ++i)
    {
        printf("child == i: = %d\n", i);
        if(i == 6)
        {
            // 使用主线程的栈内存
            p->age  =12;
            strcpy(p->name, "tom");
            p->id = 100;
            // 该函数的参数将这个地址传递给了主线程的pthread_join()
            pthread_exit(p); //p本来就是指针
        }
    }
    return NULL;
}

int main()
{
    // 1. 创建一个子线程
    pthread_t tid;

    struct Persion p;
    // 主线程的栈内存传递给子线程
    pthread_create(&tid, NULL, working, &p);

    printf("子线程创建成功, 线程ID: %ld\n", tid);
    // 2. 子线程不会执行下边的代码, 主线程执行
    printf("我是主线程, 线程ID: %ld\n", pthread_self());
    for(int i=0; i<3; ++i)
    {
        printf("i = %d\n", i);
    }

    // 阻塞等待子线程退出
    void* ptr = NULL;
    // ptr是一个传出参数, 在函数内部让这个指针指向一块有效内存
    // 这个内存地址就是pthread_exit() 参数指向的内存
    pthread_join(tid, &ptr);
    // 打印信息
    printf("name: %s, age: %d, id: %d\n", p.name, p.age, p.id);
    printf("子线程资源被成功回收...\n");
    
    return 0;
}
```

> 在上面的程序中，调用pthread_create()创建子线程，并将主线程中栈空间变量p的地址传递到了子线程中，在子线程中将要传递出的数据写入到了这块内存中。
> 也就是说在程序的main()函数中，通过指针变量ptr或者通过结构体变量p都可以读出子线程传出的数据。

---

# 5. 线程分离

> 在某些情况下，程序中的主线程有属于自己的业务处理流程，如果让主线程负责子线程的资源回收，调用pthread_join()只要子线程不退出主线程就会一直被阻塞，主要线程的任务也就不能被执行了。
在线程库函数中为我们提供了线程分离函数`pthread_detach()`，调用这个函数之后指定的子线程就可以和主线程分离，当子线程退出的时候，其占用的内核资源就被系统的其他进程接管并回收了。`线程分离之后在主线程中使用pthread_join()就回收不到子线程资源了。`

```c
#include <pthread.h>
// 参数是子线程的线程ID, 主线程就可以和这个子线程分离了
int pthread_detach(pthread_t thread);
```

```c
#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>
#include <string.h>
#include <pthread.h>

// 子线程的处理代码
void* working(void* arg)
{
    printf("我是子线程, 线程ID: %ld\n", pthread_self());
    for(int i=0; i<9; ++i)
    {
        printf("child == i: = %d\n", i);
    }
    return NULL;
}

int main()
{
    // 1. 创建一个子线程
    pthread_t tid;
    pthread_create(&tid, NULL, working, NULL);

    printf("子线程创建成功, 线程ID: %ld\n", tid);
    // 2. 子线程不会执行下边的代码, 主线程执行
    printf("我是主线程, 线程ID: %ld\n", pthread_self());
    for(int i=0; i<3; ++i)
    {
        printf("i = %d\n", i);
    }

    // 设置子线程和主线程分离
    pthread_detach(tid);

    // 让主线程自己退出即可
    pthread_exit(NULL);
    
    return 0;
}
```

---

# 6. 其他线程函数

## 6.1 线程取消

> 线程取消的意思就是在某些特定情况下在一个线程中杀死另一个线程。
> 使用这个函数杀死一个线程需要分两步：
在线程A中调用线程取消函数pthread_cancel，指定杀死线程B，这时候线程B是死不了的
在线程B中进程一次`系统调用`（从用户区切换到内核区），否则线程B可以一直运行。

```c
#include <pthread.h>
int pthread_cancel(pthread_t thread);
```
 - 参数：要杀死的线程的线程ID
 - 返回值：函数调用成功返回0，调用失败返回非0错误号。

```c
#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>
#include <string.h>
#include <pthread.h>

// 子线程的处理代码
void* working(void* arg)
{
    int j=0;
    for(int i=0; i<9; ++i)
    {
        j++;
    }
    // printf函数会调用系统函数, 因此这是个间接的系统调用
    printf("我是子线程, 线程ID: %ld\n", pthread_self());
    for(int i=0; i<9; ++i)
    {
        printf(" child i: %d\n", i);
    }

    return NULL;
}

int main()
{
    // 1. 创建一个子线程
    pthread_t tid;
    pthread_create(&tid, NULL, working, NULL);

    printf("子线程创建成功, 线程ID: %ld\n", tid);
    // 2. 子线程不会执行下边的代码, 主线程执行
    printf("我是主线程, 线程ID: %ld\n", pthread_self());
    for(int i=0; i<3; ++i)
    {
        printf("i = %d\n", i);
    }

    // 杀死子线程, 如果子线程中做系统调用, 子线程就结束了
    pthread_cancel(tid);

    // 让主线程自己退出即可
    pthread_exit(NULL);
    
    return 0;
}
```

关于系统调用有两种方式：

- 直接调用Linux系统函数
- 调用标准C库函数，为了实现某些功能，在Linux平台下标准C库函数会调用相关的系统函数


---

## 6.2 线程ID的比较

> 在Linux中线程ID本质就是一个无符号长整形，因此可以直接使用比较操作符比较两个线程的ID，但是线程库是可以跨平台使用的，在某些平台上 pthread_t可能不是一个单纯的整形，这中情况下比较两个线程的ID必须要使用比较函数

```c
#include <pthread.h>
int pthread_equal(pthread_t t1, pthread_t t2);
```
- 参数：t1 和 t2 是要比较的线程的线程ID
- 返回值：如果两个线程ID相等返回非0值，如果不相等返回0


---