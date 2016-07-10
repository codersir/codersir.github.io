title: 深入学习 Redux
date: 2016-01-25 16:33:25
tags:
- react
- redux
---

当我们谈 Redux 的时候，一般都会把它和 React 联系到一起。但是，Redux 作为一种状态管理方案，并不限于和 React 一起使用，你可以扩展到其他任何合适的框架，比如 Angular。Redux 的目标是 **使状态的改变可预测**。Redux 通过下面三个原则约束来实现这一目标：

- 单一数据源：整个应用的状态都存在单一 store 的对象树里面
- 状态只读：改变状态的唯一方法就是触发一个 action，action 是一个描述发生了什么的对象
- 变化都由纯函数生成：通过 reducers 来指定状态树怎么被 action 转化

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

action 的结构并没有什么限制，只要是 JavaScript 对象并且包含 `type` 字段即可。当然，为了团队协作和编码风格的统一，有个规范总是好的，可以通过[redux-actions](https://github.com/acdlite/redux-actions) 来写 FSA(Flux Standard Action)。

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
    // 过滤掉不合法的 reducer
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
      // 不能返回 undefined
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

`combineReducers` 函数是个闭包，它的作用是返回一个调用 reducers 的函数，调用每个 reducer 函数时传入 key 值对应的 state 片段作为相应 reducer 的初始 state。

需要注意的是：

1. **我们不改变 state 而是返回它的一个副本**，可以通过 `Object.assign({}, state, newState)` 或 对象展开操作符 `{...state, ...newState}` 来完成。
2. **任何时候 reducer 都不应该返回 `undefined`**，如果下一个状态为 `undefined`，那么就返回它之前的状态，否则 Redux 会报错。

### store 是什么

**store 用来存储应用的状态**。在 Redux 中，store 是状态的中心，提供以下 API：

- 通过 `store.dispatch(action)` 分发 action
- 通过 `store.subscribe(callback)` 添加事件监听，触发时调用回调函数
- `store.subscribe(callback)` 返回一个函数，调用这个函数注销事件监听

一个应用只有一个 `store`，它包含该应用完整的状态树。我们通过调用 `createStore` 函数来创建 store：

```
function createStore (reducer, preloadedState, enhancer) {
  if (typeof preloadedState === 'function' && typeof enhancer === 'undefined') {
    enhancer = preloadedState
    preloadedState = undefined
  }

  if (typeof enhancer !== 'undefined') {
    if (typeof enhancer !== 'function') {
      throw new Error('Expected the enhancer to be a function.')
    }
    // enhancer 通常为 applyMiddleware 函数调用后返回的函数
    return enhancer(createStore)(reducer, preloadedState)
  }
  ...
  // 返回 store 对象
  return {
    dispatch,
    subscribe,
    getState,
    replaceReducer,
    [$$observable]: observable
  }
}
```

`createStore` 最终返回一个 store 对象，包含 `dispatch`、`getState` 等方法。如果给 `createStore` 函数传入了 enhancer，那么就会先调用 enhancer 函数。enhancer 函数通常是一些中间件，被 `applyMiddleware` 函数调用后返回改变了 `dispatch` 函数行为的 store 对象。

```
function applyMiddleware(...middlewares) {
  return (createStore) => (reducer, preloadedState, enhancer) => {
    var store = createStore(reducer, preloadedState, enhancer)
    var dispatch = store.dispatch
    var chain = []
    // 对 middleware 只暴露 getState 和 dispatch 这两个 API
    var middlewareAPI = {
      getState: store.getState,
      dispatch: (action) => dispatch(action)
    }
    chain = middlewares.map(middleware => middleware(middlewareAPI))
    dispatch = compose(...chain)(store.dispatch) //
    // 返回新的 dispatch
    return {
      ...store,
      dispatch
    }
  }
}
```

那么新的 dispatch 函数究竟是怎样的呢？这里调用了 `compose` 函数，`compose` 函数实现如下：

```
function compose (...funcs) {
  if (funcs.length === 0) {
    return arg => arg
  }

  if (funcs.length === 1) {
    return funcs[0]
  }

  const last = funcs[funcs.length - 1]
  const rest = funcs.slice(0, -1)
  return (...args) => rest.reduceRight((composed, f) => f(composed), last(...args))
}
```

`compose` 函数的作用就是把中间件组合起来，通过 `Array.reduceRight` 函数实现柯里化，比如有两个中间件返回函数 f 和 g，被 compose 函数调用后结果为：

```
function f(){}
function g(){}
compose(f,g)(store.dispatch) // 得到 f(g(store.dispatch))，注意我们把 store.dispatch 作为初始值
```

这样处理后，以后每次调用 `store.dispatch` 函数时，都会依次穿过各个中间件，得到链式调用中间件的效果，有一点 express 中间件的感觉了。

## 总结

Redux 的代码非常简单，也很好理解，作者在 gist 有一个[瘦身版的 Redux](https://gist.github.com/gaearon/ffd88b0e4f00b22c3159)，去掉了不必要的条件判断和错误处理，只有百来行代码。虽然简单，但它背后的思想非常值得玩味，比如单一数据源、纯函数等。另外，Redux 作者的表达能力非常好，这一点在文档上得到了充分的展示，对一些概念的解释非常老练，由浅入深，层层推进，真是让人佩服。

## 参考资料
- [redux doc](http://redux.js.org/)
- [slim-redux.js](https://gist.github.com/gaearon/ffd88b0e4f00b22c3159)
- [Getting Started with Redux](https://egghead.io/series/getting-started-with-redux)
