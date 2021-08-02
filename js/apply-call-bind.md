call和apply的作用一样，只是接受参数的方式不一样，call接受的是多个单个参数，apply接受的是参数数组。

call和apply的作用简单地可以说成，当一个对象实例缺少一个函数/方法时，可以调用其他对象的现成函数/方法，其方式是通过替换其中的this为这个对象实例，改变函数运行时的上下文。

# apply实现
```javascript
 Function.prototype.myApply = function (context = window, args) {
      if (typeof this !== 'function') {
        throw new TypeError('Type Error');
      }
      const fn = Symbol();
      context[fn] = this;
      let result;
      if (Array.isArray(args)) { 
        result = context[fn](...args);
      } else {
        result = context[fn]();
      }
      delete context[fn];
      return result;
    }

```

# call实现
```javascript
Function.prototype.myCall = function (context = window, ...args) {
      if (typeof this !== 'function') {
        throw new TypeError('Type Error');
      }
      const fn = Symbol();
      context[fn] = this;
      const result = context[fn](...args);
      delete context[fn];
      return result;
   }
```

# bind实现

bind 和 call/apply 一样，都是用来改变上下文 this 指向的，不同的是，call/apply 是直接使用在函数上，而 bind 绑定 this 后返回一个函数（闭包）。

```javascript
Function.prototype.bind = function(context, ...args) {
  if (typeof this !== 'function') {
    throw new Error("Type Error");
  }
  // 保存this的值
  var self = this;

  return function F() {
    // 考虑new的情况
    if(this instanceof F) {
      return new self(...args, ...arguments)
    }
    return self.apply(context, [...args, ...arguments])
  }
}

```
