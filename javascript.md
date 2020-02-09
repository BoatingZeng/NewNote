## Event Loop
https://juejin.im/post/5c3d8956e51d4511dc72c200

* MacroTask(宏任务)：script全部代码(或者说同步代码本身)、setTimeout、setInterval、I/O、UI Rendering、MessageChannel
* MicroTask(微任务)：Process.nextTick（Node独有）、Promise、Object.observe(废弃)、MutationObserver
* async函数底层也是Promise，但是在不同运行环境可能有不同表现，详见下面例子

每次执行完宏任务后，检查微任务队列，把微任务队列里的全部执行。

```js
console.log('main1');

process.nextTick(function() {
    console.log('process.nextTick1');
});

setTimeout(function() {
    console.log('setTimeout');
    process.nextTick(function() {
        console.log('process.nextTick2');
    });
}, 0);

// 这个Promise一开始执行后，微任务队列就一直不空，所以上面的setTimeout会在promise then 4后打印
new Promise(function(resolve, reject) { // promise 1
    console.log('promise'); // new Promise里的那个函数是立刻执行的!所以它比main2还早打印。
    resolve();
}).then(function() { // then返回promise 2
    console.log('promise then');
    process.nextTick(function() {
        console.log('process.nextTick3'); // 会在微任务队列的最后执行
    });
    new Promise(function(resolve, reject) { // promise 3
        console.log('promise 3');
        resolve('value'); // 因为构造函数里那个函数是立刻执行的，所以这个resolve在外层函数体(console.log('promise then');开头的这个函数)返回前就执行了，所以这个promise 3比promise 2更早resolve
    }).then(function(v){ // then返回promise 4
        console.log('promise then 3 ' + v); // 会比promise then 2先执行
    }).then(function(){
      console.log('promise then 4 '); // 这个会比promise then 2迟，但是比setTimeout早
    });
    // 这里返回后，promise 2就resolve
}).then(function() {
    console.log('promise then 2'); // 这里会比setTimeout还要早执行
});

console.log('main2');
// 结果顺序：
// main1
// promise
// main2
// process.nextTick1
// promise then
// promise 3
// promise then 3 value
// promise then 2
// promise then 4
// process.nextTick3
// setTimeout
// process.nextTick2
```

```js
console.log('script start')

async function async1() {
  await async2()
  console.log('async1 end')
}
async function async2() {
  console.log('async2 end') 
}
async1()

setTimeout(function() {
  console.log('setTimeout')
}, 0)

new Promise(resolve => {
  console.log('Promise')
  resolve()
})
  .then(function() {
    console.log('promise1')
  })
  .then(function() {
    console.log('promise2')
  })

console.log('script end')

// 新版v8(chrome77上跑是新的，node10上跑是旧的)结果
// script start
// async2 end
// Promise
// script end
// async1 end 如果旧版的话，这里的async1 end会在promise2后面，原因简单来说是await使用了额外的Promise导致它延后
// promise1
// promise2
// setTimeout
```

## 内存泄漏
http://point.davidglasser.net/2013/06/27/surprising-javascript-memory-leak.html
```js
var theThing = null;
var replaceThing = function () {
    var originalThing = theThing;
    var unused = function () {
        if (originalThing)
            console.log("hi");
    };
    theThing = {
        longStr: new Array(100000000).join('*'),
        someMethod: function () {
            console.log(1111);
        } // 没有这个函数的话，也不会导致泄漏
    };
    // theThing = null; // 这样可以让上面的对象在函数内部就释放了
};

setInterval(replaceThing, 100);
```

1. unused引用了originalThing，形成闭包作用域内存空间
2. theThing的someMethod虽然没引用任何东西，但是会融合到之前的闭包作用域，这里面就包含了originalThing
3. 因为theThing是全局变量，replaceThing执行完后并不会释放
4. 下一次执行replaceThing，originalThing会指向之前的theThing，theThing的someMethod又在闭包作用域里包含了之前的originalThing，这样就形成无限链

