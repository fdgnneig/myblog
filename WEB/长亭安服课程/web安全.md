# 推荐
- 图解http请求
- 白帽子讲web安全
- hack the box
- hack one
- hack101.com
- kali tools
- 18年版本的kali 工具全 依赖好
- seebug漏洞库
- 在线工具
  - https://forum.ywhack.com/bountytips.php?tools
- hack101 书籍
  - https://wizardforcel.gitbooks.io/web-hacking-101/content/
- javascript学习网站
  - https://javascript.info
# 入侵途径
![](pic/2022-03-17-20-46-31.png)
# 渗透测试类型
![](pic/2022-03-17-20-58-39.png)
# 渗透测试手段
![](pic/2022-03-17-21-23-28.png)
![](pic/2022-03-17-21-23-39.png)
![](pic/2022-03-17-21-28-53.png)
# 渗透测试流程
![](pic/2022-03-17-21-34-07.png)
## 信息收集
![](pic/2022-03-17-21-44-06.png)
## 漏洞挖掘
![](pic/2022-03-17-21-44-32.png)
## 漏洞利用
![](pic/2022-03-17-21-46-52.png)
## 权限提升
![](pic/2022-03-17-21-49-58.png)
## 端口转发
![](pic/2022-03-17-21-50-16.png)
## 痕迹清除
![](pic/2022-03-17-21-50-36.png)
![](pic/2022-03-17-21-50-57.png)    
# cookie
- cookie由web应用生成，保存在用户浏览器中，浏览器访问服务器时会携带cookie数据，则服务器处理该请求时则使用对应用户的数据，并且在响应包中包含该用户的cooike数据，cookie有自己的生命周期
  - ![](pic/2022-03-19-08-19-31.png)
  - ![](pic/2022-03-19-08-20-39.png)
# session
- ![](pic/2022-03-19-08-21-40.png)
- ![](pic/2022-03-19-08-22-03.png)
# cookie与session的区别
![](pic/2022-03-19-08-22-58.png)
- 通过修改cookie中的标记身份的字段，尝试以其他用户身份登录，从而实现水平越权和垂直越权
# web框架
![](pic/2022-03-19-08-25-59.png)
# 网页渲染过程
![](pic/2022-03-19-08-34-51.png)
![](pic/2022-03-19-08-28-30.png)
![](pic/2022-03-19-08-31-54.png)
# 浏览器特性于安全策略
![](pic/2022-03-19-08-33-12.png)
- http请求头中如果存在相关头部，则可以用于指示允许跨域资源访问
- 为了验证相关请求来源于同一个域，可以在请求中增加referer字段，从而验证请求来源
![](pic/2022-03-19-08-46-13.png)
![](pic/2022-03-19-08-47-56.png)
![](pic/2022-03-19-08-50-48.png)
![](pic/2022-03-19-08-51-45.png)
![](pic/2022-03-19-08-54-42.png)
![](pic/2022-03-19-08-55-24.png)
# kali工具
- tools.kali.org中查询工具用法
![](pic/2022-03-19-11-25-16.png)
![](pic/2022-03-19-13-52-06.png)
- enum4linux：用于扫描windows、linux主机以及samba系统、smb相关，可以返回windows主机中共享文件夹等信息
  - enum4linux ip地址
- masscan：端口扫描器，用于探测指定网段的资产，扫描到之后，使用nmap，nc或自定义的socket扫描特定资产
  - ![](pic/2022-03-19-11-23-26.png)
  - ![](pic/2022-03-19-11-23-54.png)
- nmap 最强大的是提供了大量脚本
  - 使用redis-info脚本扫描指定ip的redis
  - ![](pic/2022-03-19-11-29-38.png)
  - nc ip地址 端口号 可以尝试直接连接打开的redis 
- nikto 用于扫描web站点，可以识别漏洞
  - ![](pic/2022-03-19-11-35-25.png)
- sublist3r 用于枚举子域名 类似的工具censys 云悉 子域名挖掘机
  - 扫描指定域名
  - ![](pic/2022-03-19-11-37-24.png)
  - 百度搜索子域名的接口
  - ![](pic/2022-03-19-11-39-48.png)
