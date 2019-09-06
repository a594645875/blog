- 描述: SpringBoot+Thymeleaf的时候,
```
@RequestMapping(value = "list", method = RequestMethod.GET)
public String list(Model model) {    
    List<TbUser> tbUsers = tbUserService.selectAll();    
    model.addAttribute("tbUsers", tbUsers);    
    return "user/list";
}
```
无法跳转至`/templates/user/list`指定页面,一直提示404,控制台无任何输出.
分析: 有可能缺少配置`prefix`
```
spring:
    thymeleaf:  
        cache: false # 开发时关闭缓存,不然没法看到实时页面  
        mode: LEGACYHTML5 # 用非严格的 HTML  
        encoding: UTF-8  servlet:    
        content-type: text/html  
        prefix: classpath:/templates/   
```
测试了添加了`prefix`也无法跳转

或者缺少依赖，看到配置中的`spring.thymeleaf`配置一直没报红,我就以为不缺Thymeleaf依赖,
直到我加上了Thymeleaf依赖之后就正常跳转了!

解决办法: 添加缺少的Thymeleaf依赖
```
<dependency>    
    <groupId>org.springframework.boot</groupId>    
    <artifactId>spring-boot-starter-thymeleaf</artifactId>
</dependency>
```