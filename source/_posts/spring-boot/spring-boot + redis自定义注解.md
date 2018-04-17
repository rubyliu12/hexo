---
title: spring boot + redis自定义注解
date: 2018-03-23 15:34:18
categories: [Java]
tags: [spring-boot, redis]
description:
permalink: spring-boot-redis
---

>注意：该方法，不兼容spring-boot 2.0

# 自定义CacheExpire注解
```java
@Target({ElementType.METHOD, ElementType.TYPE})
@Retention(RetentionPolicy.RUNTIME)
@Inherited
@Documented
public @interface CacheExpire {
    /**
     * expire time, default 60s
     */
    @AliasFor("expire")
    long value() default 60L;

    /**
     * expire time, default 60s
     */
    @AliasFor("value")
    long expire() default 60L;
}
```
<!-- more -->

# 自定义redis缓存管理器

```java
import lombok.extern.slf4j.Slf4j;
import org.springframework.beans.BeansException;
import org.springframework.beans.factory.InitializingBean;
import org.springframework.cache.Cache;
import org.springframework.cache.annotation.CacheConfig;
import org.springframework.cache.annotation.Cacheable;
import org.springframework.cache.annotation.Caching;
import org.springframework.context.ApplicationContext;
import org.springframework.context.ApplicationContextAware;
import org.springframework.core.annotation.AnnotationUtils;
import org.springframework.data.redis.cache.RedisCacheManager;
import org.springframework.data.redis.core.RedisOperations;
import org.springframework.util.ReflectionUtils;

import java.lang.reflect.Method;
import java.util.LinkedHashMap;
import java.util.List;
import java.util.Map;

/**
 * Redis 缓存管理器
 * <pre>
 *     support spring-data-redis 1.8.10.RELEASE
 *     not support spring-data-redis 2.0.0.RELEASE +
 * </pre>
 * 
 * <pre>
 *      解决的问题：
 *      Redis 容易出现缓存问题（超时、Redis 宕机等），当使用 spring cache 的注释 Cacheable、Cacheput 等处理缓存问题时，
 *      我们无法使用 try catch 处理出现的异常，所以最后导致结果是整个服务报错无法正常工作。
 *      通过自定义 CustomRedisV1CacheManager 并继承 RedisCacheManager 来处理异常可以解决这个问题。
 * </pre>
 *
 * @author Created by YL on 2017/10/2
 */
@Slf4j
public class CustomJedisCacheManager extends RedisCacheManager implements ApplicationContextAware, InitializingBean {
    private ApplicationContext applicationContext;

    private Map<String, Long> expires = new LinkedHashMap<>();

    public CustomJedisCacheManager(RedisOperations redisOperations) {
        super(redisOperations);
        log.info("Initialize RedisCacheManager. redisOperations: {}", redisOperations);
    }

    @Override
    public Cache getCache(String name) {
        Cache cache = super.getCache(name);
        return new RedisCacheWrapper(cache);
    }

    @Override
    public void setApplicationContext(ApplicationContext applicationContext) throws BeansException {
        this.applicationContext = applicationContext;
    }

    @Override
    public void afterPropertiesSet() {
        String[] beanNames = applicationContext.getBeanNamesForType(Object.class);
        for (String beanName : beanNames) {
            Class clazz = applicationContext.getType(beanName);
            doWith(clazz);
        }
        //设置有效期
        if (!expires.isEmpty()) {
            super.setExpires(expires);
        }
    }

    private void doWith(final Class clazz) {
        ReflectionUtils.doWithMethods(clazz, new ReflectionUtils.MethodCallback() {
            @Override
            public void doWith(Method method) throws IllegalArgumentException, IllegalAccessException {
                CacheExpire cacheExpire = AnnotationUtils.findAnnotation(method, CacheExpire.class);
                // cacheNames 配置优先级：Cacheable > Caching > CacheConfig
                Cacheable cacheable = AnnotationUtils.findAnnotation(method, Cacheable.class);
                Caching caching = AnnotationUtils.findAnnotation(method, Caching.class);
                CacheConfig cacheConfig = AnnotationUtils.findAnnotation(clazz, CacheConfig.class);

                List<String> cacheNames = CacheUtils.getCacheNames(cacheable, caching, cacheConfig);
                putExpires(cacheNames, cacheExpire);
            }
        }, new ReflectionUtils.MethodFilter() {
            @Override
            public boolean matches(Method method) {
                return AnnotationUtils.findAnnotation(method, CacheExpire.class) != null;
            }
        });
    }

    private void putExpires(List<String> cacheNames, CacheExpire cacheExpire) {
        for (String cacheName : cacheNames) {
            if (cacheName == null || "".equals(cacheName.trim())) {
                continue;
            }
            long expire = cacheExpire.expire();
            if (log.isDebugEnabled()) {
                log.debug("cacheNames: {}, expire: {}s", cacheNames, expire);
            }
            if (expire >= 0) {
                expires.put(cacheName, expire);
            } else {
                log.warn("{} use default expiration.", cacheName);
            }
        }
    }
}
```



