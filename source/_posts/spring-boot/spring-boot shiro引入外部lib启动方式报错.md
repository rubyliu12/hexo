---
title: spring-boot shiro引入外部lib启动方式报错
date: 2018-02-05 15:34:07
categories: [Java]
tags: [spring-boot, shiro]
description:
permalink: spring-boot-shiro-ext-lib
---

打包方式
```xml
<!--<plugin>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-maven-plugin</artifactId>
</plugin>-->
<!-- 外部 lib 打包方式 -->
<plugin>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-maven-plugin</artifactId>
    <configuration>
        <layout>ZIP</layout>
        <executable>true</executable>
        <includes>
            <include>
                <groupId>nothing</groupId>
                <artifactId>nothing</artifactId>
            </include>
        </includes>
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
<!-- 外部 lib 打包方式 -->
```

启动脚本
```shell
/usr/java/jdk1.8.0_144/bin/java -server -Djava.ext.dirs=/opt/tisson-web-open/lib -Dspring.profiles.active=local -jar tisson-web-open.jar
```

启动异常信息
```txt
  .   ____          _            __ _ _
 /\\ / ___'_ __ _ _(_)_ __  __ _ \ \ \ \
( ( )\___ | '_ | '_| | '_ \/ _` | \ \ \ \
 \\/  ___)| |_)| | | | | || (_| |  ) ) ) )
  '  |____| .__|_| |_|_| |_\__, | / / / /
 =========|_|==============|___/=/_/_/_/
 :: Spring Boot ::        (v1.5.2.RELEASE)

