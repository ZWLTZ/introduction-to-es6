# ES6简介

## 目标

1. 描述在ES6中的主要新功能

2. 解释`let`和`const`与`var`的区别

3. 描述如何使用箭头功能

4. 解释扩展值的操作

## 介绍

到目前为止,您可能已经在浏览器和服务器上使用 ECMAScript 5 的 JavaScript 版本工作过. ECMAScript 是一个不断发展的标准,完整的历史记录我们不在这里介绍.

但你可能也听说过这个叫做 ECMAScript 6, ES6或ES2015的东西 ( 他们有点错过了那段时期 ). [ES6](http://es6-features.org/) 是JavaScript的下一个规范,由于 Node.js 5,ES6最终开始在浏览器和服务器上以主要方式出现.

您可以在Node.js [here](https://nodejs.org/en/docs/es6/)中获得 ES6 功能的完整纲要; 但是我们也将给您介绍在即将到来的实验室中可能会看到的功能. 对于我们的目的,任何可以通过简单的"strict mode"声明启用的东西都是公平的游戏 - 但我们不会教你关于`--es_staging`或`--harmony`标志背后的东西.

## 旁白:严格模式

旁白:严格模式你可能还没有遇到它(至少知情),但是 ES5 有一种方法可以选择一种特殊的JavaScript版本[_strict mode_](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Strict_mode).

您可以在上面的链接中阅读有关严格模式的详细信息 - 通常,它会将 静默失败 转变为抛出错误,并且有助于防止变量潜入全局范围.

也就是说, 在标准模式下, 将运行以下代码 - 它不会执行任何操作:

```javascript
delete Object.prototype;
```

但在严格的模式下, 这将抛出TypeError:

```javascript
"use strict";

delete Object.prototype;
```

可以通过书写调用严格模式.

```javascript
'use strict';
```

在当前脚本的顶部 - 严格模式适用于整个脚本. 或者, 您可以将严格模式应用于各个功能:

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
严格模式就像它的名字所暗示的那样:它对代码的执行强制执行_stricter_规则. 请注意,一些转换器(如[babel](http://babeljs.io/)可以为您设置严格模式.

严格模式还支持ES6开发人员希望确保用户明确选择使用的一些新功能.

## ES6 功能

有许多新的功能, 但是现在我们将介绍下面新功能的一部分.

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

关键词 [`let`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/let)是一种声明局部变量的新方法. 它与 `var` 有什么不同? 用`let`声明的变量具有块级范围:

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

注意`if`块之外声明的`x`与块内声明的`x`不同. 块级作用域意味着该变量仅在定义它的块(`if`,`for`,`while`等)中可用, 它不同于正常 JavaScript 范围的函数层级, 它将变量限制为定义它的函数( 如果它是全局变量, 则限制为`global` /`window`).

#### `const`

关键字 [`const`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/const) 不需要严格模式. 与`let`一样, 它声明了一个具有块级范围的变量; 另外,它可以防止重新分配其变量标识符.

这意味着以下将抛出错误:

```javascript
const myVariable = 1;

myVariable = 2; // syntax error
```

但是,这并不意味着变量的值是不可变的 - 值仍然可以改变.

```javascript
const myObject = {};

// this works
myObject.myProperty = 1;

// 1
console.log(myObject.myProperty)
```

### Classes ("use strict")

"等等," 你说. "JavaScript具有原型, 而不是基于类的继承."

你还是对的. 但是JavaScript中的 [classes](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Classes)非常强大的, 当你深入研究ES6, 你会越来越多地看到它们.

考虑以下简单示例,主要基于MDN的示例. 我们想要创建一个`Polygon`原型并继承它. 我们将从ES5开始:

```javascript
function Polygon(height, width) {
  this.height = height;
  this.width = width;
}
```

很酷, 所以我们得到了ES6类`constructor`,它就像ES5中的构造函数一样工作. 但是现在我们如何实现`area()` getter? 
很好,但不是非常好 - 让我们备份并重写我们刚写的内容:

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
好吧,这样有效,但看看有多难以推理. 我们必须提前计划我们想要设置的属性,并且`area`不是动态计算的 - 它是在`Polygon`被实例化然后被遗忘时设置的,所以如果不知怎的话`Polygon`的`height` 和`width`改变了, 它的"区域"需要单独更新. 糟糕.

此外,扩展这个ES5`Polygon`有点麻烦:


```javascript
function Square(sideLength) {
  Polygon.call(this, sideLength, sideLength);
}

Square.prototype = new Polygon();

const square = new Square(5);

// Polygon { height: 5, width: 5, area: 25 }
console.log(square);
```

嗯,这似乎只是作品. 但是如果我们检查变量`square`的构造函数呢？

```javascript
// [Function: Polygon]
square.constructor;
```

那是不对的. 它应该是`[Function:Square]`. 我们可以解决它,但是:

```javascript
Square.prototype.constructor = Square;

const square2 = new Square(6);

// { [Function: Square] constructor: [Circular] }
square2.constructor
```

呃,足够接近？

(注意:这里的要点不是基于 JavaScript 中对象继承的完美方法,而是要表明这样的目标是不可行的,并且不会以一种很好的方式实现.)

现在让我们试试ES6:

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

让我们来扩展它:

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

哇. 这很容易.

![that was easy](http://i.giphy.com/zcCGBRQshGdt6.gif)

### 箭头函数

[Arrow functions](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Functions/Arrow_functions) 不仅提供了一种定义函数的简洁方法,而且还提供了绑定当前"this"的函数.这不是大爷的JS.

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

箭头也有隐含的返回 —— 以下是等价的:

```javascript
var a3 = a.map(s => s.length);
var a4 = a.map(s => {
  return s.length;
});
```

如果函数只接受一个参数,括号是可选的:

```javascript
// this...
var a3 = a.map(s => s.length);

// ... is the same as this
var a3 = a.map((s) => s.length);
```

如果有零点或两个或更多个参数,你必须使用括号:

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
