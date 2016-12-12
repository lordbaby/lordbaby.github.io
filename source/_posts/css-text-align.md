---
title: css-text-align
date: 2016-5-12 15:31:41
tags: CSS
categories: CSS
---
原文[Centering in CSS: A Complete Guide](https://css-tricks.com/centering-css-complete-guide/)

以下是译文

CSS的居中是众多CSS难点的代表。为啥用CSS居中这么难呢？但是我认为这个问题其实并没有那么难啦，就是有很多种不同的方式可以达到居中的目的，这取决于不同的情景，很难说用哪一种方式去实现居中。
所以就让我们做一个决策树，希望能使这个问题简单一点吧~

<!--more-->

##水平居中

###行内或者具有行内元素性质的元素（比如文字或者链接）

让一个父元素为块级元素的行内元素水平居中，可以：

```
.center-children {
    text-align: center;
}
```

###单个块级元素

你可以设置块级元素的 margin-left 和 margin-right 为 auto 来使它水平居中（这个块级元素是被设置了一个 width 属性了，否则它会占满宽度，这时候已经不需要居中了），有一种速记的写法：

```
.center-me {
    margin: 0 auto;
}
```

###多个块级元素

如果有两个或者更多的块级元素需要在被一行里水平居中，那么你最好设置他们不同的 display 属性来达到效果了。这里有两个例子：一个是设置为 inline-block， 一个是设置为 flexbox 。
HTML:

```html
<main class="inline-block-center">
  <div>
    I'm an element that is block-like with my siblings and we're centered in a row.
  </div>
  <div>
    I'm an element that is block-like with my siblings and we're centered in a row. I have more content in me than my siblings do.
  </div>
  <div>
    I'm an element that is block-like with my siblings and we're centered in a row.
  </div>
</main>
<main class="flex-center">
  <div>
    I'm an element that is block-like with my siblings and we're centered in a row.
  </div>
  <div>
    I'm an element that is block-like with my siblings and we're centered in a row. I have more content in me than my siblings do.
  </div>
  <div>
    I'm an element that is block-like with my siblings and we're centered in a row.
  </div>
</main>
```

```
.inline-block-center {
  text-align: center;
}
.inline-block-center div {
  display: inline-block;
  text-align: left;
}
.flex-center {
  display: flex;
  justify-content: center;
}
```

除非你是想让多个块级元素堆积在彼此的顶部（一列堆积啦），那么 margin: auto 还是依然适用的：

```
main div {
  background: black;
  margin: 0 auto;
  color: white;
  padding: 15px;
  margin: 5px auto;
}
```

##垂直居中

在CSS里，垂直居中是有那么一点点麻烦了。

###行内或者具有行内元素性质的元素（比如文字或者链接）

####单行

有时候行内元素或者文字显示为垂直居中，仅仅是因为它们的上下内边距相等：

```
.link {
  padding-top: 30px;
  padding-bottom: 30px;
}
```

如果 padding 出于某些原因不能用，并且你要使一些不换行的文字居中，这里有一个技巧，就是设置文字的 line-height 和 height 的值相等。

```
.center-text-trick {
  height: 100px;
  line-height: 100px;
  white-space: nowrap;
}
```

####多行

上边距和下边距相等也能让多行文字达到垂直居中的效果，但是如果这种方法不奏效的话，可能需要设置文字所在的元素为一个 `table cell`，不管它直接是 `table` 还是你用CSS使这个元素表现的像一个 `table cell`，`vertical-align `属性可以处理这种情况，它与我们通常所做的在行上处理元素对齐的方式不同：

```
table {
  background: white;
  width: 240px;
  border-collapse: separate;
  margin: 20px;
  height: 250px;
}
table td {
  background: black;
  color: white;
  padding: 20px;
  border: 10px solid white;
  /* default is vertical-align: middle; */
}
.center-table {
  display: table;
  height: 250px;
  background: white;
  width: 240px;
  margin: 20px;
}
.center-table p {
  display: table-cell;
  margin: 0;
  background: black;
  color: white;
  padding: 20px;
  border: 10px solid white;
  vertical-align: middle;
}
.flex-center-vertically {
  display: flex;
  justify-content: center;
  flex-direction: column;
  height: 400px;
}
```

如果没法用类table方式，可能你需要用 flexbox ？单个的 flex 子元素可以非常简单的被一个 flex 父元素垂直居中

```
.flex-center-vertically {
  display: flex;
  justify-content: center;
  flex-direction: column;
  height: 400px;
}
```

请记住这个方法仅仅适用于父容器具有一个固定的额高度（px，%，等等），这也是为什么容器有一个高度。

如果上面的方法都不能用，你可以试试 ”虚元素“ 技术：其中一个完整高度的伪元素放置在容器内，并与文本垂直对齐。

```
.ghost-center {
  position: relative;
}
.ghost-center::before {
  content: " ";
  display: inline-block;
  height: 100%;
  width: 1%;
  vertical-align: middle;
}
.ghost-center p {
  display: inline-block;
  vertical-align: middle;
}
```

###块级元素

####知道元素的高度

不知道元素的高度是比较常见的，有很多原因：如果宽度改变，文本回流会改变高度；文字样式改变会改变高度；文字数量改变会改变高度；一个固定比例的元素，比如图片，当重置尺寸的时候也会改变高度，等等。

但如果你知道高度，你可以这样垂直居中元素：

```
.parent {
  position: relative;
}
.child {
  position: absolute;
  top: 50%;
  height: 100px;
  margin-top: -50px; /* account for padding and border if not using box-sizing: border-box; */
}
```

####元素高度未知

可以通过 transform 来达到目的

```
.parent {
  position: relative;
}
.child {
  position: absolute;
  top: 50%;
  transform: translateY(-50%);
}
```

####能用 flexbox 吗？

毫无疑问，用 flexbox 简单太多了

```
.parent {
  display: flex;
  flex-direction: column;
  justify-content: center;
}
```


##垂直水平居中

你可以通过不同的方式结合上面的技术来得到一个完美的居中水平垂直居中元素。但是我发现，这些方法通常都属于以下三种阵营：

###元素有固定的宽和高

用负的 margin 值使其等于宽度和高度的一半，当你设置了它的绝对位置为 50% 之后，就会得到一个很棒的跨浏览器支持的居中

```
.parent {
  position: relative;
}
.child {
  width: 300px;
  height: 100px;
  padding: 20px;
  position: absolute;
  top: 50%;
  left: 50%;
  margin: -70px 0 0 -170px;
}
```

###元素的宽和高未知

如果你不知道元素的高度和宽度，你可以用 `transform` 属性，用 `translate` 设置 -50%（它以元素当前的宽和高为基础）来居中

```
.parent {
  position: relative;
}
.child {
  position: absolute;
  top: 50%;
  left: 50%;
  transform: translate(-50%, -50%);
}
```

###能用 flexbox 吗？

为了让元素在 flexbox 两个方向都居中，你需要两个居中属性：

```
.parent {
  display: flex;
  justify-content: center;
  align-items: center;
}
```

