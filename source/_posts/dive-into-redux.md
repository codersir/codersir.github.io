title: 深入学习 Redux
date: 2016-01-25 16:33:25
tags:
- react
- redux
---

Redux 是 React 的一个状态管理方案，它的名称来自 `array.reduce`，要掌握它的要义，首先要知道数组 `reduce` 方法的用法。

```
    array.reduce(callback[, initialValue])
```

`reduce` 函数接受两个参数，一个回调函数和一个可选的初始值。回调函数接受四个可选参数：

- previousValue: 上次调用回调函数的返回值，或者初始值，如果有的话。
- currentValue: 当前被传入回调函数的值
- currentIndex: 当前值在数组中的索引
- array: 调用 `reduce` 方法的数组

Redux 中有3个概念，分别是：

- action
- reducer
- store

它们之间的转换流程图如下：
![Redux-flow](/image/blog/redux-flow.png)

### 什么是 action

action 是简单的 JavaScript 对象，包含一个值唯一的 `type` 属性用**来描述状态的变化**。比如添加评论的 action:

```
    {
        type: 'ADD_COMMENT',
        content: 'add a comment'
    }
```

通常我们会写个 action creator 用来帮助创建 action，简化输入：

```
    addComment(text){
        return {
            type: 'ADD_COMMENT',
            content: text,
            create_at: Date.now()
        }
    }
```

如果 action creator 需要读取当前状态或者调用 API，或者进行其他有副作用的操作，比如路由过渡，那么它应该返回 `async action` 而不是 `action`。

async action（异步操作）是准备传递给 `dispatch()` 函数但却还没准备好被 `reducer` 调用的 `action`，它们会在传递给 `dispatch()` 函数前被 `middleware(中间件)` 转化为 `action`。

action 的结构并没有什么限制，只要是 JavaScript 对象并且包含 `type` 字段即可。当然，为了团队协作和编码风格的统一，有个规范总是好的，可以参照 [flux-aciton](https://github.com/acdlite/flux-actions) 和 [redux-actions](https://github.com/acdlite/redux-actions)。

### reducer 怎么工作

reducer 就对应这个名称来源，是 Redux 中最重要的概念。和 `arr.reduce` 方法类似，它的用法是传入当前状态和要进行的操作，返回下一个状态：

```
    (previousState, action) => newState
```
**reducer 根据 action 的描述来改变 state**。**reducer 必须是纯函数**,也就是说：

- 返回值和传入的值结构一致
- 没有副作用

所以下列事情不应该在 `reducer` 中进行：

- 改变传入的参数值
- 进行有副作用的操作，比如调用API和路由过渡
- 调用非纯函数，比如 `Date.now()` 和 `Math.random()`

通常，我们会根据职能分工不同分割成多个 reducer，对于这种情况，Redux 提供了 `combineReducers` 函数来把它们结合到一起：

```
function combineReducers (reducers) {
  var reducerKeys = Object.keys(reducers)
  var finalReducers = {}
  for (var i = 0; i < reducerKeys.length; i++) {
    var key = reducerKeys[i]
    if (typeof reducers[key] === 'function') {
      finalReducers[key] = reducers[key]
    }
  }
  var finalReducerKeys = Object.keys(finalReducers)

  return function combination (state = {} , action) {
    var hasChanged = false
    var nextState = {}
    for (var i = 0; i < finalReducerKeys.length; i++) {
      var key = finalReducerKeys[i]
      var reducer = finalReducers[key]
      var previousStateForKey = state[key]
      var nextStateForKey = reducer(previousStateForKey, action)
      <!-- 不能返回 undefined -->
      if (typeof nextStateForKey === 'undefined') {
        var errorMessage = getUndefinedStateErrorMessage(key, action)
        throw new Error(errorMessage)
      }
      nextState[key] = nextStateForKey
      hasChanged = hasChanged || nextStateForKey !== previousStateForKey
    }
    return hasChanged ? nextState : state
  }
}
```

`combineReducers` 函数是个闭包，它的作用是返回一个调用 reducers 的函数，调用每个 reducer 函数时传入 key 值相对应的 state 片段作为 reducer 的初始 state。

需要注意的是：

1. **我们不改变 state 而是返回它的一个副本**，可以通过 `Object.assign({}, state, newState)` 或 对象展开操作符 `{...state, ...newState}` 来完成。
2. **任何时候 reducer 都不应该返回 `undefined`**，如果下一个状态为 `undefined`，那么就返回它之前的状态，否则 Redux 会报错。

### store 是什么

**store 用来存储应用的状态**。在 Redux 中，store 是状态的中心，提供以下 API：

- 通过 `store.dispatch(action)` 分发 action
- 通过 `store.subscribe(callback)` 添加事件监听，触发时调用回调函数
- 通过 `store.getState()` 读取应用状态

一个应用只有一个 `store`，它包含该应用完整的状态树。我们通过调用 `createStore` 函数来创建 store：

```
function createStore (reducer, preloadedState, enhancer) {}
```



## 资源
- [Getting Started with Redux](https://egghead.io/series/getting-started-with-redux)
