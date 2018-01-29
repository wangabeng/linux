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
	login pd： Name bir
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
	1 在abeng下操作vim 提示没有权限
	2 需要在root账号下授权
	进入root账户
	visudo // 找到/wheel 
	%wheel  ALL=(ALL)       ALL // 在下面添加一行 即可在abeng下操作 sudo vim filename
	%wheel  abeng=(ALL)       ALL
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
	一 下面使用nvm安装nodejs 然后提示重新启动shell工具
	wget -qO- https://raw.githubusercontent.com/creationix/nvm/v0.30.1/install.sh | bash
	二 通过简单的命令来安装nodejs https://nodejs.org/zh-cn/
	nvm install 8.9.4  // 安装node的稳定版本 参照官方推荐
	三 nvm启用node
	nvm use node // 或 nvm use v8.9.4

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

# Linux安装MongoDB 参照 https://www.cnblogs.com/brucetang/p/5965493.html
	
	第一步 下载
	// 32位的下载
	wget   http://downloads.mongodb.org/linux/mongodb-linux-i686-v3.2-latest.tgz?_ga=2.114898759.1713724367.1498550277-1089294971.1498550277
	// 64位下载地址
	wget http://downloads.mongodb.org/linux/mongodb-linux-x86_64-3.2.1.tgz
	第二步 解压 改名
	tar zxf mongodb-linux-x86_64-3.2.1.tgz
	mv mongodb-linux-x86_64-3.2.1/ /usr/local/mongodb

	第三步 创建数据文件和日志文件

	mkdir -p /data/mongodb/
	mkdir /data/mongodb/logs
	touch /data/mongodb/logs/mongod.log
	// 坑坑坑 无法启动 网上有人和我遇到类似的问题

	data和logs应该放在安装目录下，就不会报错了，大坑，大坑。
	所以 以上路径更改为
	mkdir -p /usr/local/mongodb/data
	mkdir /usr/local/mongodb/logs
	touch /usr/local/mongodb/logs/mongod.log

	第四步 在安装mongodb的用户下添加如下环境变量,以便直接使用mongodb bin目录下的命令
	export PATH=$PATH:/usr/local/mongodb/bin/

	备注：这样创建的只是临时的环境变量，shell窗口关闭后，此环境变量就消失了。永久创建方法如下：
	通过修改.bashrc文件:
	vim ~/.bashrc 
	//在最后一行添上：
	export PATH=/usr/local/mongodb/bin:$PATH
	生效方法：（有以下两种）
	1、关闭当前终端窗口，重新打开一个新终端窗口就能生效
	2、输入“source ~/.bashrc”命令，立即生效
	有效期限：永久有效
	用户局限：仅对当前用户

	第五步 启动mongodb

	mongod --dbpath=/usr/local/mongodb/data --logpath=/usr/local/mongodb/logs/mongod.log --logappend  --port=27017

	第六步 检查端口是否启动，端口为：27017
	netstat -nlp | grep 27017
	tcp        0      0 0.0.0.0:27017               0.0.0.0:*                   LISTEN      1897/mongod         
	unix  2      [ ACC ]     STREAM     LISTENING     11098  1897/mongod         /tmp/mongodb-27017.sock
	启动成功。

	第七步 连接数据库
	# mongo
	> use test

	第八步 设置mongodb自动启动（无权限）
	将如下命令添加到 /etc/rc.local
	mongod --dbpath=/usr/local/mongodb/data --logpath=/usr/local/mongodb/logs/mongod.log --logappend  --port=27017&

	第九步 设置mongodb自动启动（有权限）
	将如下命令添加到 /etc/rc.local

	mongod --dbpath=/usr/local/mongodb/data --logpath=/usr/local/mongodb/logs/mongod.log --logappend  --port=27017& -auth


	开机启动设置不成功 原因参考
	https://www.cnblogs.com/koboshi/p/4036312.html

	http://blog.csdn.net/u013940812/article/details/60961296

	按照mongodb游记作者的方法配置 仍然不成功。

	作者提到解压的方法安装有很多缺陷。建议yum安装.

