# Http调用

## 读取超时参数和读取超时则会有更多的误区

### 第一个误区：认为出现了读取超时，服务端的执行就会中断

读取超时，服务端会一直将请求执行完毕，并不会中断。

### 第二个误区：认为读取超时只是 Socket 网络层面的概念，是数据传输的最长耗时，故将其配置得非常短，比如 100 毫秒

确切地说，读取超时指的是，向 Socket 写入数据后，我们等到 Socket 返回数据的超时时间，其中包含的时间或者说绝大部分的时间，是服务端处理业务逻辑的时间。

### 第三个误区：认为超时时间越长任务接口成功率就越高，将读取超时参数配置得太长

进行 HTTP 请求一般是需要获得结果的，属于同步调用。如果超时时间很长，在等待服务端返回数据的同时，客户端线程（通常是 Tomcat 线程）也在等待，当下游服务出现大量超时的时候，程序可能也会受到拖累创建大量线程，最终崩溃。对定时任务或异步任务来说，读取超时配置得长些问题不大。但面向用户响应的请求或是微服务短平快的同步接口调用，并发量一般较大，我们应该设置一个较短的读取超时时间，以防止被下游服务拖慢，通常不会设置超过 30 秒的读取超时。

## Feign 和 Ribbon的调用

### Feign  和 Ribbon 配置超时



## 并发限制了爬虫的抓取能力

除了超时和重试的坑，进行 HTTP 请求调用还有一个常见的问题是，并发数的限制导致程序的处理能力上不去

一般Http客户端都会对同一个域名发起的请求进行并发限制。

创建HttpClient，http客户端 PoolingHttpClientConnectionManager

```java
static CloseableHttpClient httpClient1;static { httpClient1 = HttpClients.custom().setConnectionManager(new PoolingHttpClientConnectionManager()).build();}@GetMapping("wrong")public int wrong(@RequestParam(value = "count", defaultValue = "10") int count) throws InterruptedException { return sendRequest(count, () -> httpClient1);}
```

PoolingHttpClientConnectionManager源码

```java
public PoolingHttpClientConnectionManager( 
    final HttpClientConnectionOperator httpClientConnectionOperator, 
    final HttpConnectionFactory connFactory, 
    final long timeToLive, 
    final TimeUnit timeUnit) 
{ ... this.pool = new CPool(new InternalConnectionFactory( this.configData, connFactory), 2, 20, timeToLive, timeUnit);
 ...} 
public CPool( final ConnFactory connFactory, 
             final int defaultMaxPerRoute, 
             final int maxTotal, final long timeToLive, 
             final TimeUnit timeUnit) { 
    ...}
}
```

可以发现限制了defaultMaxPerRoute=2，也就是同一个主机 / 域名的最大并发请求数为 2。我们的爬虫需要 10 个并发（开启线程池，开启10个线程同时请求），显然是默认值太小限制了爬虫的效率。

maxTotal=20，也就是所有主机整体最大并发为 20，这也是 HttpClient 整体的并发度。目前，我们请求数是 10 最大并发是 10，20 不会成为瓶颈。举一个例子，使用同一个 HttpClient 访问 10 个域名，defaultMaxPerRoute 设置为 10，为确保每一个域名都能达到 10 并发，需要把 maxTotal 设置为 100

**很多早期的浏览器也限制了同一个域名两个并发请求。对于同一个域名并发连接的限制，其实是 HTTP 1.1 协议要求的**

尝试声明一个新的 HttpClient 放开相关限制，设置 maxPerRoute 为 50、maxTotal 为 100

```java

httpClient2 = HttpClients.custom().setMaxConnPerRoute(10).setMaxConnTotal(20).build();
```



## 重点回顾

连接超时代表建立 TCP 连接的时间，读取超时代表了等待远端返回数据的时间，也包括远端程序处理的时间。在解决连接超时问题时，我们要搞清楚连的是谁；在遇到读取超时问题的时候，我们要综合考虑下游服务的服务标准和自己的服务标准，设置合适的读取超时时间。此外，在使用诸如 Spring Cloud Feign 等框架时务必确认，连接和读取超时参数的配置是否正确生效。

对于重试，因为 HTTP 协议认为 Get 请求是数据查询操作，是无状态的，又考虑到网络出现丢包是比较常见的事情，有些 HTTP 客户端或代理服务器会自动重试 Get/Head 请求。如果你的接口设计不支持幂等，需要关闭自动重试。但，更好的解决方案是，遵从 HTTP 协议的建议来使用合适的 HTTP 方法。

最后我们看到，包括 HttpClient 在内的 HTTP 客户端以及浏览器，都会限制客户端调用的最大并发数。如果你的客户端有比较大的请求调用并发，比如做爬虫，或是扮演类似代理的角色，又或者是程序本身并发较高，如此小的默认值很容易成为吞吐量的瓶颈，需要及时调整。



## 问题

### 一、为何不常见写入超时

客户端发送数据到服务端，首先接力连接（TCP），然后写入TCP缓冲区，TCP缓冲区根据时间窗口，发送数据到服务端，因此写入操作可以任务是自己本地的操作，本地操作是不需要什么超时时间的，如果真的有什么异常，那也是连接（TCP）不上，或者超时的问题，连接超时和读取超时就能覆盖这种场景

### 二、nginx的重试机制

proxy_next_upstream：语法: proxy_next_upstream
   [error|timeout|invalid_header|http_500|http_503|http_404|off]
   默认值: proxy_next_upstream error timeout
   即 error timeout会自动重试
可以修改默认值，在去掉error和timeout，这样在发生错误和超时时，不会重试
proxy_next_upstream_tries 这个参数决定重试的次数，0表示关闭该参数
Limits the number of possible tries for passing a request to the next server. The 0 value turns off this limitation.