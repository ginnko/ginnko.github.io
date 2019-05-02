---
layout: post
title: css查漏补缺之十三-box model recap
date: 2018-09-10
tag: css
---

[链接](https://developer.mozilla.org/en-US/docs/Learn/CSS/Styling_boxes/Box_model_recap)

### overflow

`overflow`的默认值是`visible`。

<!-- more -->

### background

默认情况下，`backgrounds`延伸到`border`的外边延。

使用`background-clip`属性可以设置`backgrounds`仅延伸到内容的边缘。

`background-clip`可以设置的值有：

- `border-box`

- `padding-box`

- `content-box`

- `text`

设置在文本上的时候，可以这样设置：

```css
background-clip: text;
-webkit-background-clip: text; // 针对chrome
color: transparent;
```

### outline

`outline`**不是**盒模型的一部分，看起来像是border，但实质是画在盒子的上方，同时也不会改变盒子的尺寸（具体来讲是画在边框外，在外边距内）。

### advanced box properties

1. setting width and height constraints

一个常见的用法：

```css
.container {
  width:  70%; // 给容器设置弹性宽度
  max-width: 1280px; // 设置最大值，阻止宽度过大  
  min-width： 480px; // 设置最小值，阻止宽度过小
}
```

使用`margin: 0 auto;`来给inner盒子居中在parent中时，需要设置本盒子的宽度，这个宽度设置成 **百分比** 也是有效果的。`auto`的意思是让容器左右两侧的外边距分享可用的空间，以实现居中的效果。

2. changing the box model completely

盒子的总宽 = `width` + `padding-left` + `padding-right` + `border-right` + `border-left`