# 用yum重新安装mongodb及配置安装启动
	http://www.linuxidc.com/Linux/2016-12/137790.htm
	参考 http://blog.csdn.net/zhuchuangang/article/details/51202752	
	1、准备工作
	运行yum命令查看MongoDB的包信息 
	# yum info mongodb
	（提示没有相关匹配的信息，） 说明你的centos系统中的yum源不包含MongoDB的相关资源，所以要在使用yum命令安装MongoDB前需要增加yum源，也就是在 /etc/yum.repos.d/目录中增加 *.repo yum源配置文件

	2、vim /etc/yum.repos.d/mongodb.repo，输入下面的语句：
	[mongodb]
	name=MongoDB Repository
	baseurl=https://repo.mongodb.org/yum/redhat/$releasever/mongodb-org/3.4/x86_64/
	gpgcheck=0
	enabled=1

	3、安装MongoDB的服务器端和客户端工具  
	yum install mongodb

	4、启动mongodb
	若输入 mongod -help 出现很多帮助信息 就说明安装成功了。mongod前没有加路径，是因为yum安装mongodb会把它自动添加到环境变量中，而不需要手工添加。如果是通过解压包安装，就需要手工增加环境变量。

	mongodb安装成功后配置文件：
	/etc/mongod.conf

	默认log文件
	 /var/log/mongodb/mongod.log
	默认dbpath
	dbPath: /var/lib/mongo
	默认端口号：
	27017

	如果这样启动
	mongod
	会出现如下错误提示:
	2018-01-18T14:06:24.481+0800 I STORAGE  [initandlisten] exception in initAndListen: 29 Data directory /data/db not found., terminating
	意思是/data/db没有找到 在系统中这个文件夹没有创建 (mongod 后面不加dbpath参数会自动找 /data/db 这个文件夹 即便修改了config文件的dbpath位置 还是要加参数 不加就默认找/data/db)所以报错，解决办法是创建这样一个文件夹然后用mongod 不指定dbpath路径

	还可以指定路径启动(前提是创建了/mongodb/data这个文件夹)
	mongod --dbpath=/mongodb/data
	这个时候又有错误提示：
	[initandlisten] exception in initAndListen: 20 Attempted to create a lock file on a read-only directory: /mongodb/data, terminating
	大概是有个文件锁住了 directory: /mongodb/data
	找到这个文件夹 有个文件mongod.lock 把它删除了
	再执行
	mongod --dbpath=/mongodb/data 发现还是不成功 原来是因为权限不够
	sudo mongod --dbpath=/mongodb/data  
	就成功了 提示 waiting for connections on port 27017
	再开启一个终端 输入mongo 就进入数据库了 
	>use test
	>show dbs

	也可以这样启动(配置文件里有 --fork为true 意思是守护进程 在内存中运行 但是在客户端却看不到 可以通过sudo pkill mongod把进程关掉) 
	sudo mongod --config /etc/mongod.conf


	5、设置开机启动mongodb
	编辑此文件 sudo vim /etc/rc.local
	在最后一行添加
	mongod --config /etc/mongod.conf
	（以mongod.conf此文件的配置来执行开机启动 mongod.conf里面配置了dbpath和logs路径 还有守护进程选项 可谓非常完美的 所以以此配置文件来启动mongodb是非常好的方法 如果需要更改设置 就直接更改mongod.conf文件即可）
	
	6、mongodb自由开启和关闭
	自由开启
	// 守护进程登录
	sudo mongod --dbpath=/mongodb/data --logpath=/mongodb/logs/mongod.log --logappend --port=27017 &
	
	// --auth开启用户权限认证模式登录
	sudo mongod --dbpath=/mongodb/data --logpath=/mongodb/logs/mongod.log --logappend --port=27017 --auth &
	&相当于fork 守护进程方式开启 必须同时加--logspath守护进程才会生效
	或以下两种方式（这两种方式等价 推荐）
	mongod --config /etc/mongod.conf
	mongod -f /etc/mongod.conf

	mongod --config /etc/mongod.conf  

	sudo pkill mongodb // 杀死所有mongodb进程

	使用kill杀死进程

	ps aux | grep mongod // 查看所有的mongod进程 
	kill -2 PID // 杀死特定pid的进程 -1是查看 -2和-15都会等mongodb处理完事情释放相应资源后再停止。 -9是马上停止进程 比较暴力，不要使用，否则很可能导致数据损坏.

	2018年1月28日杀死mongodb进程 用以上方法出现的问题：
		每次pid值都不一样 无法杀死
	http://blog.itpub.net/15498/viewspace-2122582/
	参照这个介绍：
	 ps -ef | grep mongod | grep -v grep
	 // root       954     1  0 Jan20 ?        00:56:28 mongod --config /etc/mongod.conf
	 sudo kill -2 954 // 成功



	7、mongodb启动相关问题
	如果出现这样的提示
	ERROR: child process failed, exited with error number 100

	解决办法  删除/var/lib/mongo/下的   mongod.lock 然后重启

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

