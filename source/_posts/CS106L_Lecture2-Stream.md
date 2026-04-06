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