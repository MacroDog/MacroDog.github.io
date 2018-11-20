---
layout: post
title:  "LiangBarsky算法"
date:   2018-09-26 20:52:35
categories: MacroDog
tags: 图形学算法 裁剪算法
excerpt: 裁剪算法
mathjax: true
---
* content
{:toc}
# LiangBarsky 算法
(1) 用参数方程表示一条直线段 
    如由顶点$P_1(x_1,y_2)$ 和$P_2(x_2,y_2)$构成的一直线段L可以用参数方程表示为

$$x=x_1+u \cdot(x_2-x_1) = x_1+u\cdot \Delta x$$

$$y=y_1+u \cdot(y_2-y_1) = y_1+u\cdot \Delta y$$ 

$$(0 \lt u \lt 1)$$

(2) 把直线堪称是一条有方向的线段，将裁剪窗口及其延长线非为入边（左边界和下边界）和出边（右边界和上边界）
    如果用$u_1 u_2$分别表示线段$$(u_1 \leq u_2)$$在窗口可见部分的起点和终点对应的参数$u_l$为直线与左裁剪窗口及其延长线的交点 $u_b$为直线和下裁剪窗口及其延长线交点

$$u_1 = max(0,u_l,u_b)$$

$$u_2 = min(1,u_t,u_r)$$

因此判断直线上的一个点是否在窗口($x_{left}$是裁剪窗口最左边的坐标点x值$x_{right}$为裁剪窗口最右的坐标点x值)

$$x_{left} \leq x_1+u\cdot \Delta x \leq x_{right}$$

$$y_{bottom} \leq y_1+u\cdot \Delta y \leq y_{top}$$

进行变式

$$u\cdot -\Delta x \leq x_1 - x_{left}$$

$$u\cdot \Delta x \leq x_{right}-x_1$$

$$u\cdot -\Delta y \leq y_1 - y_{bottom}$$

$$u\cdot \Delta y \leq y_{top}-y_1 $$

令:

$$p_1 = -\Delta x $$ , $$q_1 = x_1 - x_{left}$$ 

$$p_2 = \Delta x$$ , $$q_2 = x_2 - x_{right}$$

$$p_3 = -\Delta y$$ , $$q_3 = y_1-y_{bottom}$$

$$p_4 = \Delta y$$ , $$q_4 = y_2-y_{top}$$

那么可以变换为

$$u =p_k \leq q_k$$

k=1,2,3,4对应左，右，上，下边界

当$p_k \gt 0 $对应的边为出边

当$p_k \lt 0 $对应的边为入边

一共4个u值再加上直线段的两个端点值（u=0，u=1）,总共六个值。

$$u_k=\frac{q_k}{p_k}(p_k\neq0,k=1,2,3,4)$$

将$p_k<0$两个入边的两个u值和0去比较找最大的

$$u_{max}=max(0,u_k|p_k<0,u_k|p_k<0)$$

将$p_k>0$两个出边的两个u值和1比较去找最小的

$$u_{max}=max(u_k|p_k>0,u_k|p_k>0,1)$$

然后将$u_{min}$和$u_{max}$代回直线参数方程，即可求出直线与窗口的两个实交点坐标

$$x=x_1+u\cdot(x_2-x_1)$$

$$y = y_1+u\cdot(y_2-y_1)$$