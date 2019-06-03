title: list-style-image sucks?
date: 2016-03-02 12:53:36
tags:
---

最近在学习jQuery，写一个文件夹开合的小组件。在阿里妈妈找了文件夹的svg图后，就准备把它们丢到list-style-image里。然而发现自己handle不了svg图的大小。于是，我寄希望于list-style-image有控制图大小的value。

就去查了查之前一直没有用过的list-style。所以，list-style-image里有什么？

** 先来看看MDN上关于list-style-iamge的介绍吧 **

> ** Summary: **The list-style-image property sets the image that will be used as the list item marker. 

> ** Values: **   url/none/inherited

>** Applies to: ** list-item 

好像并没有能解决我问题的value。 ** 但后来，我学会通过改svg图的源码来设置图的大小。嗯，解决了svg图的问题 **

可大家一般都怎么使用list-style-iamge呢？ ** 再后来，我发现大家真的都不怎么用它:) **

 ----------------

##### 去看看阿里的官网

<img src="http://d.pr/i/1f9df/alibaba.png">

代码应该是这个样子的。可以看到它是在** i标签里插入图片 **，实现marker的效果

<p data-height="224" data-theme-id="0" data-slug-hash="LNEmMN" data-default-tab="result" data-user="amanda111" class='codepen'>See the Pen <a href='http://codepen.io/amanda111/pen/LNEmMN/'>LNEmMN</a> by AMANDA (<a href='http://codepen.io/amanda111'>@amanda111</a>) on <a href='http://codepen.io'>CodePen</a>.</p>
<script async src="//assets.codepen.io/assets/embed/ei.js"></script>


 ------------------


##### 再看看twitter的官网

<img src="http://d.pr/i/8GiH/twitter.png">

** 用的是伪元素:before **

<p data-height="174" data-theme-id="0" data-slug-hash="oxgyQE" data-default-tab="result" data-user="amanda111" class='codepen'>See the Pen <a href='http://codepen.io/amanda111/pen/oxgyQE/'>oxgyQE</a> by AMANDA (<a href='http://codepen.io/amanda111'>@amanda111</a>) on <a href='http://codepen.io'>CodePen</a>.</p>
<script async src="//assets.codepen.io/assets/embed/ei.js"></script>

-----------------------------------------------------------------------------------

更多的是** 用设置background-image的方法 **，设置background no-repeat，再给内容设一个合适的padding-left的值，就把background-image变到内容前面了。就有了小图标的赶脚。
<p data-height="219" data-theme-id="0" data-slug-hash="xVbWQQ" data-default-tab="html" data-user="amanda111" class='codepen'>See the Pen <a href='http://codepen.io/amanda111/pen/xVbWQQ/'>xVbWQQ</a> by AMANDA (<a href='http://codepen.io/amanda111'>@amanda111</a>) on <a href='http://codepen.io'>CodePen</a>.</p>
<script async src="//assets.codepen.io/assets/embed/ei.js"></script>

-----------------------------------------------------

现在用** list-style-image **
<p data-height="154" data-theme-id="0" data-slug-hash="pyvKdz" data-default-tab="html" data-user="amanda111" class='codepen'>See the Pen <a href='http://codepen.io/amanda111/pen/pyvKdz/'>pyvKdz</a> by AMANDA (<a href='http://codepen.io/amanda111'>@amanda111</a>) on <a href='http://codepen.io'>CodePen</a>.</p>
<script async src="//assets.codepen.io/assets/embed/ei.js"></script>

噢..发生了什么？用list-style-image插入小图标，发现image和内容之间有逼死强迫症的间距呀。**而这个间距又是什么呢？**

当不改变li标签样式的时，默认样式为list-style：disc。这时就可以看到实心点和内容的间距。这个间距应该是不能被改变的，是由浏览器决定的。所以，list-style-image 属性会有兼容性问题。

想到要调整图片位置，就想到有 *list-style-position*

#### so what is list-style-position?

> **Initial values: ** outside

> ** Values: ** inside/outside

> ** Applies to: ** list-item

<p data-height="338" data-theme-id="0" data-slug-hash="aNzgxR" data-default-tab="html" data-user="amanda111" class="codepen">See the Pen <a href="http://codepen.io/amanda111/pen/aNzgxR/">aNzgxR</a> by AMANDA (<a href="http://codepen.io/amanda111">@amanda111</a>) on <a href="http://codepen.io">CodePen</a>.</p>
<script async src="//assets.codepen.io/assets/embed/ei.js"></script>

> ** outside: **The marker box is outside the principal block box

> ** inside: **The marker box is the first inline box in the principal block box, after which the element's content flows.

就只有这两个value。所以..它并不能解决marker与内容的间距问题。但是，如果list-style-position有控制间距的value。

### Actually，list-style-image is just fine.


 -------------------------------------------


*[Reference Documentation]*

[1]<a href="https://developer.mozilla.org/en-US/docs/Web/CSS/list-style">MDN list-style</a>