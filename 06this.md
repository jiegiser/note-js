# this

### 匿名函数的缺点：

第一：匿名函数在栈追踪中不会系那是出有意义的函数名，使得调试很困难。需要引用 第二：如果没有函数名，当函数需要引用自身时只能使用已经过期的 arguments.callee 引用，比如在递归中。另一个函数需要引用自身的例子，是在事件触发后事件监听器需要解绑自身。
第三：匿名函数省略了对于代码可读性/可理解性很重要的函数名。一个描述性的名称可以让代码不言自明。始终给函数表达式命名是一个最佳实践。

---

> ### this 在任何情况下都不指向函数的词法作用域。

---

### 箭头函数的 this 指向：

是在词法分析阶段就对 this 进行了绑定。也就是绑定到该属性的词法作用域。

---

### this 是什么：

当一个函数被调用时，会创建一个执行上下文，这个执行上下文包含一些函数在哪里被调用（调用栈）、函数的调用方式、传入的参数信息等等，this 就是这个记录的一个属性，会在函数执行的过程中用到。this 实际上是在函数被调用时发生绑定，他指向什么完全取决于函数在哪里被调用。

---

### this 的指向：

第一是指向对象属性引用链中的上一层，或者说是最后一层的调用位置。
第二是不管是函数别名去引用一个函数地址还是参数传递到一个函数内部（将一个函数以参数的形式进行传递），这种就是被隐式绑定的函数会丢失绑定对象。最终看在哪里调用，this 就指向那个调用的上下文。如下

```js
function foo() {
  console.log(this.a)
}
var obj = {
  a: 2,
  foo: foo
}
var bar = obj.foo //函数别名
var a = 'opps, global'
bar() // 'opps global'--丢失了绑定对象。
```

下面的代码也一样

```js
function foo() {
  console.log(this.a)
}
function doFoo(fn) {
  // fn其实引用的是foo
  fn() // 调用位置
}
var a = 'opps, global'
doFoo(obj.foo) // 'opps global'--丢失了绑定对象。
```

---

### bind 函数：

foo.bind(obj)就是将指定的参数这里就是 obj，为 foo 执行的上下文。this 是指向 obj 的，他其实就是跟 apply 或者 call 方法一样用于绑定，只不过他是在函数的原型上添加了这个方法。Function.prototype.bind；

---

### new 关键字

使用 new 关键字来调用函数，会自动执行下面的操作：
第一、创建（或者说构造）一个全新的对象。
第二、这个新对象会被执行[[Prototype]]连接
第三、这个新对象会绑定到函数调用的 this
第四、如果函数没有返回其他对象，那么 new 表达式中的函数调用会自动返回这个新对象。
如下面代码：

```js
 function foo(a){
   this.a = a
 }
 var bar = new foo(2)
 console.log(bar.a)//2
 使用new来调用foo时候，我们会构造一个新对象并把它绑定到foo()调用中的this上。而bar就是执行这个新对象。
```

---

### this 的绑定：

    他有四种方式进行绑定：默认绑定、隐式绑定、显示绑定、new绑定。
    new绑定会修改this的指向，比如new的对象是一个硬绑定（foo.bind()）这样，就会修改bind中绑定的this指向的对象。然后会创建一个新的对象指向实例化的对象。

---

### 判断 this：

如果要判断一个运行中函数的 this，就需要找到这个函数的直接调用位置，找到之后就可以顺序应用下面这四条规则来判断 this 的绑定对象。
第一：函数是否在 new 中调用（new 绑定）？如果是的话，this 绑定的是新创建的对象。

```js
var bar = new foo()
```

第二：函数是否通过 call、apply（显示绑定）或者硬绑定（Function.prototype.bind）调用？如果是的话，this 绑定的是那个上下文对象。

```js
  var bar = foo.call(obj2)
第三：函数是否在某个上下文对象中调用（隐式绑定）？如果是的话，this绑定的是那个上下文对象。
```

```js
var bar = obj1.foo()
```

第四：如果都不是的话，使用默认绑定。如果在严格模式下，就绑定到 undefined，否则绑定到全局对象。

````
```js
  var bar = foo()
````

---

### 绑定的例外：

如果我们将 null 或者 undefined 的绑定对象传入 call、apply、或者 bind，这些值在调用的时候会被忽略，实际应用的是默认绑定的规则，就是绑定到全局变量。
例如下面的代码：

```js
function foo(a, b) {
  console.log('a:' + a + 'b:' + b)
}
// 把数组展开成参数
foo.apply(null, [2, 3]) //a:2,b:3

// 使用bind进行柯里化
var bar = foo.bind(null, 2)
bar(3) //a:2,b:3
```

不过我们有时候不需要传入一个有意义的对象去指定 this 的绑定，我们才会传入 null 或者 undefined，为了避免传入 null 或者 undefined 导致 bug，我们会使用 Object.create 创建一个空对象，去使用；比如下面的代码：

```js
function foo(a, b) {
  console.log('a:' + a + 'b:' + b)
}
// 我们创建一个空对象
var nullObj = Object.create(null)
// 把数组展开成参数
foo.apply(nullObj, [2, 3]) //a:2,b:3

// 使用bind进行柯里化
var bar = foo.bind(nullObj, 2)
bar(3) //a:2,b:3
```

---

### 间接引用

如下面代码：

```js
function foo() {
  console.log(this.a)
}
var a = 2
var o = { a: 3, foo: foo }
var p = { a: 4 }
o.foo()(
  //3
  (p.foo = o.foo)
)() //2
```

赋值表达式 p.foo = o.foo 的返回值是目标函数的引用，因此调用位置是 foo()而不是 p.foo()或者 o.foo(),所以使用的是默认绑定。
注意：对于默认绑定，决定 this 绑定对象的并不是调用位置是否处于严格模式，而是函数体是都处于严格模式。如果函数体处于严格模式，this 会绑定到 undefined，否则 this 会被绑定到全局对象。

---

### 箭头函数

箭头函数不是使用 this 的四种标准规则，而是根据外层（函数或者全局）作用域来决定 this 的，也就是说根据当前的词法作用域来决定的，具体来说，箭头函数会继承外层函数调用的 this 绑定（这个其实就是跟之前我们写的\_this = this 机制是一样的）。
例如下面代码：

```js
function foo() {
  // 返回一个箭头函数
  return a => {
    // this继承自foo
    console.log(this.a)
  }
}
var obj1 = {
  a: 2
}
var obj2 = {
  a: 3
}
var bar = foo.call(obj1)
bar.call(obj2) // 2而不是3
```

foo()内部创建的箭头函数会捕获调用时 foo()的 this。由于 foo()的 this 绑定到 obj1，bar（引用箭头函数）的 this 也会绑定到 obj1，箭头函数的绑定无法被修改。(new 也是不行)

我们经常在写函数的时候为了绑定 this 会这样:var \_this = this 其实当用到了这样的语句的时候，我们可以尝试使用下面的方法进行修改：

1. 只使用词法作用域并完全抛弃错误 this 风格的代码；（箭头函数）
2. 完全采用 this 风格，在必要的时候使用 bind 方法，尽量避免使用\_this = this 和箭头函数。
