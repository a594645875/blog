#### PageHelper不分页
- 问题:使用`pagehelper`之后,依然查出了全部的数据,没有分页的效果

- 分析:只导入了`pagehelper`包,应该导入Springboot的pagehelper包

- 解决:把pom中的pagehelper依赖修改成`pagehelper-spring-boot-starter`,就可以了
#### page 1 of 1 containing  unknown instance
- 问题：return new Result(true,StatusCode.OK,"查询成功",
new PageResult<Article>(articlePage.getTotalElements(),articlePage.getContent());
查询得元素个数2，但是无实体数据，row为空。

- 原因：查询了2个数据，但是service层page没有-1，所有取得是第二页的结果，所有实体为空。

- 解决办法：在service层“page”改成“page-1”。
#### Mybatis Plus插入时id和数据库id自增冲突
- 问题: 使用Mybatis Plus批量插入时,显示id类型不匹配
- 分析: Mybatis Plus默认会在插入的时候生成一个id,所以出错是因为没有调用数据库的主键自增函数
- 解决办法: 在id上加上`@TableId`注解,并定义id模式
```
   AUTO->"数据库ID自增"
   INPUT-> 用户输入ID"
   ID_WORKER->"全局唯一ID"
   UUID->"全局唯一ID"
```
#### Mybatis Plus 插入数据类型为numeric的数据时出错
- 问题: 在使用String向Pgsql插入数据类型为numeric的数据时出错,显示类型不匹配
- 分析: 看到旧代码上用的是BigDecimal映射数据库的numeric
- 解决: 把对应的实体类属性由String改为BigDecimal.

#### 无数据库配置时Springboot无法启动
- 日志
```
Description:

Failed to configure a DataSource: 'url' attribute is not specified and no embedded datasource could be configured.
```
- 解决办法: 在启动类配置上排除数据库自动配置
```
@SpringBootApplication(scanBasePackages = "com.funtl.myshop",exclude = DataSourceAutoConfiguration.class)
```