[17-12-07 14:41:05 INFO ] main org.springframework.boot.SpringApplication.logStarting(48)  | Starting application on im2-gf-test with PID 12160 (started by imhtp in /opt/tisson-web-open/bak)
[17-12-07 14:41:05 DEBUG] main org.springframework.boot.SpringApplication.logStarting(51)  | Running with Spring Boot v1.5.2.RELEASE, Spring v4.3.7.RELEASE
[17-12-07 14:41:05 INFO ] main org.springframework.boot.SpringApplication.logStartupProfileInfo(641)  | The following profiles are active: local
[17-12-07 14:41:05 DEBUG] main org.springframework.boot.SpringApplication.load(665)  | Loading source class cn.tisson.wechat.web.open.OpenApplication
[17-12-07 14:41:05 DEBUG] main org.springframework.boot.context.config.ConfigFileApplicationListener.replayTo(172)  | Activated profiles local
[17-12-07 14:41:05 DEBUG] main org.springframework.boot.context.config.ConfigFileApplicationListener.replayTo(172)  | Profiles already activated, '[local]' will not be applied
[17-12-07 14:41:05 DEBUG] main org.springframework.boot.context.config.ConfigFileApplicationListener.replayTo(172)  | Loaded config file 'jar:file:/opt/tisson-web-open/bak/tisson-web-open.jar!/BOOT-INF/classes!/application.yml' (classpath:/application.yml)
[17-12-07 14:41:05 DEBUG] main org.springframework.boot.context.config.ConfigFileApplicationListener.replayTo(172)  | Loaded config file 'jar:file:/opt/tisson-web-open/bak/tisson-web-open.jar!/BOOT-INF/classes!/application-local.yml' (classpath:/application-local.yml)
[17-12-07 14:41:05 DEBUG] main org.springframework.boot.context.config.ConfigFileApplicationListener.replayTo(172)  | Skipped (empty) config file 'jar:file:/opt/tisson-web-open/bak/tisson-web-open.jar!/BOOT-INF/classes!/application-local.yml' (classpath:/application-local.yml) for profile local
[17-12-07 14:41:05 DEBUG] main org.springframework.boot.context.config.ConfigFileApplicationListener.replayTo(172)  | Skipped (empty) config file 'jar:file:/opt/tisson-web-open/bak/tisson-web-open.jar!/BOOT-INF/classes!/application.yml' (classpath:/application.yml) for profile local
[17-12-07 14:41:05 INFO ] main org.springframework.boot.context.embedded.AnnotationConfigEmbeddedWebApplicationContext.prepareRefresh(582)  | Refreshing org.springframework.boot.context.embedded.AnnotationConfigEmbeddedWebApplicationContext@31190526: startup date [Thu Dec 07 14:41:05 CST 2017]; root of context hierarchy
[17-12-07 14:41:05 DEBUG] main org.springframework.boot.context.embedded.AnnotationConfigEmbeddedWebApplicationContext.obtainFreshBeanFactory(616)  | Bean factory for org.springframework.boot.context.embedded.AnnotationConfigEmbeddedWebApplicationContext@31190526: org.springframework.beans.factory.support.DefaultListableBeanFactory@663c9e7a: defining beans [org.springframework.context.annotation.internalConfigurationAnnotationProcessor,org.springframework.context.annotation.internalAutowiredAnnotationProcessor,org.springframework.context.annotation.internalRequiredAnnotationProcessor,org.springframework.context.annotation.internalCommonAnnotationProcessor,org.springframework.context.event.internalEventListenerProcessor,org.springframework.context.event.internalEventListenerFactory,openApplication]; root of factory hierarchy
[17-12-07 14:41:05 INFO ] background-preinit org.hibernate.validator.internal.util.Version.<clinit>(30)  | HV000001: Hibernate Validator 5.3.4.Final
[17-12-07 14:41:06 INFO ] main com.alibaba.dubbo.config.spring.context.annotation.DubboComponentScanRegistrar.registerServiceBeans(85)  | 0 annotated @Service Components { [] } were scanned under package[cn.tisson.wechat.web.open]
[17-12-07 14:41:06 INFO ] main org.springframework.context.support.PostProcessorRegistrationDelegate$BeanPostProcessorChecker.postProcessAfterInitialization(325)  | Bean 'shiroConfig' of type [cn.tisson.wechat.web.open.config.ShiroConfig$$EnhancerBySpringCGLIB$$d483728f] is not eligible for getting processed by all BeanPostProcessors (for example: not eligible for auto-proxying)
[17-12-07 14:41:06 WARN ] main org.springframework.boot.context.embedded.AnnotationConfigEmbeddedWebApplicationContext.refresh(550)  | Exception encountered during context initialization - cancelling refresh attempt: org.springframework.beans.factory.UnsatisfiedDependencyException: Error creating bean with name 'methodValidationPostProcessor' defined in class path resource [org/springframework/boot/autoconfigure/validation/ValidationAutoConfiguration.class]: Unsatisfied dependency expressed through method 'methodValidationPostProcessor' parameter 0; nested exception is org.springframework.beans.factory.BeanCreationException: Error creating bean with name 'methodInvokingFactoryBean' defined in class path resource [cn/tisson/wechat/web/open/config/ShiroConfig.class]: Bean instantiation via factory method failed; nested exception is org.springframework.beans.BeanInstantiationException: Failed to instantiate [org.springframework.beans.factory.config.MethodInvokingFactoryBean]: Factory method 'methodInvokingFactoryBean' threw exception; nested exception is org.springframework.beans.factory.BeanCreationException: Error creating bean with name 'securityManager' defined in class path resource [cn/tisson/wechat/web/open/config/ShiroConfig.class]: Bean instantiation via factory method failed; nested exception is org.springframework.beans.BeanInstantiationException: Failed to instantiate [org.apache.shiro.mgt.SecurityManager]: Factory method 'securityManager' threw exception; nested exception is java.lang.IllegalStateException: Unable to acquire AES algorithm.  This is required to function.
[17-12-07 14:41:06 ERROR] main org.springframework.beans.factory.support.DefaultListableBeanFactory.destroyBean(581)  | Destroy method on bean with name 'org.springframework.boot.context.properties.ConfigurationPropertiesBindingPostProcessor' threw an exception
java.lang.IllegalStateException: ApplicationEventMulticaster not initialized - call 'refresh' before multicasting events via the context: org.springframework.boot.context.embedded.AnnotationConfigEmbeddedWebApplicationContext@31190526: startup date [Thu Dec 07 14:41:05 CST 2017]; root of context hierarchy
    at org.springframework.context.support.AbstractApplicationContext.getApplicationEventMulticaster(AbstractApplicationContext.java:404)
    at org.springframework.context.support.ApplicationListenerDetector.postProcessBeforeDestruction(ApplicationListenerDetector.java:97)
    at org.springframework.beans.factory.support.DisposableBeanAdapter.destroy(DisposableBeanAdapter.java:253)
    at org.springframework.beans.factory.support.DefaultSingletonBeanRegistry.destroyBean(DefaultSingletonBeanRegistry.java:578)
    at org.springframework.beans.factory.support.DefaultSingletonBeanRegistry.destroySingleton(DefaultSingletonBeanRegistry.java:554)
    at org.springframework.beans.factory.support.DefaultListableBeanFactory.destroySingleton(DefaultListableBeanFactory.java:961)
    at org.springframework.beans.factory.support.DefaultSingletonBeanRegistry.destroySingletons(DefaultSingletonBeanRegistry.java:523)
    at org.springframework.beans.factory.support.DefaultListableBeanFactory.destroySingletons(DefaultListableBeanFactory.java:968)
    at org.springframework.context.support.AbstractApplicationContext.destroyBeans(AbstractApplicationContext.java:1033)
    at org.springframework.context.support.AbstractApplicationContext.refresh(AbstractApplicationContext.java:555)
    at org.springframework.boot.context.embedded.EmbeddedWebApplicationContext.refresh(EmbeddedWebApplicationContext.java:122)
    at org.springframework.boot.SpringApplication.refresh(SpringApplication.java:737)
    at org.springframework.boot.SpringApplication.refreshContext(SpringApplication.java:370)
    at org.springframework.boot.SpringApplication.run(SpringApplication.java:314)
    at org.springframework.boot.SpringApplication.run(SpringApplication.java:1162)
    at org.springframework.boot.SpringApplication.run(SpringApplication.java:1151)
    at cn.tisson.wechat.web.open.OpenApplication.main(OpenApplication.java:35)
    at sun.reflect.NativeMethodAccessorImpl.invoke0(Native Method)
    at sun.reflect.NativeMethodAccessorImpl.invoke(NativeMethodAccessorImpl.java:62)
    at sun.reflect.DelegatingMethodAccessorImpl.invoke(DelegatingMethodAccessorImpl.java:43)
    at java.lang.reflect.Method.invoke(Method.java:498)
    at org.springframework.boot.loader.MainMethodRunner.run(MainMethodRunner.java:48)
    at org.springframework.boot.loader.Launcher.launch(Launcher.java:87)
    at org.springframework.boot.loader.Launcher.launch(Launcher.java:50)
    at org.springframework.boot.loader.PropertiesLauncher.main(PropertiesLauncher.java:557)
