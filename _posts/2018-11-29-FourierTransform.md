---
layout: post
title:  " 傅里叶变换(Fourier Transform)"
date:   2018-08-06 20:32:00
categories: MacroDog
tags: Math
excerpt: Fourier Transform
mathjax: true
---
* content  
{:toc}  

  之前 看了一个关于[傅里叶变换视频](https://www.bilibili.com/video/av19141078?from=search&seid=16252756375990833175)，觉得挺有意思引发了我学习傅里叶变换在此记录下一些笔记。  
# 起源
&emsp; 起源是1807年傅里叶在他的《热力分析理论》一书中中提出了 **任何连续周期信号可以由一组适当的正弦曲线组合而成。** 当时拉格朗日坚决反对，在他此后生命的六年中，拉格朗日坚持认为傅里叶的方法无法表示带有棱角的函数，如在方波中出现非连续变化斜率。当然拉格朗日是对的：正弦曲线无法组合成一个带有棱角的函数。但是，我们可以用正弦曲线来非常逼近地表示它，逼近到两种表示方法不存在能量差别，基于此，傅里叶其实也是对的: )
# 连续傅里叶变换
## 定义
&emsp;**将时间域的连续函数$f(t)$表示为频率域的函数$F(w)$的积分**(一般傅里叶变换指的就是连续傅里叶变换)$w$为角频率$w=2\pi u(u为频率)$  
傅里叶变换  

$$F(w) = \int^\infty_{-\infty}f(t)e^{-iwt}dt$$   

傅里叶逆变换：   

$$f(t) = \frac{1}{2\pi}\int^\infty_{-\infty}F(w)e^{iwt}dw$$ 

其中$\mu$是表示正弦项或余弦项的频率。(这个频率对应是t单位,如果t为单位为秒那么$\mu$对应的则是1秒内周期变化的次数)。
我们通过欧拉公式变换一下   
欧拉公式：  

$$ e^{ix} =\cos x+i\sin x $$  

e是自然对数的底，i是虚数单位 

$$F(w) = \int^\infty_{-\infty}f(t)e^{-iw t}=\int^\infty_{-\infty}f(t)[\cos(w t) -i\sin(w t)]dt$$  

由此可以看出傅里叶变变换是由一个时间域或空间域与频域之间的变换
# 傅里叶级数
## 定义 
&emsp;**任何连续周期函数可以由一组适当的正弦函数组合而成**  

$$f(t) = \sum^{\infty}_{m=-\infty}F_ne^{i2\pi nt/T}$$   

$T$为周期函数的周期，$F_n$为傅里叶展开系数    

$$F_n = \frac{1}{T}\int^\frac{T}{2}_\frac{-T}{2}f(t)^{-i2\pi nt/T}dt$$   

$$n=0, \pm1,\pm2,...$$    

$$f(t) = \frac{a_0}{2}+\sum^\infty_{n=1}[a_n\cos(\frac{2\pi nt}{T})+b_n\sin(\frac{2\pi nt}{T})]$$ 

其中$a_n$和$b_n$是实频率分量的振幅。       
其实最开始傅里叶在最开始在论文中提出来的就是傅里叶级数及**任何连续周期信号可以由一组适当的正弦曲线组合而成**，而后将周期$T$扩大到从$-\infty$到$+\infty$也就是上文中的连续傅里叶变换。    
(不过这么一想还挺有意思，看似没有规律的事物实则上都是有规律可循可以被计算出来的冥冥之中必有定数。让我想起系列科幻小说《基地》中提出的心理史学)
# 离散时间傅里叶变换（discrete-time Fourier transform, DTFT）
## 定义
&emsp;将离散时间nT(T为采样周期)作为变量的函数变换到连续的频域，即产生离散时间的连续频谱，值得注意的是这一频谱是周期的。。设$\left\{ X_{n} \right\}^\infty_{n=-\infty}$为某一个数列

$$X(w) = \sum^\infty_{n=-\infty}x_ne^{-iwn}$$  

逆变换   

$$x_n = \frac{1}{2\pi}\int^\infty_{-\infty}X(w)e^{iwn}dw$$   

# 离散傅里叶变换（DFT）    
## 定义
&emsp;离散傅里叶变换（DFT）指的是傅里叶变换在时域与频域上都成离散形式，将离散时域上采样的离散时间信号转换到离散频域的采样。在形式上变换两端（时域和频域）的序列都是有限的。设有限序列$\left\{ X_{n} \right\}^{N-1}_{n=0}$
$$
 X[k] =  \sum^{N-1}_{n=0}x_ne^{-i2\pi kn/N}
$$

