# You Don't Know JS Yet: Get Started - 2nd Edition
# 第三章：挖掘 JS 的根基

如果你已经阅读了第 1 章和第 2 章，并花时间进行了消化和消化，你希望你开始 *get* 到一点 JS 了。如果你跳过了这些章节（尤其是第 2  章），我建议你回去花更多的时间阅读这些材料。

在第 2 章中，我们对语法、模式和行为进行了高水平的调查。在这一章中，我们的注意力转移到 JS 的一些低层次的根本特征上，这些特征几乎是我们写的每一行代码的基础。

请注意：这一章比你习惯于思考的编程语言要深得多。我的目标是帮助你理解 JS 工作的核心，是什么让它运转。这一章应该开始回答一些你在探索 JS 时可能会出现的 “为什么” 的问题。然而，这些材料仍然不是对该语言的详尽阐述；这就是本书系列的其他部分的作用 我们在这里的目标仍然是 *开始*，并且更加适应 JS 的 *感觉*，它是如何起伏的。

不要在这些材料中跑得太快，以至于你在杂草中迷失了方向。正如我已经说过十几次了，**要慢慢来**。即使如此，你在读完这一章后可能还会有其他问题。这没关系，因为还有一整个系列的书在等着你继续探索呢

## 迭代

由于程序本质上是为了处理数据（并对这些数据做出决定），所以用于处理数据的模式对程序的可读性有很大影响。

迭代器模式已经存在了几十年，它提出了一种 “标准化” 的方法，从一个源头一次 *chunk* 地消耗数据。这个想法是，对数据源进行迭代 — 通过处理第一部分，然后是下一部分，以此类推，逐步处理数据集合，而不是一次性处理整个数据集，这样做更常见，更有帮助。

想象一下一个数据结构，它代表了一个关系数据库的 `SELECT` 查询，它通常将结果组织成行。如果这个查询只有一条或几条记录，你可以一次性处理整个结果集，并将每条记录分配给一个局部变量，然后对这些数据进行适当的操作。

但是如果查询有 100 或 1000（或更多！）行，你就需要迭代处理这些数据（通常是一个循环）。

迭代器模式定义了一个叫做 “迭代器” 的数据结构，它有一个对底层数据源（如查询结果行）的引用，它暴露了一个像 `next()` 的方法。调用 `next()` 会返回下一块数据（即数据库查询中的 “记录” 或 “行”）。

你并不总是知道你需要遍历多少个数据，所以一旦你遍历了整个集合并 *过了终点*，该模式通常以一些特殊的值或异常来表示完成。

迭代器模式的重要性在于坚持用 *标准* 的方式来迭代处理数据，这样可以创造出更干净、更容易理解的代码，而不是让每个数据 结构/源 都定义自己的自定义方式来处理数据。

经过多年来 JS 社区围绕共同商定的迭代技术所做的各种努力，ES6 直接在语言中为迭代器模式规范了一个特定的协议。该协议定义了一个 `next()` 方法，其返回值是一个叫做 *iterator result* 的对象；该对象有 `value` 和 `done` 属性，其中 `done` 是一个布尔值，在对底层数据源的迭代完成之前是 `false`。

### 消耗迭代器

有了 ES6 的迭代协议，每次消耗一个数据源的值是可行的，在每次 `next()` 调用后检查 `done` 是否为 `true` 以停止迭代。但这种方法是相当手动的，所以 ES6 也包括了几个机制（语法和 API），用于标准化地消费这些迭代器。

其中一个机制是 “for…of” 循环：

```js
// given an iterator of some data source:
var it = /* .. */;

// loop over its results one at a time
for (let val of it) {
    console.log(`Iterator value: ${ val }`);
}
// Iterator value: ..
// Iterator value: ..
// ..
```

| 注意: |
| :--- |
| 我们在这里省略了手动循环的等价物，但它的可读性肯定比 `for…of` 循环要差! |

