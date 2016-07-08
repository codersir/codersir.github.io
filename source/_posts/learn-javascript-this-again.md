title: 再探 JavaScript 中的 this
date: 2016-04-10 12:55:20
tags:
- JavaScript
- ecma
---

JavaScript 中的 `this` 关键字其实并不复杂，之所以需要再学一次是因为标准的推进，`this` 的值在不同的情况下表现的与之前有所不同。在 ES6 之前，我们谈 `this` 的时候，说的最多的就是 `this` 是动态作用域的。但随着 ES6 箭头函数的引入，现在也会经常看到大家说起 *词法作用域* 的 `this`。

<!-- more -->

## 动态作用域

大多数情况下，`this` 的值由函数**调用时**的上下文决定，也就是我们常说的动态作用域。它不能在函数运行时指定，而且可能因为每次调用的位置不同而不同。

```
var obj = {
    a: 1,
    foo: function foo(){
        console.log(this.a)
    }
}
function doFoo(fn){
    fn()
}
var a = 2
doFoo(obj.foo)  //2
```

## `bind` 函数

前面我们说到 `this` 可能会因为调用位置不同而不同，所以 ES5 中引入了 `bind` 函数，`bind` 函数可以设置函数的 `this` 值而不管它如何调用。

## 词法作用域

箭头函数

`this` 指向定义时的作用域，且不会改变
```
var count = 9
var obj = {
    count: 0,
    plus: function() {
        setTimeout(() => {
            this.count++;
            console.log(this.count)
        },0)
    }
}
obj.plus()  //1

var count = 9
var obj = {
    count: 0,
    plus: function() {
        setTimeout(function() {
            this.count++;
            console.log(this.count)
        },0)
    }
}
obj.plus()  //10
```

```

```


call(...) 和 apply(...)

## 参考资料

- [MDN this](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/this)
