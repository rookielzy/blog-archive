# 变量提升(Variable Hoisting)

> Hoisting is JavaScript's default behavior of moving declarations to the top. --- from [w3c school](https://www.w3schools.com/js/js_hoisting.asp)


我们都知道JS中有两种提升(hoisting)，变量提升(variable hoisting)和函数提升(function hoisting)。本文将对变量提升进行深入探讨。

## 什么是变量提升

变量提升就是JS引擎会将所有变量声明提升到作用域的顶部，譬如 `var`、`const`、`let`之类的。等等，不是说`const` 和 `let` 是没有变量提升的效果的吗？其实并不是。在ECMA-262/6.0中有这么两段描述：

> let and const declarations define variables that are scoped to the running execution context’s LexicalEnvironment. The variables are created when their containing Lexical Environment is instantiated but may not be accessed in any way until the variable’s LexicalBinding is evaluated. A variable defined by a LexicalBinding with an Initializer is assigned the value of its Initializer’s AssignmentExpression when the LexicalBinding is evaluated, not when the variable is created. If a LexicalBinding in a let declaration does not have an Initializer the variable is assigned the value undefined when the LexicalBinding is evaluated.

> let和const声明定义了作用域是正在运行的执行上下文的词法环境的变量。 当它们包含词法环境被实例化时，这些变量被创建，但是在执行变量的词法绑定之前，不能以任何方式被访问。 由词法绑定和初始化程序定义的变量在执行词法绑定时被分配其初始化程序的赋值表达式的值，而不是在创建变量时。 如果let声明中的词法绑定没有初始化程序，则在执行词法绑定时将为该变量赋值undefined。

> NOTE: The environment of with statements cannot contain any lexical declaration so it doesn’t need to be checked for var/let hoisting conflicts.

