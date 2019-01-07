---
layout: post
title:  "Real_Time Rendering Thried 笔记"
date:   2018-06-18 21:22:02
categories: MacroDog
tags:  ComputerGraphies 
excerpt: Real_Time Rendering Thried Editor  
mathjax: true
---
* content
{:toc}
# Real_Time Rendering Thried Editor  
  

## Chapter5 Visual Appearance

### 5.1 Visual Phenomena
1.光被光源产生
2.光与物体交互在场景中。一部分被吸收一部分被分散。
3.最终光被传感器吸收(这里的传感器值得是人类眼睛，电子传感器或是胶片等等 接受光所包含信息的物体)

### 5.2 Light Sources
1.辐射照度 irradiance    
辐射源在某一方向的单位投影面积单位立体角单位波长的辐射能量，称为辐射亮度         
$$
E =\sum_{k=1}^n E_{Lk}\\cos\theta
$$    


### 5.3 Material
材质关联 shader程序,纹理 和一起其它属性，目的是模拟光与世界交互模型。     
光与材质交互的最终结果是$\textbf{散射}$和$\textbf{吸收}$。    
当光从一个截止到另个一个介质当中时光会发生折射或transmission和反射。
### 5.4Sensor
光产生后就会在空间弹射，一些就被会sensor吸收。但是，sensor本身不会产生图像，因为均匀的光线从不同方向传入。因此完整的图像系统包含enclosure,aperture,lens。    
这一节里提到了 辐射亮度(radiance)这一物理变量这里稍作解释：     
1.方向和立体角    
以观测点为球心，构造一个单位球面；任意物体投影到该单位球面上的投影面积，即为该物体相对于该观测点的立体角。
因此，立体角是单位球面上的一块面积，这和“平面角是单位圆上的一段弧长”类似。
点在球坐标系表达方式为
$$(r,\theta,\phi)$$    
其中r为球面半径$$\theta$$ 为圆心到点与笛卡尔坐标系中的z轴正方向的夹角，圆心到$$\phi$$是点在xoy平面上的投影点与x轴正方向夹角。
![Image]({{site.url}}/image/postImage/RealTimeRenderingThriedEditor/5_4_1.jpg)
对应的转换为:

$$x=rsin\theta \cos\phi$$     

$$y=rsin\phi sin\theta$$     

$$z=r\cos\theta$$    

所以在球坐标系中，任意球面的极小面积为:    

$$\mathrm{d}A=(rsin\theta\mathrm{d}\phi)(r\mathrm{d}\theta)$$

因此，极小立体角（单位球面上的极小面积）为:    

$$\mathrm{d}\omega=\mathrm{d}A/r^2$$

$$\mathrm{d}\omega=sin\theta\mathrm{d}\phi\mathrm{d}\theta$$

所以，立体角是投影面积与球半径平方值的比，这和“平面角是圆的弧长与半径的比”类似。 对极小立体角做曲面积分即可得立体角：

$$\omega=\iint_{S}\mathrm{d}\omega=\iint_{S}sin\theta\mathrm{d}\theta\mathrm{d}\phi$$

2.辐射亮度 radiance    
辐射度指的是沿辐射方向、单位面积、单位立体角上的辐射通量。即：通过每个每个平面和方向的光密度。用于表示光照强度的
### 5.5 Shading
shanding是一种程序结合光源，材质和视角方向使用方程计算输出的辐射亮度。   
shading方程式一般有两种diffuse和specular。    
1.Diffuse

$E_L$是辐射亮度$c_{diff}$是diffuse Color

$$M_{diff}=c_{diff}\otimes E_L \overline\cos\theta_{i}$$    

$$L_{diff}=\frac{M_{diff}}{\pi}$$    

$$L_{diff}=\frac{c_{diff}}{\pi}\otimes E_L\overline\cos\theta_{i}$$

2.Specular    
$$c_{spec}$$镜面反射颜色l为点到光源的方向v为点到观察点的方向

$$M_{spec}=c_{spec}\otimes E_L\overline\cos\theta_{i}$$

$$h=\frac{l+v}{\left|\left|l+v\right|\right|}$$

$$\theta_{h}$$是h与点法线之间的夹角，m是用于控制镜面反射的高光效果，m越大高光区域就越小越亮

$$L_{spec}(v)= \frac{m+8}{8\pi}\overline\cos^m\theta_{h}M_{spec}$$

$$L_{spec}(v)= \frac{m+8}{8\pi}\overline\cos^m\theta_{h}c_{spec}\otimes E_L\overline\cos\theta_{i}$$

结合Diffuse和Specular可以得出输出的辐射亮度$$L_o(v)$$

