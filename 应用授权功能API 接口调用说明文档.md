# 应用授权功能API 接口调用说明文档

## 背景

YuFu作为一个Authorization Server(以下简称AS)， 可以授权用户zhangsan访问指定的Resource Server(以下简称RS，RS对应YuFu中的应用系统)。授权成功之后， YuFu给RS颁发access_token， RS通过解析access_token的合法性以及内容，来判断zhangsan是否有权限访问Resource Server， 具体参见下文"RS如何信任AS"。

YuFu作为AS， 提供两种场景下的授权接口:

- 授权码模式: zhangsan通过浏览器发起授权请求；

- token更新模式: 服务端->YuFu服务端的调用，更新授权；

## 接入前提

需要在YuFu的管理控制台上添加如下配置:

- 添加应用：在YuFu创建OIDC协议的应用实例并配置该应用的redirect_uri，应用创建完成后，需保留该应用的client_id/client_secret以便后续接口调用；

![](C:\Users\cloud\Desktop\文档用图\SQAPI2.png)

- 添加Resource Server：在YuFu的API管理功能中创建API并配置该API（Resource Server）对应的audience；

![](C:\Users\cloud\Desktop\文档用图\SQAPI3.png)

- 添加Resource Server对应的权限，且把权限分配给人；

![](C:\Users\cloud\Desktop\文档用图\SQAPI4.png)

![SQAPI5](C:\Users\cloud\Desktop\文档用图\SQAPI5.png)

![SQAPI6](C:\Users\cloud\Desktop\文档用图\SQAPI6.png)

- 从YuFu侧获取授权接口的访问地址，以及access_token的签名密钥；

![](C:\Users\cloud\Desktop\文档用图\SQAPI7.png)

应用侧需要实现的接口:

- redirect_uri: 在"授权码模式"中，YuFu会把授权码(code)返回至指定的redirect_uri. 所以应用侧需要实现redirect_uri对应的接口，来获取code， 然后进行后续/token调用，详情请见下文。

## 授权码模式

授权码模式是用户zhangsan通过浏览器发起授权的一个场景， 需要两个步骤来完成整个授权流程：

- 浏览器发起调用/authorize接口请求授权码；
- 后台调用/token接口: 拿授权码换取access_token/refresh_token；

授权码模式还适用于zhangsan未登陆场景，当YuFu收到一个/authorize授权请求时，会检查当前用户zhangsan是否在YuFu测处于登陆状态，若未登陆则重定向至YuFu做登陆. 若已登陆，则返回授权码。

### authorize

发起授权请求

Endpoint: GET https://<authorization_server_base>/authorize

Params:

| **参数名**                                    | **必填** | **说明**                                                     |
| --------------------------------------------- | -------- | ------------------------------------------------------------ |
| client_id                                     | 是       | 发起授权请求的应用client_id. (IEG场景即为Gateway的client_id， 参见接入前提) |
| response_type                                 | 是       | 为code， 即授权码模式                                        |
| redirect_uri                                  | 是       | 回调链接，AS会把生成的授权码以重定向的方式返回至redirect_uri (参见接入前提) |
| state                                         | 否       | CSRF安全参数， 建议使用: 当AS重定向至redirect_uri时，会原封不动的把state参数回传 |
| nonce                                         | 否       | 防重放安全参数，建议使用: AS颁发的id_token里会通过nonce这个claim来回传， 以便于id_token合法性校验 |
| scope                                         | 是       | 授权的范围: 多值属性，以空格分割. 通常情况下传入openid profile  email， 调用者可以把Resource Server的Permission作为scope参数传递  当包括offline_access时表示请求AS返回refresh_token  注意: IEG需要额外传递ieg:bizid来表示需要在access_token里返回用户所在的业务线 |
| audience                                      | 否       | 授权请求访问Resource Server的audience属性.   当不传时，颁发的access_token只能用来访问YuFu的/userinfo接口， 而不能访问Resource Server |
| 以下参数仅针对PKCE场景(SPA)生效(协议标准参数) |          |                                                              |
| code_challenge                                | 是       | 随机生成，需要和下面Token接口的code_verifier配对，生成算法为code_challenge_method约定的算法， 参见 |
| code_challenge_method                         | 否       | 可为S256. 出于安全考虑，不支持PLAIN                          |

