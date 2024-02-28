---
title: ssh免密登录失败
date: 2024-02-28 03:22:49
tags: ssh
categories: 随笔
---
## SSH免密码失败原因定位分析
1. 服务器B上.ssh目录的权限必须是700
2. 服务器B上.authorized_keys文件权限必须是600或者644
3. 服务器B上用户家目录文件权限必须是700，比如用户名是aischang，则/home/aischang这个目录权限必须是700

如果不是700，在服务器A上查看/var/log/secure文件会报错

## 原因：
sshd为了安全，对属主的目录和文件权限有所要求。如果权限不对，则ssh的免密码登陆不生效。
> 1. 服务器B上SELinux关闭为disabled，可以使用命令修改setenforce 0 ，查看状态的命令为getenforce或者 查看/etc/selinux/config 文件中是否是disabled

> 2. 有可能是StrictModes问题<br>
    编辑 vi /etc/ssh/sshd_config<br>
    找到#StrictModes yes改成StrictModes no

> 3. 有可能是PubkeyAuthentication问题<br>
    编辑 vi /etc/ssh/sshd_config<br>
    找到PubkeyAuthentication改成yes

如果还不行，可以在服务器A上用ssh -vvv 机器B的ip 查看详情，根据输出内容具体问题具体分析了

参考链接: [https://juejin.cn/s/ssh%E5%85%8D%E5%AF%86%E7%99%BB%E5%BD%95%E5%A4%B1%E8%B4%A5%E5%8E%9F%E5%9B%A0](https://juejin.cn/s/ssh%E5%85%8D%E5%AF%86%E7%99%BB%E5%BD%95%E5%A4%B1%E8%B4%A5%E5%8E%9F%E5%9B%A0)