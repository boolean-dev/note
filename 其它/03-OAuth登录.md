## OAuth登录的四种方式

### 1. 授权码

**授权码（authorization code）方式，指的是第三方应用先申请一个授权码，然后再用该码获取令牌。**

这种方式是最常用的流程，安全性也最高，它适用于那些有后端的 Web 应用。授权码通过前端传送，令牌则是储存在后端，而且所有与资源服务器的通信都在后端完成。这样的前后端分离，可以避免令牌泄漏。

例如，有一个网站A（`高德地图`），需要向网站B（`微博`）授权登录

![](https://ws1.sinaimg.cn/large/c1ba9646ly1g43yaivod5j20qp0ecq56.jpg)

第一步，拿到授权码

A 网站（`高德`）提供一个链接，用户点击后就会跳转到 B（`微博`） 网站，授权用户数据给 A（`高德`） 网站使用。下面就是 A （`高德`）网站跳转 B （`微博`）网站的一个示意链接。

```sh
https://api.weibo.com/oauth2/authorize?
client_id=884965267
&redirect_uri=https%3A%2F%2Fid.amap.com%2Findex%2Fweibo%3Fpassport%3D1
```

`client_id`客户端id,`redirect_uri`跳转链接

第二步，在用户跳转后，B 网站会要求用户登录，然后询问是否同意给予 A（`高德`） 网站授权。用户表示同意，这时 B （`微博`）网站就会跳回`redirect_uri`参数指定的网址。

![微博授权](https://ws1.sinaimg.cn/large/c1ba9646ly1g43yi1o2ykj20l409hq38.jpg)

跳转时，会传回一个授权码，就像下面这样。

```sh
https%3A%2F%2Fid.amap.com%2Findex%2Fweibo%3Fpassport%3D1?
code=******
```

第三步，A （`高德`）网站拿到授权码以后，就可以在**后端**，向 B （`微博`）网站请求令牌。

```sh
https://api.weibo.com/oauth2/token?
 client_id=CLIENT_ID&
 client_secret=CLIENT_SECRET&
 grant_type=authorization_code&
 code=AUTHORIZATION_CODE&
 redirect_uri=CALLBACK_URL
```

上面的url中，`client_id`参数和`client_secret`参数用来让 B（`微博`） 确认 A（`高德`） 的身份（`client_secret`参数是保密的，因此只能在后端发请求），`grant_type`参数的值是`AUTHORIZATION_CODE`，表示采用的授权方式是授权码，`code`参数是上一步拿到的授权码，`redirect_uri`参数是令牌颁发后的回调网址。

第四步，B（`微博`） 网站收到请求以后，就会颁发令牌。具体做法是向`redirect_uri`指定的网址，发送一段 JSON 数据。

例如：

```json

    {    
      "access_token":"ACCESS_TOKEN",
      "token_type":"bearer",
      "expires_in":2592000,
      "refresh_token":"REFRESH_TOKEN",
      "scope":"read",
      "uid":100101,
      "info":{...}
    }

```

`access_token`就是我们所需要的令牌，这个信息在A（`高德`）网站的后端得到，即可进行用户的登录认证

因此，这种方式的登录模式图为：![授权码模式图](https://ws1.sinaimg.cn/large/c1ba9646ly1g43yrl4w67j20l90bz42f.jpg)

这种登录一般应用于前后端分离登录，安全性非常的高，使用也是最广泛的一种

### 2. 隐藏式

有些 Web 应用是纯前端应用，没有后端。这时就不能用上面的方式了，必须将令牌储存在前端。**RFC 6749 就规定了第二种方式，允许直接向前端颁发令牌。这种方式没有授权码这个中间步骤，所以称为（授权码）"隐藏式"（implicit）。**

第一步：A 网站提供一个链接，要求用户跳转到 B 网站，授权用户数据给 A 网站使用。

```sh
https://b.com/oauth/authorize?
  response_type=token&
  client_id=CLIENT_ID&
  redirect_uri=CALLBACK_URL&
  scope=read
```

上面 URL 中，`response_type`参数为`token`，表示要求直接返回令牌。

第二步：用户跳转到 B 网站，登录后同意给予 A 网站授权。这时，B 网站就会跳回`redirect_uri`参数指定的跳转网址，并且把令牌作为 URL 参数，传给 A 网站。

```sh
https://a.com/callback#token=ACCESS_TOKEN
```

上面 URL 中，`token`参数就是令牌，A 网站因此直接在前端拿到令牌。

![图](https://ws1.sinaimg.cn/large/c1ba9646ly1g43z2fcxhzj20ku0boadm.jpg)

这种方式的相对于上一种方式，更加简单，但是也更加不安全，所有的代码和登录都写在前端页面中，容易被他人破解，所以一般用于安全性不高的地方。

### 3. 密码式

**如果你高度信任某个应用，RFC 6749 也允许用户把用户名和密码，直接告诉该应用。该应用就使用你的密码，申请令牌，这种方式称为"密码式"（password）。**

第一步，A 网站要求用户提供 B 网站的用户名和密码。拿到以后，A 就直接向 B 请求令牌。(后端调用该接口)

```sh
https://oauth.b.com/token?
  grant_type=password&
  username=USERNAME&
  password=PASSWORD&
  client_id=CLIENT_ID
```

上面 URL 中，`grant_type`参数是授权方式，这里的`password`表示"密码式"，`username`和`password`是 B 的用户名和密码。

第二步，B 网站验证身份通过后，直接给出令牌。注意，这时不需要跳转，而是把令牌放在 JSON 数据里面，作为 HTTP 回应，A 因此拿到令牌。

这种方式风险很大，需要直接提供账号密码，所以只适用于其他授权方式都无法采用的情况，而且必须是用户高度信任的应用。

### 4. 凭证式

**最后一种方式是凭证式（client credentials），适用于没有前端的命令行应用，即在命令行下请求令牌。**

第一步，A 应用在命令行向 B 发出请求。

```sh
https://oauth.b.com/token?
  grant_type=client_credentials&
  client_id=CLIENT_ID&
  client_secret=CLIENT_SECRET
```

上面 URL 中，`grant_type`参数等于`client_credentials`表示采用凭证式，`client_id`和`client_secret`用来让 B 确认 A 的身份。

第二步，B 网站验证通过以后，直接返回令牌。

这种方式给出的令牌，是针对第三方应用的，而不是针对用户的，即有可能多个用户共享同一个令牌。

这种认证场景通常用于调用第三方接口，例如调用阿里云的短信接口等等

文章参考[OAuth 2.0 的四种方式](http://www.ruanyifeng.com/blog/2019/04/oauth-grant-types.html)