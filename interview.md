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