---
layout: post
title:  "RealTimeShadow"
date:   2020-05-04 09:56:33
categories: MacroDog
tags: GAMES202-高质量实时渲染
excerpt: GAMES202
mathjax: true
---
* content
{:toc}

### ShadowMap
#### 1.Reader form light
从光源渲染一张深度纹理
#### 2.Render form eye
通过计算像素点的深度与深度纹理对比判断是否显示
Fragment中像素点z在透视投影中是经过投影变换过。在对比中要注意shadow map 记录的实际距离还是经过mvp变换后的深度。
#### Issues In Shadow Mapping
self occlusion
由于记录深度是记录的深度纹理是离散的导致在从视角渲染的时候fragment中获得深度不准确
![tu1](https://github.com/MacroDog/MacroDog.github.io/blob/master/image/postImage/Game202/1620111577.jpg)
解决方案：
增加容错值
生成深度纹理的时候不仅保存最小深度还有第二深度，比较的时候使用中间深度比较