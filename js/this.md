this指向问题分为下列几种情况：

# 一. 全局 & 调用普通函数
在全局环境中，this 永远指向 window。

```javascript
console.log(this === window);     //true
```

普通函数在调用时候(非构造函数)，其中的 this 也是指向 window。

# 二.构造函数
如果函数作为构造函数使用，那么其中的 this 就代表它即将 new 出来的对象（实例）。

```javascript
//1. 构造函数
function Foo(){
    this.x = 10;
    console.log(this);    //Foo {x:10}
}
var foo = new Foo();
console.log(foo.x);      //10

//2. 普通函数
function Foo(){
    this.x = 10;
    console.log(this);    //Window
}
var foo = Foo();
console.log(foo.x);      //undefined
```

# 三.对象方法
如果函数作为对象的方法时，方法中的 this 指向该对象。

```javascript
var obj = {
    x: 10,
    foo: function () {
        console.log(this);        //Object
        console.log(this.x);      //10
    }
};
obj.foo();
```

注意：若是在对象方法中定义函数，那么情况就不同了。

```javascript
var obj = {
    x: 10,
    foo: function () {
        function f(){ //普通函数
            console.log(this);      //Window
            console.log(this.x);    //undefined
        }
        f();
    }
}
obj.foo();
```

如果 foo 函数不作为对象方法被调用：
```javascript
var obj = {
    x: 10,
    foo: function () {
        console.log(this);       //Window
        console.log(this.x);     //undefined
    }
};
var fn = obj.foo;
fn();
```
# 四. 构造函数 prototype 属性
```javascript
function Foo(){
    this.x = 10;
}
Foo.prototype.getX = function () {
    console.log(this);        //Foo {x: 10, getX: function}
    console.log(this.x);      //10
}
var foo = new Foo();
foo.getX();
```
在 Foo.prototype.getX 函数中，this 指向的 foo 对象。不仅仅如此，即便是在整个原型链中，this 代表的也是当前对象的值。

# 五.函数用 call、apply或者 bind 调用。

```javascript
var obj = {
    x: 10
}
function foo(){
    console.log(this);     //{x: 10}
    console.log(this.x);   //10
}
foo.call(obj);
foo.apply(obj);
foo.bind(obj)();
```
当一个函数被 call、apply 或者 bind 调用时，this 的值就取传入的对象的值。
# 六.箭头函数中的 this
箭头函数会继承外层函数，调用的 this 绑定（ 无论 this 绑定到什么），没有外层函数，则是绑定到全局对象（浏览器中是window）。 

箭头函数体内的this对象，就是定义该函数时所在的作用域指向的对象，而不是使用时所在的作用域指向的对象。

```javascript
//示例1
function foo() {  
  setTimeout(()=>{
    console.log(this.a);
  },100)
}
var obj = {
  a: 2
}
foo.call(obj);

//示例2
var obj = {
    x: 10,
    foo: function() {
        var fn = () => {
            return () => {
                return () => {
                    console.log(this);      //Object {x: 10}
                    console.log(this.x);    //10
                }
            }
        }
        fn()()();
    }
}
obj.foo();


//示例3
var name = 'window'; 

var A = {
   name: 'A',
   sayHello: () => {
      console.log(this.name)
   }
}

A.sayHello();// 输出window

//示例4
var name = 'window'; 

var A = {
   name: 'A',
   sayHello: function(){
      var s = () => console.log(this.name)
      return s//返回箭头函数s
   }
}

var sayHello = A.sayHello();
sayHello();// 输出A 

var B = {
   name: 'B';
}

sayHello.call(B); //还是A
sayHello.call(); //还是A

A.sayHello.call(B)();  //打印的就是B，指向父级函数被调用时候的this，这种说法更严谨

```
