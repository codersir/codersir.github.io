title: learn-typescript
date: 2016-01-12 12:58:51
tags: typescript, angular2
---

TypeScript 是微软出的一个 JavaScript 超集，给 JavaScript 添加了类型，可以编译成 plain JavaScript。最近开始学习Angular2，Angular2 支持 TypeScript 来编写，也是社区推荐的方式。

### 基本类型
TypeScript 为你添加了你期望 JavaScript 拥有的基本数据类型：numbers、strings、structures、boolean、array、enum 等。

```
    var isDone: boolean = false;    //boolean
    var height: number = 6;         //number
    var name:string = 'xuhong';     //string
    var list:number[] = [1,2,3];    //array 
    var list:Array<number> = [1,2,3]//generic array type
    
    enum Color {Red:1, Green, Blue}
    var c: Color = Color.Red;       //enum
    var colorName:string = Color[2] //get enum value use number

    var notSure:any = 4;            //any
    notSure = false;

    function warning:void{          //no return value's function's return type is void
        console.warning('warning message')
    }
```

### Interfaces
TypeScript 的一个主要原则是采用鸭式辩形或者说是构造子类来进行类型检查。在 TypeScript 中，接口就是充当这样的角色来命名这些类型，同时也是你项目内部代码以及和外部代码进行协作的有效方式。

当不使用接口时，我们这样写代码：

```
    function sayHello(person: {name: string}) {
        console.log("hello " + person.name)
    }
    var me = {name: 'xuhong', age: 24}
    sayHello(me)
```

sayHello 函数接受一个参数，该参数必须是个对象并且有一个名为 `name` 的属性。虽然传入的对象还有其他的属性，但是编译器只会检查必须的属性以及它的类型。下面用接口重新写这个函数：

```
    interface personName {
        name: string
    }

    function sayHello(person: personName) {
        console.log("hello " + person.name)
    }

    var me = {name: 'xuhong', age: 24}
    sayHello(me)
```

现在我们可以使用接口 `personName` 来描述参数的要求。注意我们不需要像其他的语言一样写明我们传进 `sayHello` 的对象实现了这个接口，只要这个对象满足接口列出的所有要求就够了。
另外，并不是所有的参数都是必须的，接口也支持可选参数，在属性名后面添加一个 ? 表明这个属性是可选的。

#### 函数类型
接口可以描述许多 JavaScript 对象的形状。除了可以描述拥有属性的对象，接口也可以描述函数类型。

``` 
    interface SearchFunc {
        (source: string, subString: string): boolean
    }

    var mySearch: SearchFunc
    mySearch = functiuon(src: string, sub: string){
        return src.search(sub) == -1
    }
```

#### 数组类型
以及描述数组类型

```
    interface StringArray {
        [index: number]: string
    }

    var myArr: stringArray
    myArr = ['abc', 'def']
```
TypeScript 支持两种索引类型：数字和字符串。而且可以同时支持两种索引，但是要求数字索引返回的值是字符串索引返回的值的子类。它也要求所有的属性匹配它们的返回值类型，下面的例子会被类型检查器报错，因为length属性的类型和索引的类型不匹配

```
    interface Dictionary {
        [index: string]: string;
        length: number;
    }
```

#### 类类型
当类实现接口时，只有类的实例部分会被检查，构造器部分是静态部分，不会被检查。

```
    interface ClockInterface {
        currentTime: Date;
        setTime(d: date);
    }

    class Clock impletements ClockInterface {
        currentTime: Date;
        setTime(d: date) {
            this.currentTime = d
        }
        constructor(h: number, m: number) {}
    }
```

#### 扩展接口
像类一样，接口也可以彼此扩展对方：

```
    interface Shape {
        color: string
    }

    interface Square extends Shape {
        sideLength: number
    }

    var square = <Square>{};
    square.color = 'blue';
    square.sideLength = 10;
```

#### 组合类型
接口可以描述 JavaScript 中许多的类型，有时候你可能会遇到一个对象充当上述几种类型的组合。

```
    interface Counter {
        (start: number): string;
        interval: number;
        reset(): void;
    }

    var c: Counter;
    c(10);
    c.reset();
    c.interval = 5.0;
```

### Class
没有类以前，JavaScript 的继承主要通过原型来实现，ES6 给 JavaScript 添加了类以后，我们也可以采用基于类的面向对象方式来实现继承了。

```
    class Animal {
        private name: string;
        constructor(theName:string){
            this.name = theName;
        }
    }

    class Rhino extends Animal {
        constructor(){ super('Rhino')
    }
    
    var animal = new Animal('Goat');
    var rhino = new Rhino();
```

TypeScript 给类添加了 公开/私有（public/priviate）属性声明，默认所有的属性都是公开的。另外，`priviate` 和 `public` 关键字用在初始化函数参数的属性声明时，可以作为创建并初始化类成员的便捷方式。
```
    class Animal {
        constructor(private name: string) {}
    }
```
TypeScript 支持 getters/setters 拦截对象成员的存取。

```
    var passcode = "secret passcode";

    class Employee {
        private _fullName: string;
        get fullName(): string {
            return this._fullName;
        }
        set fullName(newName: string) {
            if (passcode && passcode == "secret passcode") {
                this._fullName = newName;
            } else{
                alert("Error: Unauthorized update of employee!");
            }
        }
    }

    var employee = new Employee();
    employee.fullName = "Bob Smith";
    if (employee.fullName) {
        alert(employee.fullName);
    }
```

