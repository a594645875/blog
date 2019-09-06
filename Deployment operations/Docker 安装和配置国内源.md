### 国内镜像
编辑或新建`/etc/docker/daemon.json
```
{
"registry-mirrors": ["http://aad0405c.m.daocloud.io"],
  "insecure-registries": ["192.168.229.130:5000"]
}
```
启动命令
```
更新配置
systemctl daemon-reload
重启服务
systemctl restart docker

```
### Docker安装
- 第一种方法从Ubuntu的仓库直接下载安装：
安装比较简单，这种安装的Docker不是最新版本，不过对于学习够用了，依次执行下面命令进行安装。
```
$ sudo apt install docker.io
$ sudo systemctl start docker
$ sudo systemctl enable docker
```
查看是否安装成功
```
$ docker -v
Docker version 17.12.1-ce, build 7390fc6
```

