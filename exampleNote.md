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