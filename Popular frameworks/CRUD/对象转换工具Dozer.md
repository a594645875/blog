1. 引入Dozer

   - 增加依赖

   ```xml
     <dependency>
          <groupId>net.sf.dozer</groupId>
          <artifactId>dozer</artifactId>
          <version>5.4.0</version>
       </dependency>
   ```

   - 注入配置

   ```java
       @Configuration
       public class DozerBeanMapperConfigure {
           @Bean
           public DozerBeanMapper mapper() {
               DozerBeanMapper mapper = new DozerBeanMapper();
               return mapper;
           }
       }
   ```

   - 注入和使用

   ```java
   @Autowired
   protected Mapper dozerMapper;
   
   // entity -> entityVo
   EntityVo entityVo = dozerMapper.map(entity, EntityVo.class);
   ```

   - 可以包装一个List转换的工具类

   ```JAVA
   public class DozerUtils {
   
       static DozerBeanMapper dozerBeanMapper = new DozerBeanMapper();
   
       public static <T> List<T> mapList(Collection sourceList, Class<T> destinationClass){
           List destinationList = Lists.newArrayList();
           for (Iterator i$ = sourceList.iterator(); i$.hasNext();){
               Object sourceObject = i$.next();
               Object destinationObject = dozerBeanMapper.map(sourceObject, destinationClass);
               destinationList.add(destinationObject);
           }
           return destinationList;
       }
   }
   ```

