---
title: spring redirect 导致内存溢出问题核查
categories:
  - Java
tags:
  - spring
  - redirect
abbrlink: b575301a
date: 2018-08-20 11:43:05
---



# 3.x 版本

这个类AbstractCachingViewResolver里面的viewCache HashMap没有限制大小 

3.x：如果在 controller 返回的 view 是不固定的，如："redirect:form.html?entityId=" + entityId，由于 entityId 的值会存在 N 个，那么会导致产生 N 个 ViewName 被缓存起来



<!-- more -->



> 4.x 版本及以上已经修复这个问题，代码如下

```java
public abstract class AbstractCachingViewResolver extends WebApplicationObjectSupport implements ViewResolver {

	/** Default maximum number of entries for the view cache: 1024 */
	public static final int DEFAULT_CACHE_LIMIT = 1024;

	/** Dummy marker object for unresolved views in the cache Maps */
	private static final View UNRESOLVED_VIEW = new View() {
		@Override
		public String getContentType() {
			return null;
		}
		@Override
		public void render(Map<String, ?> model, HttpServletRequest request, HttpServletResponse response) {
		}
	};


	/** The maximum number of entries in the cache */
	private volatile int cacheLimit = DEFAULT_CACHE_LIMIT;

	/** Whether we should refrain from resolving views again if unresolved once */
	private boolean cacheUnresolved = true;

	/** Fast access cache for Views, returning already cached instances without a global lock */
	private final Map<Object, View> viewAccessCache = new ConcurrentHashMap<Object, View>(DEFAULT_CACHE_LIMIT);

	/** Map from view key to View instance, synchronized for View creation */
	@SuppressWarnings("serial")
	private final Map<Object, View> viewCreationCache =
			new LinkedHashMap<Object, View>(DEFAULT_CACHE_LIMIT, 0.75f, true) {
				@Override
				protected boolean removeEldestEntry(Map.Entry<Object, View> eldest) {
					if (size() > getCacheLimit()) {
                        // 超过限制大小，删除老数据
						viewAccessCache.remove(eldest.getKey());
						return true;
					}
					else {
						return false;
					}
				}
			};
    ......
```



# 4.x 版本

4.x 版本是这个AbstractAutoProxyCreator这个类里面的advisedBeans ConcurrentHashMap没有限制大小

问题描述：https://github.com/spring-projects/spring-boot/issues/13771

The problem is that `InternalResourceViewResolver` calls `AutowireCapableBeanFactory.initializeBean(Object existingBean, String beanName)` using the complete URL as the `beanName`. The bean name is then used by `AbstractAutoProxyCreator.wrapIfNecessary(Object, String, Object)` as the key when it caches the fact that no advice applies to it. 

> 截止发稿：最新的 spring 5.0.8.RELEASE 还存在这个问题