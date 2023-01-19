---
title: git常用命令
date: 2020-02-21 15:11:36
tags: 
  - git
categories: 
  - 笔记
  - git
---

<font color=gray size=2 >仅记录本人容易忘记的各条命令及部分解释</font>  

### 测试本地与服务器的连接
``` bash
$ ssh -T git@(服务器/网站名)  
例如：  
github:  
$ ssh -T git@github.com  
gitee:   
$ ssh -T git@gitee.com  
coding:  
$ ssh -T git@e.coding.net
```

### Windows直接在gitbush升级git
``` bash
$ git update-git-for-windows
```

### git查看远程仓库地址
``` bash
$ git remote -v
```

### gcc编译命令别名
``` .zshrc
#在.zsh添加
alias cc="g++ -Wall"
```
### 代理设置添加

``` .zshrc
#在.zshrc添加
function proxy_on() {
    export http_proxy=http://代理地址
    export https_proxy=https://代理地址
    echo -e "终端代理已开启。"
}

function proxy_off(){
    unset http_proxy https_proxy
    echo -e "终端代理已关闭。"
}
```



### 完善中，未完待续……
