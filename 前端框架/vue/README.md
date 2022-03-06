# vue相关实现

## 1. vue 组件通讯方式有哪些方法

1. props 和$emit 父组件向子组件传递数据是通过 prop 传递的，子组件传递数据给父组件是通过$emit 触发事件来做到的

2. $parent,$children 获取当前组件的父组件和当前组件的子组件

3. $attrs 和$listeners A->B->C。Vue 2.4 开始提供了$attrs 和$listeners 来解决这个问题

4. 父组件中通过 provide 来提供变量，然后在子组件中通过 inject 来注入变量。(官方不推荐在实际业务中使用，但是写组件库时很常用)

5. $refs 获取组件实例

6. envetBus 兄弟组件数据传递 这种情况下可以使用事件总线的方式

7. vuex 状态管理



##  2. vue响应式原理

整体思路是数据劫持+观察者模式

对象内部通过 defineReactive 方法，使用 Object.defineProperty 将属性进行劫持（只会劫持已经存在的属性），数组则是通过重写数组方法来实现。当页面使用对应属性时，每个属性都拥有自己的 dep 属性，存放他所依赖的 watcher（依赖收集），当属性变化后会通知自己对应的 watcher 去更新(派发更新)。

```js
class Observer {
  // 观测值
  constructor(value) {
    this.walk(value);
  }
  walk(data) {
    // 对象上的所有属性依次进行观测
    let keys = Object.keys(data);
    for (let i = 0; i < keys.length; i++) {
      let key = keys[i];
      let value = data[key];
      defineReactive(data, key, value);
    }
  }
}
// Object.defineProperty数据劫持核心 兼容性在ie9以及以上
function defineReactive(data, key, value) {
  observe(value); // 递归关键
  // --如果value还是一个对象会继续走一遍odefineReactive 层层遍历一直到value不是对象才停止
  //   思考？如果Vue数据嵌套层级过深 >>性能会受影响
  Object.defineProperty(data, key, {
    get() {
      console.log("获取值");

      //需要做依赖收集过程 这里代码没写出来
      return value;
    },
    set(newValue) {
      if (newValue === value) return;
      console.log("设置值");
      //需要做派发更新过程 这里代码没写出来
      value = newValue;
    },
  });
}
export function observe(value) {
  // 如果传过来的是对象或者数组 进行属性劫持
  if (
    Object.prototype.toString.call(value) === "[object Object]" ||
    Array.isArray(value)
  ) {
    return new Observer(value);
  }
}
```

