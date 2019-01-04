## 小程序工具类requestUtils

### 1.前言
开发小程序已经有一段时间了，都没有写过小程序相关的文章，踩过坑挺多，把这些坑记下来，下次就不会再犯了。
小程序自带的请求方法不是特别方便，无意中得到了一个工具类，因此把这个工具类分享出来

### 2.工具类详情

```js
function formatTime(date) {
  var year = date.getFullYear()
  var month = date.getMonth() + 1
  var day = date.getDate()

  var hour = date.getHours()
  var minute = date.getMinutes()
  var second = date.getSeconds()

  return [year, month, day].map(formatNumber).join('-') + ' ' + [hour, minute, second].map(formatNumber).join(':')
}

function formatNumber(n) {
  n = n.toString()
  return n[1] ? n : '0' + n
}

//添加请求根目录
var rootDocment = 'https://yjt.*****.com/wechat';
function req(url, data, cb) {
  wx.request({
    url: rootDocment + url,
    data: data,
    method: 'post',
    header: { 'Content-Type': 'application/x-www-form-urlencoded' },
    success: function (res) {
      return typeof cb == "function" && cb(res.data)
    },
    fail: function () {
      return typeof cb == "function" && cb(false)
    }
  })
}

function getReq(url, data, cb) {
  wx.request({
    url: rootDocment + url,
    data: data,
    method: 'get',
    header: { 'Content-Type': 'application/x-www-form-urlencoded' },
    success: function (res) {
      return typeof cb == "function" && cb(res.data)
    },
    fail: function () {
      return typeof cb == "function" && cb(false)
    }
  })
}

// 去前后空格  
function trim(str) {
  return str.replace(/(^\s*)|(\s*$)/g, "");
}

// 提示错误信息  
function isError(msg, that) {
  that.setData({
    showTopTips: true,
    errorMsg: msg
  })
}

// 清空错误信息  
function clearError(that) {
  that.setData({
    showTopTips: false,
    errorMsg: ""
  })
}

function formatTime(time) {
  var year = time.getFullYear();
  var month = time.getMonth() + 1;
  var date = time.getDate();
  var hour = time.getHours();
  var minute = time.getMinutes();
  var second = time.getSeconds();
  return year + "-" + month + "-" + date + " " + hour + ":" + minute + ":" + second;
}

module.exports = {
  formatTime: formatTime,
  req: req,
  trim: trim,
  isError: isError,
  clearError: clearError,
  getReq: getReq,
  formatTime: formatTime
}  
```

### 3.用法
在`page`页面顶部定义

```js
var wechatUtil = require('../../utils/wechatRequest.js');
```

### 3.发起请求

```js
wechatUtil.req("/member/update",{
      "unionId": unionId,
      "username": username,
      "headImg": headImg
    },function(res){
	    //res为返回的数据
      if(res.resultCode == 200){
        console.log("更新用户信息成功");
        console.log(res);
        // that.setData({
        //   member:res.resultContent
        // });
      }else{
        console.log("更新用户信息失败");
        console.log(res);
      }
    })
```

### 4.结尾
好了，一个简单的工具类结束了，拜拜