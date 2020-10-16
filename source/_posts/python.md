---
title: python入门
date: 2020-03-01 15:07:02
tags: 
  - python
categories: 
  - 学习笔记
  - python
---

## python教程  
本人通过《A Byte of Python》自学python基础知识  
英文原版在线阅读:  [A Byte of Python][1]  
中文译本在线阅读:  [简明 Python 教程][2]  
书内附有中英文版下载地址 [中文版下载][3]

## linux环境搭建
linux下python环境通过命令行安装较为简单，不再赘述，主要说明Windows下配合VScode的环境搭建

## Windows环境搭建
### python安装
直接通过[Python官网][4]下载  
本人学习版本为3.x版本，本文所说python也均为3.x版本  
安装时请注意勾选 **Add Python 3.8 to PATH**   
![添加环境变量](https://cdn.jsdelivr.net/gh/vampire610/cdn@1.6/img/python/addPATH.png) 

安装完成后打开cmd，直接输入python后回车，如果出现以下信息，就说明安装成功了
``` cmd
Microsoft Windows [版本 10.0.18363.657]
(c) 2019 Microsoft Corporation。保留所有权利。

C:\Users\vampi>python
Python 3.8.2 (tags/v3.8.2:7b3ab59, Feb 25 2020, 23:03:10) [MSC v.1916 64 bit (AMD64)] on win32
Type "help", "copyright", "credits" or "license" for more information.
>>>
```

### VScode运行python配置
* 安装和美化VScode本文不再赘述，只描述python相关的配置、操作
* 安装python插件
  * 打开VScode  
  * "Ctrl + Shift + X"或者鼠标点击打开扩展管理器
  * 输入python搜索、安装插件
* 配置VScode终端为git bash
  * 首先要确保已经安装git，能够正常使用bash
  * 打开settings.json
  * 在配置中加入如下行，记得将bash路径改为自己的gitbash路径
  ``` json
  "terminal.integrated.shell.windows": "E:\\Git\\bin\\bash.exe",
  ```
  * 保存，重新打开VScode终端，就能在VScode使用gitbash，相比原本的windows的cmd命令，gitbash的linux终端命令应该对大多数人更加友好


[1]:https://python.swaroopch.com/
[2]:https://bop.mol.uno/
[3]:https://legacy.gitbook.com/book/lenkimo/byte-of-python-chinese-edition/details
[4]:https://www.python.org/downloads/windows/
