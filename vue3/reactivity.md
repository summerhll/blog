```javascript
// 判断是否是对象
function isObject(val) {
  return typeof val === 'object' && val !== null
}

// 已有代理的存储
let toProxy = new WeakMap();


// 当前激活的effect栈
let activeEffectstack = []

// 依赖映射表
/** 
 * {
 *    target : {
 *       key : [effect1, effect2]
 *    }
 
  } 
 
 */
let targetMap = new Map();
function reactive(target) {
  return createReactiveObj(target)
}

const baseHandler = {
  get(target, key) {
    const ret = Reflect.get(target, key);
    track(target, key); //收集依赖
    return ret;
  },
  set(target, key, val) {
    Reflect.set(target, key, val);
    trigger(target, key)

  }
}

function createReactiveObj(target) {
  // 仅接受Object对象
  if (!isObject(target)) {
    return target;
  }
  // 检查是否已代理过该对象
  let proxy = toProxy.get(target);

  if (proxy) {
    return proxy
  }


  // 使用proxy代理
  let observed = new Proxy(target, baseHandler);
  // 维护proxy表
  toProxy.set(target, observed);
  return observed;
}
function track(target, key) {
  // 当前激活栈的栈顶就是当前激活的effect
  const effect = activeEffectstack[activeEffectstack.length - 1];
  if (effect) {
    // 维护依赖表
    let depsMap = targetMap.get(target);
    if (!depsMap) {
      targetMap.set(target, (depsMap = new Map()))
    }
    let deps = depsMap.get(key)

    if (!deps) {
      depsMap.set(key, (deps = new Set()))
    }

    if (!deps.has(effect)) {
      deps.add(effect)
    }
  }
}
function trigger(target, key) {
  // 从依赖表中获取effect并逐个执行
  let depsMap = targetMap.get(target)

  if (depsMap) {
    let deps = depsMap.get(key)
    if (deps) {
      deps.forEach((effect) => {
        effect();
      })
    }
  }
}

// 依赖函数
function effect(fn) {
  const effectF = function () {
    try {
      // 加入effect栈
      activeEffectstack.push(effectF)
      fn()
    } finally {
      console.log(111);
      activeEffectstack.pop();
    }
  }
  // 创建effect就会触发一次
  effectF();
  return effectF
}

const ref = raw => {
  const r = {
    get value() {
      track(r, 'value');
      return raw;
    },

    set value(newVal) {
      raw = newVal;
      trigger(r, 'value');
    }
  }
  return r;
}

const computed = getter => {
  let result = ref();
  effect(() => result.value = getter());
  return result;
}
```
