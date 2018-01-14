# linux
linux相关

# centOS7 中安装nginx
  详见 https://www.cnblogs.com/Xiao-Bing/p/6722322.html
1.添加Nginx到YUM源

添加CentOS 7 Nginx yum资源库,打开终端,使用以下命令:

sudo rpm -Uvh http://nginx.org/packages/centos/7/noarch/RPMS/nginx-release-centos-7-0.el7.ngx.noarch.rpm

2.安装Nginx

在你的CentOS 7 服务器中使用yum命令从Nginx源服务器中获取来安装Nginx：

sudo yum install -y nginx

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

