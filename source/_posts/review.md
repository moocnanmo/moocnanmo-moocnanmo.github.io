---
title: 总结一些良好的编码规范
cover: https://img-blog.csdnimg.cn/5a6a1c3011a94bc1b856c77ca9a12492.png
description: 热更新中...
categories: '烂笔头经验记录'
---


> 持续更新，有点小优化就写进来

### 一：解构请求数据和提前结束判断条件

之前是用对象接收，什么都要 `Obj.` 点出来，且没做请求失败后，后台没给失败的错误信息处理

```javascript
//之前写法，获取数据后根据code响应不同信息
//let result = request await GetFormData(xxx);
//result.code === 0 ? success : error 

//不使用三元，一般情况会写if else判断
//但其实可以少些个else(除去这种可能，剩下的一定是那种，所以可以少写层判断)
let { code, message, data } = request await GetFormData(xxx);
if( code !== 0 ){
    error("message") ?? "请求失败";
    return false;
}
A = data.A;
```

还有个小技巧，也是在根据请求结果对需要响应的Flag赋值

`isShowFlag.value = code === 0;`

之前写法： `code === 0 ? isShowFlag.value = true : isShowFlag.value = false`

### 二：嵌套解构

只是在原先基础上，做了下懒得点出来的优化

```javascript
const obj = {
  study: {
    js: 'promise',
    Vue: 'vite'
  }
}
//传统写法：console.log(obj.study.js) 
const obj = {
  study: {
    js:2,
    Vue:{
   vite:{ 
    node:111
   }
  }
  }
};
const { study: {Vue: { vite } } } = obj ;
console.log(vite ) // node:111
```

### 三：条件多的时候合理使用map对象或includes

假如有这么一个情景，头部的Tip文案样式会根据提交结果响应不同样式，有success，warning，error等样式 ，可以用map优化

```javascript
//冗余写法
 const getResult = (data) => {
     if(data === "0"){
       return "success"
     }else if(data === "-1"){
       return "error"
     }else if(data === "-2"){
       return "fail"
     }
 }
 console.log(getResult("-2"));

//改良
const map = {
  '0': 'success',
  '-1': 'error',
  '-2': 'fail',
  // ...
}
console.log(map["0"]);
```

像这样条件比较多，存在一对一的关系，列举太多if else很不雅，可以用 map 来进行映射

如果需要从众多条件中寻求是否符合的变量，可能第一想法会是for遍历，if条件判断在返回有无这个结果

```javascript
let isTrueFlag = false;
const arr = ["red","white","yellow","black","blue"];
for(let a=0; a < arr.length; a++){
  if(arr[a] === "pink" ){
    isTrueFlag = true;
  }
}

//改良
trueFlag = arr.includes("pink")
```

有些还得动态监听，可以在前  `includes`  加  `computed`  存储下来在选择

### 四：用ES6的解构进行浅拷贝,以及输入项的非空判断

之前都是用 `Object.assign(target, ...sources)`， `Array.concat()` , 来浅拷贝，现在有了ES6的解构来`{ ...obj }` 来进行浅拷贝，简直不要太方便

```javascript
//浅拷贝原始写法
const copy1 = [];
Object.assign(copy,obj);//或者 var copy = Object.assign({},obj);
let copy2 = [].concat(obj);

let copy3 = {...obj};

//非空判断原始写法
if(typeof value !== null && typeof value !== undefined && value !== ''){}

//至于为什么不写(!value),是因为能通过undefined，null和 “ ”。
if((value ?? '') !== ''){}
```

### 五：在Vue2一些死数据不需要放在return 返回，做响应式

之前放在data里返回，浪费性能，完全不需要

```javascript
  data() {
    this.selects = [
      {label: '选项一', value: 1},
      {label: '选项二', value: 2},
      {label: '选项三', value: 3}
    ]
    return { };
  }
```

### 六：可选属性和必有属性

在Typescript里差不多，？可选，! 必有此属性.

```javascript
//之前写法
const name = data && data.name;

const name = data?.name;
const name = data!.name;
```

### 七：class属性绑定

可以根据响应式或函数式进行渲染

```html
<div class="a b"  :class{c:true , d:refFlag}> </div>

<div class="a b"  :class=“changeClass()”> </div>
```

### 八：数组扁平化

有时候后台会返回这么一个对象数组，你恰好有需要将他属性值统一取出，之前我是这么写

```javascript
const resultData= {
'a':[1],
'b':[1,2],
'c':[3]
}
let newArray= [];
for (let item in resultData){
    const value = resultData[item];
    if(Array.isArray(value)){
        member = [...newArray, ...value]
    }
}
newArray= [...new Set(newArray)]
```

这情形， `for in`  完全可以用  `Object.values` 代替，后面扁平化 直接  `flat` (默认递归深度为1层，使用 Infinity，可展开任意深度的嵌套数组) 一气呵成：
`let newArray= Object.values(resultData).flat(Infinity);`

### 九：善用filter，map，find/findIndex

假如有这么个对象数组，你要寻找其中特定id的所有名字对象。

虽然for if是最容易想到也是最常用的方法，但看起来不够精简和优雅。

```javascript
const map = [{name: "笔记本",id: 1},{name: "电子书",id: 2},{name: "手机",id: 1},{name: "鼠标",id: 0}];
let data = [];
for(let a=0; a < map.length; a++){
 if(map[a].id === 1){
  data1.push(map[a].name)
 }
}
console.log(data);
```

如果使用filte过滤一遍在map遍历，是不是更直观，更优雅一点呢

```javascript
const data2 = map
  .filter(i => i.id === 1)
  .map(i => i.name)
```

可能怎么写直观的看不出太大的变化，觉得这优化可有可无

那我们换个需求，就找特定id的子项

这时你最先想到的是不是用find，而不是 for if 判断来找

```javascript
const data = () => {
 for(let a=0; a < map.length; a++){
  if(map[a].name === "手机" && map[a].id === 1){
   return map[a]
  }
 }
}

const data = map.find(i => i.name === "手机" && i.id === 1)
```

找到对应子项下标同理，我们都会第一时间想到findIndex而不是for if 判断

所以，这也算是良好的编码习惯，再有现成API的情况，少用 for if 。

