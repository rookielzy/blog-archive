# 执行上下文(Execution Context) 

# 当执行 JavaScript 代码时发生了什么 ？

> 首先我们先要明白一个概念 - 执行上下文

## 什么是执行上下文？

简而言之，执行上下文就是 JavaScript 代码所存在和执行的环境的抽象概念。每当在 JavaScript 中运行任何代码时，它都在执行上下文中运行。

### 执行上下文的种类

执行上下文分为以下三种

* ___全局执行上下文(Global Execution Context) —___ 代码首次执行的默认环境。如果代码没在任何一个函数内那它就在全局执行上下文环境中。这里表明了两点：一、它创建了一个全局变量，即 `window` 对象（在浏览器中）或 `module` 对象（在 `node` 环境下），并将 `this` 的值赋予为全局变量。二、在程序中只能存在一个全局变量。

* ___函数执行上下文(Functional Execution Context) —___ 每当一个函数被调用时，就会为这个函数创建一个全新的执行上下文。每个函数只有在被调用时才会它的执行上下文。无论一个新的执行上下文何时被创建，它都会按定义好了的顺序执行一系列步骤，这个在后面会讨论。

* ___Eval 执行上下文(Eval Function Execution Context) —___ 在 `eval` 中执行的代码也会拥有它自己的执行上下文，但我们不提倡使用 `eval` 因为它会使代码变得难以预测。

## 执行上下文堆栈(Execution Stack)

在其他编程语言中执行上下文堆栈也被称为“调用堆栈”，它是一种后进先出（`LIFO Last in, First out`）的数据结构，用于存储所有在代码执行期间生成的执行上下文。

当JavaScript引擎在初次执行代码时，它会创建一个全局执行上下文，并将其压进当前的执行上下文堆栈中。无论JS引擎何时发现调用函数，它会为这个函数创建一个新的执行上下文，并将其压进执行上下文堆栈上。

JS引擎会先执行在执行上下文堆栈顶部的函数执行上下文，当这个函数执行完毕，它的执行上下文会被弹出堆栈的顶部，JS引擎将继续执行在刚才的函数执行上下文下方的执行上下文。

让我们用代码例子来解释一下：

```js
// 1. 函数执行 - 创建全局执行上下文
let a = 'Hello World!';
function first() {
  console.log('Inside first function');
  second(); // 3. 遇到调用函数 - 创建 second 的执行上下文
  console.log('Again inside first function');
  // 5. first 执行完毕，弹出 first 的执行上下文
}
function second() {
  console.log('Inside second function');
  // 4. second 执行完毕，弹出 second 的执行上下文
}
first();  // 2. 遇到调用函数 - 创建 first 的执行上下文
console.log('Inside Global Execution Context');
```

![execution context](./images/execution-context.png)

## 执行上下文是如何创建的

执行上下文的创建分为两个阶段：
1. 创建阶段(creation phase)
2. 执行阶段(execution phase)

### 创建阶段(Creation Phase)

在JS代码执行之前，执行上下文将经历三个创建阶段：

1. 确定 `this`，也就是绑定 `this` 值（This Binding）。
2. 创建词法环境（LexicalEnvironment）
3. 创建变量环境（VariableEnvironment）

用伪码可表示为：

```js
ExecutinContext = {
  ThisBinding = <this value>,
  LexicalEnvironment = { ... },
  VariableEnviroment = { ... }
}
```

#### This Binding

在全局执行上下文中，`this` 指向 `window` 对象（在浏览器中）或 `module` 对象（在 `node` 环境下）。

在函数执行上下文中，`this` 的值取决与函数是如何被调用的。如果函数由一个对象的引用所调用，那么 `this` 将指向这个对象；否则 `this` 将指向全局变量或者 `undefined` (在严格模式下)。例如：

```js
let person = {
  name: 'peter',
  birthYear: 1994,
  calcAge: function () {
    console.log(2018 - this.birthYear);
  }
}

person.calcAge();
// this 指向 person。因为 `calcAge 由 person 这个对象所调用。

