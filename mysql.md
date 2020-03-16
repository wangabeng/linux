# mysql相关操作
## 登录：
```
mysql -uroot -p
输入空（默认密码为空 版本5.6.40 ）
```

## 查看所有数据库
```
SHOW DATABASES;
```

## 创建数据库[if not exists]可选
```
CREATE DATABASE IF NOT EXISTS bookstore;
```

## 删除数据库
```
DROP DATABASE IF EXISTS bookstore;
```

## 创建数据表
切换到数据库 然后创建表
auto_increment必须设置为主键
```
USE bookstore;
CREATE TABLE IF NOT EXISTS book(
    id INT NOT NULL AUTO_INCREMENT,
    bookname VARCHAR(50) NOT NULL DEFAULT '',
    PRIMARY KEY(id)
);
```

## 查看数据表
```
SHOW TABLES;
```

## 查看数据表详情
```
DESC book;
+----------+-------------+------+-----+---------+----------------+
| Field    | Type        | Null | Key | Default | Extra          |
+----------+-------------+------+-----+---------+----------------+
| id       | int(11)     | NO   | PRI | NULL    | auto_increment |
| bookname | varchar(50) | NO   |     |         |                |
+----------+-------------+------+-----+---------+----------------+
```

## 更改数据表结构或字段名称 类型等
```
ALTER TABLE book CHANGE bookname booksname VARCHAR(100);
```

## 更改数据表字符集（解决不能插入中文字符的问题）
ALTER TABLE book CONVERT TO CHARACTER SET utf8;

## 删除数据表
```
DROP TABLE book;
```

## 数据表内容管理
## 向mysql数据表插入行记录（增）
```
INSERT INTO book(id, booksname) values(100, 'javascript高程');
```

## 在mysql数据表里查询记录（查）
```
SELECT id, booksname FROM book;
+-----+------------------+
| id  | booksname        |
+-----+------------------+
| 123 | javascript       |
| 456 | javascript高程   |
+-----+------------------+
2 rows in set (0.00 sec)
```
## 更改数据表中的记录(where是查询)
```
UPDATE book SET id=100 WHERE id=123;
```
## 删除表中的记录
```
DELETE FROM book WHERE id=100;
```

# php操作mysql 通过mysqli连接数据库 
```
https://www.cnblogs.com/qiutianjia/p/5582632.html
```

## 解决mysql Navicat连接阿里云数据库 出错:1130-host . is not allowed to connect to this MySql server
因为外部计算机没有得到mysql的授权，解决办法：
例如，你想myuser使用mypassword从任何主机连接到mysql服务器的话。
```
GRANT ALL PRIVILEGES ON *.* TO 'myuser'@'%' IDENTIFIED BY 'mypassword' WITH GRANT OPTION;
```
以本人阿里云主机为例
```
GRANT ALL PRIVILEGES ON *.* TO 'root'@'%' IDENTIFIED BY '' WITH GRANT OPTION;
```
然后在在Navicat新建mysql连接 即可。


# 重新设置mysql密码 非最佳实践
set password for 'root'@'localhost' = password('528528');
也可以这样设置：
用root 进入mysql后
mysql>set password =password('你的密码');
mysql>flush privileges;

遇到的问题：  
在linux设置好root账户的mysql密码后，mynavicat仍然使用旧的密码，为什么？

# 重新设置mysql密码 最佳实践
https://blog.csdn.net/weixin_34221773/article/details/92434154
```
给用户设置密码后，无密码时可以登录，使用密码则不能登录。

试着删除空用户，然后刷新权限表就可以了。

mysql>delete from mysql.user where user='';

mysql>flush privileges;

所以从安全角度考虑，在Mysql安装好、启动后第一件事情就要设置密码，和删除空账户(切记)：
mysql>update mysql.user set password=password('yourpassword') where user='root';
mysql>delete from mysql.user where user='';
mysql>flush privileges;

转载于:https://my.oschina.net/yangphere/blog/103499
```
