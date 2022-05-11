---
title: js相关知识复习（上）
date: 2021-08-26 23:15:14
description: js知识点挺多的，还有很多（生命周期，箭头函数，继承，常用的类方法，防抖节流，预加载懒加载，跨域，回调地狱等）没记录上，争取把js重点都记录上，深入浅出。
cover: https://cn.bing.com/th?id=OHR.SeaSwallow_ZH-CN1134903878_1920x1080.jpg
categories: '烂笔头经验记录'
---

#### 写在前面：

> 这篇博文偏应用，改天再写篇概念性的。该篇幅内容包括：减少DOM操作，时间委托，冒泡事件，Promise相关， 微任务宏任务，作用域，变量提升，闭包，基本数据类型和检测方法，深浅拷贝，原型和作用域链后续争取把js重点都记录上，深入浅出。

## DOM操作

1. 使用文档碎片减少DOM操作
 2. 冒泡事件：stopPropagation();阻止向上冒泡
 3.  事件委托 ：减少DOM请求次数

>#### 使用文档碎片减少DOM操作

假设在一个ul中，如果持续添加一百次插入 li 的DOM操作，那这个过程无疑是非常消耗内存的，这时不妨使用文档碎片 `fragment` ，将插入的100次操作存储在这，结束后在一并添加到ul中，这样DOM操作从原本的100次变为一次。

```html
 <ul id="list">
  </ul>
  <script src="index.js"></script>
```

```javascript
const list = document.getElementById('list');
// 创建文档碎片,内容放在内存里，最后将结果一次插入list，减少DOM操作
const fragment = document.createDocumentFragment();

for (let i = 0; i < 5; i++) {
  const item = document.createElement('li');
  item.innerHTML = `事件${i}`;
  // 每次插入都是一次DOM操作，消耗性能，所以先往文档碎片放入li,循环完后在插入list
  // list.appendChild(item);
  fragment.appendChild(item);
}

list.appendChild(fragment);
```


>####  阻止冒泡事件

依旧举个栗子：当点击一个 li , 他没有绑定事件处理函数，那么他会往父级元素（一直往上找直到body）看有没有事件处理函数，有的话同样触发。

这样可以简便的为子元素绑定处理事件，但弊端也很明显：如果父元素有处理函数，即使你给子元素绑定了处理事件，那么在触发子元素的处理事件同时，还会向上冒泡在触发父元素的处理函数！！这就需要到 ` stopPropagation();` 阻止冒泡行为。

```html
<div id="one">
    <p id="p1">冒泡</p>
  </div>
  <hr>
  <div id="two">
    <p id="p4">冒泡</p>
    <p id="p5">阻止</p>
  </div>
```

```javascript
const p3 = document.getElementById('p3');
const one = document.getElementById('one');
const body = document.body;

function bindEvent (elem, type, fn) {
  elem.addEventListener(type, fn);
}

bindEvent(one, 'click', function () {
  console.log("one的click");
})
bindEvent(body, 'click', function () {
  console.log("body的click");
})
bindEvent(p3, 'click', function () {
  console.log("p5的click");
})**
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/d95ecf65803a4d4b9e26e5ef14cd09b6.png#pic_center=300X)
在没有绑定阻止冒泡时，给了body，第一个父元素和第三个子元素绑定处理事件，点击第三个会触发第三个和body的，点击第一个会触发父元素和body的。如果只想触发第三个事件绑定，可将p3的触发函数修改为：传入event，利用event下的` stopPropagation();` 方法。

```javascript
bindEvent(p3, 'click', function (event) {
  event.stopPropagation();
  console.log("阻止冒泡");
})
```



>####  事件委托

事件指的是：不在要发生的事件（直接dom）上设置监听函数，而是在其父元素上设置监听函数，通过事件冒泡，监听子元素，来做出不同的响应。

例如在ul中有这么五个小 li ，要给他们绑定点击事件，可获取他们标签名后，利用slice方法对li的类数组转化为数组，在对其中的 li 添加注册事件。
```html
<ul id="list">
    <li>事件1</li>
    <li>事件2</li>
    <li>事件3</li>
    <li>事件4</li>
    <li>事件5</li>
    <button id="btn">点击添加委托事件</button>
  </ul>
