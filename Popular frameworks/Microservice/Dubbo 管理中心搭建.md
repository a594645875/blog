### Dubbo管理中心搭建
1. 下载代码: `git clone https://github.com/apache/dubbo-admin.git`
2. 在` dubbo-admin-server/src/main/resources/application.properties`中指定注册中心地址
3. 构建
```
mvn clean package
```
4. 启动
```
mvn --projects dubbo-admin-server spring-boot:run
或者
cd dubbo-admin-distribution/target; java -jar dubbo-admin-0.1.jar
```
5. 访问 `http://localhost:8080`




