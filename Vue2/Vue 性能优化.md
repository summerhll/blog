#  一. 代码层面的优化
1. v-if 和 v-show 区分使用场景

v-if 适用于在运行时很少改变条件，不需要频繁切换条件的场景；v-show 则适用于需要非常频繁切换条件的场景。

2. computed 和 watch 区分使用场景

3. v-for 遍历必须为 item 添加 key，且避免同时使用 v-if

4. 事件的销毁
销毁使用 addEventListene 注册的事件，如：
```Vue
created() {
  addEventListener('click', this.click, false)
},
beforeDestroy() {
  removeEventListener('click', this.click, false)
}
```

5. 图片资源懒加载

6. 路由懒加载

7. 第三方插件的按需引入
借助 babel-plugin-component 示例：
```js
（1）首先，安装 babel-plugin-component ：
npm install babel-plugin-component -D


（2）然后，将 .babelrc 修改为：
{
  "presets": [["es2015", { "modules": false }]],
  "plugins": [
    [
      "component",
      {
        "libraryName": "element-ui",
        "styleLibraryName": "theme-chalk"
      }
    ]
  ]
}


（3）在 main.js 中引入部分组件：
import Vue from 'vue';
import { Button, Select } from 'element-ui';

 Vue.use(Button)
 Vue.use(Select)
```

8. 避免响应所有数据

不需要响应式的数据我们可以直接定义在实例上。
也可以使用Object.freeze冻结不需要变更的内容。


```Vue
<template>
    <view>

    </view>
</template>

<script>
    export default {
        components: {},
        data: () => ({
            
        }),
        beforeCreate: function(){
            this.timer = null;
        }
    }
</script>

<style scoped>
</style>
```

9. 函数式组件
 只做展示的静态页面可以用函数式组件实现，如“About组件”

10.使用keep-alive组件缓存组件状态，避免反复重渲染导致的性能等问题

11. 非页面一级功能组件拆分按需加载

12. 组件细分
 vue对比新旧vnode是组件级别的，细分组件可以在视图较小变动时提高性能。

13. 优化无限列表性能
  vue-virtual-scroll-list 和 vue-virtual-scroller 

14. 服务端渲染 SSR
 服务端渲染是指 Vue 在客户端将标签渲染成的整个 html 片段的工作在服务端完成，服务端形成的 html 片段直接返回给客户端这个过程就叫做服务端渲染。

（1）优点：
  更好的 SEO、首屏加载更快
（2）服务端渲染的缺点：
 更多的开发条件限制、更多的服务器负载：

# 二. 打包优化

1. SourceMap 线上环境关闭，第一次发布项目时可以开启，便于查错。
2. splitChunksPlugins： 分割公共代码。
3. 减少 ES6 转为 ES5 的冗余代码: babel-plugin-transform-runtime

4. webpack-bundle-analyzer 输出结果分析

# 三. 基础的 Web 技术优化
1. 服务器端开启 gzip 压缩
2. 浏览器缓存
3. CDN 的使用(静态资源)
4. 使用 Chrome Performance 查找性能瓶颈
