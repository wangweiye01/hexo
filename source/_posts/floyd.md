---
title: floyd最短路径算法
date: 2020-03-24 15:39:41
tags:
---

## floyd算法

弗洛伊德算法又称插点法是**解决任意两点间的最短路径的一种算法**

## 适用范围

边权可正可负，运行一次算法即可求得任意两点间的最短路径（无负权回路即可）

可以解决“多源最短路径”

### 什么是负权回路

图中1号点到3号点的最短路径是什么？

![6.10.png](http://s1.wailian.download/2020/03/24/6.10.png)

由于每次经过1->2->3这样的环，最短路径就会-1，所有永远找不到最短路径

## 实例

![6.2-floyd.png](http://s1.wailian.download/2020/03/24/6.2-floyd.png)

计算图中各个顶点到各个顶点的最短距离

首先我们定义一个二维数组来存储图中最易可见的点对点(不允许经过第三点)之间的距离，如果不能到达使用∞表示。另外规定自己到自己的距离是0，具体表示如下

[![6.3-.png](http://s1.wailian.download/2020/03/24/6.3-.png)](http://www.wailian.work/image/Ae1zve)

如果要使两点之间距离变短，唯一方法就是通过第三个点来解决。例如a点到b点的距离通过顶点k来中转那么a->k->b才有可能缩短a->b的距离。那么这个k点是哪个点呢？是否不只通过1个k点最短，而是经过两个或者更多点最短，比如a->k1->k2>b

比如图中4->3的距离是12，如果经过1点，那么4->1->3的距离就变为11。现在只允许经过1号点，求i点到j点的最短距离，只需判断e[i][1]+e[1][j]是否比e[i][j]要小。代码实现如下

```python
#!/usr/bin/env python3
# -*- coding: utf-8 -*-
import copy

# 创建一个4行4列的二维数组
m = [[0]*4 for i in range(4)]

# 为数组赋值
m[0][0] = 0
m[0][1] = 2
m[0][2] = 6
m[0][3] = 4
m[1][0] = 9999
m[1][1] = 0
m[1][2] = 3
m[1][3] = 9999
m[2][0] = 7
m[2][1] = 9999
m[2][2] = 0
m[2][3] = 1
m[3][0] = 5
m[3][1] = 9999
m[3][2] = 12
m[3][3] = 0

print('原路径数据:', m)

# 允许经过1号点的最短路径
n = copy.deepcopy(m)
i = 0
j = 0

for i in range(4):
    for j in range(4):
        if (n[i][j] > n[i][0] + n[0][j]):
                n[i][j] = n[i][0] + n[0][j]

print('在允许经过1号点后的最短路径:', n)
```

执行结果如下：

原路径数据: [[0, 2, 6, 4], [9999, 0, 3, 9999], [7, 9999, 0, 1], [5, 9999, 12, 0]]

在允许经过1号点后的最短路径: [[0, 2, 6, 4], [9999, 0, 3, 9999], [7, 9, 0, 1], [5, 7, 11, 0]]

![6.5-1.png](http://s1.wailian.download/2020/03/24/6.5-1.png)

通过图中白色背景处我们可以看到通过1号点中转的情况下,3->2,4->2,4->3的路径都变短了

接下来求只允许1号和2号点的情况下任意两点间的最短距离，如何做呢？我们只需要在只经过1号点时任意两点最短距离的结果下，再判断经过2号顶点如何使i号点到j号点的路径边的更短

```python
# 再允许经过2号点后的路径
o = copy.deepcopy(n)
i = 0
j = 0

for i in range(4):
    for j in range(4):
        if(o[i][j] > o[i][1] + o[1][j]):
            o[i][j] = o[i][1] + o[1][j]

print('再允许经过2号点后的最短路径:', o)
```

执行结果如下：

再允许经过2号点后的最短路径: [[0, 2, 5, 4], [9999, 0, 3, 9999], [7, 9, 0, 1], [5, 7, 10, 0]]

![6.6-1-2.png](http://s1.wailian.download/2020/03/24/6.6-1-2.png)


同理继续在只允许1，2，3进行中转的情况下求最短距离。

![6.7-1-2-3.png](http://s1.wailian.download/2020/03/24/6.7-1-2-3.png)


最后在允许通过所有顶点作为中转，任意两点之间最终的最短路径为：

![6.8-1-2-3-4.png](http://s1.wailian.download/2020/03/24/6.8-1-2-3-4.png)

整个算法最终代码实现如下：

```python
k = 0
i = 0
j = 0

for k in range(4):
    for i in range(4):
        for j in range(4):
            if (m[i][j] > m[i][k] + m[k][j]):
                m[i][j] = m[i][k] + m[k][j]

print('最终路径:', m)
```

最终路径: [[0, 2, 5, 4], [9, 0, 3, 4], [6, 8, 0, 1], [5, 7, 10, 0]]

通过这种方法我们可以求出任意两个点之间最短路径。它的时间复杂度是 O(n<sup>3</sup>)