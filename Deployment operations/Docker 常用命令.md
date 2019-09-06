
| 命令 | 解释 |
| --- | --- |
| docker pull [选项] [Docker Registry 地址[:端口号]/]仓库名[:标签] | 拉取一个镜像 |
|docker run -it --rm  ubuntu:16.04  bash  | 启动一个容器,结束后将其删除,用于临时调试 |
|docker run -it   ubuntu:16.04  bash  | 启动一个容器,结束后不删除 |
| exit | 退出当前容器 |
| docker image rm [选项] <镜像1> [<镜像2> ...] ,或docker rmi <镜像1> [<镜像2> ...] | 删除镜像 |
| docker rm <容器1>(空格) <容器2> <容器3>| 删除容器|
|docker exec -it 容器id bash|进入正在运行的容器|
|docker rmi $(docker images -q -f dangling=true)|删除所有虚悬镜像|
|docker stop $CONTAINER_ID|停止一个运行中的容器|
|docker run -p 8080:8080 tomcat|运行tomcat |
| docker cp mysql:/etc/mysql . | 将mysql容器中的/etc/mysql文件夹复制到当前目录 | 


参数解释:
* -it：这是两个参数，一个是 -i：交互式操作，一个是 -t 终端。我们这里打算进入 bash 执行一些命令并查看返回结果，因此我们需要交互式终端。
* --rm：这个参数是说容器退出后随之将其删除。默认情况下，为了排障需求，退出的容器并不会立即删除，除非手动 docker rm。我们这里只是随便执行个命令，看看结果，不需要排障和保留结果，因此使用 --rm 可以避免浪费空间。
* ubuntu:16.04：这是指用 ubuntu:16.04 镜像为基础来启动容器。
* bash：放在镜像名后的是命令，这里我们希望有个交互式 Shell，因此用的是 bash。