let calculateAge = person.calcAge;
calculageAge();
// this 指向 全局变量，因为它没有给出指向的对象。
```

#### Lexical Environment

在ES6的文档中，词法环境的定义为：

> A Lexical Environment is a specification type used to define the association of Identifiers to specific variables and functions based upon the lexical nesting structure of ECMAScript code. A Lexical Environment consists of an Environment Record and a possibly null reference to an outer Lexical Environment.

> 词法环境是一种规范类型，用于定义标识符(Identifier)与特定的关联，基于ECMAScript代码的词法嵌套结构的变量和函数。 词法环境由环境记录(Environment Record)和可能为null引用外部词法环境组成。 

简而言之就是词法环境是一种包含标识符与变量之间的映射结构。（这里的标识符指向变量/函数的名字，其中变量指向真正的对象[包括函数类型的对象]或基本类型）。

在词法环境中有两个部分：（1）环境记录(environment record) (2) 外部词法环境引用

1. 环境记录记录在其相关词法环境范围内创建的标识符绑定。
2. 外部词法环境引用表示它可以访问外部的词法环境。

词法环境有两种类型：

* （在全局执行上下文中）没有外部词法环境的 ___全局环境(global environment)___ 。全局环境的外部词法环境引用为 `null`，它包含一个在环境记录中的全局对象与其相关联的方法和属性（譬如数组方法）以及任何用户定义的全局变量，其中 `this` 指向这个全局对象。

* ___函数环境(function environment)___ 。函数内的用户定义的变量存储在环境记录中即为函数环境。它的外部词法环境引用可以为全局环境，或者一个包含这个函数的外部函数环境。

注意，对于函数环境，环境记录同样包含了一个 `arguments` 的对象(非数组)，这个对象包含索引和传递给函数的参数之间的映射以及传递给函数的参数的长度（数量）。譬如：

```js
function foo (a, b) {
  var c = a + b;
}
foo(2, 3);

// argument object
Arguments: { 0: 2, 1: 3, length: 2 }
```

对于环境变量，它也有两种类型：

* ___声明性环境记录
(Declarative environment record)___ 包含了变量，函数和参数等等。函数环境包含声明性环境记录。

* ___对象环境记录(Object environment record)___ 用于定义在全局执行上下文中出现的变量和函数之间的联系。全局环境包含对象环境记录。

抽象上来说，就如以下伪码：

```js
GlobalExecutionContext = {
  LexicalEnvironment: {
    EnvironmentRecord: {
      Type: "Object",
      // Identifier bindings go here
    }
    outer: <null>
  }
}

FunctionExecutionContext = {
  LexicalEnvironment: {
    EnvironmentRecord: {
      Type: "Declarative",
      // Identifier bindings go here
    },
    outer: <Global or outer function environment reference>
  }
}
```

#### 变量环境(Variable Environement)

变量环境用于标识其环境记录（EnvironmentRecord)保存由VariableStatements在此执行上下文中创建的绑定的词法环境。

如上所述，变量环境也是一个词法环境，因此它具有上面定义的词法环境的所有属性。

在ES6中，词法环境（LexicalEnvironment）与变量环境（VariableEnvironment）的一个不同点就是前者用于存储函数声明和变量（let、const）绑定，而后者仅仅用于存储变量（var)绑定。

我们来看一个例子来理解上述概念：

```js
let a = 20;
const b = 30;
var c;

function multiply(e, f) {
  var g = 20;
  return e * f * g;
}

c = multiply(20, 30)
```

它的执行上下文将会是这样：

```js
GlobalExecutionContext = {

  ThisBinding: <Global Object>,

  LexicalEnvironment: {
    EnviromentRecord: {
      Type: "Object",
      // Identifier bindings go here
      a: < uninitialized >,
      b: < uninitialized >,
      multiply: < func >
    }
    outer: <null>
  },

  VariableEnvironment: {
    EnviromentRecord: {
      Type: "Object",
      // Identifier bindings go here
      c: undefined,
    }
    outer: <null>
  }

}

FunctionExecutioncontext = {

  ThisBinding: <Global Object>,

  LexcialEnvironment: {
    EnvironmentRecord: {
      Type: "Declarative",
      // Identifier bindings go here
      Arguments: {0: 20, 1: 30, length: 2},
    },
    outer: <GlobalLexicalEnvironment>
  },

  VariableEnvironment: {
    EnvironmentRecord: {
      Type: "Declarative",
      // Identifier bindings go here
      g: undefined
    },
    outerL <GlobalLexicalEnvironment>
  }

}
```

注意，上述函数环境只有在 `multiply` 被调用时才会被创建。

你可以注意到其中定义了 `let` 和 `const` 变量，并且没关联任何值；但 定义了 `var` 并赋予 `undefined` 。

这是因为在创建阶段(creation phase)，JS引擎扫描代码中的变量和函数声明，其中函数声明将会完全存放在对应的环境记录（EnvironmentRecord）中，但 `var` 声明的变量将被初始化为 `undefined` 而 `let` 和 `const` 将不回被初始化。

这就是为什么你可以在变量声明前访问变量（`var`），虽然得到的值为 `undefined` ，但你在使用了`let` 或 `const` 声明变量前访问变量会报错（reference error）。

这就是变量提升。

### 执行阶段(Execution Phase)

这个阶段就简单多了。当所有变量都执行完毕，也就意味着代码执行完毕了。

注意，在执行阶段，如果JS引擎无法找到 `let` 声明的变量的值，那么它将会被赋予 `undefined`。
