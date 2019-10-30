#### 文件夹的功能
- home    所有用户文件
- etc    配置文件目录
- usr/local    我们安装的目录必须放置在此目录
- var    存放数据库的目录(例如数据库的db)
#### Ubuntu和CentOS的区别
CentOS 有公司背景,稳定,更新非常慢,全球顶尖黑客做出来的
Ubuntu 社区产物,一个月一个版本,6个月1个稳定版

企业较多选择CentOS 
#### Ubuntu安装注意事项
- 分配硬盘空间的时候,要选择带有LVM的选项.LVM:硬盘扩容技术,增加一块硬盘之后,可以把原来的文件夹扩容
- 要选用云服务器一致的linux版本,以防出现bug,无法定位问题
#### ubuntu静态网络配置
```
# This file is generated from information provided by
# the datasource.  Changes to it will not persist across an instance.
# To disable cloud-init's network configuration capabilities, write a file
# /etc/cloud/cloud.cfg.d/99-disable-network-config.cfg with the following:
# network: {config: disabled}
network:
    ethernets:
        ens33:
            addresses: [192.168.229.199/24]
            dhcp4: no
            dhcp6: no
            gateway4:  192.168.229.2
            optional: true
            nameservers:
                addresses: [8.8.8.8, 9.9.9.9]
    version: 2
```
#### Ubuntu启用Root远程登录

在实际生产操作中，我们基本上都是使用超级管理员账户操作 Linux 系统，也就是 Root 用户，Linux 系统默认是关闭 Root 账户的，我们需要为 Root 用户设置一个初始密码以方便我们使用。
- 设置 Root 账户密码
```
sudo passwd root
```
- 切换到 Root
```
su
```
- 设置允许远程登录 Root
```
nano /etc/ssh/sshd_config

# Authentication:
LoginGraceTime 120
#PermitRootLogin without-password     //注释此行
PermitRootLogin yes                             //加入此行
StrictModes yes

重启服务
service ssh restart
```
#### Ubuntu更换阿里软件源
修改`/etc/apt/`目录下的`sources.list`文件,先备份原来的文件,再新建同名文件,内容以下
```
deb http://mirrors.aliyun.com/ubuntu/ bionic main restricted universe multiverse
deb-src http://mirrors.aliyun.com/ubuntu/ bionic main restricted universe multiverse
deb http://mirrors.aliyun.com/ubuntu/ bionic-security main restricted universe multiverse
deb-src http://mirrors.aliyun.com/ubuntu/ bionic-security main restricted universe multiverse
deb http://mirrors.aliyun.com/ubuntu/ bionic-updates main restricted universe multiverse
deb-src http://mirrors.aliyun.com/ubuntu/ bionic-updates main restricted universe multiverse
deb http://mirrors.aliyun.com/ubuntu/ bionic-backports main restricted universe multiverse
deb-src http://mirrors.aliyun.com/ubuntu/ bionic-backports main restricted universe multiverse
deb http://mirrors.aliyun.com/ubuntu/ bionic-proposed main restricted universe multiverse
deb-src http://mirrors.aliyun.com/ubuntu/ bionic-proposed main restricted universe multiverse
```
更新软件
```
apt-get update
```
连接软件源为`xxx.aliyun.xxx`则表示更换成功

#### CentOS 使用scp命令传送文件

要先安装`yum -y install openssh-clients`

 本地复制到远程方式：  `scp /home/1.jpg root@192.168.0.200:/home/`

 将远程文件复制到本地 命令格式（这里只说一种）：  `scp root@192.168.0.200:/home/1.jpg /home/`

#### CentOS 查看端口占用

查詢端口占用 `netstat -nltp|grep 80`