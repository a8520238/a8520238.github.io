---
title: JWT
date: 2021-2-28
cover:
top_img:
categories: javaEE
tags: 
mathjax: true
katex: true
---
# JWT

## 1 含义

JWT是为了在网络应用环境间传递声明而执行的一种基于JSON的开放标准。该token被设计为紧凑且安全的，特别适用于分布式站点的单点登录（SSO）场景。JWT的声明一般被用来在身份提供者间传递被认证的用户身份信息，以便于从资源服务器获取资源，也可以增加一些额外的其它业务逻辑所必须的声明信息，该token也可以被用于认证，也可以被加密。
## 2 JWT构成

- 第一部分：头部（header）
- 第二部分：载荷（payload）
- 第三部分：签证

1. header
- 声明类型，这里是jwt
- 声明加密算法，通常直接使用HMAC SHA256
完整的头部就像下面这样的JSON：
```
{
    'typ':'JWT',
    'alg':'HS256'
}
```
然后将头部进行base64加密（该加密是可以对称解密的），构成了第一部分。
然后将头部进行base64加密（该加密是可以对称解密的），构成了第一部分。
```
eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9
```
2. playload
载荷就是存放有效信息的地方。这个名字像是特指飞机上承载的货品，这些有效信息包含三哥部分
- 标准中注册的声明
- 公共的声明
- 私有的声明

标准中注册的声明（建议但不强制使用）：
- iss:jwt签发者
- sub:jwt所面向的用户
- aud:接收jwt的一方
- exp:jwt的过期时间，这个过期时间必须要大于签发时间
- nbf:定义在什么时间之前，该jwt都是不可用的。
- iat:jwt的签发时间
- jti:jwt的唯一身份标识，主要用来作为一次性token，从而回避重放攻击。

公共的声明：

公共的声明可以添加任何的信息，一般添加用户的相关信息或其他业务需要的必要信息。但不建议添加敏感信息，因为该部分在客户端可解密。
```
{
  "sub": "1234567890",
  "name": "John Doe",
  "admin": true
}
```
然后将其进行base64加密，得到jwt的第二部分。
```
eyJzdWIiOiIxMjM0NTY3ODkwIiwibmFtZSI6IkpvaG4gRG9lIiwiYWRtaW4iOnRydWV9
```
3. signature

jwt的第三部分是一个签证信息，这个签证信息由三部分组成：
- header(base64后的)
- payload(base64后的)
- secret

这个部分需要base64加密后的header和base64加密后的payload使用.连接组成的字符串，然后通过header中声明的加密方式进行加盐secret组合加密，然后就构成了jwt的第三部分。

```
var encodedString = base64UrlEncode(header) + '.' + base64UrlEncode(payload);

var signature = HMACSHA256(encodedString, 'secret'); // TJVA95OrM7E2cBab30RMHrHDcEfxjoYZgeFONFh7HgQ
```
将上述三个部分连起来，构成了最终的jwt

注意：secret是保存在服务短的，jwt的签发生成也是在服务器端的，secret就是用来进行jwt的签发和jwt的验证，所以，他就是你服务端的私钥，在任何场景都不应该流露出去。一旦客户端直到这个secret，那就意味着客户端是可以自我签发jwt了。

如何应用：

一般是在请求头里加Authorization，并加上Bearer标注：

```
fetch('api/user/1', {
  headers: {
    'Authorization': 'Bearer ' + token
  }
})
```
![](http://note.youdao.com/yws/public/resource/bca95011244292ba9b4a461a47885868/xmlnote/472428DD33F94469B058FF1E17F9DCFB/9066)

## 3 总结

1. 优点
- 因为json的通用性，所以JWT是可以进行跨语言支持的，很多语言都可以使用。
- 因为有了payload部分，所以JWT可以在自身存储一些其他业务逻辑所必要的非敏感信息。
- 便于传输，jwt的构成非常简单，字节占用很小，所以它是非常便于传输的。
- 它不需要再服务端保存会话信息，所以它易于应用的扩展。

2. 安全相关
- 不应该在jwt的payload部分存放敏感信息，因为该部分是客户端可解密的部分。
- 保护好secret私钥，该私钥非常重要
- 尽量使用https协议