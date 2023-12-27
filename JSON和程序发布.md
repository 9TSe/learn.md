---
title: JSON和程序发布
date: 2023-10-22 09:54:42
tags:
- JSON
categories:
- Qt
cover: /pic/5.png
---


---


---

# 1. JSON

JSON(`JavaScrip Object Notation`)是一种`轻量级的数据交换格式`。它基于 ECMAScript (欧洲计算机协会制定的js规范)的一个子集，采用`完全独立于编程语言的文本格式来存储和表示数据`。简洁和清晰的层次结构使得 JSON 成为理想的数据交换语言。 易于人阅读和编写，同时也易于机器解析和生成，并有效地提升网络传输效率。

关于上面的描述可以精简为一句话：`Json是一种数据格式，和语言无关，在什么语言中都可以使用Json。`基于这种通用的数据格式，一般处理两方面的任务：

1. 组织数据（数据序列化），用于数据的网络传输
2. 组织数据（数据序列化），写磁盘文件实现数据的持久化存储（一般以.json作为文件后缀）
Json中主要有两种数据格式：Json数组和Json对象，并且这两种格式可以交叉嵌套使用，下面依次介绍下这两种数据格式：


## 1.1 Json数组
Json数组使用 [] 表示，[]里边是元素，元素和元素之间使用逗号间隔，`最后一个元素后边没有逗号`，一个Json数组中支持同时存在多种不同类型的成员，
包括：整形、 浮点、 字符串、 布尔类型、 json数组、 json对象、 空值-null。
由此可见Json数组比起C/C++数组要灵活很多。

- Json数组中的元素数据类型一致


```javascript
// 整形
[1,2,3,4,5]
// 字符串
["luffy", "sanji", "zoro", "nami", "robin"]
```

- Json数组中的元素数据类型不一致


```javascript
[12, 13.34, true, false, "hello,world", null]
```

- Json数组中的数组嵌套使用


```javascript
[
    ["cat", "dog", "panda", "beer", "rabbit"],
    ["北京", "上海", "天津", "重庆"],
    ["luffy", "boy", 19]
]
```

- Json数组和对象嵌套使用


```javascript
[
    {
        "luffy":{
            "age":19,
            "father":"Monkey·D·Dragon",
            "grandpa":"Monkey D Garp",
            "brother1":"Portgas D Ace",
            "brother2":"Sabo"
        }
    }
]
```
---
## 1.2 Json对象
Json对象使用 {} 来描述，每个Json对象中可以存储若干个元素，每一个元素对应一个键值对（key：value 结构），元素和元素之间使用逗号间隔，最后一个元素后边没有逗号。对于每个元素中的键值对有以下细节需要注意：

1. 键值（key）必须是字符串，位于同一层级的键值不要重复（因为是通过键值取出对应的value值）
2. value值的类型是可选的，可根据实际需求指定，可用类型包括：整形、 浮点、 字符串、 布尔类型、 json数组、 json对象、 空值-null。

**使用Json对象描述一个人的信息:**


```javascript
{
    "Name":"Ace",
    "Sex":"man",
    "Age":20,
    "Family":{
        "Father":"Gol·D·Roger",
        "Mother":"Portgas·D·Rouge",
        "Brother":["Sabo", "Monkey D. Luffy"]
    },
    "IsAlive":false,
    "Comment":"yyds"
}
```
---
## 1.3 注意事项
通过上面的介绍可用看到，Json的结构虽然简单，但是进行嵌套之后就可以描述很复杂的事情，在项目开发过程中往往需要我们根据实际需求自己定义Json格式用来存储项目数据。

另外，如果需要将Json数据持久化到磁盘文件中
需要注意一个问题：`在一个Json文件中只能有一个Json数组或者Json对象的根节点，不允许同时存储多个并列的根节点。`
举例说明：

**错误的写法**



```javascript
// test.json
{
    "name":"luffy",
    "age":19
}
{
    "user":"ace",
    "passwd":"123456"
}
```

> 错误原因：
> `在一个Json文件中有两个并列的Json根节点（并列包含Json对象和Json对象、Json对象和Json数组、Json数组和Json数组），根节点只能有一个。`

**正确的写法**


```javascript
// test.json
{
    "Name":"Ace",
    "Sex":"man",
    "Age":20,
    "Family":{
        "Father":"Gol·D·Roger",
        "Mother":"Portgas·D·Rouge",
        "Brother":["Sabo", "Monkey D. Luffy"]
    },
    "IsAlive":false,
    "Comment":"yyds"
}
```

在上面的例子中通过Json对象以及Json数组的嵌套描述了一个人的身份信息，并且根节点只有一个就是Json对象，如果还需要使用Json数组或者Json对象描述其他信息，需要将这些信息写入到其他文件中，`不要和这个Json对象并列写入到同一个文件里边`

---

# 2. Qt中JSON操作

从`Qt 5.0`开始提供了对Json的支持，我们可以直接使用Qt提供的Json类进行数据的组织和解析。相关的类常用的主要有四个，具体如下：

| Json类            | 介绍                                                      |
|-------------------|-----------------------------------------------------------|
| QJsonDocument     | 封装了一个完整的JSON文档，并且可以从UTF-8编码的基于文本的表示以及Qt自己的二进制格式读取和写入该文档。 |
| QJsonArray        | JSON数组是一个值列表。可以通过从数组中插入和删除QJsonValue来操作该列表。               |
| QJsonObject       | JSON对象是键值对的列表，其中键是唯一的字符串，值由QJsonValue表示。                    |
| QJsonValue        | 该类封装了JSON支持的数据类型。                                      |


## 2.1 QJsonValue
在Qt中`QJsonValue`可以封装的基础数据类型有六种（和Json支持的类型一致），分别为：

- 布尔类型：`QJsonValue::Bool`
- 浮点类型（包括整形）： `QJsonValue::Double`
- 字符串类型： `QJsonValue::String`
- Json数组类型： `QJsonValue::Array`
- Json对象类型：`QJsonValue::Object`
- 空值类型： `QJsonValue::Null`

这个类型可以通过`QJsonValue`的构造函数被封装为一个类对象:


```cpp
// Json对象
QJsonValue(const QJsonObject &o);
// Json数组
QJsonValue(const QJsonArray &a);
// 字符串
QJsonValue(const char *s);
QJsonValue(QLatin1String s);
QJsonValue(const QString &s);
// 整形 and 浮点型
QJsonValue(qint64 v);
QJsonValue(int v);
QJsonValue(double v);
// 布尔类型
QJsonValue(bool b);
// 空值类型
QJsonValue(QJsonValue::Type type = Null);
```

