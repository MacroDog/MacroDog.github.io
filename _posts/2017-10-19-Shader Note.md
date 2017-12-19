---
layout: post
title:  shader笔记
---
#shader笔记

标签（空格分隔）： 图形渲染

[TOC]
## Unity's Rendering Pipeline
###Pass
####syntax
>语法
`Pass{[Name and Tags] [RenderSetup]}`
#### Name and tag
>Pass 可以被命名
` Name "PassName"  ` 
可以定义任意个Tag
`Tags { "TagName1" = "Value1" "TagName2" = "Value2" }`
####Render state set-up
#####Cull
>`CullBack|Front|Off`用来设定剔除遮挡
#####ZTest
>`ZTesy(Less|Greater|LEqual|GEqual|Equal|NotEqual|Always)`将当前顶点深度值与深度缓存里的值进行比较如果不通过测试则不会执行Pass
#####ZWrite
>` Zwrite On|Off`
将当前深度值写入深度缓存当中关闭则不会写入，值得注意的是深度缓存只会存储之前渲染离摄像机最近的顶点深度值。
#####Offset
 >`Offset OffsetFactor,OffsetUnits`
 这个函数一般用于当两图形刚上处于同坐标或两个面在同一坐标重叠时。可以通过这个函数让其中一个图形或面的坐标自动偏移。
 ####Blend
`Blend sourceBlendMode destBlendMode
Blend sourceBlendMode destBlendMode, alphaSourceBlendMode alphaDestBlendMode
BlendOp colorOp
BlendOp colorOp, alphaOp
AlphaToMask On | Off`
控制颜色混合
#####ColorMask
>`ColorMask RGB |A |0| any combination of R,G,B,A`
设置颜色通道写入遮罩 如果是ColorMask 0 则表示关闭所有颜色通道渲染，默认是写入所有通道(RGBA)
####Legacy fixed-funtion shader commands
>Fied-function lighting and Material
`Lighting On|Off
Material {Material Block}
SeparateSpecual On|Off
Color Color-value
ColorMaterial AmbientAndDiffuse|Emission`
>Fixed-function fog
`Fog {Fog Block}`
>Fixed-function AlphaTest
    `AlphaTest(Less|Greater|LEqual|GEqual|Equal|NotEqual|AlWays)`
>Fixed-function Texture Combiners
`SetTexture textureProperty{combine options}`
##Shader入门
模板测试：一般GPU会有一块区域用于缓存对应屏幕中点的颜色值。如分辨率1024x768 那么模板缓   存区的单位数也应为1024x768
深度测试：用来记录定点的深度信息用于混合（blend） 计算出应有的颜色值。
SubShader tags
{
Queue /*Tags{"Queue" = "Transparent"}*/
ReaderType /*Tags{"ReaderType" = "Opaque"}*/ //用于分类
DisableBatching/*Tags{"DisableBatching" = "True"}*/ //是否使用批处理s
}
Unity 中如果声明一个2D变量,Unity会为其自动添加一个_ST float4 用于其缩放和偏移。 如 ：
_MainTex (Unity自动生成_MainTex_ST)

多级渐远纹理(mipmapping)：在真实世界当中人所看到近处与远处的图像清晰度是不一样的。为了模
拟这只效果。minmap技术将每一层对上一层的降采样。实时运算时更加快速。但代价就是需要
花费更多的空间储存多级渐远纹理大约多出33%。

mipmapping在unity 可以通过Texture中TextureType选为Advanced  勾选Generate Mip Maps 有两  
种模式Point,Bilinear,Trilinear 。point 模式通常对附近一个像素经行采样。所以效果看起来是像
素风格。bilinear 采用线性滤波通常会找到4个邻进的像素，然后对他们进行线性插值混合后得到
最终像素。效果就像被模糊了。Trilinear滤波会在多级渐远纹理上进行混合如果纹理没有使用多
级渐远纹理技术Trilinear得到的结果和Bilinear一样。

凹凸映射：其目的是为了改变模型表面的定点法线（改变法线是为了再不增加定点的情况下增加模型    
的细节。因为光照效果是由光向量与法线向量计算，改变定点法线向量能够产生欺骗眼睛的效           
果）

高度纹理： 使用一张纹理图的灰度值来存储，实际计算中根据高度纹理修改法线值。优点是直观。缺
点是计算复杂消耗性能。 

法线纹理：法线纹理储存的就是表面的法线方向（由于法线方向范围为[-1,1]像素分量为[0,1]所以要进 
行一个映射）实际应用中有两种纹理 分别是模型空间法线纹理和切线空间法线纹理。

模型空间法线纹理：将模型空间下的法线信息直接存入一张纹理。

切线空间法线纹理：以每个顶点的切线作为x轴法线方向作为z顶点为原点建立坐标系  法线空间好处
是由于当前纹理存储的是变量如果法线纹理与模型法线不发生变化那么纹理就是浅蓝色很容易识
别改变的地方。
切线空间计算注意点：切线空间三个轴是有顶点切线,定点的法线，定点的法线与切线的叉乘构成。需
要注意的是由于切线与法线的叉乘的方向得到与切线和法线垂直的向量方向是有2个方向所以需  
要通过顶点切线的w分量确定
需要将光照方向向量，视角方向向量转换到切线空间下。切换空间 unity中提
供了一个内置宏TANGENT_SPACE_ROTATION(在UnityCG.cginc中定义)来帮助我们直接获得
转化到切线空间的转换矩阵其原理就是一个3x3矩阵包含顶点的切线，切线与法线叉乘，法线
float3x3(v.tagent.xyz,cross(v.tangrnt.xyz,v.normal)*v.tangent.w,v.normal)
世界空间： 
向前渲染
    在unity中向前渲染中有三种处理光照的方式：逐顶点处理 ，逐像素处理，球谐函数（Spherical Harmionics）在Unity 场景中最亮的平行光按照逐像素处理。渲染模式被设置成NotImportant的光源将被逐顶点或者球谐函数处理。渲染模式被设置成为Important的光源会按逐像素处理。如果根据以上规则得到的逐像素光源数量小于QualitySetting中的逐像素光源数量 会有更多的光源以逐像素方式进行渲染。
  向前渲染中有两种Pass: Base Pass 和Additional Pass 
