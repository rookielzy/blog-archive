# 作用域(Scope)

## 什么是作用域(What is Scope)

在ECMA-262中是没有关于作用域的相关描述的，不过它确实存在与JavaScript程序中。根据[w3c school](https://www.w3schools.com/js/js_scope.asp)里的定义，我们可以知道作用域决定了你可以访问的变量。

> Scope determines the accessibility (visibility) of variables.

> 作用域决定变量的可访问性。

在JavaScript中有两种作用域：

* 局部作用域
* 全局作用域

## 全局作用域(Global Scope)

如果不在任何函数内或在`{}`中不使用 `let` 或 `const` 去定义一个变量，那么这个变量将存在与全局作用域中。

```js
var a = 'a' // 在全局作用域中
function foo () {
  console.log('foo')
}


console.log(a)  // a
foo() // foo

{
  const a = 'b' // 在局部作用域中
}

console.log(a)  // b

{
  var a = 'c' // 在全局作用域中
  function foo () { // 注意这里的变量不提升
    console.log('bar')
  }
}

console.log(a)  // c
```

## 局部作用域(Local Scope)

代码中有些变量只能在特定的范围才能访问，这个范围就称为局部作用域。这些变量也称为局部变量。

在JavaScript中有两种局部变量：

* 函数作用域
* 块作用域

### 函数作用域(Function scope)

当你在一个函数里声明一个变量时，你只能在这个函数里访问它，无法在外部去访问这个变量。这就是函数作用域。

```js
function helloWorld () {
  const greeting = 'Hello World'
  console.log(greeting)
}

helloWorld()  // Hello World
console.log(greeting)  // Error: greeting is not defined
```

### 块作用域

当你在 `{}` 中使用 `let` 或 `const` 定义变量时，你只能在这个花括号里访问它。这也就形成了块作用域。

```js
{
  const a = 'a'
  console.log(a)  // a
}

console.log(a)  // Error: a is not defined
```

实际上，块作用域是函数作用域的子集，因为声明函数需要花括号来声明（除了箭头函数的隐式声明）

## 嵌套作用域

当在一个函数里声明另一个函数时，内部的函数将能访问外部函数的变量。这种行为叫做词法作用域(lexical scoping)。词法作用域也是组成闭包的重要部分。

## 总结

作用域是JavaScript中的一大重要概念，也是我们接下来去理解闭包原理的重要基础。
