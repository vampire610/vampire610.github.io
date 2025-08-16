---
title: platformio使用技巧
date: 2023-07-06 23:00:08
tags:
  - platformio
categories:
  - 学习笔记
---

# platformio使用自定义第三方库
最近准备学习dengfoc，但是不想使用arduino ide，发现VScode可以搭配platformio可替代。
打开发现dengfoc不在platformio库里，需要自己导入。
导入方法：

在platformio的工程和路径的lib文件夹下建立文件夹，文件夹名和源文件、头文件名字一致。

在lib文件夹下的README文件中有如下例子

```zsh
|--lib

|  |

|  |--Bar

|  |  |--docs

|  |  |--examples

|  |  |--src

|  |   |- Bar.c

|  |   |- Bar.h

|  |  |- library.json (optional, custom build options, etc) https://docs.platformio.org/page/librarymanager/config.html

|  |

|  |--Foo

|  |  |- Foo.c

|  |  |- Foo.h

|  |

|  |- README --> THIS FILE

|

|- platformio.ini

|--src

  |- main.c
```

有两种文件结构方式，第一种在lib文件夹下建立对应名称的文件夹，再建立src文件夹存放自定义库的源文件，src外面可放置文档。

第二种方式为直接建立对应名称文件夹，内放置自定义库的源文件，不放置其他文件或文档。

每一组头、源文件需要一个文件夹，才可以被正常识别。