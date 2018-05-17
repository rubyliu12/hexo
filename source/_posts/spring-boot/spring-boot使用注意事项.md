---
title: spring boot issues
categories:
  - Java
tags:
  - spring-boot
abbrlink: 47aa1712
date: 2017-09-11 14:11:20
---


## 兼容问题

spring-boot-starter-parent切换回1.5.2.RELEASE版本

因为1.5.4.RELEASE、1.5.5.RELEASE、1.5.6.RELEASE、1.5.7.RELEASE、2.0.0.M1、2.0.0.M2、2.0.0.M3都存在使用spring-boot-starter-data-redis时，会出现异常

`java.lang.NoSuchMethodError: org.springframework.data.repository.config.AnnotationRepositoryConfigurationSource.<init>`异常

请查看[https://github.com/spring-projects/spring-boot/issues/9606](https://github.com/spring-projects/spring-boot/issues/9606)



## 外部 Tomcat 容器部署

> 外部 Tomcat 等容器部署war，要继承 SpringBootServletInitializer 类

```java
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.boot.builder.SpringApplicationBuilder;
import org.springframework.boot.web.support.SpringBootServletInitializer;

@SpringBootApplication
public class OutserverApplication extends SpringBootServletInitializer {

    @Override
    protected SpringApplicationBuilder configure(SpringApplicationBuilder application) {
        return application.sources(OutserverApplication.class);
    }

    public static void main(String[] args) {
        SpringApplication.run(OutserverApplication.class, args);
    }
}
```



## attach=false

```xml
<plugin>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-maven-plugin</artifactId>
    <configuration>
        <layout>ZIP</layout>
        <includes>
            <include>
                <groupId>nothing</groupId>
                <artifactId>nothing</artifactId>
            </include>
        </includes>
        <!-- 将原始的包作为 install 和 deploy 的对象，而不是包含了依赖的包 -->
        <attach>false</attach>
    </configuration>
</plugin>
```





## spring-boot升级到2.0

1. 弃用WebMvcConfigurerAdapter，替换成WebMvcConfigurer

```java
@Configuratiion
public class WebAdapter extends WebMvcConfigurerAdapter {
    @Override
    public void addCorsMappings(CorsRegistry registry) {
        registry.addMapping("/**")
            .allowedHeaders("*")
            .allowedOrigins("*")
            .allowedMethods("GET", "POST", "PUT");
    }
}
```

在 spring-boot 2.0 中需要注意，因为使用的是 spring 5，原先的方法是继承WebMvcConfigurerAdapter抽象类，现在是直接扩展 WebMvcConfigurer 这个接口。原先的方式很能生效，不过已经被 spring 5弃用了（@Deprecated）

```java
@Configuratiion
public class MyConfiguration　{
    @Bean
    public WebMvcConfigurer webMvcConfigurer() {
        return new WebMvcConfigurer() {
            // 跨域配置
            @Override
            public void addCorsMappings(CorsRegistry registry) {
                registry.addMapping("/api/**");
            }
            // 拦截器配置
            @Override
            public void addInterceptors(InterceptorRegistry registry) {
                
            }
            // ...
        }
    }
}
```

2. 其他



## @RequestBody

1. 使用 @RequestBody 注解时，如果注解的参数没有传入，则会抛出`org.springframework.http.converter.HttpMessageNotReadableException`异常
2. 配合hibernate-validator不起作用的问题

在参数前面使用`org.springframework.validation.annotation.Validated`注解，或者`javax.validation.Valid`注解均可

```java
import io.swagger.annotations.ApiModel;
import io.swagger.annotations.ApiModelProperty;
import lombok.AllArgsConstructor;
import lombok.Data;
import lombok.NoArgsConstructor;
import org.hibernate.validator.constraints.Length;
import org.hibernate.validator.constraints.NotBlank;

import java.io.Serializable;

/**
 * @author Created by YL on 2017/10/19
 */
@Data
@AllArgsConstructor
@NoArgsConstructor
@ApiModel
public class MarketRebateDTO implements Serializable {
    @ApiModelProperty(value = "营销活动Id", required = true)
    @NotBlank(message = "营销活动 Id 不能为空")
    @Length(min = 5, max = 8, message = "营销活动 Id[${validatedValue}] 长度必须在 {min} 和 {max} 之间")
    private String marketCfgId;

    @ApiModelProperty(value = "产品号", required = true)
    private String productNo;

    @ApiModelProperty(value = "返利金额/代金券面值", required = true)
    private String rebateAmt;

    @ApiModelProperty(value = "营销活动Id", required = true)
    private String tradeType;
}

import com.alibaba.dubbo.config.annotation.Reference;
import io.swagger.annotations.Api;
import io.swagger.annotations.ApiOperation;
import org.apache.shiro.authz.annotation.RequiresPermissions;
import org.springframework.validation.annotation.Validated;
import org.springframework.web.bind.annotation.PostMapping;
import org.springframework.web.bind.annotation.RequestBody;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;

import javax.validation.Valid;

/**
 * 广东省营销接口
 *
 * @author Created by YL on 2017/10/18
 */
@RestController
@RequestMapping("/api/market")
@Api(tags = "market", description = "营销返利接口")
public class MarketController extends BaseRestController {
    @Reference
    private MarketService marketService;

    @PostMapping("/rebate")
    @RequiresPermissions("api:market:rebate")
    @ApiOperation(value = "营销返利受理")
    public BaseResponse rebate(@RequestBody @Valid MarketRebateDTO rebate) throws CoreException, OkHttpException {
        // if (rebate == null) {
        //     return new BaseResponse("-99", "参数为空");
        // }
        return marketService.doRebate(rebate);
    }
}
```



## @RequestParam

1. 使用 @RequestParam 注解时，如果注解的参数没有传入，则会抛出`org.springframework.web.bind.MissingServletRequestParameterException`异常
2. 配合hibernate-validator不起作用的问题

要注册 MethodValidationPostProcessor Bean，并且在***类***上面使用`org.springframework.validation.annotation.Validated`注解

>注意：使用`javax.validation.Valid`注解对RequestParam对应的参数进行注解，是无效的，需要使用@Validated注解来使得验证生效

```java
@Bean
public Validator validator() {
    ValidatorFactory factory = Validation.byProvider(HibernateValidator.class)
        .configure()
        // 1、普通模式（默认是这个模式）
        // 普通模式(会校验完所有的属性，然后返回所有的验证失败信息)
        // 2、快速失败返回模式
        // 快速失败返回模式(只要有一个验证失败，则返回)
        // .addProperty("hibernate.validator.fail_fast", "true")
        .failFast(true)
        .buildValidatorFactory();
    return factory.getValidator();
}

@Bean
public MethodValidationPostProcessor methodValidationPostProcessor() {
    MethodValidationPostProcessor postProcessor = new MethodValidationPostProcessor();
    // 设置validator模式为快速失败返回
    postProcessor.setValidator(validator());
    return postProcessor;
}

import com.alibaba.dubbo.config.annotation.Reference;
import io.swagger.annotations.Api;
import io.swagger.annotations.ApiImplicitParam;
import io.swagger.annotations.ApiImplicitParams;
import io.swagger.annotations.ApiOperation;
import org.apache.shiro.authz.annotation.RequiresPermissions;
import org.hibernate.validator.constraints.NotBlank;
import org.springframework.validation.annotation.Validated;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.PostMapping;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RequestParam;
import org.springframework.web.bind.annotation.RestController;

/**
 * csp 平台接口封装
 *
 * @author Created by YL on 2017/9/19
 */
@RestController
@RequestMapping("/api/csp")
@Api(tags = "csp", description = "csp 接口平台")
@Validated
public class CspController extends BaseRestController {
    @Reference
    private CspService cspService;

    @GetMapping
    @RequiresPermissions("api:csp:get")
    @ApiOperation(value = "csp 接口-查询")
    @ApiImplicitParams({
            @ApiImplicitParam(name = "func", value = "组件名", required = true, paramType = "query"),
            @ApiImplicitParam(name = "param", value = "参数值", required = true, paramType = "query")
    })
    public BaseResponse get(@RequestParam
                            @NotBlank(message = "组件名不能为空")
                                    String func,
                            @RequestParam
                            @NotBlank(message = "参数值不能为空")
                                    String param) throws OkHttpException, OpenApiException {
        return new BaseResponse(cspService.get(func, param));
    }
}
```

至此，Bean Validator 和 hibernate-validator的@Size、@Min、@Max、@Length、@NotBlank等注解就生效了



