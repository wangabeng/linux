# mongodb部分
××××××××××××××××××××××××××××××××××××××××××××××××××××××××××××××××××

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



# 我现在遇到一个问题 如何把阿里云的数据库导出去 如果用git导出 不是不可以 但是数据库文件很大 git导出会很不方便

# MongoDB导入导出以及数据库备份
	https://www.cnblogs.com/qingtianyu2015/p/5968400.html
	例如我的database为
	runjie 
	集合有：
	aboutus
	case
	certification
	news
	service

	// 导出
	sudo mongoexport -d runjie -c aboutus -o /mongodb/aboutus.json --type json 
	例如：
	sudo mongoexport -d runjie -c users -o /home/python/Desktop/mongoDB/users.json --type json -f  "_id,user_id,user_name,age,status"


	语法：
		mongoexport -d dbname -c collectionname -o file --type json/csv -f field
		参数说明：
		    -d ：数据库名
		    -c ：collection名
		    -o ：输出的文件名
		    --type ： 输出的格式，默认为json
		    -f ：输出的字段，如果-type为csv，则需要加上-f "字段名"

	遇到问题：授权问题
	http://blog.csdn.net/lewistian/article/details/76693349?%3E

	// 导出成功 ( 127.0.0.1 如果换成公网IP就不成功 )
	sudo mongoexport -h 127.0.0.1 -p 27017 -u runjieAdmin -p 528528 -d runjie -c aboutus -o /mongodb/aboutus.json --type json

	// 数据导入 linux中导入成功
	sudo mongoimport -h 127.0.0.1 -p 27017 -u runjieAdmin -p 528528 -d runjie -c users --file /mongodb/aboutus.json --type json

	// window中导入成功
	mongoimport -h 127.0.0.1 -p 27017 -d runjie -c aboutus --file d:/aliyundata/aboutus.json --type json

	// 数据库之间导入导出
	mongoexport -h 47.104.182.26 -p 27017 -u runjieAdmin -p 528528 -d runjie -c aboutus -o d:/aboutus.json --type json

# node连接数据库mongodb
	使用 mongodb 遇到的db.collection is not a function 的问题 
	http://blog.csdn.net/qq_36370731/article/details/78963078
	解决办法：
	把mongodb改回以前的版本 我以前用的版本是2.2.29 然后重新安装 就成功了

	关于mongodb授权登录远程数据库的问题
	如果服务器不开启用户认证模式
	var url = "mongodb://47.104.182.26:27017/runjie"; // 就可以直接连接数据库
	
	如果服务器开启用户验证 则必须在连接的时候键入数据库密码方可登录
	var url = "mongodb://runjieAdmin:pass**word@47.104.182.26:27017/runjie"

