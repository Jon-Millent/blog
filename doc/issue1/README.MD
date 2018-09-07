<div align="center">
    <img src="https://raw.githubusercontent.com/Jon-Millent/blog/master/images/issue1/logo.png" width="120">
</div>

#### mpvue是一个使用 Vue.js 开发小程序的前端框架。框架基于 Vue.js 核心，框架整体和vue语法保持一致，整体开发体验良好。但是越往深处走，越会发现一些问题。

### 1. 同一个页面被多次打开所引发的问题
首先是有这么一个需求，从 `主页` -> `我的关注` -> `别人的主页`， `主页` 这个模块被打开了两次
![tim 20180906170054](https://raw.githubusercontent.com/Jon-Millent/blog/master/images/issue1/TIM%E6%88%AA%E5%9B%BE20180906170054.png)  

我拍拍胸脯对产品说：没问题！  

![5b7642e297f95b96f760e1ebd1a5fe8](https://raw.githubusercontent.com/Jon-Millent/blog/master/images/issue1/5b7642e297f95b96f760e1ebd1a5fe8.jpg)

这个需求是没毛病的，就是当我们从 `别人的主页` -> `我的关注` -> `主页` 的时候，我的信息变成了我刚刚打开的那个人的信息，这TM...  

![c1c9222bc38d90488d68e69edd57ed5](https://raw.githubusercontent.com/Jon-Millent/blog/master/images/issue1/c1c9222bc38d90488d68e69edd57ed5.jpg)

不要慌，有些事情需要我们 “摸索”   

![4413457d05dbb078781964138ddc0f3](https://raw.githubusercontent.com/Jon-Millent/blog/master/images/issue1/4413457d05dbb078781964138ddc0f3.jpg)


在 mpvue 路由 Page A -> Page A -> Back 的时候，其实数据公用了一个对象，不知道为什么mpvue会这么设计，在官方解决这个问题之前这里引用了 @maimake的解决办法

**首先创建一个hack文件** `/src/util/Hack.js`
```js
const pageDatas = {}

export default {
  install(_Vue) {
    // 添加全局方法或属性
    _Vue.prototype.$isPage = function isPage() {
      return this.$mp && this.$mp.mpType === 'page'
    }
    
    _Vue.prototype.$pageId = function pageId() {
      let pid = null
      try {
        pid = this.$isPage() ? this.$mp.page.__wxWebviewId__ : null
      } catch (e) { }
      return pid
    }
    
    // 注入组件
    _Vue.mixin({
      
      methods: {
        stashPageData() {
          return { data: { ...this.$data } }
        },
        restorePageData(oldData) {
          Object.assign(this.$data, oldData.data)
        },
      },
      
      onLoad() {
        if (this.$isPage()) {
          // 新进入页面
          Object.assign(this.$data, this.$options.data())
        }
      },
      
      onUnload() {
        if (this.$isPage()) {
          // 退出页面，删除数据
          delete pageDatas[this.$pageId()]
          this.$needReloadPageData = true
        }
      },
      
      onHide() {
        if (this.$isPage()) {
          // 将要隐藏时，备份数据
          pageDatas[this.$pageId()] = this.stashPageData()
        }
      },
      
      onShow() {
        if (this.$isPage()) {
          // 如果是后退回来的，拿出历史数据来设置data
          if (this.$needReloadPageData) {
            const oldData = pageDatas[this.$pageId()]
            if (oldData) {
              this.restorePageData(oldData)
            }
            this.$needReloadPageData = false
          }
        }
      },
      
    })
  },
}
```
**在main.js中引入** `/src/main.js`

```js
import Vue from 'vue'
import App from './App'
import Hack from './utils/Hack'


Vue.config.productionTip = false
App.mpType = 'app'
Vue.use(Hack)

const app = new Vue(App)
app.$mount()

```

测试后发现ok了，但是这只是一个临时解决方案，治标不治本，还是希望官方尽快解决。


### 2. `scroll-view` 组件的一些问题
其实吧，我并不推荐使用这个组件，也不是mpvue的问题，就是这个组件本身就不太好用。
* 事件触发不了的问题
在mpvue中使用下面的一些事件的时候是不会触发的
* `bindscrolltolower`
* `bindscrolltoupper`
* `bindscroll`  

解决方法你猜是什么？就是把`bind`去掉，哈哈哈  

* `scrolltolower`
* `scrolltoupper`
* `scroll`

然而，当 `scroll-view` 高度为 100% 的时候 `bindscrolltolower` 又不执行了，解决方法是给外面的父元素加一些限制
```html
<div class="cat-scroll">
    <scroll-view style="height: 100%; width: 100%">
    </<scroll-view>
</div>
```

```css
.cat-scroll {
  position: absolute;
  left: 0;
  right: 0;
  bottom: 0;
  top: 0;
  overflow: hidden;
}
```
这样就解决了，但但但但但但但但是，这样铺满屏幕后和小程序的下拉刷新冲突了，导致下拉刷新不执行。WTF

![c1c9222bc38d90488d68e69edd57ed5](https://raw.githubusercontent.com/Jon-Millent/blog/master/images/issue1/c1c9222bc38d90488d68e69edd57ed5.jpg)

回到我们开头说的不用 `scroll-view` 我们用别的方法解决。首先我们使用 `scroll-view`的原因一般就是做一个下拉加载更多么，在小程序中页面全局有一个滚动到底部的事件 `onReachBottom` 我们只需要在这个事件里面写上拉加载更多就好了，和下拉刷新也不冲突。

```js
export default {
    onPullDownRefresh(){
        //  下拉刷新
    },
    onReachBottom(){
        // 上拉加载更多
    }
}

```
### 3. slot 嵌套子模板的问题
在新版本的mpvue中支持了 `slot` 这个功能，但是我劝大家不要用。问题很多，父子之间传值有各种问题，我也没找到很好的解决办法，希望官方和社区继续完善。