如果我们得到一个`QJsonValue`对象，如何判断内部封装的到底是什么类型的数据呢？这时候就需要调用相关的判断函数了，具体如下：


```cpp
// 是否是Json数组
bool isArray() const;
// 是否是Json对象
bool isObject() const;
// 是否是布尔类型
bool isBool() const;
// 是否是浮点类型(整形也是通过该函数判断)
bool isDouble() const;
// 是否是空值类型
bool isNull() const;
// 是否是字符串类型
bool isString() const;
// 是否是未定义类型(无法识别的类型)
bool isUndefined() const;
```

通过判断函数得到对象内部数据的实际类型之后，如果有需求就可以再次将其转换为对应的基础数据类型，对应的API函数如下：


```cpp
// 转换为Json数组
QJsonArray toArray(const QJsonArray &defaultValue) const;
QJsonArray toArray() const;
// 转换为布尔类型
bool toBool(bool defaultValue = false) const;
// 转换为浮点类型
double toDouble(double defaultValue = 0) const;
// 转换为整形
int toInt(int defaultValue = 0) const;
// 转换为Json对象
QJsonObject toObject(const QJsonObject &defaultValue) const;
QJsonObject toObject() const;
// 转换为字符串类型
QString toString() const;
QString toString(const QString &defaultValue) const;
```

---
## 2.2 QJsonObject
`QJsonObject`封装了Json中的对象，在里边可以存储多个键值对，为了方便操作，键值为字符串类型，值为`QJsonValue`类型。关于这个类的使用类似于C++中的STL类，仔细阅读API文档即可熟练上手使用，下面介绍一些常用API函数:

- 如何创建空的Json对象


```cpp
QJsonObject::QJsonObject();	// 构造空对象
```

- 将键值对添加到空对象中


```cpp
iterator QJsonObject::insert(const QString &key, const QJsonValue &value);
```

- 获取对象中键值对个数


```cpp
int QJsonObject::count() const;
int QJsonObject::size() const;
int QJsonObject::length() const;
```

- 通过key得到value


```cpp
QJsonValue QJsonObject::value(const QString &key) const;    // utf8
QJsonValue QJsonObject::value(QLatin1String key) const;	    // 字符串不支持中文
QJsonValue QJsonObject::operator[](const QString &key) const;
QJsonValue QJsonObject::operator[](QLatin1String key) const;
```

- 删除键值对


```cpp
void QJsonObject::remove(const QString &key);
QJsonValue QJsonObject::take(const QString &key);	// 返回key对应的value值
```

- 通过key进行查找

```cpp
iterator QJsonObject::find(const QString &key);
bool QJsonObject::contains(const QString &key) const;
```

- 遍历，方式有三种：

	1. 使用相关的迭代器函数

	2. 使用 [] 的方式遍历, 类似于遍历数组, []中是键值

	3. 先得到对象中所有的键值, 在遍历键值列表, 通过key得到value值


```cpp
QStringList QJsonObject::keys() const;
```
---
## 2.3 QJsonArray
`QJsonArray`封装了Json中的数组，在里边可以存储多个元素，为了方便操作，所有的元素类统一为`QJsonValue`类型。关于这个类的使用类似于C++中的STL类，仔细阅读API文档即可熟练上手使用，介绍一些常用API函数:

- 创建空的Json数组


```cpp
QJsonArray::QJsonArray();
```

- 添加数据


```cpp
void QJsonArray::append(const QJsonValue &value);	// 在尾部追加
void QJsonArray::insert(int i, const QJsonValue &value); // 插入到 i 的位置之前
iterator QJsonArray::insert(iterator before, const QJsonValue &value);
void QJsonArray::prepend(const QJsonValue &value); // 添加到数组头部
void QJsonArray::push_back(const QJsonValue &value); // 添加到尾部
void QJsonArray::push_front(const QJsonValue &value); // 添加到头部
```

- 计算数组元素的个数


```cpp
int QJsonArray::count() const;
int QJsonArray::size() const;
```

- 从数组中取出某一个元素的值


```cpp
QJsonValue QJsonArray::at(int i) const;
QJsonValue QJsonArray::first() const; // 头部元素
QJsonValue QJsonArray::last() const; // 尾部元素
QJsonValueRef QJsonArray::operator[](int i);
```

- 删除数组中的某一个元素


```cpp
iterator QJsonArray::erase(iterator it);    // 基于迭代器删除
void QJsonArray::pop_back();           // 删除尾部
void QJsonArray::pop_front();          // 删除头部
void QJsonArray::removeAt(int i);      // 删除i位置的元素
void QJsonArray::removeFirst();        // 删除头部
void QJsonArray::removeLast();         // 删除尾部
QJsonValue QJsonArray::takeAt(int i);  // 删除i位置的原始, 并返回删除的元素的值
```

- Josn数组的遍历，常用的方式有两种：

	1. 可以使用迭代器进行遍历（和使用迭代器遍历STL容器一样）
	2. 可以使用数组的方式遍历

---
## 2.4 QJsonDocument
它封装了一个完整的JSON文档，并且可以从UTF-8编码的基于文本的表示以及Qt自己的二进制格式读取和写入该文档。`QJsonObject` 和 `QJsonArray`这两个对象中的数据是不能直接转换为字符串类型的，如果要进行数据传输或者数据的持久化，操作的都是字符串类型而不是 `QJsonObject` 或者 `QJsonArray`类型，我们需要通过一个Json文档类进行二者之间的转换。

介绍一下这两个转换流程应该如何操作:

> `QJsonObject` 或者 `QJsonArray` ===> 字符串

1. **创建QJsonDocument对象**


```cpp
QJsonDocument::QJsonDocument(const QJsonObject &object);
QJsonDocument::QJsonDocument(const QJsonArray &array);
```

可以看出，通过构造函数就可以将实例化之后的`QJsonObject` 或者 `QJsonArray` 转换为`QJsonDocument`对象了。

2. **将文件对象中的数据进行序列化**


```cpp
// 二进制格式的json字符串
QByteArray QJsonDocument::toBinaryData() const;	 
// 文本格式
QByteArray QJsonDocument::toJson(JsonFormat format = Indented) const;	
```

通过调用`toxxx()`方法就可以得到文本格式或者二进制格式的Json字符串了。

