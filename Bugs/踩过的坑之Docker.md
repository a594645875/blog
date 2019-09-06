#### 容器时间与宿主机时间不一致
- 问题: 部署在`docke`容器中的软件没有按时运行
- 分析: 使用以下命令之一
```
date
timedatectl status
```
查到`docker`创建的容器内时间和宿主机的不一致
- 解决: 在`Dockerfile`中添加容器时区设置的配置
```
......
FROM tomcat
ENV CATALINA_HOME /usr/local/tomcat
.......
#设置时区
RUN /bin/cp /usr/share/zoneinfo/Asia/Shanghai /etc/localtime && echo 'Asia/Shanghai' >/etc/timezone
......
```
#### 构建镜像时,dockerfile: returned a non-zero code: 100
在写Dockerfile时遇到returned a non-zero code: 100，报这个错误是因为在install之前没有update
```
RUN  apt-get install -y build-essential libpq-dev nodejs
```
修改为：
```
RUN apt-get update && apt-get install -y build-essential libpq-dev nodejs
```
就可以啦
#### docker server gave HTTP response to HTTPS client
- 1. 在Push一个镜像到本地的registry时，报错：
```
#docker push 192.168.163.131:5000/test
The push refers to a repository [192.168.163.131:5000/test]
Get https://192.168.163.131:5000/v1/_ping: http: server gave HTTP response to HTTPS client
```
原因：对应的daemon.json没有写上正确的镜像私服地址
解决: 在`/etc/docker/daemon.json`添加以下代码
```
{
"registry-mirrors": ["http://aad0405c.m.daocloud.io"],
  "insecure-registries": ["192.168.229.130:5000"]
}
```
重启
```
更新配置
systemctl daemon-reload
重启服务
systemctl restart docker
```
- 2. 在构建`Gitlab-runner`的时候,参考资料的私服设置在Docker安装完之后才复制到指定位置,导致配置无法生效
```
...
# 安装 Docker,先复制daemon.json,才能生效
RUN apt-get update && apt-get -y install apt-transport-https ca-certificates curl software-properties-common && \
    curl -fsSL http://mirrors.aliyun.com/docker-ce/linux/ubuntu/gpg | apt-key add - && \
    add-apt-repository "deb [arch=amd64] http://mirrors.aliyun.com/docker-ce/linux/ubuntu $(lsb_release -cs) stable" && \
    apt-get update -y && \
    apt-get install -y docker-ce
COPY daemon.json /etc/docker/daemon.json
...
```
- 解决办法: 安装软件前完成配置的复制
```
...
# 安装 Docker,先复制daemon.json,才能生效
COPY daemon.json /etc/docker/daemon.json
RUN apt-get update && apt-get -y install apt-transport-https ca-certificates curl software-properties-common && \
    curl -fsSL http://mirrors.aliyun.com/docker-ce/linux/ubuntu/gpg | apt-key add - && \
    add-apt-repository "deb [arch=amd64] http://mirrors.aliyun.com/docker-ce/linux/ubuntu $(lsb_release -cs) stable" && \
    apt-get update -y && \
    apt-get install -y docker-ce
...
```
#### 修改ENTRYPOINT后,重新构建相同名称标签的镜像,一直还是原来的镜像
原来是构建镜像时,名称写错成已经打包好的其他项目的镜像
consumer写成provider