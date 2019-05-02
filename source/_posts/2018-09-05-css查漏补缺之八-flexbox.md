---
layout: post
title: css查漏补缺之八-flexbox
date: 2018-09-05
tag: css
---

[链接](https://developer.mozilla.org/en-US/docs/Learn/CSS/CSS_layout/Flexbox)

flex是一维布局方法，将flex items在row方向上布局或是在column方向上布局。flex item会伸缩来适应剩余空间或空间不够的情况。**成为flex-items的元素可以认为是一个 *块级盒子*。**

### 设置在 flexbox 上的属性

1. display: box
2. flex-direction: row | column | row-reverse | column-reverse
3. flex-wrap: wrap 

    当flex items数量过多，导致超出flex box的时候，上面这个属性会让flex items分布在多行显示，消除溢出flex box的问题。

4. flex-flow = flex-direction + flex-wrap

    上面的设置可以写成： `flex-flow: row wrap;`

<!-- more -->

### 设置在 flex items 上的属性

1. flex: 1 200px;

    第1个无单位的属性表示元素将在主轴上占据多少空间（设置完padding和margin后留下的剩余空间）。

    第2个`200px`参数表明最小尺寸。

    上面这个属性的意思是：每一个flex item首先会给`200px`的可用空间。之后，剩余的空间将按照比例`1`来分。

2. flex是一个缩写属性，最多能设置三个值（**mdn上建议使用缩写形式**）：

    - flex-grow：无单位的比例
    - flex-shrink： 无单位的比例
    - flex-basis： 最小空间

3. align-items:控制flex items如何在交叉轴上摆放，可以取得值有：

    - stretch：默认值，将会拉伸flex items的高度和父元素相同
    - center：flex items将保持固有尺寸，但是会沿着交叉轴居中
    - flex-start/flex-end：定位在交叉轴的start/end位置

4. justify-content：控制flex items在主轴上如何摆放，可以取得值有：

    - flex-start(默认值):从主轴的start处开始
    - flex-end：从主轴的end处开始
    - center：定位在主轴的中间
    - space-around: 在每个元素的左右两侧都留相同的空间，导致最靠边的两个元素左右两侧距离边框有距离，且元素和元素之间的空间要比刚说的这个空间大。整个主轴的空间被这些flex items瓜分。
    - space-between: 最外侧的两个元素靠近边框的一侧没有多余的空间，整个主轴的空间被flex items瓜分，剩余的空间都分布在flex items之间了。

5. order:

    - 默认值： 0
    - 数值越大排序越靠后

### Nested flex boxes

这个已经用过多次了