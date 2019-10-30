1. #### 引入依赖

   ```xml
   <dependency>
       <groupId>org.springframework.boot</groupId>
       <artifactId>spring-boot-starter-jta-atomikos</artifactId>
   </dependency>
   ```

2. #### 修改application配置文件

   固定写法,对应jta的类属性

   ```YAML
   primarydb:
     uniqueResourceName: primary
     xaDataSourceClassName: com.mysql.jdbc.jdbc2.optional.MysqlXADataSource
     xaProperties:
       url: jdbc:mysql://192.168.1.91:3306/testdb?useUnicode=true&characterEncoding=utf-8&useSSL=false
       user: test
       password: 4rfv$RFV
     exclusiveConnectionMode: true
     minPoolSize: 3
     maxPoolSize: 10
     testQuery: SELECT 1 from dual #由于采用HikiriCP，用于检测数据库连接是否存活。
   
   secondarydb:
     uniqueResourceName: secondary
     xaDataSourceClassName: com.mysql.jdbc.jdbc2.optional.MysqlXADataSource
     xaProperties:
       url: jdbc:mysql://192.168.1.91:3306/testdb2?useUnicode=true&characterEncoding=utf-8&useSSL=false
       user: test
       password: 4rfv$RFV
     exclusiveConnectionMode: true
     minPoolSize: 3
     maxPoolSize: 10
     testQuery: SELECT 1 from dual #由于采用HikiriCP，用于检测数据库连接是否存活。
   ```

3. #### 数据源配置

   ```java
   @Configuration
   public class DataSourceConfig {
        //jta数据源primarydb
       @Bean(initMethod="init", destroyMethod="close", name="primaryDataSource")
       @Primary
       @ConfigurationProperties(prefix = "primarydb")
       public DataSource primaryDataSource() {
            //这里是关键，返回的是AtomikosDataSourceBean，所有的配置属性也都是注入到这个类里面
           return new AtomikosDataSourceBean();
       }
   
       //jta数据源secondarydb
       @Bean(initMethod="init", destroyMethod="close", name="secondaryDataSource")
       @ConfigurationProperties(prefix = "secondarydb")
       public DataSource secondaryDataSource()  {
           return new AtomikosDataSourceBean();
       }
   
       @Bean
       public JdbcTemplate primaryJdbcTemplate(@Qualifier("primaryDataSource") DataSource primaryDataSource) {
           return new JdbcTemplate(primaryDataSource);
       }
   
       @Bean
       public JdbcTemplate secondaryJdbcTemplate(@Qualifier("secondaryDataSource") DataSource secondaryDataSource) {
           return new JdbcTemplate(secondaryDataSource);
       }
   }
   ```

4. #### 事务管理器配置

   ```java
   @Configuration
   public class TransactionManagerConfig {
   
       @Bean
       public UserTransaction userTransaction() throws SystemException {
           UserTransactionImp userTransactionImp = new UserTransactionImp();
           userTransactionImp.setTransactionTimeout(10000);
           return userTransactionImp;
       }
   
       @Bean(name = "atomikosTransactionManager", initMethod = "init", destroyMethod = "close")
       public TransactionManager atomikosTransactionManager() throws Throwable {
           UserTransactionManager userTransactionManager = new UserTransactionManager();
           userTransactionManager.setForceShutdown(false);
           return userTransactionManager;
       }
   
       @Bean(name = "transactionManager")
       @DependsOn({ "userTransaction", "atomikosTransactionManager" })
       public PlatformTransactionManager transactionManager() throws Throwable {
           UserTransaction userTransaction = userTransaction();
   
           JtaTransactionManager manager = new JtaTransactionManager(userTransaction, atomikosTransactionManager());
           return manager;
       }
   }
   ```

5. #### 测试

   ```java
   @Resource
   private JdbcTemplate primaryJdbcTemplate;
   @Resource
   private JdbcTemplate secondaryJdbcTemplate;
   
   @Transactional
   public Article saveArticle( Article article) {
       articleJDBCDAO.save(article,primaryJdbcTemplate);
       articleJDBCDAO.save(article,secondaryJdbcTemplate);
       int a = 2/0;
       return article;
   }
   ```

- 用原生的datasources的時候，多个数据源连接时,事务只会对第一个开启的数据源起作用,`articleJDBCDAO.save(article,secondaryJdbcTemplate)`会将数据存入数据库,不符合事务要求
- 使用JTA的配置后,两个数据源的操作会同时修改或同时回滚
- **优点： 能够支持分布式事务**
  **缺点： 性能开销大，不适合用于高并发场景**

