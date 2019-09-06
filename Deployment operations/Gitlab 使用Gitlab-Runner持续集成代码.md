### 前言 

前面我们已经讲了如何搭建Gitlab-Runner持续集成平台. 
传送门: [使用Docker 搭建Gitlab-Runner持续集成平台](https://www.jianshu.com/p/3c1c13c51224)
本文以微服务配置项目itoken-config为例子, 介绍如何在代码中使用Gitlab-Runner进行代码的持续集成.
### 1. 在代码根目录`itoken-config`下创建`.gitlab-ci.yml`
```
stages:
  - package
  - push
  - run
  - clean

package:
  stage: package
  script:
  - /usr/local/maven/apache-maven-3.5.3/bin/mvn clean package
  - cp target/itoken-eureka-1.0.0-SNAPSHOT.jar docker
  - cd docker
  - docker build -t 192.168.229.130:5000/itoken-eureka .

push:
  stage: push
  script:
  - docker push 192.168.229.130:5000/itoken-eureka

run:
  stage: run
  script:
  - cd docker/config
  - docker-compose down
  - docker-compose up -d

clean:
  stage: clean
  script:
  - docker rmi $(docker images -q -f dangling=true)
```
`package`: 打包项目, 代码`push`后会出发`Runner`平台拉取代码,之后对`package`对代码进行打包,和构建镜像`192.168.229.130:5000/itoken-eureka`
`push`: 推送镜像, 把上一步构建好的镜像`192.168.229.130:5000/itoken-eureka`推送带`Registry`镜像私服, 192.168.229.130为笔者搭建的本地镜像私服地址.
`run`: 运行镜像,会触发`docker-compose`命令运行`itoken-config/docker`目录下的`docker-compose.yml`.
`clean`: 清除, 删除持续集成过程中产生的虚悬镜像.

### 2. 在`itoken-config/docker`目录下创建`Dockerfile`
`.gitlab-ci.yml`中`- package`构建镜像时需要的文件`Dockerfile`
```
FROM openjdk:8-jre

MAINTAINER Lusifer <topsale@vip.qq.com>

ENV APP_VERSION 1.0.0-SNAPSHOT

RUN mkdir /app

COPY itoken-config-$APP_VERSION.jar /app/app.jar
ENTRYPOINT ["java", "-Djava.security.egd=file:/dev/./urandom", "-jar", "/app/app.jar", "--spring.profiles.active=prod"]

EXPOSE 8888
```
如果该服务需要等待某个服务启动完成后再启动, 可以参考一下这个`Dockerfile`
```
FROM openjdk:8-jre

MAINTAINER Lusifer <topsale@vip.qq.com>

ENV APP_VERSION 1.0.0-SNAPSHOT
ENV DOCKERIZE_VERSION v0.6.1
# RUN wget https://github.com/jwilder/dockerize/releases/download/$DOCKERIZE_VERSION/dockerize-linux-amd64-$DOCKERIZE_VERSION.tar.gz \
# dockerize使服务等待其他服务启动完成后再启动,下载时间太长,使用自己的cdn代替网络地址
RUN wget http://192.168.229.133:81/dockerize-linux-amd64-v0.6.1.tar.gz  \
    && tar -C /usr/local/bin -xzvf dockerize-linux-amd64-$DOCKERIZE_VERSION.tar.gz \
    && rm dockerize-linux-amd64-$DOCKERIZE_VERSION.tar.gz

RUN mkdir /app
COPY itoken-eureka-$APP_VERSION.jar /app/app.jar
# dockerize等待http://192.168.229.133:8888/itoken-eureka/prod/master正常返回后再启动
ENTRYPOINT ["dockerize", "-timeout", "5m", "-wait", "http://192.168.229.133:8888/itoken-eureka/prod/master", "java", "-Djava.security.egd=file:/dev/./urandom", "-jar", "/app/app.jar", "--spring.profiles.active=prod"]

EXPOSE 8761
```
其中需要下载`dockerize`插件,由于每次下载都耗时过长, 所以笔者把该插件部署到自己的伪CDN服务器上,这样就可以每次秒下载了.
传送门: [Docker Nginx搭建伪CDN服务器](https://www.jianshu.com/p/56e8687243d)
### 2. 在`itoken-config/docker/config`目录下创建`docker-compose.yml` 
`.gitlab-ci.yml`中`- run`运行镜像时需要的文件`docker-compose.yml` 
因为docker-compose运行时会默认以文件所在的文件夹名称为项目名称,所以`docker-compose.yml`所在的文件夹名称要和其他项目区分开,以免在`down`和`up`的时候和其他项目有相关影响
```
version: '3.1'
services:
  itoken-config:
    restart: always
    image: 192.168.229.130:5000/itoken-config
    container_name: itoken-config
    ports:
      - 8888:8888
    networks:
    - config_network

networks:
  config_network:
```
### 3. `push`代码后,查看对应的Gitlab仓库的CI/CD
产生作业并完成,则持续集成代码部分完成. 
接着就可以写业务代码了, 每次`push`都会马上触发持续集成,也可以在注册`Runner`指定触发持续集成的`comment`, 例如`comment`是`deploy`才触发持续集成.


参考文献: [千锋教育-李卫民 使用 GitLab 持续集成](https://www.funtl.com/zh/spring-cloud-itoken-ci/%E4%BD%BF%E7%94%A8-GitLab-%E6%8C%81%E7%BB%AD%E9%9B%86%E6%88%90.html)