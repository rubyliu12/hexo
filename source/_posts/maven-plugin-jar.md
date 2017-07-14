---
title: maven plugin jar
date: 2017-06-19 14:11:20
categories: [Java]
tags: [Maven]
description:
permalink: maven-package-jar-lib
---
# maven打包jar，并把第三方jar包放到lib中

spring-boot插件打包
```xml
<build>
    <finalName>${project.artifactId}</finalName>
    <plugins>
        <plugin>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-maven-plugin</artifactId>
        </plugin>
    </plugins>
</build>
```
<!-- more -->
```xml
<!-- 打包 -->
<build>
    <finalName>${project.artifactId}</finalName>
    <resources>
        <!-- 指定 src/main/resources下所有文件及文件夹为资源文件 -->
        <resource>
            <targetPath>${project.build.directory}/classes</targetPath>
            <directory>src/main/resources</directory>
            <filtering>true</filtering>
            <includes>
                <include>**/*</include>
                <!-- <include>**/*.xml</include> <include>**/*.properties</include> -->
            </includes>
        </resource>
        <!-- 使用com.alibaba.dubbo.container.Main启动应用的话，要把spring配置放到/META-INF/spring下 -->
        <resource>
            <targetPath>${project.build.directory}/classes/META-INF/spring</targetPath>
            <directory>src/main/resources/spring</directory>
            <filtering>true</filtering>
            <includes>
                <include>spring-context.xml</include>
            </includes>
        </resource>
    </resources>
    <!-- more -->
    <plugins>
        <!-- 打包jar文件时，配置manifest文件，加入lib包的jar依赖 -->
        <plugin>
            <groupId>org.apache.maven.plugins</groupId>
            <artifactId>maven-jar-plugin</artifactId>
            <configuration>
                <classesDirectory>target/classes/</classesDirectory>
                <archive>
                    <manifest>
                        <mainClass>com.alibaba.dubbo.container.Main</mainClass>
                        <!-- 打包时 MANIFEST.MF文件不记录的时间戳版本 -->
                        <useUniqueVersions>false</useUniqueVersions>
                        <addClasspath>true</addClasspath>
                        <classpathPrefix>lib/</classpathPrefix>
                    </manifest>
                    <manifestEntries>
                        <Class-Path>.</Class-Path>
                    </manifestEntries>
                </archive>
            </configuration>
        </plugin>
        <!-- 打包jar时，把依赖的第三方jar包，放到lib/文件下 -->
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
                        <useUniqueVersions>false</useUniqueVersions>
                        <outputDirectory>
                            ${project.build.directory}/lib
                        </outputDirectory>
                    </configuration>
                </execution>
            </executions>
        </plugin>
    </plugins>
</build>
```
