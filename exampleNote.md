## 全局污染
在nodejs中。tem1.js忘记使用var或者let声明a和b，导致global被污染。浏览器中同理，不过nodejs中更难察觉。
```js
// tem1.js
a = 2;
function f(){
  b = 3;
}
f();
console.log('in tem1.js');
console.log(global.a);
console.log(global.b);

// tem2.js
require('./tem1.js');
console.log('in tem2.js');
console.log(a);
console.log(global.a);
console.log(b);
console.log(global.b);
```

## nodejs里的exports和module.exports
exports是module.exports的一个引用(short cut)。最终导出的是module.exports。所以如果只是改变了exports的指向，其实并没有改变module.exports。另外，在代码最外层打印this，指的也是最初的(最初的意思是，module.exports在代码中可能中途指向了其他对象)module.exports。详细参考nodejs笔记里的require部分。

## Object.create()和Object.setPrototypeOf()
这两个方法都会让目标对象的`__proto__`指向指定的对象。

```js
const person = {
  isHuman: false,
  printIntroduction: function () {
    console.log(`My name is ${this.name}. Am I human? ${this.isHuman}`);
  }
};

const me = Object.create(person);

me.__proto__ === person // true

const you = {};

Object.setPrototypeOf(you, person);

you.__proto__ === person; // true
```

## 定义getter和setter的方法

### 对象初始化时定义
```js
let o = {
  _num: 1,
  get num() {return this._num;},
  set num(n) {this._num = n;}
};
```

### 用Object.create
```js
let o = Object.create(Object.prototype,
  {
    num: {
      get(){return 1;},
      set(n){console.log(n);}
    }
  }
);
```

### 用Object.defineProperty
```js
let o = {_num: 1};
Object.defineProperty(o, 'num',{
  get() {return this._num;},
  set(n) {this._num = n}
});
```

## 防抖和节流的区别
* 防抖：在事件被触发n秒后再执行回调，如果在这n秒内又被触发，则重新计时。比如搜索框的输入，是用户停止一段时间后才会去执行搜索。比如window触发resize的时候，不断的调整浏览器窗口大小会不断的触发这个事件，用防抖来让其只触发一次。**最初动作未必是最终结果，等待最终确定动作。**
* 节流：规定在一个单位时间内，只能触发一次函数。如果这个单位时间内触发多次函数，只有一次生效。比如用户不停地点击，只让其中一部分点击生效。比如懒加载时要监听计算滚动条的位置，但不必每次滑动都触发。**取最初动作为结果，减少之后的重复动作。**

## 页面上传文件(ajax)

### 用FormData
最简单的办法。IE10以上才有FormData。

```js
var formData = new FormData(); // 可以用一个现有的form来构造
formData.append("userfile", fileInputElement.files[0]); // 添加一个文件(File对象)
var request = new XMLHttpRequest();
request.open("POST", "http://foo.com/submitform.php");
request.send(formData);
```

## js浮点计算问题
* https://gist.github.com/rockagen/9913346

基本思路是先把浮点数都倍化为整数，进行了四则运算，然后再除以10的n次幂。

## 常用正则
参考：https://github.com/cdoco/common-regex

* 邮箱：/^[A-Za-z0-9\u4e00-\u9fa5]+@[a-zA-Z0-9_-]+(\.[a-zA-Z0-9_-]+)+$/
* 金额，两位小数：/((^[1-9]\d*)|^0)(\.\d{0,2}){0,1}$/

## ArrayBuffer

### 和字符串的转换
如果字符串是UTF-16编码
```js
// ArrayBuffer 转为字符串，参数为 ArrayBuffer 对象
function ab2str(buf) {
  // 注意，如果是大型二进制数组，为了避免溢出，
  // 必须一个一个字符地转
  if (buf && buf.byteLength < 1024) {
    return String.fromCharCode.apply(null, new Uint16Array(buf));
  }

  const bufView = new Uint16Array(buf);
  const len =  bufView.length;
  const bstr = new Array(len);
  for (let i = 0; i < len; i++) {
    bstr[i] = String.fromCharCode.call(null, bufView[i]);
  }
  return bstr.join('');
}

// 字符串转为 ArrayBuffer 对象，参数为字符串
function str2ab(str) {
  const buf = new ArrayBuffer(str.length * 2); // 每个字符占用2个字节
  const bufView = new Uint16Array(buf);
  for (let i = 0, strLen = str.length; i < strLen; i++) {
    bufView[i] = str.charCodeAt(i);
  }
  return buf;
}
```

如果是UTF-8编码，可以用TextDecoder。TextEncoder默认只支持utf-8。
```js
const decoderUTF8 = new TextDecoder();
const decoderUTF16 = new TextDecoder('utf-16');
// 然后调用它们的decode方法，可以直接decode ArrayBuffer或者TypeArray。
```

## 杂项

### document.write
在head标签里用document.write写入script标签，会把script标签写入head里，但是如果在写script标签前用document.write写了诸如div这样的标签，那么script标签会被写到body里。