# JavaScript

JavaScript 最初被创建的目的是“使网页更生动”。

这种编程语言写出来的程序被称为 **脚本**。它们可以被直接写在网页的 HTML 中，在页面加载的时候自动执行。

脚本被以纯文本的形式提供和执行。它们不需要特殊的准备或编译即可运行。

> [!info] 为什么叫 JavaScript？
> 
> JavaScript 在刚诞生的时候，它的名字叫 “LiveScript”。但是因为当时 Java 很流行，所以决定将一种新语言定位为 Java 的“弟弟”会有助于它的流行。
> 
> 随着 JavaScript 的发展，它已经成为了一门完全独立的语言，并且也拥有了自己的语言规范 [ECMAScript](http://en.wikipedia.org/wiki/ECMAScript)。现在，**它和 Java 之间没有任何关系**。

如今，JavaScript 不仅可以在浏览器中执行，也可以在服务端执行，甚至可以在任意搭载了 [JavaScript 引擎](https://en.wikipedia.org/wiki/JavaScript_engine) 的设备中执行。

> [!tip] 
> 
> 浏览器中嵌入了 JavaScript 引擎，有时也称作“JavaScript 虚拟机”。
> 

> [!info] 引擎是如何工作的？
> 
> 引擎很复杂，但是基本原理很简单。
>
> 1. 引擎（如果是浏览器，则引擎被嵌入在其中）读取（“解析”）脚本。
> 2. 然后，引擎将脚本转化（“编译”）为机器语言。
> 3. 然后，机器代码快速地执行。
>
> 引擎会对流程中的每个阶段都进行优化。它甚至可以在编译的脚本运行时监视它，分析流经该脚本的数据，并根据获得的信息进一步优化机器代码
> 

不同的人想要不同的功能。JavaScript 的语法也不能满足所有人的需求。因此，最近出现了许多新语言，这些语言在浏览器中执行之前，都会被 **编译**（转化）成 JavaScript。

- [TypeScript](https://www.typescriptlang.org/) 专注于添加“严格的数据类型”以简化开发，以更好地支持复杂系统的开发。由微软开发。

> [!tip] 规范与手册
> 
> 规范：**ECMA-262 规范** 包含了大部分深入的、详细的、规范化的关于 JavaScript 的信息。这份规范明确地定义了这门语言。
> + 每年都会发布一个新版本的规范。最新的规范草案请见 [https://tc39.es/ecma262/](https://tc39.es/ecma262/)。
> 
> 手册： **MDN（Mozilla）JavaScript 索引** 是一个带有用例和其他信息的主要的手册。它是一个获取关于个别语言函数、方法等深入信息的很好的信息来源。
> 

## JavaScript 基础知识

学习 JavaScript 需要一个工作环境来运行我们的脚步，我们专注于使用 [`node.js`](https://nodejs.org/zh-cn) 作为执行环境。不过在此之前，我们需要先浏览器环境中介绍 `script` 标签

### script 标签

我们几乎可以使用 `<script>` 标签将 JavaScript 程序插入到 HTML 文档的任何位置。

```html
<!DOCTYPE html>
<html lang="zh">
<head>
    <meta charset="UTF-8">
    <title>script 标签</title>
</head>
<body>

<p>script 标签之前...</p>

<script type="text/javascript" language="JavaScript">
alert("Hello, World!")
</script>

<p>...script 标签之后</p>

</body>
</html>
```

> [!info] `<script>` 标签有一些属性需要了解
> 
> `type` 属性：在老的 HTML4 标准中，要求 script 标签有 `type` 特性。通常是 `type="text/javascript"`。现代 HTML 标准已经完全改变了此特性的含义。现在，它可以用于 JavaScript 模块。关于模块，在后续的模块化部分介绍
> 
> `language` 属性：`<script language=…>`。这个特性是为了显示脚本使用的语言。这个特性现在已经没有任何意义，因为 **语言默认就是 JavaScript**。不再需要使用它了。
> 
> `src` 属性：我们可以将 JavaScript 代码单独放入一个文件中，并通过 `src` 属性引入到 HTML 中
> + 可以是从网站根目录开始的绝对路径
> + 可以提供一个完整的 URL 地址
> + 要附加多个脚本，请使用多个标签：
>   

```html
<!DOCTYPE html>
<html lang="zh">
<head>
    <meta charset="UTF-8">
    <title>script 标签</title>
</head>
<body>

<p>script 标签之前...</p>

<script src="index.js"></script>

<p>...script 标签之后</p>

</body>
</html>
```


> [!info] 
> 一般来说，只有最简单的脚本才嵌入到 HTML 中。更复杂的脚本存放在单独的文件中。
> 
> 使用独立文件的好处是浏览器会下载它，然后将它保存到浏览器的 [缓存](https://en.wikipedia.org/wiki/Web_cache) 中。
> 
> 之后，其他页面想要相同的脚本就会从缓存中获取，而不是下载它。所以文件实际上只会下载一次。
> 
> 这可以节省流量，并使得页面（加载）更快。
> 

> [!attention] 如果设置了 `src` 特性，`script` 标签内容将会被忽略。
> 
> 一个单独的 `<script>` 标签不能同时有 `src` 特性和内部包裹的代码
> 
> 我们必须进行选择，要么使用外部的 `<script src="…">`，要么使用正常包裹代码的 `<script>`。
> 

### 代码结构

JavaScript 由 **语句** 组成。语句是执行行为的语法结构和命令。两个语句之前通过 `;` 进行分隔。如果存在换行符，可以省略 `;`

在同一行

```js
alert('Hello'); alert('World');
```

每条语句单独在一行

```js
alert('Hello');
alert('World');
```

每条语句单独在一行，可以省略结尾的 `;`

```js
alert('Hello')
alert('World')
```

随着时间推移，程序变得越来越复杂。为代码添加 **注释** 来描述它做了什么和为什么要这样做，变得非常有必要了。

+ **单行注释以两个正斜杠字符 `//` 开始**
+ **多行注释以一个正斜杠和星号开始 `/*` 并以一个星号和正斜杠结束 `*/`。**

### 现代模式："use strict"

长久以来，JavaScript 不断向前发展且并未带来任何兼容性问题。**新的特性被加入，旧的功能也没有改变。**

这么做有利于兼容旧代码，但缺点是 JavaScript 创造者的任何错误或不完善的决定也将永远被保留在 JavaScript 语言中。

这种情况一直持续到 **2009 年 ECMAScript 5 (ES5)** 的出现。ES5 规范增加了新的语言特性并且修改了一些已经存在的特性。为了保证旧的功能能够使用，大部分的修改是默认不生效的。你需要一个特殊的指令 —— **`"use strict"` 来明确地激活这些特性**。

这个指令看上去像一个字符串 `"use strict"` 或者 `'use strict'`。当它处于脚本文件的顶部时，则整个脚本文件都将 **以“现代”模式进行工作**

```js
"use strict";

// 代码以现代模式工作
...
```

学习到函数（一种组合命令的方式），所以让我们提前注意一下，**`"use strict"` 可以被放在函数体的开头**。这样则可以 **只在该函数中启用严格模式**。但通常人们会在整个脚本中启用严格模式。

```js
(function() {
  'use strict';

  // ...你的代码...
})()
```

> [!attention] 确保 `“use strict”` 出现在最顶部
> 
> 只有 `"use strict"` 之后的代码才会开启现代模式
> 

> [!attention] 没有办法取消 `use strict`
> 
> 一旦进入了严格模式，就没有回头路了。
> 

### 变量

大多数情况下，JavaScript 应用需要处理信息。这有两个例子：

1. 一个网上商店 —— 这里的信息可能包含正在售卖的商品和购物车。
2. 一个聊天应用 —— 这里的信息可能包括用户和消息等等。

变量就是用来储存这些信息的。[变量](https://en.wikipedia.org/wiki/Variable_(computer_science)) 是数据的 **“命名存储”**。我们可以使用变量来保存商品、访客和其他信息。

#### 声明变量

在 JavaScript 中创建一个变量，我们需要用到 `let` 关键字。

下面的语句创建（也可以称为 **声明** 或者 **定义**）了一个名称为 `"message"` 的变量：

```js
let message;
```

赋值运算符 `=` 可以给变量提供一些数据

```js
let message;
message = "Hello";
```

现在这个字符串已经保存到与该变量相关联的内存区域了，我们可以通过使用该变量名称访问它

```js
let message;
message = "hello";
console.log(message);
```

#### 初始化

使用 `let` 声明变量的时候，可以直接提供一些初始数据

```js
let message = "Hello";
console.log(message)
```

#### 理解变量

JavaScript 的变量在赋值时才会确定类型，并且可以使用任意类型的值赋值给变量。JavaScript 的彼岸就像标签一样，可以随意粘贴在任何数据上

### 常量

声明一个 **常数（不变）变量**，可以使用 `const` 而非 `let`：

```js
const Birthday = "1996-12-25" // 必须赋初值
```

> [!tip] 大写形式的常数
> 
> 一个普遍的做法是将常量用作别名，以便记住那些在执行之前就已知的难以记住的值。
> 
> 使用大写字母和下划线来命名这些常量。

```js
const COLOR_RED = "#F00";
const COLOR_GREEN = "#0F0";
const COLOR_BLUE = "#00F";
const COLOR_ORANGE = "#FF7F00";

// ……当我们需要选择一个颜色
let color = COLOR_ORANGE;
console.log(color); // #FF7F00
```

### 数据类型

JavaScript 中的 **值** 都具有特定的类型。例如，字符串或数字。

我们可以将任何类型的值存入变量。例如，一个变量可以在前一刻是个字符串，下一刻就存储一个数字：

```js
let message = "Hello";
message = 3.145
```

> [!tip]
> 
> 允许这种操作的编程语言，例如 JavaScript，被称为 **“动态类型”**（dynamically typed）的编程语言，意思是虽然编程语言中有不同的数据类型，但是你定义的变量并不会在定义后，被限制为某一数据类型
> 

#### Number类型

```js
let n = 123;
n = 12.345;
```

_Number_ 类型代表 **整数** 和 **浮点数**

除了常规的数字，还包括所谓的 “**特殊数值**（“special numeric values”）”也属于这种类型：`Infinity`、`-Infinity` 和 `NaN`。

> [!tip] 特殊数值
> 
> `Infinity` 代表数学概念中的 [无穷大](https://en.wikipedia.org/wiki/Infinity) ∞。是一个比任何数字都大的特殊值。
> 
> `NaN` 代表一个计算错误。它是一个不正确的或者一个未定义的数学操作所得到的结果
> 

> [!info] 数学运算是安全的
> 
> 在 JavaScript 中做数学运算是安全的。我们可以做任何事：除以 0，将非数字字符串视为数字，等等。
> 
> 脚本永远不会因为一个致命的错误（“死亡”）而停止。最坏的情况下，我们会得到 `NaN` 的结果。

#### BigInt 类型

在 JavaScript 中，“number” 类型无法安全地表示大于 $(2^{53}-1)$（即 $9007199254740991$），或小于 $-(2^{53}-1)$ 的整数

更准确的说，“number”  类型可以存储更大的整数（最多 $1.7976931348623157 * 10^{308}$），但超出安全整数范围 $\pm(2^{53}-1)$ 会出现精度问题，因为**并非所有数字都适合固定的 64 位存储**。因此，可能存储的是“近似值”。

`BigInt` 类型是最近被添加到 JavaScript 语言中的，用于表示任意长度的整数。可以通过将 `n` 附加到整数字段的末尾来创建 `BigInt` 值。

```js
// 尾部的 "n" 表示这是一个 BigInt 类型
const bigInt = 1234567890123456789012345678901234567890n;
```

#### String 类型

JavaScript 中的字符串必须被括在引号里。

```js
let str = "Hello";
let str2 = 'Single quotes are ok too';
let phrase = `can embed another ${str}`;
```

双引号和单引号都是“简单”引用，在 JavaScript 中两者几乎没有什么差别

反引号是 **功能扩展** 引号。它们允许我们通过将变量和表达式包装在 `${…}` 中，来将它们嵌入到字符串中。例如

```js
let name = "John";

// 嵌入一个变量
console.log( `Hello, ${name}!` ); // Hello, John!

// 嵌入一个表达式
console.log( `the result is ${1 + 2}` ); // the result is 3
```

`${…}` 内的表达式会被计算，计算结果会成为字符串的一部分。可以在 `${…}` 内放置任何东西：诸如名为 `name` 的变量，或者诸如 `1 + 2` 的算数表达式，或者其他一些更复杂的。

> [!attention] 需要注意的是，这仅仅在反引号内有效，其他引号不允许这种嵌入。

#### Boolean 类型（逻辑类型）

boolean 类型仅包含两个值：`true` 和 `false`。这种类型通常用于存储表示 yes 或 no 的值：`true` 意味着 “yes，正确”，`false` 意味着 “no，不正确”

```js
let nameFieldChecked = true; // yes, name field is checked
let ageFieldChecked = false; // no, age field is not checked
```

布尔值也可作为比较的结果

```js
let isGreater = 4 > 1;

console.log(isGreater ); // true（比较的结果是 "yes"）
```

#### null 值

特殊的 `null` 值不属于上述任何一种类型。

它构成了一个独立的类型，只包含 `null` 值：

```js
let age = null;
```

JavaScript 中的 `null` 不是一个“对不存在的 `object` 的引用”或者 “null 指针”。JavaScript 中的 `null` 仅仅是一个代表“无”、“空”或“值未知”的特殊值。

#### undefined 值

特殊值 `undefined` 和 `null` 一样自成类型。

`undefined` 的含义是 `未被赋值`。

如果一个变量已被声明，但未被赋值，那么它的值就是 `undefined`：

```js
let age;

console.log(age); // 输出 "undefined"
```

从技术上讲，可以显式地将 `undefined` 赋值给变量：

```js
let age = 100;

// 将值修改为 undefined
age = undefined;

console.log(age); // "undefined"
```

#### Object 类型和 Symbol 类型

`object` 类型是一个特殊的类型。**其他所有的数据类型都被称为 “原始类型”**，因为它们的 **值只包含一个单独的内容**（字符串、数字或者其他）。相反，`object` 则用于储存数据集合和更复杂的实体。

**`symbol` 类型用于创建对象的唯一标识符**。我们在这里提到 `symbol` 类型是为了完整性，但我们要在学完 `object` 类型后再学习它。

#### typeof 运算符

`typeof` 运算符返回参数的类型。当我们想要分别处理不同类型值的时候，或者想快速进行数据类型检验时，非常有用。

对 `typeof x` 的调用会以字符串的形式返回数据类型：

```js
typeof undefined // "undefined"

typeof 0 // "number"

typeof 10n // "bigint"

typeof true // "boolean"

typeof "foo" // "string"

typeof Symbol("id") // "symbol"

typeof Math // "object"  (1)

typeof null // "object"  (2)

typeof alert // "function"  (3)
```

> [!info] `typeof(x)` 语法
> 
> 可能还会遇到另一种语法：`typeof(x)`。它与 `typeof x` 相同。
> 
> 简单点说：`typeof` 是一个操作符，不是一个函数。这里的括号不是 `typeof` 的一部分。它是数学运算分组的括号。
> 
> 通常，这样的括号里包含的是一个数学表达式，例如 `(2 + 2)`，但这里它只包含一个参数 `(x)`。从语法上讲，它们允许在 `typeof` 运算符和其参数之间不打空格，有些人喜欢这样的风格。
> 
> 有些人更喜欢用 `typeof(x)`，尽管 `typeof x` 语法更为常见。
> 

### 类型转换

> [!tip] 讨论原始类型的类型转换
> 
> 对象类型我们尚未学习，这里只讨论原始类型的类型转换
> 

大多数情况下，运算符和函数会自动将赋予它们的值转换为正确的类型

比如，`alert` 会自动将任何值都转换为字符串以进行显示。算术运算符会将值转换为数字。

在某些情况下，我们需要将值显式地转换为我们期望的类型。

#### 字符串转换

当我们需要一个字符串形式的值时，就会进行字符串转换。

比如，`alert(value)` 将 `value` 转换为字符串类型，然后显示这个值。

我们也可以显式地调用 `String(value)` 来将 `value` 转换为字符串类型：

```js
let value = true;
alert(typeof value); // boolean

value = String(value); // 现在，值是一个字符串形式的 "true"
console.log(typeof value); // string
```

字符串转换最明显。`false` 变成 `"false"`，`null` 变成 `"null"` ，`2` 变成 `"2"` 等

#### 数字型转换

在算术函数和表达式中，会自动进行 number 类型转换。

比如，当把除法 `/` 用于非 number 类型：

```js
console.log( "6" / "2" ); // 3, string 类型的值被自动转换成 number 类型后进行计算
```

我们也可以使用 `Number(value)` 显式地将这个 `value` 转换为 number 类型。

```js
let str = "123";
console.log(typeof str); // string

let num = Number(str); // 变成 number 类型 123

console.log(typeof num); // number
```

当我们从 `string` 类型源（如文本表单）中读取一个值，但期望输入一个数字时，通常需要进行显式转换。如果该字符串不是一个有效的数字，转换的结果会是 `NaN`。例如：

```js
let age = Number("an arbitrary string instead of a number");

console.log(age); // NaN，转换失败
```

number 类型转换规则

| 值                | 变成……                                                                                                            |
| ---------------- | --------------------------------------------------------------------------------------------------------------- |
| `undefined`      | `NaN`                                                                                                           |
| `null`           | `0`                                                                                                             |
| `true` 和 `false` | `1` and `0`                                                                                                     |
| `string`         | 去掉首尾空白字符（空格、换行符 `\n`、制表符 `\t` 等）后的纯数字字符串中含有的数字。如果剩余字符串为空，则转换结果为 `0`。否则，将会从剩余字符串中“读取”数字。当类型转换出现 error 时返回 `NaN`。 |

#### 布尔型转换

布尔（boolean）类型转换是最简单的一个。

它发生在逻辑运算中（稍后我们将进行条件判断和其他类似的东西），但是也可以通过调用 `Boolean(value)` 显式地进行转换。

转换规则如下：

- 直观上为 **“空” 的值**（如 `0`、空字符串、`null`、`undefined` 和 `NaN`）将变为 `false`。
- 其他值变成 `true`。

### 运算符

#### 算术运算符

| 算术运算符 | 描述  |
| :---- | :-- |
| `+`   | 加法  |
| `-`   | 减法  |
| `*`   | 乘法  |
| `/`   | 除法  |
| `%`   | 取余  |
| `**`  | 求幂  |
| `++`  | 自增  |
| `--`  | 自减  |

> [!tip] 算术运算符，与 Go 类似
> + `**` 求幂，参考 Python
> + `++` 和 `--` 位于变量的前后均可，参考 C语言
> 

> [!tip] 数字转化，一元运算符 `+`：它的效果和 `Number(...)` 相同，但是更加简短。
> 
> 加号 `+` 有两种形式。一种是上面我们刚刚讨论的二元运算符，还有一种是一元运算符
> 
> 一元运算符加号，或者说，加号 `+` 应用于单个值，对数字没有任何作用。但是如果运算元不是数字，加号 `+` 则会将其转化为数字。
> 

#### 位运算符

| 位运算符  |  描述   |
| :---- | :---: |
| `&`   |  按位与  |
| \|    |  按位或  |
| `^`   | 按位异或  |
| `^`   | 按位取反  |
| `&^`  |  位清除  |
| `<<`  |  左移   |
| `>>`  |  右移   |
| `>>>` | 无符号右移 |

#### 比较运算符


| 比较运算符 | 描述              |
| :---: | :-------------- |
| `==`  | 等于，执行隐式类型转换     |
| `===` | 严格等于，不会执行隐式类型转换 |
| `!=`  | 不等于             |
|  `>`  | 大于              |
|  `<`  | 小于              |
| `>=`  | 大于或等于           |
| `<=`  | 小于或等于           |

> [!tip] `null` 和 `undefined` 的比较
> 
> 当使用数学式或其他比较方法 `< > <= >=` 时：`null/undefined` 会被转化为数字：`null` 被转化为 `0`，`undefined` 被转化为 `NaN`。
> 

> [!attention] `null` 和 `0` 的比较
> 
> **相等性检查 `==` 和普通比较符 `> < >= <=` 的代码逻辑是相互独立的**
> 
> 进行值的比较时，`null` 会被转化为数字，因此它被转化为了 `0`。这就是为什么中 `null >= 0` 返回值是 `true`， `null > 0` 返回值是 `false`
> 
> 另一方面，**`undefined` 和 `null` 在相等性检查 `==` 中不会进行任何的类型转换**，它们有自己独立的比较规则，所以除了它们之间互等外，不会等于任何其他的值。这就解释了为什么 `null == 0` 会返回 `false`
> 

```js
alert( null > 0 );  // (1) false
alert( null == 0 ); // (2) false
alert( null >= 0 ); // (3) true
```

> [!attention] `undefined` 不应该被与其他值进行比较
> 
> + `undefined > 0` 和 `undefined < 0` 都返回 `false` 是因为 `undefined` 在比较中被转换为了 `NaN`，而 `NaN` 是一个特殊的数值型值，它与任何值进行比较都会返回 `false`。
> + `undefined == 0` 返回 `false` 是因为这是一个相等性检查，而 `undefined` 只与 `null` 相等，不会与其他值相等

```js
alert( undefined > 0 ); // false (1)
alert( undefined < 0 ); // false (2)
alert( undefined == 0 ); // false (3)
```

#### 逻辑运算

| 逻辑运算符 | 描述               |
| :---- | :--------------- |
| \|\|  | 逻辑或。寻找第一个真值，短路求值 |
| &&    | 逻辑与。寻找第一个假值，短路求值 |
| !     | 逻辑非。布尔取反         |

> [!tip] `||` 和 `&&` 返回找到第一个真值/假值的原始形式，不会转换为 Boolean 类型

> [!tip] 短路求值：`||` 和 `&&` 一旦找到第一个真值/假值后就立即返回，不在测试之后的表达式

#### 空值合并运算符

**空值合并运算符(`??`)** 对待 `null` 和 `undefined` 的方式类似，所以我们将使用一个特殊的术语对其进行表示。为简洁起见，当一个值既不是 `null` 也不是 `undefined` 时，我们将其称为 “**已定义的（defined）**”。

> [!tip] `a ?? b` 的结果是：
> 
> 如果 `a` 是已定义的，则结果为 `a`
> 
> 如果 `a` 不是已定义的，则结果为 `b`
> 

### 条件分支

有时我们需要根据不同条件执行不同的操作。

我们可以使用 `if` 语句和条件运算符 `?`（也称为“问号”运算符）来实现。

#### if 语句

一个完整的 `if` 语句包含三部分

```js
if (条件1) {
	// 条件1为 true 执行的代码 
} else if (条件2) {
	//条件2为 true 执行的代码
} else {
	// 任何条件均为 false 执行的代码
}
```

#### 条件表达式

有时我们需要根据一个条件去赋值一个变量

```js
let accessAllowed;
let age = prompt('How old are you?', '');

if (age > 18) {
  accessAllowed = true;
} else {
  accessAllowed = false;
}

console.log(accessAllowed);
```

使用条件表达，使得代码更简洁

```js
let accessAllowed = (age > 18) ? true : false;
```

条件表达式的语法为

```js
let result = condition ? value1 : value2;
```

#### switch 语句

`switch` 语句可以替代多个 `if` 判断。

`switch` 语句为多分支选择的情况提供了一个更具描述性的方式。

`switch` 语句有至少一个 `case` 代码块和一个可选的 `default` 代码块。

```js
switch(x) {
  case 'value1':  // if (x === 'value1')
    ...
    [break]

  case 'value2':  // if (x === 'value2')
    ...
    [break]

  default:
    ...
    [break]
}
```

> [!tip]
> 
> 比较 `x` 值与第一个 `case`（也就是 `value1`）是否严格相等，然后比较第二个 `case`（`value2`）以此类推
> 
> 如果相等，`switch` 语句就执行相应 `case` 下的代码块，直到遇到最靠近的 `break` 语句（或者直到 `switch` 语句末尾）。
> 
> 如果没有符合的 case，则执行 `default` 代码块（如果 `default` 存在）。
> 

### 循环

我们经常需要重复执行一些操作。

例如，我们需要将列表中的商品逐个输出，或者运行相同的代码将数字 `1` 到 `10` 逐个输出。

> [!tip]　**循环** 是一种重复运行同一代码的方法。

#### while 循环

```js
while (condition) {
  // 代码
  // 所谓的“循环体”
}
```

> [!tip] 当 `condition` 为真时，执行循环体的 `code`。

#### do-while 循环

使用 `do..while` 语法可以将条件检查移至循环体 **下面**：

```js
do {
  // 循环体
} while (condition);
```

> [!tip] 确保循环体至少执行一次

#### for 循环

`for` 循环更加复杂，但它是最常使用的循环形式。

`for` 循环看起来就像这样：

 ```js
 for (begin; condition; step) {
  // ……循环体……
}
```

如下示例

```js
for (let i = 0; i < 3; i++) { // 结果为 0、1、2
  alert(i);
}
```


| 语句段         |             |                             |
| ----------- | ----------- | --------------------------- |
| `begin`     | `let i = 0` | 进入循环时执行一次。                  |
| `condition` | `i < 3`     | 在每次循环迭代之前检查，如果为 false，停止循环。 |
| `body（循环体）` | `alert(i)`  | 条件为真时，重复运行。                 |
| `step`      | `i++`       | 在每次循环体迭代后执行。                |

> [!tip] `for` 中 `begin` `condition` `step` 三部分可以任意省略
> + 省略 `condition` 默认一直为 `true`

#### break 语句

通常条件为假时，循环会终止。但我们随时都可以使用 `break` 指令强制退出。

```js
let sum = 0;

while (true) {

  let value = +prompt("Enter a number", '');

  if (!value) break; // (*)

  sum += value;

}
console.log( 'Sum: ' + sum );
```

#### continue 语句

`continue` 是 `break` 的“轻量版”。它不会停掉整个循环。而是 **停止当前这一次迭代**，**并强制启动新一轮循环**（如果条件允许的话）。

```js
for (let i = 0; i < 10; i++) {

  //如果为真，跳过循环体的剩余部分。
  if (i % 2 == 0) continue;

  console.log(i); // 1，然后 3，5，7，9
}
```

> [!tip] `break` 和 `continue` 和 Go 中的类似，可以在 `break` 和 `continue` 指定一个标签，表示要跳过的标签

### 函数

我们经常需要在脚本的许多地方执行很相似的操作。

例如，当访问者登录、注销或者在其他地方时，我们需要显示一条好看的信息。

函数是程序的主要“构建模块”。函数使该段代码可以被调用很多次，而不需要写重复的代码。

#### 函数声明

使用 **函数声明** 创建函数，需要使用关键字 `function`

```js
function showMessage() {
  console.log( 'Hello everyone!' );
}
```

`function` 关键字首先出现，然后是 **函数名**，然后是括号之间的 **参数** 列表（用逗号分隔，在上述示例中为空，我们将在接下来的示例中看到），最后是花括号之间的代码（即“函数体”）。

```js
function name(parameter1, parameter2, ... parameterN) {
  ...body...
}
```

我们的新函数可以通过名称调用：`showMessage()`。

```js
function showMessage() {
  console.log( 'Hello everyone!' );
}

showMessage();
showMessage();
```

#### 局部变量

在函数中声明的变量只在该函数内部可见。

```js
function showMessage() {
  let message = "Hello, I'm JavaScript!"; // 局部变量

  console.log( message );
}

showMessage(); // Hello, I'm JavaScript!

alert( message ); // <-- 错误！变量是函数的局部变量
```

#### 外部变量

函数也可以访问外部变量，例如：

```js
let userName = 'John';

function showMessage() {
  let message = 'Hello, ' + userName;
  console.log(message);
}

showMessage(); // Hello, John
```

> [!tip] 
> 
> **函数对外部变量拥有全部的访问权限**。函数也可以修改外部变量。
> 

```js
let userName = 'John';

function showMessage() {
  userName = "Bob"; // (1) 改变外部变量

  let message = 'Hello, ' + userName;
  console.log(message);
}

console.log( userName ); // John 在函数调用之前

showMessage();

console.log( userName ); // Bob，值被函数修改了
```

如果在函数内部声明了同名变量，那么函数会 **遮蔽** 外部变量。例如，在下面的代码中，函数使用局部的 `userName`，而外部变量被忽略：

```js
let userName = 'John';

function showMessage() {
  let userName = "Bob"; // 声明一个局部变量

  let message = 'Hello, ' + userName; // Bob
  console.log(message);
}

// 函数会创建并使用它自己的 userName
showMessage();

console.log( userName ); // John，未被更改，函数没有访问外部变量。
```

> [!tip] 外部变量也称为全局变量

#### 参数

我们可以通过 **参数** 将任意数据传递给函数

在如下示例中，函数有两个参数：`from` 和 `text`

```js
function showMessage(from, text) { // 参数：from 和 text
  console.log(from + ': ' + text);
}

showMessage('Ann', 'Hello!'); // Ann: Hello! (*)
showMessage('Ann', "What's up?"); // Ann: What's up? (**)
```

当函数在 `(*)` 和 `(**)` 行中被调用时，给定值被复制到了局部变量 `from` 和 `text`。然后函数使用它们进行计算。

这里还有一个例子：我们有一个变量 `from`，并将它传递给函数。请注意：**函数会修改 `from`，但在函数外部看不到更改，因为函数修改的是复制的变量值副本**：

```js
function showMessage(from, text) {

  from = '*' + from + '*'; // 让 "from" 看起来更优雅

  console.log( from + ': ' + text );
}

let from = "Ann";

showMessage(from, "Hello"); // *Ann*: Hello

// "from" 值相同，函数修改了一个局部的副本。
console.log( from ); // Ann
```

当一个值被作为函数参数（parameter）传递时，它也被称为 **参数（argument）**。

换一种方式，我们把这些术语搞清楚：

- **参数（parameter）** 是 _函数声明_ 中括号内列出的变量（它是函数声明时的术语），也称为 **形参**。
- **参数（argument）** 是 _调用函数_ 时传递给函数的值（它是函数调用时的术语），称为 **实参**。

##### 默认值

如果一个函数被调用，但有 **未提供实参**，那么相应的形参值就会变成 `undefined`

例如，之前提到的函数 `showMessage(from, text)` 可以只使用一个实参调用：

```js
showMessage("Ann");
```

那不是错误，这样调用将输出 `"*Ann*: undefined"`。因为参数 `text` 的值未被传递，所以变成了 `undefined`。

们可以使用 `=` 为函数声明中的参数指定所谓的“默认”（如果对应参数的值未被传递则使用）值

```js
function showMessage(from, text = "no text given") {
  console.log( from + ": " + text );
}

showMessage("Ann"); // Ann: no text given
```

现在如果 `text` 参数未被传递，它将会得到值 `"no text given"`。

这里 `"no text given"` 是一个字符串，但它可以是更复杂的表达式，并且只会在缺少参数时才会被计算和分配。所以，这也是可能的

```js
function showMessage(from, text = anotherFunction()) {
  // anotherFunction() 仅在没有给定 text 时执行
  // 其运行结果将成为 text 的值
}
```

此外，可以使用 **后备的默认参数**。将参数默认值的设置放在函数执行（相较更后期）而不是函数声明

```js
function showMessage(text) {
  // ...

  if (text === undefined) { // 如果参数未被传递进来
    text = 'empty message';
  }

  console.log(text);
}

showMessage(); // empty message
```

此外，可以使用 `||` 运算符

```js
function showMessage(text) {
  // 如果 text 为 undefined 或者为假值，那么将其赋值为 'empty'
  text = text || 'empty';
  ...
}
```

现代 JavaScript 引擎支持 **空值合并运算符 `??`**，它在大多数假值（例如 `0`）应该被视为“正常值”时更具优势：

```js
function showCount(count) {
  // 如果 count 为 undefined 或 null，则提示 "unknown"
  console.log(count ?? "unknown");
}

showCount(0); // 0
showCount(null); // unknown
showCount(); // unknown
```

#### 返回值

函数可以将一个值返回到调用代码中作为结果。最简单的例子是将两个值相加的函数：

```js
function sum(a, b) {
  return a + b;
}

let result = sum(1, 2);
console.log( result ); // 3
```

指令 `return` 可以在函数的任意位置。当执行到达时，函数停止，并将值返回给调用代码

> [!tip] 空值的 `return` 或没有 `return` 的函数返回值为 `undefined`
```js
function doNothing() { /* 没有代码 */ }

console.log( doNothing() === undefined ); // true
```

### 函数表达式与箭头函数

#### 函数表达式

另一种创建函数的语法称为 **函数表达式**。它允许我们在任何表达式的中间创建一个新函数

```js
let sayHi = function() {
  console.log( "Hello" );
};
```

在这里我们可以看到变量 `sayHi` 得到了一个值，新函数 `function() { console.log("Hello"); }`。

> [!tip] 函数是一个值：重申一次：**无论函数是如何创建的，函数都是一个值**。上面的两个示例都在 `sayHi` 变量中存储了一个函数

#### 回调函数

让我们多举几个例子，看看如何将 **函数作为值来传递** 以及如何使用函数表达式。

我们写一个包含三个参数的函数 `ask(question, yes, no)`：

```js
function ask(question, yes, no) {
  if (question != "") 
	  yes();
  else 
	  no();
}

function showOk() {
  console.log( "You agreed." );
}

function showCancel() {
  console.log( "You canceled the execution." );
}

// 用法：函数 showOk 和 showCancel 被作为参数传入到 ask
ask("Do you agree?", showOk, showCancel);
```

我们可以使用函数表达式来编写一个等价的、更简洁的函数

```js
function ask(question, yes, no) {
  if (question != "") 
	  yes();
  else 
	  no();
}

ask(
  "Do you agree?",
  function() { console.log("You agreed."); },
  function() { console.log("You canceled the execution."); }
);
```

#### 箭头函数

创建函数还有另外一种非常简单的语法，并且这种方法通常比函数表达式更好。它被称为 **“箭头函数”**，因为它看起来像这样：

```js
let func = (arg1, arg2, ..., argN) => expression;
```

这里创建了一个函数 `func`，它接受参数 `arg1..argN`，然后使用参数对右侧的 `expression` 求值并返回其结果。

换句话说，它是下面这段代码的更短的版本：

```js
let func = function(arg1, arg2, ..., argN) {
  return expression;
};
```

一个具体的例子

```js
let sum = (a, b) => a + b;

/* 这个箭头函数是下面这个函数的更短的版本：

let sum = function(a, b) {
  return a + b;
};
*/

console.log( sum(1, 2) ); // 3
```

可以看到 `(a, b) => a + b` 表示一个函数接受两个名为 `a` 和 `b` 的参数。在执行时，它将对表达式 `a + b` 求值，并返回计算结果。

> [!tip]
> + 如果我们只有一个参数，还可以省略掉参数外的圆括号，使代码更短
> + 如果没有参数，括号则是空的（但括号必须保留）：
> 

有时我们需要更复杂一点的函数，比如带 **有多行的表达式或语句**。在这种情况下，我们可以 **使用花括号将它们括起来**。主要区别在于，用花括号括起来之后，**需要包含 `return` 才能返回值（就像常规函数一样）**

```js
let sum = (a, b) => {  // 花括号表示开始一个多行函数
  let result = a + b;
  return result; // 如果我们使用了花括号，那么我们需要一个显式的 “return”
};

console.log( sum(1, 2) ); // 3
```

### 对象类型

对象则用来存储键值对和更复杂的实体。在 JavaScript 中，对象几乎渗透到了这门编程语言的方方面面

我们可以通过使用带有可选 **属性列表** 的花括号 `{....}` 来创建对象。一个属性就是一个键值对（`"key: value"`），其中 **键**（`key`）是一个字符串（也叫做 **属性名**），值（`value`）可以是任何值。

```js
let user = {     // 一个对象
  name: "John",  // 键 "name"，值 "John"
  age: 30        // 键 "age"，值 30
};
```

属性有键（或者也可以叫做“名字”或“标识符”），位于冒号 `":"` 的前面，**值在冒号的右边**。

访问和修改属性，通过成员运算符 `.` 进行

```js
// 读取文件的属性：
console.log( user.name ); // John
console.log( user.age ); // 30

// 新增属性
user.isAdmin = true;
```

关键字 `delete` 可以删除对象的属性

```js
delete user.name
```

> [!tip] 属性名本质上是字符串
> 

我们可以用 **多字词语** 来作为属性名，但 **必须给它们加上引号**

```js
let user = {
  name: "John",
  age: 30,
  "likes birds": true  // 多词属性名必须加引号
};
```

对于多词属性，点操作就不能用了：**点符号要求 `key` 是有效的变量标识符**。这意味着：不包含空格，不以数字开头，也不包含特殊字符（允许使用 `$` 和 `_`）。

有另一种方法，就是使用方括号，可用于任何字符串：

```js
let user = {};

// 设置
user["likes birds"] = true;

// 读取
console.log(user["likes birds"]); // true

// 删除
delete user["likes birds"];
```

#### 计算属性

当创建一个对象时，我们可以 **在对象字面量中使用方括号**。这叫做 **计算属性**。

```js
let fruit = prompt("Which fruit to buy?", "apple");

let bag = {
  [fruit]: 5, // 属性名是从 fruit 变量中得到的
};

console.log( bag.apple ); // 5 如果 fruit="apple"
```

计算属性的含义很简单：**`[fruit]` 含义是属性名应该从 `fruit` 变量中获取**。所以，如果一个用户输入 `"apple"`，`bag` 将变为 `{apple: 5}`。

本质上，这跟下面的语法效果相同：

```js
let fruit = prompt("Which fruit to buy?", "apple");
let bag = {};

// 从 fruit 变量中获取值
bag[fruit] = 5;
```

#### 属性值简写

在实际开发中，我们通常用已存在的变量当做属性名。

```js
function makeUser(name, age) {
  return {
    name: name,
    age: age,
    // ……其他的属性
  };
}

let user = makeUser("John", 30);
console.log(user.name); // John
```

在上面的例子中，属性名跟变量名一样。这种通过变量生成属性的应用场景很常见，在这有一种特殊的 **属性值缩写** 方法，使属性名变得更短。可以用 `name` 来代替 `name:name` 像下面那样：

```js
function makeUser(name, age) {
  return {
    name, // 与 name: name 相同
    age,  // 与 age: age 相同
    // ...
  };
}
```

#### 属性存在性测试，“in” 操作符


相比于其他语言，JavaScript 的对象有一个需要注意的特性：**能够被访问任何属性**。**即使属性不存在也不会报错**

**读取不存在的属性只会得到 `undefined`**。所以我们可以很容易地判断一个属性是否存在：

```js
let user = {};

console.log( user.noSuchProperty === undefined ); // true 意思是没有这个属性
```

这里还有一个特别的，检查属性是否存在的操作符 `"in"`

```js
"key" in object
```

> [!attention] `in` 的左边必须是 **属性名**。通常是一个带引号的字符串

#### for ... in 循环

为了 **遍历一个对象的所有键（key**），可以使用一个特殊形式的循环：`for..in`。这跟我们在前面学到的 `for(;;)` 循环是完全不一样的东西。

```js
for (key in object) {
  // 对此对象属性中的每个键执行的代码
}
```

对象有顺序吗？换句话说，如果我们遍历一个对象，我们获取属性的顺序是和属性添加时的顺序相同吗？这靠谱吗？

简短的回答是：“有特别的顺序”：**整数属性会被进行排序**，**其他属性则按照创建的顺序显示**。详情如下：

例如，让我们考虑一个带有电话号码的对象：

```js
let codes = {
  "49": "Germany",
  "41": "Switzerland",
  "44": "Great Britain",
  // ..,
  "1": "USA"
};

for(let code in codes) {
  console.log(code); // 1, 41, 44, 49
}
```

#### 对象引用和复制

对象与原始类型的根本区别之一是，**对象是 “*通过引用*” 存储和复制的**，而 **原始类型总是 *“作为一个整体”* 复制**。

**当一个对象变量被复制 —— 引用被复制，而该对象自身并没有被复制。**

```js
let user = { name: "John" };

let admin = user; // 复制引用
```

#### Object.assign

那么，**拷贝一个对象变量会又创建一个对相同对象的引用**。但是，如果我们想要复制一个对象，那该怎么做呢？

我们可以创建一个新对象，通过遍历已有对象的属性，并在原始类型值的层面复制它们，以实现对已有对象结构的复制。

```js
let user = {
  name: "John",
  age: 30
};

let clone = {}; // 新的空对象

// 将 user 中所有的属性拷贝到其中
for (let key in user) {
  clone[key] = user[key];
}

// 现在 clone 是带有相同内容的完全独立的对象
clone.name = "Pete"; // 改变了其中的数据

console.log( user.name ); // 原来的对象中的 name 属性依然是 John
```

我们也可以使用 [Object.assign](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object/assign) 方法来达成同样的效果

```js
Object.assign(dest, [src1, src2, src3...])
```

+  第一个参数 `dest` 是指目标对象
+ 更后面的参数 `src1, ..., srcN`（可按需传递多个参数）是源对象

该方法将所有源对象的属性拷贝到目标对象 `dest` 中。换句话说，从第二个开始的所有参数的属性都被拷贝到第一个参数的对象中。

```js
let user = { name: "John" };

let permissions1 = { canView: true };
let permissions2 = { canEdit: true };

// 将 permissions1 和 permissions2 中的所有属性都拷贝到 user 中
Object.assign(user, permissions1, permissions2);

// 现在 user = { name: "John", canView: true, canEdit: true }
```

如果被拷贝的属性的属性名已经存在，那么它会被覆盖：

```js
let user = { name: "John" };

Object.assign(user, { name: "Pete" });

console.log(user.name); // 现在 user = { name: "Pete" }
```

#### 深复制

到现在为止，我们都假设 `user` 的所有属性均为原始类型。但**属性可以是对其他对象的引用**。

```js
let user = {
  name: "John",
  sizes: {
    height: 182,
    width: 50
  }
};

console.log( user.sizes.height ); // 182
```

现在这样拷贝 `clone.sizes = user.sizes` 已经不足够了，因为 `user.sizes` 是个对象，它会以引用形式被拷贝。因此 `clone` 和 `user` 会共用一个 `sizes`：

```js
let user = {
  name: "John",
  sizes: {
    height: 182,
    width: 50
  }
};

let clone = Object.assign({}, user);

console.log( user.sizes === clone.sizes ); // true，同一个对象

// user 和 clone 分享同一个 sizes
user.sizes.width++;       // 通过其中一个改变属性值
console.log(clone.sizes.width); // 51，能从另外一个获取到变更后的结果
```

为了解决这个问题，并让 `user` 和 `clone` 成为两个真正独立的对象，我们应该使用一个拷贝循环来检查 `user[key]` 的每个值，如果它是一个对象，那么也复制它的结构。这就是所谓的 **“深拷贝”**。

#### 方法 

通常创建对象来表示真实世界中的实体，如用户和订单等：

```js
let user = {
  name: "John",
  age: 30
};
```

并且，在现实世界中，用户可以进行 **操作**：从购物车中挑选某物、登录和注销等。在 JavaScript 中，**行为（action）** 由 **属性中的函数** 来表示。

```js
let user = {
  name: "John",
  age: 30
};

user.sayHi = function() {
  console.log("Hello!");
};

user.sayHi(); // Hello!
```

> [!tip] 作为对象属性的函数被称为 **方法**。

在对象字面量中，有一种更短的（声明）方法的语法：

```js
// 这些对象作用一样

user = {
  sayHi: function() {
    console.log("Hello");
  }
};

// 方法简写看起来更好，对吧？
let user = {
  sayHi() { // 与 "sayHi: function(){...}" 一样
    console.log("Hello");
  }
};
```

#### this

通常，对象方法需要访问对象中存储的信息才能完成其工作

例如，`user.sayHi()` 中的代码可能需要用到 `user` 的 name 属性。

> [!tip] **为了访问该对象，方法中可以使用 `this` 关键字**
> 
> `this` 是调用该方法的对象
> 

```js
let user = {
  name: "John",
  age: 30,

  sayHi() {
    // "this" 指的是“当前的对象”
    console.log(this.name);
  }

};

user.sayHi(); // John
```

在这里 `user.sayHi()` 执行过程中，`this` 的值是 `user`。技术上讲，也可以在不使用 `this` 的情况下，通过外部变量名来引用它：

```js
let user = {
  name: "John",
  age: 30,

  sayHi() {
    console.log(user.name); // "user" 替代 "this"
  }

};
```

通过外部变量引用对象是不可靠的。如果我们决定将 `user` 复制给另一个变量，例如 `admin = user`，并赋另外的值给 `user`，那么它将访问到错误的对象。

```js
let user = {
  name: "John",
  age: 30,

  sayHi() {
    console.log( user.name ); // 导致错误
  }

};


let admin = user;
user = null; // 重写让其更明显

admin.sayHi(); // TypeError: Cannot read property 'name' of null
```

> [!tip] JavaScript 中的 `this` 可以用于任何函数，即使它不是对象的方法。
> 
> `this` 的值是在代码运行时计算出来的，它取决于代码上下文。
> 

> [!attention] 箭头函数没有自己的 `this`
> 
> 箭头函数有些特别：它们没有自己的 `this`。如果我们在这样的函数中引用 `this`，`this` 值取决于外部“正常的”函数。
> 

#### 可选链 `?.`


可选链 `?.` 语法有三种形式：

1. `obj?.prop` —— 如果 `obj` 存在则返回 `obj.prop`，否则返回 `undefined`。
2. `obj?.[prop]` —— 如果 `obj` 存在则返回 `obj[prop]`，否则返回 `undefined`。
3. `obj.method?.()` —— 如果 `obj.method` 存在则调用 `obj.method()`，否则返回 `undefined`。

### symbol 类型

根据规范，只有两种原始类型可以用作对象属性键：字符串类型和 `symbol` 类型

否则，**如果使用另一种类型**，例如数字，它 **会被自动转换为字符串**。所以 `obj[1]` 与 `obj["1"]` 相同，而 `obj[true]` 与 `obj["true"]` 相同。

`“symbol”` 值表示唯一的标识符。可以使用 `Symbol()` 来创建这种类型的值：

```js
let id = Symbol();
```

创建时，我们可以给 `symbol` 一个描述（也称为 `symbol` 名），这在代码调试时非常有用：

```js
// id 是描述为 "id" 的 symbol
let id = Symbol("id");
```

**symbol 保证是唯一的**。即使我们创建了许多具有相同描述的 `symbol`，它们的值也是不同。**描述只是一个标签，不影响任何东西**。例如，这里有两个描述相同的 symbol —— 它们不相等：

```js
let id1 = Symbol("id");
let id2 = Symbol("id");

console.log(id1 == id2); // false
```

> [!tip] 
> 
> 所以，总而言之，**symbol 是带有可选描述的“原始唯一值”**。
> 

### 对象 => 原始值转换

现在我们已经掌握了方法（method）和 symbol 的相关知识，可以开始学习对象原始值转换了。

1. **没有转换为布尔值**。所有的对象在布尔上下文（context）中均为 `true`，就这么简单。只有字符串和数字转换。
2. **数字转换发生在对象相减或应用数学函数时**。例如，`Date` 对象（将在 [日期和时间](https://zh.javascript.info/date) 一章中介绍）可以相减，`date1 - date2` 的结果是两个日期之间的差值。
3. 至于字符串转换 —— 通常发生在我们像 `alert(obj)` 这样输出一个对象和类似的上下文中。

我们 **可以使用特殊的对象方法，自己实现字符串和数字的转换**。

JavaScript 是如何决定应用哪种转换的？类型转换在各种情况下有三种变体。它们被称为 “hint”，在 [规范](https://tc39.github.io/ecma262/#sec-toprimitive) 所述：

`"string"` hint **对象到字符串的转换**，当我们对期望一个字符串的对象执行操作时

```js
// 输出
alert(obj);

// 将对象作为属性键
anotherObj[obj] = 123;
```

`"number"` hint **对象到数字的转换**，例如当我们进行数学运算时：

```js
// 显式转换
let num = Number(obj);

// 数学运算（除了二元加法）
let n = +obj; // 一元加法
let delta = date1 - date2;

// 小于/大于的比较
let greater = user1 > user2;
```

`"default"` hint 在少数情况下发生，**当运算符 “不确定” 期望值的类型时**。例如，二元加法 `+` 可用于字符串（连接），也可以用于数字（相加）。因此，当二元加法得到对象类型的参数时，它将依据 `"default"` hint 来对其进行转换。

此外，如果对象被用于与字符串、数字或 symbol 进行 `==` 比较，这时到底应该进行哪种转换也不是很明确，因此使用 `"default"` hint。

```js
// 二元加法使用默认 hint
let total = obj1 + obj2;

// obj == number 使用默认 hint
if (user == 1) { ... };
```

**为了进行转换，JavaScript 尝试查找并调用三个对象方法：**

1. 调用 `obj[Symbol.toPrimitive](hint)` —— 带有 symbol 键 `Symbol.toPrimitive`（系统 symbol）的方法，如果这个方法存在的话，
2. 否则，如果 hint 是 `"string"` —— 尝试调用 `obj.toString()` 或 `obj.valueOf()`，无论哪个存在。
3. 否则，如果 hint 是 `"number"` 或 `"default"` —— 尝试调用 `obj.valueOf()` 或 `obj.toString()`，无论哪个存在。
