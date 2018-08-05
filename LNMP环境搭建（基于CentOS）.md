# 基于CentOS7编译安装搭建LNMP服务器 #


## 1.安装MariaDB ##


安装必要组件
		
	yum install –y autoconf automake imake libxml2-devel expat-devel cmake gcc gcc-c++ libaio libaio-devel bzr bison libtool ncurses5-devel zlib-devel gnutls-devel ncurses-devel


添加用户

	useradd -M -s /sbin/nologin mysql

编译安装

	tar zxvf mariadb-10.0.32.tar.gz
	cd mariadb-10.0.32

	cmake . -DCMAKE_INSTALL_PREFIX=/usr/local/mariadb \
	-DMYSQL_DATADIR=/data/mariadb \
	-DMYSQL_UNIX_ADDR=/tmp/mysql.sock \
	-DWITHOUT_TOKUDB=1 \
	-DWITH_XTRADB_STORAGE_ENGINE=1 \
	-DWITH_INNOBASE_STORAGE_ENGINE=1 \
	-DWITH_PARTITION_STORAGE_ENGINE=1 \
	-DWITH_BLACKHOLE_STORAGE_ENGINE=1 \
	-DWITH_MYISAM_STORAGE_ENGINE=1 \
	-DDEFAULT_CHARSET=utf8 \
	-DDEFAULT_COLLATION=utf8_general_ci \
	-DEXTRA_CHARSETS=all \（去掉这行）
	-DENABLED_LOCAL_INFILE=1 \
	-DWITH_READLINE=1 \
	-DWITH_EXTRA_CHARSETS=1 \（去掉这行）
	-DWITH_SSL=bundled \
	-DWITH_ZLIB=bundled

	make && make install


执行完成也就是安装完成了，现在执行 cd /usr/local/mariadb/ 进入mysql安装目录分别执行下面命令：

	chown -R mysql:mysql /usr/local/mariadb/

创建数据库：

	cd /usr/local/mariadb/scripts
	./mysql_install_db --user=mysql --basedir=/usr/local/mariadb --datadir=/usr/local/mariadb/data/

将服务器启动文件加入到系统启动中：

	cp support-files/mysql.server /etc/init.d/mysqld
	chmod +x /etc/init.d/mysqld （启动服务/etc/init.d/mysqld start）
	chkconfig --add mysqld (查看chkconfig –list添加的服务)
	chkconfig mysqld on （添加服务随系统自启动）

bin目录加入path （添加环境变量使用MySQL命令）

	echo 'export PATH=$PATH:/usr/local/mariadb/bin' >>/etc/profile

立即生效：

	source /etc/profile

拷贝mariadb的配置文件：

	cd /usr/local/mariadb/support-files
	cp my-small.cnf /etc/my.cnf

其实初始化后就可以连接到数据库了，用户root，密码为空！

在/etc/init.d/mysqld 补全下列设置

	vim /etc/init.d/mysqld

	basedir="/usr/local/mariadb"
	datadir="/usr/local/mariadb/data"

将启动文件复制到 /usr/local/sbin/文件夹下

	cp  /etc/init.d/mysqld  /usr/local/sbin/



## 2.NGINX安装 ##

安装pcre

为了支持rewrite功能，我们需要安装pcre

	yum install pcre* //如过你已经装了，请跳过这一步

安装openssl

需要ssl的支持，如果不需要ssl支持，请跳过这一步

	yum install openssl*

安装nginx

执行如下命令：

	 ./configure --prefix=/usr/local/nginx-1.5.1 \
	--with-http_ssl_module \
	--with-http_v2_module \
	--with-http_stub_status_module \
	--with-pcre
	
	make && make install

启动nginx

	/usr/local/nginx/sbin/nginx

试试访问

	curl -s http://localhost | grep nginx.com

查看运行状态

	netstat -ntlp | grep nginx




## 3.安装PHP ##

安装依赖包

	yum install gcc make gd-devel libjpeg-devel libpng-devel libxml2-devel bzip2-devel libcurl-devel

解压

	tar -xzvf php-7.2.6.tar.gz
	cd php-7.2.6.tar.gz

编译安装

	./configure --prefix=/usr/local/php-7.2.6 \
	--with-config-file-path=/usr/local/php-7.2.6/etc \
	--with-bz2 --with-curl \
	--enable-ftp --enable-sockets --disable-ipv6 --with-gd \
	--with-jpeg-dir=/usr/local --with-png-dir=/usr/local \
	--with-freetype-dir=/usr/local --enable-gd-native-ttf \
	--with-iconv-dir=/usr/local --enable-mbstring --enable-calendar \
	--with-gettext --with-libxml-dir=/usr/local --with-zlib \
	--with-pdo-mysql=mysqlnd --with-mysqli=mysqlnd --with-mysql=mysqlnd \
	--enable-dom --enable-xml --enable-fpm --with-libdir=lib64 --enable-bcmath
	
	make && make install



配置php

	cp php.ini-production /usr/local/php/etc/php.ini
	cp /usr/local/php/etc/php-fpm.conf.default  /usr/local/php/etc/php-fpm.conf

启动php-fpm

	/usr/local/php-7.2.6/sbin/php-fpm


如果运行时提示如下错误：

WARNING: Nothing matches the include pattern '/usr/local/php/etc/php-fpm.d/*.conf' from /usr/local/php/etc/php-fpm.conf at line 125.

运行如下命令

	cp /usr/local/php/etc/php-fpm.d/www.conf.default  /usr/local/php/etc/php-fpm.d/www.conf

查看运行状态

	netstat -ntlp| grep php



配置nginx与php连接测试

	mkdir -p /data/logs/nginx/ # 用于存放nginx日志.
	mkdir -p /data/site/test.ssr.com/ # 站点根目录
	vim /data/site/test.ssr.com/info.php
	<?php
	phpinfo();
	?>

nginx配置，在nginx.conf中的http段加入以下内容
	
	server {
	listen 80;
	server_name test.ssr.com;
	access_log /data/logs/nginx/test.ssr.com.access.log main;
	 
	index index.php index.html index.html;
	root /data/site/test.ssr.com;
	 
	location /
	{
	try_files $uri $uri/ /index.php?$args;
	}
	 
	location ~ .*\.(php)?$
	{
	expires -1s;
	try_files $uri =404;
	fastcgi_split_path_info ^(.+\.php)(/.+)$;
	include fastcgi_params;
	fastcgi_param PATH_INFO $fastcgi_path_info;
	fastcgi_index index.php;
	fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
	fastcgi_pass 127.0.0.1:9000;
	}
	}

语法测试

	/usr/local/nginx/sbin/nginx -t

如果发生如下提示：

unknown log format "main" in /usr/local/nginx/conf/nginx.conf:38

需要将nginx.conf里面的http段中log_format的注释去掉（一般在21-23行）


访问测试

	curl http://test.ssr.com/info.php 

若出现以下内容，则说明php安装与配置完成

	test php


至此LNMP安装配置完成