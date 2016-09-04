title: 再探 JavaScript 中的 this
date: 2016-04-10 12:55:20
tags:
- JavaScript
- ecma
---

JavaScript 中的 `this` 关键字其实并不复杂，之所以需要再学一次是因为标准的推进，`this` 的值在不同的情况下表现的与之前有所不同。在 ES6 之前，我们谈 `this` 的时候，说的最多的就是 `this` 是动态作用域的。但随着 ES6 箭头函数的引入，现在也会经常看到大家说起 *词法作用域* 的 `this`。

<!-- more -->

## `this` 是什么？

`this` 是一个**关键字**，ECMAScirpt 标准定义 `this` 为 **ThisBindings 计算的值为当前执行上下文**。

## 词法作用域与“动态作用域”

我们都知道 ECMAScript 是没有动态作用域的，变量属于它定义时所在的作用域且不会发生改变。但是，大多数情况下，`this` 的值由函数**调用时**的上下文决定，也就是 `this` 表现的却像是“动态作用域”。它不能在函数运行前指定，而且可能因为每次调用的位置不同而不同。`this` 和作用域的关系，引用权威指南的原句：

> 和变量不同，关键字 this 没有作用域限制，嵌套的函数不会从调用它的函数中继承 this。如果嵌套函数作为方法调用，其 `this` 值指向调用它的对象，如果嵌套函数作为函数调用，其 `this` 值不是全局对象（非严格模式下）就是 undefined（严格模式下）。

```
var count = 9
var obj = {
    count: 0,
    plus: function() {
        setTimeout(function() {
            // this 指向全局对象
            this.count++;
            console.log(this.count)
        },0)
    }
}
obj.plus()  //10
```
解决这一问题的常见方法是定义一个变量 `self/that` 来保存当前的执行上下文：

```
var count = 9
var obj = {
    count: 0,
    plus: function() {
        var self = this
        setTimeout(function() {
            self.count++;
            console.log(self.count)
        },0)
    }
}
obj.plus()  //1
```
这样虽然解决了问题，但是却也容易带来困惑，人们容易记住使用这个方法就可以解决问题，却不去深究 `this` 的实现机制到底是怎样的，也不会去寻找更好的方法来解决这个问题：那就是词法作用域。

## `bind` 函数

为了更好的解决上面的问题，回到 JavaScript 熟悉的词法作用域来，ES5 中引入了 `bind` 函数，`bind` 函数可以设置函数的 `this` 值而不管它如何调用，将 `this` 绑定到当前的词法作用域。

```
var count = 9
var obj = {
    count: 0,
    plus: function() {
        setTimeout(function() {
            this.count++;
            console.log(this.count)
        }.bind(this),0)
    }
}
obj.plus() // 1
```

## 箭头函数

ES6 中新加入了对箭头函数的支持，`this` 指向定义时它的上下文且不会改变，遵循词法作用域规则。

```
var count = 9
var obj = {
    count: 0,
    plus() {
        setTimeout(() => {
            this.count++;
            console.log(this.count)
        },0)
    }
}
obj.plus()  //1
```

之所以这样，背后真正的原因是：**箭头函数没有自己的 `this`，箭头函数的 `this` 是从包裹它的作用域继承过来的**。

所以，在 ES6 后，你几乎不需要再提心吊胆担心掉进 `this` 的坑里，只要你遵循下面两条规则：

- 对方法都使用 **非** 箭头函数
- 其他的地方都使用箭头函数

## call(...) 和 apply(...)

JavaScript 中除了 `bind` 函数可以手动绑定 `this` 以外，还提供了 `call` 和 `apply` 函数用来在调用函数的时候，手动改变函数调用上下文。`call` 和 `apply` 函数的第一个参数就是本次调用的上下文：

```
var count = 9
var obj = {
    count: 0,
    plus: function() {
        setTimeout(function() {
            this.count++;
            console.log(this.count)
        }.bind(this),0)
    }
}
obj.plus() // 1
obj.plus.apply(this) // 10
```

## `new` 构造函数

当通过 `new` 操作符创建对象时，JavaScript 编译器会先创建一个新的空对象，然后设置一些内部的属性，并调用新对象的构造函数。因此，当通过 `new` 调用构造函数时，`this` 指向这个新创建的对象：

```
let F = ()=>{
  this.a = 1
  console.log(this)
}

F() // Window
new F() // {a:1}
```

## `eval`

直接调用 `eval`，`this` 和 `eval()` 执行环境中的 `this` 保持一致。

```
eval('this===window') // true
use strict; // 严格模式下
eval('this===undefined') // true
// 方法中
var obj = {  
  method: function () {
    console.log(eval('this') === obj)
  }
}
obj.method() // true
```

## `class` 类里面的 `this`

类会在调用 constructor 函数的时候初始化 `this`，子类 constructor 函数中在调用 `super()` 以后才能使用 `this`，否则会报 `ReferenceError: this is not defined`。

## 总结

上面基本上列出了 JavaScript 中与 `this` 相关的方方面面了，掌握 `this` 并不难，随着 ES6 的推进，善用箭头函数，掉进坑的机会就更少了。

## 参考资料

- [stackoverflow: How does the “this” keyword work?](http://stackoverflow.com/questions/3127429/how-does-the-this-keyword-work)
- [MDN this](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/this)
- [You-Dont-Know-This](https://github.com/getify/You-Dont-Know-JS/blob/master/scope%20&%20closures/apC.md)
