
# Ubuntu 16.04 安装docker
1. 切换到root权限或者使用sudo(下列命令均使用)

2. 升级source列表并保证https和ca整数成功安装
```
# apt-get update
# apt-get install apt-transport-https ca-certificates
```

3. 增加新的GPG 密钥
```
# apt-key adv --keyserver hkp://p80.pool.sks-keyservers.net:80 --recv-keys 58118E89F3A912897C070ADBF76221572C52609D
```

4. 新增或编辑source列表里的docker.list文件
```
vim /etc/apt/sources.list.d/docker.list  //如果不存在就新增
```

5. 删除已有的entries(若存在)

6. 按照系统版本增加entry(Ubuntu Xenial 16.04 (LTS)）到.list文件中
```
deb https://apt.dockerproject.org/repo ubuntu-xenial main
```

7. 重新执行更新操作，并删除老的rep
```
# apt-get update
# apt-get purge lxc-docker  //没有安装的话，跳过
```

8. 查看是否有正确的可用版本
```
apt-cache policy docker-engine
```

9. 从14.04版本以上开始docker推荐安装linux-image-extra
```
# apt-get install linux-image-extra-$(uname -r)
```

10. 安装docker
```
# apt-get update
# apt-get install docker-engine
# service docker start 
# docker run hello-world
```


11. 安装docker-compose
```
curl -L https://github.com/docker/compose/releases/download/1.18.0/docker-compose-`uname -s`-`uname -m` > /usr/local/bin/docker-compose  # 此命令安装的1.18.0版本，可以将1.18.0换成你想要安装的版本
chmod +x /usr/local/bin/docker-compose
```
- 查看docker-compose version
```
docker-compose --version
```
- 删除docker-compose
```
rm /usr/local/bin/docker-compose
```

**有时curl会失败，使用pip安装也可以**

- 安装
```
pip install docker-compose
# docker_compose --version #查看
```
- 删除
```
pip uninstall docker-compose
```
**Have fun with doker!**