简而言之，`const` 和 `let` 确实进行了变量提升，但是它们并未被初始化，所以无法在运行它前访问它。这里可以参考[第一章里关于变量环境的内容](https://github.com/rookielzy/daily-blog/blob/c3fca5d666ece1989ae0d6de8447dad0db271857/About-JavaScript/execution-context.md)

其实在代码执行前那几瞬间，JS首先创建了执行上下文（ExecutionContext），其中执行上下文中包含了词法环境（LexicalEnvironment）。这里同样可以参考[第一章里关于词法环境的内容](https://github.com/rookielzy/daily-blog/blob/c3fca5d666ece1989ae0d6de8447dad0db271857/About-JavaScript/execution-context.md)。

下面我们通过一些简单的例子来解释上述概念。

### `var` 的变量提升

```js
console.log(a); // undefined
var a = 3;
```
乍看起来 `a` 应该输出的是 `3` 但为什么得到的是 `undefined` 呢？

这里有个很关键的点：JavaScript只对声明进行提升，而非初始化。在编译期间，JavaScript只会将函数和变量的声明存储在内存中，而不是它们的赋值。

但为什么是 `undefined` 呢？

当JS引擎在编译期间找到一个由 `var` 声明的变量时，它会将这个变量添加进词法环境中，然后初始化它，赋予它 `undefined` ，然后当代码执行到该变量赋值的位置时，才会将变量的值赋予给变量。

它的词法环境看起来就像这样：

```js
// 编译期间
LexicalEnvironment = {
  a: undefined
}

// 执行到变量赋值的位置时
LexicalEnvironment = {
  a: 3
}
```

### `let` 和 `const` 的变量提升

首先举个例子：

```js
console.log(a);
let a = 3;

// 输出： ReferenceError: a is not defined
```

看起来 `let` 和 `const` 声明的变量根本就没有提升嘛。

其实不然，在JavaScript所有的声明（function, var, let, const 和 class）都会被提升，然而 `var` 声明的变量会被初始化为 `undefined` ，但 `let` 和 `const` 声明的对象将保持未初始化。

它们只会在JS引擎运行到它们的词法绑定（赋值）时初始化，这就意味着你无法在代码执行到它们的赋值的位置前访问它们。这也是我们所说的“ ___暂时死区(Temporal Dead Zone)___ ”，一个位于创建变量和初始化之间的的区域，并且在这里面无法访问到对应的变量。

如果JS引擎仍然无法在 `let` 和 `const` 声明的位置找到相应赋值语句，那么 `let` 会被赋予 `undefined` 而 `const` 会得到一个错误。

让我们转化为词法环境的伪码来看看：

```js
// Step 1
let a;  // Step 2
console.log(a); // undefined
a = 5;  // Step 3
```

```js
// Step 1
LexicalEnvironment = {
  a: <uninitialized>
}

// Step 2 无法在此处找到相应的赋值语句
LexicalEnvironment = {
  a: <undefined>
}

// Step 3 重新对 a 赋值
LexicalEnvironment = {
  a: 5
}
```

再看一个简单的例子：

```js
function foo () {
  console.log(a);
}
let a = 20;
foo();  // 20

function foo () {
  console.log(a);
}
foo();  // ReferenceError: a is not defined
let a = 20;
```

### `class` 的变量提升

就像 `let` 和 `const` 一样，`class` 在JavaScript中也会被提升，同样在它们被真正执行前会保持未初始化的状态，也就是说同样受 `暂时死区(Temporal Dead Zone)` 所影响。

```js
let perter = new Person('Peter', 25); // ReferenceError: Person is not defined

console.log(peter);

class Person {
  constructor (name, age) {
    this.name = name;
    this.age = age;
  }
}
```

为了能正常访问类，你必须在使用它前声明它。

```js
class Person {
  constructor (name, age) {
    this.name = name;
    this.age = age;
  }
}

let perter = new Person('Peter', 25);

console.log(peter); // Person { name: 'Peter', age: 25 }
```

再来看看对应的词法环境(LexicalEnvironment)

```js
// 在编译阶段
LexicalEnvironment = {
  Person: <uninitialized>
}

// 执行阶段
LexicalEnvironment = {
  Person: <Person object>
}
```

我们再来看一个关于 `class` 表达式的例子，先说结论。和函数表达式一样，它也不回被提升。（函数的会在后面提到）

```js
// 错误的做法 1
let peter = new Person('Peter', 25);  // ReferenceError: Person is not defined

console.log(peter);

let Person = class {
  constructor (name, age) {
    this.name = name;
    this.age = age;
  }
}

// 错误的做法 2
let peter = new Person('Peter', 25);  // Person is not a constructor ....

console.log(peter);

var Person = class {
  constructor (name, age) {
    this.name = name;
    this.age = age;
  }
}

// 正确的做法应该如下
let Person = class {
  constructor (name, age) {
    this.name = name;
    this.age = age;
  }
}
let peter = new Person('Peter', 25);

console.log(peter); // Person { name: 'Peter', age: 25 }
```

# 函数提升(Function Hoisting)

函数提升和 `var` 的提升大同小异。

```js
helloWorld(); // Hello World

function helloWorld() {
  console.log('Hello World');
}
```

同样，在编译阶段，函数声明被加进内存中，因此我们才能够在函数声明的位置之前访问它。

它对应的词法环境(LexicalEnvironment)如下：

```js
LexicalEnvironement = {
  helloWorld: <func>
}
```

在JS引擎在调用 `helloWorld()` 时，它会在词法环境中找该函数，然后执行它。

## 函数表达式

如上文提到的一样，只有函数声明才会被提升，而函数表达式不会被提升。

```js
// 错误的做法
helloWorld(); // TypeError: helloWorld is not a function

var helloWorld = function () {
  console.log('Hello World');
}

// 正确的做法
var helloWorld = function () {
  console.log('Hello World');
}

helloWorld();
```

# 总结

提升并不是JS引擎将我们的代码移动到作用域的顶部，对提升有清晰的理解将会对我们未来避免一些bug在代码中产生。谨记尽量在作用域的顶部去声明变量，尽量在声明它们时给它们一个初始化的值。
