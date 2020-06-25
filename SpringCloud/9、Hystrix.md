## Hystrix

### Hystrix是什么

#### **分布式系统面临的问题**

复杂分布式体系结构中的应用程序有数十个依赖关系，每个依赖关系在某些时候将不可避免地失败。 服务雪崩 多个微服务之间调用的时候，假设微服务A调用微服务B和微服务C，微服务B和微服务C又调用其它的微服务，这就是所谓的“扇出”。如果扇出的链路上某个微服务的调用响应时间过长或者不可用，对微服务A的调用就会占用越来越多的系统资源，进而引起系统崩溃，所谓的“雪崩效应”.  对于高流量的应用来说，单一的后端依赖可能会导致所有服务器上的所有资源都在几秒钟内饱和。比失败更糟糕的是，这些应用程序还可能导致服务之间的延迟增加，备份队列，线程和其他系统资源紧张，导致整个系统发生更多的级联故障。这些都表示需要对故障和延迟进行隔离和管理，以便单个依赖关系的失败，不能取消整个应用程序或系统。   

备注：一般情况对于服务依赖的保护主要有3中解决方案：  

（1）熔断模式：这种模式主要是参考电路熔断，如果一条线路电压过高，保险丝会熔断，防止火灾。放到我们的系统中，如果某个目标服务调用慢或者有大量超时，此时，熔断该服务的调用，对于后续调用请求，不在继续调用目标服务，直接返回，快速释放资源。如果目标服务情况好转则恢复调用。  

（2）隔离模式：这种模式就像对系统请求按类型划分成一个个小岛的一样，当某个小岛被火少光了，不会影响到其他的小岛。例如可以对不同类型的请求使用线程池来资源隔离，每种类型的请求互不影响，如果一种类型的请求线程资源耗尽，则对后续的该类型请求直接返回，不再调用后续资源。这种模式使用场景非常多，例如将一个服务拆开，对于重要的服务使用单独服务器来部署，再或者公司最近推广的多中心。  

（3）限流模式：上述的熔断模式和隔离模式都属于出错后的容错处理机制，而限流模式则可以称为预防模式。限流模式主要是提前对各个类型的请求设置最高的QPS阈值，若高于设置的阈值则对该请求直接返回，不再调用后续资源。这种模式不能解决服务依赖的问题，只能解决系统整体资源分配问题，因为没有被限流的请求依然有可能造成雪崩效应。



**Hystrix**是一个用于处理分布式系统的延迟和容错的开源库，在分布式系统里，许多依赖不可避免的会调用失败，比如超时、异常等，Hystrix能够保证在一个依赖出问题的情况下，不会导致整体服务失败，避免级联故障，以提高分布式系统的弹性。  “断路器”本身是一种开关装置，当某个服务单元发生故障之后，通过断路器的故障监控（类似熔断保险丝），向调用方返回一个符合预期的、可处理的备选响应（FallBack），而不是长时间的等待或者抛出调用方无法处理的异常，这样就保证了服务调用方的线程不会被长时间、不必要地占用，从而避免了故障在分布式系统中的蔓延，乃至雪崩。 	

### **Hystrix**能干什么

服务降级、服务熔断、服务限流、接近实时的监控。



## 服务降级Fallback

 当服务器压力剧增的时候,根据当前业务情况以及流量，对一些服务和页面有策略的降级，以此缓解服务器资源的压力以保障核心任务的正常运行，同时也保证了大部分客户能得到正常的响应。不让客户端等待，立即返回一个友好的提示。

### 什么情况发出降级

- 程序运行异常
- 超时
- 服务熔断触发服务降级
- 线程池、信号量打满也会导致服务降级

### 降级策略

- 服务接口拒绝服务：页面能访问，但是添加删除提示服务器繁忙。页面内容也可在Varnish或CDN内获取。

- 页面拒绝服务：页面提示由于服务繁忙此服务暂停。跳转到varnish或nginx的一个静态页面。

- 延迟持久化：页面访问照常，但是涉及记录变更，会提示稍晚能看到结果，将数据记录到异步队列或log，服务恢复后执行。

- 随机拒绝服务：服务接口随机拒绝服务，让用户重试，目前较少有人采用。因为用户体验不佳。

比如以前服务一秒钟能接受100个请求，当服务压力增大时，进行服务降级，我只处理50个请求/s。

## 服务熔断Break