[17-12-07 14:41:06 INFO ] main com.alibaba.dubbo.config.spring.beans.factory.annotation.ReferenceAnnotationBeanPostProcessor.destroy(246)  | class com.alibaba.dubbo.config.spring.beans.factory.annotation.ReferenceAnnotationBeanPostProcessor was destroying!
[17-12-07 14:41:06 ERROR] main org.springframework.beans.factory.support.DefaultListableBeanFactory.destroyBean(581)  | Destroy method on bean with name 'org.springframework.boot.autoconfigure.internalCachingMetadataReaderFactory' threw an exception
java.lang.IllegalStateException: ApplicationEventMulticaster not initialized - call 'refresh' before multicasting events via the context: org.springframework.boot.context.embedded.AnnotationConfigEmbeddedWebApplicationContext@31190526: startup date [Thu Dec 07 14:41:05 CST 2017]; root of context hierarchy
    at org.springframework.context.support.AbstractApplicationContext.getApplicationEventMulticaster(AbstractApplicationContext.java:404)
    at org.springframework.context.support.ApplicationListenerDetector.postProcessBeforeDestruction(ApplicationListenerDetector.java:97)
    at org.springframework.beans.factory.support.DisposableBeanAdapter.destroy(DisposableBeanAdapter.java:253)
    at org.springframework.beans.factory.support.DefaultSingletonBeanRegistry.destroyBean(DefaultSingletonBeanRegistry.java:578)
    at org.springframework.beans.factory.support.DefaultSingletonBeanRegistry.destroySingleton(DefaultSingletonBeanRegistry.java:554)
    at org.springframework.beans.factory.support.DefaultListableBeanFactory.destroySingleton(DefaultListableBeanFactory.java:961)
    at org.springframework.beans.factory.support.DefaultSingletonBeanRegistry.destroySingletons(DefaultSingletonBeanRegistry.java:523)
    at org.springframework.beans.factory.support.DefaultListableBeanFactory.destroySingletons(DefaultListableBeanFactory.java:968)
    at org.springframework.context.support.AbstractApplicationContext.destroyBeans(AbstractApplicationContext.java:1033)
    at org.springframework.context.support.AbstractApplicationContext.refresh(AbstractApplicationContext.java:555)
    at org.springframework.boot.context.embedded.EmbeddedWebApplicationContext.refresh(EmbeddedWebApplicationContext.java:122)
    at org.springframework.boot.SpringApplication.refresh(SpringApplication.java:737)
    at org.springframework.boot.SpringApplication.refreshContext(SpringApplication.java:370)
    at org.springframework.boot.SpringApplication.run(SpringApplication.java:314)
    at org.springframework.boot.SpringApplication.run(SpringApplication.java:1162)
    at org.springframework.boot.SpringApplication.run(SpringApplication.java:1151)
    at cn.tisson.wechat.web.open.OpenApplication.main(OpenApplication.java:35)
    at sun.reflect.NativeMethodAccessorImpl.invoke0(Native Method)
    at sun.reflect.NativeMethodAccessorImpl.invoke(NativeMethodAccessorImpl.java:62)
    at sun.reflect.DelegatingMethodAccessorImpl.invoke(DelegatingMethodAccessorImpl.java:43)
    at java.lang.reflect.Method.invoke(Method.java:498)
    at org.springframework.boot.loader.MainMethodRunner.run(MainMethodRunner.java:48)
    at org.springframework.boot.loader.Launcher.launch(Launcher.java:87)
    at org.springframework.boot.loader.Launcher.launch(Launcher.java:50)
    at org.springframework.boot.loader.PropertiesLauncher.main(PropertiesLauncher.java:557)
