---
title: 围住浮动元素的几种方法
date: 2015-08-16 10:45:37
tags: CSS
categories: CSS
---

元素一旦设置了浮动属性，其本身就会脱离文档流，从而它的父元素也就无法包围它，造成父元素塌陷，紧跟父元素的同级元素也会跟上来。造成页面布局混乱。这时候就得用父元素包围其浮动的子元素。以下介绍几种方法来围住浮动元素。

<!--more-->

# 为父元素添加overflow:hidden

`overflow:hidden`,它真正用途是防止包含于素被超大内容撑大，应用`overflow:hidden`后，包含元素依然保持其设定的宽度,而超大的子内容会被容器剪掉。在这里`overflow:hidden`的用途是：它能可靠地迫使父元素包含其浮动的子元素。

注意：不能在下拉菜单的顶级元素上应用overflow:hidden ，否则作为其子元素的下拉菜单就不会显示了。因为下拉菜单会显示在其父元素区域的外部，而这恰恰是 overflow:hidden 所要阻止的。再比如，不能
对已经靠自动外边距居中的元素使用“浮动父元素”技术，否则它就不会再居中，而是根据浮动值浮动到左边或右边了。

# 同时浮动父元素

既然浮动父元素，那么它也就脱离了文档流，与其同级的下面元素也会努力向上挤。为了防止向上挤给其添加`clear:left`

1. 浮动了父元素，不管其子元素是否浮动，它都会紧紧包围住其子元素。
2. 设置父元素width:100%让其与浏览器同宽。
3. 同级元素应用clear:left,其不会被 提升到浮动元素的旁边。

# 添加非浮动的清除元素

在父元素的最后添加一个非浮动的空子元素（一般用div），然后清除(clear:left|right)该子元素，由于父元素一定会包含非浮动的子元素，而清除这个子元素的一定会在浮动元素的下方，而且这个元素是最后一个。
```html
<style type="text/css">
    section {border:1px solid blue;width: 200px;}
    .floatElem{float: left;width: 50px;height: 50px;border: 1px solid #ccc;}
    .clear_me{clear: left;}
    footer{border: 1px solid red;}
</style>
<section>
    <div class="floatElem">我是浮动元素</div>
    <p>It's fun to float.It's fun to float.It's fun to float.It's fun to float.It's fun to float.It's fun to float.It's fun to float.</p>
    <div class="clear_me"></div>
</section>
<footer> Here is the footer element…</footer>
```

# .clearfix

在父元素末尾添加一个非浮动的空子元素来实现围住浮动元素，这样就破坏了整体的html结构。下面采用了css的:after选择器来实现围住浮动元素。

```html
<style type="text/css">
    section {border:1px solid blue;width: 200px;}
    .floatElem{float: left;width: 50px;height: 50px;border: 1px solid #ccc;}
    footer{border: 1px solid red;}
    .clearfix:after{
        content: ".";
        display: block;
        height: 0;
        visibility: hidden;
        clear: both;
    }
</style>
<section class="clearfix">
    <div class="floatElem">我是浮动元素</div>
    <p>It's fun to float.It's fun to float.It's fun to float.It's fun to float.It's fun to float.It's fun to float.It's fun to float.</p>
</section>
<footer> Here is the footer element…</footer>
```

:after选择器（对于 IE8 及更早版本中的 :after，必须声明 `<!DOCTYPE>`），即在父元素section内容之后插入新内容，这里插入一个点，来最为非浮动元素（非浮动元素必须有内容，这里用最小的句点），其他规则：块元素，高度为0，不可见，clear:both意味着新增的子元素会清除左右浮动的元素。

clearfix也是最常用的清除浮动的方法。

