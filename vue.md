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

### Vue 的双向数据绑定
https://github.com/Advanced-Frontend/Daily-Interview-Question/issues/34

依赖收集的说明
https://zhuanlan.zhihu.com/p/29318017

* 在计算属性第一次被读取时，把依赖关系(所需执行的函数)添加到它依赖的属性里，依赖只需添加一次
* 之后被依赖的属性被修改时，就会触发这个依赖关系(所需执行的函数)

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
      // 执行cb()的过程中会用到Dep.target，
      // 当cb()执行完了就重置Dep.target为null
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