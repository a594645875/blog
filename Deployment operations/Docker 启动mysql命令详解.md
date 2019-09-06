
| 命令 | 解释 |
| --- | --- |
| docker run -p 3306:3306 --name mysql \ | 启动mysql容器,映射端口,换行 |
| -v /usr/local/docker/mysql/conf:/etc/mysql \ | 设置配置文件数据卷(映射配置目录) |
| -v /usr/local/docker/mysql/logs:/var/log/mysql \ | 设置日志文件数据卷 |
| -v /usr/local/docker/mysql/data:/var/lib/mysql \ | 设置数据文件数据卷 |
| -e MYSQL_ROOT_PASSWORD=root \ | 设置数据库访问root密码 |
| -d mysql | 镜像名称 |

docker run -p 3306:3306 --name mysql \
-v /usr/local/docker/mysql/conf:/etc/mysql \
-v /usr/local/docker/mysql/logs:/var/log/mysql \
-v /usr/local/docker/mysql/data:/var/lib/mysql \
-e MYSQL_ROOT_PASSWORD=root \
-d mysql:5.7.22