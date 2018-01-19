一个应用最多只能打开5个界面，或者用redirectTo

注册界面Page方法中，键值不能直接使用键名，会找不到引用，必须使用this.data.键名才行
```
Page({
    data:{
        text:'text',
        num: 0,
        arr:[{text:'init data'}],
        someObj: {
            text: 'object init'
        }
    },
    setNum: function(){
        this.data.num +=1
        this.setData({
            num: this.data.num,
            'arr[0].text':'data changed',
            'someObj.text':'new object data'
        })
    }
})

```
data里面变量名可以不用引号。
setData中的数组中如果单独设置某个元素的键名则需要加引号，对于单独设置对象的某个属性也是一样的。

setData可以添加回调
```
this.setData({},callback)
```

**小程序中自定义函数的调用需要使用this去调用才能调用成功，否则会报函数未定义错误**

#### {{}}的运算
运算符号在{{}}内外是不一样的，在{{}}里面才发挥作用。比如说+在{{}}里面代表相加，在外面代表单纯的+。比如说x=1,y=2,z=3,d没有定义为变量，则
```
<view>{{x + y}} + {{z}} + d</view>
```
输出3 + 3 + d。<br>
如果x="1",y="2",z=3，则输出结果是12 + 3 + d
<br>
{{}}里面可以进行三目运算、逻辑判断、算数运算、字符串相加。
还有对象属性引用和数组元素引用
```
<view>{{object.key}} {{array[0]}}</view>
```

如果要引用某个数据，只要{{键名}}即可
```
<view> {{ message }} </view>
```
组件属性？：需要包含在""之间
```
<view id="item-{{id}}"> </view>
```
```
Page({
  data: {
    id: 0
  }
})
```
控制属性：同样要在""之间
```
<view wx:if="{{condition}}"> </view>
```
true和false，需要在""之间
```
<checkbox checked="{{false}}"> </checkbox>
```

可以根据现有变量进行组合
```
<view wx:for="{{[zero, 1, 2, 3, 4]}}"> {{item}} </view>
```
```
Page({
  data: {
    zero: 0
  }
})
```

...叫做扩展运算符，用来把一个对象的所有属性显示出来

注意： 花括号和引号之间如果有空格，将最终被解析成为字符串
```
<view wx:for="{{[1,2,3]}} ">
  {{item}}
</view>
```
等同于
```
<view wx:for="{{[1,2,3] + ' '}}">
  {{item}}
</view>
```

{{}}里面可以直接创建数组{{[1,2,3]}}
#### 条件渲染
对于单个view：
```
<view wx:if="{{condition}}"> True </view>
```
多条件判断：
```
<view wx:if="{{length > 5}}"> 1 </view>
<view wx:elif="{{length > 2}}"> 2 </view>
<view wx:else> 3 </view>
```
对于多个view来说：
```
<block wx:if="{{true}}">
  <view> view1 </view>
  <view> view2 </view>
</block>
```
block不是组件，不做渲染，只接收控制属性
#### 列表渲染
在组件上使用wx:for来绑定一个数组重复渲染该组件，默认使用index表示下标，item表示对应项
```
<view wx:for="{{array}}">
  {{index}}: {{item.message}}
</view>
```
注意数组名要用{{}}框起来，否则数组名本身会被看做一个字符数组。
<br>
如果要修改下标名和对应项名称，可以指定如下属性：
```
<view wx:for="{{array}}" wx:for-index="idx" wx:for-item="itemName">
  {{idx}}: {{itemName.message}}
</view>
```

wx:for嵌套示例：九九乘法表
```
<view wx:for="{{[1, 2, 3, 4, 5, 6, 7, 8, 9]}}" wx:for-item="i">
  <view wx:for="{{[1, 2, 3, 4, 5, 6, 7, 8, 9]}}" wx:for-item="j">
    <view wx:if="{{i <= j}}">
      {{i}} * {{j}} = {{i * j}}
    </view>
  </view>
</view>
```

块控制<block wx:for></block>

如果列表位置会动态改变或者有新项添加到列表当中，则列表中输入框或者选择框会丢失状态，要保持状态不变，需要指定wx:key属性，来唯一标识列表中的一个项。

这个key可以有两种，一种是使用数组中元素的某个属性，注意这个属性的值必须是列表中唯一的数字或者字符串，且不能动态改变。

如果数组元素不是对象，那就使用*this来使用数组元素本身作为key，注意数组元素本身的值也必须是列表中唯一的数字或者字符串。

