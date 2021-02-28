---
title: cookie_session_token
date: 2021-2-28
cover:
top_img:
categories: 
    - java
    - javaEE
tags: 
mathjax: true
katex: true
---
##  Cookie 、Session和token
- 会话：用户打开了一个浏览器，点击了很多超链接，访问多个web资源，关闭浏览器，这个过程可以成为会话。
- Cookie通过在客户端记录信息确定用户身份，Session通过在服务器端记录信息确定用户身份。
## 1 Cookie
　　
    - Cookie实际上是一小段的文本信息。客户端请求服务器，如果服务器需要记录该用户状态，就使用response向客户端浏览器颁发一个Cookie。客户端浏览器会把Cookie保存起来。当浏览器再请求该网站时，浏览器把请求的网址连同该Cookie一同提交给服务器。服务器检查该Cookie，以此来辨认用户状态。服务器还可以根据需要修改Cookie的内容。
    

    Cookie[] cookies = req.getCookies()

## 2 Session
+ 除了使用Cookie，Web应用程序中还经常使用Session来记录客户端状态。Session是服务器端使用的一种记录客户端状态的机制，使用上比Cookie简单一些，相应的也增加了服务器的存储压力。
+ 如果说Cookie机制是通过检查客户身上的“通行证”来确定客户身份的话，那么Session机制就是通过检查服务器上的“客户明细表”来确认客户身份。Session相当于程序在服务器上建立的一份客户档案，客户来访的时候只需要查询客户档案表就可以了。
+ 如果禁用了cookie，URL地址重写是对客户端不支持Cookie的解决方案。URL地址重写的原理是将该用户Session的id信息重写到URL地址中。服务器能够解析重写后的URL获取Session的id。这样即使客户端不支持Cookie，也可以使用Session来记录用户状态。
+ 浏览器第一次访问时，服务器创建Session，然后将Session的Id以Cookie的形式发送回给浏览器，response. encodeURL(java.lang.String url)方法也将URL进行了重写，当点击刷新按钮第二次访问，由于火狐浏览器没有禁用cookie，所以第二次访问时带上了cookie，此时服务器就可以知道当前的客户端浏览器并没有禁用cookie，那么就通知response. encodeURL(java.lang.String url)方法不用将URL进行重写了。

### 2.1 HttpSession 服务端的技术

- 服务器会为每一个用户 创建一个独立的HttpSession
- HttpSession原理
    + 当用户第一次访问Servlet时,服务器端会给用户创建一个独立的Session
    + 并且生成一个SessionID,这个SessionID在响应浏览器的时候会被装进cookie中,从而被保存到浏览器中
    + 当用户再一次访问Servlet时,请求中会携带着cookie中的SessionID去访问
    + 服务器会根据这个SessionID去查看是否有对应的Session对象
    + 有就拿出来使用;没有就创建一个Session(相当于用户第一次访问)

域的范围:
    Context域 > Session域 > Request域
    Session域 只要会话不结束就会存在 但是Session有默认的存活时间(30分钟)
## 3 注意
    + 单独使用cookie时，本地存了cookie中所有key-value对，每次传递要传递所有的键值对。
    + session单独使用时，每次只传递sessionid,本地只存储cookieId。
    + 如果cookie被禁用，则使用url重传的方式，sessionId存在本地缓存localStorage中。还有说法是保存在网页源码上，拼接。
## 4 token

- 流程
1. 对用户信息进行加密，制作一个签名，存到token传到前台。
2. 再次数据传到后台时，对数据再一次计算一次签名，如果与token中的签名相同，则验证成功。
![](http://note.youdao.com/yws/public/resource/bca95011244292ba9b4a461a47885868/xmlnote/B614F88C427643D8888CFA1A67624429/7081)
![](http://note.youdao.com/yws/public/resource/bca95011244292ba9b4a461a47885868/xmlnote/834BD596E8EE4D7794676445B35724FE/7083)
- Token解决的问题：
1. Token完全由应用管理，所以它可以避开同源策略。
2. Token可以避免CSRF攻击
3. Token可以是无状态的，可以在多个服务器共享
- Token优点
1. 无状态、可扩展
    + 在客户端存储的Tokens是无状态的，并且能够被扩展。基于这种无状态和不存储Session信息，负载负载均衡器能够将用户信息从一个服务传到其他服务器上。tokens自己hold住了用户的验证信息。
2. 支持移动设备
3. 跨程序调用
    + Tokens能够创建与其它程序共享权限的程序。
4. 安全
    + 请求中发送token而不再是发送cookie能够防止CSRF(跨站请求伪造)。即使在客户端使用cookie存储token，cookie也仅仅是一个存储机制而不是用于认证。不将信息存储在Session中，让我们少了对session操作。token是有时效的，一段时间之后用户需要重新验证。

## 5 CSRF（跨站请求伪造）

挟制用户在当前已登录的Web应用程序上执行非本意的操作的攻击方法。跟跨网站脚本（XSS）相比，XSS 利用的是用户对指定网站的信任，CSRF 利用的是网站对用户网页浏览器的信任。

cookie：用户点击了链接，cookie未失效，导致发起请求后后端以为是用户正常操作，于是进行扣款操作；
token：用户点击链接，由于浏览器不会自动带上token，所以即使发了请求，后端的token验证不会通过，所以不会进行扣款操作

## 6 XSS（跨站脚本攻击）

就是利用了特殊手段在用户访问的网页中植入了恶意代码，可以是js脚本等，这些脚本就会在用户的浏览器上执行，达到攻击效果。

1、盗用cookie，获取敏感信息。

2、利用植入Flash，通过crossdomain权限设置进一步获取更高权限；或者利用Java等得到类似的操作。

3、利用iframe、frame、XMLHttpRequest或上述Flash等方式，以（被攻击）用户的身份执行一些管理动作，或执行一些一般的如发微博、加好友、发私信等操作。

4、利用可被攻击的域受到其他域信任的特点，以受信任来源的身份请求一些平时不允许的操作，如进行不当的投票活动。

5、在访问量极大的一些页面上的XSS可以攻击一些小型网站，实现DDoS攻击的效果。