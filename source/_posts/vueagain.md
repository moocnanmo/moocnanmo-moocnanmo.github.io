---
title: 小南再谈Vue（Q&A）
data: '2021-11-18 11:41:01'
cover: https://cn.bing.com/th?id=OHR.HyacinthMacaws_ZH-CN1191345036_1920x1080.jpg
description: 之前看着文档，迷迷糊糊的学了一遍Vue，现在有了些自己的理解，就在梳理了一遍，但自认为目前还是小白，不是很懂，理解大都来自自学，如有不当，恳求指出。🥳🥳🥳 #文章描述，只显示该处内容
categories: '烂笔头经验记录'
---

> 之前看着文档，迷迷糊糊的学了一遍Vue，现在有了些自己的理解，但自认为目前还是小白，不是很懂，理解大都来自自学，如有不当，恳求指出。🥳🥳🥳

附个链接，[之前的vue梳理 - 上](https://blog.csdn.net/takeingloop/article/details/118057942?spm=1001.2014.3001.5501)，[之前的vue梳理 - 下](https://blog.csdn.net/takeingloop/article/details/118069182?spm=1001.2014.3001.5501)。

-----

## vue 的生命周期

简单来说就是创建 -> 挂载DOM -> 更新数据 -> 销毁的这么一个过程，按自己理解的话，可大致理解为：

- beforeCreate：vue实例的挂载元素el和数据对象data都还没有进行初始化，还是一个undefined状态；
- created: 此时实例的数据对象data已经有了，可以访问里面的数据和方法，el还没有，也没有挂载dom（一般在这时就可以绑定事件处理函数了）；
- beforeMount: 此时el和数据对象都有了，只不过在挂载到虚拟的dom节点上；
- mounted: vue实例已经挂在到真实的dom上，可以操作dom节点；
- beforeUpdate: 这时响应式数据更新开始调用，（数据是新的，页面是旧的）
- updated：数据更新完成，新的dom已经更新；
- beforeDestory： vue实例在销毁前调用，在这里还可以使用；
- destoryed：所有事件监听器会被移除；
![在这里插入图片描述](https://img-blog.csdnimg.cn/1a8a5b5cd3bb4657bfa91d6441a9f5e5.png#pic_center)

## 什么是 MVVM，原理

MVVM分别代表：model数据源，view为组件视图，viewModel将model和view关联起来，是一个层级模型，简单理解就是V和M的联调器；

**响应式原理** ：实际就是通过 Object.defineProperty方法，对于实例数据被读取就会调用get方法，当修改数据时就自动调 set 方法，检测到数据更新了通知**观察者Watcher**，生成新的Dom树，对比新旧DOM结点，通知对应节点进行数据新。

**其实是这么个情况：**  M(data)要该改变V(template) 或者V更改M数据同步更新，是通过VM的双向数据绑定操作操作的，V和M对VM都是双向通道，VM 通过双向数据绑定把 V 层和 M 层连接了起来，所以可以同步更新。

简单说就是：**数据变化更新视图，视图变化更新数据**。

提一句：
单向绑定 `v-bind`：数据只能从data流向页面;
双向绑定 `v-model`：数据可以从data流向页面，也可以从页面流向data;

## Vuex 是什么

还没怎么用过，大致理解为：专为 Vue.js 应用程序开发的**状态管理模式**，在中大型项目中使用；

## v-show 和 v-if 有什么区别

`v-if是操作DOM元素是否删除来显示隐藏的，这有可能会引发重绘或重排的问题。`
`v-show则是不管条件怎样都会通过display：是否为none ，来渲染DOM进行隐藏显示，这只是对css进行操作，没有删除DOM`

所以，根据`v-if` 会删除DOM元素特性可知，这适用于不需要频繁切换条件的场景。
`v-show` 则适用于需要频繁切换条件的场景。

## 为什么 v-for（必须要有key） 和 v-if 不建议用在一起

可以那样使用，但不建议，因为for的优先级比if高，如果if要筛选的数据比较少，但要遍历的数组很大，就要都循环，这就造成性能浪费，如果条件可以，建议用computed；

```html
  <div v-for="(item,index) in itemList" v-if="condition"></div>
```

像这个例子，是先进行遍历在判断，但实际业务情况往往是下面这种写法，先判断是否存在在遍历。

```html
 <template v-if="condition">
     <div v-for="(item, index) in itemList"></div>
</template>
```

## vue2 组件传值

**1.父子传值 props**

父组件引入子组件后，在data中定义所传参数，在子组件标签直接传递，子组件通过props接收即可

```javascript
 <!-- 父组件 -->
<div class="hello">
   <h1>父组件</h1>
   <sonVue :msg="msg"></sonVue>
</div>

import sonVue from "./son.vue";
export default {
  data () {
    return {
      msg: 11
    };
  },
  components: {
    sonVue
  }
};

 <!-- 子组件 -->
{{ msg }}

export default {
   // 1.以数组方式接受
  // props: ["msg"],
    
   // 2.以对象形式接收
  props: {
    msg: {
      typeof: Number,
      default: 3
    }
  }
}
```

**2.子父传值：$emit（'事件名'，'数据'）**

在传值组件定义个自定义事件名称，接收组件通过@接收在触发即可

```javascript
//传值组件
<button @click="sayLove">emit子向父传值</button>
  
methods: {
  sayLove () {
    this.$emit('sayLove', "生日快乐")
  }
}

//接收组件
<sonVue :msg="msg" @sayLove="getLove"></sonVue>
    
getLove (e) {
  console.log(e);
}
```

**3.兄弟传值：在Vue原型上再定义个新实例，通过$emit和$on传递**

![在这里插入图片描述](https://img-blog.csdnimg.cn/19b50440d9c74dc8961770da0d81d13d.png)

```javascript
//1.新定义个实例（好几种方法）
// 方法一
// 抽离成一个单独的 js 文件 Bus.js ，然后在需要的地方引入
import Vue from "vue"
export default new Vue()

// 方法二 在main.js直接挂载到全局
Vue.prototype.$bus = new Vue()

// 方法三 在main.js注入到 Vue 根对象上
import Vue from "vue"
new Vue({
    el:"#app",
    data:{
        Bus: new Vue()
    }
}).$mount('#app')

//2.在需要传值的组件上通过$emit事件传参
this.$bus.$emit('sayEat', "吃饭")

//3.接收组件,通过on方法接收自定义事件
created () {
    this.$bus.$on('sayEat', this.getSonLove)
  },
  methods: {
    getSonLove (ee) {
      console.log(ee);
    }
  }
```

**4.爷孙传值provide / inject**

`provide`：可以让我们指定想要提供给后代组件的数据或方法

`inject`：在任何后代组件中接收想要添加在这个组件上的数据或方法

但是！！！

在传值组件中，provide:{   }这样使用不能获取 methods 中的方法，provide(){  return{}  }使用不能获取data中的属性。

```javascript
// 1.在传值组件中
export default{
    // 方法一 不能获取 methods 中的方法
    provide:{
        name:"nanmo",
        age: this.data中的属性
    },
    // 方法二 不能获取 data 中的属性
    provide(){
        return {
            name:"nanmo",
            someMethod:this.someMethod
            //也可以将爷组件整个传入,但不推荐：father:this
        }
    },
    methods:{
        someMethod(){
            console.log("传个方法")
        }
    }
}

// 后代接收数据组件
export default{
    inject:["name","someMethod"],
    mounted(){
        console.log(this.name)
        this.someMethod()
    }
}
```

**4.slot插槽传值了，hhhh**

引入插槽后直接点出插槽数据即可

```javascript
// Child.vue
<template>
    <div>
        <slot :user="user"></slot>
    </div>
</template>
export default{
    data(){
        return {
            user:{ name:"nanmo" }
        }
    }
}

// Parent.vue
<template>
    <div>
        <child v-slot="user">
            {{ user.name }}
        </child>
    </div>
</template>
```

方法肯定不止上述这些，笔者只是写了日常中比较常用的方案，更多请移步掘金。

## vue3.2 组件传值

**1.父子传值，props**

很重要的一个API，贴个超链，[官方介绍](https://v3.cn.vuejs.org/guide/component-props.html)

跟vue2差不多，只是语法变了，在 `<script setup>` 中必须使用 `defineProps` API 来声明 `props`，不要要额外引入。

```vue
//父组件
<template>
  <Child :msg="message" />
</template>

<script setup>
import Child from './components/Child.vue'
let message = 'nanmo'
</script>

// 子组件
<template>
    {{ msg }}
</template>

<script setup>
const props = defineProps({
  msg: {
    type: String,
    default: ''
  }
})
console.log(props.msg) // 在 js 里需要使用 props.xxx 的方式使用。在 html 中使用不需要 props
</script>
```

**2.子父传值，emits**

还是跟vue2差不多，以上述为例子

```javascript
<div>父组件：{{ message }}</div>
<Child @changeMsg="changeMessage" />
    
let message = ref('nanmo')
function changeMessage(data) {
  message.value = data
}

子组件：<button @click="handleClick">子组件的按钮</button>

const emit = defineEmits(['changeMsg'])
function handleClick() {
  emit('changeMsg', '南漠')
}
```

这里可以用 `v-model` 语法糖实现相同效果,我觉得其本质还是使用了emits传值

```javascript
<Child v-model="message" />

//子组件
<div @click="handleClick">{{modelValue}}</div>

//1.接收父组件使用 v-model 传进来的值，必须用 modelValue 这个名字来接收
const props = defineProps([
  'modelValue' 
])

//2.必须用 update:modelValue 这个名字来通知父组件修改值
const emit = defineEmits(['update:modelValue']) 

function handleClick() {
  emit('update:modelValue', '南漠')
}

//更简单的还能这样,传值过程直接卸载标签内
<div @click="$emit('update:modelValue', '南漠')">{{modelValue}}</div>

const props = defineProps([
  'modelValue' 
])
```

**3.子父传值，expose / ref**

这个比较简单，一个把数据暴露出去，一个ref拿就好

子组件可以通过 `expose` 暴露自身的方法和数据，父组件通过 `ref` 获取到子组件并调用其方法或访问数据。

是不是有点像vue2的 `provide / inject`  , 哈哈

```javascript
<div>父组件：拿到子组件的message数据：{{ msg }}</div>
<button @click="callChildFn">调用子组件的方法</button>
<Child ref="com" />

const com = ref(null) // 通过ref拿到子组件实力，后续.出数据来就行
const msg = ref('')
onMounted(() => {
  msg.value = com.value.message
})

//子组件使用 defineExpose 向外暴露指定的数据和方法，一样defineExpose 不需要额外引入
defineExpose({
  message,
  changeMessage
})
```

**4.Vue3当然可以使用插槽，用法和Vue2一样**

**5.provide / inject，用法和Vue2一样**

**6.总线Bus，用法和Vue2一样**

## computed（计算属性） 和 watch （监视属性）区别

computed：要用的属性不存在，要通过已有属性计算得来，他底层借助了Object.defineProperty方法提供的getter和setter，如果computed的内容用于v-model的话，必须要有set和get 函数，否则会报错！
watch：当被监视的属性变化时，回调函数自动调用，进行相关操作；

`computed 有缓存，且依赖其他属性值，其他属性值不变的话，computed内容不会重新计算`
`watch 没有缓存，一般用于数据监听回调；`
可以这么记：computed所完成的功能，watch也可以完成，反之不然。

## v-bind(: )、v-model 和 v-on(@) 的区别

v-bind 一般用来设置HTML的属性和传递数据，[看官方文档的例子吧](https://cn.vuejs.org/v2/api/#v-bind)
 v-model 一般在表单中使用，实现双向数据绑定的，在表单元素外不起使用，语法糖
 v-on 绑定事件监听器。

## 消息订阅

发布订阅模式，一种组件间通信的方式，适用于任意组件间通信；
可以想象成食堂打饭，打饭的同学是订阅方，打菜阿姨是发布方。

```javascript
//安装pubsub
npm i pubsub-js
//引入
import pubsub from "pubsub-js"
//接收数据：订阅方  发布数据：发布方
methods(){
    demo(data){...}
    mounted(){
        this.pid = pubsub.subscribe("xxx",this.demo)//订阅消息
    }   
}
//提供数据
pubsub.publish("xxx",数据)
//最好在beforeDestroy构子中，用Pubsub.unscrible(pid)去取消订阅
```

## vue脚手架配置代理

方法一：一种简单的配置代理方法，默认端口8080；不能配置多个，如果请求了前端不存在的资源时，那么该请求会转发给服务器

```javascript
//在vue.config.js中添加如下配置：
devServer:{
    proxy:"http://localhost:5000"
}
```

方法二：编写vue.config.js配置具体代理规则：

```javascript
devServer:{
    proxy:{
        "/api1":{//匹配所有/api1开头的请求路径
            target:"http://localhost:3000",//代理目标的基础路径
            changeOrigin:true,
            pathRewrite:{"^/api1":""}
        }
    }
}
//changeOrigin设置为true ? 服务器收到的请求头中的host为：localhost:3000否则为8080
//changeOrigin默认值为true
```

## 插槽

**基本使用：** 写个插槽文件，在需要的地方import 插槽名字 from 路径 导入，slot的内容会被更新。
**v-slot:** 可以使用#替换

```javascript
<!-- MySlotCpn组件 -->
<template>
  <div class="my-slot-cpn">
    <h1>组件开始</h1>
    <slot>默认内容</slot>
    <h1>组件结束</h1>
  </div>
</template>

<!-- App组件 -->
<template>
  <div class="app">
    <MySlotCpn>
      <i>Hello Vue</i>
      <div>Hello Vue</div>
    </MySlotCpn>
  </div>
</template>
```

![在这里插入图片描述](https://img-blog.csdnimg.cn/b7fb7d1b534542d1a5a7be95a0908342.png#pic_center)
**具名插槽：** 在插槽文件设定name，引用时v-slot:对应名字即可。
一个不带 name 的 slot，会带有隐含的名字 default

```javascript
<!-- NavBar组件 -->
<template>
  <div class="nav-bar">
    <div class="left">
      <slot name="left"></slot>
    </div>
    <div class="center">
      <slot name="center"></slot>
    </div>
    <div class="right">
      <slot name="right"></slot>
    </div>
  </div>
</template>

<!-- App组件 -->
<template>
  <div class="app">
    <NavBar>
      <template v-slot:left>
        <button>button</button>
      </template>
      <template v-slot:center>
        <span>span</span>
      </template>
      <template v-slot:right>
        <i>i</i>
      </template>
    </NavBar>
  </div>
</template>
```

![在这里插入图片描述](https://img-blog.csdnimg.cn/d792ba97e00d4f4fb0f9ae55cd81eb93.png)**动态的插槽名：** 给name绑定：，引用时在通过 [] 解构

```javascript
<!-- 组件里的name是通过props动态绑定的 -->
<slot :name="name"></slot>

<!-- 通过[]解构 -->
<template v-slot:[name]></template>

```

还有个**作用域插槽**
大体就是，父组件想用插槽组件的数据，可以在调用的时候通过

```javascript
//插槽组件,将数据传递给调用方
<slot :user = "use.name">

//调用组件,可以解构，可以自定义接受名称，在.出来
<diaoyong>
 <template v-slot:default = "{user}">
  {{ user.name }}
 </template> 
</diaoyong>
```

## 路由

首先前端路由有 **hash** 和 **history** 两种模式，最直观的区别就是hash的URL中有丑丑的#符号。

**hash模式**：hash 虽然出现在 URL 中，但不会被包括在 HTTP 请求中，对后端完全没有影响，因此改变 hash 不会重新加载页面。（本地项目启动，通常是hash模式）

**history 模式**：利用了 HTML5 History Interface 中新增的 `pushState()`和 `replaceState()` 方法，这两个方法应用于浏览器的历史记录栈，在当前已有的 `back`、`forward`、`go`的基础之上，它们提供了对历史记录进行修改的功能。只是当它们执行修改时，虽然改变了当前的 URL，但浏览器不会立即向后端发送请求。

![在这里插入图片描述](https://img-blog.csdnimg.cn/d0a20c365fd248219b0d12d13566c621.png#pic_center)
浏览器url信息一般存储在 `window.history.state` 中,
下面说下history模式的 `history.pushState()` 方法，它相较于hash直接修改#后面参数具有以下优势：

 1. hash模式只能修改# 后面的部分，因此只能设置与当前 URL 同文档的 URL，而history则可以修改任意同源URL；
 2. 新 URL 可以与当前 URL 一模一样，这样也会把记录添加到栈中；而 hash 设置的新值必须与原来不一样才会触发动作将记录添加到栈中；
 3. pushState() 可额外设置 title 属性供后续使用；

```javascript
//下载 npm vue-router
//应用 Vue.use(Router)
//编写配置项 
import Router from "vue-router"
import Login from "需要跳转的静态文件地址"
import Home from "..."
const router = new VueRouter({
    routes:[
      { path: '/', redirect: '/login' },
      {
       path: '/home',
      component: Home,
       redirect: '/welcome',
       children: [
        { path: '/welcome', component: Welcome },
        { path: '/users', component: Users },
        { path: '/rights', component: Rights },
    ]
})

//路由导航守卫
router.beforeEach((to, from, next) => {
  if (to.path === '/login') return next()
  // 获取token，如果没有token就返回到登陆界面
  const tokenStr = window.sessionStorage.getItem('token')
  if (!tokenStr) return next('/login')
  next()
})

//暴露router
export default router
```

值得注意的是，路由组件通常存放在pages文件夹，一般组件通常存放在components文件夹。多级路由使用children配置项，

### ​路由的query参数

```javascript
//跳转并携带query参数，to的字符串写法
<router-link :to="/home/message/detail?id=666&title=你好">跳转</router-link>
//跳转并携带query参数，to的字符串写法
<router-link
    :to="{
        path:"/home/message/detail",
        query:{
            id:666,
            title:"你好"
        }
    }"
>跳转</router-link>

//接收参数：
$router.query.id
$router.query.title
```

### 路由命名

在多级路由跳转的时候，可以name:XX对路由命名
`children: [ { path: '/welcome', component: Welcome,name:good } ]` 
然后通过命名跳转： `router-link :to="{name:'hello'}">跳转</router-link>`

### params参数

跟命名差不多，在 `children: [ { path: 'detail/:id/:title', component: Welcome,name:good } ]` path中声明占位符接收params参数，然后 通过下例跳转。

```html
//传递参数
<router-link
    :to="{
       name:"xiangqing",
        query:{
            id:666,
            title:"你好"
        }
    }"
>跳转</router-link>

//接收参数
$routr.params.id
$route.params.title
```

另外：路由携带params参数时，若使用to的对象写法，则不能使用path配置项，必须使用name配置项。

除此之外，还有**​编程式路由导航**，可不借助<router-link>实现路由跳转，让路由跳转更加灵活.

### 编程式路由导航

```javascript
//$router的两个API
this.$router.push({
    name:"xiangqing",
    params:{
        id:xxx,
        title:xxx
    }
})
this.$router.replace({
    name:'xaingqing',
    paarams:{
        id:xxx,
        title:xxx
    }
})
this.$router.forward()//前进
this.$router.back()//后退
this.$router.go()//可前进可后退

```

### ​缓存路由组件

他可以让不展示的路由组件保持挂载，不被销毁。

```html
<keep-alive>
    <router-view></router-view>
</keep-alive>
```

## 路由懒加载

如果不懒加载，打包后文件很大，进入首页需要加载的东西很多。
将路由对应的组件分割成不同的代码块，访问时加载对应组件。

```javascript
//传统
//import Login from '../components/Login.vue'

//懒加载
//方法1,就不需要import导入了，直接在路由规则写入
export default new Router ({
routes: [
    {
     path: '/login',
     name: 'Login',
  component:resolve require([ '@/components/Login'],resolve )
  },
  ]
})

//方法2,安装babel插件，从传统的import导入变成const 形式，可以对类型相同的路由分组。
const Home = () => import(/* webpackChunkName: 'Login' */ '@/components/Login')
```

## ref属性

他是用来给组件注册引用信息的，应用在html上是获取DOM对象，应用在组件上是获取实例对象。
使用： `<h1 ref="xxx"></h1>或<school ref="xxx"></school>`
获取： `this.$refs.xxx`

## props配置项

一般用于父子传值，让组件接受外部传过来的数据： `<Demo name="xxx">`
接受数据：

 1. 只接收  `props:[]`
 2. 第二种方式（限制类型）：`props:{name:String}`
 3. 第三种方式（限制类型、限制必要性、指定默认值）：`props:{ xxx:{type:String,  required:true, default:"xxx" }}`

## 组件的自定义事件（子 ->父通信）

 1. 使用场景：A是父组件，B是子组件，B想给A传数据，那么就要在A中给B绑定自定义事件，事件的回调在A中

 2. 绑定自定义事件：
     2.1  在父组件中：`<Demo @自定义事件名="回调函数名">`;
     2.2  在父组件中：
        ```javascript
        <Demo ref="子组件名">
        mounted(){
            this.$refs.子组件名.$on("自定义事件名",this.回调函数名)
        }
```
      2.3  若想让自定义事件只触发一次，可以使用 `once` 修饰符，或 `$once` 方法

 3. 触发自定义事件： `this.$emit("自定义事件名",数据)`

 4. 解绑自定义事件： `this.$off("自定义事件名")`

这里顺带提一下**事件修饰符**：prevent:阻止默认事件，stop：阻止事件冒泡事件，capture：使用事件的捕获模式；
再说下**事件基本使用注意事项**吧：事件的回调如果配置在methods对象中，最终会在vm上（this指向vm或组件实例），在methods使用箭头函数的话，this就不是vm了；


## 聊下vite打包
鉴于vite的启动速度和热更新，现在写demo用vite创建了，简直不要太香，速度嘎嘎的。但为什么相较于cli脚手架，webpack这些，速度提升那么多，好奇之余查了下，主要的原因是浏览器以前不支持`ES module`，现在支持，vite利用这一特性（给script 标签加上 `type="module"` 属性），将打包的活一部分交给交给浏览器，分担了压力。

vite主要对**依赖**和**源码**进行打包：**依赖**使用Go语言编写 的`esbuild` 进行**预构建**，以及将零散的文件整合到一起，减少网络请求和配置 ESM 模块语法来契合浏览器支持。**源码**的话采用按需加载的，这样的好处是用到的组件模块才进行打包。

热更新块是因为，Vite 同时利用 HTTP头来加速整个页面的重新加载，**源码**的请求会进行协商缓存，而**依赖**则会通过设置 `Cache-Control` 进行强缓存，所以项目一旦被缓存下来，下次相同资源将不需要再次请求，所以热更新秒快。

vite在开发模式下广受好评，但在生产模式下，有两种声音，一是有很多小问题，不好配置，然后求稳选择webpack；二是拥抱新技术，还好，大项目都在用，每没出现啥大问题，具体啥情况，我不清楚但保持乐观的态度 ...


> 结束语：加油！努力沉淀变成光吧！
