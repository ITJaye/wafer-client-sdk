# 微信小程序客户端腾讯云增强 SDK 支持微信小程序登陆及授权获取用户信息的新要求


支持微信小程序登陆及授权获取用户信息的新要求

直接使用这个函数 不需要去在意登陆状态 会话过期会自动登陆

```
qcloud.request({
      login: true,
      url: '', 
      method: 'GET',
      data: {},
      success: function(res) {
        
      },
      fail: function(error){
      },
      complete: function(){
        wx.stopPullDownRefresh();
      }
    });
  },
  ```


  回去用户头像昵称 授权情况
  ```
bindGetUserInfo: function () {
    var that = this
    wx.getUserInfo({
      withCredentials: true,
      success: function (res) {
        var userInfo = res.userInfo
        var encryptedData = res.encryptedData
        var iv = res.iv
        that.getUserInfo(encryptedData, iv);
      },fail: function(){
        
      }
    })
  },

  getUserInfo: function (encryptedData, iv) {
    var that = this
    // 构造请求头，包含 code、encryptedData 和 iv
    var header = {};
    var session = qcloud.getSession();
    if(!session){
      util.showAlert('请刷新重试');
      return;
    }
    header[constants.WX_HEADER_ID] = session.id;
    header[constants.WX_HEADER_SKEY] = session.skey;
    header[constants.WX_HEADER_ENCRYPTED_DATA] = encryptedData;
    header[constants.WX_HEADER_IV] = iv;
    qcloud.request({
      login: true,
      url: '',
      header: header,
      method: 'get',
      data: {},
      success: function (result) {
        var data = result.data;
        wx.setStorageSync('userInfo', data.data.userInfo);
        that.setData({ userInfo: data.data.userInfo})
      },
      fail: function () {
      },
    });
  },

  ```

  按钮
  ```
<button open-type="getUserInfo" bindgetuserinfo="bindGetUserInfo">获取微信头像及昵称</button>
  ```


## 具体结合其他两个项目来一起实现

微信小程序新的登陆方式修改为：
用户每次进入小程序，通过wx. checkSession检查会话状态，调用wx.login获取code发送到服务端，服务端通过code换取用户的openId、unionId和session_key，此时注册用户信息，给定默认的头像和昵称，达到用户注册的目的，并给客户端返回用户的userInfo
1.小程序端需要用户头像和昵称的时候，小程序端布置【获取用户头像昵称按钮】，用户点击时通过wx.getUserInfo带上withCredentials:true 获取 encryptedData 和 iv 上传到服务端进行信息解密，解密后得到用户的userInfo，此时可以更新数据库中用户的信息。


原项目参考 [微信小程序客户端腾讯云增强 SDK](https://github.com/tencentyun/wafer-client-sdk)
新登陆授权要求 [小程序授权登陆新要求](https://developers.weixin.qq.com/blogdetail?action=get_post_info&lang=zh_CN&token=&docid=c45683ebfa39ce8fe71def0631fad26b)
原始项目 [Wafer](https://github.com/tencentyun/wafer)
