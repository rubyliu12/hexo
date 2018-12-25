---
title: Hibernate 配置字段加解密
categories:
  - Java
tags:
  - Hibernate
  - Oracle
  - DataJpaTest
  - spring-boot
  - jpa
abbrlink: 996dbf54
date: 2018-12-06 18:14:36
---



使用 Hibernate 配置数据库字段的加密、解密函数。

可以使用`@ColumnTransformer`注解的`read`、`write`属性，或者使用 xml 配置的`column`标签的`read`、`write`属性

> 本例子基于 spring-boot-starter-data-jpa、druid-spring-boot-starter、spring-boot-starter-test



<!-- more -->



# User 实体类配置

> wx_crypto.des_decrypt、wx_crypto.des_encrypt自定义的加解密函数，可以使用 Oracle 内置的 to_char、nvl 等函数代替，效果是一样的

- 实体类注解配置

```java
package cn.tisson.domain;

import lombok.Data;
import lombok.NoArgsConstructor;
import org.hibernate.annotations.ColumnTransformer;
import org.hibernate.annotations.DynamicInsert;
import org.hibernate.annotations.DynamicUpdate;

import javax.persistence.Column;
import javax.persistence.Entity;
import javax.persistence.GeneratedValue;
import javax.persistence.Id;
import javax.persistence.SequenceGenerator;
import javax.persistence.Table;

/**
 * 用户信息
 */
@Entity
@Table(name = "t_user")
@DynamicInsert
@DynamicUpdate
@Data
@NoArgsConstructor
public class User implements java.io.Serializable {
    private static final long serialVersionUID = 1L;

    @Id
    @SequenceGenerator(name = "sq_name", sequenceName = "sq_user")
    @GeneratedValue(generator = "sq_name")
    private Long id;
    
    private String userName;
    
    @ColumnTransformer(
        	// 注意这里的 cert_no 使用的是数据库字段名，不是使用实体类的字段名
            read = "wx_crypto.des_decrypt(cert_no)",
            write = "wx_crypto.des_encrypt(?)"
    )
    @Column(name = "cert_no")
    private String certNo; // 身份证号
}

```

> read 配置要使用数据库字段名，否则会报错：ORA-00904: "USER0_"."CERTNO": 标识符无效



- 实体类 User.hbm.xml 配置

> 注意：xml 配置 read、write 需要 hibernate 3.5.x 以上版本才支持

```xml
<?xml version="1.0" encoding="utf-8"?>
<!DOCTYPE hibernate-mapping PUBLIC "-//Hibernate/Hibernate Mapping DTD 4.0//EN"
        "http://hibernate.sourceforge.net/hibernate-mapping-3.0.dtd">
<hibernate-mapping>
    <class name="cn.tisson.domain.User" table="t_user" dynamic-insert="true" dynamic-update="true">
        <id name="id" type="java.lang.Long">
            <column name="ID" precision="22" scale="0"/>
            <generator class="sequence">
                <param name="sequence">sq_user</param>
            </generator>
        </id>

        <property name="userName" type="java.lang.String">
            <column name="user_name" length="20"/>
        </property>

        <property name="certNo" type="java.lang.String">
            <column name="cert_no" length="18"
                    read="wx_crypto.des_decrypt(cert_no)"
                    write="wx_crypto.des_encrypt(?)"/>
        </property>
    </class>
</hibernate-mapping>

```

> read 配置要使用数据库字段名，否则会报错：ORA-00904: "USER0_"."CERTNO": 标识符无效



# 测试

- UserRepository.java

```java
package cn.tisson.repository;

import cn.tisson.domain.User;
import org.springframework.data.jpa.repository.JpaRepository;
import org.springframework.data.jpa.repository.JpaSpecificationExecutor;

/**
 * @author Created by YL on 2017/8/21
 */
public interface UserRepository extends JpaRepository<User, Long>, JpaSpecificationExecutor<User> {
    User findFirstByUserName(String userName);
}
```



- 数据库配置

```yaml
spring:
  datasource:
    url: jdbc:oracle:thin:@(DESCRIPTION=(ADDRESS=(PROTOCOL=TCP)(HOST=127.0.0.1)(PORT=1521))(CONNECT_DATA=(SERVER=DEDICATED)(SERVICE_NAME=bkimDB)))
    username: test
    password: Uf+4C5nKZ9gLxZbvG+pPl4Wb6xDM5/xiPzPLM4PZ/MRESrucR1z9FQUJXnuTiXM+6bdAmJzYRcchhHM+ENmt6g==
    druid:
      initial-size: 3
      min-idle: 3
      max-active: 10
      max-wait: 60000
      keep-alive: true
      time-between-eviction-runs-millis: 60000
      min-evictable-idle-time-millis: 300000
      validation-query: SELECT 'x' FROM DUAL
      test-while-idle: true
      test-on-borrow: false
      test-on-return: false
      pool-prepared-statements: true
      max-pool-prepared-statement-per-connection-size: 20
      filter:
        wall:
          enabled: true
        stat:
          enabled: false
      stat-view-servlet:
        enabled: false
      config:
        enabled: true
      web-stat-filter:
        enabled: false
            connection-properties: config.decrypt=true;config.decrypt.key=MFwwDQYJKoZIhvcNAQEBBQADSwAwSAJBAMcX0mcr65fnwkYTEyxlfiQHxyDGHGzp3hH37na7cmN20y7hd5JjlwXq91xbRzpI/LCu/ZJs5TPhwmHwf46VPy8CAwEAAQ==
  jpa:
    show-sql: true
    database: oracle
    properties:
      hibernate:
        format_sql: true
        dialect: org.hibernate.dialect.Oracle10gDialect
```



- Test 类

```java
package cn.tisson.repository;

import cn.tisson.domain.User;
import org.junit.Test;
import org.junit.runner.RunWith;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.autoconfigure.jdbc.AutoConfigureTestDatabase;
import org.springframework.boot.test.autoconfigure.orm.jpa.DataJpaTest;
import org.springframework.test.context.junit4.SpringRunner;
import org.springframework.transaction.annotation.Propagation;
import org.springframework.transaction.annotation.Transactional;

/**
 * @author Created by YL on 2018/12/7
 */
@RunWith(SpringRunner.class)
@DataJpaTest
// 测试环境默认是会回滚数据，操作的数据是不会插入、更新到数据库的。如果要插入、更新到数据库，要配置一下事物
@Transactional(propagation = Propagation.NOT_SUPPORTED)
// 使用真实环境测试，否则使用的是内嵌的内存数据库
@AutoConfigureTestDatabase(replace = AutoConfigureTestDatabase.Replace.NONE)
// 使用 druid 连接池
@Import({DruidDataSourceAutoConfigure.class})
public class UserRepositoryTest {
    @Autowired
    private UserRepository userRepository;

    @Test
    public void save() {
        User user = new User();
        oauth.setUserName("forever杨");
        oauth.setCertNo("sldfjlsj");
        userRepository.save(user);
    }

    @Test
    public void findOne() {
        User user = userRepository.findFirstByUserName("forever杨");
        System.out.println(user);
    }
}
```

