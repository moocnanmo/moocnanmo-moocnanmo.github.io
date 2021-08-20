---
title: '使用nvm下载切换各node版本教程'
data: '2021-8-14 00:00:01'
description: 在同时开发几个node项目时难免会遇到版本与项目不兼容导致一些属性或方法不能正常使用问题，这时就需要nodejs版本管理工具nvm。 #文章描述，只显示该处内容
cover: https://img-blog.csdnimg.cn/20210814202145495.jpg?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L20wXzQ2ODkyNDky,size_16,color_FFFFFF,t_70
categories: '工具类'
---

## 使用nvm下载切换各node版本教程
	
在同时开发几个node项目时难免会遇到版本与项目不兼容导致一些属性或方法不能正常使用问题，这时就需要nodejs版本管理工具nvm。

 

##### 下载前需卸载之前的nodeJs

> 若之前跑过项目，建议备注当前nodeJs版本号;
> 若首次安装，参考[nodejs安装及环境变量配置](https://blog.csdn.net/roadlord/article/details/83302286?ops_request_misc=&request_id=&biz_id=102&utm_term=node%E9%85%8D%E7%BD%AE%E7%B3%BB%E7%BB%9F%E7%8E%AF%E5%A2%83%E5%8F%98%E9%87%8F&utm_medium=distribute.pc_search_result.none-task-blog-2~all~sobaiduweb~default-2-.first_rank_v2_pc_rank_v29&spm=1018.2226.3001.4187)。

 ## 1.nvm下载地址  ->[点我下载](https://github.com/coreybutler/nvm-windows/releases)
 选择nvm-setup.zip版本，下载后解压无需配置直接安装

 ## 2.检测nvm安装成功与否和node下载
在控制台或powerShell中输入`nvm -v`，若出现版本号即安装成功
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210523205437556.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3Rha2Vpbmdsb29w,size_16,color_FFFFFF,t_70#pic_center)使用`nvm install 14.15.1`可下载相应nodejs版本（建议选择偶数号，该类为稳定版）
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210523204908184.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3Rha2Vpbmdsb29w,size_16,color_FFFFFF,t_70#pic_center)若在安装路径中出现相应版本文件夹，以及相应文件夹下有npm对应的配置文件即安装成功。![在这里插入图片描述](https://img-blog.csdnimg.cn/20210523210144775.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3Rha2Vpbmdsb29w,size_16,color_FFFFFF,t_70#pic_center)若缺少npm文件则无法使用npm相应功能，此处可以使用`nvm uninstall 14.15.1` 卸载在重装即可。若还是不行，可手动下载对应npm包导入上述nodejs版本文件夹。

```clojure
node -v   					//可查看对应的npm版本
```
或在[npm下载站](https://nodejs.org/zh-cn/download/releases/)，查看选择对应文件进行下载。






此外需挂在淘宝镜像，在settings.txt文件下加入

> node_mirror: http://npm.taobao.org/mirrors/node/ 
npm_mirror: https://npm.taobao.org/mirrors/npm/
## 3.node版本切换

```clojure
nvm use 4.4.4                  //切换对应版本
nvm list installed 			  //查看已经安装的版本
npm  install -g nodemon      //全局下安装nodemon可实时监听后台状态
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210523212025551.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3Rha2Vpbmdsb29w,size_16,color_FFFFFF,t_70#pic_center)
到此，完成


