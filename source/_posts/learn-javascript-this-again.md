title: 再探 JavaScript 中的 this
date: 2016-04-10 12:55:20
tags:
---

JavaScript 中的 `this`

动态作用域
this 指的是 函数**调用时**的上下文。

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


箭头函数
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


bind(this)

call(...) 和 apply(...)