[17-12-07 14:41:06 ERROR] main org.springframework.boot.SpringApplication.reportFailure(815)  | Application startup failed
org.springframework.beans.factory.UnsatisfiedDependencyException: Error creating bean with name 'methodValidationPostProcessor' defined in class path resource [org/springframework/boot/autoconfigure/validation/ValidationAutoConfiguration.class]: Unsatisfied dependency expressed through method 'methodValidationPostProcessor' parameter 0; nested exception is org.springframework.beans.factory.BeanCreationException: Error creating bean with name 'methodInvokingFactoryBean' defined in class path resource [cn/tisson/wechat/web/open/config/ShiroConfig.class]: Bean instantiation via factory method failed; nested exception is org.springframework.beans.BeanInstantiationException: Failed to instantiate [org.springframework.beans.factory.config.MethodInvokingFactoryBean]: Factory method 'methodInvokingFactoryBean' threw exception; nested exception is org.springframework.beans.factory.BeanCreationException: Error creating bean with name 'securityManager' defined in class path resource [cn/tisson/wechat/web/open/config/ShiroConfig.class]: Bean instantiation via factory method failed; nested exception is org.springframework.beans.BeanInstantiationException: Failed to instantiate [org.apache.shiro.mgt.SecurityManager]: Factory method 'securityManager' threw exception; nested exception is java.lang.IllegalStateException: Unable to acquire AES algorithm.  This is required to function.
    at org.springframework.beans.factory.support.ConstructorResolver.createArgumentArray(ConstructorResolver.java:749)
    at org.springframework.beans.factory.support.ConstructorResolver.instantiateUsingFactoryMethod(ConstructorResolver.java:467)
    at org.springframework.beans.factory.support.AbstractAutowireCapableBeanFactory.instantiateUsingFactoryMethod(AbstractAutowireCapableBeanFactory.java:1173)
    at org.springframework.beans.factory.support.AbstractAutowireCapableBeanFactory.createBeanInstance(AbstractAutowireCapableBeanFactory.java:1067)
    at org.springframework.beans.factory.support.AbstractAutowireCapableBeanFactory.doCreateBean(AbstractAutowireCapableBeanFactory.java:513)
    at org.springframework.beans.factory.support.AbstractAutowireCapableBeanFactory.createBean(AbstractAutowireCapableBeanFactory.java:483)
    at org.springframework.beans.factory.support.AbstractBeanFactory$1.getObject(AbstractBeanFactory.java:306)
    at org.springframework.beans.factory.support.DefaultSingletonBeanRegistry.getSingleton(DefaultSingletonBeanRegistry.java:230)
    at org.springframework.beans.factory.support.AbstractBeanFactory.doGetBean(AbstractBeanFactory.java:302)
    at org.springframework.beans.factory.support.AbstractBeanFactory.getBean(AbstractBeanFactory.java:202)
    at org.springframework.context.support.PostProcessorRegistrationDelegate.registerBeanPostProcessors(PostProcessorRegistrationDelegate.java:223)
    at org.springframework.context.support.AbstractApplicationContext.registerBeanPostProcessors(AbstractApplicationContext.java:702)
    at org.springframework.context.support.AbstractApplicationContext.refresh(AbstractApplicationContext.java:527)
    at org.springframework.boot.context.embedded.EmbeddedWebApplicationContext.refresh(EmbeddedWebApplicationContext.java:122)
    at org.springframework.boot.SpringApplication.refresh(SpringApplication.java:737)
    at org.springframework.boot.SpringApplication.refreshContext(SpringApplication.java:370)
    at org.springframework.boot.SpringApplication.run(SpringApplication.java:314)
    at org.springframework.boot.SpringApplication.run(SpringApplication.java:1162)
    at org.springframework.boot.SpringApplication.run(SpringApplication.java:1151)
    at cn.tisson.wechat.web.open.OpenApplication.main(OpenApplication.java:35)
    at sun.reflect.NativeMethodAccessorImpl.invoke0(Native Method)
    at sun.reflect.NativeMethodAccessorImpl.invoke(NativeMethodAccessorImpl.java:62)
    at sun.reflect.DelegatingMethodAccessorImpl.invoke(DelegatingMethodAccessorImpl.java:43)
    at java.lang.reflect.Method.invoke(Method.java:498)
    at org.springframework.boot.loader.MainMethodRunner.run(MainMethodRunner.java:48)
    at org.springframework.boot.loader.Launcher.launch(Launcher.java:87)
    at org.springframework.boot.loader.Launcher.launch(Launcher.java:50)
    at org.springframework.boot.loader.PropertiesLauncher.main(PropertiesLauncher.java:557)
