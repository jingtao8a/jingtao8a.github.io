---
title: linux文件权限和属性
date: 2023-05-28 10:56:48
tags: [Linux文件权限]
categories: Linux
---

如果一个文件被设置了SUID或SGID位，会分别表现在所有者或同组用户的权限的可执行位上。例如：

1、-rwsr-xr-x 表示SUID和所有者权限中可执行位被设置

2、-rwSr–r– 表示SUID被设置，但所有者权限中可执行位没有被设置

3、-rwxr-sr-x 表示SGID和同组用户权限中可执行位被设置

4、-rw-r-Sr– 表示SGID被设置，但同组用户权限中可执行位没有被设置

给文件加SUID和SUID的命令如下：

chmod u+s filename 设置SUID位

chmod u-s filename 去掉SUID设置

chmod g+s filename 设置SGID位

chmod g-s filename 去掉SGID设置


SUID属性
例如/usr/bin/passwd  带有SUID属性 属于root用户 root用户主
其它用户只有/usr/bin/passwd的可执行权限，在执行这个命令时会暂时获取root权限

SGID属性
和SUID属性十分相似
不同的是其它用户在执行有SGID属性的命令时，会暂时获取该程序群组的支持