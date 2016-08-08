---
title: ubuntu 16.04 升级中遇到的坑 
date: 2016-08-08 23:07:57
tags: tips 
categories: tips
---
##### ubuntu 遇到 failed to start load kernel modules 的解决办法
根据提示查看  
显示loaded: loaded active: failed  
根据字面意思就是不能够加载内核的模块  
在外网上找到的解决方案 觉得很有道理啊  
下面介绍方法，主要是自己小记录一下~
<!--more-->
进入修复模式或者能够直接进入字符界面也可以  
保证能够联网  
这一步我又出错了，apt-get的时候提示错误，搜索得知dns可能有问题，修改完dns之后（方法网上都有），要记得
```
sudo /etc/init.d/networking restart
```
确定可以联网之后
```
sudo apt-get update #获取最新的软件包列表
sudo dpkg --configure -a #配置所有没有配置的软件包
sudo apt-get dis-upgrade #会根据依赖关系更新包
sudo apt-get -f install #修复依赖关系
```
