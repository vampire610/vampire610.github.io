---
title: 正点原子开发板开发小项目记录
date: 2023-03-15 22:29:34
tags: 
  - STM32
categories: 
  - 学习笔记
---

# STM32+DH11温度传感器+spi显示屏移植调试记录

## 硬件资源

- 正点原子探索者开发板

- 正点原子1.3寸LCD屏幕
- DH11温湿度传感器

## 软件资源

- MDK-ARM
- STM32CubeMX
- VOFA+上位机
- VSCode+keil Assistant插件

> 除mdk外均为免费软件

## 准备工作

- 安装软件，VOFA+是一个上位机软件，可以方便的用来做串口调试，vscode+keil Assistant插件可载入keil工程，且不用自行配置头文件路径，可以自动补全，还能在vscode内编译，烧录代码。cubeMX可以快速配置配单片机初始化。
- CubeMX安装器件包
![image-20230228183216190](https://fastly.jsdelivr.net/gh/vampire610/PicGo/Blog-PIC/20230228183223.png)