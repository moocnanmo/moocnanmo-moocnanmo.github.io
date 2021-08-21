---
title: '记录下使用hexo框架搭建博客和踩过的坑'
data: '2021-8-20 00:00:01'
description: 整个博客从开发到遇到的主要bug的解决过程，最大的感受：这排错思路有待提高，有些坑躺了近十遍还是觉得那没问题，换个角度想问题思路就有了。 #文章描述，只显示该处内容
cover: https://img-blog.csdnimg.cn/20210814202145968.jpg
categories: 'Bug排错'
---

# 记录下使用hexo框架搭建博客和踩过的坑


# 前言
------

由于经常使用csdn,博客园，简书来查阅问题的解决方案，然后开始在 CSDN 中记录些所得经验和避坑忠告，但因为广告太多和样式不满意，便萌生了自己搭建个博客的想法。之后便了解到一些搭建用的框架平台:`Hexo` `Hugo` （对新手很友好），Vue开发者可以使用 `VuePress`，`react` 开发者可以使用 `Gatsby`。



### 我猜测大部分读者觉得开发一个博客会有以下顾虑
1. **搭个博客是不是得从零开始搭建，包括静态页面和交互效果，那会不会很麻烦**（用上面框架即可拒绝造轮子）
2. **建博客是不是要买域名然后备案在服务器**（全程有白嫖的，托管到例如github，gitee，vercel上完成部署,；要买也行，XX云一个域名十几二十，服务器买学生机一年也不到一百）
3. **搭建博客是不是要会专门的语言，还要会数据库**（前端三件套加md语法就好，复杂点的要回数据库，某云可以可视化操作数据库，完成建表增删改查的基操）
4. **要是搭博客出Bug了怎么办，你能帮帮我吗**（不能，根据警告或提示找解决方法）


## 总结：只要想搭，办法有的是，甚至你还可以将白嫖贯彻到底

下面大致说下我的搭建博客的过程和踩过的坑，给后来者避雷

---


