---
title: js相关知识复习（下）
date: 2021-08-31 12:56:14
description: 宏任务微任务以及执行顺序，强缓存和协商缓存，解决异步回调地狱，事件流（相关问题：事件委托，阻止冒泡），js判断类型,数组常用方法以及数组去重，this指向，new操作符原理，性能优化。
cover: https://cn.bing.com/th?id=OHR.Mpumalanga_ZH-CN9666962271_1920x1080.jpg
categories: '烂笔头经验记录'
---

#### 写在前面

> 这篇主要复习：宏任务微任务以及执行顺序，强缓存和协商缓存，解决异步回调地狱，事件流（相关问题：事件委托，阻止冒泡），js判断类型,数组常用方法以及数组去重，箭头函数，this指向，new操作符原理，跨域，性能优化。

#### 事件循环：宏任务微任务以及执行顺序

JS的一个执行代码机制，采用单线程的事件循环方式管理异步任务，优点简化编程模型，缺点无法发挥CPU的全部性能（但对前端其实没影响）

执行顺序：

 1. 先执行同步任务
 2. 微任务：process.nextTick，Promise，Async/Await
 3. 宏任务：计时器，ajax，读取文件，setTimeout，setInterval

事件循环大致按上述执行顺序执行。值得注意的是，Async/Await就是promise的一种语法糖，有Async没有Await，相当于同步任务，有了Await相当于是promise的then。

```javascript
console.log('a');
setTimeout(()=>console.log('宏1'), 0);
(async ()=>{
  console.log(1);
  await console.log(2);//此处相当于.then()处理，将后面的打印放到微任务中，
  console.log(3);
  setTimeout(()=>console.log('宏2'), 0);
})().then(()=>{
  console.log(4);
});
console.log('b');
//a,1,2,5,3，4，宏1，宏2
```

#### 强缓存和协商缓存
浏览器向服务器发送请求和资源标识，服务器进行**Last-Modified**和 **Etag**判断看不是最新资源。否的话（**强制缓存**）返回最新资源，标识符和200状态码；是的话（**协商缓存**）就返回304，从本地缓存里拿资源。一句话(看资源有无更新，更新了就200)。

