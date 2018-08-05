---
layout: post
title:  "Text中插入emoji"
date:   2018-07-17 20:40:40
categories: MacroDog
tags:  Unity 
excerpt: Real_Time Rendering Thried Editor  
mathjax: true
---
* content
{:toc}
# Text中插入emoji
因为工作需求需要将游戏中的聊天添加表情。
## Text 
在Unity3D Text组件是继承Graphic类所有用于显示的UI组件都继承此类。其中OnPopulateMesh(VertexHelper vh)函数。这个函数是用来通过顶点生产三角面。所采用的方法也是通过这个重写这个函数，达到想要的效果。    
一般