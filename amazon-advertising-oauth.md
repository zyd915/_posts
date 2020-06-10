---
title: 亚马逊广告授权流程说明
date: 2020-05-07 21:10:28
updated: 2020-05-07 21:10:28
tags: 
    - amazon
categories: tool
toc: true
excerpt: 亚马逊广告授权流程说明
---

### 广告授权进入第三方网站流程
![](https://static.studytime.xin/article/亚马逊授权广告流程.png)

### 授权流程说明

#### 一、用户进入第三方网站，如www.ABC.com

#### 二、第三方网站引导用户进入登录授权页 
第三方网站在自己网站放置亚马逊广告授权的入口，引导用户进入亚马逊登录授权页。

亚马逊登录授权页域名在不同亚马逊站点区域是不同的：

| Region | URL prefix |
| --- | --- |
| North America (NA) | https://www.amazon.com/ap/oa |
| Europe (EU) | https://eu.account.amazon.com/ap/oa |
| Far East (FE) | https://apac.account.amazon.com/ap/oa |

API调用具有以下参数：

| 参数 | 说明 |
| --- | --- |
| client_id | 开发者client_id |
| scope |  |
| response_type | 响应类型，始终为code |
| redirect_uri | 授权登录之后跳转的跳转网址 |
| state | 用于第三方自行校验session，防止跨域攻击 |

scope针对不同的业务类型有不同值，下面是官方文档中的说明。
- For the Sponsored Brands, Sponsored Display, and Sponsored Products APIs, set scope to cpc_advertising:campaign_management.
- For the Data Provider API, set scope to advertising::audiences.
- For the DSP API, set scope to advertising::campaign_management。

针对常用api等，只需要知道sb、sd、sp广告类型设置为cpc_advertising:campaign_management即可。

例如，要在北美（NA）区域生成授权码，请用您的值替换以下URL中的值：
```
https://www.amazon.com/ap/oa?client_id=YOUR_LWA_CLIENT_ID&scope=cpc_advertising:campaign_management&response_type=code&redirect_uri=YOUR_RETURN_URL&state=YOUR_STATE
```

接下来，第三方站点根据生成的url链接地址，浏览器中进行重定向，即可引导用户登录授权亚马逊广告。

### 三、用户登录、确认并同意授权

用户进入亚马逊广告授权引导页后，即需要用户登录，登录后，亚马逊将会把用户重定向到同意书页面。
![](https://static.studytime.xin/article/20200610173658.png)

要授予应用程序访问Amazon Advertising的权限，请选择允许。要拒绝应用程序访问Amazon Advertising，请选择取消。

### 4.授权后回调URI，得到授权码code

用户允许应用程序访问Amazon Advertising api的权限，即同意授权后，将跳转到步骤一中使用的URL相同区域的Amazon网站。同时此步骤中将会获得刷新令牌使用的code码。
最后将会重定向到步骤一中设置的redirect_uri链接，即第三方回调地址，注意此时会携带生成的code码。

### 5.调用授权URL以请求授权和刷新令牌
请求授权和刷新令牌接口域名在不同亚马逊站点区域是不同的。URL为：

| Region | Authorization URL |
| --- | --- |
| North America (NA) | https://api.amazon.com/auth/o2/token |
| Europe (EU) | https://api.amazon.co.uk/auth/o2/token |
| Far East (FE) | https://api.amazon.co.jp/auth/o2/token |

特殊说明：地区为Far East (FE)的Authorization URL，在中国大陆被墙了，可以使用 North America (NA)的代替。

接下来，构造API调用以检索授权和刷新令牌。该调用具有以下查询参数：

| 参数 | 说明 |
| --- | --- |
| grant_type | 必须是authorization_code |
| code |  |
| redirect_uri | 和步骤一种的redirect_uri一样 |
| client_id | 开发者client_id |
| client_secret | 开发者client_secret |

API调用使用POST,需要以下header头：
```
Content-Type:application/x-www-form-urlencoded
charset=UTF-8
```

此请求的响应是以JSON形式返回，返回的参数有：

| 字段 | 说明 |
| --- | --- |
| access_token | The authorization token |
| refresh_token | The refresh token |
| token_type | 总是bearer |
| expires_in | 过期时间秒，默认3600 |

For example:
```
{
    "access_token": "",
    "refresh_token": "",
    "token_type": "bearer",
    "expires_in": 3600
}
```
### 6.第三方根据返回的refresh_token，变更授权状态

### 7.授权成功

### 8.授权令牌后，您就可以使用API进行调用了

## 代码实现
[基于亚马逊广告sdk easy-amazon-advertising](https://www.studytime.xin/article/easy-amazon-advertising.html)
### 授权引导页生成
```
$redirect_uri = www.ABC.com . '/api/advertising/authorize';
$config = [    
    'clientId'     => config('adv.clientId'),    
    'clientSecret' => config('adv.clientSecret'),    
    'region'       => api_region_control($seller->region),    
    'grant_type'   => 'authorization_page',    
    'redirect_uri' => $redirect_uri,    
    'state'     => encrypt_openssl(['id' => userid123])
 ];
$app = Factory::make('BaseService', $config);
$result = $app->oauth->authorizationURL();
return !empty($result['code']) && $result['code'] == 200 ? 
$result['response'] : '';
```

特殊说明:encrypt_openssl方法为可解加密方法。

### 调用授权URL以请求授权和刷新令牌，变更授权状态
```
$info = decrypt_openssl(urldecode($params['state']));
$user = User::query()->where(['id' => $info['id']])->first();


$config = [    
    'clientId'     => config('adv.clientId'),    
    'clientSecret' => config('adv.clientSecret'),    
    'region'       => api_region_control($region),   
    'grant_type'   => 'authorization_code',   
    'redirect_uri' => $redirect_uri,    
    'code'         => $code,
 ];

$app = Factory::make('BaseService', $config);
$result = $app->oauth->token();

return !empty($result['code']) && $result['code'] == 200 ? 
$result['response'] : [];
```