服务熔断 熔断机制是应对雪崩效应的一种微服务链路保护机制。 当扇出链路的某个微服务不可用或者响应时间太长时，会进行服务的降级，进而熔断该节点微服务的调用，快速返回"错误"的响应信息。当检测到该节点微服务调用响应正常后恢复调用链路。在SpringCloud框架里熔断机制通过Hystrix实现。Hystrix会监控微服务间调用的状况，当失败的调用到一定阈值，缺省是5秒内20次调用失败就会启动熔断机制。熔断机制的注解是@HystrixCommand。

 =====处理异常业务逻辑=====provider client处理====

**处理流程**

服务的降级->进行熔断->恢复调用链路

**熔断指的是熔断调用的服务，如果调用的服务响应慢或者大量超时，熔断该服务的调用，对于后续调用请求，不在继续调用目标服务，直接返回，快速释放资源如果目标服务情况好转则恢复调用。**

## 服务限流Flowlimit

秒杀高并发等操作，严禁一窝蜂的过来拥挤，请求进行排队，一秒钟只处理N个，有序进行。防止高并发将服务器打死。



## 服务降级和服务熔断异同

#### 共性

- 目的: 目的一致，都是从系统的可用性、可靠性着想。放了防止系统的整体缓慢甚至奔溃而采用的技术手段。

- 最终表现: 表现类似,最终都是给用户一种当前服务不可用或者不可达的感觉

- 粒度: 大多都是在服务级别，当然也有一些在持久层层面的应用

- 自治: 基本都是靠系统达到某一临界条件时，实现自动的降级与熔断，人工降级并不是那么稳妥。



#### 区别

- 触发原因: 服务熔断一般指某个服务的下游服务出现问题时采用的手段,而服务降级一般是从整体层面考虑的。

- 管理目标层次: 熔断是一种框架级的处理，每一个微服务都需要。而降级一般需要对业务有层级之分，降级一般都是从外围服务开始的。

- 实现方式: 代码级别实现有差异



## Hystrix案列

**Hystrix在实际中一般用在消费端，当然同样可以用于服务提供方。**

**依赖**

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <parent>
        <artifactId>cloud2020</artifactId>
        <groupId>com.atguigu.springcloud</groupId>
        <version>1.0-SNAPSHOT</version>
    </parent>
    <modelVersion>4.0.0</modelVersion>

    <artifactId>cloud-provider-hystrix-payment8001</artifactId>

    <dependencies>
        <!--hystrix-->
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-netflix-hystrix</artifactId>
        </dependency>
        <!--eureka client-->
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
        </dependency>
        <dependency>
            <groupId>com.atguigu.springcloud</groupId>
            <artifactId>cloud-api-common</artifactId>
            <version>${project.version}</version>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
        <!--监控-->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-actuator</artifactId>
        </dependency>
        <!--热部署-->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-devtools</artifactId>
            <scope>runtime</scope>
            <optional>true</optional>
        </dependency>
        <dependency>
            <groupId>org.projectlombok</groupId>
            <artifactId>lombok</artifactId>
            <optional>true</optional>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
        </dependency>
    </dependencies>

</project>
```



配置文件

```yaml
server:
  port: 8001
spring:
  application:
    name: cloud-provider-hystrix-payment
eureka:
  client:
    register-with-eureka: true
    fetch-registry: true
    service-url:
      defaultZone: http://eureka7001.com:7001/eureka,http://eureka7002.com:7002/eureka
```



## Jmeter 压力测试

当一个服务内一个接口被大量请求，还没有处理完成时，同服务内的其他接口也会被被拖累，导致访问很慢。

同服务内会使用大量资源去处理高并发的请求。因此本来很快的接口也会被拖累。



## 新建Hystix消费端

使用openfeign 和 Hystix

pom 依赖

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <parent>
        <artifactId>cloud2020</artifactId>
        <groupId>com.atguigu.springcloud</groupId>
        <version>1.0-SNAPSHOT</version>
    </parent>
    <modelVersion>4.0.0</modelVersion>

    <artifactId>cloud-consumer-feign-hystrix-order80</artifactId>

    <dependencies>
        <!--hystrix-->
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-netflix-hystrix</artifactId>
        </dependency>
        <!--openfeign-->
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-openfeign</artifactId>
        </dependency>
        <!--eureka client-->
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
        </dependency>
        <dependency>
            <groupId>com.atguigu.springcloud</groupId>
            <artifactId>cloud-api-common</artifactId>
            <version>${project.version}</version>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
        <!--监控-->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-actuator</artifactId>
        </dependency>
        <!--热部署-->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-devtools</artifactId>
            <scope>runtime</scope>
            <optional>true</optional>
        </dependency>
        <dependency>
            <groupId>org.projectlombok</groupId>
            <artifactId>lombok</artifactId>
            <optional>true</optional>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
        </dependency>
    </dependencies>

</project>
```

