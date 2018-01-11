# Ubuntu16.04安装、配置MySQL

### 1. `apt-get`安装数据库和依赖包
```bash
sudo apt-get install mysql-server
sudo apt-get install mysql-client
sudo apt-get install libmysqlclient-dev
```
安装过程会提示输入数据库密码，输入并记住密码

* 安装后查看是否安装成功
```
sudo netstat -tap |grep mysql
```
看到`mysqld`服务即说明安装成功

### 2. 设置允许远程访问(如果需要的话)

编辑`/etc/mysql/mysql.conf.d/mysqld.cnf`
```bash
sudo vim /etc/mysql/mysql.conf.d/mysqld.cnf
```
注释掉`bind-address = 127.0.0.1`
```
# bind-address = 127.0.0.1
```
保存退出

### 3. 创建`test`用户(或者其他你需要使用的名字)，并授权

以root用户登录():
```bash
mysql -u root -p
```
输入之前设置的`root`用户的密码

创建`test`用户,`test`为用户,`testpass`为密码
```sql
mysql> CREATE USER 'test'@'localhost' IDENTIFIED BY 'testpass'; 
```
或者使用:
```sql
mysql> insert into mysql.user(Host,User,Password) values("localhost","test",password("testpass"));
```
这样就创建了一个用户名为test,密码为testpass的用户

退出登录，用`test`用户登录测试一下:
```sql
mysql> exit;
$ mysql -u test -p
mysql>
```
以`root`用户登录,然后为用户创建一个数据库(testdb):
```sql
mysql> create database testdb;
```
授权test用户拥有testdb数据库的所有权限:
```sql
mysql> grant all privileges on testdb.* to test@localhost identified by 'testpass';
mysql> flush privileges; //刷新系统权限表
```
或只指定部分权限给test用户，可以这样写：
```sql
mysql>grant select,delete,update,create,drop on *.* to test@"%" identified by "testpass";
```
test用户对所有数据库都有select,delete,update,create,drop 权限。
@"%" 表示对所有非本地主机授权，不包括localhost。（localhost地址设为127.0.0.1，如果设为真实的本地地址，不知道是否可以，没有验证。）

### 4. 删除用户或数据库

```sql
$ mysql -u root -p
mysql> Delete FROM user Where User='test' and Host='localhost';
mysql> flush privileges;
mysql> drop database testdb;//删除用户的数据库
```
删除账户及权限：
```sql
mysql> drop user test@'%';
mysql> drop user test@localhost;
```

### 5. 修改指定用户的密码
```sql
mysql -u root -p
mysql> update mysql.user set password=password('新密码') where User="test" and Host="localhost";
mysql>flush privileges;
```