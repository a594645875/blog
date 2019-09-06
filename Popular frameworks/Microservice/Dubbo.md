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
### 高速序列化Kroy
1.在依赖中添加Kroy依赖
```
<!-- 高速序列化 -->
<dependency>    
    <groupId>de.javakaffee</groupId>    
    <artifactId>kryo-serializers</artifactId>    
    <version>0.42</version>
</dependency>
```
在`application.yml`中添加序列化方式
```
...
dubbo:
    protocol:
        # 高速序列化 kryo
        serialization: kryo
...
```

### 熔断器Hystrix 
1. 添加依赖
2. 启动类增加注解
3. 需要熔断的方法增加注解
- 服务提供者
```
//默认5秒20次,配置成2秒10次
@HystrixCommand(commandProperties = {        
@HystrixProperty(name = "circuitBreaker.requestVolumeThreshold", value = "10"),        
@HystrixProperty(name = "execution.isolation.thread.timeoutInMilliseconds", value = "2000")})

```
- 服务消费者
```
@HystrixCommand(fallbackMethod = "hiError")
...(需要熔断的方法)
//熔断后调用的方法
public String hiError() {    return "hi, Hystrix";}

```



