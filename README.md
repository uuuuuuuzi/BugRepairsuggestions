# 漏洞修复建议大全

> 早先一直靠老版本的Word文档苟延残喘，然后公司内网平台有了以后用公司的，怎奈太久没更新了好多不适用，加之人懒，VPN都不想登，便开设此项目。
>
> 该项目持续更新


[TOC]
## SQL注入

修改Web应用服务的软件部分，增加对客户端提交数据的合法性验证，至少严格过滤SQL语句中的关键字，并且所有验证都应该在服务器端实现，以防客户端（IE页面代码部分）控制被绕过。

验证GET、POST、COOKIE等方法中URL后面跟的参数，需过滤的关键字为：

[1] ' 单引号

[2] " 双引号

[3] ' 反斜杠单引号

[4] " 反斜杠双引号

[5] ) 括号

[6] ; 分号

[7] -- 双减号

[8] + 加号

[9]SQL关键字，如select，delete，drop等等，注意对于关键字要对大小写都识别，如:select ;SELECT;seLEcT等都应识别

建议降低Web应用访问使用较低权限的用户访问数据库。不要使用数据库管理员等高权限的用户访问数据库。

## 弱口令

1、用户登录后需要对初始口令进行修改，防止攻击者利用初始口令进行暴力破解。

2、系统设置强密码策略，建议用户密码采用10位以上数字加大小写字母。

3、对密码暴力猜解行为进行图灵验证，一旦发现用户口令破解行为及时对账户进行限时封停处理。

## 未授权访问

1、对于每个功能的访问，需要明确授予特定角色的访问权限。

2、如果某功能参与了工作流程，检查并确保当前的条件是授权访问此功能的合适状态。

## XSS跨站脚本攻击

建议过滤的关键字为：

[1] ' 单引号

[2] " 双引号

[3] / 斜杠

[4] \ 反斜杠

[5] ) 括号

[6] ; 分号

