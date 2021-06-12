#  一. 代码层面的优化
1.v-if 和 v-show 区分使用场景
v-if 适用于在运行时很少改变条件，不需要频繁切换条件的场景；v-show 则适用于需要非常频繁切换条件的场景。

2.computed 和 watch 区分使用场景
当我们需要进行数值计算，并且依赖于其它数据时，应该使用 computed，因为可以利用 computed 的缓存特性，避免每次获取值时，都要重新计算；
当我们需要在数据变化时执行异步或开销较大的操作时，应该使用 watch

3.v-for 遍历必须为 item 添加 key，且避免同时使用 v-if

4.事件的销毁
Vue 组件销毁时，会自动清理它与其它实例的连接，解绑它的全部指令及事件监听器，但是仅限于组件本身的事件。 如果在 js 内使用 addEventListene 等方式是不会自动销毁的，我们需要在组件销毁时手动移除这些事件的监听，以免造成内存泄露，如：
created() {
  addEventListener('click', this.click, false)
},
beforeDestroy() {
  removeEventListener('click', this.click, false)
}

5.图片资源懒加载

6.路由懒加载

7.第三方插件的按需引入
借助 babel-plugin-component 示例：

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


8. 避免响应所有数据
不需要响应式的数据我们可以直接定义在实例上。
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


也可以使用Object.freeze冻结不需要变更的内容。
Object.freeze冻结data中对应的对象之后，vue不会将对应数据转化成观察者，可以提高渲染速度

9. 函数式组件
可以


10.使用keep-alive组件
当在组件之间切换的时候，有时会想保持这些组件的状态，以避免反复重渲染导致的性能等问题，使用<keep-alive>包裹动态组件时，会缓存不活动的组件实例，而不是销毁它们。重新创建动态组件的行为通常是非常有用的，但是在有些情况下我们更希望那些标签的组件实例能够被在它们第一次被创建的时候缓存下来，此时使用<keep-alive>包裹组件即可缓存当前组件实例，将组件缓存到内存，用于保留组件状态或避免重新渲染，和<transition>相似它，其自身不会渲染一个DOM元素，也不会出现在组件的父组件链中。
<keep-alive>
    <component v-bind:is="currentComponent" class="tab"></component>
</keep-alive>


11. 非页面一级功能组件拆分按需加载
在刚打开页面时有些功能是不需要的，比如点击后的弹窗；可以将这些功能做拆分，结合vue的异步组件与webpack的代码分割做按需加载。

12. 组件细分
vue对比新旧vnode是组件级别的，细分组件可以在视图较小变动时提高性能。

13. 优化无限列表性能
 vue-virtual-scroll-list 和 vue-virtual-scroller 

14. 服务端渲染 SSR
服务端渲染是指 Vue 在客户端将标签渲染成的整个 html 片段的工作在服务端完成，服务端形成的 html 片段直接返回给客户端这个过程就叫做服务端渲染。
（1）服务端渲染的优点：
更好的 SEO： 因为 SPA 页面的内容是通过 Ajax 获取，而搜索引擎爬取工具并不会等待 Ajax 异步完成后再抓取页面内容，所以在 SPA 中是抓取不到页面通过 Ajax 获取到的内容；而 SSR 是直接由服务端返回已经渲染好的页面（数据已经包含在页面中），所以搜索引擎爬取工具可以抓取渲染好的页面；
更快的内容到达时间（首屏加载更快）： SPA 会等待所有 Vue 编译后的 js 文件都下载完成后，才开始进行页面的渲染，文件下载等需要一定的时间等，所以首屏渲染需要一定的时间；SSR 直接由服务端渲染好页面直接返回显示，无需等待下载 js 文件及再去渲染等，所以 SSR 有更快的内容到达时间；

（2）服务端渲染的缺点：
更多的开发条件限制： 例如服务端渲染只支持 beforCreate 和 created 两个钩子函数，这会导致一些外部扩展库需要特殊处理，才能在服务端渲染应用程序中运行；并且与可以部署在任何静态文件服务器上的完全静态单页面应用程序 SPA 不同，服务端渲染应用程序，需要处于 Node.js server 运行环境；
更多的服务器负载：在 Node.js 中渲染完整的应用程序，显然会比仅仅提供静态文件的 server 更加大量占用CPU 资源，因此如果你预料在高流量环境下使用，请准备相应的服务器负载，并明智地采用缓存策略。

二. 打包优化
1. SourceMap 线上环境关闭，第一次发布项目时可以开启，便于查错。
2. splitChunksPlugins： 分割公共代码。
3.减少 ES6 转为 ES5 的冗余代码
babel-plugin-transform-runtime

4，webpack-bundle-analyzer 输出结果分析

三. 基础的 Web 技术优化
1. 服务器端开启 gzip 压缩
2. 浏览器缓存
3. CDN 的使用(静态资源)
4.使用 Chrome Performance 查找性能瓶颈
