# html相关

## 1. 什么是事件代理（事件委托） 有什么好处

事件委托的原理：不给每个子节点单独设置事件监听器，而是设置在其父节点上，然后利用冒泡原理设置每个子节点。

**优点**

- 减少内存消耗和 `dom` 操作，提高性能。在 `JavaScript` 中，添加到页面上的事件处理程序数量将直接关系到页面的整体运行性能，因为需要不断的操作 `dom`,那么引起浏览器重绘和回流的可能也就越多，页面交互的事件也就变的越长，这也就是为什么要减少 `dom` 操作的原因。每一个事件处理函数，都是一个对象，多一个事件处理函数，内存中就会被多占用一部分空间。如果要用事件委托，就会将所有的操作放到 `js` 程序里面，只对它的父级进行操作，与 `dom` 的操作就只需要交互一次，这样就能大大的减少与 `dom` 的交互次数，提高性能；

- 动态绑定事件：因为事件绑定在父级元素 所以新增的元素也能触发同样的事件


## 2. `addEventListener` 默认是捕获还是冒泡

默认是冒泡

`addEventListener`第三个参数默认为 `false` 代表执行事件冒泡行为。

当为 `true` 时执行事件捕获行为。


## 3. html5有哪些新特性

新增功能：HTML5 现在已经不是 SGML 的子集，主要是关于图像，位置，存储，多任务等功能的增加

- 新增选择器 document.querySelector、document.querySelectorAll
- 拖拽释放(Drag and drop) API
- 媒体播放的 video 和 audio
- 本地存储 localStorage 和 sessionStorage
- 离线应用 manifest
- 桌面通知 Notifications
- 语意化标签 article、footer、header、nav、section
- 增强表单控件 calendar、date、time、email、url、search
- 地理位置 Geolocation
- 多任务 webworker
- 全双工通信协议 websocket
- 历史管理 history
- 跨域资源共享(CORS) Access-Control-Allow-Origin
- 页面可见性改变事件 visibilitychange
- 跨窗口通信 PostMessage
- Form Data 对象
- 绘画 canvas

移除的元素：

- 纯表现的元素：basefont、big、center、font、 s、strike、tt、u
- 对可用性产生负面影响的元素：frame、frameset、noframes

viewport
```html
<meta name="viewport" content="width=device-width,initial-scale=1.0,minimum-scale=1.0,maximum-scale=1.0,user-scalable=no" />
<!-- 
    width    设置viewport宽度，为一个正整数，或字符串‘device-width’
    device-width  设备宽度
    height   设置viewport高度，一般设置了宽度，会自动解析出高度，可以不用设置
    initial-scale    默认缩放比例（初始缩放比例），为一个数字，可以带小数
    minimum-scale    允许用户最小缩放比例，为一个数字，可以带小数
    maximum-scale    允许用户最大缩放比例，为一个数字，可以带小数
    user-scalable    是否允许手动缩放
-->
```
延伸：怎样处理 移动端 1px 被 渲染成 2px问题？

局部处理

meta标签中的 viewport属性 ，initial-scale 设置为 1
rem按照设计稿标准走，外加利用transfrome 的scale(0.5) 缩小一倍即可；
全局处理
mate标签中的 viewport属性 ，initial-scale 设置为 0.5
rem 按照设计稿标准走即可