另一个经常用于消耗迭代器的机制是 `…` 操作符。这个操作符实际上有两种对称的形式。*spread* 和 *rest*（或 *gather*，我更喜欢）。*spread* 形式是一个迭代器消费者。

要 *spread* 一个迭代器，你必须要有 *东西* 来传播它。在 JS 中有两种可能性：一个数组或一个函数调用的参数列表。

一个数组的传播。

```js
// spread an iterator into an array,
// with each iterated value occupying
// an array element position.
var vals = [ ...it ];
```

一个函数调用传播。

```js
// spread an iterator into a function,
// call with each iterated value
// occupying an argument position.
doSomethingUseful( ...it );
```

在这两种情况下，iterator-spread 形式的 `…` 遵循 iterator-consumption 协议（与 `for…of` 循环相同），从 iterator 中获取所有可用的值，并将其放入（又称传播）接收上下文（数组、参数列表）。

### 迭代器

迭代器消费协议在技术上是为消费 *迭代器* 而定义的；一个迭代器是一个可以被迭代的值。

该协议会自动从一个可迭代的值中创建一个迭代器实例，并且只消耗 *该迭代器实例* 来完成它。这意味着一个迭代器可以被消耗一次以上；每次都会创建并使用一个新的迭代器实例。

那么，我们在哪里可以找到迭代器呢？

ES6 将 JS 中的基本数据 结构/集合 类型定义为 iterable。这包括 strings, arrays, maps, sets 和其他。

考虑一下。

```js
// an array is an iterable
var arr = [ 10, 20, 30 ];

for (let val of arr) {
    console.log(`Array value: ${ val }`);
}
// Array value: 10
// Array value: 20
// Array value: 30
```

由于数组是可迭代的，我们可以通过 `…` 传播操作符，使用迭代器消费浅层复制一个数组。

```js
var arrCopy = [ ...arr ];
```

We can also iterate the characters in a string one at a time:

```js
var greeting = "Hello world!";
var chars = [ ...greeting ];

chars;
// [ "H", "e", "l", "l", "o", " ",
//   "w", "o", "r", "l", "d", "!" ]
```

一个 `Map` 数据结构使用对象作为键，将一个值（任何类型）与该对象相关联。Maps 有一个不同于这里的默认迭代，因为迭代不仅仅是在 Map 的值上，而是在它的 *entries* 上。一个 *entry* 是一个元组（2 元素数组），包括一个键和一个值。

考虑一下。

```js
// given two DOM elements, `btn1` and `btn2`

var buttonNames = new Map();
buttonNames.set(btn1,"Button 1");
buttonNames.set(btn2,"Button 2");

for (let [btn,btnName] of buttonNames) {
    btn.addEventListener("click",function onClick(){
        console.log(`Clicked ${ btnName }`);
    });
}
```

在默认地图迭代的 `for…of` 循环中，我们使用 `[btn,btnName]` 语法（称为 “数组析构”），将每个消耗的元组分解为各自的键/值对（`btn1` / `"Button 1”` 和 `btn2` / `"Button 2"`）。

JS 中的每个内置迭代器都暴露了一个默认的迭代，这可能与你的直觉相符。但如果有必要，你也可以选择一个更具体的迭代。例如，如果我们想只消耗上述 “buttonNames” 映射的值，我们可以调用 “values()” 来获得一个只用值的迭代器。

```js
for (let btnName of buttonNames.values()) {
    console.log(btnName);
}
// Button 1
// Button 2
```

或者如果我们想在数组迭代中获得索引 *和* 值，我们可以用 `entries()` 方法制作一个条目迭代器。

```js
var arr = [ 10, 20, 30 ];

for (let [idx,val] of arr.entries()) {
    console.log(`[${ idx }]: ${ val }`);
}
// [0]: 10
// [1]: 20
// [2]: 30
```

在大多数情况下，JS 中所有的内置迭代器都有三种迭代器形式：纯键（`keys()`），纯值（`values()`），和条目（`entries()`）。

