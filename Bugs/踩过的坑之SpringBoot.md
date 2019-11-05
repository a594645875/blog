#### ConfigurationProperties使用注意

`@ConfigurationProperties(prefix="aaa")`注解使用时,对象的成员对象属性不能是内部类,要独立一个类文件

#### @Transactional 事务管理失效

问题: 在`service`层用一个没使用`@Transactional`的方法调用加了`@Transactional`的方法的时,事务失效,发生了错误数据库也不回滚

解决: 在`controller`直接调用的service方法上加上`@Transactional`就可以,以下是使用该注解是有有效的场合

```
@Transactional 加于private方法, 无效
@Transactional 加于未加入接口的public方法, 再通过普通接口方法调用, 无效
@Transactional 加于接口方法, 无论下面调用的是private或public方法, 都有效
@Transactional 加于接口方法后, 被本类普通接口方法直接调用, 无效
@Transactional 加于接口方法后, 被本类普通接口方法通过接口调用, 有效
@Transactional 加于接口方法后, 被它类的接口方法调用, 有效
@Transactional 加于接口方法后, 被它类的私有方法调用后, 有效
```

注意: 如果只写`@Transactional`，Spring框架的事务基础架构代码将默认地只在抛出运行时和`unchecked exceptions`时才标识事务回滚。也就是说，当抛出个`RuntimeException `或其子类例的实例时。（Errors也一样 - 默认地 - 标识事务回滚。）从事务方法中抛出的`Checked exceptions`将不被标识进行事务回滚。 

写成`@Transactional(rollbackFor=Exception.class)`可以避免上述的问题