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

`speed-measure-webpack-plugin`来查看各个插件及loader的耗时，进行针对性的优化。

`webpack-bundle-analyzer`查看哪些包体积较大。

### 1. exclude / include
通过 exclude、include 配置来确保转译尽可能少的文件。

### 2. cache-loader
一些性能开销较大的 `loader` 之前添加 `cache-loader`，将结果缓存中磁盘中。默认保存在 `node_modueles/.cache/cache-loader` 目录下

如果只打算给 `babel-loader` 配置 `cache` 的话，也可以不使用 `cache-loader`，给 `babel-loader` 增加选项 `cacheDirectory`。

### 3. happypack
由于有大量文件需要解析和处理，构建是文件读写和计算密集型的操作，特别是当文件数量变多后，Webpack 构建慢的问题会显得严重。

HappyPack把任务分解给多个子进程去并发的执行，子进程处理完后再把结果发送给主进程。

happypack 默认开启 CPU核数 - 1 个进程

```js
module: {
        rules: [
            {
                test: /\.js[x]?$/,
                use: 'Happypack/loader?id=js',
                include: [path.resolve(__dirname, 'src')]
            },
            {
                test: /\.css$/,
                use: 'Happypack/loader?id=css',
                include: [
                    path.resolve(__dirname, 'src'),
                    path.resolve(__dirname, 'node_modules', 'bootstrap', 'dist')
                ]
            }
        ]
    },
    plugins: [
        new Happypack({
            id: 'js', //和rule中的id=js对应
            //将之前 rule 中的 loader 在此配置
            use: ['babel-loader'] //必须是数组
        }),
        new Happypack({
            id: 'css',//和rule中的id=css对应
            use: ['style-loader', 'css-loader','postcss-loader'],
        })
    ]
```

### 4. thread-loader

除了使用 Happypack 外，我们也可以使用 thread-loader ，把 thread-loader 放置在其它 loader 之前，那么放置在这个 loader 之后的 loader 就会在一个单独的 worker 池中运行。
在 worker 池(worker pool)中运行的 loader 是受到限制的。例如：

- 这些 loader 不能产生新的文件。
- 这些 loader 不能使用定制的 loader API（也就是说，通过插件）。
- 这些 loader 无法获取 webpack 的选项设置。


### 5. 开启 JS 多进程压缩

当前 Webpack 默认使用的是 TerserWebpackPlugin，默认就开启了多进程和缓存，构建时，你的项目中可以看到 terser 的缓存文件 node_modules/.cache/terser-webpack-plugin。

```js
optimization: {
    minimize: true,
    minimizer: [new TerserPlugin()],
  },
```

### 6. HardSourceWebpackPlugin

```js
plugins: [
  	...
    // 缓存 加速二次构建速度
    new HardSourceWebpackPlugin({
      // Either an absolute path or relative to webpack's options.context.
      //  cacheDirectory是在高速缓存写入 ，设置缓存在磁盘中存放的路径
      cacheDirectory: './../disk/.cache/hard-source/[confighash]',
      // Either a string of object hash function given a webpack config.
      recordsPath: './../disk/.cache/hard-source/[confighash]/records.json',
      //configHash在启动webpack实例时转换webpack配置，并用于cacheDirectory为不同的webpack配置构建不同的缓存
      configHash: function (webpackConfig) {
        // node-object-hash on npm can be used to build this.
        return require('node-object-hash')({ sort: false }).hash(webpackConfig);
      },
      // Either false, a string, an object, or a project hashing function.
      environmentHash: {
        root: process.cwd(),
        directories: [],
        files: ['./../package-lock.json', './../yarn.lock'],
      },
      // An object.
      info: {
        // 'none' or 'test'.
        mode: 'none',
        // 'debug', 'log', 'info', 'warn', or 'error'.
        level: 'debug',
      },
      // Clean up large, old caches automatically.
      cachePrune: {
        // Caches younger than `maxAge` are not considered for deletion. They must
        // be at least this (default: 2 days) old in milliseconds.
        maxAge: 2 * 24 * 60 * 60 * 1000,
        // All caches together must be larger than `sizeThreshold` before any
        // caches will be deleted. Together they must be at least this
        // (default: 50 MB) big in bytes.
        sizeThreshold: 100 * 1024 * 1024
      },
   }),
 ]
```

