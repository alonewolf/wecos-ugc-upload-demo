# WeCOS-UGC-DEMO——微信小程序用户资源上传COS示例

WeCOS-UGC-DEMO展示了微信小程序中有用户上传资源的场景时，如何结合[COS（腾讯云对象存储服务）](https://www.qcloud.com/product/cos)API将资源直接上传至COS

## 准备工作

* 进入[腾讯云官网](https://www.qcloud.com)，注册帐号
* 登录[云对象存储服务（COS）控制台](https://console.qcloud.com/cos4)，开通COS服务，创建资源需要上传的Bucket
* 当前 demo 是基于 V4 接口，请确认你使用的 COS Bucket 是V4，如何确定自己应该是用v4的sdk还是v3的？ 登陆https://console.qcloud.com/cos 如果左上角提示是云对象存储v4则说明要用v4的sdk否则就是v3的。
* **对 Bucket 设置好跨域配置**
* COS鉴权服务器部署及URL地址（用于调用COSAPI时的鉴权），此处直接用我们鉴权服务端示例[COS-AUTH](https://github.com/tencentyun/cos-auth)，后文也会提到
<br/>
* 在小程序官网上配置域名信息（否则无法在小程序中发起对该域名的请求）
![image](https://cloud.githubusercontent.com/assets/8219248/21755628/cb82ee76-d651-11e6-9c50-861307aaf7ba.png)


## 源码简介

```tree
README.md
app
├── app.js
├── app.json
├── app.wxss
├── pages
│   ├── index
│   │   ├── index.js
│   │   ├── index.json
│   │   ├── index.wxss
│   │   └── index.wxml
└── utils
    ├── upload.js
    └── util.js
```

app目录是小程序目录，如果你没有创建小程序项目，我们可以直接下载本项目的demo，或者通过小程序开发工具创建，具体如何注册和创建请查看[小程序入门指引](https://mp.weixin.qq.com/debug/wxadoc/introduction/index.html?t=1483674932)

    `app.js` 是小程序入口文件
    `app.json` 是小程序的微信配置，其中指定了本示例的用户资源上传页面`index`
    `pages目录` 内包含各个页面的入口和配置，业务逻辑，如index目录则为`index`页面

其中重要的文件如下：

`utils/upload.js` 上传cos的核心代码

`pages/index/index.js` 实现用户资源上传的示例

## 核心代码

```js
//upload.js

/**
 * 把以下字段替换成自己的cos相关信息，详情可看API文档 https://www.qcloud.com/document/product/436/6066
 * REGION: cos上传的地区
 * APPID: 账号的appid
 * BUCKET_NAME: cos bucket的名字
 * DIR_NAME: 上传的文件目录
 * cosSignatureUrl：填写自己的鉴权服务器地址，查看前面的[准备工作]
 */
var cosUrl = "https://" + REGION + ".file.myqcloud.com/files/v2/" + APPID + "/" + BUCKET_NAME + DIR_NAME

/**
 * 上传方法
 * filePath: 上传的文件路径
 * fileName： 上传到cos后的文件名
 */
function upload(filePath, fileName) {
    // 鉴权获取签名
    wx.request({
        url: cosSignatureUrl,
        success: function(cosRes) {
            // 头部带上签名，上传文件至COS
            wx.uploadFile({
                url: cosUrl + '/' + fileName,
                filePath: filePath,
                header: { 'Authorization': cosRes.data },
                name: 'filecontent',
                formData: { op: 'upload' },
                success: function(uploadRes){ //do something }
            })
        }
    })
}
```

## 示例

如小程序项目目录为`app`

1、在`app/pages/index/index.js`中粘贴本示例中的代码
```js
//index.js

// upload的核心代码
var uploadFn = require('../../utils/upload.js')

//获取应用实例
var app = getApp()
Page({
    //上传按钮事件处理函数
    uploadToCos: function() {
    
        // 选择上传的图片
        wx.chooseImage({
            success: function(res) {

                // 获取文件路径
                var filePath = res.tempFilePaths[0];

                // 获取文件名
                var fileName = filePath.match(/(wxfile:\/\/)(.+)/)
                fileName = fileName[2]

                // 文件上传cos，参考上面的核心代码
                uploadFn(filePath, fileName)
            }
        })
    }
})
```

2、参考本示例，在`app/pages/index/index.wxml`中把js中对应的事件绑定到dom
```html
<!--index.wxml-->
<view class="container">
  <!-- ... -->
    <button type="primary" bindtap="uploadToCos" class="user-button"> 一键上传 </button>
  <!-- ... -->
</view>
```

## 配置相关

| 参数 | 格式 | 说明 |
|:--|:--|:--|
|cosSignatureUrl|**[String]**|鉴权服务器的域名|
|REGION|**[String]**|资源上传到的地区|
|APPID|**[String]**|账号的appid|
|BUCKET_NAME|**[String]**|资源上传到的bucket|
|DIR_NAME|**[String]**|资源上传到的目录|

APPID可以在[COS控制台](https://console.qcloud.com/cos4/secret)拿到


## COS鉴权相关

鉴权有两种方式：

1.前端鉴权，前端生成算法会暴露私钥。（**不推荐**）

2.服务端鉴权，安全性高，本示例采用该种方式。（**推荐**）

* 调用COSAPI需要鉴权，用于获取签名，如果需要了解具体的鉴权算法，可查看[此处](https://www.qcloud.com/document/product/436/6054)  
* 鉴权生成签名的算法需要用到SecretId、SecretKey，可在[COS控制台](https://console.qcloud.com/cos4/secret)查看
* 鉴权服务器需要你自己部署且提供URL地址，基于这种鉴权需求，我们提供了鉴权服务端示例[COS-AUTH](https://github.com/tencentyun/cos-auth)
* 签名有分单次有效签名和多次有效签名，单次有效签名必须传文件路径，多次有效签名**一定不要传文件路径**，可查看[此处](https://www.qcloud.com/document/product/436/6054)

## 相关

* [WeCOS](https://github.com/tencentyun/wecos-ugc-upload-demo)——小程序COS瘦身方案，解决官方1MB限制的烦恼

* [COS-AUTH](https://github.com/tencentyun/cos-auth)——COS鉴权服务器DEMO

