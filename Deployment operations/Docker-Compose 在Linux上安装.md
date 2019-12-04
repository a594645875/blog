```
curl -L https://github.com/docker/compose/releases/download/1.17.1/docker-compose-`uname -s`-`uname -m` > /usr/local/bin/docker-compose
```

查看最新版本号,然后替换 https://github.com/docker/compose/releases
1.17.1替换成最新版本号1.25.0-rc1

```
curl -L https://github.com/docker/compose/releases/download/1.25.0-rc1/docker-compose-`uname -s`-`uname -m` > /usr/local/bin/docker-compose
```

加上运行权限
chmod +x /usr/local/bin/docker-compose

