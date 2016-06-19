title: Koa 是怎么让你爽的？
date: 2016-06-17 17:15:36
tags:
- nodejs
- koa
---

经常在网路上看到别人夸 Koa 写的爽，有的甚至产生了「优越感」，但写的都是种种爽的体位，却没有看到有谁详细解释为什么可以采用这些体位，以及为什么这些体位爽。其实，只要看一下 Koa 的源码，就可以知道这些原因。

<!--more  -->

当然，还是要有一些准备知识的，你需要知道 [generator](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Generator) 是什么？[yield](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/yield) 和 [yield *](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/yield*) 是怎么工作的？这些都可以在 [MDN](https://developer.mozilla.org/) 上获取到，这里只分析 Koa 的实现，假设你已经掌握了以上必需的知识。

> 以下代码分析基于 Koa V1.x。

爽发生在中间件的写法上面。Koa 使用一个数组来维护所有的中间件（middleware），通过调用 `app.use` 把中间件添加到这个数组中：

```js
app.use = function(mdw){
  this.middleware.push(mdw)
}
```

其中中间件是 generator 函数：

```js
mdw = function *(next){}

```

然后在请求过来的时候，http server 会调用 Koa 的 callback，然后请求通过 Koa 中间件，最后返回结果：

```js
app.callback = function(){
  var fn = co.wrap(compose(this.middleware))
  return function(req, res){
    fn.call(...)
  }
}
```

主要流程就是这样了，Koa 之所以支持那些很爽的写法，就是来自下面这段代码：

```js
co.wrap(compose(this.middleware))
```

那么这段代码究竟有什么黑魔法？且听我慢慢道来。

第一步，组合中间件，调用 `compose` 函数并传入中间件数组作为参数，`compose` 的实现非常简单，源码如下：

```js
function compose(middleware){
  return function *(next){
     if (!next) next = noop();
     var i = middleware.length;
     while (i--) {
       next = middleware[i].call(this, next);
     }
     return yield *next;
}

function *noop(){}
```

`compose` 函数组合中间件的方式很巧妙，它递归地把后一个中间件调用返回的 generator 对象作为参数传给上一个中间件进行调用，`while` 循环结束后，`next` 值为第一个中间件传入第二个中间件调用后的 generator 对象为参数进行调用后返回的 generator 对象。是不是有点绕？那来点简单的代码解释一下：

```js
function* noop() {}
function* log(next) {
    yield 1
    yield next
    yield 3
}

function* gen(next) {
    yield 2
}

function* compose(next) {
    if (!next) { next = noop() }
    next = gen(next)
    next = log(next)
    return yield * next
}
var a = compose()
console.log(a.next()) // {value: 1, done: false}
console.log(a.next()) // {value: gen, done: false}
console.log(a.next()) // {value: 3, done: false}
console.log(a.next()) // {value: undefined, done: true}
```

通过输出可以看出，在 Koa 中，我们通过调用 `yield next` 跳转出当前中间件，从 log 中间件跳到了 gen，然后又回到了 log。但是这里有个问题，我们希望输出的 2 并没有出现，所以并没有完全达到我们的期望。所以，接下来，该 co 登场了。

注意，`compose` 函数并没有立即就组合中间件，而是返回一个 generator 函数 gen:

```js
gen = function *(next){
  if (!next) next = noop();
  var i = middleware.length;
  while (i--) {
    next = middleware[i].call(this, next);
  }
  return yield *next;
}
```

第二步，调用 `co.wrap()` 函数，传入上一步 `compose` 返回的 `gen` 函数作为参数：

```js
fn = co.wrap(gen)
```

查看 co 的源码，`co.wrap()` 函数返回一个辅助函数 `createPromise`：

```js
fn = function createPromise(){
  return co.call(this, gen.apply(this, arguments))
}
```

回到上面 Koa 的 callback 源码，http 请求过来的时候，fn 函数调用，此时执行 `gen` 函数，并将结果作为参数传给 `co`：

```js
co.call(this, gen.apply(this, arguments))
```

注意：
>  **generator 函数在被调用的时候，函数体并不会立马执行，只是返回一个 generator 对象，调用 generator 对象的 next 方法的时候，才会执行函数体**，并且在遇到 yield 的时候暂停执行。

所以，这时候，中间件仍然没有组合，直到进入 co 中，`onFulfilled()` 函数内部调用 `gen.next()` 的时候，`compose` 函数返回的 generator 函数体才得以执行。这就是为什么进入 co 后立马调用 `onFulfilled()` 函数的原因。

我们知道 Koa 中，中间件是 generator 函数，所以去掉其他条件判断，co 处理中间件的主要代码如下：

```js
function co(gen) {
  var ctx = this;
  var args = slice.call(arguments, 1);

  return new Promise(function(resolve, reject) {
    if (typeof gen === 'function') gen = gen.apply(ctx, args);
    if (!gen || typeof gen.next !== 'function') return resolve(gen);

    onFulfilled();

    function onFulfilled(res) {
      var ret;
      try {
        ret = gen.next(res);
      } catch (e) {
        return reject(e);
      }
      next(ret);
      return null;
    }

    function onRejected(err) {
      var ret;
      try {
        ret = gen.throw(err);
      } catch (e) {
        return reject(e);
      }
      next(ret);
    }

    function next(ret) {
      if (ret.done) return resolve(ret.value);
      var value = toPromise.call(ctx, ret.value);
      if (value && isPromise(value)) return value.then(onFulfilled, onRejected);
      return onRejected(new TypeError('You may only yield a function, promise, generator, array, or object, '
        + 'but the following object was passed: "' + String(ret.value) + '"'));
    }
  });
}

function toPromise(obj) {
  if (isGeneratorFunction(obj) || isGenerator(obj)) return co.call(this, obj);

  return obj;
}
```
那么，上面的那个 2 是怎样输出来的呢？实现其实很简单：

co 反复调用内部的 next() 函数，判断上一次遍历器的返回值状态，如果它未结束并且是 GeneratorFunction 或是 Generator 对象，就再次把它作为参数传入 `co` 函数调用。

借由这样的机制，Koa 就可以在 `yield next` 被调用的时候进入下一个中间件，再遇到 `yield next` 进入下下个或者遍历结束返回上一个，Koa 文档有一张 Gif 很形象的描述了 Koa 中间件这种机制：

![co middleware](https://github.com/koajs/koa/blob/master/docs/middleware.gif?raw=true)
