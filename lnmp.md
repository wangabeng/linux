
# 参考地址： https://blog.csdn.net/wszll_alex/article/details/76285324
# Centos 7.3搭建LNMP环境


```
1. 关闭防火墙和selinux

打开文件selinux

vim  /etc/sysconfig/selinux

将文件中SELINUX=enforcing改为disabled，然后执行”setenforce 0″不用重启地关闭selinux。

SELINUX=disabled

关闭放火墙
systemctl stop firewalld.service

2.安装软件

2.1.MYSQL安装

下载mysql的repo源

wget http://repo.mysql.com/mysql-community-release-el7-5.noarch.rpm

安装mysql-community-release-el7-5.noarch.rpm包

rpm -ivh mysql-community-release-el7-5.noarch.rpm

安装MYSQL

​ sudo yum install -y  mysql-server

​ 更改MYSQL用户权限：

sudo chown -R root:root /var/lib/mysql

​ 重启服务：

systemctl restart mysql.service

登录，并修改密码：

mysql -u root
mysql > use mysql;
mysql > update user set password=password(‘123456‘) where user=‘root‘;
mysql > flush privileges;
mysql > exit;

2.2nginx安装

下载对应当前系统版本的nginx包

​ wget http://nginx.org/packages/centos/7/noarch/RPMS/nginx-release-centos-7-0.el7.ngx.noarch.rpm

建立nginx的yum仓库（默认yum是没有nginx的）

 rpm -ivh nginx-release-centos-7-0.el7.ngx.noarch.rpm

下载并安装nginx

​ yum install -y nginx

nginx启动

​ systemctl start nginx.service

2.3安装php

rpm 安装 Php7 相应的 yum源

rpm -Uvh https://dl.fedoraproject.org/pub/epel/epel-release-latest-7.noarch.rpm
rpm -Uvh https://mirror.webtatic.com/yum/el7/webtatic-release.rpm

安装php7.0

yum install -y php70w

安装php扩展

yum install -y  php70w-mysql.x86_64   php70w-gd.x86_64   php70w-ldap.x86_64   php70w-mbstring.x86_64  php70w-mcrypt.x86_64

安装PHP FPM

​ yum install -y php70w-fpm

3. 修改配置文件

3.1修改Nginx配置文件

nginx配置文件位置：（/etc/nginx/conf.d/default.conf）

vim /etc/nginx/conf.d/default.conf

​ 修改 root目录,可自定义：

root   /forest/nginxDir/html;

​ 配置php解析，修改 下面代码中黑色加粗部分：

 location ~.php$ {
 root   /forest/nginxDir/html;
​ fastcgi_pass 127.0.0.1:9000;
​ fastcgi_index index.php;
​fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
​ include    fastcgi_params;
​ }

3.2 修改php-fpm配置文件

php-fpm配置文件位置：（/etc/php-fpm.d/www.conf） 
​ 修改

user =nginx
​ group=nginx

4.放入测试文件

cd /forest/nginxDir/html
echo 'hello eric' >index.php

5.启动服务

5.1启动nginx服务：

systemctl start nginx.service

​ 查看启动状态：

systemctl status nginx  

看到以下字眼说明启动成功！ 
​Active: active (running) since 六 2016-11-19 13:40:04 CST; 50min ago

5.2.启动PHP-FPM：

systemctl start php-fpm.service

​ 查看启动状态：

systemctl status php-fpm.service 

看到以下字眼说明启动成功！ 
​Active: active (running) since 六 2016-11-19 14:14:33 CST; 18min ago

6.测试

在浏览器打开192.168.44.129：80/index.php 
看到 hello eric 就大功告成~
```


# 参考2
https://my.oschina.net/u/873934/blog/597319

# 配置很久 都不成功 基本判断是文件权限的问题
--------------------------
```
server {
  listen    80;
  server_name phptest.benkid.cn;

  index index.html index.htm index.php;
  access_log off;
  root  /data/www/testphp/;

  location ~ \.php$ {
    fastcgi_pass  127.0.0.1:9000;
    fastcgi_index index.php;
    fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
    include    fastcgi_params;
  }
}

-----------------------

sudo vim /etc/nginx/nginx.conf
sudo vim /etc/nginx/conf.d/default.conf
sudo vim /data/www/testphp/index.php

sudo service nginx restart

access_log  /var/log/nginx/access.log  main;

sudo rm /etc/nginx/conf.d/default.conf

172.31.241.139 phptest.benkid.cn
```
# 安装lnmp后命令行mysql 报错：
ERROR 1045 (28000): Access denied for user 'abeng'@'localhost' (using password: YES)
```
解决办法:
这种问题需要强行重新修改密码，方法如下：

/etc/init.d/mysql stop   (service mysqld stop )
/usr/bin/mysqld_safe --skip-grant-tables
另外开个SSH连接
[root@localhost ~]# mysql
mysql>use mysql
mysql>update user set password=password("123456") where user="root";
mysql>flush privileges;
mysql>exit

pkill -KILL -t pts/0 可将pts为0的**用户(之前运行mysqld_safe的用户窗口)强制踢出
正常启动 MySQL：/etc/init.d/mysql start   (service mysqld start)
```
