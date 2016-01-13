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

sayHello 函数接受一个参数，该参数必须是个对象并且有一个名为 ‘name’ 的属性。虽然传入的对象还有其他的属性，但是编译器只会检查必须的属性以及它的类型。下面用接口重新写这个函数：

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

现在我们可以使用接口 ｀personName｀ 来描述参数的要求。注意我们不需要像其他的语言一样写明我们传进 ｀sayHello｀的对象实现了这个接口，只要这个对象满足接口列出的所有要求就够了。
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
