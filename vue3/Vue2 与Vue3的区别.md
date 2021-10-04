# 一. 创建VUE应用实例
```javascript
//Vue3示例
//html
<div id="counter">
  Counter: {{ counter }}
</div>

//js
const Counter = {
  data() {
    return {
      counter: 0
    }
  }
}

Vue.createApp(Counter).mount('#counter')

//Vue2示例
import Vue from "vue";
import App from './App.vue'

new Vue({
  render: (h) => h(App)
}).$mount("#app");
```

# 二.生命周期
```javascript
Vue3的生命周期       Vue2的生命周期 .    组合式API
beforeCreate       beforeCreate       setup()
Created            Created .          setup()
beforeMount .      beforeMount        onBeforeMount
mounted .          mounted            onMounted
beforeUnMount .    beforeDestroy      onBeforeUnmount    
unmounted .        destroyed          onUnmounted
beforeUpdate       beforeUpdate .    onBeforeUpdate
updated            updated           onUpdated
activated .       activated         onActivated
deactivated       deactivated       onDeactivated
                                    onErrorCaptured
                                    onRenderTracked
                                    onRenderTriggered

```
# 三. v-if和v-for
 ```
 不推荐同时使用 v-if 和 v-for。
Vue3中当 v-if 与 v-for 一起使用时，v-if 具有比 v-for 更高的优先级。
Vue2中当 v-if 与 v-for 一起使用时，v-for 具有比 v-if 更高的优先级。
 ```
# 四.v-model 自定义组件
```
app.component('custom-input', {
  props: ['modelValue'],
  emits: ['update:modelValue'],
  template: `
    <input
      :value="modelValue"
      @input="$emit('update:modelValue', $event.target.value)"
    >
  `
})
<custom-input v-model="searchText"></custom-input>


这里v-model的本质为：
<custom-input
  :model-value="searchText"
  @update:model-value="searchText = $event"
></custom-input>
```
默认情况下，组件上的 v-model 使用 modelValue 作为 prop 和 update:modelValue 作为事件。我们可以通过向 v-model 传递参数来修改这些名称：
```
app.component('my-component', {
  props: {
    title: String
  },
  emits: ['update:title'],
  template: `
    <input
      type="text"
      :value="title"
      @input="$emit('update:title', $event.target.value)">
  `
})

<my-component v-model:title="bookTitle"></my-component>
```

# 五. $attrs
```
$attrs，该 property 包括组件 props 和 emits property 中未包含的所有属性 (例如，class、style、v-on 监听器等)。
$listeners被废弃。

$attrs包含了父作用域中不作为组件 props 或自定义事件的 attribute 绑定和事件。当一个组件没有声明任何 prop 时，这里会包
含所有父作用域的绑定，并且可以通过 v-bind="$attrs" 传入内部组件——这在创建高阶的组件时会非常有用。
```

# 六. 指令
```
1.指令钩子函数：

created：在绑定元素的 attribute 或事件监听器被应用之前调用。
beforeMount：当指令第一次绑定到元素并且在挂载父组件之前调用。
mounted：在绑定元素的父组件被挂载后调用。
beforeUpdate：在更新包含组件的 VNode 之前调用。
updated：在包含组件的 VNode 及其子组件的 VNode 更新后调用。
beforeUnmount：在卸载绑定元素的父组件之前调用
unmounted：当指令与元素解除绑定且父组件已卸载时，只调用一次。


2.指令钩子传递了这些参数：

el
指令绑定到的元素。这可用于直接操作 DOM。

binding
包含以下 property 的对象。
instance：使用指令的组件实例。
value：传递给指令的值。例如，在 v-my-directive="1 + 1" 中，该值为 2。
oldValue：先前的值，仅在 beforeUpdate 和 updated 中可用。值是否已更改都可用。
arg：参数传递给指令 (如果有)。例如在 v-my-directive:foo 中，arg 为 "foo"。
modifiers：包含修饰符 (如果有) 的对象。例如在 v-my-directive.foo.bar 中，修饰符对象为 {foo: true，bar: true}。
dir：一个对象，在注册指令时作为参数传递。例如，在以下指令中
        app.directive('focus', {
          mounted(el) {
            el.focus()
          }
        })
        dir 将会是以下对象：
        {
          mounted(el) {
            el.focus()
          }
        }
        
vnode
上面作为 el 参数收到的真实 DOM 元素的蓝图。
prevNode
上一个虚拟节点，仅在 beforeUpdate 和 updated 钩子中可用。
```

# 七. 全局 Vue API 已更改为使用应用程序实例
```
Vue3
app.component
app.mixin
app.directive
app.config.optionMergeStrategies.customOption
app.config.globalProperties
app.config.devtools
app.config.errorHandler
app.config.warnHandler
app.config.isCustomElement
app.config.performance

Vue2
Vue.component
Vue.mixin
Vue.directive
Vue.config.optionMergeStrategies.customOption
Vue.prototype
Vue.config.devtools
Vue.config.errorHandler
Vue.config.warnHandler 
Vue.config.ignoredElements
Vue.config.performance
```

# 八 .废弃
```
vm.$children
vm.$listeners

Vue.filters
Vue.extend

$on，$off 和 $once
全局函数 set 和 delete 以及实例方法 $set 和 $delete
```
# 九.VNode 生命周期事件
```
//Vue2 
<template>
  <child-component @hook:updated="onUpdated">
</template>

//Vue3
<template>
  <child-component @vnode-updated="onUpdated">
</template>
或者
<template>
  <child-component @vnodeUpdated="onUpdated">
</template>
```

# 十. Vue 3 现在正式支持了多根节点的组件，也就是片段！
```
//Vue2
<!-- Layout.vue -->
<template>
  <div>
    <header>...</header>
    <main>...</main>
    <footer>...</footer>
  </div>
</template>

//Vue3
<!-- Layout.vue -->
<template>
  <header>...</header>
  <main v-bind="$attrs">...</main>
  <footer>...</footer>
</template>
```

# 十一.其他
```
data 选项应始终被声明为一个函数.
来自 mixin 的 data 选项现在为浅合并.
$attrs 包含传递给组件的所有 attribute，包括 class 和 style。
$listeners 被移除或整合到 $attrs.

v-on 的 .native 修饰符已被移除。同时，新增的 emits 选项允许子组件定义真正会被触发的事件。
因此，对于子组件中未被定义为组件触发的所有事件监听器，Vue 现在将把它们作为原生事件监听器添加到子组件的根元素中 。

组件上 v-model 用法已更改，替换 v-bind.sync.
v-bind="object" 现在排序敏感.
```

