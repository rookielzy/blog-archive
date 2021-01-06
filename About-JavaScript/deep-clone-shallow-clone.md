# 深拷贝 & 浅拷贝(Deep Clone & Shallow Clone)

我们都知道赋予一个变量为对象值时，我们实际上是赋予了对象的引用（即对象的内存地址），而不是重新申请内存空间，赋予新的对象。这种时候就可能会造成一些预想不到的结果。在JavaScript中，我们有两种方法来处理这类问题，即深拷贝和浅拷贝。

## 不可靠的拷贝方法

在介绍深拷贝和浅拷贝前，或许我们曾见到过如下对象拷贝方法：

```js
function copy(mainObj) {
  let objCopy = {}  // objCopy will store a copy of the mainObj
  let key

  for (key in mainObj) {
    objCopy[key] = mainObj[key] // copies each property to the objCopy object
  }
  return objCopy
}

const mainObj = {
  a: 2,
  b: 5,
  c: {
    x: 7,
    y: 4
  }
}

console.log(copy(mainObj))
```

上述方法存在着什么问题呢？

1. `objCopy` 将会有一个新的 `Object.prototype` 方法，这并不是我们想要的，我们想要它与 `mainObj` 共享同一个原型。（I Dont Understand）
2. 属性描述符将不会被拷贝。`mainObj` 中若有属性的 `writable` 属性描述符为 `false`，对应 `objCopy` 的属性的 `writable` 属性描述符将会变为 `true`
3. 上述代码只能拷贝可枚举的属性
4. 如果属性为对象，那么 `objCopy` 将会共享这个对象属性，即指向同一个对象的内存地址

## 浅拷贝

浅拷贝的定义很简单，假设一个拷贝目标 `a`，拷贝结果 `b`。当 `b` 拷贝 `a` 时，`a` 里的基本类型属性将正常进行拷贝，即 `b` 对应的基本类型属性发生变化时，`a` 对应的属性将不会发生改变；而 `a` 里的对象类型属性被拷贝时，`b` 对应的对象类型属性将与 `a` 指向同一个内存地址，即 `b` 或 `a` 的对象类型属性发生改变时，都将会造成对方的属性发生相同的变化。

简单来说，就是对于基本类型，浅拷贝就是对值得拷贝；对于对象类型来说，浅拷贝就是对对象类型的内存 地址的拷贝。

### `Object.assign()`

在开发的工程中，我们不时会用到 `Object.assign()` 来拷贝对象。有些人会认为它是用于深拷贝的，其实并不是。

```js
let obj = {
  a: 1,
  b: 2,
}
let objCopy = Object.assign({}, obj)

console.log(objCopy) // result - { a: 1, b: 2 }
objCopy.b = 89
console.log(objCopy) // result - { a: 1, b: 89 }
console.log(obj) // result - { a: 1, b: 2 }
```

```js
let obj = {
  a: 1,
  b: {
    c: 2,
  },
}
let newObj = Object.assign({}, obj)
console.log(newObj) // { a: 1, b: { c: 2} }

obj.a = 10
console.log(obj) // { a: 10, b: { c: 2} }
console.log(newObj) // { a: 1, b: { c: 2} }

newObj.a = 20
console.log(obj) // { a: 10, b: { c: 2} }
console.log(newObj) // { a: 20, b: { c: 2} }

newObj.b.c = 30
console.log(obj) // { a: 10, b: { c: 30} }
console.log(newObj) // { a: 20, b: { c: 30} }
```

`Object.assign()` 无法做到真正的深拷贝之外，它还无法拷贝非枚举属性。

```js
let obj = {
  a: 'a'
}

Object.defineProperty(obj, 'b', {
  enumerable: false,
  value: 'b'
})

console.log(Object.assign(obj, {})) // {a: 'a'}
```

### 展开运算符 `...`

在学习了ES6之后，我们可以知道还可以通过展开运算符 `...` 来实现浅拷贝，同样要注意，它也无法拷贝非枚举属性。

```js
let obj = {
  a: 'a'
}

Object.defineProperty(obj, 'b', {
  enumerable: false,
  value: 'b'
})

let copy = {...obj}
copy.a = 'A'
console.log(copy) // {a: 'A'}
```

`Object.assign()` 循环引用对象

```js
let obj = { 
  a: 'a',
  b: { 
    c: 'c',
    d: 'd',
  },
}

obj.c = obj.b
obj.e = obj.a
obj.b.c = obj.c
obj.b.d = obj.b
obj.b.e = obj.b.c

let newObj2 = Object.assign({}, obj)

console.log(newObj2) 
```

