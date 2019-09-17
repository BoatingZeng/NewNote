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