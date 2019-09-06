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



