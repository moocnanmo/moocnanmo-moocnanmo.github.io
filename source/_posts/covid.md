---
title: '[vue2小项目]'
data: '2022-21-14'
description: 无意间知道接口，闲来无事敲着玩的，功能基本都是渲染，无交互，很简单
cover: https://img-blog.csdnimg.cn/1619accecb8748818949998abfddbb63.png
categories: '项目类'
---

> 公共接口，非个人维护，后期可能会失效，
> [疫情病毒信息](http://api.tianapi.com/ncov/index?key=b5919dd6f573907e378d0a6be7a78ff3)
> [国内疫情数据](https://api.inews.qq.com/newsqa/v1/query/inner/publish/modules/list?modules=diseaseh5Shelf)
> [城市历史数据接口](https://api.inews.qq.com/newsqa/v1/query/pubished/daily/list)
> [国外疫情数据](https://api.inews.qq.com/newsqa/v1/automation/modules/list?modules=WomWorld,WomAboard)
> [获取出行政策](https://file1.dxycdn.com/2021/0127/319/0185768311460321643-135.json)
> [风险地区可选城市列表](https://file1.dxycdn.com/2021/0127/904/9385768311460321643-135.json)
> [全国风险地区汇总](https://file1.dxycdn.com/2021/0202/196/1680100273140422643-135.json)
> [全国每日疫情数据](https://file1.dxycdn.com/2022/0220/952/6677926846218923353-135.json)
> [疫苗接种信息](https://api.inews.qq.com/newsqa/v1/automation/modules/list?modules=VaccineTopData,ChinaVaccineTrendData)
> [项目仓库](https://github.com/moocnanmo/covid.git)

## api来源：天行数据、腾讯、丁香医生

## 技术栈：vue+axios+vant+echarts+swiper

## 效果图

当前为首页Home，目前在功能三做了四个路由页面。

1. 根据接口返回信息渲染，每点为一个小`<p>`。（子组件渲染）
2. 大致同上，`v-for="item in news" :key="item.id"` 循环渲染出静态页面，且绑定`a` 可进行相应文章跳转。（子组件渲染）
3. 写了四个 `router-link` 路由文件，功能简单无交互。（父子传值）
4. 底部数据动态渲染。（父子传值）

![在这里插入图片描述](https://img-blog.csdnimg.cn/bd31b9219e504aabaefdee920a2ba8a9.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBA6Zuo54eV5a656Iul,size_12,color_FFFFFF,t_70,g_se,x_16#pic_center)

## 部分功能截图![在这里插入图片描述](https://img-blog.csdnimg.cn/56656617623341e3ba1034adcab17d78.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBA6Zuo54eV5a656Iul,size_20,color_FFFFFF,t_70,g_se,x_16#pic_center)

![在这里插入图片描述](https://img-blog.csdnimg.cn/81ff91245d894015abc89ab3622b3984.png#pic_center)

-----

## 3.29 更新

新增了出行政策模块，和利用echarts对疫情数据进行了折线图、地图绘制

![在这里插入图片描述](https://img-blog.csdnimg.cn/3380bacc9c334c8393adc626df3c60b6.png#pic_center)

地图总数据：

![在这里插入图片描述](https://img-blog.csdnimg.cn/b0a7d6c68534493b8a37254eee567116.png#pic_center)

省详情数据：

![在这里插入图片描述](https://img-blog.csdnimg.cn/08e90c0394cf427ebacae956c82ba2a1.png#pic_center)

## 4.1 更新

![](https://img-blog.csdnimg.cn/f7c9ba597c7441dbb78c6c1d14666d87.png)
地图文件单独放置，

```javascript
//导入国内 世界 省份
import 'echarts/map/js/china';
import 'echarts/map/js/world'
import 'echarts/map/js/province/beijing';
import 'echarts/map/js/province/shanghai';
import 'echarts/map/js/province/neimenggu';
//...
```

将地图组件作为一个插件，引入echarts和地图文件适配好后方便全局适用，

```javascript
const install = function (Vue, options) {
 Object.defineProperties(Vue.prototype, {
      $myChart: {
       get() {
   return {
    line(){},
    chinaMap(){},
    cityMap(){}
   }
       }
      }
 })
}
Vue.use(install)
```

在需要引入的文件下使用`this.$myChart.chinaMap("riskAreaArr", this.riskAreaArr);` 进行调用。该方法接受两个参数，一个是需要挂载到DOM元素的Id名称，一个是渲染数据。

```html
<van-tabs :value="active" animated @change="tabChange">
          <van-tab title="现存确诊">
            <div id="curConfirm" class="commonStyle"></div>
          </van-tab>
          <van-tab title="风险地区">
            <div id="riskAreaArr" class="commonStyle"></div>
          </van-tab>
          <van-tab title="累计确诊">
            <div id="confirm" class="commonStyle"></div>
          </van-tab>
</van-tabs>
```

因为默认显示现存确诊，所以在`created`阶段，获取到数据的同时渲染第一个地图，在切换tab时，通过 `change` 事件对 echarts 插件传入不同数据源，进行地图显示。

**值得注意的是：** vant在将数据渲染到页面上和调用插件渲染地图都是异步的，按理在 `mounted`  阶段可以渲染成功，但好像是vant框架对Vue2版本支持的原因，这一阶段渲染会爆错。折腾一段时间后想到 `nextTick` 延迟加载，让插件晚点渲染不就好了吗，哈哈

```javascript
this.$nextTick(() => {
    this.$myChart.chinaMap("riskAreaArr", this.riskAreaArr);
});
```

![在这里插入图片描述](https://img-blog.csdnimg.cn/a6ca83e6ef6e4cf7b78e201568cec8e8.png)

在想了解某个省份城市详情数据的时候，是这样处理的：

在echartsMap插件上，会根据你选择chinaMap()方法返回包含当前省份名称的一段模版字符串 , 通过点击模版字符串里的详情，插件可跳转到cityMap()方法中，其渲染原理和chinaMap大致，但cityMap需要接受三个参数（'DOM元素id' , 数据源 , 城市名称）

![在这里插入图片描述](https://img-blog.csdnimg.cn/8a00db77efa34db5ba73bc0113d64f01.png)
单纯的图片请求接口，图片采用 `swiper` 轮播图进行展示，将返回的图片链接覆盖到对应img标签的src上

![在这里插入图片描述](https://img-blog.csdnimg.cn/f2c239bacfdb46cf8efca84a34f4c9d2.png)

```javascript
<div>表格标题</div>
<div>
 <table>
  <thead></thead>
  <tbody></tbody>
 </table>
</div>
<div>底部收起</div>
```

表格展示和地图差不多，同样是写为组件，在需要的地方引入。

```javascript
<GTable type="tree" 
  :data="tableData" 
  :column="column" 
  :colorList="colorList" 
  showDetail limitHeight="500"
  @detailClick="detailClick" />
```

数据在组件调用的地方通过props传递，因为接口数据跟丁香医生的一致，都是处理好的对象数组类型，所以，渲染时，在tbody层只需要对td做两层for循环遍历拿到省份和对应城市数据即可。

详情方面，同样在tbody层末尾绑定点击事件，接受一个当前城市item参数，进而根据参数名称请求疫情数据折线图。

展开更多方面，因为渲染数据是对象数组格式，所以只需对数组长度进行按需显示即可。
