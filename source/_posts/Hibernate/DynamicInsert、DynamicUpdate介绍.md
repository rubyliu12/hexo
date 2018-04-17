---
title: Hibernate的DynamicInsert、DynamicUpdate介绍
date: 2017-07-26 15:47:36
categories: [Hibernate]
tags: [Hibernate, sql]
description:
permalink: hibernate-dynamic
---
## 作用

> 使用 Dynamic Update

如果使用了 Dynamic Update，需要注意的是，当select后，显式的把某些字段set为NULL，hibernate 会认为你修改了该字段，会生成到 update 语句中。一般先 select 实体出来，再 save 的话，只会 update 该实体被修改的字段。否则会 update 所有表字段。
添加Dynamic Update配置可以减少被 update 的字段。

> 使用 Dynamic Insert

如果使用了 Dynamic Insert，并且数据库配置了默认值，当 insert，并且new 实体时，该属性没有 set 值的话，会使用数据库默认值，否则会使用实体的值。

<!-- more -->
## 配置方式
> 第一种：使用注解配置
```java
@Entiry(name = "t_user")
@DynamicInsert
@DynamicUpdate
public class User {
    @Id
    @GeneratedValue
    private Long id;
    private Strin name;
    private Integer age;
    private Integer status;

    // getter setter
}
```

> 第二种：使用.hbm.xml配置
```xml
<hibernate-mapping>
    <class name="xxx.User" table="t_user" dynamic-insert="true" dynamic-update="true">
        <!-- 省略配置 -->
    </class>
</hibernate-mapping>
```

## 效果
主要比较一下insert、update语句的区别，其中update一个已有的实体对象时，hibernate会再执行一次select操作再去update
```java
@RunWith(SpringJUnit4ClassRunner.class)
@SpringBootTest
public class ApplicationTests {
    @Autowired
    private UserRepository repository;

    @Test
    public void test() throws Exception {
        // 1、插入新记录
        // 配置 dynamic insert
        // insert into user (name) values (?)

        // 未配置 dynamic insert
        // insert into user (age, name, status) values (?, ?, ?)
        User user = new User("AAA");
        User save = repository.save(user);

        
        // 2、更新记录
        // 配置 dynamic update
        // select user0_.id as id1_1_0_, user0_.age as age2_1_0_, user0_.name as name3_1_0_, user0_.status as status4_1_0_ from user user0_ where user0_.id=?
        // update user set name=? where id=?

        // 未配置 dynamic update
        // select user0_.id as id1_1_0_, user0_.age as age2_1_0_, user0_.name as name3_1_0_, user0_.status as status4_1_0_ from user user0_ where user0_.id=?
        // update user set age=?, name=?, status=? where id=?
        save.setName("BBB");
        repository.save(save);
    }
}
```
