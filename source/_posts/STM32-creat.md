---
title: 创建、移植STM32程序注意事项
date: 2020-04-07 11:06:02
tags: 
  - stm32
categories: 
  - stm32
  - 技巧
---

太长时间没有创建过新的工程，好多基础配置都不记得了，最近做毕设打算从0开始完全自己搞定，又踩了一遍坑，做一下记录。

----
1. 构建文件目录

2. 创建keil工程
	1. 确认单片机型号
	2. 添加C文件、启动文件
	3. 添加头文件路径
	4. 添加全局宏定义标识符  
	STM32F4常用标识符  ` STM32F40_41xxx,USE_STDPERIPH_DRIVER `

3. 系统时钟配置
  修改 `System_stm32f4xx.c`文件的PLL第一级分频系数M

  ``` c
  /************************* PLL Parameters *************************************/
  #if defined (STM32F40_41xxx) || defined (STM32F427_437xx) || defined (STM32F429_439xx) || defined (STM32F401xx)
  /* PLL_VCO = (HSE_VALUE or HSI_VALUE / PLL_M) * PLL_N */
  #define PLL_M      8  /* 修改这里 */
  #else /* STM32F411xE */
  #if defined (USE_HSE_BYPASS)
  #define PLL_M      8    
  #else /* STM32F411xE */   
  #define PLL_M      16
  #endif /* USE_HSE_BYPASS */
  #endif /* STM32F40_41xxx || STM32F427_437xx || STM32F429_439xx || STM32F401xx */  
  ```

  修改` stm32f4xx.h `里面修改外部时钟` HSE_VALUE `值为 8MHz，因为我们的外部高速时钟用的晶振为8M

  ``` c
  #if !defined  (HSE_VALUE) 
    #define HSE_VALUE    ((uint32_t)8000000) /*!< Value of the External oscillator in Hz */
    
  #endif /* HSE_VALUE */
  ```

  

