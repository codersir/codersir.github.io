title: Redux 生态系统
date: 2016-07-06 01:35:39
tags:
- react
- redux
---

在学习 React 的过程中，我们了解到 React 组件其实是[状态机](https://github.com/nixzhu/dev-blog/blob/master/2015-04-23-state-machine.md)，那么，随着应用庞大组件多起来，就不可避免的会面临状态管理的问题。最初，facebook 官方推荐的状态管理方案是 [Flux](https://facebook.github.io/flux)，开源社区也产生了许多基于 Flux 的变种，比如 ，相较 Flux 都有所改进，但和官方背景顶多能达到分庭抗礼，却无法一统江湖。直到 Redux 横空出世，以更简洁直接的方案和对中间件的支持，在社区中迅速获得大量的拥趸，现在几乎已经成了 React 应用的标配。由于对中间件的支持，开源社区产出了许多优秀的 Redux 中间件，比如 redux-logger, redux-undo, redux-thunk 等，慢慢的形成了一个完善的生态系统。官方文档页面也有一个 [Ecosystem](http://redux.js.org/docs/introduction/Ecosystem.html) 页面，列出了各种 Reudx 相关的中间件、组件和小工具等，本文主要介绍我日常开发中用到并且深入学习了的中间件，并结合源码解释其工作原理。

> 如果你还不了解 Redux，可以看我写的「[深入学习 Redux](/2016/01/25/dive-into-redux/)」。

<!-- more -->
## Redux 中间件

如果你了解 express 或 koa，那么你应该对中间件的概念很熟悉了，这些框架的中间件作用在接到请求和返回响应之间，进行 log 日志、添加 CORS 头等任务，中间件的最大特点就是它们可以链式组合。 Redux 中间件虽然处理的问题不一样，但是它们概念上是相似的。Redux 中间件**作用在分发 action 和被 reducer 处理之间**。官方文档对 [Redux middleware](http://redux.js.org/docs/advanced/Middleware.html) 有一个很好的介绍，通过一步步改进一个 logger 中间件让你理解中间件的机制，推荐查看。以文档 logger 中间件为例，展示一个标准 Redux 中间件的写法：

```
const logger = store => next => action => {
  console.log('dispatching', action)
  let result = next(action)
  console.log('next state', store.getState())
  return result
}
```

Redux 中间件接受 store 作为参数，并返回一个新的函数，这个函数会成为 Redux 中间件链上的一环，当调用 dispatch 函数的时候，会依次穿过所有的中间件。

## react-redux

Redux 是一种状态管理方案，虽然主要在 React 社区大放异彩，但是它并不限于和 React 一起使用。如果结合 React 使用，则最好配合这个包一起用，这个包提供两个 API，`Provider` 组件和 `connect()` 方法：

- `Provider`
`Provider` 组件的作用是挂载 store 到全局，使得每个组件都可以方便的获取 `store`。它的实现很简单，就是把 `Provider` 组件作为根组件，通过 React [context](https://facebook.github.io/react/docs/context.html) 机制，挂载 `store` 到所有子组件 context 的上。
- `connect`
`connect` 函数的作用是关联 store 到 React 组件。

## redux-actions

action 是描述发生了什么的对象，它是 Redux 中最简单的概念。对 action 对象只有一个限制，那就是包含一个合适的 `type` 字段，所以大家可能写出各种各样的 action。宽松对个人是一种自由，但是对团队则会造成混乱，产生不必要的沟通成本。所以 action 需要一个规范，社区用的最多的是 [FSA](https://github.com/acdlite/flux-standard-action)。redux-actions 就是用来创建符合 FSA 规范的 actions，同时提供工具函数方便处理 actions。它有下面三个 API：

- `createAction(type, payloadCreator = Identity, ?metaCreator)`
其实这个函数名叫`createActionCreator` 更合理，它将会返回一个 action creator 函数，creator 函数的返回值是一个符合 FSA 标准的 action，但是也有自己的扩展，也就是 `meta`，结构如下：

```
{
  type: String
  payload, any
  error?: Boolean
  meta?: any
}
```
- `handleAction(type, reducer | reducerMap, ?defaultState)`
这个函数包装 reducer 让它只处理特定类型的 FSA。第二个参数可以是 reducer 函数或者是 reducer 对象，如果是对象，要采用下面的格式（灵感来自 generator）：
```
{
  next(state, action){...},
  throw(state, action){...}
}
```

- `handleActions(reducerMap, ?defaultState)`
这个函数通过 `handleAction` 创建多个 reducers，并且把它们组合成一个可以处理多个 actions 的单一 reducer，合并 reducers 调用的是下面这个函数：

```
export default function reduceReducers(...reducers) {
  // 改成 `return (defaultState, action)` 更好理解
  return (previous, current) =>
    reducers.reduce(
      (p, r) => r(p, current),
      previous
    );
}
```

这个函数返回函数的参数名让人容易误解，把它改成 `(defaultState, action)` 就容易理解了，举个例子：

```
const reducersMap = {
    ADD_TODO: (state, action) => {...},
    TOGGLE_TODO: (state, action) => {...}
}
const reducer = handleActions(reducersMap, {todos: []})
// 最后返回的 reducer 为：
（defaultState, action) => reducersMap['TOGGLE_TODO'](reducersMap['ADD_TODO'](defaultState, action), action)
```

## redux-logger

官方文档的 logger 示例还是太简单了，有时候我们需要更详细的日志信息，比如 action 的具体信息、触发的时间、异步 action 的耗时、state 的具体变化等，甚至 log 的级别和颜色、格式化 log 信息，这些 [redux-logger](https://github.com/evgenyrodionov/redux-logger) 都很贴心的提供了。logger 中间件很简单，API 可以去 github 查看，使用时需要注意的就是** Logger 只能是最后一个中间件**。

## redux-thunk

通常说 [thunk](https://en.wikipedia.org/wiki/Thunk) 函数是指通过包裹表达式使之延迟执行的函数，以达到“传名调用”提高性能的效果。JS 是“传值调用”的语言，它的 thunk 函数其实是指“部分函数”（或者说部分柯里化），把一个多参数函数变成一个单参数函数，而且参数为回调函数。redux-thunk 允许 action creator 返回一个函数而不是 action，它的功能就是使 dispatch action 延迟执行。它源码是最简单的，只有区区十几行:

```
function createThunkMiddleware(extraArgument) {
  return ({ dispatch, getState }) => next => action => {
    if (typeof action === 'function') {
      return action(dispatch, getState, extraArgument);
    }

    return next(action);
  };
}

const thunk = createThunkMiddleware();
thunk.withExtraArgument = createThunkMiddleware;

export default thunk;
```
源码也很容易理解，如果 action 是一个函数，那么就传入一些参数调用它，否则就不进行任何操作直接传给下一个中间件。 [github](https://github.com/gaearon/redux-thunk#composition) 上有个详细的例子，用来解释 redux-thunk 是如何优化异步流程控制的。

## redux-saga

当看到 redux-thunk 的时候，心里就想着那么是否有 「redux-co」 呢？同样是异步流程控制，NodeJS 中有 [thunks](https://github.com/thunks/thunks) 和 [co](https://github.com/tj/co)，分别代表基于 callback 和基于 generator 的的异步流程控制。后来发现还真有，那就是 [redux-saga](https://github.com/yelouafi/redux-saga)。虽然官方文档主推 redux-thunk，但是 redux-saga 凭借自身机制的优势，已经超越 redux-thunk 获得了更大的关注，成为事实的标准，算是另一个社区产出超越官方主推的例子。

> 如果你还不了解 co， 可以看我写的 「[Koa 是怎么让你爽的？](/2016/06/17/dive-into-koa/)」

和 co 一样，redux-saga 也是基于 generator 的。redux-saga 是测试驱动的典型，它提供了许多 effect 构造器，用来处理异步操作的同时保持代码的可测试性。redux-saga 比较复杂，详细介绍和 API 可以查看[官方文档](https://yelouafi.github.io/redux-saga)，后面也许会单独写一篇博客分析它的工作原理。

## reselect

学习这些组件的过程中，最让我开心的是 **看到思想的闪光**，以及大家对这些想法的热情。很多组件实现其实都很简单，短短几十行或上百行代码，但却能完美的达到某一目的。比如 reselect，只有几十行代码，实现功能也非常简单，通过记忆函数来优化 React 性能，收获了 3000 多的 star，前端组件生态终于开始有点 NodeJS package 的感觉了。
