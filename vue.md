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

## 虚拟dom、diff、渲染
参考：
* https://www.zhihu.com/question/31809713
* https://juejin.im/post/5d085ce85188255e1305cda1
* https://github.com/aooy/blog/issues/2
* https://github.com/answershuto/learnVue/blob/master/docs/VirtualDOM%E4%B8%8Ediff(Vue%E5%AE%9E%E7%8E%B0).MarkDown
* https://juejin.im/post/5affd01551882542c83301da

概括：
* dom操作不是纯js层面的操作。而virtual dom操作是纯js操作，而且也没有真正的dom那么复杂，它是比真正的dom操作快的。
* diff的时候，是4个指针，从新旧节点序列从两端各自往中间遍历，互相比较，如果有相同就复用。这种比较是偷懒的，并非把新旧节点列表里的节点都一一对比。
* 如果有key的话，除了只比较这4个指针指向的节点，新节点还可以利用key，从hashmap里找key相同的旧节点进行比较，提高复用率。

## render函数
```js
// 这个属性在组件定义里作为template的代替
render: function (createElement) {
  return createElement('h1', this.blogTitle)
}

// 经常看到这种形式，这里的App是.vue文件导出的一个Object，也就是一个组件对象
render: h => h(App)
```

这个createElement返回的是一个VNode对象。

## 插槽(slot)
* 2.6.0之后，v-slot只能添加在`<template>`上
* 可以在slot中倒出子组件属性，让父组件访问
* 2.6.0后slot和slot-scope语法被废弃。2.x版本内仍可用，详见下面例子

```html
<!DOCTYPE html>
<html lang="zh">
<head>
    <meta charset="UTF-8" />
    <title>vue slot test</title>
</head>
<template id="childTemplate">
    <div>
      <div>
        <slot name="slot_1">具名slot1</slot>
      </div>
      <div>
        <slot>默认slot</slot>
      </div>
      <div>
        <slot name="slot_2">具名slot2</slot>
      </div>
      <div>
        <!-- 这里导出了user这个属性，user是子组件的属性名，outuser是导出时的别名 -->
        <slot name="slot_3" :outuser="user">
          {{ user.lastName }}
        </slot>
      </div>
      <div>
        <slot name="slot_4" :outuser="user">
            {{ user.lastName }}
        </slot>
      </div>
    </div>
</template>
<script src="vue.js"></script>
<body>
  <div id="app">
    <child>
      插入默认slot的内容
      <template v-slot:slot_1>插入具名slot1的内容</template>
      这段也会被插带默认slot
      <div>
        也在默认slot
      </div>
      <template v-slot:slot_2>插入具名slot2的内容</template>
      <div>
        所以用的时候，插入到默认slot最好还是写在一起，避免迷惑。因为在组件里他们是在一起的。
      </div>
      <template v-slot:slot_3="slotProps">
        <!-- 通过slotProps访问子组件的user，用的是导出时的别名outuser -->
        <div>{{ slotProps.outuser.firstName }}</div>
      </template>
      <!-- 下面是2.6.0之后被废弃的语法 -->
      <template slot="slot_4" slot-scope="slotProps">
        <div>{{ slotProps.outuser.firstName }}</div>
      </template>
    </child>
  </div>
<script>
Vue.component('child', {
  data() {
    return {
      user: {
        firstName: 'boating',
        lastName: 'zeng'
      }
    }
  },
  template: '#childTemplate'
});

new Vue({
  el: '#app',
  data(){
    return {
    }
  },
})
</script>
</body>
</html>
```

