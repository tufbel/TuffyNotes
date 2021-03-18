## 1 安装MySQL

### 1.1 apt存储库安装5.7

apt安装源中默认即为，可以直接使用安装命令安装MySQL5.7：

```powershell
sudo apt-get install mysql-server
```

### 1.2 yum安装



## 2 运行mysql

```powershell
#查看运行状态
sudo service mysql start
sudo service mysql status
```

## 3 设置账户

```mysql
mysql -uroot -p
# 输入密码

# 后续
mysql> select user,host from mysql.user;
+------------------+-----------+
| user             | host      |
+------------------+-----------+
| debian-sys-maint | localhost |
| mysql.session    | localhost |
| mysql.sys        | localhost |
| root             | localhost |
+------------------+-----------+

# 创建明为tuffy的可再任何电脑连接的超级账户
CREATE USER 'tuffy'@'%'
  IDENTIFIED BY 'sy14213ZM';
GRANT ALL
  ON *.*
  TO 'tuffy'@'%'
  WITH GRANT OPTION;
  
# 删除账户
DROP USER 'tuffy'@'%.example.com';

# 最后效果
mysql> select user,host from mysql.user;
+------------------+-----------+
| user             | host      |
+------------------+-----------+
| tuffy            | %         |
| debian-sys-maint | localhost |
| mysql.session    | localhost |
| mysql.sys        | localhost |
| root             | localhost |
+------------------+-----------+
```

## 4 测试效果

我选择DBeaver来测试（有些服务器记得提前去官网关闭防火墙）

<img src="https://cdn.jsdelivr.net/gh/tufbel/TImages/mark/20210121172517.png" alt="远程连接aliyun的数据库" style="zoom: 67%;" />

### 4.1 远程连接报错处理

第一次远程连接时会报错`Mysql "Lost connection to MySQL server at ‘reading initial communication packet', system error: 0` ，原因是mysql还不允许远程访问，这时需要对mysql进行设置。

修改文件`/etc/mysql/mysql.conf.d/mysqld.cnf`中内容，记得提前备份：

```powershell
# 找到bind-address，注释掉，或者将127.0.0.1改为0.0.0.0
bind-address = 127.0.0.1
```

修改完成后就可以远程连接了。
