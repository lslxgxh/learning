## 状态码
状态码	   类别	                                   原因短语
1XX	      Informational（信息性状态码）	           接收到请求正在处理
2XX	      Success（成功状态吗）	                  请求正常处理完毕
3XX	      Redirection（重定向状态码）	             需要进行附加操作以完成请求
4XX	      Client Error（客户端状态错误码）	        服务器无法处理请求
5XX	      Server Error（信息性状态码）	           服务器处理请求错误
重要的几个：
200：请求被正常处理
400：请求报文语法有误，服务器无法识别
401：请求需要认证
403：请求的对应资源禁止被访问
404：服务器无法找到对应资源
500：服务器内部错误
503：服务器正忙


## http1.0 1.1 2.0 
### HTTP1.0	
* 无状态、无连接
### HTTP1.1	
* 持久连接
* 请求管道化
* 增加缓存处理（新的字段如cache-control）
* 增加Host字段、支持断点传输等（把文件分成几部分）
### HTTP2.0	
* 二进制分帧
* 多路复用（或连接共享）
* 头部压缩
* 服务器推送

## http与https
* HTTPS协议需要到CA申请证书，一般免费证书很少，需要交费。
* HTTP协议运行在TCP之上，所有传输的内容都是明文，HTTPS运行在SSL/TLS之上，SSL/TLS运行在TCP之上，所有传输的内容都经过加密的。
* HTTP和HTTPS使用的是完全不同的连接方式，用的端口也不一样，前者是80，后者是443。
* HTTPS可以有效的防止运营商劫持，解决了防劫持的一个大问题。

## 从输入url到页面展现发生了什么？
1. 根据地址栏输入的地址向DNS（Domain Name System）查询IP （dnf就是域名系统 可以用来查域名的，域名是指www.baidu.com 这种）
2. 通过IP向服务器发起TCP连接
3. 向服务器发起请求
4. 服务器返回请求内容
5. 浏览器开始解析渲染页面并显示
6. 关闭连接

## tcp三次握手四次挥手
网上查一下

## 为什么要三次
