title: 优化你的代码风格
date: 2016-06-01 16:11:02
tags:
- 
---

AirBnb

## 各种 Linter

### JSLint

### ESLint

ES6 已经是新的标准了，

### TSLint

如果你也使用 TypeScript，那么推荐使用 TSLint 来维护你的代码风格。

### 分号之争

另外，绝大部分情况下，完全不需要写分号，通常 \n 就宣告了语句的结束。只需要在以下字符开头的代码前加上分号就足够了：

- [
- (
- +
- *
- /
- -
- ,
- .

```
foo();
[1,2,3].forEach(bar);

// =>

foo()
;[1,2,3].forEach(bar)

```

## 参考资料
[](http://blog.izs.me/post/2353458699/an-open-letter-to-javascript-leaders-regarding)