Caused by: org.springframework.beans.factory.BeanCreationException: Error creating bean with name 'methodInvokingFactoryBean' defined in class path resource [cn/tisson/wechat/web/open/config/ShiroConfig.class]: Bean instantiation via factory method failed; nested exception is org.springframework.beans.BeanInstantiationException: Failed to instantiate [org.springframework.beans.factory.config.MethodInvokingFactoryBean]: Factory method 'methodInvokingFactoryBean' threw exception; nested exception is org.springframework.beans.factory.BeanCreationException: Error creating bean with name 'securityManager' defined in class path resource [cn/tisson/wechat/web/open/config/ShiroConfig.class]: Bean instantiation via factory method failed; nested exception is org.springframework.beans.BeanInstantiationException: Failed to instantiate [org.apache.shiro.mgt.SecurityManager]: Factory method 'securityManager' threw exception; nested exception is java.lang.IllegalStateException: Unable to acquire AES algorithm.  This is required to function.
    at org.springframework.beans.factory.support.ConstructorResolver.instantiateUsingFactoryMethod(ConstructorResolver.java:599)
    at org.springframework.beans.factory.support.AbstractAutowireCapableBeanFactory.instantiateUsingFactoryMethod(AbstractAutowireCapableBeanFactory.java:1173)
    at org.springframework.beans.factory.support.AbstractAutowireCapableBeanFactory.createBeanInstance(AbstractAutowireCapableBeanFactory.java:1067)
    at org.springframework.beans.factory.support.AbstractAutowireCapableBeanFactory.getSingletonFactoryBeanForTypeCheck(AbstractAutowireCapableBeanFactory.java:923)
    at org.springframework.beans.factory.support.AbstractAutowireCapableBeanFactory.getTypeForFactoryBean(AbstractAutowireCapableBeanFactory.java:804)
    at org.springframework.beans.factory.support.AbstractBeanFactory.isTypeMatch(AbstractBeanFactory.java:558)
    at org.springframework.beans.factory.support.DefaultListableBeanFactory.doGetBeanNamesForType(DefaultListableBeanFactory.java:432)
    at org.springframework.beans.factory.support.DefaultListableBeanFactory.getBeanNamesForType(DefaultListableBeanFactory.java:395)
    at org.springframework.beans.factory.BeanFactoryUtils.beanNamesForTypeIncludingAncestors(BeanFactoryUtils.java:220)
    at org.springframework.beans.factory.support.DefaultListableBeanFactory.findAutowireCandidates(DefaultListableBeanFactory.java:1260)
    at org.springframework.beans.factory.support.DefaultListableBeanFactory.doResolveDependency(DefaultListableBeanFactory.java:1101)
    at org.springframework.beans.factory.support.DefaultListableBeanFactory.resolveDependency(DefaultListableBeanFactory.java:1066)
    at org.springframework.beans.factory.support.ConstructorResolver.resolveAutowiredArgument(ConstructorResolver.java:835)
    at org.springframework.beans.factory.support.ConstructorResolver.createArgumentArray(ConstructorResolver.java:741)
    ... 27 common frames omitted