TypeScript 支持静态属性声明，所有的静态属性只在类的内部可见，类的实例无法访问静态属性。在类的内部，通过在静态属性名前添加类名前缀来访问该属性(className.staticPropertity)。
另外，因为类创建类型，所以你也可以在某些地方用它们替代接口。

```
    class Point {
        x: number;
        y: number;
    }

    interface Point3d extends Point {
        z: number;
    }

    var point3d: Point3d = {x:1, y:2, z:3}
```

### 模块


### 函数

和 JavaScript 一样，函数可以被定义为命名函数和匿名函数。在 JavaScript 中，这两种命名方式如下：

``` 
    // named function
    function add(x, y){
        return x+y
    }
    // Anonymous function
    var myAdd = function(x, y){
        return x+y
    }
```

#### 函数类型
在 TypeScript 中，我们给上面的函数加上类型：

```
    function add(x:number, y:number): number {
        return x+y
    }

    var myAdd: (x:number, y:number)=>number  = function(x: number, y: number){ return x+y }
    // 上面的写法太冗余了，TypeScript 可以通过‘上下文类型‘推断出函数的类型：
    var myAdd = function(x:number, y:number): number { return x+y }
    var myAdd: (baseValue:number, increment:number)=>number = function(x,y){ return x+y }
```

#### 可选参数和默认参数
和 JavaScript 不一样的是，TypeScript 默认所有参数都是必须的。但是我们可以在参数后面添加 '?' 来表示该参数是可选的。

``` 
    function buildName(firstName: string, middleName?: string, lastName = "xuhong){
        if(middleName){ return firstName + " " + middleName + " " + lastName }
        else { return firstName + " " + lastName }
    }
```

#### 剩余参数
剩余参数是一个参数数组，让你可以一次传入任意数量的参数：

```
    //restOfName is an Array
    function buildName(firstName: string, ...restOfName: string[]){
        return firstName + " " + restOfName.join(" ")
    }

    var myname = buildName("chen","xuhong","codersir")
```

#### 重载
重载我们都很熟悉，就是根据传入的参数的不同进行不同的操作或返回不同的值。

```
    function pickCard(x: {suit: string; card: number; }[]): number;
    function pickCard(x: number): {suit: string; card: number; };
    function pickCard(x): any {
        if(typeof x == "object"){
            var pickedCard = Math.floor(Math.random() * x.length);
            return pickedCard;
        }else if (typeof x == "number"){
            var pickedSuit = Math.floor(x / 13);
            return { suit: suits[pickedSuit], card: x % 13 };
        }
    }
```

### 泛型
软件工程的一个重要部分就是开发不仅具有良好设计和一致的 API，而且还可以复用的组件。在 C# 和 Java 中，泛型就是实现可复用组件的重要工具，泛型使组件可以支持一系列的类型而不仅仅是某一种类型。
Angular2 的 Component 就是通过泛型实现复用。

```
    function identify<T>(arg: T): T {
        return arg;
    }
    var output = identity<string>("myString")
    var output = identity("myString")           //type 参数可以省略，编译器可以根据传入的参数推断类型
```

#### 接口泛型
接口泛型可以使所有实现该接口的对象元素保持同一种类型：

```
    interface GenericIdentityFn<T> {
        (arg: T): T;
    }

    function identity<T>(arg: T): T {
        return arg;
    }

    var myIdentity: GenericIdentityFn<number> = identity;
```

#### 类泛型
和接口泛型一样，类泛型可以确保所有的成员都是同一种类型。但是请注意，泛型只和类的实例部分相关，类的静态部分可以是其他的类型。

#### 约束泛型
有时候，我们不仅仅要求组件可以支持所有的类型，还会有其他的限制，比如要求至少有一个 length 属性：

```
    interface Lengthwise {
        length: number; 
    }

    function identity<T extends Lengthwise>(arg: T): T {
        console.log(arg.length)
        return;
    }
```

### 混合（mixin ）
TypeScript 的 mixin 并不够优雅。在 TypeScript 中，mixin 使用关键字 `implements` 而不是 `extends`，这意味着类被当成接口来使用，所以只会检查被混合类成员的类型而不是实现，也就是说我们必须在混合后的类中提供类成员的实现。然而这正是我们使用 mixin 想要避免的。

```
    class Disposable {
        isDisposed: boolean;
        dispose() {
            this.isDisposed = true;
        }
    }

    class Activatable {
        isActive: boolean;
        activate() {
            this.isActive = true;
        }
    }

    class smartObj implements Disposable, Activatable {
        constructor() {
            setInterval(() => console.log(this.isActive + ":" + this.isDisposed), 500);
        }

        interact() {
            this.activate();
        }
        
        //这里要再写一遍
        isDisposed: boolean = false;
        dispose: () => void;
        isActive: boolean = false;
        activate: () => void;
    }
```

### 声明合并
在 TypeScript 中，声明无非三种：命名空间/模块，类型，值。

#### 合并接口
同名接口合并其成员：
- 非函数成员必须唯一。
- 函数成员视为重载。

#### 合并模块
声明模块会同时创建命名空间和值：
- 命名空间的合并： 每个模块中  `export` 的类型定义自行合并，组成一个新的命名空间
- 值的合并：如果模块已经有同名的值，合并时会把第二个模块中 `export` 的值添加到第一个模块中


