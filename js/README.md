# js相关

## 1. apply、call、bind区别

- 三者都可以改变函数的 `this` 对象指向。
- 三者第一个参数都是 `this` 要指向的对象，如果没有这个参数或参数为 `undefined` 或 `null`，则默认指向全局 `window`。
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

## 3. 手写bind，call，apply

`call`
```js
Function.prototype.myCall(context) {
  context = context || window;
  context.fn = this;

  const args = [...arguments].slice(1);
  const result = context.fn(args);
  delete context.fn;
  return result;
}
```

`apply`

```js
Function.prototype.myApply(context, arr) {
  context = context || window;
  context.fn = this;

  let result;
  if(!arr) {
    result = context.fn();
  } else {
    result = context.fn(arr);
  }
  delete context.fn;
  return result;
}
```

`bind`

```js
Function.prototype.myBind(context) {
  if(typeof this !== 'function') {
    throw new Error('not a function');
  }

  const args = [...arguments].slice(1);
  
  let Transit = function(){};
  const _ = this;
  const FunctionToBind = function() {
    const bindArgs = [...arguments];
    return _.apply(this instanceOf Transit ? this : context, args.concat(bindArgs));
  }

  Transit.prototype = this.prototype;
  FunctionToBind.prototype = new Transit();
  return FunctionToBind;
}
```
## 4. 手写promise.all和race

## 5. 实现一个寄生组合继承

这道题也可以这样问ES6的class继承用ES5如何实现

>先来回顾一下构造函数、原型对象和实例之间的关系
>```js
> function F() {}
> let f = new F(){};
> F.prototype.constructor === F;
> F.__proto__ === Function.prototype;
> Function.prototype.__proto__ ==== Object.prototype;
> Object.prototype.__proto__ === null;
>
>f.__proto__ === F.prototype;
>F.prototype.__proto__ === Object.prototype;
>```

其实`ES6 extends` 继承，主要就是
- 把子类构造函数(`Child`)的原型(`__proto__`)指向了父类构造函数(`Parent`)，继承父类的静态方法
- 把子类实例`child`的原型对象(`Child.prototype`) 的原型(`__proto__`)指向了父类`parent`的原型对象(`Parent.prototype`),继承父类的方法。
- 子类构造器里调用父类构造器，继承父类的属性(在子构造函数中调用`call`方法)

```js
function _inherits(Child, Parent) {
  Child.prototype = Object.create(Parent.prototype);
  Child.prototype.constructor = Child;
  Child.__proto__ = Parent;
}

// 设置父类
   function SuperType(name) {
       this.name = name;
       this.colors = ["red", "blue", "green"];
       SuperType.prototype.sayName = function () {
         console.log(this.name)
       }
   }
   // 设置子类
   function SubType(name, age) {
       //构造函数式继承--子类构造函数中执行父类构造函数
       SuperType.call(this, name);
       this.age = age;
   }
   // 核心：因为是对父类原型的复制，所以不包含父类的构造函数，也就不会调用两次父类的构造函数造成浪费
   _inherits(SubType, SuperType)
   // 添加子类私有方法
   SubType.prototype.sayAge = function () {
      console.log(this.age);
   }
   var instance = new SubType("leo",24)
   console.dir(instance)
```


## 6. 实现new

>`new`的作用：
>- 创建一个新对象
>- 将`this`指向创建的新对象
>- 创建的新对象会被链接到该函数的`prototype`对象上（新对象的`__proto__`属性指向函数的`prototype`）;
>- 利用函数的`call`方法，将原本指向`window`的绑定对象`this`指向了`obj`。（这样一来，当我们向函数中再传递实参时，对象的属性就会被挂载到`obj`上。）

```js
function createObject() {
  const obj = {};
  const Constructor = [].shift.call(arguments);
  obj.__proto__ = Constructor.prototype;
  const result = Constructor.apply(obj, arguments);
  return typeof result === 'obj' ? result : obj;
}

```


## 7. setTimeout 模拟实现 setInterval

```js
function setIntervalSim(fn, delay) {
  setTimeout(function() {
    fn();
    setTimeout(arguments.callee, delay)
  }, delay)
}
```

