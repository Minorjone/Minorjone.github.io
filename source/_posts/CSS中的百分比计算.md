---
title: CSS中的百分比计算
date: 2019-08-27 22:10:30
tags: [前端,CSS]
categories: 前端
keywords: [前端,CSS,CSS3,百分比]
description: 
---

CSS布局中会用到很多百分比，但他们究竟都是如何计算的呢，你可能一个不小心就会把他们混淆在一起，所以这里就对CSS中所有可以用到百分比的属性做一个总结，方便更加系统的理解和记忆。

<!--more-->

font-size
-------------
根据元素字体大小，当前大小为100%

line-height
-------------
根据元素字体大小，当前大小为100%

translate
-------------
根据元素大小，当前大小为100%

width
-------------
+ 正常文档流和浮动：百分比相对于父元素content-box
+ 绝对定位：百分比相对于父元素padding-box

height
-------------
+ 正常文档流和浮动：百分比相对于父元素content-box
+ 绝对定位：百分比相对于父元素padding-box
+ 当父元素高度为auto：子元素高度百分比会被忽略

margin
-------------
+ 正常文档流和浮动：百分比相对于父元素的content-box宽度
+ 绝对定位：百分比相对于父元素的padding-box宽度

padding
-------------
+ 正常文档流和浮动：百分比相对于父元素的content-box宽度
+ 绝对定位：百分比相对于父元素的padding-box宽度

top/bottom/left/right
-------------
+ 非定位元素：无效果
+ 相对定位： top和bottom百分比是相对父元素的content-box高度，left和right百分比是相对父元素的content-box宽度
+ 绝对定位：top和bottom百分比是相对最近一级非static定位父元素的padding-box高度，left和right百分比是相对最近一级非static定位父元素的padding-box宽度