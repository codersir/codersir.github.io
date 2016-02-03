title: what-is-decorator-in-javascript
date: 2016-01-31 00:36:01
tags: [es2016, decorator, typescript]
---

装饰器是 ES7（也就是ES2016）的提案，虽然听着感觉好像还有点遥远，毕竟 ES6 还没有走多久呢，但实际上，2016年已经到来，也就是说，ES7 的提案今年就会成为新的标准，同时借助一些编译器（比如babel），我们早已经可以在正式环境中使用它了。另外，TypeScript 在去年和
AtScript 合并后发布的 1.5 版本，就已经支持装饰器语法了。基于 TypeScript 开发的 Angular2 里面也大量的使用了装饰器。

## 什么是装饰器

装饰器是

## 怎么使用装饰器

TypeScript 已经完整的支持了 ES7 装饰器的接口，通过 [TypeScript 装饰器的实现](https://github.com/Microsoft/TypeScript/blob/master/src/lib/core.d.ts#L1254)，来进一步掌握装饰器的使用：

```
declare type ClassDecorator = <TFunction extends Function>(target: TFunction) => TFunction | void;
declare type PropertyDecorator = (target: Object, propertyKey: string | symbol) => void;
declare type MethodDecorator = <T>(target: Object, propertyKey: string | symbol, descriptor: TypedPropertyDescriptor<T>) => TypedPropertyDescriptor<T> | void;
declare type ParameterDecorator = (target: Object, propertyKey: string | symbol, parameterIndex: number) => void;
```

通过上面的代码可以知道，装饰器可以用来注解**类**，**属性**，**方法**和**参数**，下面分别看不同情况下如何使用和实现。

### 装饰类

TypeScript 类装饰器接口如下：

```
declare type ClassDecorator = <TFunction extends Function>(target: TFunction) => TFunction | void;
```
通过接口可以知道类装饰器接受一个目标函数作为参数，考虑下面简单的类：

```
class Hero {
    public name: string
    public power: string

    constructor(name: string, power: string) {
        this.name = name
        this.power = power
    }

    showPower(){
    	return `${this.name} has special power: ${this.power}`
    }
}
```

把上面的代码编译为 JavaScript，得到：

```
var Hero = (function () {
    function Hero(name, power) {
        this.name = name;
        this.power = power;
    }
    Hero.prototype.showPower = function () {
        return this.name + " has special power: " + this.power;
    };
    return Hero;
})();
```

下面给 `Hero` 类添加一个 `@logClass` 的装饰器：

```
@logClass
class Hero {
    public name: string
    public power: string

    constructor(name: string, power: string) {
        this.name = name
        this.power = power
    }

    showPower(){
    	return `${this.name} has special power: ${this.power}`
    }
}
```

重新编译成 JavaScript ：

```
var Hero = (function () {
    function Hero(name, power) {
        this.name = name;
        this.power = power;
    }
    Hero.prototype.showPower = function () {
        return this.name + " has " + this.power;
    };
    Hero = __decorate([
        logClass
    ], Hero);
    return Hero;
})();

```

上面的代码在返回`Hero`构造函数之前，将它作为参数传给了`__decorate`函数，那`__decorate`函数做了什么？看编译后的 JavaScript 代码，可以看到该函数实现如下：

```
// 保证上下文中只有一个 __decorate 函数，不会被反复重载
var __decorate = (this && this.__decorate) || function(decorators, target, key, desc) {
    var c = arguments.length,
    	// 装饰类的时候，__decorate 函数接受两个参数，所以 r = target，也就是上文的构造函数 Hero
        r = c < 3 ? target : desc === null ? desc = Object.getOwnPropertyDescriptor(target, key) : desc,
        d;
    // 这是另外一个 ES7 API 了，先不管，看fallback方案
    if (typeof Reflect === "object" && typeof Reflect.decorate === "function") r = Reflect.decorate(decorators, target, key, desc);
    else
    	// 从右向左依次执行 decorator 函数，对 class 而言，相当于 decorators.reduceRight(function(o, d){ return (d && d(r)) || o }, r)
        for (var i = decorators.length - 1; i >= 0; i--)
            if (d = decorators[i]) r = (c < 3 ? d(r) : c > 3 ? d(target, key, r) : d(target, key)) || r;
    最后返回 r
    return c > 3 && r && Object.defineProperty(target, key, r), r;
};
```
`__decorate` 函数体其实很简单，主要是进行一些条件判断，根据传入参数个数的不同判断装饰器的类型，并执行相关的装饰器函数。

需要注意的两个函数是：
- [Object.getOwnPropertyDescriptor](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object/getOwnPropertyDescriptor)
- [Object.defineProperty](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object/defineProperty)
第一个函数是获取对象自有属性的描述符，第二是添加或修改对象的自有属性，具体用法查看 MDN。

知道装饰器装饰类是如何进行工作后，下面来完善 `@logClass` 装饰器。通过类装饰器接口可以知道，它接受一个函数作为参数，并返回函数。 `@logClass` 装饰器的目的是，在每次英雄被派出去拯救世界的时候，
记录下来是哪个英雄出去了，为他祈祷，实现如下：

```
function logClass(target: any) {

  // 保存原始构造器
  var original = target;

  // 工具函数，生成类的实例
  function construct(constructor, args) {
    var c : any = function () {
      return constructor.apply(this, args);
    }
    c.prototype = constructor.prototype;
    return new c();
  }

  // 添加行为到构造器调用时
  var f : any = function (...args) {
    console.log("God bless " + args[0]); 
    return construct(original, args);
  }

  // 复制原始构造器的原型
  f.prototype = original.prototype;

  // 返回新的构造器
  return f;
}
```
上面的代码已经清晰的注释了，就不添加更多的说明了，可以看到，类装饰器就是**给构造函数添加额外的动作**。现在当派出一个新的英雄之前，我们会为他祈福：

```
var batman = new Hero('batman', 'fly')
// God bless batman

batman instanceof Hero
// true
```

### 装饰属性

TypeScript 属性装饰器接口如下：

```
declare type PropertyDecorator = (target: Object, propertyKey: string | symbol) => void;
```

由接口可以知道，属性装饰器接受两个参数，一个是目标对象，另一个是属性名称，没有返回值。
继续使用上面 Hero 类，这次给 `power` 属性加上 `@logProperty` 属性装饰器，在给英雄使用超能力的时候，添加 Bgm，代码如下：

```
class Hero {
    public name: string
    @logProperty
    public power: string

    constructor(name:string, power:string){
        this.name = name
        this.power = power
    }

    showPower(){
        return `${this.name} has special power: ${this.power}`
    }
}
```

同样，编译成 JavaScript 后得到：

```
var Hero = (function () {
    function Hero(name, power) {
        this.name = name;
        this.power = power;
    }
    Hero.prototype.showPower = function () {
        return this.name + " has " + this.power;
    };
    __decorate([
        logProperty
    ], Hero.prototype, "power");
    return Hero;
})();
```

此处再次用到了 `__decorate` 函数，但传入参数的个数是3个，装饰器，构造函数的原型和属性名，所以精简一下 `__decorator` 函数，得到属性装饰器的版本：

```
var __decorate = (this && this.__decorate) || function(decorators, target, key, desc) {
    var c = arguments.length,
    	r = Object.getOwnPropertyDescriptor(target, key)
        d;
    if (typeof Reflect === "object" && typeof Reflect.decorate === "function") r = Reflect.decorate(decorators, target, key, desc);
    else
    	// 从右向左依次执行 decorator 函数，相当于 decorators.reduceRight(function(o, d){ return (d && d(target, key) || o }, r)
        for (var i = decorators.length - 1; i >= 0; i--)
            if (d = decorators[i]) r = d(target, key) || r;
    return r;
};
```
另外对比类装饰器，可以看到这次没有用 `__decorate` 函数的返回值覆盖原始的类，属性装饰器的接口是没有返回值（=>void)的。