# Mongodb专项学习(参考：https://www.jianshu.com/p/75ba25415218)
	1 开启权限认证 增强安全性
	安装好数据库后 开启数据库
	然后连接数据库

	第一步：默认auth未开启。登录数据库后 
	mongo -port 27017
	登录数据库后 进入admin数据库
	>show dbs
	// admin local
	>user admin
	>db.createUser({user:'imooc', pwd: 'imooc', roles: [{role: "userAdminAnyDatabase", db: "admin"}]})

	//
	Successfully added user: {
	        "user" : "imooc",
	        "roles" : [
	                {
	                        "role" : "userAdminAnydatabase",
	                        "db" : "admin"
	                },
	                {
	                        "role" : "read",
	                        "db" : "test"
	                }
	        ]
	}
	第二步： 然后重新启动数据库 加上auth参数（如果通过配置文件启动 可以在配置文件里添加这个选项）
	sudo mongod --dbpath=/mongodb/data --logpath=/mongodb/logs/mongod.log --logappend --port=27017 --auth &  
	此时 如果用默认账号登录 
	mongo
	> show dbs// 就会出错 说没有权限 
	以账号密码方式登录
	mongo -u imooc -p imooc --authenticationDatabase 'admin' // 这样登录很麻烦 也可以直接mongo登录 然后 use admin 
	db.auth('imooc', 'imooc') //主账号登录
	然后就可以操作数据库了

	当然不能一直用主账号登录，要在主账号登录的环境下 创建特定数据库的管理员账号
	use admin
	db.auth('imooc', 'imooc') //主账号登录 如果加参数登录了 这里就不需要db.auth了

	然后创建特定数据库的管理员
	use test
	db.createUser({user: "myTester", pwd: "xyz123", roles: [{role: "readWrite", db: "test"}, {role: "read", db: "reporting"}]}) // 未创建

	db.createUser({user: "runjieAdmin", pwd: "528528", roles: [{role: "readWrite", db: "runjie"}]}) // 已经创建

	使用普通用户账号登陆
	mongo --port27017
	use test
	db.auth("myTester","xyz123")
 
	mongo -u runjieAdmin -p 528528 --authenticationDatabase 'runjie' //
	db.auth("runjieAdmin","528528")

	向数据集中添加纪录
	db.foo.insert({x:1, y:1})

	修改密码：(需要管理员权限)
	db.changeUserPassword('mytest','newpassword')
	查看有哪些用户 (需要管理员权限)
	show users
	删除用户 (需要管理员权限)
	db.dropUser('mytest')

	# 出现的问题：
	使用用户认证登录后 无法删除数据库 暂时的解决办法是 非用户授权登录 删除 数据库 然后再用户授权登录

	# mongodb数据库导出导入

