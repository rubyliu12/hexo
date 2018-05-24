---
title: spring boot 打包相关
categories:
  - Java
tags:
  - spring-boot
abbrlink: ab825d27
date: 2018-04-08 09:11:20
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

<!-- more -->

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



# JAR部署方式

> 外部 lib 打包方式

```xml
<plugin>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-maven-plugin</artifactId>
    <version>${spring-boot.version}</version>
    <executions>
        <execution>
            <goals>
                <goal>repackage</goal>
            </goals>
        </execution>
    </executions>
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
    <version>3.0.2</version>
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



# WAR包部署方式

```xml
<plugins>
    <plugin>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-maven-plugin</artifactId>
        <configuration>
            <layout>WAR</layout>
            <includes>
                <include>
                    <groupId>nothing</groupId>
                    <artifactId>nothing</artifactId>
                </include>
            </includes>
            <attach>false</attach>
        </configuration>
    </plugin>
</plugins>
```

