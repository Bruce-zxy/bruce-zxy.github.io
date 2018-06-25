## 目录
- [一、预备工作](#1)
	- [1.1 gcc 安装](#1.1)
	- [1.2 PCRE pcre-devel 安装](#1.2)
	- [1.3 zlib 安装](#1.3)
	- [1.4 OpenSSL 安装](#1.4)
- [二、下载、安装以及配置Nginx服务](#2)
	- [2.1 Nginx 下载](#2.1)
	- [2.2 Nginx 安装包解压](#2.2)
	- [2.3 Nginx 配置](#2.3)
	- [2.4 Nginx 编译安装](#2.4)
	- [2.5 Nginx 添加进环境变量](#2.5)
	- [2.6 Nginx 服务启动与停止](#2.6)
	- [2.7 Nginx 开机自启动](#2.7)
- [三、自签名证书配置HTTPS](#3)
	- [3.1 创建服务器证书密钥文件](#3.1)
	- [3.2 创建服务器证书的申请文件](#3.2)
	- [3.3 备份服务器密钥文件](#3.3)
	- [3.4 删除密钥文件保护密码](#3.4)
	- [3.5 生成证书文件](#3.5)
	- [3.6 配置Nginx的ssl模块](#3.6)
	- [3.7 修改本地hosts文件](#3.7)
- [四、Nginx 设置HTTP、HTTPS二级域名](#4)
	- [4.1 设置HTTP二级域名](#4.1)
	- [4.2 设置HTTP二级域名](#4.2)
	- [4.3 本地二级域名配置注意事项](#4.3)
- [五、Nginx 设置反向代理服务至二级域名下](#5)


## <span id="1">一、预备工作

**<span id="1.1">1.1 gcc 安装**

安装 nginx 需要先将官网下载的源码进行编译，编译依赖 gcc 环境，如果没有 gcc 环境，则需要安装：

    yum install gcc-c++

**<span id="1.2">1.2 PCRE pcre-devel 安装**

PCRE(Perl Compatible Regular Expressions) 是一个Perl库，包括 perl 兼容的正则表达式库。nginx 的 http 模块使用 pcre 来解析正则表达式，所以需要在 linux 上安装 pcre 库，pcre-devel 是使用 pcre 开发的一个二次开发库。nginx也需要此库。命令：

	yum install -y pcre pcre-devel

**<span id="1.3">1.3 zlib 安装**

zlib 库提供了很多种压缩和解压缩的方式， nginx 使用 zlib 对 http 包的内容进行 gzip ，所以需要在 Centos 上安装 zlib 库。

	yum install -y zlib zlib-devel

**<span id="1.4">1.4 OpenSSL 安装**

OpenSSL 是一个强大的安全套接字层密码库，囊括主要的密码算法、常用的密钥和证书封装管理功能及 SSL 协议，并提供丰富的应用程序供测试或其它目的使用。
nginx 不仅支持 http 协议，还支持 https（即在ssl协议上传输http），所以需要在 Centos 安装 OpenSSL 库。

    yum install -y openssl openssl-devel

## <span id="2">二、下载、安装以及配置Nginx服务

**<span id="2.1">2.1 Nginx 下载**

到[Nginx官网](https://nginx.org/en/download.html)寻找需要下载的tar.gz安装包的链接地址，

![Nginx官网下载列表](https://i.imgur.com/9MFw49G.png)

使用wget命令下载：

	wget -c https://nginx.org/download/nginx-1.10.3.tar.gz

**<span id="2.2">2.2 Nginx 安装包解压**

	tar -zxvf nginx-1.10.3.tar.gz
	cd nginx-1.10.3

**<span id="2.3">2.3 Nginx 配置**

在标准配置安装以外，我们需要开启 **http\_ssl\_module** 以及 **http\_gzip\_static\_module** 两个模块来开启 https 服务以及 encoding-gzip 传输

	./configure --prefix=/usr/local/nginx --with-http_ssl_module --with-http_gzip_static_module

**<span id="2.4">2.4 Nginx 编译安装**

	make
	make install

命令执行完成后会默认安装在 **/usr/local/nginx** 中

或者执行以下命令查找 Nginx 所在文件夹：

	whereis nginx

![whereis nginx](https://i.imgur.com/pzDPXGv.png)

**<span id="2.5">2.5 Nginx 添加进环境变量**

如果上一步提示 nginx 不是一个命令，那就在这一步将 nginx 添加进环境变量中：

	vim /etc/profile

在最后一行加上：

	export PATH=/usr/local/nginx/sbin:$PATH

![环境变量](https://i.imgur.com/YV1ZWi2.png)

此时再次执行nginx将不会报找不到命令了

**<span id="2.6">2.6 Nginx 服务启动与停止**

服务命令简介：

	nginx 				# 启动服务
	nginx -s quit		# 退出服务，此方式停止步骤是待nginx进程处理任务完毕进行停止
	nginx -s stop		# 停止服务，此方式相当于先查出nginx进程id再使用kill命令强制杀掉进程
	nginx -s reload		# 重载服务

服务**停止操作流程**：

	nginx				# 先开启服务
	nginx -s quit		# 停止服务前先退出服务
	nginx -s stop		# 再停止服务

服务**重启操作流程**：

	nginx				# 先开启服务
	nginx -s quit		# 重启服务前先退出服务
	nginx				# 最后重启服务

服务**重载操作流程**：

	nginx -s reload		# 当 nginx 的配置文件 nginx.conf 被修改后，要想让配置生效需要重启 nginx，使用-s reload不用先停止再启动即可将配置信息在 nginx 中生效

查询nginx进程：

	ps aux|grep nginx

**<span id="2.7">2.7 Nginx 开机自启动**

在 **/etc/rc.local** 中增加启动代码即可：

	vi /etc/rc.local

在文本末尾增加一行 **/usr/local/nginx/sbin/nginx**

![开机自启动](https://i.imgur.com/R8KxcmA.png)


> *参考链接：[CentOS 7 下安装 Nginx](https://www.linuxidc.com/Linux/2016-09/134907.htm)*

## <span id="3">三、自签名证书配置HTTPS

**<span id="3.1">3.1 创建服务器证书密钥文件**

	[root@CentOS ~]# cd /usr/local/nginx/conf && mkdir -p ssl && cd ssl
	[root@CentOS ssl]# openssl genrsa -des3 -out ca.key 1024

然后填写密钥保护密码：

	Enter PEM pass phrase:						# 输入密钥保护密码
	Verifying - Enter PEM pass phrase:			# 确认密钥保护密码

回车结束

**<span id="3.2">3.2 创建服务器证书的申请文件**

	[root@CentOS ssl]# openssl req -new -key ca.key -out ca.csr
	Enter pass phrase for root.key: 												# 输入前面创建的密码 
	Country Name (2 letter code) [AU]:CN 											# 国家代号，中国输入CN 
	State or Province Name (full name) [Some-State]:BeiJing 						# 省的全名，拼音 
	Locality Name (eg, city) []:BeiJing 											# 市的全名，拼音 
	Organization Name (eg, company) [Internet Widgits Pty Ltd]:MyCompany Corp. 		# 公司英文名 
	Organizational Unit Name (eg, section) []: 										# 可以不输入 
	Common Name (eg, YOUR name) []: 												# 此时不输入 
	Email Address []:admin@mycompany.com 											# 电子邮箱，可随意填
	Please enter the following ‘extra’ attributes 
	to be sent with your certificate request 
	A challenge password []: 														# 可以不输入 
	An optional company name []: 													# 可以不输入

**<span id="3.3">3.3 备份服务器密钥文件**

	[root@CentOS ssl]# cp ca.key ca.key.backup

**<span id="3.4">3.4 删除密钥文件保护密码**

	[root@CentOS ssl]# openssl rsa -in ca.key.backup -out ca.key

**<span id="3.5">3.5 生成证书文件**

	[root@CentOS ssl]# openssl x509 -req -days 365 -in ca.csr -signkey ca.key -out ca.crt

**<span id="3.6">3.6 配置Nginx的ssl模块</span>**

修改 Nginx 服务下的 nginx.conf

	[root@CentOS ~]# vi /usr/local/nginx/conf/nginx.conf

然后在 # HTTPS server 下将原有的 server 修改如下：

    server {
        listen       443 default ssl;
        server_name  zxy.com;

        ssl     on;

        ssl_certificate      /home/yg-zxy/ssl-file/ssl-30-server.crt;
        ssl_certificate_key  /home/yg-zxy/ssl-file/ssl-30-server.key;

        location / {
            root   html;
            index  index.html index.htm;
        }
    }

***Tips：***

**listen:** 设置ssl监听端口，默认设置443即可，default可以省略

**server_name:** 设置服务器域名，这里配合接下来介绍到的修改hosts文件可以做到通过域名访问，域名随意设置，比如 **zxy.com**

**ssl:** 开启ssl模块

**ssl_certificate:** 证书、公钥，需要发送到客户端的

**ssl_certificate_key:** 私钥

**location:** 服务器路由匹配配置

**<span id="3.7">3.7 修改本地hosts文件</span>**

ssl证书设置好了，但是想要验证https能不能访问则得需要域名才行。

当然我们不一定真的要去买一个域名来验证，可以通过修改本地hosts文件的方式来拦截、修改本地的域名解析过程：

**Windows平台：**

	attrib -R C:\WINDOWS\system32\drivers\etc\hosts
	@echo 127.0.0.1 zxy.com >>C:\WINDOWS\system32\drivers\etc\hosts
	@echo 127.0.0.1 www.zxy.com >>C:\WINDOWS\system32\drivers\etc\hosts

在本地新建一个文本文档，然后将这两行代码复制到文本文档中，另存为 **changeHosts.bat**

然后双击（可能需要右击管理员权限运行）执行 **changeHosts.bat** 文件

等到CMD窗口执行完成以后，即可在浏览器访问 [https://zxy.com](https://zxy.com "https://zxy.com")

**Linux平台：**

*root用户：*

	[root@CentOS ~]# vi /etc/hosts

*非root用户：*

	[user@CentOS ~]# sudo vi /etc/hosts
	[sudo] password for user:					# 输入当前用户的密码

然后在文末添加一行： **127.0.0.1 zxy.com**

	127.0.0.1   localhost localhost.localdomain localhost4 localhost4.localdomain4
	::1         localhost localhost.localdomain localhost6 localhost6.localdomain6

	127.0.0.1   zxy.com
	127.0.0.1	www.zxy.com

保存然后退出

> **Tips: 修改host的操作一定要在访问的主机上进行，而不是被访问的服务器上进行**

**在本地（浏览器）访问 [https://zxy.com](https://zxy.com "https://zxy.com") 时会遇到如下问题：**

![证书不安全](https://i.imgur.com/vd0x2Zx.png)

提示你访问的地址不安全，这是因为我们的ssl证书是我们自己创建的，并未经过任何可信的第三方认证。所以浏览器会告诉用户，这个证书是未经可信第三方认证的证书，会有风险。但是在正式的公网环境中，我们肯定是需要取得可信第三方认证的DV或者EV证书的

我们点击“高级”，继续点击“[继续前往zxy.com（不安全）](https://zxy.com)”即可

![继续访问](https://i.imgur.com/WKPr9kT.png)

> *参考链接：[Nginx配置 HTTPS 证书网站](https://www.cnblogs.com/eaglezb/p/6074811.html)*

## <span id="4">四、Nginx 设置HTTP、HTTPS二级域名

**<span id="4">4.1 设置HTTP二级域名**

修改 Nginx 服务下的 nginx.conf

	[root@CentOS ~]# vi /usr/local/nginx/conf/nginx.conf

将http中server的内容：

	server {
    	listen 80;
    	server_name zxy.com;

    	location / {
            root   html;
            index  index.html index.htm;
        }
	}

修改为二级域名正则通配形式：

	server {
    	listen 80;
    	server_name zxy.com;

		if ( $host ~* (\b(?!www\b)\w+)\.\w+\.\w+ ) {
			set $subdomain /$1;
		}
		location / {
			root html$subdomain;
			index index.html index.htm;
		}
	}

以这种方式配置二级域名的好处就是不需要再单独配置其他二级域名了，只需要在 **/usr/local/nginx/html** 下创建响应的文件夹即可访问相应的二级域名。

检测 Nginx 服务配置是否正确：

	[root@CentOS ~]# nginx -t
	nginx: the configuration file /usr/local/nginx/conf/nginx.conf syntax is ok
	nginx: configuration file /usr/local/nginx/conf/nginx.conf test is successful

然后重载 Nginx 服务：

	[root@CentOS ~]# nginx -s reload

创建二级域名所属的文件夹（需要root权限）：

	[root@CentOS ~]# cd /usr/local/nginx/html && mkdir -p test && cd test
	[root@CentOS test]# vi index.html

给首页文件输入内容：

	Hello! This is a test page!

保存并退出

这时访问 [http://test.zxy.com](http://test.zxy.com "http://test.zxy.com") 即可看到刚刚编写的内容

![访问test二级域名](https://i.imgur.com/MfzspQ3.png)

如果我们需要配置另一个二级域名 [http://second.zxy.com](http://second.zxy.co "http://second.zxy.com") 的话，可以直接在 html 文件夹中创建相应的二级域名所属文件夹：

	[root@CentOS test]# cd /usr/local/nginx/html && mkdir -p second && cd second
	[root@CentOS second]# vi index.html
	Hello! This is second subdomain!

保存并退出

这时访问 [http://second.zxy.com](http://second.zxy.com "http://second.zxy.com") 即可看到新域名下的内容

> *如有问题请参考：**[4.3 本地二级域名配置注意事项](#4.3)**

**<span id="4.2">4.2 设置HTTP二级域名</span>**

修改 Nginx 服务下的 nginx.conf

	[root@CentOS ~]# vi /usr/local/nginx/conf/nginx.conf

将我们在 [3.6 配置Nginx的ssl模块](#3.6) 中配置的内容修改为如下配置：

    server {
        listen       443 default ssl;
        server_name  zxy.com;

        ssl     on;

        ssl_certificate      /home/yg-zxy/ssl-file/ssl-30-server.crt;
        ssl_certificate_key  /home/yg-zxy/ssl-file/ssl-30-server.key;

        if ( $host ~* (\b(?!www\b)\w+)\.\w+\.\w+ ) {
            set $subdomain $1;
        }

        location / {
            root   html/$subdomain;
            index  index.html index.htm;
        }
    }

保存并退出

这时访问 [https://test.zxy.com](https://test.zxy.com "https://test.zxy.com") 以及 [https://second.zxy.com](https://second.zxy.com "https://second.zxy.com") 即可看到HTTPS域名下的内容

> *如有问题请参考：**[4.3 本地二级域名配置注意事项](#4.3)**

**<span id="4.3">4.3 本地二级域名配置注意事项</span>**

***注意！！！***

**有可能在访问 [http://test.zxy.com](http://test.zxy.com "http://test.zxy.com") 以及 [http://second.zxy.com](http://second.zxy.com "http://second.zxy.com") 时地址可能会被302重定向到 [http:/zxy.com](http://zxy.com "http://zxy.com") ，这时需要将二级域名的地址解析加进 hosts 文件中：**

	attrib -R C:\WINDOWS\system32\drivers\etc\hosts
	@echo 127.0.0.1 test.zxy.com >>C:\WINDOWS\system32\drivers\etc\hosts
	@echo 127.0.0.1 second.zxy.com >>C:\WINDOWS\system32\drivers\etc\hosts

具体操作请回到 **[3.7 修改本地hosts文件](#3.7)**

在往hosts文件中添加完二级域名地址解析后，关闭浏览器，重新打开访问二级域名即可。

> *参考链接：[配置服务器 —— Nginx添加多个二级子域名](https://blog.csdn.net/LBinin/article/details/70188752)*

## <span id="5">五、Nginx 设置反向代理服务至二级域名下

如果我们在某个端口运行了一个Web应用程序，比如我们在5000端口下有一个静态Web服务器：

![某个静态服务器](https://i.imgur.com/LXzkIML.png)

那我们可能在实际需求中不被允许通过端口访问这个Web应用程序，比如在微信小程序的开发中，微信小程序不允许接口的地址带有端口号，那么我们就需要将这个应用服务通过 Nginx 的代理模块来将对 Nginx 的请求转发到该内部端口上。

首先，修改 Nginx 服务下的 nginx.conf

	[root@CentOS ~]# vi /usr/local/nginx/conf/nginx.conf

然后在我们在 [3.6 配置Nginx的ssl模块](#3.6) 配置的内容之前添加如下配置（一定要在之前）：

    server {

        listen          443;
        server_name     porxy.zxy.com;

        location / {
                proxy_pass http://127.0.0.1:5000;
                proxy_http_version 1.1;
                proxy_set_header Upgrade $http_upgrade;
                proxy_set_header Connection 'upgrade';
                proxy_set_header Host $host;
                proxy_cache_bypass $http_upgrade;
        }

    }

这时访问 [https://proxy.zxy.com](https://proxy.zxy.com "https://proxy.zxy.com") 即可看到 Nginx 代理5000端口下的内容

> *参考链接：[nginx配置nodejs服务二级域名](https://blog.csdn.net/tujiaw/article/details/70176312?locationNum=10&fps=1)*