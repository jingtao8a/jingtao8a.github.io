---
title: ASLM:Adaptive Single Layer Model for Learned Index
date: 2024-09-28 18:51:06
tags: Learned Index
categories: Learned Index
---

### 应用场景：
读写场景

### 问题描述：
Learned Index<br>
(1)RMI 采用的分区策略没有考虑数据之间的相似性<br>
(2)RMI 不支持更新

![img](../images/ASLM-Adaptive-Single-Layer-Model-for-Learned-Index/1.png)
### 方法：
#### 分区算法
##### Method1:
![img](../images/ASLM-Adaptive-Single-Layer-Model-for-Learned-Index/2.png)
SLM:这是一个简单的模型，只有一层，这一层包含K个子模型

##### Method2：
![img](../images/ASLM-Adaptive-Single-Layer-Model-for-Learned-Index/3.png)
数据点之间三角形面积可替换为欧式距离，用来表示两个数据点之间的相似度

#### 数据插入：
![img](../images/ASLM-Adaptive-Single-Layer-Model-for-Learned-Index/4.png)

![img](../images/ASLM-Adaptive-Single-Layer-Model-for-Learned-Index/5.png)

### 结果：
![img](../images/ASLM-Adaptive-Single-Layer-Model-for-Learned-Index/6.png)

![img](../images/ASLM-Adaptive-Single-Layer-Model-for-Learned-Index/7.png)
