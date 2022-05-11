---
title: 【深信服实习】一些模块开放中的感悟
cover: https://img-blog.csdnimg.cn/92d833800d364dca8ae9df4eb0c44d51.png
description: 三个月的实习结束，非常不舍得我的逗比同事，我的560项目，希望项目能保质交付吧。感谢新哥导师和努力的自己，技术增长了很多，明白了很多，愚钝者多学，涅槃重生。
categories: 'Bug排错'
---

![在这里插入图片描述](https://img-blog.csdnimg.cn/ea105c3cec7b4b9994f9d97ee178c606.png)
![在这里插入图片描述](https://img-blog.csdnimg.cn/8605fac1cf5a4f95a05a284a6b5921fc.png)
> 领导问我，会给这次经历打几分，我说80。
> 导师和组长每隔一个月，就会问我，项目进度怎么样，有没有进步，我都不自觉跟他们说些掏心窝子的话。
> 三个月其实很累，身体明显感觉到了，但我却乐此不疲。我很喜欢我所在的团队，喜欢这的学术研发氛围，喜欢同事牛逼的技术，总想着有一天，期望成为新哥，凯哥，mzj那样厉害的人物



#### 1.table-item不显示（没给table高度导致）
描述：数据是这样的[{},{}] 对象数组形式，能获取到数据，能成功给 `gridOptions` 赋值，但就是显示不出

![在这里插入图片描述](https://img-blog.csdnimg.cn/37598aaadb8b4a73823b79e7517d3bcc.png#pic_center)
修复思路：

- 有自定义组件 `SfTableOptions` 和 `useTableOptions` hook辅助编码，最开始选第一个，打印 `gridOptions` 发现是undefined，然后用第二钟方法。
- 后面觉得数据从获取到赋值这一整个过程都没问题，打印的结果都理想，就猜测是内容高度不够问题。


<font style="background:#c0d6cb;color=green"   size=4>其实像这种功能正常，数据流也很合理的错误，我犯过好几个，都在想是不是所用的hook或方法返回不对，往往遗忘了渲染的条件</font>

#### 2.组件复用过程
描述：认证算法和加密算法非无选项都用到了密码和确认密码小组件，属于高频使用了，在没复用之前，遇到的BUG有

- 因为在同一个form表单，ref值相同，这导致在做radio-switch表单检验的时候，校验规则不知现在监听的是那个item项，导致相同也会报错
![在这里插入图片描述](https://img-blog.csdnimg.cn/1a01b1877bd0416eabce6391485375ac.png#pic_center)![在这里插入图片描述](https://img-blog.csdnimg.cn/e819d42d60e340c2ab14bc58d2a13cea.png#pic_center)
解决方案：抽离公共代码，复用成组件，想法是：把ref和对应的disabled条件传入，在组件中利用组件的setData和getSubmitData方法拿到传入和传出的数据，在通过watch监听那两个input的值，进行ref.value.validate()的标红告警清除

```javascript
<snmp-password
 	ref="X1"
:disabled="" />
    
//清除校验
watch(() => [A, B], v =>{
    ARef.value?.validate();
    BRef.value?.validate();
});
```

解决，但来了个需求，认证算法共用一个数据源，就是一个改，切换另一个radio也跟着改，用现在的组件实现比较复杂，因为得传入三个ref值，得对三个radio项进行判断，看是哪个传入，在渲染的时候对页面视图同步不方便，提交也多种分情况。

于是，进一步优化：认证算法共用一个数据源，利用v-model传入数据对象进去，子组件在渲染，这样就省去了提交的判断。

```javascript
//将数据对象通过v-model传入
const A = ref({
    password: '',
    confirmPassword: ''
});

//子组件

//定义数据类型
const props = defineProps<{
    disabled: boolean;
    value: {
        password: string;
        confirmPassword: string;
    };
}>();

const emit = defineEmits(['input']);
const formData = computed({
    get() {
        return props.value || {
            password: '',
            confirmPassword: ''
        };
    },
    set(v) {
        emit('input', v);
    }
});
```

完美解决！




#### 3.if else条件判断性能优化

有这么一个场景：foreach遍历原始表格数组，对要编辑的表格行进行key排查，改行不能先找到改行数据，修改时再看有无重名地址，最后提交，下面是待优化的，用了很多if-else条件判断

```javascript
newGridData.forEach((value, index) => {
        const trapArray = newGridData.map(item => item.trapAddr);
        let same = trapArray.find(value => value == data.trapAddr);
        if (value.trapAddr === id) {
            if (data.trapAddr === id){
                newGridData[index] = data;
                isSend = true;
            } else if (same === undefined) {
                newGridData[index] = data;
                isSend = true;
        } else {
       		fail();
    	}
}
```

优化后，先对不满足的条件判断，接着退出，最后剩下的就是满足的，

```javascript
if (value.trapAddr !== id) {
    return;
}
if (data.trapAddr !== id && typeof same !== 'undefined') {
    fail(tr('snmpOptMgt.edit'));
    return;
}
newGridData[index] = data;
isSend = true;
```

#### 4.查找元素优化

注释的写法太小白了，我要找的就是所选数组下标，为此专门写了个所有key的数组，在通过传入的key在这数组查找，在比对，饶了一大远路，忘记了findIndex方法，这波是忘记专门的API，拿到需求直接上手开造造成的逻辑缺陷。

```javascript
    // 1.根据传入的id找到当前行的数据
    const rowIndex = newGridData.findIndex(item => item.trapAddr === id);
    // 2.判断当前修改数据的id是否存在于旧数据中
    const sameIndex = newGridData.findIndex(item => item.trapAddr === data.trapAddr);
    if (sameIndex !== -1){
        fail(tr('snmpOptMgt.edit'));
        return;
    }
    // 3.修改
    newGridData[rowIndex] = data;
    isSend = true;

    // newGridData.forEach((value, index) => {
    //     // 定义一个表格所有数据key的数组
    //     const trapArray = newGridData.map(item => item.trapAddr);
    //     // 查找trapAddr,存在就返回数值没有返回undefined
    //     let same = trapArray.find(value => value == data.trapAddr);
    //     // 如果选中行的地址和原始数据的地址不同就退出
    //     if (value.trapAddr !== id) {
    //         return;
    //     }
    //     // 如果修改的地址存在原始数据中就退出
    //     if (data.trapAddr !== id && typeof same !== 'undefined') {

    //     }
    //     newGridData[index] = data;
    //     isSend = true;
    // });
```

#### 5.radio选择优化

有这么个条情况，两个radio的item项，有个change事件，只在由启用变为禁用的时候，才会触发确认弹框。

![在这里插入图片描述](https://img-blog.csdnimg.cn/7119f10b6c9e4e9197ded78c3c3f7775.png#pic_center)
于是我我把那两种情况写了个遍，由启用变为禁用，或者初始值为禁用，更改后（还写了变量判定）提交值变为禁用，才触发，很不错啊，思路很清晰。但这条件一判定下来，就又臭又长了。

```javascript
watch(()=>formData.value.enableSDK, v=>{
    // 判断radio是否修改
    if (v !== oldData?.enableSDK && oldData?.enableSDK !== undefined){
        sendFlag.value = true;
    }
});

const openApiChange = (enableSDK: string) => {
    // 启用：1，禁用：0
    // 进入判断的两种情况：由启用更改为禁用;由禁用改为启用在禁用
    if ((enableSDK === '0' && oldData?.enableSDK === '1')
        || (oldData?.enableSDK === '0' && sendFlag.value && enableSDK === '0')){
        let modal = useModal(tr('sysCfg.mail.mail-title'), {
            title: tr('sysCfg.mail.confirm_title'),
            width: MODAL_SIZE_MAP.md,
            icon: 'confirm',
            autoDestroy:true,
            submit: () => {
                formData.value.enableSDK = '0';
                modal.hide();
            },
            cancel: () => {
                formData.value.enableSDK = '1';
                modal.hide();
            }
        });
        modal.open();
    }
```

咋一看没啥问题哈，很不错，但大有优化空间，预期判定触发有几种条件，不如判定触发的共性！

说白了就是当处于禁用的时候触发嘛，那就优化！！

```javascript
const openApiChange = (enableSDK: string) => {
    if (enableSDK === '1' || isInit){
        return;
    }
    const modal = useModal({
        submit: () => {
            modal.hide();
        },
        cancel: () => {
            formData.value.enableSDK = '1';
            modal.hide();
        }
    });
    modal.open();
};
```

这次我们直接判定为触发的条件，然后退出，剩下的就是要触发的了。

首先第一种情况，就是判断当前是启用状态，或者第二种是获取数据失败的（定义个初始值为true的判断变量，当初次加载成功获取接口数据时置为false），这两都退出，剩下的就爆弹窗，甚至可以把确定按钮的赋值操作省去，简单明了。

> 持续学习吧，去成为想成为的人，去想去的地方
