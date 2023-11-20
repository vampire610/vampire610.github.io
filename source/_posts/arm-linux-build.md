---
title: 正点原子mp157linux出厂内核源码编译
date: 2023-11-20 14:34:26
tags:
  - linux
  - arm
---

# 正点原子mp157linux出厂内核源码编译

## arm交叉编译工具安装

https://developer.arm.com/tools-and-software/open-source-software/developer-tools/gnu-toolchain/gnu-a/downloads

下载`x86_64 Linux hosted cross compilers`下的`AArch32 target with hard float (arm-none-linux-gnueabihf)`

``` bash
sudo mkdir /usr/local/arm
sudo cp gcc-arm-9.2-2019.12-x86_64-arm-none-linux-gnueabihf.tar.xz /usr/local/arm/ -f
sudo tar -vxf gcc-arm-9.2-2019.12-x86_64-arm-none-linux-gnueabihf.tar.xz
sudo vi /etc/profile
```

在打开的文件最后添加`export PATH=$PATH:/usr/local/arm/gcc-arm-9.2-2019.12-x86_64-arm-none-linux-gnueabihf/bin`

## 安装相关库

``` bash
sudo apt-get install lsb-core 
sudo apt-get install lib32stdc++6
sudo apt-get install lzop
sudo apt-get install libssl-dev
sudo apt-get install u-boot-tools
sudo apt-get install build-essential
sudo apt-get install libncurses5-dev
sudo apt-get install bison 
sudo apt-get install flex
```

## 解压源码

``` bash
tar -vxjf linux-5.4.31-gb8d3ec3ac-v1.1.tar.bz2
//解压缩
```

## 创建shell脚本`stm32mp157d_atk.sh`

以下文件需要特别注意空格位置

``` shell
#!/bin/sh
# 按需清理
# make ARCH=arm CROSS_COMPILE=arm-none-linux-gnueabihf- distclean
make ARCH=arm CROSS_COMPILE=arm-none-linux-gnueabihf- stm32mp1_atk_defconfig
make ARCH=arm CROSS_COMPILE=arm-none-linux-gnueabihf- menuconfig
make ARCH=arm CROSS_COMPILE=arm-none-linux-gnueabihf- uImage dtbs LOADADDR=0XC2000040 -j16
```

## 修改权限运行

编译前建议将终端全屏，终端过小可能无法打开menuconfig

``` bash
chmod 777 stm32mp157d_atk.sh
//给予可执行权限
./stm32mp157d_atk.sh
//执行编译脚本
```

