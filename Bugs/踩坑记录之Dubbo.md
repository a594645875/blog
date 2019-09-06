#### zookeeper错误KeeperErrorCode = ConnectionLoss解决
- 描述：启动dubbo服务连接Zookeeper的时候，显示以下错误
```
java.io.IOException: java.net.UnknownHostException: 192.168.229.134:2181: invalid IPv6 address
...
... KeeperErrorCode = ConnectionLoss
```
- 分析：查看zookeeper版本和正常运行的demo版本一致,检查配置发现两个项目的配置不同

错误的配置:
```
registry:  id: zookeeper  address: 
zookeeper://192.168.229.134:2181?backup=192.168.229.134:2181:2182,192.168.229.134:2181:2183
```
正确的配置:
```
registry:  id: zookeeper  address: 
zookeeper://192.168.229.134:2181?backup=192.168.229.134:2182,192.168.229.134:2183
```
- 解决办法: 更改成正确的配置则正常连接.