除了使用内置的迭代器，你还可以确保你自己的数据结构遵守迭代协议；这样做意味着你选择了用 `for…of` 循环和 `…` 操作符消耗你的数据的能力。该协议的 “标准化” 意味着代码整体上更容易被识别和阅读。

| 注意: |
| :--- |
| 你可能已经注意到在这次讨论中发生了一个细微的变化。我们一开始讨论的是消耗 **迭代器**，但后来转而讨论在 **迭代器** 上进行迭代。迭代消费协议期望一个 **迭代器**，但是我们可以提供一个直接的 **迭代器** 的原因是，一个迭代器只是它本身的一个迭代器，当从一个现有的迭代器创建一个迭代器实例时，迭代器本身被返回。|

## 闭包

也许在不知不觉中，几乎每个 JS 开发者都使用过闭包。事实上，闭包是大多数语言中最普遍的编程功能之一。它甚至可能和变量或循环一样重要，这就是它的基础。

然而，它给人的感觉是隐藏的，几乎是神奇的。而且，人们经常用非常抽象或非常不正式的术语来谈论它，这几乎不能帮助我们确定它到底是什么。

我们需要能够识别程序中使用闭合的地方，因为闭合的存在或缺乏有时是导致错误的原因（甚至是性能问题的原因）。

因此，让我们以一种务实而具体的方式来定义闭合。

> 闭包是指一个函数记住并继续访问其作用域外的变量，即使该函数在不同的作用域中被执行。

我们在这里看到两个定义上的特点。首先，封闭是函数性质的一部分。对象不会得到闭包，而函数会。第二，为了观察闭包，你必须在不同的作用域中执行一个函数，而不是该函数最初定义的地方。

考虑一下。

```js
function greeting(msg) {
    return function who(name) {
        console.log(`${ msg }, ${ name }!`);
    };
}

var hello = greeting("Hello");
var howdy = greeting("Howdy");

hello("Kyle");
// Hello, Kyle!

hello("Sarah");
// Hello, Sarah!

howdy("Grant");
// Howdy, Grant!
```

首先，`greeting(..)` 外部函数被执行，创建一个内部函数 `who(..)` 的实例；该函数关闭了变量 `msg`，它是来自 `greeting(..)` 外部范围的参数。当这个内部函数被返回时，它的引用被分配给外部作用域中的 `hello` 变量。然后我们第二次调用 `greeting(..)`，创建一个新的内部函数实例，在一个新的 `msg` 上有一个新的闭包，并返回该引用以分配给 `howdy`。

当 `greeting(..)` 函数运行结束后，通常我们会期望它的所有变量都被垃圾回收（从内存中删除）。我们会期望每个 `msg` 消失，但它们没有。原因是闭包。由于内部函数实例仍然活着（分别分配给 `hello` 和 `howdy`），它们的闭包仍然保留着 `msg` 变量。

这些闭包不是 `msg` 变量值的快照；它们是变量本身的直接链接和保存。这意味着闭包实际上可以观察到（或进行！）这些变量随时间的更新。

```js
function counter(step = 1) {
    var count = 0;
    return function increaseCount(){
        count = count + step;
        return count;
    };
}

var incBy1 = counter(1);
var incBy3 = counter(3);

incBy1();       // 1
incBy1();       // 2

incBy3();       // 3
incBy3();       // 6
incBy3();       // 9
```

内部 `increaseCount()` 函数的每个实例在其外部 `counter(..)` 函数的范围内对 `count` 和 `step` 两个变量进行封闭。`step` 一直保持不变，但 `count` 在每次调用该内部函数时被更新。由于闭包是针对变量的，而不仅仅是数值的快照，所以这些更新被保留下来。

闭包在处理异步代码时最为常见，例如回调。考虑一下。

