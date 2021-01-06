# 【译】2019年JavaScript中的计算机科学：链表（Computer science in JavaScript 2019: Linked list）

> * 原文地址：[Computer science in JavaScript 2019: Linked list](https://humanwhocodes.com/blog/2019/01/computer-science-in-javascript-linked-list/)
> * 原文作者：[Nicholas C. Zakas](https://humanwhocodes.com/)

早在2009年，我就挑战自己一年内坚持每周写一篇博客文章。我曾经读到过，坚持发表文章是为博客带来流量的最好的方法。基于我的所有文章的理念，一周发表一篇文章看起来是一个很实际的目标，而事实上我缺少了博客文章的52个理念。（译者注：不太清楚这里的意思，查阅到 [52 Ideas For Blog Posts](https://scoopindustries.com/ideas-for-blog-posts/) 这篇文章比较符合语境）我挖掘了一些写到一半的章节，并最终编撰了 `JavaScript 高级程序设计`，在其中发掘了很多关于经典计算机科学的材料，包括数据结构与算法。我将这些材料在2009年及2012年加工成为了几篇文章，并从其中得到了许多积极的反馈。

在这些文章发布的十周年之际，我决定在2019年使用 JavaScript 更新、拓展并重新发表它们。去看看其中哪些内容变了，哪些没有变也不失为一种乐趣，希望大家喜欢。

## 什么是链表？

链表是一种使用线性的方式来存储不同的值的数据结构。链表中的每个值都包含于自己的节点中，且这个节点包含了其指向链表中下一个节点的链接数据。这个链接为指向另一个节点对象的指针，如果没有下一个节点，则为 `null`。如果链表中每个节点只有一个指向另一个节点的指针（常见的为指向下一个节点），那么这种链表称作为**单链表（或就称为链表）**；而如果链表中每个节点有两个指针（常见的为指向上一个节点和下一个节点），通常我们称之为双向链表。本文中，我将主要探讨单链表。

## 为什么要使用单链表？

链表最主要的优点就是它可以存储任意数量的值，同时只占用这些值所需的内存大小。对于内存小的旧电脑来说，充分利用内存空间还是很重要的。相反，在 `C` 语言中的内置数组类型要求你指定数组的长度（有多少项），然后根据数组长度来分配内存。预留内存空间意味着这部分内存不能用于运行任何其他程序，即使这部分内存从未被使用。你可以很轻松在一台内存小的机器上使用数组来耗尽内存；而链表就是用于解决这个问题的。

虽然最初的目的是为了更好的内存管理，但是当开发人员无法预估一个数组最终会包含多少项时，链表也就变得更受欢迎。使用链表来根据需要添加数据要比精准地猜测数组可能包含的最多数量的项要简单得多。因此，链表在各种编程语言中也作为一种基础的内置数据结构。

## 链表的设计

链表最重要的部分就是它的节点的数据结构。每一个节点都必须包含一个指向链表中下一个节点的指针和一些数据。如下为 JavaScript 中简单的表示：

```js
class LinkedListNode {
  constructor (data) {
    this.data = data;
    this.next = null;
  }
}
```

在 `LinkedListNode` 类中，`data` 属性包含了此节点应存储的值，`next` 属性为指向链表中下一个节点的指针。`next` 的初始值应该为 `null`，因为你暂时无法知道此节点在链表中的下一个节点是什么。你可以像下面的例子一样使用 `LinkedListNode` 来创建一个链表：

```js
// 创建第一个节点
const head = new LinkedListNode(12);

// 添加第二个节点
head.next = new LinkedListNode(99);

// 添加第三个节点
head.next.next = new LinkedListNode(37);
```

第一个节点我们一般称为 `头`，因此 `head` 标识符在此例子中表示链表中的第一个节点。创建第二个节点并将 `head.next` 赋值于第二个节点，这样链表中就有了两个节点。通过给 `head.next.next` 赋值一个新节点来创建第三个节点，其中 `head.next.next` 为第二个节点中指向下一个节点的指针。第三个节点的 `next` 指针在链表中保持为 `null`。下图展示了最终的数据结构：

![singly linked list](./images/linked-list/singly-linked-list.svg)

我们可以通过链表每个节点中的 `next` 指针来遍历所有数据。如下就是一个简单的遍历链表数据并打印每个节点的值的例子：

```js
let current = head;

while (current !== null) {
  console.log(current.data);
  current = current.next;
}
```

上述代码使用 `current` 来作为链表中移动的指针，它的初始值为链表的 `头` 即 `head`，`while` 循环的条件为当 `current` 不为 `null`。在循环内部，`current` 对应的节点所存储的值将会被打印出来，接着 `current` 的 `next` 指针将往前移动，即 `current` 将被赋值为 `current.next` 所指向的节点。

大部分的链表操作都是使用这种遍历算法或其他类似的算法，因此了解这种算法对于理解链表一般来说非常重要。

## `LinkedList` 类

如果你曾使用 `C` 写过链表，你可能会在这一步就停下来，认为自己的任务已经完成了（尽管你将会用 `struct` 而不是类来表示每个节点）。然而，在面向对象的语言中，譬如 JavaScript，更多的是创建一个类来封装这个功能。如下就是一个简单的例子：

```js
const head = Symbol("head");

class LinkedList {
  constructor() {
    this[head] = null;
  }
}
```

`LinkedList` 类表示为包含了链表中的节点之间交互的方法的链表类。其唯一的属性 `head` 为使用 `symbol` 定义的变量，并指向链表中的第一个节点。使用 `symbol` 数据类型而不用 `string` 来定义此属性就是为了明确地表明并不想要此属性在此类外进行修改。

### 给链表添加新数据

在链表中添加新数据需要遍历链表找到正确的位置，然后创建新的节点，最后插到正确的位置。这里有个特殊的例子就是，当链表为空时，也就是你只需要新建一个节点然后赋予 `head` 即可：

```js
const head = Symbol("head")

class LinkedList {
  constructor() {
    this[head] = null;
  }

  add(data) {

    // 新建一个节点
    const newNode = new LinkedListNode(data);

    // 特殊情况：链表中无节点
    if (this[head] === null) {

      // 只需要将新建的节点赋予 `head`
      this[head] = newNode;
    } else {

      // 从第一个节点开始寻找
      let current = this[head];

      // 跟随 `next` 链接直到到链表的尾部
      while (current.next !== null) {
        current = current.next;
      }

      // 将新建的节点赋予 `next` 指针
      current.next = newNode;
    }
  }
}
```

`add()` 方法接收单个参数，可以为任何数据，然后将它添加到链表的尾部。如果链表为空（`this[head]` 为 `null`），你只需要将新建的节点 `newNode` 赋予给 `this[head]` 即可。如果链表不为空，那么你需要遍历链表中已有的节点，找打最后一个节点。这个遍历的过程开始于 `this[head]` 并跟随每个节点的 `next` 链接，直到找到最后一个节点。最后一个节点的 `next` 指针指向 `null`，因此在这个节点停止遍历非常重要，而不是当 `current` 为 `null` 时停止（如上一节所述）。然后你可以将新建的节点赋予其 `next` 属性，以此来将数据添加到链表中。

> 注意
>
> 传统的算法使用了两个指针 `current` 与 `previous`，`current` 指向当前节点，`previous` 指向 `current` 的上一个节点。当 `current` 为 `null` 时，意味着 `previous` 指向了链表中的最后一项。我认为当你可以检查 `current.next` 的值并在其为 `null` 时退出循环时还要使用这种方法并不合乎逻辑。

`add()` 方法的算法时间复杂度为 `O(n)`，因为你必须遍历整个链表来找到正确的位置来插入新的节点。你可以通过从链表尾部开始遍历（除去头部之外）来将时间复杂度降低为 `O(1)` ，这样你可以立即将新节点插入到正确的位置。

### 检索链表中的数据

链表不允许随机访问其数据，但你仍然可以通过遍历链表并返回数据来检索任何给定位置的数据。为此，你需要添加一个 `get()` 方法，它接受一个从零开始的索引作为参数来检索数据，如下所示：

```js
class LinkedList {

  // 为了简洁，先隐藏之前添加的方法

  get(index) {

    // 确保 `index` 为非负整数
    if (index > -1) {

      // 用于遍历的指针
      let current = this[head];

      // 用于跟踪链表中的位置
      let i = 0;

      // 遍历链表直到找到 `index` 索引或到达链表尾部
      while ((current !== null) && (i < index)) {
        current = current.next;
        i++;
      }

      // 如果 `current` 不为 `null`，返回对应的数据
      return current !== null ? current.data : undefined;
    } else {
      return undefined;
    }
  }

}
```

`get()` 方法首先先确保 `index` 为非负数，否则返回 `undefined`。变量 `i` 是用于跟踪遍历链表的深度；其中的循环就像你之前看到的基本遍历一样，只不过是多了一个条件：当 `i` 等于 `index` 时应该退出循环；这也就意味着要考虑如下两种退出循环的情况：

1. `current` 为 `null`，意味着链表实际长度小于 `index`
2. `i` 等于 `index`，意味着 `current` 就是 `index` 索引所在的节点

如果 `current` 为 `null`，那么就返回 `undefined`；反之返回 `current.data`。这个检查确保了 `get()` 将不会在链表中找不到索引时抛出错误（尽管或许你会用抛出错误而不是返回 `undefined`）。

`get()` 方法的算法时间复杂度范围从删除第一个节点的 `O(1)`（不需要遍历）到删除最后一个节点的 `O(n)`（需要遍历整个链表）。我们很难再去减小其算法时间复杂度，因为总需要检索来检查正确的返回值。

### 从链表中移除数据

从链表中移除数据会有点棘手，因为你要确保在移除节点后，所有节点的 `next` 指针都保持有效。譬如，如果你想要在一个有三个节点的链表中移除第二个节点，你需要确保第一个节点的 `next` 指针指向第三个节点，而不再是第二个节点。以这种方式来跳过第二个节点可以很有效地从链表中移除第二个节点。

![single linked list delete after](./images/linked-list/Singly_linked_list_delete_after.png)

移除的操作其实就两个步骤：

1. 找到特定的索引（与 `get()` 一样的算法）
2. 移除特定索引的节点

寻找这个特定的索引就和 `get()` 方法一样，但在循环内部你还需要追踪 `current` 的上一个节点，因为你需要修改上一个节点的 `next` 指针。

同时你还需要考虑以下四种特殊情况：

1. 链表为空（不需要遍历）
2. 索引小于零
3. 索引大于链表的节点数
4. 索引为零（移除链表头部 `head`）

在前三个特殊情况中，是无法完成移除操作，因此抛出错误也就很合理了。第四个特殊情况要求重新对 `this[head]` 进行赋值。如下是 `remove()` 方法的相关实现：

```js
class LinkedList {

  // 为了简洁，先隐藏之前添加的方法

  remove(index) {

    // 特殊情况：空链表或非法 `index`
    if ((this[head] === null) || (index < 0)) {
      throw new RangeError(`index ${index} does not exist in the list.`)
    }

    // 特殊情况：移除第一个节点
    if (index === 0) {

      // 临时存储节点数据
      const data = this[head].data;

      // 用下一个节点（第二个节点）来替换链表的 head
      this[head] = this[head].next

      // 返回链表原来的 head 的数据
      return data;
    }

    // 用于遍历链表的指针
    let current = this[head];

    // 追踪循环中 current 的上一个节点
    let previous = null;

    // 用于追踪链表中的位置
    let i = 0;

    // 与 `get()` 相同的循环算法
    while ((current !== null) && (i < index)) {

      // 保存 current 的值
      previous = current;

      // 遍历到下一个节点
      current = current.next;

      // 增加次数
      i++;
    }

    // 如果找到要移除的节点则移除它
    if (current !== null) {

      // 通过跳过此节点（即不再链接此节点）来移除它
      previous.next = current.next;

      // 返回刚才被移除的节点的值
      return current.data;
    }

    // 如果找不到要移除的节点则抛出错误
    throw new RangeError(`Index ${index} does not exist in the list.`);
  }
}
```

`remove()` 方法首先先检查前两个特殊情况，空链表 `this[head]` 为 `null` 和 `index` 小于零；两种情况都将抛出错误。

下一个特殊情况就是当 `index` 为 `0` 时，意味着你需要删除链表的头部。新的链表的头部将会成为原来链表中的第二个节点，因此你可以将 `this[head].next` 赋值于 `this[head]`。不用担心如果链表中只有一个节点，因为 `this[head]` 最终会等于 `null`，也就意味着移除节点后链表变成了空链表。唯一的问题就是，我们需要将链表原来的头部的数据存储在一个局部变量 `data` 中，以便返回它。

在处理了四个特殊情况的前三个之后，现在你可以继续进行类似于 `get()` 方法中的遍历。就像之前提到的，`remove()` 方法中的循环有一点不一样，那就是它使用了 `previous` 来跟踪 `current` 的上一个节点，因为 `previous` 是能否正确移除节点的关键信息。和 `get()` 方法类似，当退出循环时，`current` 可能为 `null`，也就表示未找到 `index`。如果发生了这种情况，那就抛出错误；反之将 `current.next` 赋值于 `previous.next`，这样就很有效地将 `current` 从链表中移除了。最后一步就是将 `current` 中所储存的值返回即可。

`remove()` 方法的算法时间复杂度与 `get()` 方法一样，当移除第一个节点时，时间复杂度为 `O(1)`；当一处最后一个节点时，时间复杂度为 `O(n)`。

## 使链表可迭代

为了能使用 JavaScript 中的 `for-of` 循环和数组解构，数据的集合必须使可迭代的。在默认情况下，JavaScript 中的内置的集合（如 Array 和 Set）都是可迭代的，你可以通过在类上指定 `Symbol.iterator` 生成器（`genenrator`）方法来使你自己定义的类可迭代。我更倾向于先实现一个 `values()` 生成器方法（用来匹配内置集合类中找到的方法），然后直接使用 `Symbol.iterator` 来调用 `values()`。

`values()` 方法只需要对链表进行基本的遍历并 `yield` 每个节点的值即可：

```js
class LinkedList {

  // 为了简洁，先隐藏之前添加的方法

  *values() {

    let current = this[head];

    while (current !== null) {
      yield current.data;
      current = current.next;
    }
  }

  [Symbol.iterator]() {
    return this.values();
  }

}
```

`values()` 方法通过前置的 `*` 星号来标明它为一个生成器方法。该方法用于遍历链表，并使用 `yield` 来返回其遇到的每个节点的值。（注意 `Symbol.iterator` 方法并不是生成器，因为它是从 `values()` 生成器方法中返回一个迭代器）

## 使用类

实现了上述的方法后，你就可以如下例子一样使用这个链表类：

```js
const list = new LinkedList();
list.add("red");
list.add("orange");
list.add("yellow");

// 获取链表的第二个节点
console.log(list.get(1));         // orange

// 打印所有节点
for (const color of list) {
  console.log(color);
}

// 移除链表中第二个节点
console.log(list.remove(1));      // orange

// 获取链表中新的第二个节点
console.log(list.get(1));         // yellow

// 将链表转换为数组
const array1 = [...list.values()];
const array2 = [...list];
```

这个链表的基本实现还可以添加 `size` 属性来计算链表中的节点个数，或者其他类似 `indexOf()` 的方法。完整的代码在我的 GitHub 的 [Computer Science in JavaScript](https://github.com/humanwhocodes/computer-science-in-javascript) 中。

## 总结

链表可能并不是你每天都会用到的东西，但它在计算机科学技术中式非常基础的一种数据结构。其利用节点来指向彼此节点的概念在很多其他的数据结构中及其他的高级编程语言中都有所体现。能够很好地理解链表的工作原理对于如何全面理解怎么创建及使用其他数据结构非常重要。

而对于 JavaScript 来说，你应该尽量使用内置的集合类，譬如 `Array`，而不是自己写一个。因为内置的集合类已经针对生产使用进行了优化，并在不同的执行环境下有良好的支持。