- exploitdb
  - 可以用kali中的工具直接搜索exp
  - ![](pic/2022-03-19-11-42-51.png)
- linux exploit suggester 提权漏洞库
  - ![](pic/2022-03-19-11-46-06.png)
- beef xss相关框架
- armitage 后渗透 msf图形化工具
- backdoor-factory 后门工具，kali自带的不好用
- routersploit 类似msf，针对路由器设备攻击的漏洞库
- msf
- setoolkit 社工钓鱼框架
- sqlmap
- bp
- WPScan
- wfuzz 神器 用于webfuzz
- websploit 中间人框架
- owasp zap
- w3af
- gobuster 目录fuzz扫描器，可以爆破域名
- recom-ng 信息收集工具
- netdiscover 网络发现
- shodan
- what cms 测试cms指纹
- DIRB 扫描目录的工具 类似于御剑
  - ![](pic/2022-03-19-13-53-50.png)
- curl
- john the ripper
- wordlist 字典
- nishang
- dns2tcp
- webshells 生成各种语言的webshell
- hydra 用于爆破
- apktool
- impacket
- locate 搜索本地文件位置
- goby 用于识别网站bander 免费版云悉
# 信息收集
- whois？
  - ![](pic/2022-03-19-14-42-39.png)
  - ![](pic/2022-03-19-14-43-59.png)
  - 搜索黑伞攻防实验室的腾讯云文章
  - 用来搜索域名对应的ip和所有者等信息
  - 可以作为kali中的一个工具使用
    - ![](pic/2022-03-19-14-35-05.png)
  - 获得域名所有者以及邮箱后可以进行反查(站长之家)
    - ![](pic/2022-03-19-14-37-16.png)
  - 部分资产没有域名，只有ip，需要搜索已经的存在域名的的ip，根据该ip猜测其他资产ip
    - 存在泛解析问题即域名abc.com会将*abc.com （*即任意字符） 解析为abc.com
- DNS解析记录，如果dns解析记录存在，说明对应子域名的确存在，故可以尝试搜索dns解析记录，也可以从解析记录中发现设备真实ip
  - ![](pic/2022-03-19-14-45-03.png)
  - ![](pic/2022-03-19-14-45-22.png)
  - ![](pic/2022-03-19-14-54-14.png)
  - ![](pic/2022-03-19-14-54-47.png)
  - ![](pic/2022-03-19-14-55-47.png)
  - ![](pic/2022-03-19-14-56-34.png)
- 子域名
  - 发现子域名的意义在于发现更多的目标资产，例如边缘的资产
  - 可以使用一系列工具发现
  - sublist3r、censys、云悉、子域名挖掘机、gobuster
  - ![](pic/2022-03-19-15-06-56.png)
  - 子域名接管漏洞，主要原因是厂商相关子域名到期未注册，任何人均可注册，且dns记录未被清除
- 代码泄露
  - searchcode.com
- 主机ip段有哪些？
- 公网ip有哪些？
- 公司人员？
- 安全防御与防火墙
- 网站开发者？开发软件？开发方？网页底部是否有信息？找指纹信息，类似Power by
- 服务器信息？容器类型？服务器真实路径？从报错信息中找
- 搜索引擎：谷歌hack fofa shodan(kali中的shodan可以结合自己账户的api密钥，从而在kali中使用shodan进行搜索，可以用于搜索摄像头，搜索webcam) zoomeye（国内资产）
  - ![](pic/2022-03-19-14-19-07.png)
  - shodan.io
    - ![](pic/2022-03-19-15-19-27.png)
  - google hack
    - ![](pic/2022-03-19-15-09-57.png)
    - ![](pic/2022-03-19-15-12-05.png)
    - ![](pic/2022-03-19-15-14-48.png)
- 社会工程学
  - 公开信息
    - 盘搜搜 针对百度云盘信息进行搜索
    - 社交网络 脉脉 qq 微信 人人网 求职网站
    - 新闻报道
  - 社工库
- 分析网站架构
  - 工具：谷歌插件，发现类似网站，从而找到与目标网站类似架构的网站 
  - 谷歌插件 wappalyzer 用于分析网站架构
  - fofa和shodan对应的谷歌插件
  - 云悉网站搜索指纹
  - ![](pic/2022-03-19-15-25-05.png)
  - ![](pic/2022-03-19-15-29-10.png)
  - 谷歌插件 linkgrabber 可以搜索与当前网页有关的其他链接  
