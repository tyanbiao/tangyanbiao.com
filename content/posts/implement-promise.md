---
title: '实现一个最简单的 Promise'
cover: 'https://images-1252366546.cos.ap-guangzhou.myqcloud.com/20210402170827.png'
date: '2020-05-07'
description: '手把手实现一个最简单的 Promise'
---
## 一、前言

Promise 是 ES6 新增的一个内置对象，用来避免回调地狱的一种解决方案，相比回调函数，使用Promise 处理异步流程可以使代码层次清晰，便于理解，配合 async/await 语法糖更是直接以同步的方式写异步代码，再也不用担心回调地狱了。

Promise 有3种状态：pending、fulfilled、rejected。一个 Promise 最初的状态是 pending，在 resolve 被调用时变为 fulfilled，或者在 reject 被调用时变为 rejected。可以简单理解为 Promise 就是一个发布订阅模型，通过 then 方法注册状态改变时的回调函数，当 Promise 的状态改变时调用注册的回调函数并返回新的 Promise。

![](https://images-1252366546.cos.ap-guangzhou.myqcloud.com/promise-resolve-reject.svg)

目前 Promise 的主流规范主要是 [Promises/A+](https://promisesaplus.com/)，对照标准就可以实现一个完整的 Promise，本文只是对 Promise 的简单实现，复杂情况暂时先不考虑。

## 二、内部属性

Promise 的主要逻辑就是对其内部状态的维护，要实现一个 Promise，需要定义一些属性来保存状态。

```jsx
function Promise () {
    this.status = 'pending'
    this.value = undefined
    this.onFulfilledCallback = []
    this.onRejectedCallback = []
}
```

## 三、构造函数

Promise 的构造函数接受一个带有 resolve 和 reject 两个参数的函数，先把它称之为 executor，Promise 构造函数执行时立即调用 executor 并传递 resolve 和 reject 两个函数作为参数，resolve 和 reject 函数被调用时，会分别将 Promise 的状态改变为 fulfilled 或者 rejected，如果 executor 中抛出一个错误，Promise 状态也会变为 rejected。了解了这些，接下来继续完善 Promise 的构造函数。

```jsx
function Promise (executor) {
    this.status = 'pending'
    this.value = undefined
    this.onFulfilledCallback = []
    this.onRejectedCallback = []
    try {
				executor(this.resolve.bind(this), this.reject.bind(this))
    } catch (e) {
        this.reject(e)
    }
}
```

浏览器中的 resolve 和 reject 函数是使用原生代码在JavaScript虚拟机中实现的，这里需要我们自己实现。resolve 和 reject 主要的功能就是改变Promise的状态，然后触发回调函数，这里我们把 resolve 和 reject 定义在 Promise 的原型上。

```jsx
Promise.prototype.resolve = function(value) {
    if (this.status !== 'pending') return
    this.status = 'fulfilled'
    this.value = value
    this.onFulfilledCallback.forEach(function(cb) {
		    cb(value)
    })
}
Promise.prototype.reject = function(reason) {
    if (this.status !== 'pending') return
    this.status = 'rejected'
    this.value = reason
    this.onRejectedCallback.forEach(function(cb) {
		    cb(reason)
    })
}
```

## 四、then 和 catch 方法

then 方法用来注册 Promise 状态确定后的回调，then 方法最多需要 onFulfilled， onRejected 两个参数：分别对应 Promise 的成功和失败情况的回调函数，并返回一个新的Promise。对应不同的状态，调用 then 方法执行的逻辑也不一致。

- fulfilled: 返回一个新的 Promise，在新 Promise 的 executor 中调用 onFulfilled，并立即调用新Promise 的 resolve 函数。
- rejected：和 fulfilled 类似
- pending：这是平时使用过程中最常见情况，旧 Promise 的执行结果还未确定，分别将处理代码注册为回调函数：onFulfilledCallback、onRejectedCallback，等到旧 Promise 的结果确定后调用。

根据规范：如果 onFulfilled 或 onRejected 的返回结果是 Promise，那么 then 返回的 Promise 的状态与之相同。

```jsx
Promise.prototype.then = function(onFulfilled, onRejected) {
    var _this = this
    var defaultFn = function(v) { return v} 
    onFulfilled = typeof onFulfilled === 'function' ? onFulfilled : defaultFn
    onRejected = typeof onRejected === 'function' ? onRejected : defaultFn

    if (_this.status === 'fulfilled') {
        return new Promise(function(resolve, reject) {
            var ret = onFulfilled(_this.value)
            if (ret instanceof Promise) {
                ret.then(resolve, reject)
            } else {
                resolve(ret)
            }
        })
    }
    if (_this.status === 'rejected') {
        return new Promise(function(resolve, reject) {
            var ret = onRejected(_this.value)
            if (ret instanceof Promise) {
                ret.then(resolve, reject)
            } else {
                reject(ret)
            }
        })
    }
    if (_this.status === 'pending') {
        return new Promise(function(resolve, reject) {
            _this.onFulfilledCallback.push(function(value) {
                try {
                    var ret = onFulfilled(value)
                    if (ret instanceof Promise) {
                      ret.then(resolve, reject)
                    } else {
                      resolve(ret)
                    }
                  } catch (e) {
                    reject(e)
                }
            })

            _this.onRejectedCallback.push(function(reason) {
                try {
                    var ret = onRejected(reason)
                    if (ret instanceof Promise) {
                        ret.then(resolve, reject)
                    } else {
                        resolve(ret)
                    }
                } catch (e) {
                    reject(e)
                }
            })
        })
    }
}
```

顺便实现 catch 方法

```jsx
Promise.prototype.catch = function(onRejected) {
    return this.then(null, onRejected)
}
```

## 五、执行时机

Promise 构造函数中传入的 executor 函数会立即执行，但是 then 方法中传入的 onFulfilled, onRejected 函数需要异步延迟调用，Promise/A+ 规范中解释：实践中要确保 onFulfilled 和 onRejected 方法异步执行，且应该在 then 方法被调用的那一轮事件循环之后的新执行栈中执行。这个事件队列可以采用宏任务 macro-task 机制或微任务 micro-task 机制来实现。

因此即便一个 promise 立即被 resolve，在 then 之后的代码也会在 onFulfilled, onRejected 前面执行，如下例打印结果是1，3，2 而不是 1，2，3 。

```jsx
let promise = new Promise((resolve, reject) => {
    console.log(1)
    resolve()
})
promise.then(() => {
    console.log(2)
})
console.log(3)
// 打印结果
// 1
// 3
// 2
```

在浏览器中 onFulfilled 和 onRejected 的异步执行是使用微任务机制实现的，我们使用宏任务setTimeout 来实现异步执行，稍微修改一下 resolve、reject 的实现代码。

```jsx
Promise.prototype.resolve = function(value) {
    var _this = this
    if (_this.status !== 'pending') return
    setTimeout(function() {
        _this.status = 'fulfilled'
        _this.value = value
        _this.onFulfilledCallback.forEach(function(cb) {
            cb(value)
        })
    })
}
Promise.prototype.reject = function(reason) {
    var _this = this
    if (_this.status !== 'pending') return
    setTimeout(function() {
        _this.status = 'rejected'
        _this.value = reason
        _this.onRejectedCallback.forEach(function(cb) {
            cb(reason)
        })
    }, 0)
}
```

## 六、测试

```jsx
let promise = new Promise((resolve, reject) => {
    console.log('start')
    setTimeout(() => {
        resolve(1)
    }, 1000)
})
promise.then(v => {
    console.log(v)
    return v + 1
}).then(v => {
    return new Promise((resolve, reject) => {
        setTimeout(() => {
            console.log(v)
            resolve(v + 1)
        }, 1000)
    })
}).then(v => {
    console.log(v)
    return v + 1
})
```

打印结果

```jsx
start
1
2
3
```