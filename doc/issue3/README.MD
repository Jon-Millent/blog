<div align="center">
    <img src="https://raw.githubusercontent.com/Jon-Millent/blog/master/images/main/3b1a8efc35a023b743bd442d93f001f1.jpg" width="260">
</div>  
<br />  

咳咳，直入主题。敲黑板，请看下面一段对话。

> 产品：我想要小程序发红包这个功能
> 程序员：目前不支持啊
> 产品：我不管！
> 程序员：做不到啊，这。。。
> 产品：我不管！我不管！你们程序员都是大猪蹄子！哼(￢︿̫̿￢☆)
> 程序员：我。。。TM，一樱桃MX BAORD机械键盘砸死你
  
在目前的小程序版本尚不支持发红包这个功能，开发者们表面笑嘻嘻，心里妈卖批！  

![c1c9222bc38d90488d68e69edd57ed5](https://github.com/Jon-Millent/blog/raw/master/images/issue3/0479275f5134e92e7bcc5c003ca36246.gif)

### 在这里，提供一种已经实践过的解决思路  

![c1c9222bc38d90488d68e69edd57ed5](https://github.com/Jon-Millent/blog/raw/master/images/issue3/20180909155313.png)

### 详细解析，就是下面的几个步骤  

* 首先是在小程序里面的领红包入口
* 点击后跳转webview， 附带当前的小程序的token给webview验证，webview 的src 指向具备付款能力的服务号的一个获取openid页面，根据附加的token来判断用户是否可以领取红包，此时用户拥有两个openid，一个是相对于小程序的openid，一个是相对于服务号的用来发红包的openid，如下面伪代码
  ```js
      // 伪代码
      weixinApi.angerPay.sendRedPacket({
        total_num: 1,
        mch_billno: '1000000',
        send_name: '测试标题',
        re_openid: 'xaxadsadas', // 获取到的openid
        total_amount: 100,
        wishing: '恭喜发财，大吉大利',
        act_name: '扫码领红包',
        remark: '备注信息',
        client_ip: '127.0.0.1',
        scene_id: 'PRODUCT_2'
        }, (err, result) => {
        console.log(err, result)
    });
  ```
* 返回领取结果

### 注意事项
* 跳转的webview的src要添加到小程序业务域名
* 链接必须是 https 协议的

### 未来的接口

在不久的将来，会开放一个发红包的接口，就是 `wx.sendBizRedPacket`，用法实例。

```js
    wx.sendBizRedPacket({
      timeStamp: e.timeStamp,
      nonceStr: e.nonceStr,
      package: e.package,
      signType: e.signType,
      paySign: e.paySign,
      success: function (res) {
        console.log('红包success')
        let url = config.HTTP_Prize_URL + '/v1/sign_tmp/sendSuccess.do';
        let data = {
          minipid: that.data.minipid,
          date: that.data.date
        }
        console.log('红包成功以后接口请求参数数据:' + JSON.stringify(data))
        util.request(url, 'post', data, '正在加载数据', function (res) {
          console.log('红包成功以后接口返回结果:' + JSON.stringify(res.data))
        })
        wx.reLaunch({
          url: '../my_prize/my_prize_2?reward=' + res.data.body.reward,
        })
      },
      fail: function (res) {
        console.log('红包fail')
      },
      complete: function (res) {
        console.log('红包complete')
        // wx.showModal({
        //   title: '红包complete',
        //   content: '红包complete',
        // })
      }
    })

```
### 欢迎关注公众号
<img src="https://raw.githubusercontent.com/Jon-Millent/blog/master/images/main/weixin.png">
