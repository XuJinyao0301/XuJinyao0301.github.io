---
title: CS106L_Lecture2-Stream
author: XuJinyao
date: 2026-3-26 20:58:09
tags:
- CSDIY
- CS106L
categories:
- Programming
description: 
- 这篇文章讲解了CS106L在Lecture2里讲到的Stream
---
## 什么是Stream

stream直译过来就是“流”

我们可以把它看作是一个数据流动的管道：

数据可以通过键盘输入，流进程序
数据可以通过程序输出，流到屏幕
文件里的数据可以流进程序里
程序里的数据可以流到文件里

常见的stream有 cin 和 cout

## ostringstream

### ostringstream 是什么

ostringstream 可以拆成两部分看：

output：输出
string stream：字符串流

```c++
cout << 123;
```
是把123输出到屏幕

而
```c++
ostringstream oss;
oss << 123;
```
是把123输出到oss这一个变量里面，最后变成一个字符串

例如
```c++
#include <iostream>
#include <sstream>
using namespace std;

int main() {
    ostringstream oss;
    oss << "age = " << 18;

    string s = oss.str();
    cout << s << endl;
}
```
输出结果是
```c++
age = 18
```
这里不是直接打印到屏幕，而是先“攒”成字符串，再取出来。

这里的oss.str()是ostringstream很关键的一个函数
```c++
string s = oss.str();
```
意思是把这个流里面积累的内容全部取出来给字符串s

比如
```c++
ostringstream oss;
oss << "Hello " << "world " << 123;
```
这时候 oss 里面其实已经存着：Hello world 123了

这个东西的作用就在于 面对处理复杂字符串拼接时更加自然。

### ostringstream 的写位置

```c++
ostringstream oss("Ito-En Green Tea");
cout << oss.str() << endl;

oss << "16.9 Ounces";
cout << oss.str() << endl;
```

这段代码的输出结果是
```c++
Ito-En Green Tea
16.9 Ouncesn Tea
```
为什么第二行的结果是这样的呢？

因为ostringstream 有一个“写位置”（put pointer）。
它不会默认跳到字符串末尾去追加，而是从当前写位置开始覆盖写。
在这种构造下，写位置一开始在开头，所以会把原来前面的字符覆盖掉。
但是我们可以用ostringstream::ate把初始写位置定位到末尾

ate 可以理解成 at end
```c++
ostringstream oss("Ito-En Green Tea", ostringstream::ate);
oss << " 16.9 Ounces";
cout << oss.str() << endl;
```
这样结果就会是:
```c++
Ito-En Green Tea 16.9 Ounces
```

## istringstream

### istringstream 是什么

同样istringstream可以分为两部分：

input：输入

string stream：字符串流

```c++
string s = "123";
istringstream iss(s);

int x;
iss >> x;
```
这里不是键盘输入，而是字符串 "123" 充当了输入源。
它的读取方式和 cin 很像，
所以可以把它理解成“假装这个字符串是键盘输入”

### istringstream 的读位置

```c++
#include <iostream>
#include <sstream>
using namespace std;

int main() {
    string line = "101 Jinyao 18";
    istringstream iss(line);

    int id, age;
    string name;

    iss >> id >> name >> age;

    cout << id << "\n";
    cout << name << "\n";
    cout << age << "\n";

    return 0;
}
```
这段代码的输出结果是：
```c++
101
Jinyao
18
```
它内部有一个读指针，读一次，就往后走一次。

istringstream 用 >> 读取时，一般会在这几种情况停下：

第一种：遇到空白字符

空格、换行、tab

第二种：遇到不符合当前类型格式的字符

比如你在读 int，结果遇到了字母

第三种：读到字符串结尾

也就是没有更多内容了

例如
```c++
#include <iostream>
#include <sstream>
using namespace std;

int main() {
    istringstream iss("123 abc 45");

    int x;
    string s;
    int y;

    iss >> x;   // 读到 123，遇到空格停
    iss >> s;   // 读到 abc，遇到空格停
    iss >> y;   // 读到 45，读到结尾停

    cout << x << "\n" << s << "\n" << y << endl;
}
```
这里三次停止分别是：

第一次：遇到空格停
第二次：遇到空格停
第三次：遇到字符串结尾停

## stringToIntegerTest

现在我们来编写一个把字符串转化成整数的函数：
```c++
int stringToInteger(const string& s){
    istringstream iss(s);
    int result;
    iss >> result;
    return result;
}
```
如果我们在这里输入的不是数字的话，该程序的结果就会是一个垃圾值，
但是如果我们输入的字符串开头可被读取整数，例如 123a ，那么程序就会读取123，然后结果就是123了。

## Four bits状态位

### Good bit
Ready for read/write
stream就绪，可执行读写操作
表示stream无错误，可以正常读写

### Fail bit
previous operation failed, all future operations frozen
上一次操作失败，后续所有操作被冻结
通常是格式错误、输入不匹配等可恢复错误

### EOF bit
previous operation reached the end of buffer content
上一次操作已到达缓冲区内容末尾
表示读取到了stream的末尾(End of File)

### Bad bit
external error,likely irrecoverable
发生外部错误，通常不可恢复
多位硬件/系统级错误（如磁盘故障、stream断开）

## buffer stream

### buffer stream 是什么

buffer stream ：characters are stored in an intermediate buffer before being moved to the external source
带缓冲的流：字符在被转移到外部数据源之前，会先存储在中间缓冲区

### buffer stream 的工作流程

```c++
cout << "hello";
```
"hello"会先写进cout的缓冲区
当满足以下条件时，缓冲区才会被刷新（flush），把字符真正输出到控制台
1. 缓冲区被写满
2. 程序执行到 endl ，或者手动调用 flush()
3. 程序正常结束
4. 输入流（cin）被触发（会自动刷新cout缓冲区）

### 为什么要用 buffer

直接频繁地向控制台进行 IO 操作，会非常慢，用缓冲区可以把多次小操作合并成一次大操作，从而大幅提升程序的运行效率

## manipulators

有些操纵符插入到流中，会改变流的行为,
这里我们直接通过代码展示

```c++
#include <iostream>
#include <iomanip>
using namespace std;

int main() {
    // endl ：插入换行符并刷新流
    cout << "Hello" << endl << "World" << endl; // 换行+刷新
    
    // boolalpha ：以 true/false 字符串形式输出布尔值
    cout << boolalpha << true << " " << false << endl; // 输出 true false
    
    // hex ：以十六进制格式输出数字
    cout << hex << 255 << endl; // 输出 ff（十六进制）
    
    // setprecision ：调整输出数字的精度
    cout << setprecision(3) << 3.14159 << endl; // 输出 3.14（3位有效数字）
    cout << fixed << setprecision(2) << 3.14159 << endl; // 输出 3.14（小数点后2位）
    
    return 0;
}
```