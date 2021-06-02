# Vue的MVVM响应式原理
 
vue.js 是采用数据劫持及发布者-订阅者模式，通过Object.defineProperty()来劫持各个属性的setter，getter。在数据变动时发布消息给订阅者，触发相应的监听回调，去更新视图。

Vue内部整合了Observer、Dep、Watcher、Compile四者，来达到数据变化=>视图更新，视图交互变化=>数据model变更的双向绑定效果。

* 监听器Observer，用来劫持并监听所有属性，如果有变动的，就通知订阅者。
* 消息订阅器Dep，订阅器Dep主要负责收集订阅者，然后在属性变化的时候执行对应订阅者的更新函数。
* 订阅者Watcher，每一个Watcher都绑定一个更新函数，watcher可以收到属性的变化通知并执行相应的函数，从而更新视图。
* 解析器Compile，解析指令，初始化模版，绑定订阅者。

myVue.html
```html
<!DOCTYPE html>
<html lang = "en">
    <head>
        <meta charset = "UTF-8">
        <meta name = "viewport" content = "width=device-width,initial-scale=1.0">
        <title>myVue</title>
    </head>
    <body>
        <div id = "app">
            <h2>{{person.name}} -- {{person.age}}</h2>
            <h3>{{person.fav}}</h3>
            <ul>
                <li>1</li>
                <li>2</li>
                <li>3</li>
            </ul>
            <h3>{{msg}}</h3>
            <div v-text = "person.age"></div>
            <div v-text = "msg"></div> 
             <div v-html = "htmlStr"></div> 
            <input type = "text" v-model = "msg">
           
            <button v-on:click = "handleClick">点击事件on </button>
            <button @click = "handleClick">点击事件@ </button> 
        </div>
        <script src = "js/compileUtil.js"></script>
        <script src = "js/observer.js"></script>
        <script src = "js/dep.js"></script>
        <script src = "js/watcher.js"></script>
        <script src = "js/compile.js"></script>
        <script src = "js/myVue.js"></script>
       <script>
            const myVue = new MyVue({
                el : "#app",
                data: {
                    msg : "hello",
                    info: "world",
                    person : {
                        age: 18,
                        name : '张三',
                        fav:"美女"
                    },
                    htmlStr: "<h1>使用v-html 示例</h1>"
                },
                methods: {
                    handleClick(){
                        console.log("handleClick...")
                    }
                }
            })
        </script>
    </body>
</html>

```

compileUtil.js

```javascript
const compileUtil = {
    getValue(key, vm){
        key = key.replace(/\s+/g, '');
        return key.split('.').reduce((data, currentVal) => {
            return data[currentVal]
        }, vm.$data)

    },
    setValue(expr, vm, inputVal){
        expr = expr.replace(/\s+/g, '');
        return expr.split('.').reduce((data, currentVal) => {
             data[currentVal] = inputVal;
        }, vm.$data)
    },

    getContentVal(con, vm){
        //解析双花括号
        let reg = /\{\{(.+?)\}\}/g; //匹配{{}}
        return con.replace(reg, (...args) => {
            return this.getValue(args[1], vm);
        });
    },
    text(node, expr, vm){ //expr: msg
      let value;
      if(expr.indexOf("{{") != -1){
          // {{person.name}} -- {{person.name}}
          value = expr.replace(/\{\{(.+?)\}\}/g, (...args) => {
              //绑定观察者，将来数据发生变化，触发这里的回调进行更新
              new Watcher(vm, args[1], (newVal) => {
                  this.updater.textUpdater(node, this.getContentVal(expr, vm));
              });
              return this.getValue(args[1], vm);
          });
      }else{
          new Watcher(vm, expr, (newVal) => {
              this.updater.textUpdater(node, newVal);
          });
          value = this.getValue(expr, vm);
      }
      this.updater.textUpdater(node, value);
    },
   html(node, expr, vm){
      const value = this.getValue(expr, vm);
      new Watcher(vm, expr, (newVal) => {
          this.updater.htmlUpdater(node, newVal);
      });
      this.updater.htmlUpdater(node, value);
    },
    model(node, expr, vm){
      const value = this.getValue(expr, vm);
      //绑定更新函数  数据=》视图
      new Watcher(vm, expr, (newVal) => {
          this.updater.modelUpdater(node, newVal);
      });

      //视图=》数据
      node.addEventListener("input", (e)=>{
          //设置值
          this.setValue(expr, vm, e.target.value);
      });
      this.updater.modelUpdater(node, value);
    },
     on(node, expr, vm, eventName){
        let fn = vm.$options.methods && vm.$options.methods[expr];
        node.addEventListener(eventName, fn.bind(vm), false);
    },
    bind(node, expr, vm, attrName){
        const value = this.getValue(expr, vm);
        node.setAttribute(attrName, value); //设置属性
    },
    updater : {
      textUpdater(node, value){
          node.textContent = value;
      },
      htmlUpdater(node, value){
          node.innerHTML = value;
      },
      modelUpdater(node, value){
          node.value = value;
      }
    }
}

```

dep.js

```javascript
class Dep {
    constructor(){
        this.subs = [];

    }
    //收集观察者
    addSub(watcher){
        this.subs.push(watcher);
    }

    //通知观察者
    notify(){
        this.subs.forEach(w => {w.update()});
    }
}
```

observer.js:

```javascript
class Observer{
  constructor(data){
      this.observe(data);
  }
  observe(data){
      /** 
       {
           person: {
               name : '张三',
               fav : {
                   a: "爱好"
               }
           }
       }
      */

       if(data && typeof data === "object"){
           Object.keys(data).forEach(key => {
               this.defineReactive(data, key, data[key]);
           })
       }  
    }
   defineReactive(obj, key, value){
      //递归遍历
      this.observe(value);
      const dep = new Dep();

      //劫持并监听所有的属性
      Object.defineProperty(obj, key, {
          enumerable: true,
          configurable: false,
          get(){
              //订阅数据变化时，往Dep中添加观察者
              Dep.target && dep.addSub(Dep.target);
              return value;
          },
          set: (newVal) => {
              //this.observe(newVal);
              if(newVal !== value){
                  value = newVal;
              }
              //Dep通知变化
              dep.notify();

          }
      })
  }
}
```
 
 watcher.js
 
 ```javascript
  class Watcher{
    constructor(vm, expr, cb){
        this.vm = vm;
        this.cb = cb;
        this.expr = expr;
        //先把旧值保存起来
        this.oldVal = this.getOldVal();
    }
    getOldVal(){
      Dep.target = this;
      const oldVal = compileUtil.getValue(this.expr, this.vm);
      Dep.target = null;
      return oldVal;
    }
    update(){
      const newVal = compileUtil.getValue(this.expr, this.vm);
      if( newVal != this.oldVal){
          this.cb(newVal);
      }
    }
}
```

compile.js

```javascript
class ComPile {
    constructor(el, vm) {
        this.el = this.isElementNode(el) ? el :document.querySelector(el);
        this.vm = vm;

        //编译模版：本质是检查模板容器的每一个子节点，当遇到特殊标记时（双花括号，指令），对其进行解析。
        //思路： 遍历容器中所有的子节点：childNodes, 但是频繁地操作Dom元素会造成严重的性能问题。

        /** 
            解决办法： 通过文档片段来编译模板
            1. createDocumentFragment()方法，是用来创建一个虚拟的节点对象。
            2. DocumentFragment节点不属于文档树，它有如下特点
              1>当把该节点插入文档树时，插入的不是该节点本身，而是它所有的子孙节点。
              2>当添加各个dom元素时，如果先将这些元素添加到DocumentFragment中，再统一将DocumentFragment添加
              到页面，会减少页面的重排和重绘，进而提升页面渲染性能。
              3>使用appendChild 方法将dom树中的节点添加到DocumentFragment中时，会删除原来的节点。
        */
          // 1.将模板容器中的所有子节点添加到文档片段中
        const fragment = this.nodeToFragment();
    
        // 2.编译文档片段（解析{{}}、指令）
        this.compileFragment(fragment);

        // 3.将编译后的文档片段追加到模板容器中 
        this.el.appendChild(fragment);
    }

    //将模板容器中的所有子节点添加到文档片段中
    nodeToFragment() {
        //创建文档片段
        const f = document.createDocumentFragment();

        //将模板容器中的所有子节点添加到文档片段中
        while (this.el.firstChild) {
            f.appendChild(this.el.firstChild);
        }
        return f;
    }
    
     //编译文档片段（解析{{}}、指令）
    compileFragment(fragment) {
        //获取所有的子节点
        const childNodes = fragment.childNodes;
        [...childNodes].forEach(child => {
            if(this.isElementNode(child)){ //元素节点， 解析指令
                this.compileElement(child);
            } else{ //文档节点， 解析{{}}
                this.compileText(child);
            }
            //如果node有子节点，则递归调用compile方法
            if(child.childNodes && child.childNodes.length > 0){
                this.compileFragment(child)
            }
        })
    }
    
      //解析文档节点 {{}}
    compileText(node){ 
        const content = node.textContent;
        let reg = /\{\{(.+?)\}\}/g; //匹配文本节点
        if(reg.test(content)){
            compileUtil['text'](node, content, this.vm);
        }   
    }
    
     //解析元素节点
    compileElement(node){
       const attrs =  node.attributes;
       Array.from(attrs).forEach(attr => {
         const { name, value } = attr;
         if(this.isDirective(name)){ //是一个指令 v-text v-html v-model v-on:click v-bind:src
             const [, directive] = name.split('-');  //[v, model]
             const [dirName, eventName] = directive.split(":");  // text html model on
             //更新数据 数据驱动视图
             compileUtil[dirName](node, value, this.vm, eventName);
            //删除有指令的标签上的属性
            node.removeAttribute("v-" + directive);
         }else if(this.isEventName(name)){ //@click = "handleClick"
            let [, eventName ] = name.split('@');
            compileUtil['on'](node, value, this.vm, eventName);
         }
       })
    }

    //是否是指令
    isDirective(attrName){
        return attrName.startsWith('v-');
    }
    
     //是否是事件函数
    isEventName(attrName){
        return attrName.startsWith('@');
    }

    //判断是否是元素节点
    isElementNode(node){
        return node.nodeType === 1;
    }

}

```

MyVue.js
```javascript
class MyVue {
    constructor(options){
        this.$options = options;
        this.$data = options.data;
        this.$el = options.el;
        if(this.$el){  
            //1.实现一个数据观察者 数据劫持
            new Observer(this.$data);
            //2.实现一个指令解析器 模板编译
            new ComPile(this.$el, this);
            //3.数据代理
            this.proxyData();
        }
    }

    //数据代理 访问vm.key 实际上访问的是 vm.$data.key
    proxyData(){
        for(const key in this.$data){
            Object.defineProperty(this, key, {
                enumerable: true,
                configurable: false,
                get(){
                    return this.$data[key];
                },
                set(newVal){
                    this.$data[key] = newVal;
    
                }
            })
        }
    }
}
```