Caused by: org.springframework.beans.BeanInstantiationException: Failed to instantiate [org.springframework.beans.factory.config.MethodInvokingFactoryBean]: Factory method 'methodInvokingFactoryBean' threw exception; nested exception is org.springframework.beans.factory.BeanCreationException: Error creating bean with name 'securityManager' defined in class path resource [cn/tisson/wechat/web/open/config/ShiroConfig.class]: Bean instantiation via factory method failed; nested exception is org.springframework.beans.BeanInstantiationException: Failed to instantiate [org.apache.shiro.mgt.SecurityManager]: Factory method 'securityManager' threw exception; nested exception is java.lang.IllegalStateException: Unable to acquire AES algorithm.  This is required to function.
    at org.springframework.beans.factory.support.SimpleInstantiationStrategy.instantiate(SimpleInstantiationStrategy.java:189)
    at org.springframework.beans.factory.support.ConstructorResolver.instantiateUsingFactoryMethod(ConstructorResolver.java:588)
    ... 40 common frames omitted
Caused by: org.springframework.beans.factory.BeanCreationException: Error creating bean with name 'securityManager' defined in class path resource [cn/tisson/wechat/web/open/config/ShiroConfig.class]: Bean instantiation via factory method failed; nested exception is org.springframework.beans.BeanInstantiationException: Failed to instantiate [org.apache.shiro.mgt.SecurityManager]: Factory method 'securityManager' threw exception; nested exception is java.lang.IllegalStateException: Unable to acquire AES algorithm.  This is required to function.
    at org.springframework.beans.factory.support.ConstructorResolver.instantiateUsingFactoryMethod(ConstructorResolver.java:599)
    at org.springframework.beans.factory.support.AbstractAutowireCapableBeanFactory.instantiateUsingFactoryMethod(AbstractAutowireCapableBeanFactory.java:1173)
    at org.springframework.beans.factory.support.AbstractAutowireCapableBeanFactory.createBeanInstance(AbstractAutowireCapableBeanFactory.java:1067)
    at org.springframework.beans.factory.support.AbstractAutowireCapableBeanFactory.doCreateBean(AbstractAutowireCapableBeanFactory.java:513)
    at org.springframework.beans.factory.support.AbstractAutowireCapableBeanFactory.createBean(AbstractAutowireCapableBeanFactory.java:483)
    at org.springframework.beans.factory.support.AbstractBeanFactory$1.getObject(AbstractBeanFactory.java:306)
    at org.springframework.beans.factory.support.DefaultSingletonBeanRegistry.getSingleton(DefaultSingletonBeanRegistry.java:230)
    at org.springframework.beans.factory.support.AbstractBeanFactory.doGetBean(AbstractBeanFactory.java:302)
    at org.springframework.beans.factory.support.AbstractBeanFactory.getBean(AbstractBeanFactory.java:197)
    at org.springframework.context.annotation.ConfigurationClassEnhancer$BeanMethodInterceptor.obtainBeanInstanceFromFactory(ConfigurationClassEnhancer.java:389)
    at org.springframework.context.annotation.ConfigurationClassEnhancer$BeanMethodInterceptor.intercept(ConfigurationClassEnhancer.java:361)
    at cn.tisson.wechat.web.open.config.ShiroConfig$$EnhancerBySpringCGLIB$$d483728f.securityManager(<generated>)
    at cn.tisson.wechat.web.open.config.ShiroConfig.methodInvokingFactoryBean(ShiroConfig.java:119)
    at cn.tisson.wechat.web.open.config.ShiroConfig$$EnhancerBySpringCGLIB$$d483728f.CGLIB$methodInvokingFactoryBean$5(<generated>)
    at cn.tisson.wechat.web.open.config.ShiroConfig$$EnhancerBySpringCGLIB$$d483728f$$FastClassBySpringCGLIB$$29d8f88d.invoke(<generated>)
    at org.springframework.cglib.proxy.MethodProxy.invokeSuper(MethodProxy.java:228)
    at org.springframework.context.annotation.ConfigurationClassEnhancer$BeanMethodInterceptor.intercept(ConfigurationClassEnhancer.java:358)
    at cn.tisson.wechat.web.open.config.ShiroConfig$$EnhancerBySpringCGLIB$$d483728f.methodInvokingFactoryBean(<generated>)
    at sun.reflect.NativeMethodAccessorImpl.invoke0(Native Method)
    at sun.reflect.NativeMethodAccessorImpl.invoke(NativeMethodAccessorImpl.java:62)
    at sun.reflect.DelegatingMethodAccessorImpl.invoke(DelegatingMethodAccessorImpl.java:43)
    at java.lang.reflect.Method.invoke(Method.java:498)
    at org.springframework.beans.factory.support.SimpleInstantiationStrategy.instantiate(SimpleInstantiationStrategy.java:162)
    ... 41 common frames omitted