带关闭的interval:
```js
function setIntervalSim(fn, delay) {
  let timer = null;
  function start() {
    fn();
    timer = setTimeout(arguments.callee, delay);
  }
  setTimeout(start, delay);
  setIntervalSim.cancel = () => {
    clearTimeout(timer);
  }
}
```

> `setInterval`缺点 
> - 使用`setInterval`时，某些间隔会被跳过；
> - 可能多个定时器会连续执行； 
> 
> 定时器指定的时间间隔，表示的是何时将定时器的代码添加到消息队列，而不是何时执行代码。所以真正何时执行代码的时间是不能保证的，取决于何时被主线程的事件循环取到，并执行。
> 
> 例如： `setInterval`每隔100ms往队列中添加一个事件；100ms后，添加T1定时器代码至队列中，主线程中还有任务在执行，所以等待，some event执行结束后执行T1定时器代码；又过了100ms，T2定时器被添加到队列中，主线程还在执行T1代码，所以等待；又过了100ms，理论上又要往队列里推一个定时器代码，但由于此时T2还在队列中，所以T3不会被添加，结果就是此时被跳过；这里我们可以看到，T1定时器执行结束后马上执行了T2代码，所以并没有达到定时器的效果。
>
> 采用`setTimeout`模拟实现。每次执行的时候都会创建一个新的定时器，第二个`setTimeout`使用了`arguments.callee`获取当前函数的引用，并且为其设置另一个定时器。这样就保证了
> - 在前一个定时器执行完前，不会向队列插入新的定时器（解决缺点一）
> - 保证定时器间隔（解决缺点二）


## 8. 手写发布订阅模式

## 9. 手写防抖节流，哪些场景会用到

防抖函数原理：在事件被触发n秒后再执行回调，如果在这n秒内又被触发，则重新计时。
debounce:
```js
function debounce(fn, delay) {
  let timer;
  return function(...args) {
    if(timer) {
      clearTimeout(timer)
    }
    timer = setTimeout(() => {
      fn.apply(this, args)
    }, delay)
  }
}
```

节流函数原理:规定在一个单位时间内，只能触发一次函数。如果这个单位时间内触发多次函数，只有一次生效
throttle:
```js
function throttle(fn, delay) {
  let isEmit = false;
  return (...args) => {
    if(isEmit) return;

    isEmit = true;
    setTimeout(() => {
      fn.apply(this, args);
      isEmit = false;
    }, delay)
  }
}
```


## 10. 将虚拟 Dom 转化为真实 Dom
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

## 11. 实现一个对象的 flatten 方法

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


## 12. 原型链判断

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

## 13. RAF 和 RIC 是什么

`requestAnimationFrame`： 告诉浏览器在下次重绘之前执行传入的回调函数(通常是操纵 dom，更新动画的函数)；由于是每帧执行一次，那结果就是每秒的执行次数与浏览器屏幕刷新次数一样，通常是每秒 60 次。

`requestIdleCallback`: 会在浏览器空闲时间执行回调，也就是允许开发人员在主事件循环中执行低优先级任务，而不影响一些延迟关键事件。如果有多个回调，会按照先进先出原则执行，但是当传入了 timeout，为了避免超时，有可能会打乱这个顺序。