- 威胁情报
  - threatcrowd.org
- js文件中泄露相关数据或者接口，通过构造url访问指定路径下的js文件，可以读取该文件内容，泄露其中数据
- 信息收集目标
  - ![](pic/2022-03-20-11-16-46.png)
# 信息泄露
- 信息泄露分析整体思路
  - ![](pic/2022-03-19-16-20-49.png)
- 概要
  - ![](pic/2022-03-20-11-50-58.png)
  - 可以在url之后写错误字符串，从而产生报错信息
  - 通过在url后面加robots.txt
- ![](pic/2022-03-20-12-29-17.png)
- ![](pic/2022-03-20-12-29-39.png)
- ![](pic/2022-03-20-12-30-11.png)
- ![](pic/2022-03-20-12-30-24.png)
- ![](pic/2022-03-20-12-30-36.png)
- ![](pic/2022-03-20-12-31-26.png)
- ![](pic/2022-03-20-12-32-10.png)
- ![](pic/2022-03-20-12-32-58.png)
- ![](pic/2022-03-20-12-33-44.png)
- ![](pic/2022-03-20-12-33-57.png)
- ![](pic/2022-03-20-12-34-47.png)
- svn信息泄露
  - ![](pic/2022-03-20-12-35-27.png)
- ![](pic/2022-03-20-12-36-46.png) 
- ![](pic/2022-03-20-12-37-42.png)
- ![](pic/2022-03-20-12-38-01.png)
- ![](pic/2022-03-20-12-38-48.png)
- ![](pic/2022-03-20-12-39-27.png)
- ![](pic/2022-03-20-12-42-03.png)
- ![](pic/2022-03-20-12-42-14.png)
- ![](pic/2022-03-20-12-42-28.png)
- ![](pic/2022-03-20-12-42-42.png)
- ![](pic/2022-03-20-13-13-53.png)
- ![](pic/2022-03-20-13-17-12.png)
- ![](pic/2022-03-20-13-18-43.png)
- ![](pic/2022-03-20-13-19-34.png)
- ![](pic/2022-03-20-13-19-56.png)
- ![](pic/2022-03-20-13-20-36.png)
- ![](pic/2022-03-20-13-29-39.png)
- ![](pic/2022-03-20-13-29-51.png)
- ![](pic/2022-03-20-13-30-07.png)
- ![](pic/2022-03-20-13-30-24.png)
- ![](pic/2022-03-20-13-31-20.png)
- ![](pic/2022-03-20-13-31-40.png)
- js敏感文件
- ![](pic/2022-03-20-13-32-15.png)
# 渗透测试工具集
- nessus
- AWVS
- burpsuite
  - 针对无回显的命令执行，sql注入等漏洞，可以使用带外的技巧验证漏洞是否存在以及触发成功
    - ![](pic/2022-03-20-17-17-58.png)
  - bp中存在带外平台
  - 以下payload可以通过带外平台带出数据
    - ![](pic/2022-03-20-17-22-39.png)
    - ![](pic/2022-03-20-17-23-27.png)  
  - ![](pic/2022-03-20-17-32-49.png)
- OWASP ZAP 类似于bp
- nmap扫描套路
  - ![](pic/2022-03-20-20-57-40.png)
  - ![](pic/2022-03-20-20-59-05.png)
# html+css+js
- 尝试网页页面的xss注入时，部分waf会过滤<script> </script> 字符串，此时可以使用<a href=xxx> </a>或<img src=xxx />等html标签进行注入，若注入成功，则会在当前html页面显示对应html元素
- <img src=xxx />作为xss中的注入内容时，如果在标签中增加onerror标志，则可以实现弹窗
- <!--xxx--> html中的注释
- 不同的html标签均可以作为前端输入，从而尝试将前端输入回显到html页面中，即尝试触发xss，例如<input /> <img />等标签
  - 下述标签可以用于尝试注入
  - ![](pic/2022-03-21-21-50-58.png)
  - ![](pic/2022-03-21-21-51-29.png)
  - 使用iframe标签可以用于测试内网情况，iframe标签用于嵌套页面，可以在一个html页面中嵌套其他html页面，从而在一个浏览器界面中显示不止一个页面
    - ![](pic/2022-03-21-22-07-30.png)
    - ![](pic/2022-03-21-22-07-46.png)
    - ![](pic/2022-03-21-22-10-26.png)