# RedisCacheWrapper.java

```java
import lombok.extern.slf4j.Slf4j;
import org.springframework.cache.Cache;

import java.util.concurrent.Callable;

/**
 * @author Created by YL on 2018/3/28
 */
@Slf4j
public class RedisCacheWrapper implements Cache {
    private final Cache cache;

    public RedisCacheWrapper(Cache cache) {
        this.cache = cache;
    }

    @Override
    public String getName() {
        // log.info("name: {}", cache.getName());
        try {
            return cache.getName();
        } catch (Exception e) {
            log.error("getName ---> errmsg: {}", e.getMessage(), e);
            return null;
        }
    }

    @Override
    public Object getNativeCache() {
        // log.info("nativeCache: {}", cache.getNativeCache());
        try {
            return cache.getNativeCache();
        } catch (Exception e) {
            log.error("getNativeCache ---> errmsg: {}", e.getMessage(), e);
            return null;
        }
    }

    @Override
    public ValueWrapper get(Object o) {
        // log.info("get ---> o: {}", o);
        try {
            return cache.get(o);
        } catch (Exception e) {
            log.error("get ---> o: {}, errmsg: {}", o, e.getMessage(), e);
            return null;
        }
    }

    @Override
    public <T> T get(Object o, Class<T> aClass) {
        // log.info("get ---> o: {}, clazz: {}", o, aClass);
        try {
            return cache.get(o, aClass);
        } catch (Exception e) {
            log.error("get ---> o: {}, clazz: {}, errmsg: {}", o, aClass, e.getMessage(), e);
            return null;
        }
    }

    @Override
    public <T> T get(Object o, Callable<T> callable) {
        // log.info("get ---> o: {}", o);
        try {
            return cache.get(o, callable);
        } catch (Exception e) {
            log.error("get ---> o: {}, errmsg: {}", o, e.getMessage(), e);
            return null;
        }
    }

    @Override
    public void put(Object o, Object o1) {
        // log.info("put ---> o: {}, o1: {}", o, o1);
        try {
            cache.put(o, o1);
        } catch (Exception e) {
            log.error("put ---> o: {}, o1: {}, errmsg: {}", o, o1, e.getMessage(), e);
        }
    }

    @Override
    public ValueWrapper putIfAbsent(Object o, Object o1) {
        // log.info("putIfAbsent ---> o: {}, o1: {}", o, o1);
        try {
            return cache.putIfAbsent(o, o1);
        } catch (Exception e) {
            log.error("putIfAbsent ---> o: {}, o1: {}, errmsg: {}", o, o1, e.getMessage(), e);
            return null;
        }
    }

    @Override
    public void evict(Object o) {
        // log.info("evict ---> o: {}", o);
        try {
            cache.evict(o);
        } catch (Exception e) {
            log.error("evict ---> o: {}, errmsg: {}", o, e.getMessage(), e);
        }
    }

    @Override
    public void clear() {
        // log.info("clear");
        try {
            cache.clear();
        } catch (Exception e) {
            log.error("clear ---> errmsg: {}", e.getMessage(), e);
        }
    }
}
```



# CacheUtils工具类

