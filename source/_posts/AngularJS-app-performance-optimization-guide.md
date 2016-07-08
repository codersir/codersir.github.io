title: AngularJS 应用优化指南
date: 2016-05-10 16:15:33
tags:
- AngularJS
- perf
------

![Batarang](/image/ng-perf/batarang.jpg)

前两天因为想用国内的 JS CDN，访问到 [staticfile](http://www.staticfile.org/)（七牛提供的一个免费 CDN 服务），导致我的 Chrome 直接卡死了两次，页面关也关不掉，只能退出重启。后来换到 Chrome Canary 才勉强可以访问，用谷歌开发者工具一看，发现前端是用的 AngularJS ，大量的 `ng-repeat`，5万多 `watcher`，没有做任何性能优化，怎么可能不卡。
<!-- more -->

![Profile](/image/ng-perf/staticfile.png)

今年以来一直在陆陆续续的优化公司基于 AngularJS 开发的应用，也积累了一些经验，正好总结一下 AngularJS webapp 的性能优化指南。

### 1. 减少 `watcher`

我们知道 AngularJS 通过 *脏检查*（digest cicle）来更新视图，保持数据和视图的同步，脏检查的效率是和 `watcher` 的多少成正相关的，一般来说超过 2000 后就会明显感觉到变慢，所以提高 AngularJS 性能的关键就是减少 `watcher` 的数量。

首先，我们要知道的是，\** 什么会产生 `watcher` \** ？

-	`$scope.$watch`
-	`{ { stuff } }` 类模板语法
-	大多数指令 (比如 `ng-show`、`ng-if`\)
-	Scope 变量 `scope: { bar: '='}`
-	过滤器 `{ { value | myFilter } }`
-	`ng-repeat` 指令

上面这些情况都会产生 `watcher`，那么问题来了，\** 怎么减少 `watcher` \** 呢?

1.	使用单次绑定语法 `{ {::} }`
AngularJS 从1.3版本开始支持单向绑定语法 `::` ，它可以明确的告诉 AngularJS 哪些绑定获取到数据以后就不用关注了，这可以极大的减少 `watcher` 的数量，尤其是在 `ng-repeat` 内使用。
2.	避免在 `ng-repeat` 中使用 `filter`
可以先把数据过滤后再传给 `ng-repeat`，这样就能避免因为过滤器产生的 `watcher` 了。
3.	尽可能的使用 `ng-if` 而不是 `ng-show`
`ng-if` 可以从 Dom 中移除元素，触发 `element.$destory()`，删除 `ng-if` 内的元素的 `watcher`。`ng-show` 仍然会 render 元素，只是设置样式为 `display:none`。 但是如果元素需要经常变动隐藏还是显示，那么使用 `ng-show` 可能会更好，`ng-show` 会缓存 Dom，不需要重复解析。
4.	使用 `$watchCollection` 替代 `$watch`

### 2. 减少 `digest` 次数和范围

减少 `watcher` 是从根本上解决问题，如果 `watcher` 的优化已经做到极致了，那么这时候就应该换一种思路了。导致 AngularJS App 变慢的原因是 `watcher` 太多导致 `digest` 变慢，`watcher` 已经无法优化了，那么就应该考虑从 `digest` 的下手了。

同样，首先要知道的是，什么情况下会触发 AngularJS 脏检查？

-	用户行为（`ng-click`、`ng-change`、`ng-model`,etc)
-	`$http` 接口响应
-	`$q` promises resolved
-	使用 `$timeout` 和 `$interval`
-	你手动调用 `$scope.$apply` 或 `$scope.$digest`

优化主要从两个方向进行，**减少脏检查的次数** 和 **缩小脏检查的范围**。

1.	尽量使用 `$scope.$digest` 替代 `$scope.$apply`
`$scope.$digest` 从当前 scope 向下进行脏检查，而 `$scope.$apply` 会触发整个应用自顶向下进行脏检查，所以，使用 `$scope.$digest` 一般能大大的缩小脏检查的范围。

