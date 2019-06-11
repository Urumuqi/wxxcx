# 感谢 🙏iwanli

# Laravel 5 微信小程序扩展

> 小程序官方的加解密 SDK 已经非常清楚了，只不过改成 Laravel 风格而已，仅仅相当于搬砖工。至于重复造轮子，我发现其他人的扩展解密用户信息的时候代码出错了，并且需要安装一个 Laravel 的 Curl 扩展也没说。只好自己去根据他们的源码自己写一个0.0 ，不依赖其他扩展，直接安装使用即可。

## 小程序API接口

* 用户登录：[wx.login](https://mp.weixin.qq.com/debug/wxadoc/dev/api/api-login.html)
* 获取用户信息：[wx.getUserInfo](https://mp.weixin.qq.com/debug/wxadoc/dev/api/open.html#wxgetuserinfoobject)

## 安装

执行以下命令安装最新稳定版本:

**PHP > 7.1**

```bash
composer require urumuqi/wxxcx=2.1
```

或者添加如下信息到你的 `composer.json` 文件中 :

**PHP > 7.1**

```bash
"iwanli/wxxcx": "^2.0"
```

然后注册服务提供者到 Laravel中 具体位置：`/config/app.php` 中的 `providers` 数组:

```php
Urumuqi\Wxxcx\WxxcxServiceProvider::class,
```
发布配置文件:

```bash
php artisan vendor:publish --tag=wxxcx
```
命令完成后，会添加一个`wxxcx.php`配置文件到您的配置文件夹 如 : `/config/wxxcx.php`。

生成配置文件后，将小程序的 `AppID` 和 `AppSecret` 填写到 `/config/wxxcx.php` 文件中

## 在Laravel 5控制器中使用 (示例)

```php
...

use Urumuqi\Wxxcx\Wxxcx;

class WxxcxController extends Controller
{
    protected $wxxcx;

    function __construct(Wxxcx $wxxcx)
    {
        $this->wxxcx = $wxxcx;
    }

    /**
     * 小程序登录获取用户信息
     * @author 晚黎
     * @date   2017-05-27T14:37:08+0800
     * @return [type]                   [description]
     */
    public function getWxUserInfo()
    {
        //code 在小程序端使用 wx.login 获取
        $code = request('code', '');
        //encryptedData 和 iv 在小程序端使用 wx.getUserInfo 获取
        $encryptedData = request('encryptedData', '');
        $iv = request('iv', '');

        //根据 code 获取用户 session_key 等信息, 返回用户openid 和 session_key
        $userInfo = $this->wxxcx->getLoginInfo($code);

        // 增加session_key 参数
        return $this->wxxcx->getUserInfo($encryptedData, $iv, $userInfo['session_key']);
    }
}
```

用户信息返回格式:

```
{
    "openId": "xxxx",
    "nickName": "晚黎",
    "gender": 1,
    "language": "zh_CN",
    "city": "",
    "province": "Shanghai",
    "country": "CN",
    "avatarUrl": "http://wx.qlogo.cn/mmopen/xxxx",
    "watermark": {
        "timestamp": 1495867603,
        "appid": "your appid"
    }
}
```

## 小程序端获取 code、iv、encryptedData 向服务端发送请求示例代码：

```javascript
//调用登录接口
wx.login({
    success: function (response) {
        var code = response.code
        wx.getUserInfo({
            success: function (resp) {
                wx.request({
                    url: 'your domain',
                    data: {
                        code: code,
                        iv: resp.iv,
                        encryptedData: resp.encryptedData
                    },
                    success: function (res) {
                        console.log(res.data)
                    }
                })
            }
        })
    },
    fail:function(){
        ...
    }
})
```
