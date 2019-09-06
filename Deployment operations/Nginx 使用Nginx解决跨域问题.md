### 前言
在使用`Nginx`搭建`伪CDN`后,发现字体静态资源无法加载,其他的资源能加载.

### 使Nginx的静态资源能够跨域访问
需要在`nginx.conf`配置文件中增加以下配置
```
                add_header Access-Control-Allow-Origin  *;
                add_header Access-Control-Allow-Headers X-Requested-With;
                add_header Access-Control-Allow-Methods GET,POST,PUT,DELETE,PATCH,OPTIONS;
```
完整配置如下
```
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
        server_name  192.168.229.131;
	    # 所有的请求都以 / 开始，所有的请求都可以匹配此 location
        location / {
                add_header Access-Control-Allow-Origin  *;
                add_header Access-Control-Allow-Headers X-Requested-With;
                add_header Access-Control-Allow-Methods GET,POST,PUT,DELETE,PATCH,OPTIONS;
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