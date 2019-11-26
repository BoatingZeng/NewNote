## 组件

### Image
这个组件有点难用

https://www.hangge.com/blog/cache/detail_1542.html

* 可以把图片放到`android/app/src/main/res/drawable`目录下，打包，然后Image的uri直接填这个图片的名字(**必须不带文件后缀，带了反而出错**)。
* resizeMode可以作为组件的prop传入，也可以放在style里(可能是新版才支持)。
* 没有原生的placeholder功能，所以要自己代码实现。