知乎上别人的解释：https://www.zhihu.com/question/56806069/answer/150493483

在执行函数的时候，如果遇到闭包，会创建闭包作用域内存空间，将该闭包所用到的局部变量添加进去，然后再遇到闭包，会在之前创建好的作用域空间添加此闭包会用到而前闭包没用到的变量。函数结束，清除没有被闭包作用域引用的变量。

在此例中，有两个闭包。第一个unused，引用了origin，如果没有后面的闭包，unused会在函数结束后清除，闭包作用域也跟着清除了，但是因为后面闭包是全局变量，其所引用的闭包作用域一直存在，而这个作用域是包括unused的闭包作用域的（就是同一个函数内部的闭包作用域只有一个，所有闭包共享，第一段说明），所以origin因为在闭包作用域里不会被清除，而随着不断调用，我们很容易发现，origin指向前一次replace函数执行后留下的对象（该对象再通过作用域链指向闭包作用域），从而形成一个链条。造成内存泄漏。

### 参考
* https://auth0.com/blog/four-types-of-leaks-in-your-javascript-code-and-how-to-get-rid-of-them/

js中引用了dom对象，会导致即使dom对象从dom树中移除也无法回收，所以要注意把这个js中的引用也清除。另外，自节点没有被回收，父节点也不会被回收。

## let和const

在for里使用let

```js
for(let i=0;i<10;i++){}
// 相当于
{
  let i;
  for(i=0;i<10:i++){}
}
```

let只能出现在当前作用域的顶层。这个规则适用于function声明。

```js
// 报错。没有大括号，所以不存在块级作用域，let只能出现在当前作用域的顶层
// SyntaxError: Lexical declaration cannot appear in a single-statement context
if (true) let x = 1;

// 不报错
if (true) {
  let x = 1;
}

// 不报错
'use strict';
if (true) {
  function f() {}
}

// 报错 SyntaxError: In strict mode code, functions can only be declared at top level or inside a block.
'use strict';
if (true) function f() {}
```

## 解构赋值

常用情景

```js
// 1、交换变量的值
let x = 1;
let y = 2;

[x, y] = [y, x];

// 2、从函数返回多个值
function example() {
  return [1, 2, 3];
}
let [a, b, c] = example();

function example() {
  return {
    foo: 1,
    bar: 2
  };
}
let { foo, bar } = example();

// 3、函数参数的定义，将一组参数与变量名对应起来
function f([x, y, z]) {}
f([1, 2, 3]);

function f({x, y, z}) {}
f({z: 3, y: 2, x: 1});

// 4、提取json数据
let jsonData = {
  id: 42,
  status: "OK",
  data: [867, 5309]
};

let { id, status, data: number } = jsonData;

// 5、函数参数的默认值
function move({x = 0, y = 0} = {}) {
  // x和y默认是0
  return [x, y];
}
move({x: 3, y: 8}); // [3, 8]
move({x: 3}); // [3, 0]
move({}); // [0, 0]
move(); // [0, 0]
// 注意下面这种写法意义不同。下面这种是为move的参数提供了默认值，而不是为对象里的x和y提供默认值。
function move({x, y} = { x: 0, y: 0 }) {
  return [x, y];
}
move({x: 3, y: 8}); // [3, 8]
move({x: 3}); // [3, undefined]
move({}); // [undefined, undefined]
move(); // [0, 0]

// 6、遍历Map结构
const map = new Map();
map.set('first', 'hello');
map.set('second', 'world');

for (let [key, value] of map) {
  console.log(key + " is " + value);
}

// 7、输入模块的指定方法
const { SourceMapConsumer, SourceNode } = require("source-map");
```

## 字符串

js的字符串是Unicode字符集的UTF16BE编码。对于超出0xFFFF码点的字符，length是不可信的。

