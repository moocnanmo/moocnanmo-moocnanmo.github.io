---
title: 仿网易云音乐的小程序项目（粗糙版）
date: 2021-08-23 18:02:35
description: 一个简简单单，用来上手小程序开发的音乐项目，利用跨站请求伪造 (CSRF), 伪造请求头 , 调用官方 API，获取真实网易云数据。
cover: https://cn.bing.com/th?id=OHR.LittleBlueHeron_ZH-CN0892428603_1920x1080.jpg&rf=LaDigue_1920x1080.jpg
categories: '项目类'
---

# 前言
------

本项目是参考B站的小程序开发视频跟学而成，数据为真实的网易云音乐数据，接口参考至下方链接，参照视频学习 **只完成了** ：

 - 每日推荐的歌曲，歌单，视频页面，账号登陆的展示数据信息功能
 - 歌曲和视频的播放切换，进度条实时跟进功能
 - 视频区下拉刷新，搜索歌曲功能
 

开发完成后最大的感受就是一定要会看，会查微信小程序的官方API文档，出bug时首先检查语法是否正确（刚入门比较容易犯拼写错误），根据控制台给错的错误信息定位到相应文件，进行逻辑检查，注意后端返回的数据类型和和格式转换，看清接口文档明白后端要接收到的实际参数！！

