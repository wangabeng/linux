# linux
linux相关

# centOS7 中安装nginx
  详见 https://www.cnblogs.com/Xiao-Bing/p/6722322.html
1.添加Nginx到YUM源

添加CentOS 7 Nginx yum资源库,打开终端,使用以下命令:

sudo rpm -Uvh http://nginx.org/packages/centos/7/noarch/RPMS/nginx-release-centos-7-0.el7.ngx.noarch.rpm

2.安装Nginx

在你的CentOS 7 服务器中使用yum命令从Nginx源服务器中获取来安装Nginx：

sudo yum install -y nginx // 备注 阿里云服务器可以直接安装nginx 无需下载

Nginx将完成安装在你的CentOS 7 服务器中。

附：nginx 官方给不同系统准备了很多包，可到这个地址依次查找 http://nginx.org/packages/

//
始终安装不成功 不仅nginx安装不成功 其他的任何软件都无法安装成功了 vim也不无法安装了 报错信息如下

[abeng@localhost ~]$ sudo yum install vim
[sudo] abeng 的密码：
已加载插件：fastestmirror
http://mirror.centos.org/altarch/7/os/i386/repodata/repomd.xml: [Errno 12] Timeo                                             ut on http://mirror.centos.org/altarch/7/os/i386/repodata/repomd.xml: (28, 'Oper                                             ation too slow. Less than 1000 bytes/sec transferred the last 30 seconds')
正在尝试其它镜像。
base                                                     | 3.6 kB     00:00
extras                                                   | 3.3 kB     00:00
http://nginx.org/packages/centos/7/i386/repodata/repomd.xml: [Errno 14] HTTP Err                                             or 404 - Not Found
正在尝试其它镜像。
To address this issue please refer to the below knowledge base article

https://access.redhat.com/articles/1320623

If above article doesn't help to resolve this issue please create a bug on https                                             ://bugs.centos.org/



 One of the configured repositories failed (nginx repo),
 and yum doesn't have enough cached data to continue. At this point the only
 safe thing yum can do is fail. There are a few ways to work "fix" this:

     1. Contact the upstream for the repository and get them to fix the problem.

     2. Reconfigure the baseurl/etc. for the repository, to point to a working
        upstream. This is most often useful if you are using a newer
        distribution release than is supported by the repository (and the
        packages for the previous distribution release still work).

     3. Run the command with the repository temporarily disabled
            yum --disablerepo=nginx ...

     4. Disable the repository permanently, so yum won't use it by default. Yum
        will then just ignore the repository until you permanently enable it
        again or use --enablerepo for temporary usage:

            yum-config-manager --disable nginx
        or
            subscription-manager repos --disable=nginx

     5. Configure the failing repository to be skipped, if it is unavailable.
        Note that yum will try to contact the repo. when it runs most commands,
        so will have to try and fail each time (and thus. yum will be be much
        slower). If it is a very temporary problem though, this is often a nice
        compromise:

            yum-config-manager --save --setopt=nginx.skip_if_unavailable=true

failure: repodata/repomd.xml from nginx: [Errno 256] No more mirrors to try.
http://nginx.org/packages/centos/7/i386/repodata/repomd.xml: [Errno 14] HTTP Err                                             or 404 - Not Found

折腾了一天，把镜像源删除，用系统自带的yum就可以正确执行，但是无法安装nginx。
通过比对cd /etc/yum.repos.d的CentOS-Base.repo的配置
baseurl=http://mirrors.163.com/centos/$releasever/os/$basearch/
和错误信息：
http://nginx.org/packages/centos/7/i386/repodata/repomd.xml
包括163的镜像源，发现对centos7都不支持32位的系统。
终于明白了 我装的centos系统是7版本i386（即32位系统）的，但是163镜像 或其他版本的镜像 都不支持32位的系统的，只支持X86-64即64位系统的，这就导致一旦安装了yum镜像 就会报以上的错误。解决办法: 我的电脑是32位的 装的是32位的操作系统。既然centos7以上的版本 各大镜像网站都不支持32位的镜像源，我只能选择安装centos6.9及以下的版本， 他们支持6.9及以下的镜像

# nginx操作

查看nginx
service nginx status
查看开启的nginx进程
ps -A | grep nginx
 
现在我们可以使用nginx的控制:
```
sudo service nginx stop 
sudo service nginx start 
sudo service nginx restart
sudo service nginx reload
```

#  关于修改用户权限操作:
https://www.cnblogs.com/fefjay/p/6047820.html

