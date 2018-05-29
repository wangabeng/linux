
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

# 一些linux操作mysql的常见命令
```
1 查看当前用户
mysql
mysql> status   或者  select user();
2 创建用户
CREATE USER  'user_name'@'host'  IDENTIFIED BY  'password'；
user_name：要创建用户的名字。

host：表示要这个新创建的用户允许从哪台机登陆，如果只允许从本机登陆，则 填　‘localhost’ ，如果允许从远程登陆，则填 ‘%’

password：新创建用户的登陆数据库密码，如果没密码可以不写。

例：

CREATE USER  ‘aaa’@‘localhost’ IDENTIFED BY ‘123456’；          //表示创建的新用户，名为aaa，这个新用户密码为123456，只允许本机登陆

CREATE USER  'bbb'@'%' IDENTIFED BY '123456'；//表示新创建的用户，名为bbb，这个用户密码为123456，可以从其他电脑远程登陆mysql所在服务器

CREATE USER  ‘ccc’@‘%’ ；//表示新创建的用户ccc，没有密码，可以从其他电脑远程登陆mysql服务器

我用 CREATE USER  'aaa'@‘%’；创建新用户，再用 select * from user；查看用户列表

3.授权用户

命令：GRANT privileges ON  databasename.tablename  TO  ‘username’@‘host’

privileges：表示要授予什么权力，例如可以有 select ， insert ，delete，update等，如果要授予全部权力，则填 ALL

databasename.tablename：表示用户的权限能用在哪个库的哪个表中，如果想要用户的权限很作用于所有的数据库所有的表，则填 *.*，*是一个通配符，表示全部。

’username‘@‘host’：表示授权给哪个用户。



例：

GRANT  select，insert  ON  zje.zje  TO ‘aaa’@‘%’；         //表示给用户aaa授权，让aaa能给zje库中的zje表 实行 insert 和 select。

GRANT  ALL  ON  *.*  TO  ‘aaa’@‘%’；//表示给用户aaa授权，让aaa能给所有库所有表实行所有的权力。


用GRANT  ALL  ON  *.*  TO  ‘aaa’@‘%’ ；再看用户列表，可以发现权限都变成 Y了。

注意：

用以上命令授权的用户不能给其他用户授权，如果想这个用户能够给其他用户授权，就要在后面加上   WITH GRANT OPTION

如： GRANT  ALL  ON   *.*   TO  ’aaa‘@'%'  WITH GRANT OPTION； 

4.删除用户

命令：DROP  USER ‘user_name’@‘host’ 

例：

DROP USER 'aaa'@‘%’；//表示删除用户aaa；



5.设置与更改用户密码

SET  PASSWORD  FOR  ‘username’@‘host’ = PASSWORD(‘newpassword’)； 

如果是设置当前用户的密码：

SET  PASSWORD = PASSWORD('newpassword')；

如： SET  PASSWORD = PASSWORD(‘123456’)；


6.撤销用户权限：

命令：REVOKE   privileges   ON  database.tablename  FROM  ‘username’@‘host’；

例如： REVOKE  SELECT ON  *.*  FROM  ‘zje’@‘%’；



但注意：

若授予权利是这样写： GRANT  SELECT  ON  *.*  TO ‘zje’@‘%’；

则用 REVOKE  SELECT ON   zje.aaa  TO  ‘zje’@‘%’；是不能撤销用户zje 对 zje.aaa 中的SELECT 权利的。


反过来 GRANT SELECT  ON  zje.aaa  TO  ‘zje’@‘%’；授予权力

用 REVOKE SELECT ON  *.*  FROM  ‘zje’@‘%’；也是不能用来撤销用户zje 对zje库的aaa表的SELECT 权利的
```
# linux忘记密码
1．首先确认服务器出于安全的状态，也就是没有人能够任意地连接MySQL数据库。 
因为在重新设置MySQL的root密码的期间，MySQL数据库完全出于没有密码保护的 
状态下，其他的用户也可以任意地登录和修改MySQL的信息。可以采用将MySQL对 
外的端口封闭，并且停止Apache以及所有的用户进程的方法实现服务器的准安全 
状态。最安全的状态是到服务器的Console上面操作，并且拔掉网线。 
2．修改MySQL的登录设置： 
# vi /etc/my.cnf 
在[mysqld]的段中加上一句：skip-grant-tables 
例如： 
[mysqld] 
datadir=/var/lib/mysql 
socket=/var/lib/mysql/mysql.sock 
skip-grant-tables 
保存并且退出vi。 
3．重新启动mysqld 
# /etc/init.d/mysqld restart 
Stopping MySQL: [ OK ] 
Starting MySQL: [ OK ] 
4．登录并修改MySQL的root密码 
# /usr/bin/mysql 
Welcome to the MySQL monitor. Commands end with ; or \g. 
Your MySQL connection id is 3 to server version: 3.23.56 
Type 'help;' or '\h' for help. Type '\c' to clear the buffer. 
mysql> USE mysql ; 
Reading table information for completion of table and column names 
You can turn off this feature to get a quicker startup with -A 
Database changed 
mysql> UPDATE user SET Password = password ( 'new-password' ) WHERE User = 'root' ; 
Query OK, 0 rows affected (0.00 sec) 
Rows matched: 2 Changed: 0 Warnings: 0 
mysql> flush privileges ; 
Query OK, 0 rows affected (0.01 sec) 
mysql> quit 
Bye 
5．将MySQL的登录设置修改回来 
# vi /etc/my.cnf 
将刚才在[mysqld]的段中加上的skip-grant-tables删除 
保存并且退出vi。 
6．重新启动mysqld 
# /etc/init.d/mysqld restart 
Stopping MySQL: [ OK ] 
Starting MySQL: [ OK ]
