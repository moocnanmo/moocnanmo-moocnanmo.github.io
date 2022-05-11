---
title: HTML&CSS的一些知识点
date: 2021-08-24 21:33:14
description: 记录些平常生活中常用到的html&css知识点，其中包括：盒模型，浮动定位，垂直水平居中，圣杯双飞翼布局。 #文章描述，只显示该处内容
cover: https://cn.bing.com/th?id=OHR.GiantManta_ZH-CN0594951444_1920x1080.jpg
categories: '烂笔头经验记录'
---

> 之前学习记录的，比较杂乱无章，将就看下吧

### 盒模型

```css
#box {
      width: 200px;
      border: 1px solid #666;
      padding: 10px;
      margin: 10px;
    }
```

```html
<div id="box">
    盒模型
  </div>
```

效果出来是这样，使用 `document.getElementById('box').offsetWidth`  打印宽度显示的222，其实很好理解，浏览器将盒子宽度的计算纳入了padding，border。
![在这里插入图片描述](https://img-blog.csdnimg.cn/ca9ed41f7d634336b770d2ce4cfd1695.png#pic=center)如果只想显示给定的200px宽度呢，怎么设置？很简单，在样式表中设置 `box-sizing: border-box`即可，`box-sizing`默认值：`content-box`，大小为上述222px。
![在这里插入图片描述](https://img-blog.csdnimg.cn/9958eae1e9614cf9862de16b18892bde.png#pic=center)

### margin合并,负值

- margin合并看的是并排的间距最大值，而不是俩间距相加。
- 当margin-top / margin-left为负值时，整体盒子会向上 / 左移动。
- 当margin-bottom 为负值时，整体位置不变，下边元素网上移动。margin-right同理。

### BFC

形成一个独立的渲染区域，内部的渲染不会影响外界。
举个栗子：当我们渲染父元素里的一个给定宽高的图片时，希望旁边的文字也具有同样的高度，这时就需要给父元素设定BFC。（一般用于需要独立的样式，不影响整体布局，广告））

- display: inline-block或flex

- position：absolute或者fixed

- overflow：不是默认的visible就行，hidden，auto,scroll就行

- float：非none

### position 相关

- static（默认）：会忽略 left、top、right、bottom 和 z-index 属性。
- relative（相对）：相对原本位置进行定位，其他的元素位置不会受到影响
- absolute （绝对）：祖先元素有 relative 或者 absolute定位 ？相对祖先位置定位 ： 相对body
- fixed （固定）：相对于 body 定位
- sticky （粘性）：跟随窗口位置，在设定位置会变成 fixed 定位，不在就不受定位影响

### 圣杯布局、双飞翼布局（三栏布局）

- **圣杯布局**，让中间栏优先加载渲染，两侧宽带固定，中间自适应。利用float+margin实现，margin用到负值的知识点。遇到底部栏和三栏挨在一起，可以用 `clear:both` 清除浮动，移动时可设置距离中间栏的-100%，在根据两边调解再移动的大小。
- **双飞翼布局**，大致和圣杯一样，只不过实现方式不同。

这里一般有个问题：**怎么实现三列布局，左右200，中间自适应且优先加载中间区域**。
我是这么想的：使用相对定位，左右绝对，设置左或右上边距为0，中间设置padding 0 200px；双飞翼，圣杯

### ink标签和import标签的区别

- link是html的标签，在页面加载时link会同步加载，权限比import高。
- import则是css提供的方法，在IE5以上才能使用，在页面加载完后才会加载import内容。

### 居中问题

 **垂直居中方法（行内元素line-height =height值；其他子绝父相定位 ）**
每个居中都有几种实现方式，需要用哪种方法，就取消注释掉该方法

```html
<!DOCTYPE html>
<html>
 <head>
  <meta charset="utf-8">
  <title></title>
  <style type="text/css">
   * {
    padding: 0;
    margin: 0;
   }

   #shuipin {
    width: 100%;
    height: 100px;
    background-color: #00A4FF;
    /* 1. */
    display: block;
    text-align: center;
    
    /* 2.2 */
    /* text-align: center; */
    
    /* 4. */
    /* display: flex;
    justify-content: center; */
   }
   #zhong{
    /* 2.1有宽度直接：margin: 0 auto; */
    /* width: 80px;
    margin: 0 auto; */
    
    /* 2.2 */
    /* display: inline; */
    
    /* 3. */
    /* position: absolute;
    left: 50%;
    transform: translateX(-50%); */
    }
    
    /* --------------------------------------------------------- */
    #chuizhi {
     width: 100%;
     height: 100px;
     background-color:#e74c3c ;
     
     /* 1.2多行垂直 */
     /* display: table-cell;
     vertical-align:middle; */
     
     /* 2. */
     /* position: relative; */
     
     /* 3. */
     display: flex; align-items: center;
    }
    #chuizhi #zhong{
     /* 1.单行垂直 */
     /* line-height: 100px; */
     
     /* 2. */
     /* position: absolute;
     top: 50%;
     transform: translateY(-50%); */
    }
    
    /* ---------------------------------------------- */
    #shuichui{
     width: 100%;
     height: 100px;
     background-color:#2ecc71;
     
     /* 1. */
     position: relative;
     
     /* 2. */
     /* position: relative; */
     
     /* 3. */
     /* display: flex;
     justify-content: center;
     align-items: center; */
    }
    #shuichui #zhong{
     
     /* 1. */
     width: 68px;
     height: 24px;
     border: 1px solid #EDEEF0;
     position: absolute;
     left: 0;
     right: 0;
     top: 0;
     bottom: 0;
     margin: auto;
     
     /* 2. */
     /* position: absolute;
     border: 1px solid #EDEEF0;
     left: 50%;
     top: 50%;
     transform: translateX(-50%) translateY(-50%); */
    }
  </style>
 </head>
 <body>
  <!-- 水平居中：
		1.给父元素设置block块级元素，在text-align: center
		2.需要谁居中，给其设置 margin: 0 auto;(看有无宽度)
		3.子元素使用绝对定位，在左移50%在transform移回去
		4.父元素使用flex布局,在使用justify-content: center;-->
  <div id="shuipin">
   <div id="zhong">
    水平居中
   </div>
  </div>

  <!-- 垂直居中 
		1.单行：父子元素等高，给子元素设置line-height；多行：父级设置display: table-cell;vertical-align:middle
		2.定位：子绝父相，在top移动，transform移动回来
		3.flex定位，给父级添加display: flex; align-items: center;-->
  <div id="chuizhi">
   <div id="zhong">
    垂直居中
   </div>
  </div>
  
  <!-- 水垂居中
		 1.父子有宽高，并设置子绝父相后，在给子元素四周设为0，magin:auto
		 2.父有宽高，子绝父相后，给上左设置50%，在transform负一半
		 3.给父级设置宽度100%，和宽度，在flex定位，-->
  <div id="shuichui">
   <div id="zhong">
    水平垂直居中
   </div>
  </div>
 </body>
</html>

```

### 多行文本显示省略号

```css
overflow：hidden; 
display：-webkit-box; 将盒子转换为弹性盒子
-webkit-line-clamp：2; 设置显示多少行
text-overflow：ellipsis; 文本以省略号显示   
```

### 清除浮动

`(给元素设定浮动后，元素脱离文档流，如果没有高度，样式会失效) => 浮动塌陷`

1. 在元素后面添加clear
2. BFC:overflow:hidden
3. **伪元素**：添加空元素在底部::after{ content: '';  display:block;  clear:both} 实际中用的最多

### CSS的选择器和优先级

id ，class ，标签，伪元素，伪类选择器等
优先级顺序为：!important>内联样式> 内部样式 > 外部样式 > 浏览器用户自定义样式 > 浏览器默认样式 ，同一个元素会被后面相同选择器属性覆盖

`这里顺带提一下常见的块行元素`

##### 块级元素：div、h、from、table、p、ul，li

##### 行内元素：span，img、a、input、select、button

`在提一下 ~  +  >`

- [ ] ~ ：A~B表示选择A标签后的所有B标签，但是A和B标签必须有相同的父元素（要同级，所有B）
- [ ] + ：A+B表示选择紧A后面的第一个B元素，且A和B必须拥有相同的父元素（要同级，一个B）
- [ ] > ：A>B指选择A元素里面的所有B元素，但要B元素是A元素的第一代（父子级，所有B）

### 重绘和重排

浏览器需要重新构造渲染树，这个过程称之为重排，受到影响的部分重新绘制在屏幕上叫重绘。

重排的可能有：添加或者删除比较多的DOM元素，窗口大小发生了改变，页面初始化。
减少重绘和重排的方法：不在布局信息改变时做DOM查询， 使用csstext,className一次性改变属性，使用fragment 。

### 隐藏元素

用display:none和visibilty:hidden，他俩区别：

- visibility：hidden，不会改变页面布局，不会触发该元素已经绑定的事件
- display:none， 会改变页面布局，相当于页面中把该元素删除掉。

### flex

先看MDN的说法吧：[flex](https://developer.mozilla.org/zh-CN/docs/Web/CSS/flex)

1. **flex-direction**：决定主轴的方向
 1. row 主轴为水平方向，起点在左端
 2. row-reverse：设置主轴为水平方向，起点在右端
 3. column：设置主轴为垂直方向，起点在上沿
 4. column-reverse: 设置主轴为垂直方向，起点在下沿
2. **flex-wrap**：换行的几种情况
 1. norap(默认值)：不换行
 2. wrap：换行，第一行在上方
 3. wrap-reverse：换行，第一行在下方
3. **flex-flow**：是flex-direction和flex-wrap的简写, 默认值为row nowrap。
4. **justify-content**： 我理解为水平居中对齐
 1. flex-start（默认值）：左对齐
 2. flex-end：右对齐
 3. center：居中
 4. space-betweet: 两端对齐，成员之间的间隔全都相等
 5. space-around: 每个成员两侧的间隔相等。
5. **align-items**：我理解为垂直对齐
 1. flex-start：交叉轴的起点对齐
 2. flex-end：交叉轴的终点对齐
 3. center: 交叉轴的中点对齐
 4. baseline: 成员的第一行文字的基线对齐
 5. stretch(默认值)：如果成员未设置高度或设为auto，将占满整个容器的高度
6. **align-content** ：交叉轴上对齐
7. **order:** 设置  `order:0`  ，决定子元素的排列顺序，越小越排前，可以说负数
8. **flex：1**  （0, 1, auto ）
 1. 有三个属性：
  1. flex-grow：属性定义成员的放大比例，默认为0，如果存在剩余空间，不放大
  2. flex-shrink：属性定义了项目的缩小比例，默认为1，如果空间不足，该项目将缩小
  3. **flex-basis**：定义了在分配多余空间之前，项目占据的主轴空间，默认值为 auto, 即项目本身的大小（重要）， 如果设置为 auto, 那么这盒子就会按照自己内容的多少来等比例的放大和缩小，如果随便设置一个带有长度单位的数字，那他就会等比例放大或缩小
 2. flex: 1; === flex: 1 1 auto

所以，这也解释了为什么flex设置为1，就能平分空间且水平居中了，实际就改了flex-grow的值，其余两项均为默认。
