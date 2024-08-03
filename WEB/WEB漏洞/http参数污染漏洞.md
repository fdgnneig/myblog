https://www.cnblogs.com/wjrblogs/p/12966636.html

HTTP参数污染学习
HTTP参数污染 --- HPP#
参考：

参数污染漏洞（HPP）挖掘技巧及实战案例全汇总
视频内容
HPP，简而言之，就是给参数赋上多个值。

比如： https://www.baidu.com/s?wd=asdqwe&wd=http参数污染
百度究竟会给出有关 asdqwe 的相关信息，还是 http参数污染 的相关信息，还是两者兼并？事实证明，百度只取第一个关键词 asdqwe，不同的服务器架构会取不同的值

原因：由于现行的HTTP标准没有提及在遇到多个输入值给相同的参数赋值时应该怎样处理，而且不同的网站后端做出的处理方式是不同的，从而造成解析错误

下面是不同服务端所处理的方式

Web服务器	参数获取函数	获取到的参数
PHP/Apache	$_GET(“par”)	Last
JSP/Tomcat	Request.getParameter(“par”)	First
Perl(CGI)/Apache	Param(“par”)	First
Python/Apache	getvalue(“par”)	All (List)
ASP/IIS	Request.QueryString(“par”)	All (comma-delimited string)
推荐一个自动识别前后端框架的插件 wappalyzer，chrome、firefox、edge 都有

用处：

逻辑漏洞：
任意URL跳转
例如：
www.famous.website?ret_url=subdomain.famous.website ，由于后端做了限制，当我们把 ret_url 改成别的不同源的域名(如baidu.com)时会报错
但是我们可以利用HPP，将请求地址变成 www.famous.website?ret_url=subdomain.famous.website&ret_url=baidu.com 时，由于服务器逻辑错误
使用第一个 ret_url 做校验参数，而第二个 ret_url 参数做跳转目的地址。于是这样便可成功绕过限制，形成任意 URL 跳转

任意密码重置(短信爆破)
一般重置密码的时候，会发送短信到用户手机
比如GET/POST传递的参数为：phone=13888888888
我们一般会去想，能不能发送验证码到自己的手机，于是可以构造成： phone=13888888888,12345678901 或者 phone=13888888888;12345678901 或者 phone={13888888888,12345678901}
等等一些情况，有时候能通过，但是有些时候会出现 号码不合法 的情况，此时便可以考虑利用 HPP —— phone=13888888888&phone=12345678901，如果恰好服务器用第一个号码验证是否存在该用户，而使用第二个号码发送短信时，我们便可以接管该用户

绕过 WAF
原理：当 WAF 检测端与服务器端webapp使用的参数不一致时，便可绕过 WAF 限制
WAF框架：

框架	取值
flask	First
django	Last
SQL注入绕过
方式1：
例如：WAF 端采用 flask 框架而服务器 webapp 端采用 PHP/Apache 框架时，由于 WAF 取第一个参数，app 取第二个参数，利用 HPP 便可绕过 WAF 检测
www.xxx.com/search.php?id=1&id=7 union select 1,2,database()

方式2：
例如：WAF 端只检测单个参数，而服务器 webapp 端会将所有参数合并起来检测
www.xxx.com/search.php?id=1&id=%20union%20&id=select%201,2&id=,database() 利用拼接的方式绕过 WAF

XSS
跟 SQL 注入相似，检测过滤与实际采纳的参数不一致
可能有 HPP 的地方#
很多可能存在越权漏洞的地方，当做了权限验证时，可以试一试提交多个参数
绕 WAF 的时候
 分类: WEB安全知识学习笔记
 标签: HPP