## Prop
注意，用prop给子组件的data赋初值后，改变prop，子组件里被赋值的那个data不会跟着变。下面的例子里，点击改变父message按钮后，fromParent会变new message，但是子组件里的localMessage不会变。另外，关于sync修饰符(语法糖)，可以通过`this.$emit('update:fromParent', 'child change parent');`在子组件里改变父组件的fromParent。
```html
<template id="childTemplate">
  <div>
    <div>fromParent：{{fromParent}}</div>
    <div>localMessage：{{localMessage}}</div>
    <button @click="testSync">子改父</button>
  </div>
</template>

<div id="app">
  <div>message：{{message}}</div>
  <div>子</div>
  <child :from-parent="message"></child>
  <button @click="test">改变父message</button>
</div>

<script>
let Child = Vue.component('child', {
  props: ['fromParent'],
  data() {
    return {
      localMessage: this.fromParent
    }
  },
  methods: {
    testSync() {
      this.$emit('update:fromParent', 'child change parent');
    }
  },
  template: '#childTemplate'
})

new Vue({
  el: '#app',
  data(){
    return {
      message: 'old message'
    }
  },
  methods: {
    test() {
      this.message = 'new message';
    }
  },
})
</script>
```

## 组件的name作用
https://www.jb51.net/article/140702.htm

1. keep-alive要求被切换到的组件都有自己的名字，不论是通过组件的name选项还是局部/全局注册
2. 组件递归时通过name调用
3. 用vue-tools时显示组件名字

## 给data里添加属性
给data添加原本没有的属性的方法。注意，根属性是不可以再添加的，实际上，访问一个原本不存在的根属性，本身就会报warning，而且原则上，也没有要这样做的理由。
```js
data(){
  return {
      a1: {} // 原本a1里没有b1属性
  }
},
methods: {
  test1() {
      this.a1.b1 = 'a1.b1 test1'; // 无效
  },
  test2() {
      this.a1 = {b1: 'a1.b1 test2'}; // 有效，直接替换a1
  },
  test3() {
      this.$set(this.a1, 'b1', 'a1.b1 test3'); // 有效
  }
}
```

## v-model
```html
<!DOCTYPE html>
<html lang="zh">
  <head>
    <meta charset="UTF-8" />
    <title>vue v-model test</title>
    <template id="appTemplate">
        <div>
            <div>a2(根属性)：{{a2}}</div>
            <div>a1.b2(原本不存在)：{{a1.b2}}</div>
            <div>a1.b3(原本存在)：{{a1.b3}}</div>
            a2(v-model)：<input v-model="a2">
            a2(自定)：<input :value="a2" @input="a2 = $event.target.value">
            a1.b2(v-model)：<input v-model="a1.b2">
            a1.b2(自定)：<input :value="a1.b2" @input="$set(a1, 'b2', $event.target.value)">
            a1.b3(v-model)：<input v-model="a1.b3">
            a1.b3(自定)：<input :value="a1.b3" @input="a1.b3 = $event.target.value">
        </div>
    </template>
    <script src="vue.js"></script>
  </head>
  <body>
    <div id="app"></div>
    <script>
    new Vue({
      el: '#app',
      data(){
        return {
            a1: {b3: ''},
            a2: ''
        }
      },
      template: '#appTemplate'
    })
    </script>
  </body>
</html>
```

上面例子里，`a1.b2`和`a1.b3`的两个自定input，都可以正常运作，但实际上，vue的v-model生成的是`$set(a1, 'b2', $event.target.value)`这种形式(具体可以查看源码的`genAssignmentCode`函数)。这里之所以可以正常运作，是因为`a1.b3`这个属性是一开始已经定义的，所以直接用`=`赋值没有问题。

参考(深入的部分，要理解vue的编译才能看懂)：https://www.jianshu.com/p/8e2b5e04a1f7

## vue的编译

### 前置知识
* js里，可以通过`new Function()`从字符串中创建一个函数，这意味着可以创建源码中未定义的函数。
* 注意`with`关键字的用法，在生成的render函数里用`with(this)`，把函数内的作用域绑到对象本身，所以函数内访问对象的各种属性都不用加`this.`。
* vue里，模板里的字符串，最终是被创建成render函数的。
* AST(抽象语法树)。

### 参考
* https://segmentfault.com/a/1190000012922342

### 举例说明
因为这个话题涉及的东西比较多，先从一些简单例子开始。

**例1**
```html
<template id="appTemplate">
    <div>
        <button @click="console.log(1)">test</button>
    </div>
</template>
```
1. 上面这个模板，被编译成怎样的render函数？
2. `@click`里的这个`console.log(1)`怎样最终变成这个button的click的listener的？

