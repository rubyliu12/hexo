---
title: spring-boot parent
categories:
  - Java
tags:
  - spring-boot
abbrlink: 3ef60930
date: 2018-11-12 09:11:20
---





# End of Life

see：[https://spring.io/projects/platform](https://spring.io/projects/platform)

The Platform will reach the end of its supported life on 9 April 2019. Maintenence releases of both the Brussels and Cairo lines will continue to be published up until that time. Users of the Platform are encourage to start using Spring Boot’s dependency management directory, either by using `spring-boot-starter-parent` as their Maven project’s parent, or by importing the `spring-boot-dependencies` bom.



<!-- more -->



# spring-boot-starter-parent

```xml
<parent>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-parent</artifactId>
    <version>2.0.4.RELEASE</version>
    <relativePath/>
</parent>
```



# spring-boot-dependencies

```xml
<properties>
    <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
    <maven.compiler.source>1.8</maven.compiler.source>
    <maven.compiler.target>1.8</maven.compiler.target>

    <spring-boot.version>2.0.4.RELEASE</spring-boot.version>
</properties>


<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-dependencies</artifactId>
    <version>${spring-boot.version}</version>
    <type>pom</type>
    <scope>import</scope>
</dependency>
```