- html字符实体可以用于尝试绕过一些限制
  - ![](pic/2022-03-21-22-18-05.png) 
- css即在html元素中使用style属性
  - ![](pic/2022-03-21-22-21-04.png)
- hide属性中可能隐藏xss漏洞
  - ![](pic/2022-03-21-22-24-14.png)
- xss输出cookie值信息
  - alert(document.cookie)
- html事件(可以用于触发xss)
  - ![](pic/2022-03-21-22-26-36.png)
  - ![](pic/2022-03-21-22-27-06.png)
  - ![](pic/2022-03-21-22-28-47.png)
- 蜜罐通过在攻击者的浏览器中执行js代码，从而实现对攻击者进行溯源，包括攻击者的账户密码、攻击者源ip 
- js
  - ![](pic/2022-03-22-19-47-01.png)
  - ![](pic/2022-03-22-19-48-19.png)
  - ![](pic/2022-03-22-19-48-29.png)
    - js使用alert输出的过程中可以使用alert`xxx` 替代alert(xxx)
  - ![](pic/2022-03-22-19-55-18.png)
  - ![](pic/2022-03-22-19-55-33.png)
  - ![](pic/2022-03-22-19-56-43.png)
  - ![](pic/2022-03-22-19-57-47.png)
  - ![](pic/2022-03-22-19-58-24.png)
  - ajax中会针对某个接口发起请求，可以针对此类接口进行测试，从而扩展攻击面
  - ![](pic/2022-03-22-19-59-07.png)
  - ![](pic/2022-03-22-20-10-11.png)
- dom
  - ![](pic/2022-03-22-20-11-27.png)
  - ![](pic/2022-03-22-20-12-28.png)
  - ![](pic/2022-03-22-20-13-28.png)
# 靶机练习
- 若网站的keystone文件泄露，使用keytool可以从keystone文件中提取出ssl加密密钥以及证书，获得证书后，可以在wireshark中导入该证书，此后即可解密该网站的https流量包
- 获得设备命令执行权限后，可以通过运行python的指定库，从而获取一个tty
  - ![](pic/2022-03-22-20-47-00.png)
# php基础
- ![](pic/2022-03-22-21-43-45.png)
- ![](pic/2022-03-22-21-48-10.png)
- ![](pic/2022-03-22-21-50-06.png)
- ![](pic/2022-03-22-21-51-10.png)
# csrf 客户端请求伪造
- ![](pic/2022-03-23-20-22-54.png)
- ![](pic/2022-03-23-20-26-30.png)
- ![](pic/2022-03-23-20-27-11.png)
- ![](pic/2022-03-23-20-29-08.png)
- ![](pic/2022-03-23-20-29-26.png)
- ![](pic/2022-03-23-20-29-41.png)
- ![](pic/2022-03-23-20-29-54.png)
- ![](pic/2022-03-23-20-30-03.png)
- ![](pic/2022-03-23-20-30-23.png)
- ![](pic/2022-03-23-20-30-33.png)
- ![](pic/2022-03-23-20-31-12.png)
# xss
- ![](pic/2022-03-23-20-41-56.png)
- ![](pic/2022-03-23-20-42-11.png)
# SSRF 服务端请求伪造
- ![](pic/2022-03-23-20-45-08.png) 
- ![](pic/2022-03-23-20-49-06.png)
- ![](pic/2022-03-23-20-54-23.png)
- ![](pic/2022-03-23-20-54-39.png)
- ![](pic/2022-03-23-20-55-15.png)
- ![](pic/2022-03-23-20-56-50.png)
- ![](pic/2022-03-23-21-00-02.png)
- ![](pic/2022-03-23-21-02-21.png) 
# 常见的编码方式
- ![](pic/2022-03-26-15-48-43.png)
- ![](pic/2022-03-26-15-50-03.png)
- ![](pic/2022-03-26-15-51-14.png)
- ![](pic/2022-03-26-15-51-51.png)
- ![](pic/2022-03-26-15-57-33.png)
- ![](pic/2022-03-26-15-58-35.png)
- ![](pic/2022-03-26-15-59-05.png)
- ![](pic/2022-03-26-15-59-45.png)
- ![](pic/2022-03-26-16-00-32.png)
- ![](pic/2022-03-26-16-01-39.png)
- ![](pic/2022-03-26-16-01-51.png)
- ![](pic/2022-03-26-16-02-07.png)
- ![](pic/2022-03-26-16-02-51.png)
- http请求中携带cookie的过程中可能会使用rot5/13进行编码
- ![](pic/2022-03-26-16-04-05.png)
- ![](pic/2022-03-26-16-04-30.png)
- ![](pic/2022-03-26-16-04-50.png)
- base64可以用于编码图片，视频等可以编码的数据
# 爆破
- ![](pic/2022-03-26-16-15-05.png)
- ![](pic/2022-03-26-16-15-40.png)
- ![](pic/2022-03-26-16-15-56.png)
- ![](pic/2022-03-26-16-20-18.png)
- ![](pic/2022-03-26-16-20-30.png)
- ![](pic/2022-03-26-16-20-50.png)
- ![](pic/2022-03-26-16-21-57.png)
- ![](pic/2022-03-26-16-24-57.png)
- ![](pic/2022-03-26-16-25-38.png)
- ![](pic/2022-03-26-16-26-01.png)
- ![](pic/2022-03-26-16-28-54.png)
# 未授权访问漏洞
- 检测redis未授权访问接口的脚本
  - nmap的脚本
  - ![](pic/2022-03-26-16-44-53.png)
  - 其他脚本 redis-unauth-check.py
