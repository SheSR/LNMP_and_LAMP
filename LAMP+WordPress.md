# 基于CentOS+LAMP搭建Wordpress博客 #


## 1.安装LAMP环境 ##

**安装 MySQL**

使用 yum 安装 mariadb：

	yum install mariadb-server -y

安装完成后，启动 MySQL 服务：

	service mariadb start

将 MySQL 设置为开机自动启动：

	systemctl enable mariadb

修改mysql密码：

	mysqladmin -u root -p 旧密码 password 新密码
（mysql初始root密码为空，所以旧密码位置为空）


**安装 Apache组件**

使用 yum 安装 Apache 组件：

	yum install httpd -y

安装之后，启动 httpd 进程：

	service httpd start

把 httpd 也设置成开机自动启动：

	systemctl enable httpd


**安装 PHP**

使用 yum 安装 PHP：

	yum install php php-fpm php-mysql -y

安装之后，启动 PHP-FPM 进程：

	service php-fpm start

启动之后，可以使用下面的命令查看 PHP-FPM 进程监听哪个端口 

	netstat -nlpt | grep php-fpm

把 PHP-FPM 也设置成开机自动启动：

	chkconfig php-fpm on

重启apache服务，然后写个php测试页面:

	service httpd restart

	cd /var/www/html/

	vim  index.php
		<?php
    		phpinfo();
		?>
打开浏览器，输入localhost/index.php，查看php版本相关信息



## 2.安装并配置 WordPress ##

**安装 WordPress**

	cd ~
	wget https://cn.wordpress.org/wordpress-4.8.1-zh_CN.tar.gz

将下载的文件解压到Apache文件夹下：

	tar -xzvf wordpress-4.8.1-zh_CN.tar.gz
	mv wordpress /var/www/html/

**配置数据库**

进入 MySQL：

	mysql -uroot --password='Your_Password'

为 WordPress 创建一个数据库：

	CREATE DATABASE wordpress;

MySQL 部分设置完了，我们退出 MySQL 环境：

	exit

把上述的 DB 配置同步到 WordPress 的配置文件中，参考下面修改的几项：

	cd /var/www/html
	cp wp-config-sample.php wp-config.php
	vim wp-config.php
	
	// ** MySQL settings - You can get this info from your web host ** //
	/** The name of the database for WordPress */
	define('DB_NAME', 'wordpress');
	
	/** MySQL database username */
	define('DB_USER', 'root');
	
	/** MySQL database password */
	define('DB_PASSWORD', 'Your_Password');

修改WordPress文件的用户和组：

	chown -R apache:apache /var/www/html/wordpress/*

关闭防火墙：

	systemctl stop firewalld

因为firewalld默认不允许外网的数据访问虚拟机的httpd，不关闭会导致windows访问不了虚拟机的网页


## 大功告成！ ##

通过浏览器访问博客进行相关设置：

	http://xxx.xxx.xxx.xxx/wordpress/install.php

