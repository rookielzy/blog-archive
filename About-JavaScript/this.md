# `this`

JavaScript 中的 `this` 是怎么样的一个存在呢？以及它为什么会存在呢？

先说结论：`this` 是指当调用一个函数时，有着一个指向这个函数当前的执行上下文的引用，这个引用就是 `this`。

## 词法作用域（lexical scope）与动态作用域（dynamic scope）

我们都知道 JavaScript 是没有动态作用域这种概念的，只有词法作用域。那什么是词法作用域、什么是动态作用域呢？

```js
// 词法作用域，也是我们常见的场景
function foo () {
  var bar = 'bar'

  function baz () {
    console.log(bar)
  }

  baz()
}

foo() // bar
```

```js
// 如果 JavaScript 中存在动态作用域
function foo () {
  console.log(bar)  // 引用了 baz 里声明的 bar 变量
}

function baz () {
  var bar = 'bar'
  foo()
}

baz() // 那么这里将会打印 bar，但实际上这种机制是不存在的
```

那我们是否可以模拟动态作用域的效果呢？当然可以，我们只需要通过 `this` 的特性即可模拟，也就是说 JavaScript 的动态作用域就是 `this`。

## `this` 的应用

我们先来看一个最基本的 `this` 绑定的效果

### `this` 的默认行为（隐式绑定行为）

```js
function foo () {
  console.log(this.bar)
}

var bar = 'bar'
var obj1 = {
  bar: 'bar1',
  foo: foo
}
var obj2 = {
  bar: "bar2',
  foo: foo
}

foo() // bar
obj1.foo()  // bar1
obj2.foo()  // bar2
```

首先，为什么在直接调用 `foo()` 时，打印的是 `bar` 呢？我们可以很快的知道在调用 `foo()` 时，我们所处的执行上下文（理解为作用域也可以）为 `Global（全局）`，因此此时的 `this` 指向了 `Global`，并且我们在全局下声明了 `bar` 这个变量且赋予 `"bar"` 值。

那为什么调用 `obj1.foo()` 时，打印的是 `bar1` 呢？因为 `foo()` 是通过 `obj1` 调用的，此时的执行上下文为 `obj1`，即局部执行上下文；那么此时的 `this` 指向了 `obj1`，`obj1`中刚好有 `bar` 属性值，即为 `bar1`。

同理 `obj2.foo()` 执行时，`this` 指向了 `obj2`。

### `this` 的显式绑定

```js
function foo () {
  console.log(this.bar)
}

var bar = 'bar'
var obj1 = {
  bar: 'bar1',
  foo: foo
}

foo() // bar
foo.call(obj1)  // bar1
```

这里就不再解释直接调用 `foo()` 时打印 `bar` 的原因了。我们主要看 `foo.call(obj1)`，它表示了什么意思呢？简单来说就是 `call` 将 `this` 显式地绑定到 `obj1` 上了。 [`call` 相关介绍](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Function/call)

### `this` 的强绑定

```js
function foo () {
  console.log(this.bar)
}

var bar = 'bar'
var obj1 = {
  bar: 'bar1',
  foo: foo
}
var obj2 = {
  bar: 'bar2',
  foo: foo
}

var foo1 = foo
foo = function () {
  foo1.call(obj1)
}

foo() // bar1
foo.call(obj2)  // bar1
```

同样，直接调用 `foo()` 打印 `bar1` 很好理解，因为在 `foo` 内部的 `foo1` 的 `this` 已经强绑定了 `obj1`。那为什么调用 `foo.call(obj2)` 时打印的不是 `bar2` 呢？其实很简单，和前面的原理一样，在 `foo` 内部的 `foo1` 的 `this` 已经强绑定了 `obj1`，所以外部的任何显式绑定都无法改变它。但实际上 `foo` 内部的 `this` 仍然由外部条件所决定。

```js
function foo () {
  console.log(this.bar)
}

var bar = 'bar'
var obj1 = {
  bar: 'bar1',
  foo: foo
}
var obj2 = {
  bar: 'bar2',
  foo: foo
}

var foo1 = foo
foo = function () {
  console.log(this)
  foo1.call(obj1)
}

foo() // this: Window
foo.call(obj2)  // this: obj2
```