总结：浅拷贝只能为我们解决对象第一层为基本类型的属性值的拷贝，如果存在有对象类型的属性值时，我们就必须使用深拷贝了。

## 深拷贝

深拷贝与浅拷贝的区别在于，无论拷贝的属性类型是为基本类型还是对象类型，拷贝的结果都会拥有全新的内存地址，即拷贝结果修改属性值不再会影响被拷贝对象的属性值。

### `JSON.parse(JSON.stringify(object))`

很多时候，我们可以使用 `JSON.parse(JSON.stringify(object))` 来解决深拷贝的需求。

```js
let obj = {
  a: 1,
  b: {
    c: 2
  }
}

let newObj = JSON.parse(JSON(stringify(obj)))

obj.b.c = 20
console.log(obj)  // {a:1, b: {c: 20}}
console.log(newObj) // {a: 1, b: {c: 2}}
```

不过这种方法也有局限性：
* 会忽略 `undefined`
* 会忽略 `symbol`
* 不能序列化函数
* 不能解决循环引用的对象

```js
let obj = {
  name: 'abc',
  symbol: Symbol('abc'),
  age: undefined,
  sex: null,
  greet: function () {
    console.log('hello')
  },
  a: 'a',
  b: {
    c: 'c',
    d: 'd'
  }
}

let newObj = JSON.parse(JSON.stringify(obj))

console.log(newObj) // {a: 'a', b: {c: 'c', d: 'd'}, name: 'abc', sex: null}
```

测试循环引用：

```js
obj.c = obj.b
obj.e = obj.a
obj.b.c = obj.c
obj.b.d = obj.b
obj.b.e = obj.b.c

// Uncaught TypeError: Converting circular structure to JSON
//    at JSON.stringify (<anonymous>)
```

可以发现循环引用对象将会抛出错误。可以看出如果不是复杂数据的话，`JSON.parse(JSON.stringify(obj))` 已经可以满足我们大部分需求了。

如果需要真正的实现一个深拷贝，需要考虑的边界条件会变得特别多，譬如原型链、DOM处理等等。这里就推荐阅读 [lodash的深拷贝函数](https://lodash.com/docs/4.17.11#cloneDeep)，我们就实现一个简单的深拷贝方法来理解理解。

```js
function deepClone(obj) {
  function isObject(o) {
    return typeof o === 'object' && o !== null
  }

  function isFunction(o) {
    return typeof o === 'function'
  }

  if (!isObject(obj)) {
    throw new Error('非对象')
  }

  let isArray = Array.isArray(obj)
  let newObj = isArray ? [...obj] : { ...obj }
  Reflect.ownKeys(newObj).forEach(key => {
    newObj[key] = isFunction(obj[key]) ? new Function('return ' + obj[key].toString())() : (isObject(obj[key]) ? deepClone(obj[key]) : obj[key])
  })

  return newObj
}


let obj = {
  name: 'abc',
  symbol: Symbol('abc'),
  age: undefined,
  sex: null,
  greet: function () {
    console.log('hello')
  },
  a: 'a',
  b: {
    c: 'c',
    d: 'd'
  }
}

let newObj = deepClone(obj)
newObj.b.c = 'C'
newObj.greet()
```

> [deepClone参考](https://smalldata.tech/blog/2018/11/01/copying-objects-in-javascript)，缺点只能复制可枚举的属性(for in loop)。

```js
function deepClone(obj) {
  var copy;

  // Handle the 3 simple types, and null or undefined
  if (null == obj || "object" != typeof obj) return obj;

  // Handle Date
  if (obj instanceof Date) {
    copy = new Date();
    copy.setTime(obj.getTime());
    return copy;
  }

  // Handle Array
  if (obj instanceof Array) {
    copy = [];
    for (var i = 0, len = obj.length; i < len; i++) {
        copy[i] = deepClone(obj[i]);
    }
    return copy;
  }

  // Handle Function
  if (obj instanceof Function) {
    copy = function() {
      return obj.apply(this, arguments);
    }
    return copy;
  }

  // Handle Object
  if (obj instanceof Object) {
      copy = {};
      for (var attr in obj) {
        if (obj.hasOwnProperty(attr)) copy[attr] = deepClone(obj[attr]);
      }
      return copy;
  }

  throw new Error("Unable to copy obj as type isn't supported " + obj.constructor.name);
}


let obj = {
  name: 'abc',
  symbol: Symbol('abc'),
  age: undefined,
  sex: null,
  greet: function () {
    console.log('hello')
  },
  a: 'a',
  b: {
    c: 'c',
    d: 'd'
  }
}

let newObj = deepClone(obj)
newObj.b.c = 'C'
newObj.greet()
```