$$L_o(v)=\frac{m+8}{8\pi}\overline\cos^m\theta_{h}c_{spec}\otimes E_L\overline\cos\theta_{i}+\frac{c_{diff}}{\pi}\otimes E_L\overline\cos\theta_{i}$$   

$$L_o(v)=(\frac{c_{diff}}{\pi}+\frac{m+8}{8\pi}\overline\cos^m\theta_hc_{spec})\otimes E_L\overline\cos\theta_i$$

这个函数很像Blinn-Phong模型 如下：

$$L_o(V)=(\overline\cos\theta_ic_{diff}+\overline\cos^m\theta_hc_{spec})\otimes B_L$$

这里要注意的是在blinn—phong模型中$B_L$并不等于$E_L$。因为blinn-Phone模型中并不是物理光照模型。

#### 5.5.1 Implementing the shading Equation
因为实际应用当中光线一般并不是单个光源，大部分时候还是多个光源与场景交互所以需要对结果进行一次求和。

$$L_o(v)=\sum_{k=1}^n((\frac{c_{diff}}{\pi}+\frac{m+8}{8\pi}\overline\cos^m\theta_{h_k}c_{spec})\otimes E_{L_k}\overline\cos\theta_{i_k})$$

由于光源很多所以可以利用shader的dynamic branching capabilities去遍历光源。    
在设计shading的实现的时候，这些计算需要通过计算的频率(frequency of evaluation)被分为不同的级别。最低的等级为每个模型(per-model),这个级别当中每个3D模型将会被计算(evaluation)一次在application阶段，并将结果传递到graphics API。其次每片源(per-primitive)计算一次在geometry shader当中，计算的结果将会全部被pixels使用在primitive中。然后是每顶点(pre-vertex)在定点着色器上被计算一次，结果将被线性插值传入pixel shader中。最后，就是每像素(per-pixel)在pixel shader当中计算。
所以可以将之前算法中的per-modle分离出来。

$$L_o(v)=\sum_{k=1}^n((K_d+K_s\overline\cos^m\theta_{h_k}c_{spec})\otimes E_{L_k}\overline\cos\theta_{i_k})$$

$$K_d = \frac{c_{diff}}{\pi}$$

$$ K_s=\frac{8+m}{8\pi}$$

&k_s& &k_d&都是在application阶段被计算好的可以直接使用API直接访问 ，$\overline\cos\theta_{i_k}$可以通过$l_k\cdot n$获得,同理$\overline\cos\theta_{h_k}$也可以通过$h_k\cdot n$获得视角向量v 也可以通过顶点P和观察者$P_v$位置得到

$$v=\frac{P_v-P}{\left|\left|P_v+P\right|\right|}$$

1.Flat Shading 
对整个三角面只计算一次光照值。通常，计算光照的“位置”为三角形中心，表面法向量为三角形法向量
2.Gouraud Shading     
对于一个三角面上的三个顶点颜色进行插值决定三角面颜色，所以会在vertex shader 中传出世界坐标系下的法线和位置，归一化法线，结果经过线性插值在pixel shader中输出。但是这种输出的结果并不完美，对于线性的值来说没有什么问题，但是遇到非线性值比如 镜面高光（specular） 。线性插值基本问题是内插值不可能大于最大的顶点值，所以高光只能在顶点出现。
3.Phone shading    
Phone shading 是一个完全的pixel shding 因此性能消耗很高。在vertex shading 中产生世界坐标系下的法线和位置，pixel shading 中接受结果并输出。

### 5.6 Aliasing and Antialiasing    
出现走样的原因就是集合图形中线是由无数没有大小的点组成的，而在计算机显示当中是由有大小的像素点组成的。    
#### 5.6.1 Sampling and Filtering Theory    
渲染一张图片是对一个3维的场景将一个连续的信号采样为一个离散的信号。然后，在被重建成连续信号。重建这一过程称为，过滤。   
事实上出现走样这种现象的主要原因就是对于一个连续信号采样的频率过低。     
为了防止这种现象出现奈奎斯特(nyquist)提出了采样频率必须大于信号最大频率的两倍。这一理论也被称为Nyquist rate或是Nyquist limit。    

$$C=B*\log_2N(bps)$$

