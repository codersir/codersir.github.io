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

### `Zone.fork(ZoneSpec)`
复制一个子 zone，传入子 zone 的配置规则，通常是一系列的生命钩子：

- `onFork`，拦截 zone 的复制
- `onIntercept`，拦截回调函数的包装
- `onInvoke`，拦截回调函数的调用
- `onHandleError`，拦截错误处理
- `onScheduleTask`，拦截任务调度
- `onInvokeTask`，拦截任务执行
- `onCancelTask`，拦截任务取消
- `onHasTask`，任务队列状态变化通知

通过 `properties` 参数，还可以给子 zone 传入其他的属性，通过 `zone.get` 方法获取这些属性：

```
function main(){
  Zone.current.get('reset')()
  setTimeout(function(){
    console.log('Timeout callback called after ' + Zone.current.get('time')())
  }, 1000)
}

var mySpec = (function(){
  var time = 0, start = 0
  var timer = performance ?
                performance.now.bind(performance) :
                Date.now.bind(Date)
  return {
    onScheduleTask: function(delegate, current, target, task){
      start = timer()
      delegate.scheduleTask(target, task)
      console.log('scheduling ' + task.source + ' => ' + task.data.handleId)
    },
    onInvokeTask: function(delegate, current, target, task){
      delegate.invokeTask(target, task)
      time += timer() - start
      console.log('Invoking ' + task.source + ' => ' + task.data.handleId + ' after ' + time + ' ms')
    },
    properties: {
      reset: function(){
        time = 0
        start = 0
      },
      time: function(){
        return timer() - start + ' ms'
      }
    }
  }
})()

Zone.current.fork(mySpec).run(main)
```

### `Zone.wrap`

包装回调函数使之在调用过程中可以正确的恢复当前 zone。在函数被包装起来之前，可以通过配置 `ZoneSpec.onIntercept` 来进行拦截。

### `Zone.run`

在指定的 zone 中调用函数，返回回调函数执行后的返回值。在回调函数被调用之前可以通过配置 `ZoneSpec.onInvoke` 来进行拦截。

### `Zone.runGuarded`

`Zone.run` + 错误处理，任何的错误都会被转到 `ZoneDelegate.HandleError`。错误在处理之前可以通过配置 `ZoneSpec.onHandleError` 来进行拦截。

### `Zone.runTask`

在任务的 zone 中恢复当前 zone 后执行任务。任务在执行之前，可以通过配置 `Zone.onInvokeTask` 来进行拦截。

### `Zone.scheduleMicroTask` `Zone.scheduleMacroTask` `Zone.scheduleEventTask`

安排不同类型的任务，通过 `ZoneSpec.onScheduleTask` 来进行拦截。

### `Zone.cancelTask`
拦截已安排任务的取消，任务取消之前可以通过 `ZoneSpec.onCancelTask` 来进行拦截，默认情况下任务取消会调用 `Task.cancelFn`。

## NgZone in Angular2

Angular2 通过 **变化检测** 来更新视图，那么谁来告诉 Angular2 有状态发生了改变呢？那就是 NgZone。在 Angular2 中，将不再需要不停的进行脏检查来保持视图和状态的同步，当有状态发生变化时，NgZone 的事件钩子会通知 Angular 来更新视图。Angular2 的 ZoneSpec 如下：

```javaScript
{
    name: 'angular',
    properties:<any>{'isAngularZone': true},
    onInvokeTask: (delegate: ZoneDelegate, current: Zone, target: Zone, task: Task,
                   applyThis: any, applyArgs: any): any => {
      try {
        this.onEnter();
        return delegate.invokeTask(target, task, applyThis, applyArgs);
      } finally {
        this.onLeave();
      }
    },

    onInvoke: (delegate: ZoneDelegate, current: Zone, target: Zone, callback: Function,
               applyThis: any, applyArgs: any[], source: string): any => {
      try {
        this.onEnter();
        return delegate.invoke(target, callback, applyThis, applyArgs, source);
      } finally {
        this.onLeave();
      }
    },

    onHasTask:(delegate: ZoneDelegate, current: Zone, target: Zone, hasTaskState: HasTaskState) => {
        delegate.hasTask(target, hasTaskState);
        if (current == target) {
          // We are only interested in hasTask events which originate from our zone
          // (A child hasTask event is not interesting to us)
          if (hasTaskState.change == 'microTask') {
            this.setMicrotask(hasTaskState.microTask);
          } else if (hasTaskState.change == 'macroTask') {
            this.setMacrotask(hasTaskState.macroTask);
          }
        }
    },

    onHandleError: (delegate: ZoneDelegate, current: Zone, target: Zone, error: any): boolean => {
       delegate.handleError(target, error);
       this.onError(new NgZoneError(error, error.stack));
       return false;
     }
}
```

下面这些情况会被 Angular2 判断为有状态发生了改变：

- 用户行为
- http 返回
- 定时器，`setTimeout`,`setInterval`

Zone 为这些事件都添加了钩子，用来通知 Angular 再完美不过了。

## 参考资料
- [zone.js](https://github.com/angular/zone.js)
- [angular](https://github.com/angular/angular)
