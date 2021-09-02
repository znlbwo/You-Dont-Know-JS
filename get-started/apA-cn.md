# You Don't Know JS Yet: Get Started - 2nd Edition
# 附录 A：进一步探索

在本附录中，我们将对主要章节文本中的一些主题进行更详细的探讨。把这些内容看作是对本书其余部分所涉及的一些更细微的细节的选择性预览。

## Values vs. References

在第二章中，我们介绍了两种主要的数值类型：primitives and objects。但我们还没有讨论这两者之间的一个关键区别：这些值是如何分配和传递的。

在许多语言中，开发者可以选择将一个值作为值本身，或作为值的引用进行分配/传递。然而，在 JS 中，这个决定完全由值的种类决定。这让很多来自其他语言的开发者在开始使用 JS 时感到惊讶。

如果你 分配/传递 一个值本身，这个值就被复制了。比如说

```js
var myName = "Kyle";

var yourName = myName;
```

这里，`yourName` 变量有一个单独的 “Kyle” 字符串副本，与存储在 `myName` 的值不同。这是因为该值是一个 primitive，而 primitive 值总是作为 *值副本* 被 分配/传递。

下面是如何证明有两个独立的值。

```js
var myName = "Kyle";

var yourName = myName;

myName = "Frank";

console.log(myName);
// Frank

console.log(yourName);
// Kyle
```

看到 `yourName` 没有受到 `myName` 重新分配到 `"Frank”` 的影响吗？这是因为每个变量都持有自己的值的副本。

相比之下，引用是指两个或更多的变量指向同一个值，这样，修改这个共享的值就会被通过任何一个引用的访问所反映。在 JS 中，只有对象值（数组、对象、函数等）被视为引用。

考虑一下。

```js
var myAddress = {
    street: "123 JS Blvd",
    city: "Austin",
    state: "TX"
};

var yourAddress = myAddress;

// I've got to move to a new house!
myAddress.street = "456 TS Ave";

console.log(yourAddress.street);
// 456 TS Ave
```

因为分配给 `myAddress` 的值是一个对象，它是通过引用 持有/分配 的，因此分配给 `yourAddress` 变量的是一个引用的拷贝，而不是对象值本身。这就是为什么当我们访问 `yourAddress.street` 时，分配给 `myAddress.street` 的更新值被反映出来。`myAddress` 和`yourAddress` 有单个共享对象的引用副本，所以对一个对象的更新就是对两个对象的更新。

同样，JS 根据值类型来选择值拷贝和引用拷贝的行为。Primitives 是通过值持有的，对象是通过引用持有的。在 JS 中没有办法覆盖这一点，无论哪个方向。

## 这么多的函数形式

回顾一下第二章中 “函数” 部分的这个片段。

```js
var awesomeFunction = function(coolThings) {
    // ..
    return amazingStuff;
};
```

这里的函数表达式被称为 *匿名函数表达式*，因为它在 `function` 关键字和 `(..)` 参数列表之间没有名称标识。这一点让很多 JS 开发者感到困惑，因为从 ES6 开始，JS 对匿名函数进行了 “名称推断”。

```js
awesomeFunction.name;
// "awesomeFunction"
```

一个函数的 `name` 属性将显示其直接给出的名称（在声明的情况下）或在匿名函数表达式的情况下推断出的名称。在检查函数值或报告错误堆栈跟踪时，该值通常被开发者工具使用。

因此，即使是一个匿名函数表达式也 *可能* 得到一个名字。然而，名字推理只发生在有限的情况下，比如当函数表达式被赋值时（用 `=`）。例如，如果你把一个函数表达式作为参数传递给一个函数调用，就不会发生名称推断；`name` 属性将是一个空字符串，并且开发者控制台通常会报告“（匿名函数）”。

即使推断出一个名字，**它仍然是一个匿名函数。** 为什么？因为推断出来的名字是一个元数据字符串值，而不是一个可用来指代函数的标识符。一个匿名函数没有一个标识符可以用来从自身内部引用它，用于递归、事件解除绑定等。

