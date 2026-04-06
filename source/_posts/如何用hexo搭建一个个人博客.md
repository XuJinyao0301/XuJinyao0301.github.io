---
title: 如何用hexo搭建一个个人博客
author: XuJinyao
date: 2025-10-29 16:23:25
tags:
  - Hexo
categories:
  - Notes
description: 
- 记录使用 Hexo 和 Butterfly 搭建个人博客的基本过程与配置。
permalink: /blog/how-to-build-a-personal-blog-with-hexo/ 
---
## 1.前言  
  
其实关于博客的搭建我花了不少时间，在网上搜集查找了很多文章，期间还向AI请教了一下，前前后后差不多有两天吧，问题主要出现在了一些文件的配置上，所以借此机会，整理一下自己的经验，希望能帮助到大家。  

  
## 2.准备工作  
  
由于我使用的是Windows系统，所以以下操作均在Windows环境下进行。  

### 2.1 安装Node
打开[Node官网](https://nodejs.org/en/download/)，下载与你的系统适配的安装程序，版本的话可以是选择低一点的，不然可能会出现不兼容的问题。  
安装完成后记得检查一下是否安装成功。键盘按下win+R，输入cmd，打开命令行窗口，输入以下命令：

```bash
node -v
```

如果出现版本号，则说明安装成功。

### 2.2 安装Git
  
1. 打开[Git官网](https://git-scm.com/downloads)下载安装程序。  
2. 点击电脑开始菜单，我们可以看见<mark>Git Bash</mark>。等会我们会一直用到它。

### 2.3 安装Hexo

1. 在<mark>Git Bash</mark>中输入以下命令：

```bash
npm install -g hexo-cli
```

2. 安装完成后，输入以下命令：

```bash
hexo -v
``` 

如果出现版本号，则说明安装成功。

### 2.4 创建Github仓库(首先要确保你已经注册了Github账号)

1. 打开[Github](https://github.com/)，点击右上角的<mark>+ New repository</mark>，创建一个<mark><用户名>.github.io</mark>的仓库，其中<用户名>是你的Github用户名。  
2. 勾选<mark>Initialize this repository with a README</mark>，然后点击<mark>Create repository</mark>。  

## 3.配置用户名和邮箱

在<mark>Git Bash</mark>中输入以下命令：

```bash
git config --global user.name "你的用户名"
git config --global user.email "你的邮箱"
```

通过
```bash
git config -l
```
检查一下是否配置成功。

## 4.连接Github仓库

1. 进入任意文件夹，打开<mark>Git Bash</mark>。  
输入以下命令：

```bash
ssh-keygen -t rsa -C "你的邮箱"
```
然后回车，一路回车，直到生成完毕。
然后进入C:\Users\用户名，在里面进入.ssh文件
用记事本打开里面的id_rsa.pub,全选复制里面的代码

2. 回到Github，点击右上角的<mark>Settings</mark>，然后点击<mark>SSH and GPG keys</mark>，点击<mark>New SSH key</mark>，把刚才复制的代码粘贴进去，点击<mark>Add SSH key</mark>。

3. 回到<mark>Git Bash</mark>测试是否成功，输入以下命令：

```bash
ssh -T git@github.com
```
回车，然后输入yes

## 5.本地生成博客

在喜欢的位置新建一个文件夹，然后在<mark>Git Bash</mark>中输入以下命令：

```bash
hexo init
```
如果‘command not find’，就在前面加上npx，比如：
```bash
npx hexo init
```
然后
```bash
hexo install
```
再依次输入以下命令：

```bash
hexo g
hexo s
```
如果不成功就看看是不是梯子忘记关了，或者问问AI。

然后就可以复制生成的链接，在浏览器中打开，就可以看到自己的博客了。

接着回到命令行，Ctrl+C停止hexo s。

## 6.部署博客

进入到博客文件夹，用VSCode打开_config.yml
拉到最下面，将deploy下面的删掉，复制粘贴以下代码：

```yaml
  type: git
  repo: 
  branch: master
```
把你在Github上创建的仓库地址粘贴到repo后面，然后保存(记得有一个空格)。然后保存退出。

回到博客文件夹，输入以下命令安装自动部署发布工具：

```bash
npm install hexo-deployer-git --save
```

再输入
```bash
hexo g
hexo d
```

如果你是第一次部署，会提示你输入Github的用户名和密码
在命令行中输入
```bash
git config --global user.email "你的邮箱"
git config --global user.name "你的名字"
```

然后再输入
```bash
hexo d
```
---
就完成了，打开浏览器输入你的博客地址，就可以看到自己的博客了。网址就是你在Github上创建的仓库地址
https://<用户名>.github.io/。
至此，我们就完成了最基本的博客搭建，虽然看起来有点简陋。。。。。
关于Vercel部署与自定义域名还有美化博客的教程，可以看看https://www.fomal.cc/categories/%E9%AD%94%E6%94%B9%E6%95%99%E7%A8%8B/
当然你也可以去B站、知乎等平台上找找别人的教程。