- ![](pic/2022-03-26-16-47-03.png)
- ![](pic/2022-03-26-16-51-09.png)
- ![](pic/2022-03-26-16-51-22.png)
- ![](pic/2022-03-26-16-51-30.png)
- ![](pic/2022-03-26-16-52-27.png)
- ![](pic/2022-03-26-16-53-50.png)
  - 若没有redis-cli，则可以使用nc对设备redis进行连接
- ![](pic/2022-03-26-16-59-39.png)
- ![](pic/2022-03-26-17-00-14.png)
- ![](pic/2022-03-26-16-59-49.png)
- ![](pic/2022-03-26-17-02-05.png)
- ![](pic/2022-03-26-17-02-14.png)
- ![](pic/2022-03-26-17-03-10.png)
- ![](pic/2022-03-26-17-03-37.png)
- ![](pic/2022-03-26-17-03-50.png)
- ![](pic/2022-03-26-17-09-46.png)
- ![](pic/2022-03-26-17-10-09.png)
- ![](pic/2022-03-26-17-10-26.png)
- ![](pic/2022-03-26-17-10-42.png)
- ![](pic/2022-03-26-17-11-20.png)
- ![](pic/2022-03-26-17-11-31.png)
- ![](pic/2022-03-26-17-11-42.png)
- ![](pic/2022-03-26-17-20-26.png)
- ![](pic/2022-03-26-17-21-01.png)
- ![](pic/2022-03-26-17-21-33.png)
- ![](pic/2022-03-26-17-22-13.png)
- ![](pic/2022-03-26-17-22-37.png)
- ![](pic/2022-03-26-17-23-48.png)
- ![](pic/2022-03-26-17-24-30.png)
- ![](pic/2022-03-26-17-25-03.png)
- ![](pic/2022-03-26-17-25-14.png)
- ![](pic/2022-03-26-17-26-23.png)
- ![](pic/2022-03-26-17-47-25.png)
- ![](pic/2022-03-26-17-47-38.png)
- ![](pic/2022-03-26-17-48-38.png)
- ![](pic/2022-03-26-17-49-07.png)
- ![](pic/2022-03-26-17-49-22.png)
- ![](pic/2022-03-26-17-50-53.png)
- ![](pic/2022-03-26-17-51-02.png)
- ![](pic/2022-03-26-17-51-17.png)
# 目录遍历 
- ![](pic/2022-03-27-10-31-30.png)
- ![](pic/2022-03-27-10-33-06.png)
- ![](pic/2022-03-27-10-36-34.png)
- ![](pic/2022-03-27-10-38-27.png)
- ![](pic/2022-03-27-10-38-43.png)
- ![](pic/2022-03-27-10-38-52.png)
- ![](pic/2022-03-27-10-40-23.png)
- 将下述配置数据写入nginx主配置文件，从而可以实现文件上传
  - ![](pic/2022-03-27-10-47-02.png)