# mongodb配置文件参数设置
	https://www.jianshu.com/p/f9f1454f251f

# 关于虚拟主机和虚拟服务器的区别：
	参考网站：https://zhidao.baidu.com/question/518640025224832685.html
	独立ip的有什么优势呢？ 非独立ip的虚拟主机： 1.共享ip主机不能用ip直接打开您的网站。 2.ip是共享的，一个服务器分很多虚拟主机，也就是说很多网站同用一个ip 3.如果收到黑客攻击，那么整个服务器上同ip的网站都受到影响 4.如果这个服务器上某些虚拟主机站点存放了违法信息，被网监查到了，那么就会封ip（现在网络查的都比较严格），那么您的站点也同时受到影响 5.搜索引擎收录，如果因为这个服务器上的某个站点因为作弊，或者违反了搜索引擎收录规则，或者有违法信息等，就有可能被搜索引擎降低权重，那么排名就会靠后了，同时就有可能影响整台服务器上的站点排名。 6.绑定域名有限制，虚拟主机绑定域名是有限制的，并且解析了域名还需要在虚拟主机管理那里进行域名绑定，这样才能访问网站。 独立ip的虚拟主机： 1.可以直接用自己的ip来打开自己的网站。 2.每个站点是独立的ip，完全属于自己的网站使用。 3.如果一台服务器上的其他某个虚拟主机站点收到别人的攻击，那么自己的网站不受影响。 4.如果服务器上的某个虚拟主机站点存放了违法信息，假如网监封了这个站点的ip，您的站点也不受影响 5.提升用户网站被搜索引擎收录级别与机会。如果一个IP只对应一个网站，则搜索引擎会评定该网站质量高从而提高收录级别 6.可以实现泛域名绑定（无限域名绑定空间）。单独IP后可以实现以往Windows虚机实现不了的泛域名绑定功能。

××××××××××××××××××××××××××××××××××××××××××××××××××××××××××××××××××
# git问题汇总
# git的编译安装最新版本方法
	我购买的阿里云yum安装的git 竟然是很低的版本1.8.1的
	卸载重新安装步骤 参见https://www.cnblogs.com/oufeng/p/6614042.html

		--安装gcc
		sudo yum install gcc

		--安装g++
		sudo yum install gcc-c++

		--安装编译所需的包
		sudo yum install curl-devel expat-devel gettext-devel openssl-devel zlib-devel
		sudo yum install gcc perl-ExtUtils-MakeMaker
		复制代码
		下载源码(*.tar.gz)到指定的目录

		--下载文件到/usr/src/git-2.12.2目录
		wget -P ~/src/git-2.12.2 https://www.kernel.org/pub/software/scm/git/git-2.12.2.tar.gz
		切换到刚刚保存下载文件的目录并解压文件

		--切换到指定目录
		cd /usr/src/git-2.12.2/

		--解压源码包
		tar zxvf git-2.12.2.tar.gz
		进入解压目录

		cd git-2.12.2/
		配置安装目录并编译和安装
		./configure 是待编译文件的路径
		--prefix=/usr/local/git-2.12.2意思是设置这个程序编译后的安装的路径 你编译后的程序会放在/usr/local/git-2.12.2目录下
		sudo ./configure --prefix=/usr/local/git-2.12.2 && sudo make install

		编译安装完成后 新的版本会自动添加到环境变量中


