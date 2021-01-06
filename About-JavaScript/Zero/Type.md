# Type

## ToPrimitive

```ts
function TypeOf (value) {
  const result = typeof value

  switch (result) {
    case 'function':
      return 'object'
    case 'object':
      if (value === null) {
        return 'null'
      } else {
        return 'object'
      }
    default:
      return result
  }
}

function isCallable (x) {
  return typeof x === 'function'
}

function ToPrimitive (input: any, hint: 'string'|'number'|'default' = 'default') {
  if (TypeOf(input) === 'object') {
    let exoticToPrim = input[Symbol.toPrimitive]
    if (exoticToPrim !== undefined) {
      let result = exoticToPrim.call(input, hint)
      if (TypeOf(result) !== 'object') {
        return result
      }
      throw new TypeError()
    }
    if (hint === 'default') {
      hint = 'number'
    }
    return OrdinaryToPrimitive(input, hint)
  } else {
    // input is already primitive
    return input
  }
}

function OrdinaryToPrimitive(O: object, hint: 'string'|'number') {
  let methodNames

  if (hint === 'string') {
    methodNames = ['toString', 'valueOf']
  } else {
    methodNames = ['valueOf', 'toString']
  }

  for (let name of methodNames) {
    let method = O[name]
    if (isCallable(method)) {
      let result = method.call(O)
      if (TypeOf(result) !== 'object') {
        return result
      }
    }
  }
  throw new TypeError()
}

Date.prototype[Symbol.toPrimitive] = function (hint: 'default' | 'string' | 'number') {
  let O = this
  if (TypeOf(O) !== 'object') {
    throw new TypeError()
  }
  let tryFirst
  if (hint === 'string' || hint === 'default') {
    tryFirst = 'string'
  } else if (hint === 'number') {
    tryFirst = 'number'
  } else {
    throw new TypeError()
  }
  return OrdinaryToPrimitive(O, tryFirst)
}
```

Which hints do callers of ToPrimitive() use?

The parameter hint can have one of three values:

* 'number' means: if possible, input should be converted to a number
* 'string' means: if possible, input should be converted to a string
* 'default' means: there is no preference for either numbers or strings

There are a few examples of how various operations use ToPrimitive():

* hint === 'number'. The following operations prefer numbers:
  * ToNumeric()
  * ToNumber()
  * ToBigInt(), BigInit()
  * Abstract Relational Comparison (<)
* hint === 'string'. The following operations prefer strings:
  * ToString()
  * ToPropertyKey()
* hint === 'default'. The following operations are neutral w.r.t. the type of the returned primitive value:
  * Abstract Equality Comparison (==)
  * Addition Operator(+)
  * new Date(value) (value can be either a number or a string)

As we have seen, the default behavior is for 'default' being handled as if it were 'number'. Only instances of Symbol and Date override this behavior (shown later).

Which methods are called to convert objects to Primitives?

If the conversion to primitive isn't overridden via Symbol.toPrimitive, OrdinaryToPrimitive() calls either or both of the following two methods:

* 'toString' is called first if hint indicates that we'd like the primitive value to be a sting
* 'valueOf' is called first if hint indicates that we'd like the primitive value to be a number

The following code demonstrates how that works:

```js
const obj = {
  toString() { return 'a' },
  valueOf() { return 1 }
}
String(obj) // a
Number(obj) // 1
```

**Date.prototype[Symbol.toPrimitive]()**

The only difference with the default algorithms is that 'default' becomes 'string' (and not 'number'). This can be observed if we use operations that set hint to 'default'

ToString() and related operations

This is the JavaScript version of ToString():

```ts
function ToString (argument) {
  if (argument === undefined) {
    return 'undefined'
  } else if (argument === null) {
    return 'null'
  } else if (argument === true) {
    return 'true'
  } else if (argument === false) {
    return 'false'
  } else if (TypeOf(argument) === 'number') {
    return Number.toString(argument)
  } else if (TypeOf(argument) === 'string') {
    return argument
  } else if (TypeOf(argument) === 'symbol') {
    throw new TypeError()
  } else if (TypeOf(argument) === 'bigint') {
    return BigInt.toString(argument)
  } else {
    let primValue = ToPrimitive(argument, 'string')
    return ToString(primValue)
  }
}
```

Addition operator (+)

```js
function Addition (leftHandSide, rightHandSide) {
  let lprim = ToPrimitive(leftHandSide)
  let rprim = ToPrimitive(rightHandSide)
  if (TypeOf(lprim) === 'string' || TypeOf(rprim) === 'string') {
    return ToString(lprim) + ToString(rprim)
  }
  let lnum = ToNumeric(lprim)
  let rnum = ToNumeric(rprim)

  if (TypeOf(lnum) !== TypeOf(rnum)) {
    throw new TypeError()
  }
  let T = Type(lnum)
  return T.add(lnum, rnum)
}
```

Steps of this algorithm:

* Both operands are converted to primitive values.
* If one of the results is a string, both are converted to strings and concatenated
* Otherwise, both operands are converted to numeric values and added. Type() return the ECMAScript specification type of lnum. .add() is a method of numeric types.
