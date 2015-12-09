title: 盒模型以及 box-sizing
tags: css
date: 2015-12-08 13:14:27
---

前两天一个准备入前端坑的朋友问我：标准盒模型到底是什么？我不假思索的说:


> 盒模型有两种，一种是 w3c 标准盒模型，一种是 IE 盒模型。标准盒模型就是 `width = content_width + padding_width + border_width`。

朋友立马打断我说道：

> 不对不对，你又把我搞晕了，这不是 IE 盒模型么？

<!-- more -->

我感觉到有点不对劲了，赶紧 Google 了一下，才发现我真的记反了。这让我大吃一惊，这些基本概念我明明都很清楚的，怎么工作一年后居然把这个都弄混了。然后我突然想起了另一个属性：`box-sizing:border-box`，我说的盒模型计算公式就是 border-box 啊。自己一直用的 css reset 把 box-sizing 默认设置成 border-box，用的久了慢慢的就忘了它的存在，心想着自己在 chrome 下用的应该是标准的盒模型，就默认把 border-box 当成标准盒模型了。

先来回顾一下标准盒模型和 IE 盒模型，一图胜千言。

![盒模型](/image/blog/box-model.png)

所以标准盒模型是不包括 padding 和 border 的，也就是 `width=content_width`。

那么为什么所有的 reset CSS 的 box-model 都默认是 border-box 呢？因为 border-box 的计算方式**更符合我们的心理预期**，同时也**更方便计算**。设想下面这种情况，我们希望内容宽度100%，同时还有 1em 的边框。

- 如果是 content-box（内容盒模型），则会出现横向滚动条，这显然不是我们想要的

<p data-height="268" data-theme-id="0" data-slug-hash="zrxEVY" data-default-tab="result" data-user="xuhong" class='codepen'>See the Pen <a href='http://codepen.io/xuhong/pen/zrxEVY/'>content-box</a> by 旭鸿 (<a href='http://codepen.io/xuhong'>@xuhong</a>) on <a href='http://codepen.io'>CodePen</a>.</p>
<script async src="//assets.codepen.io/assets/embed/ei.js"></script>

-  border-box（边框盒模型）才是我们预期的展现方式

<p data-height="268" data-theme-id="0" data-slug-hash="gPbGyG" data-default-tab="result" data-user="xuhong" class='codepen'>See the Pen <a href='http://codepen.io/xuhong/pen/gPbGyG/'>border-box</a> by 旭鸿 (<a href='http://codepen.io/xuhong'>@xuhong</a>) on <a href='http://codepen.io'>CodePen</a>.</p>
<script async src="//assets.codepen.io/assets/embed/ei.js"></script>

通常，最常见的 CSS reset 的 box-sizing 是下面这样的：

``` css
*,
*:before,
*:after{
	-webkit-border-sizing: border-box;
	-moz-border-sizing: border-box;
	border-sizing: border-box;
}
```

但是 CSS-Tricks 提供了[一种更好的 box-sizing reset](https://css-tricks.com/inheriting-box-sizing-probably-slightly-better-best-practice/)，添加了继承，方便覆盖效率更高，同时去掉了浏览器前缀（国外 IE7 及以下已经不用考虑了）：

``` css
html {
  box-sizing: border-box;
}
*, *:before, *:after {
  box-sizing: inherit;
}
```

一些资料：

-[wikipedia: Internet Explorer box model bug](https://en.wikipedia.org/wiki/Internet_Explorer_box_model_bug#Background)
-[Can i use box-sizing](http://caniuse.com/#feat=css3-boxsizing)
-[CSS-Tricks Box Sizing](https://css-tricks.com/box-sizing/)
-[International box-sizing Awareness Day](https://css-tricks.com/international-box-sizing-awareness-day/)