一张图了解整个过程：
![在这里插入图片描述](https://img-blog.csdnimg.cn/267d2753d62f40029f921820ca7c1989.png#pic_center)
那么cache-control里有什么内容呢？有资源的状态码 `status`，缓存的有效时间 `max-age`。另外提个知识点：no-cache是弱缓存，要进行验证；no-store是不缓存，只允许你向服务器发送请求，不缓存在本地。


#### 解决异步回调地狱
问题：在ES5前，当要获取一些异步的数据，就无法通过return拿，这时需要回调获取，如果要的数据太多，回调就要注意顺序。然后请求的代码就像大箭头一样。

1. Promise 的实例就是一个异步操作，调用 .then() 方法，指定成功（resolve）将数据传递出来，回调函数拿到异步数据；
2. 使用 async/await。（结合promise，await返回的就是promise的resolve数据）
3. 使用 generator。

例如按顺序依次读取众多文件，会出现回调地狱，采用 Promise 异步的方法使代码能向下延伸，更具可读和维护性。后续需要添加新的读取文件，只需按格式添加。
```javascript
 // 产生回调地狱:获取一些异步数据，无法通过return拿到，回调获取产生回掉地狱
    function work(fn) {
      setTimeout(() => {
        fn("工作中");
      }, 2000)
    }

    function sleep(fn) {
      setTimeout(() => {
        fn("睡觉");
      }, 1000)
    }

    // 通过回调获取异步数据,产生回调地狱
    work(function (data) {
      console.log(data);
      sleep(function (data) {
        console.log(data);
        sleep1(function (data) {
          console.log(data);
          sleep2(function (data) {
            console.log(data);
            sleep3(function (data) {
              console.log(data);
            })
          })
        })
      })
    })
```

正常情况是先打印睡觉在打印工作，如果需要的数据过多且按一定的顺序执行，上面的方法显然不合不适，下面我就列举俩解决方法

```javascript
function work() {
      return new Promise(function (resolve) {
        setTimeout(() => {
          resolve("工作中");
        }, 2000)
      })
    }

    function sleep() {
      return new Promise(function (resolve) {
        setTimeout(() => {
          resolve("睡觉");
        }, 1000)
      })
    }

    // 使用promise解决回调地狱
    // work().then(function (data) {
    //   console.log(data);
    //   return sleep();
    // }).then(function (data) {
    //   console.log(data);
    // })

    // async/await解决
    async function getData() {
      let work1 = await work();
      console.log(work1);
      let sleep1 = await sleep();
      console.log(sleep1);
    }
    getData();
```

#### 事件流（相关问题：事件委托，阻止冒泡）
事件流描述的是从页面中接收事件的顺序，一共三个阶段：捕获阶段，目标阶段，冒泡阶段。一般事件在浏览器中处于冒泡阶段才被执行，如果想在捕获阶段就触发，可用`addEventListener` 方法，这个方法接收3个参数：要处理的事件名、处理函数和布尔值（true就表示在捕获阶段就触发）。

另外相关问题可看我之前写的博客：[相关问题：附例子解释](https://blog.csdn.net/takeingloop/article/details/119898047?spm=1001.2014.3001.5501)。


#### 图片懒加载和预加载
预加载：一下子把页面中的图片缓存到本地，加载时从本地读取，不用等，优化了用户体验，如果网页图片过多会造成加载区域空白的情况。（拿时间换体验）

懒加载：先加载**可视区域**的图片，在将剩下的img标签中的src链接设为同一张图片, 真正的地址存储在img标签的自定义属性中(比如data-src); 当js监听到该图片元素进入可视窗口（scrollTop方法）时,即将自定义属性中的地址存储到src属性中,达到懒加载的效果。


#### [节流和防抖](https://www.cnblogs.com/coco1s/p/5499469.html)
防抖：在事件被触发n秒后再执行事件回调，如果在这n秒内又被触发，则重置定时器。
节流：在一个单位时间内，不管怎样都只能触发一次函数

```javascript
// 简单的防抖动函数
function debounce(func, wait, immediate) {
    // 定时器变量
    var timeout;
    return function() {
        // 每次触发 scroll handler 时先清除定时器
        clearTimeout(timeout);
        // 指定 xx ms 后触发真正想进行的操作 handler
        timeout = setTimeout(func, wait);
    };
};
 
// 实际想绑定在 scroll 事件上的 handler
function realFunc(){
    console.log("Success");
}
 
// 采用了防抖动
window.addEventListener('scroll',debounce(realFunc,500));
// 没采用防抖动
window.addEventListener('scroll',realFunc);
```
上面例子的大概功能就是如果 500ms 内没有连续触发两次 scroll 事件，那么才会触发我们真正想在 scroll 事件中触发的函数（停止滑动才触发）。实际过程中，我们更希望边滑动边加载图片。

与防抖相比，节流函数多了一个 mustRun 属性，代表在 X 毫秒内至少执行一次我们希望触发的事件 handler。而不会像防抖那样，需要达到条件才会触发。

```javascript
// 简单的节流函数
function throttle(func, wait, mustRun) {
    var timeout,
        startTime = new Date();
 
    return function() {
        var context = this,
            args = arguments,
            curTime = new Date();
 
        clearTimeout(timeout);
        // 如果达到了规定的触发时间间隔，触发 handler
        if(curTime - startTime >= mustRun){
            func.apply(context,args);
            startTime = curTime;
        // 没达到触发间隔，重新设定定时器
        }else{
            timeout = setTimeout(func, wait);
        }
    };
};
// 实际想绑定在 scroll 事件上的 handler
function realFunc(){
    console.log("Success");
}
// 采用了节流函数
window.addEventListener('scroll',throttle(realFunc,500,1000));
```
大概功能就是如果在一段时间内 scroll 触发的间隔一直短于 500ms ，那么能保证事件我们希望调用的 handler 至少在 1000ms 内会触发一次。


#### js判断类型,数组常用方法以及数组去重

**js判断类型：**

 1. typeof(A)：只能简单的区分原始类型，遇到数组（输出obj）、对象（obj）、null（obj）无法区分。
 2. A instanceof B：判断A是否为B的实例（其实是判断A是否在B原型链原型构造函数的属性），所以只能测对象。可以对数组、对象类型加以区分。
 3. constructor: 利用原型对象上的 constructor 属性检测，能测基本数据类型，测不了undefined和null。
 4. Object.prototype.toString.call()：对象通过原型链的方法对类型进行判断，数组不能直接使用。

**数组常用方法：**
push()，pop()，shift()，unshift()，splice()，slice()，sort()，map()，forEach()，concat()，fill()，filter()，some()，join()，reduce()，from()， ，参照例子：[例子](https://blog.csdn.net/weixin_45935734/article/details/109843599?ops_request_misc=%257B%2522request%255Fid%2522%253A%2522163022925516780255248691%2522%252C%2522scm%2522%253A%252220140713.130102334..%2522%257D&request_id=163022925516780255248691&biz_id=0&utm_medium=distribute.pc_search_result.none-task-blog-2~all~top_positive~default-1-109843599.pc_search_all_es&utm_term=%E6%95%B0%E7%BB%84%E5%B8%B8%E7%94%A8%E6%96%B9%E6%B3%95&spm=1018.2226.3001.4187)。

这里提一句：splice()，slice()看起来很像容易搞混，`splice`接收（从什么位置开始，要删除的元素，要添加的元素）三个参数，用作数组删除替换或新增，而 `slice` 接收一个包头不包尾的下标参数，用来浅拷贝一个数组，。

**数组去重：**

 1. 简单且常用es6yu方法：Set();

```javascript
let arr =[1, 1, 2, 2, 3 , 3, 4, 5, 5];
console.log([...new Set(arr)]);
```
 2. 借助indexOf()方法判断此元素在该数组中首次出现的位置下标与循环的下标是否相等

```javascript
 var arr = [1,23,1,1,1,3,23,5,6,7,9,9,8,5,5,5]; 
   function norepeat(arr) {
       for (var i = 0; i < arr.length; i++) {
           if (arr.indexOf(arr[i]) != i) {
               arr.splice(i,1);//删除数组元素后数组长度减1后面的元素前移
               i--;//数组下标回退
           }
       }
       return arr;
   }
   var arr2 = norepeat(arr);
   console.log(arr2);   			  //[1, 23, 3, 5, 6, 7, 9, 8]
```

#### 遍历数组的几种方式和不同
你能想到几个，虽说都是遍历，但却各有独特功能
常见的：`for...of`，`for...in`，`for...of`，
功能API：`forEach`，`map`，`filter`，`find`，`findIndex`，`include`，`includes`，`some`，`every`
大都数遍历方法都是callback接收三个可选参数这么个形式，如：arr.forEach((当前item项，当前项的id索引，当前数组) => {})

先说常见的吧：
`for...in`大部分情况是用来遍历对象属性（除Symbol以外的可枚举属性），**输出该对象所包含的属性**（键值），可遍历原型，继承链上的属性。不建议用来遍历数组，因为那样只会得到数组下标。
`for...of`用来遍历数组，**输出该属性的值**，区别就是输出的内容不同，`for...in`因为遍历的更深，所以更耗时

功能API：
`forEach`:超常见，对数组的每个item执行一个回调，万金油API
`map`:这可不是 new Map（），只是个单纯的创建个数组，里面在执行一个`forEach`的回调函数
`filter`: 过滤很好用，相当于for+if，返回一个符合条件的数组
`find`和`findIndex`: 很好用前者返回符合条件的item项，后者返回该项id
`includes`: 检测数组有没有包含该内容，返回一个布尔值
`some`: 用的就比较少了，大致上用于检测是否至少有1个元素通过了被提供的函数测试，需要配套检测逻辑函数来使用。返回的是一个Boolean。
`every`: 跟some差不多，只不过检测的是用例是否全部通过测试。


#### 对象方法
遍历对象的方法有哪些呢?一张图说明白
![在这里插入图片描述](https://img-blog.csdnimg.cn/8eda466902ab4062985bb7b13ff01cb7.png)
常见的有：
- Object.keys()：可遍历自身属性，不可遍历原型链上属性，非枚举属性，symbol属性。
- Object.getOwnPropertyNames():  用法和Object.keys()一样,多了可遍历非枚举属性。
- Object.getOwnPropertySymbols() ：可遍历自身symbol属性（枚举+非枚举）
- for in ：可遍历原型链和自身的可枚举属性，不包括symbol属性，非枚举属性
- Object.values()：获取属性值
```javascript
var eat = Symbol();
var person = {
name: 'kreme',
age: 12,
[eat]: 'male'
}
console.log(Object.keys(person)); // ["name", "age"]
console.log(Object.values(person)); // [1, 2]

```


#### 箭头函数
**箭头函数和普通函数区别：**
箭头函数没有自己的this，他的this指向定义时所在的外层第一个普通函数，且 this指向永远不会改变，call、apply、bind 并不会影响其 this 的指向
![在这里插入图片描述](https://img-blog.csdnimg.cn/6b925efbae0d40b997bc5b9a1c7ebce4.png)
没有原型prototype，不能作为构造函数使用(构造函数的this要是指向创建的新对象,但是箭头的this不会变)，不能new，new了就报错

箭头函数没有自己的arguments参数，他的参数是外层普通函数的，取而代之用rest参数...代替arguments对象，来访问箭头函数的参数列表

```javascript
let a = () => {};
console.log(a.prototype); // undefined

function a() {};
console.log(a.prototype); // {constructor:f}

let obj = {
  a: 10,
  b: () => {
    console.log(this); // window
  },
  c: function() {
    console.log(arguments); 
    console.log(this); // {a: 10, b: ƒ, c: ƒ}
  }
}
obj.b(); 
obj.c();

// rest参数...
let C = (...c) => {
  console.log(c);
}
C(3,82,32,11323);  // [3, 82, 32, 11323]
```

#### this指向
 1. 默认是全局对象：window（普通函数调用和定时器函数指向也是window）
 2. 被构造函数调用时，，this指向该对象（谁调用指向谁）
 3. 对象的方法调用（绑定事件同理）， this 指向该方法所属的对象

更改this指向：call() ，apply()，bind()

#### new操作符原理
1. 创建一个类的实例：创建一个空对象obj，然后把这个空对象的`__proto__`设置为构造函数的prototype。
2. 初始化实例：构造函数被传入参数并调用，关键字this被设定指向该实例obj。
3. 返回实例obj。

#### call ,apply, bind 方法及手写
这三个方法都是改变函数的this指向，期中call ,apply方法一样，只是传入的参数不同，详情见下面栗子，bind只是将结果以函数返回，接收后在调用即可，整体都是采用：B对象.方法.call(A，"参数")形式，表现为A对象要调用B的方法，下面看例子。

```javascript
let dog = {
  name: "小狗",
  can (p1, p2) {
    console.log('我会' + p1 + p2);
  }
}

let cat = {
  name: "小猫"
}

// dog.can.call(cat, "睡觉", "钓鱼");
// dog.can.apply(cat, ["睡觉", "钓鱼"]);
let fn = dog.can.bind(cat, "睡觉", "钓鱼");
fn();
```

不难看出 call ,apply, bind 的整体使用相差无几，根据原理，怎么售手写该类方法呢？


```javascript
Function.prototype.myCall = function(context){
	if(typeof this !== "function"){
		throw new TypeError("Error")
	}
	context = context || window
	context.fn = this
	const args = [...arguments].slice(1)
	const result = context.fn(...args)
	delete context.fn
	return result
}

Function.prototype.myApply = function(context){
	if(typeof this !== "function"){
		throw new TypeError("Error")
	}
	context = context || window
	context.fn = this
	let result
	if(arguments[1]){
		result = context.fn(...arguments[1])
	}else{
		result = context.fn()
	}
	delete context.fn
	return result
}

Function.prototype.myBind = function(context){
	if(typeof this !== 'function'){
		throw new TypeErroe('Error')
	}
	const _this = this
	const args = [...arguments].slice(1)
	return functions F(){
		if(this instanceof F){
			return new _this(...args,...arguments)
		}
		return _this.apply(context,args.concat(...arguments))
	}
}
```

#### 什么是跨域和为什么产生跨域

当一个请求url的`协议、域名、端口`三者之间任意一个与当前页面url不同即为跨域。为什么产生跨域呢？因为浏览器的同源策略限制，当客户端向服务器请求数据时会产生跨域问题。

通过href，src请求下来的资源文件或图片视频文件不存在跨域，Ajax请求才产生跨域

解决方法有：(一般后端操作)

- JSONP（不推荐，因为只能支持GET请求，POST不支持），在请求端设置传入函数，需要返回的数据作为调用函数，
- 修改请求头，CrossOrigin（由spring-web包提供在接口处引入，即可解决，一般用于小程序，**原理：在响应头加入允许跨域参数** `response.addHeader("Access-Control-Allow-Origin","*")`
- Nginx代理或者网关：模拟一个服务器，发送数据时候，客户端->nginx->服务端；返回数据：服务端->nginx->客户端 ，**过程**(在模拟的服务器设置监听端口，location 接口，和 `Access-Control-Allow-Origin *`，然后客户端访问的是模拟服务器端口)
- CORS：先判断请求，根据请求类型自定义请求头来让服务器和浏览器进行沟通
  - 简单请求（get，post，head），在前（Accept，ContentType）后端（Access-Control-Allow-Origin）设置请求头
  - 非简单：会发个header头为 option的请求进行预检（浏览器检查Origin、Access-Control-Allow-Method和Access-Control-Request-Header），预检验过后接下来的请求就相当于简单请求

实现方法参考：[跨域问题解决](https://segmentfault.com/a/1190000011145364)。


#### 性能优化：
**一：包的大小**

 1. 使用路由懒加载，分包
 2. 第三方库按需加载
 3. 使用compressionWebpackPlugin使用gizp压缩
 4. 使用uglifyJS或者terserWebpackPlugin去压缩js代码
 5. 打包的时候取消.map文件

**二：请求速度**

 1. 使用CDN
 2. 减少http请求(合并部分http请求)
 3. 合理使用缓存
 4. 对频繁触发的请求使用防抖节流

**三：页面性能**

 1. 多图的页面使用图片懒加载，骨架屏，优化首屏来加载速度
 2. 减少重排和重绘，使用transform去改变dom的位置
 3. 善用图片格式，比如png质量较高，可以用来做logo，jp(e)g 质量较低(有从上到下和模糊到清晰的两种模式)，webp虽好，但是不是所有浏览器都兼容，70%左右吧。
 4. 合理使用缓存

**四：其他**

 1. for循环先把length取出来，避免多次取值。同理的还有vue对data中某个数据频繁取值，可以缓存下来，避免重复添加依赖
 2. 去除一些绑定的事件，定时器等，或者
 3. HTML语义化

#### 写在最后：

> 每一天都是新一天，争取早些写出高质量代码。