### 7. noParse
如果一些第三方模块没有AMD/CommonJS规范版本，可以使用 noParse 来标识这个模块，这样 Webpack 会引入这些模块，但是不进行转化和解析，从而提升 Webpack 的构建性能 ，例如：jquery 、lodash。

noParse 属性的值是一个正则表达式或者是一个 function。
```js
module.exports = {
    //...
    module: {
        noParse: /jquery|lodash/
    }
}
```

### 8. IgnorePlugin
webpack 的内置插件，作用是忽略第三方包指定目录。

### 9. externals
我们可以将一些JS文件存储在 CDN 上(减少 Webpack打包出来的 js 体积)，在 index.html 中通过 `<script>` 标签引入

### 10. 抽离公共代码

抽离公共代码是对于多页应用来说的，如果多个页面引入了一些公共模块，那么可以把这些公共的模块抽离出来，单独打包。公共代码只需要下载一次就缓存起来了，避免了重复下载。

```js
module.exports = {
    optimization: {
        splitChunks: {//分割代码块
            cacheGroups: {
                vendor: {
                    //第三方依赖
                    priority: 1, //设置优先级，首先抽离第三方模块
                    name: 'vendor',
                    test: /node_modules/,
                    chunks: 'initial',
                    minSize: 0,
                    minChunks: 1 //最少引入了1次
                },
                //缓存组
                common: {
                    //公共模块
                    chunks: 'initial',
                    name: 'common',
                    minSize: 100, //大小超过100个字节
                    minChunks: 3 //最少引入了3次
                }
            }
        }
    }
}
```

## 3. 打包加速方法
同上

## 4. loader、plugin原理
loader本质上就是一个函数，会读取文件内容并进行一些操作。
```js
module.exports = function (source) {
 console.log('source>>>>', source)
 return source
}
```

plugin实际是一个类（构造函数），通过在plugins配置中实例化进行调用
```js
function MyExampleWebpackPlugin() {
  
};
// 在插件函数的prototype上定义一个 apply 方法
MyExampleWebpackPlugin.prototype.apply = function(compiler) {
  // 指定一个挂载到webpack自身的事件钩子。
  compiler.plugin('webpacksEventHook', function(compilation, callback) {
    console.log('这是一个插件demo');

    // 功能完成后调用 webpack 提供的回调
    callback();
  })
}

// 导出plugin
module.exports = MyExampleWebpackPlugin;
```
`apply`方法给插件传递compiler对象。插件实列获取该compiler对象后，就可以通过 compiler.plugin('事件名称', '回调函数'); 监听到webpack广播出来的事件.

## 5. module、chunk、bundle、asset的区别

- module：只要是文件，都是一个module
- chunk：代码块，是webpack根据功能拆分出来的，如：entry、import() 、splitChunk
- bundle：webpack打包之后的各个文件，一般就是和chunk是一对一的关系，bundle就是对chunk进行编译压缩打包等处理之后的产出。
- assets： 项目中被引用的资源，通常为各种格式的图片和字体文件

## 6. chunk一定是通过入口生成的吗

不是

可以在optimization中配置生成
```js
optimization: {
    splitChunks: {
      chunks: 'all'
    }
  },
```
import 也行

## 7. css-loader的作用
处理依赖关系

## 8. css中的路径是如何解析的

## 9. css-loader和file-loader如何一起工作

1. 看css-loader的包介绍，意思是把@import 和 url() 这种方式的转换成import/require()方式
2. 看File Loader的介绍，他是返回对应文件的publicUrl

## 10. 打包体积优化

- `webpack-bundle-analyzer`插件可以可视化的查看webpack打包出来的各个文件体积大小，以便我们定位大文件，进行体积优化
- 提取第三方库或通过引用外部文件的方式引入第三方库
- 代码压缩插件UglifyJsPlugin
- 服务器启用gzip压缩
- 按需加载资源文件 require.ensure=
- 剥离css文件，单独打包
- 去除不必要插件，开发环境与生产环境用不同配置文件
- SpritesmithPlugin雪碧图，将多个小图片打包成一张，用background-image，backgroud-pisition，width，height控制显示部分
- url-loader 文件大小小于设置的尺寸变成base-64编码文本，大与尺寸由file-loader拷贝到目标目录


