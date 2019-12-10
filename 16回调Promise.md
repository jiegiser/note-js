# 回调

`javascript`中并不存在并发，而是在一个进程中（也就是一个事件中）执行任务的快速切换，使得导致人们以为是在并发执行事件。

回调表达程序异步和管理并发的两个主要缺陷：缺乏顺序性和可信任性。

# Promise

一旦`Promise`决议，他就永远保持这个状态。此时他就称为了不变值，可以根据需求多次查看。`Promise`是一种封装和组合未来值得易于复用的机制。

如果你试图使用恰好有`then(..)`函数的一个对象或函数值完成一个`Promise`，但是并不希望他被当做`Promise`或`thenable`，那就有点麻烦了，因为他会自动识别为`thenable`，并被按照特定的规则处理。这就说明，我们的对象或者函数补鞥呢拥有`then(..)`函数，否则这个值在`Promise`系统中就会被误认为是一个`thenable`，这可能会导致难以追踪的`bug`，有些早期实现`Promise`机制的库就会改变自己本来有的`then(..)`方法，而有的就会直接说明：与基于`Promise`的库不兼容。

> 什么是 thenable 对象 ？

```js
let thenable = {
  then: function(resolve, reject) {
    resolve(42)
  }
}
```

我们使用异步回调函数可能会出现的问题：

- 调用回调过早
- 调用回调过晚（或不被调用）
- 调用回调次数过少或过多
- 未能传递所需的环境和参数
- 吞掉可能出现的错误异常

`Promise`的特性就是专门用来为这些问题提供一个有效的可复用的答案。

一个`Promise`决议后，这个`Promise`上所有的通过`then(..)`注册的回调都会在下一个异步时机点上依次被立即调用。这些回调中的任意一个都无法影响或延误对其他回调的调用。比如下面的代码：
```js
p.then(function () {
  p.then(function () {
    console.log('c')
  })
  console.log('a')
})
p.then(function () {
  console.log('b')
})
// a b c
```
这里的`c`不会先于`b`执行的，他不会打算`b`的执行。

### promise的调度技巧
如果两个`promise p1`和`p2`都可以决议，那么`p1.then(..);p2.then(..)`应该最终会先调用`p1`的回调，然后是`p2`的那些。那还有一些微妙的场景可能不是这样的，如下面的代码：
```js
var p3 = new Promise(function (resolve, reject) {
  resolve('B')
})
var p1 = new Promise(function (resolve, reject) {
  resolve(p3)
})
var p2 = new Promise(function (resolve, reject) {
  console.log('A')
})
p1.then(function (v) {
  console.log(v)
})
p2.then(function (v) {
  console.log(v)
})
// A B  并不是我们想象中的B A
```
### promise解决回调未调用问题
首先一个`Promise`如果永远不会决议，即使这样，`Promise`也提供了一个解决方案，其使用了一种称为竟态的高级抽象机制：
```js
// 一个用于计算超市的promise工具
function timeoutPromise(delay) {
  return new Promise(function (resolve, reject) {
    setTimeout(function () {
      reject('Timeout!')
    }, delay)
  })
}
// 假设foo方法会超时
Promise.race([foo(), timeoutPromise(3000)])
  .then(function () {
    // foo如果完成执行这里
  },
  function (err) {
    // foo被拒绝或者foo执行超时
  })
```
### promise解决调用次数过多或过少的问题
`Promise`的定义方式使得他只能被决议一次，如果处于某种原因，`Promise`创建的代码试图调用`resolve(..)`或`reject(..)`多次，或者试图两者都调用，那么这个`Promise`将只会接受第一次决议，并默默忽略后续调用。所以任何通过`then(..)`注册的（每个）回调就只会被调用一次。

`Promise.resolve()`会将不是`Prmomise`对象的参数转化为一个`Promise` 对象。比如说，我们要调用一个函数`foo()`，但是我们不确定得到的返回值是不是一个`Promise`对象，但是我们可以知道他至少是一个`thenable`，我们可以使用`Promise.resolve()`将他封装成一个`Promise`对象：
```js
// 下面这种是不能保证的
foo(42)
.then(function (v) {
  console.log(v)
})
// 下面这个是有保障的
Promise.resolve(fun(42))
.then(function (v) {
  console.log(v)
})
```