那这个原理的话就是带有key的组件会重新排序，而不是重新创建，提高效率并保持状态。

### 事件处理

#### 事件冒泡
事件绑定格式是bind或者catch开头，加上要绑定的事件，比如bindTap，作为属性名，属性值为Page中定义的函数。1.5.0开始支持新语法bind:事件名="函数名"

bind事件绑定不会阻止冒泡事件向父节点冒泡，catch事件绑定可以阻止冒泡事件向父节点冒泡。

以bindTap为例，添加id属性、data-键名="键值"属性来传递数据
```
<view id='testBindTap' data-hello="world" bindtap='testBindTap'>test bindTap</view>
```
js中可以获取到相关属性：
```
testBindTap: function(event){
  console.log(event)
}
```
输出的event格式为：
```
{
"type":"tap",
"timeStamp":895,
"target": {
  "id": "testBindTap",
  "dataset":  {
    "hello":"world"
  }
},
"currentTarget":  {
  "id": "tapTest",
  "dataset": {
    "hi":"WeChat"
  }
},
"detail": {
  "x":53,
  "y":14
},
"touches":[{
  "identifier":0,
  "pageX":53,
  "pageY":14,
  "clientX":53,
  "clientY":14
}],
"changedTouches":[{
  "identifier":0,
  "pageX":53,
  "pageY":14,
  "clientX":53,
  "clientY":14
}]
}
```
#### 事件捕获
1.5.0开始支持事件捕获，事件捕获在事件冒泡之前，传递顺序是从父节点到子节点，语法格式为capture-bind:事件名="函数名"，或者capture-catch:事件名=:"函数名"。capture-catch会中断捕获阶段和取消冒泡阶段。
```
<view id="outer" bind:touchstart="handleTap1" capture-catch:touchstart="handleTap2">
  outer view
  <view id="inner" bind:touchstart="handleTap3" capture-bind:touchstart="handleTap4">
    inner view
  </view>
</view>
```
### 引入
import可以在该文件中使用目标文件定义的template 
```
<import src="item.wxml"/>
<template is="templateName" data="{{text: 'forbar'}}"/>
```

include可以将目标文件除了<template/>的整个代码引入，相当于是把目标文件拷贝到include位置

```
<!-- index.wxml -->
<include src="header.wxml"/>
<view> body </view>
<include src="footer.wxml"/>
```

### WXS
WXS跟WXML不一样，不受基础库版本影响。

require用来导入WXS

回调的三种写法：
```
wx.getSetting({
  success: (res) => {
    // res要不要加括号都可以
  }
})
```

```
wx.getSetting({
    success(res) {
        
    }
})
```

```
wx.getSetting({
    success: function(res) {
        
    }
})
```

### WXSS
规定屏幕宽为750rpx
@import导入外联样式表
```
@import "common.wxss";
```

内联样式：分为style和class两种。<br>
style用来接收动态样式，不要写入静态样式影响渲染速度。
```
<view style="color:{{color}};" />
```
class用来指定样式规则，使用的样式就是所有指定的类选择器的集合，类选择器不用带.，多个类选择器间用空格隔开
```
<view class="normal_view" />
```

app.wxss为全局样式。各个子wxss为局部样式，会覆盖app.wxss。

### 兼容
可以通过以下方式判断api是否可用
```
if (wx.openBluetoothAdapter) {
  wx.openBluetoothAdapter()
} else {
  // 如果希望用户在最新版本的客户端上体验您的小程序，可以这样子提示
  wx.showModal({
    title: '提示',
    content: '当前微信版本过低，无法使用该功能，请升级到最新微信版本后重试。'
  })
}
```
#### 本地资源路径
```
<image src='../images/img.png'/>
```
假设为wxml所在路径为/pages/index/index.wxml，则../表示上级目录，路径为/pages/images/img.png。
../../表示上两级目录，路径为/images/img.png。

### view默认是块级元素

### 图片处理
宽高比固定充满容器：
```
image {
    width:100%;
}

<image mode='widthFix'></image>
```

### 页面跳转参数传递
使用navigator去跳转的话可以在页面路径中添加参数，然后在新页面的onLoad(options)中使用options去获取参数：
```
<navigator url="../product/product?productId={{item.id}}"></navigator>
```
```
data: {
    productId: 0,
}

onLoad: function(options){
    this.setData({
        productId: options.productId,
    })
}
```

### Html富文本解析
使用WxParse库
https://github.com/icindy/wxParse

