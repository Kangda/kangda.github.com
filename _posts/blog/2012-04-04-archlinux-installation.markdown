---
date: 2012-04-04 17:31:23+00:00
layout: post
title: ArchLinux Installation
categories: blog
tags:
- Archlinux
description: ArchLinux的安装和其上的LNMP配置
---

之前的Fedora开机的时候总是很慢..让人受不了...然后又遇到了arch，所以索性在毕业论文完了之后把系统重装了一遍。


#### 系统安装


关于系统安装的问题arch的wiki写的很清楚了，就不再记录了，留个链接就行了 [https://wiki.archlinux.org/index.php/Beginners%27_Guide](https://wiki.archlinux.org/index.php/Beginners%27_Guide)

其中，这次安装的主要问题在两个：分区和网络

首先说分区的问题，分区的话，这次不是简单的分成swap和root了，而是分出了150M给boot，15G给home，另15G给root，最后的给swap。单独分出来boot和home有一些好处，单独分boot只是大致知道是有利于管理启动，在多系统的环境下比较有用；单独分出来home可以便于以后重装..因为重装的时候不把home分区给格式化的话还是可以在新的系统上挂载，而且之前的数据都还在，如果依然用的是同样的桌面环境，那很多应用的设置也都可以保留，不用每次都重新做备份。

网络问题就是，arch在安装完成后是命令行界面的，想要安装桌面环境就要联网从库上获取，但是..命令行的网络设置...有线网络的话，ADSL，wiki上有相关的介绍，其他的可以分为要认证和不要认证的，不要认证的当然简单了，你直接插上网线就OK了，但是要认证的，就完了..比如学校的校园网..真的无解了..无线网络的设置在wiki上也有介绍，而且之前做毕业论文也因为这个搞过一天...这次安装幸亏囧的熟人端口，我才能那么顺利的装。

另外，中科大的源真心快，一定上M的速度。

其实，系统装完之后装应用的时候出现了很多的问题..但是当时wordpress的环境还没有搭好，所以很多东西都没有地方记..今晚终于搞好了，而且是在Niginx上的搞的哦，哈哈～那既然这样，其他的先不说了，以后想起来再说，先把ArchLinux上搭建LNMP说下吧～


#### LNMP（Linux+Niginx+Mysql+PHP）


直接上命令吧，把简单的一笔带过

    
    sudo pacman -S mysql mysql-clients mysql-python phpmyadmin php php-fpm php-mcrypt nginx


从上面的命令说开来：

首先，mysql安装挺简单的，上面的命令完了之后基本就剩一个问题了，就是root密码的设置，其实这个问题也比较简单，命令行执行

    
    mysql_secure_installation


这程序会一步一步的设置mysql，起初没有root密码的话，直接enter就OK了

mysql-python是为后面django的运行做准备，phpmyadmin的运行还得依靠nginx和php

php-fpm全称是FastCGI Process Manager for PHP，主要是实现php的执行，貌似在nginx上，php和python都是以CGI程序的形式存在的，大致意思是，php作为一个nginx的后端，如果nginx分析出当前页面需要php来执行了，就会调用php来执行页面，然后获取产生的结果返回给访问者，而php-fpm的全称也就好理解了。

php-mcrypt貌似是一个加密的东西，phpmyadmin要用到

在php的配置文件/etc/php.ini里，要把一些需要的extention给反注释掉，比如mysql.so之类的

nginx的最重要的问题就是配置文件的编写..先把写好的贴上

    
    http {
        include       mime.types;
        default_type  application/octet-stream;
    
        #log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
        #                  '$status $body_bytes_sent "$http_referer" '
        #                  '"$http_user_agent" "$http_x_forwarded_for"';
    
        #access_log  logs/access.log  main;
    
        sendfile        on;
        #tcp_nopush     on;
    
        #keepalive_timeout  0;
        keepalive_timeout  65;
    
        #gzip  on;
    
        server {
            listen       80;
            server_name  localhost;
    	root /srv/http/nginx;
            access_log  logs/host.access.log  main;
    
    	if (!-e $request_filename) {
    		rewrite ^/blog/([_0-9a-zA-Z-]+/?) /blog/index.php last;
    	}
    
    	location / {
                index  index.html index.htm index.php;
            }
    
            error_page   500 502 503 504  /50x.html;
            location = /50x.html {
                root   html;
            }
    
            location ~ \.php$ {
                fastcgi_pass   unix:/var/run/php-fpm/php-fpm.sock;
                fastcgi_index  index.php;
                fastcgi_param  SCRIPT_FILENAME  /srv/http/nginx/$fastcgi_script_name;
                include        fastcgi_params;
            }
    
        }
    }


这只是一个简单的配置文件，现在可以支持phpmyadmin和其中，root是类似apache的www目录路径的设置，主要是最后一个block，[这里](https://wiki.archlinux.org/index.php/Phpmyadmin#NGINX_Configuration)有关于phpmyadmin在nginx的配置，但是这个貌似对新手来说有些难度..因为nginx的很多配置的命令不熟，只能看个大概，可以在[这里](http://wiki.nginx.org/Main)了解，里面的搜索还是可以用的。另外，找到[一篇文章](http://blog.yangtse.me/2011/09/lnmp/)介绍的还是听清楚的..我又懒了..不想自己再重新写一遍了..以后再遇到再说吧..

但是有个东西得记一下，就是那个rewrite，这个是为wp的固定链接而做的，和apache里的mod_rewrite类似。其实，固定链接的实现都是在wp根目录的index.php里，而这个rewrite就相当与把这些页面的请求发给了index.php，index.php里固定链接和实际链接的映射方法，然后会得到要访问的页面。大致是这样的过程，但是关于nginx的rewrite的具体过程和index.php的具体实现过程还不知道..


