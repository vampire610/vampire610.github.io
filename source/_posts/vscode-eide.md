---
title: vscode-eide
date: 2025-01-03 22:25:21
tags: 
  - vscode
  - eide
  - 单片机
categories: 
  - 教程
---
# vscode+EIDE+cubemx配置stm32编程环境

所有安装路径不建议有中文，避免出现奇怪的问题。

## 1. 自行安装vscode[Visual Studio Code - Code Editing. Redefined](https://code.visualstudio.com/)

   ### 安装vscode插件

   必要插件：  
   C语言支持：  
   C/C++  
   C/C++ Themes  
   单片机IDE：  
   Embedded IDE  
   单片机调试插件：  
   Cortex-Debug  
   debug-tracker-vscode  
   RTOS查看：   
   RTOS Views  
   内存查看：  
   MemoryView   
   外设寄存器查看：  
   Peripheral Viewer     



   推荐插件:  
   Chinese (Simplified) (简体中文) Language Pack for Visual Studio Code   
   CodeGeeX  （AI辅助）  
   Hex Editor  （16进制文件查看）  
   Todo Tree  （Todo插件）  
   Serial Monitor （简易串口助手）  
   Keil Assistant  （加载keil工程）    



## 2. 安装arm编译工具链

   EIDE自带版本过旧，建议手动安装。

   官方下载地址：[Arm GNU Toolchain Downloads – Arm Developer](https://developer.arm.com/downloads/-/arm-gnu-toolchain-downloads)

   下载`arm-gnu-toolchain-14.2.rel1-mingw-w64-i686-arm-none-eabi`的exe安装或者zip自行解压，记住安装路径备用。 

## 3. 安装openocd

   可以使用EIDE直接安装，自行安装可自行下载[官方版本](https://github.com/openocd-org/openocd/releases)，解压即可。

   ![EIDE安装openocd](https://fastly.jsdelivr.net/gh/vampire610/PicGo/Blog-PIC/image-20250102135528094.png)

## 4. 安装stm32cubemx

   自行去[ST官网](![image-20250102140748713](eide配置.assets/image-20250102140748713.png))下载`STM32CubeMX-Win`安装。

## 5. 配置EIDE

   配置工具链：
   查看图中2的位置是否有`√`,如果没有则单击，选择本地安装位置，选择路径到刚才安装的编译工具链的`/bin`文件夹，示例如图。
   ![配置工具链](https://fastly.jsdelivr.net/gh/vampire610/PicGo/Blog-PIC/image-20250102135937468.png)

   配置openocd

   如果使用EIDE安装则无需额外配置，如果手动安装则需要点击`打开插件设置`，找到openOCD路径设置，填入`openocd.exe`的绝对路径。

   ![image-20250102140609988](https://fastly.jsdelivr.net/gh/vampire610/PicGo/Blog-PIC/image-20250102140609988.png)

## 6. 创建并导入配置工程

   使用cubeMX创建单片机工程，选择cubeIDE版本。

   使用vscode打开工程文件夹(.ioc所在文件夹)

   选择EIDE插件->点击导入->eclipse，选择.cproject文件。

   打开后会提示切换工作区，一定要选择继续

   ![image-20250102141355916](https://fastly.jsdelivr.net/gh/vampire610/PicGo/Blog-PIC/image-20250102141355916.png)

​	此时会有一个warning.txt，一般可以不管，或者删除。

​	选择编译器和烧录配置

​	一个例子：我使用GCC编译，单片机为`Cortex-M3`内核，使用openOCD调试程序，芯片配置为stm32f1，接口配置我使用daplink。

​	![image-20250102141613636](https://fastly.jsdelivr.net/gh/vampire610/PicGo/Blog-PIC/image-20250102141613636.png)

​	生成调试配置：以此点击，直接新建。

​	![image-20250102141937669](https://fastly.jsdelivr.net/gh/vampire610/PicGo/Blog-PIC/image-20250102141937669.png)

## 7. 基本配置到此结束，如果遇到问题可以带图留言讨论。