### 安装
- docker-compose
```
version: '3.1'
services:
  jenkins:
    restart: always
    image: jenkinsci/jenkins
    container_name: jenkins
    ports:
      # 发布端口
      - 8080:8080
      # 基于 JNLP 的 Jenkins 代理通过 TCP 端口 50000 与 Jenkins master 进行通信
      - 50000:50000
    environment:
      TZ: Asia/Shanghai
    volumes:
      - ./data:/var/jenkins_home
```
- 安装过程中会出现`Docker 数据卷`权限问题，用以下命令解决：
```
chown -R 1000 /usr/local/docker/jenkins/data
```
- 登录解锁
Jenkins 第一次启动时需要输入一个初始密码用以解锁安装流程，使用 docker logs jenkins 即可方便的查看到初始密码
- 选择自定义插件安装`select plugins to install`
- 插件选择
注意： 除了默认勾选的插件外，一定要勾选 `Publish over SSH` 插件，这是我们实现持续交付的重点插件。

### 基本配置
#### 配置本地化（显示中文）
- `Manage Jenkins` -> `Manage Plugins` -> `Available` 搜索`Local`并安装,然后重启Jenkins
- `Manage Jenkins` -> `Configure System`  
- `Manage Jenkins` -> `Global Tool Configuration` ->`Locale`  ->`Default Language` 填上`zh_CN`
- (勾上)Ignore browser preference and force this language to all users
- 保存
#### 配置JDK和Maven
- 上传 JDK 和 Maven 的 tar 包到服务器（容器数据卷目录`/data`）,并解压到当前目录(解压后tar包可删除)
- 新增JDK一栏填上
```
jdk1.8.0_152
/var/jenkins_home/jdk1.8.0_152
(去掉勾)Install automatically
```
- 新增Maven一栏填上
```
apache-maven-3.6.1
/var/jenkins_home/apache-maven-3.6.1
(去掉勾)Install automatically
```
- 点击`save`
### 安装动态参数插件
该插件的主要目的是为了方便我们后面在做项目构建时可以按照版本进行构建（支持一键回滚哦）
- `Manage Jenkins` -> `Manage Plugins` -> `Available` 搜索`Choice Parameter`并安装 `Extended Choice Parameter`,然后重启Jenkins
```
Extended Choice Parameter
Adds extended functionality to Choice parameter
0.78
```
### 配置GitLab SSH 免密登录
- 交互式进入 Jenkins 容器
```
docker exec -it jenkins /bin/bash
```
- 生成 SSH KEY
```
ssh-keygen -t rsa -C "your_email@example.com"
```
- 查看公钥
```
cat /var/jenkins_home/.ssh/id_rsa.pub
```
- 复制公钥到 GitLab,`添加公钥`
- `手动克隆一次项目`，该步骤的主要作用是为了生成和服务器的验证信息,输入`yes`
```
git clone ssh://git@192.168.229.136:2222/xxx/xxxxxx
```
![](https://www.funtl.com/assets/Lusifer_20181029040629.png)
会在 `.ssh/` 文件夹下生产`know hosts`
- 到Jenkins配置`Publish over SSH`
`系统管理` -> `系统设置` -> `Publish over SSH`
```
# name 随意
虚拟机环境-192.168.229.136

# Hostname
192.168.229.136

#Username
root

# Remote Directory
/usr/local/jenkins

(√)Use password ...
# Passphrase/Password
*******
```
- 其中`Remote Directory`是Jenkins可以操作的目录,需要`提前建好`.
- 点击`test xxx`,显示`success`表示和GitLab勾搭成功.然后保存
![](https://www.funtl.com/assets/Lusifer_20181029042803.png)
![](https://www.funtl.com/assets/Lusifer_20181029042948.png)