#  关于修改nginx根目录地址打开后出现403错误的问题:
	参考：http://blog.csdn.net/reblue520/article/details/52294555
	1 新的目录为 /data/www/index.html
	路径的任何一级目录或index.html文件没有w权限，就无法访问。解决办法，修改不具有w权限的文件或文件夹的权限。
	2 我已经修改了权限，仍然报错403错误。SELinux设置为开启状态（enabled）的原因。解决办法： 一 临时关闭SELinux（暂时理解为一种安全机制）
	setenforce 0 // 临时关闭
	二 永久关闭：修改配置文件 /etc/ selinux/config，将SELINUX=enforcing改为SELINUX=disabled
	vi /etc/selinux/config

# 配置nginx反向代理失败 不能定向 原因待研究
配置文件 /etc/nginx/conf.d/imooc.def

代码如下：

upstream imooc_hosts {
    server 118.89.106.129:80;
}

server {
    listen       80;
    server_name  www.imooc.test www.wangabeng.test;
    root   /data/www;
    index  index.html index.htm;

    location / {
        # rewrite ^(.*)\.htmp$  /index.html;
        proxy_set_header Host www.54php.cn;
        proxy_pass  http://118.89.106.129:80;
    }
}
原因是标点符号漏写了。

# 每次重新启动电脑 无法正常打开 原因是标点符号漏写了

# 设置负载均衡：主机1是本地服务器主机 主机2是百度主机 百度主机需要设置请求头hsot 为www.baidu.com. 注意：如果修改配置文件后不生效 restart nginx试试
这里应该怎样写 proxy_pass http://imooc_hosts; 
而不是 proxy_pass imooc_hosts;

upstream imooc_hosts {
    server 192.168.159.133:80;
    server 61.135.169.125:80; 
}

server {
    listen       80;
    server_name  www.imooc.test www.wangabeng.test;
    root   /data/www;
    index  index.html index.htm;

    location / {
        # rewrite ^(.*)\.htmp$  /index.html;
        proxy_pass http://imooc_hosts;
        proxy_set_header Host www.baidu.com;
    }
}


# nginx php mysql在centos的运行环境搭建
参考： http://blog.csdn.net/lanxe/article/details/7399300

# 阿里云远程登录相关配置
	远程登录分配PD：230884
	login pd： Name bir（注意 N大写）
	百度站长：Name bir
	公网IP： 47.104.182.26
	内网：172.31.241.139

# 阿里云服务器根目录结构
	[root@izm5e9aw3dxj3q07rx3w7fz ~]# ls / -al
	total 68
	dr-xr-xr-x. 18 root root  4096 Jan 14 13:46 .
	dr-xr-xr-x. 18 root root  4096 Jan 14 13:46 ..
	-rw-r--r--   1 root root     0 Oct 15 15:22 .autorelabel
	lrwxrwxrwx.  1 root root     7 Oct 15 23:19 bin -> usr/bin
	dr-xr-xr-x.  5 root root  4096 Oct 15 23:24 boot
	drwxr-xr-x  20 root root  3040 Jan 14 13:46 dev
	drwxr-xr-x. 80 root root  4096 Jan 14 13:46 etc
	drwxr-xr-x.  2 root root  4096 Nov  5  2016 home
	lrwxrwxrwx.  1 root root     7 Oct 15 23:19 lib -> usr/lib
	lrwxrwxrwx.  1 root root     9 Oct 15 23:19 lib64 -> usr/lib64
	drwx------.  2 root root 16384 Oct 15 23:18 lost+found
	drwxr-xr-x.  2 root root  4096 Nov  5  2016 media
	drwxr-xr-x.  2 root root  4096 Nov  5  2016 mnt
	drwxr-xr-x.  2 root root  4096 Nov  5  2016 opt
	dr-xr-xr-x  73 root root     0 Jan 14 13:46 proc
	dr-xr-x---.  5 root root  4096 Jan 14 12:58 root
	drwxr-xr-x  21 root root   580 Jan 14 13:46 run
	lrwxrwxrwx.  1 root root     8 Oct 15 23:19 sbin -> usr/sbin
	drwxr-xr-x.  2 root root  4096 Nov  5  2016 srv
	dr-xr-xr-x  13 root root     0 Jan 14  2018 sys
	drwxrwxrwt.  8 root root  4096 Jan 14 15:10 tmp
	drwxr-xr-x. 13 root root  4096 Oct 15 23:19 usr
	drwxr-xr-x. 19 root root  4096 Oct 15 15:22 var

