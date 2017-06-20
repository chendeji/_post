---
title: linux安装过程中遇到的一些问题
date: 2015-06-20 11:45:32
tags: [linux]
categories: [linux]
---

# linux安装软件中遇到的问题合集

**被锁住**
无法获得锁 /var/lib/dpkg/lock - open (11: 资源暂时不可用) 
E: 无法锁定管理目录(/var/lib/dpkg/)，是否有其他进程正占用它？”

解决办法如下：
1.终端输入 ps  -aux ，列出进程,找到含有apt-get的进程，直接sudo kill PID解决。
2.强制解锁--命令:
sudo rm /var/cache/apt/archives/lock
sudo rm /var/lib/dpkg/lock