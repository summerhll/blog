# 一. 接口权限
接口权限目前一般采用 jwt 的形式来验证，没有通过的话会返回错误信息，跳转到登录页面重新进行登录。
登录完拿到token，将 token 存起来，通过 axios 请求拦截器进行拦截，每次请求的时候会携带 token。

# 二.菜单权限
初始化的时候先挂载不需要权限控制的路由，比如登录页，404 等错误页。如果用户通过URL进行强制访问，则会直接进入 404，相当于从源头上做了控制.
登录后，获取用户的权限信息，然后筛选有权限访问的路由，在全局路由守卫beforeEach里进行调用 addRoutes 添加路由。

# 三.按钮权限
通过自定义指令进行按钮权限的判断
```javascript
//首先配置路由
{
    path: '/permission',
    component: Layout,
    name: '权限测试',
    meta: {
        btnPermissions: ['admin', 'supper', 'normal']
    },
    //页面需要的权限
    children: [{
        path: 'supper',
        component: _import('system/supper'),
        name: '权限测试页',
        meta: {
            btnPermissions: ['admin', 'supper']
        } //页面需要的权限
    },
    {
        path: 'normal',
        component: _import('system/normal'),
        name: '权限测试页',
        meta: {
            btnPermissions: ['admin']
        } //页面需要的权限
    }]
}


//自定义权限鉴定指令
import Vue from 'vue'
/**权限指令**/
const has = Vue.directive('has', {
    bind: function (el, binding, vnode) {
        // 获取页面按钮权限
        let btnPermissionsArr = [];
        if(binding.value){
            // 如果指令传值，获取指令参数，根据指令参数和当前登录人按钮权限做比较。
            btnPermissionsArr = Array.of(binding.value);
        }else{
            // 否则获取路由中的参数，根据路由的btnPermissionsArr和当前登录人按钮权限做比较。
            btnPermissionsArr = vnode.context.$route.meta.btnPermissions;
        }
        if (!Vue.prototype.$_has(btnPermissionsArr)) {
            el.parentNode.removeChild(el);
        }
    }
});

// 权限检查方法
Vue.prototype.$_has = function (value) {
    let isExist = false;
    // 获取用户按钮权限
    let btnPermissionsStr = sessionStorage.getItem("btnPermissions");
    if (btnPermissionsStr == undefined || btnPermissionsStr == null) {
        return false;
    }
    if (value.indexOf(btnPermissionsStr) > -1) {
        isExist = true;
    }
    return isExist;
};
export {has}

//在使用的按钮中只需要引用 v-has 指令
<el-button @click='editClick' type="primary" v-has>编辑</el-button>

```
