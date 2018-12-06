---
title: spring-session + redis 保存会话 session
categories:
  - Java
tags:
  - spring-boot
  - spring-session
  - redis
abbrlink: 74b23c9e
date: 2018-12-04 14:11:20
---

# spring-boot 1.x

```java
import com.alibaba.fastjson.support.spring.GenericFastJsonRedisSerializer;
import org.springframework.beans.factory.annotation.Qualifier;
import org.springframework.context.annotation.Bean;
import org.springframework.data.redis.serializer.RedisSerializer;
import org.springframework.session.config.annotation.web.http.SpringHttpSessionConfiguration;
import org.springframework.session.data.redis.config.ConfigureRedisAction;
import org.springframework.session.data.redis.config.annotation.web.http.EnableRedisHttpSession;
import org.springframework.session.data.redis.config.annotation.web.http.RedisHttpSessionConfiguration;
import org.springframework.session.web.http.CookieHttpSessionStrategy;
import org.springframework.session.web.http.CookieSerializer;
import org.springframework.session.web.http.DefaultCookieSerializer;
import org.springframework.session.web.http.HttpSessionStrategy;

/**
 * 使用 redis 存储 HttpSession
 * <pre>
 *     spring-boot 两种方式启用 spring-session + redis
 *     1、使用 @EnableRedisHttpSession 注解
 *     2、在application.properties 配置 spring.session.store-type: redis
 *     优先级：@EnableRedisHttpSession > application.properties
 * </pre>
 *
 * @author Created by YL on 2018/11/7
 */
@EnableRedisHttpSession
public class RedisHttpSessionConfig {
    /**
     * disable sessions expire event notifications
     */
    @Bean
    public ConfigureRedisAction configureRedisAction() {
        return ConfigureRedisAction.NO_OP;
    }

    /**
     * spring-session 使用 fastjson 序列化
     * <p>bean name 要配置成 <strong>springSessionDefaultRedisSerializer</strong>，否则不生效，参考：
     * {@link RedisHttpSessionConfiguration#setDefaultRedisSerializer(org.springframework.data.redis.serializer.RedisSerializer)}</p>
     */
    @Bean
    @Qualifier("springSessionDefaultRedisSerializer")
    public RedisSerializer<Object> springSessionDefaultRedisSerializer() {
        return new GenericFastJsonRedisSerializer();
    }

    /**
     * cookie serializer
     * <p>
     * {@link SpringHttpSessionConfiguration#setCookieSerializer(org.springframework.session.web.http.CookieSerializer)}
     */
    @Bean
    @Qualifier("cookieSerializer")
    public CookieSerializer cookieSerializer() {
        DefaultCookieSerializer serializer = new DefaultCookieSerializer();
        serializer.setCookieName("x-auth-token");
        // serializer.setDomainName("localhost");
        // serializer.setDomainNamePattern("(\\w+)");//("^.+?\\.(\\w+\\.[a-z]+)$");
        serializer.setCookiePath("/");
        serializer.setUseHttpOnlyCookie(true);
        // 由于spring-boot 1.x和2.x这个默认值不一样，所以统一设置成 false
        serializer.setUseBase64Encoding(false);
        // 防止 http、https 混用的情况
        serializer.setUseSecureCookie(false);
        return serializer;
    }

    /**
     * HttpSessionStrategy
     * <p>
     * {@link SpringHttpSessionConfiguration#setHttpSessionStrategy(org.springframework.session.web.http.HttpSessionStrategy)}
     */
    @Bean
    @Qualifier("httpSessionStrategy")
    public HttpSessionStrategy httpSessionStrategy() {
        CookieHttpSessionStrategy strategy = new CookieHttpSessionStrategy();
        strategy.setCookieSerializer(cookieSerializer());
        return strategy;
    }

    // /**
    //  * HttpSessionStrategy
    //  * <p>
    //  * {@link SpringHttpSessionConfiguration#setHttpSessionStrategy(org.springframework.session.web.http
    // .HttpSessionStrategy)}
    //  */
    // @Bean
    // @Qualifier("httpSessionStrategy")
    // public HttpSessionStrategy httpSessionStrategy() {
    //     HeaderHttpSessionStrategy strategy = new HeaderHttpSessionStrategy();
    //     strategy.setHeaderName(UTConst.COOKIE_NAME);
    //     return strategy;
    // }
}

```



