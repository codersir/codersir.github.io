title: 理解 Zone 的实现机制
date: 2016-03-13 21:02:10
tags:
- angular2
- zone
---
Zone 是一种用来拦截和追踪异步任务的机制。

Zone 可以做以下事情：
1. 拦截异步任务调度
2. 包装异步操作的错误处理和 zone 追踪的回调函数
3. 提供一种办法添加数据到 zones
4. 提供最后一帧错误处理的具体上下文
5. 拦截阻塞方法

## zone 的原理

zone 本身并不做任何事情，它依赖其他代码让平台API穿过它。

最简单的形式是，zone 通过 patch 异步API，允许我们拦截异步操作的调用和调度，并且在异步任务之前和之后执行额外的代码。

拦截规则通过 ZoneConfig 来配置。

一个系统中可以同时存在多个 zone 的实例，但是在任意时刻都只有一个处于激活状态，可以通过 `Zone.current` 获取到。

### 包装回调函数

zone 的一个重要部分就是在异步操作的过程中保持一致。为了做到这一点，当一个未来任务通过异步API被调用的时候，需要捕获并随后重现当前 zone。

```
const oldZone = _currentZone
currentZone = this
try{
  // do stuff ...
} finally {
  _currentZone = oldZone
}
```

如果你把异步操作当做线程执行来看，那么当前 Zone 就是线程局部变量。

### 异步操作调度

基本上有三种类型的任务可以被调度,每个异步 API 都通过以下 API 来建模和路由：

1. 在当前任务之后立即执行的 MicroTask，不可取消
2. 稍后执行的 TimerTask，可以取消，通常包括这些方法： `setTimeout`, `setImmediate`, `setInterval`, `requestAnimationFrame` 等
3. 用来监听未来事件的 EventTask，可能多次执行

### 组合性

Zones 可以通过 `Zone.fork()` 方法组合在一起。子 zone 可以创建它自己规则，一个子 zone ：

1. 通过父 zone 代理拦截，并且可选地在包装回调之前和之后添加钩子
2. 或不用代理自己处理请求

组合性让 zones 可以彼此保持独立不干扰。比如顶层 zone 可以选择处理错误，但子 zone 可以追踪用户行为。

### 根 zone
浏览器在开始的时候会创建一个特殊的根 zone，所有的 zone 都是根 zone 的子 zone。

## API

## NgZone in Angular2


## 参考资料
[zone.js@github](https://github.com/angular/zone.js/blob/master/lib/zone.ts)
