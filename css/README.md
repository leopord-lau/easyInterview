# css相关

## 1. `css` 的渲染层合成是什么?

在 `DOM` 树中每个节点都会对应一个渲染对象（`RenderObject`），当它们的渲染对象处于相同的坐标空间（z 轴空间）时，就会形成一个 `RenderLayers`，也就是渲染层。渲染层将保证页面元素以正确的顺序堆叠，这时候就会出现层合成（`composite`），从而正确处理透明元素和重叠元素的显示。对于有位置重叠的元素的页面，这个过程尤其重要，因为一旦图层的合并顺序出错，将会导致元素显示异常。


## 2. 浏览器如何创建新的渲染层

- 根元素 `document`
- 有明确的定位属性（`relative、fixed、sticky、absolute`）
- `opacity < 1`
- 有 `CSS fliter` 属性
- 有 `CSS mask` 属性
- 有 `CSS mix-blend-mode` 属性且值不为 `normal`
- 有 `CSS transform` 属性且值不为 `none`
- `backface-visibility` 属性为 `hidden`
- 有 CSS reflection 属性
- 有 `CSS column-count` 属性且值不为 `auto` 或者有 `CSS column-width` 属性且值不为 `auto`
- 当前有对于 `opacity`、`transform`、`fliter`、`backdrop-filter` 应用动画
- `overflow` 不为 `visible`