```js
console.log('中');
console.log('\u4e2d');//中，Unicode和UTF16BE一致
console.log('𠮷');
console.log('\u{20BB7}');//𠮷，Unicode
console.log('\uD842\uDFB7');//𠮷，UTF16BE

let str1 = '𠮷';
console.log(str1.length);//2
console.log(Buffer.from(str1)); // 默认是utf8，<Buffer f0 a0 ae b7>，文件是utf-8编码时是正确的。如果文件保存成GBK编码，会得到错误结果。文件是utf-16的话，nodejs运行不了。
console.log(Buffer.from(str1, 'utf16le')); // <Buffer 42 d8 b7 df>
let str2 = '中';
console.log(str2.length);//1
console.log(Buffer.from(str2)); // <Buffer e4 b8 ad>
console.log(Buffer.from(str2, 'utf16le')); // <Buffer 2d 4e>
```

## 函数

### 默认值
参数默认值不是传值的，而是每次都重新计算默认值表达式的值

```js
let x = 99;
function foo(p = x + 1) {
  console.log(p);
}
foo() // 100
x = 100;
foo() // 101
```

利用参数默认值，可以指定某一个参数不得省略，如果省略就抛出一个错误。

```js
function throwIfMissing() {
  throw new Error('Missing parameter');
}
function foo(mustBeProvided = throwIfMissing()) {
  return mustBeProvided;
}
foo()
// Error: Missing parameter
```

### 函数的length
1. 指定了默认值以后，函数的length属性，将返回没有指定默认值的参数个数
2. rest参数也不会计入length属性
3. 如果设置了默认值的参数不是尾参数，那么length属性也不再计入后面的参数

```js
(function (a) {}).length // 1
(function (a = 5) {}).length // 0
(function (a, b, c = 5) {}).length // 2
(function(...args) {}).length // 0
(function (a = 0, b, c) {}).length // 0
(function (a, b = 1, c) {}).length // 1
```

### 作用域

```js
var x = 1;
function f(x, y = x) {// 这里y=x的x是前面的参数x
  console.log(y);
}
f(2) // 2

let x = 1;
function f(y = x) {// 这y=x的x是外层x
  let x = 2;
  console.log(y);
}
f() // 1

function f(y = x) {
  let x = 2;
  console.log(y);
}
f() // ReferenceError: x is not defined

var x = 1;
function foo(x = x) {// 相当于let x=x
  // ...
}
foo() // ReferenceError: x is not defined

var x = 1;
function foo(x, y = function() { x = 2; }) {
  var x = 3; // 函数体内部声明var x，函数体后面的x都是函数体内部声明的x。
  y();
  console.log(x);
}
foo() // 3
x // 1

var x = 1;
function foo(x, y = function() { x = 2; }) {
  x = 3; // 这个x就是参数x。函数参数相当于函数体的外层作用域。
  y();
  console.log(x);
}
foo() // 2
x // 1

function foo(x, y = function() { x = 2;}) {}
// 相当于
{
  let x;
  let y = function() { x = 2;};
  function foo() {// 这里可以访问x和y};
}
```

### rest参数
```js
function add(...values) {
  let sum = 0;
  for (var val of values) { // values是个数组
    sum += val;
  }
  return sum;
}
add(2, 5, 3) // 10

// 代替arguments
// arguments变量的写法
function sortNumbers() {
  return Array.prototype.slice.call(arguments).sort();
}

// rest参数的写法
const sortNumbers = (...numbers) => numbers.sort();
```

### 箭头函数
要注意的问题

* 函数体内的this对象，就是定义时外层作用域的this，会随外层作用域的this而改变。或者说箭头函数本身没有this，它只是外层作用域this的一个引用。不能(用call等)直接改变箭头函数的this

