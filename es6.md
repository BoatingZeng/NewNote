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