### @JsonField不生效
- 问题: 旧项目改造,代码中有很多的`@JsonField`注解,但是改造了一部分后发现这个注解不起作用了,研究了很久,怎么都不生效
- 分析: FastJson有两种注入方法,原来是用`@Bean`在`WebConfig`中注册的,可能版本兼容性或者其他原因,暂时不清楚,需要换成继承然后重载方法的方式配置
- 解决: 把`@Bean`方法换成`@Override`重写方法注入FastJson,`@JsonField`就可以生效了