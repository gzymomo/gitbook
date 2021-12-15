[TOC]

# 备份Docker数据

```bash
#!/bin/bash
DATE=`date +%Y%m%d%H%M`                #every minute
DATABASE=table1,table2     #database name
DB_USERNAME=root                       #database username
DB_PASSWORD=123456                   #database password
BACKUP_PATH=/home/backup          #backup pathhome/backup
mysqldump -uroot -p123456 table1 > /home/backup/table1_${DATE}.sql
mysqldump -uroot -p123456 table2 > /home/backup/table2_${DATE}.sql
find $BACKUP_PATH -type f -mtime +14 -name "*.sql" -exec rm -rf {} \;
```

# grant用户权限
grant 权限 on 数据库对象 to 用户

```mysql
mysql> grant all privileges on *.* to 'yangxin'@'%' identified by 'yangxin123456' with grant option; 

grant select, insert, update, delete on testdb.* to common_user@'%'

# grant 普通 DBA 管理某个 MySQL 数据库的权限。
grant all privileges on testdb to dba@'localhost'

其中，关键字 “privileges” 可以省略。

# grant 高级 DBA 管理 MySQL 中所有数据库的权限。
grant all on *.* to dba@'localhost'


```

- all privileges：表示将所有权限授予给用户。也可指定具体的权限，如：SELECT、CREATE、DROP等。
- on：表示这些权限对哪些数据库和表生效，格式：数据库名.表名，这里写“*”表示所有数据库，所有表。如果我要指定将权限应用到test库的user表中，可以这么写：test.user
- to：将权限授予哪个用户。格式：”用户名”@”登录IP或域名”。%表示没有限制，在任何主机都可以登录。比如：”yangxin”@”192.168.0.%”，表示yangxin这个用户只能在192.168.0IP段登录
- identified by：指定用户的登录密码
- with grant option：表示允许用户将自己的权限授权给其它用户

可以使用GRANT给用户添加权限，权限会自动叠加，不会覆盖之前授予的权限，比如你先给用户添加一个SELECT权限，后来又给用户添加了一个INSERT权限，那么该用户就同时拥有了SELECT和INSERT权限。

## 授予数据库权限时，<权限类型>可以指定为以下值：
SELECT：表示授予用户可以使用 SELECT 语句访问特定数据库中所有表和视图的权限。
INSERT：表示授予用户可以使用 INSERT 语句向特定数据库中所有表添加数据行的权限。
DELETE：表示授予用户可以使用 DELETE 语句删除特定数据库中所有表的数据行的权限。
UPDATE：表示授予用户可以使用 UPDATE 语句更新特定数据库中所有数据表的值的权限。
REFERENCES：表示授予用户可以创建指向特定的数据库中的表外键的权限。
CREATE：表示授权用户可以使用 CREATE TABLE 语句在特定数据库中创建新表的权限。
ALTER：表示授予用户可以使用 ALTER TABLE 语句修改特定数据库中所有数据表的权限。
SHOW VIEW：表示授予用户可以查看特定数据库中已有视图的视图定义的权限。
CREATE ROUTINE：表示授予用户可以为特定的数据库创建存储过程和存储函数的权限。
ALTER ROUTINE：表示授予用户可以更新和删除数据库中已有的存储过程和存储函数的权限。
INDEX：表示授予用户可以在特定数据库中的所有数据表上定义和删除索引的权限。
DROP：表示授予用户可以删除特定数据库中所有表和视图的权限。
CREATE TEMPORARY TABLES：表示授予用户可以在特定数据库中创建临时表的权限。
CREATE VIEW：表示授予用户可以在特定数据库中创建新的视图的权限。
EXECUTE ROUTINE：表示授予用户可以调用特定数据库的存储过程和存储函数的权限。
LOCK TABLES：表示授予用户可以锁定特定数据库的已有数据表的权限。
ALL 或 ALL PRIVILEGES：表示以上所有权限。

## 授予表权限时，<权限类型>可以指定为以下值：
SELECT：授予用户可以使用 SELECT 语句进行访问特定表的权限。
INSERT：授予用户可以使用 INSERT 语句向一个特定表中添加数据行的权限。
DELETE：授予用户可以使用 DELETE 语句从一个特定表中删除数据行的权限。
DROP：授予用户可以删除数据表的权限。
UPDATE：授予用户可以使用 UPDATE 语句更新特定数据表的权限。
ALTER：授予用户可以使用 ALTER TABLE 语句修改数据表的权限。
REFERENCES：授予用户可以创建一个外键来参照特定数据表的权限。
CREATE：授予用户可以使用特定的名字创建一个数据表的权限。
INDEX：授予用户可以在表上定义索引的权限。
ALL 或 ALL PRIVILEGES：所有的权限名。

## 授予列权限时，<权限类型>的值只能指定为 SELECT、INSERT 和 UPDATE，同时权限的后面需要加上列名列表 column-list

## 刷新权限
对用户做了权限变更之后，一定记得重新加载一下权限，将权限信息从内存中写入数据库。

` mysql> flush privileges; `

## 查看用户权限
```mysql
mysql> grant select,create,drop,update,alter on *.* to 'yangxin'@'localhost' identified by 'yangxin0917' with grant option;
# 查看其他 MySQL 用户权限：
mysql> show grants for 'yangxin'@'localhost';

# 查看当前用户（自己）权限：
show grants;
```

# 修改密码
## 更新mysql.user表
```mysql
mysql> update user set authentication_string=password('123456') where user='root';
mysql> flush privileges;
```

## 用set password命令
语法：set password for ‘用户名’@’登录地址’=password(‘密码’)

` mysql> set password for 'root'@'localhost'=password('123456'); `

## mysqladmin
语法：mysqladmin -u用户名 -p旧的密码 password 新密码

` mysql> mysqladmin -uroot -p123456 password 1234abcd `

注意：mysqladmin位于mysql安装目录的bin目录下。

