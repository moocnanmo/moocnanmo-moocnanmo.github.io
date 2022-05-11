---
title: Promise之初级认识（是什么，有什么，怎么用，为什么）
cover: https://img-blog.csdnimg.cn/e1f2dd569f054e538e50e091dca7c79f.jpg
description: 以下所述内容大部分为本人在MDN-Promise，ES6标准入门（第二版）192页开始，和一些博客介绍的消化咀嚼产出，带着粗浅的认识和个人主观偏向，可能理解的不是很到位 #文章描述，只显示该处内容
categories: '烂笔头经验记录'
---

### 写在前面
> 以下所述内容大部分为本人在MDN-Promise，ES6标准入门（第二版）192页开始，和一些博客介绍的消化咀嚼产出，带着粗浅的认识和个人主观偏向，可能理解的不是很到位，想看具体讲解的点下面传送门

-  [MDN对Promise介绍](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Promise)
- [ES6标准入门介绍](https://es6.ruanyifeng.com/?search=promise&x=0&y=0#docs/promisec)

### Promise是什么

按照官方的话来说，Promise是异步编程的一种解决方案，里面通常保存着异步操作事件的结果。以前用嵌套的回调函数解决异步（产生回调地狱），现在用Promise。
所以promise是用来

- 解决异步的问题
- 可以并发多个请求，获取并发请求中的数据

### Promise有什么

1. 有三种状态：Pending（进行中）、Resolved（成功，又称Fulfilled）和Rejected（失败），只有两种变化情况，由Pending变为Resolved或Rejected。这些状态只由操作的结果决定，状态一旦发生了改变，就会一直保持，后续回调无法改变。
2. 有各种内置方法：reject，resolve，then，catch，all，race。

### Promise怎么用

基本使用：

```javascript
var promise = new Promise(function(resolve, reject) {
	if ( /* 异步操作成功 */ ) {
		resolve(value);
		} else {
		reject(error);
	}
});
```

Promise构造函数接收一个函数作为参数，该函数的两个参数分别是resolve和reject。resolve为成功状态，并返回操作结果，reject为失败，且返回错误报告。

Promise实例生成后，可以用**then**方法（在Promise原型对象内置有），接收两个参数，分别指定Resolved状态和Reject状态的回调函数。成功value，失败为reason或error。

```javascript
promise.then(function(value) {
// success
}, function(error) {
// failure
});
```

完整示例：

```javascript
function doDelay(time){
    return new Promise((resolve,reject) => {
        setTimeout(() => {
            if(){
                resolve("xxx")
               }
            else{
                reject("xxx")
             }
        }),time
    })
}
doDelay(200).then(value => {
    
},reason => {
    
})
```

简直不要太简单，下面简单说下Promise其他方法

- **Promise.prorotype.catch()** 方法：失败的回调函数,用来指定reject的回调，即使上面resolve报错了，也能执行到catch。
说明：是then的语法糖，相当于：then(undefined,onRejected)
- **Promise.all** 方法：接收一个包含n个promise的数组
说明：返回一个新的promise，只有所有的promise都成功才成功，只要有一个失败则直接失败且第一个失败的实例作为返回值，一般用在一次加载很多请求，全加载成功在渲染。
- **Promise.race** 方法：跟 all 一样，接收一个包含n个promise的数组，只要有一个实例改变状态，该实例就作为返回值返回
说明：返回一个新的promise，第一个完成的promise的结果状态就是最终的结果状态
- **Promise.allSettled()** ：all的改进版，跟all一样，但是等入参的promise状态都发生了变更才返回，返回一个包含他们状态的Promise。
- **finally()** ：不接收参数，不管Promimse对象最后状态是什么，都会执行的操作。

```javascript
myPromise()
.then(
    function(value){
        console.log('resolved');
    }, 
    function(err, data){
        console.log('rejected');
    }
);

//或者
myPromise()
.then(function(data){
    console.log('resolved');
})
.catch(function(reason){
    console.log('rejected');
});

myPromise()
.all([runAsync1(), runAsync2(), runAsync3()])
.then(function(results){
    console.log(results);
});
```

### 为什么要用Promise

1. 指定回调函数方式更加灵活
2. 支持链式调用，不阻塞Promise相关的任何程序，且保持异步关系来解决回调地狱问题


### Promise在ajax中的应用（封装）

发送ajax请求的四个步骤

```javascript
// 1.创建 XMLHttpRequest 对象
var xhr = new XMLHttpRequest();
// 2.设置状态监听函数
xhr.onreadystatechange = function () {// 状态发生变化时，触发回调函数
    if (xhr.readyState !== 4) return;
    if (xhr.status === 200) {
        // 成功：从服务器获取数据，通过responseText拿到响应的文本
        console.log(xhr.responseText)
    } else {
        // 失败，根据响应码判断失败原因
        new Error(xhr.statusText)
    }
};
// 3.规定请求的类型、URL 以及是否异步处理请求
xhr.open("GET", url, true);
// 4.将请求发送到服务器
xhr.send();
```

开发中一般使用promise对ajax进行一个异步封装，具体如下：

```javascript
// Promise封装Ajax请求
function ajax(method, url, data) {
    var xhr = new XMLHttpRequest();
    return new Promise(function (resolve, reject) {
        xhr.onreadystatechange = function () {
            if (xhr.readyState !== 4) return;
            if (xhr.status === 200) {
                resolve(xhr.responseText);
            } else {
                reject(xhr.statusText);
            }

        };
        xhr.open(method, url);
        xhr.send(data);
    });
}

ajax('GET', '/api/xxx').then(function (data) {
    // 成功，拿到响应数据
    console.log(data);
}).catch(function (status) {
    // 失败，根据响应码判断失败原因
    new Error(status)
});

```
  
对该技术有更深的认识后，在更...
贴个大佬的手写源码

```javascript
//注：promise对象状态与结果只能改变一次
function Promise(executor){
    //初始哈Promise状态：pending
    this.PromiseState = "pending"
    //定义Promise结果
    this.PromiseResult = null
    this.callbacks = []
    //用self接收this指向：Promise构造函数
    const self = this
    //声明resolve函数
    function resolve(data){
        //Promise状态只能改变一次
        if(self.PromiseState !== "pending") return 
        self.PromiseState = "fulfilled"
        self.PromiseResult = data
        //异步任务then方法执行回调
        self.callbacks.forEach(item => {
            item.onresolved(data)
        })
    }
    function reject(data){
        //Promise状态只能改变一次
        if(self.PromiseState !== "pending") return 
        self.PromiseState = "rejected"
        self.PromiseResult = data
        //异步任务then方法执行回调
        self.callbacks.forEach(item => {
            item.onrejected(data)
        })
    }
    try {
        //executor构造器，里面为resolve与reject函数
        executor(resolve,reject)
    } catch (error) {
        //当throw抛出错误时进行捕获执行reject回调
        reject(error)
    }
    
}
//定义Promise对象的then方法
Promise.prototype.then = function(onresolved,onrejected){
    //用self接收this指向：promise对象
    const self = this
    //对接收的实参进行判断，如果不是函数则将实参定义为函数并返回该实参
    if(typeof onresolved !== "function"){
        onresolved = value => value
    }
    //对接收的实参进行判断，如果不是函数则将实参定义为函数并返回该实参
    if(typeof onrejected !== "function"){
        onrejected = reason => {
            throw reason
        }
        }
    //then方法返回值是一个promise对象
    return new Promise((resolve,reject) => {
        //声明callback方法实现函数复用
        function callback(type){
            try {
                //判断返回的promise对象的结果
                let result = type(self.PromiseResult)
                //如果是promise对象则调用then方法，改变promise状态与结果
                if(result instanceof Promise){
                    result.then(v => {
                        resolve(v)
                    },r => {
                        reject(r)
                    })
                }else{
                //如果不是promise对象则调用resolve函数改变promise对象状态与结果
                    resolve(result)
                } 
                //如果是抛出错误则调用reject函数改变promise对象状态与结果
            } catch (error) {
                reject(error)
            } 
        }
        //实现then方法异步执行
        if(this.PromiseState==="fulfilled"){
            setTimeout(() => {
                callback(onresolved)
            });
        }
        //实现then方法异步执行
        if(this.PromiseState === "rejected"){
            setTimeout(() => {
                callback(onrejected)
            });
        }
        //每执行一次then方法当promise状态为pending时就将onresolved与onrejected函数放入callbacks中
        if(this.PromiseState === "pending"){
            this.callbacks.push({
                onresolved:function(){
                    callback(onresolved)
                },
                onrejected:function(){
                    callback(onrejected)
                }
            })
        }
    })  
}
//promise对象catch方法的实现，当抛出错误时进行捕获执行onrejected函数
Promise.prototype.catch = function(onrejected){
    return this.then(undefined,onrejected)
}
//promise内置方法resolve实现
Promise.resolve = function(value){
    //resolve返回结果是一个promise对象
    return new Promise((resolve,reject) => {
        //当传入实参为promise对象时，实参调用then方法改变返回promise对象状态与结果
        if(value instanceof Promise){
            value.then(v => {
                resolve(v)
            },r => {
                reject(r)
            })
        //如果实参不是promise对象，执行resolve函数改变返回promise对象状态与结果
        }else{
            resolve(value)
        }
    })
}
//promise内置方法reject实现
Promise.reject = function(reason){
    //reject返回结果是一个promise对象，直接调用reject方法改变promise结果与状态
    return new Promise((resolve,reject) => {
        reject(reason)    
    })
}
//promise内置方法all实现
Promise.all = function(promises){
    //all返回结果是一个promise对象
    return new Promise((resolve,reject) => {
        //用count进行计数
        let count = 0
        let arr = []
        //遍历promises数组
        for(let i = 0;i < promises.length;i++){
            //数组元素调用then方法，执行resolve函数则count自增
            promises[i].then(v => {
                count++
                //用arr接收每一个promise调用resolve结果
                arr[i] = v
                //当全部数组元素都执行resolve方法，才会执行resolve方法改变返回promise对象状态与结果
                if(count == promises.length){
                    resolve(arr)
                }
                //只要执行一次reject方法则直接改变返回promise对象状态与结果
            },r => {
                reject(r)
            })
        }
    })
}
//promise内置方法race实现
Promise.race = function(promises){
    //race方法返回结果是一个promise对象
    return new Promise((resolve,reject) => {
        for(let i = 0;i < promises.length;i++){
            //哪一个数组元素先执行resolve或者reject函数即改变返回promise对象结果与状态
            promises[i].then(v => {
                resolve(v)
            },r => {
                reject(r)
            })
        }
    })
}

```
