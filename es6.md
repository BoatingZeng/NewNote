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

// 报错
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

```js
const cat = {
  lives: 9,
  jumps: () => {
    this.lives--; // 这里this是指向全局，所以箭头函数不要作为对象的方法
  }
}

// 需要this是动态的时候，也不要用箭头函数，比如事件
var button = document.getElementById('press');
button.addEventListener('click', () => {
  this.classList.toggle('on'); // 这里this也是全局对象
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
用途：对象监听

https://github.com/mqyqingfeng/Blog/issues/107

## Reflect
把Object的一些方法放到Reflect。并且和Proxy的方法一一对应。

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
Generator里的this是没有意义的
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
var f = F.call(F.prototype); // 类似上面

f.next();  // Object {value: 2, done: false}
f.next();  // Object {value: 3, done: false}
f.next();  // Object {value: undefined, done: true}

f.a // 1
f.b // 2
f.c // 3
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
    result.value.then(function(data){
      next(data);
    });
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
function Point(){};
Point.st() = function(){}; // 实例没有这个方法，无法正常继承
Point.prototype = {
  constructor() {this.x = 0},
  toString() {},
  toValue() {},
};
```

**class定义的方法是不可枚举的，但是ES5通过prototype定义的是可枚举的。**可以利用

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

### 类的prototype属性和__proto__属性

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
B.prototype = new F();
Object.setPrototypeOf(B, A);

B.prototype.__proto__ === A.prototype // true
B.__proto__ === A // true
B.__proto__ === Function.prototype // true
```

作为一个对象，子类（B）的原型（__proto__属性）是父类（A）；作为一个构造函数，子类（B）的原型对象（prototype属性）是父类的原型对象（prototype属性）的实例。

实例的__proto__就是类的prototype
```js
let a = new A();
let b = new B();

b.__proto__ === B.prototype // true
a.__proto__ === A.prototype // true
b.__proto__.__proto__ === a.__proto__ // true
```

### Babel编译Class

1. ES6里Class的方法在prototype里是不可枚举的，Babel实现了这个特性，利用defineProperty设置enumerable为false。
2. ES6静态方法，ES5直接挂在构造函数上。Babel也是这样做。
3. 静态属性和静态方法类似。
4. ES6的class必须要用new来调用。Babel也实现了这点，通过`this instanceof Constructor`检查当前对象是不是构造函数的实例来判断是不是通过new调用。
5. Babel通过寄生组合式继承来实现。注意子类的prototype的constructor属性，要指向子类；注意子类的__proto__是父类；注意子类的prototype.__proto是父类prototype。

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