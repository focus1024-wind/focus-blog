---
title: 基于flask-oidc的OIDC协议授权码模式单点登录SSO实现
description: 基于flask-oidc的OIDC协议授权码模式单点登录SSO实现
date: 2023-07-05
slug: python/sso/sso_with_flask-oidc
image: 
categories:
    - Python
tags:
    - Python
    - Flask
    - SSO
    - OIDC
---

关于SSO单点登录、OIDC协议、授权码模式等相关概念详见[基于OIDC的SSO单点登录文章内容](https://blog.csdn.net/qq_42482078/article/details/131555103?spm=1001.2014.3001.5501)


## 安装flask-oidc
```shell
pip install flask-oidc
```

## flask-oidc实现OIDC的授权码模式流程
- 应用注册: 在提供OIDC服务的认证系统中注册应用信息
- 授权流程:
	- 携带由应用生成的`state`(防CSRF跨域信息)和在认证系统中注册的应用信息重定向请求认证系统的授权服务
	- 授权服务重定向到登录界面完成身份认证，返回`code`信息应用回调地址中
	- 应用根据`code`信息请求认证系统的令牌服务，获取`access_token`令牌
	- 应用根据已获取`access_token`令牌来获取到注册用户的身份认证等信息

## 代码实现
### 依赖库安装
```shell
pip instal flask-oidc2
```

### flask-oidc认证配置
- 添加oidc_client_secrets.json文件
	- oidc_client_secrets.json主要为设置认证服务器等相关信息
	- client_id：在认证系统注册的应用ID，必要参数
	- client_secret：在认证系统注册的应用密钥，必要参数
	- auth_uri: 认证系统授权接口，必要参数
	- token_uri: 认证系统获取令牌信息接口，必要参数
	- userinfo_uri: 认证系统后去userinfo信息接口
	- issuer: 身份提供程序的“颁发者”接口
```json
{
  "web": {
    "client_id": "${client_id}",
    "client_secret": "${client_secret}",
    "auth_uri": "${auth_uri}",
    "token_uri": "${token_uri}",
    "userinfo_uri": "${userinfo_uri}",
    "issuer": "${issuer}"
  }
}
```
- flask-oidc参数配置:
	- OIDC_CLIENT_SECRETS: oidc_client_secrets.json文件路径
	- OIDC_SCOPES: 需要获取的用户信息值（认证系统提供）
	- OVERWRITE_REDIRECT_URI: 在认证系统中注册的本应用回调接口
以下为flask-oidc的参数配置示例：
```python
class OIDC_Config:
	OIDC_CLIENT_SECRETS = r'./oidc_client_secrets.json'
	OIDC_SCOPES = ['openid', 'email', 'profile']
	OVERWRITE_REDIRECT_URI = "${your_domain}/api/sso"
```
- flask-oidc初始化
```python
# flask导入参数
app.config.from_object(OIDC_Config)

# flask-oidc获取flask关于OIDC的相关参数配置
from flask_oidc import OpenIDConnect

oidc = OpenIDConnect()
oidc.init_app(app)
```

### 获取授权码
flask-oidck库实现的`require_login`装饰器可以根据当前的参数信息，自动前往配置的`${auth_uri}`请求授权。`require_login`装饰器会自动生成`state`携带应用信息作为参数请求`${auth_uri}`。当`${auth_uri}`的返回类型`response_type`类型为`id_token`时完成授权认证操作。认证服务器执行完后，返回应用系统回调地址。
```python
@oidc.require_login
@app.route('/api/sso')
def sso_login():
	return 'SSO Login'
```

### 兑换token令牌
在授权码模式下，调用`require_login`装饰器，认证系统会返回防止跨域的`state`信息和授权码`code`。授权码`code`作为中间参数，并不携带我们希望获得的用户信息，无法完成登录操作。flask-oidc无法自动根据`code`获取toekn，所以我们需要手动发起请求获取token令牌。

其中，请求参数应设置为`"Content-Type": "application/x-www-form-urlencoded"`，并且对参数使用`urlencode()`方法进行编码，否则认证系统可能无法识别参数信息。
```python
import json
import requests
from urllib.parse import urlencode

@oidc.require_login
@app.route('/api/sso')
def sso_login(code, state):
	logger.info(f"SSO Login: code: {code}, state: {state}")

	# 使用code向授权服务器发送请求换取token
    headers = {
    	"Accept": "*/*",
    	"Content-Type": "application/x-www-form-urlencoded"
    }

    payload = urlencode({
    	"client_id": oidc.flow.client_id,
    	"client_secret": oidc.flow.client_secret,
    	"grant_type": "authorization_code",
    	"redirect_uri": current_app.config['OVERWRITE_REDIRECT_URI'],
    	"code": code
    })

	# 获取授权token
    token_response = json.loads(requests.request("POST", oidc.flow.token_uri, headers=headers, data=payload).text)

	# 登录
    oidc.set_cookie_id_token(token_response['id_token'])
    # 获取username
    username = oidc.user_getfield('email', access_token=token_response['access_token']).split('@')[0]
    logger.info(f'sso login, username: {username}')
	return 'SSO Login'
```