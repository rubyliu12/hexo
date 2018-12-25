---
title: Nacos 实现 gateway 路由动态更新
categories:
  - Java
tags:
  - spring-boot
  - spring-cloud-gateway
  - Nacos
abbrlink: da547d1d
date: 2018-12-25 14:11:20
---



通过[Nacos](https://nacos.io/zh-cn/)配置`spring-cloud-gateway`的路由规则，实现路由规则的动态更新



<!-- more-->



# 依赖

```xml
<properties>
    <spring-cloud.version>Finchley.SR2</spring-cloud.version>
</properties>

<dependencyManagement>
    <dependencies>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-dependencies</artifactId>
            <version>${spring-cloud.version}</version>
            <type>pom</type>
            <scope>import</scope>
        </dependency>
    </dependencies>
</dependencyManagement>

<dependencies>
    <dependency>
        <groupId>org.springframework.cloud</groupId>
        <artifactId>spring-cloud-starter-gateway</artifactId>
    </dependency>
    <dependency>
        <groupId>org.springframework.cloud</groupId>
        <artifactId>spring-cloud-starter-alibaba-nacos-config</artifactId>
        <version>0.2.1.RELEASE</version>
    </dependency>
</dependencies>
```

>  `spring-cloud-starter-alibaba-nacos-config` 的原理是将配置信息注入到`Spring`的`environment`中，并在配置更新时自动触发`context refresh`事件，从而将`environment`环境中的配置变更为最新配置



# application.yml

```yaml
server:
  port: 8080
```



 # bootstrap.properties

```properties
spring.application.name=nacos-spring-cloud-gateway-example
spring.cloud.nacos.config.server-addr=192.168.56.101:8848

spring.cloud.nacos.config.ext-config[0].data-id=gateway.yaml
spring.cloud.nacos.config.ext-config[0].refresh=true
```

> refresh要配置为true，否则不能动态更新 gateway.yaml 的配置项



# App.java

```java
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

@SpringBootApplication
public class App {

    public static void main(String[] args) {
        SpringApplication.run(App.class, args);
    }
}

```



# 在 Nacos 控制台新建 gateway.yaml 配置

Data ID: gateway.yaml

Group: DEFAULT_GROUP
```yaml
spring:
  cloud:
    gateway:
      routes:
        - id: baidu
          uri: https://www.baidu.com
          predicates:
            - Path=/s
        - id: taobao
          uri: https://www.taobao.com/
          predicates:
            - Path=/markets/3c/tbdc
```
配置完成后，执行 `App.java` 启动网关项目



# 浏览器访问

分别访问[http://localhost:8080/s](http://localhost:8080/s)和[http://localhost:8080/markets/3c/tbdc](http://localhost:8080/markets/3c/tbdc)，能正常转发到百度、淘宝网站



# 修改 gateway.yaml 配置

```yaml
spring:
  cloud:
    gateway:
      routes:
        - id: baidu
          uri: https://www.baidu.com
          predicates:
            - Path=/s
```
控制台打印以下日志，说明配置修改已经被监听到
```text
org.springframework.cloud.endpoint.event.RefreshEventListener.handle(37) | Refresh keys changed: [spring.cloud
.gateway.routes.xxx]
```


再次访问[http://localhost:8080/markets/3c/tbdc](http://localhost:8080/markets/3c/tbdc)，会返回404页面
> 说明修改的配置已经动态更新了



# 动态更新的原理

> `spring-cloud-gateway` 的 RouteRefreshListener 已经监听了 ApplicationEvent 事件，所以当 Nacos 中的路由规则变更后，会监听到`context refresh`事件，从而调用 reset 方法 publish 路由更新的事件

**org.springframework.cloud.gateway.route.RouteRefreshListener.java**

```java
/*
 * Copyright 2013-2018 the original author or authors.
 *
 * Licensed under the Apache License, Version 2.0 (the "License");
 * you may not use this file except in compliance with the License.
 * You may obtain a copy of the License at
 *
 *      http://www.apache.org/licenses/LICENSE-2.0
 *
 * Unless required by applicable law or agreed to in writing, software
 * distributed under the License is distributed on an "AS IS" BASIS,
 * WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
 * See the License for the specific language governing permissions and
 * limitations under the License.
 *
 */

package org.springframework.cloud.gateway.route;

import org.springframework.cloud.client.discovery.event.HeartbeatEvent;
import org.springframework.cloud.client.discovery.event.HeartbeatMonitor;
import org.springframework.cloud.client.discovery.event.InstanceRegisteredEvent;
import org.springframework.cloud.client.discovery.event.ParentHeartbeatEvent;
import org.springframework.cloud.context.scope.refresh.RefreshScopeRefreshedEvent;
import org.springframework.cloud.gateway.event.RefreshRoutesEvent;
import org.springframework.context.ApplicationEvent;
import org.springframework.context.ApplicationEventPublisher;
import org.springframework.context.ApplicationListener;
import org.springframework.context.event.ContextRefreshedEvent;
import org.springframework.util.Assert;

// see ZuulDiscoveryRefreshListener
// TODO: make abstract class in commons?
public class RouteRefreshListener
		implements ApplicationListener<ApplicationEvent> {

	private HeartbeatMonitor monitor = new HeartbeatMonitor();
	private final ApplicationEventPublisher publisher;

	public RouteRefreshListener(ApplicationEventPublisher publisher) {
		Assert.notNull(publisher, "publisher may not be null");
		this.publisher = publisher;
	}

	@Override
	public void onApplicationEvent(ApplicationEvent event) {
		if (event instanceof ContextRefreshedEvent
				|| event instanceof RefreshScopeRefreshedEvent
				|| event instanceof InstanceRegisteredEvent) {
			reset();
		}
		else if (event instanceof ParentHeartbeatEvent) {
			ParentHeartbeatEvent e = (ParentHeartbeatEvent) event;
			resetIfNeeded(e.getValue());
		}
		else if (event instanceof HeartbeatEvent) {
			HeartbeatEvent e = (HeartbeatEvent) event;
			resetIfNeeded(e.getValue());
		}
	}

	private void resetIfNeeded(Object value) {
		if (this.monitor.update(value)) {
			reset();
		}
	}

	private void reset() {
		this.publisher.publishEvent(new RefreshRoutesEvent(this));
	}

}

```