# spring-boot 2.x

```java
import com.alibaba.fastjson.support.spring.GenericFastJsonRedisSerializer;
import org.springframework.beans.factory.annotation.Qualifier;
import org.springframework.context.annotation.Bean;
import org.springframework.data.redis.serializer.RedisSerializer;
import org.springframework.session.config.annotation.web.http.SpringHttpSessionConfiguration;
import org.springframework.session.data.redis.config.ConfigureRedisAction;
import org.springframework.session.data.redis.config.annotation.web.http.EnableRedisHttpSession;
import org.springframework.session.data.redis.config.annotation.web.http.RedisHttpSessionConfiguration;
import org.springframework.session.web.http.CookieHttpSessionIdResolver;
import org.springframework.session.web.http.CookieSerializer;
import org.springframework.session.web.http.DefaultCookieSerializer;
import org.springframework.session.web.http.HttpSessionIdResolver;

/**
* 使用 redis 存储 HttpSession
* <pre>
*     spring-boot 两种方式启用 spring-session + redis
*     1、使用 @EnableRedisHttpSession 注解
*     2、在application.properties 配置 spring.session.store-type: redis
*     优先级：@EnableRedisHttpSession > application.properties
* </pre>
*
* @author Created by YL on 2018/11/7
*/
@EnableRedisHttpSession
public class RedisHttpSessionConfig {
   /**
    * disable sessions expire event notifications
    */
   @Bean
   public ConfigureRedisAction configureRedisAction() {
       return ConfigureRedisAction.NO_OP;
   }

   /**
    * spring-session 使用 fastjson 序列化
    * <p>bean name 要配置成 <strong>springSessionDefaultRedisSerializer</strong>，否则不生效，参考：
    * {@link RedisHttpSessionConfiguration#setDefaultRedisSerializer(RedisSerializer)}</p>
    */
   @Bean
   @Qualifier("springSessionDefaultRedisSerializer")
   public RedisSerializer<Object> springSessionDefaultRedisSerializer() {
       return new GenericFastJsonRedisSerializer();
   }

   /**
    * cookie serializer
    * <p>
    * {@link SpringHttpSessionConfiguration#setCookieSerializer(CookieSerializer)}
    */
   @Bean
   @Qualifier("cookieSerializer")
   public CookieSerializer cookieSerializer() {
       DefaultCookieSerializer serializer = new DefaultCookieSerializer();
       serializer.setCookieName("x-auth-token");
       // serializer.setDomainName("localhost");
       // serializer.setDomainNamePattern("(\\w+)");//("^.+?\\.(\\w+\\.[a-z]+)$");
       serializer.setCookiePath("/");
       serializer.setUseHttpOnlyCookie(true);
       // 由于spring-boot 1.x和2.x这个默认值不一样，所以统一设置成 false
       serializer.setUseBase64Encoding(false);
       // 防止 http、https 混用的情况
       serializer.setUseSecureCookie(false);
       return serializer;
   }

   /**
    * HttpSessionIdResolver
    * <p>
    * {@link SpringHttpSessionConfiguration#setHttpSessionIdResolver(HttpSessionIdResolver)}
    */
   @Bean
   @Qualifier("httpSessionIdResolver")
   public HttpSessionIdResolver httpSessionIdResolver() {
       CookieHttpSessionIdResolver resolver = new CookieHttpSessionIdResolver();
       resolver.setCookieSerializer(cookieSerializer());
       return resolver;
   }

   // /**
   //  * HttpSessionIdResolverPackagesUtil
   //  * <p>
   //  * {@link SpringHttpSessionConfiguration#setHttpSessionIdResolver(HttpSessionIdResolver)}
   //  */
   // @Bean
   // @Qualifier("httpSessionIdResolver")
   // public HttpSessionIdResolver httpSessionIdResolver() {
   //     return new HeaderHttpSessionIdResolver("x-gdwxkf-token");
   // }
}

```

