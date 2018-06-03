---
title: Maven 使用
categories:
  - Java
tags:
  - Maven
abbrlink: e3a1de67
date: 2017-06-19 14:11:20
---
# Maven 打包 jar

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



# 配置阿里云私有库

```xml
<mirror>
    <id>aliyun-nexus</id>
    <mirrorOf>central</mirrorOf>
    <url>http://maven.aliyun.com/nexus/content/groups/public/</url>
</mirror>
<!--
<mirror>
    <id>aliyun-nexus-public-snapshots</id>
    <mirrorOf>public-snapshots</mirrorOf>
    <url>http://maven.aliyun.com/nexus/content/repositories/snapshots/</url>
</mirror>
-->
```



# 发布 jar 到私有仓库

## Maven setting.xml 配置

```xml
<server>
    <id>maven2-weixin-public</id>
    <username>weixin</username>
    <password>weixin</password>
</server>
<server>
    <id>maven2-weixin-release</id>
    <username>weixin</username>
    <password>weixin</password>
</server>
<server>
    <id>maven2-weixin-snapshots</id>
    <username>weixin</username>
    <password>weixin</password>
</server>
```



## 项目 pom.xml 配置

```xml
<!-- 发布 jar 到中央仓库 -->
<distributionManagement>
    <repository>
        <id>maven2-weixin-release</id>
        <name>Nexus Release Repository</name>
        <url>http://maven.ctim:8081/repository/maven2-weixin-release/</url>
    </repository>
    <snapshotRepository>
        <id>maven2-weixin-snapshots</id>
        <name>Nexus Snapshot Repository</name>
        <url>http://maven.ctim:8081/repository/maven2-weixin-snapshots/</url>
    </snapshotRepository>
</distributionManagement>
```

>  注意，pom.xml中的id要和setting.xml中配置的一致



## 发布源码

```xml
<plugin>
    <groupId>org.apache.maven.plugins</groupId>
    <artifactId>maven-source-plugin</artifactId>
    <version>2.4</version>
    <executions>
        <execution>
            <id>attach-sources</id>
            <goals>
                <goal>jar</goal>
            </goals>
        </execution>
    </executions>
</plugin>
```

> 如果不发布源码，可以使用如下配置，只打包源码，不发布到私有仓库

```xml
<plugin>
    <groupId>org.apache.maven.plugins</groupId>
    <artifactId>maven-source-plugin</artifactId>
    <version>2.4</version>
    <executions>
        <execution>
            <id>attach-sources</id>
            <!-- 只打包源码，不发布元源码到私有仓库 -->
            <phase>none</phase>
            <goals>
                <goal>jar</goal>
            </goals>
        </execution>
    </executions>
</plugin>
```



## 执行 deploy 发布 jar 到私有仓库

```sh
mvn clean deploy -Dmaven.test.skip=true
```

> 注意：release 版本只能上传一次，SNAPSHOT 版本可以无限次上传



## 使用私有仓库下载 jar

> 在 setting.xml 配置 mirror

```xml
<mirror>
    <id>aliyun-nexus</id>
    <mirrorOf>*</mirrorOf>
    <url>http://maven.aliyun.com/nexus/content/groups/public/</url>
</mirror>
<mirror>
    <id>maven2-weixin-public</id>
    <mirrorOf>maven2-weixin-public</mirrorOf>
    <url>http://maven.ctim:8081/repository/maven2-weixin-public/</url>
</mirror>
```

> 在 setting.xml 配置 profile 并激活

```xml
<profile>
    <id>wechat</id>
    <repositories>
        <repository>
            <id>maven2-weixin-public</id>
            <name>Nexus Release Repository</name>
            <url>http://maven.ctim:8081/repository/maven2-weixin-public/</url>
            <releases>
                <enabled>true</enabled>
            </releases>
            <snapshots>
                <enabled>true</enabled>
                <updatePolicy>always</updatePolicy>
            </snapshots>
        </repository>
    </repositories>
</profile>
```

激活 profile

```xml
<activeProfiles>
    <activeProfile>wechat</activeProfile>
</activeProfiles>
```
