---
title: cmake学习
date: 2023-05-28 11:02:55
tags: cmake
categories: cmake
---

语法手册：cmake语法手册及教程_可克的博客-CSDN博客_cmake语法

[哔哩哔哩视频](https://www.bilibili.com/video/BV1vR4y1u77h?spm_id_from=333.337.search-card.all.click)

视频下方有笔记

![图片](images/cmake学习/1.jpg)


add_definitions()添加编译选项



include_directories()将指定目录添加到编译器的头文件搜索路径下

[参考](https://www.jianshu.com/p/e7de3de1b0fa)

add_library()生成静态库（STATIC)或者动态库（SHARED）

add_executable()生成可执行文件

aux_source_directory(dir VAR) 发现一个目录(dir)下所有的源代码文件并将列表存储在一个变量(VAR)中

target_link_libraries( # 目标库 demo # 目标库需要链接的库 ${log-lib} )