# 文件包含
- ![](pic/2022-03-27-10-47-47.png)
- ![](pic/2022-03-27-10-48-57.png)
- ![](pic/2022-03-27-10-50-58.png)
- ![](pic/2022-03-27-10-54-44.png)
- ![](pic/2022-03-27-10-56-32.png)
- ![](pic/2022-03-27-10-58-05.png)
- ![](pic/2022-03-27-11-06-26.png)
- ![](pic/2022-03-27-11-18-38.png)
- ![](pic/2022-03-27-11-20-29.png)
- ![](pic/2022-03-27-15-03-31.png)
- ![](pic/2022-03-27-15-03-48.png)
- ![](pic/2022-03-27-15-04-14.png)
- ![](pic/2022-03-27-15-04-39.png)
- ![](pic/2022-03-27-15-05-05.png)
- ![](pic/2022-03-27-15-05-55.png)
- ![](pic/2022-03-27-15-07-38.png)
- ![](pic/2022-03-27-15-07-59.png)
- ![](pic/2022-03-27-15-08-19.png)
- ![](pic/2022-03-27-15-08-34.png)
- ![](pic/2022-03-27-15-09-49.png)
- ![](pic/2022-03-27-15-10-08.png)
- ![](pic/2022-03-27-15-10-23.png)
- ![](pic/2022-03-27-15-10-35.png)
# 命令执行漏洞
- ![](pic/2022-03-27-15-23-35.png)
- ![](pic/2022-03-27-15-53-34.png)
- ![](pic/2022-03-27-16-00-41.png)
- ![](pic/2022-03-27-16-02-57.png)
- ![](pic/2022-03-27-16-12-03.png)
- ![](pic/2022-03-27-16-13-39.png)
- ![](pic/2022-03-27-16-14-36.png)
  - 反弹shell的接收端必须是公网
- ![](pic/2022-03-27-16-16-43.png)
- ![](pic/2022-03-27-16-17-26.png)
- ![](pic/2022-03-27-16-19-44.png)
- ![](pic/2022-03-27-16-24-16.png)
# 文件上传漏洞
- ![](pic/2022-03-27-16-35-15.png)
- ![](pic/2022-03-27-16-39-30.png)
  - 可以将文件上传漏洞与条件竞争漏洞相结合，从而在持续发起对设备的文件上传，设备中防护机制会对上传的文件进行检测删除，只要上传频率够高，则会有一个时刻成功上传恶意文件
