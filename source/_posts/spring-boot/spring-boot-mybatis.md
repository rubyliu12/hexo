---
title: mybatis整合spring boot
date: 2017-03-21 15:44:18
categories: [Java]
tags: [spring-boot, mybatis]
description:
permalink: spring-boot-mybatis
---

1. 在pom.xml加入如下内容
```xml
<dependency>
    <groupId>org.mybatis.spring.boot</groupId>
    <artifactId>mybatis-spring-boot-starter</artifactId>
    <version>1.3.0</version>
</dependency>
```
<!-- more -->
2. mybatis-config.xml
```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE configuration PUBLIC "-//mybatis.org//DTD Config 3.0//EN" "http://mybatis.org/dtd/mybatis-3-config.dtd">
<configuration>
    <typeAliases>
        <typeAlias alias="Integer" type="java.lang.Integer"/>
        <typeAlias alias="Long" type="java.lang.Long"/>
        <typeAlias alias="HashMap" type="java.util.HashMap"/>
        <typeAlias alias="LinkedHashMap" type="java.util.LinkedHashMap"/>
        <typeAlias alias="ArrayList" type="java.util.ArrayList"/>
        <typeAlias alias="LinkedList" type="java.util.LinkedList"/>
    </typeAliases>
    <typeAliases>
        <package name="cn.tisson.wechat.job.pojo"/>
    </typeAliases>
    <mappers>
        <mapper resource="cn/tisson/wechat/job/mapper/UserMapper.xml"/>
    </mappers>
</configuration>
```
或者使用在application.yml中加入
```
mybatis:
  config-location: classpath:mybatis/mybatis-config.xml
  type-aliases-package: cn.tisson.wechat.job.pojo
#  mapper-locations这个配置参数仅当mapper xml与mapper class不在同一个目录下时有效。所以一般可以忽略。
#  mapper-locations: classpath:cn/tisson/wechat/job/mapper/*.xml
```
在Mapper类上面使用 `@Mapper` 注解
```java
@Mapper
public interface UserMapper {
    long count(User user);
}
```

Mapper.xml配置
```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN" "http://mybatis.org/dtd/mybatis-3-mapper.dtd" >
<mapper namespace="cn.tisson.wechat.job.mapper.UserMapper">
    <resultMap id="BaseResultMap" type="cn.tisson.wechat.job.pojo.User">
        <constructor>
            <arg column="ID" javaType="java.math.BigDecimal" jdbcType="DECIMAL"/>
            <arg column="AGE" javaType="java.math.BigDecimal" jdbcType="DECIMAL"/>
            <arg column="CITYID" javaType="java.math.BigDecimal" jdbcType="DECIMAL"/>
        </constructor>
    </resultMap>

    <select id="count" parameterType="cn.tisson.wechat.job.pojo.User" resultType="java.lang.Long">
        select count(*) from user where cityid = #{user.cityid}
    </select>
</mapper>
```