# git push的权限问题 虽然我更新了git的最新版本 但是每到git push的时候 	还是会出现权限的问题 
	提示信息如下：
	Permission denied (publickey).
	fatal: Could not read from remote repository.

	解决方法：
	https://www.phpsong.com/1860.html

	git config --global push.default simple // 成功 即把git push的默认模式改为simple模式 何为simple模式：

	在中央仓库工作流程模式下，拒绝推送到上游与本地分支名字不同的分支。也就是只有本地分支名和上游分支名字一致才可以推送，就算是推送到不是拉取数据的远程仓库，只要名字相同也是可以的。在GIT 2.0中，simple将会是push.default的默认值。simple只会推送本地当前分支。
	修改以后 ~/.gitconfig文件的配置如下： 
	[user]
	        name = wangabeng
	        email = wangabeng@163.com
	[push]
	        default = simple


	备注：如果不设置 在第一次推送的时候 设置好推送的分支 应该也可以
	git push -u origin master 

	在git push的时候不需要sudo权限

	above all
	如果总是出现权限的问题 一般都是文件夹的权限问题 就把项目文件夹权限设置最高
	chmod -R 777 /www/itbulu.com/wp-content（参照 http://www.jb51.net/article/110277.htm）

# 遇到的问题 在本地删除了几个文件后 
	git add . // 出现错误提示
	* 'git add --all <pathspec>' will let you also record the removals.

	Run 'git status' to check the paths you removed from your working tree.
	
	// 于是执行
	git add --all  // 成功解决


# 需求1 (仅供参考 不是最佳实践)
本地建了一个仓库 有文件a 需要上传到github上
 第一步 在本地文件夹根目录 初始化git仓库 git init

 第二步 本地仓库添加git源 git remote add origin git@github.com:wangabeng/aliyundata.git  此时 查看git源 git remote -v origin git@github.com:wangabeng/aliyundata.git (fetch) origin git@github.com:wangabeng/aliyundata.git (push)

 git修改源镜像
 git remote set-url origin newurl

 第三步 把本地代码提交到暂存区 工作区  git add . git commit -m 'add'

 第四步 合并当前分支到远程master分支 sudo git fetch origin master  此时会报错 因为本地没有工作流记录 fatal: refusing to merge unrelated histories  所以要添加参数 --allow-unrelated-histories 
 git fetch origin master --allow-unrelated-histories

 第五步 git push -u origin master  提交到远程仓库的master分支 -u设置默认提交的主机名（origin） 分支名（master）

 成功

# git需求 最佳实践
	远程仓库已经创建好 本地只有一个空目录 如何把远程代码下载到本地
	https://www.douban.com/note/622560328/
	1 进入本地目录
	git init

	2 获取remote
	git remote add origin git@github.com:wangabeng/aliyundata.git

	3 从远程获取最新版本到本地
	使用如下命令可以在本地新建一个temp分支，并将远程origin仓库的master分支代码下载到本地temp分支(没有temp分支 就创建这个分支)
	$  git fetch origin master:temp  // 此时只有一个分支 temp
	
	4比较本地仓库与下载的temp分支
	使用如下命令来比较本地代码与刚刚从远程下载下来的代码的区别：

	$  git diff temp  

	合并temp分支到本地的master分支对比区别之后，如果觉得没有问题，可以使用如下命令进行代码合并：

	 $  git merge temp // 合并下载的temp分支到本地的master
	 合并以后 git branch查看 就有两个分支了
	   dev
		* master // 当前分支


	5删除temp分支
	如果temp分支不想要保留，可以使用如下命令删除该分支：

	$ git branch -d temp  // 删除分支     

	如果该分支的代码之前没有merge到本地，那么删除该分支会报错，可以使用git branch -D temp强制删除该分支。

	6 git add .
		git commit -m 'add'
		git push -u origin master // 第一次提交 设置提交当前分支到远程主机origin的master分支 并设置以后默认提交到origin master分支
		// 或者这样提交也可以
		git push --set-upstream origin master
#
#
#
#
#
#

#
#
#
#
#

#
#
#
#
#

#
#
#
#
#

#
#
#
#
#

#
#
#
#
#

#
#
#
#
#

#
#
#
#
#

#
#
#
#
#