下面的过程边对源码断点边观察。

找到源码里`createCompileToFunctionFn`函数
```js
function createCompileToFunctionFn (compile) {
    var cache = Object.create(null);

    return function compileToFunctions (
      template,
      options,
      vm
    ) {
      options = extend({}, options);
      var warn$$1 = options.warn || warn;
      delete options.warn;
// 省略很多代码
      // compile
      var compiled = compile(template, options);
// 省略很多代码
      res.render = createFunction(compiled.render, fnGenErrors);
// 省略很多代码
    }
}
```

关注里面的`compileToFunctions`这个函数，在这个函数里断点，观察`template`参数，在这个例子里，`template`的值就是上面的模板(不含template标签)字符串。

断点观察`compiled`的值(这里先不讨论编译过程)，它有一个`render`属性，是个字符串，值如下：
```js
`with(this){return _c('div',[_c('button',{on:{"click":function($event){return console.log(1)}}},[_v("test")])])}`
```
它是个函数体，而且就是render函数的函数体，另外可以看到我们定义的click的listener也在里面了。从前面的参考文档得知，里面几个函数的含义如下：

* _c：对应的是`createElement`方法，顾名思义，它的含义是创建一个元素(Vnode)
* _v：创建一个文本结点。
* _s：把一个值转换为字符串。(eg: {{data}})
* _m：渲染静态内容

上面的这个`render`字符串会被传到`createFunction`里，通过`new Function()`创建一个render函数，这就是上面定义的模板生成的渲染函数。可以通过debugger查看到这个函数的内容，会在(伪)文件VMxxxx(xxxx是数字)里，函数如下：
```js
(function anonymous(
) {
with(this){return _c('div',[_c('button',{on:{"click":function($event){return console.log(1)}}},[_v("test")])])}
})
```

问题1结束，接下来是问题2。

找到`updateDOMListeners`函数
```js
function updateDOMListeners (oldVnode, vnode) {
    if (isUndef(oldVnode.data.on) && isUndef(vnode.data.on)) {
      return
    }
    var on = vnode.data.on || {};
    var oldOn = oldVnode.data.on || {};
    target$1 = vnode.elm; // 这个target$1是个在外层定义的临时变量，指向这个等着添加listener的元素
    normalizeEvents(on);
    updateListeners(on, oldOn, add$1, remove$2, createOnceHandler$1, vnode.context);
    target$1 = undefined;
}
```

注意这里的`vnode`参数，它就是`createElement`的返回值，断点关注它的两个属性。一个是`elm`，它就是`<button>`这个元素，可以看到此时它的`onclick`属性还是`null`。另一个是`data.on`，它是个函数，而且就是上面提及的VMxxxx里的代码里的click属性所指的函数。

找到`updateListeners`函数
```js
function updateListeners (
  on, // {"click": fn} fn就是原本定义的那个listener函数
  oldOn,
  add,
  remove$$1,
  createOnceHandler,
  vm
) {
// 省略很多
      def$$1 = cur = on[name]; // 从on里取了click属性所指的函数
// 省略很多
      cur = on[name] = createFnInvoker(cur, vm); // 这里把原本定义的listener封装到一个统一函数里
// 省略很多
      add(event.name, cur, event.capture, event.passive, event.params); // add函数把上面这个函数作为listener添加到目标元素
// 省略很多
}
```

下面这个`add$1`函数，就是上面的`add`函数的定义。注意对齐参数。
```js
function add$1 (
  name,
  handler,
  capture,
  passive
) {
  if (useMicrotaskFix) {
    var attachedTimestamp = currentFlushTimestamp;
    var original = handler;
    // 这里被封了一层，如果useMicrotaskFix，最终用的就是这个函数
    // 所以查看元素的Event Listeners的时候，指向的是这个函数
    handler = original._wrapper = function (e) {
      if (
        e.target === e.currentTarget ||
        e.timeStamp >= attachedTimestamp ||
        e.timeStamp <= 0 ||
        e.target.ownerDocument !== document
      ) {
        return original.apply(this, arguments)
      }
    };
  }
  // 这里就真正把listener注册到dom了，要记得这个target$1，在前面是被赋值成目标元素的
  target$1.addEventListener(
    name,
    handler,
    supportsPassive
      ? { capture: capture, passive: passive }
      : capture
  );
}
```

