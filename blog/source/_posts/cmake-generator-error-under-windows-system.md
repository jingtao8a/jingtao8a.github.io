---
title: cmake-generator-error-under-windows-system
date: 2023-12-07 00:22:51
tags: cmake
categories: cmake
---

1.安装windows版本cmake（配置环境变量）
2.安装windows版本mingw（配置环境变量）
3.创建工程目录
执行以下命令
```
mkdir build
cd build
cmake -G "MinGW Makefiles" -D "CMAKE_MAKE_PROGRAM:PATH=your path to make.exe"
make
```

参考：
https://blog.csdn.net/dcrmg/article/details/103918543
https://codeantenna.com/a/ELzh11ElWs