```js
function foo() {
  setTimeout(() => {
    console.log('id:', this.id); // 这里的this保持和foo的this一致
  }, 100);
}

// 或者用老方法(闭包，通过that引用指向外面的this)
function foo() {
    var that = this;
    setTimeout(() => {
    console.log('id:', that.id); // 这里的that保持和foo的this一致
  }, 100);
}

var id = 21;
foo.call({ id: 42 }); // id: 42
foo(); // id: 21

const bar = {
  id: 233,
  baz: () => {
    console.log('id:', this.id); // 外层作用域就是全局了，所以这里的this永远是全局对象
  }
}
bar.baz.call(bar); // id: 21 这里this依然是全局
bar.baz(); // id: 21

const cat = {
  lives: 9,
  jumps: () => {
    console.log(lives); // 这里this是指向全局，所以箭头函数不要作为对象的方法
  }
}

// 在class里
class Cat{
  lives = 9;

  jumps = () => {
    console.log(lives); // 这里let c = new Cat之后，c.jumps()里的this却是c本身。
  }
}

Cat.prototype.jumps; // undefined 惊了，jumps不在prototype里
// 详细解释：https://javascriptweblog.wordpress.com/2015/11/02/of-classes-and-arrow-functions-a-cautionary-tale/
// 这里这个jumps，并不是类的方法，而是每个实例自己的属性，相当于下面这种写法
function Cat() {
  this.lives = 9;
  this.jumps = () => {
    console.log(this.lives); // 外层是Cat函数，所以这个jumps的this就是Cat的this
  }
}
var c = new Cat(); // 类比：var c = {}; Cat.call(c);
c.jumps(); // 9
c.jumps.call({lives: 999}); // 9

// 事件注册时注意
var button = document.getElementById('press');
button.addEventListener('click', () => {
  this.classList.toggle('on'); // 这里this也是外层作用域的，因为这里外层就是全局，所以this是window
});
```

## 数组

### 扩展运算符(...)
将一个数组转为用逗号分隔的参数序列

```js
console.log(1, ...[2, 3, 4], 5); // 1 2 3 4 5

let arr1 = [1,2,3];
let arr2 = [0, ...arr1, 4]; // [0,1,2,3,4]

const [first, ...rest] = [1, 2, 3, 4, 5];
first // 1
rest  // [2, 3, 4, 5]

// 正确的获取字符串长度，下面的\uD83D\uDE80是一个字符，直接获取length会被当成2个
function length(str) {
  return [...str].length;
}
length('x\uD83D\uDE80y') // 3
```

### 空位(坑)
```js
let emp = Array(3); // [empty × 3]
emp[0] // undefined
undefined in emp // false
```

## 对象

### 遍历对象属性的方法
1. for...in：循环遍历对象自身的和继承的可枚举属性（不含 Symbol 属性）。需要用hasOwnProperty过滤掉继承的。
2. Object.keys(obj)：返回一个数组，包括对象自身的（不含继承的）所有可枚举属性（不含 Symbol 属性）的键名。不需要hasOwnProperty。
3. Object.getOwnPropertyNames(obj)：返回一个数组，包含对象自身的（不包含继承的）所有属性（不含 Symbol 属性，但是包括不可枚举属性）的键名。
4. Object.getOwnPropertySymbols(obj)：返回一个数组，包含对象自身的所有 Symbol 属性的键名。
5. Reflect.ownKeys(obj)：返回一个数组，包含对象自身的所有键名，不管键名是 Symbol 或字符串，也不管是否可枚举。

注：属性的是否可枚举，是可以设定的。比如数组的`length`就是不可枚举属性。给一个对象直接用点运算符赋值的是可枚举，`defineProperties`定义的属性默认不可枚举。

### 扩展运算符(...)
用于取出参数对象的所有可遍历属性，拷贝到当前对象之中

```js
let z = { a: 3, b: 4 };
let n = { ...z }; // { a: 3, b: 4 }
```

## Symbol

Symbol.for()是全局的。在不同的 iframe 或 service worker里也是这样。

```js
// nodejs
// tem1.js
module.exports = Symbol.for("bar");

// tem2.js
let tem1_s = require('./tem1.js');
let s = Symbol.for("bar");
console.log(s === tem1_s); // true

// 浏览器
iframe = document.createElement('iframe');
iframe.src = String(window.location);
document.body.appendChild(iframe);
iframe.contentWindow.Symbol.for('foo') === Symbol.for('foo') // true
```

