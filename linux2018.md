
# 参考地址： https://blog.csdn.net/wszll_alex/article/details/76285324
Centos 7.3搭建LNMP环境


1. 关闭防火墙和selinux

打开文件selinux

vim  /etc/sysconfig/selinux
1
将文件中SELINUX=enforcing改为disabled，然后执行”setenforce 0″不用重启地关闭selinux。

SELINUX=disabled
1
关闭放火墙

systemctl stop firewalld.service
1
2.安装软件

2.1.MYSQL安装

下载mysql的repo源

wget http://repo.mysql.com/mysql-community-release-el7-5.noarch.rpm
1
安装mysql-community-release-el7-5.noarch.rpm包

rpm -ivh mysql-community-release-el7-5.noarch.rpm
1
安装MYSQL

​ sudo yum install -y  mysql-server
1
​ 更改MYSQL用户权限：

sudo chown -R root:root /var/lib/mysql
1
​ 重启服务：

systemctl restart mysql.service
1
登录，并修改密码：

mysql -u root
mysql > use mysql;
mysql > update user set password=password(‘123456‘) where user=‘root‘;
mysql > flush privileges;
mysql > exit;
1
2
3
4
5
2.2nginx安装

下载对应当前系统版本的nginx包

​ wget http://nginx.org/packages/centos/7/noarch/RPMS/nginx-release-centos-7-0.el7.ngx.noarch.rpm
1
建立nginx的yum仓库（默认yum是没有nginx的）

 rpm -ivh nginx-release-centos-7-0.el7.ngx.noarch.rpm
1
下载并安装nginx

​ yum install -y nginx
1
nginx启动

​ systemctl start nginx.service
1
2.3安装php

rpm 安装 Php7 相应的 yum源

rpm -Uvh https://dl.fedoraproject.org/pub/epel/epel-release-latest-7.noarch.rpm
rpm -Uvh https://mirror.webtatic.com/yum/el7/webtatic-release.rpm
1
2
安装php7.0

yum install -y php70w
1
安装php扩展

yum install -y  php70w-mysql.x86_64   php70w-gd.x86_64   php70w-ldap.x86_64   php70w-mbstring.x86_64  php70w-mcrypt.x86_64
1
安装PHP FPM

​ yum install -y php70w-fpm
1
3. 修改配置文件

3.1修改Nginx配置文件

nginx配置文件位置：（/etc/nginx/conf.d/default.conf）

vim /etc/nginx/conf.d/default.conf
1
​ 修改 root目录,可自定义：

root   /forest/nginxDir/html;
1
​ 配置php解析，修改 下面代码中黑色加粗部分：

 location ~.php$ {
 root   /forest/nginxDir/html;
​ fastcgi_pass 127.0.0.1:9000;
​ fastcgi_index index.php;
​fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
​ include    fastcgi_params;
​ }
1
2
3
4
5
6
7
3.2 修改php-fpm配置文件

php-fpm配置文件位置：（/etc/php-fpm.d/www.conf） 
​ 修改

user =nginx
​ group=nginx
1
2
4.放入测试文件

cd /forest/nginxDir/html
echo 'hello eric' >index.php
1
2
5.启动服务

5.1启动nginx服务：

systemctl start nginx.service
1
​ 查看启动状态：

systemctl status nginx  
1
看到以下字眼说明启动成功！ 
​Active: active (running) since 六 2016-11-19 13:40:04 CST; 50min ago

5.2.启动PHP-FPM：

systemctl start php-fpm.service
1
2
​ 查看启动状态：

systemctl status php-fpm.service 
1
看到以下字眼说明启动成功！ 
​Active: active (running) since 六 2016-11-19 14:14:33 CST; 18min ago

6.测试

在浏览器打开192.168.44.129：80/index.php 
看到 hello eric 就大功告成~
