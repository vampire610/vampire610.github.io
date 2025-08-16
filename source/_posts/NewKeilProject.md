---
title: STM32单片机keil工程新建记录
date: 2023-02-15 20:52:16
tags: 
  - keil
  - 单片机
categories: 
  - 记录
---

# 寄存器版本工程新建记录

## 1、新建工程文件夹

| 文件夹名    | 作用                                               |
| :---------- | -------------------------------------------------- |
| Drivers     | 存放于硬件相关的驱动层文件                         |
| Middlewares | 存放正点原子提供的中间层组件文件和第三方中间层文件 |
| Output      | 存放工程编译输出文件                               |
| Projects    | 存放MDK工程文件                                    |
| User        | 存放用户编写的代码，如main.c                       |

| Drivers | 作用                                                        |
| ------- | ----------------------------------------------------------- |
| BSP     | 存放开发板板级支持包驱动代码，如各种外设驱动                |
| CMSIS   | 存放CMSIS底层代码，如启动文件、stm32f4xx.h等                |
| SYSTEM  | 存放正点原子系统级核心驱动代码，如sys.c、delay.c和usart.c等 |

## 2、拷贝工程相关文件

- CMSIS：从现有工程直接拷贝
- Output：在MDK设置路径即可
- Projects：存放编译器工程文件，用MDK则新建`MDK-ARM`文件夹

## 3、新建工程

- 新建工程，选择路径为`Projects/MDK-ARM`
- 选择芯片
- 跳过`Manage Run-Time Environment`不选择组件
- 删除自动创建的`MDK-ARM/Listings`和`MDK-ARM/Objects`，之后统一改到Output

## 4、工程内添加文件

### 4.1、设置工程名和分组名

![image-20230215233759790](https://fastly.jsdelivr.net/gh/vampire610/PicGo/Blog-PIC/20230215233804.png)

### 4.2、添加启动文件

双击软件左侧文件夹或者进入Manage界面，添加启动文件

![image-20230216113209306](https://fastly.jsdelivr.net/gh/vampire610/PicGo/Blog-PIC/20230216113213.png)

### 4.3、添加SYSTEM源码

![image-20230216113858841](https://fastly.jsdelivr.net/gh/vampire610/PicGo/Blog-PIC/20230216113900.png)

## 5、keil设置

点击魔术棒

- Target页面选择编译器，部分旧工程可能无法使用AC6，且AC6对中文不够友好

- Output页面更改输出文件夹为Output

- 按需勾选生成hex文件，设置输出浏览信息

- Listing页面更改输出文件夹为Output

- C/C++页面

  - 设置全局宏定义
  - 选择-O0优化等级
  - 设置c99模式
  - 设置头文件路径。头文件使用相对路径，可以减少MDK工程的头文件路径设置

  ![image-20230216115214037](https://fastly.jsdelivr.net/gh/vampire610/PicGo/Blog-PIC/20230216115216.png)

- Debug页面

  ![image-20230216120514361](https://fastly.jsdelivr.net/gh/vampire610/PicGo/Blog-PIC/20230216120516.png)

- Utilities页面

  ![image-20230216130405066](https://fastly.jsdelivr.net/gh/vampire610/PicGo/Blog-PIC/20230216130410.png)

## 6、编写代码

使用串口助手记得安装CH340驱动，连接上CH340芯片再装，不然可能会安装失败，串口助手推荐[VOFA+](https://www.vofa.plus/)

# HAL库新建，详情见正点原子开发指南
