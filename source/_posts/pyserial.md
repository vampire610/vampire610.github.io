---
title: python串口程序记录
date: 2023-06-17 20:01:51
tags:
  - PyQt5
  - PySide2
categories:
  - 学习笔记
  - PyQt5
  - PySide2
---

# python串口编程记录



## 起步

> 初次尝试python+qt读取串口内容，简单实现了一下，后续有时间再完善
>
> 代码地址：[gitee](https://gitee.com/vampire610/python-serial-port.git)  [github](https://github.com/vampire610/python-serial-port.git)

python使用串口需要导入模块`serial`

如果还需读取串口列表等信息，则还需要导入`serial.tools.list_ports`

由于串口读取会阻塞UI界面，需要引入线程`threading`

- 具体导入模块参考如下

``` python
from PySide2.QtWidgets import QApplication, QWidget, QMessageBox
from window_sdie2 import Ui_Form
import threading
import serial
import serial.tools.list_ports
import sys
```

- python主要部分

``` python
if __name__ == '__main__':
    # qt必要步骤 创建实例
    app = QApplication([])
    # qt必要步骤 创建窗口实例
    mainw = MyWindow()
    # qt必要步骤 显示运行创建的窗口
    mainw.show()
    # 退出程序 若需要打包成EXE，则需要调用sys函数
    # 若不需要打包，则直接使用app.exec_()即可
    sys.exit(app.exec_())
```

- qt界面相关内容

``` python
class MyWindow(QWidget):
    def __init__(self):
        super().__init__()
        # 创建窗口UI实例
        self.ui = Ui_Form()
        # 将qtdesigner配置的界面属性设置给自己创建的窗口
        self.ui.setupUi(self)

        # 获取当前系统可使用的串口端口信息
        port_list = list(serial.tools.list_ports.comports())
        # 将获取到的串口信息循环填充入UI界面的combobox
        for i in port_list:
            self.ui.comboBox.addItem(str(i))
        # 连接信号，当combobox被更改的时候，连接到chuankou函数，进行串口相关操作
        self.ui.comboBox.currentIndexChanged.connect(self.chuankou)

    def chuankou(self):
        # 该函数内进行串口操作
        # 读取当前选择的串口信息
        scom = self.ui.comboBox.currentText()
        # 将读取的串口信息截断，只留下前3个字符，即为端口号
        # 该方法只能在端口号只有1位时使用
        # 同时将获取的端口号强转位字符串格式
        scom = str(scom[0:4])
        print(scom)
        # 打开串口，并创建串口实例
        ser = serial.Serial("COM5", 115200)
        # 判断串口是否打开
        if ser.isOpen():
            print("串口", ser.name, "打开成功")
        else:
            QMessageBox.about(self.ui, '发生错误', "串口打开失败")
        # 创建线程实例，同时填入参数ser和ui
        myTread = CThread(ser, self.ui)
        # 设置线程为守护线程，当主进程退出时会强制结束进程，避免死循环进程残留
        myTread.daemon = True
        # 启动线程
        myTread.start()
```

- 串口线程相关

``` python
class CThread(threading.Thread):
    # 线程类定义
    # 定义线程入口参数ser，ui
    def __init__(self, ser, ui):
        threading.Thread.__init__(self)
        # 创建ser和ui
        self.ser = ser
        self.ui = ui
    
    # 定义进程函数
    def run(self):
        # print("开始线程")
        # srun(self.ser, self.ui)
        # print("退出线程")
        # 串口读取进程死循环
        while True:
            # 阻塞读取串口ser7字节
            data = self.ser.read(7)
            # 将读到的字符串转为浮点型
            data = float(data[0:5])
            # 将值更新到ui界面
            self.ui.lcdNumber.display(data)
            print(data)

```

界面展示

![image-20230617200854179](https://fastly.jsdelivr.net/gh/vampire610/PicGo/Blog-PIC/20230617200902.png)
