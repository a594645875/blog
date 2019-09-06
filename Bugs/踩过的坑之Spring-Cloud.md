#### Spring-Config1
场景:使用prod部署到服务器的zipkin一直没能注册到eureka，gitlab中的zipkin配置明明没有错误

错误分析:复制出`bootstrap-prod.yml`时，没有把里面的`spring.profiles.active=dev`修改为prod

解决办法:把`bootstrap-prod.yml`里的`spring.profiles.active=dev`修改为`prod`
#### springcloud config客户端无法获取云配置
问题:`pring cloud config`客户端一直无法获取云配置

分析:一开始以为是配置文件出现问题,经过多重检查,发现是依赖错了,

解决:把
```
<dependency>
<groupId>org.springframework.cloud</groupId>
<artifactId>spring-cloud-config</artifactId>
</dependency>
```
更换为:
```
<dependency>
<groupId>org.springframework.cloud</groupId>
<artifactId>spring-cloud-starter-config</artifactId>
</dependency>
```
就好了
#### Zuul无法生效
问题:zuul启动后, 输入`http://myzuul.com:9001/SPRINGCLOUD-PROVIDER-DEPT/dept/findAll`路由功能无法生效

分析:URL要区分大小写,应该用小写

解决:把url中的服务名称改成路由映射的名称,映射为全小写,把`http://myzuul.com:9001/SPRINGCLOUD-PROVIDER-DEPT/dept/findAll`改为`http://myzuul.com:9001/springcloud-provider-dept/dept/findAll`就可以了
#### 使用Feign的时候参数问题
问题:使用feign做负载均衡时,测试带参数的url总是报错`PathVariable annotation was empty on param 0`,不带参数的url顺利通过

分析:使用Feign的时候,如果参数中带有`@PathVariable`形式的参数,则要用value=""标明对应的参数,否则会抛出`IllegalStateException`异常

解决:在feign接口上的参数上添加valus,把`@PathVariable Long deptNo`改为`@PathVariable(value = "deptNo") Long deptNo`
#### maven项目中springcloud中的健康检查spring-boot-starter-actuator和jdbc之间形成循环依赖
报错:依赖循环
```
Description:

The dependencies of some of the beans in the application context form a cycle:

   servletEndpointRegistrar defined in class path resource [org/springframework/boot/actuate/autoconfigure/endpoint/web/ServletEndpointManagementContextConfiguration.class]
      ↓
   healthEndpoint defined in class path resource [org/springframework/boot/actuate/autoconfigure/health/HealthEndpointConfiguration.class]
      ↓
   org.springframework.boot.actuate.autoconfigure.jdbc.DataSourceHealthIndicatorAutoConfiguration
┌─────┐
|  dataSource
↑     ↓
|  scopedTarget.dataSource defined in class path resource [org/springframework/boot/autoconfigure/jdbc/DataSourceConfiguration$Hikari.class]
↑     ↓
|  org.springframework.boot.autoconfigure.jdbc.DataSourceInitializerInvoker
└─────┘
```
分析:SpringBoot2.0.1的bug,

办法:把
```
<parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>2.0.1.RELEASE</version>
</parent>
```
中的2.0.1.RELEASE,改成2.0.2.RELEASE即可解决
#### 报错:引用SpringCouldconfiguration的时候,启动服务报错
报错:引用`SpringCouldconfiguration`的时候,启动服务报错
```
Failed to auto-configure a DataSource: 'spring.datasource.url' is not specified and no embedded datasource could be auto-configured.
Reason: Failed to determine a suitable driver class
```

分析:没能启动云配置,缺少`spring-cloud-stater-config`依赖

解决办法:添加依赖
```
<dependency>
<groupId>org.springframework.cloud</groupId>
<artifactId>spring‐cloud‐starter‐config</artifactId>
</dependency>
```
#### Spring cloud测试报错: feign.codec.DecodeException
报错: 
```
feign.codec.DecodeException:
Type definition error:
[simple type, class entity.Result];
nested exception is com.fasterxml.jackson.databind.exc.InvalidDefinitionException:
Cannot construct instance of `entity.Result` (no Creators, like default construct, exist):
cannot deserialize from Object value (no delegate- or property-based Creator) at [Source: (PushbackInputStream); line: 1, column: 2]
```

分析: 由于pojo类Result没有写空参构造方法,导致无法构造Result实例

解决办法: 在Result类中添加空参构造方法
#### @EnableEurekaServer注解不可用
描述: @EnableEurekaServer一直报红,找不到包,
原来的版本控制器
```
<dependencyManagement>
    <dependencies>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring‐cloud‐dependencies</artifactId>
            <version>Finchley.M9</version>
            <type>pom</type>
            <scope>import</scope>
        </dependency>
    </dependencies>
</dependencyManagement>
```
依赖
```
<dependencies>
    <dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-netflix-eureka-server</artifactId>
    </dependency>
</dependencies>
```

问题分析: Eureka的版本和SpringBoot的版本不对应,
办法: 将版本控制器的版本`<version>Finchley.M9</version>`修改为`<version>Finchley.RELEASE</version>`







