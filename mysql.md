#mysql相关操作
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
CREAT TABLE IF NOT EXISTS book(
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
```
