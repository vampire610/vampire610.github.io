---
title: 配置VSCode编辑keil的STM32工程
date: 2020-04-02 15:07:02
tags: 
  - stm32
categories: 
  - stm32
  - 技巧
---

keil的代码编辑界面看久了实在是难受，更改配色很麻烦，而且效果也不是很好，所以研究了一下用VSCode来作为keil代码的编辑器，但直接打开的话没有自动补全，本文就着重说一下配置VSCode实现对STM32代码的自动补全。

## VSCode必要插件
C/C++
C++ Intellisense
![vscode插件](https://cdn.jsdelivr.net/gh/vampire610/cdn@1.7/img/STM32/使用VSCode/code插件.png)

## 用VSCode打开keil工程文件夹

## 打开 **c_cpp_properties.json** 文件，参照下图修改配置

![vscode配置](https://cdn.jsdelivr.net/gh/vampire610/cdn@1.7/img/STM32/使用VSCode/code配置.png)

### 这是我的配置文件
``` json
{
    "configurations": [
        {
            "name": "Win32",
            "includePath": [
                "${workspaceFolder}/**",
				/****************/
				//注意这里路径要改成自己的keil安装目录
                "E:/Keil_MDK/Core/ARM/ARMCC/include",
				/*****************/

				//这里下面的都是keil工程所有头文件的包含路径
                "${workspaceFolder}/CORE",
                "${workspaceFolder}/Hardware_Drive",
                "${workspaceFolder}/Normal",
                "${workspaceFolder}/User",
                "${workspaceFolder}/STM32_Libraries"
            ],
            "defines": [
				//这里是宏
                "_DEBUG",
                "UNICODE",
                "_UNICODE",
                "STM32F4xx",
                "USE_HAL_DRIVER",
                "USE_STDPERIPH_DRIVER",
                "ARM_MATH_CM4",
                "__CC_ARM",
                "ARM_MATH_M"
            ],
            "compilerPath": "E:\\MinGW\\bin\\gcc.exe",
            "cStandard": "c11",
            "cppStandard": "c++17",
            "intelliSenseMode": "clang-x86"
        }
    ],
    "version": 4
}

```
上面的头文件路径和宏可以在这里查看，上面红框是宏，下面是头文件路径
![keil路径](https://cdn.jsdelivr.net/gh/vampire610/cdn@1.7/img/STM32/使用VSCode/keil路径.png)


#### 如果有什么疑问，欢迎留言询问
