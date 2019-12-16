# OAuth2.0

[参考资料]: http://www.ruanyifeng.com/blog/2019/04/oauth-grant-types.html

## 简单解释

OAuth是一种授权机制。数据的所有者告诉系统，同意授权第三方应用进入系统获取这些数据。系统从而产生一个短期的进入令牌（token），用来代替密码，供第三方应用使用。

### 令牌&密码

- 令牌是短期的，到期会自动失效，用户自己无法修改。密码一般长期有效
- 令牌可以被数据所有者撤销，会立即失效
- 令牌有权限范围（scope），对于网络服务来说，只读令牌比读写令牌更安全。密码一般是完整权限。

## 四种授权方式

### 授权码(authorization code)

第三方应用先申请一个授权码，然后用该码获取令牌

最常用，安全性最高，适用于有后端的web应用。授权码通过前端传送，令牌储存在后端，所有与资源服务器的通信都在后端完成，可以避免令牌泄露。

### 隐藏式

 纯前端应用，必须将令牌储存在前端。没有授权码这个中间步骤

### 密码式

高度信任某个应用，允许把用户名和密码直接告诉该应用。该应用就使用密码申请令牌。

### 凭证式（client credentials）

适用于没有前端的命令行应用，在命令行下请求令牌。这种方式给的令牌是针对第三方应用的，不是用户，即有可能多个用户共享同一个令牌。

## 令牌使用

发的API请求都必须带令牌。在请求头加一个Authorization字段，令牌就放在这个字段里面。

## 更新令牌

颁发令牌时一次性颁发两个令牌，一个用于获取数据，另一个用于获取新的令牌。



## 代码样例

```javascript
const Koa = require('koa');
const path = require('path');
const serve = require('koa-static');
const route = require('koa-route');
const axios = require('axios');

const app = new Koa();

const main = serve(path.join(__dirname + '/public'));

const clientID = '';
const clientSecret = '';

const oauth = async ctx => {
  const requestToken = ctx.request.query.code;
  console.log('authorization code: ', requestToken);

  const tokenResponse = await axios({
    method: 'post',
    url: `https://github.com/login/oauth/access_token?client_id=${clientID}&client_secret=${clientSecret}&code=${requestToken}`,
    headers: {
      accept: 'application/json'
    }
  });

  const accessToken = tokenResponse.data.access_token;
  console.log(`access_token: ${accessToken}`);

  const result = await axios({
    method: 'get',
    url: 'https://api.github.com/user',
    headers: {
      accept: 'application/json',
      Authorization: `token ${accessToken}`,
    }
  });

  console.log(result.data);
  const {name} = result.data;

  ctx.response.redirect(`/welcome.html?name=${name}`);
}

app.use(main);
app.use(route.get('/oauth/redirect', oauth));
app.listen(8080);
```



