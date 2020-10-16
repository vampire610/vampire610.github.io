---
title: 通过hexo快速搭建个人博客
date: 2019-10-23 00:30:39
tags: 
  - 教程
  - hexo
  - 博客
categories: 
  - 教程
  - hexo
---

>这是一个小教程,适用于windows  
>需要提前安装git  
>教程中所有命令均在git bush中输入运行
>本教程默认大家会使用git命令行和GitHub的相关操作
-----
### 安装**nodejs**  
[点击此处进入下载页面](http://nodejs.cn/download/)  
>安装时自行修改安装路径，**注意路径不要出现中文**，其他全默认选项
---
### 安装hexo框架  
``` bash
$ npm install -g hexo-cli  
```

安装时可以先行创建一个文件夹用来存放hexo，在文件夹内右键，选择Git Bush Here在弹出的命令窗内再输入运行命令即可  

---
### 初始化博客  
建立一个文件夹用来寻访博客相关文件  
在文件夹内打开Git Bush  
输入命令  
``` bash
$ hexo init  
```

---
### 本地查看生成博客  
以下命令均在博客文件夹内打开Git Bush进行操作  
命令  
生成博客  
``` bash
$ hexo g  
```


本地运行博客  
``` bash
$ hexo s  
```
在浏览器输入 http://localhost:4000 查看博客页面

---

### 上传博客到github

首先安装一个插件  
命令  
``` bash 
$ npm install --save hexo-deployer-git
```
登陆自己的GitHub  
新建仓库  
命名为自己的GitHub 用户名.github.io  
记录克隆地址  
打开博客根目录的_config.yml文件  
将此处修改为如下格式  
``` yml
# Deployment
## Docs: http://hexo.io/docs/deployment.html
deploy:
- type: git
  #在这里输入你的github仓库地址
  repo: git@github.com:用户名/仓库名.github.io.git
  branch: master
  message: update
```

修改完成后保存  
运行命令  
``` bash
$ hexo d  
```
稍等片刻即可打开你的个人博客  
地址为 仓库名.github.io  
例如：vampire610.github.io

---
<font color=orange > 注意，每次修改过代码之后，都要重新运行</font>
``` bash
$ hexo clean  
$ hexo g  
$ hexo d  
```
改动才会更新到网站

---
### 新建博客  
``` bash
$ hexo n "文件名"  
```
新建文件路径
>博客根目录/source/_posts

---
本次教程比较简洁，希望大家喜欢






