---
title: CSS3中的transition，transform，animation
date: 2015-08-09 17:39:40
tags: [transition,transform,animation]
categories: CSS
---

`transition`, `transform`, `animation` 按字面意义可以理解为过渡，变换，动画。3个与动画相关的属性。

<!--more-->

# transition

作用是平滑的改变css的属性值，无论是点击事件，焦点事件还是鼠标hover，只要值改变了，就是平滑的。就是动画，实用价值挺高，

transition有下面具体属性

```
transition-property :* /*指定过渡的性质，比如transition-property:backgrond 就是指backgound参与这个过渡*/
transition-duration:*/*指定这个过渡的持续时间*/
transition-delay:* /*延迟过渡时间*/
transition-timing-function:* /*指定过渡类型，有ease | linear | ease-in | ease-out | ease-in-out | cubic-bezier*/

动画效果
linear 线性。不解释
ease-in ,ease-out 
张大神的解释很生动形象，深入浅出。
在xxoo时，in表示进入，开始进入的时候是慢的，等瞄准就绪后才能快速冲刺，于是乎ease-in表示先慢后快。
ease-out表示出来，出来肯定是先快后慢，因为临近洞口速度肯定要降下来。于是ease-out表示先快后慢.
最后easy-in-out表示一进一出，也就是先慢后快再慢了。哈哈。。
```

一个小例子
```html
<style type="text/css">
    .tran{
        -webkit-transition: background-color 0.3s ease;
        -moz-transition: background-color 0.3s ease;
        -o-transition: background-color 0.3s ease;
        transition: background-color 0.3s ease;
    }
    .tran:hover{
        background-color: #486AAA;
        color: #fff;
    }

</style>

<a href="#" class="tran">经过我</a>
```

[Demo](/Demo/transition.html)

# transoform

`transform`指可以拉伸，压缩，旋转，偏移。

```
//通过 skew() 方法，元素翻转给定的角度，根据给定的水平线（X 轴）和垂直线（Y 轴）参数
//值 skew(30deg,20deg) 围绕 X 轴把元素翻转 30 度，围绕 Y 轴翻转 20 度。
.trans_skew { transform: skew(30deg,20deg); }
//通过 scale() 方法，元素的尺寸会增加或减少，根据给定的宽度（X 轴）和高度（Y 轴）参数
//值 scale(1,0.5) 把宽度转换为原始尺寸的 1 倍，把高度转换为原始高度的 0.5 倍。
.trans_scale { transform:scale(1, 0.5); }
//通过 rotate() 方法，元素顺时针旋转给定的角度。允许负值，元素将逆时针旋转。
//值 rotate(30deg) 把元素顺时针旋转 45 度。
.trans_rotate { transform:rotate(45deg); }
//通过 translate() 方法，元素从其当前位置移动，根据给定的 left（x 坐标） 和 top（y 坐标） 位置参数
//值 translate(50px,100px) 把元素从左侧移动 50 像素，从顶端移动 100 像素。
.trans_translate { transform:translate(50px,100px); }
```

一般情况transform属性要是加上transition的过渡特性，可以产生漂亮的动画，下面这个例子，代码如下

```html
.trans_effect {
    -webkit-transition:all 2s ease-in-out;
    -moz-transition:all 2s ease-in-out;
    -o-transition:all 2s ease-in-out;
    -ms-transition:all 2s ease-in-out;    
    transition:all 2s ease-in-out;
}
.trans_effect:hover {
    -webkit-transform:rotate(720deg) scale(2,2);
    -moz-transform:rotate(720deg) scale(2,2);
    -o-transform:rotate(720deg) scale(2,2);
    -ms-transform:rotate(720deg) scale(2,2);
    transform:rotate(720deg) scale(2,2);        
}
```

[缩放旋转动画demo](/Demo/transform.html)

# animation

animation是个简写属性

|值|描述|
|--------------|------------------------------|
|animation-name|  规定需要绑定到选择器的 keyframe 名称。。|
|animation-duration | 规定完成动画所花费的时间，以秒或毫秒计。|
|animation-timing-function|   规定动画的速度曲线。|
|animation-delay| 规定在动画开始之前的延迟。|
|animation-iteration-count|   规定动画应该播放的次数。|
|animation-direction| 规定是否应该轮流反向播放动画。|

animation不仅适用于CSS2中的属性，CSS3也是支持的，例如box-shadow盒阴影效果，所以，我们可以实现外发光效果的。使用过web qq的人应该有印象，当鼠标移到聊天对象脑袋上的时候会有蓝色外发光的动画，但是那是gif动画图片实现的（现在自动跳转到web qq 2.0已看不到效果）。gif动画实现的效果类似于下面（很兼容）

[Demo](/demo/animation.html)

