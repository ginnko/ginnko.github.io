---
layout: post
title: css查漏补缺之十二-multiple-column layout
date: 2018-09-09
tag: css
---

[链接](https://developer.mozilla.org/en-US/docs/Learn/CSS/CSS_layout/Multiple-column_Layout)

### 列属性

真是出乎意料，这个属性竟然几乎被全部的浏览器支持！！！

- `column-count`将创建属性值那么多数量的列。

使用`column-count`创建出来的列具有弹性宽度。

- `column-width`

使用这个属性将按照设定的属性值根据container的尺寸提供尽可能多的列，剩余的空间将`填充`到已生成的列中。也就是说没有办法得到设定的宽度，除非container的宽度恰好能被设定的宽度整除。

<!-- more -->

### 设置列的样式

出乎意料！！！通过`multicol`创建的列不能单独设置样式。没有办法做到一列比另一列宽，或是改变单独一列文本的颜色。只能通过`column-gap`或是`column-rule`这两个属性来整体改变每一列的样式。

**好像是个问题：通过column-\*设置的样式貌似都没有办法通过浏览器的样式查看器进行查看和调试**

`column-rule`是`column-rule-color`，`column-rule-style`以及`column-rule-width`三个属性的缩写。用于设置列边的样式。比如：

```css
.container {
  column-count: 3;
  column-gap: 20px;
  column-rule: 4px dotted rgb(79, 185, 227);
}
```

值得注意的是`column-rule`属性并不会为它自己占据什么空间，而是`lies across`通过`column-gap`属性设置的`gap上`。如果想增加`column-rule`两边的空间，只有增加`column-gap`的值才能做到。

### columns and fragmentation

> The content of a multi-column layout is fragmented. It essentially behaves the same way as content behaves in paged media ---- such as when you print a webpage. When you turn your content into a multicol container it is fragmented into columns, and the content breaks to allow this to happen.

这个[例子](https://codepen.io/pen/)中会出现标题和文本被分割的情况。

在容器中的元素上使用`break-inside: avoid`，明确告知我们不想对这个盒子使用片段化。

避免浏览器的不支持，建议混合使用下面的代码：

```css
.card {
  break-inside: avoid;
  page-break-inside: avoid;
}
```