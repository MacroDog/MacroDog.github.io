---
layout: post
title: "全局关照技术笔记"
date:   2019-02-11 14:24:22
categories: MacroDog
tags:  ComputerGraphies 
excerpt: Global Illumination
mathjax: true
---
* content  
{:toc}

# 1 光与表面的交互
## 1.2 辐射度量学
### 辐射通(radiant flux)
每秒发射功率   
符号:$\Phi$    
单位:瓦特$W$
### 辐射亮度(Radiance)
在辐射源上的某一点在一定方向上的辐射强弱的物理量。辐射源在单位时间内通过面元法线方向n上单位面积，单位立体角的辐射出的能量，辐射源在单位投影面积上，单位立体角内的辐射通量。(即某个点在某个方向上的亮度,一般是渲染方程最终需要计算的值)L的值不予距离发射点距离变换而变换只与位置($\xi,\eta$)和方向($\alpha,\beta$)有关    
符号:$L$        
单位:$W/(sr*m^2)$    
关系: $d\Phi=L\cos\theta dAdw$
### 辐射强度
点辐射源在某方向上单位立体角内传送的辐射通量    
符号: $I$    
单位: $W/sr$  
关系: 
$$dI=\frac{d\Phi}{dw}$$    
$$dI=L\cos\theta dAw$$  
$$I(\alpha,\beta) = \int L\cos\theta dA$$ 
(表示I为辐射源面A在方向($\alpha,\beta$)上的辐射强度,与距离无关)    
当发生反射的时，L随方向变换取决于这个面的性质：光滑与粗糙，自发光还是反射或是透射。如果在近似的下，L与方向无关，这时的辐射成为各项同性。如果是各项同性且辐射面为平面那么可以将方程简化为：   
$$I(\alpha,\beta) = I_0\cos\theta$$  
$$I_0=\int LdA$$   
这时任何方向上的辐射强度随该方向与面法线方向间夹脚的余弦值变化而变化。上式 也被称为朗伯余弦定理(Lambert's cosine law)。如果是发射面被称为漫发射，如果是反射被称为漫反射。
### 辐射照度
通过该点的单位面积下所有光照。也用来表示光进入一个单位表面的强度。   
单位:$w/m^2$    
关系:
$$dE=\frac{d\Phi}{dA}=L\overline{\cos}\theta dw$$    
$$E(\xi,\eta) = \int L\overline{\cos}\theta dw$$
而一般光照贴图(ligh map)存的信息即是辐射照度。     
也可以用来表示光离开一个单位表面的强度，也叫出射度或辐射度(radiosity)    
在平行光中光源的辐射强度不随表面带光源的距离发生改变。    
对于非平行光$dA$是点P的面元,QP=r，$\theta$是光线方向QP与$dA$的法线的夹脚$dw$是对Q的立体角   
由立体角关系
$$dw = \frac{dS}{r^2}$$
$$dS =\cos\theta dA$$   
和辐射通量关系
$$d\phi =Idw$$
得到:
$$E =\frac{I\cos\theta}{r^2} $$
　
## 1.3物体表面着色
### 光与表面交互
1.光源发射光线   
2.光线与物体表面交互   
3.被感应器吸收形成图像
#### 光源
发出光并照亮整个场景，不会反射或吸收其它光照   
1.平行光   
2.点光源   
3.聚光灯   
**一般图形学中使用辐射强度I表示光源。** 因为图形学中大部分光源的亮度分布只与方向有关。
一般引擎不允许对所有方向都自定义分布的光源。    
计算光照与物体表面交互的最终结果是辐射照度   
$$E=\frac{I\cos\theta}{r^2}$$
虽然，物理上光照强度随距离平方变化，但是为了方便计算一般使用
$$E=I\cos\theta f_{dist}(r)$$
### 材质
光与媒物体表面交互的一切，都是由材质及其属性决定。   
材质包含纹理和着色器（以及着色器需要的参数）。这些数据在渲染时被传入GPU中。
一般光与物体表面交互分为折射和反射
而当折射进入物体内部的时候有无材质的结构不同分为**金属**和**非金属**
#### 金属
内部吸收所有折射
#### 非金属 
透明物体：折射的光在物品内部延直线传播直到另一个表面射出    
非透明物体:由于物体内部介质不连续导致折射的光延不同方向折射或反射，最后光的部分会被重新发射到外面（位置和方向不确定），部分被吸收。这种反射被称为**漫散射(scattering)**   
对于散射光重新发射到表面外的位置与入射点之间的距离的分布，取决于该物质表面属性。如果距离小于一个像素尺寸，着色的时候设为0这种现象被称为局部次表面散射(local subsurface scattering)。否则被称为全局次表面散射(Global subsurface scattering)   
#### 反射
光在物体表面直接反射形成(也被称为光泽)    
反射光的范围的大小取决于表面的粗糙度，表面越粗糙，反射范围越大，表面越模糊；反之越光滑，范围越小，也越亮。
#### 漫反射
光通过散射回物体表面形成
### 感应器
光在场景中与物体交互以后一部分会进入传感器中形成图像。传感器接受的是辐射亮度L(由渲染管线中的像素着色器计算fragment shader)，摄像机就是一个典型的感应器(Sensor)   

## 采样与反走样
3D图像实际上是将连续函数转换对应离散函数。   
### 采样
采样：连续信号转换为离散信号   
对于连续函数$f(x)$利用傅里叶函数可得
$$F(\mu) = \int^\infty_{-\infty}f(t)e^{-i2\pi ut}dt$$
在通过欧拉公式$e^{i\theta}=\cos\theta+i\sin\theta$可得
 $$F(\mu)=\int^\infty_{-\infty}f(t)[\cos(2\pi ut)-i\sin(2\pi ut)]dt $$
 通过变换$F(\mu)$的作用域为频域（关于傅里叶变换详情[]）这样的话就能知道该连续函数是否存在一个最大频率，如果存在为带限函数(bandlimited function)这个函数是可以完美重建的。
#### 卷积
两个函数$f$和&g&，其中一个函数反转180度两个函数的乘积的积分
$$f(t)\star g(t)=\int^{\infty}_{-\infty}f(\tau)g(t-\tau)d\tau$$ 
同时对于傅里叶变换来说  两个函数的傅里叶变换卷积等于连个函数的卷积的傅里叶变换即
$$f(t)\star h(t) \Leftrightarrow H(\mu)F(\mu) $$
$$f(t)h(t) \Leftrightarrow H(\mu)\star F(\mu)$$
**冲激函数**
$$ \delta(x)=\left\{
\begin{aligned}
\infty\quad t=0 \\
0\quad t\neq0
\end{aligned}
\right.
$$
同时满足
$$\int_{-\infty}^\infty\delta(t)dt=1$$
采样特性     
如果函数$f(t)$在t=0处是连续那么
$$\int^\infty_{-\infty}f(t)\delta(t)dt = f(0)$$
对于任意位置$t_0$则有
$$\int_{-\infty}^\infty f(t)\delta(t-t_0)dt =f(t_0)$$
$$ s_{\Delta T}(t) = \sum_{n=-\infty}^\infty\delta(t-n\Delta T)$$
$$\tilde{f}(t)=f(t)s_{\Delta T}(t)=\sum_{n=-\infty}^{\infty}f(t)\delta(t-n\Delta T)$$  
所以每一个分量由该冲激位置$f(t)$的值加权($s_{\Delta T(t)}$) 得出。因此采样$f_k$为
$$f_k=\int^\infty_{-\infty}f(t)\delta(t-k\Delta T)dt=f(k\Delta T)$$  
**由此式可以得出 冲激函数实际上是对原函数进行等间隔采样**
设冲激函数$\delta(t)$的傅里叶变换为$S(\mu)$f(t)的傅里叶变换为$F(\mu)$$\tilde{f}(t)$的傅里叶变换为$\tilde{F}(\mu)$  傅里叶变换符号为 $\mathcal{F}$   
$$\mathcal{F}(\delta (t))=\mathcal{F}\{\tilde f(t)\}=\mathcal{F}\{f(t)s_{\Delta T}(t)\}=F(\mu)\star S(\mu)$$
冲激函数的傅里叶变换为:
$$S(\mu)=\frac{1}{\Delta T}\sum_{-\infty}^{\infty}\delta(\mu-\frac{n}{\Delta T})$$
所以f(t)采样后的函数$\tilde{f}(t)$的傅里叶变换为:
$$
\begin{aligned}
\tilde {F}(\mu) = F(\mu)\star S(\mu)=\int_{-\infty}^{\infty}F(\tau)S(\tau-\mu)d\tau \\
=\frac{1}{\Delta T}\int_{-\infty}^{\infty}F(\tau)\sum_{n=-\infty}^{\infty}\delta(\mu-\tau-\frac{n}{\Delta T})d\tau  \\
=\frac{1}{\Delta T}\sum_{n=-\infty}^{\infty}\int_{-\infty}^{\infty}F(\tau)\delta((\mu-\frac{n}{\Delta T})-\tau)d\tau \\
=\frac{1}{\Delta T}\sum_{n=-\infty}^{\infty}F(\mu-\frac{n}{\Delta T}) \\ 
\end{aligned}
$$

通过这个式子可以看出来$\tilde {F}(\mu)$在频域是连续的这是因为$\tilde {F}(\mu)$ 是由$F(\mu)$的几个拷贝组成的。
由此可知**当一个连续函数被采样成一个离散函数之后，其能够被重建为原函数的能力取决于采样点的密度(density of sample)，或称为采样率(sample rate)。** 对于一个带限函数$f(t)$其最大的频率为$u_{max}$,它能够被采样间隔$1/(2u_{max})$离散函数表示。如果采样率不足，就会导致走样。
#### 几何走样
**几何走样(geometric aliasing)**：由于像素点有大小导致    
**着色走样(shader aliasing)**： 在着色器中对一些分析的方式得到的连续函数的采样不足导致的    
**时间走样(temporal aliasing)**：由于渲染帧率的限制导致对运动过程的采样不足导致的走样  
### 重建
重建：离散信号转换为连续信号 
#### 平滑
利用卷积的特性可以用来平滑架设$f(x,y)$为一定分辨率的图像，$g(x,y)$为一个$m\times n$的矩形设$m=2a+1$和$n=2b+1$
$$g(x,y)\star f(x,y)=\sum_{s=-a}^{a}\sum_{t=-b}^{b}g(s,t)f(x-s,y-t)$$
#### 重建
重建滤波器(reconstruction filter)    
**盒装滤波器**   
$\quad\quad$ 设$\tilde{F}(\mu)$为$f(x)$采样后的离散函数$\tilde{f}(x)$的傅里叶变换,为了使$f(x)$能够被完美重建必须是用高于奈奎斯特采样率的频率进行采样，有盒装滤波器$H(\mu)$
$$F(\mu)=H(\mu)\tilde{F}(\mu)$$
$$
H(\mu)=\left\{
\begin{gathered}
\Delta T\quad {-u}_{nax}\leq\mu\leq u_{max} \\
0\quad 
\end{gathered}
\right.
$$
$$f(t)=\int_{-\infty}^{\infty}F(\mu)e^{j2\pi\mu t}d\mu$$  
盒装滤波器也被称为低通道滤波器(low-pass filter)，因为它通过的频率范围低端的频率，并且消除所有较高频率，因此想要完美复原函数那么这个函数必须是带限函数。
对：
$$
\begin{gathered}
f(t)=\mathcal F^{-1}\{F(\mu)\} \\
=\mathcal F^{-1}\{H(\mu)\tilde{F}(\mu)\} \\
=h(\mu)\star\tilde f(\mu) \\
(没搞懂怎么变换　直接写了结果如下)\\
=\sum_{n=-\infty}^{\infty}f(n\Delta T)sinc[(t-n\Delta T)/\Delta T]
\end{gathered}
$$
相当于对离散函数使用一个辛克函数(sinc function)
$$sinc(x)=\frac{\sin(\pi x)}{\pi x}$$
实际上盒状滤波器的傅里叶反变换就是辛克函数
#### 重采样
在渲染场景的时候，程序会大量使用预采样的数据，如图像，或者场景的某些离散的表示空间结构数据等。这些数据是有一定分辨率的，当程序中需要不同分辨率下使用这些数据，我们需要对这些函数进行重采样(resampling)。    
当摄像机靠近物体表面时，即发生放大操作(magnification)，这使摄像机能开到更多的细节。所以我们需要对原先的图像提供一个滤波器，可以对任意点进行插值。  
**双线性插值法(bilinear interpolation)**
假如我们想得到未知函数f在点P= (x,y) 的值，假设我们已知函数f在$Q_{11} = (x_1,y_1)、Q_{12} = (x_1,y_2),Q_{21} = (x_2,y_1) 以及Q_{22} = (x_2,y_2)$ 四个点的值那么：
$$
f(x,y_1)\approx\frac{x_2-x}{x_2-x_1}f(x_1,y_1)+\frac{x-x_1}{x_2-x_1}f(x_2,y_1)\\
f(x,y_2)\approx\frac{x_2-x}{x_2-x_1}f(x_1,y_2)+\frac{x-x_1}{x_2-x_1}f(x_2,y_2)\\
f(x,y)\approx\frac{y_2-y}{y_2-y_1}f(x,y_1)+\frac{y-y_1}{y_2-y_1}f(x,y_2)
$$