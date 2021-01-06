# 原型(Prototype)

在 JavaScript 中，原型是一个最基础且不可或缺的概念。因为在 JavaScript 中是不存在类的概念的，你可能会说 ES6 中的 `class` 不就是类吗，但其实它只是 JavaScript 实现的一种语法糖而已，我们要实现类似类的行为就必须依靠 `prototype` 原型来处理。

## 从对象 `Object` 出发

对象都有键值对 `key/value`，我们最常见的创建一个对象的方法就是使用大括号 `{}`，然后可以在里面添加属性及方法。

```js
let animal = {}
animal.name = 'Leo'
animal.energy = 10

animal.eat = function (amount) {
  console.log(`${this.name} is eating.`)
  this.energy += amount
}

animal.sleep = function (amount) {
  console.log(`${this.name} is sleeping.`)
  this.energy += amount
}

animal.play = function (length) {
  console.log(`${this.name} is playing.`)
  this.energy -= length
}
```

很简单是吧？现在假设我们需要创建不止一个动物，那就说明我们需要把上述逻辑封装进一个函数里面，也就是我们常见的工厂函数，它负责构造出我们想要的实例，在这里就是不同的动物。

## 工厂函数

```js
function Animal (name, energy) {
  let animal = {}
  animal.name = name
  animal.energy = energy

  animal.eat = function (amount) {
    console.log(`${this.name} is eating.`)
    this.energy += amount
  }

  animal.sleep = function (amount) {
    console.log(`${this.name} is sleeping.`)
    this.energy += amount
  }

  animal.play = function (length) {
    console.log(`${this.name} is playing.`)
    this.energy -= length
  }

  return animal
}

const leo = Animal('Leo', 7)
const snoop = Animal('Snoop', 10)
```

也很简单是吧？如果你阅读过 JavaScript 高级程序设计，你可以马上发现这种方法有个缺点。那就是我们每次创建新的实例（动物）时，都需要重新对实例的方法 (eat，sleep，play) 进行重新赋值，这样会造成内存空间的浪费。我们想要的应该是这些公共方法能共享（指向同一个函数对象，即指向相同的内存地址）

## 工厂函数与共享方法

```js
const animalMethods = {
  eat(amount) {
    console.log(`${this.name} is eating.`)
    this.energy += amount
  },
  sleep(length) {
    console.log(`${this.name} is sleeping.`)
    this.energy += length
  },
  play(length) {
    console.log(`${this.name} is playing.`)
    this.energy -= length
  }
}

function Animal (name, energy) {
  let animal = {}
  animal.name = name
  animal.energy = energy
  animal.eat = animalMethods.eat
  animal.sleep = animalMethods.sleep
  animal.play = animalMethods.play

  return animal
}

const leo = Animal('Leo', 7)
const snoop = Animal('Snoop', 10)
```

上述方法中，我们通过把公共方法放到一个对象中，然后动物的工厂函数里对应的方法指向新的对象的方法，这样也就解决了内存浪费的问题。

## `Object.create`

接下来我们使用 `Object.create` 来重构我们的动物工厂函数。`Object.create` 有什么优点呢？它可以为我们创建一个新的对象，并且将会自动将新创建对象的原型对象作为原型。简单来理解就是如果新创建的对象 `newObj` 访问某个属性 `test`，它可能本身没有这个属性，那它就会去从它的原型上找。我们来看一个例子来理解它：

```js
const parent = {
  name: 'Stacey',
  age: 35,
  heritage: 'Irish'
}

const child = Object.create(parent)
child.name = 'Ryan'
child.age = 7

console.log(child.name) // Ryan
console.log(child.age) // 7
console.log(child.heritage) // Irish
```

上述的例子中，由于 `child` 是通过 `Object.create(parent)` 创建的，因此每当在 `child` 上找不到相应的属性时，JavaScript 会自动从 `parent` 上去找。例子中，`child` 是没有 `heritage` 属性的，`parent` 有。所以你访问 `child.heritage` 时会打印 `Irish`。现在我们正式来重构之前的动物工厂函数。

## 工厂函数与`Object.create`共享方法

