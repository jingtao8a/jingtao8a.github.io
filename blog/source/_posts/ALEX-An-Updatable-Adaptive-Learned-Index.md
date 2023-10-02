---
title: ALEX-An-Updatable-Adaptive-Learned-Index
date: 2023-10-01 21:47:41
tags: Learned Index
categories: Learned Index
---


## 3 ALEX OVERVIEW
### 3.1 DESIGN OVERVIEW
 1.gapped array
2.Exponential search without error bounds
3.Model-based insertion
4.Dynamically adjusts the shape and height of the RMI depending on the workload
5.Use Cost model to adjust the structure of RMI

### 3.2 NODE LAYOUT

- 每个Data Node保存一个线性回归模型(用于预测key的位置)，两个gapped arrays，并使用一个bitmap跟踪节点中每个位置是否被key占用或者是间隙(Array使用gaps右侧最近的key填充间隙)

- 每个Internal Node保存一个线性回归模型和一个数组(保存指向child node的指针)，模型均匀地将keys分摊到每个child node，Internal Node中的指针数量限制始终为2的幂，它的作用是提供一种灵活的方法来划分key空间

## 4. ALEX ALGORITHMS
**点查询：**
> 从root node开始，遍历到data node，data node通过线性回归模型预测key在key array的位置（有必要的话，使用指数查找，没找到返回null），然后查找对应的payload array

范围查询:
> 查找first key和点查询一样，之后使用bitmap跳过gaps，如果有需要的话，跳到下一个data node

**插入未满的Data Node：**
```
点查询遍历到data node，使用模型预测插入的key的位置
If 插入无法维持排序顺序
使用指数查找找到正确的插入位置
If 插入位置是一个gap
直接插入
Else 
移动右侧的key一个位置，插入 
```
**插入已满的Data Node：**
扩展和拆分策略，使用cost模型从两者之间选择
- Criteria for Node Fullness: ALEX 不会等到data node100% full，因为这样gap array的插入性能会随着gap数量的减少而恶化，提出dl和du密度（0~1之间，用于描述已插入key的空间比例），用于在空间和查找性能之间权衡
- Node Expansion Mechanism:
- Node Split Mechanism: 1.split sideway  2.split down
- Cost Models:
  每个Data node记录两个统计数据信息 1.指数搜索操作的平均次数 2.插入时的平均移位次数，intra-node cost model根据信息1来预测查找性能，根据信息1和信息2来预测插入性能。对于新创建的节点，没有这两个信息，信息1为预测所有key的模型的预测误差的以2为底的平均对数，信息2为所有key到gap array中最近gap的平均距离。
- Tra-verseToLeaf cost model: 
  预测从root node遍历到data node的时间(使用两个数据，data node的深度，所有internal node和data node 的metadata的总大小bytes)
- Insertion Algorithm: 
  使用intra-node cost model计算Data Node 的empirical cost，将expetected cost 和 empirical cost比较，如果没有较大的偏离，我们执行Node Expansion Mechanism，

**删除、更新、其它操作**
如果删除一个key后，该Data Node的密度低于dl，需要收缩数据节点

**Bulk Load：**
目的是找到一个RMI结构(cost最小)，使用TraverseToLeaf和intra-node models预测cost
- Bulk Load Algorithm
- The Fanout Tree:
  生成RMI结构时，我们主要的挑战就是决定每个Node的最好的扇出数量，FT可以用来帮助我们决定每个RMI节点的扇出数量，在Bulk Load Algorithm中，每当我们要决定RMI node的扇出数量时，我们都要构建一颗FT