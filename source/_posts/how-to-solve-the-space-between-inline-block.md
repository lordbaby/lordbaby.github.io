title: 解决inline-block元素的空白间隙
date: 2015-08-10 19:09:16
tags: [inline-block]
categories: CSS
---

在制作导航的时候采用`inline-block`代替`float:left`,但问题来了在`inline-block`的元素之间多出来4px的空白。
<!--more-->

# 举例

```html
<style type="text/css">
    *{padding: 0;margin: 0;}
    ul{
        list-style: none outside none;
        padding: 0;
        background-color: green;
        text-align: center;
    }
    ul li{
        display: inline-block;
        *display: inline;
        zoom:1;
        background-color: orange;
        padding: 5px;
    }
</style>
<ul>
    <li>item1</li>
    <li>item2</li>
    <li>item3</li>
    <li>item4</li>
</ul>

```

![inline-block](/images/inline-block.png)

明显的可以看出，在inline-block的元素之间存在“4px”的空白

# 方法1

改变html结构

```html
<style type="text/css">
    *{padding: 0;margin: 0;}
    ul{
        list-style: none outside none;
        padding: 0;
        background-color: green;
        text-align: center;
    }
    ul li{
        display: inline-block;
        *display: inline;
        zoom:1;
        background-color: orange;
        padding: 5px;
    }
</style>
<!--结构1-->
<ul>
    <li>
    item1</li><li>
    item2</li><li>
    item3</li><li>
    item4</li>
</ul>
<!--结构2-->
<ul>
    <li>item1</li
    ><li>item2</li
    ><li>item3</li
    ><li>item4</li>
</ul>
<!--结构3-->
<ul>
    <li>item1</li><!--
    --><li>item2</li><!--
    --><li>item3</li><!--
    --><li>item4</li>
</ul>
<!--结构4 -->
<ul>
    <li>item1</li><li>item2</li><li>item3</li><li>item4</li>
</ul>

```

结构4也许是比较常见的结构，但标签是后台生成的话又不行了，还是得通过css来解决。

# 方法2

```html
<style type="text/css">
    
    *{padding: 0;margin: 0;}
    ul{
        list-style: none outside none;
        padding: 0;
        background-color: green;
        text-align: center;
    }
    ul li{
        display: inline-block;
        *display: inline;
        zoom:1;
        background-color: orange;
        padding: 5px;
        /*负margin*/
        margin-right: -4px;
        *margin-right: 0;
    }

</style>
```

这种解决方法并不完美，如果你的父元素设置的字号不一样，可能你的“-4px”就不能解决问题

```html
<style type="text/css">
    
    *{padding: 0;margin: 0;}
    ul{
        list-style: none outside none;
        padding: 10px;
        background-color: green;
        text-align: center;
        /*font-size: 0*/
        font-size: 0;
    }
    ul li{
        display: inline-block;
        *display: inline;
        zoom:1;
        background-color: orange;
        padding: 5px;

        font-size: 12px;
    }

</style>
```

设置父元素的字体为“0”，然后在“inline-block”元素上重置字体需要的大小,这样处理在Firexfox,chrome等浏览器下是达到了效果，可是在Safari下可问题依然存在。

```javascript
//借助jquery
$('ul').contents().filter(function() {
    return this.nodeType === 3;
    }).remove();

```

# 全兼容的样式解决方法

使用纯CSS还是找到了兼容的方法，就是在父元素中设置font-size:0,用来兼容chrome，而使用letter-space:-n px来兼容safari:

```html
<style type="text/css">
    ul {
      letter-spacing: -4px;/*根据不同字体字号或许需要做一定的调整*/
      word-spacing: -4px;
      font-size: 0;
    }
    ul li {
      font-size: 16px;
      letter-spacing: normal;
      word-spacing: normal;
      display:inline-block;
      *display: inline;
      zoom:1;
    }

</style>
```

# 总结

既然是inline-block，那么肯定就会有inline的特性，如果不想有空隙，还是用浮动来代替吧。