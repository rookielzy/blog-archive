# 组合函数(Functional Composition)

## 什么是组合函数

组合函数其实是数学上的一种概念，允许你将两个或多个函数组合成一个新函数。

如果你之前有了解过组合函数，或许你看到过如下代码

```js
const add = x => y => x + y
const multiply = x => y => x * y
const add2Muliply3 = compose(multiply(3), add(2))
```

看着挺绕的是吧？其实上述函数等价于数学中：

```js
f(x) = (x + 2) * 3
```

这就是一个很简单的组合函数的例子。现在的关键就是我们如何在JavaScript中将函数组合起来呢？

让我们先回想一下组合函数的概念，组合函数的关键点就是需要可组合的函数。那什么是可组合的函数呢？一个可组合的函数应该有一个输入参数和一个输出值。听起来很奇怪是吧?其实我们可以将任何函数进行柯里化之后，这个函数就是可组合的了。如果你对函数柯里化的概念不够清晰，你可以阅读上一章的内容。

## 组合函数的简单例子

接下来我们以代码来解释组合函数的概念。我们将用关于 `html` 标签转义方法来做演示。

```js
const tag = t => contents => `<${t}>${contents}</${t}>`
tag('b')('this is bold')

// <b>this is bold</b>
```

上述代码很简单，我们给出一个我们想要的标签，然后输出转义后的字符串。

现在我想要添加标签属性的处理方法，譬如 `<div class="title">...</div>`

```js
const encodeAttribute = (x = '') => {
  x.replace(/"/g, '$quot;')
}

const toAttributeString = (x = {}) =>
  Object.keys(x)
    .map(attr => `${encodeAttribute(attr)}="${encodeAttribute(x[attr])}"`)
    .join(' ')

const tagAttributes = x => c => 
  `<${x.tag}${x.attr ? ' ' : ''}${toAttributeString(x.attr)}>${c}</${x.tag}>`
```

现在我们重构一下 `tag`

```js
const tag = x =>
  typeof x === 'string'
    ? tagAttributes({ tag: x })
    : tagAttributes(x)
```

我们来试试重构的 `tag`

```js
const bold = tag('b')

bold('this is bold!')
// <b>this is bold!</b>

tag('b')('this is bold!')
// <b>this is bold!</b>

tag({ tag: 'div', attr: { 'class': 'title' }})('this is a title!')
// <div class="title">this is a title!</div>
```

## 组合函数在实际开发中的应用

上述的例子都比较简单，现在我们来看看实际开发中可能遇到的情况。

假设现在我们需要构造如下的列表：

```html
<ul class="list-group">
  <li class="list-group-item">Cras justo odio</li>
  <li class="list-group-item">Dapibus ac facilisis in</li>
  <li class="list-group-item">Morbi leo risus</li>
  <li class="list-group-item">Porta ac consectetur ac</li>
  <li class="list-group-item">Vestibulum at eros</li>
</ul>
```

快速分析一下，我们首先需要能生成 `ul` 的函数、 能生成 `li` 的函数、能循环生成 `li` 的函数。

```js
// 生成 ul
const listGroup = tag({ tag: 'ul', attr: { class: 'list-group' }})

// 生成 li
const listGroupItem = tag({ tag: 'li', attr: { class: 'list-group-item' }})

// 循环生成 li
const listGroupItems = items =>
  items.map(listGroupItem)
    .join('')

listGroup()
// <ul class="list-group"></ul>

listGroupItem('Cras justo')
// <li class="list-group-item">Cras justo</li>

listGroupItems(['Cras justo', 'Dapibus ac'])
// <li class='list-group-item'>Cras justo</li>
// <li class='list-group-item'>Dapibus ac</li>

listGroup(listGroupItems(['Cras justo', 'Dapibus ac']))
// <ul class='list-group'>
//   <li class='list-group-item'>Cras justo</li>
//   <li class='list-group-item'>Dapibus ac</li>
// </ul>
```

上述代码其实还可以优化，`listGroup` 其实看起来挺鸡肋的，我想要这个函数直接给我想要的结果就好了。我们现在来重构一下：

```js
// 将原来的 listGroup 替换为 listGroupTag
const listGroupTag = tag({ tag: 'ul', attr: { class: 'list-group' }})

const listGroup = items =>
  listGroupTag(listGroupItems(items))
```

本文截至为止介绍了组合函数的具体应用，不过我们还没看见关键点 `compose`，所以接下来就是正戏了。

不要以为接下来是正戏就觉得难度很高，相反的是接下来的内容却是最简单的。

我们再来看看之前的 `listGroup`

```js
const listGroup = items =>
  listGroupTag(listGroupItems(items))
```

如果我们将这个函数进行组合：

```js
const listGroup = items =>
  compose(listGroupTag, listGroupItems)(items)
```

为什么会这样组合呢？其实我们从右往左来看，它其实就像一个普通的函数，右边第一个函数执行完之后返回的值传给下一个函数执行。

再往深一层来看，我们的 `compose` 会返回一个函数，这个函数将接收 `items` 作为形参；我们的 `listGroup` 也同样接收 `items` 作为形参，因此我们可以根据组合函数的概念改造一下 `listGroup`，将 `items` 移除。

```js
const listGroup =
  compose(listGroupTag, listGroupItems)
```

希望看到这里，你还没被搞晕。组合函数的优点就在于它让我们明白 ___随着我们的代码量的增长，我们可以写出很多的组合函数___

## `compose` & `pipe`

到底什么是 `compose` 呢？其实前面已经给出答案了。“其实我们从右往左来看，它其实就像一个普通的函数，右边第一个函数执行完之后返回的值传给下一个函数执行。”

转化为代码后它就是这样的：

```js
const compose = (...functions) => data =>
  functions.reduceRight((value, func) => func(value), data)
```

我们现在反过来思考一下，`compose` 是从右往左，那从左往右，像流水线一样的 `pipe` 又是怎么样的呢？其实很简单：

```js
const pipe = (...functions) => data =>
  functions.reduce((value, func) => func(value), data)
```

这里再简单举个 `pipe` 的实际应用场景：

```js
const login = pipe(
  validateInput,
  getAuthToken,
  getUserInfo
)
```

## 总结

组合函数要求我们以可组合的形式来编写函数，也就是你的函数必须是有一个输入和一个输出；有多个输入的函数必须柯里化。组合函数给我们带来的优势也不用多说了，回顾上面的例子，你就会发现它很好地呈现了函数式编程的优点，特别是代码复用。
