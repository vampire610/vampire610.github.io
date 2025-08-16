---
title: vscode-keil
date: 2023-01-14 20:10:13
tags: 
  - vscode
  - keil
  - 单片机
categories: 
  - 教程
---

### 安装keil-MDK

- 官网下载安装，破解方法自行百度

- 新版keil编译器版本为6，旧版本工程可能在新版keil无法编译通过，故需要安装ARMCompiler 5

  - ARMCompiler 5官网[下载地址](https://developer.arm.com/downloads/view/ACOMP5)，网盘 [下载地址](https://pan.baidu.com/s/1i1VNFKQ_CJm__A18kUTgVA)提取码：vbv3
    
  - 注意需要安装到keil的ARM文件夹下，我的电脑路径为`C:\Keil_v5\ARM`，否则仍会报错，显示license问题。

### 配置vscode

- 必备插件
  - [C/C++](https://marketplace.visualstudio.com/items?itemName=ms-vscode.cpptools)
  - [Keil Assistant](https://marketplace.visualstudio.com/items?itemName=CL.keil-assistant)（已停止更新，截至2023-1-14还能正常使用）

- 一些推荐
  - 编程字体[Hack](https://sourcefoundry.org/hack/)
  - VScode主题[One Dark Pro](https://marketplace.visualstudio.com/items?itemName=zhuangtongfa.Material-theme)



### 使用方法

- **注意**：该插件仅通过读取keil配置文件获取工程内容、文件路径等，故在使用前需要保证工程在keil已经配置完成并且能够编译通过，且之后添加文件、头文件路径也需在keil完成

- 使用vscode打开工程

  - 若上述步骤已完成，则在vscode的资源管理器最下方能看到`KEIL UVISION PROJECT`

  - 点击该行后方按钮打开keil工程文件

    ![屏幕截图 2023-01-14 204432](https://fastly.jsdelivr.net/gh/vampire610/PicGo/Blog-PIC/20230114204618.png)

  - 打开工程后右下角会提示切换工作空间，点击OK即可，不要试图自己打开文件夹，否则会无法识别头文件，无法自动补全等。猜测解决办法，调整keil工程结构目录。

    ![image-20230114205315916](https://fastly.jsdelivr.net/gh/vampire610/PicGo/Blog-PIC/20230114205341.png)

  - 之后即可在下方打开文件愉快的用vscode写keil代码了，自动补全等功能都正常

    ![image-20230114205641535](https://fastly.jsdelivr.net/gh/vampire610/PicGo/Blog-PIC/20230114205646.png)





### 一些注意事项

- 如果点击编译、下载后出现终端界面，但没有其他反应，可将VScode默认终端换成powershell等windows自带终端。
