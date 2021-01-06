# JavaScript基础知识

## 混乱之治（Rule of Chaos）

### `==` & `===`

先假设变量 `x` 与变量 `y` 作比较。

我们首先来看看 `===`，严格相等，比较的双方必须类型与值都相同才返回 `true`，否则返回 `false`。

需要注意的一点是如果两个值作比较，假设 `x` 为 `Number` 类型

1. 如果 `x` 为 `NaN`，返回 `false`
2. 如果 `y` 为 `NaN`，返回 `false`
3. 如果 `x` 与 `y` 的值相同，返回 `true`
4. 如果 `x` 为 `+0`，`y` 为 `-0`，返回 `true`
5. 如果 `x` 为 `-0`，`y` 为 `+0`，返回 `true`
6. 如果不满足上述条件，返回 `false`

对于 `==` 非严格相等，就有可能发生类型转换。[关于类型转换可以参考这篇文章](https://github.com/rookielzy/daily-blog/blob/c9ae363dbbf8cca63b2b1b9b8d1eb3bddd71bff3/About-JavaScript/basic-js-data-type.md)

接下来我们来看看 `==` 是遵循了什么规则来做判断的：

1. 如果 `x` 和 `y` 的类型相同，那么它们比较返回的结果将和严格相等一样 `x === y`
2. 如果 `x` 为 `null`，`y` 为 `undefined`，返回 `true`
3. 如果 `x` 为 `undefined`，`y` 为 `null`，返回 `true`
4. 如果 `x` 的类型为 `Number`， `y` 的类型为 `String`，返回 `x == ToNumber(y)`（将 `y` 转换为数字类型）的对比结果
5. 如果 `x` 的类型为 `String`， `y` 的类型为 `Number`，返回 `ToNumber(x) == y` 的对比结果
6. 如果 `x` 的类型为 `Boolean`，返回 `ToNumber(x) == y` 的对比结果
7. 如果 `y` 的类型为 `Boolean`，返回 `x == ToNumber(y)` 的对比结果
8. 如果 `x` 属于 `String`，`Number` 或 `Symbol` 中的一种，且 `y` 的类型为 `Object`，返回 `x == ToPrimitive(y)`（将 `y` 转换为原始类型）的对比结果
9. 如果 `x` 的类型为 `Object`，`y` 属于 `String`，`Number` 或 `Symbol` 中的一种，返回 `ToPrimitive(x) == y` 的对比结果
10. 如果不符合上述条件，返回 `false`

[ECMA参考](https://www.ecma-international.org/ecma-262/9.0/index.html#sec-abstract-equality-comparison)

特殊情况

```js
[] == ![] // true
// [] == false
/// "" == false
// 0 == false
// 0 === 0
// true
[] != []  // true
// !([] == [])
// !(false)
// true
```

避免以下情况：

1. 使用 `==` 与 `0` 或 ""，甚至是 " " 作比较
2. 使用 `==` 与非原始类型值作比较
3. `== true` 或 `== false`

谨记如下要求：

1. 不要在不知道值的类型的情况下使用 `==`
2. 不是大部分情况下需要使用 `===`，用 `==` 更简洁


### `NaN` & `isNaN`

`NaN` 并不表示 `not a number` 非数字类型，它表示某个值为非法数字类型，也就是说 `NaN` 其实也是一种数字类型，只不过不合法而已。

`NaN` 是唯一一个自身不相等的值

```js
var test = Number('test') // NaN

test === test // false

isNaN(test) // true 这是为什么呢？
```

为什么 `isNaN(test)` 会返回 `true` 呢？如果我们继续看 `test` 这个变量的值，当然它既不是数字类型，也不是非法数字类型（`NaN`）。 那么问题就出在 `isNaN` 身上了，使用 `isNaN` 时，它会在判断前将传入的值强制转换为数字类型，然后判断是否为 `NaN`


我们有更安全的方法来判断一个值是否真的为 `NaN`，即使用 `Number.isNaN`，它并不会将传入的值强制转换为数字类型，因此可以准确判断一个值是否为 `NaN`。

```js
Number.isNaN('test')  // false
```
