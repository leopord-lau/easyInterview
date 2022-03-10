# 性能优化

## 1. 前端性能定位以及优化指标

前端性能优化 已经是老生常谈的一项技术了 很多人说起性能优化方案的时候头头是道 但是真正的对于性能分析定位和性能指标这块却一知半解 所以这道题虽然和性能相关 但是考察点在于平常项目如何进行性能定位和分析

1. 我们可以从 前端性能监控-埋点以及 window.performance相关的 api 去回答
2. 也可以从性能分析工具 Performance 和 Lighthouse
3. 还可以从性能指标 LCP FCP FID CLS 等去着手

[5 分钟撸一个前端性能监控工具](https://juejin.cn/post/6844903662020460552)

[前端性能优化之谈谈通用性能指标及上报策略](https://juejin.cn/post/6844904150057091086)

[前端性能优化指标 + 检测工具](https://juejin.cn/post/6974565176427151397)


**错误类型**
1. `SyntaxError`（语法错误）

   解析代码时发生的语法错误

    eg:var 1a;
　　Uncaught SyntaxError: Unexpected number

2. `ReferenceError`（引用错误）

    a.引用了一个不存在的变量

    eg: console.log(a);

　　Uncaught ReferenceError: a is not defined

    b.将变量赋值给一个无法被赋值的对象

    eg:console.log()= 1;

　　Uncaught ReferenceError: Invalid left-hand side in assignment

3. `RangeError`（范围错误）

    超出有效范围

    eg:var a= new Array(-1);

　　Uncaught RangeError: Invalid array length

4. `TypeError`（类型错误）

    a.变量或参数不是预期类型，比如，对字符串、布尔值、数值等原始类型的值使用new命令，就会抛出这种错误，因为new命令的参数应该是一个构造函数。

    eg: var a= new 123;

　　Uncaught TypeError: 123 is not a function

    b.调用对象不存在的方法

    eg:var a;a.aa();

　　Uncaught TypeError: Cannot read property 'aa' of undefined

　　
5. `URLError`（URL错误）

    与url相关函数参数不正确，主要是encodeURI()、decodeURI()、encodeURIComponent()、decodeURIComponent()、escape()和unescape()这六个函数。

    eg: decodeURI('%2')

　　Uncaught URIError: URI malformed

6. `EvalError`（eval错误）

    eval函数没有被正确执行


**监控方式**
- try...catch
- Promise.catch
- window.onerror


## 2. 说说白屏优化的方式有哪些

