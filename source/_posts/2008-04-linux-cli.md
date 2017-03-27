---
layout:     post
title:      "Linux的一些命令"
date:       2015-04-08
author:     "Bin"
tags:
    - linux
    - 笔记
---

locale 查看本地的语言环境
/ect/default/locale 本地语言配置文件

nano 一个文本编辑器
blkid 列出设备的UUID

apt-get update 更新软件包列表
apt-get upgrede 升级软件包
apt-get dist-update 如果运行apt-get upgrede后有些软件提示被保留，而不升级，这条可用来升级所有可用软件包
do-release-upgrade 版本升级

apt-cache 搜索某个软件包的名子，或显示软件包的信息
apt-cache show 显示软件包的信息

aptitude 高级包管理

tasksel 完成安装任务 定义到/usr/share/tasksel/ .desc 中，有图形界面

dpkg 比较底层的安装命令，以后都是调用这个
* dpkg -l <package> 查看某个包是否被安装
* dpkg -L <package> 查看软件包中包含哪些文件
* dpkg -S /pack/to/file 查看系统中某些文件是由哪个包提供的
* dpkg -C 查看哪些包末安装完成

sudo su 进入root用户

/etc/network/interfaces 网络配置
/etc/resolv.conf NDS配置



mc 文件管理器
mcedit mc带的一个文本编辑器

find 查找文件
locate 查找文件，在/etc/cron.daily/mlocate中查找，这个文件每天更新一次
updatedb 更新/etc/cron.daily/mlocate文件

/etc/ssh/sshd_config ssh配制

jobs 列出后台的任条
fg   将后台程序转入到前台 
bg   将前台程序转入到后台 

/etc/profile 环境变量文件

whereis 检查程序安装路径

/proc/cpuinfo cpu信息

getconf PAGESIZE 查看系统的页大小