# 1.如何快速搭建一篇博客
 第一步： 这里推荐去B站根据框架（可参考上述）搜“ 如何使用XXX搭建博客 ”，选个整个过程看起来还算**完整的，近期** 线下使用，嫌视频太长或喜欢看文档的也可以去平台上搜完整的搭建过。以`Hexo`为例，本地搭建过程我参考的是 [Hexo建站教程](https://sunhwee.com/posts/6e8839eb.html#toc-heading-3)
 
 第二步：大概看遍/读遍后，有编程基础的可以去所选框架的官网读使用文档，因为版本迭代更新快，你看到的或许不适用了；接着选好主题，再看主题的使用文档，一般都会很详细，每个组件，功能，模块的搭建使用都贴有使用说明，跟着做就行
 
 第三步： 部署上线，这步很容易出错，每台真机的环境都不一样，以推到github来说，你可能会遇到网络问题，推送命令问题，开发工具环境问题等等，为解决一个bug而衍生出两个bug很正常，按下耐心找方法解决。
 
  下面列举下部署静态网站的大致步骤：
- [ ] 有Git，nodejs环境
- [ ] 安装hexo，根目录初始化 hexo init、，和下载它的上传工具
- [ ] 根据主题文档，配置本地仓库主题，一般在系统配置文件_config.yml下修改themes：
- [ ] 上面那个是系统配置文件，还有个**主题配置文件** ，由于使用npm或git下载的方式不同，这文件位置也不同，**看文档看文档看文档**配置
- [ ] 根据主题配置时候，有些外链指向github，这时自己解决网络问题。配置不成功看github的Issues讨论区。
- [ ] 遇到本地加载不同步的，清理缓存（官方文档有）在启动
 - [ ] 本地没问题，可以部署到pages上线了（**注意**：配置文件中root路径记得更改为： /），托管我选github，这段时间部署到码云容易涉嫌违规，排查违规词太难这就不推荐了。


# 2.踩过的坑以及解决方案
### 2.1网络问题的坑
#### 坑1：网络延时10054、443错误
大家都懂，不用点魔法，拉取推送总能因为网络延迟导致推送失败，具体为：
![在这里插入图片描述](https://img-blog.csdnimg.cn/bbc25095b02f4b17b3f49b8a44f5ff12.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3Rha2Vpbmdsb29w,size_16,color_FFFFFF,t_70)
- fatal: unable to access‘https://github . com/ xxx. github. io.git/': OpenSSL SSL read: Connection was reset, errno 10054
-  fatal: unable to access ' https://github . com/xxx . github.io.git/': Failed to connect to github.com port 443: Timed out

#### 坑2：代理问题
没有魔法的读者只能过会才上传了，当我开始掌握魔法时，为Git设置代理又出大问题（没注意SSR端口号），设置的过程是根据c*dn配置的，非常不可靠，配完后gitconfig文件是这样的：
![在这里插入图片描述](https://img-blog.csdnimg.cn/8556bed5a0164a76805304b5881c5d59.png)
上传时自动吧https更改为git前导致推送失败![请添加图片描述](https://img-blog.csdnimg.cn/806c1f823d334cb38da3619b0d35706a.png)![在这里插入图片描述](https://img-blog.csdnimg.cn/c835be0f28f9496dac3cd9d244b87b14.png)*  fatal: remote error:
You can't push to git://github . com/ xxx. github. io. git
Use https:/ / github . com/ xxx . github . io. git

使用了set-url 强制把请求头更改为https也无济于事，预算取消全局代理配置，返回gitconfig文件查看，发现无效，于是把有关github的[ ] 配置行删除，重新配代理，找到SSR端口号，在下载个**tortoiseGit**管理器，在设置→Git→编辑全局.gitconfig替换域名![请添加图片描述](https://img-blog.csdnimg.cn/0222a7af6fb04bd699dcb9f2fbb9e6ce.png)这里耗时挺久的，为解决一个上传问题，找个工具产生一堆问题，好在都修复了


### 2.2部署时的坑
#### 坑3：上传文件问题
我是这样保存仓库的：给github托管，码云实时保存仓库，所以完成一份，要给俩仓库推送，看效果和备份。然后就把整个文件同时推给俩仓库，发现github上样式文件老丢失，首页缺少index.html文件等错误。当时一时想不出那里错，本地运行好好的，为什么推上去就有问题，就觉得还是网络问题，到时传输时文件丢失，沿着这条错误的方向排错了很久，删了建删了建，无果才换方向。
最后发现，是我上传的文件不对，只需要上传打包生成好的public文件即可，也就是编译好的build文件，不需整个项目上传，往后部署到服务器也是同理。

正确的文件布局：
![在这里插入图片描述](https://img-blog.csdnimg.cn/f955210734a14598bbe244afbcd3be3b.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3Rha2Vpbmdsb29w,size_16,color_FFFFFF,t_70)
#### 坑4：public文件更新问题
在本地修改了_config.yml的系统配置文件后， `hexo clean` 清除缓存，和  `hexo g ` 生成静态页面，在`hexo d ` 上传后，在推送上去，发现g远程长裤的内容没发生该改变，试了多次无果发现还是删除重建简单暴力有效：删除后重输入命令，强制推送覆盖远程仓库内容即可



#### 坑5：js、css等样式文件404问题
在上传到时候，加载出来的页面因为样式文件丢失，和本地差别太大，功能显示都有问题，这时候首先F12打开控制台看下报的什么错，根据错误索引到相关文件，发现样式的引用路径有问题，大问题，怎么显示的是我本地路径？

![在这里插入图片描述](https://img-blog.csdnimg.cn/efa7db1750404e358b76670107b0c53d.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3Rha2Vpbmdsb29w,size_16,color_FFFFFF,t_70)
 - Failed to load resource: the server responded with a status of 404 ()

**根据控制台报的错误定位问题！**，发现404的几个文件都是因为路径问题，前缀是我本地的地址，这里应该用相对路径，于是我我想到配置文件有个root 属性，刚开始配置上，文档所示跟路径，我迷糊就填了本地的，404！于是乎我按照他的理解，填了远程仓库的，又是404！ 后来觉得index文件在根目录下，填个 ： ./ 应该可以，发现除了首页正常显示，其他还是404，我giao！在细细的捋了下思路，发现自己真傻，为什么不设置： / 让他自己寻找。果不其然，这次成功了


### 2.3 浏览器缓存问
#### 坑6：页面404
页面404，查看设置的pages选项，发现没问题，已经生成子网，但页面就是重定向不到，于是想到，在这之前，我是删库重建了，而我又在本地生成重新上传，仓库的地址一样，会不会这页面是上个仓库的？于是过一小会刷新下果不其然出来了！
![在这里插入图片描述](https://img-blog.csdnimg.cn/a3b15df398cb45439c8d3a81367c364d.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3Rha2Vpbmdsb29w,size_16,color_FFFFFF,t_70)
同理的坑：在我配置好根路径确认无误后，按理说托管显示的网页和本地的应该一致，但是控制台报的错误还是和上次一致，定位错误文件，发现和我现在的仓库路径显示完全不一致，这明显是上次修改的，于是乎稍等刷新重新加载，看network，发现文件都在pending（等待）阶段，一会就正常显示啦！


### 2.4 奇奇怪怪的bug
#### 坑7：github不认识某些tag标签
![在这里插入图片描述](https://img-blog.csdnimg.cn/df7770c363604274abff2bc473dffd57.png)

当时提交时github为黄灯，说明在工作流状态，报了这个说什么不认识这文件下的第一行标签，于是搜了很多方法，发现有一博主给出只需在前面加光合回车就回识别通过，具体为啥也不清楚，试了下发现这篇通过了，接着的下一篇又爆了，没办法老样子全部加回车，发现这地方的错误通过后，下一个标签有报错，当时挺晚困得不行睡了，第二早起床跑一边居然通过了，奇奇怪怪。

工作流状态：
![在这里插入图片描述](https://img-blog.csdnimg.cn/ae3278bbaef04a208bbe756261e19b6f.png)

类似的错误：具体咋产生的我也不清楚，不过清理缓存后过会重新生成就没有了，那个标签显示是对的，只不过内包含了而已。
![在这里插入图片描述](https://img-blog.csdnimg.cn/64a73455e6e84a44970b1c0a293b0ce3.png)
 - Uncaught SyntaxError: Unexpected token '<'


# 3 总结
不管遇到什么bug,不要害怕，勇敢的面对它，加油！！
