# 一. Vue2数据响应式
在 Vue2 中，对于Object类型的数据响应式的实现是利用了JS 语言特性 Object.defineProperty()，通过遍历data数据，给每个key定义 getter 和 setter 方法来实现数据响应化。

而对于Array类型的数据，则是通过改写数组的'push', 'pop', 'shift', 'unshift', 'splice','sort', 'reverse' 等原型方法来实现数据响应式。

但是，Vue2 的数据响应式也有其缺点:
对于对象，其响应化过程需要递归遍历对象的每个key，给每个key设置getter/setter 方法，性能消耗较大，当新加或删除一个属性时无法监听。
对于数组的响应化需要额外实现，而对于Map、Set、Class 等数据结构则无法对其进行响应化。

# 二. Vue3数据响应式
在 Vue3 中，使用 ES6 的 Proxy 和 Reflect 来实现数据响应化，能更好的解决Vue2数据响应化过程的问题。Proxy 既可以监听整个Object对象，也可以监听数组。

## 1.嵌套对象的响应式
对嵌套对象进行递归处理
```javascript
// 提取帮助方法
const isObject = val => val !== null && typeof val === 'object'
function reactive(obj) {
  //判断是否对象
  if (!isObject(obj)) {
    return obj
  }
  const observed = new Proxy(obj, {
    get(target, key, receiver) {
      // ...
      // 如果是对象需要递归
      return isObject(res) ? reactive(res) : res
    },
    //...
  })
}
```

## 2. 避免重复代理
将之前代理结果缓存，get时直接使用
```javascript
const toProxy = new WeakMap() // 形如obj:observed
const toRaw = new WeakMap() // 形如observed:obj
function reactive(obj) {
  //...
  // 查找缓存，避免重复代理
  if (toProxy.has(obj)) {
    return toProxy.get(obj)
  }
  if (toRaw.has(obj)) {
    return obj
  }
  const observed = new Proxy(obj, {})
  // 缓存代理结果
  toProxy.set(obj, observed)
  toRaw.set(observed, obj)
  return observed
}
```
## 3.依赖收集
其实现原理是：建立响应数据key和更新函数之间的对应关系

在Vue3中针对所有的被监听的对象，存在一张关系表targetMap,key为target，value为另一张关系表depsMap。
depsMap的key为target的每个key，value为由effect函数传入的参数的Set集。
```
type Dep = Set<ReactiveEffect>
type KeyToDepMap = Map<string | symbol, Dep>
const targetMap = new WeakMap<any, KeyToDepMap>()
// 大概结构如下所示
// target | depsMap
//    obj | key | Dep
//     k1 | effect1,effect2...
//     k2 | effect3,effect4...
// {target: {key: [effect1,...]}}
```
整体的依赖关系如下：
名称	      类型	       key.    	    值
targetMap	 WeakMap	 object	      depsMap
depsMap.   Map	     property	    dep
dep	       Set		                effect

● targetMap: 存放每个响应式对象（所有属性）的依赖项

● targetMap: 存放响应式对象每个属性对应的依赖项

● dep: 存放某个属性对应的所有依赖项（当这个对象对应属性的值发生变化时，这些依赖项函数会重新执行）

```
//用法

// 设置响应函数
effect(() => console.log(state.foo))

// 用户修改关联数据会触发响应函数
state.foo = 'xxx'
```
通过设计三个函数来实现：
* effect：将回调函数保存起来备用，立即执行一次回调函数触发它里面一些响应数据的getter 
* track：getter中调用track，把前面存储的回调函数和当前target,key之间建立映射关系
* trigger：setter中调用trigger，把target,key对应的响应函数都执行一遍

effect、tract、trigger、与代理对象中的 getter、setter 方法之间的关系：

![yuque_diagram](https://user-images.githubusercontent.com/15389512/137591559-9c447e1b-06b0-4263-ab5c-b3dbd630d067.jpg)

## 实现
创建 effect 函数，设置响应函数
```
// 保存当前活动响应函数作为getter和effect之间桥梁
const effectStack = []

// effect任务：执行fn并将其入栈
function effect(fn) {
	const rxEffect = function() {
    // 1.捕获可能的异常
    try {
      // 2.入栈，用于后续依赖收集
      effectStack.push(rxEffect)

      // 3.运行fn，触发依赖收集
      return fn()
    } finally {
      // 4.执行结束，出栈
      effectStack.pop()
    }
  }
  // 默认执行一次响应函数
  rxEffect()
  // 返回响应函数
  return rxEffect
}
```

依赖收集和触发
```
const isObject = val => val !== null && typeof val === 'object'

function reactive(obj) {
  
  if (!isObject(obj)) {
    return obj
  }
  // Proxy相当于在对象外层加拦截
  // http://es6.ruanyifeng.com/#docs/proxy
  const observed = new Proxy(obj, {
    get(target, key, receiver) {
      // Reflect用于执行对象默认操作，更规范、更友好
      // Proxy和Object的方法Reflect都有对应
      // http://es6.ruanyifeng.com/#docs/reflect
      const res = Reflect.get(target, key, receiver)
      console.log(`获取${key}:${res}`)
      // 依赖收集
      track(target, key)
      
      // 如果是对象，则递归处理对象的响应式
      return isObject(res) ? reactive(res) : res
    },
    set(target, key, value, receiver) {
      const res = Reflect.set(target, key, value, receiver)
      console.log(`设置${key}:${value}`)
      
      // 触发响应函数
      trigger(target, key)
      return res
    },
    deleteProperty(target, key) {
      const res = Reflect.deleteProperty(target, key)
      console.log(`删除${key}:${res}`)
      trigger(target, key)
      return res
    }
  })
  return observed
}

// 依赖收集，建立target，key和上面的effect函数之间映射关系
// 需要一个数据结构存储该关系
// {target: {key: [cb1, cb2, ...]}}
let targetMap = new WeakMap()
function track(target, key) {
  // 从栈中取出响应函数
  const effect = effectStack[effectStack.length - 1]

  if (effect) {
    // 获取target对应Map
    let depsMap = targetMap.get(target)
    if (!depsMap) {
      depsMap = new Map()
      targetMap.set(target, depsMap)
    }
    console.log('depsMap: ', depsMap)
    // 获取depsMap中key和其值，也就是Set
    let deps = depsMap.get(key)
    if (!deps) {
      // 首次deps不存在，创建之
      deps = new Set()
      depsMap.set(key, deps)
    }
    console.log('deps: ', deps)

    // 将响应函数添加到Set里面
    deps.add(effect)
  }
}

//触发 target，key对应的响应函数
function trigger(target, key) {
  // 获取映射关系
  const depsMap = targetMap.get(target)
  if (depsMap) {
    // 获取响应函数集合
    const deps = depsMap.get(key)
    // 执行所有的响应函数
    deps.forEach(effect => {
      effect()
    })
  }
}

```
