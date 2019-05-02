---
layout: post
title: css查漏补缺之五-css布局介绍
date: 2018-09-04
tag: css
---

[链接](https://developer.mozilla.org/en-US/docs/Learn/CSS/CSS_layout/Introduction)

![一张神图](/images/css/css定位方案.png)


### Normal flow

能够改变元素的布局的方法如下：

1. `display`
2. `Floats`
3. `position`
4. `Table layout`
5. `Multi-column layout`

<!-- more -->

#### Flexbox

在父元素上设置`display：flex`，然后它的直属子元素就变成了`flex items`。

比如下面这个例子：

```css
.wrapper {
  display: flex;
}
```
```html
<div class="wrapper">
  <div class="box1">One</div>
  <div class="box2">Two</div>
  <div class="box3">Three</div>
</div>
```

flex初始设置：

没有给`wrapper`设置`display:flex`之前，它的三个子元素将竖向堆叠;设置后，它们将排成一行，**这是由于它们成为了flex items 并且拥有了flex box给它们的初始属性**，其中`flex-direction: row`，所以就排成了一行。它们都被拉伸到最高`item`的高度，这是因为`align-items: stretch`,这意味着`items`拉伸到flex容器的高度，这里是由最高的`item`决定的。`items`从容器的开始排成一行，在该行的后面留下多余的空间。

上面提到的`flex-direction`和`align-items`都是应用在`flexbox`上面的。

#### Grid Layout

flexbox是为一维布局设计的，也就是说只能横向或竖向布局。grid布局是为二维布局设计的，同时在横向和竖向两个方向上布局。

使用`display: grid`将布局方式转换为grid布局。

```css
.wrapper {
  display: grid;
  grid-template-columns: 1fr 1fr 1fr;
  grid-template-rows: 100px 100px;
  grid-gap: 10px;
}
```

```html
<div class="wrapper">
  <div class="box1">One</div>
  <div class="box2">Two</div>
  <div class="box3">Three</div>
  <div class="box4">Four</div>
  <div class="box5">Five</div>
  <div class="box6">Six</div>
</div>
```

上面提到的`grid-template-rows`以及`grid-template-columns`属性是应用在`gridbox`上的。上面的例子定义了三列，每列1fr，两行，每行100px。

下面的这个例子简直惊了...

```css
.wrapper {
  display: grid;
  grid-template-columns: 1fr 1fr 1fr;
  grid-template-rows: 100px 100px;
  grid-gap: 10px;
}

.box1 {
  grid-column: 2 / 4;
  grid-row: 1; 
}

.box2 {
  grid-column: 1;
  grid-row: 1 / 3;
}

.box3 {
  grid-row: 2;
  grid-column: 3;
}
```

```html
<div class="wrapper">
  <div class="box1">One</div>
  <div class="box2">Two</div>
  <div class="box3">Three</div>
</div>
```

#### Floats

#### Positioning techniques

positioning允许你将处在正常流中的元素所在位置移到他处。**Positioning不是创建主页面布局的方法，更多的是用来微调页面上个别元素的位置。**

positioning可以取以下五个值：

1. `static`：默认值，在`Normal flow`中
2. `relative`：在`Normal flow`中
3. `absolute`：完全脱离`Normal flow`

    在[格式化上下文](./2018-08-31-css查漏补缺之一-格式上下文.md)一文中，有记录：子元素浮动导致父元素高度出现坍塌的原因是在计算页面排版的时候，如果没有设置父元素的高度，那么该父元素的高度是由其他的子元素的高度撑开的。但是如果子元素是设置了浮动，**脱离了文档流**,那么父元素计算高度的时候就会忽略该子元素，出现父元素高度为0的现象。解决办法是让父元素新建一个BFC，其在计算高度的时候，会把浮动资源素包进来。

    `absolute`也会导致元素完全脱了`normal flow`，但即便给其父元素设置了创建BFC的属性，也不能解决副元素高度坍塌的问题，看来，对于`absolute`是包不进来了。

4. `fixed`：完全脱离`Normal flow`
5. `sticky`：元素开始表现的像`static`，直到它 hit a defined offset from the viewport，此刻就开始按`fiexed`行事。

点击[此处](https://codepen.io/ginnko/pen/OogrwQ)详见示例。

>  A stickily positioned element is treated as relatively positioned until it crosses a specified threshold, at which point it is treated as fixed until it reaches the boundary of its parent. 

一个使用`position: sticky`布局的元素在到达设置的阈值之前将表现为`relative`的布局形式，到达阈值后将被当作`fixed`布局，直到它碰到它的父元素。


#### Table layout

现代浏览器使用`flexbox`以及`grid`，这个就不看了

#### Multi-column layout

点击[此处](https://codepen.io/ginnko/pen/BOZEoR)详见示例。这个例子在容器元素上使用`column-width: 200px;`，使得浏览器创建尽可能多的200像素的`column`填充进容器，然后将剩余的空间布置在创建的`column`周围。
