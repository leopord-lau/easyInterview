# 浏览器相关

## 1. 从输入一个 URL 地址到浏览器完成渲染的整个过程


1. 浏览器地址栏输入 URL 并回车
2. 浏览器查找当前 URL 是否存在缓存，并比较缓存是否过期
3. DNS 解析 URL 对应的 IP
4. 根据 IP 建立 TCP 连接（三次握手）
5. 发送 http 请求
6. 服务器处理请求，浏览器接受 HTTP 响应
7. 浏览器解析并渲染页面
8. 关闭 TCP 连接（四次握手）

[相关博文](https://juejin.cn/post/6844903832435032072)


## 2. 事件循环相关题目--必考（一般是代码输出顺序判断）

```js
setTimeout(function () {
  console.log("1");
}, 0);
async function async1() {
  console.log("2");
  const data = await async2();
  console.log("3");
  return data;
}
async function async2() {
  return new Promise((resolve) => {
    console.log("4");
    resolve("async2的结果");
  }).then((data) => {
    console.log("5");
    return data;
  });
}
async1().then((data) => {
  console.log("6");
  console.log(data);
});
new Promise(function (resolve) {
  console.log("7");
  //   resolve()
}).then(function () {
  console.log("8");
});
```

输出结果：247536 async2 的结果 1


## 3. 浏览器缓存策略是怎样的（强缓存 协商缓存）具体是什么过程？

[前端浏览器缓存知识梳理](https://juejin.cn/post/6947936223126093861)


## 4. 浏览器渲染机制

[你不知道的浏览器页面渲染机制](https://juejin.cn/post/6844903815758479374)

## 5. 浏览器内核

- 主要分成两部分：渲染引擎(layout engineer或Rendering Engine)和JS引擎
- 渲染引擎：负责取得网页的内容（HTML、XML、图像等等）、整理讯息（例如加入CSS等），以及计算网页的显示方式，然后会输出至显示器或打印机。浏览器的内核的不同对于网页的语法解释会有不同，所以渲染的效果也不相同。所有网页浏览器、电子邮件客户端以及其它需要编辑、显示网络内容的应用程序都需要内核
- JS引擎则：解析和执行javascript来实现网页的动态效果
- 最开始渲染引擎和JS引擎并没有区分的很明确，后来JS引擎越来越独立，内核就倾向于只指渲染引擎

常见的浏览器内核有哪些

- Trident内核：IE,MaxThon,TT,The World,360,搜狗浏览器等。[又称MSHTML]
- Gecko内核：Netscape6及以上版本，FF,MozillaSuite/SeaMonkey等
- Presto内核：Opera7及以上。 [Opera内核原为：Presto，现为：Blink;]
- Webkit内核：Safari,Chrome等。 [ Chrome的Blink（WebKit的分支）]

## 6. cookies、sessionStorage、localStorage 和 indexDB 的区别

- cookie是网站为了标示用户身份而储存在用户本地的数据
- 是否在http请求只能够携带
  - cookie数据始终在同源的http请求中携带，跨域需要设置withCredentials = true
  - sessionStorage和localStorage不会自动把数据发给服务器，仅在本地保存
- 存储大小：
  - cookie数据大小不能超过4k；
  - sessionStorage和localStorage虽然也有存储大小的限制，但比cookie大得多，可以达到5M或更大，因不同浏览器大小不同；
- 有效时间：
  - cookie 设置的cookie过期时间之前一直有效，即使窗口或浏览器关闭
  - localStorage 硬盘存储持久数据，浏览器关闭后数据不丢失除非主动删除数据
  - sessionStorage 存在内存中，数据在当前浏览器窗口关闭后自动删除

## 7. 浏览器缓存

浏览器缓存分为强缓存和协商缓存。当客户端请求某个资源时，获取缓存的流程如下


- 先根据这个资源的一些 http header 判断它是否命中强缓存，如果命中，则直接从本地获取缓存资源，不会发请求到服务器；
- 当强缓存没有命中时，客户端会发送请求到服务器，服务器通过另一些request header验证这个资源是否命中协商缓存，称为http再验证，如果命中，服务器将请求返回，但不返回资源，而是告诉客户端直接从缓存中获取，客户端收到返回后就会从缓存中获取资源；
- 强缓存和协商缓存共同之处在于，如果命中缓存，服务器都不会返回资源； 区别是，强缓存不对发送请求到服务器，但协商缓存会。
- 当协商缓存也没命中时，服务器就会将资源发送回客户端。
- 当 ctrl+f5 强制刷新网页时，直接从服务器加载，跳过强缓存和协商缓存；
- 当 f5刷新网页时，跳过强缓存，但是会检查协商缓存；

强缓存

- Expires（该字段是 http1.0 时的规范，值为一个绝对时间的 GMT 格式的时间字符串，代表缓存资源的过期时间）
- Cache-Control:max-age（该字段是 http1.1的规范，强缓存利用其 max-age 值来判断缓存资源的最大生命周期，它的值单位为秒）

协商缓存

- Last-Modified（值为资源最后更新时间，随服务器response返回，即使文件改回去，日期也会变化）
- If-Modified-Since（通过比较两个时间来判断资源在两次请求期间是否有过修改，如果没有修改，则命中协商缓存）
- ETag（表示资源内容的唯一标识，随服务器response返回，仅根据文件内容是否变化判断）
- If-None-Match（服务器通过比较请求头部的If-None-Match与当前资源的ETag是否一致来判断资源是否在两次请求之间有过修改，如果没有修改，则命中协商缓存）

## 8. 重绘（Repaint）和回流（Reflow）

重绘和回流会在我们设置节点样式时频繁出现，同时也会很大程度上影响性能。


重绘：当节点需要更改外观而不会影响布局的，比如改变 color 就叫称为重绘
回流：布局或者几何属性需要改变就称为回流。
回流必定会发生重绘，重绘不一定会引发回流。回流所需的成本比重绘高的多，改变父节点里的子节点很可能会导致父节点的一系列回流。

以下几个动作可能会导致性能问题：

- 改变 window 大小
- 改变字体
- 添加或删除样式
- 文字改变
- 定位或者浮动
- 盒模型

重绘和回流其实也和 Eventloop 有关。

- 当 Eventloop 执行完 Microtasks 后，会判断 document 是否需要更新，因为浏览器是 60Hz 的刷新率，每 16.6ms 才会更新一次。
- 然后判断是否有 resize 或者 scroll 事件，有的话会去触发事件，所以 resize 和 scroll 事件也是至少 16ms 才会触发一次，并且自带节流功能。
- 判断是否触发了 media query
- 更新动画并且发送事件
- 判断是否有全屏操作事件
- 执行 requestAnimationFrame回调
- 执行 IntersectionObserver 回调，该方法用于判断元素是否可见，可以用于懒加载上，但是兼容性不好 更新界面
- 以上就是一帧中可能会做的事情。如果在一帧中有空闲时间，就会去执行 requestIdleCallback回调

如何减少重绘和回流：

1. 使用 transform 替代 top
2. 使用 visibility 替换display: none ，因为前者只会引起重绘，后者会引发回流（改变了布局）
3. 不要把节点的属性值放在一个循环里当成循环里的变量
4. 不要使用 table 布局，可能很小的一个小改动会造成整个 table 的重新布局
5. 动画实现的速度的选择，动画速度越快，回流次数越多，也可以选择使用requestAnimationFrame
6. CSS 选择符从右往左匹配查找，避免节点层级过多
7. 将频繁重绘或者回流的节点设置为图层，图层能够阻止该节点的渲染行为影响别的节点。比如对于 video 标签来说，浏览器会自动将该节点变为图层。
8. 避免使用css表达式(expression)，因为每次调用都会重新计算值（包括加载页面）
9. 尽量使用 css 属性简写，如：用 border 代替 border-width, border-style, border-color
10. 批量修改元素样式：elem.className 和 elem.style.cssText 代替 elem.style.xxx
11. 需要要对元素进行复杂的操作时，可以先隐藏(display:"none")，操作完成后再显示
12. 需要创建多个DOM节点时，使用DocumentFragment创建完后一次性的加入document
13. 缓存Layout属性值，如：var left = elem.offsetLeft; 这样，多次使用 left 只产生一次回流


设置节点为图层的方式有很多，我们可以通过以下几个常用属性可以生成新图层

- will-change
- video、iframe 标签


## 9. 首屏加载优化

1. Vue-Router路由懒加载（利用Webpack的代码切割）
2. 使用CDN加速，将通用的库从vendor进行抽离
3. Nginx的gzip压缩
4. Vue异步组件
5. 服务端渲染SSR
6. 如果使用了一些UI库，采用按需加载
7. Webpack开启gzip压缩
8. 如果首屏为登录页，可以做成多入口
9. 图片懒加载减少占用网络带宽
10. 页面使用骨架屏
11. 利用好script标签的async和defer这两个属性。功能独立且不要求马上执行的js文件，可以加入async属性。如果是优先级低且没有依赖的js，可以加入defer属性。


## 10. 说说浏览器的消息循环机制

在JavaScript中，任务被分为两种，一种宏任务（MacroTask）也叫Task，一种叫微任务（MicroTask）。

MacroTask（宏任务）
script全部代码、setTimeout、setInterval、setImmediate（浏览器暂时不支持，只有IE10支持，具体可见MDN）、I/O、UI Rendering。

MicroTask（微任务）
Process.nextTick（Node独有）、Promise、Object.observe(废弃)、MutationObserver

事件循环的进程模型

- 选择当前要执行的任务队列，选择任务队列中最先进入的任务，如果任务队列为空即null，则执行跳转到微任务（MicroTask）的执行步骤。
- 将事件循环中的任务设置为已选择任务。
- 执行任务。
- 将事件循环中当前运行任务设置为null。
- 将已经运行完成的任务从任务队列中删除。
- microtasks步骤：进入microtask检查点。
- 更新界面渲染。
- 返回第一步。


## 11. 浏览器的控制台是怎么渲染的？说说在浏览器控制台输出console到输出显示的过程

## 12. 单页和多页应用怎么通讯

websocket

## 13. 如何防范iframe被钓鱼网站嵌套导致的安全问题
iframe如何判断是否被嵌套?
```js
window.self === window.top
```


## 14. 模块化

CommonJs: node.js的模块系统，就是参照CommonJS规范实现的。在CommonJS中，有一个全局性方法require()，用于加载模块

## 15. 浏览器的线程有哪些？

浏览器的渲染进程是多线程的。js是阻塞单线程的。

1. GUI渲染线程

- 负责渲染浏览器界面，解析HTML，CSS，构建DOM树和RenderObject树，布局和绘制等。
- 当界面需要重绘（Repaint）或由于某种操作引发回流(reflow)时，该线程就会执行
- 注意，GUI渲染线程与JS引擎线程是互斥的，当JS引擎执行时GUI线程会被挂起（相当于被冻结了），GUI更新会被保存在一个队列中等到JS引擎空闲时立即被执行。

2. JS引擎线程

- 也称为JS内核，负责处理Javascript脚本程序。（例如V8引擎）
- JS引擎线程负责解析Javascript脚本，运行代码。
- JS引擎一直等待着任务队列中任务的到来，然后加以处理，一个Tab页（renderer进程）中无论什么时候都只有一个JS线程在运行JS程序
- 同样注意，GUI渲染线程与JS引擎线程是互斥的，所以如果JS执行的时间过长，这样就会造成页面的渲染不连贯，导致页面渲染加载阻塞。

3. 事件触发线程

- 归属于浏览器而不是JS引擎，用来控制事件循环（可以理解，JS引擎自己都忙不过来，需要浏览器另开线程协助）
- 当JS引擎执行代码块如setTimeOut时（也可来自浏览器内核的其他线程,如鼠标点击、AJAX异步请求等），会将对应任务添加到事件线程中
- 当对应的事件符合触发条件被触发时，该线程会把事件添加到待处理队列的队尾，等待JS引擎的处理
- 注意，由于JS的单线程关系，所以这些待处理队列中的事件都得排队等待JS引擎处理（当JS引擎空闲时才会去执行）

4. 定时触发器线程

- 传说中的setInterval与setTimeout所在线程
- 浏览器定时计数器并不是由JavaScript引擎计数的,（因为JavaScript引擎是单线程的, 如果处于阻塞线程状态就会影响记计时的准确）
- 因此通过单独线程来计时并触发定时（计时完毕后，添加到事件队列中，等待JS引擎空闲后执行）
- 注意，W3C在HTML标准中规定，规定要求setTimeout中低于4ms的时间间隔算为4ms。

5. 异步http请求线程

- 在XMLHttpRequest在连接后是通过浏览器新开一个线程请求
- 将检测到状态变更时，如果设置有回调函数，异步线程就产生状态变更事件，将这个回调再放入事件队列中。再由JavaScript引擎执行。
