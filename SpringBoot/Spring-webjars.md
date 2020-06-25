# SpringBoot webjars 和静态资源映射



## webjars

通过jar包的方式打包web资源，例如js，css等文件。可以通过maven的pom.xml方式直接引入静态资源。



## 静态资源映射

默认在下面五个路径下的资源，都可以作为静态资源访问

- classpath:/META-INF/resources/  
- classpath:/resources/
- classpath:/static/
- classpath:/public/
- / : 当前项目的根路径



如果在application.porperties文件中，配置了spring.resources.static-locations属性，则会覆盖SpringBoot默认静态资源映射地址

```properties
spring.resources.static-locations = classpath:/dist/,classpath:/hehe/
```



## SpringBoot 静态资源默认配置

SpringBoot的静态资源映射是通过SpringMVC的自动配置类（WebMvcAutoConfiguration）配置的。

```java
@Override
		public void addResourceHandlers(ResourceHandlerRegistry registry) {
			if (!this.resourceProperties.isAddMappings()) {
				logger.debug("Default resource handling disabled");
				return;
			}
			Duration cachePeriod = this.resourceProperties.getCache().getPeriod();
			CacheControl cacheControl = this.resourceProperties.getCache()
					.getCachecontrol().toHttpCacheControl();
			if (!registry.hasMappingForPattern("/webjars/**")) {
				customizeResourceHandlerRegistration(registry
						.addResourceHandler("/webjars/**")
						.addResourceLocations("classpath:/META-INF/resources/webjars/")
						.setCachePeriod(getSeconds(cachePeriod))
						.setCacheControl(cacheControl));
			}
			String staticPathPattern = this.mvcProperties.getStaticPathPattern();
			if (!registry.hasMappingForPattern(staticPathPattern)) {
				customizeResourceHandlerRegistration(
						registry.addResourceHandler(staticPathPattern)
								.addResourceLocations(getResourceLocations(
										this.resourceProperties.getStaticLocations()))
								.setCachePeriod(getSeconds(cachePeriod))
								.setCacheControl(cacheControl));
			}
		}      
```

默认的静态资源路径为resourceProperties.getStaticLocations()即：

```java
private static final String[] CLASSPATH_RESOURCE_LOCATIONS = {
			"classpath:/META-INF/resources/", "classpath:/resources/",
			"classpath:/static/", "classpath:/public/" };
```