[7] [ 中括号

[8] < 尖括号

[9] > 尖括号

比如把<编码为<

在cookie中加入httponly属性可以在一定程度上保护用户的cookie，减少出现XSS时损失。

## 目录列表

1、对于apache服务器，在apache配置文件httpd.conf中，从Options中删除Indexes。

2、对于IIS服务器，禁用目录浏览属性。

## CVE-2019-0708远程代码执行

1、及时打对应系统的安全补丁

2、关闭3389端口或添加防火墙安全策略限制对3389端口的访问

3、打不了补丁的可以开启远程桌面（网络级别身份验证(NLA)），可以临时防止漏洞攻击

## Apache Tomcat文件包含(CNVD-2020-10487/CVE-2020-1938)

目前，Apache 官方已发布 9.0.31、8.5.51 及 7.0.100 版本对此漏洞进行修 复，建议用户尽快升级新版本：

https://tomcat.apache.org/download-70.cgi

https://tomcat.apache.org/download-80.cgi

https://tomcat.apache.org/download-90.cgi

如果暂时无法升级最新版本，可采取临时缓解措施：

1、如未使用 Tomcat AJP 协议：

如未使用 Tomcat AJP 协议，可以直接将 Tomcat 升级到 9.0.31、8.5.51 或 7.0.100 版本进行漏洞修复。

如无法立即进行版本更新、或者是更老版本的用户，建议直接关闭 AJPCon nector，或将其监听地址改为仅监听本机 localhost。

具体操作：

（1）编辑 Code: [["", [], []], "<CATALINA_BASE>"]/conf/server.xml，找到如下行（Code: [["", [], []], "<CATALINA_B ASE>"]为 Tomcat 的工作目录）：

<Connector port="8009"protocol="AJP/1.3" redirectPort="8443" /> 

（2）将此行注释掉（也可删掉该行）：

<!--<Connectorport="8009" protocol="AJP/1.3"redirectPort="8443" />--> 

（3）保存后需重新启动，规则方可生效。

2、如果使用了 Tomcat AJP 协议：

建议将 Tomcat 立即升级到 9.0.31、8.5.51 或 7.0.100 版本进行修复，同时 为 AJP Connector 配置 secret 来设置 AJP 协议的认证凭证。例如（注意必须将 Y OUR_TOMCAT_AJP_SECRET 更改为一个安全性高、无法被轻易猜解的值）：

\```

<Connector port="8009"protocol="AJP/1.3" redirectPort="8443"address="YO UR_TOMCAT_IP_ADDRESS" secret="YOUR_TOMCAT_AJP_SECRET"/>

\```

如无法立即进行版本更新、或者是更老版本的用户，建议为 AJPConnector 配置 requiredSecret 来设置 AJP 协议认证凭证。例如（注意必须将 YOUR_TOMC AT_AJP_SECRET 更改为一个安全性高、无法被轻易猜解的值）：

\```<Connector port="8009"protocol="AJP/1.3" redirectPort="8443"address="YO UR_TOMCAT_IP_ADDRESS"requiredSecret="YOUR_TOMCAT_AJP_SECRET" />

\```

## 信息泄露

1、对身份验证时传输的用户名密码等作加密处理，为了防止重放攻击可以在验证时加个随机数，以保证验证单次有效。

2、关闭错误输出，防止调试信息泄露，或者当web应用程序出错时，统一返回一个错误页面或直接跳转至首页

3、合理设置服务器端各种文件的访问权限

4、尽量避免跨域的数据传输，对于同域的数据传输使用xmlhttp的方式作为数据获取的方式，依赖于javascript在浏览器域里的安全性保护数据。如果是跨域的数据传输，必须要对敏感的数据获取做权限认证，例如对referer的来源做限制，加入token等

## 任意文件上传

处理用户上传文件，要做以下检查：

1、 检查上传文件扩展名白名单，不属于白名单内，不允许上传。

2、 上传文件的目录必须是http请求无法直接访问到的。如果需要访问的，必须上传到其他（和web服务器不同的）域名下，并设置该目录为不解析jsp等脚本语言的目录。

3、 上传文件要保存的文件名和目录名由系统根据时间生成，不允许用户自定义。

4、 图片上传，要通过处理（缩略图、水印等），无异常后才能保存到服务器。

## 任意文件下载或读取

1、对相应参数进行过滤，依次过滤“.”、“..”、“/”、“”等字符。

2、或者对于下载文件的目录做好限制，只能下载指定目录下的文件，或者将要下载的资源文件路径存入数据库，附件下载时指定数据库中的id即可，id即对应的资源。

## 命令执行注入

1、尽量避免使用exec、system、passthru、popen、shell_exec、eval、execute、assert等此类执行命令/执行代码相关函数。

2、执行代码/命令的参数，或文件名，不要使用外部获取，禁止和用户输入相关，只能由开发人员定义代码内容，用户只能提交“1、2、3”参数，代表相应代码，防止用户构造。

## 短信炸弹

1、限制每个手机号的每日发送次数，超过次数则拒发送，提示超过当日次数。

2、每个ip限制最大限制次数。超过次数则拒发送，提示超过ip当日发送最大次数。

3、限制每个手机号发送的时间间隔，比如两分钟，没超过2分钟，不允许发送，提示操作频繁。

4、应具有页面图片验证码、IP访问次数限制功能!

## 垂直越权访问

1、对于每个功能的访问，需要明确授予特定角色的访问权限。

2、如果某功能参与了工作流程，检查并确保当前的条件是授权访问此功能的合适状态。

## CSRF 跨站请求伪造

1、验证 HTTP Referer 字段

2、在请求地址中增加 csrftoken 验证，csrftoken 可以在用户登录后产生并放于 session 之中，然后在每次请求时把 csrftoken 从 session 中拿出，与请求中的 csrftoken 进行比对。对于 GET 请求，csrftoken 将附在请求地址之后，对于 POST 请求来说，要在 form 的最后加上 `<input type="hidden" name="csrftoken" value="tokenvalue"/>；`

3、在 HTTP 头中自定义属性并验证。

4、对于 web 站点，将持久化的授权方法（例如 cookie 或者 HTTP 授权）切换为瞬时的授权方法（在每个 form 中提供隐藏 field），可以帮助网站防止 CSRF 攻击。

5、过滤用户输入，不允许发布含有站内操作 URL 的链接；

6、在浏览其它站点前登出站点或者在浏览器会话结束后清理浏览器的 cookie。

## CORS 跨域漏洞

1. 正确配置跨域请求, 如果 Web 资源包含敏感信息，则应在 Access-Control-Allow-Origin 标头中正确指定来源。
2. 只允许信任的网站, Access-Control-Allow-Origin 中指定的来源只能是受信任的站点。使用通配符来表示允许的跨域请求的来源而不进行验证很容易被利用，应该避免。
3. 避免将 null 列入白名单, 避免使用标题 Access-Control-Allow-Origin: null。来自内部文档和沙盒请求的跨域资源调用可以指定 null 来源。应针对私有和公共服务器的可信来源正确定义 CORS 头。
4. 避免在内部网络中使用通配符。当内部浏览器可以访问不受信任的外部域时，仅靠信任网络配置来保护内部资源是不够的。
5. CORS 不能替代服务器端安全策略, CORS 定义了浏览器的行为，绝不能替代服务器端对敏感数据的保护 - 攻击者可以直接从任何可信来源伪造请求。因此，除了正确配置的 CORS 之外，Web 服务器还应继续对敏感数据应用保护，例如身份验证和会话管理。

## SlowHttp 缓慢攻击漏洞

1. 设置合适的 timeout 时间（Apache 已默认启用了 reqtimeout 模块），规定了 Header 发送的时间以及频率和 Body 发送的时间以及频率。
2. 增大 MaxClients(MaxRequestWorkers)：增加最大的连接数。根据官方文档，两个参数是一回事，版本不同，MaxRequestWorkers was called MaxClients before version 2.3.13.Theold name is still supported。
3. 默认安装的 Apache 存在 Slow Attack 的威胁，原因就是虽然设置的 timeoute，但是最大连接数不够，如果攻击的请求频率足够大，仍然会占满Apache的所有连接。
4. 将标题和消息体限制在最小的合理长度上。针对接受数据的每个资源，设置更严格的特定于 URL 的限制。
5. 如果 Web 服务器从相同的 IP 接收到数千个连接，同一个用户代理在短时间内请求相同的资源，直接禁掉 IP 并且记录日志。
6. 对 web 服务器的 http 头部传输的最大许可时间进行限制，修改成最大许可时间为 20 秒。

- Weblogic

  1. 在配置管理界面中的协议 -> 一般信息下设置 完成消息超时时间小于 400。
  2. 在配置管理界面中的协议 ->HTTP 下设置 POST 超时、持续时间、最大 POST 大小为安全值范围。

- Nginx

  1. 通过调整 $request_method，配置服务器接受 http 包的操作限制；
  2. 在保证业务不受影响的前提下，调整 client_max_body_size, client_body_buffer_size, client_header_buffer_size,large_client_header_buffersclient_body_timeout, client_header_timeout 的值，必要时可以适当的增加；
  3. 对于会话或者相同的 ip 地址，可以使用 HttpLimitReqModule and HttpLimitZoneModule 参数去限制请求量或者并发连接数；
  4. 根据 CPU 和负载的大小，来配置 worker_processes 和 worker_connections 的值，公式是：max_clients = worker_processes * worker_connections。

- Apache

  1. mod_reqtimeout 用于控制每个连接上请求发送的速率。配置例如：
     请求正文部分，设置超时时间初始为 10 秒，将超时时间延长 1 秒，但最长不超过 40 秒。可以防护 slow message body 型的慢速攻击。

  2. mod_qos 用于控制并发连接数。配置例如：

     当服务器并发连接数超过 600 时，关闭 keepalive
     QS_SrvMaxConnClose 600
     每个源 IP 最大并发连接数为 50
     QS_SrvMaxConnPerIP 50

## log4j2(CVE-2021-44228)远程代码执行漏洞

1、设置系统环境变量 LOG4J_log4j2_formatMsgNoLookups=True

2、升级 Apache Log4j2 相关应用到最新版本，地址 https://github.com/apache/logging-log4j2/tags

## Struts2 反序列化漏洞

1、请及时安装 Struts2 中间件的最新版本 http://struts.apache.org/download.cgi

## iis 短文件名泄露漏洞

修改 Windows 配置，关闭短文件名功能。

1. 关闭 NTFS 8.3 文件格式的支持。该功能默认是开启的，对于大多数用户来说无需开启。
2. 如果是虚拟主机空间用户, 可采用以下修复方案：
   - 修改注册列表 HKLM\SYSTEM\CurrentControlSet\Control\FileSystem\NtfsDisable8dot3NameCreation 的值为 1(此修改只能禁止 NTFS8.3 格式文件名创建, 已经存在的文件的短文件名无法移除)。
   - 如果你的 web 环境不需要 asp.net 的支持你可以进入 Internet 信息服务 (IIS) 管理器 --- Web 服务扩展 - ASP.NET 选择禁止此功能。
   - 升级 net framework 至 4.0 以上版本。
   - 将 web 文件夹的内容拷贝到另一个位置，比如 D:\www 到 D:\www.back，然后删除原文件夹 D:\www，再重命名 D:\www.back 到 D:\www。如果不重新复制，已经存在的短文件名则是不会消失的。

## Elasticsearch 未授权访问漏洞

1、限制 IP 访问，绑定固定 IP。

2、在 config/elasticsearch.yml 中为9200端口设置认证。

## Kibana 未授权访问漏洞

1、设置防火墙策略，仅允许指定的 IP 来访问服务。

2、通过 nginx 设置反向代理，设置密码登录验证。

3、设置 kibana 监听本地地址，并设置 ElasticSearch 登录的账号和密码。

4、使用 htpasswd 创建 kibana 登录的账号密码，这里可以复用 ElasticSearch 创建的账号密码，也可以重新创建一 个。

5、配置 nginx 反向代理，配置好后，重启 nginx 和 kibana，通过 15601 登录验证访问 Kibana。

## Shiro 反序列化

1、对于 shiro 应用,管理员需及时关注官方的安全公告 https://issues.apache.org/jira/projects/SHIRO/issues

## Druid Monitor 未授权访问

做好权限的控制，不允许未登录的用户直接访问 Druid Monitor 页面。

## Spring Actuator 组件未授权访问

1、禁用所有接口，将配置改为: endpoints.enable=false

2、引入 spring-boot-starter-security 依赖

```
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-security</artifactId>
</dependency>
```

3、在application.properties中指定actuator的端口以及开启security功能，配置访问权限验证，这时再访问actuator功能时就会弹出登录窗口，需要输入账号密码验证后才允许访问。

```
management.port=8099
management.security.enabled=true
security.user.name=admin
security.user.password=admin
```

## solr 未授权访问漏洞

1、添加 Apache Solr 访问权限控制。

2、禁止外网或白名单以外的 IP 访问 solr/admin 目录。

## Redis 未授权访问漏洞

1、禁用一些高危命令,常见如：flushdb，flushall，config，keys 等。

2、禁止使用 root 权限启动 redis 服务，以低权限运行 Redis 服务。

3、对 redis 访问启动密码认证。

4、禁止外网访问 Redis。

5、添加 IP 访问限制，并更改默认 6379 端口。

6、保证 authorized_keys 文件的安全。

## Mongodb 未授权访问漏洞

1、为 MongoDB 添加认证：MongoDB 启动时添加 --auth 参数、为 MongoDB 添加用户。

2、MongoDB 自身带有一个 HTTP 服务和并支持 REST 接口。在2.6以后这些接口默认是关闭的。mongoDB 默认会使用默认端口监听 web 服务，一般不需要通过web方式进行远程管理，建议禁用。修改配置文件或在启动的时候选择 -nohttpinterface 参数 nohttpinterface=false。

3、启动时加入参数 --bind_ip 127.0.0.1 或在 /etc/mongodb.conf 文件中添加以下内容：bind_ip = 127.0.0.1。

## Memcahce 未授权访问

1. 配置 memcached 监听本地回环地址 127.0.0.1。
    vim /etc/sysconfig/memcached
    OPTIONS="-l 127.0.0.1"  # 设置本地为监听
    /etc/init.d/memcached restart   # 重启服务

2. 当 memcached 配置为监听内网 IP 或公网 IP 时， 使用主机防火墙(iptalbes、 firewalld 等)和 网络防火墙对 memcached 服务端口 进行过滤。

## VNC 未授权

1、设置 VNC 服务的密码。

2、限制白名单 IP 连接到对应的 VNC 端口。

## SMB弱口令

1. 不要使用常见的弱口令作为密码。
2. 定期修改密码。
3. 及时 SMB 服务更新到最新版本。
4. 对采用白名单的方式，仅允许授权主机 IP 访问该端口，避免漏洞被攻击者恶意利用。

## Hadoop 未授权访问漏洞

1. 如无必要，关闭 Hadoop Web 管理页面。
2. 开启身份验证，防止未经授权用户访问。
3. 设置“安全组”访问控制策略，将 Hadoop 默认开放的多个端口对公网全部禁止或限制可信任的 IP 地址才能访问包括 50070 以及 WebUI 等相关端口。

## ZooKeeper 未授权访问漏洞

1. 修改 ZooKeeper 默认端口，采用其他端口服务。
2. 添加访问控制，配置服务来源地址限制策略。
3. 增加 ZooKeeper 的认证配置。

## Docker Remote API 未授权访问漏洞

1. 修改 Docker Remote API 服务默认参数。
2. 修改 Docker 的启动参数：定位到 DOCKER_OPTS 中的 tcp://0.0.0.0:2375，将0.0.0.0修改为127.0.0.1 或将默认端口 2375 改为自定义端口为 Remote API 设置认证措施。
3. 设置防火墙策略。如果正常业务中 API 服务需要被其他服务器来访问，可以配置安全组策略或 iptables 策略，仅允许指定的 IP 来访问 Docker 接口。
4. 修改 Docker 服务运行账号。请以较低权限账号运行 Docker 服务；另外，可以限制攻击者执行高危命令。

总之、不要将端口直接暴露在公网，内网中使用需要设置严格的访问规则.

## Kubelet 未授权访问

1. 将 Kubelet 组件的启动参数 --anonymous-auth 值设为 false，即不允许匿名访问
2. 将 Kubelet 组件的启动参数 --authorization-mode 值设为 Webhook
3. 如非业务需要，可以关闭 Web 端口的服务，保持最小化原则，避免安全风险的产生。

## Kubernetes API Server 未授权访问

1、禁止在 Kubernetes APIServer 组件的配置文件中修改 --insecure-port 启动参数值为 8080，使用默认配置值

2、禁止在 Kubernetes APIServer 组件的配置文件中修改 --insecure-bind-address 启动参数值为 0.0.0.0，使用默认配置值

3、使用 API Server 的安全端口（6443），并为其设置证书

## MS17-010

针对 MS17-010 漏洞，微软官方已放出相应的补丁，建议及时安装官方补丁:https://docs.microsoft.com/zh-cn/security-updates/securitybulletins/2017/ms17-010

## CVE-2019-0708

针对 CVE-2019-0708 漏洞，Microsoft 已经为此发布了一个安全公告以及相应补丁:
https://portal.msrc.microsoft.com/zh-CN/security-guidance/advisory/CVE-2019-0708
https://support.microsoft.com/en-us/help/4500331/windows-update-kb4500331

## FTP 弱口令

1. 不要使用常见的弱口令作为密码。
2. 定期修改密码。
3. 及时 FTP 服务更新到最新版本。
4. 对采用白名单的方式，仅允许授权主机 IP 访问该端口，避免漏洞被攻击者恶意利用。

## FTP匿名登录

1. 关闭匿名访问。
2. 不要使用常见的弱口令作为密码。
3. 定期修改密码。
4. 及时 ftp 服务更新到最新版本。
5. 采用白名单的方式，仅允许授权主机 IP 访问该端口，避免漏洞被攻击者恶意利用。

## SSH 弱口令

1. 修改口令，增加口令复杂度，如包含大小写字母、数字和特殊字符等。
2. 修改默认口令，避免默认口令被猜解。
3. 指定健壮的口令策略，比如指定每隔30天修改一次密码，密码不得与历史密码相同。
4. 采用白名单的方式，仅允许授权主机 IP 访问该端口，避免漏洞被攻击者恶意利用。

## 数据库弱口令

1. 修改口令，增加口令复杂度，如包含大小写字母、数字和特殊字符等。
2. 修改默认口令，避免默认口令被猜解。
3. 指定健壮的口令策略，比如指定每隔30天修改一次密码，密码不得与历史密码相同。
4. 采用白名单的方式，仅允许授权主机 IP 访问该端口，避免漏洞被攻击者恶意利用。

## 任意文件包含

1. 配置文件：在配置文件中限制访问的文件目录，比如 PHP 中 php.ini 配置 open_basedir。
2. 特殊字符过滤：检查用户输入，过滤或转义含有“../”、“..\”、“%00”，“..”，“./”，“#”等跳转目录或字符终止符、截断字符的输入。
3. 合法性判断：严格过滤用户输入字符的合法性，比如文件类型、文件地址、文件内容等。
4. 白名单：白名单限定访问文件的路径、名称及后缀名。

## svn 源码泄露漏洞

1、查找服务器上所有 .svn 隐藏文件夹，删除。

2、开发人员在使用 SVN 时，严格使用导出功能。禁止直接复制代码。

## **.git 源码泄漏**

1. 查找服务器上所有 .git 隐藏文件夹，删除。
2. 中间件上设置 .git 目录访问权限，禁止访问。

## DS_Store 信息泄漏

1. 在不影响代码运行的情况下，删除 .DS_Store 文件。
2. 在中间件中设置对应的文件、目录的访问权限。

## Swagger API 泄露

1. 对该页面加上授权访问检查。
2. 鉴权，服务端对请求的数据和当前用户身份做校验;完善基础安全架构，完善用户权限体系。
3. 对于后台接口，确保所有 API 接口先经过登录控制器。
4. 在验证用户身份权限前不进行任何数据的交互。

## XXE

- 使用语言中推荐的禁用外部实体的方法
  - PHP：libxml_disable_entity_loader(true);
  - Java:
    - DocumentBuilderFactory dbf =DocumentBuilderFactory.newInstance();
    - dbf.setExpandEntityReferences(false);
  - Python:
    - from lxml import etree
    - xmlData = etree.parse(xmlSource,etree.XMLParser(resolve_entities=False))
- 手动黑名单过滤
  - 过滤 XML 中的相关关键词，比如：<!DOCTYPE、<!ENTITY SYSTEM、PUBLIC 等。

## SSRF服务器请求伪造

1. 禁用不需要的协议，只允许 HTTP 和 HTTPS 请求，可以防止类似于 file://, gopher://, ftp:// 等引起的问题。
2. 白名单的方式限制访问的目标地址，禁止对内网发起请求。
3. 过滤或屏蔽请求返回的详细信息，验证远程服务器对请求的响应是比较容易的方法。如果 web 应用是去获取某一种类型的文件。那么在把返回结果展示给用户之前先验证返回的信息是否符合标准。
4. 验证请求的文件格式,禁止跟随 301、302 跳转。
5. 限制请求的端口为 http 常用的端口，比如 80、443、8080、8000 等。
6. 统一错误信息，避免用户可以根据错误信息来判断远端服务器的端口状态。

- 解析目标URL
  - 获取scheme、host（推荐使用系统内置函数完成,避免自己使用正则提取）。
- 校验scheme
  - 检查 scheme 是否为合法 (如非特殊需求请只允许 http 和 https)。
- 获取解析IP
  - 解析 host 获取 dns 解析后的 IP 地址。
- 检测IP是否合法
  - 检查解析后的 IP 地址是否为外网地址或者合法 IP。

## Ghostcat(CVE-2020-1938)漏洞

针对 CVE-2020-1938 漏洞，Apache Tomcat 官方已放出相应的修复新版本，建议及时安装官方新版本。