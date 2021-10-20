## 扫码
```
https://p7api.qiecdn.com/api/v2/account/qr_code/generate
返回：
url: "http://p7api.qiecdn.com/api/v2/account/qr_code/scan?qr_code=f086fa5a4f2c124bdf5b3b13889d17ec"

手机扫码后：
http://p7api.qiecdn.com/api/v2/account/qr_code/scan?qr_code=f086fa5a4f2c124bdf5b3b13889d17ec


微信扫码后：
http://live.qq.com/api/v2/account/qr_code/new_api/wx_authorize?qr_code=ee6e6ddde2a95a9edda2704dbaba6983&code=031H971w3mNFjX2FvK0w3vMmvL2H971g&state=STATE
headers中的
location: https://m.live.qq.com/login/quick?qr_code=ee6e6ddde2a95a9edda2704dbaba6983

打开登录页面
https://m.live.qq.com/login/quick?qr_code=f086fa5a4f2c124bdf5b3b13889d17ec8b70098c
然后就到了移动端登录页面

点击确认登陆调用
https://p7api.qiecdn.com//api/v2/account/qr_code/confirm
```