3. **使用得到的字符串进行数据传输, 或者磁盘文件持久化**

>字符串 ===> `QJsonObject` 或者 `QJsonArray`

一般情况下，通过网络通信或者读磁盘文件就会得到一个Json格式的字符串，如果想要得到相关的原始数据就需要对字符串中的数据进行解析，具体解析流程如下：

1. 将得到的Json格式字符串通过 `QJsonDocument` 类的静态函数转换为`QJsonDocument`类对象


```cpp
[static] QJsonDocument QJsonDocument::fromBinaryData(const QByteArray &data, DataValidation validation = Validate);
// 参数文件格式的json字符串
[static] QJsonDocument QJsonDocument::fromJson(const QByteArray &json, QJsonParseError *error = Q_NULLPTR);
```

2. 将文档对象转换为json数组/对象


```cpp
// 判断文档对象中存储的数据是不是数组
bool QJsonDocument::isArray() const;
// 判断文档对象中存储的数据是不是json对象
bool QJsonDocument::isObject() const
    
// 文档对象中的数据转换为json对象
QJsonObject QJsonDocument::object() const;
// 文档对象中的数据转换为json数组
QJsonArray QJsonDocument::array() const;
```

3. 通过调用`QJsonArray` , `QJsonObject` 类提供的 API 读出存储在对象中的数据。
关于Qt中Json数据对象以及字符串之间的转换的操作流程是固定的，我们在编码过程中只需要按照上述模板处理即可，相关的操作是没有太多的技术含量可言的。

---
## 2.5 举例
### 2.5.1 写文件

```cpp
void writeJson()
{
    QJsonObject obj;
    obj.insert("Name", "Ace");
    obj.insert("Sex", "man");
    obj.insert("Age", 20);

    QJsonObject subObj;
    subObj.insert("Father", "Gol·D·Roger");
    subObj.insert("Mother", "Portgas·D·Rouge");
    QJsonArray array;
    array.append("Sabo");
    array.append("Monkey D. Luffy");
    subObj.insert("Brother", array);
    obj.insert("Family", subObj);
    obj.insert("IsAlive", false);
    obj.insert("Comment", "yyds");

    QJsonDocument doc(obj);
    QByteArray json = doc.toJson();

    QFile file("d:\\ace.json");
    file.open(QFile::WriteOnly);
    file.write(json);
    file.close();
}
```
---
### 2.5.2 读文件

```cpp
void MainWindow::readJson()
{
    QFile file("d:\\ace.json");
    file.open(QFile::ReadOnly);
    QByteArray json = file.readAll();
    file.close();

    QJsonDocument doc = QJsonDocument::fromJson(json);
    if(doc.isObject())
    {
        QJsonObject obj = doc.object();
        QStringList keys = obj.keys();
        for(int i=0; i<keys.size(); ++i)
        {
            QString key = keys.at(i);
            QJsonValue value = obj.value(key);
            if(value.isBool())
            {
                qDebug() << key << ":" << value.toBool();
            }
            if(value.isString())
            {
                qDebug() << key << ":" << value.toString();
            }
            if(value.isDouble())
            {
                qDebug() << key << ":" << value.toInt();
            }
            if(value.isObject())
            {
                qDebug()<< key << ":";
                // 直接处理内部键值对, 不再进行类型判断的演示
                QJsonObject subObj = value.toObject();
                QStringList ls = subObj.keys();
                for(int i=0; i<ls.size(); ++i)
                {
                    QJsonValue subVal = subObj.value(ls.at(i));
                    if(subVal.isString())
                    {
                        qDebug() << "   " << ls.at(i) << ":" << subVal.toString();
                    }
                    if(subVal.isArray())
                    {
                        QJsonArray array = subVal.toArray();
                        qDebug() << "   " << ls.at(i) << ":";
                        for(int j=0; j<array.size(); ++j)
                        {
                            // 因为知道数组内部全部为字符串, 不再对元素类型进行判断
                            qDebug() << "       " << array[j].toString();
                        }
                    }
                }
            }
        }
    }
}
```

一般情况下，对于Json字符串的解析函数都是有针对性的，因为需求不同设计的Json格式就会有所不同，所以不要试图写出一个通用的Json解析函数，这样只会使函数变得臃肿而且不易于维护，每个Json格式对应一个相应的解析函数即可。

上面的例子中演示Qt中Json类相关API函数的使用将解析步骤写的复杂了，因为在解析的时候我们是知道Json对象中的所有key值的，可以直接通过key值将对应的value值取出来，因此上面程序中的一些判断和循环其实是可以省去的。


---

# 3. cjson库的使用

C语言的Json库 – cJson。cJSON是一个超轻巧，携带方便，单文件，简单的可以作为ANSI-C标准的JSON解析器。

