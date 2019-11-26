## XSS
Cross Site Scripting、跨站脚本攻击
https://www.freebuf.com/articles/web/185654.html

概念：XSS攻击是页面被注入了恶意的代码。

预防方法：

1. 对于可能插入`<script>`的，通过HTML转义，escape掉，主要是'<'和'>'，让它不能生成标签。
2. 对于带href的，禁止以 javascript: 开头的链接，和其他非法的scheme。
3. 输入过滤。
4. 前端不要用一段来历不明的字符串来作为javascript来运行。一般来说运行的代码应该都是自己的，外来的东西只能认为是数据。
5. 避免内联事件，尤其是带了模板参数的作为输入的。这些参数可能是攻击代码。

注：在HTML里插入可运行的`<script>`，通常要用`document.createElement('script')`创建标签，然后进行appendChild之类的操作。

## CSRF
Cross-site request forgery、跨站请求伪造

概念：攻击者诱导受害者进入第三方网站，在第三方网站中，向被攻击网站发送跨站请求。利用受害者在被攻击网站已经获取的注册凭证，绕过后台的用户验证，达到冒充用户对被攻击的网站执行某项操作的目的。通常是用了受害者的cookie。

预防方法：

1. 同源验证。阻止来自不可信域名的请求。简单但是并非万无一失。对于在本站发起的攻击，就无法防范。
2. CSRF Token。这个token不在cookie里发送。可以放在header或者查询参数里。
3. 双重Cookie验证。前端生成一个随机cookie，向后端请求的时候，在URL查询参数中也带上这个cookie的值，这样后端可以对比cookie中和URL中的值是否一致。这个方法相对简单，但难以做到子域名的隔离（因为前端取cookie是有域名限制的）。

## HTTP1.0、HTTP1.1、HTTP2.0、SPDY

* HTTP1.0虽然有Connection:keep-alive的定义，但是没有标准实现，到了HTTP1.1才有了标准实现并且默认开启
* SPDY位于HTTP之下，TCP和SSL之上，这样可以轻松兼容老版本的HTTP协议
* HTTP2基于SPDY，虽然HTTP2不要求https，但实际上都是需要https的
* nodejs的http2和http模块的api不同，express(版本4.x)不支持nodejs的http2模块，要用spdy
* koa支持nodejs的http2

## DOM事件
![CSS Specifishity](https://raw.githubusercontent.com/BoatingZeng/NewNote/master/img/dom_event.png)

例子，点击元素#c
```html
<ul id="p">
    <li id="c">1</li>
    <li>2</li>
    <li>3</li>
    <li>4</li>
</ul>
```

```js
let p = document.getElementById('p');
let c = document.getElementById('c');

// 情况1，因为listener默认在冒泡阶段触发，所以是先c后p
p.addEventListener('click', function(e){
    console.log('p');
    console.log(e.currentTarget);
    console.log(e.target);
});

c.addEventListener('click', function(e){
    console.log('c');
    console.log(e.currentTarget);
    console.log(e.target);
});

// 打印结果，注意顺序
// c
// <li id=​"c">​1​</li>​
// <li id=​"c">​1​</li>​
// p
// <ul id=​"p">​…​</ul>​
// <li id=​"c">​1​</li>​

// currentTarget永远都是绑事件那个元素
// target是dispatch事件那个，或者说实际被点那个

// 情况2，如果给p添加listen时设定了useCapture为true，那么p的listener在捕获阶段就触发，所以先p后c
p.addEventListener('click', function(e){
    console.log('p');
}, true);

c.addEventListener('click', function(e){
    console.log('c');
});

// 结果
// p
// c

// 情况3，在上面基础上，p的listener阻止了事件继续传播，所以事件在被p捕获后就没了
p.addEventListener('click', function(e){
    console.log('p');
    e.stopPropagation();
}, true);

c.addEventListener('click', function(e){
    console.log('c');
});

// 结果
// p

// 情况4，因为默认在冒泡阶段触发listener，所以当在c的listener停止事件传播，事件就冒泡不到p，也就触发不了p的listener，尽管它曾被p捕获过，但是这里捕获阶段不触发listener
p.addEventListener('click', function(e){
    console.log('p');
});

c.addEventListener('click', function(e){
    console.log('c');
    e.stopPropagation();
});

// 结果
// c
```

总结

* addEventListener的第三个参数useCapture，指示的是listener在什么阶段触发，true的话就是在捕获阶段触发，默认为false，在冒泡阶段触发。它并不影响事件的传播。
* stopPropagation是阻止事件的传播，在listener里调用，就是说事件触发了这个listener，调用了stopPropagation，这个事件就没后续了。

## Web Worker
* 实例参考：https://developer.mozilla.org/en-US/docs/Web/API/Web_Workers_API/Using_web_workers

* 开启一个新线程，主要通过`postMessage`和`onmessage`通讯

## Service Worker
Service Worker是一种特殊的Web Worker

* 完整的参考实例：https://developer.mozilla.org/en-US/docs/Web/API/Service_Worker_API/Using_Service_Workers
* 另外也可以参考docsify库使用的sw.js文件：https://docsify.js.org/#/pwa

* 主要用与缓存资源。还可以在浏览器端客制化请求。
* 生产环境只支持https。本地开发localhost不要求https。
* 更新过sw代码后，刷新或者载入页面会安装(install)新的sw，但是不会激活(active)，要等到所有使用旧版sw的页面都关闭后，才会激活新的。
* fetch后的response要clone一份存入cache，原本那份给浏览器，因为response只能读取一次。

## WebSocket

* ping、pong心跳，目前，浏览器中没有相关api发送ping给服务器，只能由服务器发ping给浏览器，浏览器(自动)返回pong消息。ping、pong和message是分开的，所以onmessage里是不会收到ping、pong信息的。