## vue-cli

### babel的配置

#### babel.config.js
默认使用`@vue/babel-preset-app`。不过不同版本的vue-cli配置写法不同。3版本：`@vue/app`；4版本：`@vue/cli-plugin-babel/preset`。

#### browserslist
因为用了`@babel/preset-env`，所以通过`browserslist`选项配置。vue-cli默认的`browserslist`是`> 1%, last 2 versions`。也就是支持使用率超过1%的浏览器，并且支持所有浏览器的最新两个版本。

## vue-router

### 动态去除路由
vue-router(现在版本3.1.3，现在时间2020-02-05)有动态添加路由的方法`addRoutes`，但是没有动态删除路由的方法，只能替换(matcher)：https://github.com/vuejs/vue-router/issues/1234#issuecomment-357941465

### 怎样监听URL变化
在地址栏里按回车，浏览器怎么知道不需要往服务器发请求？

* hash mode的话，可以监听hashchange事件，毕竟只是改变hash的话，浏览器本身就不会发请求
* history mode的话，在地址栏里按回车，是怎么都会发请求的，即使是同一个url。但是监听popstate事件，可以在点击回退或前进按钮时，避免重新发请求。

参考：
* https://zhuanlan.zhihu.com/p/27588422

## 杂项

### vue-devtools是怎么检测vue的
找到vue-devtools中的detector.js，可以发现有两种方法。

1. 检测Nuxt.js(通过`window.__NUXT__ || window.$nuxt`)
2. 扫描全部页面元素，如果有元素包含`__vue__`属性，则使用了vue

### 使用scoped时注意事项

#### 子组件的根节点问题
子组件的根节点的`data-v-xxxxxx`，会被父组件影响，也就是说它同时有了子组件自己的`data-v-xxxxxx`属性和父组件的`data-v-xxxxxxx`属性。这个特性是特意设计的，可以利用它来在父组件来控制子组件的样式。同时要注意这个特性，以免出现意图之外的情况。

#### 使用deep的情况
```html
<style scoped>
.a >>> .b { /* ... */ }
</style>
<!-- 上面使用了deep，会生成如下css -->
<style>
.a[data-v-f3f3eg9] .b { /* ... */ }
</style>
<!-- 如果不用deep的话，生成的css应该如下 -->
<style>
.a .b[data-v-f3f3eg9] { /* ... */ }
</style>
```

## 面试题中关于vue的

### 写React/Vue项目时为什么要在列表组件中写key，其作用是什么？
https://github.com/Advanced-Frontend/Daily-Interview-Question/issues/1

* key是用于识别节点的
* 有key的话，如果节点内容没有变化，可以复用节点，比如只是顺序改变了
* diff的时候，如果有key，就可以直接根据key去map，而不需要遍历
* 有key的话，如果节点换了(key不同就是换了)，那会替换整个节点
* 如果没key，节点修改只需修改节点里的内容，节点本身可以复用，但是会有副作用，比如可能不会产生过渡效果，或者在某些节点有绑定数据（表单）状态，会出现状态错位。所以除非列表是很简单的内容，否则尽可能提供key。文档里说，(不用key的默认模式)只适用于不依赖子组件状态或临时DOM状态(例如：表单输入值)的列表渲染输出。

### Redux和Vuex的设计思想
https://github.com/Advanced-Frontend/Daily-Interview-Question/issues/45

* 主要是用来处理一些全局状态
* 有规范的api去操作这些状态

#### 为什么Vuex的mutation和Redux的reducer中不能做异步操作
https://github.com/Advanced-Frontend/Daily-Interview-Question/issues/65

官方文档解释：https://vuex.vuejs.org/zh/guide/mutations.html#mutation-%E5%BF%85%E9%A1%BB%E6%98%AF%E5%90%8C%E6%AD%A5%E5%87%BD%E6%95%B0