# putty设置ssh登录
	第一步： run puttygen生成公钥代码和私钥文件。
	第二步： 保存私钥文件到本地，打开putty客户端（非命令行），载入私钥文件。
	第三步： 拷贝公钥代码，putty登录到远程服务器， 打开 ~/.ssh/authorized_keys，把代码复制到此文件中即可。

	如果设置了免密码登录 仍然登录不了 是因为authorized_keys的权限不够 必须设置为600 权限大了或小了 都不可以

# centos新建用户 并设置密码
	进入 /home目录
	useradd newusername //新增用户名 userdel username 删除帐户名
	passwd newusername // 设置密码
	
# 提权操作
```
	1 在abeng下操作vim 提示没有权限
	2 需要在root账号下授权
	进入root账户
	visudo // 找到/wheel 
	## Allow root to run any commands anywhere
	root    ALL=(ALL)       ALL // 在下面添加一行 即可在abeng下操作 sudo vim filename
	abeng=(ALL)       ALL
```

# 防火墙安装 等命令 参照 https://www.cnblogs.com/lambertwe/p/7352552.html
	yum install firewalld //安装
	service firewalld start // 开启
	service firewalld status // 查看防火墙状态
	service firewalld stop/disable 关闭/禁用 防火墙
	yum list |grep firewall  // 检查防火墙是否安装好了
	
	放行一个端口（即开启一个端口）
	添加
	firewall-cmd --zone=public --add-port=3000/tcp --permanent    （--permanent永久生效，没有此参数重启后失效）
	重新载入
	firewall-cmd --reload
	查看
	firewall-cmd --zone= public --query-port=80/tcp
	删除
	firewall-cmd --zone=public --remove-port=3000/tcp --permanent
	列出所有的开放端口
	firewall-cmd --list-all

# centos安装nvm及node 参照 https://www.jianshu.com/p/f97b436da0ba
```
	一 下面使用nvm安装nodejs 然后提示重新启动shell工具
	wget -qO- https://raw.githubusercontent.com/creationix/nvm/v0.30.1/install.sh | bash 要重启终端
	二 通过简单的命令来安装nodejs https://nodejs.org/zh-cn/
	nvm install 8.9.4  // 安装node的稳定版本 参照官方推荐
	三 nvm启用node
	nvm use node // 或 nvm use v8.9.4
```
# 改变node的npm镜像源为cnpm
	npm install -g cnpm --registry=https://registry.npm.taobao.org

# 在阿里云服务器的端口有两道门，外层是阿里云服务器，内层是防火墙。
	如果想开放3000端口，须设置两道门同时开放3000端口才可。

# 在阿里云上遇到一个问题 开启了一个node服务后 shell关闭 node仍然持续服务。如何办？
	ps -ax | grep node //找出所有node应用
	kill :pid // 例如 kill 14754  杀死特定pid的进程
	pkill node // 杀死所有的node进程
# pm2 命令大全
	http://blog.csdn.net/chengxuyuanyonghu/article/details/74910875
	pm2 list // 列表 PM2 启动的所有的应用程序
	pm2 show [app-name] // 显示应用程序的所有信息
	pm2 stop 0  // 停止 id为 0的指定应用程序
	pm2 restart all   // 重启所有应用
	pm2 delete all  // 关闭并删除所有应用
	pm2 delete 0   // 删除指定应用 id 0
	pm2 startup    // 创建开机自启动命令
# 解决node在非root账户环境不可以设置80端口的问题 
	解决方法 通过nginx反向代理
	我的公网IP xx.xxx.x.xx
	通过访问 xx.xxx.x.xx:80 代理到 xx.xxx.x.xx:3000

	cd /etc/nginx/conf.d/目录  新建文件 wangabeng-cn-3000.conf

	upstream imooc_hosts {
	    server 47.104.182.26:3000;  // 这里改为本地地址 阿里云的本机地址 而不是我的机器的地址 127.0.0.1:3000
	}

	server {
	    listen       80;
	    server_name  47.104.182.26;

	    location / {
	        proxy_pass http://imooc_hosts;
	        proxy_set_header Host $host;
	        proxy_set_header X-Nginx-proxy true;
	        proxy_set_header X-Real-IP $remote_addr;
	        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
	        proxy_redirect off;
	    }
	}

