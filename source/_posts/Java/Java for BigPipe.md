---
title: Java 实现 BigPipe
categories:
  - Java
tags:
  - jsp
  - freemarker
  - BigPipe
abbrlink: d7b08969
date: 2018-08-18 14:47:17
---
# JPipe Project

A Java implementation of Facebook's bigPipe technology.



JPipe是通过自定义标签实现的，所以对后端代码零侵入。

> HTML 是完成前台页面的功能，而自定义标签可以在后台完成某些操作。

JPipe 提供了多线程模式、单线程模式、和混合模式3种使用方式。


> 出于降低后期维护性，这里不打算使用 PageletService 可配置方法，即是在 pagelet 标签增加 method 属性以自定义要执行的函数。
> 统一使用 doGet 函数，只要找到对应的 bean 即可，不用参照前端代码去搜找要执行的函数。



## 特性

- jsp 标签支持（已实现），demo：jpipe-demo-jsp
- freemarker 指令支持（未实现）
- freemarker 中使用 jsp 标签（已实现），demo：jpipe-demo-freemarker-jsp
- thymeleaf 方言支持（未实现）



<!-- more -->



## 开始
### JSP 标签
#### Maven 依赖
```xml
<dependency>
    <groupId>cn.tisson.jpipe</groupId>
    <artifactId>jpipe-jsp</artifactId>
    <version>{version}</version>
</dependency>
```


#### Service 层实现 PageletService 接口的 doGet 方法

这里的 `@Pagelet` 可以使用 Spring 的 `@Service` 代替
```java
@Pagelet("t_pagelet_one")
public class TestPagelet1 implements PageletService {

    @Override
    public Map<String, Object> doGet(Map<String, String> params) {
        Map<String, Object> data = new HashMap<>(params);
        try {
            TimeUnit.MILLISECONDS.sleep(new Random().nextInt(5000));
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        return data;
    }
}
```


#### index.jsp 代码

- 引入标签 `<%@ taglib prefix="jp" uri="http://java.tisson.cn/tags/jsp/jpipe" %>`
- 使用自定义标签，最好放到`</body>`上面，这样就不会堵塞首屏dom的渲染
```html
<%@ page contentType="text/html;charset=UTF-8" trimDirectiveWhitespaces="true" %>
<%@ taglib prefix="jp" uri="http://java.tisson.cn/tags/jsp/jpipe" %>
<html>
<head>
    <title>index</title>
    <meta http-equiv="Content-Type" content="text/html; charset=UTF-8"/>
    <meta name="viewport" content="width=device-width,initial-scale=1.0,maximum-scale=1.0,user-scalable=no">
    <script type="text/javascript">
        ;(function (root, factory) {
            if (typeof exports === 'object') {
                module.exports = exports = factory();
            } else if (typeof define === 'function' && define.amd) {
                define([], factory);
            } else {
                root.JP = factory();
            }
        }(this, function () {
            var JP = JP || (function (window) {
                return {
                    view: function (json) {
                        var code = json['code'],
                            message = json['message'],
                            bn = json['bn'],
                            domId = json['domId'],
                            templateId = json['templateId'],
                            data = json['data'];
        
                        document.getElementById(domId).innerHTML =
                            '<p>code: ' + code
                            + '<br/>message: ' + message
                            + '<br/>bn: ' + bn
                            + '<br/>domId: ' + domId
                            + '<br/>templateId: ' + templateId
                            + '<br/>data: ' + JSON.stringify(data)
                            + '</p>';
                    }
                };
            }(window));
            return JP;
        }));
    </script>
</head>
<body>
<h1>index</h1>
<div id="t_pagelet_1">JPipe 分块会替换这里的内容 1</div>
<div id="t_pagelet_2">JPipe 分块会替换这里的内容 2</div>
<jp:pipe>
    <jp:pagelet domId="t_pagelet_1" bean="t_pagelet_one">
        <jp:param name="id" value="${id}"/>
        <jp:param name="name" value="${name}"/>
    </jp:pagelet>
    <jp:pagelet domId="t_pagelet_2" bean="t_pagelet_one">
        <jp:param name="id" value="年级信息"/>
        <jp:param name="name" value="班级信息"/>
        <jp:param name="age">21</jp:param>
    </jp:pagelet>
</jp:pipe>
</body>
</html>
```
略：部署到 Tomcat、Jetty 等容器



### FTL 指令

未实现
#### Maven 依赖
```xml
<dependency>
    <groupId>cn.tisson.jpipe</groupId>
    <artifactId>jpipe-freemarker</artifactId>
    <version>{version}</version>
</dependency>
```



### 在 FTL 中使用自定义 JSP 标签

FTL 是支持使用 JSP 标签的。如果你的项目本来没有使用 JSP 模版，不推荐这种使用做法。因为自定义 JSP 标签是在 JSP 环境中来写作操作的，需要引入 支持 JSP 1.1或者 JSP 1.2的 Servlet 容器（Tomcat、Jetty等Servlet容器部署），而 FTL 可以在非 Servlet 等 Web 环境中使用。