Caused by: org.springframework.beans.BeanInstantiationException: Failed to instantiate [org.apache.shiro.mgt.SecurityManager]: Factory method 'securityManager' threw exception; nested exception is java.lang.IllegalStateException: Unable to acquire AES algorithm.  This is required to function.
    at org.springframework.beans.factory.support.SimpleInstantiationStrategy.instantiate(SimpleInstantiationStrategy.java:189)
    at org.springframework.beans.factory.support.ConstructorResolver.instantiateUsingFactoryMethod(ConstructorResolver.java:588)
    ... 63 common frames omitted
Caused by: java.lang.IllegalStateException: Unable to acquire AES algorithm.  This is required to function.
    at org.apache.shiro.crypto.AbstractSymmetricCipherService.generateNewKey(AbstractSymmetricCipherService.java:59)
    at org.apache.shiro.crypto.AbstractSymmetricCipherService.generateNewKey(AbstractSymmetricCipherService.java:43)
    at org.apache.shiro.mgt.AbstractRememberMeManager.<init>(AbstractRememberMeManager.java:99)
    at org.apache.shiro.web.mgt.CookieRememberMeManager.<init>(CookieRememberMeManager.java:87)
    at org.apache.shiro.web.mgt.DefaultWebSecurityManager.<init>(DefaultWebSecurityManager.java:75)
    at cn.tisson.wechat.web.open.config.ShiroConfig.securityManager(ShiroConfig.java:88)
    at cn.tisson.wechat.web.open.config.ShiroConfig$$EnhancerBySpringCGLIB$$d483728f.CGLIB$securityManager$0(<generated>)
    at cn.tisson.wechat.web.open.config.ShiroConfig$$EnhancerBySpringCGLIB$$d483728f$$FastClassBySpringCGLIB$$29d8f88d.invoke(<generated>)
    at org.springframework.cglib.proxy.MethodProxy.invokeSuper(MethodProxy.java:228)
    at org.springframework.context.annotation.ConfigurationClassEnhancer$BeanMethodInterceptor.intercept(ConfigurationClassEnhancer.java:358)
    at cn.tisson.wechat.web.open.config.ShiroConfig$$EnhancerBySpringCGLIB$$d483728f.securityManager(<generated>)
    at sun.reflect.NativeMethodAccessorImpl.invoke0(Native Method)
    at sun.reflect.NativeMethodAccessorImpl.invoke(NativeMethodAccessorImpl.java:62)
    at sun.reflect.DelegatingMethodAccessorImpl.invoke(DelegatingMethodAccessorImpl.java:43)
    at java.lang.reflect.Method.invoke(Method.java:498)
    at org.springframework.beans.factory.support.SimpleInstantiationStrategy.instantiate(SimpleInstantiationStrategy.java:162)
    ... 64 common frames omitted
