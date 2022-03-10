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

`nextTick` 中的回调是在下次 DOM 更新循环结束之后执行的延迟回调。在修改数据之后立即使用这个方法，获取更新后的 DOM。主要思路就是采用微任务优先的方式调用异步方法去执行 nextTick 包装的方法

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

## 8. MVC 和 MVVM

**MVC**

MVC 全名是 Model View Controller，是模型(model)－视图(view)－控制器(controller)的缩写。

- Model（模型）：是应用程序中用于处理应用程序数据逻辑的部分。通常模型对象负责在数据库中存取数据
- View（视图）：是应用程序中处理数据显示的部分。通常视图是依据模型数据创建的
- Controller（控制器）：是应用程序中处理用户交互的部分。通常控制器负责从视图读取数据，控制用户输入，并向模型发送数据


**MVVM**

MVVM 新增了 VM 类

- ViewModel 层：做了两件事达到了数据的双向绑定 一是将【模型】转化成【视图】，即将后端传递的数据转化成所看到的页面。实现的方式是：数据绑定。二是将【视图】转化成【模型】，即将所看到的页面转化成后端的数据。实现的方式是：DOM 事件监听。


MVVM 与 MVC 最大的区别就是：它实现了 View 和 Model 的自动同步，也就是当 Model 的属性改变时，我们不用再自己手动操作 Dom 元素，来改变 View 的显示，而是改变属性后该属性对应 View 层显示会自动改变（对应Vue数据驱动的思想）


整体看来，MVVM 比 MVC 精简很多，不仅简化了业务与界面的依赖，还解决了数据频繁更新的问题，不用再用选择器操作 DOM 元素。因为在 MVVM 中，View 不知道 Model 的存在，Model 和 ViewModel 也观察不到 View，这种低耦合模式提高代码的可重用性


> Vue 并没有完全遵循 MVVM 的思想.
> 严格的 MVVM 要求 View 不能和 Model 直接通信，而 Vue 提供了$refs 这个属性，让 Model 可以直接操作 View，违反了这一规定，所以说 Vue 没有完全遵循 MVVM。

## 9. 为什么 data 是一个函数

组件中的 data 写成一个函数，数据以函数返回值形式定义，这样每复用一次组件，就会返回一份新的 data，类似于给每个组件实例创建一个私有的数据空间，让各个组件实例维护各自的数据。而单纯的写成对象形式，就使得所有组件实例共用了一份 data，就会造成一个变了全都会变的结果。

## 10. vue的生命周期

- `beforeCreate` 在实例初始化之后，数据观测(`data observer`) 和 `event/watcher` 事件配置之前被调用。在当前阶段 `data`、`methods`、`computed` 以及 `watch` 上的数据和方法都不能被访问.

- `created` 实例已经创建完成之后被调用。在这一步，实例已完成以下的配置：数据观测(`data observer`)，属性和方法的运算， `watch/event` 事件回调。这里没有`$el`,如果非要想与 `Dom` 进行交互，可以通过 `vm.$nextTick` 来访问 `Dom`。

- `beforeMount` 在挂载开始之前被调用：相关的 `render` 函数首次被调用。

- `mounted` 在挂载完成后发生，在当前阶段，真实的 `Dom` 挂载完毕，数据完成双向绑定，可以访问到 `Dom` 节点

- `beforeUpdate` 数据更新时调用，发生在虚拟 `DOM` 重新渲染和打补丁（`patch`）之前。可以在这个钩子中进一步地更改状态，这不会触发附加的重渲染过程。

- `updated` 发生在更新完成之后，当前阶段组件 `Dom` 已完成更新。要注意的是避免在此期间更改数据，因为这可能会导致无限循环的更新，该钩子在服务器端渲染期间不被调用。
  
- `beforeDestroy` 实例销毁之前调用。在这一步，实例仍然完全可用。我们可以在这时进行善后收尾工作，比如清除计时器。

- `destroyed` `Vue` 实例销毁后调用。调用后，`Vue` 实例指示的所有东西都会解绑定，所有的事件监听器会被移除，所有的子实例也会被销毁。 该钩子在服务器端渲染期间不被调用。

- `activated` `keep-alive` 专属，组件被激活时调用
- `deactivated` `keep-alive` 专属，组件被销毁时调用


异步请求在哪步发起？

可以在钩子函数 `created`、`beforeMount`、`mounted` 中进行异步请求，因为在这三个钩子函数中，`data` 已经创建，可以将服务端端返回的数据进行赋值。

