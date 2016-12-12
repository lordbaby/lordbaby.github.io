---
title: CSS选择器备忘录
date: 2015-07-17 15:47:11
tags: CSS
categories: CSS
---

# CSS选择器

不经常写CSS有时候会遗忘，抄一遍w3c加固以下，顺便当个笔记
CSS列指示该属性是在哪个CSS版本中定义。

|选择器|例子|例子描述|CSS|浏览器|
|----------------|:----------:|:-----------------------------------:|:---:|--------:|
|`*`|*|选择所有元素|2|All|
|`.class`|.intro|选择class="intro"的所有元素|1|All|
|`.#id`|#fisrtname|id="fisrtname"的所有元素|1|All|
|`element`|p|选择所有`<p>`元素|1|All|
|`element,element`|div,p|选择所有`<p>`元素和所有`<div>`元素|1|All|
|`element element`|div p|选择`<div>`元素内部的所有`<p>`元素|1|All|
|`element>element`|div>p|选择父元素为 `<div>` 元素的所有 `<p>` 元素。|2|ie8及以下，必须声明`<!DOCTYPE>`|
|`element+element`|div+p|选择紧接在 `<div>` 元素之后的所有`<p>` 元素。|2|ie8及以下，必须声明`<!DOCTYPE>`|
|`[attribute]`|[target]|选择带有 target 属性所有元素。|2|ie8及以下，必须声明`<!DOCTYPE>`|
|`[attribute=value]`|[target=_blank]|选择 target="_blank" 的所有元素。|2|ie8及以下，必须声明`<!DOCTYPE>`|
|`[attribute~=value]`|[title~=flower]|选择 title 属性包含单词 "flower" 的所有元素。|2|ie8及以下，必须声明`<!DOCTYPE>`|
|`[attribute|=value]`|`[lang|=en]`|选择 lang 属性值以 "en" 开头的所有元素。|2|ie8及以下，必须声明`<!DOCTYPE>`|
|`:link`|a:link|选择所有未被访问的链接。|1|All|
|`:visited`|a:visited|选择所有已被访问的链接。|1|All|
|`:active`|a:active|选择活动链接。|1|All|
|`:hover`|a:hover|选择鼠标指针位于其上的链接。|1|All|
|`:focus`|input:focus|选择获得焦点的 input 元素。|2|ie8必须声明`<!DOCTYPE>`
|`:first-letter`|p:first-letter|选择每个 `<p>` 元素的首字母。|2|all|
|`:first-line`|p:first-line|选择每个 `<p>` 元素的首行。|1|all|
|`:first-child`|p:first-child|选择属于父元素的第一个子元素的每个 `<p>` 元素。|2|ie8及以下，必须声明`<!DOCTYPE>`|
|`:before`| p:before |  在每个 `<p>` 元素的内容之前插入内容。|2|ie8及以下，必须声明`<!DOCTYPE>`|
|`:after`|  p:after| 在每个 `<p>` 元素的内容之后插入内容。|2|ie8及以下，必须声明`<!DOCTYPE>`|
|`:lang(language)` |p:lang(it)|  选择带有以 "it" 开头的 lang 属性值的每个 `<p>` 元素。|2|ie8及以下，必须声明`<!DOCTYPE>`|
|`element1~element2`|p~ul|选择前面有 `<p>` 元素的每个 `<ul>` 元素。    |3|ie8必须声明`<!DOCTYPE>`|
|`[attribute^=value]`|a[src^="https"]| 选择其 src 属性值以 "https" 开头的每个 `<a>` 元素。  |3|ie8及以下，必须声明`<!DOCTYPE>`|
|`[attribute$=value]`|a[src$=".pdf"]|  选择其 src 属性以 ".pdf" 结尾的所有 `<a>` 元素。    |3|ie8及以下，必须声明`<!DOCTYPE>`|
|`[attribute*=value]`|a[src*="abc"]|   选择其 src 属性中包含 "abc" 子串的每个 `<a>` 元素。   |3|ie8及以下，必须声明`<!DOCTYPE>`|
|`:first-of-type`| p:first-of-type| 选择属于其父元素的首个 `<p>` 元素   |3|不适用于IE8 及更早的版本|
|`:last-of-type`| p:last-of-type|  选择属于其父元素的最后 `<p>` 元素。   |3|不适用于IE8 及更早的版本|
|`:only-of-type`| p:only-of-type|  选择属于其父元素唯一的 `<p>` 元素。   |3|不适用于IE8 及更早的版本|
|`:only-child` |p:only-child|   选择属于其父元素的唯一子元素的每个 `<p>` 元素。   |3|不适用于IE8 及更早的版本|
|`:nth-child(n)`| p:nth-child(2) | 选择属于其父元素的第二个子元素 `<p>` 。  |3|不适用于IE8 及更早的版本|
|`:nth-last-child(n)`|p:nth-last-child(2)| 同上，从最后一个子元素开始计数。    |3|不适用于IE8 及更早的版本|
|`:nth-of-type(n)`| p:nth-of-type(2) |选择属于其父元素第二个 `<p>` 元素。   |3|不适用于IE8 及更早的版本|
|`:nth-last-of-type(n)`| p:nth-last-of-type(2)| 同上，但是从最后一个子元素开始计数。  |3|不适用于IE8 及更早的版本|
|`:last-child`| p:last-child |  选择属于其父元素最后一个子元素每个 <p> 元素。   |3|不适用于IE8 及更早的版本|
|`:root` |  :root |  选择文档的根元素。   |3|不适用于IE8 及更早的版本|
|`:empty`|  p:empty |选择没有子元素的每个 `<p>` 元素（包括文本节点）。  |3|不适用于IE8 及更早的版本|
|`:target`| #news:target |   选择当前活动的 #news 元素。| 3|不适用于IE8 及更早的版本|
|`:enabled`| input:enabled |  选择每个启用的 `<input>` 元素。| 3|不适用于IE8 及更早的版本|
|`:disabled`|   input:disabled | 选择每个禁用的 `<input>` 元素 | 3|不适用于IE8 及更早的版本|
|`:checked`|    input:checked  | 选择每个被选中的 `<input>` 元素。   | 3|不适用于IE8 及更早的版本|
|`:not(selector)`|  :not(p)| 选择非 `<p>` 元素的每个元素。   | 3|不适用于IE8 及更早的版本|
|`::selection`| ::selection| 选择被用户选取的元素部分。   |3|不适用于IE8 及更早的版本|