[浏览器层合成与页面渲染优化](https://juejin.cn/post/6844903966573068301)


## 3. css优先级怎么计算

- 第一优先级：!important 会覆盖页面内任何位置的元素样式
- 内联样式，如 style="color: green"，权值为 1000
- ID 选择器，如#app，权值为 0100
- 类、伪类、属性选择器，如.foo, :first-child, div[class="foo"]，权值为 0010
- 标签、伪元素选择器，如 div::first-line，权值为 0001
- 通配符、子类选择器、兄弟选择器，如*, >, +，权值为 0000
- 继承的样式没有权值


## 4. position 有哪些值，作用分别是什么

- `static`

  `static`(没有定位)是 `position` 的默认值，元素处于正常的文档流中，会忽略 `left、top、right、bottom` 和 `z-index` 属性。


- `relative`
  
  `relative`(相对定位)是指给元素设置相对于原本位置的定位，元素并不脱离文档流，因此元素原本的位置会被保留，其他的元素位置不会受到影响。

  使用场景：子元素相对于父元素进行定位

- `absolute`

  `absolute`(绝对定位)是指给元素设置绝对的定位，元素相对定位的对象可以分为两种情况
  1. 设置了 `absolute` 的元素如果存在有祖先元素设置了 `position` 属性为 `relative` 或者 `absolute`，则这时元素的定位对象为此已设置 `position` 属性的祖先元素。
  2. 如果并没有设置了 `position` 属性的祖先元素，则此时相对于 `body` 进行定位

  使用场景：跟随图标 图标使用不依赖定位父级的 `absolute` 和 `margin` 属性进行定位，这样，当文本的字符个数改变时，图标的位置可以自适应

- `fixed`

  可以简单说 `fixed` 是特殊版的 `absolute`，`fixed` 元素总是相对于 `body` 定位的。

  使用场景：侧边栏或者广告图

- `inherit` 
  
  继承父元素的 `position` 属性，但需要注意的是 `IE8` 以及往前的版本都不支持 `inherit` 属性。

- `sticky`

  在屏幕范围（`viewport`）时该元素的位置并不受到定位影响（设置是 `top`、`left` 等属性无效），当该元素的位置将要移出偏移范围时，定位又会变成 `fixed`，根据设置的 `left`、`top` 等属性成固定位置的效果。
  
  当元素在容器中被滚动超过指定的偏移值时，元素在容器内固定在指定位置。亦即如果你设置了 `top: 50px`，那么在 `sticky` 元素到达距离相对定位的元素顶部 `50px` 的位置时固定，不再向上移动（相当于此时 `fixed` 定位）。

  使用场景：跟随窗口

## 5. 垂直水平居中实现方式

[面试官：你能实现多少种水平垂直居中的布局（定宽高和不定宽高）](https://juejin.cn/post/6844903982960214029)


## 6. css 怎么开启硬件加速(GPU 加速)

浏览器在处理下面的 `css` 的时候，会使用 `GPU` 渲染

- `transform`（当 3D 变换的样式出现时会使用 GPU 加速）
- `opacity`
- `filter`
- `will-change`



采用 `transform: translateZ(0)`
采用 `transform: translate3d(0, 0, 0)`
使用 `CSS` 的 `will-change`属性。 `will-change` 可以设置为`opacity`、`transform`、`top`、`left`、`bottom`、`right`。


> 注意！层爆炸，由于某些原因可能导致产生大量不在预期内的合成层，虽然有浏览器的层压缩机制，但是也有很多无法进行压缩的情况，这就可能出现层爆炸的现象（简单理解就是，很多不需要提升为合成层的元素因为某些不当操作成为了合成层）。解决层爆炸的问题，最佳方案是打破 `overlap` 的条件，也就是说让其他元素不要和合成层元素重叠。简单直接的方式：使用 3D 硬件加速提升动画性能时，最好给元素增加一个 `z-index` 属性，人为干扰合成的排序，可以有效减少创建不必要的合成层，提升渲染性能，移动端优化效果尤为明显。

## 7. flex:1 是哪些属性组成的

`flex` 实际上是 `flex-grow`、`flex-shrink` 和 `flex-basis` 三个属性的缩写

`flex-grow`：定义项目的的放大比例；

默认为`0`，即 即使存在剩余空间，也不会放大；

所有项目的`flex-grow`为1：等分剩余空间（自动放大占位）；
`flex-grow`为`n`的项目，占据的空间（放大的比例）是`flex-grow`为`1`的`n`倍。


`flex-shrink`：定义项目的缩小比例；

默认为`1`，即 如果空间不足，该项目将缩小；
所有项目的`flex-shrink`为`1`：当空间不足时，缩小的比例相同；
`flex-shrink`为`0`：空间不足时，该项目不会缩小；
`flex-shrink`为`n`的项目，空间不足时缩小的比例是`flex-shrink`为`1`的`n`倍。

`flex-basis`： 定义在分配多余空间之前，项目占据的主轴空间（`main size`），浏览器根据此属性计算主轴是否有多余空间
默认值为`auto`，即 项目原本大小；
设置后项目将占据固定空间。


## 8. 前端性能优化之css

CSS优化、提高性能的方法有哪些？

- 多个`css`合并，尽量减少`HTTP`请求
- 将`css`文件放在页面最上面
- 移除空的`css`规则
- 避免使用`CSS`表达式
- 选择器优化嵌套，尽量避免层级过深
- 充分利用`css`继承属性，减少代码量
- 抽象提取公共样式，减少代码量
- 属性值为0时，不加单位
- 属性值为小于1的小数时，省略小数点前面的0
- 使用`CSS Sprites`将多张图片拼接成一张图片，通过`CSS background` 属性来访问图片内容

## 9. 移除inline-block间隙

- 移除空格
- 使用`margin`负值
- 使用`font-size:0`
- `letter-spacing`
- `word-spacing`


## 10. 清除浮动

[清除浮动](https://www.html.cn/qa/css3/13783.html)

## 11. （外）边距重叠
布局垂直方向上两个元素的间距不等于margin的和，而是取较大的一个

同级相邻元素

现象：上方元素设置margin-bottom: 20px，下方元素设置margin-top: 10px，实际的间隔是20px
避免办法：同级元素不要同时设置，可都设置margin-bottom或margin-top的一个，或者设置padding

父子元素

现象：父元素设置margin-top: 20px，下方元素设置margin-top: 10px，实际的间隔是20px
避免办法：父元素有padding-top，或者border-top。或者触发BFC

## 12. BFC

[CSS 中你应该了解的 BFC](https://juejin.cn/post/6904568682555375624)

## 13. flex

## 14. css动画
[CSS动画简介](http://www.ruanyifeng.com/blog/2014/02/css_transition_and_animation.html)

## 15.  transition和animation的区别？

1. animation 其实也叫关键帧，通过和 keyframe 结合可以设置中间帧的一个状态；transition 是过渡，是样式值的变化的过程，只有开始和结束；

2. animation配合 @keyframe 可以不触发事件就触发这个过程，而 transition 需要通过 hover 或者 js 事件来配合触发；

3. animation 可以设置很多的属性，比如循环次数，动画结束的状态等等，transition 只能触发一次；

4. animation 可以结合 keyframe 设置每一帧，但是 transition 只有两帧；

## 16. 说说requestAnimationFrame的作用，并实现获取每秒的帧数
`requestAnimationFrame`： 告诉浏览器在下次重绘之前执行传入的回调函数(通常是操纵 dom，更新动画的函数)；由于是每帧执行一次，那结果就是每秒的执行次数与浏览器屏幕刷新次数一样，通常是每秒 60 次。

```js
let a = 0
function step(timestamp) {
  a++
  element.style.transform = 'translateX(' + a + 'px)';      
  if (a<100) { // 在100次调用后停止动画
    window.requestAnimationFrame(step);
  }
}           
window.requestAnimationFrame(step);
```

## 17. css实现一个正三角形
```css
 .triangle {
      width: 50px;
      height: 50px;
      border-width: 50px;
      border-style: solid;
      border-color: red transition transition transition;
 }
```

## 18.  display:none、visibility:hidden 和 opacity:0 之间的区别？

这三者都是隐藏，但是具有以下不同点：
- `display: none` 隐藏后不占位置， `visibility: hidden`、`opacity: 0` 隐藏后占位置、
- `display: none` 不会被子元素继承，`visibility: hidden`会被子元素继承，通过设置子元素的`visibility: visible`来显示。`opacity: 0`会被子元素继承，但是不能通过设置来重新显示。
- `display: none` 不会触发绑定事件，元素不会在页面存在，没意义。`visibility: hidden`也不会触发。`opacity: 0`可以触发
- `transition`只对`opacity`有效