如果异步请求不需要依赖 `Dom` 推荐在 `created` 钩子函数中调用异步请求，因为在 `created` 钩子函数中调用异步请求有以下优点：
- 能更快获取到服务端数据，减少页面  `loading` 时间；
- `ssr` 不支持 `beforeMount` 、`mounted` 钩子函数，所以放在 `created` 中有助于一致性；


## 11. v-if 和 v-show 的区别

- `v-if` 在编译过程中会被转化成三元表达式,条件不满足时不渲染此节点。
- `v-show` 会被编译成指令，条件不满足时控制样式将对应节点隐藏 （`display:none`）

**使用场景**

- `v-if` 适用于在运行时很少改变条件，不需要频繁切换条件的场景
- `v-show` 适用于需要非常频繁切换条件的场景



## 12. vue事件修饰符

@click.xxx

- .stop 阻止事件继续传播
- .prevent 阻止标签默认行为
- .capture 使用事件捕获模式,即元素自身触发的事件先在此处处理，然后才交由内部元素进行处理
- .self 只当在 event.target 是当前元素自身时触发处理函数
- .once 事件将只会触发一次
- .passive 告诉浏览器你不想阻止事件的默认行为

## 13. v-model 的修饰符

v-model.xxx

- .lazy 通过这个修饰符，转变为在 change 事件再同步

- .number 自动将用户的输入值转化为数值类型

- .trim 自动过滤用户输入的首尾空格


## 14. 键盘事件的修饰符

@keyup.xxx 可以带多个修饰符

- .enter
- .tab
- .delete (捕获“删除”和“退格”键)
- .esc
- .space
- .up
- .down
- .left
- .right

## 15. 系统修饰键

@click.xxx 按住xxx再点击

- .ctrl
- .alt
- .shift
- .meta

@click.xxx 点击鼠标xx键即可触发事件
鼠标按钮修饰符

- .left
- .right
- .middle


## 16. vue内置指令

- `v-once` 定义它的元素或者组件只渲染一次，包括元素或者组件的所有子节点，首次渲染后，不会随数据的变化重新渲染，被视为静态内容。
- `v-cloak` 解决初始化慢导致页面闪动的问题，
  ```html
  <div>{{message}}<div>

  如果在网络慢的情况下，vue.js文件未加载，页面上就会显示 {{message}} 文本
  
  <div v-cloak>{{message}}</div>
  <style>
    [v-cloak] {
      display: none;
    }
  </style>
  ```
- `v-bind` 绑定属性，动态更新元素上的属性
- `v-on` 用于监听DOM事件，例如`click`
- `v-html` 传入解析`html`（需要注意防止xss攻击）
- `v-text` 更新元素的textContent
- `v-model` 变成value和input的语法糖，并且会处理拼音输入法的问题
- `v-if/v-else/v-else-if` 是否渲染元素
- `v-show` 是否展示元素（通过控制display值）
- `v-for` 循环指令，优先级比`v-if`高，需要注意增加唯一key值。
- `v-pre` 跳过这个元素以及子元素的编译过程，加快项目编译速度

## 17. 怎样理解 Vue 的单向数据流

数据总是从父组件传到子组件，子组件没有权利修改父组件传过来的数据，只能请求父组件对原始数据进行修改。这样会防止从子组件意外改变父级组件的状态，从而导致你的应用的数据流向难以理解。

如果实在要改变父组件的 prop 值 可以再 data 里面定义一个变量 并用 prop 的值初始化它 之后用$emit 通知父组件去修改


## 18. computed 和 watch 的区别和运用的场景

`computed` 是计算属性，依赖其他属性计算值，并且 `computed` 的值有缓存，只有当计算值变化才会返回内容，它可以设置 `getter` 和 `setter`。

```js
<p>{{addOne}}</p>

computed: {
  addOne() {
    return this.val + 1;
  }
}

当this.val变动之后addOne才会更新
```

设置`getter`和`setter`
```js
computed: {
  addOne: {
    get() {
      return this.val + 1;
    },
    set(newValue) {
      console.log('computed setter');
      this.val = newValue;
      return this.val;
    }
  }
}
```


`watch` 监听到值的变化就会执行回调，在回调中可以进行一些逻辑操作。

计算属性（`computed`）一般用在模板渲染中，某个值是依赖了其它的响应式对象甚至是计算属性计算而来；而侦听属性（`watch`）适用于观测某个值的变化去完成一段复杂的业务逻辑。

## 19. v-if 与 v-for 为什么不建议一起使用

