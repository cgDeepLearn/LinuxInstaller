# Ubuntu16.04安装、配置MongoDB

## 目录

- [安装](#安装)
  - 导入软件源公钥
  - 为`mongodb`创建软件源list文件
  - 更新软件源并安装`mongodb`
- [配置](#配置)
  - [开机启动](#开机启动)
  - service服务
  - 本地连接
  - 允许远程访问
- [创建管理员帐号与数据库](#创建管理员帐号与数据库)
  - 创建管理员帐号
  - 创建数据库
- [阿里云数据库MongoDB](#阿里云数据库)
  - 创建实例
  - 重置密码
  - 白名单配置
  - 网络设置
  - 连接云数据库

## 安装

### 导入软件源公钥

```shell
sudo apt-key adv --keyserver hkp://keyserver.ubuntu.com:80 --recv EA312927
```

### 为`mongodb`创建软件源list文件

ubuntu16.04:

```shell
echo "deb http://repo.mongodb.org/apt/ubuntu xenial/mongodb-org/3.4 multiverse" | sudo tee /etc/apt/sources.list.d/mongodb-org-3.4.list
```

其中`mongodb-org/3.4` 的`3.4` 为版本号，可更换为你想要安装的版本

### 更新软件源并安装`mongodb`

```shell
sudo apt-get update
sudo apt-get install -y mongodb-org
```

如果想安装指定的版本(例如3.2.9,上面命令请改成3.2)，使用下面命令：

```shell
sudo apt-get install -y mongodb-org=3.2.9 mongodb-org-server=3.2.9 mongodb-org-shell=3.2.9 mongodb-org-mongos=3.2.9 mongodb-org-tools=3.2.9
```

##配置

### 开机启动

启动mongodb服务，默认安装后，是启动mongodb服务的
执行下面命令开启开机启动：

```shell
sudo systemctl enable mongod
```

运行后会有以下输出：

```text
Created symlink from /etc/systemd/system/multi-user.target.wants/mongod.service to /lib/systemd/system/mongod.service.
```

下面说说`mongod.service`

### 或者编写`mongod.service`文件

ubuntu16.04 由`init`切换到了`Systemd`(init缺点:启动时间长、启动脚本复杂)
使用了 `Systemd`，就不需要再用`init`了。`Systemd` 取代了`initd`，成为系统的第一个进程（PID 等于 1），其他进程都是它的子进程。

手动创建`/lib/systemd/system/mongod.service`文件，并写入下面内容(上面生成的其实和下面差不多):

```
[Unit]
Description=High-performance, schema-free document-oriented database
After=network.target
Documentation=https://docs.mongodb.org/manual

[Service]
User=mongodb
Group=mongodb
ExecStart=/usr/bin/mongod --quiet --config /etc/mongod.conf

[Install]
WantedBy=multi-user.target
```

### 启动、重启和关闭命令

```shell
sudo service mongod stop　　#停止服务
sudo service mongod start　　#启动服务
sudo service mongod restart #重新启动服务
sudo service mongod status #查看状态
```

### 本地连接

本机连接至mongodb服务，使用mongo命令连接:

```shell
mongo
```

输出：

```
ongoDB shell version v3.4.13
connecting to: mongodb://127.0.0.1:27017
MongoDB server version: 3.4.13
Welcome to the MongoDB shell.
For interactive help, type "help".
For more comprehensive documentation, see
	http://docs.mongodb.org/
Questions? Try the support group
	http://groups.google.com/group/mongodb-user
Server has startup warnings: 
2018-02-27T17:20:16.256+0800 I STORAGE  [initandlisten] 
2018-02-27T17:20:16.256+0800 I STORAGE  [initandlisten] ** WARNING: Using the XFS filesystem is strongly recommended with the WiredTiger storage engine
2018-02-27T17:20:16.256+0800 I STORAGE  [initandlisten] **          See http://dochub.mongodb.org/core/prodnotes-filesystem
2018-02-27T17:20:16.704+0800 I CONTROL  [initandlisten] 
2018-02-27T17:20:16.704+0800 I CONTROL  [initandlisten] ** WARNING: Access control is not enabled for the database.
2018-02-27T17:20:16.704+0800 I CONTROL  [initandlisten] **          Read and write access to data and configuration is unrestricted.
2018-02-27T17:20:16.704+0800 I CONTROL  [initandlisten] 

```

上面可以看到`WARNING: Access control is not enabled`，mongodb默认不需要用户密码这样不安全，我们在修改一下配置(mongodb配置文件在/etc/mongod.conf)：


- 打开编辑配置文件： sudo vim /etc/mongod.conf

```conf
# mongod.conf

# for documentation of all options, see:
#   http://docs.mongodb.org/manual/reference/configuration-options/

# Where and how to store data.
storage:
  dbPath: /var/lib/mongodb
  journal:
    enabled: true
#  engine:
#  mmapv1:
#  wiredTiger:

# where to write logging data.
systemLog:
  destination: file
  logAppend: true
  path: /var/log/mongodb/mongod.log

# network interfaces
net:
  port: 27017
  bindIp: 127.0.0.1


#processManagement:

#security:

#operationProfiling:

#replication:

#sharding:

## Enterprise-Only Options:

#auditLog:

#snmp:

```

- 修改security项为如下:

```diff
#security:
+ security:
+    authorization: enabled
```

- 修改完成后，保存文件，并重启mongod

### 允许远程访问

修改`mongod.conf`中bindIP:

```diff
net:
  port: 27017
- bindIp: 127.0.0.1
+ bindIp: 0.0.0.0

```

### shell连接

```shell
$> mongo --host host:port/db -u root -p
# 本地连接 --host host:port/db可直接改成要连接的数据库即可
# 安全连接请添加--authenticationDatabase(详细请查看下面创建账号)：
$> mongo db --authenticationDatabase admin -u root -p

```

## 创建管理员帐号与数据库

### 创建管理员帐号

用户管理员是第一个要创建的用户。在没有创建任何用户之前，你可以随意创建用户；但数据库中一旦有了用户，那么未登录的客户端就没有权限做任何操作了，除非使用db.auth(username, password)方法登录。
用户管理员的角色名叫 userAdminAnyDatabase，这个角色只能在 admin 数据库中创建。下面是一个例子：

```shell
> use admin
switched to db admin
> db.createUser({user:"root",pwd:"root123",roles:["userAdminAnyDatabase"]})
Successfully added user: { "user" : "root", "roles" : [ "userAdminAnyDatabase" ] }
这个例子创建了一个名为 root 的用户管理员。创建完了这个用户之后，我们应该马上以该用户的身份登录：
> db.auth("root","root123")
1
db.auth() 方法返回 1 表示登录成功。接下来我们为指定的数据库创建访问所需的账号。

退出后再用root连接admin或其他数据库：
$> mongo admin --authenticationDatabase admin -u root -p
```

### 创建数据库

首先保证你已经以用户管理员的身份登录 admin 数据库。然后用 use 命令切换到目标数据库，同样用 db.createUser() 命令来创建用户，其中角色名为 “readWrite”。
普通的数据库用户角色有两种，read 和 readWrite。顾名思义，前者只能读取数据不能修改，后者可以读取和修改。
下面是一个例子：

```shell
> use test
switched to db test
> db.createUser({user:"testuser",pwd:"testpass",roles:["readWrite"]})
Successfully added user: { "user" : "testuser", "roles" : [ "readWrite" ] }
> db.auth("testuser","testpass")
```

使用testuser登陆test数据库：

```shell
$> mongo test --authenticationDatabase admin -u testuser -p
# 输入password
```

## 阿里云数据库

### 创建实例

进入阿里云管理控制台 -->> 点击云数据库MongoDB版 -->> 选区然后新建实例 -->>点击实例ID进入基本信息

### 重置密码

点击基本信息下的账号管理旁的重置密码 -->> 重置密码
该密码为admin数据库下的拥有root权限的root账号的密码

### 网页登陆数据管理控制台

点击基本信息 -->> 实例关系下的Primary/Secondary登陆 -->> 输入数据库用户名(root)-数据库名(admin)-密码 -->> 登陆创建用户或者数据库等操作

### 白名单配置

点击数据安全性 -->> 添加你的ECS服务器的内网IP

这样你的内网ECS服务器就可以访问你的云数据库MongoDB了

若想外网也能访问，请点击 数据库连接 -->> 公网连接 -->> 申请公网地址

### 网络设置

确保ECS和云数据库MongoDB是同一个网络(经典网络或专有网络)

如果你的ECS为专有网络，你的MongoDB为经典网络，请点击 数据库连接 -->> 内网连接 -->> 右上角切换为专有网络 -->> 等待创建即可

### 连接云数据库

#### 使用Mongo Shell连接

```shell
$> mongo --host xxx.mongodb.rds.aliyuncs.com:3717 --authenticationDatabase admin -u root -p
# xxx为你的云数据库节点名称，有两个：primary(具有读写权限)和second(只读)
```

#### 客户端使用Connection String URI连接实例 （****部分替换为为root密码）：

```
mongodb://root:****@xxx.mongodb.rds.aliyuncs.com:3717/admi?replicaSet=mgset-4843525
```

副本集名称:replicaSet=mgset-4843525换成你的