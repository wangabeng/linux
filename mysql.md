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
