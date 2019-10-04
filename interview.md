## 参考链接
* 前端面试准备(不含讲解)：https://zhuanlan.zhihu.com/p/69304513
* 前端面试题(含讲解)：https://github.com/woai3c/Front-end-basic-knowledge

## MVC
Model、View、Controller

https://blog.csdn.net/u013282174/article/details/51220199

1. M：负责操作（获取、更新）数据（一般是指后端数据）。
2. V：展示数据和接收用户操作。
3. C：中间层。沟通M和V。
4. MV：M和V之间不应该有交互。
5. VC：V把事件传给C处理。C根据数据来告诉V怎么进行渲染。通常来说，一个V有特定的数据格式需求，C就是要把数据整成V所需的。
6. MC：C观测M的变化，把数据变化反应到V。或者C主动调用M获取数据。

## MVVM
Model、View、ViewModel

https://zhuanlan.zhihu.com/p/53703176

1. M和V的概念和MVC里类似。
2. VM代替C沟通M和V。
3. VM把M的变化反应到V，同时也会把V的变化反应到M。比如在Vue里，就是用VM把V和M绑定起来。

## 数据库范式

https://www.zhihu.com/question/24696366/answer/29189700
https://www.zhihu.com/question/24696366/answer/29049568

1. 第一范式(1NF)：一范式就是属性不可分割。
2. 第二范式(2NF)：二范式就是要有主键，要求其他字段都依赖于主键。
3. 第三范式(3NF)：三范式就是要消除传递依赖，方便理解，可以看做是“消除冗余”，就是各种信息只在一个地方存储，不出现在多张表中。

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