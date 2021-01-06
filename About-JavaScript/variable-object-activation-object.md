# 弃用！在ES5 中变量对象/激活对象已被词法环境/变量环境所取代。请查阅第一章关于执行上下文的内容。

# 变量对象(Variable Object)

变量对象是与执行上下文相关联的数据容器。它是一个特殊的对象，在执行上下文中定义的变量和函数声明都存放在变量对象里。注意函数表达式是不会在变量对象里的。

变量对象是一种抽象的概念，在不同的执行上下文中，它通过不同的对象来呈现。譬如，在全局执行上下文中，变量对象就是全局对象它本身，这就是为什么我们可以通过全局对象的属性名来访问全局变量。

```js
var foo = 10;

function bar () {}  // function declaration
(function baz() {}) // function expression

console.log(
  this.foo == foo // true
  window.bar == bar // true
)

console.log(baz)  // ReferenceError, "baz" is not defined
```

上述代码中的全局执行上下文的变量对象(VO)也就有以下属性：

![variable object](./images/variable-object.png)

我们可以看到，由于 `baz` 是函数表达式，所以它不存在与变量对象中。这也就是为什么在这个表达式以外去访问它会得到一个 `ReferenceError` 的错误。

# 激活对象(Activation Object)

当函数被调用时，对应的激活对象就会被创建。它包含了形式参数和特殊参数对象 `arguments` 。在函数上下文中，激活对象和变量对象时一样的。除了变量和函数声明之外，它还存储形参和 `arguments` 对象并调用激活对象。

```js
function foo (x, y) {
  var z = 30
  function bar() {}
  (function baz() {})
}

foo(10, 20)
```

在 `foo` 的函数上下文中，我们有如下激活对象(AO)：

![activation object](./images/activation-object.png)

注意函数表达式 `baz` 并不会在变量/激活对象中。
