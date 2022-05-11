---
title: '基于项目开发的MonggoDB基本使用命令'
data: '2021-8-14 00:00:01'
auto_excerpt:
cover: https://img-blog.csdnimg.cn/20210814204849758.jpg?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L20wXzQ2ODkyNDky,size_16,color_FFFFFF,t_70
categories: '工具类'
---

# 基于项目开发的MonggoDB基本使用命令
------
### 1.关于MonggoDB的简单介绍

> MongoDB 是由C++语言编写的，是一个基于分布式文件存储的开源数据库系统，旨在为WEB应用提供可扩展的高性能数据存储解决方案。MongoDB 将数据存储为一个文档，数据结构由键值(key=>value)对组成。

 - [中文手册](https://www.mongodb.org.cn/manual/)
 - [官网](https://www.mongodb.com/)
 - [可视(图形)化管理工具下载地址](https://www.mongodb.com/try/download/enterprise)
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210602160140522.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3Rha2Vpbmdsb29w,size_16,color_FFFFFF,t_70)【注意】在MongoDB版本中，是偶数：如3.2.x、3.4.x、3.6.x表示正式版(可用于生产环境)，奇数：3.1.x、3.3.x、3.5.x表示开发版。根据自己情况选择。
------

### 2.数据库连接

mongodb的默认端口为27017，连接时后面接数据库名称，`{ useNewUrlParser: true, useCreateIndex: true}`在控制台过滤冗余信息。
```javascript
// 引入数据库处理模块
const mongoose = require('mongoose');
// 数据库连接
mongoose.connect('mongodb://localhost:27017/myweb', { useNewUrlParser: true, useCreateIndex: true})
	.then(() => console.log('数据库连接成功'))
	.catch(() => console.log('数据库连接失败'));
```
创建其他集合,无非三步：

 1. 导入monggodb模版
 2. 创建集合规则
 3. 创建集合，并导出

```javascript
const mongoose = require('mongoose');
const userSchema = new mongoose.Schema({
    username: {
        type: String,
        required: true,
        minlength: 2,
        maxlength: 20
    },
    email: {
        type: String,
        unique: true,
        required: true
    },
    password: {
        type: String,
        required: true
    },
    role: {
        type: String,
        required: true
    }
});

// 创建集合
const User = mongoose.model('User', userSchema);

module.exports = {
    // 将用户集合作为模块成员导出
    User
}
```

```javascript
// createUser();// 初始化一个用户，
//此为我的一个blog用户集合，此处初始化一个用户，方便登陆
async function createUser() {
    const salt = await bcrypt.genSalt(10);
    const pass = await bcrypt.hash('11', salt);
    const user = await User.create({
        username: 'nanmo',
        email: 'qq@qq.com',
        password: pass,
        role: 'admin',
    });
}
```

![在这里插入图片描述](https://img-blog.csdnimg.cn/20210602161213784.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3Rha2Vpbmdsb29w,size_16,color_FFFFFF,t_70)![上述blog项目用户集合](https://img-blog.csdnimg.cn/20210602162536839.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3Rha2Vpbmdsb29w,size_16,color_FFFFFF,t_70)某个blog项目用户集合，处于安全性考虑，将用户密吗通过`bcrypt.hash`加密处理。

---

### 3.基本数据库命令

```javascript
db.userl.find().help();		//Help查看命令提示
show dbs  //查看数据库
use student  //切换数据库
show collections  //查看表
db.version();  //查看当前db版本


//基于集合规则插入数据    --增删查改--
//添加用户
db.集合名.insert({name:"",age:xx})
db.user.insert({"name":"suyl","sex":"男",age:32});
 
 
//删除用户
db.集合名.remove(条件[,是否删除一条])
db.users.remove({age: 132});
db.user.remove({}) 		//删除表数据
db.dropDatabase() 		//删除当前数据库


//查询用户
db.集合名.find({查询条件},{查询列})
db.user.find({"sex":"男"});  //查找性别男
db.user.find({age:{$gt:10}})  //查找年龄大10岁
//$gt大于 $gte 大于等于 $lt小于  $lte小于等于 $ne 不等于
db.user.find({"name":/su/});  //查找名带su的用户
db.user.find().limit(2)      //查找前两条数据
db.user.find().skip(1)		//查找后一条数据


//修改用户信息
db.集合名.update(条件，新数据[,是否新增，是否修改多条])
db.user.update({"name":"张三"},{"name":"李四","sex":"男",age:32})


//其他命令
db.user.find({}).sort({"age":1})  //升序
db.user.find({}).sort({"age":-1})  //降序
db.student.getIndexes()		//查看当前集合索引
db.student.dropIndex({"name":1}) 		//删除索引
```

到此，完毕



