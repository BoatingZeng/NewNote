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
https://www.zhihu.com/question/31809713

dom操作不是纯js层面的操作。而virtual dom操作是纯js操作，而且也没有真正的dom那么复杂，它是比真正的dom操作快的。

https://juejin.im/post/5d085ce85188255e1305cda1
https://github.com/aooy/blog/issues/2
https://github.com/answershuto/learnVue/blob/master/docs/VirtualDOM%E4%B8%8Ediff(Vue%E5%AE%9E%E7%8E%B0).MarkDown

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
        <!-- 通过slotProps访问子组件的user，用的时导出时的别名outuser -->
        <div>{{ slotProps.outuser.firstName }}</div>
      </template>
      <!-- 下面是2.6.0之后被废弃的语法 -->
      <template slot="slot_4" slot-scope="slotProps">
        <div>{{ d.outuser.firstName }}</div>
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
注意，用prop给子组件的data赋初值后，改变prop，子组件里被赋值的那个data不会跟着变。下面的例子里，点击按钮后，fromParent会变new message，但是子组件里的localMessage不会变。
```html
<template id="childTemplate">
  <div>
    <div>fromParent：{{fromParent}}</div>
    <div>localMessage：{{localMessage}}</div>
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

优化版
```js
class Dep {
  constructor () {
    this.deps = []
  }

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

Dep.target = null

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
* 事件代理的一个好处是方便添加和去除事件，但是vue帮你做了。
* 关于性能，列表里的不同listener实际上用的是同一个函数。除非列表真的很大(或者客户端性能很差)，否则不需要事件代理。