```java
import org.springframework.cache.annotation.CacheConfig;
import org.springframework.cache.annotation.Cacheable;
import org.springframework.cache.annotation.Caching;

import java.util.ArrayList;
import java.util.Arrays;
import java.util.List;

/**
 * 缓存工具
 *
 * @author Created by YL on 2018/4/2
 */
public class CacheUtils {
    /**
     * 获取 cacheNames
     * <pre>
     *     处理优先级 cacheable > caching > cacheConfig
     * </pre>
     *
     * @param cacheable   缓存注解 Cacheable
     * @param caching     缓存注解 Caching
     * @param cacheConfig 缓存注解 CacheConfig
     */
    public static List<String> getCacheNames(Cacheable cacheable, Caching caching, CacheConfig cacheConfig) {
        List<String> list = new ArrayList<>();
        // Cacheable
        if (cacheable != null) {
            String[] cacheNames = cacheable.cacheNames();
            if (cacheNames.length > 0) {
                list.addAll(Arrays.asList(cacheNames));
            }
        }
        if (list.size() > 0) {
            return list;
        }

        // Caching
        if (caching != null) {
            Cacheable[] cacheables = caching.cacheable();
            for (Cacheable cache : cacheables) {
                String[] cacheNames = cache.cacheNames();
                if (cacheNames.length > 0) {
                    list.addAll(Arrays.asList(cacheNames));
                }
            }
        }
        if (list.size() > 0) {
            return list;
        }

        // CacheConfig
        if (cacheConfig != null) {
            String[] cacheNames = cacheConfig.cacheNames();
            if (cacheNames.length > 0) {
                list.addAll(Arrays.asList(cacheNames));
            }
        }
        return list;
    }
}
```



# Redis相关Bean初始化

