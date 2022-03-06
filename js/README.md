# js相关

## 1. apply、call、bind区别

- 三者都可以改变函数的 `this` 对象指向。
- 三者第一个参数都是 `this` 要指向的对象，如果如果没有这个参数或参数为 `undefined` 或 `null`，则默认指向全局 `window`。
- 三者都可以传参，但是 `apply` 是数组，而 `call` 是参数列表，且 `apply` 和 `call` 是一次性传入参数，而 `bind` 可以分为多次传入。
- `bind` 是返回绑定 `this` 之后的函数，便于稍后调用；`apply` 、`call` 则是立即执行 。
- `bind()`会返回一个新的函数，如果这个返回的新的函数作为构造函数创建一个新的对象，那么此时 `this` 不再指向传入给 `bind` 的第一个参数，而是指向用 `new` 创建的实例。

## 2. 举出闭包实际场景运用的例子

1. 比如常见的防抖节流
```js
function debounce(fn, delay = 300) {
  let timer;
  return function() {
    const args = arguments;
    if(timer) {
      clearTimeout(timer);
    }
    timer = setTimeout(() => {
      fn.apply(this, args)
    }, delay);
  }
}
```

2. 使用闭包可以在 `JavaScript` 中模拟块级作用域

```js
function outputNumbers(count) {
  (function () {
    for (var i = 0; i < count; i++) {
      alert(i);
    }
  })();
  alert(i); //导致一个错误！
}
```

3. 闭包可以用于在对象中创建私有变量

```js
var obj = (function () {
  var a = 1;
  function getA() {
    return a++;
  }
  return {
    b: getA,
  };
})();
console.log(obj.a); //undefined
obj.b(); // 1
```

## 2. 手写bind

## 3. 手写promise.all和race

## 4. 实现一个寄生组合继承

## 5. 实现new

## 6. setTimeout 模拟实现 setInterval

## 7. 手写发布订阅模式

## 8. 手写防抖节流

## 9. 将虚拟 Dom 转化为真实 Dom
```js
{
  tag: 'DIV',
  attrs:{
  id:'app'
  },
  children: [
    {
      tag: 'SPAN',
      children: [
        { tag: 'A', children: [] }
      ]
    },
    {
      tag: 'SPAN',
      children: [
        { tag: 'A', children: [] },
        { tag: 'A', children: [] }
      ]
    }
  ]
}
把上诉虚拟Dom转化成下方真实Dom
<div id="app">
  <span>
    <a></a>
  </span>
  <span>
    <a></a>
    <a></a>
  </span>
</div>
```

```js
// 真正的渲染函数
function _render(vnode) {
  // 如果是数字类型转化为字符串
  if (typeof vnode === "number") {
    vnode = String(vnode);
  }
  // 字符串类型直接就是文本节点
  if (typeof vnode === "string") {
    return document.createTextNode(vnode);
  }
  // 普通DOM
  const dom = document.createElement(vnode.tag);
  if (vnode.attrs) {
    // 遍历属性
    Object.keys(vnode.attrs).forEach((key) => {
      const value = vnode.attrs[key];
      dom.setAttribute(key, value);
    });
  }
  // 子数组进行递归操作 这一步是关键
  vnode.children.forEach((child) => dom.appendChild(_render(child)));
  return dom;
}
```

## 10. 实现一个对象的 flatten 方法

```js
const obj = {
 a: {
        b: 1,
        c: 2,
        d: {e: 5}
    },
 b: [1, 3, {a: 2, b: 3}],
 c: 3
}

flatten(obj) 结果返回如下
// {
//  'a.b': 1,
//  'a.c': 2,
//  'a.d.e': 5,
//  'b[0]': 1,
//  'b[1]': 3,
//  'b[2].a': 2,
//  'b[2].b': 3
//   c: 3
// }


```

```js
function isObject(val) {
  return typeof val === "object" && val !== null;
}

function flatten(obj) {
  if (!isObject(obj)) {
    return;
  }
  let res = {};
  const dfs = (cur, prefix) => {
    if (isObject(cur)) {
      if (Array.isArray(cur)) {
        cur.forEach((item, index) => {
          dfs(item, `${prefix}[${index}]`);
        });
      } else {
        for (let k in cur) {
          dfs(cur[k], `${prefix}${prefix ? "." : ""}${k}`);
        }
      }
    } else {
      res[prefix] = cur;
    }
  };
  dfs(obj, "");

  return res;
}
flatten();

```


## 11. 原型链判断

```js
Object.prototype.__proto__;
Function.prototype.__proto__;
Object.__proto__;
Object instanceof Function;
Function instanceof Object;
Function.prototype === Function.__proto__;
```

```js
Object.prototype.__proto__; //null
Function.prototype.__proto__; //Object.prototype
Object.__proto__; //Function.prototype
Object instanceof Function; //true
Function instanceof Object; //true
Function.prototype === Function.__proto__; //true
```

[JavaScript原型系列](https://juejin.cn/post/6844903937418461198)

## 12. RAF 和 RIC 是什么

requestAnimationFrame： 告诉浏览器在下次重绘之前执行传入的回调函数(通常是操纵 dom，更新动画的函数)；由于是每帧执行一次，那结果就是每秒的执行次数与浏览器屏幕刷新次数一样，通常是每秒 60 次。

requestIdleCallback：: 会在浏览器空闲时间执行回调，也就是允许开发人员在主事件循环中执行低优先级任务，而不影响一些延迟关键事件。如果有多个回调，会按照先进先出原则执行，但是当传入了 timeout，为了避免超时，有可能会打乱这个顺序。

[requestIdleCallback和requestAnimationFrame详解](https://juejin.cn/post/6844903848981577735)

## 13. Es6 的 let 实现原理

原始 es6 代码
```js
var funcs = [];
for (let i = 0; i < 10; i++) {
  funcs[i] = function () {
    console.log(i);
  };
}
funcs[0](); // 0
```
```js
babel 编译之后的 es5 代码（polyfill）
var funcs = [];

var _loop = function _loop(i) {
  funcs[i] = function () {
    console.log(i);
  };
};

for (var i = 0; i < 10; i++) {
  _loop(i);
}
funcs[0](); // 0
```
其实我们根据 babel 编译之后的结果可以看得出来 let 是借助闭包和函数作用域来实现块级作用域的效果的 在不同的情况下 let 的编译结果是不一样的

## 14. `require` 具体实现原理是什么

[深入Node.js的模块加载机制，手写require函数](https://juejin.cn/post/6866973719634542606)