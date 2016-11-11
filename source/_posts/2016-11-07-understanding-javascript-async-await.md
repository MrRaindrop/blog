---
layout: post
title:  "理解 javascript 的 async await"
date:   2016-11-07 15:53:01
categories: javascript
tags: 
  - javascript
  - async
  - await
author: "_danZ"
---

`async` / `await` 并没有作为 ES2016 的一部分, 但这不意味着 Javascript 不会加入 这一语法特性。就在本文撰写的此刻，它正处于 [Stage 3](https://github.com/tc39/ecma262/tree/82bebe057c9fca355cfbfeb36be8e42f18c61e94) 的阶段, 并处于活跃更新状态. 这个功能[在 Edge 里已经可用](https://blogs.windows.com/msedgedev/2015/09/30/asynchronous-code-gets-easier-with-es2016-async-function-support-in-chakra-and-microsoft-edge/), 并且[如果在更多浏览器中被实现则进入 Statge 4](https://twitter.com/bterlson/status/692464374842290176) —— 可以说，下个版本该功能已经在路上了 [(参考: TC39 流程)](https://tc39.github.io/process-document/).

<!--more-->

原文链接 : [https://ponyfoo.com/articles/understanding-javascript-async-await](https://ponyfoo.com/articles/understanding-javascript-async-await)

我们听说过这个特性已经有段时间了，但是并没有真正深入的探索它到底是怎样工作的。本文会帮助你理解这方面的内容，但此之前需要你对 promise 和 generator 已经有所了解。

* [了解 ES6 的 350 个要点](https://ponyfoo.com/articles/es6)

* [深入 ES6 Promises](https://ponyfoo.com/articles/es6-promises-in-depth)

* [深入 ES6 Generators](https://ponyfoo.com/articles/es6-generators-in-depth)

* [Generators，Promises 和异步 I/O](https://ponyfoo.com/articles/asynchronous-i-o-with-generators-and-promises)

* [Promisees 可视化工具](http://bevacqua.github.io/promisees/)

# 使用 Promise

假设我们有如下代码，我们将一个 HTTP 请求封装在一个 `Promise` 里，若请求成功则将 `body` 成功返回，否则将 `err` reject 出来。这个请求每次会拉取本博客里的随机一篇文章的 HTML 内容。

```javascript
var request = require('request');

function getRandomPonyFooArticle () {
  return new Promise((resolve, reject) => {
    request('https://ponyfoo.com/articles/random', (err, res, body) => {
      if (err) {
        reject(err); return;
      }
      resolve(body);
    });
  });
} 
```

典型的使用上面 promise 的方法如下代码所示。我们构造了一个 promise 链，将 HTML 页面的一部分子集 DOM 结构转换为对应的 markdown 文档，并最终以适用于终端的方式用 `console.log` 打印出来。记住最好给你的 promise 都加上 `.catch` 错误处理。

```javascript
var hget = require('hget');
var marked = require('marked');
var Term = require('marked-terminal');

printRandomArticle();

function printRandomArticle () {
  getRandomPonyFooArticle()
    .then(html => hget(html, {
      markdown: true,
      root: 'main',
      ignore: '.at-subscribe,.mm-comments,.de-sidebar'
    }))
    .then(md => marked(md, {
      renderer: new Term()
    }))
    .then(txt => console.log(txt))
    .catch(reason => console.error(reason));
} 
```

以上代码运行起来的效果如下截图所示。

[![Screenshot](http://p0.qhimg.com/t016eec12d425680f47.png)](https://github.com/bevacqua/read-ponyfoo/blob/966b4ce513250561826871b52cea436e0edbce09/cli)

运行截图

从代码可读性的角度来看，这段代码比使用回调更好更更有序。

# 使用 Generator

我们[之前已经了解怎么使用 generator 通过一种伪"同步"的方式构建可用的 `html`](https://ponyfoo.com/articles/es6-generators-in-depth). 虽然当时的代码某种程度上可以说是同步的代码，但是需要包裹很多代码结构，而且 generator 也许并不是最直接的能够达成我们目的方式，所以我们可能还是要依靠 Promise.

```javascript
function getRandomPonyFooArticle (gen) {
  var g = gen();
  request('https://ponyfoo.com/articles/random', (err, res, body) => {
    if (err) {
      g.throw(err); return;
    }
    g.next(body);
  });
}

getRandomPonyFooArticle(function* printRandomArticle () {
  var html = yield;
  var md = hget(html, {
    markdown: true,
    root: 'main',
    ignore: '.at-subscribe,.mm-comments,.de-sidebar'
  });
  var txt = marked(md, {
    renderer: new Term()
  });
  console.log(txt);
}); 
```

> 记住你需要在 `yield` 外面加上 `try` / `catch` 来捕获之前 promise 的错误处理。

像这样使用 generator 不易于扩展这一点是不言自明的。况且由于这种不是很自观的语法，你的迭代器代码需要和你使用的 generator 高度耦合。这意味着想要向你的 generator 中加入新的 `await` 表达式需要频繁修改代码。比较好的替代方法是使用即将到来的 **async 函数**。

# 使用 `async` / `await`

使用 _async 函数_时我们可以实现基于 Promise 的类似 generator 那样写同步代码的方式。另一个好处是你不需要修改 `getRandomPonyFooArticle`，只要它返回的是一个 promise，它就可以使用 await 获取.

**注意 `await` 只能用于标注了 `async` 关键字的函数内部**。它的工作方式类似 generator，在 promise 确定状态之前它的执行流程会挂起。如果 await 的不是 promise，会自动转化为一个 promise.

```javascript
read();

async function read () {
  var html = await getRandomPonyFooArticle();
  var md = hget(html, {
    markdown: true,
    root: 'main',
    ignore: '.at-subscribe,.mm-comments,.de-sidebar'
  });
  var txt = marked(md, {
    renderer: new Term()
  });
  console.log(txt);
} 
```
> 和 generator 一样，记住你应该把 `await` 部分放到 `try` / `catch` 里，用这种方式对 await 的那个 promise 进行错误捕获和处理。

另外，一个 _Async 函数_ 总是返回一个 `Promise`. 未捕获异常会被这个 promise reject，否则 promise 会 resolve 这个 `async` 函数的返回值。因此我们可以调用一个 `aysnc` 函数并将其和 promise 的链式调用方法相结合，接下来的实例看看怎么组合使用这两者 _[(参见 Babel REPL)](https://t.co/QgSKcLpww9)_.

```javascript
async function asyncFun () {
  var value = await Promise
    .resolve(1)
    .then(x => x * 3)
    .then(x => x + 5)
    .then(x => x / 2);
  return value;
}
asyncFun().then(x => console.log(`x: ${x}`));
// <- 'x: 4' 
```

回到前面一个例子，这意味着我们可以在 `async read` 函数里 `return txt`, 这样用户可以接着使用 promise 或者另一个 async 函数。这样你的 `read` 函数只需要关注怎么从 Pony Foo 获取一篇随机文章并转换为终端可读的 markdown 形式。

```javascript
async function read () {
  var html = await getRandomPonyFooArticle();
  var md = hget(html, {
    markdown: true,
    root: 'main',
    ignore: '.at-subscribe,.mm-comments,.de-sidebar'
  });
  var txt = marked(md, {
    renderer: new Term()
  });
  return txt;
} 
```
然后，你可以进一步在另一个 _Async 函数_里 `await read()`.

```javascript
async function write () {
  var txt = await read();
  console.log(txt);
} 
```

或者直接使用 promise 以进行更多后续处理。

```javascript
read().then(txt => console.log(txt));
```

# 重要抉择

在异步代码流程中，并行执行两个甚至多个任务的情形十分常见。**Async 函数**使编写异步代码变得简单，同时它们也可以用在**串行**的代码中，亦即，那些**同一时间只执行一个操作**的代码。内部包含多个 `await` 表达式的函数，在每个 `await` 表达式处都会挂起，直到 `Promise` 的状态确定并继续执行到下一个`await` 表达式——_这和我们观察到的 generator 和 `yield` 的行为略有不同_。

绕开这一点的办法是使用 [`Promise.all`](https://ponyfoo.com/articles/es6-promises-in-depth#leveraging-promiseall-and-promiserace) 创建一个单独的 promise ，然后 `await` 这个 promise. 当然，最大的问题是培养使用 `Promise.all` 的习惯而不是让所有事情都序列执行，后者会拖累你代码的性能表现。

接下来的例子展示如何 `await` 三个不同的 promise，同时让它们完全可以并发执行。 `await` 会挂起你的 `async` 函数并且 `await Promise.all` 表达式最终会 resolve 为一个 `results` 数组，我们可以通过解构拿到数组里单独的每一个结果。

```javascript
async function concurrent () {
  var [r1, r2, r3] = await Promise.all([p1, p2, p3]);
} 
```

在某段历史时期，上面的代码可以用 `await*` 来实现，你不需要将你的 promise 用 `Promise.all` 包起来，_Babel 5_ 支持这个特性。但是因为_[某些原因](https://github.com/rwaldron/tc39-notes/blob/aad8937063ab32eb33ec2a5b40325b1d9f171180/es6/2014-04/apr-10.md#preview-of-asnycawait)_**标准已经不再支持这种用法**了（Babel 6 也不支持）。

```javascript
async function concurrent () {
  var [r1, r2, r3] = await* [p1, p2, p3];
} 
```

你仍然可以使用 `all = Promise.all.bind(Promise)` 得到一个简洁版本的 `Promise.all`，你也可以对 `Promise.race` 做同样的事情，即便它没有类似 `await*` 的等同写法。

```javascript
const all = Promise.all.bind(Promise);
async function concurrent () {
  var [r1, r2, r3] = await all([p1, p2, p3]);
} 
```

# 异常处理

注意在一个 `async` 函数里**异常会被悄无声息地吞没**，就和在一个 Promise 里发生的一样。除非我们为 `await` 表达式显式加上 `try` / `catch` 块，否则未捕获异常——不管是在 `async` 函数体内部还是在 `await` 的挂起执行部分抛出——都会被 `async` 函数返回的 promise 直接 reject.

这里自然可以视为一个优点：你可以延续 `try` / `catch` 的传统，这在 callback 里是没法做的——而且在 promise 里不知怎么的又是可以的。在这一点上，_Async_ 函数和 [generator](https://ponyfoo.com/articles/es6-generators-in-depth) 是雷同的，你都可以使用 `try` / `catch`，因为它们都将异步流程通过执行函数挂起的方式变为同步代码。

更进一步，你还可以在 `async` 函数的外部捕获异常，只需要在返回的 promise 后面加上 `.catch` 语句即可。这种使用 `.catch` 语句实现 `try` / `catch` 的异常捕获机制是一种弹性的方式，但是可能让人感觉迷惑并最终导致异常没有被处理。

```javascript
read()
  .then(txt => console.log(txt))
  .catch(reason => console.error(reason)); 
```

我们有必要小心对待并学习通过不同的方法来对异常进行处理，记录以及避免它们。

# 今天使用 `async` / `await`

在你的代码里使用 Async 函数的一种方法是 Babel. 这涉及到一系列的模块，但是[你总是可以找到某一个模块](https://ponyfoo.com/articles/controversial-state-of-javascript-tooling)，如果你喜欢的话它会帮助你解决所有这些事情。我一般使用 [`npm-run`](https://github.com/timoxley/npm-run) 让所有模块都只需要装在本地。

```shell
npm i -g npm-run
npm i -D \
  browserify \
  babelify \
  babel-preset-es2015 \
  babel-preset-stage-3 \
  babel-runtime \
  babel-plugin-transform-runtime

echo '{
  "presets": ["es2015", "stage-3"],
  "plugins": ["transform-runtime"]
}' > .babelrc 
```

下面这条命令会把 `example.js` 通过 `browserify` 进行编译，并使用 `babelify` 使之支持 **Async 函数**。接着将代码 pipe 给 `node` 执行或者存储到磁盘文件。

```shell
npm-run browserify -t babelify example.js | node
```

# 参考阅读

[**Async 函数**标准草案](https://tc39.github.io/ecmascript-asyncawait/) 并不长，如果你想了解更多关于这一新特性，阅读它会是有趣的体验。

我粘贴了一段代码在下面，以帮助你理解 `async` 函数的内部是怎样工作的。即便我们不能使用新关键词的 polyfill，但是理解 `async` / `await` 背后的原理对你仍然是有帮助的。

> 换句话说，学习 Async 函数对使用 **generator 和 promise** 绝对是有帮助的。

以下代码展示了怎样把一个 `async function` 声明转换成普通的 `function`，它返回将一个 generator 作为参数传递给 `spawn` 的调用结果，其内部的 `await` 在句法上和 `yield` 完全等同。

```javascript
async function example (a, b, c) {
  example function body
}

function example (a, b, c) {
  return spawn(function* () {
    example function body
  }, this);
}
```

在 `spawn` 里返回了一个 promise, 它封装了将 generator 函数——_源于用户代码_——进行逐步迭代的一段代码，串行的传递值给你的 _"generator" 代码_（`async` 函数的函数体）。我们可以发现 _Async 函数_其实就是基于 generator 和 promise 的语法糖，这一点对于我们更好的理解这些概念的工作原理以方便我们更好的混合，匹配以及组合这些不同的异步代码流程的使用是非常重要的。

```javascript
function spawn (genF, self) {
  return new Promise(function (resolve, reject) {
    var gen = genF.call(self);
    step(() => gen.next(undefined));
    function step (nextF) {
      var next;
      try {
        next = nextF();
      } catch(e) {
        // finished with failure, reject the promise
        reject(e);
        return;
      }
      if (next.done) {
        // finished with success, resolve the promise
        resolve(next.value);
        return;
      }
      // not finished, chain off the yielded promise and `step` again
      Promise.resolve(next.value).then(
        v => step(() => gen.next(v)),
        e => step(() => gen.throw(e))
      );
    }
  });
} 
```

> 这些代码能帮助你更好的理解 `async` / `await` 怎么对 generator 序列进行迭代求值，并封装到一个 promise 里的。每一步的 promise 会串接成一个 promise 链，直到序列结束或者某个 promise 被 reject，使得整个 generator 函数返回的 promise 的状态被确定。
