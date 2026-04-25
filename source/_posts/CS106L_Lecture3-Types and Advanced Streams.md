---
title: CS106L_Lecture3-Types and Advanced Streams
author: XuJinyao
date: 2026-4-7 20:58:09
tags:
- CSDIY
- CS106L
categories:
- Programming
description: 
- 这篇文章主要记录了CS106L在Lecture3里讲到的 Types and Advanced Streams
---
## cin 的读取
![](/images/cs106l_lec3-01.jpg)

我们可以看到，当我们输入 Avery Wang 时，
在 cin 的缓冲区里，只有 Avery 被读走了，后面还剩 Wang\n
而下一个将要被读取的位置在 W 这里

### 为什么 cin >> name; 只读取出 "Avery"?

因为对于字符串 name 来说，>> 默认是 按空白分隔读取 的
也就是说它会：
跳过前导空白
一直读，直到遇到空格、换行、tab为止

### 为什么 cin >> age; 会失败

在输出结果中，显示的 age 是 0 ，
这是因为 age 是 int，
cin 期待读取到一个整数，
而缓冲区里下一个有效字符是 W ，而不是数字
于是：
cin 无法将 W 解析成 int
从而本次读取失败
fail bit 被设置为 1
所以我们可以看到右边的状态位中，F被点亮了

### The worst

一旦 fail bit 被置上，后续所有的 cin 读取都会直接失败！

## 第一版修复

```
int getInteger(const string& prompt) {
    cout << prompt;
    string token;
    cin >> token;   // still a problem
    istringstream iss(token);
    int result;
    char trash;
    if (!(iss >> result) || iss >> trash)
        return getInteger(prompt); // bad recursion
    return result;
}
```

这里是先把输入读成字符串，再自己解析。
但是仍然存在问题

### 问题 A

cin >> token 还是只读取一个token
例如我们输入
```
20 lol
```
那cin >> token 只会读到
```
token = "20"
```
后面的 lol 还留在原始输入缓冲区里

### 问题 B

```
return getInteger(prompt);
```
这个写法叫 递归重试

它的问题是：

* 输入错误很多次，会一层一层递归下去
* 虽然程序里未必立刻出事，但设计上不优雅
* 重试逻辑本来就更适合 while (true) 循环

## 终版修复

```
int getInteger(const string& prompt) {
    while (true) {
        cout << prompt;
        string line;
        if (!getline(cin, line))
            throw domain_error("...");
        istringstream iss(line);
        int result;
        char trash;
        if (iss >> result && !(iss >> trash))
            return result;
    }
}
```
这里先用 getline 把整行读出来，
再把这一整行去解析

这里 getline(cin, line) 的意思是： 从 cin 这个输入流里，读取一整行，存到 line 里
函数原型是 getline(输入流, 字符串变量);
它还可以从字符串流读
```
istringstream iss("hello\nworld");
string line;
getline(iss, line);
```

## cin 与 getline 的区别

cin >> line
读一个 token，遇到空白就停
getline(cin, line)
读一整行，默认遇到换行符 \n 才停，并且会把这个换行符消费掉，但不会放进结果字符串里。

## 混用 >> 与 getline 出现的 bug
```
istringstream iss("16.9 Ounces\n Pack of 12");
double amount;
string unit;

iss >> amount;
getline(iss, unit);
```

我们会发现：
amount 读出来是对的，但 unit 却不是你以为的 "Ounces" 或 " Ounces"，而是先变成了空字符串

这是因为 >> 和 getline 对“分隔符/空白”的处理方式不同

### iss >> amount 做了什么

假设流里一开始是
```
16.9 Ounces
 Pack of 12
```
执行
```
iss >> amount
```

amount = 16.9
读取位置停在 16.9 后面的那个空格前后附近 的关键位置上

更准确地说，>> 在读 double 时，会把能构成 double 的部分读完，然后停下来，后面的空格/换行还在流里等待后续处理。

### 马上 getline(iss, unit) 会发生什么

getline 的规则是：
从当前位置开始，一直读到换行符 \n 为止。
它不会像 >> 一样先主动跳过前导空白。这一点特别重要。
所以如果当前位置前面正好残留了一个空格，或者更常见的是残留了一个换行符，那么 getline 可能直接读到“空内容”

### 最常见的坑

虽然说在CS106L中用的是 istirngstream, 但最常见的坑应该是在 cin 上，原因和 iss 的情况一样

### 重点1: getline 不会自动跳过前导空白

>>
会先跳过前导空白
getline
不会跳过，它从当前位置直接开始读
所以只要你前面用 >> 留下了换行符，后面紧接一个 getline，就特别容易出问题

### 修复方法
```
iss >> amount;
iss.ignore();
getline(iss, unit);
```

这里加上 iss.ignore(), 再 getline 就可以正常读到"Ounces"

#### ignore() 的作用

可以把它理解成：
手动丢掉流里当前那个你不想要的字符

在这个例子里，就是把前面 >> 留下的那个分隔字符先扔掉，然后再让 getline 从真正想读的位置开始读

一般情况，ignore()指忽略接下来的1个字符

如果是带参数
```
ignore(n, '\n')
```
表示 最多忽略 n 个字符，遇到分隔符 \n 时把该分隔符丢掉后停止