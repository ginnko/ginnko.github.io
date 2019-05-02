---
layout: post
title: css查漏补缺之十一-positioning
date: 2018-09-08
tag: css
---

[链接](https://developer.mozilla.org/en-US/docs/Learn/CSS/CSS_layout/Positioning)

### static positioning

*static positioning* 是每个元素获得的默认属性，这个属性的意思是：把这个元素摆在文档正常流的正常位置就好。

### relative positioning

设置为`position: relative`的元素：依然占据文档正常流中的位置，但是可以通过`top`，`bottom`，`left`，`right`调整位置。

<!-- more -->

### absolute positioning

一个绝对定位的元素不再在文档普通流中存在，**它存在于自己的层中**。这意味着我们可以创建孤立的ui特性同时不会干扰页面上的其他元素。比如 *弹出消息框*，*控制菜单*, *拖拽或低落的效果*。

对于绝对定位元素，margin属性依然有效。

### positioning contexts

>if no ancestor elements have their position property explicitly defined, then by default all ancestor elements will have a static position. The result of this is, the absolutely positioned element will be contained in the **initial containing block**. The initial containing block has the dimensions of the viewport, and is also the block that contains the html element. Simply put, the absolutely positioned element will be contained outside of the html element, and be positioned relative to the initial viewport.

解决了我的一大困惑：**父元素没有做任何特殊处理的绝对定位元素，将被包含在html元素之外（这意思是不再被html元素包裹？），将相对于初始包含块定位。**

可以通过给父元素指定`position: relative`或`position: absolute`来更改positioning contexts。

### z-index

首先要知道的是：当我们绝对定位一个元素之后，它就会出现在最顶层，因为*绝对定位元素*在和*non-positioned*元素争夺高空权的时候，胜出。

但是，当有多个（同一父元素下的兄弟？示例里使用的这种形式）元素时，就可以使用`z-index`属性来控制z-axis方向上的顺序。

### fixed positioning

固定定位元素相对于 **浏览器视口** 定位。

### sticky positioning

之前有看过这个