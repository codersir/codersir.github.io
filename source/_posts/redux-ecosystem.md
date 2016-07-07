title: Redux 生态系统
date: 2016-07-06 01:35:39
tags:
- react
- redux
---
在学习 React 的过程中，我们了解到 React 组件其实是[状态机](https://github.com/nixzhu/dev-blog/blob/master/2015-04-23-state-machine.md)，那么，随着应用庞大组件多起来，就不可避免的会面临状态管理的问题。最初，facebook 官方推荐的状态管理方案是 [Flux](https://facebook.github.io/flux)，开源社区也产生了许多基于 Flux 的变种，比如 ，相较 Flux 都有所改进，但和官方背景顶多能达到分庭抗礼，却无法一统江湖。直到 Redux 横空出世，以更简洁直接的方案和对中间件的支持，在社区中迅速获得大量的拥趸，现在几乎已经成了 React 应用的标配。由于对中间件的支持，开源社区产出了许多优秀的 Redux 中间件，比如 redux-logging, redux-undo, redux-thunk 等，慢慢的形成了一个完善的生态系统。官方文档页面也有一个 [Ecosystem](http://redux.js.org/docs/introduction/Ecosystem.html) 页面，列出了大部分热门的中间件，本文主要介绍我日常开发中用到并且深入学习了的中间件，并结合源码解释其工作原理。

> 如果你还不了解 Redux，可以看我写的「[深入学习 Redux](/2016/01/25/dive-into-redux/)」。

<!-- more -->

## react-redux
Redux 是一种状态管理方案，虽然主要在 React 社区大放异彩，但是它并不限于和 React 一起使用。如果结合 React 使用，则最好配合这个包一起用，这个包提供两个 API，`Provider` 组件和 `connect()` 方法：

- `Provider`
`Provider` 组件的作用是挂载 store 到全局，使得每个组件都可以方便的获取 `store`。它的实现很简单，就是把 `Provider` 组件作为根组件，通过 React [context](https://facebook.github.io/react/docs/context.html) 机制，挂载 `store` 到所有子组件 context 的上。
- `connect` 方法

## redux-logger

## redux-actions

## redux-thunk

## redux-saga

当看到 redux-thunk 的时候，心里就想着那么是否有 redux-co 呢？同样是异步流程控制，NodeJS 中有 [thunks](https://github.com/thunks/thunks) 和 [co](https://github.com/tj/co)，分别代表基于 callback 和基于 generator 的的异步流程控制。后来发现还真有，那就是 [redux-saga](https://github.com/yelouafi/redux-saga)。虽然官方文档主推 redux-thunk，但是 redux-saga 凭借自身机制的优势，已经超越 redux-thunk 获得了更大的关注，成为事实的标准，算是另一个社区产出超越官方主推的例子。

> 如果你还不了解 co， 可以看我写的 「[Koa 是怎么让你爽的？](/2016/06/17/dive-into-koa/)」

和 co 一样，redux-saga 也是基于 generator 的。

## reselect

学习这些组件的过程中，最让我开心的是 **看到思想的闪光**，以及大家对这些想法的热情。很多组件实现其实都很简单，短短几十行或上百行代码，但却能完美的达到某一目的。比如 reselect，只有几十行代码，实现功能也非常简单，通过记忆函数来优化 React 性能，收获了 3000 多的 star，前端组件生态终于开始有点 NodeJS package 的感觉了。