现在知道了属性装饰器接受构造器的原型和属性名作为参数，并且没有返回值，下面开始着手实现 `@logProperty` 装饰器：

```
function logProperty(target: any, key: string) {

  // 存储属性值
  var _val = this[key];

  // 获取属性
  var getter = function () {
    console.log(`此处应有 Bgm ${_val}`);
    return _val;
  };

  // 设置属性
  var setter = function (newVal) {
    console.log(`切换特技 ${newVal}`);
    _val = newVal;
  };

  // 删除属性并重新设置该属性
  if (delete this[key]) {

    // 创建一个新的属性，使用我们自定义的存储器
    Object.defineProperty(target, key, {
      get: getter,
      set: setter,
      enumerable: true,
      configurable: true
    });
  }
}
```
// todo
// this在此处的使用需要进一步说明，为什么 this 的上下文是构造器原型？

现在，英雄放大招和切换大招的时候，我们就会看到控制台输出了：

```
var batman = new Hero('batman', 'fly')
// 切换特技 fly
batman.showPower()
// 此处应有 Bgm
batman.power = 'doublekill'
// 切换特技 doublekill
```

### 装饰方法

TypeScript 属性方法接口如下：

```
declare type MethodDecorator = <T>(target: Object, propertyKey: string | symbol, descriptor: TypedPropertyDescriptor<T>) => TypedPropertyDescriptor<T> | void;
```
方法装饰器接受的参数最多，接受三个参数，返回属性描述符或者没有返回。这次我们装饰 `showPower`方法，给它添加 `@logMethod` 装饰器：

```
class Hero {
    public name: string
    public power: string

    constructor(name:string, power:string){
        this.name = name
        this.power = power
    }
    
    @logMethod
    showPower(){
        return `${this.name} has special power: ${this.power}`
    }
}
```

编译为 JavaScript 得到：

```
var Hero = (function () {
    function Hero(name, power) {
        this.name = name;
        this.power = power;
    }
    Hero.prototype.showPower = function () {
        return this.name + " has special power: " + this.power;
    };
    __decorate([
        logMethod
    ], Hero.prototype, "showPower");
    return Hero;
})();
````
此时 `__decoreate`函数去掉条件判断得到：

```
var __decorate = (this && this.__decorate) || function(decorators, target, key, desc) {
    var c = arguments.length,
        r = desc || Object.getOwnPropertyDescriptor(target, key),
        d;
    if (typeof Reflect === "object" && typeof Reflect.decorate === "function") r = Reflect.decorate(decorators, target, key, desc);
    else
        for (var i = decorators.length - 1; i >= 0; i--)
            if (d = decorators[i]) r = d(target, key, r) || r;
    return r && Object.defineProperty(target, key, r), r;
};
```
这里的实现我觉得是有问题的，
下面我们来实现 `@logMethod` 装饰器：
```
function logMethod(target:any, key:string, descriptor: object){
  var originalMethod = descriptor.value; 

  descriptor.value =  function (...args: any[]) {
      var a = args.map(a => JSON.stringify(a)).join();
      var result = originalMethod.apply(this, args);
      var r = JSON.stringify(result);
      console.log(`Call: ${key}(${a}) => ${r}`);
      return result;
  }

  return descriptor;
}
```


### 装饰参数


### 给装饰器传入参数

## reference

- [javascript-decorators提案](https://github.com/wycats/javascript-decorators)以及 [我的翻译版本](https://github.com/xuhong/javascript-decorators)
- [decorators-reflection-javascript-typescript](http://blog.wolksoftware.com/decorators-reflection-javascript-typescript)
