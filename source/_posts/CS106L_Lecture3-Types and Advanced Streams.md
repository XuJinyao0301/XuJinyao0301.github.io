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