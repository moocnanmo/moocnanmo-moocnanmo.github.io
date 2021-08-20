---
title: '传统前后端分离的电商后台管理系统项目'
data: '2021-8-14 00:00:01'
description: 模仿黑马程序员的一个前后端分离的电商后台管理系统项目，后期打算应用到我的博客中，以及二开该项目，所以简单说下该项目。 #文章描述，只显示该处内容
cover: https://img-blog.csdnimg.cn/20210814204922436.jpg?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L20wXzQ2ODkyNDky,size_16,color_FFFFFF,t_70
categories: '项目类'
---

这是模仿黑马程序员的一个前后端分离的电商后台管理系统项目，后期打算应用到我的博客中，以及二开该项目，所以简单说下该项目。
## 演示视频：[B站传送门](https://www.bilibili.com/video/BV12f4y157kY?spm_id_from=0.0.dynamic.content.click)

# 一：简介

整体项目并不复杂，相对而言比较容易上手，采用的技术栈有：

> * Vue 的基本使用 以及  Vue-Route
> * Element-UI  样式框架
> * Axios
> * Echarts （绘制数据图表）
 > * MySQL

项目整体样式为：
![在这里插入图片描述](https://img-blog.csdnimg.cn/4bf5e85f50374c99a286c714a22efe66.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3Rha2Vpbmdsb29w,size_16,color_FFFFFF,t_70)文件结构：
![在这里插入图片描述](https://img-blog.csdnimg.cn/90d318e83e904b33b2d979669d0eb7de.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3Rha2Vpbmdsb29w,size_16,color_FFFFFF,t_70#pic_center)项目运行依赖：

```javascript
"dependencies": {
    "axios": "^0.21.1",
    "core-js": "^3.6.5",
    "echarts": "^4.1.0",
    "element-ui": "^2.4.5",
    "less": "^3.9.0",
    "nprogress": "^0.2.0",
    "vue": "^2.6.11",
    "vue-router": "^3.2.0",
    "vue-table-with-tree-grid": "^0.2.4"
  },
```

------
# 二：基本功能介绍
#### 2.1、表单的双向绑定和数据验证
在表单头部加入model和rules属性后，在script中绑定数据项，再在元素中使用v-model即可双向绑定数据，rules同理，添加规则，再在元素追加prop属性与之绑定。这部分在注册，修改功能这块用的频繁，后续可复用该类代码。
![在这里插入图片描述](https://img-blog.csdnimg.cn/f1eb9ee1b11c4be084d311d94fc0b2f7.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3Rha2Vpbmdsb29w,size_16,color_FFFFFF,t_70)
#### 2.2、路由导航守卫控制访问权限
用户成功登陆服务器会发放token,对于一些只有登陆成功后才能看的页面，不能通过地址栏访问到相关页面

```javascript
// 挂载路由导航守卫
// ()里有三个参数，第一个为要访问的路径，第二个为从哪来，第三个为一个函数表示放行（'/home'）
router.beforeEach((to, from, next) => {
  if (to.path === '/login') return next()
  // 获取token
  const tokenStr = window.sessionStorage.getItem('token')
  if (!tokenStr) return next('/login')
  next()
})
```
除了登陆界面不需要token，后续的所有接口都要借用token进行使用，在 **main.js** 入口文件调用

```javascript
axios.interceptors.request.use(config => {
  // Authorization->授权
  NProgress.start()
  config.headers.Authorization = window.sessionStorage.getItem('token')
  return config
})
```
这里的NProgress插件的进度条显示功能忽略，这里的请求拦截思路是：

axios调用其中的intercepts属性，里面有个request成员（请求拦截器），当调用时会调用use函数为拦截器挂载一个回调函数，你向服务器发送了一个axios数据请求，那么在发送期间会优先调用这个回调函数，return返回的是预处理结果，表示已对请求头做了预处理，这样的请求才能发到服务器

此外，这里说下路由重定向，一般用在显示默认页面这块，思路大体就是：比如登陆到home页面，需要默认显示welcome页面，这时就需要新建W页面，在路由表里导入W文件，在H规则下下设置children路由，重定向到W页面，最后在H文件设置路由占位符`<router-view>`就行。

#### 2.3、添加用户
整体过程大致为：首先需要通过点击事件触发添加函数，接着validate的预校验，通过规则后发起**post**请求，传递之前设置好的用户对象，接着关闭弹窗，重新加载数据
![在这里插入图片描述](https://img-blog.csdnimg.cn/9679bd05eac344f18df8518e4d03224b.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3Rha2Vpbmdsb29w,size_16,color_FFFFFF,t_70)此外，再添加用户时如果点击了取消，需要调用 `ref` (引用)的 `addFormRef.resetFields()` 方法重置用到重置消除用户填写信息。

#### 2.4、修改用户
更新操作大致和添加相同，为修改按钮添加弹窗和验证， 方法大致和上述相同
![在这里插入图片描述](https://img-blog.csdnimg.cn/f098da5bd02f4b26a9c5afb0158bd2d3.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3Rha2Vpbmdsb29w,size_16,color_FFFFFF,t_70)
#### 2.5、删除用户
整体过程大致为：

 1. 利用element的MessageBox 弹框，在elment-ui中全局挂载confirm方法；
 2. 在点击删除时传递用户id过去；
 3. 复制elementUI中的确认弹框代码，根据用户选择决定是否删除

![在这里插入图片描述](https://img-blog.csdnimg.cn/7d127442402b4935b264866e3309890e.png)
![在这里插入图片描述](https://img-blog.csdnimg.cn/8d89f410fceb4048827131e1d11f91c9.png)
![在这里插入图片描述](https://img-blog.csdnimg.cn/b51ce6c3d11444919a5548ead04cc433.png)
![在这里插入图片描述](https://img-blog.csdnimg.cn/cea1581ac86d439c9b9cb4c47032229d.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3Rha2Vpbmdsb29w,size_16,color_FFFFFF,t_70)
#### 2.5、查找用户
在获取数据时发送 `get` 请求，通过返回的数据成员 `data.users`  赋值给数据对象，在页面中重新加载数据即可。

```javascript
const { data: res } = await this.$http.get('users', {
        params: this.queryInfo
      })
      if (res.meta.status !== 200) {
        return this.$message.error('获取用户列表失败！')
      }
      this.userlist = res.data.users
      this.total = res.data.total
      console.log(res)
    }
```

# 三：主要功能介绍
#### 3.1、角色权限分配
当管理员在用户列表中给成员分配角色后，可依据特定角色的拥有默认的权限，后续可追加或删除相关权限。![在这里插入图片描述](https://img-blog.csdnimg.cn/0e59bb29d7444e1d8ac1fb93b520f5cf.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3Rha2Vpbmdsb29w,size_16,color_FFFFFF,t_70)
![在这里插入图片描述](https://img-blog.csdnimg.cn/e252dabdaf8b4fdd91ba550fd2deac7d.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3Rha2Vpbmdsb29w,size_16,color_FFFFFF,t_70)
这里将权限分为三个等级，最左侧为一级权限，另外为二级，三级权限，在同一梯度显示。功能实现思路大致为：

 1. 在显示分配对话框前就拿到用户id,通过发送get请求获取到所有用户数据。
 2. 为每个结点设置递归函数，在获取用户id时，拿到他们的递归结点id。
 3. 设置第三级节点没有children属性，获取到该节点后向上返回数据，将结果以数组的形式保存，最后根据插槽显示数据。

修改用户权限大致相同，这里用到了element提供的树形控件，多了监听和关闭后清楚权限的步骤，在修改完成后想服务器发送post提交请求，刷新数据

![在这里插入图片描述](https://img-blog.csdnimg.cn/eb91d376e11f483695b7774b096cb07b.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3Rha2Vpbmdsb29w,size_16,color_FFFFFF,t_70)
同理，下个功能模块中的商品分类大抵相同。




#### 3.2、为商品添加静态/动态属性
通过级联选择器，判断所选项是否在第三级，在通过发送 get 数据请求，为选中商品添加对应属性，因为动态参数和静态参数模块内容差不多，所以公用一套模版。此处的功能开发和上述的角色权限分配差不多，渲染模版后，根据成员属性进行相应修改，新增内容以节点形式保存到数组中，最后发送post请求更新节点数据

![在这里插入图片描述](https://img-blog.csdnimg.cn/23d68cedd17b48718d0c1da35831fcc8.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3Rha2Vpbmdsb29w,size_16,color_FFFFFF,t_70)


#### 3.3、添加商品
设计思想：添加商品前必须为其选定所选类，通过该类，商品每点击一次左边的tab栏就发起一次数据请求，获取当前商品可选参数，添加的商品每个属性都以独立的数组进行存储，部分数组需要更改为字符串，整个商品对象存储在对象中，提交时需要预验证，
![在这里插入图片描述](https://img-blog.csdnimg.cn/d7d50968418d49e9bcbe939b97377835.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3Rha2Vpbmdsb29w,size_16,color_FFFFFF,t_70)
#### 3.4、数据报表

利用数据绘制报表可利用echarts插件辅助完成，详见 [官网](https://echarts.apache.org/zh/index.html) ，（我没利用数据库的数据，利用的是官网案例...）

过程大体为：

 1. 导入echarts包，在入口文件或当下文件都行
 2. 准备一个具有宽度和高度的容器Dom
 3. 基于准备好的dom，初始化echarts实例
 4. 指定图表的配置项和数据,使用刚指定的配置项和数据显示图表


-----

以上，其实项目中有较多闲置的静态写死的数据，后端也没开放接口，但整体基本功能还算完善，复制些的功能也就是角色列表展开，权限分配，添加商品这三类,对于接口的调用和分析返回的数据包类型有很好的技术提升。