```js
function getSomeData(url) {
    ajax(url,function onResponse(resp){
        console.log(
            `Response (from ${ url }): ${ resp }`
        );
    });
}

getSomeData("https://some.url/wherever");
// Response (from https://some.url/wherever): ...
```

内部函数 `onResponse(..)` 是在 `url` 上关闭的，因此保留并记住了它，直到 Ajax 调用返回并执行 `onResponse(..)`。即使 `getSomeData(…)` 马上就结束了，`url` 参数变量在闭包中保持活力，只要需要就可以。

外层作用域不一定是一个函数 — 通常是，但不一定是 — 只需要在外层作用域中至少有一个变量从内部函数访问。

```js
for (let [idx,btn] of buttons.entries()) {
    btn.addEventListener("click",function onClick(){
       console.log(`Clicked on button (${ idx })!`);
    });
}
```

因为这个循环使用了 `let` 声明，每次迭代都会得到新的块范围（也就是本地）的 `idx` 和 `btn` 变量；这个循环每次都会创建一个新的内部 `onClick(..)` 函数。这个内部函数关闭了 `idx`，只要 `btn` 上的点击处理程序被设置，它就一直保留着。所以当每个按钮被点击时，它的处理程序可以打印其相关的索引值，因为处理程序记住了其各自的`idx`变量。

记住：这个闭包不是在值上（像 `1` 或 `3`），而是在变量 `idx` 本身上。

闭包是任何语言中最普遍和最重要的编程模式之一。但在 JS 中尤其如此；如果不以这样或那样的方式利用闭包，很难想象会有什么有用的事情发生。

如果你仍然对关闭感到不清楚或摇摆不定，第二册的大部分内容，*范围和关闭* 都集中在这个主题上。

## `this` 关键字

JS 最强大的机制之一也是最被误解的机制之一：`this` 关键字。一个常见的误解是，一个函数的 `this` 指的是函数本身。由于 `this` 在其他语言中的工作方式，另一个误解是 `this` 指向方法所属的实例。这两种说法都是不正确的。

正如前面所讨论的，当一个函数被定义时，它是通过闭包来 *附属* 到它的封闭作用域的。作用域是控制如何解决对变量的引用的一组规则。

但是，除了它们的作用域之外，函数还有另一个特性，影响着它们可以访问的内容。这个特性最好被描述为 *执行上下文*，它通过其 `this` 关键字暴露给函数。

作用域是静态的，包含了在你定义一个函数的时刻和位置上可用的一组固定的变量，但是一个函数的执行 *上下文* 是动态的，完全取决于 **它是如何被调用的**（不管它是在哪里定义的，甚至是从哪里调用的）。

`this` 不是一个基于函数定义的固定特性，而是一个动态特性，在每次函数被调用时都会确定。

思考 *执行上下文* 的一种方式是，它是一个有形的对象，其属性在函数执行时被提供给它。与作用域相比，它也可以被认为是一个 *对象*；只不过，*作用域对象* 隐藏在 JS 引擎内部，它对该函数来说总是相同的，它的 *属性* 采取的是函数内部可用的标识符变量的形式。

```js
function classroom(teacher) {
    return function study() {
        console.log(
            `${ teacher } says to study ${ this.topic }`
        );
    };
}
var assignment = classroom("Kyle");
```

外部的 `classroom(..)` 函数没有引用 `this` 关键字，所以它就像我们到目前为止看到的其他函数一样。但是内部的 `study()` 函数确实引用了 `this`，这使得它成为一个 `this` 感知的函数。换句话说，它是一个依赖于其 *执行上下文* 的函数。

| 注意: |
| :--- |
| `study()` 也是在其外部作用域关闭的 `teacher` 变量。|

由 `classroom(“Kyle")` 返回的内部 `study()` 函数被分配给一个名为 `assignment` 的变量。那么如何调用 `assignment()`（又名 `study()`）呢？

```js
assignment();
// Kyle says to study undefined  -- Oops :(
```

在这个片段中，我们把`assignment()`作为一个普通的、正常的函数来调用，没有给它提供任何 *执行上下文*。

