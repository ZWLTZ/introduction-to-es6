# ES6简介

## 目标

1. 描述在ES6中的主要新功能

2. 解释`let`和`const`与`var`的区别

3. 描述如何使用箭头功能

4. 解释扩展值的操作

## 介绍

到目前为止，您可能已经在浏览器和服务器上使用 ECMAScript 5 的 JavaScript 版本工作过. ECMAScript 是一个不断发展的标准，完整的历史记录我们不在这里介绍。

但你可能也听说过这个叫做 ECMAScript 6, ES6或ES2015的东西 ( 他们有点错过了那段时期 )。 [ES6]（http://es6-features.org/）是JavaScript的下一个规范，由于 Node.js 5，ES6最终开始在浏览器和服务器上以主要方式出现。

您可以在Node.js [here]（https://nodejs.org/en/docs/es6/）中获得 ES6 功能的完整纲要; 但是我们也将给您介绍在即将到来的实验室中可能会看到的功能。 对于我们的目的，任何可以通过简单的"strict mode"声明启用的东西都是公平的游戏 - 但我们不会教你关于`--es_staging`或`--harmony`标志背后的东西。

## 旁白：严格模式

旁白：严格模式你可能还没有遇到它（至少知情），但是 ES5 有一种方法可以选择一种特殊的JavaScript版本[_strict mode_](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Strict_mode).

您可以在上面的链接中阅读有关严格模式的详细信息 - 通常，它会将 静默失败 转变为抛出错误，并且有助于防止变量潜入全局范围.

也就是说, 在标准模式下, 将运行以下代码 - 它不会执行任何操作：

```javascript
delete Object.prototype;
```

But in strict mode, this would throw a `TypeError`:

```javascript
"use strict";

delete Object.prototype;
```

Strict mode can be invoked by writing

```javascript
'use strict';
```

在当前脚本的顶部 - 严格模式然后适用于整个脚本。 或者，您可以将严格模式应用于各个功能：

```javascript
function strictFunction() {
  "use strict"

  // this will throw an error in strict mode
  delete Object.prototype
}

function standardFunction() {
  // this will silently fail in standard mode
  delete Object.prototype
}
```
严格模式就像它的名字所暗示的那样：它对代码的执行强制执行_stricter_规则。 请注意，一些转换器（如[babel]（http://babeljs.io/））可以为您设置严格模式。

严格模式还支持ES6开发人员希望确保用户明确选择使用的一些新功能。

## ES6 功能

There are a lot of new features, but for now we'll cover a subset of the new features below.