## Set、Map、WeakSet、WeakMap

### Set
1. 在Set内部，NaN是同一个值
2. 对象是按引用来区分

### WeakSet
1. 成员只能是对象
2. 如果外部不再引用WeakSet内某个对象，它会被回收

### Map
1. 跟Object相比，Map可以用对象作为键

### WeakMap
1. 键只能是对象
2. 如果外部不再引用WeakMap内某个键，它会被回收

## Proxy
用途：对象监听。比起definePropety的setter和getter，Proxy可以拦截关键字或者运算符还有函数等的默认行为。而且不用一个个属性通过definePropety定义，而是可以通过统一的handler拦截。

set和get这两个拦截器的receiver参数，一般情况都不用，主要是在当对象的属性不是正常访问和设置时使用。(正常访问和设置，一般就是通过点运算符来访问和设置)。见下面例子。

```js
var proxy = new Proxy({}, {
  get: function(target, property, receiver) {
    return receiver; // 访问什么属性都是获得这个receiver
  }
});
proxy.getReceiver === proxy; // true
var inherits = Object.create(proxy);
inherits.getReceiver === inherits; // true
```

https://github.com/mqyqingfeng/Blog/issues/107

## Reflect
是个对象，不是函数,包含13个静态方法。其方法和Proxy的方法一一对应。其实就是个函数库。

## Iterator和for...of
有Symbol.iterator属性的对象，就是可遍历的(可for..of)
```js
const obj = {
  [Symbol.iterator] : function () {
    return {
      next: function () {
        return {
          value: 1,
          done: true
        };
      }
    };
  }
};
```

### yield*
yield*后面跟的是一个可遍历的结构，它会调用该结构的遍历器接口。

```js
let generator = function* () {
  yield 1;
  yield* [2,3,4];
  yield 5;
};
var iterator = generator();
iterator.next() // { value: 1, done: false }
iterator.next() // { value: 2, done: false }
iterator.next() // { value: 3, done: false }
iterator.next() // { value: 4, done: false }
iterator.next() // { value: 5, done: false }
iterator.next() // { value: undefined, done: true }
```

## Generator
Generator里的this
```js
function* F() {
  this.a = 1;
  yield this.b = 2;
  yield this.c = 3;
}
var obj = {};
var f = F.call(obj); // 这里把F绑到obj，让F里的this指向obj

f.next();  // Object {value: 2, done: false}
f.next();  // Object {value: 3, done: false}
f.next();  // Object {value: undefined, done: true}

obj.a // 1
obj.b // 2
obj.c // 3

function* F() {
  this.a = 1;
  yield this.b = 2;
  yield this.c = 3;
}
var f = F.call(F.prototype); // 类似上面，F里的this是F.prototype，而f.__proto__就是F.prototype，所以下面的f通过原型链访问到了abc

f.next();  // Object {value: 2, done: false}
f.next();  // Object {value: 3, done: false}
f.next();  // Object {value: undefined, done: true}

f.a // 1
f.b // 2
f.c // 3

var myIterable = {
  a:1
}

myIterable[Symbol.iterator] = function* () {
  yield this.a;
  yield this.a;
  yield this.a;
};

[...myIterable] // [1,1,1]，就是myIterable.a

function* g() {
  this.b = 11; // 这个this其实就是全局对象
}

let obj = g();

obj.next();

console.log(b); // 打印11
```

### next 方法的参数
yield表达式本身没有返回值，或者说总是返回undefined。next方法可以带一个参数，该参数就会被当作**上一个**yield表达式的返回值。

```js
function* f() {
  for(var i = 0; true; i++) {
    var reset = yield i;
    if(reset) { i = -1; }
  }
}
var g = f();
g.next() // { value: 0, done: false }
g.next() // { value: 1, done: false }
g.next(true) // { value: 0, done: false }

function* foo(x) {
  var y = 2 * (yield (x + 1));
  var z = yield (y / 3);
  return (x + y + z);
}
var a = foo(5);
a.next() // Object{value:6, done:false}
a.next() // Object{value:NaN, done:false}
a.next() // Object{value:NaN, done:true}
var b = foo(5);
b.next() // { value:6, done:false }
b.next(12) // { value:8, done:false } 此时内部y=2*12=24
b.next(13) // { value:42, done:true } 此时内部z=13。所以最终x+y+z=42
```

