> * 原文地址：[What are JavaScript Generators and how to use them](https://codeburst.io/what-are-javascript-generators-and-how-to-use-them-c6f2713fd12e)
> * 译文地址：[什么是JavaScript generator 以及如何使用它们](https://github.com/yued-fe/y-translation/blob/master/en/What%20are%20JavaScript%20Generators%20and%20how%20to%20use%20them.md)
> * 译者：[li-z](https://github.com/li-z)
> * 校对者：[张舰](https://github.com/zhangjiana) [赵冉](https://github.com/heiranran)

# 什么是JavaScript generator 以及如何使用它们

在本文中，我们将要介绍 ECMAScript 6 中的 generator 是什么，以及关于它们的使用案例。

### 什么是 JavaScript generator

generators 是可以控制 iterator (迭代器)的函数。并在任何时候都可以暂停和恢复。

如果这不好理解，那让我们看一些示例，这些示例将解释 generator 是什么，以及它和 iterator(迭代器，如 for-loop) 之间的区别。

这是一个立即输出值的 **for** 循环。这段代码是做什么的？—— 只是输出 0 到 5 之间的数

```js
for (let i = 0; i < 5; i += 1) {
  console.log(i);
}
// 将会立即输出 0 -> 1 -> 2 -> 3 -> 4
```

现在让我们看看 generator 函数

```js
function * generatorForLoop(num) {
  for (let i = 0; i < num; i += 1) {
    yield console.log(i);
  }
}

const genForLoop = generatorForLoop(5);

genForLoop.next(); // 首先 console.log —— 0
genForLoop.next(); // 1
genForLoop.next(); // 2
genForLoop.next(); // 3
genForLoop.next(); // 4
```

这段代码做了什么呢？实际上，它只是做了一些修改来包装上面的 **for** 循环。但是最主要的变化是，它不会立即执行。这是 generator 中最重要的特性 — 我们可以在真正需要下一个值的时候，才去获取它，而不是一次获得所有值。在有些情况下，这会非常方便。

### generator 语法

如何声明一个 generator 函数呢？我们有很多方法可以实现，但是最主要的是在函数关键字之后添加一个星号。

```js
function * generator () {}
function* generator () {}
function *generator () {}

let generator = function * () {}
let generator = function* () {}
let generator = function *() {}

let generator = *() => {} // SyntaxError
let generator = ()* => {} // SyntaxError
let generator = (*) => {} // SyntaxError
```

从上面的示例中可以看出，我们不能使用箭头函数来创建一个 generator。

接下来 —— generator 作为一种方法。它的声明方式与函数相同。

```js
class MyClass {
  *generator() {}
  * generator() {}
}

const obj = {
  *generator() {}
  * generator() {}
}
```

#### Yield

现在，让我们来看看新的关键字 *yield*。它有点像 **return**，但不是。**return** 只是在函数调用之后返回值，它不允许你在 **return** 语句之后执行任何其他操作。

```js
function withReturn(a) {
  let b = 5;
  return a + b;
  b = 6; // 不会重新定义 b 了
  return a * b; // 永远不会返回新的值了
}

withReturn(6); // 11
withReturn(6); // 11
```

而 **yield** 的执行方式不同。

```js
function * withYield(a) {
  let b = 5;
  yield a + b;
  b = 6; // 第一次执行之后仍可以重新定义变量
  yield a * b;
}

const calcSix = withYield(6);

calcSix.next().value; // 11
calcSix.next().value; // 36
```


**yield** 只返回一次值，下次调用 **next()** 时，它将执行到下一个 **yield** 语句。

在 generator 中我们通常都会获得一个对象作为输出。它有两个属性 **value** 和 **done**。正如你所想，**value** 表示返回的值，**done** 告诉我们 generator 是否完成了它的工作(是否迭代完成)。

```js
function * generator() {
    yield 5;
}

const gen = generator();

gen.next(); // {value: 5, done: false}
gen.next(); // {value: undefined, done: true}
gen.next(); // {value: undefined, done: true} - 继续调用 next() 将返回相同的输出。
```

在 generator 中不仅可以使用 **yield**，**return** 语句也会返回相同的对象，但是当执行完第一个 **retrurn** 语句时，迭代就会停止了。

```js
function * generator() {
  yield 1;
  return 2;
  yield 3; // 这个 yield 永远都不会执行
}

const gen = generator();

gen.next(); // {value: 1, done: false}
gen.next(); // {value: 2, done: true}
gen.next(); // {value: undefined, done: true}
```

#### Yield delegator

带星号的 **yield** 可以代理执行另一个 generator。这样你就可以根据需要连续调用多个 generator。

```js
function * anotherGenerator(i) {
  yield i + 1;
  yield i + 2;
  yield i + 3;
}

function * generator(i) {
  yield* anotherGenerator(i);
}

var gen = generator(1);

gen.next().value; // 2
gen.next().value; // 3
gen.next().value; // 4
```

在我们详细介绍 generator 中的方法之前，让我们来看看第一次看到会觉得很奇怪的一些行为。

下面这段代码向我们展示了 **yield** 可以返回在 **next()** 方法调用中传递的值。

```js
function * generator(arr) {
  for (const i in arr) {
    yield i;
    yield yield;
    yield(yield);
  }
}

const gen = generator([0,1]);

gen.next(); // {value: "0", done: false}
gen.next('A'); // {value: undefined, done: false}
gen.next('A'); // {value: "A", done: false}
gen.next('A'); // {value: undefined, done: false}
gen.next('A'); // {value: "A", done: false}
gen.next(); // {value: "1", done: false}
gen.next('B'); // {value: undefined, done: false}
gen.next('B'); // {value: "B", done: false}
gen.next('B'); // {value: undefined, done: false}
gen.next('B'); // {value: "B", done: false}
gen.next(); // {value: undefined, done: true}
```

正如你在这个例子中所看到的，默认情况下，**yield** 是 **undefined** 的，但是如果我们传递任何值，并且只调用 **yield**，它将返回我们传递的值。我们很快就会使用这个功能。

#### 方法和初始化

generator 是可重用的，但是你需要初始化它们，幸运的是，这非常简单。

```js
function * generator(arg = 'Nothing') {
  yield arg;
}

const gen0 = generator(); // OK
const gen1 = generator('Hello'); // OK
const gen2 = new generator(); // Not OK

generator().next(); // 这样也可以，但是会每一次从都头开始
```

**gen0** 和 **gen1** 不会相互影响。**gen2** 不会起作用，甚至会报一个错误。合理的初始化一个 **generator** 对于保持其内部的执行进度非常重要。

现在让我们看看 generator 提供给我们的一些方法。

**方法 next():**

```js
function * generator() {
  yield 1;
  yield 2;
  yield 3;
}

const gen = generator();

gen.next(); // {value: 1, done: false}
gen.next(); // {value: 2, done: false}
gen.next(); // {value: 3, done: false}
gen.next(); // {value: undefined, done: true} 继续调用 next() 将返回相同的输出。
```

这将会是你最常使用的方法。每次调用它时，它都会为我们输出一个对象。当所有的 yield 表达式被执行完之后，**next()** 会把属性 **done** 设置为 **true**，将属性 **value** 设置为 **undfined**。

我们不仅可以用 **next()** 来迭代 generator，还可以用 **for of** 循环来一次得到生成器所有的值（而不是对象）。

```js
function * generator(arr) {
  for (const el in arr)
    yield el;
}

const gen = generator([0, 1, 2]);

for (const g of gen) {
  console.log(g); // 0 -> 1 -> 2
}

gen.next(); // {value: undefined, done: true}
```

但这不适用于 **for in** 循环，也不能直接用数字下标来访问属性：`generator[0] = undefined`。

**方法 return():**

```js
function * generator() {
  yield 1;
  yield 2;
  yield 3;
}

const gen = generator();

gen.return(); // {value: undefined, done: true}
gen.return('Heeyyaa'); // {value: "Heeyyaa", done: true}

gen.next(); // {value: undefined, done: true} - 在 return() 之后的所有 next() 调用都会返回相同的输出

```

**return()** 将会忽略生成器中的任何代码。它会根据传值设定 value，并将 **done** 设为 true。任何在 **return()** 之后进行的 **next()** 调用都会返回 done 属性为 true 的对象。

**方法 throw():**

```js
function * generator() {
  yield 1;
  yield 2;
  yield 3;
}

const gen = generator();

gen.throw('Something bad'); // Error Uncaught Something bad
gen.next(); // {value: undefined, done: true}
```

**throw()** 做的事情非常简单 — 就是抛出错误。我们可以用 **try-catch** 来处理。

#### 实现自定义方法

我们无法直接访问 **generator** 构造函数，因此我们需要另想办法来添加自定义方法。 示例是我的办法，当然你也可以选择不同的方式。

```js
function * generator() {
  yield 1;
}

generator.prototype.__proto__; // Generator {constructor: GeneratorFunction, next: ƒ, return: ƒ, throw: ƒ, Symbol(Symbol.toStringTag): "Generator"}

// 由于 generator 不是一个全局变量，所以我们只能这么写：
generator.prototype.__proto__.math = function(e = 0) {
  return e * Math.PI;
}

generator.prototype.__proto__; // Generator {math: ƒ, constructor: GeneratorFunction, next: ƒ, return: ƒ, throw: ƒ, …}

const gen = generator();
gen.math(1); // 3.141592653589793
```

#### generator 的作用

在前面，我们用了已知迭代次数的 generator。但如果我们不知道要迭代多少次会怎么样呢？要想解决这个问题，在 generator 函数中创建一个死循环就足够了。下面是一个返回随机数的函数的例子：

```js
function * randomFrom(...arr) {
  while (true)
    yield arr[Math.floor(Math.random() * arr.length)];
}

const getRandom = randomFrom(1, 2, 5, 9, 4);

getRandom.next().value; // 返回一个随机数
```

这很容易，但是对于更复杂的功能，例如节流函数。 如果你不知道它是什么，请参考[这篇文章](https://medium.com/@_jh3y/throttling-and-debouncing-in-javascript-b01cad5c8edf)。

```js
function * throttle(func, time) {
  let timerID = null;
  function throttled(arg) {
    clearTimeout(timerID);
    timerID = setTimeout(func.bind(window, arg), time);
  }
  while (true)
    throttled(yield);
}

const thr = throttle(console.log, 1000);

thr.next(); // {value: undefined, done: false}
thr.next('hello'); // {value: undefined, done: false} 1秒后输出 -> 'hello'
```

还有没有更好的使用 generator 的例子呢？如果你听说过递归，我相信你也听说过[斐波那契数列](https://en.wikipedia.org/wiki/Fibonacci_number)。 通常它是通过递归来解决的，但是在 generator 的帮助下我们可以这样写它：

```js
function * fibonacci(seed1, seed2) {
  while (true) {
    yield (() => {
      seed2 = seed2 + seed1;
      seed1 = seed2 - seed1;
      return seed2;
    })();
  }
}

const fib = fibonacci(0, 1);
fib.next(); // {value: 1, done: false}
fib.next(); // {value: 2, done: false}
fib.next(); // {value: 3, done: false}
fib.next(); // {value: 5, done: false}
fib.next(); // {value: 8, done: false}
```

不再需要递归了！我们可以在需要的时候获得数列中的下一个数字。

#### 将 generator 用在 HTML 中

既然我们讨论的是 JavaScript，那最显著的方式就是使用 generator 对 HTML 进行一些操作。

因此，假设我们有一些 HTML 块要处理，我们可以很容易地通过 generator 实现，但要记住，在不使用 generator 的情况下，还有很多可能的方法可以实现这一点。

[Using generator-iterator with HTML elements
A PEN BY Vlad](https://codepen.io/vldvel/pen/LQqegv)

我们只需要很少的代码就能完成此需求。

```js
const strings = document.querySelectorAll('.string');
const btn = document.querySelector('#btn');
const className = 'darker';

function * addClassToEach(elements, className) {
  for (const el of Array.from(elements))
    yield el.classList.add(className);
}

const addClassToStrings = addClassToEach(strings, className);

btn.addEventListener('click', (el) => {
  if (addClassToStrings.next().done)
    el.target.classList.add(className);
});
```

实际上，仅有 5 行逻辑代码。

#### 总结

使用 generator 还有很多能做的事情。例如，在进行异步操作时，它可以很有用。或者是遍历一个按需项循环。

我希望这篇文章能帮助你更好地理解 JavaScript generator。
