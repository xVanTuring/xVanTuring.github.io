---
title: '算法: 优先列队 | Algorithm: Priority Queue'
date: 2019-05-04 21:31:37
tags: [Algorithm, Sort, Queue, C]
---
## Brief
优先列队是一个非常常用的数据结构，它始终可返回数据列表中最大或最小的数据，其每次只须 O(lgn) 时间来取得最大或最小的数据。
在项目 [leptonica](http://www.leptonica.com/) 中，用于存储 MMCQ 算法所需的数据。
笔者在提取 MMCQ 算法时也将该结构提出独立出来，项目地址为 [xVanTuring/heap](https://github.com/xVanTuring/heap).
<!-- More -->
## 二叉堆

![tree](https://s2.ax1x.com/2019/05/04/Ed7FtU.png)
如图所示的完全二叉树，其右侧为个结点的ID，，自左向右，自上而下，逐渐增加。
我们可以发现一个规律：
* 任何一个节点的左子节点（如果有），其ID为：i*2+1
* 任何一个节点的右子节点（如果有），其ID为：i*2+2
* 任何一个节点的父节点（如果有），其ID为：(i-1)/2
通过这种特性，我们可以把二叉树存储至数组中，并通过以上公式来访问相关节点,减少存储节点地址所需的空间。
如下图所示
![Array](https://s2.ax1x.com/2019/05/04/EdqK7q.png)
## 实现
``` js
var heap=[]
```
### 添加