```

```javascript
const lis = document.getElementsByTagName('li');
listAray = Array.prototype.slice.call(lis);
listAray.forEach(li => {
  addEvent(li, 'click', () => {
    alert(li.innerHTML);
  })
});
```
要是有500个li呢，那岂不是遍历绑定500次，这产生的事件监听器非常消耗内存。这时可以找到**其父元素**，设置事件监听器，利用 `target` 给 li 绑定事件监听。
```javascript
// 绑定事件处理函数
function addEvent (elem, type, fn) {
  elem.addEventListener(type, fn);
}

addEvent(list, 'click', (e) => {
  const target = e.target;
  if (target.nodeName === 'LI') {
    alert(target.innerHTML);
  }
})

btn.addEventListener('click', () => {
  const li = document.createElement('li');
  li.innerHTML = '新增绑定事件';
  list.insertBefore(li, btn);
})
```
这样，即使不循环遍历 ul 下的 li ,也能通过对 ul 的绑定打印出子元素 li 的信息。


----

## 异步
 1. 使用promise加载一张图片
 2. then，catch改变promise的状态
 3. async , await , 和Promise
 4. 微任务和宏任务


>####  使用promise加载一张图片

考验对promise的基本使用情况。思路：实例化一个promise对象，对图片的加载分别成功与否用`resolve, reject` 处理。

```javascript
function loadImage (src) {
  // 创建并实例化一个promise对象
  const promise = new Promise((resolve, reject) => {
    const img = document.createElement('img');
    // 如果图片加载成功，就传入图片进resolve
    img.onload = function () {
      resolve(img);
    }
    img.onerror = function () {
      const err = new Error(`图片加载失败， url：${{ src }}`)
      reject(err);
    }
    img.src = src;
  });
  return promise;
}

const url = 'https://cn.bing.com/th?id=OHR.WalhallaOverlook_ZH-CN1059655401_1920x1080.jpg&rf=LaDigue_1920x1080.jpg'
loadImage(url).then(img => {
  console.log('img', img);
}).catch(e => {
  console.log('err', e);
})
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/519f09a914c04951a6bcdf97fcf1ee3d.png#pic_center)
>####  使用then，catch改变promise的状态

先说结论：

then 和 catch 正常返回的时候，Promise状态是 fulfilled ,抛出错误的时候，Promise 状态是 rejected。
fulfilled 状态的Promise执行then 的回调，而rejected则执行catch的回调。

怎么理解？看下面例子：

