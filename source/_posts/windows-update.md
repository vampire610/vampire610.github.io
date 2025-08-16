---
title: windows_update
date: 2023-02-17 10:06:43
tags: 
  - windows
  - update
categories: 
  - 记录
---

# windows更新出错时取消更新

windows更新时，如果出现报错不能更新，可以通过以下方法取消已下载更新。

1. win+R后，输入services.msc，回车
2. 找到Windows Update，手动停掉
3. 定位到C:\Windows\Software Distribution，清掉Datastore和Download里的内容
4. 重启Windows Update，重新检查更新。
