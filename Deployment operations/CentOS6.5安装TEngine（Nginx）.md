### 要点，先安装编译器、pcre库、openssl库

#### 1.安装编译器，输入命令

yum install gcc

#### 2.安装pcre库

yum  install pcre-devel

#### 3.安装openssl库

yum install openssl-devel

(快捷命令:`yum install gcc pcre-devel openssl-devel -y`)

### 正式安装

1.下载TEngine，tengine-1.5.2.tar.gz，放到目录/home/tengine下

2.输入命令tar zxvf tengine-2.1.0.tar.gz

3.进入解压目录tengine-2.1.0，输入命令`./configure --prefix=/home/tengine/tengine21 --with-http_ssl_module`  （指定安装目录、安装ssl模块）

4.编译安装，输入命令，

`make && make install`

5.启动，进入tengine15，输入命令./sbin/nginx -s start（停止用stop）

6.测试是否成功，浏览器输入http://localhost，或者http://IP（注意要开放80端口）

7.在`/etc/init.d`添加`nginx`文件,并赋予执行权限`chmod +x nginx`

```shell
#!/bin/bash
#
# nginx - this script starts and stops the nginx daemon
#
# chkconfig: - 85 15
# description: Nginx is an HTTP(S) server, HTTP(S) reverse
# proxy and IMAP/POP3 proxy server
# processname: nginx
# config: /etc/nginx/nginx.conf
# config: /etc/sysconfig/nginx
# pidfile: /var/run/nginx.pid
  
# Source function library.
. /etc/rc.d/init.d/functions
  
# Source networking configuration.
. /etc/sysconfig/network
  
# Check that networking is up.
[ "$NETWORKING" = "no" ] && exit 0
  
TENGINE_HOME="/usr/local/nginx/"
nginx=$TENGINE_HOME"sbin/nginx"
prog=$(basename $nginx)
  
NGINX_CONF_FILE=$TENGINE_HOME"conf/nginx.conf"
  
[ -f /etc/sysconfig/nginx ] && /etc/sysconfig/nginx
  
lockfile=/var/lock/subsys/nginx
  
start() {
    [ -x $nginx ] || exit 5
    [ -f $NGINX_CONF_FILE ] || exit 6
    echo -n $"Starting $prog: "
    daemon $nginx -c $NGINX_CONF_FILE
    retval=$?
    echo
    [ $retval -eq 0 ] && touch $lockfile
    return $retval
}
  
stop() {
    echo -n $"Stopping $prog: "
    killproc $prog -QUIT
    retval=$?
    echo
    [ $retval -eq 0 ] && rm -f $lockfile
    return $retval
    killall -9 nginx
}
  
restart() {
    configtest || return $?
    stop
    sleep 1
    start
}
  
reload() {
    configtest || return $?
    echo -n $"Reloading $prog: "
    killproc $nginx -HUP
    RETVAL=$?
    echo
}
  
force_reload() {
    restart
}
  
configtest() {
    $nginx -t -c $NGINX_CONF_FILE
}
  
rh_status() {
    status $prog
}
  
rh_status_q() {
    rh_status >/dev/null 2>&1
}
  
case "$1" in
start)
    rh_status_q && exit 0
    $1
;;
stop)
    rh_status_q || exit 0
    $1
;;
restart|configtest)
    $1
;;
reload)
    rh_status_q || exit 7
    $1
;;
force-reload)
    force_reload
;;
status)
    rh_status
;;
condrestart|try-restart)
    rh_status_q || exit 0
;;
*)
  
echo $"Usage: $0 {start|stop|status|restart|condrestart|try-restart|reload|force-reload|configtest}"
exit 2
esac
```

