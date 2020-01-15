系统地学一次比较好：https://zhuanlan.zhihu.com/p/35193339

## 选择器

### 基本选择器
1. #id：id选择器
2. .class：类选择器
3. [attribute]或者[attribute=value]：属性选择器。除了`=`全匹配，还有以下几种：
  * `~=`：包含某个单词(必须用空格分隔)
  * `|=`：以某个单词开头(后面可以跟`-`但是不能跟空格)
  * `*=`：包含某个子串(任何字符都可以)
  * `^=`：以某几个字符开头
  * `$=`：以某几个字符结尾
4. :link：伪类或伪元素，标准来说`:`开头是伪类，`::`开头是伪元素

### 伪类和伪元素的区分
* 伪类，标准来说要用`:`开头，表示元素的一些状态或者属性
* 伪元素，标准来说要用`::`开头，会创建一些不存在于文档树中的元素
* 简单来说，它们的主要区别是有没有创建一个文档树之外的元素。例如：`:first-child`是伪类，表示作为其父元素第一个子元素，这个元素本来就存在的。`::first-letter`是伪元素，表示(innerText)第一个字母，这个第一个字母本身虽然存在，但是它并不是一个元素，用这个伪元素，就可以把它作为一个元素来选择，所以是创建了一个文档树之外的元素。

### 组合
1. 两个选择器间没有空格，表示交集
2. 空格，表示后代(包括后代的后代)，多次空格，则是指定后代的后代
3. `>`：表示子元素(直接后代)，多个>，则是子元素的子元素
4. `+`：表示同辈的后一个元素。p+p:与前一个p同辈的后一个p。
5. `~`：选择同辈的后面的所有元素。p~p，与前p同辈的后面所有p。

