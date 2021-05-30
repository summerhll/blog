
# promise 源码手写

```javascript
const PENDING = 'PENDING';
const FULLFILLED = 'FULLFILLED';
const REJECTED = 'REJECTED';


/** 
   通过判断x的值 => 来推导promise2的状态:
   判断x是不是promise
   如果是Promise就采用他的状态
   如果不是promise，直接将结果传递下去即可
 * 
 */
const resolvePromise = (promise2, x, resolve, reject) => {
    //判断x的值和promise2是否一样，如果一样，则抛出错误
    if (promise2 == x) {
        return reject("触发循环引用，出错了");
    }

    //x是对象或者函数
    if (x != null && (typeof x === "object" || typeof x === "function")) {
        let called; //防止多次调用成功和失败方法, called可以去除
        try {
            let then = x.then; //then可能是通过defineProperty定义的，可能会出错
            if (typeof then === "function") {
                then.call(x, y => {
                    if (called) {
                        return;
                    }
                    called = true; //防止多次调用成功和失败方法
                    resolvePromise(promise2, y, resolve, reject);
                }, r => {
                    if (called) {
                        return;
                    }
                    called = true;
                    reject(r);
                })

            } else {
                resolve(x); //说明x是一个普通对象，直接成功即可
            }
        } catch (e) {
            if (called) {
                return;
            }
            called = true;
            reject(e);
        }

    } else { //x是普通值
        promise2.resolve(x);
    }
}



class Promise {

    constructor(excutor) {
        this.status = PENDING;
        this.value = undefined; //成功的值
        this.reason = undefined; //失败的原因

        this.onResolvedCallbacks = []; //成功回调的数组
        this.onRejectedCallbacks = []; //失败回调的数组

        //成功函数
        let resolve = (value) => {
            if (this.status = PENDING) {
                this.value = value;
                this.status = FULLFILLED;
                this.onResolvedCallbacks.forEach((fn) => fn(this.value));
            }
        }

        //失败函数
        let reject = (reason) => {
            if (this.status = PENDING) {
                this.reason = reason;
                this.status = REJECTED;
                this.onRejectedCallbacks.forEach((fn) => fn(this.reason));
            }
        }
        try {
            excutor(resolve, reject); //默认执行器会立刻执行
        } catch (err) {
            reject(err); //执行出错
        }
    }


    then(onfulfilled, onrejected) {
        //onfulfilled onrejected可选参数
        onfulfilled = typeof onfulfilled === "function" ? onfulfilled : data => data;
        onrejected = typeof onrejected === "function" ? onrejected : err => {
            throw err;
        };

        let promise2 = new Promise((resolve, reject) => { //executor会立刻执行
            //同步
            if (this.status === FULLFILLED) {
                //通过setTimeout来获取promise2的返回结果
                setTimeout(() => {
                    //异常处理
                    try {
                        let x = onfulfilled(this.value);
                        resolvePromise(promise2, x, resolve, reject);

                    } catch (e) {
                        reject(e);
                    }
                }, 0)
            }

            //同步
            if (this.status === REJECTED) {
                setTimeout(() => {
                    try {
                        let x = onrejected(this.reason);
                        resolvePromise(promise2, x, resolve, reject);
                    } catch (e) {
                        reject(e);
                    }

                }, 0) // >=4ms

            }

            //异步
            if (this.status === PENDING) { //如果是异步 就先订阅好
                this.onResolvedCallbacks.push(() => {
                    setTimeout(() => {
                        try {
                            let x = onfulfilled(this.value);
                            resolvePromise(promise2, x, resolve, reject);
                        } catch (e) {
                            reject(e);
                        }
                    }, 0);
                });

                this.onRejectedCallbacks.push(() => {
                    setTimeout(() => {
                        try {
                            let x = onrejected(this.reason);
                            resolvePromise(promise2, x, resolve, reject);
                        } catch (e) {
                            reject(e);
                        }
                    }, 0);
                })
            }
        });
        return promise2;
    }

}

module.exports = Promise;
```
