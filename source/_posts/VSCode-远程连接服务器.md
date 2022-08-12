---
title: VSCode 远程连接服务器
date: 2022-06-30 22:41:00
tags: 
- VSCode
- SSH
categories: 教程
mathjax: true
---

宇宙第一IDE，当属 VSCode。作为一名高级炼丹师，怎么能不会远程连接服务器进行开发呢。接下来就讲述如何使用 VSCode 远程连接服务器进行炼丹操作。

<!-- more -->

## 一、安装

### step 1：安装 Anaconda

前往[Anaconda](https://www.anaconda.com/products/individual)官网，下载对应版本Anaconda安装包。

![anaconda安装](VSCode-%E8%BF%9C%E7%A8%8B%E8%BF%9E%E6%8E%A5%E6%9C%8D%E5%8A%A1%E5%99%A8/anaconda%E5%AE%89%E8%A3%85.jpg)

安装包下载完成后，进行安装，记得自己Anaconda的安装路径。

### step2：安装 VSCode

前往[Visual studio code](https://code.visualstudio.com/)页面进行下载安装。

![vscode安装](VSCode-%E8%BF%9C%E7%A8%8B%E8%BF%9E%E6%8E%A5%E6%9C%8D%E5%8A%A1%E5%99%A8/vscode%E5%AE%89%E8%A3%85.png)

### step3：在VSCode中安装插件

【在 VSCode 插件商店中搜索 Python、Jupyter、Remote-SSH，选择插件并点击 install 安装程序】

![Python插件安装](./VSCode-远程连接服务器/Python插件安装.jpg)

![jupyter插件安装](./VSCode-远程连接服务器/jupyter插件安装.jpg)

![ssh插件安装](./VSCode-远程连接服务器/ssh插件安装.jpg)

## 二、远程连接服务器

### step1：Remote-SSH 连接配置

【打开SSH TARGETS连接服务器】输入ssh 用户名@+服务器IP

![远程连接](./VSCode-远程连接服务器/远程连接.jpg)

【选 C:\用户名\\.ssh\config 进行配置】

![连接配置](./VSCode-远程连接服务器/连接配置.jpg)

【修改 config 配置文件】Host后面为你的服务器备注（随意命名），HostName 为连接的服务器 ip 地址，User 为服务器用户名

![config](./VSCode-远程连接服务器/config.jpg)

### step2：SSH  插件设置

【勾选 Remote-SSH 设置中的 show login terminal】以便我们可以在 Terminal 中观察到我们的Connect Information

![设置](./VSCode-远程连接服务器/设置.jpg)

![ssh设置](./VSCode-远程连接服务器/ssh设置.jpg)

### step3：连接服务器

【连接刚刚建好的 SSH Target】

![连接](./VSCode-远程连接服务器/连接.jpg)

【选择服务器对应的系统平台】

![连接平台](./VSCode-远程连接服务器/连接平台.jpg)

【输入 服务器密码 进行登陆】

![输入密码](./VSCode-远程连接服务器/输入密码.jpg)

左下角 蓝色 为连接成功，通过 Terminal 进行 conda 操作

![终端](./VSCode-远程连接服务器/终端.jpg)

## 三、配制 jupyter 环境

### step1：在 VSCode 中选择 Python 环境

【选择自己配置好的 conda 环境】这里选择事先安装好的虚拟环境

![选择编译器](./VSCode-远程连接服务器/选择编译器.jpg)

![选择conda](./VSCode-远程连接服务器/选择conda.jpg)

### step2：文件操作

【文件 - 打开文件夹】即可对服务器下文件进行操作

![文件夹选取](./VSCode-远程连接服务器/文件夹选取.jpg)

### step3：代码运行测试，成功！

![成功](./VSCode-远程连接服务器/%E6%88%90%E5%8A%9F.jpg)
