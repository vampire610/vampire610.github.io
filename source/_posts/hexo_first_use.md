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
在浏览器输入 `http://localhost:4000` 查看博客页面

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
  branch: main
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
### 博客备份

- 安装插件 `hexo-git-backup`

``` bash
$ npm install hexo-git-backup --save
```

- 在GitHub中新建仓库或新建分支

- 在Hexo配置中添加

  ``` yaml
  backup:
      type: git
      repository:
         backup: git@github.com:githubusername/Hexo-Backup.git,backup(分支名,新库为main,备份分支可用backup)
  ```

- 备份使用

  ``` bash
  $ hexo b
  ```

  

### 博客恢复

- 有条件时拷贝整个文件夹转移
- 必备文件
  - `config.yml`（站点配置）
  - `theme`（主题文件夹）
  - `source`（博客内容文件）
- 次要文件
  - `scaffolds`（文章的模板）
  - `package.json`（使用包的说明文件）
  - `.gitignore`
  - 三个次要文件为自动生成，`hexo init` 时会自动生成
- 需要删除的文件
  - `.git`
  - `node_modules`（进行`npm install`会自动生成
  - `public`（执行`hexo g`时会生成）
  - `.deploy_git`文件夹（执行hexo d时会重新生成）
  - `db.json`文件
  - 其实上面这些可删除的文件即为`.gitignore`文件里面记载的可以忽略的内容

### 恢复博客

- 环境搭建（同上）

- 拷贝文件

  - 若使用`hexo-git-backup`插件备份，只需从github拉入本地

	  ``` bash
    $ git clone https://github.com/username/备份仓库名.git
    ```

  
  - 若是拷贝文件，必要文件或目录
  
    ``` 
    _config.yml
     package.json
     scaffolds/
     source/
     themes/
    ```
  
    

