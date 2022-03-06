# node

## 1. 常用模块

- path
- fs
- http，url
- express
- koa
- mongoose
- process
- cookie, session
- crypto加密相关
- os

## 2. express和koa的区别

- 编码风格：express采用回调函数风格，koa1 采用 generator，koa2(默认)使用await，风格上更加优雅，koa与es6,7的结合更加紧密;
- 错误处理：express采用在回调中错误优先的处理方式，深层次的异常捕获不了，使得必须在每一层回调里面处理错误，koa的使用try catch捕获错误，将错误上传可以统一处理错误;
- Koa 把 Express 中内置的 router、view 等功能都移除了，使得框架本身更轻量;
- express社区较大，文档也相对较多，koa社区相对较小;