第一步：拷贝wxParse文件夹到小程序工程中（本示例中放到pages文件夹中）

第二步：在js文件中根据路径导入wxParse模块
```
var wxParse = require('../../wxParse/wxParse.js')
```

第三步：在wxss文件中根据路径导入wxParse.wxss
```
@import '../../wxParse/wxParse.wxss';
```

第四步：在xml中使用template
```
<import src="../../wxParse/wxParse.wxml"/>
<template is="wxParse" data="{{wxParseData:content.nodes}}"/>
```

第五步：js中根据html内容进行解析

注意，后台返回的html可能在原本已转义过的基础上再进行转义了，比如原本空格已经转义成`&nbsp;`了，然后后台为了安全起见又可能进行转义，变成`&amp;nbsp;`，这种情况下wxParse就只会解析一层，变成`&nbsp;`，无法解析两层变成空格，所以需要我们自己先解析一层，把`&amp;`转成`&`，这种情况下可以通过`str.replace(/&amp;/g,'&')`把`&amp;`替换成`&`，replace中的g是global，全局替换的意思，如果不加g则只会替换第一个。
```
/**
* WxParse.wxParse(bindName , type, data, target,imagePadding)
* 1.bindName绑定的数据名，对应xml中的content这个变量名，两个必须一致(必填)
* 2.type可以为html或者md(必填)
* 3.data为传入的具体数据(必填)
* 4.target为Page对象,一般为this(必填)
* 5.imagePadding为当图片自适应是左右的单一padding(默认为0,可选)
*/
wxParse.wxParse('content', 'html', content, this, 5);
```

