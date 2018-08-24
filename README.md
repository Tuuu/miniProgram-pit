# miniProgram-pit
小程序踩坑总结

# 自定义导航栏适配问题

page.json 配置：

```json
{
  "navigationStyle": "custom"
}
```

当自定义导航栏时，由于不同机型的状态栏高度不定,比如 iPhone X 之类的刘海屏手机、Android、iPhone 6/7/8 等，需要针对不同的机型对顶部导航栏的高度进行动态设置

获取系统状态栏高度

```js
const statusBarHeight = wx.getSystemInfoSync().statusBarHeight

statusBarHeight  // 单位 px
// iPhone X 44
// iPhone 6/7/8 20
```

计算导航栏高度

```js
// 标题栏高度
const titleBarHeight = 40  // 单位 px
// 导航栏高度
const navBarHeight = statusBarHeight + titleBarHeight
```

注：
1. 在自定义导航栏的情况下，所有固定到顶部(`position: fixed`)的元素，都需要进行适配操作；

2. 在自定义导航栏的情况下，`"enablePullDownRefresh": true` 时，小程序自带的下拉动画会从标题栏顶部开始出现，也就是用户需要下拉超过 `navBarHeight` 的高度才能看到对应的 Loading 动画，可以尝试自定义动画，用户下拉时能立刻看到对应的动画效果；

3. 自定义导航栏的情况下，通过 `wx.navigateTo()` 跳转的页面左上角没有返回图标，需要自定义设置。


# 调试

## 微信开发者工具

- 在微信开发者工具中，app.js 的 `onShow` 函数中存在 `wx.navigateTo` 跳转页面时，无法跳转成功，但是 `wx.navigateTo` 的 `success` 回调会返回成功值，真机上则可以进行跳转。

```js
App({
  onLaunch: function(options) {
    // ...
  },
  onShow: function (options){
    wx.navigateTo({
      url: `/pages/index/index?userId=${userId}`,
      success: res => {
        // 跳转成功
      }
    })
  },
})
```

- 当小程序的 AppId 更换时，必须在开发者工具中重新创建一个项目，然后填入最新的 AppId，否则在调用 `wx.login()` 登录的时候后台会收到 `code` 失效的报错，在开发者工具中的 `project.config.json` 文件中修改 AppId 也是无效的


# Canvas

- Canvas 可以通过 `wx.createCanvasContext("canvasId")` 创建 canvas 绘图上下文后进行 Canvas 的高度设置，这样可以动态绘制不同高度的内容
- `canvasContext.drawImage()` 中的 `imageResource` 参数不支持网络图片，需要先调用 `wx.getImageInfo()` 或者 `wx.downloadFile()` 获取到临时图片路径(是以 `wx://` 开头)

```js
let getImageInfo = function (imageSrc) {
  return new Promise((resolve, reject) => {
    // 网络图片，需先配置download域名才能生效
    wx.getImageInfo({
      src: imageSrc,
      success: res => resolve(res),
      fail: res => reject(res)
    })
  })
}

// 绘制
getImageInfo(imageSrc).then(res => {
  ctx.drawImage(res.path, x, y, res.width, res.height)
}).catch(res => console.log(res))
```

- 自动保存 Canvas 

可以在所有绘制结束之后直接调用 `wx.canvasToTempFilePath()` 获取到生成后的临时图片路径，但是如果有网络图片的话，会出现图片没有及时绘制上去就生成了临时图片路径，此时需要将 `wx.canvasToTempFilePath()` 封装一下，延时执行即可

```js
let getCanvasTempFilePath = function (canvasId) {
  return new Promise((resolve, reject) => {
    // 延时执行即可
    setTimeout(() => {
      wx.canvasToTempFilePath({
        canvasId: canvasId,
        success: res => resolve(res),
        fail: res => reject(res)
      })
    }, 1500)
  })
}

// 使用
getCanvasTempFilePath("canvasId").then(res => {
  wx.saveImageToPhotosAlbum({
    filePath: res.tempFilePath,
    success:  res => {
      // 成功保存
    },
    fail: res => {
      // 失败保存
    }
  })
}).catch(res => console.log(res))
```

# API

## 媒体

#### wx.getImageInfo(OBJECT)

[官方网站](https://developers.weixin.qq.com/miniprogram/dev/api/media-picture.html#wxgetimageinfoobject)

**问题：**

安卓手机(小米 8，华为 P10)，基础库版本 2.2.3，`src` 为相对文件路径时(`../../assets/background.png`)的情况下，此 API 会进入 `fail` 回调，返回信息为 `errMsg: "getImageInfo:fail file not found"`

iPhone 无此问题

**解决方案：** 直接使用相对路径


# WXML

## data

在 WXML 中 以 `data-` 开头的自定义的数据会在事件的回调中会通过 `e.currentTarget.dataset` 获取到；

书写方式： 以 `data-` 开头，多个单词由连字符 `-` 链接，不能有大写(大写会自动转成小写)如 `data-element-type`，最终在 `event.currentTarget.dataset` 中会将连字符转成驼峰`elementType`

示例：

```html
<view data-alpha-beta="1" data-alphaBeta="2" bindtap="bindViewTap"> DataSet Test </view>
```

```js
Page({
  bindViewTap:function(event){
    event.currentTarget.dataset.alphaBeta === 1 // - 会转为驼峰写法
    event.currentTarget.dataset.alphabeta === 2 // 大写会转为小写
  }
})
```