* [`const` and `let`](#block-scoping)
* [Classes](#classes-use-strict)
* [Arrow Functions](#arrow-functions)
* [Promises](#promises)
* [Object Literal Extensions](#object-literal-extensions)
* [Spread Operator](#spread-operator)
* [Template Strings](#template-strings)
* [Destructuring](#destructuring)

### 块作用域

#### `let` ("use strict")

The keyword [`let`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/let) is a new way of declaring local variables. How does it differ from good ol' `var`? Variables declared with `let` have block-level scope:

```javascript
// https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/let#Scoping_rules
function letTest() {
  let x = 31;
  if (true) {
    let x = 71;  // different variable
    console.log(x);  // 71
  }
  console.log(x);  // 31
}
```

Notice how `x` declared outside of the `if` block differs from the `x` declared inside the block. Block-level scope means that the variable is available only in the block (`if`, `for`, `while`, etc.) in which it is defined; it differs from JavaScript's normal function-level scope, which restricts the variable to the function in which it is defined (or `global`/`window` if it's a global variable).

#### `const`

The keyword [`const`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/const) does not require strict mode. Like `let`, it declares a variable with block-level scope; additionally, it prevents its variable identifier from being reassigned.

That means that the following will throw an error:

```javascript
const myVariable = 1;

myVariable = 2; // syntax error
```

However, this does not mean that the variable's value is immutable — the value can still change.

```javascript
const myObject = {};

// this works
myObject.myProperty = 1;

// 1
console.log(myObject.myProperty)
```

### Classes ("use strict")

"Wait," you say. "JavaScript has prototypal, not class-based, inheritance."

You're still right. But [classes](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Classes) in JavaScript are awesome, and you'll be seeing them increasingly as ES6 adoption increases.

Consider the following simple example, based largely on the examples from MDN. We want to create a `Polygon` prototype and inherit from it. We'll start with ES5:

```javascript
function Polygon(height, width) {
  this.height = height;
  this.width = width;
}
```

Cool, so we got to the ES6 class `constructor`, which works just like a constructor in ES5. But now how do we implement the `area()` getter? Well, not very nicely — let's back up and rewrite what we just wrote:

```javascript
function Polygon(height, width) {
  this.height = height;
  this.width = width;

  // :(
  this.area = this.calcArea();
}

Polygon.prototype.calcArea = function() {
  return this.height * this.width;
};

const rectangle = new Polygon(10, 5);

console.log(rectangle.area);
```

Okay, so that worked, but look at how difficult it is to reason about. We have to plan in advance for the properties that we want to set, and `area` is not calculated dynamically — it's set when the `Polygon` is instantiated and then forgotten about, so if somehow a `Polygon`'s `height` and `width` changed, its `area` would need to be updated separately. Gross.

Moreover, extending this ES5 `Polygon` is a bit onerous:

```javascript
function Square(sideLength) {
  Polygon.call(this, sideLength, sideLength);
}

Square.prototype = new Polygon();

const square = new Square(5);

// Polygon { height: 5, width: 5, area: 25 }
console.log(square);
```

Well, that seems like it just about works. But what if we check the variable `square`'s constructor?

```javascript
// [Function: Polygon]
square.constructor;
```

That's not right. It should be `[Function: Square]`. We can fix it, though:

```javascript
Square.prototype.constructor = Square;

const square2 = new Square(6);

// { [Function: Square] constructor: [Circular] }
square2.constructor
```

Eh, close enough?

(Note: The point here isn't to land on a perfect approach to object inheritance in JavaScript, it's to show that such a goal isn't feasible and won't be achieved in a nice way.)

Now let's try with ES6:

```javascript
class Polygon {
  constructor(height, width) {
    this.height = height;
    this.width = width;
  }

  // whaaaaat -- getters!
  get area() {
    return this.calcArea();
  }

  calcArea() {
    return this.height * this.width;
  }
}

const rectangle = new Polygon(10, 5);

console.log(rectangle.area);
```

Let's extend it:

```javascript
class Square extends Polygon {
  constructor(sideLength) {
    super(sideLength, sideLength)
  }
}

const mySquare = new Square(5);

// Square { height: 5, width: 5 }
mySquare;

// [Function: Square]
mySquare.constructor;

// 25
mySquare.area;
```

Whoa. That was easy.

![that was easy](http://i.giphy.com/zcCGBRQshGdt6.gif)

### Arrow functions

[Arrow functions](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Functions/Arrow_functions) provide not only a terser way to define a function but also _lexically bind the current `this` value_. This ain't your grandpa's JS.

```javascript
const greet = (greeting, person) => {
  return greeting + ', ' + person + '!';
};

// 'Hello, Marv'
greet('Hello', 'Marv');

var a = [
  'Hydrogen',
  'Helium',
  'Lithium',
  'Beryl­lium'
];

// compare this implementation...
var a2 = a.map(function(s){ return s.length });

// ... to this implementation with the fat arrow
var a3 = a.map(s => s.length);
```

Fat arrows also have implicit returns — the following are equivalent:

```javascript
var a3 = a.map(s => s.length);
var a4 = a.map(s => {
  return s.length;
});
```

If the function only accepts one argument, parentheses are optional:

```javascript
// this...
var a3 = a.map(s => s.length);

// ... is the same as this
var a3 = a.map((s) => s.length);
```

If there are zero or two or more arguments, though, you must use parens:

```javascript
var evens = [1, 2, 3, 4].reduce((memo, i) => {
  if (i % 2 === 0) {
    memo.push(i)
  }

  return memo;
}, []);
```

### Promises

[Promises](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Promise) offer a new way of handling asynchronicity.

We'll cover them in greater detail in a later lesson, but for now know that `Promise` is available globally in Node.js.

Also know that it's awesome.

```javascript
const promise = new Promise((resolve, reject) => {
  return someIntenseTask().then(result => {
    if (result.success) {
      return resolve(result)
    }

    return reject(result.error)
  })
})

promise.then(result => {
  return doSomething(result);
}).catch(error => handleError(error))
```

### Object literal extensions

ES6 gives us a number of handy [new ways to deal with objects](https://github.com/lukehoban/es6features#enhanced-object-literals). They're features that you either wish JavaScript had, or ones you didn't know you needed.

```javascript
const prop = function() {
  return "I'm a prop!";
}

const myObj = {
  // computed (dynamic) property names
  ['foo' + 'bar']: 'something',

  // methods
  shout() {
    return 'AH!'
  },

  // short for `prop: prop`
  prop
}

// 'something'
myObj.foobar

// "I'm a prop!"
myObj.prop()

// 'AH!'
myObj.shout()
```

### Spread operator

The [spread operator](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/Spread_operator) — `...` — is unassuming but incredibly powerful.

We can use it for arrays:

```javascript
const a = [1, 2, 3]
const b = [0, ...a, 4, 5]

// [0, 1, 2, 3, 4, 5]
b
```

functions:

```javascript
function printArgs() {
  // recall that every function gets an `arguments`
  // object
  console.log(arguments);
}

// using `a` from above
// [1, 2, 3]
printArgs(...a);
```


### Template Strings

[Template strings](https://nodejs.org/en/docs/es6/) in ES6 are most commonly used for string interpolation. Instead of writing:

```javascript
var foo = 'bar';
var sentence = 'I went to the ' + foo + ' after working in ES5 for too long.';
```

we can now write:

```javascript
var foo = 'bar';
var sentence = `I went to the ${foo} after working in ES5 for too long.`;
```

and we'll get the same result.

You can also use _tagged template literals_ to perform more advanced manipulation:

A _tag_ is simply a function whose first argument is an array of strings and whose subsequent arguments are the values of the substitution expressions (the things in `${}`).

Here's the example from [MDN](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Template_literals#Tagged_template_literals):

```javascript
var a = 5;
var b = 10;

function tag(strings, ...values) {
  console.log(strings[0]); // "Hello "
  console.log(strings[1]); // " world "
  console.log(values[0]);  // 15
  console.log(values[1]);  // 50

  return "Bazinga!";
}

tag`Hello ${ a + b } world ${ a * b }`;
// "Bazinga!"
```

### Destructuring

Destructuring makes it easier than ever to pull values out of objects and arrays and store them in variables. We destructure an array by putting our new variable names at the corresponding index and an object by giving our variable the same name as the key we are interested in.

```js
const [a, b] = [1, 2];
// a === 1 && b === 2

const { a, b } = { a: 1, b: 2 }
// a === 1 && b === 2
```

To see what other amazing things we can to with destructuring, check out the [docs](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/Destructuring_assignment).

## Conclusion

There are _tons_ of new features in ES6, and we don't have time to cover them all here. Check out [the docs](https://nodejs.org/en/docs/es6/), play around in console, and have fun!

## Resources

- [let](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/let): https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/let
- [const](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/const): https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/const
- [classes](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Classes): https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Classes
- [Promise](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Promise): https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Promise
- [Enhanced object literals](https://github.com/lukehoban/es6features#enhanced-object-literals): https://github.com/lukehoban/es6features#enhanced-object-literals
- [Template literals](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Template_literals): https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Template_literals
- [Arrow functions](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Functions/Arrow_functions): https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Functions/Arrow_functions

<p class='util--hide'>View <a href='https://learn.co/lessons/introduction-to-es6'>Introduction To ES6/ECMA2015</a> on Learn.co and start learning to code for free.</p>
