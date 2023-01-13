---
title: AD的一些注意事项
date: 2019-11-02 21:07:17
tags: 
  - AD
categories: 
  - 教程
  - AD

---

# AD的一些使用技巧

## 重新生成(Updata PCB Document xxx.PcbDoc)PCB图时出现莫名其妙的错误的解决办法  

>例如全图变绿等  
>可能原因：未删除 **网络表** 和 **类**

## 解决办法  

#### 第一步：

>在XXX.pcbdoc文件里点击清除全部网络，如下图所示  
>![清除网络](https://cdn.jsdelivr.net/gh/vampire610/cdn@1.5/img/AD/clear.png)
>
>***

#### 第二步：

>删除对应的对象类，找到与自己的原理图相同名字的选项（如我的是sheet1），删掉它

![删除对象类，图1](https://cdn.jsdelivr.net/gh/vampire610/cdn@1.5/img/AD/sheet.png)  
图1  
![删除对象类，图2](https://cdn.jsdelivr.net/gh/vampire610/cdn@1.5/img/AD/sheet1.png)  
图2  
![删除对象类，图3](https://cdn.jsdelivr.net/gh/vampire610/cdn@1.5/img/AD/sheet2.png)  
图3  

## 做完以上步骤之后，再回到原理图文件重新生成PCB图即可