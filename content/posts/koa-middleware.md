---
title: Koa原理及中间件模型实现
cover: https://images-1252366546.cos.ap-guangzhou.myqcloud.com/20210402172711.png
date: '2020-10-27'
description: Koa 中间件的洋葱模型最为人所知，本文探究一下洋葱模型的实现原理
---
Koa带来了什么
--------

> Koa 是一个新的 web 框架，由 Express 幕后的原班人马打造， 致力于成为 web 应用和 API 开发领域中的一个更小、更富有表现力、更健壮的基石。 Koa 并没有捆绑任何中间件， 而是提供了一套优雅的方法，帮助您快速而愉快地编写服务端应用程序。

相比Express，Koa使用了ES7的 async/await 语法实现异步，避免了回调地狱，十分优雅。Koa足够简洁，只提供基础功能，并不提供任何额外的功能。本身代码不到2000行，所有的功能都通过中间件实现。

短短几行代码就实现一个hello world应用。

```js
const Koa = require('koa');
const app = new Koa();

app.use(async ctx => {
  ctx.body = 'Hello World';
});

app.listen(3000);

```

源码结构
----

Koa极其精简，只有区区4个文件：

```text
── lib
   ├── application.js
   ├── context.js
   ├── request.js
   └── response.js

```

`application.js` 是Koa的入口，在这里主要做了这几件事：

1.  创建HTTP服务
2.  实现洋葱模型中间件机制
3.  创建context，request，response
4.  统一处理错误

`context.js` 主要功能就是代理了response对象和request对象的部分属性和方法。

`request.js` 是定义了request的原型，基于http模块的request抽象封装了一系列属性和方法。

`response.js` 与request类似，定义了 response 的原型，封装了一系列属性和方法。

Koa中间件机制
--------

Koa最核心的功能就是它的中间件机制，中间件通过use方法注册，运行的时候从最外层开始执行，遇到 next 后加入下一个中间件，执行完毕后回到上一个中间件，这就是大家所熟知的洋葱模型。

![](https://images-1252366546.cos.ap-guangzhou.myqcloud.com/20210402171824.png)

### 中间件概念

Koa中间件其实就是一个函数，有两个参数：context和next，context就是Koa的context对象，next是一个函数，调用next就进入下一个中间件。

```js
const Koa = require('koa');
const app = new Koa();

app.use(async (ctx, next) => {
  console.log('middleware1 start')
  next && await next()
  console.log('middleware1 end')
})

app.use(async (ctx, next) => {
  console.log('middleware2 start')
  next && await next()
  console.log('middleware2 end')
})

app.use(async ctx => {
  console.log('middleware3 start')
  ctx.body = 'Hello World';
  console.log('middleware3 end')
});

app.listen(3000);

```

当有请求进来后，上述的代码执行结果为：

```text
middleware1 start
middleware2 start
middleware3 start
middleware3 end
middleware2 end
middleware1 end

```

### 同步实现

可以看到其实就是洋葱模型其实就是函数调用栈，查看Koa这部分代码，`use`方法把中间件函数 push 进一个数组，之后用一个 `componse` 的函数组合起来，对于同步的中间件，可以简单实现一个 `componse` 函数。

```js
const compose = (middlewares) => {
  return (ctx) => {
    const dispatch = (i) => {
      if (!middlewares[i]) return
      return middlewares[i](ctx, () => dispatch(i + 1))
    }
    return dispatch(0)
  }
}

```

测试一下：

```js
const fnFactory = (n) => (ctx, next) => {
  console.log(`middleware${n} start`)
  next && next()
  console.log(`middleware${n} end`)
}

const middlewares = [fnFactory(1), fnFactory(2), fnFactory(3)]

const fnMiddleware = compose(middlewares)
fnMiddleware()


```

输出结果：

```text
middleware1 start
middleware2 start
middleware3 start
middleware3 end
middleware2 end
middleware1 end

```

### 异步实现

Koa 的中间件模型不止能处理同步的中间件，还能处理异步的中间件，修改一下上面的代码，用 `Promise` 包装一下，支持异步的中间件模型就实现了。

```js
const compose = (middlewares) => {
  return (ctx) => {
    const dispatch = (i) => {
      if (!middlewares[i]) return Promise.resolve()
      return Promise.resolve(middlewares[i](ctx, () => dispatch(i + 1)))
    }
    return dispatch(0)
  }
}

```

测试一下：

```js
const fnFactory = (n) => async (ctx, next) => {
  console.log(`middleware${n} start`)
  next && await next()
  console.log(`middleware${n} end`)
}

const middlewares = [fnFactory(1), fnFactory(2), fnFactory(3)]

const fnMiddleware = compose(middlewares)
fnMiddleware()


```

输出结果：

```text
middleware1 start
middleware2 start
middleware3 start
middleware3 end
middleware2 end
middleware1 end 
```