```javascript
const p1 = Promise.resolve();

console.log('p1', p1);

const p1Then = p1.then(() => {
  console.log('p1Then', p1Then);
})

const p1Error = p1.then(() => {
  throw new Error('p1 then error')
})
console.log('p1Error', p1Error);

p1Then.then(() => {
  console.log('fulfilled状态走then回调');
}).catch(() => {
  console.log('fulfilled状态走catch回调');
})

p1Error.then(() => {
  console.log('p1Error走then回调');
}).catch(() => {
  console.log('p1Error走catch回调');
})
```
首先定义一个 resolve 返回值的promise对象，打印显示状态为 fulfilled  ，保存当前信息在打印走catch还是then；然后在手动跑出个错误，打印当前状态，在判断走catch还是then。结果显示：
![在这里插入图片描述](https://img-blog.csdnimg.cn/c9eb2423e84947639478a5ca6fc90aaa.png#pic_center)
再回过头看先前的结论，发现也不难理解。reject类型的演示也差不多，结论见上。


>####  async , await , 和Promise

 这里也先说这三者的结论：
执行async 函数返回的都是Promise对象；promise.then的情况对应await ；promise.catch异常情况对应try...catch。

```javascript
// async结论 ：async返回的都是Promise对象
async function test1 () {
  return 1;
}
const result1 = test1();
console.log('result1', result1);

// await 结论:promise.then成功情况对应await
async function test2 () {
  const p2 = Promise.resolve(2);
  p2.then(data => {
    console.log('promise情况：', data);
  })
  const data = await p2;
  console.log('await情况：', data);
}
test2();

// promise.catch异常情况对应try...catch
async function test3 () {
  const p3 = Promise.reject(6);
  try {
    const data3 = await p3;
    console.log('data3', data3);
  } catch (e) {
    console.log('try-catch捕捉的promise异常：', e);
  }
}
test3();
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/a7eac95f2d9d45fe9ce796369a550bd6.png#pic_center)例子比较通俗易懂，配合结果应该能更清晰的理解结论。

>####  微任务和宏任务

宏任务有：setTimeout , setInterval , Dom事件，AJAX请求
微任务有：Promise ，async / await 
微任务会比宏任务先执行，具体可搜该类型的答应题目进行深入理解。

---

## 作用域，变量提升，和闭包

关于作用域，有全局和局部（函数作用域，块级作用域ES6才有），局部能访问全局，全局不能访问局部。在查找变量的时候，变量的父子级关系或者查找的过程就是一条作用域链。

来个设计之处的小BUG：按理来说全局作用下不能访问局部变量，但事实好像是：JS在设计之初的好像压根就没把块级作用域的事情考虑进去，直到后来ES6出let 和 const 代替 var 关键字来解决这一现象。

```javascript
if (true) {
var a = 1;
}
console.log(a); // 1

//解决方式，函数声明作用域，解决了全局能访问局部作用域情况
var b = () => {
var a = 1;
}
b();
console.log(a); // a is not defined
```

上述乱使用var会找出变量污染，在let 和 const 没出之前，用函数声明作用域可解决上述现象。实际业务中基本不会使用var，取而代之的const和let。这也是**JS模块化**的体现。

问题来了，如果业务中就是要引用某局部作用域的变量呢，那怎么办？这时候**闭包**的作用就体现了，往下看

作用域声明里，还有个知识点：**变量提升**，这有关JS编译执行过程，挺有意思，要理解这得多看几道作用域输出变量的题(下面这张我是去牛客网刷js专项训练截来的)
![在这里插入图片描述](https://img-blog.csdnimg.cn/4f61ec055721416a854a46430e442f1c.png#pic_center)闭包和匿名函数不要搞混！那么什么是闭包，红宝书说：**闭包是有权访问另一个函数作用域的变量函数**。也就是说，局部作用域之间并非是不能互相访问的，可以通过闭包来访问局部的变量。下面举俩个小例子：

```javascript
// 函数作为返回值，在定义的地方向上找变量，而不是再调用的地方
function test1 () {
  const a = 1;
  return function () {
    console.log('a', a);  //输出：a 1
  }
}
const a1 = test1();
const a = 2;
a1();

// 函数作为参数
function test2 (fn) {
  const b = 6;
  fn();
}

const b = 7;
function fn () {
  console.log('b', b); //输出：b 7
}
test2(fn);
```

就一句话，**闭包调用的函数，变量在定义的地方向上找，而不是再调用的地方**

**为什么要使用闭包：** 闭包可以缓存结果不释放变量内存，方便下次调用直接拿取。 因为正常情况下，函数执行完存储在内存的变量会被销毁，再次调用该函数需要重新计算。假设这函数计算量大且耗时，每次调用都得重新计算，那将会浪费内存，影响用户体验，这时就需要道闭包。

闭包的弊端：由于闭包的变量被保存在内存中，所以内存消耗很大，容易导致内存泄漏；容易改变父函数内部变量的值。


---

## 变量类型，深浅拷贝

变量类型有：基本类型（String、Number、Boolean、Symbol、Undefined、Null，BigInt），和引用类型（object , array ,function ）。原始类型的存储改变发生在桟中执行，使用具体数值直接存储；引用类型则在堆中，且`存储的数据是地址`而非具体数值。

顺带说下类型的判断：

- typeof：分不清普通对象（typeof null =object）还是数组对象( typeof []=object)
- instanceof：用来测A 是不是B的原型的，能清楚判断对象类型但也只能测对象类型
- constructor： 利用创建时产生的原型链追溯类型，null 和 undefined 是无效的对象，所以测不了
- Object.prototype.toString.call()：都能测

```javascript
//类型判断
typeof("")

[] instanceof Array;// true
{} instanceof Object;// true
newDate() instanceof Date;// true

let arr=[];
console.log(b.constructor === Array);
console.log("".constructor === String);

Object.prototype.toString.call(newDate()) ;// [object Date]
Object.prototype.toString.call([]) ;// [object Array]

// 基础类型
let a = 10;
let b = a;
a = 20;
console.log('b:', b);  //输出b: 10

// 引用类型
let c = { age: 20 };
let d = c;
c.age = 21;
console.log('d:', d);  //输出d: { age: 21 }
```

在上面的例子中，原始类型的变量有私有内存各自存储数值，一方的改变不会牵动自身；引用类型d指向c的存储地址，c值改变后,d用的依旧是c的地址所以输出21。

上面其实引出了深拷贝和浅拷贝的概念，浅拷贝：复制的同时一方变另一放跟着变（修改的是堆内存中的同一个值），深拷贝，完全复制了一个对象，一方的改变不会牵动另一方（修改堆的不同的值）。

如何实现浅拷贝？
如果是数组，可以使用`slice，concat，Array.from , 展开运算符` ，来返回一个新数组的特性实现拷贝。但是如果数组嵌套了其他引用类型， `concat` 方法将复制不完整。

原理：

```javascript
function copy (obj){
    if(typeof obj === 'object' && obj !== '' ){
  let copy = Array.isArray(obj) ? [] :{};
        for(var p in obj){
            copy[p]=obj[p];
        }
        return copy;  
    }
    else {return obj;}
}
```

例子：

```javascript
var obj = {
  A: '1',
  B: {
    b: '12',
    c: 28,
  },
}

var copy1 = [];
Object.assign(copy,obj);//或者 var copy = Object.assign({},obj);
var copy2 = [].concat(obj);
var copy3 = {...obj};

copy.color = '33'; // 改变原数据第一层属性 A （基本数据类型）
copy.person.name = '44'; // 改变原数据第一层属性 B（引用数据类型）

console.log(obj,copy); // 原数据的引用类型会改变，copy全变
```

深拷贝才是重点：
 `JSON.parse(JSON.stringify())`  (如果原对象中有`undefined、Symbol、函数`时，会导致该键值被丢失，有`正则`，会被转换为空对象`{}`，有`Date`，会被转换成字符串)

`_.cloneDeep`: lodash的Api，直接克隆

```javascript
var new_arr = JSON.parse( JSON.stringify(old_arr) );
```

原理是JOSN对象中的stringify可以把一个js对象序列化为一个JSON字符串，parse可以把JSON字符串反序列化为一个js对象，通过这两个方法，也可以实现对象的深复制，但是这个方法不能够拷贝函数 。

手写一个深拷贝（巩固理解）：
  
```javascript
function cloneDeep(obj){
    if(typeof obj === "object" && obj !== ''){
  return shenCopy(obj);
    }else{
        return obj;
    }
}
function shenCopy(obj){
    let copy =Array.isAraray(obj) ? [] : {};
    for(let p in obj){
  copy[p]=shenCopy(obj[p]);
    }else{
        copy[p]=obj[p]
    }
    return copy;
}
```

![在这里插入图片描述](https://img-blog.csdnimg.cn/6e4808449e2241b7b6fc0947f172d505.png#pic_center)深拷贝是拷贝对象各个层级的属性，看对象的属性是否是对象或数组类型，进行相应属性的一一复制。

## 原型和原型链

#### 原型：每一个对象都会从原型继承属性或方法

官话：每一个JavaScript对象在创建的时候就会**预制管理另一个对象**，这个对象就是我们所说的**原型**。
大白话：
构造函数内部有个prototype 属性，通过这个属性能访问到原型，这里的原型就是指构造函数.prototype 。
前面也说了，原型是预制管理一个对象，那么那个对象是谁呢？其实是构造函数new出来的实例对象，好了，现在有三个角色：构造函数，原型，实例。

它们是怎么联系的呢？官方设定，实例不能直接访问构造函数，要不设计原型干嘛。

```javascript
function Person() {
}
Person.prototype.name="南"
const nan = new Person();
console.log(nan.name); //南
console.log(nan instanceof Person);//True
console.log(nan.__proto__ === Person.prototype);//True 实例通过__proto__访问原型
console.log(nan.__proto__.constructor === Person);//True 实例通过__proto__访问原型后在通过constructor 访问构造函数
console.log(Person.prototype.constructor === Person);//True 原型通过constructor 访问构造函数
```

![在这里插入图片描述](https://img-blog.csdnimg.cn/035f986a6d084b2695d969791bc4db96.png#pic_center)
说的再多不如结合图看遍代码。

构造函数通过原型的方式添加属性，成功调用实例，说明1.实例确实是通过原型访问到构造函数的属性或方法的。2.通过new 出来的实例可以继承原型方法，instanceof 足以说明两者关系。

现在，你知道上述基础类型判断里的`instanceof`和 `constructor`为什么能判断对象类型了吧，还是因为原型！

清楚了原型和实例的关系后，更能透彻理解 new 实例化的过程中发生了什么，具体可见我的js复习下篇。

#### 原型链

每个对象都有属于自己的隐式原型 `__proto__`，每个函数或方法都有自己的显示原型 `prototype` 。实例对象继承使用原型方法的时候通过隐式原型向上查找，如果在原型的显示原型中找到，就不用在通过原型的隐式原型再向上找。

这么说有点拗口，其实蛮好理解的。下面通过一张图说明：

![在这里插入图片描述](https://img-blog.csdnimg.cn/d13db80bdbe0448194e7ecad5a160d4b.png#pic_center)
![在这里插入图片描述](https://img-blog.csdnimg.cn/cd5c1799f16645138ba5413457e3a605.png#pic_center)

如图所示，实例 teacher 要想调用 teach 方法，首先通过显示原型查找有没有 teach 这个方法，发现没有就通过 `_proto_`  隐式原型查找他的上一级原型（Teacher.prototype），看他有没有。Teacher同样先看自己的显示原型，发现有就返回。同理，如果要找 drink 方法，则还要通过 `_proto_`  再向上找原型（Person.prototype）看他有没有，找到就层层返回。
![](https://img-blog.csdnimg.cn/99fe4761307b466f919f42a8a8f29dd0.png#pic_center)
如果找到尽头都没有那个方法，就回返回null。这样，一条完整的方法查找路径就是原型链。

#### ES6新增特性

- let和const（看变量提升，块级作用域，能否重定义）,以及Symbol，Bigint变量。
- 解构赋值：let arr = [1,2,3,4,5]; const [a,b,c,d,e] = arr; 业务中很常用
- 模块化：export default { 暴露的对象 } 导出；  import { a } from ‘暴露的对象’ 导入
- 拓展运算符（...）：可以将多个数组或对象解构形成一个新元素(在去重，浅拷贝，解构很常用) ：
  ​ `const c = [...new Set([...a,...b])];`
- 箭头函数：()=>{} ，箭头函数和普通函数的区别（this，实例化，argument）
- [扁平化数组](https://juejin.cn/post/7016520448204603423#comment)：**Object.values(数组).flat(Infinity)**，Infinity为数组的维度，flat不支持IE浏览器
- 模版字符串：let str3=`我爱中国的${str2}`

- 数组新增的api：原来的：[传送门](https://www.cnblogs.com/obel/p/7016414.html#:~:text=%E6%95%B0%E7%BB%84%E7%9A%84%E6%96%B9%E6%B3%95%E6%9C%89%E6%95%B0%E7%BB%84%E5%8E%9F%E5%9E%8B%E6%96%B9%E6%B3%95%EF%BC%8C%E4%B9%9F%E6%9C%89%E4%BB%8Eobject%E5%AF%B9%E8%B1%A1%E7%BB%A7%E6%89%BF%E6%9D%A5%E7%9A%84%E6%96%B9%E6%B3%95%EF%BC%8C%E8%BF%99%E9%87%8C%E6%88%91%E4%BB%AC%E5%8F%AA%E4%BB%8B%E7%BB%8D%E6%95%B0%E7%BB%84%E7%9A%84%E5%8E%9F%E5%9E%8B%E6%96%B9%E6%B3%95%EF%BC%8C%E6%95%B0%E7%BB%84%E5%8E%9F%E5%9E%8B%E6%96%B9%E6%B3%95%E4%B8%BB%E8%A6%81%E6%9C%89%E4%BB%A5%E4%B8%8B%E8%BF%99%E4%BA%9B%EF%BC%9A%20join%28%29%20push%28%29%E5%92%8Cpop%28%29%20shift%28%29%20%E5%92%8C%20unshift%28%29%20sort%28%29%20reverse%28%29,concat%28%29%20slice%28%29%20splice%28%29%20indexOf%28%29%E5%92%8C%20lastIndexOf%28%29%20%EF%BC%88ES5%E6%96%B0%E5%A2%9E%EF%BC%89%20forEach%28%29%20%EF%BC%88ES5)     新增:(includes，startsWith，endsWith）

- 双问号`??`：作用就是判断该对象有没有默认值，没有就采取问号后面的值，比如输入框非空判断`if((value??'') !== '')`，`?.`则是判断该对象有无该属性或方法 `obj?.eat`,没有就返回undefined，不会报错，在业务中算是常用），（在TS中，还有`!.`就告诉编译器，一定有这属性，别给我报这属性可能不存在的错）