### generator的自执行
```js
function wait(time, data){
  console.log('wait start');
  return new Promise((resolve, reject)=>{
    setTimeout(function(){
      console.log('wait end');
      resolve(data);
    }, time);
  });
}

function* gen(){
  console.log('start');
  let d1 = yield wait(2000, 'd1');
  console.log('after wait 2 second', d1);
  let d2 = yield wait(1000, 'd2');
  console.log('after wait 1 second', d2);
}

function run(gen){
  var g = gen();

  function next(data){
    var result = g.next(data);
    if (result.done) return result.value;
    result.value.then(next);
  }

  next();
}

run(gen);
```

## async函数
1. await其实就是等待后面那个Promise resolve
2. async函数返回Promise，如果return一个普通值，那它会被转化为Promise
3. await后面的Promise如果被reject，是可以try..catch的

## class

### 基本语法
```js
class Point {
  x = 0 // 实例属性，也可以写在constructor里，this.x=0
  constructor() {}
  toString() {}
  toValue() {}
  // 静态方法，实例没有这个方法，子类可以继承这个方法，静态方法内的this指向的是类
  static st(){}
}

// 等同于
function Point(){this.x = 0};
Point.st() = function(){}; // 实例没有这个方法，无法正常继承
Point.prototype.toString = function() {};
Point.prototype.toValue = function() {};
```

**class定义的方法是不可枚举的，但是ES5通过prototype用点运算符定义的是可枚举的。**

```js
class Point {
  constructor(x, y) {}

  toString() {}
}
Object.keys(Point.prototype)// []
Object.getOwnPropertyNames(Point.prototype)// ["constructor","toString"]

var Point = function (x, y) {};
Point.prototype.toString = function() {};

Object.keys(Point.prototype)// ["toString"]
Object.getOwnPropertyNames(Point.prototype)// ["constructor","toString"]

// 用defineProperties定义的话，默认就不可枚举了。
Object.defineProperties(Point.prototype, {
    toString: {
        value: function() {}
    }
});
```

### new.target关键字
该属性一般用在构造函数之中，返回new命令作用于的那个构造函数。比如`new Point()`，那`new.target===Point`。

### 继承
基本语法
```js
class Point {}

class ColorPoint extends Point {}

class ColorPoint extends Point {
  constructor(x, y, color) {
    super(x, y); // 调用父类的constructor(x, y)，必须在this前，而且必须调用super
    super.p(); // 通过super关键字调用父类方法
    this.color = color;
  }

  toString() {
    return this.color + ' ' + super.toString(); // 调用父类的toString()
  }
}

Object.getPrototypeOf(ColorPoint) === Point // true 可以用这个方法获取父类，其实是获取__proto__
```

### super
可以作为父类构造函数，但是它返回的是子类实例。
```js
class A {}

class B extends A {
  constructor() {
    super(); // 相当于A.prototype.constructor.call(this)
  }
}
```

super作为对象时，在普通方法中，指向父类的原型对象；在静态方法中，指向父类。

```js
class A {
  static st() {
    console.log('static');
  }
  p() {
    return 2;
  }
}

class B extends A {
  constructor() {
    super();
    console.log(super.p()); // 2 super.p()就相当于A.prototype.p()
  }

  static newSt() {
    super.st(); // super.st()相当于A.st()
  }
}

let b = new B();
```

在子类普通方法中通过super调用父类的方法时，方法内部的this指向当前的子类实例。

```js
class A {
  constructor() {
    this.x = 1;
  }
  print() {
    console.log(this.x);
  }
}

class B extends A {
  constructor() {
    super();
    this.x = 2;
  }
  m() {
    super.print(); // print里的this是B类实例
  }
}

let b = new B();
b.m() // 2
```

