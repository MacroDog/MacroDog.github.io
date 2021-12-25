---
layout: post
title:  "BRDF推导与实践"
date:   2021-10-30 14:50:00
categories: MacroDog
tags:  URP
excerpt: URP
mathjax: true
---
* content
{:toc}
### UnityPer..
一些内置参数用来表明CBUFFER block 参考[https://docs.unity3d.com/Manual/SRPBatcher.html]   
UnityPerCamera  
UnityLighting   
UnityShadows    
UnityPerDraw       
UnityPerFrame   
UnityPerMaterial 使用只能是在shader中定义的property
### CBUFFER
#### Constant Buffer
允许通过一块大小可变的buffer向shader提供常量数据。在一次draw call的执行过程中都是不变的，在不同的draw call之间，我们可以修改其中的数据。单个Buffer大小限制在64KB    
SRP中可以使用CBUFFER关键字使用
CBUFFER_START(name)
value
CBUFFER_END     
CBUFFER
### TEXTURE2D_SHADOW
在一些图形API中等于TEXTURE2D效果
### SAMPLER_CMP
一般来说，我们在Shader为了采样，会使用samper2D来声明，不过实际上这一举动导致绝大部分情况下，纹理和采样器是耦合在一起的，不过也没关系，我们就是要这么用的，而且旧的图形API（如OpenGL ES）上这是唯一支持的做法。

另一种情况是纹理和采样器是相互独立的。没错，因为有些图形API允许使用比纹理更少的采样器，比如D3D11就支持单个着色器中使用128个纹理，但是只能有16个采样器。这种情况下，需要特殊的命名规范：具有【“sampler”+ 纹理命名】形式名称的采样器将从该纹理中获取采样状态。

### 级联阴影 （CSM）
用多张阴影贴图实现阴影
1.在摄像机空间中用深度分割不同区域
2.在光源空间中生成每个视锥体分段的“包围盒”。