这里的解释意思看上去就是为了方便debug。感觉主要是一种约定，另外就是模仿Redux。

### Vue 的双向数据绑定
https://github.com/Advanced-Frontend/Daily-Interview-Question/issues/34

依赖收集的说明
https://zhuanlan.zhihu.com/p/29318017

* 在计算属性第一次被读取时，把依赖关系(所需执行的函数)添加到它依赖的属性里，依赖只需添加一次
* 之后依赖的属性被修改时，就会触发这个依赖关系(所需执行的函数)

```js
/**
 * 定义一个“依赖收集器”
 */
const Dep = {
  target: null
}

/**
 * 使一个对象转化成可观测对象
 * @param { Object } obj 对象
 * @param { String } key 对象的key
 * @param { Any } val 对象的某个key的值
 */
function defineReactive (obj, key, val) {
  const deps = []
  Object.defineProperty(obj, key, {
    get () {
      console.log(`我的${key}属性被读取了！`)
      if (Dep.target && deps.indexOf(Dep.target) === -1) {
        // 依赖只需收集一次，这里是在第一次读取计算属性时收集
        deps.push(Dep.target)
      }
      return val
    },
    set (newVal) {
      console.log(`我的${key}属性被修改了！`)
      val = newVal
      deps.forEach((dep) => {
        dep()
      })
    }
  })
}

/**
 * 把一个对象的每一项都转化成可观测对象
 * @param { Object } obj 对象
 */
function observable (obj) {
  const keys = Object.keys(obj)
  for (let i = 0; i < keys.length; i++) {
    defineReactive(obj, keys[i], obj[keys[i]])
  }
  return obj
}

/**
 * 当计算属性的值被更新时调用
 * @param { String } key 计算属性的键
 * @param { Any } val 计算属性的值
 */
function onComputedUpdate (key, val) {
  console.log(`计算属性改变：(${key}:${val})`)
}

/**
 * 观测者
 * @param { Object } obj 被观测对象
 * @param { String } key 被观测对象的key
 * @param { Function } cb 回调函数，返回“计算属性”的值
 */
function watcher (obj, key, cb) {
  // 定义一个被动触发函数，当这个“被观测对象”的依赖更新时调用
  const onDepUpdated = () => {
    const val = cb()
    onComputedUpdate(key, val)
  }

  Object.defineProperty(obj, key, {
    get () {
      Dep.target = onDepUpdated
      // 这里是重点！！！！！
      // cb里是用到依赖的属性的，所以会触发依赖属性的getter，因此可以把依赖添加到依赖属性里
      // 执行cb()的过程中会用到Dep.target，target会被添加到依赖属性维护的deps数组里
      // 当cb()执行完了就重置Dep.target为null，因为依赖添加一次就好了
      const val = cb()
      Dep.target = null
      return val
    },
    set () {
      console.error('计算属性无法被赋值！')
    }
  })
}

const hero = observable({
  health: 3000,
  IQ: 150
})

watcher(hero, 'type', () => {
  return hero.health > 4000 ? '坦克' : '脆皮'
})

console.log(`英雄初始类型：${hero.type}`)
hero.health = 5000
```

