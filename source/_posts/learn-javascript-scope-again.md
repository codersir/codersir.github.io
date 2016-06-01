title: 再探 JavaScript 中的作用域
date: 2016-04-10 12:59:34
tags: javascript
---

我们常说，JS 是一门动态（类型）语言，但当我们讨论 JS 中的作用域的时候，大多数时候说的都是静态作用域，也叫**词法作用域**。作用域是和变量息息相关的，所以当我们谈变量的时候，提到的全局变量、局部变量这些术语，其实说的就是在不同范围（作用域）中的变量。
<!-- more -->

## 什么是作用域

作用域是一套规则，用于确定在何处以及如何查找变量（标识符）。JS 中常见的作用域有：
- 全局作用域
- 函数作用域
- 块级作用域
- 模块作用域（存在于 NodeJS 和 ES6 Module中）

### 全局作用域

全局作用域的一种判定方式就是看它是否影响到整个程序。在浏览器中，全局作用域就是 `window` 对象。NodeJS 采用全局命名空间对象（[global](https://nodejs.org/api/globals.html)）来存放全局变量，但是需要注意的是，`global` 对象中存放却不一定都是全局变量，有些变量是属于模块的，比如 `__dirname`、`__filename`。

定义在全局作用域中的变量就是全局变量，全局变量在代码的各个位置都可以被访问，所以就可能会出现命名冲突，特别是在使用第三方脚本的时候。为了尽量避免暴露太多的全局变量到全局作用域，推荐使用命名空间或者模块化来解决这个问题。

### 函数作用域

函数作用域的含义是：在函数内定义的变量可以在整个函数范围内（包括嵌套的作用域中）使用。在函数外的作用域中是无法访问函数内部定义的变量很函数的。
```
function foo(){
	var a = 1
	function bar(){
		var b = 2
		console.log(a+b)
	}
	bar()
}
foo() // 3
console.log(a) // ReferenceError: a is not defined
```

通常，我们习惯的思维方式是：先定义一个函数，然后再在里面添加代码。但是反过来想，我们可以认为：从已有的代码中选择一些片段来进行封装，把它们放进一个函数作用域中“隐藏”起来。这种“隐藏”的思维是非常有用的，它能让我们规避命名冲突，同时符合最小暴露原则（再进一步其实就是模块的概念了）。同时，为了避免产生新的变量污染，一般采用立即执行函数表达式（IIFE）来“封闭”一个作用域。

```
(function IIFE(global, undefined){
	// balabala
})(window)
```

### 块级作用域

在 ES6 之前，我们通常会说 JS 是没有块级作用域的，最常用的一个例子就是 `for` 循环：
```js
for(var i=0;i<10;i++){

}
console.log(i)	// 10
```
这段代码会输出 10 而不是 `undefined`，这就是由于缺少块级作用域导致的问题：我们原想定义在循环体内的变量在外部也可以访问。ES6 为了改变现状，新增了 `let` 用来解决这一问题。
```js
for(let i=0;i<10;i++){

}
console.log(i)	// ReferenceError: i is not defined
```
`var` 和 `let` 的另一个区别在于，`var` 和 `let` 声明的变量都会被提升到代码块的顶部，但 `var` 在声明前使用会得到 `undefined`，`let` 在声明前使用则会报`ReferenceError`的错误。

```js
console.log(a)	//undefined
var a = 1

console.log(b)	//ReferenceError: b is not defined
let b = 2
```
同样会创建块级作用域的除了 `let` 还有 `const`，但它的值是固定的。

那么 ES6 之前的 JavaScript 是否真的没有块级作用域呢？其实还是有的。那就是 `try/catch` 代码块所创建的作用域。

```js
try{
	undefined()	//TypeError: undefined is not a function(…)
} catch(err){
	console.log(err)
}
console.log(err)	//ReferenceError: err is not defined
```

### 模块作用域

模块作用域的含义是：在模块中定义的变量，只在这个模块范围内可以访问。在 ES6 之前，JavaScript 中并没有原生的模块的支持，所以模块作用域只对 NodeJS 有意义。ES6 引入了对模块的支持。

### 其他作用域

和模块作用域类似的还有文件作用域，比如 C/C+++ 采用的就是文件作用域，定义在文件中的全局变量和全局函数都属于这个文件，在更高级的语言中，文件作用域被模块作用域所取代，比如 Python、NodeJS、ES6 module。

## 词法作用域查找的特点

- 作用域会在找到第一个匹配的标识符时停止，在多层嵌套的作用域中可以定义同名的标识符，内部的标识符会“遮蔽”外部的标识符，这叫做“遮蔽效应”。
- 词法作用域只会查找一级标识符，比如 `foo.bar.baz`，词法作用域只会试图查找 `foo` 标识符，找到这个变量后，对象访问属性规则接管对 `bar` 和 `baz` 的访问。

作用域是个很基础但是很重要的概念，所有的计算机语言都会花大量的篇幅来解释作用域，各个语言的作用域划分也都各有不同，这里我们对作用域的讨论只在 JavaScript 的范围之内。

动态类型的好处是书写简单，我们写 JS 的时候不需要定义类型或接口，也不需要担心把 string 变量赋值 boolean 会报错。但任何事物都是两面的，享受了书写代码时的便利，就要付出不便于调试和理解的代价。如果你对
“静态 JavaScript ”感兴趣的话，强烈推荐你试一试 TypeScript，可以参考我写的[TypeScript 入门教程](/2016/01/12/learn-typescript/)

## 其他资源

- <a href="https://en.wikipedia.org/wiki/Scope_(computer_science)">Wikipedia scope</a>
- (JavaScript Scoping and Hoisting)[http://www.adequatelygood.com/JavaScript-Scoping-and-Hoisting.html]
