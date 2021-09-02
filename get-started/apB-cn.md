# You Don't Know JS Yet: Get Started - 2nd Edition
# 附录 B：练习，练习，练习!

在这个附录中，我们将探讨一些练习和它们的建议解决方案。这些只是为了 *让你开始* 对书中的概念进行练习。

## 练习比较

让我们来练习一下值类型和比较的工作（第 4 章，支柱 3），在这里需要涉及到强制转换。

`scheduleMeeting(..)` 应该接受一个开始时间（24 小时格式的字符串 “hh:mm”）和一个会议持续时间（分钟数）。如果会议完全在工作日内（根据 `dayStart` 和 `dayEnd` 中指定的时间），它应该返回 `true`；如果会议违反了工作日的范围，则返回 `false`。

```js
const dayStart = "07:30";
const dayEnd = "17:45";

function scheduleMeeting(startTime,durationMinutes) {
    // ..TODO..
}

scheduleMeeting("7:00",15);     // false
scheduleMeeting("07:15",30);    // false
scheduleMeeting("7:30",30);     // true
scheduleMeeting("11:30",60);    // true
scheduleMeeting("17:00",45);    // true
scheduleMeeting("17:30",30);    // false
scheduleMeeting("18:00",15);    // false
```

先试着自己解决这个问题。考虑平等和关系比较运算符的用法，以及强制法如何影响这段代码。一旦你有了可行的代码，将你的解决方案与本附录末尾 “建议的解决方案” 中的代码进行 *比较*。

## 练习闭包

现在我们来练习一下闭合（第 4 章，支柱 1）。

`range(..)` 函数的第一个参数是一个数字，代表所需数字范围内的第一个数字。第二个参数也是一个数字，代表所需范围的终点（包括）。如果第二个参数被省略，那么应该返回另一个期望该参数的函数。

```js
function range(start,end) {
    // ..TODO..
}

range(3,3);    // [3]
range(3,8);    // [3,4,5,6,7,8]
range(3,0);    // []

var start3 = range(3);
var start4 = range(4);

start3(3);     // [3]
start3(8);     // [3,4,5,6,7,8]
start3(0);     // []

start4(6);     // [4,5,6]
```

先试着自己解决这个问题。

一旦你有了可以工作的代码，将你的解决方案与本附录末尾 “建议的解决方案” 中的代码进行 *比较*。

## 练习原型

最后，让我们研究一下 `this` 和通过原型链接的对象（第四章，支柱 2）。

定义一个有三个转轮的老虎机，可以单独 `旋转()`，然后 `显示()` 所有转轮的当前内容。

单个卷轴的基本行为在下面的 `reel` 对象中定义。但是老虎机需要单独的卷轴 — 委托给 `reel` 的对象，并且每个对象都有一个 `位置` 属性。

卷轴只 *知道如何* `显示()` 它当前的老虎机符号，但是老虎机通常在每个卷轴上显示三个符号：当前老虎机（`position`），上面的一个老虎机（`position - 1`），下面的一个老虎机（`position + 1`）。所以显示老虎机最终应该显示一个 3 x 3 的老虎机符号的网格。

```js
function randMax(max) {
    return Math.trunc(1E9 * Math.random()) % max;
}

var reel = {
    symbols: [
        "♠", "♥", "♦", "♣", "☺", "★", "☾", "☀"
    ],
    spin() {
        if (this.position == null) {
            this.position = randMax(
                this.symbols.length - 1
            );
        }
        this.position = (
            this.position + 100 + randMax(100)
        ) % this.symbols.length;
    },
    display() {
        if (this.position == null) {
            this.position = randMax(
                this.symbols.length - 1
            );
        }
        return this.symbols[this.position];
    }
};

var slotMachine = {
    reels: [
        // this slot machine needs 3 separate reels
        // hint: Object.create(..)
    ],
    spin() {
        this.reels.forEach(function spinReel(reel){
            reel.spin();
        });
    },
    display() {
        // TODO
    }
};

slotMachine.spin();
slotMachine.display();
// ☾ | ☀ | ★
// ☀ | ♠ | ☾
// ♠ | ♥ | ☀

slotMachine.spin();
slotMachine.display();
// ♦ | ♠ | ♣
// ♣ | ♥ | ☺
// ☺ | ♦ | ★
```

先试着自己解决这个问题。

提示。

* 使用 `%` modulo 操作符来包装 `position`，因为你是围绕着卷轴来访问符号的。

* 使用 `Object.create(..)` 来创建一个对象并将其原型链接到另一个对象。一旦链接，委托允许对象在方法调用时共享 `this` 上下文。

