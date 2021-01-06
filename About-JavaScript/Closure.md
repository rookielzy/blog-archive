# 闭包(Closure)

## 什么是闭包

闭包是函数与该函数的词法作用域的组合。简单来说，闭包可以让我们从内部函数访问外部函数的作用域。在JavaScript中，每当创建函数时，都会创建闭包。

使用闭包很简单，只需要在一个函数内定义另一个函数，然后将内部函数暴露出去，即返回它或将它传给另一个函数。

这个内部函数可以访问外部函数作用域的变量，无论是否返回这个外部函数。

## 使用闭包

闭包通常用于对象属性、属性私有化，这在将程序接口化而非命令化有着重要的作用，在程序的健壮性上也有着重要的意义。

在JavaScript中，闭包是使数据私有化的主要机制。当你使用闭包来实现数据私有化时，哪些被包含的变量只能在外部函数作用域内被访问到，你无法在其他外部作用域访问到这些变量，除了一些在函数作用域范围内定义的特权方法。

```js
const getSecret = (secret) => {
  return {
    get: () => secret
  }
}

// console.log(secret) // secret is not defined
const mySecret = getSecret('hello')
const secret = mySecret.get()
console.log(secret) // hello
```

在上述例子中， `get()` 就是一种在`getSecret()` 作用域范围内的特权方法，它可以访问 `getSecret()` 作用域里的任何变量，譬如例子中的形参 `secret`。

闭包在函数式编程里有着重要的作用，这个可以阅读第三章关于函数式编程的内容。这里就不展开描述。