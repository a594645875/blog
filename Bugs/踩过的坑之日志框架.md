#### slf4j没产生日志文件
- 问题描述:导入日志`slf4j-log4j12`日志依赖后,运行项目在设置的位置并没有产生配置好的日志文件
- 分析:因为`slf4j`门面框架只能绑定一个底层实现日志框架,`springboot-stater`已经含有`logback`了,与`log4j12`冲突,所以日志框架在选择底层的时候有可能选择`logback`底层框架,导致无法启用`log.properties`里的配置.
- 解决办法:在含有`logback`的pom依赖中,排除`logback`依赖
```
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter</artifactId>
    <exclusions>
        <exclusion>
            <groupId>ch.qos.logback</groupId>
            <artifactId>logback-core</artifactId>
        </exclusion>
        <exclusion>
            <groupId>ch.qos.logback</groupId>
            <artifactId>logback-classic</artifactId>
        </exclusion>
    </exclusions>
</dependency>
```
