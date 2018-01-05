+++
title = "Meteor(19):部署和监控"
description = "How to deploy, run, and monitor your Meteor app in production."
tags = [
    "JavaScript",
    "NodeJs",
    "Web application",
    "Mobile application",
    "Platform",
    "MongoDB",
]
date = "2017-11-24"
categories = [
    "Development",
    "nodejs",
]
thumbnail = "images/meteor-guide/Meteor-cordova.png"
+++

如何在生产中部署、运行和监视您的Meteor应用程序。

<!--more-->

##  部署Meteor应用程序

### 部署环境

-   VPS/ CentOS 7 : 2.6.32-042stab123.9, CentOS Linux release 7.4.1708 (Core)
-   MongoDB: v3.6.1
-   Nginx: nginx/1.12.2
-   Node:  v8.9.3 

### 环境变量和配置

##  其他考虑

-   用nginx作为门户，代理后台应用，保持域名和端口统一。
-   用nginx作为负载均衡服务器。
-   用nginx代理后台meteor应用，为meteor使用不同的端口提供方便。
-   用nginx代理后台应用，统一使用SSL加密传输。
-   用防火墙(iptables)隔离后台应用端口，比如meteor端口，mysql端口，MongoDB端口。

    ``` 
    iptables -I INPUT 1 -i lo -p tcp --destination-port 4000 -j ACCEPT
    iptables -A INPUT -p tcp --destination-port 4000 -j DROP
    service iptables save
    ```
    
-   在meteor应用中去除ssl服务，该服务不应该由meteor负责。在Package.js中，删除force-ssl依赖。

### 域名

-   Meteor应用**在前端(nginx)和后端(meteor)的域名保持一致**，比如前后端皆为tomato.sitexa.com，只是端口不同。（未测试前后端不一致的情形，比如后端为tomato-api.sitexa.com）

### SSL证书

使用LetsEncrypt生成SSL证书。

####   1. https://certbot.eff.org/#centosrhel7-nginx
####   2. sudo yum install certbot-nginx
####   3. sudo certbot --nginx

```
       Congratulations! You have successfully enabled 
       https://meteor.sitexa.com,
       https://rocket.sitexa.com, 
       https://todos.sitexa.com, 
       https://tomato.sitexa.com,
       and https://www.sitexa.com
       IMPORTANT NOTES:
        - Congratulations! Your certificate and chain have been saved at:
          /etc/letsencrypt/live/meteor.sitexa.com/fullchain.pem
          Your key file has been saved at:
          /etc/letsencrypt/live/meteor.sitexa.com/privkey.pem
          Your cert will expire on 2018-04-01. To obtain a new or tweaked
          version of this certificate in the future, simply run certbot again
          with the "certonly" option. To non-interactively renew *all* of
          your certificates, run "certbot renew"
        - Your account credentials have been saved in your Certbot
          configuration directory at /etc/letsencrypt. You should make a
          secure backup of this folder now. This configuration directory will
          also contain certificates and private keys obtained by Certbot so
          making regular backups of this folder is ideal.    
```
 
####   4, sudo certbot renew —dry-run

```
    --------------------
    Processing /etc/letsencrypt/renewal/meteor.sitexa.com.conf
    --------------------
    Cert not due for renewal, but simulating renewal for dry run
    Plugins selected: Authenticator nginx, Installer nginx
    Starting new HTTPS connection (1): acme-staging.api.letsencrypt.org
    Renewing an existing certificate
    Performing the following challenges:
    tls-sni-01 challenge for meteor.sitexa.com
    tls-sni-01 challenge for rocket.sitexa.com
    tls-sni-01 challenge for todos.sitexa.com
    tls-sni-01 challenge for tomato.sitexa.com
    tls-sni-01 challenge for www.sitexa.com
    Waiting for verification...
    Cleaning up challenges
    
    -------------------
    new certificate deployed with reload of nginx server; fullchain is
    /etc/letsencrypt/live/meteor.sitexa.com/fullchain.pem
    -------------------
    
    -------------------
    ** DRY RUN: simulating 'certbot renew' close to cert expiry
    **          (The test certificates below have not been saved.)
    
    Congratulations, all renewals succeeded. The following certs have been renewed:
      /etc/letsencrypt/live/meteor.sitexa.com/fullchain.pem (success)
    ** DRY RUN: simulating 'certbot renew' close to cert expiry
    **          (The test certificates above have not been saved.)
    -------------------
    
    IMPORTANT NOTES:
     - Your account credentials have been saved in your Certbot
       configuration directory at /etc/letsencrypt. You should make a
       secure backup of this folder now. This configuration directory will
       also contain certificates and private keys obtained by Certbot so
       making regular backups of this folder is ideal.
```    

####   5, cron : certbot renew | system timer

参考：

-   http://www.unixgeeks.org/security/newbie/unix/cron-1.html
-   https://wiki.archlinux.org/index.php/Systemd/Timers

### CDN

##  部署选项

### Galaxy (推荐)

-   国外服务器，显然不适合中国开发者和用户。

### Meteor Up (MUP)

-   对于vps来说，不能升级内核，就不能安装Docker.
-   据社区反应，此方法十分便捷。

### Docker

-   由于在VPS上不能升级内核，达不到安装Docker的需求，所以未能测试。

### 自定义部署

-   动作不复杂，只是每次更新都是重复动作，可以写自动化脚本解决。
-   在开发环境生成发布版本， ```meteor build ./.deploy --architecture os.linux.x86_64 --server  https://tomato.sitexa.com```
-   上传包到服务器:  ```scp tomato.tar.gz user@vps:~/. ```
-   解压缩到网站目录: ```tar xzf tomato.tar.gz /var/www/tomato.sitexa.com/ ```
-   重启meteor服务。（据称可以热部署，未测试）
    
    ```
       export MONGO_URL=mongodb://ip:port/dbname
       export ROOT_URL=http://tomato.sitexa.com
       export PORT=4000
       node main.js &
    ```


##  MongoDB 选项

-   在生产环境，当然是独立服务，或者是独立服务器。MONGO_URL=mongodb://ip:port/dbname.

### 托管服务 (推荐)
    
-   托管服务，或者托管服务器

### 自有服务器

##  部署过程

### 连续部署

### 滚动部署和数据版本

##  通过分析监视用户

##  监视应用程序

### 了解Meteor性能

### 使用Galaxy监控

##  启用 SEO