* 你可以使用另一个临时对象（`Object.create(..)` 再次），它有自己的 `position'，而不是直接修改卷轴对象来显示三个位置中的每一个。

一旦你有了可行的代码，将你的解决方案与本附录末尾 “建议的解决方案” 中的代码进行 *比较*。

## 建议的解决方案

请记住，这些建议的解决方案仅仅是：建议。有许多不同的方法来解决这些练习。将你的方法与你在这里看到的进行比较，并考虑每种方法的优点和缺点。

“比较”（支柱 3）练习的建议解决方案。

```js
const dayStart = "07:30";
const dayEnd = "17:45";

function scheduleMeeting(startTime,durationMinutes) {
    var [ , meetingStartHour, meetingStartMinutes ] =
        startTime.match(/^(\d{1,2}):(\d{2})$/) || [];

    durationMinutes = Number(durationMinutes);

    if (
        typeof meetingStartHour == "string" &&
        typeof meetingStartMinutes == "string"
    ) {
        let durationHours =
            Math.floor(durationMinutes / 60);
        durationMinutes =
            durationMinutes - (durationHours * 60);
        let meetingEndHour =
            Number(meetingStartHour) + durationHours;
        let meetingEndMinutes =
            Number(meetingStartMinutes) +
            durationMinutes;

        if (meetingEndMinutes >= 60) {
            meetingEndHour = meetingEndHour + 1;
            meetingEndMinutes =
                meetingEndMinutes - 60;
        }

        // re-compose fully-qualified time strings
        // (to make comparison easier)
        let meetingStart = `${
            meetingStartHour.padStart(2,"0")
        }:${
            meetingStartMinutes.padStart(2,"0")
        }`;
        let meetingEnd = `${
            String(meetingEndHour).padStart(2,"0")
        }:${
            String(meetingEndMinutes).padStart(2,"0")
        }`;

        // NOTE: since expressions are all strings,
        // comparisons here are alphabetic, but it's
        // safe here since they're fully qualified
        // time strings (ie, "07:15" < "07:30")
        return (
            meetingStart >= dayStart &&
            meetingEnd <= dayEnd
        );
    }

    return false;
}

scheduleMeeting("7:00",15);     // false
scheduleMeeting("07:15",30);    // false
scheduleMeeting("7:30",30);     // true
scheduleMeeting("11:30",60);    // true
scheduleMeeting("17:00",45);    // true
scheduleMeeting("17:30",30);    // false
scheduleMeeting("18:00",15);    // false
```

----

对 “结束”（第一支柱）实践的建议解决方案。

```js
function range(start,end) {
    start = Number(start) || 0;

    if (end === undefined) {
        return function getEnd(end) {
            return getRange(start,end);
        };
    }
    else {
        end = Number(end) || 0;
        return getRange(start,end);
    }


    // **********************

    function getRange(start,end) {
        var ret = [];
        for (let i = start; i <= end; i++) {
            ret.push(i);
        }
        return ret;
    }
}

range(3,3);    // [3]
range(3,8);    // [3,4,5,6,7,8]
range(3,0);    // []

var start3 = range(3);
var start4 = range(4);

start3(3);     // [3]
start3(8);     // [3,4,5,6,7,8]
start3(0);     // []

start4(6);     // [4,5,6]
```

----

对 “原型”（第二支柱）实践的建议解决方案。

```js
function randMax(max) {
    return Math.trunc(1E9 * Math.random()) % max;
}

var reel = {
    symbols: [
        "♠", "♥", "♦", "♣", "☺", "★", "☾", "☀"
    ],
    spin() {
        if (this.position == null) {
            this.position = randMax(
                this.symbols.length - 1
            );
        }
        this.position = (
            this.position + 100 + randMax(100)
        ) % this.symbols.length;
    },
    display() {
        if (this.position == null) {
            this.position = randMax(
                this.symbols.length - 1
            );
        }
        return this.symbols[this.position];
    }
};

var slotMachine = {
    reels: [
        Object.create(reel),
        Object.create(reel),
        Object.create(reel)
    ],
    spin() {
        this.reels.forEach(function spinReel(reel){
            reel.spin();
        });
    },
    display() {
        var lines = [];

        // display all 3 lines on the slot machine
        for (
            let linePos = -1; linePos <= 1; linePos++
        ) {
            let line = this.reels.map(
                function getSlot(reel){
                    var slot = Object.create(reel);
                    slot.position = (
                        reel.symbols.length +
                        reel.position +
                        linePos
                    ) % reel.symbols.length;
                    return slot.display();
                }
            );
            lines.push(line.join(" | "));
        }

        return lines.join("\n");
    }
};

slotMachine.spin();
slotMachine.display();
// ☾ | ☀ | ★
// ☀ | ♠ | ☾
// ♠ | ♥ | ☀

slotMachine.spin();
slotMachine.display();
// ♦ | ♠ | ♣
// ♣ | ♥ | ☺
// ☺ | ♦ | ★
```

本书的内容就到此为止。但现在是时候寻找真正的项目来实践这些想法了。继续编码吧，因为这是最好的学习方法。