v-for 和 v-if 不要在同一个标签中使用,因为解析时先解析 v-for 再解析 v-if。这样会带来性能方面的浪费（每次渲染都会先循环再进行条件判断）

如果遇到需要同时使用时可以考虑写成计算属性的方式。
```js
computed: {
  items: function() {
   return this.list.filter(function (item) {
    return item.isShow
   })
  }
}
```

## 20. vue如何检测数组变化

数组考虑性能原因没有用 `defineProperty` 对数组的每一项进行拦截，而是选择对 7 种数组（`push,shift,pop,splice,unshift,sort,reverse`）方法进行重写。

所以在 `Vue` 中修改数组的索引和长度是无法监控到的。需要通过以上 `7` 种变异方法修改数组才会触发数组对应的 `watcher` 进行更新。

## 21. vue3.0 了解多少

- 响应式原理的改变。`Vue3.x` 使用 `Proxy` 取代 `Vue2.x` 版本的 `Object.defineProperty`

- 组件选项声明方式。`Vue3.x` 使用 `Composition API`，`setup` 是 `Vue3.x` 新增的一个选项， 他是组件内使用 `Composition API` 的入口。

- 模板语法变化，`slot` 具名插槽语法，自定义指令，`v-model` 升级

- 其它方面的更改。`Suspense` 支持 `Fragment`（多个根节点）和 `Protal`（在 `dom` 其他部分渲染组建内容）组件，针对一些特殊的场景做了处理。基于 `treeshaking` 优化，提供了更多的内置功能。