```java
import com.alibaba.fastjson.support.spring.GenericFastJsonRedisSerializer;
import lombok.extern.slf4j.Slf4j;
import org.springframework.boot.autoconfigure.condition.ConditionalOnProperty;
import org.springframework.boot.autoconfigure.data.redis.RedisProperties;
import org.springframework.boot.context.properties.EnableConfigurationProperties;
import org.springframework.cache.CacheManager;
import org.springframework.cache.annotation.CachingConfigurerSupport;
import org.springframework.cache.annotation.EnableCaching;
import org.springframework.cache.interceptor.KeyGenerator;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.data.redis.connection.RedisConnectionFactory;
import org.springframework.data.redis.connection.jedis.JedisConnectionFactory;
import org.springframework.data.redis.core.RedisTemplate;
import org.springframework.data.redis.serializer.StringRedisSerializer;
import redis.clients.jedis.JedisPoolConfig;

/**
 * 初始化 Redis 相关 Bean
 * <pre>
 *     support spring-data-redis 1.8.10.RELEASE
 *     not support spring-data-redis 2.0.0.RELEASE +
 * </pre>
 *
 * @author Created by YL on 2017/12/6
 */
@Configuration
// 如果 application.properties、application.yml 中没有 spring.redis.host 配置，则不初始化这些 Bean
@ConditionalOnProperty(name = "spring.redis.host")
@EnableConfigurationProperties(RedisProperties.class)
@EnableCaching(proxyTargetClass = true) // 加上这个注解是为了支持 @Cacheable、@CachePut、@CacheEvict 等缓存注解
@Slf4j
public class CustomJedisAutoConfiguration extends CachingConfigurerSupport {
    private final RedisConnectionFactory redisConnectionFactory;

    CustomJedisAutoConfiguration(RedisConnectionFactory redisConnectionFactory) {
        this.redisConnectionFactory = redisConnectionFactory;
    }

    private static final StringRedisSerializer SERIALIZER = new StringRedisSerializer();
    /**
     * fastjson serializer
     * <pre>
     *     使用 FastJsonRedisSerializer 会报错：java.lang.ClassCastException
     *     FastJsonRedisSerializer<Object> fastSerializer = new FastJsonRedisSerializer<>(Object.class);
     * </pre>
     */
    private static final GenericFastJsonRedisSerializer FAST_SERIALIZER = new GenericFastJsonRedisSerializer();

    /**
     * 配置 RedisTemplate，设置序列化器
     * <pre>
     *     在类里面配置 RestTemplate，需要配置 key 和 value 的序列化类。
     *     key 序列化使用 StringRedisSerializer, 不配置的话，key 会出现乱码。
     * </pre>
     */
    @Bean
    public RedisTemplate<String, Object> redisTemplate() {
        RedisTemplate<String, Object> template = new RedisTemplate<>();
        // set key serializer
        // 设置key序列化类，否则key前面会多了一些乱码
        template.setKeySerializer(SERIALIZER);
        template.setHashKeySerializer(SERIALIZER);

        // fastjson serializer
        template.setValueSerializer(FAST_SERIALIZER);
        template.setHashValueSerializer(FAST_SERIALIZER);
        // 如果 KeySerializer 或者 ValueSerializer 没有配置，则对应的 KeySerializer、ValueSerializer 才使用这个 Serializer
        template.setDefaultSerializer(FAST_SERIALIZER);

        JedisConnectionFactory factory = (JedisConnectionFactory) redisConnectionFactory;
        log.info("spring.redis.database: {}", factory.getDatabase());
        log.info("spring.redis.host: {}", factory.getHostName());
        log.info("spring.redis.port: {}", factory.getPort());
        log.info("spring.redis.timeout: {}", factory.getTimeout());
        log.info("spring.redis.password: {}", factory.getPassword());
        JedisPoolConfig pool = factory.getPoolConfig();
        log.info("spring.redis.use-pool: {}", factory.getUsePool());
        log.info("spring.redis.min-idle: {}", pool.getMinIdle());
        log.info("spring.redis.max-idle: {}", pool.getMaxIdle());
        log.info("spring.redis.max-active: {}", pool.getMaxTotal());
        log.info("spring.redis.max-wait: {}", pool.getMaxWaitMillis());
        log.info("spring.redis.test-on-borrow: {}", pool.getTestOnBorrow());
        log.info("spring.redis.test-on-create: {}", pool.getTestOnCreate());
        log.info("spring.redis.test-while-idle: {}", pool.getTestWhileIdle());
        log.info("spring.redis.time-between-eviction-runs-millis: {}", pool
                .getTimeBetweenEvictionRunsMillis());

        template.setConnectionFactory(redisConnectionFactory);
        template.afterPropertiesSet();
        return template;
    }

    /**
     * 如果 @Cacheable、@CachePut、@CacheEvict 等注解没有配置 key，则使用这个自定义 key 生成器
     * <pre>
     *     但自定义了缓存的 key 时，难以保证 key 的唯一性，此时最好指定方法名，比如：@Cacheable(value="", key="{#root.methodName, #id}")
     * </pre>
     */
    @Bean
    @Override
    public KeyGenerator keyGenerator() {
        return (o, method, objects) -> {
            StringBuilder sb = new StringBuilder(32);
            sb.append(o.getClass().getSimpleName());
            sb.append(".");
            sb.append(method.getName());
            if (objects.length > 0) {
                sb.append("#");
            }
            String sp = "";
            for (Object object : objects) {
                sb.append(sp);
                if (object == null) {
                    sb.append("NULL");
                } else {
                    sb.append(object.toString());
                }
                sp = ".";
            }
            return sb.toString();
        };
    }

    /**
     * 配置 RedisCacheManager，使用 cache 注解管理 redis 缓存
     */
    @Bean
    @Override
    public CacheManager cacheManager() {
        CustomJedisCacheManager manager = new CustomJedisCacheManager(redisTemplate());
        // manager.setLoadRemoteCachesOnStartup(true);
        // 如果没有配置过期时间，则使用默认的过期时间：30 min
        manager.setDefaultExpiration(30 * 60L);
        // 注 默认会添加 key 前缀以防止两个单独的缓存使用相同的 key，否则 Redis 将存在重复的 key，有可能返回不可用的值。
        // 如果创建自己的 RedisCacheManager，强烈建议你保留该配置处于启用状态。
        manager.setUsePrefix(true);
        return manager;
    }
}
```


# pom.xml

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-redis</artifactId>
    <version><version>1.5.10.RELEASE</version></version>
</dependency>
<dependency>
    <groupId>com.alibaba</groupId>
    <artifactId>fastjson</artifactId>
    <version>1.2.44</version>
</dependency>
```



# 使用例子

```java
@Service
public class TestService {

    @Cacheable(value = "test", unless = "#result == null or #result.empty")
    @CacheExpire(expire = 60)
    public Map<String, Object> get(String name) {
        Map<String, Object> map = new HashMap<>();
        map.put("id", 123);
        map.put("name", name);
        return map;
    }
}
```
