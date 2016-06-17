title: Koa 是怎么让你爽的？
date: 2016-06-17 17:15:36
tags:
- nodejs
- koa
---

经常在网路上看到别人夸 Koa 写的爽，有的甚至产生了「优越感」，但写的都是种种爽的体位，却没有看到有谁详细解释为什么可以采用这些体位，以及为什么这些体位爽。其实，只要看一下 Koa 的源码，就可以知道这些原因。

<!--more  -->

当然，还是要有一些准备知识的，你需要知道 generator 是什么？`yield` 和 `yield *` 是怎么工作的？这些都可以在 [MDN](http://mdn.com) 上获取到，这里只分析 Koa 的实现，假设你已经掌握了必需的知识。

> 以下代码分析基于 Koa v1.x。

爽的地方发生在中间件的写法上面。Koa 使用一个数组来维护所有的中间件（middleware），通过调用 `app.use` 把中间件添加到这个数组中：

```
app.use = function(mdw){
  this.middleware.push(mdw)
}
```

其中中间件是 generator 函数：

```
mdw = function *(next){}

```

然后在请求过来的时候，http server 会调用 Koa 的 callback，然后请求通过 Koa 中间件，最后返回结果：

```
app.callback = function(){
  var fn = co.wrap(compose(this.middleware))
  return function(req, res){
    fn.call(...)
  }
}
```

主要流程就是这样了，Koa 之所以支持那些很爽的写法，就是来自下面这段代码：

```
co.wrap(compose(this.middleware))
```

那么这段代码究竟有什么黑魔法？且听我慢慢道来。

第一步，组合中间件，调用 `compose` 函数并传入中间件数组作为参数，`compose` 的实现非常简单，源码如下：

```
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

`compose` 函数组合中间件的方式很巧妙，它递归地把后一个中间件调用返回的 generator 对象作为参数传给上一个中间件进行调用，
`while` 循环结束后 `next` 值为第一个中间件传入第二个中间件调用后的 generator 对象为参数进行调用后返回的 generator 对象，
是不是有点绕？那来点简单的代码解释一下：

```
function* log(next) {
    yield [1]
    yield next
    yield [3]
}

function* gen(next) {
    yield [2]
    yield next
    yield [4]
}

function* compose(next) {
    if (!next) { next = noop() }
    next = gen(next)
    next = log(next)
    return yield * next
}
var a = compose()
console.log(a.next())
console.log(a.next())
console.log(a.next())

function* noop() {}
```

在 Koa 中，可以通过 `yield next` 跳转出当前中间件，

`compose` 函数返回一个 generator 函数 gen:

```
gen = function *(next){
  if (!next) next = noop();
  var i = middleware.length;
  while (i--) {
    next = middleware[i].call(this, next);
  }
  return yield *next;
}
```

第二步，调用 `co.wrap()` 函数，传入上一步得到的 `gen` 函数作为参数：

```
fn = co.wrap(gen)
```

`co.wrap()` 函数返回一个辅助函数 `createPromise`：

```
fn = function createPromise(){
  return co.call(this, gen.apply(this, arguments))
}
```

http 请求过来的时候，fn 函数调用，此时执行 `gen` 函数，并将结果作为参数传给 `co`：

```
co.call(this, gen.apply(this, arguments))
```

注意，generator 函数在被调用的时候，函数体并不会立马执行，只是返回一个 generator 对象，调用 generator 对象的 next 方法的时候，才会执行函数体，并且在遇到 yield 的时候暂停执行。

这时候，co 开始上场了。我们知道 kao1 中，中间件是 generator 函数，所以去掉其他条件判断，co 的主要代码如下：

```
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

所以先调用了 `onFulfilled()` 函数，调用 `gen.next()`，`compose` 函数的函数体开始执行，

```
function* log(next) {
    console.log(1)
    yield [1, 'a']
    yield next
    console.log(2)
    yield [2]
}

function* gen(next) {
    console.log(3)
    yield [3, 'b']
    yield next
    console.log(4)
    yield [4]
}

function* compose(next) {
    if (!next) {
        next = noop()
    }
    next = gen(next)
    next = log(next)
    return yield * next
}
var a = compose()
console.log(a.next())
console.log(a.next())
console.log(a.next())
console.log(a.next())
console.log(a.next())

function* noop() {}
```

简化一下 middleware 流程：

```
function* log(next) {
    console.log(this.url)
    console.log(1)
    yield [1, 'a']
    yield next
    console.log(2)
    yield [2]
}

function* gen(next) {
    console.log(3)
    yield [3, 'b']
    yield next
    console.log(4) yield [4]
}

function* compose(next) {
    if (!next) { next = noop() }
    next = gen(next)
    next = log(next)
    return yield * next
}
var a = compose()
a.next()
a.next()
a.next()
a.next()
a.next()

function* noop() {}
```