[响应式原理](https://juejin.cn/post/6935344605424517128)

## 3. vue nextTick原理

nextTick 中的回调是在下次 DOM 更新循环结束之后执行的延迟回调。在修改数据之后立即使用这个方法，获取更新后的 DOM。主要思路就是采用微任务优先的方式调用异步方法去执行 nextTick 包装的方法

```js
// src/util/next-tick.js

let callbacks = [];
let pending = false;
function flushCallbacks() {
  pending = false; //把标志还原为false
  // 依次执行回调
  for (let i = 0; i < callbacks.length; i++) {
    callbacks[i]();
  }
}
let timerFunc; //定义异步方法  采用优雅降级
if (typeof Promise !== "undefined") {
  // 如果支持promise
  const p = Promise.resolve();
  timerFunc = () => {
    p.then(flushCallbacks);
  };
} else if (typeof MutationObserver !== "undefined") {
  // MutationObserver 主要是监听dom变化 也是一个异步方法
  let counter = 1;
  const observer = new MutationObserver(flushCallbacks);
  const textNode = document.createTextNode(String(counter));
  observer.observe(textNode, {
    characterData: true,
  });
  timerFunc = () => {
    counter = (counter + 1) % 2;
    textNode.data = String(counter);
  };
} else if (typeof setImmediate !== "undefined") {
  // 如果前面都不支持 判断setImmediate
  timerFunc = () => {
    setImmediate(flushCallbacks);
  };
} else {
  // 最后降级采用setTimeout
  timerFunc = () => {
    setTimeout(flushCallbacks, 0);
  };
}

export function nextTick(cb) {
  // 除了渲染watcher  还有用户自己手动调用的nextTick 一起被收集到数组
  callbacks.push(cb);
  if (!pending) {
    // 如果多次调用nextTick  只会执行一次异步 等异步队列清空之后再把标志变为false
    pending = true;
    timerFunc();
  }
}

```

[](https://juejin.cn/post/6939704519668432910#heading-4)

## 4. Vue diff 原理

[diff算法原理](https://juejin.cn/post/6953433215218483236)

## 5. 路由原理 history 和 hash 两种路由方式的特点

- hash 模式

`location.hash` 的值实际就是 URL 中#后面的东西 它的特点在于：`hash` 虽然出现 `URL` 中，但不会被包含在 `HTTP` 请求中，对后端完全没有影响，因此改变 `hash` 不会重新加载页面。

可以为 `hash` 的改变添加监听事件

```js
window.addEventListener("hashchange", funcRef, false);
```

每一次改变 `hash（window.location.hash）`，都会在浏览器的访问历史中增加一个记录。利用 `hash` 的以上特点，就可以来实现前端路由“更新视图但不重新请求页面”的功能了。

特点：兼容性好但是不美观


- history 模式

利用了 HTML5 History Interface 中新增的 `pushState()` 和 `replaceState()` 方法。

这两个方法应用于浏览器的历史记录站，在当前已有的 `back、forward、go` 的基础之上，它们提供了对历史记录进行修改的功能。这两个方法有个共同的特点：当调用他们修改浏览器历史记录栈后，虽然当前 `URL` 改变了，但浏览器不会刷新页面，这就为单页应用前端路由“更新视图但不重新请求页面”提供了基础。

特点：虽然美观，但是刷新会出现 404 需要后端进行配置


## 6. 手写 Vue.extend

```js
//  src/global-api/initExtend.js
import { mergeOptions } from "../util/index";
export default function initExtend(Vue) {
  let cid = 0; //组件的唯一标识
  // 创建子类继承Vue父类 便于属性扩展
  Vue.extend = function (extendOptions) {
    // 创建子类的构造函数 并且调用初始化方法
    const Sub = function VueComponent(options) {
      this._init(options); //调用Vue初始化方法
    };
    Sub.cid = cid++;
    Sub.prototype = Object.create(this.prototype); // 子类原型指向父类
    Sub.prototype.constructor = Sub; //constructor指向自己
    Sub.options = mergeOptions(this.options, extendOptions); //合并自己的options和父类的options
    return Sub;
  };
}
```

[组件原理](https://juejin.cn/post/6954173708344770591)

## 7. vue-router 中路由方法 pushState 和 replaceState 能否触发 popSate 事件

答案是：不能

pushState 和 replaceState

HTML5 新接口，可以改变网址(存在跨域限制)而不刷新页面，这个强大的特性后来用到了单页面应用如：vue-router，react-router-dom 中。

注意:仅改变网址,网页不会真的跳转,也不会获取到新的内容,本质上网页还停留在原页面
window.history.pushState(state, title, targetURL);
@状态对象：传给目标路由的信息,可为空
@页面标题：目前所有浏览器都不支持,填空字符串即可
@可选url：目标url，不会检查url是否存在，且不能跨域。如不传该项,即给当前url添加data

window.history.replaceState(state, title, targetURL);
@类似于pushState,但是会直接替换掉当前url,而不会在history中留下记录

popstate 事件会在点击后退、前进按钮(或调用 history.back()、history.forward()、history.go()方法)时触发

注意:用 history.pushState()或者 history.replaceState()不会触发 popstate 事件


[最全的 Vue 面试题](https://juejin.cn/post/6961222829979697165#heading-44)