Caused by: java.security.NoSuchAlgorithmException: AES KeyGenerator not available
    at javax.crypto.KeyGenerator.<init>(KeyGenerator.java:169)
    at javax.crypto.KeyGenerator.getInstance(KeyGenerator.java:223)
    at org.apache.shiro.crypto.AbstractSymmetricCipherService.generateNewKey(AbstractSymmetricCipherService.java:56)
    ... 79 common frames omitted
```


分析原因
```txt
https://stackoverflow.com/questions/41295301/java-security-nosuchalgorithmexception-aes-keygenerator-not-available-while-dep

Every implementation of Java is required to support a few standard algorithms like AES or DES. This is stated in the documentation of KeyGenerator. So you probably have a problem with your Java environment setup.

In oracle's java implementation the algorithm classes should be located in `sunjce_provider.jar` (at least with version 1.7 and 1.8), which is usually located under $JAVA_HOME/jre/lib/ext.

A common failure is, that this directory is not in your classpath, which might happen, when you explicitly define an extension-dir via

java -Djava.ext.dirs=/my/other/dir <more arguments...>
If you specify an extension-dir this way, you should also include $JAVA_HOME/jre/lib/ext (and make sure JAVA_HOME is set correctly):

java -Djava.ext.dirs=/my/other/dir:$JAVA_HOME/jre/lib/ext  <more arguments...>
In JBoss/Wildfy this is usually done in the configuration file bin/standalone.conf (or bin/run.conf in older versions). Details about java extensions and their configuration can be found here.
```

正确的启动脚本
```shell
/usr/java/jdk1.8.0_144/bin/java -server -Djava.ext.dirs=/opt/tisson-web-open/lib/:/usr/java/jdk1.8.0_144/jre/lib/ext/ -Dspring.profiles.active=local -jar tisson-web-open.jar
```
或者
```shell
/usr/java/jdk1.8.0_144/bin/java -server -Dloader.path=/opt/tisson-web-open/lib/ -Dspring.profiles.active=local -jar tisson-web-open.jar
```



> 重点：-Djava.ext.dirs 和 -Dloader.path的区别