2.	使用 `$applyAsync` 合并 http 请求
通常在 App 启动的时候，会同时发起好几个 http 请求，来获取用户权限或账户信息之类的信息，每次接口返回值的时候，都会触发 AngularJS 的脏检查。这时候，如果可以等到这几个接口都返回以后，再触发脏检查，就能将脏检查的数量由几次减小到1次了。`$httpProvider` 的 [useApplyAsync](https://code.angularjs.org/1.3.8/docs/api/ng/provider/$httpProvider#useApplyAsync) 方法就是来解决这个问题，它通过 [ $rootScope.$applyAsync](https://code.angularjs.org/1.3.8/docs/api/ng/type/$rootScope.Scope#$applyAsync) 把大约同一时间（10ms左右）收到的返回值组合到一起处理。`applyAsync`的实现机制其实就是事件循环，通过 `setTimeout(fn,0)` 来延迟执行函数，可以参考我写的[《深入学习 Zone》](/2016/03/13/dive-into-zone/)了解更多。
```javascript
  app.config(function ($httpProvider) {
    $httpProvider.useApplyAsync(true)
  })
```

3.	ng-model 防抖动（ Debounce ）
搜索框通常会监听用户的 keyup 事件来进行实时匹配推荐，如果每次用户按下按键都调用接口，会出现多次连续的调用接口，导致连续的触发 AngularJS 脏检查，这样很容易造成页面卡顿。这时，可以通过 `ng-model` 的 `debounce` 参数来限制脏检查的间隔，比如 `ng-model-options="{ debounce: 250 }`，限制每 250ms 内只进行一次脏检查。

4.	使用 `$watchCollection` 替代 `$watch` 的第三个参数
`$watch` 只会比较对象引用是否相同，如果新值和原始值指向同一个索引，那么 `$digest` 时就不会触发回调函数。如果要监视对象的每个属性，我们可以给 `$watch` 传入第三个参数 `true`，这样 AngularJS 就会对对象进行深比较（使用 `angular.equals`)，遍历对象的每个值判断是否发生了变化。但如果对象比较复杂，这样做就会带来很大的性能损耗。所以，AngularJS 提供了 `$watchCollection` 方法来解决这一问题。`$watchCollection` 在脏检查的时候对对象进行浅比较，只会比较对象的第一层属性。

5.	尽量把 DOM 操作移到指令中
比如 `ng-show` 和 `ng-hide`，我们经常通过这些指令来控制元素的显示和隐藏，但这些指令的表达式值都会被 AngularJS 监听，导致 `watcher` 增加，而且这些值的变化通常也会引发 AngularJS 的 digest。我们应该尽可能的把这些逻辑移到指令的 `link` 函数中。当然，这一点最后考虑。

### 3. 其他建议

1.	使用 `track by` 提高 `ng-repeat`性能

2.	禁用 debug 信息 我们看到使用 AngularJS 指令的元素上被添加了许多类，比如 `ng-binding`、`ng-scope`等，这些类除了调试没有任何作用，
```
$compileProvider.debugInfoEnabled(false)
```

3.	耗时的计算考虑移到 web workers 执行

### 工具

怎么测时间都花在哪了？如果我们担心某个函数会很耗时，可以简单的把`console.time()` 和 `console.timeEnd()` 放在代码的前后来测试代码的运行时间。

```js
console.time('myTimer')
// your code here
console.timeEnd('myTimer')
```

这两个函数可以帮助我们测试某一小段代码的运行时间，如果要观测整个应用的运行时间，就要使用下面这两个工具了：

-	[AngularJS Batarang](https://chrome.google.com/webstore/detail/angularjs-batarang/ighdmehidhipcmcojjgiloacoafjmpfk)
-	Chrome Timeline

具体怎么使用这两种工具就不细说了，尤其是 Chrome 开发者工具，每个前端工程师都应该学会用它进行性能调优。

> [Batarang](https://en.wikipedia.org/wiki/Batarang) 是一种很有意思的武器，蝙蝠形状的回旋刀，是各种电影动画里面蝙蝠侠的武器。

### 总结

从去年下半年开始，国内很多大公司都开始用 AngularJS 开发用户后台了，比如 upyun、 Ucloud（新版 UCloud 用户后台做的很不错） 和 阿里云。但 F12 查看源代码就会发现，基本上都没有做任何的性能优化。当然，现在电脑性能基本上都处于过剩的状态，即使不做任何优化，只要页面的 `watcher` 数量没有多到 staticfile 那样，基本上也不会有什么问题，最多就是把页面响应时间从几百毫秒提升到几十毫秒，1s 以内通常用户都是还可以接受的。但作为一个有追求的程序猿，对自己开发的产品有归属感，你还是可以明显感受到几百毫秒到几十毫秒的巨大差异的。另外一个就是代码规范，社区已经有许多最佳实践了，借鉴最佳实践来改善自己的代码风格是另一个优化的方向。

### 参考资料

- [11 Tips to Improve AngularJS Performance](http://www.alexkras.com/11-tips-to-improve-angularjs-performance/)
- [Angular Performance 101](http://www.codelord.net/2014/06/17/angular-performance-101-slides/)