但是无论采用多高的频率去采样，我们仍然无法完全消除走样，当我们使用点采样去渲染图形的时候。有一点要注意的是当采样的频率越高占用的带宽也就越高。     
Reconstruction     
当我们将一个采样信号重建的时候就要使用到过滤器(filter)。过滤器有很多种，但是当使用时只能使用一个否者就会出现伸展或者压缩。     
Box Filter    
因为非常简易被经常使用在计算机图形学当中。     
Box Filter 放置在每一个采样点，过滤后的值都取最高的采样点值（采样的图很像柱状图）
Tent Filter 
Sinc Filter      
ReSamlping    
这一过程是对重建后的连续信号在次采样获得所需的数据。有这一过程的原因是，在计算机中我们是没有办法直接显示出连续信号的。有这一过程我们也可以对型号进行一些操作比如放大(Magnify)和缩小(Minify)。    
Magnify     
这一过程很好理解即，使用对原始型号尽量不损失的较低频率采样，然后rescontruction,再用高频率的采样获得信号。(相当于压缩了)      
Minify    
这一过程就是更高频率采样然后重建连续信号。再用低频率采样这样获得的信号就会比原来的丢失一些信号，使图像模糊，可以适应更低的分辨率。    
（ps:这里的放大是指采样频率增加, 缩小是采样频率减少）
##### 5.6.2 Screen-Based Antialiasing
走样经常发生在图元边缘，阴影边界，镜面反射高亮等等一些颜色改变剧烈的情况。解决这种情况的方法有很多方式，有一些方法是基于屏幕图像进行处理。基于屏幕图像处理的好处就是，并不需要关心模型本身如何被渲染。      
一些反走样方案聚焦于图元的渲染管线，比如：texture aliasing和 line aliasing  这里说一下line aliasing 对于线我们的处理方式一种是把线当成一个像素宽的长方体。另外一种是当成无限细带有光环(halo)？的透明物体。第三种是渲染线作为一张antialiased texture。这些都是基于屏幕的反走样方案，最流行的线反走样硬件可以提供更快质量跟高的渲染。具体可以参考 Nelson's article, Chan and Durand 提供了 GPU-specific的方案用于 prefiltered lines(预过滤线条)。    
对于一般的渲染只有一个采样点在每一个像素中这导致这个像素是否被上色完全取决与中心点是否被片元覆盖。但是如果有更多的采样点并且给采样点附上权重在相加最终获得该像素点颜色。  

$$p(x,y)=\sum_{i=k}^nw_ic(i.x,y)$$

(x,y)像素在屏幕的位置，n是采样点数量，function c(i,x,y)采样点颜色，$w_i$是采样点权重。对于屏幕网格上每一个点如何得到位置都有不同的方法，采样的参数也会改变。一般采样是实时的。所以，函数C可以被当成两个函数，第一个函数是f(i,n)用来检索查找，采样点在屏幕上的位置。    
对每一帧整个屏幕上所有像素点采样的方法被称为超级采样方法。概念上最简单的是FSAA(Full-Scene Antialiaing)    
FSAA    
渲染场景更高分辨率然后平均相邻的采样最终渲染出图像。例如，如果想要渲染出一个1000x800，先在幕后渲染出2000x1600然后每一个2x2区域进行采样平均，最终的呈现出来的图像是由4个像素的采样获得。supreSampling生成的样本带有各种计算的shades，depths和location所有极其昂贵。FSAA的优势是简单。除了2x2采样还可以2x1或者1x2这种沿着一个轴的方向采样。
与FSAA相关的另一种方法是Accumulation Buffer，并不是使用更大分辨率缓冲，而是相同分辨率，但更多位颜色。得到2x2采样那么就生成四张图像，每个视角移动x轴或y轴移动半个像素。每个图像生成都在不同的采样点。这些图像被求和在 accumulation buff 中然后加权平均显示在显示器上。Accumulation Buff是OpenGL Api 一部分。可以用来实现动态模糊。代价是在一帧当中渲染多次。
现代GPU中不一定有accumulation buff,但是可以通过像素混合多个图像来模拟。Accumulation Buff 还有一个有点就是不需要每一个像素网格单元不一定需要一个直角模式，每个pass都是相互独立的，所以可以交替使用 sampling pattern, 例如使用一个旋转的正方形 pattern 如(0, 0.25), {0.5, 0}, {0.75, 0.5}, (0.25, 0.75), 有时称呼这个为 rotated grid supersampling(RGSS), 该方法对于接近于垂直和水平的边沿的反锯齿效果更好。事实上Naiman发现产生锯齿很多时候是在水平和垂直边缘，然后是45度角。     
supersampling技术带有各自独立生成的shader和，深度和位置信息。代价太高收益太低。也正是这个原因DirectX 并不直接支持supersamlping作为反走样方法。
SSAA
SSAA直接将图像映射到缓存中并将其放大。再用超级采样把放大后的图像进行采样，一般选区2个或4个邻近像，把颜色混合起，生成最终的像素，使每个像素
拥有邻近像素的特征。这样会让图像显得平滑。
MSAA
MSAA是一种特殊的超级采样，MSAA概念来自与OpenGL。MSAA只对Z-Buffer和Stencil-Buffer中的数据进行超级采样操作。可以理解位只对多边形的边缘进
行抗锯齿处理。
CSAA