比较一下匿名函数的表达形式。

```js
// let awesomeFunction = ..
// const awesomeFunction = ..
var awesomeFunction = function someName(coolThings) {
    // ..
    return amazingStuff;
};

awesomeFunction.name;
// "someName"
```

这个函数表达式是一个 *命名的函数表达式*，因为标识符 `someName` 在编译时直接与函数表达式相关联；与标识符 `awesomeFunction` 的相关联仍然要到运行时该语句时才发生。这两个标识符不一定要匹配；有时让它们不同是有意义的，其他时候最好让它们相同。

还要注意的是，在为 “name” 属性指定 *name* 时，明确的函数名，即标识符 `someName`，具有优先权。

函数表达式应该是命名的还是匿名的？在这个问题上，人们的意见大相径庭。大多数开发者倾向于不关心使用匿名函数的问题。它们更短，而且在广泛的 JS 代码领域无疑更常见。

在我看来，如果一个函数存在于你的程序中，它就有一个目的；否则，就把它拿掉！如果它有一个目的，就把它拿掉。如果它有一个目的，它就有一个自然的名字来描述这个目的。

如果一个函数有一个名字，你这个代码作者应该在代码中包含这个名字，这样读者就不必通过阅读和心理执行该函数的源代码来推断这个名字。即使是像 `x * 2` 这样的微不足道的函数体，也必须通过阅读来推断出一个像 “double” 或 “multBy2” 这样的名字；当你只需花一秒钟将函数命名为 “double” 或 “multBy2” *一次*，省去读者今后每次阅读时的重复脑力劳动时，这种简短的额外脑力劳动是不必要的。

遗憾的是，在某些方面，截至 2020 年初，JS 中还有许多其他的函数定义形式（也许将来会更多！）。

下面是一些更多的声明形式。

```js
// generator function declaration
function *two() { .. }

// async function declaration
async function three() { .. }

// async generator function declaration
async function *four() { .. }

// named function export declaration (ES6 modules)
export function five() { .. }
```

这里还有一些（很多！）函数表达形式。

```js
// IIFE
(function(){ .. })();
(function namedIIFE(){ .. })();

// asynchronous IIFE
(async function(){ .. })();
(async function namedAIIFE(){ .. })();

// arrow function expressions
var f;
f = () => 42;
f = x => x * 2;
f = (x) => x * 2;
f = (x,y) => x * y;
f = x => ({ x: x * 2 });
f = x => { return x * 2; };
f = async x => {
    var y = await doSomethingAsync(x);
    return y * 2;
};
someOperation( x => x * 2 );
// ..
```

请记住，箭头函数表达式是 *交互式匿名* 的，这意味着该语法不提供为函数提供直接名称标识的方法。函数表达式可能会得到一个推断的名字，但只有当它是赋值形式之一，而不是作为函数调用参数传递的（更常见！）形式（如片段的最后一行）。

因为我认为匿名函数在你的程序中经常使用不是一个好主意，所以我不喜欢使用 `=>` 箭头函数形式。这种函数实际上有一个特定的用途（即以词法处理 `this` 关键字），但这并不意味着我们应该把它用于我们编写的每个函数。为每项工作使用最合适的工具。

函数也可以在类定义和对象字面定义中指定。在这些形式中，它们通常被称为 “方法”，尽管在 JS 中，这个术语与 “函数” 没有什么明显的区别。

```js
class SomethingKindaGreat {
    // class methods
    coolMethod() { .. }   // no commas!
    boringMethod() { .. }
}

var EntirelyDifferent = {
    // object methods
    coolMethod() { .. },   // commas!
    boringMethod() { .. },

    // (anonymous) function expression property
    oldSchool: function() { .. }
};
```

唷! 这是很多种定义函数的方法。

这里没有简单的捷径；你只需要熟悉所有的函数形式，这样你就可以在现有的代码中识别它们，并在你写的代码中适当地使用它们。仔细研究它们，并加以练习!