```js
const animalMethods = {
  eat(amount) {
    console.log(`${this.name} is eating.`)
    this.energy += amount
  },
  sleep(length) {
    console.log(`${this.name} is sleeping.`)
    this.energy += length
  },
  play(length) {
    console.log(`${this.name} is playing.`)
    this.energy -= length
  }
}

function Animal (name, energy) {
  let animal = Object.create(animalMethods)
  animal.name = name
  animal.energy = energy

  return animal
}

const leo = Animal('Leo', 7)
const snoop = Animal('Snoop', 10)

leo.eat(10)
snoop.play(5)
```

当我们调用 `leo.eat(10)` 时，JavaScript 将会先在 `leo` 上找是否有 `eat` 这个方法，如果没有，由于我们是通过 `Object.create` 创建的 `leo`，那么JavaScript会继续从 `animalMethods` 中查找 `eat` 方法。

其实看到这里，我们上面所描述的东西就与本文的核心概念 `prototype` 非常接近了。那么到底什么是 `prototype` 呢？

## `prototype`

简单来说，在 JavaScript 里所有函数都有一个 `prototype` 属性，并且它是一个对象；也就是说 `prototype` 是函数对象上的预设属性。

```js
function doThing () {}
console.log(doThing.prototype)  // {constructor: ƒ}
```

现在我们就可以将 `animalMethods` 上的方法移植到我们的 `prototype` 上了，我们也将这种方法称为 `Prototypal Instantiation` 原型实例化

```js
function Animal (name, energy) {
  let animal = Object.create(Animal.prototype)
  animal.name = name
  animal.energy = energy

  return animal
}

Animal.prototype.eat = function (amount) {
  console.log(`${this.name} is eating.`)
  this.energy += amount
}

Animal.prototype.sleep = function (length) {
  console.log(`${this.name} is sleeping.`)
  this.energy += length
}

Animal.prototype.play = function (length) {
  console.log(`${this.name} is playing.`)
  this.energy -= length
}

const leo = Animal('Leo', 7)
const snoop = Animal('Snoop', 10)

leo.eat(10)
snoop.play(5)
```

`prototype` 是 JavaScript 中每个函数都有的属性，它可以让我们在某个函数的实例间共享属性、方法。


## 继续深入

截至到这里，我们学习了三个知识点：

1. 如何创建一个构造函数
2. 如何给构造函数的原型 `prototype` 添加方法
3. 如何使用 `Object.create` 来模拟原型链 `prototype chain`。

我们再观察一下 `Animal` 这个构造函数（工厂函数），其中有两个最重要的部分，就是创建一个对象、返回这个对象。如果不使用 `Object.create`，我们就无法做到类似原型链的行为；如果不返回创建的对象，我们就无法获取创建的对象。

```js
function Animal (name, energy) {
  let animal = Object.create(Animal.prototype)
  animal.name = name
  animal.energy = energy

  return animal
}
```

现在我们就来介绍 `new` 关键字，当我们使用 `new` 时，JavaScript 将自动完成上述的两个重要部分。

```js
function Animal (name, energy) {
  // const this = Object.create(Animal.prototype)

  this.name = name
  this.energy = energy

  // return this
}

Animal.prototype.eat = function (amount) {
  console.log(`${this.name} is eating.`)
  this.energy += amount
}

Animal.prototype.sleep = function (length) {
  console.log(`${this.name} is sleeping.`)
  this.energy += length
}

Animal.prototype.play = function (length) {
  console.log(`${this.name} is playing.`)
  this.energy -= length
}

const leo = new Animal('Leo', 7)
const snoop = new Animal('Snoop', 10)
```

使用了 `new`，它也会将 `this` 自动绑定到对应的对象上。其实这个方法就实现了 JavaScript 中的类，但不是我们常见的使用 `class` 来定义类。还好在ES6中，JavaScript 实现了 `class` 的语法糖。

```js
class Animal {
  contructor (name, energy) {
    this.name = name
    this.energy = energy
  }
  eat (amount) {
    console.log(`${this.name} is eating`)
    this.energy += amount
  }
  sleep (length) {
    console.log(`${this.name} is sleeping.`)
    this.energy += length
  }
  play (length) {
    console.log(`${this.name} is playing.`)
    this.energy -= length
  }
}

const leo = new Animal('Leo', 7)
const snoop = new Animal('Snoop', 10)
```

到这里就介绍完了 `prototype` 的基础内容。关于原型还有很多细节知识还未介绍，我将会在后续慢慢丰富它。

最后补上一张关于原型的图

![prototype](./images/prototype.png)
