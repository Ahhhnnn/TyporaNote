# Spring -AOP

在程序运行期间，能够在程序运行时动态的将某段代码切入到指定方法指定位置进行执行（动态代理）

## Aop常用注解

@Before 前置处理 **在目标方法执行前执行**

@Afrer 后置处理	**在目标方法执行后执行**

@AfterReturning 返回处理  **在目标方法返回时执行**

@AfterThrowing 抛异常时处理  **在目标方法抛异常执行**

@Around 环绕处理  **动态代理，手动推进目标方法执行 （point.proceed()）**

@Pointcut 切入点，**指定切入点，支持表达式（execution(public * com.xkcoding.log.aop.controller.*Controller.*(..))）**

@Aspect 申明为切面类

**执行顺序**

@Around  -> @Before -> @Afrer ->@AfterReturning or @AfterThrowing 

## SpringBoot 引入AOP

引入Spring-Boot-starter-aop

```java
<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-aop</artifactId>
</dependency>
```

### 创建切面类

```java
@Aspect
@Component
@Slf4j
public class AopLog {
	private static final String START_TIME = "request-start";

	/**
	 * 切入点
	 */
	@Pointcut("execution(public * com.xkcoding.log.aop.controller.*Controller.*(..))")
	public void log() {

	}

	/**
	 * 前置操作
	 *
	 * @param point 切入点
	 */
	@Before("log()")
	public void beforeLog(JoinPoint point) {
		/*ServletRequestAttributes attributes = (ServletRequestAttributes) RequestContextHolder.getRequestAttributes();

		HttpServletRequest request = Objects.requireNonNull(attributes).getRequest();

		log.info("【请求 URL】：{}", request.getRequestURL());
		log.info("【请求 IP】：{}", request.getRemoteAddr());
		log.info("【请求类名】：{}，【请求方法名】：{}", point.getSignature().getDeclaringTypeName(), point.getSignature().getName());

		Map<String, String[]> parameterMap = request.getParameterMap();
		log.info("【请求参数】：{}，", JSONUtil.toJsonStr(parameterMap));
		Long start = System.currentTimeMillis();
		request.setAttribute(START_TIME, start);*/
		log.error("Before");
	}

	/**
	 * 环绕操作
	 *
	 * @param point 切入点
	 * @return 原方法返回值
	 * @throws Throwable 异常信息
	 */
	@Around("log()")
	public Object aroundLog(ProceedingJoinPoint point) throws Throwable {
		/*Object result = point.proceed();
		log.info("【返回值】：{}", JSONUtil.toJsonStr(result));
		return result;*/
		log.error("Around");
		return point.proceed();
	}

	/**
	 * 后置操作
	 */
	@AfterReturning("log()")
	public void afterReturning() {
		/*ServletRequestAttributes attributes = (ServletRequestAttributes) RequestContextHolder.getRequestAttributes();
		HttpServletRequest request = Objects.requireNonNull(attributes).getRequest();

		Long start = (Long) request.getAttribute(START_TIME);
		Long end = System.currentTimeMillis();
		log.info("【请求耗时】：{}毫秒", end - start);

		String header = request.getHeader("User-Agent");
		UserAgent userAgent = UserAgent.parseUserAgentString(header);
		log.info("【浏览器类型】：{}，【操作系统】：{}，【原始User-Agent】：{}", userAgent.getBrowser().toString(), userAgent.getOperatingSystem().toString(), header);*/
	    log.error("afterReturning");
	}

	@After(value ="log()")
    public void After(){
	    log.error("After");
    }

    @AfterThrowing(value = "log()" , throwing = "e")
    public void AfterThrowing(JoinPoint point,Exception e){
	    log.error("AfterThrowing");
    }
}
```



## Spring 开启Aop

非SpringBoot项目开启Aop需要在配置类中显示开启AOP

使用**@EnableAspectJAutoProxy**注解开启Aop