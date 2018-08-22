---
title: 返回 JSON 格式数据，Long 前端精准度丢失问题
categories:
  - Java
abbrlink: a56434f6
date: 2018-06-14 13:11:20
tags:
---
# 配置 HttpMessageConverter

```java
@Configuration
public class WebMvcConfig extends WebMvcConfigurerAdapter {
    @Override
    public void configureMessageConverters(List<HttpMessageConverter<?>> converters) {
        // 返回 JSON，Long 前端精准度丢失问题
        MappingJackson2HttpMessageConverter converter = new MappingJackson2HttpMessageConverter();
        ObjectMapper mapper = new ObjectMapper();
        /*
         * 序列换成json时,将所有的long变成string
         * 因为js中得数字类型不能包含所有的java long值
         */
        SimpleModule module = new SimpleModule();
        module.addSerializer(Long.class, ToStringSerializer.instance);
        module.addSerializer(Long.TYPE, ToStringSerializer.instance);
        mapper.registerModule(module);
        converter.setObjectMapper(mapper);
        converters.add(converter);
    }
}
```



<!-- more -->



> 在`spring-boot` `2.x`以上版本，要在`WebMvConfig`上添加`@Order(0)`注解，否则配置不生效

```java
import com.fasterxml.jackson.databind.module.SimpleModule;
import com.fasterxml.jackson.databind.ser.std.ToStringSerializer;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.core.annotation.Order;
import org.springframework.http.converter.HttpMessageConverter;
import org.springframework.http.converter.json.Jackson2ObjectMapperBuilder;
import org.springframework.http.converter.json.MappingJackson2HttpMessageConverter;
import org.springframework.web.servlet.config.annotation.InterceptorRegistry;
import org.springframework.web.servlet.config.annotation.WebMvcConfigurer;

import java.math.BigInteger;
import java.text.SimpleDateFormat;
import java.util.List;

/**
 * Web Mvc 自定义配置
 *
 * <pre>
 *     spring-boot 2.0.0.RELEASE configureMessageConverters生效，2.0.3.RELEASE不生效问题
 *     2.0.3.RELEASE要加&#64;Order(0)才可以，而2.0.0.RELEASE以下版本不用加
 * </pre>
 *
 * @author Created by YL on 2017/6/5.
 */
@Configuration
@Order(0)
public class WebMvcConfig implements WebMvcConfigurer {

    @Override
    public void configureMessageConverters(List<HttpMessageConverter<?>> converters) {
        Jackson2ObjectMapperBuilder builder = Jackson2ObjectMapperBuilder.json();
        builder.dateFormat(new SimpleDateFormat("yyyy-MM-dd HH:mm:ss"));

        // if (this.applicationContext != null) {
        //     builder.applicationContext(this.applicationContext);
        // }
        // messageConverters.add(new MappingJackson2HttpMessageConverter(builder.build()));
        // 返回 JSON，Long 前端精准度丢失问题
        // ObjectMapper mapper = new ObjectMapper();
        // // 格式化日志格式
        // mapper.setDateFormat(new SimpleDateFormat("yyyy-MM-dd HH:mm:ss"));


        /*
         * 序列换成json时,将所有的long变成string
         * 因为js中得数字类型不能包含所有的java long值
         */
        SimpleModule module = new SimpleModule();
        module.addSerializer(Long.TYPE, ToStringSerializer.instance);
        module.addSerializer(Long.class, ToStringSerializer.instance);
        module.addSerializer(BigInteger.class, ToStringSerializer.instance);
        builder.modules(module);

        // mapper.registerModule(module);
        // MappingJackson2HttpMessageConverter converter = new MappingJackson2HttpMessageConverter(builder.build());
        // converter.setObjectMapper(mapper);
        converters.add(new MappingJackson2HttpMessageConverter(builder.build()));
    }
}

```

