## 1.识别域名

```nginx
   ...
   server {
        listen 80;
        server_name www.czc.com;
        location / {
                root /mnt;
                autoindex on;
        }
    }
    server {
        listen       80;
        server_name  www.360.com;
        location / {
            root   html;
            index  index.html index.htm;
        }
        error_page   500 502 503 504  /50x.html;
        location = /50x.html {
            root   html;
        }
    }
	...
```

设置两个服务器,监听`www.czc.com`和`www.360.com`80端口  
`www.czc.com`跳转到`/mnt`目录下,生成自动主页  
`www.360.com`跳转到`index.html`主页

## 2.反向代理

```nginx
    ...
    server {
        listen       80;
        server_name  www.360.com;
        location / {
            root   html;
            index  index.html index.htm;
        }
        error_page   500 502 503 504  /50x.html;
        location = /50x.html {
            root   html;
        }
        location /ooxx.go {
            proxy_pass http://192.168.211.12/;
        }	
    }
	...
```

访问`www.360.com/ooxx.go`的时候,会请求`http://192.168.211.12/`的资源  
如果把`http://192.168.211.12/`改为`http://www.baidu.com/`,会出现页面跳转到`https://www.baidu.com/`,页面地址也会改变,解决办法:把目标地址`http://www.baidu.com/`改成`httpx://www.baidu.com/`直接访问`https`,防止跳转

## 3.反向代理的负载均衡

```nginx
    ...
    keepalive_timeout  0;
    upstream bula {
    	server 192.168.211.12;
    	server 192.168.211.13;
    }
    server {
        listen 80;
        server_name www.czc.com;
        location / {
                root /mnt;
                autoindex on;
        }
    }
    server {
        listen       80;
        server_name  www.360.com;
        location / {
            root   html;
            index  index.html index.htm;
        }
        error_page   500 502 503 504  /50x.html;
        location = /50x.html {
            root   html;
        }
        location /ooxx.go {
            proxy_pass http://bula/;
        }	
    }
	...
```

`keepalive_timeout  0`不保持和单个服务器的连接  
访问`www.360.com/ooxx.go`的时候,会轮询请求`http://192.168.211.12/`和`http://192.168.211.13/`的资源