---
title: spring boot 打包相关
date: 2018-04-08 09:11:20
categories: [Java]
tags: [spring-boot]
description:
permalink: spring-boot-package
---

# webjars打包
包路径：src\main\resources\webjars
```xml
<build>
    <finalName>${project.artifactId}</finalName>
    <resources>
        <resource>
            <directory>${project.basedir}/src/main/resources</directory>
            <targetPath>${project.build.outputDirectory}/META-INF/resources/</targetPath>
        </resource>
    </resources>
</build>
```

# 使用外部配置文件，静态资源
```xml
<resources>
    <resource>
        <directory>src/main/resources</directory>
        <excludes>
            <exclude>freemarker/**/*</exclude>
        </excludes>
    </resource>
    <resource>
        <directory>src/main/resources</directory>
        <targetPath>${project.build.directory}/</targetPath>
        <includes>
            <include>freemarker/**/*</include>
        </includes>
    </resource>
</resources>
```

# 外部 lib 打包方式
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
        <attach>false</attach>
    </configuration>
</plugin>
<plugin>
    <groupId>org.apache.maven.plugins</groupId>
    <artifactId>maven-dependency-plugin</artifactId>
    <executions>
        <execution>
            <id>copy-dependencies</id>
            <phase>package</phase>
            <goals>
                <goal>copy-dependencies</goal>
            </goals>
            <configuration>
                <type>jar</type>
                <includeTypes>jar</includeTypes>
                <includeScope>runtime</includeScope>
                <outputDirectory>
                    ${project.build.directory}/lib
                </outputDirectory>
            </configuration>
        </execution>
    </executions>
</plugin>
```
