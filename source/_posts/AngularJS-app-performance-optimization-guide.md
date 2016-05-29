title: AngularJS 应用优化指南
date: 2016-05-10 16:15:33
tags:
- angularjs
- perf
---
![batarang](/image/ng-perf/batarang.jpg)
前两天因为想用国内的 JS CDN，访问到 [staticfile](http://www.staticfile.org/)（七牛提供的一个免费 CDN 服务），导致我的 Chrome 直接卡死了两次，页面关也关不掉，只能退出重启。后来换到 Chrome Canary 才勉强可以访问，用谷歌开发者工具一看，发现前端是用的 AngularJS ，大量的 `ng-repeat`，5万多 `watcher`，没有做任何性能优化，怎么可能不卡。
<!-- more -->

![](/image/ng-perf/staticfile.png)

今年以来一直在陆陆续续的优化公司基于 AngularJS 开发的应用，也积累了一些经验，正好总结一下 AngularJS webapp 的性能优化指南。

### 减少 `watcher`

我们知道 AngularJS 通过 *脏检查*（digest cicle）来更新视图，保持数据和视图的同步，脏检查的效率是和 `watcher` 的多少成正相关的，所以提高 AngularJS 性能的关键就是减少 `watcher` 的数量。

首先我们要知道的是，什么会产生 `watcher`？

- `$scope.$watch`
- `{ { } }` type bindings
- Most directives (i.e. `ng-show`)
- Scope variables `scope: { bar: '='}`
- Filters `{ { value | myFilter } }`
- `ng-repeat`

问题来了，怎么减少 `watcher`？

1. 使用单次绑定语法 `{ {::} }`

#### 

### 避免在 `ng-repeat` 中使用 `filter`

### 尽可能的使用 `ng-if` 而不是 `ng-show`

### 工具
怎么测时间都花在哪了？AngularJS 很贴心的内置了两个函数，`console.time()` 和 `console.timeEnd()` 放在代码的前后来测试代码的运行时间。

```
console.time('myTimer')
// your code here
console.timeEnd('myTimer')
```
这两个函数可以帮助我们测试某一小段代码的运行时间，如果要观测整个应用的运行时间，就要使用下面这两个工具了：
- [AngularJS Batarang](https://chrome.google.com/webstore/detail/angularjs-batarang/ighdmehidhipcmcojjgiloacoafjmpfk)
- Chrome Timeline

[Batarang](https://en.wikipedia.org/wiki/Batarang) 是一种很有意思的武器，蝙蝠形状的回旋刀，是各种电影动画里面蝙蝠侠的武器。

### 总结
从去年下半年开始，国内很多大公司都开始用 AngularJS 开发用户后台了，比如 upyun、 Ucloud（新版 UCloud 用户后台做的很不错） 和 阿里云。但 F12 查看源代码就会发现，基本上都没有做任何的性能优化。当然，现在电脑性能基本上都处于过剩的状态，即使不做任何优化，只要页面的 `watcher` 数量没有多到 staticfile 那样，基本上也不会有什么问题，最多就是把页面响应时间从几百毫秒提升到几十毫秒，1s 以内通常用户都是还可以接受的。但作为一个有追求的程序猿，对自己开发的产品有归属感，你还是可以明显感受到几百毫秒到几十毫秒的巨大差异的。另外一个就是代码规范，社区已经有许多最佳实践了，借鉴最佳实践来改善自己的代码风格是另一个优化的方向。

### 参考资料
[11 Tips to Improve AngularJS Performance](http://www.alexkras.com/11-tips-to-improve-angularjs-performance/)
