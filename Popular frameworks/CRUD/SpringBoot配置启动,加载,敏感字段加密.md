## 1. SpringBoot启动配置装配过程

1. SpringBoot启动的时候加载主配置类，默认开启了自动配置功能 @EnableAutoConfiguration

2. **@EnableAutoConfiguration 作用**

   将类路径下 META-INF/spring.factories 里面配置的所有EnableAutoConfiguration的值（自动装配类）加入到了执行计划中。

3. 每一个自动配置类进行自动配置功能,

用到的一些注解：

```java
@Configuration
@EnableConfigurationProperties({HttpEncodingProperties.class}) 
@Bean

//当web容器类型是servlet的时候可以初始化本类为一个bean
@ConditionalOnWebApplication(type = Type.SERVLET)

//当有一个CharacterEncodingFilter的bean时可以初始化本类为一个bean
@ConditionalOnClass({CharacterEncodingFilter.class})

//当配置文件里面有如下配置的时候可以初始化本类为一个bean
@ConditionalOnProperty(
    prefix = "spring.http.encoding",
    value = {"enabled"},
    matchIfMissing = true
)

//当没有CharacterEncodingFilter初始化CharacterEncodingFilter的bean
@ConditionalOnMissingBean
```

## 2. 获取自定义配置（默认配置文件名）

- 使用@Value获取配置值

```java
@Data
@Component
public class Family {

    @Value("${family.family-name}")
    private String familyName;

}
```

- 使用@ConfigurationProperties获取配置值

```java
@Data
@Component
@ConfigurationProperties(prefix = "family")
public class Family {

    //@Value("${family.family-name}")
    private String familyName;
    private Father father;
    private Mother mother;
    private Child child;

}
```

比较

|                          | @ConfigurationProperties | @Value             |
| :----------------------- | :----------------------- | :----------------- |
| 功能                     | 批量注入属性到java类     | 一个个属性指定注入 |
| 松散语法绑定             | 支持                     | 不支持             |
| SpEL                     | 不支持                   | 支持               |
| 复杂数据类型(对象、数组) | 支持                     | 不支持             |
| JSR303数据校验           | 支持                     | 不支持             |

## 3. 加载外部配置文件（自定义配置文件名）

- #### 加载`family.properties`配置文件

```java
@Data
@Component
@ConfigurationProperties(prefix = "family")
@PropertySource(value = {"classpath:family.properties"})
@Validated
public class Family {
    private String familyName;
    private Father father;
    private Mother mother;
    private Child child;
}
```

- #### 加载`family.yml`配置文件

以为原生的`@PropertySource`是不支持YAML语法的,所以需要改造的默认factory屬性`PropertySourceFactory.class`

```java
public class MixPropertySourceFactory extends DefaultPropertySourceFactory {

  @Override
  public PropertySource<?> createPropertySource(String name, EncodedResource resource) throws IOException {
    String sourceName = name != null ? name : resource.getResource().getFilename();
    if (!resource.getResource().exists()) {
      return new PropertiesPropertySource(sourceName, new Properties());
    } else if (sourceName.endsWith(".yml") || sourceName.endsWith(".yaml")) {
      Properties propertiesFromYaml = loadYml(resource);
      return new PropertiesPropertySource(sourceName, propertiesFromYaml);
    } else {
      return super.createPropertySource(name, resource);
    }
  }

  private Properties loadYml(EncodedResource resource) throws IOException {
    YamlPropertiesFactoryBean factory = new YamlPropertiesFactoryBean();
    factory.setResources(resource.getResource());
    factory.afterPropertiesSet();
    return factory.getObject();
  }
}
```

注入的时候,加上factory属性

```java
@Data
@Component
@ConfigurationProperties(prefix = "family")
@PropertySource(value = {"classpath:family.yml"},factory = MixPropertySourceFactory.class)
@Validated
public class Family {
    private String familyName;
    private Father father;
    private Mother mother;
    private Child child;
}
```

- #### 加载`bean.xml`配置文件

上古时代的配置文件`bean.xml`

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd">
    <bean id="testBeanService" class="com.czc.springboot.demo.service.TestBeanService"></bean>
</beans>
```

然后在启动类上增加注解,注入属性

```java
@SpringBootApplication
@ImportResource(locations = {"classpath:beans.xml"})
public class SpringbootDemoApplication {

    public static void main(String[] args) {
        SpringApplication.run(SpringbootDemoApplication.class, args);
    }

}
```

## 4. 配置文件中敏感字段的加密

- #### 准备加密字段,加密盐,加密结果

从Maven仓库获取`jasypt-1.9.2.jar`,在pom文件中增加依赖,拉取后,在本地仓库中找出

```xml
<!-- 引入依赖,只是为了下载jar-->
<dependency>
    <groupId>org.jasypt</groupId>
    <artifactId>jasypt</artifactId>
    <version>1.9.2</version>
</dependency>
```

编写bat脚本`jasypt.bat`名字随便起,放在`jasypt-1.9.2.jar`同一个文件目录下

````bash
@echo off
set/p input=待加密的明文字符串：
set/p password=加密密钥(盐值)：
echo 加密中......
java -cp jasypt-1.9.2.jar org.jasypt.intf.cli.JasyptPBEStringEncryptionCLI  input=%input% password=%password% algorithm=PBEWithMD5AndDES
pause
````
双击`jasypt.bat`根据提示输入原文,盐,获得加密后的字段

**注意：相同的盐值(密钥)，每次加密的结果是不同的。**

结果:

```
加密字段: root123
加密盐: 123456
加密结果: 3Bn4zZtrCWpVjS1Bwvf2eg==
```

- #### 进行配置加密和获取

引入依赖

```xml
<!--配置文件敏感字段加密-->
<dependency>
    <groupId>com.github.ulisesbocchio</groupId>
    <artifactId>jasypt-spring-boot-starter</artifactId>
	<version>1.18</version>
</dependency>
```

配置盐和加密字段

```yaml
# 设置盐值（加密解密密钥），我们配置在这里只是为了测试方便
# 生产环境中，切记!不要这样直接进行设置，可通过环境变量、命令行等形式进行设置。
jasypt:
  encryptor:
    password: 123456

# 演示需要加密的值
sql:
  password: ENC(3Bn4zZtrCWpVjS1Bwvf2eg==)
```

获取加密字段,和正常字段一样读取

```java
@Value("${sql.password}")
private String password;
```



