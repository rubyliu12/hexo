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



代码在托管[nacos-spring-cloud-gateway-example](https://github.com/foreveryang321/java-learning-examples/tree/master/nacos-spring-cloud-gateway-example)



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



# 动态更新原理

1. [Nacos](https://nacos.io/zh-cn/) 配置更新的时候，`spring-cloud-starter-alibaba-nacos-config`会 `publish` 一个 `RefreshEvent` 事件，从而使 `spring-cloud-commons` 的 `RefreshEventListener` 监听到并触发 `ContextRefresher.refresh()` 方法。
2. `spring-cloud-gateway` 的 `RouteRefreshListener` 监听了 `ApplicationEvent` 事件，当 `Nacos` 触发 `ContextRefresher.refresh()`后，会监听到 `RefreshScopeRefreshedEvent`事件并调用`RouteRefreshListener.reset()` 方法 `publish`  一个 `RefreshRoutesEvent` 路由更新事件，达到路由动态更新的目的。




## spring-cloud-alibaba-nacos-config

在`spring-cloud-alibaba-nacos-config`中，会默认监听配置的更新，并publish refresh事件

- NacosRefreshProperties.java

```java
@Component
public class NacosRefreshProperties {

	@Value("${spring.cloud.nacos.config.refresh.enabled:true}")
	private boolean enabled = true;

	public boolean isEnabled() {
		return enabled;
	}

	public void setEnabled(boolean enabled) {
		this.enabled = enabled;
	}
}
```

> `...refresh.enabled:true`默认开启配置更新事件推送



- NacosContextRefresher.java

```java
public class NacosContextRefresher
		implements ApplicationListener<ApplicationReadyEvent>, ApplicationContextAware {

	private final static Logger LOGGER = LoggerFactory
			.getLogger(NacosContextRefresher.class);

	public static final AtomicLong loadCount = new AtomicLong(0);

	private final NacosRefreshProperties refreshProperties;

	private final NacosRefreshHistory refreshHistory;

	private final ConfigService configService;

	private ApplicationContext applicationContext;

	private AtomicBoolean ready = new AtomicBoolean(false);

	private Map<String, Listener> listenerMap = new ConcurrentHashMap<>(16);

	public NacosContextRefresher(NacosRefreshProperties refreshProperties,
			NacosRefreshHistory refreshHistory, ConfigService configService) {
		this.refreshProperties = refreshProperties;
		this.refreshHistory = refreshHistory;
		this.configService = configService;
	}

	@Override
	public void onApplicationEvent(ApplicationReadyEvent event) {
		// many Spring context
		if (this.ready.compareAndSet(false, true)) {
			this.registerNacosListenersForApplications();
		}
	}

	@Override
	public void setApplicationContext(ApplicationContext applicationContext) {
		this.applicationContext = applicationContext;
	}

	private void registerNacosListenersForApplications() {
		if (refreshProperties.isEnabled()) {
			for (NacosPropertySource nacosPropertySource : NacosPropertySourceRepository
					.getAll()) {

				if (!nacosPropertySource.isRefreshable()) {
					continue;
				}

				String dataId = nacosPropertySource.getDataId();
				registerNacosListener(nacosPropertySource.getGroup(), dataId);
			}
		}
	}

	private void registerNacosListener(final String group, final String dataId) {

		Listener listener = listenerMap.computeIfAbsent(dataId, i -> new Listener() {
			@Override
			public void receiveConfigInfo(String configInfo) {
				loadCount.incrementAndGet();
				String md5 = "";
				if (!StringUtils.isEmpty(configInfo)) {
					try {
						MessageDigest md = MessageDigest.getInstance("MD5");
						md5 = new BigInteger(1, md.digest(configInfo.getBytes("UTF-8")))
								.toString(16);
					}
					catch (NoSuchAlgorithmException | UnsupportedEncodingException e) {
						LOGGER.warn("[Nacos] unable to get md5 for dataId: " + dataId, e);
					}
				}
				refreshHistory.add(dataId, md5);
				applicationContext.publishEvent(
						new RefreshEvent(this, null, "Refresh Nacos config"));
				if (LOGGER.isDebugEnabled()) {
					LOGGER.debug("Refresh Nacos config group{},dataId{}", group, dataId);
				}
			}

			@Override
			public Executor getExecutor() {
				return null;
			}
		});

		try {
			configService.addListener(dataId, group, listener);
		}
		catch (NacosException e) {
			e.printStackTrace();
		}
	}

}
```



## spring-cloud-gateway

- RefreshEventListener.java

```java
public class RefreshEventListener {
	private static Log log = LogFactory.getLog(RefreshEventListener.class);
	private ContextRefresher refresh;
	private AtomicBoolean ready = new AtomicBoolean(false);

	public RefreshEventListener(ContextRefresher refresh) {
		this.refresh = refresh;
	}

	@EventListener
	public void handle(ApplicationReadyEvent event) {
		this.ready.compareAndSet(false, true);
	}

	@EventListener
	public void handle(RefreshEvent event) {
		if (this.ready.get()) { // don't handle events before app is ready
			log.debug("Event received " + event.getEventDesc());
			Set<String> keys = this.refresh.refresh();
			log.info("Refresh keys changed: " + keys);
		}
	}
}
```



- RouteRefreshListener.java

```java
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