配置文件

```yaml
server:
  port: 80
eureka:
  client:
    register-with-eureka: false
    fetch-registry: true
    service-url:
      defaultZone: http://eureka7001.com:7001/eureka,http://eureka7002.com:7002/eureka
feign:
  hystrix:
    # 在feign中开启Hystrix
    enabled: true
```

主启动类

```java
@SpringBootApplication
@EnableEurekaClient
@EnableFeignClients
@EnableHystrix
public class OrderHystrixMain80 {
    public static void main(String[] args) {
        SpringApplication.run(OrderHystrixMain80.class, args);
    }
}
```

feign 接口 调用服务端方法

**备注**：fallback参数表示

```java
@Component
@FeignClient(value = "CLOUD-PROVIDER-HYSTRIX-PAYMENT",fallback = PaymentFallbackService.class)
public interface PaymentHystrixService {


    /**
     * 正常访问
     * @param id
     * @return
     */
    @GetMapping("/payment/hystrix/ok/{id}")
    String paymentInfo_OK(@PathVariable("id") Integer id);

    /**
     * 超时访问
     *
     * @param id
     * @return
     */
    @GetMapping("/payment/hystrix/timeout/{id}")
    String paymentInfo_TimeOut(@PathVariable("id") Integer id);

}
```

当服务提供方8001被大量请求访问时，导致服务内其他接口访问变慢，这个时候服务消费方（80端口），去访问服务提供方（8001）同样也会被拖累导致变慢。（因为服务提供方tomcat线程池中的工作线程已经被消耗完毕）

所以需要进行降级、容错、限流



### 解决服务降级等要求

- 超时导致服务器变慢 -超时不在等待，给出一个友好提示
- 出错（宕机或程序运行出错） -出错要有一个兜底的方法



## 处理服务降级

降级配置 使用注解 **@HystrixCommand**



## 服务提供方服务降级

设置自身调用超时时间的峰值，峰值内可以正常运行，超过了需要有兜底的方法处理，做服务降级fallback



在需要服务降级的方法使用**@HystrixCommand**，参数fallbackMethod指定兜底的方法，使用**commandProperties**属性设置哪些情况会进行调用兜底的方法。

服务器启动类增加注解**@EnableCircuitBreaker**

```java
@SpringBootApplication
@EnableEurekaClient
@EnableCircuitBreaker
public class PaymentHystrixMain8001 {
    public static void main(String[] args) {
        SpringApplication.run(PaymentHystrixMain8001.class, args);
    }
}
```

```java
/**
 * 超时访问
 * HystrixCommand:一旦调用服务方法失败并抛出了错误信息后,会自动调用@HystrixCommand标注好的fallbckMethod调用类中的指定方法
 * execution.isolation.thread.timeoutInMilliseconds:线程超时时间3秒钟
 *
 * @param id
 * @return
 */
@HystrixCommand(fallbackMethod = "payment_TimeOutHandler", commandProperties = {
        @HystrixProperty(name = "execution.isolation.thread.timeoutInMilliseconds", value = "3000")
})
public String paymentInfo_TimeOut(Integer id) {
    ///int age = 10 / 0;

    int timeNumber = 5;
    try {
        // 暂停5秒钟
        TimeUnit.SECONDS.sleep(timeNumber);
    } catch (InterruptedException e) {
        e.printStackTrace();
    }
    return "线程池:" + Thread.currentThread().getName() + " paymentInfo_TimeOut,id:" + id + "\t" +
            "O(∩_∩)O哈哈~  耗时(秒)" + timeNumber;
}

/**
     * 兜底方案  超过3s则进入兜底方案
     *
     * @param id
     * @return 
     */
    public String payment_TimeOutHandler(Integer id) {
        return "线程池:" + Thread.currentThread().getName() + " 系统繁忙或运行错误,请稍后重试,id:" + id + "\t" + "o(╥﹏╥)o";
    }
```

## 服务调用方服务降级

配置文件