注意事项：wxParse.css会设置view的overflow:auto属性，导致一些属性无法正常生效，可以在css中再声明over:visible即可恢复正常。
### HTML富文本循环解析
[wxParse多数据循环使用方法](https://github.com/icindy/wxParse/wiki/wxParse%E5%A4%9A%E6%95%B0%E6%8D%AE%E5%BE%AA%E7%8E%AF%E4%BD%BF%E7%94%A8%E6%96%B9%E6%B3%95/)

使用方法：

（1）首先有个字符串数组，注意是字符串数组，如果是对象数组会报html.replace不是function错误。

（2）第二步，遍历调用每一项的wxParse方法

（3）第三步，调用wxParseTemParse方法

```
for (let i = 0; i < 数组名.length; i++) {
    wxParse.wxParse('与下句同样的变量名' + i, 'html', 数组名[i], that);
    if (i === 数组名.length - 1) {
      wxParse.wxParseTemArray("循环块变量名",'与上句同样的变量名', 数组名.length, that)
    }
}
```
（4）第四步是设置wxml，注意data='{{wxParseData:item}}'的格式是固定的，如果item再去点某个属性也会报错

```
<block wx:for="{{循环块变量名}}" wx:key="">
        回复{{index}}:<template is="wxParse" data="{{wxParseData:item}}"/>
</block>
```

但是以上方法只能使用在数组元素为字符串的情况下，如果数组元素是对象，可以把wxParse语句的数组名改为其他数组，即可使用其他数组的值。


### WxParse循环详细步骤解析
1. js中要有个目标数组
```
var targetArr = []
var item1 = {title:'<p>title1</p>',label:'label1'}
var item2 = {title:'<p>title2</p>',label:'label2'}
var item3 = {title:'<p>title3</p>',label:'label3'}
targetArr.push(item1)
targetArr.push(item2)
targetArr.push(item3)

this.setData({
    targetArr : targetArr
})

var varName = 'arr'

for (let i = 0; i < targetArr.length; i++) {
    WxParse.wxParse(varName + i, 'html', targetArr[i].具体包含HTML格式的属性名, that);
    if (i === targetArr.length - 1) {
        WxParse.wxParseTemArray("对应wxml中的变量名", varName, targetArr.length, that)
    }
}
```
在循环中首先会解析所设置数组的每个html内容，然后在最后把解析到的东西设置给wxml，这样就完成解析了。然后如果我们有自己的非html数据的话，就要通过setData来设置到wxml中，因为wxml中循环不能用真实数组变量去循环，否则会报错html.replace is not a function。
2. wxml文件，注意循环必须是循环wxParseTemArray中的变量名才行。然后循环中的非html数据我们使用真实数组去获取值。
```
<block wx:for="{{对应js中的变量名}}" wx:key="">
    <text>{{targetArr[index].label}}</text>
    回复{{index}}:<template is="wxParse" data="{{wxParseData:item}}"/>
</block>
```
3. 是

### 动画
https://mp.weixin.qq.com/debug/wxadoc/dev/api/api-animation.html

第一步，创建动画实例
```
var animation = wx.createAnimation({
    transformOrigin: "50% 50% 0",
    duration: 2000,
    timingFunction: 'ease',
    delay: 0
})
```
timingFunction定义变化类型、transformOrigin定义变形原点。

第二步，animation实例调用各种动画方法，可以链式调用，最后以.step()来将前面调用的各种动画设为一组，一组动画里的动画是同时进行播放的，一组动画播放完毕才会播放另一组动画。

第三步，调用动画实例的export方法，更新给Page内自定义变量
```
Page({
  data: {
    animationData: {}
  },
  ...
})
```
```
this.setData({
  animationData:animation.export()
})
```
第四步，将该自定义变量绑定到view当中的animation属性
```
<view animation="{{animationData}}" style="background:red;height:100rpx;width:100rpx"></view>
```

可以在调用动画的函数内，调用一个函数setTimeOut，它接收一个函数和一个数值为变量，函数表示动画执行完毕要调用的函数，数值为动画执行完毕后延迟多久调用传入的函数
```
setTimeout(function () {
    // animation.translate(30).step()
    // this.setData({
    //     animationData: animation.export()
    // })
    console.log('time has been out')
}.bind(this), 5000)
```
### 文本
使用text标签，内容使用js中的data数据，文本内容使用\n来换行。

文本缩进，必须要块级元素才能缩进，内联元素无法缩进，所以原生的text标签无法缩进，除非是设成inline-block

### flex布局
小程序中flex布局的flex权重属性，在flex-direction:row的情况下可以正常生效，flex弹性盒子的宽是固定的。

但是当flex-direction:column的情况下，需要手动指定flex弹性盒子整体的高，盒子子元素的flex权重属性才能正常生效。初步猜测原因是默认高度是可延伸的。

### scrollview
scrollview通过设置white-space:nowrap来设置不换行，即可以水平滑动

scroll-view的直接子元素如果有box-shadow，则底部的shadow会因为scroll-view的限制无法显示出来，此时可以给直接子元素设置一个margin-bottom把空间空出来显示shadow

### 实现类似Android TabLayout+ViewPager布局
方法1：

Tab
```
.typeTabs {
    position: fixed;
    z-index: 1;
}
```
swiper
```
<swiper style='height:{{contentHeight}}px;position:relative;top:{{tabHeight}}px;' bindchange='onPageChange' current='{{pageIndex}}'>
</swiper>
```

上面这样如果swiper里的swiper-item里直接放view的话，难以计算内容高度，而没设置高度的话，swiper就无法完全显示。

方法2：使用js动态计算tab高度，然后剩下的高度给swiper，swiper高度就固定了，然后swiper-item里面放一个scroll-view，scroll-view的高度也是设成跟swiper一样的，这样就可以不用计算内容高度了。但是实际实施时，如果底部有原生TabBar，好像会差一个tab的高度，导致scroll-view滑到底时还会tab和scrollview都往上滑动一个tab高度，解决方法可以结合方法1，把tab设成fix的。

### position
position设为fixed之后,块级元素会变成内联元素？反正宽不会自动充满了。

### image
image的mode也可能会影响到image的宽高

image如果是widthFix，会根据宽度，去生成实际高度，此时即使指定了高度也无效。可能出现实际高度比指定高度高的情况。

如果高度需要是固定的话，最好使用aspectFill，指定高度为一个固定值，然后上传的时候要根据750rpx去比设定的高度，得到宽高比，根据这个比例去上传图片，保证显示完全。

### scroll-view
水平滑动的scroll-view如果设置左右padding会导致视窗可以左右滑动，所以最好不要设置左右padding。

### 页面背景
```
page {
    background-color:#123456;
}
```

### input输入框
bindConfirm属性设置完成按钮的触发事件，比如为search，然后可以通过e.detail.value来获取输入的值
```
search: function(e){
    console.log(e.detail.value)
}
```

### image加上一个有向上偏移的relative的view加上一个scroll-view
如果要准确计算scroll-view使得刚好充满整个窗口不滚动，则计算方法最好是image设置固定高度，然后通过windowHeight-relative的bottom-偏移高度的px值这样去算才会刚好，如果通过windowHeight-image高度-relative高度得出的是不一样的，应该是实际上并没有像第二种这样排列。
