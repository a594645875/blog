1. 修改`/etc/sysconfig/network-scripts/ifcfg-eth0`

   ```shell
   DEVICE=eth0
   TYPE=Ethernet
   ONBOOT=yes
   NM_CONTROLLED=yes
   BOOTPROTO=static
   IPADDR=192.168.206.10	#注意和NAT设置中的网段相同
   NETMASK=255.255.255.0
   GATEWAY=192.168.206.2	#和和NAT设置中的网段相同,最后为2
   DNS1=114.114.114.114
   DNS2=8.8.8.8
   ```

2. 删除`/etc/udev/rules.d/70-persistent-net.rules`

   因为克隆出来的服务器,网卡mac改变了,而该配置中仍然是旧的mac,所以需要重新生成新的`70-persistent-net.rules`就正确了

3. 重启,使新生成的配置文件生效,

   不行的话手动重启下网络试下`service network restart`()

