# Nginx搭建CDN服务器
参考文献:[Nginx 搭建伪 CDN 服务器](https://www.funtl.com/zh/spring-cloud-itoken-codeing/Nginx-%E7%AE%80%E4%BB%8B.html)
#### 1.搭建Nginx
在`/usr/local/docker/nginx`新建`docker-compose.yml`
```docker
version: '3.1'
services:
  nginx:
    restart: always
    image: nginx
    container_name: nginx
    ports:
      - 81:80
    volumes:
      - ./conf/nginx.conf:/etc/nginx/nginx.conf
      - ./wwwroot:/usr/share/nginx/wwwroot
```
#### 2.配置Nginx
在文件夹`/usr/local/docker/nginx/conf`新建`nginx.conf`
```nginx
worker_processes  1;

events {
    worker_connections  1024;
}

http {
    include       mime.types;
    default_type  application/octet-stream;

    sendfile        on;

    keepalive_timeout  65;
    # 配置虚拟主机 192.168.75.145
    server {
	    # 监听的ip和端口，配置 192.168.75.145:80
        listen       80;
	    # 虚拟主机名称这里配置ip地址
        server_name  192.168.75.145;
	    # 所有的请求都以 / 开始，所有的请求都可以匹配此 location
        location / {
	    # 使用 root 指令指定虚拟主机目录即网页存放目录
	    # 比如访问 http://ip/index.html 将找到                     /usr/local/docker/nginx/wwwroot/html80/index.html
	    # 比如访问 http://ip/item/index.html 将找到 /usr/local/docker/nginx/wwwroot/html80/item/index.html

            root   /usr/share/nginx/wwwroot/cdn;
	        # 指定欢迎页面，按从左到右顺序查找
            index  index.html index.htm;
        }

    }
}
```
#### 2.运行测试
在`/usr/local/docker/nginx/wwwroot/cdn`路径下添加文件
```
# touch 1.zip
```
在`/usr/local/docker/nginx`路径下运行
```
# docker-compose up -d
```
在浏览器中访问`ip:81/1.zip`,下载`1.zip`成功则表明搭建文件服务器成功


