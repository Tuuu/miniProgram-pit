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

在微信开发者工具中，app.js 的 `onShow` 函数中存在 `wx.navigateTo` 跳转页面时，无法跳转成功，但是 `wx.navigateTo` 的 `success` 回调会返回成功值，真机上则可以进行跳转。

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