# 函数柯里化(Currying in JavaScript)

## 预备知识

想要更好地了解函数柯里化，首先我们必须先对[函数式编程 & 高阶函数](https://github.com/rookielzy/daily-blog/blob/68c25aff3b7c2fc58e43dff69d2f0eeec8e3ba54/About-JavaScript/functional-programming.md)有个基本的概念。

## 什么是函数柯里化

Wikipedia 中的定义：
> 在计算机科学中，柯里化（英语：Currying），又译为卡瑞化或加里化，是把接受多个参数的函数变换成接受一个单一参数（最初函数的第一个参数）的函数，并且返回接受余下的参数而且返回结果的新函数的技术。这个技术由克里斯托弗·斯特雷奇以逻辑学家哈斯凯尔·加里命名的，尽管它是Moses Schönfinkel和戈特洛布·弗雷格发明的。

简单来说就是，柯里化接受一个带多参数的函数，然后让它返回一个函数去处理剩下的参数。

首先我们先来看一个简单的例子：

```js
function multiply(a, b, c) {
  return a * b * c
}
```

上述函数接受三个参数，然后返回三个参数的乘积。

```js
multiply(1, 2, 3) // 6
```

上述都是简单的正常调用函数的过程，接下来我们来把它柯里化：

```js
function multiply(a) {
  return (b) => {
    return (c) => {
      return a * b * c
    }
  }
}

multiply(1)(2)(3) // 6
```

这里我们将 `multiply(1, 2, 3)` 转化成功了 `multiply(1)(2)(3)`，简单点来说就是将一次函数调用转化成了多次的函数调用。

看到这里，你可能会问为什么要这么大费周章地将这个函数变得这么复杂，这个问题我先不回答，继续看下去，你就会理解其中的原理。

我们先来拆解上述代码：

```js
const mul1 = multiply(1)
const mul2 = mul1(2)
const mul3 = mul2(3)  // 6

// 等同于
const mul1 = function (b) {
  return (c) => {
    return 1 * b * c
  }
}

const mul2 = function (c) {
  return 1 * 2 * c
}

const mul3 = function (3) {
  return 1 * 2 * 3
}
```

可以看到，每次我们都传入一个参数，如果传入的参数不是最后一个参数，那么以函数作为返回值返回。值得注意的是，这里我们利用了闭包的特性。

这种写法到底有什么优点呢？且继续往下看。

## 函数柯里化的好处

先说结论：函数柯里化最大的好处就是它可以让我们更好地组织代码结构，利用纯函数、高阶函数等函数式编程的概念的优点，使我们的代码更易于维护，重构，复用性更强。

为什么会有这样的结论呢？

我们再以一个简单的例子来解释：

```js
// 我们来写一个计算矩形面积的方法
// 这个函数很简单，只需要接受两个参数： 长度、
function getRectangularArea (length, width) {
  return length * width
}

// 柯里化这个函数
function getRectangularArea (length) {
  return (width) => {
    return length * width
  }
}

// 关键的地方来了
// 假设现在我们在做一个数学题，已知条件有一些矩形，它的长度固定为 10
// 这时候我们可以声明一个函数变量 （请忽略这垃圾变量命名）
const length10 = getRectangularArea(10)

// 现在我想要分别求宽度为5 和 10 的矩形
const width5Result = length10(5)  // 50
const width10Result = length10(10)  // 100
```

从上述的例子中，我们可以发现 `length10` 是可以重复使用的，并且不会有任何副作用，并且我们可以很明确地知道它的作用是什么。这也就体现了开始说到的函数柯里化的优点。

## 如何生成柯里化的函数

说了这么多，上述例子我们都还需要手动来组合函数，以达到柯里化的目的，那如何更快地实现函数的柯里化呢？

其实也很简单，就是将函数柯里化的概念转化为代码就行了。（接受一个带多参数的函数，然后让它返回一个函数去处理剩下的参数）

```js
function curry(fn, ...args) {
  return (..._arg) => {
    return fn(...args, ..._arg)
  }
}
```

上述的代码到底做了什么事呢？首先我们接受了一个函数参数，这个函数就是我们需要柯里化的目标函数；还接受了一系列的其他参数（...args）。主题这里使用了ES6中的 `rest参数（剩余参数）`

然后我们返回一个也将剩余参数(`..._arg`)作为形参的函数。这个函数将调用目标函数，并把 `...args` 和 `..._arg` 作为形参传给目标函数。最后返回给我们。

我们来简单验证一下是否正确：

```js
function volume(l, h, w) {
  return l * h * w
}

const hCy = curry(volume, 100)

hCy(200, 900)
```

注意，这里还没有引入组合函数的概念，所以我们无法链式调用函数。组合函数将会在后面进行解析。

当然这里只是简单地介绍了函数柯里化的基本概念，真正的开发过程中我们可以发现或多或少能够柯里化的地方，这也是要求我们首先要有这个概念，才能写出更好的代码。
