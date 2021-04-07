# ProgramID:wxdacac6b4f7605911

# flex布局

```css
.menu{
  height: 1000px;
  display: flex;
  /* 规定主轴的方向 row/colum */
  flex-direction: row;
  /* 在主轴方向如何展示 flex-start/flex-end/space-between/space-around/center */
  justify-content: space-around;
  /* 在副轴方向如何展示 flex-start/flex-end/space-between/space-around/center */
  align-items: center
}
```

像素用 **rpx** rpx规定所有的屏幕宽度都为750rpx

# 跳转

## 对标签绑定点击时间

```html
// index.wxml
<view bindtap="clickMe" data-nid="123" data-name="hb">点我跳转</view>
```

## 页面跳转

```js
// index.js  
clickMe: function (e) {
    console.log('clickme')
    var nid = e.currentTarget.dataset.nid
    
    // 跳转(非NavicatBar页面)
    wx.navigateTo({
      url: '/pages/redirct/redirct?id=' + nid,
    })
  }
```

## 接收参数

```js
// redirct.js
/**
   * 生命周期函数--监听页面加载
   */
  onLoad: function (options) {
    console.log(options.id)
  },
```

## 通过标签跳转

```html
// 跳转(非NavicatBar页面)
<navigator url="/pages/redirct/redirct?id=666">点我跳转</navigator>
```

# 数据绑定

```html
<view>数据：{{message}}</view>
<button bindtap="changeData">点击修改数据</button>
```

```js
  data: {
    message:"bbboy"
  },

  changeData: function () {
    // 获取数据
    console.log(this.data.message)

    // 修改数据(不能修改前端数据 只能改后端)
    this.data.message = 'ojbk'

    // 修改数据
    this.setData({message: 'ojbk'})
  }
```

## UserInfo相关修改

### 方式一(官方不推荐)

```html
<view>
  当前用户头像:
  <image src="{{path}}" style="height:200rpx;width:200rpx"></image>
</view>
<view bindtap="getUsername">获取当前用户名</view>
```

```js
  getUsername: function(){
    var that = this
    console.log(name)
    wx.getUserInfo({
      success: function(res){
        // 调用成功后触发
        console.log('success', res.userInfo.nickName)
        that.setData({
          name: res.userInfo.nickName,
          path: res.userInfo.avatarUrl
        })
      },
      fail: function(res){
        // 调用失败后触发
        console.log('fail', res)
      }
    })
  }
```

### 第二种(官方推荐)

```html
<view>
  当前用户头像:
  <image src="{{path}}" style="height:200rpx;width:200rpx"></image>
</view>
<button open-type="getUserInfo" bindgetuserinfo="xxx">获取信息</button>
```

```js
  xxx: function(){
    var that = this
    wx.getUserInfo({
      success: function(res){
        // 调用成功后触发
        console.log('success', res.userInfo.nickName)
        that.setData({
          name: res.userInfo.nickName,
          path: res.userInfo.avatarUrl
        })
      },
      fail: function(res){
        // 调用失败后触发
        console.log('fail', res)
      }
    })
  }
```

注意事项：

​	-想要获取用户信息，必须经过用户授权(button)

​	-已授权

​	-未授权 通过调用wx.openSetting({})

## 获取用户地址信息

```html
<view bindtap="getLocalPath">当前位置:{{location}}</view>
```

```js
  getLocalPath: function(){
    var that = this
    wx.chooseLocation({
      success: function(res){
        console.log(res)
        that.setData({
          location: res.address + res.name
        })
      }
    })
  }
```

# for指令

```html
<!--pages/goods/goods.wxml-->
<text>商品列表</text>
<view wx:for="{{details}}">{{index}}-{{item}}</view>
<view wx:for="{{userInfo}}">{{index}}-{{item}}</view>
```

```js
  data: {
    details: ['dog', 'cat', 'elephent'],
    userInfo: {
      name: 'alice',
      age: 18
    }
  }
```

## 获取图片

```html
<text bindtap="uploadImage">上传图片</text>
<view class="container">
  <image wx:for="{{imageList}}" src="{{item}}"></image>
</view>
```

```js
  data: {
    imageList: ['/static/1.jpeg', '/static/2.jpeg', '/static/3.jpeg']
  },

  uploadImage: function(){
    var that = this
    wx.chooseImage({
      count: 9,
      sizeType: ['original', 'compressed'],
      sourceType: ['album', 'camera'],
      success: function(res){
        // 默认的图片 + 选择的图片
        var newList = that.data.imageList.concat(res.tempFilePaths)
        that.setData({
          imageList: newList
        })
      },
      complete: function(){
        // 无论怎么样最终都会执行
      }
    })
  }
```

注意：图片只是上传到了内存

# 双向绑定

```html
<input type="text" value="{{phone}}" bindinput="bindPhone" placeholder="请输入手机号"></input>
```

```js
  data: {
    phone: ''
  },

  bindPhone: function(e){
    this.setData({
      phone: e.detail.value
    })
  }
```

# 手机号登录

## 网络请求API

```js
wx.request({
      url: 'http://127.0.0.1:8000/api/login/',
      data: {
        phone: this.data.phone,
        code: this.data.code
      },
      method: 'POST',
      success: (result) => {
        console.log(result)
      },
      fail: (result) => {
        console.log(result)
      },
    })
```

- 在使用 wx.request 等有网络请求的API时，需要遵循：

    -网络地址https

    -后台必须设置要访问的域名

## api

- 创建虚拟环境
- 在虚拟环境中
    - django==1.11.7
    - drf
- 项目API
    - login
    - message

# 登录

- 小程序公共对象

    ```js
    // App.js
    App({
        gloableData: {
        userInfo: null
      },
    })
    ```

- 其他页面操作公共值

    ```js
    var app = getApp()
    Page({
        data: {
            
        },
        onShow: function(){
            app.gloableData
        }
    })
    ```

    注意：修改globalData之后 其他页面引用的值不会自动变化 需要手动设置

- 本地存储

    ```js
    wx.getStorageSync('userInfo')
    wx.setStorageSync('userInfo', 'dsad')
    wx.removeStorageSync('userInfo')
    ```

- 页面调用栈

    ```js
    var pages = getCurrentPages()
    var prev_pages = pages[pages.length-2]
    ```

- 跳转回上一个页面

    ```js
    wx.navigateBack({})
    ```

- 小程序页面的生命周期

    - onLoad(一次)
    - onShow(只要展示这个页面 就会自动调用)
    - onReady(一次)
    - onHide(每次页面隐藏 就会自动调用)
    - onUnload(卸载 小程序关闭时自动调用)

- 全局app.js

    ```js
    App({
        /**
       * 当小程序初始化完成时，会触发 onLaunch（全局只触发一次）
       */
      onLaunch: function () {
        var userInfo = wx.getStorageSync('userInfo')
        if (userInfo){
        this.gloableData.userInfo = userInfo
        }
      },
      gloableData: {
        userInfo: null
      },
    })
    ```

- wx:if指令

# 页面传值

## 父页面向子页面传值

父页面：

```text
/pages/xx/xxx?id=1
```

子页面：

```text
onload:function(option){

}
```

## 子页面向父页面传值

子页面：

```text
var pages = getCurrentPages()
var prevPage = pages[pages.length - 2]
// prevPage.setData({
	topicText: topicInfo.title
})
```

注意：data-xx可以给事件传值

