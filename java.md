虽说这个分类叫java，但并不只是放语言本身相关的笔记。

## java语言

### 泛型
参考：
* 使用场景：https://www.cnblogs.com/mzzcy/p/7231892.html

```java
class Food {}
class Fruit extends Food {}
class Meat extends Food {}
class Apple extends Fruit {}
class Orange extends Fruit {}
class Plate<T> {
    private T f;
    public Plate() {};
    public Plate(T f) {this.f = f;}
    public void set(T f) {this.f = f;}
    public T get() {return f;}
}

// 下面的capture of ?、capture of ? extends Fruit、capture of ? super Fruit，都是一个实际的类
// 声明了Plate<? extends Fruit> p1后面的new Plate<Apple>之类的，并没不是说让这个p1的类型确定为Plate<Apple>，
// p1依然是类型Plate<? extends Fruit>，这里是比较迷惑人的
// 只是类型Plate<Apple>和类型Plate<? extends Fruit>是兼容的，或者可以认为Plate<Apple>是Plate<? extends Fruit>的子类，后面的几个例子也是类似

Plate<? extends Fruit> p1 = new Plate<Apple>(new Apple()); // 构造函数的参数必须是Apple类，因为new Plate<Apple>声明了Apple
p1.set(new Fruit()); // 报错，required type：capture of ? extends Fruit. provided：Fruit.
p1.set(new Apple()); // 报错，和上面差不多
p1.set(null);
// 两个set方法报错，是因为capture of ? extends Fruit实际上是一个确定的类，只是不知道它是Apple还是Orange，
// 假如是Orange，自然是不能用Apple赋值的

Object a0 = p1.get();
Food a1 = p1.get();
Fruit a2 = p1.get();
Apple a3 = p1.get(); // 报错，required type：Apple. provided：capture of ? extends Fruit.
// 这里理由同上，万一里面是个Orange呢？事实上这里能get到的最具体的类只能到Fruit，毕竟无法确定具体是哪种水果，只知道是水果。

Plate<? super Fruit> p2 = new Plate<Food>(new Food());
p2.set(new Object()); // 报错，required type：capture of ? super Fruit. provided：Object.
p2.set(new Food()); // 报错，和上面差不多
p2.set(new Fruit());
p2.set(new Apple());
p2.set(null);
// 因为这里都是被转成Fruit存进去的(能转成Fruit自然可以转成Fruit的父类)，Apple可以安全转成Fruit，但是Object和Food不行
// 记住这里的界是包括Fruit自己的，这就是为什么不能set个Food进去，因为万一Plate里面是Fruit呢

Object a4 = p2.get();
Food a5 = p2.get();
Fruit a6 = p2.get(); // 报错，required type：Fruit. provided：capture of ? super Fruit.
Apple a7 = p2.get(); // 报错，和上面差不多
// 这里只能取出Object，因为Fruit的父类可能性太多，只有全部类的父类Object才能兼容全部

Plate<?> p3 = new Plate<Apple>(new Apple());
p3.set(new Object()); // 报错，required type：capture of ?. provided：Object.
p3.set(new Fruit()); // 报错，和上面差不多
p3.set(new Apple()); // 报错，和上面差不多
p3.set(null);

Object a8 = p3.get();
Food a9 = p3.get();
Fruit a10 = p3.get(); // 报错，required type：Fruit. provided：capture of ?.
Apple a11 = p3.get(); // 报错，和上面差不多

// 无论上界还是下界，都包含自己本身
Plate<? extends Fruit> p4 = new Plate<Fruit>();
Plate<? super Fruit> p5 = new Plate<Fruit>();
```

## 框架或库

### srping-boot
参考列表：

* 比较完整的入门例子：https://github.com/jiwenxing/spring-boot-demo
* MyBatis的原理解析：https://www.jianshu.com/p/ec40a82cae28

### Mybatis
参考列表：

* MyBatis的原理解析：https://www.jianshu.com/p/ec40a82cae28
* 通用Mapper用法：https://mapperhelper.github.io/docs

## 工具

### gradle

#### build.gradle文件配置

##### buildscript
buildscript中的声明是gradle脚本自身需要使用的资源。

参考：https://www.cnblogs.com/qiangxia/p/4826532.html

##### ext
可读写的键值对。整个工程的其他地方都可以访问。

##### springboot相关的插件
1.5和2.0的区别：https://spring.io/blog/2017/04/05/spring-boot-s-new-gradle-plugin