除了 `call`，还有 `apply` 以及 `bind`。这里不再详细介绍 `apply` 重点会放在 `bind` 上。[`apply` 相关资料](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Function/apply)

`bind` 方法在ES5+中才支持，也是一种 `this` 强绑定的方法。`bind()` 方法创建一个新的函数，在调用时设置 `this` 关键字为提供的值。并在调用新函数时，将给定参数列表作为原函数的参数序列的前若干项。

借用 MDN 中的例子来解释 `bind` 的原理

```js
var module = {
  x: 42,
  getX: function() {
    return this.x
  }
}

var unboundGetX = module.getX   // 注意这个步骤，JavaScript引擎在执行代码前，先声明变量（即变量提升），然后执行到这里时，才给 unboundGetX 赋值，而 unboundGetX 是存在于 Global（全局）中的，即 Window 下
console.log(unboundGetX())  // undefined  执行 unboundGetX 时，它的 this 将指向 Global，而 Global 中无 x 属性，即返回 undefined

var boundGetX = unboundGetX.bind(module)  // 这里我们使用 bind 强绑定 this 指向 mobule
console.log(boundGetX())  // 42
```

### `this` 中的 `new` 绑定

`new` 运算符创建一个用户定义的对象类型的实例或具有构造函数的内置对象的实例。

```js
function foo () {
  this.bar = 'barfoo'
  console.log(this.bar, this.baz, bar)
}

var bar = 'bar'
var foo1 = new foo()  // barfoo undefined undefined
```

通过 `new` 的概念，我们可以很好理解为什么打印 `this.bar` 时有值，而打印 `this.baz` 和 `bar` 时没有值。因为使用 `new` 时，对应的 `foo1` 的 `this` 直接绑定到自身，而不再是 `Global` 或其他。而在 `foo1` 中，我们只定义了 `bar` 这个属性值，且通过 `this.bar` 进行访问。

### 箭头函数中的 `this`

实际上箭头函数是不没有 `this` 这个概念的，然而它又是存在的，因为它是继承于箭头函数所处的执行上下文，并且在定义箭头函数时，它的 `this` 指向就已经决定好了。我们改一下刚才的例子来理解理解。

```js
var x = 24

var module = {
  x: 42,
  getX: () => {
    return this.x
  }
}

var unboundGetX = module.getX
console.log(unboundGetX())  // 24

var boundGetX = unboundGetX.bind(module)
console.log(boundGetX())  // 24

console.log(unboundGetX.call(module)) // 24
```

快速分析一下，`module` 中的 `getX` 为箭头函数，且 `module` 所处的执行上下文为 `Global`，而 `Global` 中有属性 `x` 为 `24`。实际上箭头函数就是隐式的强绑定，其他的任何绑定都无法改变它的行为。

再来看一个常见的例子：

```js
var obj = {
  id: 1,
  foo: function () {
    setTimeout(function (){
      console.log(this.id)
    }, 1000)
  }
}

obj.foo() // undefined

var obj = {
  id: 1,
  foo: function () {
    setTimeout(() => {
      console.log(this.id)
    }, 1000)
  }
}

obj.foo() // 1
```

不使用箭头函数时，在调用 `obj.foo()` 时，JavaScript 的调用栈已清空，经过至少一秒后，执行 `setTimeout` 里的逻辑。这时候执行 `setTimeout` 时所处的执行上下文为 `Global`，全局内无 `id` 这个属性，所以返回 `undefined`。（关于 JavaScript 的调用栈及事件轮询可参考[第十一章 - 事件轮询的内容](https://github.com/rookielzy/daily-blog/blob/master/About-JavaScript/event-loop.md)）

使用了箭头函数，在定义 `setTimeout` 里的箭头函数时，它仍处于 `obj` 的局部执行上下文中，所以在经过至少一秒后打印 `1`。

## 总结

`this` 的绑定有很多种方法，每种方法的优先级也都不一样。`this` 让 JavaScript拥有了动态作用域的特性，使得 JavaScript 更加灵活。

`this` 绑定的速记图：

![this](./images/this.png)