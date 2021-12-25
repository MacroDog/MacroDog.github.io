---
layout: post
title:  "RealTimeShadow"
date:   2021-05-04 09:56:33
categories: MacroDog
tags: GAMES202-高质量实时渲染
excerpt: GAMES202
mathjax: true
---
* content   
{:toc}

# Shadow Mapping
## Hard Shadows
### Generate Shadow Mapping
#### 1.Reader form light     
从光源渲染一张纹理记录深度值      
![Image](https://github.com/MacroDog/MacroDog.github.io/blob/master/image/postImage/Game202/1622886355.jpg)     
#### 2.Render form eye       
通过计算像素点的深度与深度纹理对比判断是否显示
（注：Fragment中像素点z在透视投影中是经过投影变换过。在对比中要注意shadow map 记录的实际距离还是经过mvp变换后的深度。）     
![Image](https://github.com/MacroDog/MacroDog.github.io/blob/master/image/postImage/Game202/1622886326.jpg)                 
### Issues In Shadow Mapping       
#### 1.Self Occlusion         
![Image](https://github.com/MacroDog/MacroDog.github.io/blob/master/image/postImage/Game202/1622886662.jpg)    
由于记录深度是记录的深度纹理是离散的导致在从视角渲染的时候fragment中获得深度不准确如下图渲染红色像素的时候获取深度值时认为黄色像素点的深度小于红色有遮挡关系           
![Image](https://github.com/MacroDog/MacroDog.github.io/blob/master/image/postImage/Game202/1620111577.jpg)    
**解决方案**      
**a.Adding a (variable) bias to reduce self occlusion**         
增加一个可变（主要是和光源和阴影接受面角度）的容错值
![Image](https://github.com/MacroDog/MacroDog.github.io/blob/master/image/postImage/Game202/1622887060.jpg)    
但是也会有其他问题          
![Image](https://github.com/MacroDog/MacroDog.github.io/blob/master/image/postImage/Game202/1622886662.jpg)    
**b.Second-depth shadow mapping**       
生成深度纹理的时候不仅保存最小深度还有第二深度，比较的时候使用中间深度比较。    
![Image](https://github.com/MacroDog/MacroDog.github.io/blob/master/image/postImage/Game202/1622887299.jpg)         
当然实际项目基本不会采用这种方式太复杂了   
#### 2.Aliasing
![Image](https://github.com/MacroDog/MacroDog.github.io/blob/master/image/postImage/Game202/1622888187.jpg)     


### Approximation in RTR
一些在实时渲染当中使用的近似方程      
$$ \int_{\omega}f(\mathcal{ x})g(\mathcal{x})d x\approx \frac{\int_\omega f(\mathcal{x})dx}{\int_\omega dx}\cdot \int_\omega g(\mathcal{x})dx$$
约等式成立      
1.积分区域$\omega$很小      
2.$g(x)$平滑 即在范围内变化不大     
## Soft Shadow
### Percentage Closer Filtering(PCF)
图像空间中对像素点取深度值时考虑取得相邻的深度值（filter）加权平均获得结果。    
表达式：    
$$[\mathcal{w}*\mathcal{f}](p) = \sum_{q\in N(p)}\mathcal{w}(p,q)f(q) $$
p是在深度贴图中计算阴影的点    
q是与p相邻点    
$\mathcal{w}$通过p,q两点关系获得的加权值    
$\mathcal{f}$点在深度贴图上的值     

### Persentage Closer Soft Shadows(PCSS)   
半影的范围和渲染点与遮挡物之间的距离存在关系   
![Image](https://github.com/MacroDog/MacroDog.github.io/blob/master/image/postImage/Game202/1622191695.jpg)    
$$ w_{Penumbra} = (d_{Receive}-d_{Blocker})\cdot w_{Light}/d_{Blocker}$$       
$w_{Penumbra}$ 半影的大小   
$d_{Blocker}$遮挡物到光源的距离     
$d_{Receive}$光源到点距离   
step 1: Blocker Search   
以shading point为中心获得在当前区域中平均深度深度       
step 2: Penumbra Estimation    
使用半影和平均深度关系计算获得半影的范围    
step 3: Percentage Closer Filtering(PCF)       
表达式:    
$$V(x)=\sum_{q\in\mathcal{N(p)}} \mathcal{w}(p,q)\cdot \chi^+{\mathcal{[D_{SM}(q)-D_{scene}(x)]}}$$       
![Image](https://github.com/MacroDog/MacroDog.github.io/blob/master/image/postImage/Game202/1622192485.jpg)
由于第一步和第三步比较慢大需要加速   
### Variance Soft Shadow Mapping  
**主要思想**    
在PCF中如果不加权计算的话实际上是为了获得在当前区域中有多少像素的深度小于当前像素的深度（遮挡）。可以利用当前区域中正太分布快速获得。正太分布需要当前区域的期望和方差。  
**期望（Mean）**    
-mipmap    
-Summed Area Tables  
**方差（Verance）**     
根据方差，期望关系
$$Var(X) = E(X^2)-E^2(X) $$   
方差等于平方期望减去期望的平方   
在生成Shaddow Map中计算方差可以写在其他通道里。     
CDF     
![Image](https://github.com/MacroDog/MacroDog.github.io/blob/master/image/postImage/Game202/1622383979.jpg)    
获得当前点像值   
但是计算CDF是个非常麻烦的事情VSSM使用另外一个方法获取在当前区域中有多少像素的深度小于当前像素的深度         
#### 切比雪夫不等式   
![Image](https://github.com/MacroDog/MacroDog.github.io/blob/master/image/postImage/Game202/1622384038.jpg)    
$$P(x>t)\leq\frac{\sigma^2}{\sigma^2+(t-\mu)^2}$$   
$\mu$期望    
$\sigma$ 方差   
（ps:查询的值必须在期望的右边否则不准，即查询深度必须大于平均深度）    
**性能**    
-范围内深度期望O(1)    
-范围内深度期望平方O(1)    
-切比雪夫不等式O(1)     
**VSSM数学表达式**      
![Image](https://github.com/MacroDog/MacroDog.github.io/blob/master/image/postImage/Game202/1622473953.jpg)  
$Z_{Avg}$平均深度
$z_{occ}$ 区域中深度小于渲染点平均深度（图中蓝色部分）      
$z_{unocc}$区域中深度大于渲染点深度平均深度（图中红色部分）         
$N$区域中像素数量       
$N_1$区域中像素深度大于渲染点深度平均深度像素个数       
$N_2$区域中像素深度小于渲染点深度平均深度像素个数    
$$
N_1+N_2 = N\\
\frac{N_1}{N}z_{unocc}+\frac{N_2}{N}z_{occ} = Z_{Avg}
\\
\frac{N_1}{N}\approx P(x>t)
$$  
$$
\frac{N_2}{N}=1- \frac{N_1}{N}
\\
z_unoc=t （这里认为对于没有遮挡的深度默认取值认为渲染点深度）
$$              
#### MIPMAP For Range Query
![Image](https://github.com/MacroDog/MacroDog.github.io/blob/master/image/postImage/Game202/1622878868.jpg)  
对于不正好处于区域块的轴对齐方形区域需要各向异性处理 
Note: still approximate even with trilinear interpolation       
#### SAT of Range Query      
**前缀和算法(prefix sum)**      
对于一维数组数组存储的是之前数字的和加上需要数值 例：   
![Image](https://github.com/MacroDog/MacroDog.github.io/blob/master/image/postImage/Game202/1622878868.jpg)     

方便数组之间连续值求和                
对于二维图像要两边存储第一遍X轴然后得到数组再以Y轴做一次最后得到SAT     
![Image](https://github.com/MacroDog/MacroDog.github.io/blob/master/image/postImage/Game202/1622881283.jpg)    
（ps:二维数组之间行与行，列于列之间不存在计算关系可以并行。累加可能损失数值精度）

#### Revisit: VSSM       
VSSM也有自己的问题  
**1.正态分布描述区域内深度分布不准**      
![Image](https://github.com/MacroDog/MacroDog.github.io/blob/master/image/postImage/Game202/1622880397.jpg)         
在左图的情况下 深度符合正太分布关系时效果不错但是在右图不符合正太分布的时候效果很差         
由于不符合关系就会导致计算平均深度不准导致实际效果偏黑或者偏白(LIGHT LEAKING)如下图蓝色部分是实际深度分布，红色是计算的分布导致实际平均深度大于计算出的平均深度     
![Image](https://github.com/MacroDog/MacroDog.github.io/blob/master/image/postImage/Game202/1622880951.jpg)          
实际情况            
![Image](https://github.com/MacroDog/MacroDog.github.io/blob/master/image/postImage/Game202/1622881553.jpg)      
图中车底有漏光            
**2.阴影接受非平面**             
![Image](https://github.com/MacroDog/MacroDog.github.io/blob/master/image/postImage/Game202/1622882160.jpg)  
**3.切比雪夫不等式局限性**
查询深度必须大于平均深度否则不准                   
### Moment Shadow Mapping  
**//TODO 这部分没有搞明白课程中只说了结论没有解释原理 只是简单复制粘贴一下 收集了些资料之后学习  [MomentShadowMapping](https://cg.cs.uni-bonn.de/aigaion2root/attachments/MomentShadowMapping.pdf)**            
解决正态分布描述区域内深度分布不准问题，引入高阶矩(Moments)。
主要思想**
引入高阶矩描述分布
**Moments**         
采样数的点的次方    
$x,x^2,x^3,x^4,...$         
（VSSM使用了前两阶的矩）
- Conclusion:
first $m$ orders of moments can represent a function with $m/2$ steps
Usually, 4 is good enough to approximate the actual CDF of depth dist.    
![Image](https://github.com/MacroDog/MacroDog.github.io/blob/master/image/postImage/Game202/1622883156.jpg)         
### 作业笔记
#### PCF

# BRDF推导
辐射亮度（Radiance,符号为L）是指辐射源在某一个方向，每个单位投影面积，每单位立体角内的辐射通量，其单位为瓦特/(球面度*平方米)即$W/(steradian*m^2)$       
$L(x,\Theta)$   
$\Theta$为方向 x是方向上的一个点$\Phi$是单位时间从某一表面发射或到达某一表面的总能量流量，焦耳每秒(J/s)
$d\omega$球面度
$$
d\omega = sin{\theta}d{\theta}d{\phi}
$$ 
Irradiance       
Definition: power per unit area     
$E(x)=\frac{d\Phi}{dA}$     
Radiance 
Definition:power per unit solid angle per projected unit area   
$L(p,\omega)=\frac{d^\Phi(p,\omega)}{{d\omega}{dAcos\theta}}$

$E(p) = {\int} _HL_i(p,\omega)cos\theta dw$     
**BRDF**    
入射光在平面上向不同方向上反射的能量分布    
$f_r(\omega_i\rightarrow\omega_r)=\frac{dL_r(\omega_r)}{dE_i(\omega_i)}$
# Probability Density Function
对于随机变量x
# Monte Carlo Integration
蒙特卡洛积分
对于一个函数f(x)
# 马尔科夫链