## 强制条件比较

是的，这一节的名字很拗口。但我们在说什么呢？我们说的是条件表达式需要进行面向强制的比较来做出决定。

`if` 和 `? :` 三元组语句，以及 `while` 和 `for` 循环中的测试子句，都执行隐含的值比较。但是是哪一种呢？是 “严格的” 还是 “强制的”？实际上，两者都有。

考虑一下。

```js
var x = 1;

if (x) {
    // will run!
}

while (x) {
    // will run, once!
    x = false;
}
```

你可以这样看待这些 `(x)` 条件表达式。

```js
var x = 1;

if (x == true) {
    // will run!
}

while (x == true) {
    // will run, once!
    x = false;
}
```

在这个特定的情况下 — `x` 的值是 `1` — 这个心理模型是有效的，但它在更广泛的范围内是不准确的。考虑一下。

```js
var x = "hello";

if (x) {
    // will run!
}

if (x == true) {
    // won't run :(
}
```

糟糕了。那么，`if` 语句实际上在做什么？这是更准确的心理模型。

```js
var x = "hello";

if (Boolean(x) == true) {
    // will run
}

// which is the same as:

if (Boolean(x) === true) {
    // will run
}
```

由于 `Boolean(..)` 函数总是返回一个布尔类型的值，所以这个片段中的 `==` 与 `===` 并不相关；它们都会做同样的事情。但重要的是，在比较之前，发生了一个强制作用，从当前的 `x` 类型到布尔类型。

在 JS 的比较中，你是无法摆脱强制因素的。认真学习一下吧。

## 原型的 “类”

在第三章中，我们介绍了原型，并展示了我们如何通过一个原型链来连接对象。

另一种连接这种原型链的方式是 ES6 “类” 系统（见第 2 章，“类”）的优雅的前身（说实话，很丑），它被称为原型类。

| 小贴士： |
| :--- |
| 虽然这种风格的代码在现在的 JS 中很不常见，但在求职面试中被问到这种问题还是令人费解的，相当常见! |

让我们首先回顾一下 `Object.create(..)` 的编码风格。

```js
var Classroom = {
    welcome() {
        console.log("Welcome, students!");
    }
};

var mathClass = Object.create(Classroom);

mathClass.welcome();
// Welcome, students!
```

在这里，一个 `mathClass` 对象通过它的原型链接到一个 `Classroom` 对象。通过这种联系，函数调用 `mathClass.welcome()` 被委托给 `Classroom` 上定义的方法。

原型类模式会将这种委托行为标记为 “继承”，或者将其定义为（具有相同的行为）。

```js
function Classroom() {
    // ..
}

Classroom.prototype.welcome = function hello() {
    console.log("Welcome, students!");
};

var mathClass = new Classroom();

mathClass.welcome();
// Welcome, students!
```

所有的函数都默认在一个名为 `prototype` 的属性处引用一个空对象。尽管命名很混乱，但这并 **不是** 函数的 *原型*（函数的原型被链接到这里），而是当用 `new` 调用函数创建其他对象时要 *链接的* 原型对象。

我们在那个空对象（称为 `Classroom.prototype`）上添加一个 `welcome` 属性，指向 `hello()` 函数。

然后 `new Classroom()` 创建一个新的对象（分配给 `mathClass`），并将其原型链接到现有的 `Classroom.prototype` 对象。

尽管 `mathClass` 没有 `welcome()` 属性/函数，但它成功地委托给了 `Classroom.prototype.welcome()` 函数。

这种 “原型类” 模式现在被强烈反对，而倾向于使用 ES6 的 `class` 机制。

```js
class Classroom {
    constructor() {
        // ..
    }

    welcome() {
        console.log("Welcome, students!");
    }
}

var mathClass = new Classroom();

mathClass.welcome();
// Welcome, students!
```

掩盖之下，同样的原型链接被连接起来，但这种 `class` 语法符合面向类的设计模式，比 “原型类” 更干净。