- ![](pic/2022-03-27-16-43-45.png)
- ![](pic/2022-03-27-16-45-49.png)
- ![](pic/2022-03-27-16-48-23.png)
- ![](pic/2022-03-27-16-57-40.png)
# 文件上传绕过
- ![](pic/2022-03-27-17-02-46.png)
- ![](pic/2022-03-27-19-36-37.png)
- ![](pic/2022-03-27-19-37-19.png)
- ![](pic/2022-03-27-19-37-57.png)
- ![](pic/2022-03-27-19-41-50.png)
- ![](pic/2022-03-27-19-42-05.png)
- ![](pic/2022-03-27-19-42-16.png)
- ![](pic/2022-03-27-19-43-27.png)
- ![](pic/2022-03-27-19-55-01.png)
- ![](pic/2022-03-27-19-57-08.png)
- ![](pic/2022-03-27-19-58-47.png)
- ![](pic/2022-03-27-20-00-15.png)
- ![](pic/2022-03-27-20-01-30.png)
- ![](pic/2022-03-27-20-10-20.png)
- ![](pic/2022-03-27-20-10-49.png)
- ![](pic/2022-03-27-20-12-18.png)
- ![](pic/2022-03-27-20-12-45.png) 
- ![](pic/2022-03-27-20-13-01.png)
- ![](pic/2022-03-27-20-13-22.png)
- ![](pic/2022-03-27-20-14-40.png)
- ![](pic/2022-03-27-20-15-07.png)
- ![](pic/2022-03-27-20-15-53.png)
- ![](pic/2022-03-27-20-16-11.png)
- ![](pic/2022-03-27-20-19-03.png)
- ![](pic/2022-03-27-20-19-38.png)
- ![](pic/2022-03-27-20-20-17.png)
# 解析漏洞
- ![](pic/2022-04-01-19-54-59.png)
- ![](pic/2022-04-01-19-55-26.png)
- ![](pic/2022-04-01-19-57-34.png)
- ![](pic/2022-04-01-19-58-13.png)
- ![](pic/2022-04-01-19-59-17.png)
- ![](pic/2022-04-01-20-01-31.png)
- ![](pic/2022-04-01-20-02-21.png)
- ![](pic/2022-04-01-20-05-50.png)
- ![](pic/2022-04-01-20-06-37.png)
- ![](pic/2022-04-01-20-07-02.png)
- ![](pic/2022-04-01-20-07-11.png)
- ![](pic/2022-04-01-20-07-34.png)
- ![](pic/2022-04-01-20-08-38.png)
- ![](pic/2022-04-01-20-10-01.png)
- ![](pic/2022-04-01-20-10-29.png)
- ![](pic/2022-04-01-20-11-19.png)
- ![](pic/2022-04-01-20-12-07.png)
- ![](pic/2022-04-01-20-13-48.png)
# XML与DTD
- ![](pic/2022-04-01-20-35-29.png)
- ![](pic/2022-04-01-20-35-50.png)
- ![](pic/2022-04-01-20-39-07.png)
- ![](pic/2022-04-01-20-40-51.png)
- ![](pic/2022-04-01-20-42-12.png)
- ![](pic/2022-04-01-20-47-23.png)
- ![](pic/2022-04-01-20-50-51.png)
- ![](pic/2022-04-01-20-51-12.png)
- ![](pic/2022-04-01-20-53-51.png)
- ![](pic/2022-04-01-20-54-11.png)
- ![](pic/2022-04-01-20-56-37.png)
- ![](pic/2022-04-01-20-57-43.png)
# XPATH
- ![](pic/2022-04-01-20-59-04.png)
- ![](pic/2022-04-01-20-59-37.png)
- ![](pic/2022-04-01-21-00-05.png)
- ![](pic/2022-04-01-21-00-17.png)
- ![](pic/2022-04-01-21-00-50.png)
- ![](pic/2022-04-01-21-01-12.png)
- ![](pic/2022-04-01-21-02-16.png)
- ![](pic/2022-04-01-21-02-51.png)
- ![](pic/2022-04-01-21-03-29.png)
# XXE注入 xml外部实体注入 对于有回显的xxe，可以用于获取文件内容， 对于无回显的xxe，这时可以借助http协议，将获取到文件内容发送到远程服务器上，从而获取文件内容，最简单也最彻底的防御就是禁用外部实体
- ![](pic/2022-04-01-21-09-35.png)
- ![](pic/2022-04-01-21-10-15.png)
- ![](pic/2022-04-01-21-23-15.png)
- ![](pic/2022-04-01-21-25-20.png)
- ![](pic/2022-04-01-21-27-50.png)
- ![](pic/2022-04-01-21-28-30.png)
- ![](pic/2022-04-01-21-29-04.png)
- ![](pic/2022-04-01-21-30-27.png)
- ![](pic/2022-04-01-21-43-25.png)
- ![](pic/2022-04-01-21-43-50.png)
- ![](pic/2022-04-01-21-47-15.png)
- ![](pic/2022-04-01-21-48-23.png)
- 参考资料
  - https://www.kingkk.com/2018/07/%E7%AE%80%E6%9E%90XXE/
  - 本地备份：简析XXE.pdf