## 优先级(特殊性、specific)
![CSS Specifishity](https://raw.githubusercontent.com/BoatingZeng/NewNote/master/img/css_specifishity.png)

## 继承
* 后代元素用了祖先元素的样式，就叫继承
* 大多数框模型属性（包括外边距、内边距、背景和边框）都不能继承
* 继承的值连0特殊性都没有，也就是0特殊性(比如通配符选择器)的声明比继承的优先级高
* 通常，直接指定的样式优先于继承样式，但是通过inherit则反过来(当然，通过inherit继承样式，其实也是指定样式，只是值是继承过来)

## 层叠
当特殊性相同时，怎样决定使用哪个规则

* 有!important的(权重)高于没有!important
* 通常页面作者的样式权重高于用户
* 用户带!important的样式权重高于一切
* 用户和作者的样式都高于浏览器默认
* 排在后面的声明，权重更高

## 块级元素、行内元素

### 块级元素
大多为结构性标记，例如：`<address>,<h1>,<div>,<p>,<table>,<ul>`

1. 总是从新的一行开始
2. 高度、宽度都是可控的
3. 宽度没有设置时，默认为100%
4. 块级元素中可以包含块级元素和行内元素

### 行内元素
大多为描述性标记，例如：`<span>,<a>,<b>,<input>,<select>`

1. 和其他元素都在一行
2. 行内元素只能包含行内元素，不能包含块级元素(否则行为会很诡异)
3. 不可以设置宽高，宽度高度随文本内容的变化而变化，但是可以设置行高
4. margin上下无效，左右有效；padding上下左右都有效

## 浮动
* 浮动元素的外边距不会合并
* 浮动元素的包含块是其最近的块级祖先元素
* 浮动元素会生成块级框(即使它本身是行内元素)
* 浮动元素不会互相覆盖，如果会发生互相覆盖，新来的会往下面排
* 多个浮动元素排起来不能超出父元素宽度(但是单个很宽的浮动元素会超出父元素宽度)
* 浮动元素的顶端与其所在行之后的行框的顶端对齐

## flex布局
* http://www.ruanyifeng.com/blog/2015/07/flex-grammar.html
* http://www.ruanyifeng.com/blog/2015/07/flex-examples.html
* https://blog.csdn.net/cc18868876837/article/details/88138057

容器属性
* flex-direction：主轴方向，下面的说明都以row为主轴
* flex-wrap：一行排不下是否要换行
* flex-flow：flex-direction和flex-wrap的组合
* justify-content：项目在主轴上的对齐方式
* align-items：一行内各个项的对齐方式(在交叉轴方向上)
* align-content：每行作为整体在flex box中的对齐方式(在交叉轴方向上)

项目属性
* order：排列顺序，越小越靠前
* flex-grow：放大比例，默认为0不放大。不为0的项，按照比例占据剩余空间
* flex-shrink：缩小比例，默认为1，即如果空间不足，该项目将缩小。如果为0，则该项不缩小
* flex-basis：项目被缩放前的原始大小
* flex：以上三项的组合
* align-self：相当于针对单个项目设置align-items。默认为auto，继承父元素的align-items。

![flex](https://raw.githubusercontent.com/BoatingZeng/NewNote/master/img/flex.png)

## grid布局

## BFC(Block Formatting Context、块格式化上下文)
https://zhuanlan.zhihu.com/p/25321647
https://www.sitepoint.com/understanding-block-formatting-contexts-in-css/

符合一下其中之一，会产生BFC

* float不为none
* position不是static和relative
* display值是table-cell,table-caption,inline-block,flex,inline-flex其中之一
* overflow不是visible

作用

* 阻止margin合并，因为在同一个BFC内的margin会合并，如果不希望合并，把它们分在不同BFC
* 用BFC包含浮动元素
* 用BFC阻止文字环绕(浮动元素)
* 用于多列布局(事实上用flex更好)

## 媒体查询
http://nec.netease.com/framework/css-media.html
```css
/* media */
/* 横屏 */
@media screen and (orientation:landscape){
}
/* 竖屏 */
@media screen and (orientation:portrait){
}
/* layout viewport见viewport一节 */
/* 设计宽度，是指在这个layout viewport范围内，按照这个设计宽度来布局网页比较合适 */
/* layout viewport宽度<960,就使用下面样式，下同。设计宽度=768 */
@media screen and (max-width:959px){
}
/* layout viewport宽度<768,设计宽度=640 */
/* 注意，max-width:959px块里的样式依然生效(除非主动覆盖)，所以使用max-width的话，小的要放下面以便覆盖上面样式，使用min-width则相反 */
@media screen and (max-width:767px){
}
/* layout viewport宽度<640,设计宽度=480 */
@media screen and (max-width:639px){
}
/* layout viewport宽度<480,设计宽度=320 */
@media screen and (max-width:479px){
}
/* windows UI 贴靠 */
@media screen and (-ms-view-state:snapped){
}
/* 打印 */
@media print{
}
```

## viewport
通俗来说，就是拿个放大镜(手机)去看电脑浏览器上的网页。参考：https://www.cnblogs.com/xiaocaiyuxiaoniao/p/8084707.html

```html
<!-- 一般都这样设置，这样其实就是layout viewport和visual viewport都和设备屏幕一样大 -->
<!-- 也就是放大镜缩放比例是1，而且电脑浏览器的大小和你放大镜镜片大小一样 -->
<meta name="viewport" content="width=device-width, initial-scale=1">
```

* layout viewport：就是放置网页的假想空间。
* visual viewport：理解成一个投影层，它把layout viewport的一部分投影到设备屏幕，它越大，它所投影的layout viewport范围就越大(也就是网页被展示的范围越大，直观上看，就是看到的字更小，因为屏幕大小是不变的)。
* 设备屏幕：就是实际展示画面的地方。
* width：layout viewport的宽度。
* initial-scale：直观上理解，就是内容的缩放比例，值越大，内容就越大。实际上，它其实等于(设备屏幕大小/visual viewport大小)，而设备的大小是不变的，scale越大，visual viewport越小。

## 参考链接汇总
因为实在太杂太多了。所以放参考记录参考链接方便查水表。先记录一些汇总链接，然后后面补充一些特定主题的。

### 无主题或者杂主题
* HTML和CSS常见问题整理：https://github.com/yygmind/blog/issues/3
* 壹题汇总的CSS部分：https://muyiy.cn/question/

### 面试题
* https://segmentfault.com/a/1190000013325778
* https://zhuanlan.zhihu.com/p/66516864