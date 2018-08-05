---
layout: post
title:  "Liner Math"
date:   2017-08-4 14:52:35
categories: MacroDog
tags: 数学
excerpt: 读书的一些笔记不定期更新
mathjax: true
---
# 线性代数笔记
## 矩阵
矩阵的秩决定了解有多少个 Ax=b 
r(rank), m(row),n(cow)
free variable =max(0, n-m)
r=m=n
{ 1 resolution }
r=m<n
{  0 or 1 resolution }
r=n<m
{  ∞ resolution }
definition:	
Independence: vector combination not being zero
Spanning: vector all the combinations 
Basis: one the conbinatons independence and spanning
dimension of the space: the number of the vectors in any basis     //ps: dimension of null  = n - r 		 
    the number of the free variables

零空间的基向量就是使矩阵某一行变为0的向量
四个基本向量
Am×n，列空间C(A)，零空间N(A)，行空间C(AT)，A转置的零空间（通常叫左零空间），线性代数的核心内容，研究这四个基本子空间及其关系

零空间N(A)
n维向量，是Ax=0的解，所以N(A)在Rn里。

列空间C(A)
列向量是m维的，所以C(A)在Rm里。

行空间C(AT)
A的行的所有线性组合，即A转置的列的线性组合（因为我们不习惯处理行向量），C(AT)在Rn里。

A转置的零空间N(AT)—A的左零空间
N(AT)在Rm里。

## 行列式10性质
#### (1)
$$
a \left[
 \begin{matrix}
   1 & 2 & 3 \\
   4 & 5 & 6 \\
   7 & 8 & 9
  \end{matrix}
  \right] = \left[\begin{matrix}
   a*1 & a*2 & a*3 \\
   4 & 5 & 6 \\
   7 & 8 & 9
  \end{matrix}
  \right]
$$

#### (2)
交换行，行列式的值符号会相反

#### (3)
$$
    \left[
    \begin{matrix}
    a+a' & b+b' \\
    c & d
    \end{matrix}
    \right] = \left[
    \begin{matrix}
    a & b \\
    c & d
    \end{matrix}
    \right] +
    \left[
    \begin{matrix}
    a' & b' \\
    c & d
    \end{matrix}
    \right]
$$
#### (4)
如果两行相等 或者成比例 那么det为0
#### (5)
 如果k行减去i倍的l行 不会改变  
 $$
  \left[
    \begin{matrix}
    a & b \\
    c-la & d-lb
    \end{matrix}
    \right] = \left[
    \begin{matrix}
    a & b \\
    c & d
    \end{matrix}
    \right] + \left[
    \begin{matrix}
    a & b \\
    -la & -lc
    \end{matrix}
    \right] = \left[
    \begin{matrix}
    a & b \\
    c & d
    \end{matrix}
    \right] + 0
 $$
#### (6)
若有一行全为0 那么det为0
#### (7)
$$ det
\left[
\begin{matrix}
d1 & * & *\\
0 & d2 & *\\
0 & 0 & d3\\
\end{matrix}
\right] = d1*d2*d3
$$
####(8)
如果det A = 0那么 A 是奇异矩阵
如果det A != 0 那么 A是可逆的
#### (9) 
det(AB) = det(A)*det(B)
det2A = 2^n detA
#### (10)
det A<sup>T</sup> = det A
|U<sup>T</sup>L<sup>T</sup>|= |LU|
| U<sup>T</sup>||L<sup>T</sup>|=|L||U|

### 行列式公式和代数余子式
A<sub>ij</sub>=(-1)<sup>ij</sup>M<sub>ij</sub>
A<sub>ij</sub>是矩阵D中a<sub>ij</sub>的代数余子式
$$
det(D)= \sum_{i=1,0<k<n}^n a_{ki}A_{ki}   
$$ 
证：
$$ 
\left[
\begin{matrix}
a_{11} &a_{12}&... & a_{1n}\\
... &...& ...&...\\
a_{j1}&a_{j2}& ...&a_{jn}\\
... &...& ...&...\\
a_{n1}&a_{n2}&...&a_{nn}\\
\end{matrix}
\right] =\left[
\begin{matrix}
a_{11} &a_{12}&... & a_{1n}\\
... &...& ...&...\\
a_{j1}+0&a_{j2}+0& ...&a_{jn}+0\\
... &...& ...&...\\
a_{n1}&a_{n2}&...&a_{nn}\\
\end{matrix}
\right]
$$ $$ =\left[
\begin{matrix}
a_{11} &a_{12}&... & a_{1n}\\
... &...& ...&...\\
a_{j1}+0&0& ...&0\\
... &...& ...&...\\
a_{n1}&a_{n2}&...&a_{nn}\\
\end{matrix}
\right]+\left[
\begin{matrix}
a_{11} &a_{12}&... & a_{1n}\\
... &...& ...&...\\
0+0&a_{j2}+0& ...&0\\
... &...& ...&...\\
a_{n1}&a_{n2}&...&a_{nn}\\
\end{matrix}
\right]+...+\left[
\begin{matrix}
a_{11} &a_{12}&... & a_{1n}\\
... &...& ...&...\\
0&0& ...&a_{jn}+0\\
... &...& ...&...\\
a_{n1}&a_{n2}&...&a_{nn}\\
\end{matrix}
\right]
$$ 
$$
=> D=a_{11}A_{11}+...a_{nn}A_{nn}
$$
$$
=>det(D)= \sum_{i=1,0<k<n}^n a_{ki}A_{ki}   
$$ 
由此可以引伸至
求非奇异矩阵$$
D=\left[\begin{matrix}a_{11}&a_{12}&...&a_{1n}\\
...&...&...&...\\
a_{i1}&a_{i2}&...&a_{in}\\
...&...&...&...\\
a_{n1}&a_{n2}&...&a_{nn}\\
\end{matrix}\right]
$$
的逆矩阵 A<sub>ij</sub>是D矩阵中a<sub>ij</sub>的代数余子式C如下
$$
C=\left[\begin{matrix}
A_{11}&A_{21}&...&A_{n1}\\
...&...&...&...\\
A_{12}&A_{22}&...&A_{1n}\\
...&...&...&...\\
A_{1n}&A_{2n}&...&A_{nn}\\
\end{matrix}\right]
$$
$$  D^{-1}=det(D)^{-1}C^{T}$$