```yaml
server:
  port: 80
eureka:
  client:
    register-with-eureka: false
    fetch-registry: true
    service-url:
      defaultZone: http://eureka7001.com:7001/eureka,http://eureka7002.com:7002/eureka
feign:
  hystrix:
    # 在feign中开启Hystrix
    enabled: true
```

使用@HystrixCommand注解客户端设置服务降级

```java
@HystrixCommand(fallbackMethod = "paymentTimeOutFallbackMethod", commandProperties = {
            @HystrixProperty(name = "execution.isolation.thread.timeoutInMilliseconds", value = "1500")
    })

/**
     * 超时方法fallback
     * @param id
     * @return
     */
    public String paymentTimeOutFallbackMethod(@PathVariable("id") Integer id) {
        return "我是消费者80,对方支付系统繁忙请10秒种后再试或者自己运行出错请检查自己,o(╥﹏╥)o";
    }
```

**当服务提供方的接口响应超过1.5秒，则客户端不在等待，直接调用兜底的方法给用户一个提示。**



### 全局服务降级兜底方法

```java
/**
     * 全局fallback
     *
     * @return
     */
public String payment_Global_FallbackMethod() {
    return "Global异常处理信息,请稍后重试.o(╥﹏╥)o";
}
```

在Controller类上使用注解**@DefaultProperties**,避免代码膨胀和冗余，设置通用的服务降级方法

```java
@DefaultProperties(defaultFallback = "payment_Global_FallbackMethod")
```



### 服务降级FeignFallback(避免代码冗余)

在客户端新建一个类，实现Feign接口（远程服务调用接口），统一为里面的方法进行异常处理。

在Feign接口的**@FeignClient**注解添加fallback参数，指定统一处理异常的类。（即实现了当前接口的类）

**Feign接口**

```java
@Component
@FeignClient(value = "CLOUD-PROVIDER-HYSTRIX-PAYMENT",fallback = PaymentFallbackService.class)
public interface PaymentHystrixService {


    /**
     * 正常访问
     * @param id
     * @return
     */
    @GetMapping("/payment/hystrix/ok/{id}")
    String paymentInfo_OK(@PathVariable("id") Integer id);

    /**
     * 超时访问
     *
     * @param id
     * @return
     */
    @GetMapping("/payment/hystrix/timeout/{id}")
    String paymentInfo_TimeOut(@PathVariable("id") Integer id);

}
```

**服务异常处理类**

```java
@Component
public class PaymentFallbackService implements PaymentHystrixService {
    @Override
    public String paymentInfo_OK(Integer id) {
        return "----PaymentFallbackService fall back-paymentInfo_OK,o(╥﹏╥)o";
    }

    @Override
    public String paymentInfo_TimeOut(Integer id) {
        return "----PaymentFallbackService fall back-paymentInfo_TimeOut,o(╥﹏╥)o";
    }
}
```

当服务提供方突然宕机了，那么**Feign接口**在注册中心找不到对应的微服务（**CLOUD-PROVIDER-HYSTRIX-PAYMENT**），那么就会走入服务异常处理类，找到调用接口对应的异常处理接口。





## 服务熔断处理

当访问一个服务接口错误次数超过一定次数时，将进行服务的熔断，断路器将起作用，直接不请求接口了，直接访问一个提示。

```java
//====服务熔断

    /**
     * 在10秒窗口期中10次请求有6次是请求失败的,断路器将起作用
     * @param id
     * @return
     */
    @HystrixCommand(
            fallbackMethod = "paymentCircuitBreaker_fallback", commandProperties = {
            @HystrixProperty(name = "circuitBreaker.enabled", value = "true"),// 是否开启断路器
            @HystrixProperty(name = "circuitBreaker.requestVolumeThreshold", value = "10"),// 请求次数
            @HystrixProperty(name = "circuitBreaker.sleepWindowInMilliseconds", value = "10000"),// 时间窗口期/时间范文
            @HystrixProperty(name = "circuitBreaker.errorThresholdPercentage", value = "60")// 失败率达到多少后跳闸
    }
    )
    public String paymentCircuitBreaker(@PathVariable("id") Integer id) {
        if (id < 0) {
            throw new RuntimeException("*****id不能是负数");
        }
        String serialNumber = IdUtil.simpleUUID();
        return Thread.currentThread().getName() + "\t" + "调用成功,流水号:" + serialNumber;
    }

    public String paymentCircuitBreaker_fallback(@PathVariable("id") Integer id) {
        return "id 不能负数,请稍后重试,o(╥﹏╥)o id:" + id;
    }
```

