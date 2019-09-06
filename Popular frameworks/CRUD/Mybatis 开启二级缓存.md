### 1. 实体类设置序列号
- IDEA 提示生成序列号
默认情况下 Intellij IDEA 不会提示继承了 Serializable 接口的类生成 serialVersionUID 的警告。如果需要生成 serialVersionUID，需要手动配置。勾选以下配置：
`File -> Settings -> Inspections -> Serialization issues -> Serialization class without 'serialVersionUID'`
![IDEA配置](https://www.funtl.com/assets/Lusifer1534116410.png)

- 实体类实现序列化接口`Serializable`,IDEA会提示添加`serialVersionUID`,按提示添加即可
```
...
public class Post extends Model implements Serializable {    
    private static final long serialVersionUID = -5670222965464951507L;
...
```

### 2. 实现 Spring ApplicationContextAware 接口，用于手动注入 Bean
创建一个名为 `ApplicationContextHolder` 的工具类，代码如下：
```
package com.funtl.itoken.common.context;

import org.apache.commons.lang3.Validate;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.beans.BeansException;
import org.springframework.beans.factory.DisposableBean;
import org.springframework.context.ApplicationContext;
import org.springframework.context.ApplicationContextAware;
import org.springframework.stereotype.Component;

@Component
public class ApplicationContextHolder implements ApplicationContextAware, DisposableBean {
    private static final Logger logger = LoggerFactory.getLogger(ApplicationContextHolder.class);

    private static ApplicationContext applicationContext;

    /**
     * 获取存储在静态变量中的 ApplicationContext
     *
     * @return
     */
    public static ApplicationContext getApplicationContext() {
        assertContextInjected();
        return applicationContext;
    }

    /**
     * 从静态变量 applicationContext 中获取 Bean，自动转型成所赋值对象的类型
     *
     * @param name
     * @param <T>
     * @return
     */
    public static <T> T getBean(String name) {
        assertContextInjected();
        return (T) applicationContext.getBean(name);
    }

    /**
     * 从静态变量 applicationContext 中获取 Bean，自动转型成所赋值对象的类型
     *
     * @param clazz
     * @param <T>
     * @return
     */
    public static <T> T getBean(Class<T> clazz) {
        assertContextInjected();
        return applicationContext.getBean(clazz);
    }

    /**
     * 实现 DisposableBean 接口，在 Context 关闭时清理静态变量
     *
     * @throws Exception
     */
    public void destroy() throws Exception {
        logger.debug("清除 SpringContext 中的 ApplicationContext: {}", applicationContext);
        applicationContext = null;
    }

    /**
     * 实现 ApplicationContextAware 接口，注入 Context 到静态变量中
     *
     * @param applicationContext
     * @throws BeansException
     */
    public void setApplicationContext(ApplicationContext applicationContext) throws BeansException {
        ApplicationContextHolder.applicationContext = applicationContext;
    }

    /**
     * 断言 Context 已经注入
     */
    private static void assertContextInjected() {
        Validate.validState(applicationContext != null, "applicationContext 属性未注入，请在 spring-context.xml 配置中定义 ApplicationContextHolder");
    }
}
```

### 3.实现 MyBatis Cache 接口，用于自定义缓存为 Redis
创建一个名为 `RedisCache` 的工具类，代码如下：
```
package com.funtl.itoken.common.utils;

import com.funtl.itoken.common.context.ApplicationContextHolder;
import org.apache.ibatis.cache.Cache;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.data.redis.core.RedisCallback;
import org.springframework.data.redis.core.RedisTemplate;
import org.springframework.data.redis.core.ValueOperations;

import java.util.concurrent.TimeUnit;
import java.util.concurrent.locks.ReadWriteLock;
import java.util.concurrent.locks.ReentrantReadWriteLock;

/**
 * Redis 缓存工具类
 * <p>Title: RedisCache</p>
 * <p>Description: </p>
 *
 * @author Lusifer
 * @version 1.0.0
 * @date 2018/8/13 6:03
 */
public class RedisCache implements Cache {
    private static final Logger logger = LoggerFactory.getLogger(RedisCache.class);

    private final ReadWriteLock readWriteLock = new ReentrantReadWriteLock();
    private final String id; // cache instance id
    private RedisTemplate redisTemplate;

    private static final long EXPIRE_TIME_IN_MINUTES = 30; // redis过期时间

    public RedisCache(String id) {
        if (id == null) {
            throw new IllegalArgumentException("Cache instances require an ID");
        }
        this.id = id;
    }

    @Override
    public String getId() {
        return id;
    }

    /**
     * Put query result to redis
     *
     * @param key
     * @param value
     */
    @Override
    public void putObject(Object key, Object value) {
        try {
            RedisTemplate redisTemplate = getRedisTemplate();
            ValueOperations opsForValue = redisTemplate.opsForValue();
            opsForValue.set(key, value, EXPIRE_TIME_IN_MINUTES, TimeUnit.MINUTES);
            logger.debug("Put query result to redis");
        } catch (Throwable t) {
            logger.error("Redis put failed", t);
        }
    }

    /**
     * Get cached query result from redis
     *
     * @param key
     * @return
     */
    @Override
    public Object getObject(Object key) {
        try {
            RedisTemplate redisTemplate = getRedisTemplate();
            ValueOperations opsForValue = redisTemplate.opsForValue();
            logger.debug("Get cached query result from redis");
//            System.out.println("****" + opsForValue.get(key).toString());
            return opsForValue.get(key);
        } catch (Throwable t) {
            logger.error("Redis get failed, fail over to db", t);
            return null;
        }
    }

    /**
     * Remove cached query result from redis
     *
     * @param key
     * @return
     */
    @Override
    @SuppressWarnings("unchecked")
    public Object removeObject(Object key) {
        try {
            RedisTemplate redisTemplate = getRedisTemplate();
            redisTemplate.delete(key);
            logger.debug("Remove cached query result from redis");
        } catch (Throwable t) {
            logger.error("Redis remove failed", t);
        }
        return null;
    }

    /**
     * Clears this cache instance
     */
    @Override
    public void clear() {
        RedisTemplate redisTemplate = getRedisTemplate();
        redisTemplate.execute((RedisCallback) connection -> {
            connection.flushDb();
            return null;
        });
        logger.debug("Clear all the cached query result from redis");
    }

    /**
     * This method is not used
     *
     * @return
     */
    @Override
    public int getSize() {
        return 0;
    }

    @Override
    public ReadWriteLock getReadWriteLock() {
        return readWriteLock;
    }

    private RedisTemplate getRedisTemplate() {
        if (redisTemplate == null) {
            redisTemplate = ApplicationContextHolder.getBean("redisTemplate");
        }
        return redisTemplate;
    }
}
```
### 4. Mapper 接口中增加注解
在 Mapper 接口中增加注解，声明需要使用二级缓存
```
package com.funtl.itoken.common.mapper;

import com.funtl.itoken.common.domain.TbSysUser;
import com.funtl.itoken.common.utils.RedisCache;
import org.apache.ibatis.annotations.CacheNamespace;
import tk.mybatis.mapper.MyMapper;

@CacheNamespace(implementation = RedisCache.class)
public interface TbSysUserMapper extends MyMapper<TbSysUser> {
}
```

### 测试
启动项目,操作一次查询,查看redis中是否有缓存,有缓存则表示开启成功.
### 踩坑
- 项目配置需要有Redis相关配置
- `@SpringBootApplication` 注解要扫描到 ApplicationContextHolder 所在的包可以在后面添加`scanBasePackages   @SpringBootApplication(scanBasePackages = "com.funtl.itoken")`
- 相关依赖需配齐