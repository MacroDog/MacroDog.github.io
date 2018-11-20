---
layout: post
title:  "ESC框架"
date:   2018-08-05 10:30:00
categories: MacroDog
tags:  Unity3D ESC
excerpt: Unity3D ESC
mathjax: true
---
* content
{:toc}
# ESC框架学习
   偶然接触到ESC框架发现非常特别，所以想学习一下扩展一下自己的思考方式。
## 目的
  我对ESC框架设计目的理解是为了将数据和逻辑进行分开。那Unity3D开发来说，当我们定义一个对象的时候会将这个对象包含的数据，和方法写在一个脚本里。数据描述对象，方法描述对象的行为，看起来很合理，但是，这么做就会导致一些问题，比如多个相同的对象有相同的行为，但是每个对象下的脚本都包含了这个行为，造成冗余。由于，GameObject行为涉及到各个模块所以导致GameObejct 聚合性很高，内聚性很差
