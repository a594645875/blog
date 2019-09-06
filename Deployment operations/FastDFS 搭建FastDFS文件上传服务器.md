### 1. 准备文件资料FastDFS.zip
- 链接链接：https://pan.baidu.com/s/1xtP4steVSQTI5xSMh3Zdgg 
- 提取码：si2c 
- 文件目录如下
```
└── FastDFS.zip
	├── client.conf
	├── config
	├── entrypoint.sh
	├── fastdfs-5.11.tar.gz
	├── fastdfs-nginx-module_v1.16.tar.gz
	├── libfastcommon.tar.gz
	├── mod_fastdfs.conf
	├── nginx-1.13.6.tar.gz
	├── nginx.conf
	├── storage.conf
	└── tracker.conf
 ```
 - 把下载的压缩解压后上传到虚拟机的新建目录`/usr/local/docker/fastdfs/environment`
 
 ### 2. 准备和编写配置
 - 在`/usr/local/docker/fastdfs`目录下新建`
docker-compose.yml`
```
version: '3.1'
services:
  fastdfs:
    build: environment
    restart: always
    container_name: fastdfs
    volumes:
      - ./storage:/fastdfs/storage
    network_mode: host
 ```
 - 在`
/usr/local/docker/fastdfs/environment`目录下新建`
Dockerfile`
```

FROM ubuntu:xenial
MAINTAINER topsale@vip.qq.com

# 更新数据源
WORKDIR /etc/apt
RUN echo 'deb http://mirrors.aliyun.com/ubuntu/ xenial main restricted universe multiverse' > sources.list
RUN echo 'deb http://mirrors.aliyun.com/ubuntu/ xenial-security main restricted universe multiverse' >> sources.list
RUN echo 'deb http://mirrors.aliyun.com/ubuntu/ xenial-updates main restricted universe multiverse' >> sources.list
RUN echo 'deb http://mirrors.aliyun.com/ubuntu/ xenial-backports main restricted universe multiverse' >> sources.list
RUN apt-get update

# 安装依赖
RUN apt-get install make gcc libpcre3-dev zlib1g-dev --assume-yes

# 复制工具包
ADD fastdfs-5.11.tar.gz /usr/local/src
ADD fastdfs-nginx-module_v1.16.tar.gz /usr/local/src
ADD libfastcommon.tar.gz /usr/local/src
ADD nginx-1.13.6.tar.gz /usr/local/src

# 安装 libfastcommon
WORKDIR /usr/local/src/libfastcommon
RUN ./make.sh && ./make.sh install

# 安装 FastDFS
WORKDIR /usr/local/src/fastdfs-5.11
RUN ./make.sh && ./make.sh install

# 配置 FastDFS 跟踪器
ADD tracker.conf /etc/fdfs
RUN mkdir -p /fastdfs/tracker

# 配置 FastDFS 存储
ADD storage.conf /etc/fdfs
RUN mkdir -p /fastdfs/storage

# 配置 FastDFS 客户端
ADD client.conf /etc/fdfs

# 配置 fastdfs-nginx-module
ADD config /usr/local/src/fastdfs-nginx-module/src

# FastDFS 与 Nginx 集成
WORKDIR /usr/local/src/nginx-1.13.6
RUN ./configure --add-module=/usr/local/src/fastdfs-nginx-module/src
RUN make && make install
ADD mod_fastdfs.conf /etc/fdfs

WORKDIR /usr/local/src/fastdfs-5.11/conf
RUN cp http.conf mime.types /etc/fdfs/

# 配置 Nginx
ADD nginx.conf /usr/local/nginx/conf

COPY entrypoint.sh /usr/local/bin/
ENTRYPOINT ["/usr/local/bin/entrypoint.sh"]

WORKDIR /
EXPOSE 8888
CMD ["/bin/bash"]
```
- 在`/usr/local/docker/fastdfs/environment`目录下给`entrypoint.sh`增加运行权限
```
chmod +x entrypoint.sh
```
- 修改`tracker.conf`其中的配置
```
base_path=/fastdfs/tracker
```
- 修改`storage.conf`其中的配置
```
base_path=/fastdfs/storage
store_path0=/fastdfs/storage
tracker_server=192.168.75.128:22122
http.server_port=8888
```
- 修改`client.conf`其中的配置
```
base_path=/fastdfs/tracker
tracker_server=192.168.75.128:22122
```
- 修改`config`其中的配置
```
# 修改前
CORE_INCS="$CORE_INCS /usr/local/include/fastdfs /usr/local/include/fastcommon/"
CORE_LIBS="$CORE_LIBS -L/usr/local/lib -lfastcommon -lfdfsclient"

# 修改后
CORE_INCS="$CORE_INCS /usr/include/fastdfs /usr/include/fastcommon/"
CORE_LIBS="$CORE_LIBS -L/usr/lib -lfastcommon -lfdfsclient"
```
- 修改`mod_fastdfs.conf`其中的配置
```
connect_timeout=10
tracker_server=192.168.75.128:22122
url_have_group_name = true
store_path0=/fastdfs/storage
```
- 准备完成最终的目录结构
```
└── fastdfs
    ├── docker-compose.yml
    └── environment
        ├── client.conf
        ├── config
        ├── Dockerfile
        ├── entrypoint.sh
        ├── fastdfs-5.11.tar.gz
        ├── fastdfs-nginx-module_v1.16.tar.gz
        ├── libfastcommon.tar.gz
        ├── mod_fastdfs.conf
        ├── nginx-1.13.6.tar.gz
        ├── nginx.conf
        ├── storage.conf
        └── tracker.conf
```
### 3. 构建镜像,运行
- 在`/usr/local/docker/fastdfs`目录下构建镜像
```
# 构建镜像
docker-compose build

# 运行
docker-compose up -d
```
### 4. 测试上传
- 以交互的方式进入容器
```
docker exec -it fastdfs /bin/bash
```
- 测试文件上传
```
/usr/bin/fdfs_upload_file /etc/fdfs/client.conf /usr/local/src/fastdfs-5.11/INSTALL
```
- 服务器反馈的文件地址
```
group1/M00/00/00/wKhLi1oHVMCAT2vrAAAeSwu9TgM3976771
```
- 测试 Nginx 访问
```
http://192.168.75.128:8888/group1/M00/00/00/wKhLi1oHVMCAT2vrAAAeSwu9TgM3976771
```
文件下载成功,则表示FastFDS搭建成功.