面向对象版
```js
class Dep {
  constructor () {
    this.deps = []
  }

  static target = null;

  depend () {
    if (Dep.target && this.deps.indexOf(Dep.target) === -1) {
      this.deps.push(Dep.target)
    }
  }

  notify () {
    this.deps.forEach((dep) => {
      dep()
    })
  }
}

class Observable {
  constructor (obj) {
    return this.walk(obj)
  }

  walk (obj) {
    const keys = Object.keys(obj)
    keys.forEach((key) => {
      this.defineReactive(obj, key, obj[key])
    })
    return obj
  }

  defineReactive (obj, key, val) {
    const dep = new Dep()
    Object.defineProperty(obj, key, {
      get () {
        dep.depend()
        return val
      },
      set (newVal) {
        val = newVal
        dep.notify()
      }
    })
  }
}

class Watcher {
  constructor (obj, key, cb, onComputedUpdate) {
    this.obj = obj
    this.key = key
    this.cb = cb
    this.onComputedUpdate = onComputedUpdate
    return this.defineComputed()
  }

  defineComputed () {
    const self = this
    const onDepUpdated = () => {
      const val = self.cb()
      this.onComputedUpdate(val)
    }

    Object.defineProperty(self.obj, self.key, {
      get () {
        Dep.target = onDepUpdated
        const val = self.cb()
        Dep.target = null
        return val
      },
      set () {
        console.error('计算属性无法被赋值！')
      }
    })
  }
}

const hero = new Observable({
  health: 3000,
  IQ: 150
})

new Watcher(hero, 'type', () => {
  return hero.health > 4000 ? '坦克' : '脆皮'
}, (val) => {
  console.log(`我的类型是：${val}`)
})

console.log(`英雄初始类型：${hero.type}`)
hero.health = 5000
```

### 在Vue中，子组件为何不可以修改父组件传递的Prop，如果修改了，Vue是如何监控到属性的修改并给出警告的
https://github.com/Advanced-Frontend/Daily-Interview-Question/issues/60

官方文档的说法：父级prop的更新会向下流动到子组件中，但是反过来则不行。这样会防止从子组件意外改变父级组件的状态，从而导致你的应用的数据流向难以理解。额外的，每次父级组件发生更新时，子组件中所有的prop都将会刷新为最新的值。这意味着你不应该在一个子组件内部改变prop。

监控是在prop的setter里监控的。如果不是根组件，而且不是更新子组件，这次setter调用就会触发警告。

```html
<!DOCTYPE html>
<html lang="zh">
<head>
    <meta charset="UTF-8" />
    <title>vue prop test</title>
</head>
<template id="childTemplate">
  <div>
    <div>{{count}}</div>
    <div>{{fromParent}}</div>
    <button @click="test">子</button>
  </div>
</template>
<script src="vue.js"></script>
<body>
  <div id="app">
    {{message}}{{rootprop}}
    <child :from-parent=message></child>
    <button @click="test">父</button>
  </div>
  <script>
    Vue.component('child', {
      props: ['fromParent'],
      data() {
        return {
          count: 1
        }
      },
      methods: {
        test() {
          console.log(this.fromParent);
          this.fromParent = 'change by child'; // 子组件的fromParent被改变，但是父组件的message不会被改变，并且抛出warn
        }
      },
      template: '#childTemplate'
    })

    new Vue({
      props: ['rootprop'], // 根组件也可以有props的，通过propsData传入
      el: '#app',
      data(){
        return {
          message: 'Hello Vue!',
        }
      },
      methods: {
        test() {
          this.message = 'change by parent';
        }
      },
      propsData: {
        rootprop: 'rp'
      }
    })
  </script>
</body>
</html>
```

### Vue的响应式原理中Object.defineProperty有什么缺陷
https://github.com/Advanced-Frontend/Daily-Interview-Question/issues/90

* 对于对象，需要进行深度遍历来进行defineProperty。这个一般不是大问题。
* 对数组的监测能力有限，最明显的就是无法监测数组下标的变化。需要通过几个特定函数来监听数组变化。
* 由于js里数组也是对象，数组下标其实也是对象属性，事实上defineProperty是可以对数组下标设置getter和setter的。但是数组下标也是对象属性，这本来就是js的一个奇怪特性(哈哈，数组下标也是key？)，不应该利用，就算用，性能也是有问题。
* 如果用proxy的话，不需要对每个属性进行setter和getter的设置，而是对象整体监听。不过Prxoy这个特性是es6的，而且无法polyfill。

### vue在v-for时给每项元素绑定事件需要用事件代理吗？为什么？
https://github.com/Advanced-Frontend/Daily-Interview-Question/issues/145

* vue没有做事件代理。
* 事件代理的一个好处是方便添加和去除事件，但是vue帮你做了(添加和去除事件这个操作)。
* 关于性能，列表里的不同listener实际上用的是同一个函数。除非列表真的很大(或者客户端性能很差)，否则不需要事件代理。