> * [微信小程序官方文档](https://developers.weixin.qq.com/miniprogram/dev/api/)
>*  [网易云接口调用文档](https://neteasecloudmusicapi.vercel.app/#/?id=%E6%8E%A5%E5%8F%A3%E6%96%87%E6%A1%A3)
> * [源码](https://gitee.com/nanmooo/nan_music.git)


页面预览：
![在这里插入图片描述](https://img-blog.csdnimg.cn/57f7e296b3e749688b229455ef807f72.png#pic_center)
# 功能介绍：获取诸如每日推荐的静态数据
诸如上述的每日轮播图，歌曲，歌单数据，获取方式十分简单，只需要参考调用文档，发送GET
数据请求，解析收到的数据，将需要的信息展示到预先设好的图片插槽上即可。

在相应模块中的js文件里设置存放诸如每日轮播图，歌曲，歌单数据的数组

```javascript
 data: {
    // 轮播图数据
    bannerList: [],
    // 精心推荐歌单数据
    recommendList: [],
    // 排行榜数据
    topList: [],
  },
```
然后在onLoad声明周期函数中，调用接口（注意文档所要求携带的参数），获取数据，将接口返回的数据通过setData方法保存至刚设置的数组里。

```javascript
 onLoad: async function (options) {
    let bannerListData = await request('/banner', { type: 2 });
    this.setData({
      bannerList: bannerListData.banners
    })

    // 获取推荐歌单数据
    let recommendListData = await request('/personalized', { limit: 10 });
    this.setData({
      recommendList: recommendListData.result
    })

    // 获取排行榜的数据
    let index = 0;
    let resultArr = [];
    while (index < 5) {
      let topListData = await request('/top/list', { idx: index++ });
      let topListItem = { name: topListData.playlist.name, tracks: topListData.playlist.tracks.slice(0, 3) }
      resultArr.push(topListItem);
      this.setData({
        topList: resultArr
      })
    }
  },
```
最后通过插槽展示出来，对于一些组件，可以添加官方给出的样式属性，让布局的交互效果更佳。

```javascript
 <!-- 轮播图区域 添加的属性：自动播放，切换时间间隔，下方小圆点 -->
  <swiper autoplay interval="4000" class="banners" indicator-dots="true" indicator-color="ivory" indicator-active-color="#d43c33">
    <swiper-item wx:for="{{bannerList}}" wx:key="bannerId">
      <image src="{{item.pic}}"></image>
    </swiper-item>
  </swiper>
```
推荐歌曲部分大抵如此：

```javascript
	<!-- 推荐歌曲内容区 -->
    <scroll-view class="recommendScroll" enable-flex scroll-x>
      <view class="scrollItem" wx:for="{{recommendList}}" wx:key="id" wx:for-item="recommendItem">
        <image src="{{recommendItem.picUrl}}"></image>
        <text>{{recommendItem.name}}</text>
      </view>
    </scroll-view>
```

榜单部分：这里以item项进行滑动，每个item项宽度固定，只需调节宽度大小即可控制下一item项在页面的显示位置。
![在这里插入图片描述](https://img-blog.csdnimg.cn/bff57f5dc7f14bd4b4927ae3f990b8b7.png#pic_center)


```javascript
    <swiper class="topListSwiper" circular next-margin="110rpx">
      <swiper-item class="swiperBack" wx:for="{{topList}}" wx:key="name">
        <view class="title">{{item.name}}</view>
        <view class="musicItem" wx:for="{{item.tracks}}" wx:key="id" wx:for-item="musicItem">
          <image src="{{musicItem.al.picUrl}}"></image>
          <text class="count">{{index + 1}}</text>
          <text class="musicName">{{musicItem.name}}</text>
        </view>
      </swiper-item>
    </swiper>
```

# 功能介绍：用户登陆
整个流程大致为：

 1. **前端验证**
    1. 实时收集用户输入框数据（给账密绑定 id ,通过事件委托实时存储用户输入值）
    2. 验证账号密码是否合法（验证是否为空，格式正确，长度这几种情况）
    3. 验证通过就将账密发个请求给客户端，反之不通过则提示用户，不用发请求
 2. **后端验证**
 	1. 调用数据库验证用户是否存在
 	2. 用户存在即验证密码是否正确，跟下一步一样，账密任何一个输错停止下一步，并告诉前端**错误
 	3. 都正确就返回用户的数据给前端，并提示登陆成功
 	
 	
在实时存储和验证用户输入的账号密码时用的是同一个处理函数，怎么却分他们谁是谁呢，一般用id进行区别。这里有个重要知识点：**事件委托**。事件委托就是把子元素的事件让父元素完成，可通过`event.target` 找到触发的子元素，`currentTarget` 要求绑定事件的元素一定是触发事件的元素。

根据id存储信息

```javascript
let type = event.currentTarget.dataset.type;
    this.setData({
      [type]: event.detail.value
    })
```

点击登陆，携带账号密码信息触发后端验证（状态码为：200通过，400手机号错误，502密码错误），`wx.showToast` 显示登陆情况。成功后 `wx.setStorageSync` 将用户信息保存到本地（存储为json字符串格式类型），接着 `reLaunch` 重定向到某页面。

```javascript
wx.setStorageSync('userInfo', JSON.stringify(result.profile))
```
```javascript
wx.reLaunch({
        url: '/pages/personal/personal'
      })
```



# 功能介绍：视频播放
网易获取视频需要用户登陆，用户登陆后把保存在本地的cookies取出。因为项目是做成上边导航栏，下边根据导航栏的索引 id 获取相关视频的开发模式，所以先获取导航栏的数据，和标签id分别以数组的形式存放在data数据中。
![在这里插入图片描述](https://img-blog.csdnimg.cn/af7f9e6f9ba64fcf9a3c0f39b58c6235.png#pic_center)
获取导航栏数据方式跟先前拉取静态歌曲方式一样，均通过异步请求，来获取保存显示数据。不过这里有些不同，官方给的导航api中，标签id不能以数字开头，所以在显示时，均加以 ”scroll“ 字符串来满足要求，

```javascript
<!-- 视频导航栏区域 -->
 <view id="{{'scroll' + item.id}}" class="navItem" wx:for="{{videoGroupList}}" wx:key="id">
      <view class="navContent {{navId === item.id?'active': ''}}" bindtap="changeNav" id="{{item.id}}" data-id="{{item.id}}">
        {{item.name}}
      </view>
    </view>
```
默认是第一个标签页视频，在点击第一个导航栏标签时，会将上面的字符串更改为数值类型，同时初始化该变迁下的存储视频数组，接着携带id触发获取视频调用函数。因为获取的视频里，单个视频没有唯一的id，所以我需要用map加工一下

```javascript
  async getVideoList (navId) {
    if (!navId) { // 判断navId为空串的情况
      return;
    }
    let videoListData = await request('/video/group', { id: navId });
    // 关闭消息提示框
    wx.hideLoading();
    let index = 0;
    let videoList = videoListData.datas.map(item => {
      item.id = index++;
      return item;
    })
    this.setData({
      videoList,
      isTriggered: false // 关闭下拉刷新
    })
  },
```
到这里，视频获取就算完成额了，但是存在一些小问题。比如：同时播放多个视频。不能对视频进行上一次播放进度的跟踪。

## 解决多个视频同时播放的问题(单例模式)和回到上个视频的播放进度
单例模式主要思想：在多个对象的场景下，通过一个变量接收，始终保持只有一个对象。那么用户在播放第二个视频时，我们需要通过上述map生成的唯一id判断这俩者不是同一个视频。

回到上个视频的播放进度的整体思路是：当我们每点击一个视频，存储该视频id,创建一个事件对象，

```javascript
 let vid = event.currentTarget.id;
    this.setData({
      videoId: vid
    })
 this.videoContext = wx.createVideoContext(vid);
```
查看他是否有播放记录，有就跳转到指定位置，没有就重头开始播放，然后实时将记录存储到刚创建的对象里的记录播放时长属性中。

```javascript
 // 判断当前的视频之前是否播放过，是否有播放记录, 如果有，跳转至指定的播放位置
    let { videoUpdateTime } = this.data;
    let videoItem = videoUpdateTime.find(item => item.vid === vid);
    if (videoItem) {
      this.videoContext.seek(videoItem.currentTime);
    }
```
关于播放时长的回调，播放前需判断播放时长是否有值，有就覆盖，在播放。

```javascript
if (videoItem) { // 之前有
      videoItem.currentTime = event.detail.currentTime;
    } else { // 之前没有
      videoUpdateTime.push(videoTimeObj);
      // 有bug
      this.videoContext.play();
    }
    // 更新videoUpdateTime的状态
    this.setData({
      videoUpdateTime
    })
```
再有就是视频播放完后，需要善删掉视频播放状态

```javascript
videoUpdateTime.splice(videoUpdateTime.findIndex(item => item.vid === event.currentTarget.id), 1);
```

下拉刷新，重新根据标签id发数据请求：上拉同理

```javascript
  handleRefresher () {
    // 拿到数据后更改控制回弹属性是否为false即可
    this.getVideoList(this.data.navId)
  },
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/b9b924284531442c9828b1128389e25d.png#pic_center)
![在这里插入图片描述](https://img-blog.csdnimg.cn/2f69f3c00ed547e4ad375dc35acbdab5.png#pic_center)
# 功能介绍：音乐播放
界面预览：
![在这里插入图片描述](https://img-blog.csdnimg.cn/49829c0bd49e4222a07f1c5c9bb88b2a.png#pic_center)

上面的日期采用 `date.getMonth()`  来定义进行实时显示，歌曲部分除付费歌曲外，都能通过接口调用获取。获取前需要验证用户是否登陆，通过先前登陆存储在本地的 `userInfo` 进行判断，没登陆即 `reLaunch` 到 login 页面。

这里稍提一下图像文本居中排布问题：![在这里插入图片描述](https://img-blog.csdnimg.cn/42d52b1f36664e3b8dcbd41316c82525.png#pic_center)
例如左侧图片，右侧文字需要做到居中对其、文本框内图标，与默认文字对其的情况。
根据子绝父相设置定位，接着父子元素都要设置 ` flex` 弹性布局。每首歌曲是一个item项目，给图片设置宽高圆角底边距后（不用设浮动），右侧的文字也将看作是一个item，设置flex，和经典居中三命令执行。

```css
.musicInfo text{
  height: 40rpx;
  line-height: 40rpx;
  font-size: 24rpx;
  white-space: normal;
  overflow: hidden;
  text-overflow: ellipsis;
}
```
最右侧图标利用绝对定位加边距调整即可；图标在搜索框中与文字齐平，图标可用 `vertical-align` 来设置center值。

居中排版问题说完，继续说音乐播放相关的事。

**音乐暂停播放键的显示**：
对暂停还是播放的图标有俩个类名来取决显示那个图标，默认暂停（isplay：false），设置isPlay属性，没点一次通过 ` setData（）` 对 isplay 进行取反更新操作。



**点击歌曲后跳转到播放页面获取详情信息**：
点击歌曲后，需要携带所点击的歌曲的名称，id信息发送到播放页面
```javascript
url: '/songPackage/pages/songDetail/songDetail?musicId=' + song.id
```
播放页面那边接受id发送请求获取该歌曲详情信息，其中durationTime为已播放时长。

```javascript
 // 获取音乐功能详情的功能函数
  async getmusicInfo (musicId) {
    let songData = await request('/song/detail', { ids: musicId });
    let durationTime = moment(songData.songs[0].dt).format('mm:ss');

    this.setData({
      song: songData.songs[0],
      durationTime
    })

    wx.setNavigationBarTitle({
      title: this.data.song.name
    })
  },
```

**播放或暂停音乐**：
播放一首歌曲，需要知道该播放或暂停键的isPlay状态，音乐id，和该音乐的链接地址，音乐的链接地址可通过id向接口发送请求获取。


**小问题：当播放一首歌退出界面在点进来，会重头播放**：
这时只需判断全局下的播放状态为真和id等于现在点的这首歌id就改变播放按钮状态即可，相同则不做播放处理。

```javascript
if (appInstance.globalData.isMusicPlay && appInstance.globalData.musicId === musicId) {
      this.setData({
        isPlay: true
      })
    }
```

**上一首或下一首切换**：
给上一首或下一首分别绑定id值，但用统一处理事件，用户点击时连同id值一起传入，通过播放事件判断是上一首还是下一首，上一首的话就把当前歌曲id所在每日推荐的位置减一进行播放，反之+1。
点击的同时需要暂停当前播放曲目，

```javascript
  handleSwitch (event) {
    let type = event.currentTarget.id;

    // 点以下一首后关闭当前音乐
    this.backgroundAudioManager.stop();

    // 定于recommend的页面数据
    PubSub.subscribe('musicId', (msg, musicId) => {

      // 拿到id后切换上下一首的歌曲详情信息，在调用是否播放函数
      this.getmusicInfo(musicId);
      this.musicControl(true, musicId);
      // 拿到id后在取消订阅，因为订阅依赖回调，会累加
      PubSub.unsubscribe('musicId')
    })

    // 发布消息给recommend页面
    PubSub.publish('switchType', type)
  },
```


```javascript
PubSub.subscribe('switchType', (msg, type) => {
      let { recommendList, index } = this.data;
      if (type === 'pre') {
        (index === 0) && (index = recommendList.length)
        index -= 1;
      } else {
        (index === recommendList.length - 1) && (index = -1)
        index += 1;
      }
      this.setData({
        index
      })
      let musicId = recommendList[index].id;
      PubSub.publish('musicId', musicId)
    });
```

还会继续开发，计划完善
- 各类歌单功能：点击歌单，跳转到歌单列表，再点击歌曲，进行播放
- 显示歌曲的歌词，应该有现成的库可直接调用
- 显示歌曲评论，完成歌曲的交互功能
- 根据搜索名称得到单曲，歌单，mv之类的数据

呃呃呃，，，差不多这样
