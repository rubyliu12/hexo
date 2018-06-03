---
title: JPA Criteria 查询
categories:
  - Java
tags:
  - Hibernate
abbrlink: 6202485a
date: 2017-07-30 09:21:36
---

#构建 CriteriaQuery 实例API说明

## CriteriaBuilder 安全查询创建工厂

`CriteriaBuilder`是一个工厂对象，可以从`EntityManager`或`EntityManagerFactory`类中获得。可以用于创建`CriteriaQuery`、`Predicate`等
```java
CriteriaBuilder builder = entityManager.getCriteriaBuilder();
```


<!-- more -->


## CriteriaQuery 安全查询主语句

`CriteriaQuery`对象必须在实体类型或嵌入式类型上的`Criteria`查询上起作用。它通过调用 `CriteriaBuilder.createQuery`或`CriteriaBuilder.createTupleQuery`等获得。
```java
CriteriaQuery<User> query = builder.createQuery(User.class);
```
## Root 定义查询的 From 子句中能出现的类型

```java
// 获取 User 实体查询的根对象
Root<User> root = query.from(User.class);
```
## Predicate 过滤条件

过滤条件应用到SQL语句的FROM子句中。

在`Criteria`查询中，查询条件通过`Predicate`或`Expression`实例应用到`CriteriaQuery` 对象上。这些条件使用`CriteriaQuery.where`方法应用到`CriteriaQuery`对象上。

`Predicate`对象通过调用`CriteriaBuilder`的`equal`、`notEqual`、`gt`、`ge`、`lt`、`le`、`between`、`like`等方法创建。

`Predicate`实例也可以用`Expression`实例的`isNull`、`isNotNull`和`in`方法获得，复合的`Predicate`语句可以使用`CriteriaBuilder`的`and`、`or`方法构建。
```java
// 查询 id 大于10的用户
Predicate predicate = builder.gt(root.get("id"), 10);
query.where(predicate);
```
## Expression

`Expression`对象用在查询语句的`select`、`where`和`having`子句中，该接口有`isNull`、`isNotNull`和`in`方法
```java
// 第一种
CriteriaQuery<User> query = builder.createQuery(User.class);
Root<User> root = query.from(User.class);
query.where(root.get("id").in(1, 5));
entityManager.createQuery(query).getResultList();
// select * from user where id in (1, 5)

// 第二种
Expression<String> exp = root.get("id");
List<Integer> list = new ArrayList<>();
list.add(1);
list.add(5);
predicates.add(exp.in(list));
query.where(predicates.toArray(new Predicate[predicates.size()]));
// select * from user where id in (1, 5)
```
## 高级用法
### 复合查询
```java
// 使用 CriteriaQuery，查询年龄为10，且名字以 ts 开头的用户
Path<String> name = root.get("name");
query.where(
    builder.and(
        builder.equal(root.get("age"), 10),
        builder.like(name, "ts%")
    )
);
entityManager.createQuery(query).getResultList();

// 使用 CriteriaBuilder，查询年龄为10，名字以 ts 开头的用户
Path<String> name = root.get("name");
Predicate and = builder.and(
    builder.equal(root.get("age"), 10),
    builder.like(name, "ts%")
);
```
```sql
SELECT
    user0_.id AS id1_1_,
    user0_.age AS age2_1_,
    user0_.create_date AS create_d3_1_,
    user0_. NAME AS name4_1_,
    user0_. STATUS AS status5_1_
FROM
    USER user0_
WHERE
    user0_.age = 10
AND (user0_. NAME LIKE ?)
```
