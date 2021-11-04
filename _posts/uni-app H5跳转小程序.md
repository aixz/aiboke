---
layout: post
title: Hilt笔记
date: 2021-11-04
Author: aixz
categories:
tags: [笔记, uni-app, 小程序]
comments: true

---

### uni-app 开发H5跳转小程序

注意要点：

1.H5调用云函数引入cloud.js文件，在工程main.html的header标签中引入

```
<script src="https://res.wx.qq.com/open/js/cloudbase/1.1.0/cloud.js"></script>
```

然后在manifest.json 文件 h5栏设置

```
"template" : "main.html",
```

2.引入jweixin 

```
npm install jweixin-module --save
```

在vue界面中使用时

```
var jweixin = require('jweixin-module');
```

3.wx-open-launch-weapp使用,这个标签在微信浏览器中打开才有效，通过静态网站云托管可以免签名跳转小程序

```
<wx-open-launch-weapp id="launch-btn" username="gh_______" path="/pages/common/nav">
	<script type="text/wxtag-template">
		<style>.btn{}</style>
		<view class="btn">打开小程序</view>
	</script>
</wx-open-launch-weapp>
```

注册

```
if (this.isWeixin()) {
	jweixin.config({
		debug: false, // 调试时可开启
		appId: 'wx__________', // <!-- replace -->
		timestamp: 0, // 必填，填任意数字即可
		nonceStr: 'nonceStr', // 必填，填任意非空字符串即可
		signature: 'signature', // 必填，填任意非空字符串即可
		jsApiList: ['chooseImage', 'previewImage'], // 必填，随意一个接口即可 
		openTagList:['wx-open-launch-weapp'], // 填入打开小程序的开放标签名
	})
}
```

4.在外部浏览器网页打开小程序，通过云函数跳转小程序

创建云函数，并上传

```
// 云函数入口文件
const cloud = require('wx-server-sdk')

cloud.init()

// 云函数入口函数
exports.main = async (event, context) => {
  const wxContext = cloud.getWXContext()

  switch (event.action) {
    case 'getUrlScheme': {
      return getUrlScheme()
    }
  }

  return 'action not found'
}

async function getUrlScheme() {
  return cloud.openapi.urlscheme.generate({
    jumpWxa: {
      path: '', // 期望跳转的page
      query: '',
    },
    // 如果想不过期则置为 false，并可以存到数据库
    isExpire: false,
    // 一分钟有效期
    expireTime: parseInt(Date.now() / 1000 + 60),
  })
}
```

在vue onLoad中注册云函数,通过云函数获取跳转连接，直接跳转

```
if (this.isMobileWeb()) {
	var c = new cloud.Cloud({
		// 必填，表示是未登录模式
		identityless: true,
		// 资源方 AppID
		resourceAppid: 'wx——————————', // <!-- replace -->
		// 资源方环境 ID
		resourceEnv: '云ID', // <!-- replace -->
	})
	await c.init()
	window.c = c
	await this.openapplet()
} 

openapplet:async function(){
	const res = await window.c.callFunction({
		name: 'public',
		data: {
			action: 'getUrlScheme',
		},
	})
	location.href = res.result.openlink
},
```

折腾的比较久的是jweixin-module的使用以及 cloud.js的导入问题，记录一下
