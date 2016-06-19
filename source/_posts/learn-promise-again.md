title: 再探 Promise
date: 2016-06-17 17:15:19
tags:
- JavaScript
- promise
---


Promise 对象用来处理异步操作,promise 永远都是异步的。Promise 有三种状态：

- pending 初始状态
- fullfilled 操作成功
- rejected 操作失败

## Promise 方法

- `promise.all`  method returns a promise that resolves when all of the promises in the iterable argument have resolved, or rejects with the reason of the first passed promise that rejects.
- `promise.race` method returns a promise that resolves or rejects as soon as one of the promises in the iterable resolves or rejects, with the value or reason from that promise.
- `promise.resolve(value)` returns a Promise.then object that is resolved with the given value
- `promise.reject(reason)` returns a Promise object that is rejected with the given reason.

## Promise prototype 方法

- promise.prototype.catch(onRejected)
- promise.prototype.then(onFullfilled, onRejected)

## 使用案例

- Battery API
- fetch API
- ServiceWorker API

```js
var p = new Promise(function(resolve, reject) {
        // Do an async task async task and then...
        if(/* good condition */) {
            resolve('Success!');
        }

        else {
            reject('Failure!');
        }
});

p.then(function() {
        /* do something with the result */
}).catch(function() {
        /* error :( */
})
```

## promise vs callback

- 流程控制更自然，就像写同步代码一样
- 链式操作更简单，没有回调地狱
- 组合异步操作更简单
- 错误处理更优雅，可复用
- 代码更干净，传入的都是参数，不像回调有些负责传入参数，callback 又负责传出结果

## 常见的错误使用 Promise(anti-promise)：

- 使用嵌套 promise => 使用 `promise.all` 替代。
- forEach/for/while 循环体内使用 promise => 使用 `promise.all` 替代。
- promise 链断裂 => 总是返回 `promise.then`
- 忘记添加 `catch()`  => 导致看不到任何错误提示，出错了定位不到
- 使用有副作用的操作而不是 return => then 函数中，只能： 1. 返回一个promise 2.返回一个同步的值 3.抛出一个错误
- 使用 deffered =>

## 参考资料

- [lie](https://github.com/calvinmetcalf/lie)：一个很小的promise库，用简洁的代码实现了 promise A+ 标准，推荐阅读
- [We have a problem with promises](https://pouchdb.com/2015/05/18/we-have-a-problem-with-promises.html)