Response:

通过302 Redirect， 以HTTP GET的方式， 返回state以及生成的授权码code至redirect_uri. 后续可以通过code来调用Token接口返回access_token/id_token/refresh_token

Example:GET https://<authorization_server_base>/authorize?  

response_type=code

&scope=openid%20profile%20email%20read:meeting

&client_id=s6BhdRkqt3

&state=af0ifjsldkj

&nonce=af867asfdbcda1safd

&audience=https%3A%2F%2Fmeeting-api

&ieg_biz_external_id=12345678

&redirect_uri=https%3A%2F%2Fclient.example.org%2Fcb

HTTP/1.1 302 Found 

Location: https://client.example.org/cb?  code=SplxlOBeZQQYbYS6WxSbIA

&state=af0ifjsldkj

### token

通过授权码请求access_token/id_token/refresh_token， 仅适用于应用后台->YuFu后台的调用场景

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

 HTTP/1.1 200 OK 

 Content-Type: application/json 

 {

   "access_token": "jwt_access_token_tbd"， //详细内容见下文

   "token_type": "Bearer"， 

   "refresh_token": "8xLOxBtZp8"，

   "expires_in": 3600，  // access_token的过期时间

   "id_token": "jwt_id_token" // optional 

   "refresh_token_expires_in": 36000 // YuFu私有属性，表示refresh_token的过期时间 

}

## token更新模式

token更新模式仅适用于应用后台→YuFu后台的调用场景， 携带通过授权码模式返回的refresh_token来更新access_token/id_token。

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

 HTTP/1.1 200 OK Content-Type: application/json

 {

   "access_token": "jwt_access_token_tbd"， // 详细内容见下文

   "token_type": "Bearer"，

   "expires_in": 3600，  // access_token的过期时间

   "id_token": "jwt_id_token" // optional

 }

若YuFu侧检查到refresh_token过期后，该接口会返回400， 且返回json格式的error. 用户必须使用授权码模式重新发起授权调用

比如: 

 {  

   "error": "invalid_grant"，

   "error_description": "invalid refresh_token"

 }

## logout登出接口

YuFu的登出接口需要调用者在浏览器中打开， 第三方调用者可以通过HTTP REDIRECT的方式引导浏览器执行YuFu侧的登出。

Endpoint: GET https://<yufu_idp_base>/logout

Params:

| 参数名    | 必填 | 说明                                                         |
| --------- | ---- | ------------------------------------------------------------ |
| return_to | 否   | 值为合法的URI.   当YuFu侧登出之后，会通过HTTP REDIRECT的方式跳转至return_to指定的URI.   若不携带该参数或者该参数是个非法的URI， YuFu将会默认跳转至YuFu的登陆界面. |

Limitation:

- YuFu目前仅支持SAML认证源的SLO

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
- access_token里包含这个token是由哪个应用代为发起授权的: azp字段.(IEG场景为Gateway的client_id)
- access_token里包含有效期: exp(过期时间)， iat(颁发时间)

参见下面一个真实access_token解码之后的例子: 

![](C:\Users\cloud\Desktop\文档用图\SQAPI1.png)

### RS对access_token的校验

- 需要校验签名的合法性
- 签名算法alg: 只认RS256或HS256， 取决于RS在YuFu侧的签名算法配置
- 需要校验有效期: exp， iat
- 需要校验aud， azp， iss字段
- 需要解析用户的perms权限: 当perms字段的值为null或者空数组时，认为没有权限
- 通过sub或者preferred_username得知用户在YuFu侧的唯一标识