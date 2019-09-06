
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


