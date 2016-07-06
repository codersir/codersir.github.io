title: redux 生态系统
date: 2016-07-06 01:35:39
tags:
- react
- redux
---
在学习 react 的过程中，我们了解到 react 组件其实是[状态机](https://github.com/nixzhu/dev-blog/blob/master/2015-04-23-state-machine.md)，那么，随着应用庞大组件多起来，就不可避免的会面临状态管理的问题。最初，facebook 官方推荐的状态管理方案是 [Flux](https://facebook.github.io/flux)，开源社区也产生了许多基于 Flux 的变种，比如 ，相较 Flux 都有所改进，但和官方背景顶多能达到分庭抗礼，却无法一统江湖。直到 redux 横空出世，以更简洁直接的方案和对中间件的支持，在社区中迅速获得大量的拥泵，现在几乎已经成了 react 应用的标配。由于对中间件的支持，开源社区产出了许多优秀的 redux 中间件，比如 redux-logging, redux-redo-undo, redux-thunk 等，慢慢的形成了一个生态系统。
如果你还不了解 redux，可以看我写的[「深入学习 redux」](/2016/01/25/dive-deep-in-redux/)。

<!-- more -->

## react-redux
redux 是一种状态管理方案，虽然主要在 react 社区大放异彩，但是它并不限于和 react 一起使用。如果结合 react 使用，则最好配合这个包一起用，这个包提供两个 API，`Provider` 组件和 `connect()` 方法：

- `Provider`
`Provider` 组件的作用是挂载 store 到全局，使得每个组件都可以方便的获取 `store`。它的实现很简单，就是把 `Provider` 组件作为根组件，通过 react context 机制，挂载 `store` 到所有子组件的 props 上。
- `connect` 方法

## redux-logging

## redux-actions

## redux-thunk

## redux-saga

## reselect
