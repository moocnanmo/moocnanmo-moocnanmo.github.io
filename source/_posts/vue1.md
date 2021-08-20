---
title: Vue结合文档的笔记整理-上篇
cover: https://img-blog.csdnimg.cn/2021081420214666.jpg?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L20wXzQ2ODkyNDky,size_16,color_FFFFFF,t_70
description: 前段时间本着一通则百通的态度想系统的学习下前端框架，着手拍定为上手较为简易，适用于移动端开发的Vue。但安排的时间被零散打散，学着不过瘾今天好好总结一下 #文章描述，只显示该处内容
categories: '烂笔头经验记录'
---

# Vue笔记整理：结合官方文档的专项复习-上篇
------

前段时间本着一通则百通的态度想系统的学习下前端框架，着手拍定为上手较为简易，适用于移动端开发的**Vue**。但安排的时间被零散打散，学着不过瘾今天好好总结一下，[Vue官网。](https://cn.vuejs.org/)


> * 1.基础模版语法指令
> * 2.比较常用的特性
> * **3.组件化开发**
> * 4.前后端的一些交互功能
> * **5.一些路由知识**

### 一、基础模版语法指令
基本模版：[官方样例](https://cn.vuejs.org/v2/guide/)，这也是我学习Vue的第一个代码。很好理解：导入Vue.js文件，实例化一个样例在挂在到DOM上，如果打开网页控制台，修改`app.msg`的值，你会发现视图和数据间的更新是同步的。这块说下数据绑定，属性绑定，事件绑定，和分支循环吧。

```javascript
<!DOCTYPE html>
<html lang="en">

<head>
    <meta charset="UTF-8">
    <title>Document</title>
</head>

<body>
    <div id="app">
        <div>{{msg}}</div>
    </div>
    <script type="text/javascript" src="js/vue.js"></script>
    <script type="text/javascript">
        var app = new Vue({
            el: '#app',
            data: {
                msg: 'Hello Vue'
            }
        });
    </script>
</body>
</html>
```

 - el:元素挂载位置（值可以是css选择器或DOM元素），这里为`#app`
 - data：模型数据（值为对象）
 - 后续一些`v-`指令需绑定到DOM元素上使用

##### 1.1、数据绑定指令（v-tetx,v-html,v-pre，v-bind）
数据绑定最常见的形式就是使用“Mustache”语法 (双大括号) 的文本插值:
`<span>Message: {{ msg }}</span>`
Mustache 标签将会被替代为对应数据对象上 `msg` 的property 值。若`msg`的值发生改变，就会发现视图和数据间的更新是同步的。
若不想同步，可使用`v-once`指令，这只会更新一次！
另外还有：
 - `v-text`：填充纯文本。
 - `v-html`：填充HTML片段，但有安全性问题，容易导致XSS攻击。
 - `v-pre`：填充原始信息，跳过编译过程。
 - `v-bind`：Mustache 语法不能作用在 HTML attribute 上，此时需要到`v-bind`。
 - [官方API](https://cn.vuejs.org/v2/guide/syntax.html#%E6%8F%92%E5%80%BC)


##### 1.2、双向数据绑定指令（v-model）
[v-model 官方API](https://cn.vuejs.org/v2/api/#v-model)，v-model的本质不过是语法糖，能在input 、textarea 及select 元素上创建双向数据绑定，监听用户的输入事件以更新数据，以及处理一些输入处理，比如使用修饰符：

`trim`：过滤用户输入的首尾空白字符、
`.number`：将用户的输入值转为数值类型、
`.lazy`：解决model在文本输入框内自动触发数据同步的情况，转为在 change 事件_之后才进行同步
 
这里设计到了MVVM的设计模式：M (model)，V (view)，VM (View-Model)。![在这里插入图片描述](https://img-blog.csdnimg.cn/20210619210316571.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3Rha2Vpbmdsb29w,size_16,color_FFFFFF,t_70#pic_center)这点最核心的就是双向绑定方式：从视图到模型用的是事件监听，从模型到视图用的是时间绑定。

##### 1.3、事件绑定（v-on）
[v-on  官方文档](https://cn.vuejs.org/v2/api/#v-on)，这是绑定事件监听器，事件类型由参数指定。可监听原生DOM事件(" : " 后面可以添加事件名称)，自组件的触发事件，监听器键值对的对象。大致有：

 - v-on:click='函数名称'
 - v-on:[event]  动态事件
 - v-on:click="handel(a, b，$event)"   传递多个参数
 - @click.stop 阻止冒泡
 -  @click.prevent 组织默认行为
 - @keyup.13    键修饰符(.enter回车键   .delete删除键)
 - v-on="{ mousedown: doThis, mouseup: doThat }  对象语法
 
如果是事件函数绑定调用，那么最后一个参数必须是事件对象(且名称必须为 $event).

##### 1.4、属性绑定（v-bind）
[v-bind  官方文档](https://cn.vuejs.org/v2/api/?#v-bind)，动态的绑定一个或多个属性，在绑定`class`或`style`时，支持数组或对象格式的数据类型。大致有如下用法：

 -  v-bind:src="属性值"
 - :class="{ red: isRed }"    class 绑定
 - class="{ active: isActive }"    isActive为布尔类型，只有为真时类名才显示
 - :style="{ fontSize: size + 'px' }"   style 绑定
##### 1.5、分支循环结构（v-if , v-else , v-else-if , v-show，v-for）
[分支循环API](https://cn.vuejs.org/v2/api/?#v-if)，这块比较简单，基本看一遍就明白，没太多可以说的，值得注意的是，当v-for和v-if一起使用时，`v-for`的优先级比后者高！`v-show`为控制元素是否显示在页面（已经渲染成功）

```html
<div v-if="type === 'A'">
  A
</div>
<div v-else-if="type === 'B'">
  B
</div>
<div v-else-if="type === 'C'">
  C
</div>
<div v-else>
  Not A/B/C
</div>
```

```html
//遍历对象  值，键，索引
<div v-for="(item, index) in items"></div>  
<div v-for="(val, key) in object"></div>
<div v-for="(val, name, index) in object"></div>
```

### 二、比较常用的特性
这里想到的特性有：表单操作，计算属性，**侦听器**，**过滤器**，还有Vue的生命周期，分主次介绍。

##### 2.1、表单操作
[官方文档](https://cn.vuejs.org/v2/guide/forms.html)，大致记住 input，textarea，select，radio，checkbox这些基本用法，此处的表单修饰符也可同1.2双向数据绑定中的一致。

```html
  <div id="app">
    <form action="http://itcast.cn">
      <div>
        <span>姓名：</span>
        <span>
          <input type="text" v-model='uname'>
        </span>
      </div>
      <div>
        <span>性别：</span>
        <span>
          <input type="radio" id="male" value="1" v-model='gender'>
          <label for="male">男</label>
          <input type="radio" id="female" value="2" v-model='gender'>
          <label for="female">女</label>
        </span>
      </div>
      <div>
        <span>爱好：</span>
        <input type="checkbox" id="ball" value="1" v-model='hobby'>
        <label for="ball">发呆</label>
        <input type="checkbox" id="sing" value="2" v-model='hobby'>
        <label for="sing">唱歌</label>
        <input type="checkbox" id="code" value="3" v-model='hobby'>
        <label for="code">写代码</label>
      </div>
      <div>
        <span>职业：</span>
        <select v-model='occupation' multiple>
          <option value="0">请选择职业...</option>
          <option value="1">矿工</option>
          <option value="2">农名工</option>
          <option value="3">工程师</option>
        </select>
      </div>
      <div>
        <span>个人简介：</span>
        <textarea v-model='desc'></textarea>
      </div>
      <div>
        <input type="submit" value="提交" @click.prevent='handle'>
      </div>
    </form>
  </div>
```

```javascript
 var vm = new Vue({
      el: '#app',
      data: {
        uname: 'jiayou',
        gender: 2,
        hobby: ['2','3'],
        occupation: ['2','3'],
        desc: '加油'
      },
      methods: {
        handle: function(){
          console.log(this.desc);
        }
      }
    });
```
##### 2.2、计算属性
[官方文档](https://cn.vuejs.org/v2/guide/computed.html)，计算属性目的是为了让复杂逻辑简单化和让模版更简洁。例如官方文档里的例子，让字符串翻转

```html
<div id="example">
  {{ message.split('').reverse().join('') }}
</div>
```
如果页面中有多出需要该类效果，复杂而又冗余的代码显然不是最好的选择，这时我们只需在js中加入计算属性，元素使用时调用该类方法就好。

```javascript
 computed: {
        reverseString: function(){
          return this.msg.split('').reverse().join('');
        }
      }
```
刚开始学这里的时候我就有个疑惑，这特喵跟函数方法有什么区别 ，后来我猜清楚计算属性是基于他们的依赖进行缓存的，而方法不存在缓存

##### 2.3、侦听器
[官方文档](https://cn.vuejs.org/v2/guide/computed.html#%E4%BE%A6%E5%90%AC%E5%99%A8)    虽然计算属性在大多数情况下更合适，但有时也需要一个自定义的侦听器。所以Vue通过`.watch`选项提供了一个通用方法。其作用大致为：数据变化时执行异步或开销比较大的操作。
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210619231806107.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3Rha2Vpbmdsb29w,size_16,color_FFFFFF,t_70#pic_center)
##### 2.4、过滤器
[官方文档](https://cn.vuejs.org/v2/guide/filters.html#ad)，过滤器 `filter` 通常用在两个地方：双花括号插值和 `v-bind` 表达式 ，作用与常见的文本格式化。比如常见的格式化数据(首字母大写，指定日期格式等)
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210619233008127.png#pic_center)

```html
<!-- 过滤器使用-->
<!-- 在双花括号中 -->
{{ message | capitalize }}
<!-- 在 `v-bind` 中 -->
<div v-bind:id="rawId | formatId"></div>
<!-- 过滤器是 JavaScript 函数，因此可以接收参数 -->
{{ message | filterA('arg1', arg2) }}
```

```javascript
//自定义过滤器
Vue.filter('自定义过滤器名称', function (value) {
 //业务逻辑
})
```
还有局部过滤器，大体相同，就不说了

##### 2.4、生命周期
[官方文档](https://cn.vuejs.org/v2/guide/instance.html#%E7%94%9F%E5%91%BD%E5%91%A8%E6%9C%9F%E5%9B%BE%E7%A4%BA)，Vue实例从创建到销毁的过程，就是生命周期。详细来说也就是从开始创建、初始化数据、编译模板、挂载Dom、渲染→更新→渲染、卸载等一系列过程。期中生命周期主要分为挂载，更新，销毁三个阶段。
 
 - 挂载:(初始化相关属性)：beforeCreate，created，beforeMount，mounted
 - 更新(元素或组件更新变化)： beforeUpdate,Update
 - 销毁(销毁相关属性)： beforeDestroy,destroyed
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210619235145191.jpg?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3Rha2Vpbmdsb29w,size_16,color_FFFFFF,t_70#pic_center)
这里只大概知道个条理，具体深入方面的知识点还是没能说出，可能是目前阶段还用不到，感觉不是很重要吧哈哈哈哈，生命周期这块我是参照[该博主的](https://blog.csdn.net/weixin_46383743/article/details/107655477?ops_request_misc=%257B%2522request%255Fid%2522%253A%2522162411819316780357279613%2522%252C%2522scm%2522%253A%252220140713.130102334..%2522%257D&request_id=162411819316780357279613&biz_id=0&utm_medium=distribute.pc_search_result.none-task-blog-2~all~top_positive~default-1-107655477.first_rank_v2_pc_rank_v29&utm_term=%E7%94%9F%E5%91%BD%E5%91%A8%E6%9C%9F&spm=1018.2226.3001.4187)。
明天在补完剩下的吧，累了