# 关于虚拟主机和虚拟服务器的区别：
	参考网站：https://zhidao.baidu.com/question/518640025224832685.html
	独立ip的有什么优势呢？ 非独立ip的虚拟主机： 1.共享ip主机不能用ip直接打开您的网站。 2.ip是共享的，一个服务器分很多虚拟主机，也就是说很多网站同用一个ip 3.如果收到黑客攻击，那么整个服务器上同ip的网站都受到影响 4.如果这个服务器上某些虚拟主机站点存放了违法信息，被网监查到了，那么就会封ip（现在网络查的都比较严格），那么您的站点也同时受到影响 5.搜索引擎收录，如果因为这个服务器上的某个站点因为作弊，或者违反了搜索引擎收录规则，或者有违法信息等，就有可能被搜索引擎降低权重，那么排名就会靠后了，同时就有可能影响整台服务器上的站点排名。 6.绑定域名有限制，虚拟主机绑定域名是有限制的，并且解析了域名还需要在虚拟主机管理那里进行域名绑定，这样才能访问网站。 独立ip的虚拟主机： 1.可以直接用自己的ip来打开自己的网站。 2.每个站点是独立的ip，完全属于自己的网站使用。 3.如果一台服务器上的其他某个虚拟主机站点收到别人的攻击，那么自己的网站不受影响。 4.如果服务器上的某个虚拟主机站点存放了违法信息，假如网监封了这个站点的ip，您的站点也不受影响 5.提升用户网站被搜索引擎收录级别与机会。如果一个IP只对应一个网站，则搜索引擎会评定该网站质量高从而提高收录级别 6.可以实现泛域名绑定（无限域名绑定空间）。单独IP后可以实现以往Windows虚机实现不了的泛域名绑定功能。

# CentOS查看和修改PATH环境变量的方法
	参考： http://blog.csdn.net/boolbo/article/details/52437760

	查看PATH：echo $PATH
	以添加mongodb server为列
	修改方法一：
	export PATH=/usr/local/mongodb/bin:$PATH
	//配置完后可以通过echo $PATH查看配置结果。
	生效方法：立即生效
	有效期限：临时改变，只能在当前的终端窗口中有效，当前窗口关闭后就会恢复原有的path配置
	用户局限：仅对当前用户
	 
	修改方法二：
	通过修改.bashrc文件:
	vim ~/.bashrc 
	//在最后一行添上：
	export PATH=/usr/local/mongodb/bin:$PATH
	生效方法：（有以下两种）
	1、关闭当前终端窗口，重新打开一个新终端窗口就能生效
	2、输入“source ~/.bashrc”命令，立即生效
	有效期限：永久有效
	用户局限：仅对当前用户

	修改方法三:
	通过修改profile文件:
	vim /etc/profile
	/export PATH //找到设置PATH的行，添加
	export PATH=/usr/local/mongodb/bin:$PATH
	生效方法：系统重启
	有效期限：永久有效
	用户局限：对所有用户

	修改方法四:
	通过修改environment文件:
	vim /etc/environment
	在PATH="/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin:/usr/games:/usr/local/games"中加入“:/usr/local/mongodb/bin”
	生效方法：系统重启
	有效期限：永久有效
	用户局限：对所有用户

## 使用PuTTY在Windows中向Linux上传文件
1 进入putty安装目录
C:\Program Files\PuTTY
然后打开命令行工具输入
传文件, username@IP ADDRESS:/------
```
 pscp d:/abc.txt abeng@116.62.240.193:/data/www
 ```
 传文件夹 加上参数-r 表示发送目录
 ```
pscp -r d:/test abeng@116.62.240.193:/data/www
 ```
 即可把window的数据传送到linux上
 
## 使用PuTTY在Windows中向Linux上传文件2
1 进入putty安装目录  
2 打开psftp.exe  
3 进入linux相关文件夹
4 执行上传 put上传 get下载
```
put something.txt
get something.txt another.txt

put -r /local/test
get -r something
```
如果是上传文件夹 加上 -r

# LINUX忘记普通用户密码 重新设置密码
用超级用户登录，然后输入passwd username 然后重新设置即可

# Linux配置java及maven环境
```
yum search java | grep open // 搜索所有的java带open的安装文件
sudo yum install java-1.8.0-openjdk*  // 执行安装
java -version // 测试是否安装成功
```
# 安装maven
1 下载maven linux版本安装包  
2 上传到linux
3 解压文件  
```
tar -xzvf apache-maven-3.5.2
```
4 配置为全局环境变量
```
sudo ln -s ~/apache-maven-3.6.2/bin/mvn /usr/bin/mvn
```
5 随意文件目下下，输入mvn -version 可以测试是否安装成功