由于这个程序不是在严格模式下（见第 1 章，“严格说”），在 **没有指定任何上下文** 的情况下调用的上下文感知函数默认为全局对象（浏览器中的 `window`）。由于没有名为 “topic” 的全局变量（因此全局对象上也没有这样的属性），“this.topic” 被解析为 `undefined`。

现在考虑。

```js
var homework = {
    topic: "JS",
    assignment: assignment
};

homework.assignment();
// Kyle says to study JS
```

`assignment` 函数引用的副本被设置为 `homework` 对象的一个属性，然后以 `homework.assignment()` 的形式调用它。这意味着该函数调用的 `this` 将是 `homework` 对象。因此，`this.topic` 被解析为 `"JS"`。

最后。

```js
var otherHomework = {
    topic: "Math"
};

assignment.call(otherHomework);
// Kyle says to study Math
```

调用函数的第三种方法是使用 `call(..)` 方法，它接收一个对象（这里是 `otherHomework`），用于设置函数调用的 `this` 引用。属性引用 `this.topic` 解析为`"Math"`。

同样的上下文感知函数以三种不同的方式调用，每次都会给出不同的答案，即 `this` 将引用什么对象。

`this` 感知函数及其动态上下文的好处是能够更灵活地用不同对象的数据重复使用一个函数。一个在一个作用域上关闭的函数永远无法引用不同的作用域或变量集。但是一个具有动态 `this` 上下文意识的函数对于某些任务来说是很有帮助的。

## 原型

当 `this` 是函数执行的一个特征时，原型是一个对象的特征，特别是对一个属性访问的解析。

可以把原型看作是两个对象之间的联系；这种联系隐藏在幕后，尽管有一些方法可以暴露和观察它。这种原型链接发生在一个对象被创建时；它被链接到另一个已经存在的对象。

一系列通过原型链接在一起的对象被称为 “原型链”。

这种原型链接（即从一个对象 B 到另一个对象 A）的目的是为了使针对 B 的 属性/方法 的访问被 *委托* 给 A 来处理。属性/方法 访问的委托允许两个（或更多！）对象相互合作来完成一项任务。

考虑将一个对象定义为一个普通的字面。

```js
var homework = {
    topic: "JS"
};
```

`homework` 对象上只有一个属性：`topic`。然而，它的默认原型链接连接到 `Object.prototype` 对象，该对象上有常见的内置方法，如 `toString()` 和 `valueOf()`，等等。

我们可以观察到这个原型链接从 `homework` 到 `Object.prototype` 的 *委托*。

```js
homework.toString();    // [object Object]
```

尽管 `homework` 没有定义 `toString()` 方法，但 `homework.toString()` 还是起作用了；这个委托调用了 `Object.prototype.toString()`。

### 对象链接

要定义一个对象的原型链接，你可以使用 `Object.create(..)` 工具来创建这个对象。

```js
var homework = {
    topic: "JS"
};

var otherHomework = Object.create(homework);

otherHomework.topic;   // "JS"
```

`Object.create(..)` 的第一个参数指定了一个对象来链接新创建的对象，然后返回新创建（和链接！）的对象。

图 4 显示了三个对象（`otherHomework`, `homework`, 和 `Object.prototype`）是如何在一个原型链中被链接的。

<figure>
    <img src="images/fig4.png" width="200" alt="Prototype chain with 3 objects" align="center">
    <figcaption><em>Fig. 4: Objects in a prototype chain</em></figcaption>
    <br><br>
</figure>

通过原型链进行的委托只适用于查询属性中的值的访问。如果你对一个对象的属性进行赋值，这将直接适用于该对象，无论该对象的原型链接到哪里。

| 提示： |
| :--- |
| `Object.create(null)` 创建一个没有原型链接的对象，所以它纯粹是一个独立的对象；在某些情况下，这可能是最好的。|

考虑一下。