[requestIdleCallback和requestAnimationFrame详解](https://juejin.cn/post/6844903848981577735)

## 14. Es6 的 let 实现原理

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
// babel 编译之后的 es5 代码（polyfill）
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
其实我们根据 `babel` 编译之后的结果可以看得出来 let 是**借助闭包和函数作用域来实现块级作用域的效果**的 在不同的情况下 let 的编译结果是不一样的

## 15. `require` 具体实现原理是什么

[深入Node.js的模块加载机制，手写require函数](https://juejin.cn/post/6866973719634542606)


## 16. `this`指向问题

函数的`this`在调用时绑定的，完全取决于函数的调用位置。

1. `new` 调用：绑定到新创建的对象，注意：显示`return`函数或对象，返回值不是新创建的对象，而是显式返回的函数或对象。
2. `call` 或者 `apply`（ 或者 `bind`） 调用：严格模式下，绑定到指定的第一个参数。非严格模式下，`null`和`undefined`，指向全局对象（浏览器中是`window`），其余值指向被`new Object()`包装的对象。
3. 对象上的函数调用：绑定到那个对象。
4. 普通函数调用： 在严格模式下绑定到 undefined，否则绑定到全局对象


示例：
```js
function Foo() {
    getName = function () { alert (1); };
    return this;
}
Foo.getName = function () { alert (2);};
Foo.prototype.getName = function () { alert (3);};
var getName = function () { alert (4);};
function getName() { alert (5);}

//请写出以下输出结果：
Foo.getName(); // 2
getName(); // 4
Foo().getName(); // 1
getName(); // 1
new Foo.getName(); // 2
new Foo().getName(); // 3
new new Foo().getName(); // 3
```

## 17. ES6的语法用过哪些

## 18. 用过symbol吗？有哪些用处
避免命名重复、实现私有变量、在call和apply中就有用到

## 19. 讲打印结果

```js
setTimeout(function() {
    console.log('setTimeout1');
    new Promise(function(resolve) {
        console.log('promise0');
        resolve()
    }).then(function() {
        console.log('settimeout promise resolveed');
    })
});
setTimeout(function() {
    console.log('setTimeout2');
});
const P = new Promise(function(resolve) {
    console.log('promise');
    for (var i = 0; i < 10000; i++) {
        if(i === 10) {
            console.log('for');
        }
        if (i === 9999) {
            resolve('resolve');
        }
    }
}).then(function(val) {
    console.log('resolve1');
}).then(function(val) {
    console.log('resolve2');
});
new Promise(function(resolve) {
    console.log('promise2');
    resolve('resolve');
}).then(function(val) {
    console.log('resolve3');
})
console.log('console');
```

结果:
promise -> for -> promise2 -> console -> resolve1 -> resolve3 -> resolve2 -> setTimeout1 -> promise0 -> setTimeout promise resolved -> setTimeout2


## 20. 实现一个深拷贝你会考虑到哪些点

## 21. 为什么js中0.1+0.2不等于0.3，怎样处理使之相等？

原因在于在JS中采用的IEEE 754的双精度标准.计算机内部存储数据的编码的时候，0.1在计算机内部根本就不是精确的0.1，而是一个有舍入误差的0.1。当代码被编译或解释后，0.1已经被四舍五入成一个与之很接近的计算机内部数字，以至于计算还没开始，一个很小的舍入错误就已经产生了。这也就是 0.1 + 0.2 不等于0.3 的原因。

那为什么 0.1 + 0.1 就等于 0.2

两个有舍入误差的值在求和时，相互抵消了，但这种“负负得正，相互抵消”不一定是可靠的，当这两个数字是用不同长度数位来表示的浮点数时，舍入误差可能不会相互抵消。

又如，对于 0.1 + 0.3 ，结果其实并不是0.4，但0.4是最接近真实结果的数，比其它任何浮点数都更接近。许多语言也就直接显示结果为0.4了，而不展示一个浮点数的真实结果了。


## 22.前端性能优化之js

- 节流、防抖
- 长列表滚动到可视区域动态加载（大数据渲染）
- 图片懒加载（data-src）
- 使用闭包时，在函数结尾手动删除不需要的局部变量，尤其在缓存dom节点的情况下
- DOM 操作优化
  - 批量添加dom可先createElement创建并添加节点，最后一次性加入dom
  - 批量绑定事件，使用事件委托绑定父节点实现，利用了事件冒泡的特性
  - 如果可以使用innerHTML代替appendChild
  - 在 DOM 操作时添加样式时尽量增加 class 属性，而不是通过 style 操作样式，以减少重排（Reflow）


## 23. 判断输入输出

```js
function T1() {
 this.name = 't1';
 this.age = 19;
}

function T2() {
 this.name = 't2';
 this.age = 19;
 return 19;  
}

function T3() {
 this.name = 't3';
 this.age = 19;
 return {
   name: 't',
   age: 20
 };  
}

function T4() {
 this.name = 't4';
 this.age = 19;
}

console.log(new T1()); // {name:'t1',age:19}
console.log(new T2()); // {name:'t1',age:19}
console.log(new T3()); // {name:'t',age: 20}

T4.prototype = new T1();
T4.prototype.type = 'expert';

const t4 = new T4();
console.log(t4); //  {name:'t4',age:19}
console.log(t4.type); // expert
console.log(t4 instanceof T1); // true
console.log(t4 instanceof T2); // false
console.log(t4 instanceof T4); // true
```