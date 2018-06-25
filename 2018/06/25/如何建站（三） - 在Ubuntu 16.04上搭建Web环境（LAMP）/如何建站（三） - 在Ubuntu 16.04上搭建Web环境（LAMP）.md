---
title: 如何建站（三） - 在Ubuntu 16.04上搭建Web环境（LAMP）
---

*接下来我们开始在服务器上搭建Web环境*

在Linux上Web环境主要境分为LAMP和LNMP两种：
- LAMP 是指Linux + Apache + Mysql +PHP
- LNMP 是指Linux + Nginx + Mysql + PHP （如果要在StackOverflow等外文社区搜索相关资料请用LEMP）

两者的区别在于Apache是重量级的，且对PHP友好的。如果你的后台处理语言更多的是使用PHP，那么就选择LAMP是最好的。而Nginx是轻量级的，对静态资源友好的。如果你的网站更多的只是静态资源，那么选择LNMP绝对没错。

两者环境的搭建大同小异，接下来本文主要介绍LAMP的搭建。

##### 更新Ubuntu软件库

在搭建LAMP环境前，我们为了保险起见，先更新一下软件库：

<pre>zxy@zxy-MIBook:~$ sudo apt update</pre>

##### 安装Apache2

也可以安装Apache最原始的版本，但是本文建议安装Apache2，包括这步操作之后的Mysql以及PHP相关依赖都是基于Apache2的。所以**如果你非要安装Apache原始版本的话，本文不能保证安装的顺利**。

<pre>zxy@zxy-MIBook:~$ sudo apt install apache2</pre>

##### 验证

在浏览器地址栏中输入localhost，如果打开显示了Apache2，那么这一步配置成功！

##### 安装PHP

这里安装的PHP的版本是7.0，因为Ubuntu 16.04默认支持PHP7.0，如果安装PHP5就会报错。
*不信。。。你可以试一试。。。*

<pre>zxy@zxy-MIBook:~$ sudo apt install php 
zxy@zxy-MIBook:~$ sudo apt install libapache2-mod-php </pre>

##### 验证

<pre>zxy@zxy-MIBook:~$ cd /var/www/html
zxy@zxy-MIBook:~$ sudo vim info.php</pre>

在info.php中输入：
<pre><?php 
phpinfo();
 ?></pre>

然后保存文件（:wq!）退出
再在浏览器地址栏里输入localhost/info.php，如果出现PHP的配置信息，那么这一步配置成功！

##### 安装MySQL

<pre>zxy@zxy-MIBook:~$ sudo apt install mysql-server php7.0-mysql
zxy@zxy-MIBook:~$ sudo apt-get install mysql-client
      // <!--  /*  这是注释：这里会让你设置Mysql的密码  */ --> 

zxy@zxy-MIBook:~$ mysql_secure_installation 
      // <!--  /*  这是注释：这一项可以不用设置  */ --> </pre>

##### 安装PHPMyadmin

<pre>zxy@zxy-MIBook:~$ sudo apt-get install phpmyadmin
zxy@zxy-MIBook:~$ sudo apt-get install php-mbstring
      // <!--  /*  这是注释：这一步非常非常重要！千万别漏！  */ -->
zxy@zxy-MIBook:~$ sudo apt-get install php-gettext
zxy@zxy-MIBook:~$ sudo ln -s /usr/share/phpmyadmin /var/www/html/phpmyadmin
      // <!--  /*  这是注释：把phpmyadmin链接到Apache的默认目录/var/www/html下  */ --></pre>

##### 配置PHPMyadmin

<pre>zxy@zxy-MIBook:~$ vim /etc/php/7.0/apache2/php.ini </pre>

打开了php.ini后，在文件里面找到以下几行：
**display_errors = Off** 改为：
**display_errors = On** (显示错误日志，文件中出现的都要改，除非是注释，不然无效)


**;extension=php_mbstring.dll** 改为：
**extension=php_mbstring.dll** (也就是取消注释，为了开启mbstring)  

php.ini文件改动好了以后就重启Apache2服务：

<pre>zxy@zxy-MIBook:~$ /etc/init.d/apache2 restart </pre>

##### 验证

在浏览器地址栏中输入localhost/phpmyadmin，如果显示PHPMyadmin登录界面，那么这一步配置成功！

那么所有的步骤配置完成了，你的本地Web服务器也就配置完成了，你就可以尽情地被它蹂躏了！（斜眼笑）

-完-

See U Next Chapter!
@转载请说明出处，谢谢！