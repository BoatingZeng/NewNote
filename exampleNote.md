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
exports是module.exports的一个引用(short cut)。最终导出的是module.exports。所以如果只是改变了exports的指向，其实并没有改变module.exports。

## Object.create()和Object.setPrototypeOf()
这两个方法都会让目标对象的__proto__指向指定的对象。

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

Object.setPrototypeOf(me, person);

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
* 防抖：在事件被触发n秒后再执行回调，如果在这n秒内又被触发，则重新计时。比如搜索框的输入，是用户停止一段时间后才会去执行搜索。比如window触发resize的时候，不断的调整浏览器窗口大小会不断的触发这个事件，用防抖来让其只触发一次。
* 节流：规定在一个单位时间内，只能触发一次函数。如果这个单位时间内触发多次函数，只有一次生效。比如用户不停地点击，只让其中一部分点击生效。比如懒加载时要监听计算滚动条的位置，但不必每次滑动都触发。

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