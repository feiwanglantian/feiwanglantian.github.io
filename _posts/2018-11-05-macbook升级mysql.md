---
layout:     post
title:      mac升级mysql
subtitle:   mysql
date:       2018-11-05
author:     LIWC
header-img: img/post-bg-desk.jpg
catalog: true
tags:
    - mysql
---
## mac升级mysql
本文转载自http://www.mumayi.com/jiaocheng-20170704-12235.html，如有侵权，请联系删除。
### 1、首先停止Mysql服务
```
sudo /usr/local/mysql/support-files/mysql.server stop
```
### 2、然后下载你需要的 Mysql 安装包，并安装。安装好以后你文件会存储在mysql-5.7.24-macos10.14-x86_64，并且 mysql 的链接会指向同样的位置/usr/local/mysql，而你之前的数据库应在在同样的位置/usr/local/mysql-5.5.13-osx10.6-x86_64

### 3、现在我们要做的就是替换数据库文件  data 文件夹。 首先将新数据库文件夹改名
```
sudo mv /usr/local/mysql-5.7.24-macos10.14-x86_64/data /usr/local/mysql-5.7.24-macos10.14-x86_64/dataold
```
### 4、然后将老的数据库目录的数据库文件复制过去

```
sudo cp -rf /usr/local/mysql-5.5.47-osx10.8-x86_64/data /usr/local/mysql-5.7.24-macos10.14-x86_64/
```
### 5、然后设置正确的权限
```
sudo chown -R _mysql /usr/local/mysql-5.7.24-macos10.14-x86_64/data
```
### 6、启动Mysql 然后修复数据库
```
sudo /usr/local/mysql/support-files/mysql.server start
```
### 7、运行升级程序
```
/usr/local/mysql/bin/mysql_upgrade
```
### 8、如果出现错误就再运行一次，随后重启 Mysql 服务
```
sudo /usr/local/mysql/support-files/mysql.server restart
```
### 9、查看新的版本号
```
/usr/local/mysql/bin/mysql
```
### 10、如果需要，请重新设定root 密码
```
/usr/local/mysql/bin/mysqladmin -u root password 'yourpasswordhere'
```