如果通过super对某个属性赋值，这时super就是this，赋值的属性会变成子类实例的属性。

```js
class A {
  constructor() {
    this.x = 1;
  }
}

class B extends A {
  constructor() {
    super();
    this.x = 2;
    super.x = 3;
    console.log(super.x); // undefined
    console.log(this.x); // 3
  }
}

let b = new B();
```

### 类的`prototype`属性和`__proto__`属性

1. 子类的`__proto__`属性，表示构造函数的继承，总指向父类
2. 子类的`prototype`属性的`__proto__`属性，表示方法的继承，总指向父类的`prototype`属性

```js
class A {
}

A.__proto__ === Function.prototype // true
A.prototype.__proto__ === Object.prototype // true

class B extends A {
}

B.__proto__ === A // true
B.prototype.__proto__ === A.prototype // true

class C extends null{}

C.__proto__ === Function.prototype; // true
C.prototype.__proto__ === undefined; // true
```
上面的继承相当于这样
```js
class A {
}

class B {
}

// B 的实例继承 A 的实例
Object.setPrototypeOf(B.prototype, A.prototype);

// B 继承 A 的静态属性
Object.setPrototypeOf(B, A);

// Object.setPrototypeOf方法的实现
Object.setPrototypeOf = function (obj, proto) {
  obj.__proto__ = proto;
  return obj;
}
```
也和下面这种方法相似
```js
function A(){}

function B(){}

var F = function(){};

F.prototype = A.prototype
B.prototype = new F(); // 这里没有保证B.prototype的constructor属性
Object.setPrototypeOf(B, A);

B.prototype.__proto__ === A.prototype // true
B.__proto__ === A // true 因为Object.setPrototypeOf(B, A);
A.__proto__ === Function.prototype // true

// 原生类Array为例
Array.prototype.__proto__ === Object.prototype // true
Array.__proto__ === Function.prototype // true 任何一个没修改过__proto__的函数，都是Function的实例

// 作为一个对象，子类（B）的原型（__proto__属性）是父类（A）；作为一个构造函数，子类（B）的原型对象（prototype属性）是父类的原型对象（prototype属性）的实例。

// 实例的__proto__就是类的prototype
let a = new A();
let b = new B();

b.__proto__ === B.prototype // true
a.__proto__ === A.prototype // true
b.__proto__.__proto__ === a.__proto__ // true
```

### Babel编译Class

1. ES6里Class的方法在`prototype`里是不可枚举的，Babel实现了这个特性，利用`defineProperty`设置`enumerable`为`false`。
2. ES6静态方法，ES5直接挂在构造函数上。Babel也是这样做。
3. 静态属性和静态方法类似。
4. ES6的class必须要用new来调用。Babel也实现了这点，通过`this instanceof Constructor`检查当前对象是不是构造函数的实例来判断是不是通过new调用。
5. Babel通过寄生组合式继承来实现。注意子类的`prototype`的`constructor`属性，要指向子类；注意子类的`__proto__`是父类；注意子类的`prototype.__proto__`是父类`prototype`。

```js
// ES6继承
class Parent {
    constructor(name) {
        this.name = name;
    }
}

class Child extends Parent {
    constructor(name, age) {
        super(name); // 调用父类的 constructor(name)
        this.age = age;
    }
}

// Babel实现和下面类似
function Parent (name) {
    this.name = name;
}

function Child (name, age) {
    Parent.call(this, name);
    this.age = age;
}

function prototype(c, p) {
    // c.prototype.__proto__ === p.prototype
    var prototype = Object.create(p.prototype, {
      constructor: {
        value: c,
        writable: true,
        configurable: true
      }
    });
    c.prototype = prototype;
    Object.setPrototypeOf(c, p); // c.__proto__ === p
}

prototype(Child, Parent);

var child1 = new Child('kevin', '18');
```

