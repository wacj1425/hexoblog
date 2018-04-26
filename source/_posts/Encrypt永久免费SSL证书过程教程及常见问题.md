---
layout: 申请let’s
title: Encrypt永久免费SSL证书过程教程及常见问题
date: 2018-04-10 09:37:47
tags: [linux,php,ssh]
categories: 运维冷热知识
---
> 虽然目前Let’s Encrypt免费SSL证书默认是90天有效期，但是我们也可以到期自动续约，不影响我们的尝试和使用，为了考虑到文章的真实性和以后的实战性，老左准备利用一些时间分篇幅的展现在应用Let’s Encrypt证书的过程，这篇文章分享申请的方法教程。

<!--more-->

**第一、安装Let’s Encrypt前的准备工作**

根据官方的要求，我们在VPS、服务器上部署Let’s Encrypt免费SSL证书之前，需要系统支持Python2.7以上版本以及支持GIT工具。这个需要根据我们不同的系统版本进行安装和升级，因为有些服务商提供的版本兼容是完善的，尤其是debian环境兼容性比CentOS好一些。
比如CentOS 6 64位环境不支持GIT，我们还可以参考”Linux CentOS 6 64位系统安装Git工具环境教程“和”[9步骤升级CentOS5系统Python版本到2.7](http://www.laozuo.org/2573.html)“进行安装和升级。最为 简单的就是Debian环境不支持，可以运行
`apt-get -y install git`
直接安装支持，如果是CentOS直接运行：
`yum -y install git-core`
支持。这个具体遇到问题在讨论和搜索解决方案，因为每个环境、商家发行版都可能不同。在这篇文章中，老左采用的是debian 7 环境

**第二、快速获取Let’s Encrypt免费SSL证书**

在之前的博文中老左也分享过几篇关于SSL部署的过程，我自己也搞的晕乎晕乎的，获取证书和布局还是比较复杂的，Let’s Encrypt肯定是考虑到推广HTTPS的普及型会让用户简单的获取和部署SSL证书，所以可以采用下面简单的一键部署获取证书。
> PS：在获取某个站点证书文件的时候，我们需要在安装PYTHON2.7以及GIT，更需要将域名解析到当前VPS主机IP中。

~~~
git clone https://github.com/letsencrypt/letsencrypt

cd letsencrypt

./letsencrypt-auto certonly --standalone --email admin@laozuo.org -d laozuo.org -d www.laozuo.org
~~~

然后执行上面的脚本，我们需要根据自己的实际站点情况将域名更换成自己需要部署的。
快速获取Let’s Encrypt免费SSL证书
看到这个界面，直接Agree回车。
![](https://images.laozuo.org/wp-content/uploads/2015/12/letsencrypt-2.jpg)

然后看到这个界面表示部署成功。目前根据大家的反馈以及老左的测试，如果域名是用的国内DNS，包括第三那方DNSPOD等，都可能获取不到域名信息。

![](https://images.laozuo.org/wp-content/uploads/2015/12/letsencrypt-3.jpg)

这里我们可以看到有"The server could not connect to the client to verify the  domain"的错误提示信息，包括也有其他提示错误，"The server experienced an internal error :: Error creating new registration"我们在邮局的时候不要用国内免费邮局。所以，如果我们是海外域名就直接先用域名自带的DNS。

![](https://images.laozuo.org/wp-content/uploads/2015/12/letsencrypt-4.jpg)

**第三、Let's Encrypt免费SSL证书获取与应用**

在完成Let's Encrypt证书的生成之后，我们会在"/etc/letsencrypt/live/laozuo.org/"域名目录下有4个文件就是生成的密钥证书文件
 cert.pem  - Apache服务器端证书
 chain.pem  - Apache根证书和中继证书
 fullchain.pem  - Nginx所需要ssl_certificate文件
 privkey.pem - 安全证书KEY文件

如果我们使用的Nginx环境，那就需要用到fullchain.pem和privkey.pem两个证书文件，在部署Nginx的时候需要用到（参考：[LNMP一键包环境安装SSL安全证书且部署HTTPS网站URL过程](http://www.laozuo.org/5571.html)）。在这篇文章中老左就不详细演示Let’s Encrypt证书证书的安装，后面再重新折腾一篇文章详细的部署证书的安装Nginx和Apache。

~~~
ssl_certificate /etc/letsencrypt/live/laozuo.org/fullchain.pem;
ssl_certificate_key /etc/letsencrypt/live/laozuo.org/privkey.pem;
~~~

比如我们在Nginx环境中，只要将对应的ssl_certificate和ssl_certificate_key路径设置成我们生成的2个文件就可以，最好不要移动和复制文件，因为续期的时候直接续期生成的目录文件就可以，不需要再手工复制。

**第四、解决Let's Encrypt免费SSL证书有效期问题**

我们从生成的文件中可以看到，Let's Encrypt证书是有效期90天的，需要我们自己手工更新续期才可以。

`./letsencrypt-auto certonly --renew-by-default --email admin@laozuo.org -d laozuo.org -d www.laozuo.org`

这样我们在90天内再去执行一次就可以解决续期问题，这样又可以继续使用90天。如果我们怕忘记的话也可以制作成定时执行任务，比如每个月执行一次。

**第五、关于Let's Encrypt免费SSL证书总结**

> 通过以上几个步骤的学习和应用，我们肯定学会了利用Let's Encrypt免费生成和获取SSL证书文件，随着Let's Encrypt的应用普及，SSL以后直接免费不需要购买，因为大部分主流浏览器都支持且有更多的主流商家的支持和赞助，HTTPS以后看来也是趋势。在Let's Encrypt执行过程在中我们需要解决几个问题。

A - 域名DNS和解析问题。在配置Let's Encrypt免费SSL证书的时候域名一定要解析到当前VPS服务器，而且DNS必须用到海外域名DNS，如果用国内免费DNS可能会导致获取不到错误。

B - 安装Let's Encrypt部署之前需要服务器支持PYTHON2.7以及GIT环境，要不无法部署。

C - Let's Encrypt默认是90天免费，需要手工或者自动续期才可以继续使用。

## nginx 配置方法 红色部分替换 图上 标红的地方
~~~
server {
	listen 80;
	listen 443 ssl;
	ssl_certificate /etc/letsencrypt/live/www.emenotec.com/fullchain.pem;
	ssl_certificate_key /etc/letsencrypt/live/www.emenotec.com/privkey.pem;
	ssl_ciphers “EECDH+AESGCM:EDH+AESGCM:AES256+EECDH:AES256+EDH”;
	ssl_protocols TLSv1 TLSv1.1 TLSv1.2;
	ssl_prefer_server_ciphers on;
	ssl_session_cache shared:SSL:10m;
	server_name www.emenotec.com emenotec.com;
	# rewrite ^(.*)$ https://$host$1 permanent;
	# access_log /var/log/nginx/access.log main;
	location / {
		root /www/magento2/blog;
		index index.php index.html index.htm;
		if (-f $request_filename/index.html){
			rewrite (.*) $1/index.html break;
		}
		if (-f $request_filename/index.php){
			rewrite (.*) $1/index.php;
		}
		if (!-f $request_filename){
			rewrite (.*) /index.php;
		}
	}
	rewrite /wp-admin$ $scheme://$host$uri/ permanent;
	location ~ \.php$ {
		root /www/magento2/blog;
		fastcgi_pass unix:/run/php/php7.0-fpm.sock;
		fastcgi_index index.php;
		fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
		include fastcgi_params;
	}
}
~~~

证书上传添加到 stackpath

自动续期
首先我们需要将可执行文件 移动一个公共目录。
如果要放到root目录下，只要处理好权限问题也是可以的。
手动延期

`./certbot-auto renew –dry-run`

利用Cron自动延期
注意路径问题。

`./certbot-auto renew –quiet`

crontab -e
在打开的编辑器中添加如下内容（每个月1号凌晨3点更新）

`0 0 3 * * ./certbot-auto renew –dry-run`