_LightColor0 float4类型 当前Pass逐像素光源的颜色
_WorldSpaceLightPos0 float4类型 xyz表示当前Pass处理的逐像素光源的位置 w如果为0表示平行光
1则为其它
####shader中的默认函数
>_LightMartix0
>>float4x4类型从世界空间到光源空间的变换矩阵。可用于采样cookie和光强衰减(attenuation)纹理处理

>unity_4LightPosX<sub>i</sub>,unity_4LightPosY<sub>i</sub>,unity_4LightPosZ<sub>i</sub>,unity_4LightPosW<sub>i</sub>(0<=i<=3)  
>>类型float4  仅Base Pass。前4个非常重要的点光源在世界空间中位置
unity_4LightAtten0 float4类型仅用于BasePass 粗存前四个点光源的衰减因子
unity_LightColor half[4] 类型 储存类前四个重要点光源的颜色 

>float4 WorldSpaceLightDir(float4 v)  
>>输入模型空间中的顶点位置返回世界空间从改点到光源的光照方向

>>float3 UnityWorldSpaceLightDir(float4 v)

>>输入世界空间中的顶点位置返回该点到光源的光照方向

>float3 ObjSpaceLightDir(float 4)
>>输入一个模型空间中的顶点位置返回模型空间中从改点到光源的光照方向

>float3 Shade4PointLights(...) 
>>计算四个点光源的光照。。。

>float3 lightCoord = mul(_LightMatrix0,float(i.worldPosition,1)).xyz;
>>unity中光照的衰减是使用一张纹理(_LithtTexture0)作为查找表来在片元着色器中计算逐像素衰减。
通过_LightMatrix0将顶点转换到光照空间下 。然后使用坐标的模的平方进行纹理查询 ：
float3 lightCoord = mul(_LightMatrix0,float(i.worldPosition,1)).xyz;
float atten = tex2D(_LightTexture0,dot(lightCoord,lightCoord).rr).UNITY_ANNTEN_CHANNEL;

>float3 WorldSpaceViewDir(float4 v)
>>输入一个模型空间返回世界坐标系该点到摄像机的观察方向  

Unity的阴影


####10.1高级纹理
Cubemap
HDR  high-Dynamic Range 比普通的图像能够提供更多的动态范围和图像细节
reflect 根据反射Dir 纹理查询Cunemap 得到反射的图像
####10.2 渲染纹理
    现代GPU允许我们把整个三维场景渲染到一个中间缓存中，即渲染目标纹纹理(Render Terget Texture,RTT)，而不是传统的帧缓冲或者后背缓冲(back buffer)。与之相关的是多重渲染目标(Multiple Render Target,MRT)，这种技术指的是GPU允许我们把场景同时渲染到多个渲染目标纹理中，而不再需要为每个渲染目标纹理单独完整的场景。延迟渲染就是使用多重渲染目标的应用。
10.2.2玻璃效果
    在unity中，我们可以在UnityShader中使用一种特殊的Pass来完成屏幕图像的目的---GrabPass
语法：
`GrabPass{"_TextureName"}` properties 定义一个纹理_TextureName，使用该语句 unity会把当前屏幕的图像绘制在纹理上这样我们可以在后续的pass中访问。值得注意的是要设注意施渲染队列，即使GrabPass不包含Blend指令但是我们仍然将渲染队列设置为透明`"Queue"="Transparent"`这样可以保证透明物体可以被抓取到。
####10.3程序纹理
   **程序纹理(Procedural Texture)**指的是由计算机生成的图像
unity3D 脚本中 通过object上的render访问material Texture2D类用于持有纹理。通过setPixel(w,h,Color)来设置颜色最后调用Apply()去应用之前的改变。
 纹理生成工具Substance Designer
###11 Shader动画
#### 11.1 UnityShader 中的内置时间
|名称|类型|描述
|----|:----|:----
|_Time|float4|t是自该场景开始加载4个分量的值分别是(t/20,t,2t,3t) t的单位是秒。
|_SinTime|float4|t是时间的正弦值，4个分量分别是(t/8,t/4,t/2,t)
|_CosTime|float4|t是时间的余弦值4个分量分别是(t/8，t/4,t/2,t)
|unity_DeltaTime|float4|dt是时间增量，4个分量的值分别是(dt,1/dt,smoothDt,1/smoothDt)
####11.2 纹理动画
   
##名称代码段解释
### Render Target
渲染目标是一个缓冲，显卡使用一个Effect类绘制场景的像素。
默认的渲染目标叫后备缓存(BackBuffer)物理上就是下一帧要绘制的信息的一块显存。一个渲染目标具有高和宽。后背缓存的宽和高就是游戏的最终分辨率（有些平台可能需要进行一些缩放）!
###Stencil Test
模板测试：一般GPU会有一块区域用于缓存对应屏幕中点的颜色值。如分辨率1024x768 那么模板缓   存区的单位数也应为1024x768
###multi_compile_fwdbase
