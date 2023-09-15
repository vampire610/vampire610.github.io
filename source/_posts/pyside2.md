---
title: pyside2的一点使用记录
date: 2023-09-15 21:33:16
tags:
  - python
  - pyside2
  - Qt
categories:
  - 学习笔记
---

最近折腾pyside2，写了一个简单的串口上位机，遇到了一些奇怪的问题，记录一下。

## 环境搭建

python环境我习惯了用anaconda来进行管理，安装各种包都很方便，也不用担心包的版本和python版本不匹配。不过anaconda体积庞大，也可以使用`miniconda`，使用`conda`命令来管理python环境。

pyside2和pyqt5一样基于qt5，搭配python3.8还能支持win7，所以我选择了安装python3.8和pyside2，再加上上位机需要打包，所以还需要安装`pyinstaller`。



anaconda使用十分简单，都是图形化操作，网上教程很多，选择需要的python版本建立环境，再安装需要的python包就可以。

这里整理一下使用`miniconda`的常用命令。

使用`miniconda`建议使用安装后菜单里的`Anaconda Powershell Prompt`或者`Anaconda Prompt`，安装conda后会自动配置这两个终端的环境，其他终端可能需要自己配置，这里就不提了。

- 创建环境(`myenv`为创建的环境名称)

``` bash
conda create --name myenv
```

- 创建指定python版本的环境，如需要3.7.9

``` bash
conda create --name myenv python=3.7.9
```

- 删除环境

``` bash
conda env remove --name myenv
```

- 切换到conda环境

``` bash
conda activate myenv
```

- 查询环境的python版本号

``` bash
python --version
```

- 退出当前conda环境

``` bash
conda deactivate
```

- 查询电脑中现存的conda环境

``` bash
conda info --envs
```

- 添加渠道（如conda-forge）

``` bash
conda config --env --add channels conda-forge
```

- 查看当前环境配置的渠道

``` bash
conda config --show channels
```

- 删除渠道

``` bash
conda config --remove channels channel-name
```

- 安装`python`包（需要使用`conda activate`切换到需要的环境再安装，下面的命令是安装`pyside2`）

``` bash
conda install pyside2
```

- 卸载python包

``` bash
conda remove package-name
```

如果想要安装的包不在`conda`默认`channels`或者`conda-forge`，可能需要使用`pip`安装，或者到官方网站`[:: Anaconda.org](https://anaconda.org/)`搜索需要的包在哪个channels，添加channels后再进行安装。

## IDE选择

### `VScode`

简单谢谢可以直接用`vscode`，安装python插件，`PySide2-VSC`插件，就可以简单的写qt界面了。

- 对`PySide2-VSC`插件进行配置

  > 设置designer路径(不带引号)，designer在`anaconda3`的安装路径下，可以参照我的路径`"XXXX\anaconda3\envs\gui\Scripts\pyside2-designer.exe"`
  >
  >   
  >
  > 设置uic路径(不带引号)
  >
  > `"XXXX\anaconda3\envs\gui\Scripts\pyside2-uic.exe"`

- 正常来说设置完这两项就够用了，接下来就可以在`vscode`的资源管理器中右键，点击`PySide2: New Form in Designer/Creator`打开`QtDesigner`设计你的`ui`，保存后在`xxx.ui`文件上点右键，点击`PySide2: Compile Form to Python`即可生成界面的`python`代码。

- 注意，有时会有使用`pyside2-uic`转换后文件内为`C++`代码的情况，此时可以在插件设置中配置`uic`命令`-g python`这样再生成就是`python`代码了。

### `pycharm`

如果习惯了`pycharm`或者`pycharm community`，也可以配置`pycharm`。

创建项目后右下角点击进入解释器设置，点击`Python`解释器，显示全部，即可管理`pycharm`中的解释器。如果没有`conda`创建的python环境，可以点击左上角+号，选择`Conda`环境，选择`Conda`可执行文件，例如`XXXX\Anaconda3\condabin\conda.bat`，然后加载环境，选择使用现有环境，就可以选择`conda`创建的python环境了。

配置`pycharm`外部工具

在设置中打开外部工具配置界面，添加一项`designer`，参数如下

程序：`XXXX\anaconda3\envs\gui\Scripts\pyside2-designer.exe`

工作目录：`$FileDir$`

添加一项`uic`，参数如下

程序：`XXXX\anaconda3\envs\gui\Scripts\pyside2-uic.exe`

实参：`$FileName$ -o $FileNameWithoutExtension$_ui.py`

工作目录：`$FileDir$`

如果生成代码为C++，将实参改为：`-g python $FileName$ -o $FileNameWithoutExtension$_ui.py`

设置好外部工具，就可以像`vscode`一样通过右键打开`designer`或者将`ui`文件转换成`py`文件。

