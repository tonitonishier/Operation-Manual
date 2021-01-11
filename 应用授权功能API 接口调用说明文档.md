# 应用授权功能API 接口调用说明文档
- [应用授权功能API 接口调用说明文档](#应用授权功能api-接口调用说明文档)
  - [背景](#背景)
  - [接入前提](#接入前提)
  - [授权流程](#授权流程)
    - [authorize获取code](#authorize获取code)
    - [获取token](#获取token)
    - [更新token](#更新token)
  - [logout登出接口](#logout登出接口)
  - [RS如何信任AS](#rs如何信任as)
    - [AS颁发的access_token具备如下特点](#as颁发的access_token具备如下特点)
    - [RS对access_token的校验](#rs对access_token的校验)
## 背景

YuFu作为一个Authorization Server(以下简称AS)， 可以授权用户访问指定的Resource Server(以下简称RS，RS对应YuFu中的应用系统)。授权成功之后， YuFu给RS颁发access_token， RS通过解析access_token的合法性以及内容，来判断zhangsan是否有权限访问Resource Server， 具体参见下文"RS如何信任AS"。

## 接入前提

需要在YuFu的管理控制台上添加如下配置:

- 添加应用：在YuFu创建OIDC协议的应用实例并配置该应用的redirect_uri，应用创建完成后，需保留该应用的client_id/client_secret以便后续接口调用；添加应用路径：管理员页面—导航栏应用—应用管理—添加应用—创建自定义应用—选择OpenID Connect—选择web—按照步骤完成应用的创建。

![](https://github.com/tonitonishier/Operation-Manual/blob/main/%E5%BA%94%E7%94%A8%E6%8E%88%E6%9D%83%E5%8A%9F%E8%83%BDAPI%E6%8E%A5%E5%8F%A3%E8%B0%83%E7%94%A8%E8%AF%B4%E6%98%8E%E6%96%87%E6%A1%A3%E7%94%A8%E5%9B%BE/SQAPI2.png?raw=true)

- 添加Resource Server：在YuFu的API管理功能中创建API并配置该API（Resource Server）对应的audience；添加Resource Server路径：管理员页面—导航栏应用—API管理—点击创建。

![](https://github.com/tonitonishier/Operation-Manual/blob/main/%E5%BA%94%E7%94%A8%E6%8E%88%E6%9D%83%E5%8A%9F%E8%83%BDAPI%E6%8E%A5%E5%8F%A3%E8%B0%83%E7%94%A8%E8%AF%B4%E6%98%8E%E6%96%87%E6%A1%A3%E7%94%A8%E5%9B%BE/SQAPI3.png?raw=true)

- 添加Resource Server对应的权限，且把权限分配给人；添加权限组路径：管理员页面—导航栏对象管理—权限组管理—添加权限组—在创建完成的权限组中进行关联人员和关联权限操作。

![](https://github.com/tonitonishier/Operation-Manual/blob/main/%E5%BA%94%E7%94%A8%E6%8E%88%E6%9D%83%E5%8A%9F%E8%83%BDAPI%E6%8E%A5%E5%8F%A3%E8%B0%83%E7%94%A8%E8%AF%B4%E6%98%8E%E6%96%87%E6%A1%A3%E7%94%A8%E5%9B%BE/SQAPI4.png?raw=true)

![SQAPI5](https://github.com/tonitonishier/Operation-Manual/blob/main/%E5%BA%94%E7%94%A8%E6%8E%88%E6%9D%83%E5%8A%9F%E8%83%BDAPI%E6%8E%A5%E5%8F%A3%E8%B0%83%E7%94%A8%E8%AF%B4%E6%98%8E%E6%96%87%E6%A1%A3%E7%94%A8%E5%9B%BE/SQAPI5.png?raw=true)

![SQAPI6](https://github.com/tonitonishier/Operation-Manual/blob/main/%E5%BA%94%E7%94%A8%E6%8E%88%E6%9D%83%E5%8A%9F%E8%83%BDAPI%E6%8E%A5%E5%8F%A3%E8%B0%83%E7%94%A8%E8%AF%B4%E6%98%8E%E6%96%87%E6%A1%A3%E7%94%A8%E5%9B%BE/SQAPI6.png?raw=true)

- 从YuFu侧获取授权接口的访问地址，并选择验证token签名公钥；YuFu提供了两种验证签名的公钥，代理API的OIDC应用的公钥（更方便）和API独有的公钥（更安全）。
  - 代理API的OIDC应用的公钥获取路径：管理员页面—导航栏应用—应用管理—选择代理API的OIDC应用—登录配置—Well-known 接口—点击进入接口详情页面复制“jwks_uri”；
  - API独有的公钥获取路径：管理员页面—导航栏应用—API管理—选择已创建的API—进入详情页面点选token设置—验证签名公钥Public Key设置项—选择使用当前API独有的公钥并复制；

![](https://github.com/tonitonishier/Operation-Manual/blob/main/%E5%BA%94%E7%94%A8%E6%8E%88%E6%9D%83%E5%8A%9F%E8%83%BDAPI%E6%8E%A5%E5%8F%A3%E8%B0%83%E7%94%A8%E8%AF%B4%E6%98%8E%E6%96%87%E6%A1%A3%E7%94%A8%E5%9B%BE/SQAPI9.png?raw=true)

![](https://github.com/tonitonishier/Operation-Manual/blob/main/%E5%BA%94%E7%94%A8%E6%8E%88%E6%9D%83%E5%8A%9F%E8%83%BDAPI%E6%8E%A5%E5%8F%A3%E8%B0%83%E7%94%A8%E8%AF%B4%E6%98%8E%E6%96%87%E6%A1%A3%E7%94%A8%E5%9B%BE/SQAPI10.png?raw=true)

![](https://github.com/tonitonishier/Operation-Manual/blob/main/%E5%BA%94%E7%94%A8%E6%8E%88%E6%9D%83%E5%8A%9F%E8%83%BDAPI%E6%8E%A5%E5%8F%A3%E8%B0%83%E7%94%A8%E8%AF%B4%E6%98%8E%E6%96%87%E6%A1%A3%E7%94%A8%E5%9B%BE/SQAPI7.png?raw=true)

![](https://github.com/tonitonishier/Operation-Manual/blob/main/%E5%BA%94%E7%94%A8%E6%8E%88%E6%9D%83%E5%8A%9F%E8%83%BDAPI%E6%8E%A5%E5%8F%A3%E8%B0%83%E7%94%A8%E8%AF%B4%E6%98%8E%E6%96%87%E6%A1%A3%E7%94%A8%E5%9B%BE/SQAPI8.png?raw=true)

应用侧需要实现的接口:

- redirect_uri: 在"授权"过程中，YuFu会把授权码(code)返回至指定的redirect_uri. 所以应用侧需要实现redirect_uri对应的接口，来获取code， 然后进行后续/token调用，详情请见下文。

## 授权流程

YuFu作为AS，授权流程步骤如下：

- 应用侧获取AS颁发的授权码code: 用户通过浏览器访问应用时，应用会向AS发起授权请求，AS向应用返回授权码code；
- 应用侧使用code换取AS颁发的access_token/id_token/refresh_token；
- 应用侧使用refresh_token更新access_token/id_token，即为应用侧更新授权；

授权流程的前置条件是用户已经在AS侧处于登录状态，当YuFu收到一个/authorize授权请求时，会检查当前用户是否在YuFu测处于登陆状态，若未登陆则重定向至YuFu做登陆，若已登陆，则返回授权码。

### authorize获取code

发起授权请求

Endpoint: GET https://<authorization_server_base>/authorize

Params:

| **参数名**                                    | **必填** | **说明**                                                     |
| --------------------------------------------- | -------- | ------------------------------------------------------------ |
| client_id                                     | 是       | 发起授权请求的应用client_id                                  |
| response_type                                 | 是       | 为code， 即授权码模式                                        |
| redirect_uri                                  | 是       | 回调链接，AS会把生成的授权码以重定向的方式返回至redirect_uri (参见接入前提) |
| state                                         | 否       | CSRF安全参数， 建议使用: 当AS重定向至redirect_uri时，会原封不动的把state参数回传 |
| nonce                                         | 否       | 防重放安全参数，建议使用: AS颁发的id_token里会通过nonce这个claim来回传， 以便于id_token合法性校验 |
| scope                                         | 是       | 授权的范围: 多值属性，以空格分割. 通常情况下传入openid profile  email， 调用者可以把Resource Server的Permission作为scope参数传递  当包括offline_access时表示请求AS返回refresh_token |
| audience                                      | 否       | 授权请求访问Resource Server的audience属性.   当不传时，颁发的access_token只能用来访问YuFu的/userinfo接口， 而不能访问Resource Server |
| 以下参数仅针对PKCE场景(SPA)生效(协议标准参数) |          |                                                              |
| code_challenge                                | 是       | 随机生成，需要和下面Token接口的code_verifier配对，生成算法为code_challenge_method约定的算法， 参见 |
| code_challenge_method                         | 否       | 可为S256. 出于安全考虑，不支持PLAIN                          |

Response:

通过302 Redirect， 以HTTP GET的方式， 返回state以及生成的授权码code至redirect_uri. 后续可以通过code来调用Token接口返回access_token/id_token/refresh_token

Example:

```
GET https://<authorization_server_base>/authorize?    
response_type=code
&scope=openid%20profile%20email%20read:meeting
&client_id=s6BhdRkqt3
&state=af0ifjsldkj
&nonce=af867asfdbcda1safd
&audience=https%3A%2F%2Fmeeting-api
&redirect_uri=https%3A%2F%2Fclient.example.org%2Fcb
```

```
HTTP/1.1 302 Found  
Location: https://client.example.org/cb?    code=SplxlOBeZQQYbYS6WxSbIA
&state=af0ifjsldkj
```

### 获取token

应用侧通过授权码code请求access_token/id_token/refresh_token，。

Endpoint: POST https://<authorization_server_base>/token

Content-Type: application/json或者application/x-www-form-urlencoded

Params:

| **参数名**                                    | 必填 | **说明**                                                     |
| --------------------------------------------- | ---- | ------------------------------------------------------------ |
| client_id                                     | 是   | 发起授权请求的应用client_id(参见接入前提)                    |
| client_secret                                 | 说明 | 当用授权码模式(WEB)时， 必填，可以通过basic auth或者form-post或者json的方式传入   当用PKCE模式(SPA)时， 不需要填 |
| grant_type                                    | 是   | 固定为authorization_code                                     |
| code                                          | 是   | 上述authorize接口返回的授权码                                |
| redirect_uri                                  | 是   | 需要和发起authorize请求时保持一致                            |
| 以下参数仅针对PKCE场景(SPA)生效(协议标准参数) |      |                                                              |
| code_verifier                                 | 是   | 通过S256(code_verifier)换算成authorize请求时的code_challenge |

Response: 以Content-Type: application/json的形式返回token. 

```
 HTTP/1.1 200 OK  
 Content-Type: application/json 
 {
     "access_token": "jwt_access_token_tbd", //详细内容见下文
     "token_type": "Bearer", 
      "refresh_token": "8xLOxBtZp8",
      "expires_in": 3600,    // access_token的过期时间
      "id_token": "jwt_id_token" // optional 
      "refresh_token_expires_in": 36000 // YuFu私有属性，表示refresh_token的过期时间 
}
```

### 更新token

更新token步骤避免了token过期后重复获取授权码code的过程，支持应用侧通过授权码code返回的refresh_token来更新access_token/id_token。

Endpoint: POST https://<authorization_server_base>/token

Content-Type: application/json或者application/x-www-form-urlencoded

Params:

| **参数名**    | **必填** | **说明**                                                     |
| ------------- | -------- | ------------------------------------------------------------ |
| client_id     | 是       | 发起授权请求的应用client_id(参见接入前提)                    |
| client_secret | 见说明   | 当用授权码模式(WEB)时， 必填，可以通过basic auth或者form-post或者json的方式传递   当用PKCE模式(SPA)时， 不需要填 |
| grant_type    | 是       | 固定为refresh_token                                          |
| refresh_token | 是       | 授权码模式返回的refresh_token                                |
| audience      | 否       | 授权请求访问的Resource Server的audience属性.   当不传时，颁发的access_token只能用来访问YuFu的/userinfo接口，   而不能访问Resource Server |
| scope         | 否       | 同/authorize接口的scope参数                                  |

Response: 以Content-Type: application/json的形式返回token. 

```
 HTTP/1.1 200 OK  Content-Type: application/json
 {
     "access_token": "jwt_access_token_tbd", // 详细内容见下文
     "token_type": "Bearer",
     "expires_in": 3600,    // access_token的过期时间
     "id_token": "jwt_id_token" // optional
 }
```

若YuFu侧检查到refresh_token过期后，该接口会返回400， 且返回json格式的error. 用户必须使用授权码模式重新发起授权调用

比如: 

```
  {   
      "error": "invalid_grant",
      "error_description": "invalid refresh_token"
  }

```

## logout登出接口

YuFu的登出接口需要调用者在浏览器中打开， 第三方调用者可以通过HTTP REDIRECT的方式引导浏览器执行YuFu侧的登出。

Endpoint: GET https://<yufu_idp_base>/logout

Params:

| 参数名    | 必填 | 说明                                                         |
| --------- | ---- | ------------------------------------------------------------ |
| return_to | 否   | 值为合法的URI.   当YuFu侧登出之后，会通过HTTP REDIRECT的方式跳转至return_to指定的URI.   若不携带该参数或者该参数是个非法的URI， YuFu将会默认跳转至YuFu的登陆界面. |

YuFu目前仅支持SAML认证源的SLO

一个合法的登出接口，比如: https://xifeng-idp.i.yufuid.com/logout?return_to=https://www.baidu.com

YuFu登出之后将会跳转至https://www.baidu.com

## RS如何信任AS

AS作为access_token的颁发者， RS作为access_token的消费者，两者之间如何建立互信，即RS为什么要信任AS颁发的access_token 。

### AS颁发的access_token具备如下特点

- access_token是个经过YuFu签名的JWT格式的token. AS和RS之间通过共享签名密钥(对称或者非对称)的方式来建立信任关系
- access_token里包含iss字段: 即AS的唯一标识，表明这个access_token是由RS信任的AS颁发的
- access_token里包含zhangsan对应的租户id: tnt_id
- access_token里会包含zhangsan的唯一标识: sub
- access_token里会包含zhangsan的显示名: name
- access_token里会包含zhangsan的登陆名: preferred_username
- access_token里包含zhangsan在这个RS上的权限: perms
- access_token里包含这个token是颁发给RS的: aud字段里包括RS的audience
- access_token里包含这个token是由哪个应用代为发起授权的: azp字段
- access_token里包含有效期: exp(过期时间)， iat(颁发时间)

参见下面一个真实access_token解码之后的例子: 

![](https://github.com/tonitonishier/Operation-Manual/blob/main/%E5%BA%94%E7%94%A8%E6%8E%88%E6%9D%83%E5%8A%9F%E8%83%BDAPI%E6%8E%A5%E5%8F%A3%E8%B0%83%E7%94%A8%E8%AF%B4%E6%98%8E%E6%96%87%E6%A1%A3%E7%94%A8%E5%9B%BE/SQAPI1.png?raw=true)

### RS对access_token的校验

- 需要校验签名的合法性
- 签名算法alg: 只认RS256或HS256， 取决于RS在YuFu侧的签名算法配置
- 需要校验有效期: exp， iat
- 需要校验aud， azp， iss字段
- 需要解析用户的perms权限: 当perms字段的值为null或者空数组时，认为没有权限
- 通过sub或者preferred_username得知用户在YuFu侧的唯一标识