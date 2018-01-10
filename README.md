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


解决参照：
https://www.unix.com/linux/174307-errno-256-no-more-mirrors-try.html 
http://www.178linux.com/27073
http://www.voidcn.com/article/p-ofemgywu-kn.html

http://www.voidcn.com/article/p-rlzldoay-uw.html

解决方法，yum -y install ftp，即可
http://www.voidcn.com/article/p-oxxtlnel-rs.html
