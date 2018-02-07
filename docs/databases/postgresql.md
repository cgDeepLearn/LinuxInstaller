# Ubuntu16.04安装和配置PostgreSQL

## 1. 安装postgreSQL

```bash
sudo apt-get update
sudo apt-get install postgresql(指定版本使用---> sudo apt-get install postgresql-9.5)
```

在Ubuntu下安装Postgresql后，会自动注册为服务，并随操作系统自动启动。

在Ubuntu下安装Postgresql后，会自动添加一个名为postgres的操作系统用户，密码是随机的。并且会自动生成一个名字为postgres的数据库，用户名也为postgres，密码也是随机的。

## 2. 修改postgres数据库用户的密码为XXXXXX(你认为安全的密码)

`psql`进入数据库：

```bash
sudo -u postgres psql
```

其中，sudo -u postgres 是使用postgres 用户登录的意思

修改密码：

```sql
postgres=# ALTER USER postgres WITH PASSWORD 'mypwd'; 
```

`postgres=#`为`PostgreSQL`下的命令提示符，--注意最后的分号；`mypwd`更换为自己的密码

退出数据库：

```sql
postgres=# \q
```

## 3. 修改ubuntu操作系统的postgres用户的密码（密码要与数据库用户postgres的密码相同）

切换到root用户：
```

su root
```

删除postgresql用户密码

```
sudo passwd -d postgres   //passwd -d 是清空指定用户密码的意思
```

设置postgresql系统用户的密码：

```
sudo -u postgres passwd
```

按照提示，输入两次新密码\
输入新的 UNIX 密码:\
重新输入新的 UNIX 密码:\
passwd：已成功更新密码

## 4.修改postgresql数据库配置实现远程访问

```bash postgresql.conf
sudo vim /etc/postgresql/9.5/main/postgresql.conf 
```

* 监听任何地址访问，修改连接权限

```yml
#listen_addresses = 'localhost' 改为 listen_addresses = '*'
```

* 启用密码验证

```yml
#password_encryption = on 改为 password_encryption = on
```

```vim pg_hba.conf
vim /etc/postgresql/9.5/main/pg_hba.conf
```

在文档末尾加上一下内容：

```config
host all all 0.0.0.0 0.0.0.0 md5
```

## 5. 重启服务

```shell
sudo /etc/init.d/postgresql restart
或者：
sudo systemctl restart postgresql
```

## 6.5432端口的防火墙设置

`5432`为`postgreSQL`默认的端口

```shell
iptables -A INPUT -p tcp -m state --state NEW -m tcp --dport 5432 -j ACCEPT
```

## 7.创建管理数据库

* 登录postgre SQL数据库

psql -U postgres -h 127.0.0.1

* 创建新用户testzhangps，但不给建数据库的权限

```sql
postgres=# create user "test" with password 'pgpass' nocreatedb;
```

用户名处是双引号

* 建立数据库，并制定所有者

```sql
postgres=#create database "testdb" with owner = "test";
```

* 用创建的用户连接创建的数据库

```shell 
psql -U test -h localhost -d testdb
```






