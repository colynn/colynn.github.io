---
layout: post
title: 小程序实践归档
subtitle: 原生小程序&wepy
tags: [小程序]
comments: true
---

### 登录-推荐实践
[![wechat-auth-login]({{ site.url }}/img/mp_practice/wechat_auth_login.png)]({{ site.url }}/img/mp_practice/wechat_auth_login.png)

### 登录-实现原则
* 小程序与开发服务器端通过 3rd_session（第三方session）【即自定义登录状态】来实现通信，微信官方也不建议直接通过openid/session_key来通信；

*原因*： openid是微信用户的唯一标识，直接用于传参通信的话，会存在被第三方组织截获的风险，进而用于其他用途，
如本地模拟请求，使用截获的openid作为传参，从而就可以伪装成相应的人。

* 会话 session_key 是对用户数据进行 加密签名 的密钥。为了应用自身的数据安全，开发者服务器不应该把会话密钥下发到小程序，也不应该对外提供这个密钥。

* 数据校验方式：通过调用接口（如 wx.getUserInfo）获取数据时，接口会同时返回 rawData、signature，其中 signature = sha1( rawData + session_key )
开发者将 signature、rawData 发送到开发者服务器进行校验。服务器利用用户对应的 session_key 使用相同的算法计算出签名 signature2 ，比对 signature 与 signature2 即可校验数据的完整性。
小程序登录判断依赖wx.checksession， 但对于自身的登录态也是需要维护的，当前没有想到一个比较完整的解决方案。

### 问题记录
1. 子组件的方法调用时没有生效，注意将方法调整至methods内， 示例代码：
    ```
        ......
        methods = {
            async addFavorites() {
                wepy.showToast({
                        title: '接口待完善',
                        duration: 2000
                });
            }
        }
        ....
    ```
2. 绑定事件获取要操作的属性时均要通过```data-*```的形式来传值，示例代码：
    ```
    <template>
        <view data-parent="{{index}}" data-index="{{current}}" @tap.stop="setChoice">示例代码</view>
    </template>

    <script>

    setChoice(event) {
            var parent_id = event.currentTarget.dataset.parent; 
            var index = event.currentTarget.dataset.index;
            ...
    </script>
    ```

3. 小程序当前窗口高度获取（单位为rpx）

    * 获取高度

    ```
    try {
      const res = wx.getSystemInfoSync()
      console.log(res.model)
      console.log(res.pixelRatio)
      console.log(res.screenWidth)
      console.log(res.screenHeight)
      console.log(res.language)
      console.log(res.version)
      console.log(res.platform)

      var clientHeight=res.screenHeight,
          clientWidth=res.screenWidth,
          rpxR=750/clientWidth;
      var calc=clientHeight*rpxR;
      console.log('SystemScreenHeight: ' + calc)
      wepy.setStorageSync('SystemScreenHeight', calc)
    } catch (e) {
      console.log("获取系统信息失败，使用默认高度")
      wepy.setStorageSync('SystemScreenHeight', 1026)
      // Do something when catch error
    }
    ```

    * 使用高度

    ```
    var winHeight = wepy.getStorageSync('SystemScreenHeight');
    ```