---
title: keil编译报错
date: 2023-08-04 23:18:57
tags:
  - 单片机
categories:
  - 踩坑日记
---

之前突然遇到keil无法编译STM32F7的单片机代码，报错`startup_stm32f767xx.s: error: A3903U: Argument 'Cortex-M7.fp.sp' not permitted for option 'cpu'.`

![81f2b1ca4cafd11dff44518e77a6b72f](https://fastly.jsdelivr.net/gh/vampire610/PicGo/Blog-PIC/20230804232539.png)

在网上搜了很久，只找到一条让升级keil的解决方法，但我使用的keil为5.38，已经是最新的版本，百思不得其解。重装软件，器件包都试过好多次了，最后发现是因为`Arm_Complier_V5`编译器版本过旧，我自己电脑该编译器的文件日期是2013年，而我单位电脑能正常编译的编译器文件日期问2020年。

于是怀疑是编译器过于老旧导致无法编译。

在Arm网站可下载最新版本。[Download Arm Compiler 5](https://developer.arm.com/documentation/ka005184/latest)
