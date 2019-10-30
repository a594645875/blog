配置JDBC多数据源的时候

```yaml
  datasource:
    primary:
      url: jdbc:mysql://127.0.0.1:3306/test?useUnicode=true&characterEncoding=utf-8&useSSL=false
      username: root
      password: root
      driver-class-name: com.mysql.jdbc.Driver
    secondary:
      url: jdbc:mysql://127.0.0.1:3306/test2?useUnicode=true&characterEncoding=utf-8&useSSL=false
      username: root
      password: root
      driver-class-name: com.mysql.jdbc.Driver
```

会遇到这个错误

```
jdbcUrl is required with driverClassName
```

官方文档的解释是：

```
因为连接池的实际类型没有被公开，所以在您的自定义数据源的元数据中没有生成密钥，而且在IDE中没有完成(因为DataSource接口没有暴露属性)。另外，如果您碰巧在类路径上有Hikari，那么这个基本设置就不起作用了，因为Hikari没有url属性(但是确实有一个jdbcUrl属性)。在这种情况下，您必须重写url的名字
```

正确写法应该是:

```yaml
  datasource:
    primary:
      jdbc-url: jdbc:mysql://127.0.0.1:3306/test?useUnicode=true&characterEncoding=utf-8&useSSL=false
      username: root
      password: root
      driver-class-name: com.mysql.jdbc.Driver
    secondary:
      jdbc-url: jdbc:mysql://127.0.0.1:3306/test2?useUnicode=true&characterEncoding=utf-8&useSSL=false
      username: root
      password: root
      driver-class-name: com.mysql.jdbc.Driver
```

