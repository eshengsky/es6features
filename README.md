# ECMAScript 6

翻译自 [https://github.com/lukehoban/es6features](https://github.com/lukehoban/es6features)

## 简介
ECMAScript 6，也被称为 ECMAScript 2015，是 ECMAScript 标准的最新版本。ES6 是 JavaScript 语言的重大革新，同时也是 ES5 在 2009 年被标准化以来的首次更新。主要的 JavaScript 引擎对这些特性的实现正在[进行中](http://kangax.github.io/es5-compat-table/es6/)。

点击 [ES6 标准](http://www.ecma-international.org/ecma-262/6.0/) 以查看 ECMAScript 6 的完整说明。

ES6 包含了如下新特性：
- [arrows](#arrows)
- [classes](#classes)
- [enhanced object literals](#enhanced-object-literals)
- [template strings](#template-strings)
- [destructuring](#destructuring)
- [default + rest + spread](#default--rest--spread)
- [let + const](#let--const)
- [iterators + for..of](#iterators--forof)
- [generators](#generators)
- [unicode](#unicode)
- [modules](#modules)
- [module loaders](#module-loaders)
- [map + set + weakmap + weakset](#map--set--weakmap--weakset)
- [proxies](#proxies)
- [symbols](#symbols)
- [subclassable built-ins](#subclassable-built-ins)
- [promises](#promises)
- [math + number + string + array + object APIs](#math--number--string--array--object-apis)
- [binary and octal literals](#binary-and-octal-literals)
- [reflect api](#reflect-api)
- [tail calls](#tail-calls)

## ECMAScript 6 特性

### 箭头函数
箭头函数是使用 `=>` 语法简写的函数，这在语法上和 C#，Java 8 以及 CoffeeScript 的相关特性类似。箭头函数既支持语句块，也支持表达式。与普通函数不同的是，箭头函数中的 `this` 始终指向函数定义时所在的对象，而非函数执行时所在的对象。

```JavaScript
// 表达式
const odds = evens.map(v => v + 1);
const nums = evens.map((v, i) => v + i);
const pairs = evens.map(v => ({even: v, odd: v + 1}));

// 语句块
nums.forEach(v => {
    if (v % 5 === 0) {
        fives.push(v);
    }
});

// 箭头函数中的this
var bob = {
    _name: 'Bob',
    _friends: ['Tom', 'Kathy'],
    printFriends() {
        this._friends.forEach(f => {
            // 此处的this指向bob，而非window
            console.log(`${this._name} knows ${f}`);
        });
    }
}
```

> 更多信息：[MDN 箭头函数](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Functions/Arrow_functions)

### 类
ES6 的类是现有的基于原型的面向对象模式的一个简单的语法糖。简单且方便的声明形式使得类模式更易于使用，并鼓励互操作性。类支持基于原型的继承、父类调用、实例方法、静态方法、构造函数等。

```JavaScript
// 定义一个继承自THREE.Mesh的类
class SkinnedMesh extends THREE.Mesh {
    // 类的构造函数
    constructor(geometry, materials) {
        // 调用父类构造函数
        super(geometry, materials);

        this.idMatrix = SkinnedMesh.defaultMatrix();
        this.bones = [];
        this.boneMatrices = [];
        // ...
    }

    // 普通（实例）方法，通过 new SkinnedMesh().update 调用
    update(camera) {
        // ...
        super.update();
    }

    // get访问器
    get boneCount() {
        return this.bones.length;
    }

    // set访问器
    set matrixType(matrixType) {
        this.idMatrix = SkinnedMesh[matrixType]();
    }

    // 静态方法，直接通过 SkinnedMesh.defaultMatrix 调用
    static defaultMatrix() {
        return new THREE.Matrix4();
    }
}
```

> 更多信息：[MDN 类](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Classes)

### 增强的对象字面量
在 ES2015，对象字面量被扩展到支持在创建对象时设置原型，简写 `foo: foo` 赋值，简写方法的定义，调用父函数，动态计算的属性名。这些增强使得对象字面量和类声明紧密联系起来，让基于对象的设计也在这种便利中收益。

```JavaScript
var obj = {
    // 设置原型
    __proto__: theProtoObj,

    // 'handler: handler'的简写
    handler,

    // 简写的方法定义，省略了function
    toString() {
        // 调用父函数
        return "d " + super.toString();
    },

    // 动态计算的属性名
    [ 'prop_' + (() => 42)() ]: 42
};
```

> 更多信息：[MDN 语法和数据类型：对象字面量](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Guide/Grammar_and_types#对象字面量_(Object_literals))

### 模板字符串
模板字符串提供了构建字符串的语法糖。这类似于 Perl，Python 等语言中的字符串插值特性。作为可选项，你还可以加入标签来自定义字符串的构建，这可以避免注入攻击，或者从字符串内容构建高阶数据结构。

```JavaScript
// 基本的模板字符串
`In JavaScript '\n' is a line-feed.`

// 多行字符串
`In JavaScript this is
 not legal.`

// 字符串插值
var name = "Bob", time = "today";
`Hello ${name}, how are you ${time}?`

// 构建一个HTTP请求，其中标签POST是一个自定义的函数，用来进行替换和构建
POST`http://foo.org/bar?a=${a}&b=${b}
     Content-Type: application/json
     X-Credentials: ${credentials}
     { "foo": ${foo},
       "bar": ${bar}}`(myOnReadyStateChangeHandler);
```

> 更多信息：[MDN 模板字符串](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/template_strings)

### 解构
解构使用模式匹配进行绑定，支持匹配数组和对象。解构具有良好的容错性，类似于标准对象的查询 `foo['bar']`，如果找不到则返回 `undefined` 。

```JavaScript
// 数组匹配
var [a, , b] = [1, 2, 3];

// 对象匹配
var {op: a, lhs: {op: b}, rhs: c} = getASTNode();

// 对象匹配的简写
// 绑定 `op`, `lhs` and `rhs` 到作用域
var {op, lhs, rhs} = getASTNode();

// 可以被用在参数位置
function g({name: x}) {
    console.log(x);
}
g({name: 5});

// 解构的容错性
var [a] = [];
a === undefined;

// 带默认值时的容错性
var [a = 1] = [];
a === 1;
```

> 更多信息：[MDN 解构赋值](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Operators/Destructuring_assignment)

### 默认参数、剩余参数和扩展参数
函数可以设置参数的默认值。调用函数时可以传入一个扩展参数，这将被视为连续的参数形式。函数尾部的剩余参数会被绑定到一个数组，它取代了 `arguments` 的功能，可以更直接地处理常见情况。

```JavaScript
function f(x, y = 12) {
    // y 将被赋值为 12 如果没有传值（或者传值为 undefined）
    return x + y;
}
f(3) == 15
```
```JavaScript
function f(x, ...y) {
    // y 是一个数组
    return x * y.length;
}
f(3, "hello", true) == 6
```
```JavaScript
function f(x, y, z) {
    return x + y + z;
}
// 将数组的每一个元素作为连续参数传入方法f
f(...[1,2,3]) == 6
```

> 更多信息：[MDN 默认参数](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Functions/Default_parameters)，[MDN 剩余参数](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Functions/rest_parameters)，[MDN 扩展参数](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Operators/Spread_operator)

### Let 和 Const
两者都用来声明块级作用域的变量。`let` 是新的 `var`，`const` 是单次赋值，且必须在赋值之后才能使用。

```JavaScript
function f() {
    {
        let x;
        {
            // 块级作用域
            const x = "sneaky";
            // 错误，不允许再次赋值
            x = "foo";
        }
        // 错误，在当前作用域下已有定义
        let x = "inner";
    }
}
```

更多信息：[MDN let 语句](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Statements/let)，[MDN const 语句](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Statements/const)

### 迭代器
迭代器对象让 JavaScript 能够像 CLR 的 IEnumerable 接口或 Java 的 Iterable 接口一样进行自定义迭代。通常我们将 `for..in` 转换成自定义的基于迭代器的 `for..of` 迭代。不需要实现一个像 LINQ 那样的惰性设计模式的数组。

```JavaScript
let fibonacci = {
    [Symbol.iterator]() {
        let pre = 0, cur = 1;
        return {
            next() {
                [pre, cur] = [cur, pre + cur];
                return { done: false, value: cur }
            }
        }
    }
}

for (var n of fibonacci) {
    // 在1000处停止
    if (n > 1000)
        break;
    console.log(n);
}
```

迭代器是基于这些鸭子类型的接口（这里使用 [TypeScript](http://typescriptlang.org) 类型的语法只是用以阐述问题）：
```TypeScript
interface IteratorResult {
    done: boolean;
    value: any;
}
interface Iterator {
    next(): IteratorResult;
}
interface Iterable {
    [Symbol.iterator](): Iterator
}
```

更多信息：[MDN for...of](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Statements/for...of)

### Generators
Generators simplify iterator-authoring using `function*` and `yield`.  A function declared as function* returns a Generator instance.  Generators are subtypes of iterators which include additional  `next` and `throw`.  These enable values to flow back into the generator, so `yield` is an expression form which returns a value (or throws).

Note: Can also be used to enable ‘await’-like async programming, see also ES7 `await` proposal.

```JavaScript
var fibonacci = {
  [Symbol.iterator]: function*() {
    var pre = 0, cur = 1;
    for (;;) {
      var temp = pre;
      pre = cur;
      cur += temp;
      yield cur;
    }
  }
}

for (var n of fibonacci) {
  // truncate the sequence at 1000
  if (n > 1000)
    break;
  console.log(n);
}
```

The generator interface is (using [TypeScript](http://typescriptlang.org) type syntax for exposition only):

```TypeScript
interface Generator extends Iterator {
    next(value?: any): IteratorResult;
    throw(exception: any);
}
```

More info: [MDN Iteration protocols](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Iteration_protocols)

### Unicode
Non-breaking additions to support full Unicode, including new Unicode literal form in strings and new RegExp `u` mode to handle code points, as well as new APIs to process strings at the 21bit code points level.  These additions support building global apps in JavaScript.

```JavaScript
// same as ES5.1
"𠮷".length == 2

// new RegExp behaviour, opt-in ‘u’
"𠮷".match(/./u)[0].length == 2

// new form
"\u{20BB7}"=="𠮷"=="\uD842\uDFB7"

// new String ops
"𠮷".codePointAt(0) == 0x20BB7

// for-of iterates code points
for(var c of "𠮷") {
  console.log(c);
}
```

More info: [MDN RegExp.prototype.unicode](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/RegExp/unicode)

### Modules
Language-level support for modules for component definition.  Codifies patterns from popular JavaScript module loaders (AMD, CommonJS). Runtime behaviour defined by a host-defined default loader.  Implicitly async model – no code executes until requested modules are available and processed.

```JavaScript
// lib/math.js
export function sum(x, y) {
  return x + y;
}
export var pi = 3.141593;
```
```JavaScript
// app.js
import * as math from "lib/math";
alert("2π = " + math.sum(math.pi, math.pi));
```
```JavaScript
// otherApp.js
import {sum, pi} from "lib/math";
alert("2π = " + sum(pi, pi));
```

Some additional features include `export default` and `export *`:

```JavaScript
// lib/mathplusplus.js
export * from "lib/math";
export var e = 2.71828182846;
export default function(x) {
    return Math.log(x);
}
```
```JavaScript
// app.js
import ln, {pi, e} from "lib/mathplusplus";
alert("2π = " + ln(e)*pi*2);
```

More MDN info: [import statement](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/import), [export statement](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/export)

### Module Loaders
Module loaders support:
- Dynamic loading
- State isolation
- Global namespace isolation
- Compilation hooks
- Nested virtualization

The default module loader can be configured, and new loaders can be constructed to evaluate and load code in isolated or constrained contexts.

```JavaScript
// Dynamic loading – ‘System’ is default loader
System.import('lib/math').then(function(m) {
  alert("2π = " + m.sum(m.pi, m.pi));
});

// Create execution sandboxes – new Loaders
var loader = new Loader({
  global: fixup(window) // replace ‘console.log’
});
loader.eval("console.log('hello world!');");

// Directly manipulate module cache
System.get('jquery');
System.set('jquery', Module({$: $})); // WARNING: not yet finalized
```

### Map + Set + WeakMap + WeakSet
Efficient data structures for common algorithms.  WeakMaps provides leak-free object-key’d side tables.

```JavaScript
// Sets
var s = new Set();
s.add("hello").add("goodbye").add("hello");
s.size === 2;
s.has("hello") === true;

// Maps
var m = new Map();
m.set("hello", 42);
m.set(s, 34);
m.get(s) == 34;

// Weak Maps
var wm = new WeakMap();
wm.set(s, { extra: 42 });
wm.size === undefined

// Weak Sets
var ws = new WeakSet();
ws.add({ data: 42 });
// Because the added object has no other references, it will not be held in the set
```

More MDN info: [Map](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Map), [Set](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Set), [WeakMap](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/WeakMap), [WeakSet](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/WeakSet)

### Proxies
Proxies enable creation of objects with the full range of behaviors available to host objects.  Can be used for interception, object virtualization, logging/profiling, etc.

```JavaScript
// Proxying a normal object
var target = {};
var handler = {
  get: function (receiver, name) {
    return `Hello, ${name}!`;
  }
};

var p = new Proxy(target, handler);
p.world === 'Hello, world!';
```

```JavaScript
// Proxying a function object
var target = function () { return 'I am the target'; };
var handler = {
  apply: function (receiver, ...args) {
    return 'I am the proxy';
  }
};

var p = new Proxy(target, handler);
p() === 'I am the proxy';
```

There are traps available for all of the runtime-level meta-operations:

```JavaScript
var handler =
{
  get:...,
  set:...,
  has:...,
  deleteProperty:...,
  apply:...,
  construct:...,
  getOwnPropertyDescriptor:...,
  defineProperty:...,
  getPrototypeOf:...,
  setPrototypeOf:...,
  enumerate:...,
  ownKeys:...,
  preventExtensions:...,
  isExtensible:...
}
```

More info: [MDN Proxy](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Proxy)

### Symbols
Symbols enable access control for object state.  Symbols allow properties to be keyed by either `string` (as in ES5) or `symbol`.  Symbols are a new primitive type. Optional `description` parameter used in debugging - but is not part of identity.  Symbols are unique (like gensym), but not private since they are exposed via reflection features like `Object.getOwnPropertySymbols`.


```JavaScript
var MyClass = (function() {

  // module scoped symbol
  var key = Symbol("key");

  function MyClass(privateData) {
    this[key] = privateData;
  }

  MyClass.prototype = {
    doStuff: function() {
      ... this[key] ...
    }
  };

  return MyClass;
})();

var c = new MyClass("hello")
c["key"] === undefined
```

More info: [MDN Symbol](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Symbol)

### Subclassable Built-ins
In ES6, built-ins like `Array`, `Date` and DOM `Element`s can be subclassed.

Object construction for a function named `Ctor` now uses two-phases (both virtually dispatched):
- Call `Ctor[@@create]` to allocate the object, installing any special behavior
- Invoke constructor on new instance to initialize

The known `@@create` symbol is available via `Symbol.create`.  Built-ins now expose their `@@create` explicitly.

```JavaScript
// Pseudo-code of Array
class Array {
    constructor(...args) { /* ... */ }
    static [Symbol.create]() {
        // Install special [[DefineOwnProperty]]
        // to magically update 'length'
    }
}

// User code of Array subclass
class MyArray extends Array {
    constructor(...args) { super(...args); }
}

// Two-phase 'new':
// 1) Call @@create to allocate object
// 2) Invoke constructor on new instance
var arr = new MyArray();
arr[1] = 12;
arr.length == 2
```

### Math + Number + String + Array + Object APIs
Many new library additions, including core Math libraries, Array conversion helpers, String helpers, and Object.assign for copying.

```JavaScript
Number.EPSILON
Number.isInteger(Infinity) // false
Number.isNaN("NaN") // false

Math.acosh(3) // 1.762747174039086
Math.hypot(3, 4) // 5
Math.imul(Math.pow(2, 32) - 1, Math.pow(2, 32) - 2) // 2

"abcde".includes("cd") // true
"abc".repeat(3) // "abcabcabc"

Array.from(document.querySelectorAll('*')) // Returns a real Array
Array.of(1, 2, 3) // Similar to new Array(...), but without special one-arg behavior
[0, 0, 0].fill(7, 1) // [0,7,7]
[1, 2, 3].find(x => x == 3) // 3
[1, 2, 3].findIndex(x => x == 2) // 1
[1, 2, 3, 4, 5].copyWithin(3, 0) // [1, 2, 3, 1, 2]
["a", "b", "c"].entries() // iterator [0, "a"], [1,"b"], [2,"c"]
["a", "b", "c"].keys() // iterator 0, 1, 2
["a", "b", "c"].values() // iterator "a", "b", "c"

Object.assign(Point, { origin: new Point(0,0) })
```

More MDN info: [Number](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Number), [Math](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Math), [Array.from](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array/from), [Array.of](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array/of), [Array.prototype.copyWithin](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array/copyWithin), [Object.assign](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object/assign)

### Binary and Octal Literals
Two new numeric literal forms are added for binary (`b`) and octal (`o`).

```JavaScript
0b111110111 === 503 // true
0o767 === 503 // true
```

### Promises
Promises are a library for asynchronous programming.  Promises are a first class representation of a value that may be made available in the future.  Promises are used in many existing JavaScript libraries.

```JavaScript
function timeout(duration = 0) {
    return new Promise((resolve, reject) => {
        setTimeout(resolve, duration);
    })
}

var p = timeout(1000).then(() => {
    return timeout(2000);
}).then(() => {
    throw new Error("hmm");
}).catch(err => {
    return Promise.all([timeout(100), timeout(200)]);
})
```

More info: [MDN Promise](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Promise)

### Reflect API
Full reflection API exposing the runtime-level meta-operations on objects.  This is effectively the inverse of the Proxy API, and allows making calls corresponding to the same meta-operations as the proxy traps.  Especially useful for implementing proxies.

```JavaScript
// No sample yet
```

More info: [MDN Reflect](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Reflect)

### Tail Calls
Calls in tail-position are guaranteed to not grow the stack unboundedly.  Makes recursive algorithms safe in the face of unbounded inputs.

```JavaScript
function factorial(n, acc = 1) {
    'use strict';
    if (n <= 1) return acc;
    return factorial(n - 1, n * acc);
}

// Stack overflow in most implementations today,
// but safe on arbitrary inputs in ES6
factorial(100000)
```
