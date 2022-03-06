# webpack相关

## 1. plugin和loader的区别

- `Loader：`

用于对模块源码的转换，`loader` 描述了 `webpack` 如何处理非 `javascript` 模块，并且在 `buld` 中引入这些依赖。`loader` 可以将文件从不同的语言（如 `TypeScript`）转换为 `JavaScript`，或者将内联图像转换为 `data URL`。比如说：`CSS-Loader`，`Style-Loader` 等。


- `Plugin`
- 
目的在于解决 `loader` 无法实现的其他事,它直接作用于 `webpack`，扩展了它的功能。在 `webpack` 运行的生命周期中会广播出许多事件，`plugin` 可以监听这些事件，在合适的时机通过 `webpack` 提供的 `API` 改变输出结果。插件的范围包括，从打包优化和压缩，一直到重新定义环境中的变量。插件接口功能极其强大，可以用来处理各种各样的任务。

[前端进阶高薪必看-Webpack篇](https://juejin.cn/post/6844904150988226574)


## 2. Webpack 有哪些优化手段

随着项目越来越大，Webpack 构建速度可能会越来越慢，构建出来的 js 的体积也越来越大，此时就需要对 Webpack 的配置进行优化。

[深度解锁Webpack系列](https://juejin.cn/post/6844904093463347208)

## 3. 打包加速方法

