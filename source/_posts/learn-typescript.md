title: learn-typescript
date: 2016-01-12 12:58:51
tags: typescript, angular2
---

TypeScript 是微软出的一个 JavaScript 超集，给 JavaScript 添加了类型，可以编译成 plain JavaScript。最近开始学习Angular2，Angular2 支持 TypeScript 来编写，也是社区推荐的方式。

### 基本类型
TypeScript 为你添加了你期望 JavaScript 拥有的基本数据类型：numbers、strings、structures、boolean。

```
    var isDone: boolean = false;    //boolean
    var height: number = 6;         //number
    var name:string = 'xuhong';     //string
    var list:number[] = [1,2,3];    //array
    
    enum Color {Red, Green, Blue}
    var c: Color = Color.Red;       //enum

    var notSure:any = 4;            //any
    function warning:void{          //no return value function typed void
        console.warning('warning message')
    }
```

### Interfaces