> 更准确的解释是：尽管 Servlet 容器没有本地的JSP支持，你也可以在 FreeMarker 中使用JSP标签库。 只是确保对JSP 1.2版本(或更新)的 javax.servlet.jsp.* 包在Web应用程序中可用就行。如果你的servlet容器只对JSP 1.1支持， 那么你不得不将下面六个类(比如你可以从Tomcat 5.x或Tomcat 4.x的jar包中提取)复制到Web应用的 WEB-INF/classes/...目录下： javax.servlet.jsp.tagext.IterationTag， javax.servlet.jsp.tagext.TryCatchFinally， javax.servlet.ServletContextListener， javax.servlet.ServletContextAttributeListener， javax.servlet.http.HttpSessionAttributeListener， javax.servlet.http.HttpSessionListener。但是要注意， 因为容器只支持JSP 1.1，通常是使用较早的Servlet 2.3之前的版本，事件监听器可能就不支持，因此JSP 1.2标签库来注册事件监听器会正常工作。

> 截止发稿：JSP已经发布到 2.3版本



#### index.ftl 代码

使用 `<#assign jp=JspTaglibs["http://java.tisson.cn/tags/jsp/jpipe"] />` 引入自定义 JSP 标签
```html
<#assign jp=JspTaglibs["http://java.tisson.cn/tags/jsp/jpipe"] />
<html>
<head>
    <title>index</title>
    <meta http-equiv="Content-Type" content="text/html; charset=UTF-8"/>
    <meta name="viewport" content="width=device-width,initial-scale=1.0,maximum-scale=1.0,user-scalable=no">
    <script type="text/javascript">
        ;(function (root, factory) {
            if (typeof exports === 'object') {
                module.exports = exports = factory();
            } else if (typeof define === 'function' && define.amd) {
                define([], factory);
            } else {
                root.JP = factory();
            }
        }(this, function () {
            var JP = JP || (function (window) {
                return {
                    view: function (json) {
                        var code = json['code'],
                            message = json['message'],
                            bn = json['bn'],
                            domId = json['domId'],
                            templateId = json['templateId'],
                            data = json['data'];
        
                        document.getElementById(domId).innerHTML =
                            '<p>code: ' + code
                            + '<br/>message: ' + message
                            + '<br/>bn: ' + bn
                            + '<br/>domId: ' + domId
                            + '<br/>templateId: ' + templateId
                            + '<br/>data: ' + JSON.stringify(data)
                            + '</p>';
                    }
                };
            }(window));
            return JP;
        }));
    </script>
</head>
<body>
<h1>index</h1>
<div id="pagelet1">JPipe 分块会替换这里的内容 1</div>
<div id="pagelet2">JPipe 分块会替换这里的内容 2</div>
<@jp.pipe>
    <@jp.pagelet domId="pagelet1" bean="t_pagelet_one">
        <@jp.param name="id" value="${id}"/>
        <@jp.param name="name" value="${name}"/>
    </@jp.pagelet>
    <@jp.pagelet domId="pagelet2" bean="t_pagelet_one">
        <@jp.param name="id" value="年级信息"/>
        <@jp.param name="name" value="班级信息"/>
        <@jp.param name="age">21</@jp.param>
    </@jp.pagelet>
</@jp.pipe>
</body>
</html>
```



#### 部署到 undertow

> maven 依赖


```xml
<!-- undertow 部署 -->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-undertow</artifactId>
</dependency>
<!-- undertow 部署 -->

<dependency>
    <groupId>javax.servlet.jsp</groupId>
    <artifactId>javax.servlet.jsp-api</artifactId>
    <version>2.3.3</version>
</dependency>
```
由于 undertow 等容器没有 jsp-api 环境，所以需要依赖 javax.servlet.jsp-api 包，同时要通过 TaglibFactory 配置 freemarker 的 classpathTlds。没有这个配置，会报错：freemarker.ext.jsp.TaglibFactory$TaglibGettingException: No TLD was found for the "http://java.tisson.cn/tags/jsp/jpipe" JSP taglib URI. (TLD-s are searched according the JSP 2.2 specification. In development- and embedded-servlet-container setups you may also need the "MetaInfTldSources" and "ClasspathTlds" freemarker.ext.servlet.FreemarkerServlet init-params or the similar system properites.)
> Configuration


```java
@Configuration
public class MvcWevConfig {
    @Resource
    private FreeMarkerConfigurer freeMarkerConfigurer;

    @PostConstruct
    public void loadClassPathTlds() {
        List<String> classpathTlds = new ArrayList<>();
        classpathTlds.add("/META-INF/JPipe.tld");
        freeMarkerConfigurer.getTaglibFactory().setClasspathTlds(classpathTlds);
    }
}
```


#### 部署到 Tomcat

> maven 依赖


```xml
<!-- 外部 Tomcat 部署 -->
<dependency>
    <groupId>javax.servlet</groupId>
    <artifactId>javax.servlet-api</artifactId>
    <scope>provided</scope>
</dependency>
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-tomcat</artifactId>
    <scope>provided</scope>
</dependency>
<dependency>
    <groupId>org.apache.tomcat.embed</groupId>
    <artifactId>tomcat-embed-jasper</artifactId>
    <scope>provided</scope>
</dependency>
<!-- 外部 Tomcat 部署 -->
```
由于 Tomcat、Jetty中已经有 jsp-api 环境了，这里不需要再依赖 javax.servlet.jsp-api 包



### thymeleaf 方言

未实现