# XPATH注入
- ![](pic/2022-04-02-20-56-37.png)
- ![](pic/2022-04-02-20-57-33.png)
- ![](pic/2022-04-02-20-58-51.png)
- ![](pic/2022-04-02-20-59-03.png)
- ![](pic/2022-04-02-21-05-05.png)
- ![](pic/2022-04-02-21-05-47.png)
- ![](pic/2022-04-02-21-06-01.png)
# 逻辑漏洞
- ![](pic/2022-04-02-21-23-15.png)
- ![](pic/2022-04-02-21-27-57.png)
- ![](pic/2022-04-02-21-34-12.png)
- ![](pic/2022-04-02-21-43-26.png)
- ![](pic/2022-04-03-10-34-20.png)
- ![](pic/2022-04-03-10-44-23.png)
- ![](pic/2022-04-03-10-44-47.png)
- 当水平越权漏洞存在时，垂直越权漏洞大概率也存在，平行越权会伴随着垂直越权存在
- ![](pic/2022-04-03-10-52-33.png)
- ![](pic/2022-04-03-10-53-07.png)
- ![](pic/2022-04-03-10-55-27.png)
- ![](pic/2022-04-03-10-56-57.png)
- ![](pic/2022-04-03-11-00-38.png)
- ![](pic/2022-04-03-11-05-49.png)
- ![](pic/2022-04-03-11-06-54.png)
- ![](pic/2022-04-03-11-07-39.png)
- ![](pic/2022-04-03-11-49-37.png)
- 通过注册多个账号，拦截多个账号触发相同功能时的数据包，寻找两者不同的字段，可以找到请求中区分不同账户的关键字段
- ![](pic/2022-04-03-11-53-13.png)
- ![](pic/2022-04-03-11-58-54.png)
- ![](pic/2022-04-03-12-02-22.png)
- ![](pic/2022-04-03-12-15-38.png)
- ![](pic/2022-04-03-12-17-53.png)
- ![](pic/2022-04-03-12-22-16.png)
- ![](pic/2022-04-03-11-18-02.png)
- ![](pic/2022-04-03-11-18-18.png)
- ![](pic/2022-04-03-11-23-09.png)
- ![](pic/2022-04-03-11-24-35.png)
- ![](pic/2022-04-03-11-28-27.png)
- ![](pic/2022-04-03-11-35-02.png)
- ![](pic/2022-04-03-11-39-39.png)
- ![](pic/2022-04-03-11-43-33.png)
- ![](pic/2022-04-03-11-43-59.png)
- ![](pic/2022-04-03-11-44-29.png)  
- ![](pic/2022-04-03-11-44-48.png)
- ![](pic/2022-04-03-11-45-52.png)
- ![](pic/2022-04-03-11-46-35.png)
- ![](pic/2022-04-03-11-46-45.png)
- ![](pic/2022-04-03-12-45-22.png)
- ![](pic/2022-04-03-14-47-49.png)
- ![](pic/2022-04-03-14-47-58.png)
- ![](pic/2022-04-03-14-48-11.png)
- ![](pic/2022-04-03-14-53-28.png)
- ![](pic/2022-04-03-14-53-53.png)
- ![](pic/2022-04-03-14-54-13.png)
- ![](pic/2022-04-03-14-54-36.png)
- ![](pic/2022-04-03-14-57-01.png)
- ![](pic/2022-04-03-14-57-34.png)  
- ![](pic/2022-04-03-14-58-41.png)
- ![](pic/2022-04-03-14-59-19.png)
- ![](pic/2022-04-03-14-59-41.png)
- ![](pic/2022-04-03-14-59-51.png)
- ![](pic/2022-04-03-15-00-00.png)
- ![](pic/2022-04-03-15-00-40.png)
- ![](pic/2022-04-03-15-02-05.png)
- ![](pic/2022-04-03-15-02-45.png)
- ![](pic/2022-04-03-15-06-00.png)
- ![](pic/2022-04-03-15-07-27.png)
- ![](pic/2022-04-03-15-08-21.png)
- ![](pic/2022-04-03-15-08-31.png)
- ![](pic/2022-04-03-15-09-32.png)
- ![](pic/2022-04-03-15-10-03.png)
- ![](pic/2022-04-03-15-10-15.png)
- ![](pic/2022-04-03-15-11-35.png)
- ![](pic/2022-04-03-15-11-55.png)
- ![](pic/2022-04-03-15-12-31.png)
- ![](pic/2022-04-03-15-13-01.png)
- ![](pic/2022-04-03-15-13-22.png)
- ![](pic/2022-04-03-15-13-44.png)
