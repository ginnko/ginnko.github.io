---
layout: post
title: css查漏补缺之十四-backgrounds
date: 2018-09-11
tag: css
---

[链接](https://developer.mozilla.org/en-US/docs/Learn/CSS/Styling_boxes/Backgrounds)

### What exactly is a background

1. 默认情况下，背景会延伸到边框的外延。外边距并不算元素区域的一部分，而是算元素的外部区域。

### The basics: color, image, position, repeat

#### background color

1. 元素默认的背景色是`transparent`

### background image

`background-image: url(https://mdn.mozillademos.org/files/13026/fire-ball-icon.png);` url中不用必须写成字符串

1. background images使用css属性设置，所以对于辅助设备是不可见的。

下面这段话解释了什么时候用background，什么时候用img元素。

>Background images are not content images -- they are just for decoration -- if you want to include an image on your page that is part of the content, then you should do so with an `<img>` element.

### background repeat

可以取得值有：

- no-repeat: 横向，竖向都完全不重复，只会显示一次
- repeat-x: 会在横向重复
- repeat-y：会在纵向重复
- repeat： *默认值* ，在横向和纵向皆重复

### background position

这个属性允许摆放background image的位置，属性值是两个由空格分隔的值，可以是绝对值也可以是相对值，这两个值定义了`背景图片`的横纵坐标。

>Generally the property will take two values separated by a space, which specify the horizontal (x) and vertical (y) coordinates of the image. The top left corner of the image is the origin — (0,0). 

下面的观点出自[这里](https://www.cnblogs.com/xiaochaohuashengmi/archive/2011/02/01/1948644.html)以及[这里](https://blog.csdn.net/u013778905/article/details/52811146)，其实并没有说清楚，结合自己的感觉，罗列如下：

- 使用绝对值定位其实指的是图片的左上角相对于背景的左上角的偏移。

- 使用百分数定位是改变了背景图和元素的对齐基点，background-position： 100% 50%; 就是将背景图片的 100%（right） 50%（center） 这个点，和元素的 100%（right） 50%（center） 这个点对齐。

- 使用关键字定位感觉类似百分比

### background image:gradients

有两种gradient方式：线性和辐射

#### 线性gradient

`background-image: linear-gradient(to bottom, orange, yellow);`

linear-gradient函数接收三个参数：方向、起始颜色、结束颜色。

color stops: 下面的`orange 40%`就是一个color stops，可以指定任意个color stops，40%表示的是位置而不是颜色的程度，是沿着指定的方向按此值确定的，此处可以使用绝对值。

`background-image: linear-gradient(to bottom, yellow, orange 40%, yellow);`

### background attachment

首先，这个属性只有在内容出现滚动的时候才会生效。

这个属性有三个值可用：

- scroll：页面滚动，背景图也跟着滚动，元素滚动，背景图固定

- fixed：使背景图相对于视口fixed，无论内容还是页面滚动背景图都不会滚动

- local：页面滚动或元素滚动，背景图都跟着滚动

要看一下这个简写属性：https://developer.mozilla.org/zh-CN/docs/Web/CSS/Shorthand_properties