cJSON 是一个开源项目，[github下载地址](https://github.com/DaveGamble/cJSON)

cJSON，目前来说，主要的文件有两个，一个`cJSON.c` 一个`cJSON.h`。
使用的时候，将`头文件include进去即可`。
如果是在Linux操作系统中使用，编译 到时候需要添加数据库libm.so，如下所示：


```bash
gcc  *.c  cJSON.c  -lm
```


## 3.1 cJSON结构体
在`cJSON.h`中定义了一个非常重要的结构体`cJSON`，
想要熟悉使用cJSON库函数可从cJSON结构体入手，cJSON结构体如下所示：


```c
typedef struct cJSON {  
     struct cJSON *next,*prev;   
     struct cJSON *child;   
     int type;   
     char *valuestring;        // value值是字符串类型
     int valueint;  
     double valuedouble;   
     char *string;             // 对象中的key
} cJSON; 
```

关于这个结构体做如下几点的说明:

1. `cJOSN`结构体是一个双向链表，并且可通过`child`指针访问下一层。

2. 结构体成员`type`变量用于描述数据元素的类型（如果是键值对表示`value`值的类型），数据元素可以是字符串可以是整形，也可以是浮点型。

	- 如果是整形值的话可通过`valueint`将值取出
	- 如果是浮点型的话可通过`valuedouble`将值取出
	- 如果是字符串类型的话可通过`valuestring`将值取出

3. 结构体成员`string`表示键值对中键值的名称。

`cJSON`作为`Json`格式的解析库，其主要功能就是构建和解析`Json`格式了，比如要发送数据：用途就是发送端将要发送的数据以`json`形式封装，然后发送，接收端收到此数据后，还是按`json`形式解析，就得到想要的数据了。

---
## 3.2 cJson API
Json格式的数据无外乎有两种Json对象和Json数组，创建的Json数据串可能是二者中 的一种，也可能是二者的组合，不管哪一种通过调用相关的API函数都可以轻松的做到这一点。

### 3.2.1 数据的封装
在`cJSON.h`头文件中可以看到一些函数声明，通过调用这些创建函数就可以将`Json`支持的数据类型封装为`cJSON`结构体类型：


```c
// 空值类型
extern cJSON *cJSON_CreateNull(void);
// 布尔类型
extern cJSON *cJSON_CreateTrue(void);
extern cJSON *cJSON_CreateFalse(void);
extern cJSON *cJSON_CreateBool(int b);
// 数值类型
extern cJSON *cJSON_CreateNumber(double num);
// 字符串类型
extern cJSON *cJSON_CreateString(const char *string);
// json数组(创建空数组)
extern cJSON *cJSON_CreateArray(void);
// json对象(创建空对象)
extern cJSON *cJSON_CreateObject(void);
```

另外，cJson库中还给我我们提供了一些更为简便的操作函数，在创建数组的同时还可以进行初始化


```c
// 创建一个Json数组, 元素为整形
extern cJSON *cJSON_CreateIntArray(const int *numbers,int count);
// 创建一个Json数组, 元素为浮点
extern cJSON *cJSON_CreateFloatArray(const float *numbers,int count);
extern cJSON *cJSON_CreateDoubleArray(const double *numbers,int count);
// 创建一个Json数组, 元素为字符串类型
extern cJSON *cJSON_CreateStringArray(const char **strings,int count);
```
---
### 3.2.2 Json对象操作
当得到一个`Json`对象之后，就可以往对象中添加键值对了，可以使用`cJSON_AddItemToObject()`


```c
extern void cJSON_AddItemToObject(cJSON *object,const char *string,cJSON *item);
```

在`cJSON`库中节点的从属关系是通过树来维护的，每一层节点都是通过链表来维护的，这样就能分析出该函数参数的含义：

- object：要添加的键值对从属于那个节点
- string：添加的键值对的键值
- item：添加的键值对的value值（需要先将其封装为cJSON类型的结构体）

为了让我的操作更加方便，cJson库还给我们提供了一些宏函数，方便我们快速的往Json对象中添加键值对


```c
#define cJSON_AddNullToObject(object,name)      cJSON_AddItemToObject(object, name, cJSON_CreateNull())
#define cJSON_AddTrueToObject(object,name)      cJSON_AddItemToObject(object, name, cJSON_CreateTrue())
#define cJSON_AddFalseToObject(object,name)     cJSON_AddItemToObject(object, name, cJSON_CreateFalse())
#define cJSON_AddBoolToObject(object,name,b)    cJSON_AddItemToObject(object, name, cJSON_CreateBool(b))
#define cJSON_AddNumberToObject(object,name,n)  cJSON_AddItemToObject(object, name, cJSON_CreateNumber(n))
#define cJSON_AddStringToObject(object,name,s)  cJSON_AddItemToObject(object, name, cJSON_CreateString(s))
```

我们还可以根据`Json`对象中的键值取出相应的`value`值，API函数原型如下:


```c
extern cJSON *cJSON_GetObjectItem(cJSON *object,const char *string);
```

---
### 3.2.3 Json数组操作
- 添加数据到Json数组中（原始数据需要先转换为cJSON结构体类型）


```c
extern void cJSON_AddItemToArray(cJSON *array, cJSON *item);
```

- 得到Json数组中元素的个数:


```c
extern int cJSON_GetArraySize(cJSON *array);
```

- 得到Json数组中指定位置的原素，如果返回NULL表示取值失败了。


```c
extern cJSON *cJSON_GetArrayItem(cJSON *array,int item);
```

---
### 3.2.4 序列化
序列化就是将`Json`格式的数据转换为字符串的过程，`cJson`库中给我们提供了3个转换函数，具体如下：

第一个参数`item`表示Json数据块的根节点。


```c
extern char  *cJSON_Print(cJSON *item);
extern char  *cJSON_PrintUnformatted(cJSON *item);
extern char *cJSON_PrintBuffered(cJSON *item,int prebuffer,int fmt);
```

- 调用`cJSON_Print()`函数我们可以得到一个带格式的`Json`字符串（有换行，看起来更直观）
- 调用`cJSON_PrintUnformatted()`函数会得到一个没有格式的`Json`字符串（没有换行，所有的数据都在同一行）。
- 调用`cJSON_PrintBuffered()`函数使用缓冲策略将`Json`实体转换为字符串，参数`prebuffer`是指定缓冲区的大小，参数`fmt==0`表示未格式化，`fmt==1`表示格式化。

我们在编码过程中可以根据自己的实际需求调用相关的操作函数得到对应格式的Json字符串。

---
### 3.2.5 Json字符串的解析
如果我们得到了一个Json格式的字符串，想要读出里边的数据，就需要对这个字符串进行解析，处理方式就是将字符串转换为`cJSON`结构体，然后再基于这个结构体读里边的原始数据，转换函数的函数原型如下：


```c
extern cJSON *cJSON_Parse(const char *value);
```

---
### 3.2.6 内存释放
当我们将数据封装为`cJSON`结构类型的节点之后都会得到一块堆内存，当我们释放某个节点的时候可以调用`cJson`库提供的删除函数`cJSON_Delete()`，函数原型如下：


```c
extern void   cJSON_Delete(cJSON *c);
```

该函数的参数为要释放的节点的地址，强调：
`在进行内存地址释放的时候，当前节点以及其子节点都会被删除。`

---
## 3.3 Json数据的封装
### 3.3.1 Json对象操作举例
>创建一个对象，并向这个对象里添加字符串和整型键值：


```c
#include<stdio.h>
#include<stdlib.h>
#include<string.h>
#include"cJSON.h"
 
int main()
{
    cJSON * root;
    cJSON *arry;

    root=cJSON_CreateObject();                     // 创建根数据对象
    cJSON_AddStringToObject(root,"name","luffy");  // 添加键值对
    cJSON_AddStringToObject(root,"sex","man");     // 添加键值对
    cJSON_AddNumberToObject(root,"age",19);        // 添加键值对

    char *out = cJSON_Print(root);   // 将json形式转换成字符串
    printf("%s\n",out);

    // 释放内存  
    cJSON_Delete(root);  
    free(out);        
}
```

>运行结果


```javascript
{
	"name":	"luffy",
	"sex":	"man",
	"age":	19
}
```

若干说明:

1. `cJSON_CreateObject`函数可创建一个根对象，返回的是一个 `cJSON`指针，在这个指针用完了以后，需要手动调用 `cJSON_Delete(root)`进行内存回收。
2. 函数`cJSON_Print()`内部封装了malloc函数，所以需要使用`free()`函数释放被`out`占用的内存空间。

---
### 3.3.2 Json数组操作举例
>创建一个数组，并向数组添加一个字符串和一个数字

```c
int main(int argc, char **argv)
{
    cJSON *root;
    root = cJSON_CreateArray();
    cJSON_AddItemToArray(root, cJSON_CreateString("Hello world"));
    cJSON_AddItemToArray(root, cJSON_CreateNumber(10)); 
    // char *s = cJSON_Print(root);
    char *s = cJSON_PrintUnformatted(root);
    if(s)
    {
        printf(" %s \n",s);
        free(s);
    }
    cJSON_Delete(root);
    return 0;
}
```

>运行结果:


```javascript
["Hello world",10]
```

---
### 3.3.3 Json对象、数组嵌套使用
>对象里面包括一个数组，数组里面包括对象，对象里面再添加一个字符串和一个数字


```javascript
{
    "person":[{
        "name":"luffy",
        "age":19
    }]
}
```

**示例代码:**

```c
int main(int argc, char **argv)
{
    cJSON *root, *body, *list;
    // josn 对象 root
    root = cJSON_CreateObject();
    // root 添加键值对 person:json数组A
    cJSON_AddItemToObject(root,"person", body = cJSON_CreateArray());
    // json数组A 添加Json对象B
    cJSON_AddItemToArray(body, list = cJSON_CreateObject());
    // 在json对象B中添加键值对: "name":"luffy"
    cJSON_AddStringToObject(list,"name","luffy");
    // 在json对象B中添加键值对: "age":19
    cJSON_AddNumberToObject(list,"age",19);
 
    // char *s = cJSON_Print(root);
    char *s = cJSON_PrintUnformatted(root);
    if(s)
    {
        printf(" %s \n",s);
        free(s);
    }
    if(root)
    {
        cJSON_Delete(root); 
    }
    return 0;
}
```

**运行结果:**


```javascript
{"person":[{"name":"luffy","age":19}]}
```

---
## 3.4 解析Json字符串
### 3.4.1 解析Json对象
Json字符串的解析流程和数据的封装流程相反，假设我们有这样一个Json字符串（字符串中的双引号需要通过转义字符将其转译为普通字符）：


```javascript
{\"name\":\"luffy\",\"sex\":\"man\",\"age\":19}
```

**示例代码如下：**


```c
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include "cJSON.h"
 
int main()
{
    cJSON *json, *name, *sex, *age;
    char* out="{\"name\":\"luffy\",\"sex\":\"man\",\"age\":19}";
 
    json = cJSON_Parse(out); //解析成json形式
    name = cJSON_GetObjectItem(json, "name");  //获取键值内容
    sex = cJSON_GetObjectItem(json, "sex");
    age = cJSON_GetObjectItem(json, "age");
 
    printf("name:%s,sex:%s,age:%d\n", name->valuestring, sex->valuestring, age->valueint);
 
    cJSON_Delete(json);  //释放内存 
}
```

**输出的结果:**


```javascript
name:luffy,sex:man,age:19
```

`如果是在严格的场所，应该先判定每个 item 的 type，然后再考虑去取值。`

---
### 3.4.2 解析嵌套的Json对象
>解析一个嵌套的Json对象，数据如下：


```javascript
{\"list\":{\"name\":\"luffy\",\"age\":19},\"other\":{\"name\":\"ace\"}}
```

```c
int main()
{
    char *s = "{\"list\":{\"name\":\"luffy\",\"age\":19},\"other\":{\"name\":\"ace\"}}";
    cJSON *root = cJSON_Parse(s);
    if(!root) 
    {
        printf("get root faild !\n");
        return -1;
    }

    cJSON *js_list = cJSON_GetObjectItem(root, "list");
    if(!js_list) 
    {
        printf("no list!\n");
        return -1;
    }
    printf("list type is %d\n",js_list->type);

    cJSON *name = cJSON_GetObjectItem(js_list, "name");
    if(!name) 
    {
        printf("No name !\n");
        return -1;
    }
    printf("name type is %d\n",name->type);
    printf("name is %s\n",name->valuestring);

    cJSON *age = cJSON_GetObjectItem(js_list, "age");
    if(!age) 
    {
        printf("no age!\n");
        return -1;
    }
    printf("age type is %d\n", age->type);
    printf("age is %d\n",age->valueint);

    cJSON *js_other = cJSON_GetObjectItem(root, "other");
    if(!js_other) 
    {
        printf("no list!\n");
        return -1;
    }
    printf("list type is %d\n",js_other->type);

    cJSON *js_name = cJSON_GetObjectItem(js_other, "name");
    if(!js_name) 
    {
        printf("No name !\n");
        return -1;
    }
    printf("name type is %d\n",js_name->type);
    printf("name is %s\n",js_name->valuestring);

    if(root)
    {
        cJSON_Delete(root);
    }
    return 0;
}
```

打印结果:


```javascript
list type is 6
name type is 4
name is luffy
age type is 3
age is 19
list type is 6
name type is 4
name is ace
```
---
### 3.4.3 解析Json数组
如果我们遇到的Json字符串是一个Json数组格式，处理方式和Json对象差不多，比如我们要解析如下字符串：


```javascript
{\"names\":[\"luffy\",\"robin\"]}
```


```c
int main(int argc, char **argv)
{
    char *s = "{\"names\":[\"luffy\",\"robin\"]}";
    cJSON *root = cJSON_Parse(s);
    if(!root) 
    {
        printf("get root faild !\n");
        return -1;
    }
    cJSON *js_list = cJSON_GetObjectItem(root, "names");
    if(!js_list)
    {
        printf("no list!\n");
        return -1;
    }
    int array_size = cJSON_GetArraySize(js_list);
    printf("array size is %d\n",array_size);
    for(int i=0; i< array_size; i++) 
    {
        cJSON *item = cJSON_GetArrayItem(js_list, i);
        printf("item type is %d\n",item->type);
        printf("%s\n",item->valuestring);
    }

    if(root)
    {
        cJSON_Delete(root);
    }
    return 0;
}
```

---
### 3.4.4 解析嵌套的Json对象和数组
对于Json字符串最复杂的个数莫过于Json对象和Json数组嵌套的形式，通过一个例子演示一下应该如何解析，字符串格式如下：


```javascript
{\"list\":[{\"name\":\"luffy\",\"age\":19},{\"name\":\"sabo\",\"age\":21}]}
```

在解析的时候，我们只需要按照从属关系，一层层解析即可：

1. 根节点是一个Json对象，基于根节点中的key值取出对应的value值，得到一个Json数组
2. 读出Json数组的大小，遍历里边的各个元素，每个元素都是一个Json对象
3. 将Json对象中的键值对根据key值取出对应的value值
4. 从取出的Value值中读出实际类型对应的数值

**示例代码如下：**


```c
#include "cJSON.h"
#include <stdio.h>
#include <stdlib.h>

int main(int argc, char **argv)
{
    char *s = "{\"list\":[{\"name\":\"luffy\",\"age\":19},{\"name\":\"sabo\",\"age\":21}]}";
    cJSON *root = cJSON_Parse(s);
    if(!root) 
    {
        printf("get root faild !\n");
        return -1;
    }
    cJSON *list = cJSON_GetObjectItem(root, "list");
    if(!list)
    {
        printf("no list!\n");
        return -1;
    }
    int array_size = cJSON_GetArraySize(list);
    printf("array size is %d\n",array_size);
    
    for(int i=0; i< array_size; i++) 
    {
        cJSON* item = cJSON_GetArrayItem(list, i);
        cJSON* name = cJSON_GetObjectItem(item, "name");
        printf("name is %s\n",name->valuestring);
        cJSON* age = cJSON_GetObjectItem(item, "age");
        printf("age is %d\n",age->valueint);
    }

    if(root)
    {
        cJSON_Delete(root);
    }
    return 0;
}
```

---

# 4. jsoncpp的编译和使用

## 4.1 下载和编译
### 4.1.1 下载
**下载 jsoncpp**
Jsoncpp是个跨平台的C++开源库，提供的类为我们提供了很便捷的操作，而且使用的人也很多。在使用之前我们首先要从github仓库下载[源码](https://github.com/open-source-parsers/jsoncpp)

**下载 cmake 工具**
由于都是基于VS进行项目开发，下载的源码我们一般不会直接使用，而且将其编译成相应的库文件（动态库或者静态库），这样不论是从使用或者部署的角度来说，操作起来都会更方便一些。

但是，从github下载的源码不能直接通过VS打开，编译就更谈不上了。它提供的默认编译方式是cmake。我们可以通过使用cmake工具将下载的jsoncpp源码生成一个VS项目，这样就可以通过VS编译出需要的库文件了。

[官方下载地址](https://cmake.org/download/)

---

### 4.1.2 生成VS项目


打开CMake

![在这里插入图片描述](img/b.1.png)


需要在工具中指定本地的jsoncpp路径（git clone 之后就会得到这个目录），这是我本地的目录：
![在这里插入图片描述](img/b.2.png)



第二个需要指定的是一个文件存储路径（生成的VS项目会保存到这个目录下），保证这是一个本地的有效目录即可。

![在这里插入图片描述](img/b.3.png)


设置好之后进行配置，点击Configure按钮

![在这里插入图片描述](img/b.4.png)


此处需要设置一下，VS的版本以及生成器的平台位数，不填默认就是64位。

![在这里插入图片描述](img/b.5.png)


配置完成，开始生成VS项目。

![在这里插入图片描述](img/b.6.png)


打开在CMake工具中指定的生成目录，我这里是D:\output-project，基于项目文件jsoncpp.sln打开这个VS项目。

---
### 4.1.3 编译
基于生成的项目文件打开VS项目之后，可以看到里边有很多子项目

![在这里插入图片描述](img/b.7.png)


我们只需要编译上图标记的那一个就可以了，编译成功之后就可以得到我们需要的库文件了。

![在这里插入图片描述](img/b.8.png)


通过输出的日志信息，就能找到我们想要的动态库了，把这两个文件收集起来备用。

---
## 4.2 jsoncpp 的使用
`jsoncpp`库中的类被定义到了一个Json命名空间中，建议在使用这个库的时候先声明这个命名空间：


```cpp
using namespace Json;
```

使用`jsoncpp`库解析`json`格式的数据，我们只需要掌握三个类：

1. `Value 类`：将json支持的数据类型进行了包装，最终得到一个Value类型
2. `FastWriter类`：将Value对象中的数据序列化为字符串
3. `Reader类`：反序列化, 将json字符串 解析成 Value 类型


### 4.2.1 Value类
这个类可以看做是一个包装器，它可以封装Json支持的所有类型，这样我们在处理数据的时候就方便多了。

| 枚举类型        | 说明                                   | 翻译              |
|-----------------|----------------------------------------|-------------------|
| nullValue       | 'null' value                           | 不表示任何数据，空值  |
| intValue        | signed integer value                   | 表示有符号整数        |
| uintValue       | unsigned integer value                 | 表示无符号整数        |
| realValue       | double value                           | 表示浮点数           |
| stringValue     | UTF-8 string value                     | 表示utf8格式的字符串  |
| booleanValue    | bool value                             | 表示布尔数           |
| arrayValue      | array value (ordered list)             | 表示数组，即JSON串中的[]  |
| objectValue     | object value (collection of name/value pairs) | 表示键值对，即JSON串中的{}  |

**构造函数**
Value类为我们提供了很多构造函数，通过构造函数来封装数据，最终得到一个统一的类型。


```cpp
// 因为Json::Value已经实现了各种数据类型的构造函数
Value(ValueType type = nullValue);
Value(Int value);
Value(UInt value);
Value(Int64 value);
Value(UInt64 value);
Value(double value);
Value(const char* value);
Value(const char* begin, const char* end);
Value(bool value);
Value(const Value& other);
Value(Value&& other);
```

**检测保存的数据类型**

```cpp
// 检测保存的数据类型
bool isNull() const;
bool isBool() const;
bool isInt() const;
bool isInt64() const;
bool isUInt() const;
bool isUInt64() const;
bool isIntegral() const;
bool isDouble() const;
bool isNumeric() const;
bool isString() const;
bool isArray() const;
bool isObject() const;
```

**将Value对象转换为实际类型**

```cpp
Int asInt() const;
UInt asUInt() const;
Int64 asInt64() const;
UInt64 asUInt64() const;
LargestInt asLargestInt() const;
LargestUInt asLargestUInt() const;
JSONCPP_STRING asString() const;
float asFloat() const;
double asDouble() const;
bool asBool() const;
const char* asCString() const;
```

**对json数组的操作**

```cpp
ArrayIndex size() const;
Value& operator[](ArrayIndex index);
Value& operator[](int index);
const Value& operator[](ArrayIndex index) const;
const Value& operator[](int index) const;
// 根据下标的index返回这个位置的value值
// 如果没找到这个index对应的value, 返回第二个参数defaultValue
Value get(ArrayIndex index, const Value& defaultValue) const;
Value& append(const Value& value);
const_iterator begin() const;
const_iterator end() const;
iterator begin();
iterator end();
```

**对json对象的操作**

```cpp
Value& operator[](const char* key);
const Value& operator[](const char* key) const;
Value& operator[](const JSONCPP_STRING& key);
const Value& operator[](const JSONCPP_STRING& key) const;
Value& operator[](const StaticString& key);

// 通过key, 得到value值
Value get(const char* key, const Value& defaultValue) const;
Value get(const JSONCPP_STRING& key, const Value& defaultValue) const;
Value get(const CppTL::ConstString& key, const Value& defaultValue) const;

// 得到对象中所有的键值
typedef std::vector<std::string> Members;
Members getMemberNames() const;
```

**将Value对象数据序列化为string**

```cpp
// 序列化得到的字符串有样式 -> 带换行 -> 方便阅读
// 写配置文件的时候
std::string toStyledString() const;
```

---

### 4.2.2 FastWriter 类

```cpp
// 将数据序列化 -> 单行
// 进行数据的网络传输
std::string Json::FastWriter::write(const Value& root);
```
---
### 4.2.3 Reader 类

```cpp
bool Json::Reader::parse(const std::string& document,
    Value& root, bool collectComments = true);
    参数:
        - document: json格式字符串
        - root: 传出参数, 存储了json字符串中解析出的数据
        - collectComments: 是否保存json字符串中的注释信息

// 通过begindoc和enddoc指针定位一个json字符串
// 这个字符串可以是完成的json字符串, 也可以是部分json字符串
bool Json::Reader::parse(const char* beginDoc, const char* endDoc,
    Value& root, bool collectComments = true);
	
// write的文件流  -> ofstream
// read的文件流   -> ifstream
// 假设要解析的json数据在磁盘文件中
// is流对象指向一个磁盘文件, 读操作
bool Json::Reader::parse(std::istream& is, Value& root, bool collectComments = true);
```
---
## 4.3 VS的配置
如果想要在VS中使用编译出的`jsoncpp`库，我们还需要做如下配置：

### 4.3.1 头文件
在编码过程中需要在项目文件中包含从github下载得到的头文件，有两种处理方式：

1. 将头文件放到项目目录下，直接被项目包含引用

2. 将头文件放到一个本地固定目录，以后就不再动了，在VS项目属性中设置包含这个目录，推荐这种

![在这里插入图片描述](img/b.9.png)


另外，在这个`include`目录中还有一个`json`子目录，所有的头文件都在这个子目录中，我们不要破坏这个目录结构：

![在这里插入图片描述](img/b.10.png)


在包含需要的头文件的时候，使用如下这种方式：


```cpp
#include <json/json.h>
```

把本地的头文件目录在项目属性窗口中进行配置：

![在这里插入图片描述](img/b.11.png)


这里头文件是放到了C盘的jsoncpp目录中：

![在这里插入图片描述](img/b.12.png)

---
### 4.3.2 库文件
我这里也是将生成的`jsoncpp.lib`和`jsoncpp.dll`放到了C盘（`C:\jsoncpp\lib`），在VS项目中需要指定这个库路径：


![在这里插入图片描述](img/b.13.png)
![在这里插入图片描述](img/b.14.png)



另外，还需要告诉VS需要加载的动态库是哪一个

![在这里插入图片描述](img/b.15.png)


此处指定的是动态库对应的`lib`文件，也就是`jsoncpp.lib`

![在这里插入图片描述](img/b.16.png)


配置完成之后，如果项目中使用了`jsoncpp`就可以编译通过了。在程序执行的时候，如果提示找不到`jsoncpp`的动态库，`把 jsoncpp.dll 拷贝到可执行所在的目录下就可以解决了。`

---
## 4.3 示例代码
比如：我们要将下面这个Json数组写入的一个文件中


```javascript
[
    12, 
    12.34, 
    true, 
    "tom", 
    ["jack", "ace", "robin"], 
    {"sex":"man", "girlfriend":"lucy"}
]
```

```cpp
#include <json/json.h>
#include <fstream>
using namespace Json;

int main()
{
    writeJson();
    readJson();
}
```
---
### 4.3.1 写json文件

```cpp
void writeJson()
{
    // 将最外层的数组看做一个Value
    // 最外层的Value对象创建
    Value root;
    // Value有一个参数为int 行的构造函数
    root.append(12);	// 参数进行隐式类型转换
    root.append(12.34);
    root.append(true);
    root.append("tom");
    
    // 创建并初始化一个子数组
    Value subArray;
    subArray.append("jack");
    subArray.append("ace");
    subArray.append("robin");
    root.append(subArray);
    
    // 创建并初始化子对象
    Value subObj;
    subObj["sex"] = "woman";  // 添加键值对
    subObj["girlfriend"] = "lucy";
    root.append(subObj);
    
    // 序列化
#if 1
    // 有格式的字符串
    string str = root.toStyledString();
#else
    FastWriter f;
    string str = f.write(root);
#endif
    // 将序列化的字符串写磁盘文件
    ofstream ofs("test.json");
    ofs << str;
    ofs.close();
}
```
---
### 4.3.2 读json文件

```cpp
void readJson()
{
    // 1. 将磁盘文件中的json字符串读到磁盘文件
    ifstream ifs("test.json");
    // 2. 反序列化 -> value对象
    Value root;
    Reader r;
    r.parse(ifs, root);
    // 3. 从value对象中将数据依次读出
    if (root.isArray())
    {
        // 数组, 遍历数组
        for (int i = 0; i < root.size(); ++i)
        {
            // 依次取出各个元素, 类型是value类型
            Value item = root[i];
            // 判断item中存储的数据的类型
            if (item.isString())
            {
                cout << item.asString() << ", ";
            }
            else if (item.isInt())
            {
                cout << item.asInt() << ", ";
            }
            else if (item.isBool())
            {
                cout << item.asBool() << ", ";
            }
            else if (item.isDouble())
            {
                cout << item.asFloat() << ", ";
            }
            else if (item.isArray())
            {
                for (int j = 0; j < item.size(); ++j)
                {
                    cout << item[j].asString() << ", ";
                }
            }
            else if (item.isObject())
            {
                // 对象
                // 得到所有的key
                Value::Members keys = item.getMemberNames();
                for (int k = 0; k < keys.size(); ++k)
                {
                    cout << keys.at(k) << ":" << item[keys[k]] << ", ";
                }
            }
            
    	}
        cout << endl;
    }
}
```

在上面读Json文件的这段代码中，对读出的每个Value类型的节点进行了类型判断，其实一般情况下是不需要做这样的判断的，因为我们在解析的时候是明确地知道该节点的类型的。

虽然Json这种格式无外乎数组和对象两种，但是需求不同我们设计的Json文件的组织方式也不同，一般都是特定的文件对应特定的解析函数，一个解析函数可以解析任何的Json文件这种设计思路是坚决不推荐的。

---

# 5. Qt程序打包和发布
## 5.1 程序的发布

### 5.1.1 生成Release版程序
在编写Qt程序的时候，不管我们使用的什么样的IDE都可以进行编译版本的切换，如果要发布程序需要切换为Release版本（Debug为调试版本），编译器会对生成的Release版可执行程序进行优化，生成的可执行程序会更小。这里以QtCreator为例，截图如下：

![在这里插入图片描述](img/b.17.png)


模式选择完毕之后开始构建当前项目，最后找到生成的带`Release`后缀的构建目录，如下图所示：

![在这里插入图片描述](img/b.18.png)

进图到release目录中，在里面就能找到我们要的可执行程序了

![在这里插入图片描述](img/b.19.png)

---
### 5.1.2 发布
生成的可执行程序在运行的时候需要加载相关的Qt库文件，因此需要将这些动态库一并发布给使用者，Qt官方给我们提供了相关的发布工具，通过这个工具就可以非常轻松的找出这些动态库文件了，这个工具叫做`windeployqt.exe`，该文件位于Qt安装目录的编译套件目录的bin目录中，以我本地为例：`C:\Qt\5.15.2\mingw81_64\bin`

- C:\Qt是Qt的安装目录
- 5.15.2是Qt的版本
- mingw81_64是编译套件目录
- bin存储windeployqt.exe文件的目录

如果已经将这个路径设置到环境变量中了，那么在当前操作系统的任意目录下都可以访问`windeployqt.exe`


知道Qt提供的这个工具之后就可以继续向下进行了，首先将生成的`Release`版本的可执行程序放到一个新建的空目录中：
![在这里插入图片描述](img/b.20.png)


进入到这个目录，按住键盘`shift`键然后鼠标右键就可以弹出一个右键菜单
![在这里插入图片描述](img/b.21.png)


打开Powershell窗口执行命令：


```bash
# LordCard.exe 是可执行程序的名字
# windeployqt.exe 的后缀 .exe 可以省略不写
windeployqt.exe LordCard.exe
```

![在这里插入图片描述](img/b.22.png)


这样`LordCard.exe`需要的动态库会被全部拷贝到当前的目录中，如下图：
![在这里插入图片描述](img/b.23.png)


`使用这种方式Qt会将一些用不到的动态库也拷贝到当前的目录中，如果确定用不到可以手动将其删除，如果不在意这些，完全可以不用理会，选择后者。`

现在一个绿色免安装版的程序就得到了，可以将这个目录打个压缩包发送

---
## 5.2 Qt程序打包
将应用程序和相关的动态库打包成安装包的工具有很多，我自己用过两个一个是`NIS Edit`，一个是`Inno Setup`这是一个免费的 Windows 安装程序制作软件，小巧、简便、精美。

[官方下载地址](http://www.jrsoftware.org/isdl.php#stable)

其实这两个工具的使用方法是几乎一样的，下面拿`Inno Setup`使用举例。

**第一步：创建一个带向导的脚本文件**
![在这里插入图片描述](img/b.24.png)


**第二步：直接 Next，不要创建空的脚本文件**

![在这里插入图片描述](img/b.25.png)


**第三步：填写相关的应用程序信息**
![在这里插入图片描述](img/b.26.png)


**第四步：指定应用程序的安装目录相关的信息**
![在这里插入图片描述](img/b.27.png)


**第五步：选择可执行程序和相关的动态库，此处参考的是前边的 1.2 章节中的目录**
![在这里插入图片描述](img/b.28.png)


基于这个目录选择相关的文件和目录：
![在这里插入图片描述](img/b.29.png)


由于可执行程序关系的动态库有很多，所以可以直接添加动态库的目录，选中对应的目录之后，如果里边还有子目录会弹出如下对话框，选择是即可，需要包含这些子目录。
![在这里插入图片描述](img/b.30.png)


**第六步：给可执行程序关联本地的某种格式的磁盘文件（比如记事本程序会自动关联本地的 .txt 文件），对于我的可执行程序来说无需关联，因此没有做任何设置，直接下一步**
![在这里插入图片描述](img/b.31.png)


**第七步：给应用程序创建快捷方式，此处没有进行任何设置，使用的默认选项**
![在这里插入图片描述](img/b.32.png)


**第八步：指定许可文件，文件中的内容会显示到安装向导的相关窗口中，可以选择不指定，直接跳过。**
![在这里插入图片描述](img/b.33.png)


**第九步：选择安装模式（给系统的当前用户安装还是给所有用户安装），根据自己喜好指定即可**
![在这里插入图片描述](img/b.34.png)


**第十步：选择安装语言（这个工具没有提供中文，因此只能选择英文）**
![在这里插入图片描述](img/b.35.png)


**第十一步：指定安装包文件的相关信息**
![在这里插入图片描述](img/b.36.png)


**第十二步：向导结束**
![在这里插入图片描述](img/b.37.png)


**第十三步：提示是否要编译生成的脚本文件，脚本编译完成之后，安装包就生成了。**

![在这里插入图片描述](img/b.38.png)

之后弹出第二个对话框，建议通过向导生成的这个脚本文件，这样以后就可以直接基于这个脚本打包程序生成安装包了。
![在这里插入图片描述](img/b.39.png)


**编译完成之后，就可以去保存脚本文件的目录找生成的安装文件了**
![在这里插入图片描述](img/b.40.png)

---