```js
homework.topic;
// "JS"

otherHomework.topic;
// "JS"

otherHomework.topic = "Math";
otherHomework.topic;
// "Math"

homework.topic;
// "JS" -- not "Math"
```

对 `topic` 的赋值直接在 `otherHomework` 上创建了一个该名称的属性；对 `homework` 上的 `topic` 属性没有影响。接下来的语句访问了 `otherHomework.topic`，我们从这个新属性中看到了非授权的答案。`"Math"`。

图 5 显示了创建 `otherHomework.topic` 属性的赋值之后的 对象/属性。

<figure>
    <img src="images/fig5.png" width="200" alt="3 objects linked, with shadowed property" align="center">
    <figcaption><em>Fig. 5: Shadowed property 'topic'</em></figcaption>
    <br><br>
</figure>

`otherHomework` 上的 `topic` 正在 “影射” 链中 `homework` 对象上的同名属性。

| 注意: |
| :--- |
| 另一种坦率地说更复杂但也许仍然更常见的创建具有原型链接的对象的方式是使用 “prototypal class” 模式，从 ES6 中加入`class`（见第 2 章，“类”）之前就有了。我们将在附录 A “原型 ‘类’” 中更详细地介绍这个话题。|

### `this` 重新审视

我们在前面介绍了 `this` 关键字，但是当考虑到它是如何为原型委托的函数调用提供动力时，它的真正重要性就显现出来了。事实上，`this` 支持基于函数调用方式的动态上下文的主要原因之一，是为了让通过原型链委托的对象上的方法调用仍然保持预期的 `this`。

考虑一下。

```js
var homework = {
    study() {
        console.log(`Please study ${ this.topic }`);
    }
};

var jsHomework = Object.create(homework);
jsHomework.topic = "JS";
jsHomework.study();
// Please study JS

var mathHomework = Object.create(homework);
mathHomework.topic = "Math";
mathHomework.study();
// Please study Math
```

两个对象 `jsHomework` 和 `mathHomework` 各自的原型链接到单一的 `homework` 对象，后者有 `study()` 功能。`jsHomework` 和 `mathHomework` 各自被赋予自己的 `topic` 属性（见图 6）。

<figure>
    <img src="images/fig6.png" width="495" alt="4 objects prototype linked" align="center">
    <figcaption><em>Fig. 6: Two objects linked to a common parent</em></figcaption>
    <br><br>
</figure>

`jsHomework.study()` 委托给 `homework.study()`，但其执行的 `this`（`this.topic`）由于函数的调用方式而解析为 `jsHomework`，所以 `this.topic` 是 `"JS"`。同样，`mathHomework.study()` 委托给 `homework.study()`，但仍然将 `this` 解析为`mathHomework`，因此 `this.topic` 为 `"Math"`。

如果 `this` 被解析为 `homework`，前面的代码片断就不那么有用了。然而，在许多其他语言中，似乎 `this` 会是 `homework`，因为 `study()` 方法确实定义在 `homework` 上。

与其他许多语言不同，JS 的 `this` 是动态的，是允许原型委托的关键部分，事实上 `class` 也是如此，可以像预期的那样工作!

## 问 “为什么？”

本章的预期收获是，JS 的内部有很多东西比从表面上看出来的要多。

当你开始学习和了解 JS 时，你可以练习和加强的最重要的技能之一是好奇心，以及当你遇到语言中的某些东西时问 “为什么？”的艺术。

尽管本章对一些主题进行了相当深入的探讨，但许多细节仍被完全略过了。这里还有很多东西需要学习，而通往这些的道路是从你对你的代码提出 *正确* 的问题开始。提出正确的问题是成为一个更好的开发者的关键技能。

在本书的最后一章，我们将简要介绍 JS 是如何划分的，这在 *You Don't Know JS Yet* 系列图书的其他部分都有涉及。另外，不要跳过本书的附录 B，其中有一些练习代码来复习本书涉及的一些主要内容。
