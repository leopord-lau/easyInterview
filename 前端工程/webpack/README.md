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

- devtool 的 sourceMap较为耗时
- 开发环境不做无意义的操作：代码压缩、目录内容清理、计算文件hash、提取CSS文件等
- 第三方依赖外链script引入：vue、ui组件、JQuery等
- HotModuleReplacementPlugin：热更新增量构建
- DllPlugin& DllReferencePlugin：动态链接库，提高打包效率，仅打包一次第三方模块，每次构建只重新打包业务代码。
- thread-loader,happypack：多线程编译，加快编译速度
- noParse：不需要解析某些模块的依赖
- babel-loader开启缓存cache
- splitChunks（老版本用CommonsChunkPlugin）：提取公共模块，将符合引用次数(minChunks)的模块打包到一起，利用浏览器缓存
- Tree Shaking 摇树：基于ES6提供的模块系统对代码进行静态分析, 并在压缩阶段将代码中的死代码（dead code)移除，减少代码体积。

## 4. 打包体积优化

- webpack-bundle-analyzer插件可以可视化的查看webpack打包出来的各个文件体积大小，以便我们定位大文件，进行体积优化
- 提取第三方库或通过引用外部文件的方式引入第三方库
- 代码压缩插件UglifyJsPlugin
- 服务器启用gzip压缩
- 按需加载资源文件 require.ensure=
- 剥离css文件，单独打包
- 去除不必要插件，开发环境与生产环境用不同配置文件
- SpritesmithPlugin雪碧图，将多个小图片打包成一张，用background-image，backgroud-pisition，width，height控制显示部分
- url-loader 文件大小小于设置的尺寸变成base-64编码文本，大与尺寸由file-loader拷贝到目标目录


