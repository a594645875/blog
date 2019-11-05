```shell
1. lvs主机(192.168.211.11) 设置虚拟ip(vip):
ifconfig eth0:2 192.168.211.100/24
(删除vip ifconfig eth0:2 down)


2. 修改real server的网络配置(192.168.211.12,192.168.211.13):
echo "1" >/proc/sys/net/ipv4/conf/eth0/arp_ignore 
echo "2" >/proc/sys/net/ipv4/conf/eth0/arp_announce 
echo "1" >/proc/sys/net/ipv4/conf/all/arp_ignore 
echo "2" >/proc/sys/net/ipv4/conf/all/arp_announce 
ifconfig lo:8 192.168.211.100 netmask 255.255.255.255

3. 模拟启动服务real server:
yum install httpd -y
vi /var/www/html/index.html(写上from 192.168.211.12)
service httpd start
(浏览器访问192.168.211.12,测试
如果测试404,可以service firewalld status或者service iptables status,查看防火墙是否开启
如果有开启,则service iptables stop关闭防火墙(练习情况),或者配置对应的策略(生产情况))

4. lvs主机:
yum install ipvsadm -y
ipvsadm -A -t 192.168.211.100:80 -s rr 					  # 设置监控的包 rr:轮询
ipvsadm -a -t 192.168.211.100:80 -r 192.168.211.12:80 -g 	# 添加负载的列表
ipvsadm -ln 											# 查看lvs的状态
ipvsadm -C												# 清空配置

5. 浏览器访问`192.168.211.100`测试,成功轮询访问`192.168.211.12`,`192.168.211.13`
```

