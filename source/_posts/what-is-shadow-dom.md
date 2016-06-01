title: 什么是 ShadowDOM
date: 2016-02-26 11:13:16
tags:
- webcomponent
- shadowdom

---

Shadow DOM 是 webcomponent 的一部分，通过向 `document` 添加一种新的节点来解决 webcomponent DOM 树的封装问题。

## 内容和展现分离

通过 Shadow DOM，可以给元素添加一种与之相关联的新节点，也就是 `shadow root`，被关联的元素我们称它 `shadow host`。

```html
<button>Hello, world!</button>
<script>
  var host = document.querySelector('button');
  var root = host.createShadowRoot();
  root.textContent = '影的世界!';
</script>
```

<pre><button>Hello, world!</button>
<script>
var host = document.querySelector('button');
var root = host.createShadowRoot();
root.textContent = '影的世界!';
</script></pre>

## 封装样式

由于 shadow dom 的存在，产生了一种新类型的节点与元素相关，那就是 shadow root， 拥有 shadow root 的这个元素则成为 shadow host。
shadow host 里的内容不会被渲染输出，取而代之的是 shadow root 里的内容。


style shadow dom

:host style

:host(selector)

:host-context(selector)

::shadow

/deep/

::content


http://webcomponents.org/
