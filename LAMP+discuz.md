# 基于CentOS+LAMP搭建Discuz论坛 #


## 1.安装LAMP环境 ##
即Linux、Apache、MySQL 和 PHP 的缩写，是 Discuz 论坛系统依赖的基础运行环境。


**安装 MySQL**

使用 yum 安装 MariaDB：

	yum install mariadb mariadb-server -y

完成安装后，启动 MySQL 服务：

	service mariadb restart

设置自己的 MariaDB 账户名和密码，参考下面的内容：

	mysql_secure_installation

首先是设置密码，会提示先输入密码
Enter current password for root (enter for none):<–初次运行直接回车

设置密码

	Set root password? [Y/n] <– 是否设置root用户密码，输入y并回车或直接回车
	New password: <– 设置root用户的密码
	Re-enter new password: <– 再输入一次你设置的密码

将 MariaDb 设置为开机自动启动：

	chkconfig mariadb on


**安装 Apache 组件**

使用 yum 安装 Apache 组件：

	yum install httpd -y

安装之后，启动 httpd 进程：

	service httpd start

把 httpd 也设置成开机自动启动：

	chkconfig httpd on

**安装 PHP**

使用 yum 安装 PHP：

	yum install php php-fpm php-mysql -y

安装之后，启动 PHP-FPM 进程：

	service php-fpm start

启动之后，可以使用下面的命令查看 PHP-FPM 进程监听哪个端口：

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


## 2.安装并配置 Discuz ##

CentOS 没有Discuz 的 yum 源，所以我们需要下载一个Discuz 压缩包：

	wget http://download.comsenz.com/DiscuzX/3.2/Discuz_X3.2_SC_UTF8.zip

下载完成后，解压这个压缩包：

	unzip Discuz_X3.2_SC_UTF8.zip

解压完后，就能在 upload 文件夹里看到discuz的源码了。


**配置 Discuz**

由于PHP默认访问 /var/www/html/ 文件夹，所以我们需要把upload文件夹里的文件都复制到 /var/www/html/ 文件夹：

	cp -r upload/* /var/www/html/

给 /var/www/html 目录及其子目录赋予权限：

	chmod -R 777 /var/www/html

重启 Apache：

	service httpd restart

关闭防火墙：

	systemctl stop firewalld

因为firewalld默认不允许外网的数据访问虚拟机的httpd，不关闭会导致windows访问不了虚拟机的网页

部署完成后，通过浏览器访问论坛启动相关设置
通过IP地址访问：http://xxx.xxx.xxx.xxx/install
（配置discuz服务器的ip地址） </br>
设置完成后通过访问：http://xxx.xxx.xxx.xxx 即可查看效果。



**可能遇到的问题**

yum运行时提示Another app is currently holding the yum lock; waiting for it to exit...

可能是系统自动升级正在运行，yum在锁定状态中。 
已经有一个yum进程在运行了，使用kill干掉它：

	# kill -s 9 25960
	# ps aux|grep yum

如果还是不行，可以尝试强制关掉yum进程：

	#rm -f /var/run/yum.pid

应该就可以使用了。