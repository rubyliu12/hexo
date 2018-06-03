---
title: 使用RSA注意事项
categories:
  - Java
tags:
  - RSA
abbrlink: ef742936
date: 2017-05-23 21:33:43
---

# 比较以下两种写法
```java
// 第一种：这种写法会造成内存溢出
Cipher cipher = Cipher.getInstance("RSA", new BouncyCastleProvider());
```

```java
// 第二种：推荐这种写法
// BC可用BouncyCastleProvider.PROVIDER_NAME代替
if (StringUtils.isNullOrEmpty(Security.getProperty("BC"))) {
    Security.addProvider(new BouncyCastleProvider());
}
Cipher cipher = Cipher.getInstance("RSA", "BC");
```
还有一种写法是要替换jdk的jar包，不推荐



<!-- more -->



# 要依赖bcprov-jdk16包
```xml
<dependency>
    <groupId>org.bouncycastle</groupId>
    <artifactId>bcprov-jdk16</artifactId>
    <version>1.46</version>
</dependency>
```