### 链式流
我们可以将多个`Promise`连接在一起来表示一系列的异步操作：

```js
var p = Promise.resolve(21)
p.then(function(v) {
  console.log(v)
  return v * 2
})
.then(function(v) {
  console.log(v)
})
// 21
// 42
```

如果需要步骤二等待步骤一是一个异步操作，可以返回一个`Promise`对象：

```js
var p = Promise.resolve(21)
p.then(function (v) {
  console.log(v)
  // 创建一个promise并返回
  return new Promise(function(resolve, reject) {
    // 引入异步
    setTimeout(function () {
      // 用值42进行填充
      resolve(42)
    }, 1000)
  })
})
.then(function (v) {
  // 在前面的1秒中延迟之后运行
  console.log(v)
  // 42
})
```
在上面的例子中，一步步传递值是可选的，如果不显示返回一个值，就会隐式返回`undefined`，并且这些`promise`仍然会以同样的方式链接在一起。这样，每个`Promise`的决议就是下一个步骤开始的信号。

我们可以将延迟`Promise`创建（没有决议消息）过程一般化到一个工具中，以便在多个步骤中复用：
```js
function delay(time) {
  return new Promise(function(resolve, reject) {
    setTimeout(resolve, time)
  })
}
delay(100)
.then(function STEP2(v) {
  console.log(v)
  console.log('step 2 (after 100ms)')
  return delay(200)
})
.then(function STEP3(v) {
  console.log(v)
  console.log('step3 (after another 200ms)')
})
.then(function STEP4(v) {
  console.log(v)
  console.log('step4 (next Job)')
  return delay(50)
})
.then(function STEP5(v) {
  console.log(v)
  console.log('step5 (after another 50ms)')
})
...
```
调用`delay(100)`创建一个将在100毫秒后完成的`promise`。

上面的代码是没有消息延迟的情况，我们大多数的情况是等待前面的异步完成回调值传给下一个异步进行执行：
```js
function request(url) {
  return new Promise(function (resolve, reject) {
    // ajax的回调应该是我们这个promise的resolve函数
    ajax(url, resolve)
  })
}
request('http://url/l1')
.then(function (response) {
  return request('http://url/l2/?v='+response)
})
.then(function (response) {
  console.log(response)
})
```
如果在这个异步链中，在某一步骤中出现了错误，他就会在异步链中寻找显示定义的拒绝处理函数，也就是`reject`函数，或者是`.then`的第二个函数。如果遇到了就会执行，然后继续执行后面的异步链。如下模拟代码：
```js
// 步骤1
request('http://some.url.1')
// 步骤2
.then(function (response) {
  foo.bar() // undefined 出错
  // 永远不会执行这里
  return request('http://some.url.2/?v='+ response)
})
// 步骤3
.then(
  function fulfilled(response) {
    // 永远不会执行这里
  },
  // 捕捉错误的拒绝处理函数
  function rejected(err) {
    console.log(err)
    // 来自foo.bar()的错误TypeError
    return 42
  }
)
// 步骤4
.then(function (msg) {
  // 42
  console.log(msg)
})
```
如果是如下代码：
```js
var p = new Promise(function (resolve, reject) {
  reject('Oops')
})
var p2 = p.then(
  function fullfilled() {
    // 永远不会执行这里
  }
  // 假定的拒绝处理函数，如果省略或者传入任何非函数值
  // function(err) {
  //   throw err
  // }
)
```
上面的代码，默认拒绝处理函数只是把错误重新抛出，这最终会使得`p2`（链接的`promise`）用同样的错误理由拒绝。从本质上说，这使得错误可以继续沿着`Promise`链传播下去，直到遇到显示定义的拒绝处理函数。

如果没有给`then(..)`传递一个适当有效的函数作为完成处理函数参数，还是会有作为替代的一个默认处理函数：
```js
var p = Promise.resolve(42)
p.then(
  // 假设的完成处理函数，如果省略或者传入任何非函数值
  // function (v) {
  //   return v
  // },
  null,
  function rejected(err) {
    // 永远不会到达这里
  }
)
```
可以看到，默认的完成处理函数只是把接收到的任何传入值传递给下一个步骤（`Promise`）而已。

> then(null, function() {})这个模式，只处理拒绝，但又把完成值传递下去。是有一个缩写的`API`，`catch(function(err){..})`