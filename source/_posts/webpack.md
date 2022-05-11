---
title: 记录一下关于webpack打包遇到的bug
cover: https://img-blog.csdnimg.cn/20210814202145398.jpg
description: 因为之前着手做个几个小项目，需要到的node版本不一，所以安装了nvm以用来切换，但在这webpack打包时遇到很多小问题，着实耗费了大量时间，想给大家避避坑。 #文章描述，只显示该处内容
categories: 'Bug排错'
---

# 这贴记录一下关于webpack打包遇到的bug，及解决方案
因为之前着手做个几个小项目，需要到的node版本不一，所以安装了nvm以用来切换，但在这webpack打包时遇到很多小问题，着实耗费了大量时间，想给大家避避坑。


----------

贴一下之前的错误报告：


> Error: EINVAL: invalid argument, mkdir 'C:\remaining\‪C:\nvm\nodejs\node_global'

这问题是.npmrc文件配置问题了，因为当时webpack采用的是局部安装，运行后报错，根据错误语句上网找了很多解决方案，**绝大部分帖子**都是建议重装上node官网重装，然后在配置`.npmrc`文件配置一下信息，就跟着博客照做，发现情景根本不适合此我这机子，最后不仅弄乱了前先配置好的环境变量，还把npm命令弄的无法正常使用。

排错老半天，最后把官网安装的node和之前配置的都删了，还原.npmrc文件配置成默认信息就好，该bug解决。（差点想把nvm也卸了，全都清洗重装）

```javascript
prefix=‪C:\nvm\nodejs\node_global
cache=C:\nvm\nodejs\node_cache
```
                                                                                                

> webpack : 无法将“webpack”项识别为 cmdlet、函数、脚本文件或可运行程序的名称。请检查名称的拼写，如果包括路径，请确保路径正确，然后再试一次

这里我选择全局下载3.X版本的webpack，最后根据我自身机子的情况设置了环境变量，告诉系统该插件位置在哪就好。

具体为：在系统变量的path中添加全局位置，在新建一个NODE_PATH变量用来存放该位置下的node_modules值，最后再在自身变量的path中添加全局位置。  


![在这里插入图片描述](https://img-blog.csdnimg.cn/20210701181416563.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3Rha2Vpbmdsb29w,size_16,color_FFFFFF,t_70#pic_center=200*160 )   

![在这里插入图片描述](https://img-blog.csdnimg.cn/2021070118181220.png#pic_center)

> npm WARN webpack-cli@3.3.12 requires a peer of webpack@4.x.x but none is installed. You must install peer dependencies yourself.
> 

这问就是版本问题了，在这项目我是指定了3.5.5的版本安装，前面还有个小插曲，之前我的webpack-cli默认转最新版的导致webpack也安装出错，在这也给他降低版本就行。  
                                              

> Insufficient number of arguments or no entry found.
Alternatively, run 'webpack(-cli) --help' for usage info.

这里把之前的webpack.config.js文件内容

```javascript
module.exports = {
    mode: 'development'
}
```
改为：

```javascript
const path = require('path')
module.exports = {
    //入口文件
    entry: path.join(__dirname, './src/index.js'),
    //出口文件
    output: {
        path: path.join(__dirname, './dist'),
        //指定输出文件的名称
        filename: 'index.js'
    }
}
```

![在这里插入图片描述](https://img-blog.csdnimg.cn/2021070118313082.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3Rha2Vpbmdsb29w,size_16,color_FFFFFF,t_70#pic_center)
另外，在打包css、less、图片，安装的插件时容易出现版本不兼容的错误，建议使用一下版本号：
![在这里插入图片描述](https://img-blog.csdnimg.cn/2021070319200445.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3Rha2Vpbmdsb29w,size_16,color_FFFFFF,t_70#pic_center)在打包vue项目文件时，也容易产生bug , 主要是路径和版本号问题 ，排错时优先考虑。



