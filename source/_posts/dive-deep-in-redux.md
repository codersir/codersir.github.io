title: 'dive-deep-in-redux '
date: 2016-01-25 16:33:25
tags:
- react
- redux
---

redux 是 react 的一个状态管理方案，它的名称来自 `array.reduce`，要掌握它的要义，首先要知道数组 `reduce` 方法的用法。

```
    arr.reduce(callback[, initialValue])
```

`reduce` 函数接受两个参数，一个回调函数和一个可选的初始值。回调函数接受四个可选参数：
- previousValue: 上次调用回调函数的返回值，或者初始值，如果有的话。
- currentValue: 当前被传入回调函数的值
- currentIndex: 当前值在数组中的索引
- array: 调用 `reduce` 方法的数组

redux 中有3个概念，分别是：
- action
- reducer
- store

它们之间的转换流程图如下：
![redux-flow](/image/blog/redux-flow.png)

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

action 的结构并没有什么限制，只要是 JavaScript 对象并且包含 `type` 字段即可。当然，为了团队协作和编码风格的统一，有个规范总是好的，可以参照 [flux-aciton](https://github.com/acdlite/flux-actions) 和 [redux-action](https://github.com/acdlite/redux-actions)。

### reducer 怎么工作
reducer 就对应这个名称来源，是 redux 中最重要的概念。和 `arr.reduce` 方法类似，它的用法是传入当前状态和要进行的操作，返回下一个状态：

```
    (previousState, action) => newState
```
**reducer 根据 action 的描述来改变 state**。**reducer 必须是纯函数**,也就是说：
- 返回值和传入的值结构一致
- 没有副作用

所以下列事情不应该在 `reducer`中进行：
- 改变传入的参数值
- 进行有副作用的操作，比如调用API和路由过渡
- 调用非纯函数，比如 `Date.now()` 和 `Math.random()`

### store 是什么
**store 用来存储应用的状态**。在 redux 中，store 是状态的中心，提供以下 API：
- 通过 `store.dispatch(action)` 分发 action
- 通过 `store.subscribe(callback)` 添加事件监听，触发时调用回调函数
- 通过 `store.getState()` 读取应用状态

一个应用只有一个 `store`，它包含该应用完整的状态树。