### 构造函数有返回值
* https://www.cnblogs.com/guanghe/p/11356347.html

原则上构造函数是不应该有返回值的。如果有，分下面两种情况。

1. 返回值类型，对函数没影响
2. 如果返回引用值，则new出来的对象是这个引用值

## Promise

### 关于then和catch的分析

* 每次then或者catch，都会产生一个新的Promise
* 在同一条链上的Promise，如果前面有rejected，后面就都rejected的
* Promise链是可以分支的，而且要在每个分支的末端catch好，否则可能会有错误抛出到外面
* catch里如果抛出错误，那么这个catch后产生的Promise是rejected的，否则是resolved的。所以原则上catch里不能抛出错误，否则就无止境了。
* 如果没有特殊需求，一般来说都不去分支Promise，一条链比较直观，然后最末端catch。像下面的分支示例就比较迷惑人了。

```js
var p = new Promise((resolve, reject) => {
  setTimeout(() => {
    reject(new Error('error'));
  }, 1000);
});

// 下面p1和p2是两个不同分支了
var p1 = p.then(v => {
  console.log(v);
  return 'p1 then';
});

var p2 = p.catch(e => {
  console.error('catch p', e); // catch p Error: error
  return 'p2 then';
});

// 因为p本身是rejected的，所以p1分支的末尾要catch
p1.catch(e => {
  console.error('catch p1', e); // catch p1 Error: error
});

// 这里then之后又从p2分了一个分支
p2.then(v => {
  console.log('v in p2：', v); // v in p2： p2 then 说明catch后也是返回一个Promise，如果catch的回调里没有抛出错误，它就resolved了
}).catch(e => {
  console.error('catch in p2 then：', e); // 走不到这里，因为前面没有未catch错误。但是这里是应该catch的，因为是分支末端。
});

// 因为catch之后还是返回Promise的，所以理论上也是可以继续catch(因为上面p2.then后面catch过，所以这个p2.catch是没必要的，如果上面p2.then后面没有catch，那这里是有必要的)，详细看下一个例子
p2.catch(e => {
  console.error('catch p2', e); // 走不到这里，因为p里的错误已经被catch过而且p2分支里也没出现错误
});

console.log(p1); // PromiseStatus: rejected; PromiseValue: Error: error
console.log(p2); // PromiseStatus: resolved; PromiseValue: p2 then
```

```js
var p = new Promise((resolve, reject) => {
  setTimeout(() => {
    reject(new Error('error'));
  }, 1000);
});

var p1 = p.then(v => {
  console.log(v);
  return 'p1 then';
});

// 这里catch后又抛出错误，导致p2后的分支都是rejected的
var p2 = p.catch(e => {
  console.error('catch p', e); // catch p Error: error
  throw new Error('throw in p2'); // 这会导致p2 reject
});

p1.catch(e => {
  console.error('catch p1', e); // catch p1 Error: error
});

p2.then(v => {
  console.log('v in p2：', v); // 走不到这里
}).catch(e => {
  console.error('catch in p2 then：', e); // catch in p2 then： Error: throw in p2 因为p2是rejected的，所以这里也要catch
});

// 这里就是刚catch过又catch的情况，没完没了，因为之前的p.catch里抛出了错误(因为上面p2.then后面catch过，所以这个p2.catch是没必要的，如果上面p2.then后面没有catch，那这里是有必要的)
p2.catch(e => {
  console.error('catch p2', e); // catch p2 Error: throw in p2
});

console.log(p1); // PromiseStatus: rejected; PromiseValue: Error: error
console.log(p2); // PromiseStatus: rejected; PromiseValue: Error: throw in p2
```

then的第一个参数传个非函数参数，那么它会被忽略，上一个Promise的结果会传到下一个then。当然，这些奇怪行为都是因为js没有类型检查导致的，实际代码中根本就不应该传个非函数参数进去。

```js
let p1 = new Promise(function(resolve, reject) {resolve('foo');});
p1.then(1).then(function(res){console.log(res);}); // 打印foo
```