[Vue3.0 新特性以及使用经验总结](https://juejin.cn/post/6940454764421316644)

## 22. Vue.3.x和 2.x 的响应式原理区别

`Vue3.x` 改用 `Proxy` 替代 `Object.defineProperty`。因为 `Proxy` 可以直接监听对象和数组的变化，并且有多达 13 种拦截方法。


## 23. Vue的父子组件生命周期钩子函数执行顺序

- 加载渲染过程

父 `beforeCreate`->父 `created` ->父 `beforeMount`->子 `beforeCreate`->子 `created`->子 `beforeMount`->子 `mounted`->父 `mounted`

- 子组件更新过程

父 `beforeUpdate` ->子 `beforeUpdate`->子 `updated`->父 `updated`

- 父组件更新过程

父 `beforeUpdate`->父 `updated`

- 销毁过程

父 `beforeDestroy`->子 `beforeDestroy`->子 `destroyed`->父 `destroyed`

## 25. 虚拟 DOM 是什么 有什么优缺点

由于在浏览器中操作 `DOM` 是很昂贵的。频繁的操作 `DOM`，会产生一定的性能问题。这就是虚拟 `Dom` 的产生原因。Vue2 的 `Virtual DOM` 借鉴了开源库 snabbdom 的实现。`Virtual DOM` 本质就是用一个原生的 `JS` 对象去描述一个 `DOM` 节点，是对真实 `DOM` 的一层抽象。

**优点**

- 保证性能下限： 框架的虚拟 `DOM` 需要适配任何上层 `API` 可能产生的操作，它的一些 `DOM` 操作的实现必须是普适的，所以它的性能并不是最优的；但是比起粗暴的 `DOM` 操作性能要好很多，因此框架的虚拟 `DOM` 至少可以保证在你不需要手动优化的情况下，依然可以提供还不错的性能，即保证性能的下限；

- 无需手动操作 `DOM`： 我们不再需要手动去操作 `DOM`，只需要写好 `View-Model` 的代码逻辑，框架会根据虚拟 `DOM` 和 数据双向绑定，帮我们以可预期的方式更新视图，极大提高我们的开发效率；

- 跨平台： 虚拟 `DOM` 本质上是 `JavaScript` 对象,而 `DOM` 与平台强相关，相比之下虚拟 `DOM` 可以进行更方便地跨平台操作，例如服务器渲染、weex 开发等等。

**缺点:**

- 无法进行极致优化： 虽然虚拟 `DOM` + 合理的优化，足以应对绝大部分应用的性能需求，但在一些性能要求极高的应用中虚拟 `DOM` 无法进行针对性的极致优化。

- 首次渲染大量 `DOM` 时，由于多了一层虚拟 `DOM` 的计算，会比 `innerHTML` 插入慢。


## 26. v-model原理

`v-model` 只是语法糖而已

`v-model` 在内部为不同的输入元素使用不同的 `property` 并抛出不同的事件：

- `text` 和 `textarea` 元素使用 `value property` 和 `input` 事件；
- `checkbox` 和 `radio` 使用 `checked property` 和 `change` 事件；
- `select` 字段将 `value` 作为 `prop` 并将 `change` 作为事件。


普通标签
```html
<input v-model="sth" />  //这一行等于下一行
<input v-bind:value="sth" v-on:input="sth = $event.target.value" />
```

组件

```html
<currency-input v-model="price"></currentcy-input>
<!--上行代码是下行的语法糖
 <currency-input :value="price" @input="price = arguments[0]"></currency-input>
-->

<!-- 子组件定义 -->
Vue.component('currency-input', {
 template: `
  <span>
   <input
    ref="input"
    :value="value"
    @input="$emit('input', $event.target.value)"
   >
  </span>
 `,
 props: ['value'],
})
```

## 27. v-for 为什么要加 key

如果不使用 key，Vue 会使用一种最大限度减少动态元素并且尽可能的尝试就地修改/复用相同类型元素的算法。key 是为 Vue 中 vnode 的唯一标记，通过这个 key，我们的 diff 操作可以更准确、更快速

更准确：在 sameNode 函数中直接 a.key === b.key 进行对比。所以会更加准确。

更快速：利用 key 的唯一性生成 map 对象来获取对应节点，比遍历方式更快

```js
// 判断两个vnode的标签和key是否相同 如果相同 就可以认为是同一节点就地复用
function isSameVnode(oldVnode, newVnode) {
  return oldVnode.tag === newVnode.tag && oldVnode.key === newVnode.key;
}

// 根据key来创建老的儿子的index映射表  类似 {'a':0,'b':1} 代表key为'a'的节点在第一个位置 key为'b'的节点在第二个位置
function makeIndexByKey(children) {
  let map = {};
  children.forEach((item, index) => {
    map[item.key] = index;
  });
  return map;
}
// 生成的映射表
let map = makeIndexByKey(oldCh);
```

## 28. Vue 事件绑定原理

原生事件绑定是通过 addEventListener 绑定给真实元素的，组件事件绑定是通过 Vue 自定义的$on 实现的。如果要在组件上使用原生事件，需要加.native 修饰符，这样就相当于在父组件中把子组件当做普通 html 标签，然后加上原生事件。

$on、$emit 是基于发布订阅模式的，维护一个事件中心，on 的时候将事件按名称存在事件中心里，称之为订阅者，然后 emit 将对应的事件进行发布，去执行事件中心里的对应的监听器

## 29. vue-router 路由钩子函数是什么 执行顺序是什么

路由钩子的执行流程, 钩子函数种类有:全局守卫、路由守卫、组件守卫

**完整的导航解析流程:**

1. 导航被触发。
2. 在失活的组件里调用 beforeRouteLeave 守卫。
3. 调用全局的 beforeEach 守卫。
4. 在重用的组件里调用 beforeRouteUpdate 守卫 (2.2+)。
5. 在路由配置里调用 beforeEnter。
6. 解析异步路由组件。
7. 在被激活的组件里调用 beforeRouteEnter。
8. 调用全局的 beforeResolve 守卫 (2.5+)。
9. 导航被确认。
10. 调用全局的 afterEach 钩子。
11. 触发 DOM 更新。
12. 调用 beforeRouteEnter 守卫中传给 next 的回调函数，创建好的组件实例会作为回调函数的参数传入。

## 30. vue-router 动态路由是什么 有什么问题

我们经常需要把某种模式匹配到的所有路由，全都映射到同个组件。例如，我们有一个 User 组件，对于所有 ID 各不相同的用户，都要使用这个组件来渲染。那么，我们可以在 `vue-router` 的路由路径中使用“动态路径参数”(`dynamic segment`) 来达到这个效果：

```js
const User = {
  template: "<div>User</div>",
};

const router = new VueRouter({
  routes: [
    // 动态路径参数 以冒号开头
    { path: "/user/:id", component: User },
  ],
});
```

> 如果vue-router 组件复用导致路由参数失效怎么办？

解决方法：
1.通过 watch 监听路由参数再发请求
```js
watch: { //通过watch来监听路由变化

 "$route": function(){
 this.getData(this.$route.params.xxx);
 }
}
```

2.用 :key 来阻止“复用”
<router-view :key="$route.fullPath" />

## 31. vuex的理解
vuex 是专门为 vue 提供的全局状态管理系统，用于多个组件中数据共享、数据缓存等。（无法持久化、内部核心原理是通过创造一个全局实例 new Vue）

主要包括以下几个模块：

- State：定义了应用状态的数据结构，可以在这里设置默认的初始状态。
- Getter：允许组件从 Store 中获取数据，mapGetters 辅助函数仅仅是将 store 中的 getter 映射到局部计算属性。
- Mutation：是唯一更改 store 中状态的方法，且必须是同步函数。
- Action：用于提交 mutation，而不是直接变更状态，可以包含任意异步操作。
- Module：允许将单一的 Store 拆分为多个 store 且同时保存在单一的状态树中。

## 32. Vuex 页面刷新数据丢失怎么解决

需要做 `vuex` 数据持久化 一般使用本地存储的方案来保存数据 可以自己设计存储方案 也可以使用第三方插件

推荐使用 `vuex-persist` 插件，它就是为 `Vuex` 持久化存储而生的一个插件。不需要你手动存取 `storage` ，而是直接将状态保存至 `cookie` 或者 `localStorage` 中


## 33. Vuex 为什么要分模块并且加命名空间

模块:由于使用单一状态树，应用的所有状态会集中到一个比较大的对象。当应用变得非常复杂时，store 对象就有可能变得相当臃肿。为了解决以上问题，Vuex 允许我们将 store 分割成模块（module）。每个模块拥有自己的 state、mutation、action、getter、甚至是嵌套子模块。

命名空间：默认情况下，模块内部的 action、mutation 和 getter 是注册在全局命名空间的——这样使得多个模块能够对同一 mutation 或 action 作出响应。如果希望你的模块具有更高的封装度和复用性，你可以通过添加 namespaced: true 的方式使其成为带命名空间的模块。当模块被注册后，它的所有 getter、action 及 mutation 都会自动根据模块注册的路径调整命名。


## 34. 说说 SSR

SSR 也就是服务端渲染，也就是将 Vue 在客户端把标签渲染成 HTML 的工作放在服务端完成，然后再把 html 直接返回给客户端。

优点：
- SSR 有着更好的 SEO、并且首屏加载速度更快


缺点：
- 开发条件会受到限制，服务器端渲染只支持 beforeCreate 和 created 两个钩子，当我们需要一些外部扩展库时需要特殊处理，服务端渲染应用程序也需要处于 Node.js 的运行环境。

- 服务器会有更大的负载需求


## 35. vue 中使用了哪些设计模式

1. 工厂模式 - 传入参数即可创建实例
虚拟 DOM 根据参数的不同返回基础标签的 Vnode 和组件 Vnode
2. 单例模式 - 整个程序有且仅有一个实例
vuex 和 vue-router 的插件注册方法 install 判断如果系统存在实例就直接返回掉
3. 发布-订阅模式 (vue 事件机制)
4. 观察者模式 (响应式数据原理)
5. 装饰模式: (@装饰器的用法)
6. 策略模式 策略模式指对象有某个行为,但是在不同的场景中,该行为有不同的实现方案-比如选项的合并策略


## 36. 做过哪些 Vue 的性能优化

- 对象层级不要过深，否则性能就会差
- 不需要响应式的数据不要放到 data 中（可以用 Object.freeze() 冻结数据）
- v-if 和 v-show 区分使用场景
- computed 和 watch 区分使用场景
- v-for 遍历必须加 key，key 最好是 id 值，且避免同时使用 v-if
- 大数据列表和表格性能优化-虚拟列表/虚拟表格
- 防止内部泄漏，组件销毁后把全局变量和事件销毁
- 图片懒加载
- 路由懒加载
- 第三方插件的按需引入
- 适当采用 keep-alive 缓存组件
- 防抖、节流运用
- 服务端渲染 SSR or 预渲染

## 37. keep-alive 使用场景和原理

keep-alive 是 Vue 内置的一个组件，可以实现组件缓存，当组件切换时不会对当前组件进行卸载。

- 常用的两个属性 include/exclude，允许组件有条件的进行缓存。

- 两个生命周期 activated/deactivated，用来得知当前组件是否处于活跃状态。

- keep-alive 的中还运用了 LRU(最近最少使用) 算法，选择最近最久未使用的组件予以淘汰。

LRU 的核心思想是如果数据最近被访问过，那么将来被访问的几率也更高，所以我们将命中缓存的组件 key 重新插入到 this.keys 的尾部，这样一来，this.keys 中越往头部的数据即将来被访问几率越低，所以当缓存数量达到最大值时，我们就删除将来被访问几率最低的数据，即 this.keys 中第一个缓存的组件。
