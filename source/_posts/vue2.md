---
title: Vue结合文档的笔记整理-下篇
cover: https://img-blog.csdnimg.cn/202108142021464.jpg
description: 接着昨晚落下的，还剩下 组件化的开发 ，前后端交互，和一些路由知识，今天就慢慢写吧。 #文章描述，只显示该处内容
categories: '烂笔头经验记录'
---

# Vue笔记整理：结合官方文档的专项复习-下篇


--------

接着昨晚落下的，还剩下 **组件化的开发** ，前后端交互，和一些路由知识，今天就慢慢写吧。还是老规矩，看着之前学习敲的代码，想知识点，不清楚或模糊的我在结合官方文档在捋一捋，有些学起来不是很清楚的知识点就不往上写了，私底下看大佬博客自行解决，废话有点多，现在开始吧。

想接着看 **上篇**   的戳 [传送门](https://blog.csdn.net/takeingloop/article/details/118057942?spm=1001.2014.3001.5502)，顺带贴张可爱的喵喵。
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210620141719638.jpg?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3Rha2Vpbmdsb29w,size_16,color_FFFFFF,t_70#pic_center#pic_center =300x300)
### 三、 组件化的开发
##### 3.1、 全局注册和局部注册
[组件基础-官方文档](https://cn.vuejs.org/v2/guide/components.html)  ，作为Vue最强大的功能之一，学习它着实用了我不少时间。我理解的组件为：类似于页面中导航栏，回到顶部的模块功能，它把HTML内容结构，CSS外观样式，JS动画特效，封装在一起成为的一个功能，就叫组件。这样在开发项目时，哪需要直接引入即可。下面看个 **全局组件** 的基本样例模版：

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Title</title>
    <script src="vue.js"></script>
</head>
<body>
<div id="app">
<!-- 2. 调用组件：【注意】这里的组件可任意次复用，也就是说每个组件相互独立互不影响的 -->
    <mynav></mynav>
    <mynav></mynav>
    <mynav></mynav>
</div>

<script>
    // 1. 通过vue类注册组件，“mynav”为自定义标签名
    Vue.component("mynav", {
     // 模版必须是单个根元素
        template:`<div>
                    <button @click="sub">-</button>
                    <input type="text" v-model="num">
                    <button @click="add">+</button>
                   </div>`,
        data(){  // 组件里面的变量数据，【注意】data返回的必须是一个函数，正因为这条规则，上述的组件才相互独立
            return {
                num: 0
            }
        },
        methods:{ // 组件里面的方法
            sub(){
                this.num--;
            },
            add(){
                this.num++;
            }
        }
    });
     // 创建根实例
    var vm = new Vue({
        el:'#app',
        data:{},
    })
</script>
</body>
</html>
```


为了能让Vue识别和在模版中使用组件，所以组件分为全局和局部，全局通过`Vue.component`注册，局部通过`new Vue`注册，这里面有很多小细节需要注意，比如上述的全局组件（data返回值，组件模版要求，甚至组件名命名使用的限制），下面再说 **局部注册** ：

```html
  <div id="app">
      <my-component></my-component>
  </div>

<script>
    // 定义组件的模板
    var Child = {
      template: '<div>局部注册</div>'
    }
    new Vue({
      //局部注册组件  
      components: {
        // <my-component> 将只在父模板可用  一定要在实例上注册了才能在html文件中使用
        'my-component': Child
      }
    })
 </script>
```
局部注册组件只能在当前注册它的vue实例中才能使用，不过值得注意的是，**局部注册的组件在其子组件中不可用** ，怎么理解这句话，就是说如果你定义了好几个局部组件，组件间是不通的，像局部函数间不能直接访问一样，要通的话得换个写法，如

```javascript
var ComponentA = { /* ... */ }

var ComponentB = {
  components: {
    'component-a': ComponentA
  },
  // ...
}
```
各不相同的组件中有着父子，兄弟之类关系，这使得在数据交互方面得分门别类，下面将说道父组件，子组件，兄弟组件俩俩之间的传值。
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210620160423193.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3Rha2Vpbmdsb29w,size_16,color_FFFFFF,t_70#pic_center)
##### 3.2、 父组件向子组件传值
整过过程大致为：1.子组件内部通过 `props` 接收父组件传递过来的值； 2：父组件通过属性将值发给子组件。传递的值可以是数字、对象、数组等等。

```html
  <div id="app">
    <div>{{pmsg}}</div>  
    <menu-item title='来自父组件的值'></menu-item>
    <!-- 2、 需要动态的数据的时候 需要属性绑定的形式设置，此时 ptitle  来自父组件data 中的数据. -->
    <menu-item :title='ptitle' content='hello'></menu-item>
  </div>

  <script type="text/javascript">
    Vue.component('menu-item', {
      // 3、 子组件用属性props接收父组件传递过来的数据  
      props: ['title', 'content'],
      data: function() {
        return {
          msg: '我是子组件'
        }
      },
      template: '<div>{{msg + "----" + title + "-----" + content}}</div>'
    });
    var vm = new Vue({
      el: '#app',
      data: {
        pmsg: '我是父组件',
        ptitle: '动态数据'
      }
    });
  </script>
```
上述父组件向子组件传递了静态的数据，动态的数据（content）。

【注意】：在使用 `props` 命名时，在 `#app`  标签下必须使用短横线的命名规则，而在组件模版 `template` 下短横线驼峰都可以，注意双向命名相同。



##### 3.3、 子组件向父组件传值
整个过程大致为：

 1. 子组件用`$emit()`触发事件 
 2. `$emit()`  第一个参数为自定义的事件名称（传给父组件），第二个参数为需要传递的数据
 3. 父组件用 `v-on` 监听子组件的事件。v-on:事件名称='事件处理逻辑'



##### 3.4、 非父子组间传值（兄弟关系）
与上述不同，该类交互需要事件中心管理，如A要同兄弟B通信，需要借助时事件中心管理器作为中介，当它被触发后负责监听另一方的通信。整个过程大致为：
1.创建一个事件中心实例  `var hub = new Vue();`
**2.监听事件**  ` hub.$on('事件名称',事件函数)` , 销毁事件:`hub.$off('事件名称');`
**3.触发事件** ` hub.$emit('事件名称', id);;`可以携带参数



##### 3.5、 组件插槽

[插槽-官方文档](https://cn.vuejs.org/v2/guide/components-slots.html) ，缩写 "**#**" , 简而言之就是父组件向子组件传递内容（模版的内容）：子组件会有一个插槽 `<slot></slot>` ,用来接收父组件标签的内容。基本用法大致为：

 1. 插槽在 `template` 下装入 `<slot>默认内容</slot>` 插槽；如果有默认内容的话会直接输出到插槽上
 2. 要插入的标签的内容：  `<alert-box>要插槽的内容</alert-box>`

插槽还可赋予名字，用法和上述有些许差别，用法如下：

```html
 <div id="app">
    <cc>
      <p slot='header'>头部信息</p>
      <p>匿名默认内容</p>
    </cc>
 </div>
 Vue.component('cc', {
      template: `<div>
            <slot name='header'></slot>
            <slot></slot>
        </div>
      `
    });
```
这块比较简单，注意下插槽编译规则：子插槽无法访问父插槽的作用域，这里无论是父插槽还是子插槽，他们的内容只在自身级别编译。
大致说了下组件化开发的一些零碎知识点，感觉差不多了，那就下一个，前后端交互吧

-----
### 四、 前后端交互
我理解的前后端交互模式是这样的：前端发请求调用后台接口，获取到服务器端的数据。这里的接口调用大都为异步操作，传统使用AJAX，现使用Promise进行异步的调用（fetch、axios、async/await等）。**fetch** 可以理解为jQuery ajax 的升级版，**axios**为第三方的库，可以更简单方便的调用接口。

其中，客户端要想想服务器端发送请求，就得借助URL上传到互联网，借助互联网到达服务器端，服务器端拿到请求后，响应一个HTML页面或数据。客户端可在地址请求中完成对数据的增(POST)删(DELETE)查(GET)改(PUT)操作。

##### 4.1、 Promise基本用法

**Promise** 是异步编程的一种解决方案，其基本用法为：首先实例化一个 `Promise` 对象，然后在构造函数的参数中传入一个函数，这函数用来处理异步任务。异步结果通过传入的函数的两个参数(`resolve`,`reject`)处理成功失败两种情况，使用 `p.then` 调用。大致为：

```html
  <script type="text/javascript">
    var p = new Promise(function(resolve, reject){
      setTimeout(function(){
        var flag = true;
        if(flag) {
          //成功情况
          resolve('hello');
        }else{
          //失败情况
          reject('出错了');
        }
      }, 100);
    });
    p.then(function(data){
      //打印成功信息
      console.log(data)
    },function(info){
     //打印失败信息
      console.log(info)
    });
  </script>
```

Promise实例生成以后，可以用then方法指定resolved状态和reject状态的回调函数 ，在then方法中，可以直接return数据而不是Promise对象，在后面的then中就可以接收到数据了 。

更多关于API的实例方法参考其他博主的 [Promise学习](https://blog.csdn.net/momdiy/article/details/77856099) 。其中，`.then` 是成功时执行，`.catch` 为失败时执行 ，`finally` 为无论成功与否都要执行。另外，Promise还有些对象方法，如：`Promise.all()`并发处理多个异步任务，所有任务都执行完才能得到结果；`Promise.race() ` 大概同上，但只有一个任务完成就打印结果。

##### 4.2、 fetch基本用法
基于Promise实现，基本用法为：
```html
  <script type="text/javascript">
    /*  第一个参数请求的路径 Fetch会返回Promise 所以可以使用then 拿到请求成功的结果 */
    fetch('URL').then(function(data){
      // text()方法属于fetchAPI的一部分，它返回一个Promise实例对象，用于获取后台返回的数据
      return data.text();
    }).then(function(data){
      //   在这个then里面我们能拿到最终的数据  
      console.log(data);
    })
  </script>
```
fetch API  中的 一些HTTP 请求，` fetch(url, options).then()` ,默认的是 GET 请求，需要在在 `options` 对象中 指定对应的 `method` 中指明需要的请求方法。

GET和DELETE传参需要在url配置数据，在在`options` 里说明是哪种方式；
POST和PUT需要在  `options`  设置 `body `和 `headers` 用来传递数据和设置请求头；

上述请求得到相应并正常返回的话，需要通过调用方法将其转换为相应格式的数据，比如json格式数据类型 `JSON.parse(data)` ，处理方法和上面的 `return data.text(); ` text() 字符串类型大致相同。

##### 4.3、 axios基础用法
这博主的axios详解已经讲的很详细了，这里我就偷懒下，想看的点 [传送门](https://blog.csdn.net/qq_31947477/article/details/106328200?ops_request_misc=%257B%2522request%255Fid%2522%253A%2522162422706716780265496069%2522%252C%2522scm%2522%253A%252220140713.130102334..%2522%257D&request_id=162422706716780265496069&biz_id=0&utm_medium=distribute.pc_search_result.none-task-blog-2~all~top_positive~default-1-106328200.first_rank_v2_pc_rank_v29&utm_term=axios&spm=1018.2226.3001.4187)就好。

简单说下axios ：它跟ajax一样，都是网络异步请求，基于Promise ，所以可以使用上述Promise API。在**发送请求之前**，需要设置请求拦截器，**获取数据之前**需要响应拦截器对数据进行加工处理，这样可以根据特定要求如请求头信息筛选出特定数据。配置大体如下：

```js
	// 1. 请求拦截器 
	axios.interceptors.request.use(function(config) {
      console.log(config.url)
      //  任何请求都会经过这一步   在发送请求之前做些什么   
      config.headers.mytoken = 'jiayou';
     //  这里一定要return   否则配置不成功  
      return config;
    }, function(err){
      // 对请求错误做点什么    
      console.log(err)
    })
	// 响应拦截器 
    axios.interceptors.response.use(function(res) {
      //  在接收响应做些什么  
      var data = res.data;
      return data;
    }, function(err){
     // 对响应错误做点什么  
      console.log(err)
    })
```

```javascript
//发送post请求
axios.post('/user',{
    uName: 'jiayou',
    pwd: 123
}).then(res=>console.log(res))
    .catch(err=>console.log(err));
```

##### 4.4、 async  和 await
async 和 await 大家或许会更熟悉，前面在学习 jQuery  异步操作时就经常得以使用。async 放到函数前面，`await` 只能在使用`async`定义的函数中使用，最后该函数则会返回一个`promise` 对象。

```javascript
   async function queryData() {
      var ret = await new Promise(function(resolve, reject){
        setTimeout(function(){
          resolve('jiayou')
        },1000);
      })
      return ret;
    }
```
前端交互部分就到这吧，虽然还有很多零碎的API使用规则及限制，但整理的有点累就不想写了，开始路由部分吧。


### 五、 前端路由
[官方文档](https://cn.vuejs.org/v2/guide/routing.html#ad) ， 路由都不陌生，URL之所以能根据地址访问对应对应资源，就是因为我们配置了路由。所以，路由就是这一种地址和资源对应的关系。路由也分为前端路由和后端路由，**前端靠hash值实现**，后端的靠服务器实现。

#### 5.1、路由基本使用：
[Vue-Router路由管理器](https://router.vuejs.org/zh/) ，其使用步骤大致为：

1.定义route,  一条路由的实现。它是一个对象，由 `path` 和 `component` 两个部分组成，如

```javascript
    const routes = [
      { path: '/home', component: Home },
    ]
```
2.根据构造函数 `new vueRouter() ` 创建router 路由，可以接收 routes  参数。

```javascript
const router = new VueRouter({
      routes // routes: routes
})
```

3.定义路由组件：地址匹配成功后，路由找到相应组件，最后将组件渲染到前面设置的占位符中

```javascript
//占位符
<router-view></router-view>
var User = { template:"<div>This is User</div>" }
```

4.配置完成后，把router 挂载到  vue  根实例中即可

```javascript
 new Vue({
 el:'#app'
  router
});
```
这里面省略了一点执行过程的小细节：前面需设置 router-link 标签，当标签被点击时路由会去寻找它的 地址属性，如果路径和 js ` { path: '/home', component: Home} `  中的path相互对应，那么就找到了匹配的组件， 最后把组件渲染到 <router-view> 标签所在的地方，到此路由完成。上述的这些都是基于hash 实现的。


#### 5.2、路由重定向(redirect)：

一般路由默认的页面都是通过这个来设计的，实现也很简单， `redirect` 表示要被重定向的新地址，设置为一个路由即可。

```javascript
{ path:"/",redirect:"/user"},
```

#### 5.3、嵌套路由，动态路由的实现方式
#### 5.3、命名路由，编程式导航
还有一些没写完，先留坑，后期补上...

-----