逆变换
$$
x_n = \frac{1}{N}\sum^{N-1}_{k=0}X[k]e^{i2\pi kn/N}
$$   
&emsp;DFT的就算复杂度为$O_(N^2)$在计算机中为了减少计算复杂度而更多使用快速傅里叶变换(FFT)不过在借介绍FFT之前先了解一下傅里叶变换的几个性质。
# 性质
## 线性性质
&emsp;两个函数的线性组合的傅里叶变换等于两个函数分别傅里叶变换之后在进行线性组合。
$$
\mathscr{F}[\alpha f+\beta g] = \alpha \mathscr{F}[f]+\beta \mathscr{F}[g]
$$
## 尺度变换性质
&emsp;若函数$f(x)$的傅里叶变换$F(w)$,对任意非零实数a$f_a(x)=f(ax)$的傅里叶变换$F_a(w)$存在且等于
$$
F_a(w) = \frac{1}{|a|}\int^{\infty}_{-\infty}f(x)e^{iwt}dt
$$
## 对偶性
&emsp;若函数$f(x)$的傅里叶变换为$F(w)$，则存在
$$
\mathscr{F}(F(x)) = 2\pi f(-w)
$$
## 平移性质
&emsp;若函数$f(x)$的傅里叶变换为$F(w)$ ，则对任意实数$w_0$，函数$f_{w0}(x)=f(x)e^{iw_0x}$也存在傅里叶变换，且其傅里叶变换F_{w_0}(w)等于
$$
F_{w_0}(w) =F(w-w_0);
$$
## 微分关系
若函数$f(x)$的傅里叶变换$F(x)$,其导函数$f(x)$的傅里叶变换存在，则有
$$
\mathscr{F}[f'(x)] = iwF(w)
$$
如果n阶导数的傅里叶变换存在的话则
$$
\mathscr{F}[f^{(n)}(x)] = (iw)^{(n)}F(w)
$$
# 快速傅里叶变换（FFT）   
FFT是离散傅氏变换（DFT）的快速算法
在理解FFT之前先理解以一种函数表达方法：在DFT中对于有限序列$x\epsilon\left\{ X_{n} \right\}^{N-1}_{n=0}$有$X[k] = \sum^{N-1}_{n=0}xe^{-i2\pi kn/N}$ 如果全部求和的话算法的复杂度就是$O(n^2)$   
$$
e^{-ix} =\cos x-i\sin x$$
$$
X[k] = \sum^{N-1}_{n=0}x_ne^{-i2\pi kn/N}
$$
$$
X[k]  = \sum_{n=0}^{N-1}x_n\cos{\frac{2\pi kn}{N}}-i\sum_{n=0}^{N-1}x_n\sin{\frac{2\pi kn}{N}}
$$
$$
X[k]= \sum_{n=0 偶数}^{N-1}x_ne^{-2i\pi kn/N}+ \sum_{n=1 奇数}^{N-1}x_ne^{-2i\pi kn/N}
$$
$$
X[k]= \sum_{r=0}^{\frac{N}{2}-1}x_{(2r)}e^{-2i\pi k2r/N}+ \sum_{r=0}^{\frac{N}{2}-1}x_{(2r+1)}e^{-2i\pi k(2r+1)/N}
$$
根据$e^{-i2\pi kn/N}$周期性
$$
e^{-i2\pi kn/N}=e^{-i2\pi (k+N)n/N}=e^{-i2\pi k(n+N)/N}
$$
可以得出
$$
X[k]= \sum_{r=0}^{\frac{N}{2}-1}x_{(r)}e^{-2i\pi kr/\frac{N}{2}}+ e^{\frac{-2i\pi k}{N}}\sum_{r=0}^{\frac{N}{2}-1}x_{r}e^{-2i\pi kr/\frac{N}{2}}
$$
$$
X[k]= X_0(k)+ e^{\frac{-2i\pi k}{N}}X_1(k)
$$
但是你可能注意到此时k值的取值范围变为了0到$\frac{N}{2}-1$
根据$e^{-i2\pi k(n+N/2)/N}=-e^{-i2\pi kn/N}$和可以将$X_0(k+N/2) = \sum_{r=0}^{\frac{N}{2}-1}x_{(r)}e^{-2i\pi r(k+N/2)/\frac{N}{2}}=\sum_{r=0}^{\frac{N}{2}-1}x_{(r)}e^{-2i\pi kr/\frac{N}{2}}$而$X_1(k)=e^{-i2\pi (k+N/2)/N} \sum_{r=0}^{\frac{N}{2}-1}x_{(r)}e^{-2i\pi r(k+N/2)/\frac{N}{2}}=-e^{-i2\pi k/N} \sum_{r=0}^{\frac{N}{2}-1}x_{(k+N/2)}e^{-2i\pi rk/\frac{N}{2}}$
得到
$$
\left\{
\begin{gathered}
X[k]= X_0(k)+ e^{\frac{-2i\pi k}{N}}X_1(k) \\
X[k]= X_0(k)- e^{\frac{-2i\pi k}{N}}X_1(k)
\end{gathered}
\right.
$$
其中$e^{\frac{-2i\pi k}{N}}$就是蝶形变换中的 $W_N^{k}$得出的结果可以继续进行蝶形变换

$$
\left\{
\begin{gathered}
X[k]= X_0(k)+ W_N^kX_1(k) \\
X[k]= X_0(k)-W_N^kX_1(k)
\end{gathered}
\right.
$$
为了方便代码实现我们将实部和虚部分开

